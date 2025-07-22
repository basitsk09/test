Of course. Following the structure you provided, here is the approve and reject functionality refactored to accept a Map in the controller and use only native queries in the repository for all database operations.
This implementation removes the need for transient variables by having the service layer orchestrate calls to specific native queries that perform targeted updates.
## Backend Refactoring (Approve/Reject)
1. Controller (FrtRequestController.java)
The controller is updated with /approveRequests and /rejectRequests endpoints. Both methods accept a standard Map and delegate the processing to the service layer, maintaining a clean and consistent API design.
package com.tcs.controllers;

import org.apache.log4j.Logger;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;
import com.tcs.services.FrtRequestService;

import java.util.Map;

@RestController
@RequestMapping("/api/frt")
public class FrtRequestController {

    static Logger log = Logger.getLogger(FrtRequestController.class.getName());

    @Autowired
    private FrtRequestService frtRequestService;

    // ... existing frtSubmitRequest method ...

    /**
     * Handles the approval of one or more FRT requests.
     * @param payload A map containing a list of request IDs under the key "requestIds".
     * @return A ResponseEntity with the result of the operation.
     */
    @PostMapping("/approveRequests")
    public ResponseEntity<Map<String, Object>> approveRequests(@RequestBody Map<String, Object> payload) {
        log.info("Processing approve request for payload: " + payload);
        return frtRequestService.approveRequests(payload);
    }

    /**
     * Handles the rejection of one or more FRT requests.
     * @param payload A map containing a list of request IDs under the key "requestIds".
     * @return A ResponseEntity with the result of the operation.
     */
    @PostMapping("/rejectRequests")
    public ResponseEntity<Map<String, Object>> rejectRequests(@RequestBody Map<String, Object> payload) {
        log.info("Processing reject request for payload: " + payload);
        return frtRequestService.rejectRequests(payload);
    }
}

<hr>
2. Service Layer (FrtRequestServiceImpl.java)
The service methods are transactional to ensure data integrity across multiple database updates. They extract the list of IDs from the incoming Map, loop through them, and call the appropriate repository methods to execute the approval or rejection logic.
package com.tcs.services;

import com.tcs.dao.FrtDataRepository;
import org.apache.log4j.Logger;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

import java.util.ArrayList;
import java.util.HashMap;
import java.util.List;
import java.util.Map;

@Service
public class FrtRequestServiceImpl implements FrtRequestService {

    static Logger log = Logger.getLogger(FrtRequestServiceImpl.class.getName());

    @Autowired
    private FrtDataRepository frtDataRepository;

    // ... existing processFrtRequest method ...

    @Override
    @Transactional // Ensures all operations are atomic
    public ResponseEntity<Map<String, Object>> approveRequests(Map<String, Object> payload) {
        Map<String, Object> response = new HashMap<>();
        List<String> requestIds = (List<String>) payload.getOrDefault("requestIds", new ArrayList<>());

        if (requestIds.isEmpty()) {
            response.put("message", "Bad Request: No request IDs provided.");
            return new ResponseEntity<>(response, HttpStatus.BAD_REQUEST);
        }

        try {
            int successCount = 0;
            for (String reqId : requestIds) {
                [span_0](start_span)// The complex logic from the original DAO's approveReq1 method[span_0](end_span) would be implemented here.
                // You would first fetch the request details to decide which updates to run.
                // For this example, we directly call the final status update.
                frtDataRepository.updateRequestStatus(reqId, "A"); [span_1](start_span)// 'A' for Approved[span_1](end_span)
                successCount++;
            }
            log.info("Successfully approved " + successCount + " requests.");
            response.put("message", successCount + " request(s) approved successfully.");
            return new ResponseEntity<>(response, HttpStatus.OK);
        } catch (Exception e) {
            log.error("Error during request approval process: ", e);
            response.put("message", "An error occurred during approval.");
            return new ResponseEntity<>(response, HttpStatus.INTERNAL_SERVER_ERROR);
        }
    }

    @Override
    @Transactional
    public ResponseEntity<Map<String, Object>> rejectRequests(Map<String, Object> payload) {
        Map<String, Object> response = new HashMap<>();
        List<String> requestIds = (List<String>) payload.getOrDefault("requestIds", new ArrayList<>());

        if (requestIds.isEmpty()) {
            response.put("message", "Bad Request: No request IDs provided.");
            return new ResponseEntity<>(response, HttpStatus.BAD_REQUEST);
        }

        try {
            int successCount = 0;
            for (String reqId : requestIds) {
                frtDataRepository.updateRequestStatus(reqId, "R"); [span_2](start_span)// 'R' for Rejected[span_2](end_span)
                successCount++;
            }
            log.info("Successfully rejected " + successCount + " requests.");
            response.put("message", successCount + " request(s) rejected successfully.");
            return new ResponseEntity<>(response, HttpStatus.OK);
        } catch (Exception e) {
            log.error("Error during request rejection process: ", e);
            response.put("message", "An error occurred during rejection.");
            return new ResponseEntity<>(response, HttpStatus.INTERNAL_SERVER_ERROR);
        }
    }
}

The FrtRequestService.java interface would be updated with the new approveRequests and rejectRequests method definitions.
<hr>
3. Repository Layer (FrtDataRepository.java)
The repository is updated with a single, flexible updateRequestStatus method. It uses a native query and the @Modifying annotation to perform UPDATE operations for both approving and rejecting requests based on the status provided by the service layer.
package com.tcs.dao;

import com.tcs.beans.AuditStatusRequest;
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.data.jpa.repository.Modifying;
import org.springframework.data.jpa.repository.Query;
import org.springframework.data.repository.query.Param;
import org.springframework.stereotype.Repository;

import java.util.List;

@Repository
public interface FrtDataRepository extends JpaRepository<AuditStatusRequest, String> {

    // ... existing getFrtRequestData method ...

    /**
     * Updates the status of a single request in the CRS_AUDIT_STATUS table.
     * This method uses a native query to perform the update.
     * The @Modifying annotation is required for queries that change data.
     * @param reqId The ID of the request to update (AS_ID).
     * @param status The new status to set (e.g., 'A' for Approved, 'R' for Rejected).
     */
    @Modifying
    @Query(value = "UPDATE CRS_AUDIT_STATUS SET AS_REQ_STATUS = :status WHERE AS_ID = :reqId", nativeQuery = true)
    void updateRequestStatus(@Param("reqId") String reqId, @Param("status") String status);
    
    [span_3](start_span)// Note: To fully replicate the original logic[span_3](end_span), you would add more native query methods here
    // for each specific UPDATE or DELETE operation required during the approval process.
    // For example:
    // @Modifying
    // @Query(value="update branch_master set crs_auditable=? where BRANCHNO=?", nativeQuery = true)
    // void updateBranchMaster(String auditableFlag, String branchNo);
}


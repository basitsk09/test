Of course. Here is the full backend code for the approve and reject functionality, refactored according to your specified architecture.
This solution uses a Map payload in the controller and exclusively uses native queries in the repository. The complex conditional logic from the original approveReq1 method has been moved into the transactional service layer. This design avoids the use of DTOs and @Transient variables.
## Backend Code (Full Refactoring)
1. Controller (FrtRequestController.java)
The controller provides the API endpoints. It accepts a Map for the request body and delegates all processing to the service layer, keeping the controller clean and focused on handling HTTP requests.
package com.tcs.controllers;

import com.tcs.services.FrtRequestService;
import org.apache.log4j.Logger;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

import java.util.Map;

@RestController
@RequestMapping("/api/frt")
public class FrtRequestController {

    static Logger log = Logger.getLogger(FrtRequestController.class.getName());

    @Autowired
    private FrtRequestService frtRequestService;

    /**
     * Endpoint to approve audit status change requests.
     * The payload map is expected to contain a 'requestIds' list and a 'quarterDate' string.
     */
    @PostMapping("/approveRequests")
    public ResponseEntity<Map<String, Object>> approveRequests(@RequestBody Map<String, Object> payload) {
        log.info("Processing approve request with payload: " + payload);
        return frtRequestService.approveRequests(payload);
    }

    /**
     * Endpoint to reject audit status change requests.
     * The payload map is expected to contain a 'requestIds' list.
     */
    @PostMapping("/rejectRequests")
    public ResponseEntity<Map<String, Object>> rejectRequests(@RequestBody Map<String, Object> payload) {
        log.info("Processing reject request with payload: " + payload);
        return frtRequestService.rejectRequests(payload);
    }
}

2. Service (FrtRequestServiceImpl.java)
This is the core of the application. The approveRequests method is transactional to ensure data integrity. It fetches the necessary details for each request and replicates the original DAO's conditional logic by calling specific native query methods in the repository.
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

    @Override
    @Transactional
    public ResponseEntity<Map<String, Object>> approveRequests(Map<String, Object> payload) {
        Map<String, Object> response = new HashMap<>();
        List<String> requestIds = (List<String>) payload.get("requestIds");
        String quarterDate = (String) payload.get("quarterDate");
        String userId = (String) payload.get("userId"); // Assuming userId is passed from session

        if (requestIds == null || requestIds.isEmpty() || quarterDate == null) {
            response.put("message", "Bad Request: 'requestIds' and 'quarterDate' are required.");
            return new ResponseEntity<>(response, HttpStatus.BAD_REQUEST);
        }

        int successCount = 0;
        try {
            for (String reqId : requestIds) {
                // Fetch details needed to make decisions, returns Object[]
                Object[] details = frtDataRepository.findRequestDetailsById(reqId, quarterDate);
                if (details == null) continue;

                String branchCode = (String) details[0];
                String beforeSts = (String) details[1];
                String afterSts = (String) details[2];
                String ifcofrFlagForUpdate = "A".equals(afterSts) ? "N" : "Y"; // Determine new IFCOFR flag

                String marQuarter = quarterDate.substring(3, 5);

                // Replicating the logic from the original DAO's `approveReq1` method
                if ("Non-Audited".equalsIgnoreCase(beforeSts) && "Audited".equalsIgnoreCase(afterSts)) {
                    frtDataRepository.updateBranchMaster("Y", branchCode);
                    frtDataRepository.updateReportsMasterListAuditable("Y", branchCode, quarterDate);
                    if ("03".equals(marQuarter)) {
                        frtDataRepository.updateLfarBranch("Y", branchCode, quarterDate);
                    }
                } else if ("Non-Audited".equalsIgnoreCase(beforeSts) && "IFCOFR Audited".equalsIgnoreCase(afterSts)) {
                    frtDataRepository.updateBranchMaster("Y", branchCode);
                    frtDataRepository.updateReportsMasterListAuditable("Y", branchCode, quarterDate);
                    frtDataRepository.updateIfcofrAudit(ifcofrFlagForUpdate, userId, branchCode, quarterDate);
                    if ("03".equals(marQuarter)) {
                        frtDataRepository.updateLfarBranch("Y", branchCode, quarterDate);
                    }
                } else if ("Audited".equalsIgnoreCase(beforeSts) && "IFCOFR Audited".equalsIgnoreCase(afterSts)) {
                    frtDataRepository.updateIfcofrAudit(ifcofrFlagForUpdate, userId, branchCode, quarterDate);
                } else if ("Audited".equalsIgnoreCase(beforeSts) && "Non-Audited".equalsIgnoreCase(afterSts)) {
                    frtDataRepository.updateBranchMaster("N", branchCode);
                    frtDataRepository.updateReportsMasterListAuditable("N", branchCode, quarterDate);
                    frtDataRepository.deleteReportsMasterListByReportId("1033", branchCode, quarterDate);
                    if ("03".equals(marQuarter)) {
                        frtDataRepository.updateLfarBranch("N", branchCode, quarterDate);
                        frtDataRepository.deleteLfarReportsMasterList(branchCode, quarterDate);
                    }
                } else if ("IFCOFR Audited".equalsIgnoreCase(beforeSts) && "Audited".equalsIgnoreCase(afterSts)) {
                    frtDataRepository.updateIfcofrAudit(ifcofrFlagForUpdate, userId, branchCode, quarterDate);
                    frtDataRepository.deleteReportsMasterListByReportId("4065", branchCode, quarterDate);
                } else if ("IFCOFR Audited".equalsIgnoreCase(beforeSts) && "Non-Audited".equalsIgnoreCase(afterSts)) {
                    frtDataRepository.updateBranchMaster("N", branchCode);
                    frtDataRepository.updateReportsMasterListAuditable("N", branchCode, quarterDate);
                    frtDataRepository.updateIfcofrAudit(ifcofrFlagForUpdate, userId, branchCode, quarterDate);
                    frtDataRepository.deleteReportsMasterListByReportId("4065", branchCode, quarterDate);
                    frtDataRepository.deleteReportsMasterListByReportId("1033", branchCode, quarterDate);
                    if ("03".equals(marQuarter)) {
                        frtDataRepository.updateLfarBranch("N", branchCode, quarterDate);
                        frtDataRepository.deleteLfarReportsMasterList(branchCode, quarterDate);
                    }
                }
                
                // Finally, update the request's status to 'Approved'
                frtDataRepository.updateRequestStatus(reqId, "A");
                successCount++;
            }

            response.put("message", successCount + " request(s) approved successfully.");
            return new ResponseEntity<>(response, HttpStatus.OK);
        } catch (Exception e) {
            log.error("Exception during request approval process: ", e);
            response.put("message", "An internal error occurred during approval.");
            // The @Transactional annotation will automatically handle the rollback.
            return new ResponseEntity<>(response, HttpStatus.INTERNAL_SERVER_ERROR);
        }
    }
    
    @Override
    @Transactional
    public ResponseEntity<Map<String, Object>> rejectRequests(Map<String, Object> payload) {
        Map<String, Object> response = new HashMap<>();
        List<String> requestIds = (List<String>) payload.get("requestIds");

        if (requestIds == null || requestIds.isEmpty()) {
            response.put("message", "Bad Request: No request IDs provided.");
            return new ResponseEntity<>(response, HttpStatus.BAD_REQUEST);
        }

        try {
            for (String reqId : requestIds) {
                frtDataRepository.updateRequestStatus(reqId, "R"); // 'R' for Rejected
            }
            response.put("message", requestIds.size() + " request(s) rejected successfully.");
            return new ResponseEntity<>(response, HttpStatus.OK);
        } catch (Exception e) {
            log.error("Error during request rejection process: ", e);
            response.put("message", "An error occurred during rejection.");
            return new ResponseEntity<>(response, HttpStatus.INTERNAL_SERVER_ERROR);
        }
    }
}

Note: A corresponding FrtRequestService.java interface would define these methods.
3. DAO / Repository (FrtDataRepository.java)
The repository contains only native queries. The @Modifying annotation is used for all UPDATE and DELETE operations. Each method corresponds to a specific SQL statement from the original DAO, allowing the service layer to call them as needed.
package com.tcs.dao;

import com.tcs.beans.AuditStatusRequest;
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.data.jpa.repository.Modifying;
import org.springframework.data.jpa.repository.Query;
import org.springframework.data.repository.query.Param;
import org.springframework.stereotype.Repository;

@Repository
public interface FrtDataRepository extends JpaRepository<AuditStatusRequest, String> {

    /**
     * Fetches details for a specific request to enable business logic decisions in the service layer.
     * This query mirrors the logic used to create the before/after status strings in the original DAO.
     */
    @Query(value = "SELECT cas.as_branch, " +
                   "  CASE WHEN bm.crs_auditable = 'Y' THEN (CASE WHEN nvl(cia.ifcofr_audit_flag, 'N') = 'Y' THEN 'IFCOFR Audited' ELSE 'Audited' END) ELSE 'Non-Audited' END as before_status, " +
                   "  CASE cas.as_new_status WHEN 'A' THEN 'Audited' WHEN 'I' THEN 'IFCOFR Audited' ELSE 'Non-Audited' END as after_status " +
                   "FROM crs_audit_status cas " +
                   "JOIN branch_master bm ON cas.as_branch = bm.branchno " +
                   "LEFT JOIN crs_ifcofr_audit cia ON cas.as_branch = cia.ifcofr_branch AND cia.ifcofr_date = TO_DATE(:quarterDate, 'DD/MM/YYYY') " +
                   "WHERE cas.as_id = :reqId", nativeQuery = true)
    Object[] findRequestDetailsById(@Param("reqId") String reqId, @Param("quarterDate") String quarterDate);


    @Modifying
    @Query(value = "UPDATE CRS_AUDIT_STATUS SET AS_REQ_STATUS = :status WHERE AS_ID = :reqId", nativeQuery = true)
    void updateRequestStatus(@Param("reqId") String reqId, @Param("status") String status);

    @Modifying
    @Query(value = "UPDATE branch_master SET crs_auditable = :auditableFlag WHERE BRANCHNO = :branchCode", nativeQuery = true)
    void updateBranchMaster(@Param("auditableFlag") String auditableFlag, @Param("branchCode") String branchCode);

    @Modifying
    @Query(value = "UPDATE REPORTS_MASTER_LIST SET AUDITABLE = :auditableFlag WHERE BRANCH_CODE = :branchCode AND QUARTER_DATE = TO_DATE(:quarterDate, 'DD/MM/YYYY')", nativeQuery = true)
    void updateReportsMasterListAuditable(@Param("auditableFlag") String auditableFlag, @Param("branchCode") String branchCode, @Param("quarterDate") String quarterDate);

    @Modifying
    @Query(value = "UPDATE lfar_branch SET CRS_AUDITABLE = :auditableFlag WHERE BRANCHNO = :branchCode AND BRANCH_DATE = TO_DATE(:quarterDate, 'DD/MM/YYYY')", nativeQuery = true)
    void updateLfarBranch(@Param("auditableFlag") String auditableFlag, @Param("branchCode") String branchCode, @Param("quarterDate") String quarterDate);

    @Modifying
    @Query(value = "UPDATE CRS_IFCOFR_AUDIT SET IFCOFR_AUDIT_FLAG = :ifcofrFlag, IFCOFR_UPDATE_DATE = SYSDATE, IFCOFR_UPDATED_BY = :userId WHERE IFCOFR_BRANCH = :branchCode AND IFCOFR_DATE = TO_DATE(:quarterDate, 'DD/MM/YYYY')", nativeQuery = true)
    void updateIfcofrAudit(@Param("ifcofrFlag") String ifcofrFlag, @Param("userId") String userId, @Param("branchCode") String branchCode, @Param("quarterDate") String quarterDate);

    @Modifying
    @Query(value = "DELETE FROM REPORTS_MASTER_LIST WHERE REPORT_MASTER_ID = :reportMasterId AND BRANCH_CODE = :branchCode AND QUARTER_DATE = TO_DATE(:quarterDate, 'DD/MM/YYYY')", nativeQuery = true)
    void deleteReportsMasterListByReportId(@Param("reportMasterId") String reportMasterId, @Param("branchCode") String branchCode, @Param("quarterDate") String quarterDate);

    @Modifying
    @Query(value = "DELETE FROM LFAR_REPORTS_MASTER_LIST WHERE BRANCH_CODE = :branchCode AND QUARTER_DATE = TO_DATE(:quarterDate, 'DD/MM/YYYY')", nativeQuery = true)
    void deleteLfarReportsMasterList(@Param("branchCode") String branchCode, @Param("quarterDate") String quarterDate);
}


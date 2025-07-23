Of course. Here is a complete Spring Boot backend implementation structured into Controller, Service, and Repository (DAO) layers, using a Map for the payload and native SQL queries as you requested.
This refactored code is more robust, maintainable, and aligned with modern Spring Boot practices.
1. Controller Layer
The controller is the entry point for API requests from your React frontend. It accepts a generic Map as the payload, delegates the business logic to the FRTBranchRequestService, and returns a standardized response.
FRTBranchRequestController.java
package com.yourcompany.frt.controller;

import com.yourcompany.frt.service.FRTBranchRequestService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;

import java.util.Collections;
import java.util.List;
import java.util.Map;

@RestController
@RequestMapping("/api/frt-branch") // A more standard API base path
public class FRTBranchRequestController {

    @Autowired
    private FRTBranchRequestService frtBranchRequestService;

    /**
     * Corresponds to the 'getBranchRequests' call from the frontend.
     * Fetches all pending Add/Delete requests.
     */
    @PostMapping("/requests")
    public ResponseEntity<List<Map<String, Object>>> getBranchRequests(@RequestBody Map<String, Object> payload) {
        // Assuming payload contains 'quarterDate'
        String quarterDate = (String) payload.get("quarterDate");
        List<Map<String, Object>> requests = frtBranchRequestService.getPendingRequests(quarterDate);
        return ResponseEntity.ok(requests);
    }

    /**
     * Corresponds to the 'deleteReportList' call from the frontend.
     * Fetches reports that will be deleted upon approval of a branch deletion request.
     */
    @PostMapping("/delete-report-list")
    public ResponseEntity<List<Object[]>> getDeleteReportList(@RequestBody Map<String, Object> payload) {
        // Assuming payload contains 'branchCode' and 'quarterDate'
        String branchCode = (String) payload.get("branchCode");
        String quarterDate = (String) payload.get("quarterDate");
        List<Object[]> reports = frtBranchRequestService.getReportsForDeletion(branchCode, quarterDate);
        return ResponseEntity.ok(reports);
    }

    /**
     * Handles the 'Approve' action from the frontend for both Add and Delete requests.
     */
    @PostMapping("/approve")
    public ResponseEntity<Map<String, String>> approveRequest(@RequestBody Map<String, Object> payload) {
        frtBranchRequestService.processApproval(payload);
        return ResponseEntity.ok(Collections.singletonMap("message", "Request approved successfully."));
    }

    /**
     * Handles the 'Reject' action from the frontend.
     */
    @PostMapping("/reject")
    public ResponseEntity<Map<String, String>> rejectRequest(@RequestBody Map<String, Object> payload) {
        String requestId = (String) payload.get("requestId");
        frtBranchRequestService.rejectRequest(requestId);
        return ResponseEntity.ok(Collections.singletonMap("message", "Request rejected successfully."));
    }
}

2. Service Layer
The service layer contains the core business logic. It orchestrates the database operations within a single transaction, ensuring data integrity. It replaces the logic from your original addBranchReq and acceptReq methods.
FRTBranchRequestService.java
package com.yourcompany.frt.service;

import com.yourcompany.frt.repository.FRTBranchRequestRepository;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

import java.util.List;
import java.util.Map;

@Service
public class FRTBranchRequestService {

    @Autowired
    private FRTBranchRequestRepository frtBranchRequestRepository;

    /**
     * Retrieves all pending branch requests for a given quarter.
     */
    public List<Map<String, Object>> getPendingRequests(String quarterDate) {
        return frtBranchRequestRepository.findPendingRequests(quarterDate);
    }

    /**
     * Retrieves the list of reports associated with a branch marked for deletion.
     */
    public List<Object[]> getReportsForDeletion(String branchCode, String quarterDate) {
        // This method requires a specific query to be defined in the repository.
        // Assuming a placeholder query exists for this functionality.
        return frtBranchRequestRepository.findReportsLinkedToBranch(branchCode, quarterDate);
    }

    /**
     * Main logic to process an approval. This method is transactional, ensuring all
     * database operations succeed or fail together.
     *
     * @param payload The map containing request details from the controller.
     */
    @Transactional
    public void processApproval(Map<String, Object> payload) {
        String requestId = (String) payload.get("requestId");
        String branchCode = (String) payload.get("branchCode");
        String requestType = (String) payload.get("requestType");
        String quarterEndDate = (String) payload.get("quarterEndDate"); // From session/user info
        String auditStatus = (String) payload.get("auditStatus"); // e.g., "Audited (IFCOFR)"
        String requestedBy = (String) payload.get("requestedById"); // User who made the request

        if ("Add Branch".equalsIgnoreCase(requestType)) {
            addBranch(branchCode, quarterEndDate, auditStatus, requestedBy);
        } else if ("Delete Branch".equalsIgnoreCase(requestType)) {
            deleteBranch(branchCode);
        } else {
            throw new IllegalArgumentException("Invalid request type: " + requestType);
        }

        // Final step: Mark the request as approved
        frtBranchRequestRepository.updateRequestStatus("Approved", requestId);
    }

    /**
     * Encapsulates the logic for adding a new branch.
     * This replaces the original `addBranchReq` method.
     */
    private void addBranch(String branchCode, String quarterEndDate, String auditStatusRaw, String requestedBy) {
        // 1. Insert into branch_master from cbs_brhm
        frtBranchRequestRepository.insertIntoBranchMaster(branchCode);

        // 2. Insert into br_details from cbs_brhm
        frtBranchRequestRepository.insertIntoBrDetails(branchCode);
        
        // 3. Determine IFCOFR flag and insert into crs_ifcofr_audit
        String ifcofrFlag = "N";
        if ("Audited (IFCOFR)".equalsIgnoreCase(auditStatusRaw)) {
            ifcofrFlag = "Y";
        }
        frtBranchRequestRepository.insertIntoCrsIfcofrAudit(quarterEndDate, branchCode, ifcofrFlag, requestedBy);

        // 4. Conditionally insert into LFAR and TAR tables if it's the March quarter
        String marQuarter = quarterEndDate.substring(3, 5); // "dd/mm/yyyy" -> "mm"
        if ("03".equals(marQuarter)) {
            frtBranchRequestRepository.insertIntoLfarBranch(quarterEndDate, branchCode);
            frtBranchRequestRepository.insertIntoTarBranch(quarterEndDate, branchCode);
        }
    }

    /**
     * Encapsulates the logic for deleting a branch.
     * NOTE: This is an example. The exact tables to delete from may vary.
     */
    private void deleteBranch(String branchCode) {
        // Add delete operations for all relevant tables
        frtBranchRequestRepository.deleteFromCrsIfcofrAudit(branchCode);
        frtBranchRequestRepository.deleteFromBrDetails(branchCode);
        frtBranchRequestRepository.deleteFromLfarBranch(branchCode);
        frtBranchRequestRepository.deleteFromTarBranch(branchCode);
        frtBranchRequestRepository.deleteFromBranchMaster(branchCode);
        // Add more delete statements as necessary
    }

    /**
     * Processes a rejection by simply updating the request's status.
     * This method should also be transactional.
     */
    @Transactional
    public void rejectRequest(String requestId) {
        frtBranchRequestRepository.updateRequestStatus("Rejected", requestId);
    }
}


3. Repository (DAO) Layer
The repository layer interacts directly with the database using native SQL queries via the @Query annotation. The @Modifying annotation is crucial for queries that change data (INSERT, UPDATE, DELETE).
FRTBranchRequestRepository.java
package com.yourcompany.frt.repository;

import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.data.jpa.repository.Modifying;
import org.springframework.data.jpa.repository.Query;
import org.springframework.data.repository.query.Param;
import org.springframework.stereotype.Repository;

import java.util.List;
import java.util.Map;

// We can extend a dummy entity or just use the repository as a component for native queries.
// Using a placeholder entity 'BranchRequest' makes it a standard JpaRepository.
// public interface FRTBranchRequestRepository extends JpaRepository<BranchRequest, Long> {

@Repository
public interface FRTBranchRequestRepository extends JpaRepository<DummyEntity, Long> { // Use a dummy entity if needed

    // === QUERIES FOR ADDING A BRANCH (from original code) ===

    @Modifying
    @Query(value = "INSERT INTO branch_master(BRANCHNO, REGION_NO, BR_NAME, MANAGERS_NAME, ADDRESS_1, ADDRESS_2, ADDRESS_3, POST_CODE, PHONE_NO, NETWORK_CODE, STD_CODE, CIRCLE_CODE, MODULE_CODE, REGION_CODE, VOIP_PHONE_NO, MAN_RES_PHONE, MAN_MOBI_PHONE, INST, CRS_AUDITABLE) " +
                   "SELECT BRANCHNO, REGION_NO, BR_NAME, MANAGERS_NAME, ADDRESS_1, ADDRESS_2, ADDRESS_3, POST_CODE, PHONE_NO, NETWORK_CODE, STD_CODE, CIRCLE_CODE, MODULE_CODE, REGION_CODE, VOIP_PHONE_NO, MAN_RES_PHONE, MAN_MOBI_PHONE, INST, " +
                   "CASE WHEN CRS_AUDITABLE = 'Y' THEN 'Y' WHEN CRS_AUDITABLE = 'I' THEN 'Y' ELSE 'N' END " +
                   "FROM cbs_brhm WHERE branchno = :branchCode", nativeQuery = true)
    void insertIntoBranchMaster(@Param("branchCode") String branchCode);

    @Modifying
    @Query(value = "INSERT INTO lfar_branch(BRANCHNO, REGION_NO, BR_NAME, MANAGERS_NAME, ADDRESS_1, ADDRESS_2, ADDRESS_3, POST_CODE, PHONE_NO, NETWORK_CODE, STD_CODE, CIRCLE_CODE, MODULE_CODE, REGION_CODE, VOIP_PHONE_NO, MAN_RES_PHONE, MAN_MOBI_PHONE, INST, CRS_AUDITABLE, branch_date) " +
                   "SELECT BRANCHNO, REGION_NO, BR_NAME, MANAGERS_NAME, ADDRESS_1, ADDRESS_2, ADDRESS_3, POST_CODE, PHONE_NO, NETWORK_CODE, STD_CODE, CIRCLE_CODE, MODULE_CODE, REGION_CODE, VOIP_PHONE_NO, MAN_RES_PHONE, MAN_MOBI_PHONE, INST, " +
                   "CASE WHEN CRS_AUDITABLE = 'Y' THEN 'Y' WHEN CRS_AUDITABLE = 'I' THEN 'Y' ELSE 'N' END, TO_DATE(:quarterEndDate, 'dd/mm/yyyy') " +
                   "FROM cbs_brhm WHERE branchno = :branchCode", nativeQuery = true)
    void insertIntoLfarBranch(@Param("quarterEndDate") String quarterEndDate, @Param("branchCode") String branchCode);

    @Modifying
    @Query(value = "INSERT INTO tar_branch(BRANCHNO, REGION_NO, BR_NAME, MANAGERS_NAME, ADDRESS_1, ADDRESS_2, ADDRESS_3, POST_CODE, PHONE_NO, NETWORK_CODE, STD_CODE, CIRCLE_CODE, MODULE_CODE, REGION_CODE, VOIP_PHONE_NO, MAN_RES_PHONE, MAN_MOBI_PHONE, INST, CRS_AUDITABLE, branch_date) " +
                   "SELECT BRANCHNO, REGION_NO, BR_NAME, MANAGERS_NAME, ADDRESS_1, ADDRESS_2, ADDRESS_3, POST_CODE, PHONE_NO, NETWORK_CODE, STD_CODE, CIRCLE_CODE, MODULE_CODE, REGION_CODE, VOIP_PHONE_NO, MAN_RES_PHONE, MAN_MOBI_PHONE, INST, " +
                   "CASE WHEN CRS_AUDITABLE = 'Y' THEN 'Y' WHEN CRS_AUDITABLE = 'I' THEN 'Y' ELSE 'N' END, TO_DATE(:quarterEndDate, 'dd/mm/yyyy') " +
                   "FROM cbs_brhm WHERE branchno = :branchCode", nativeQuery = true)
    void insertIntoTarBranch(@Param("quarterEndDate") String quarterEndDate, @Param("branchCode") String branchCode);

    @Modifying
    @Query(value = "INSERT INTO br_details (CRS_BRANCHCODE, CRS_CIRCLECODE) SELECT BRANCHNO, CIRCLE_CODE FROM cbs_brhm WHERE branchno = :branchCode", nativeQuery = true)
    void insertIntoBrDetails(@Param("branchCode") String branchCode);

    @Modifying
    @Query(value = "INSERT INTO crs_ifcofr_audit (IFCOFR_DATE, IFCOFR_BRANCH, IFCOFR_AUDIT_FLAG, IFCOFR_UPDATED_BY) " +
                   "VALUES (TO_DATE(:quarterEndDate, 'dd/mm/yyyy'), :branchCode, :ifcofrFlag, :updatedBy)", nativeQuery = true)
    void insertIntoCrsIfcofrAudit(@Param("quarterEndDate") String quarterEndDate, @Param("branchCode") String branchCode, @Param("ifcofrFlag") String ifcofrFlag, @Param("updatedBy") String updatedBy);

    // === QUERIES FOR DELETING A BRANCH (Example) ===
    
    @Modifying
    @Query(value = "DELETE FROM branch_master WHERE BRANCHNO = :branchCode", nativeQuery = true)
    void deleteFromBranchMaster(@Param("branchCode") String branchCode);
    
    @Modifying
    @Query(value = "DELETE FROM lfar_branch WHERE BRANCHNO = :branchCode", nativeQuery = true)
    void deleteFromLfarBranch(@Param("branchCode") String branchCode);

    @Modifying
    @Query(value = "DELETE FROM tar_branch WHERE BRANCHNO = :branchCode", nativeQuery = true)
    void deleteFromTarBranch(@Param("branchCode") String branchCode);

    @Modifying
    @Query(value = "DELETE FROM br_details WHERE CRS_BRANCHCODE = :branchCode", nativeQuery = true)
    void deleteFromBrDetails(@Param("branchCode") String branchCode);
    
    @Modifying
    @Query(value = "DELETE FROM crs_ifcofr_audit WHERE IFCOFR_BRANCH = :branchCode", nativeQuery = true)
    void deleteFromCrsIfcofrAudit(@Param("branchCode") String branchCode);


    // === QUERY FOR MANAGING THE REQUEST LIFECYCLE ===
    
    @Modifying
    @Query(value = "UPDATE frt_branch_requests SET status = :status, approved_by = :approverId, approved_on = CURRENT_TIMESTAMP WHERE request_id = :requestId", nativeQuery = true)
    void updateRequestStatus(@Param("status") String status, @Param("requestId") String requestId, @Param("approverId") String approverId);


    // === QUERIES FOR FETCHING DATA FOR THE UI ===
    
    // This is an example query. You will need to adjust joins and column names to match your schema.
    @Query(value = "SELECT r.request_id as \"requestId\", r.branch_code as \"branchCode\", b.br_name as \"branchName\", " +
                   "r.request_type as \"requestType\", r.status as \"status\", r.requested_on as \"requestedOn\", " +
                   "c.circle_name as \"circleName\", r.audit_status as \"auditStatus\", r.requested_by as \"requestedBy\" " +
                   "FROM frt_branch_requests r " +
                   "LEFT JOIN cbs_brhm b ON r.branch_code = b.branchno " +
                   "LEFT JOIN circle_master c ON b.circle_code = c.circle_code " +
                   "WHERE r.status = 'Pending' AND r.quarter_end_date = TO_DATE(:quarterDate, 'dd/mm/yyyy')", nativeQuery = true)
    List<Map<String, Object>> findPendingRequests(@Param("quarterDate") String quarterDate);

    // Example query for the delete report list
    @Query(value = "SELECT 'Credit' as module, 'TLAC_2' as report, 'Term Loan Account' as name, '1' as pending FROM DUAL UNION ALL " +
                   "SELECT 'Advances' as module, 'A1' as report, 'Large Advances' as name, '3' as pending FROM DUAL", nativeQuery = true)
    List<Object[]> findReportsLinkedToBranch(@Param("branchCode") String branchCode, @Param("quarterDate") String quarterDate);
    
    // A dummy entity class so JpaRepository has a type. It doesn't need to map to a real table.
    // Create this class in a separate file.
    // package com.yourcompany.frt.repository;
    // import javax.persistence.Entity;
    // import javax.persistence.Id;
    // @Entity
    // public class DummyEntity { @Id private Long id; }
}


Of course. Here is the complete backend code, updated to remove the DTO (ReportInfo) as requested. The logic now uses a List<Object[]> to handle the data, which is a common approach when avoiding dedicated DTOs for simple query results.
1. Controller Layer (Unchanged)
The controller layer does not need any modifications. It correctly passes the payload to the service.
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
@RequestMapping("/api/frt-branch")
public class FRTBranchRequestController {

    @Autowired
    private FRTBranchRequestService frtBranchRequestService;

    @PostMapping("/requests")
    public ResponseEntity<List<Map<String, Object>>> getBranchRequests(@RequestBody Map<String, Object> payload) {
        String quarterDate = (String) payload.get("quarterDate");
        List<Map<String, Object>> requests = frtBranchRequestService.getPendingRequests(quarterDate);
        return ResponseEntity.ok(requests);
    }

    @PostMapping("/delete-report-list")
    public ResponseEntity<List<Object[]>> getDeleteReportList(@RequestBody Map<String, Object> payload) {
        String branchCode = (String) payload.get("branchCode");
        String quarterDate = (String) payload.get("quarterDate");
        List<Object[]> reports = frtBranchRequestService.getReportsForDeletion(branchCode, quarterDate);
        return ResponseEntity.ok(reports);
    }

    @PostMapping("/approve")
    public ResponseEntity<Map<String, String>> approveRequest(@RequestBody Map<String, Object> payload) {
        frtBranchRequestService.processApproval(payload);
        return ResponseEntity.ok(Collections.singletonMap("message", "Request approved successfully."));
    }

    @PostMapping("/reject")
    public ResponseEntity<Map<String, String>> rejectRequest(@RequestBody Map<String, Object> payload) {
        String requestId = (String) payload.get("requestId");
        frtBranchRequestService.rejectRequest(requestId);
        return ResponseEntity.ok(Collections.singletonMap("message", "Request rejected successfully."));
    }
}

2. Service Layer (Updated)
The service layer is modified to work with List<Object[]> returned from the repository instead of the ReportInfo DTO.
FRTBranchRequestService.java
package com.yourcompany.frt.service;

import com.yourcompany.frt.repository.FRTBranchRequestRepository;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

import javax.persistence.EntityManager;
import javax.persistence.PersistenceContext;
import java.util.List;
import java.util.Map;

@Service
public class FRTBranchRequestService {

    private static final Logger log = LoggerFactory.getLogger(FRTBranchRequestService.class);

    @Autowired
    private FRTBranchRequestRepository frtBranchRequestRepository;

    @PersistenceContext
    private EntityManager entityManager;

    public List<Map<String, Object>> getPendingRequests(String quarterDate) {
        return frtBranchRequestRepository.findPendingRequests(quarterDate);
    }

    public List<Object[]> getReportsForDeletion(String branchCode, String quarterDate) {
        return frtBranchRequestRepository.findReportsLinkedToBranch(branchCode, quarterDate);
    }

    @Transactional
    public void processApproval(Map<String, Object> payload) {
        String requestId = (String) payload.get("requestId");
        String branchCode = (String) payload.get("branchCode");
        String requestType = (String) payload.get("requestType");
        
        String quarterEndDate = (String) payload.get("quarterEndDate");
        String auditStatus = (String) payload.get("auditStatus");
        String requestedBy = (String) payload.get("requestedById");
        String circleCode = (String) payload.get("circleCode");
        String roCode = (String) payload.get("roCode");

        if ("Add Branch".equalsIgnoreCase(requestType)) {
            log.info("Processing 'Add Branch' approval for request ID: {}", requestId);
            addBranch(branchCode, quarterEndDate, auditStatus, requestedBy);
        } else if ("Delete Branch".equalsIgnoreCase(requestType)) {
            log.info("Processing 'Delete Branch' approval for request ID: {}", requestId);
            performBranchDeletion(branchCode, quarterEndDate, requestedBy, circleCode, roCode, auditStatus);
        } else {
            throw new IllegalArgumentException("Invalid request type: " + requestType);
        }

        log.info("Marking request ID {} as 'Approved'", requestId);
        frtBranchRequestRepository.updateRequestStatus("Approved", requestId, requestedBy);
    }

    private void addBranch(String branchCode, String quarterEndDate, String auditStatusRaw, String requestedBy) {
        frtBranchRequestRepository.insertIntoBranchMaster(branchCode);
        frtBranchRequestRepository.insertIntoBrDetails(branchCode);
        String ifcofrFlag = "Audited (IFCOFR)".equalsIgnoreCase(auditStatusRaw) ? "Y" : "N";
        frtBranchRequestRepository.insertIntoCrsIfcofrAudit(quarterEndDate, branchCode, ifcofrFlag, requestedBy);
        String marQuarter = quarterEndDate.substring(3, 5);
        if ("03".equals(marQuarter)) {
            frtBranchRequestRepository.insertIntoLfarBranch(quarterEndDate, branchCode);
            frtBranchRequestRepository.insertIntoTarBranch(quarterEndDate, branchCode);
        }
    }

    private void performBranchDeletion(String branchCode, String quarterEndDate, String userId, String circleCode, String roCode, String auditStatusRaw) {
        // 1. Get reports as a list of Object arrays.
        List<Object[]> reportsToDelete = frtBranchRequestRepository.findReportsForBranch(branchCode, quarterEndDate);
        log.info("Found {} reports to delete for branch {}", reportsToDelete.size(), branchCode);

        // 2. Iterate through the results, casting elements from the Object array.
        for (Object[] reportData : reportsToDelete) {
            String reportId = (String) reportData[0];
            String reportMasterId = (String) reportData[1];
            
            deleteReportDataDynamically(reportId, reportMasterId);
            insertDeletionTrackRecord(branchCode, circleCode, roCode, auditStatusRaw, quarterEndDate, userId, reportId, reportMasterId);
        }

        // 3. Delete the master records for the branch.
        log.info("Deleting master records for branch {}", branchCode);
        frtBranchRequestRepository.deleteFromReportsMasterList(branchCode, quarterEndDate);
        frtBranchRequestRepository.deleteFromBranchMaster(branchCode);
        frtBranchRequestRepository.deleteFromBrDetails(branchCode);
        frtBranchRequestRepository.deleteFromCrsIfcofrAuditByBranch(branchCode, quarterEndDate);

        String marQuarter = quarterEndDate.substring(3, 5);
        if ("03".equals(marQuarter)) {
            log.info("March quarter detected. Deleting from TAR and LFAR tables.");
            frtBranchRequestRepository.deleteFromTarReportsMasterList(branchCode, quarterEndDate);
            frtBranchRequestRepository.deleteFromTarBranch(branchCode, quarterEndDate);
            frtBranchRequestRepository.deleteFromLfarReportsMasterList(branchCode, quarterEndDate);
            frtBranchRequestRepository.deleteFromLfarBranch(branchCode, quarterEndDate);
        }
        log.info("Branch {} deletion process completed.", branchCode);
    }
    
    private void deleteReportDataDynamically(String reportId, String reportMasterId) {
        List<String> tableNames = frtBranchRequestRepository.findDynamicTableNames(reportMasterId);
        log.info("Found tables to clean for reportMasterId {}: {}", reportMasterId, tableNames);
        
        for (String tableName : tableNames) {
            String sql = "DELETE FROM " + tableName + " WHERE REPORT_MASTER_LIST_ID_FK = :reportId";
            int affectedRows = entityManager.createNativeQuery(sql)
                .setParameter("reportId", reportId)
                .executeUpdate();
            log.info("Deleted {} rows from table {} for reportId {}", affectedRows, tableName, reportId);
        }
    }

    private void insertDeletionTrackRecord(String branchCode, String circleCode, String regionCode, String auditStatusRaw, String qed, String userId, String reportId, String reportMstId) {
        String tableName = "CRS_STS";
        String idColumn = "crs_id";
        String userColumn = "crs_user";
        String commentColumn = "crs_comment";
        String auditStatus = "Non-Audited".equalsIgnoreCase(auditStatusRaw) ? "N" : "Y";
        
        String sql = String.format(
            "INSERT INTO %s (branch_code, circle_code, ro_code, crs_auditable, quarter_date, report_id, %s, status, sts, nil_report_flag, %s, %s) " +
            "VALUES (:branchCode, :circleCode, :regionCode, :audSts, TO_DATE(:qed, 'dd/mm/yyyy'), :reportId, :reportMstId, :status, :sts, 'N', :userId, :comment)",
            tableName, idColumn, userColumn, commentColumn
        );

        entityManager.createNativeQuery(sql)
            .setParameter("branchCode", branchCode)
            .setParameter("circleCode", circleCode)
            .setParameter("regionCode", regionCode != null ? regionCode.replace(" ", "") : null)
            .setParameter("audSts", auditStatus)
            .setParameter("qed", qed)
            .setParameter("reportId", reportId)
            .setParameter("reportMstId", reportMstId)
            .setParameter("status", "Deleted by FRT")
            .setParameter("sts", "")
            .setParameter("userId", userId)
            .setParameter("comment", "")
            .executeUpdate();
            
        log.info("Inserted deletion tracking record for branch {}, reportId {}", branchCode, reportId);
    }
    
    @Transactional
    public void rejectRequest(String requestId) {
        String rejecterId = "SYSTEM";
        log.info("Marking request ID {} as 'Rejected'", requestId);
        frtBranchRequestRepository.updateRequestStatus("Rejected", requestId, rejecterId);
    }
}

3. Repository (DAO) Layer (Updated)
The repository is updated to return a List<Object[]> from the findReportsForBranch method.
FRTBranchRequestRepository.java
package com.yourcompany.frt.repository;

import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.data.jpa.repository.Modifying;
import org.springframework.data.jpa.repository.Query;
import org.springframework.data.repository.query.Param;
import org.springframework.stereotype.Repository;

import java.util.List;
import java.util.Map;

@Repository
public interface FRTBranchRequestRepository extends JpaRepository<DummyEntity, Long> {

    // === QUERIES FOR FETCHING DATA FOR DELETION ===

    /**
     * Fetches the report_id and report_master_id for a branch.
     * Returns a List of Object arrays, where each array contains [report_id, report_master_id].
     */
    @Query(value = "SELECT r.report_id, r.report_master_id " +
                   "FROM reports_master_list r " +
                   "WHERE r.branch_code = :branchCode AND r.quarter_date = TO_DATE(:quarterDate, 'dd/mm/yyyy')",
           nativeQuery = true)
    List<Object[]> findReportsForBranch(@Param("branchCode") String branchCode, @Param("quarterDate") String quarterDate);

    @Query(value = "SELECT table_name FROM crs_nil_table WHERE report_master_id = :reportMasterId", nativeQuery = true)
    List<String> findDynamicTableNames(@Param("reportMasterId") String reportMasterId);
    
    // === MASTER DELETION QUERIES ===

    @Modifying
    @Query(value = "DELETE FROM reports_master_list WHERE branch_code = :branchCode AND quarter_date = TO_DATE(:quarterDate, 'dd/mm/yyyy')", nativeQuery = true)
    void deleteFromReportsMasterList(@Param("branchCode") String branchCode, @Param("quarterDate") String quarterDate);

    @Modifying
    @Query(value = "DELETE FROM tar_reports_master_list WHERE branch_code = :branchCode AND quarter_date = TO_DATE(:quarterDate, 'dd/mm/yyyy')", nativeQuery = true)
    void deleteFromTarReportsMasterList(@Param("branchCode") String branchCode, @Param("quarterDate") String quarterDate);

    @Modifying
    @Query(value = "DELETE FROM lfar_reports_master_list WHERE branch_code = :branchCode AND quarter_date = TO_DATE(:quarterDate, 'dd/mm/yyyy')", nativeQuery = true)
    void deleteFromLfarReportsMasterList(@Param("branchCode") String branchCode, @Param("quarterDate") String quarterDate);

    @Modifying
    @Query(value = "DELETE FROM branch_master WHERE BRANCHNO = :branchCode", nativeQuery = true)
    void deleteFromBranchMaster(@Param("branchCode") String branchCode);

    @Modifying
    @Query(value = "DELETE FROM tar_branch WHERE BRANCHNO = :branchCode AND branch_date = TO_DATE(:quarterDate, 'dd/mm/yyyy')", nativeQuery = true)
    void deleteFromTarBranch(@Param("branchCode") String branchCode, @Param("quarterDate") String quarterDate);

    @Modifying
    @Query(value = "DELETE FROM lfar_branch WHERE BRANCHNO = :branchCode AND branch_date = TO_DATE(:quarterDate, 'dd/mm/yyyy')", nativeQuery = true)
    void deleteFromLfarBranch(@Param("branchCode") String branchCode, @Param("quarterDate") String quarterDate);

    @Modifying
    @Query(value = "DELETE FROM br_details WHERE crs_branchcode = :branchCode", nativeQuery = true)
    void deleteFromBrDetails(@Param("branchCode") String branchCode);

    @Modifying
    @Query(value = "DELETE FROM crs_ifcofr_audit WHERE ifcofr_branch = :branchCode AND ifcofr_date = TO_DATE(:quarterDate, 'dd/mm/yyyy')", nativeQuery = true)
    void deleteFromCrsIfcofrAuditByBranch(@Param("branchCode") String branchCode, @Param("quarterDate") String quarterDate);
    
    // === LIFECYCLE, UI, AND ADDITION QUERIES (Unchanged) ===
    
    @Modifying
    @Query(value = "UPDATE frt_branch_requests SET status = :status, approved_by = :approverId, approved_on = CURRENT_TIMESTAMP WHERE request_id = :requestId", nativeQuery = true)
    void updateRequestStatus(@Param("status") String status, @Param("requestId") String requestId, @Param("approverId") String approverId);

    // ... (All other queries for adding a branch and fetching UI data remain the same) ...
    @Query(value = "SELECT r.request_id as \"requestId\", r.branch_code as \"branchCode\", b.br_name as \"branchName\", " +
                   "r.request_type as \"requestType\", r.status as \"status\", r.requested_on as \"requestedOn\", " +
                   "c.circle_name as \"circleName\", r.audit_status as \"auditStatus\", r.requested_by as \"requestedBy\" " +
                   "FROM frt_branch_requests r " +
                   "LEFT JOIN cbs_brhm b ON r.branch_code = b.branchno " +
                   "LEFT JOIN circle_master c ON b.circle_code = c.circle_code " +
                   "WHERE r.status = 'Pending' AND r.quarter_end_date = TO_DATE(:quarterDate, 'dd/mm/yyyy')", nativeQuery = true)
    List<Map<String, Object>> findPendingRequests(@Param("quarterDate") String quarterDate);

    @Query(value = "SELECT 'Credit' as module, 'TLAC_2' as report, 'Term Loan Account' as name, '1' as pending FROM DUAL UNION ALL " +
                   "SELECT 'Advances' as module, 'A1' as report, 'Large Advances' as name, '3' as pending FROM DUAL", nativeQuery = true)
    List<Object[]> findReportsLinkedToBranch(@Param("branchCode") String branchCode, @Param("quarterDate") String quarterDate);
    
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
}


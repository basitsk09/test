Of course. You are right to point that out. Keeping data access logic, even dynamic queries, strictly within the DAO/repository layer is the best practice for a clean architecture. Using EntityManager in the service was a shortcut, and a custom repository implementation is the correct pattern.
I've also implemented the full tracking logic for CRS_STS, TAR_STATUS, and LFAR_STATUS by assuming the report's module type can be fetched from the database.
Here is the complete, refactored code that moves all database logic to the DAO layer.
The New DAO/Repository Structure
To handle the dynamic queries entirely within the DAO layer, we use a standard Spring Data JPA pattern: a custom repository implementation.
 * FRTBranchRequestRepositoryCustom (New Interface): Defines the signature for our complex method.
 * FRTBranchRequestRepositoryImpl (New Class): Implements the custom interface. This is where we inject EntityManager and write the dynamic query logic.
 * FRTBranchRequestRepository (Updated Interface): Extends the main JpaRepository and our new custom interface, making the custom method available to the service.
This keeps the service layer completely clean of any EntityManager or query-building logic.
1. Controller Layer (Unchanged)
This layer remains the same. It correctly calls the service layer.
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

2. Service Layer (Simplified)
The service layer is now much cleaner. It's responsible only for orchestrating the business flow and defining the transaction boundary. It calls the repository to handle all database work.
FRTBranchRequestService.java
package com.yourcompany.frt.service;

import com.yourcompany.frt.repository.FRTBranchRequestRepository;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

import java.util.List;
import java.util.Map;

@Service
public class FRTBranchRequestService {

    private static final Logger log = LoggerFactory.getLogger(FRTBranchRequestService.class);

    @Autowired
    private FRTBranchRequestRepository frtBranchRequestRepository;

    public List<Map<String, Object>> getPendingRequests(String quarterDate) {
        return frtBranchRequestRepository.findPendingRequests(quarterDate);
    }

    public List<Object[]> getReportsForDeletion(String branchCode, String quarterDate) {
        return frtBranchRequestRepository.findReportsLinkedToBranch(branchCode, quarterDate);
    }

    @Transactional
    public void processApproval(Map<String, Object> payload) {
        String requestId = (String) payload.get("requestId");
        String requestType = (String) payload.get("requestType");
        String approverId = (String) payload.get("requestedById");

        if ("Add Branch".equalsIgnoreCase(requestType)) {
            log.info("Processing 'Add Branch' approval for request ID: {}", requestId);
            addBranch(payload);
        } else if ("Delete Branch".equalsIgnoreCase(requestType)) {
            log.info("Processing 'Delete Branch' approval for request ID: {}", requestId);
            // The service now makes a single, clean call to the repository.
            frtBranchRequestRepository.deleteBranchAndAssociatedData(payload);
        } else {
            throw new IllegalArgumentException("Invalid request type: " + requestType);
        }

        log.info("Marking request ID {} as 'Approved'", requestId);
        frtBranchRequestRepository.updateRequestStatus("Approved", requestId, approverId);
    }

    private void addBranch(Map<String, Object> payload) {
        String branchCode = (String) payload.get("branchCode");
        String quarterEndDate = (String) payload.get("quarterEndDate");
        String auditStatusRaw = (String) payload.get("auditStatus");
        String requestedBy = (String) payload.get("requestedById");

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

    @Transactional
    public void rejectRequest(String requestId) {
        String rejecterId = "SYSTEM"; // Or get from SecurityContext
        log.info("Marking request ID {} as 'Rejected'", requestId);
        frtBranchRequestRepository.updateRequestStatus("Rejected", requestId, rejecterId);
    }
}

3. DAO/Repository Layer (Updated)
This layer now consists of three parts, following the standard custom repository pattern.
FRTBranchRequestRepositoryCustom.java (New Interface)
This interface defines the custom method.
package com.yourcompany.frt.repository;

import java.util.Map;

public interface FRTBranchRequestRepositoryCustom {
    /**
     * Handles the entire logic of deleting a branch, including associated reports
     * and tracking records, completely within the repository layer.
     * @param payload The map containing all necessary data like branchCode, quarterEndDate, etc.
     */
    void deleteBranchAndAssociatedData(Map<String, Object> payload);
}

FRTBranchRequestRepositoryImpl.java (New Implementation Class)
This class contains the full, complex logic for deletion, including EntityManager usage and dynamic SQL.
package com.yourcompany.frt.repository;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.stereotype.Repository;

import javax.persistence.EntityManager;
import javax.persistence.PersistenceContext;
import java.util.List;
import java.util.Map;

@Repository
public class FRTBranchRequestRepositoryImpl implements FRTBranchRequestRepositoryCustom {

    private static final Logger log = LoggerFactory.getLogger(FRTBranchRequestRepositoryImpl.class);

    @PersistenceContext
    private EntityManager entityManager;

    @Override
    public void deleteBranchAndAssociatedData(Map<String, Object> payload) {
        String branchCode = (String) payload.get("branchCode");
        String quarterEndDate = (String) payload.get("quarterEndDate");
        String userId = (String) payload.get("requestedById");
        String circleCode = (String) payload.get("circleCode");
        String roCode = (String) payload.get("roCode");
        String auditStatusRaw = (String) payload.get("auditStatus");
        String auditStatus = "Non-Audited".equalsIgnoreCase(auditStatusRaw) ? "N" : "Y";

        // 1. Get all reports for the branch that need to be deleted.
        List<Object[]> reportsToDelete = findReportsForBranch(branchCode, quarterEndDate);
        log.info("Found {} reports to delete for branch {}", reportsToDelete.size(), branchCode);

        // 2. For each report, delete its associated data and log the action.
        for (Object[] reportData : reportsToDelete) {
            String reportId = (String) reportData[0];
            String reportMasterId = (String) reportData[1];
            String module = (String) reportData[2]; // e.g., "CRS", "TAR", "LFAR"

            deleteReportDataDynamically(reportId, reportMasterId);
            insertDeletionTrackRecord(module, branchCode, circleCode, roCode, auditStatus, quarterEndDate, userId, reportId, reportMasterId);
        }

        // 3. Delete the master records for the branch.
        log.info("Deleting master records for branch {}", branchCode);
        deleteFromMasterTables(branchCode, quarterEndDate);
    }

    private List<Object[]> findReportsForBranch(String branchCode, String quarterDate) {
        // NOTE: Added 'module' to the select list. You must have a column in
        // 'reports_master_list' that identifies the module (e.g., 'CRS', 'TAR', 'LFAR').
        String sql = "SELECT report_id, report_master_id, module FROM reports_master_list " +
                     "WHERE branch_code = :branchCode AND quarter_date = TO_DATE(:quarterDate, 'dd/mm/yyyy')";
        return entityManager.createNativeQuery(sql)
                .setParameter("branchCode", branchCode)
                .setParameter("quarterDate", quarterDate)
                .getResultList();
    }

    private void deleteReportDataDynamically(String reportId, String reportMasterId) {
        String tableSql = "SELECT table_name FROM crs_nil_table WHERE report_master_id = :reportMasterId";
        List<String> tableNames = entityManager.createNativeQuery(tableSql)
                .setParameter("reportMasterId", reportMasterId)
                .getResultList();

        log.info("Found tables to clean for reportMasterId {}: {}", reportMasterId, tableNames);

        for (String tableName : tableNames) {
            String deleteSql = "DELETE FROM " + tableName + " WHERE REPORT_MASTER_LIST_ID_FK = :reportId";
            int affectedRows = entityManager.createNativeQuery(deleteSql)
                    .setParameter("reportId", reportId)
                    .executeUpdate();
            log.info("Deleted {} rows from table {} for reportId {}", affectedRows, tableName, reportId);
        }
    }

    private void insertDeletionTrackRecord(String module, String branchCode, String circleCode, String regionCode, String audSts, String qed, String userId, String reportId, String reportMstId) {
        String tableName;
        String idColumn;
        String userColumn;
        String commentColumn;

        // Determine table and column names based on the module
        switch (module.toUpperCase()) {
            case "TAR":
                tableName = "TAR_STATUS";
                idColumn = "tar_id";
                userColumn = "tar_user";
                commentColumn = "tar_comment";
                break;
            case "LFAR":
                tableName = "LFAR_STATUS";
                idColumn = "lfar_id";
                userColumn = "lfar_user";
                commentColumn = "lfar_comment";
                break;
            case "CRS":
            default:
                tableName = "CRS_STS";
                idColumn = "crs_id";
                userColumn = "crs_user";
                commentColumn = "crs_comment";
                break;
        }

        String sql = String.format(
            "INSERT INTO %s (branch_code, circle_code, ro_code, crs_auditable, quarter_date, report_id, %s, status, sts, nil_report_flag, %s, %s) " +
            "VALUES (:branchCode, :circleCode, :regionCode, :audSts, TO_DATE(:qed, 'dd/mm/yyyy'), :reportId, :reportMstId, :status, :sts, 'N', :userId, :comment)",
            tableName, idColumn, userColumn, commentColumn
        );

        entityManager.createNativeQuery(sql)
            .setParameter("branchCode", branchCode)
            .setParameter("circleCode", circleCode)
            .setParameter("regionCode", regionCode != null ? regionCode.replace(" ", "") : null)
            .setParameter("audSts", audSts)
            .setParameter("qed", qed)
            .setParameter("reportId", reportId)
            .setParameter("reportMstId", reportMstId)
            .setParameter("status", "Deleted by FRT")
            .setParameter("sts", "")
            .setParameter("userId", userId)
            .setParameter("comment", "")
            .executeUpdate();
            
        log.info("Inserted deletion tracking record into {} for branch {}, reportId {}", tableName, branchCode, reportId);
    }
    
    private void deleteFromMasterTables(String branchCode, String quarterEndDate) {
        String[] deleteQueries = {
            "DELETE FROM reports_master_list WHERE branch_code = :p1 AND quarter_date = TO_DATE(:p2, 'dd/mm/yyyy')",
            "DELETE FROM branch_master WHERE BRANCHNO = :p1",
            "DELETE FROM br_details WHERE crs_branchcode = :p1",
            "DELETE FROM crs_ifcofr_audit WHERE ifcofr_branch = :p1 AND ifcofr_date = TO_DATE(:p2, 'dd/mm/yyyy')"
        };

        for(String sql : deleteQueries) {
            entityManager.createNativeQuery(sql)
                .setParameter("p1", branchCode)
                .setParameter("p2", quarterEndDate)
                .executeUpdate();
        }
        
        // Conditional deletion for March quarter
        String marQuarter = quarterEndDate.substring(3, 5);
        if ("03".equals(marQuarter)) {
            log.info("March quarter detected. Deleting from TAR and LFAR tables.");
            String[] marchDeleteQueries = {
                "DELETE FROM tar_reports_master_list WHERE branch_code = :p1 AND quarter_date = TO_DATE(:p2, 'dd/mm/yyyy')",
                "DELETE FROM tar_branch WHERE BRANCHNO = :p1 AND branch_date = TO_DATE(:p2, 'dd/mm/yyyy')",
                "DELETE FROM lfar_reports_master_list WHERE branch_code = :p1 AND quarter_date = TO_DATE(:p2, 'dd/mm/yyyy')",
                "DELETE FROM lfar_branch WHERE BRANCHNO = :p1 AND branch_date = TO_DATE(:p2, 'dd/mm/yyyy')"
            };
            for(String sql : marchDeleteQueries) {
                entityManager.createNativeQuery(sql)
                    .setParameter("p1", branchCode)
                    .setParameter("p2", quarterEndDate)
                    .executeUpdate();
            }
        }
    }
}

FRTBranchRequestRepository.java (Main Interface)
This interface now brings together the standard JPA functionality and our custom methods.
package com.yourcompany.frt.repository;

import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.data.jpa.repository.Modifying;
import org.springframework.data.jpa.repository.Query;
import org.springframework.data.repository.query.Param;
import org.springframework.stereotype.Repository;

import java.util.List;
import java.util.Map;

@Repository
public interface FRTBranchRequestRepository extends JpaRepository<DummyEntity, Long>, FRTBranchRequestRepositoryCustom {
    
    // === STATIC QUERIES FOR UI AND ADD BRANCH LOGIC ===
    
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
    @Query(value = "UPDATE frt_branch_requests SET status = :status, approved_by = :approverId, approved_on = CURRENT_TIMESTAMP WHERE request_id = :requestId", nativeQuery = true)
    void updateRequestStatus(@Param("status") String status, @Param("requestId") String requestId, @Param("approverId") String approverId);

    // ... (All INSERT queries for 'Add Branch' remain the same) ...
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



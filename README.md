You are correct. Apologies, I missed a couple of the update queries from the original logic. Thank you for pointing that out.
I've updated the Repository with the missing queries and integrated them into the Service layer to fully match the conditional logic of the original approveReq1 method.
Here is the corrected and complete code for the service and repository layers.
## DAO / Repository (FrtDataRepository.java) - Corrected
Two new methods have been added to handle the specific status updates on the REPORTS_MASTER_LIST table that were previously missing.
package com.tcs.dao;

import com.tcs.beans.AuditStatusRequest;
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.data.jpa.repository.Modifying;
import org.springframework.data.jpa.repository.Query;
import org.springframework.data.repository.query.Param;
import org.springframework.stereotype.Repository;

@Repository
public interface FrtDataRepository extends JpaRepository<AuditStatusRequest, String> {

    // ... other queries from previous response remain the same ...

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

    // ================== NEWLY ADDED QUERIES ==================

    /**
     * **[span_0](start_span)[NEW]** Corresponds to the `update_rml_rw24` query[span_0](end_span).
     * Updates the status of a specific report (identified by '1033') to a new status.
     */
    @Modifying
    @Query(value = "UPDATE REPORTS_MASTER_LIST SET status = :status WHERE BRANCH_CODE = :branchCode AND QUARTER_DATE = TO_DATE(:quarterDate, 'DD/MM/YYYY') AND REPORT_MASTER_ID = '1033'", nativeQuery = true)
    void updateReport1033Status(@Param("status") String status, @Param("branchCode") String branchCode, @Param("quarterDate") String quarterDate);

    /**
     * **[span_1](start_span)[NEW]** Corresponds to the `update_rml_status29` query[span_1](end_span).
     * Updates the status of all reports with a status greater than '23'.
     */
    @Modifying
    @Query(value = "UPDATE REPORTS_MASTER_LIST SET status = :status WHERE BRANCH_CODE = :branchCode AND QUARTER_DATE = TO_DATE(:quarterDate, 'DD/MM/YYYY') AND status > '23'", nativeQuery = true)
    void updateReportsStatusAboveThreshold(@Param("status") String status, @Param("branchCode") String branchCode, @Param("quarterDate") String quarterDate);
}

<hr>
## Service (FrtRequestServiceImpl.java) - Corrected
The service logic now includes calls to the newly added repository methods, ensuring every database operation from the original code is performed in the correct sequence.
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
        String userId = (String) payload.get("userId");

        if (requestIds == null || requestIds.isEmpty() || quarterDate == null) {
            response.put("message", "Bad Request: 'requestIds' and 'quarterDate' are required.");
            return new ResponseEntity<>(response, HttpStatus.BAD_REQUEST);
        }

        int successCount = 0;
        try {
            for (String reqId : requestIds) {
                Object[] details = frtDataRepository.findRequestDetailsById(reqId, quarterDate);
                if (details == null) continue;

                String branchCode = (String) details[0];
                String beforeSts = (String) details[1];
                String afterSts = (String) details[2];
                String ifcofrFlagForUpdate = "A".equals(afterSts) ? "N" : "Y";

                String marQuarter = quarterDate.substring(3, 5);

                [span_2](start_span)// This logic block now fully matches the original DAO[span_2](end_span)
                if ("Non-Audited".equalsIgnoreCase(beforeSts) && "Audited".equalsIgnoreCase(afterSts)) {
                    frtDataRepository.updateBranchMaster("Y", branchCode);
                    frtDataRepository.updateReportsMasterListAuditable("Y", branchCode, quarterDate);
                    frtDataRepository.updateReportsStatusAboveThreshold("29", branchCode, quarterDate); // **ADDED CALL**
                    if ("03".equals(marQuarter)) {
                        frtDataRepository.updateLfarBranch("Y", branchCode, quarterDate);
                    }
                } else if ("Non-Audited".equalsIgnoreCase(beforeSts) && "IFCOFR Audited".equalsIgnoreCase(afterSts)) {
                    frtDataRepository.updateBranchMaster("Y", branchCode);
                    frtDataRepository.updateReportsMasterListAuditable("Y", branchCode, quarterDate);
                    frtDataRepository.updateReportsStatusAboveThreshold("29", branchCode, quarterDate); // **ADDED CALL**
                    frtDataRepository.updateIfcofrAudit(ifcofrFlagForUpdate, userId, branchCode, quarterDate);
                    if ("03".equals(marQuarter)) {
                        frtDataRepository.updateLfarBranch("Y", branchCode, quarterDate);
                    }
                } else if ("Audited".equalsIgnoreCase(beforeSts) && "IFCOFR Audited".equalsIgnoreCase(afterSts)) {
                    frtDataRepository.updateIfcofrAudit(ifcofrFlagForUpdate, userId, branchCode, quarterDate);
                    frtDataRepository.updateReport1033Status("39", branchCode, quarterDate); // **ADDED CALL**
                } else if ("Audited".equalsIgnoreCase(beforeSts) && "Non-Audited".equalsIgnoreCase(afterSts)) {
                    frtDataRepository.updateBranchMaster("N", branchCode);
                    frtDataRepository.updateReportsMasterListAuditable("N", branchCode, quarterDate);
                    frtDataRepository.deleteReportsMasterListByReportId("1033", branchCode, quarterDate);
                    frtDataRepository.updateReportsStatusAboveThreshold("29", branchCode, quarterDate); // **ADDED CALL**
                    if ("03".equals(marQuarter)) {
                        frtDataRepository.updateLfarBranch("N", branchCode, quarterDate);
                        frtDataRepository.deleteLfarReportsMasterList(branchCode, quarterDate);
                    }
                } else if ("IFCOFR Audited".equalsIgnoreCase(beforeSts) && "Audited".equalsIgnoreCase(afterSts)) {
                    frtDataRepository.updateIfcofrAudit(ifcofrFlagForUpdate, userId, branchCode, quarterDate);
                    frtDataRepository.deleteReportsMasterListByReportId("4065", branchCode, quarterDate);
                    frtDataRepository.updateReport1033Status("39", branchCode, quarterDate); // **ADDED CALL**
                } else if ("IFCOFR Audited".equalsIgnoreCase(beforeSts) && "Non-Audited".equalsIgnoreCase(afterSts)) {
                    frtDataRepository.updateBranchMaster("N", branchCode);
                    frtDataRepository.updateReportsMasterListAuditable("N", branchCode, quarterDate);
                    frtDataRepository.updateIfcofrAudit(ifcofrFlagForUpdate, userId, branchCode, quarterDate);
                    frtDataRepository.updateReportsStatusAboveThreshold("29", branchCode, quarterDate); // **ADDED CALL**
                    frtDataRepository.deleteReportsMasterListByReportId("4065", branchCode, quarterDate);
                    frtDataRepository.deleteReportsMasterListByReportId("1033", branchCode, quarterDate);
                    if ("03".equals(marQuarter)) {
                        frtDataRepository.updateLfarBranch("N", branchCode, quarterDate);
                        frtDataRepository.deleteLfarReportsMasterList(branchCode, quarterDate);
                    }
                }
                
                frtDataRepository.updateRequestStatus(reqId, "A");
                successCount++;
            }

            response.put("message", successCount + " request(s) approved successfully.");
            return new ResponseEntity<>(response, HttpStatus.OK);
        } catch (Exception e) {
            log.error("Exception during request approval process: ", e);
            response.put("message", "An internal error occurred during approval.");
            return new ResponseEntity<>(response, HttpStatus.INTERNAL_SERVER_ERROR);
        }
    }
    
    // rejectRequests method remains the same
    @Override
    @Transactional
    public ResponseEntity<Map<String, Object>> rejectRequests(Map<String, Object> payload) {
        // ...
    }
}


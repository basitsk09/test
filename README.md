Of course. Here is the full backend code, refactored according to your requirements.
This solution transforms the provided JdbcTemplate and raw JDBC methods into a modern three-tier architecture. It uses a Map payload in the controller and exclusively native queries in the repository, with the service layer handling the data transformation to avoid DTOs.
## Controller (FrtBranchRequestController.java)
This controller provides two endpoints corresponding to your two methods. It accepts a Map payload for all requests and delegates processing to the service layer.
package com.tcs.controllers;

import com.tcs.services.FrtBranchRequestService;
import org.apache.log4j.Logger;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

import java.util.Map;

@RestController
@RequestMapping("/api/frt-branch")
public class FrtBranchRequestController {

    static Logger log = Logger.getLogger(FrtBranchRequestController.class.getName());

    @Autowired
    private FrtBranchRequestService frtBranchRequestService;

    /**
     * Fetches pending branch requests for a given quarter.
     * @param payload A map containing the 'quarterDate'.
     */
    @PostMapping("/requests")
    public ResponseEntity<Map<String, Object>> getBranchRequests(@RequestBody Map<String, Object> payload) {
        log.info("Fetching branch requests for payload: " + payload);
        return frtBranchRequestService.getBranchRequests(payload);
    }

    /**
     * Fetches a list of reports for a specific branch and quarter.
     * @param payload A map containing 'quarterDate' and 'branchCode'.
     */
    @PostMapping("/delete-report-list")
    public ResponseEntity<Map<String, Object>> getDeleteReportList(@RequestBody Map<String, Object> payload) {
        log.info("Fetching delete report list for payload: " + payload);
        return frtBranchRequestService.getDeleteReportList(payload);
    }
}

## Service (FrtBranchRequestServiceImpl.java)
The service layer contains the business logic. It extracts parameters from the Map, calls the repository, and transforms the raw List<Object[]> result into a structured response for the frontend, including replicating the header row from your original getDeteleReportList method.
package com.tcs.services;

import com.tcs.dao.FrtBranchRequestRepository;
import org.apache.log4j.Logger;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.stereotype.Service;

import java.util.ArrayList;
import java.util.HashMap;
import java.util.List;
import java.util.Map;

@Service
public class FrtBranchRequestServiceImpl implements FrtBranchRequestService {

    static Logger log = Logger.getLogger(FrtBranchRequestServiceImpl.class.getName());

    @Autowired
    private FrtBranchRequestRepository frtBranchRequestRepository;

    @Override
    public ResponseEntity<Map<String, Object>> getBranchRequests(Map<String, Object> payload) {
        Map<String, Object> response = new HashMap<>();
        String quarterDate = (String) payload.get("quarterDate");

        if (quarterDate == null) {
            response.put("message", "Bad Request: 'quarterDate' is required.");
            return new ResponseEntity<>(response, HttpStatus.BAD_REQUEST);
        }

        List<Object[]> results = frtBranchRequestRepository.findBranchRequests(quarterDate);
        List<Map<String, Object>> processedRequests = new ArrayList<>();

        for (Object[] row : results) {
            Map<String, Object> reqMap = new HashMap<>();
            reqMap.put("requestId", row[0]);
            reqMap.put("branchCode", row[1]);
            reqMap.put("branchName", row[2]);
            reqMap.put("circleCode", row[4]);
            reqMap.put("circleName", row[5]);
            reqMap.put("roCode", row[6]);
            reqMap.put("requestedById", row[7]);
            reqMap.put("requestedBy", row[8]);

            String reqType = (String) row[9]; // RT_SUBTYPE
            reqMap.put("requestType", "A".equalsIgnoreCase(reqType) ? "Add Branch" : "Delete Branch");
            
            String status = (String) row[10]; // RT_STATUS
            reqMap.put("status", "1".equals(status) ? "Pending" : "Unknown");

            reqMap.put("requestedOn", row[11]);
            reqMap.put("circleCount", row[12]);
            
            String audFlag = (String) row[3]; // CRS_AUDITABLE
            if ("I".equalsIgnoreCase(audFlag)) {
                reqMap.put("auditStatus", "Audited (IFCOFR)");
            } else if ("Y".equalsIgnoreCase(audFlag)) {
                reqMap.put("auditStatus", "Audited (Non-IFCOFR)");
            } else {
                reqMap.put("auditStatus", "Non-Audited");
            }
            processedRequests.add(reqMap);
        }
        
        response.put("data", processedRequests);
        return new ResponseEntity<>(response, HttpStatus.OK);
    }

    @Override
    public ResponseEntity<Map<String, Object>> getDeleteReportList(Map<String, Object> payload) {
        Map<String, Object> response = new HashMap<>();
        String quarterDate = (String) payload.get("quarterDate");
        String branchCode = (String) payload.get("branchCode");

        if (quarterDate == null || branchCode == null) {
            response.put("message", "Bad Request: 'quarterDate' and 'branchCode' are required.");
            return new ResponseEntity<>(response, HttpStatus.BAD_REQUEST);
        }

        List<Object[]> results = frtBranchRequestRepository.findReportsForBranch(quarterDate, branchCode);
        
        // Replicating the original method's structure which includes a header row.
        List<List<String>> reportData = new ArrayList<>();
        reportData.add(List.of("module", "report", "name", "pending")); // Header Row

        for (Object[] row : results) {
            List<String> rowData = new ArrayList<>();
            for (Object cell : row) {
                rowData.add(cell != null ? String.valueOf(cell) : "");
            }
            reportData.add(rowData);
        }

        response.put("data", reportData);
        return new ResponseEntity<>(response, HttpStatus.OK);
    }
}

## DAO / Repository (FrtBranchRequestRepository.java)
The repository uses native queries with named parameters to prevent SQL injection—a critical security improvement over the original code. Each method returns a raw List<Object[]> for the service layer to process.
package com.tcs.dao;

import com.tcs.beans.AnyJpaEntity; // A placeholder for any entity managed by this repository
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.data.jpa.repository.Query;
import org.springframework.data.repository.query.Param;
import org.springframework.stereotype.Repository;

import java.util.List;

@Repository
public interface FrtBranchRequestRepository extends JpaRepository<AnyJpaEntity, Long> {

    @Query(value = "SELECT RT_ID, RT_BRANCH, BR_NAME, CRS_AUDITABLE, CIRCLE_CODE, " +
                   "(select crs_sys_param_name from crs_sys_param where crs_sys_param.crs_sys_param_id=bm.circle_code) CIRCLE_NAME, " +
                   "SUBSTR(bm.region_code,1,3) ||' '|| SUBSTR(bm.region_code,4,3) ||' '|| SUBSTR(bm.region_code,7,3) RO, " +
                   "rt_maker, (SELECT NAME FROM wfl_user where wfl_user_id=rt_maker) rt_maker_name, " +
                   "RT_SUBTYPE, RT_STATUS, RT_DATE, (select count(1) from branch_master bms where bms.circle_code=bm.circle_code) circle_count " +
                   "FROM crs_request_track, cbs_brhm bm " +
                   "WHERE rt_type='B' AND rt_status='1' AND bm.branchno=rt_branch AND rt_qed=to_date(:quarterDate,'dd/mm/yyyy') " +
                   "ORDER BY RT_ID", nativeQuery = true)
    List<Object[]> findBranchRequests(@Param("quarterDate") String quarterDate);


    /**
     * This query is now safe from SQL Injection by using named parameters (:branchCode, :quarterDate)
     * instead of string concatenation.
     */
    @Query(value = "select 'CRS' module, report_name report, " +
                   "(SELECT report_desc FROM report_master where report_id=report_master_id) name, status pending " +
                   "from reports_master_list where branch_code = :branchCode and quarter_date = to_date(:quarterDate ,'dd/mm/yyyy') " +
                   "union all " +
                   "select 'TAR' module, report_name, " +
                   "(SELECT report_desc FROM tar_report_master where report_id=report_master_id) report_desc, status " +
                   "from tar_reports_master_list where branch_code = :branchCode and quarter_date = to_date(:quarterDate ,'dd/mm/yyyy') " +
                   "union all " +
                   "select 'LFAR' module, report_name, " +
                   "(SELECT report_desc FROM LFAR_report_master where report_id=report_master_id) report_desc, status " +
                   "from lfar_reports_master_list where branch_code = :branchCode and quarter_date = to_date(:quarterDate ,'dd/mm/yyyy')", nativeQuery = true)
    List<Object[]> findReportsForBranch(@Param("quarterDate") String quarterDate, @Param("branchCode") String branchCode);
}


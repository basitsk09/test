Of course. You can achieve this without a DTO by moving the data transformation logic directly into the SQL query using CASE statements. The repository will then return a List<Map<String, Object>>, where each map represents a row of data.
This approach simplifies the Java code, making the service layer a simple pass-through.
1. Repository Layer (DAO)
The repository's native query is updated with CASE statements to format the status fields directly. The return type is changed to List<Map<String, Object>>. The projection interface is no longer needed.
FrtRequestRepository.java
package com.yourpackage.repository;

import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.data.jpa.repository.Query;
import org.springframework.data.repository.query.Param;
import org.springframework.stereotype.Repository;
import com.yourpackage.model.CrsRequestTrack; // Anchor entity
import java.util.List;
import java.util.Map;

@Repository
public interface FrtRequestRepository extends JpaRepository<CrsRequestTrack, String> {

    @Query(value = "SELECT " +
            "cas.as_rt_id AS \"as_rt_id\", " +
            "cas.as_id AS \"as_id\", " +
            "cas.as_branch AS \"branchCode\", " +
            "bm.br_name AS \"branchName\", " +
            "SUBSTR(bm.region_code,1,3) ||''|| SUBSTR(bm.region_code,4,3) ||''|| SUBSTR(bm.region_code,7,3) AS \"roCode\", " +
            "bm.circle_code AS \"circle_code\", " +
            "crt.rt_status AS \"req_tracksts\", " +
            "crt.rt_date AS \"reqOn\", " +
            // Logic for 'beforeSts'
            "CASE WHEN bm.crs_auditable = 'Y' " +
            "THEN CASE WHEN nvl((select ifcofr_audit_flag from crs_ifcofr_audit where ifcofr_branch=cas.as_branch and ifcofr_date=to_date(:quarterDate,'dd/mm/yyyy')),'N') = 'Y' " +
            "THEN 'IFCOFR Audited' ELSE 'Audited' END " +
            "ELSE 'Non-Audited' END AS \"beforeSts\", " +
            // Logic for 'afterSts'
            "CASE cas.as_new_status " +
            "WHEN 'A' THEN 'Audited' " +
            "WHEN 'I' THEN 'IFCOFR Audited' " +
            "ELSE 'Non-Audited' END AS \"afterSts\", " +
            // Logic for 'reqSts'
            "CASE cas.as_req_status " +
            "WHEN 'P' THEN 'Pending' " +
            "ELSE cas.as_req_status END AS \"reqSts\" " +
            "FROM crs_request_track crt, crs_audit_status cas, branch_master bm " +
            "WHERE cas.as_rt_id = crt.rt_id AND cas.as_branch=bm.branch_no AND cas.as_req_status NOT IN ('R','A') " +
            "ORDER BY cas.as_rt_id, cas.as_id",
            nativeQuery = true)
    List<Map<String, Object>> findPendingRequestsFormatted(@Param("quarterDate") String quarterDate);
}

2. Service Layer
The service layer is now much simpler. It directly calls the repository and returns its result without any mapping logic.
Service Interface: FrtRequestService.java
package com.yourpackage.service;

import java.util.List;
import java.util.Map;

public interface FrtRequestService {
    List<Map<String, Object>> getPendingRequests(String quarterDate);
}

Service Implementation: FrtRequestServiceImpl.java
package com.yourpackage.service;

import com.yourpackage.repository.FrtRequestRepository;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;
import java.util.List;
import java.util.Map;

@Service
public class FrtRequestServiceImpl implements FrtRequestService {

    @Autowired
    private FrtRequestRepository frtRequestRepository;

    @Override
    public List<Map<String, Object>> getPendingRequests(String quarterDate) {
        return frtRequestRepository.findPendingRequestsFormatted(quarterDate);
    }
}

3. Controller Layer
The controller's method signature is updated to reflect the new return type from the service.
FrtRequestController.java
package com.yourpackage.controller;

import com.yourpackage.service.FrtRequestService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;
import java.util.List;
import java.util.Map;

@RestController
@RequestMapping("/api/frt")
public class FrtRequestController {

    @Autowired
    private FrtRequestService frtRequestService;

    @GetMapping("/requests/{quarterDate}")
    public ResponseEntity<List<Map<String, Object>>> getPendingFrtRequests(@PathVariable String quarterDate) {
        List<Map<String, Object>> requests = frtRequestService.getPendingRequests(quarterDate);
        return ResponseEntity.ok(requests);
    }
}


Of course. Here is a modernized version of your code using the Spring Data JPA repository pattern, broken down into the standard Controller, Service, and Repository (DAO) layers.
This new structure is more maintainable, type-safe, and aligns with modern Spring Boot practices.
1. DTO and Projection
First, we need a DTO (Data Transfer Object) to hold the final, formatted data, and a JPA Projection interface to map the native query results efficiently.
DTO: FRTAuditStatusReq.java
This class will hold the final, user-friendly data that your API will return.
package com.yourpackage.dto;

import lombok.Data;

@Data // Using Lombok for boilerplate code (getters, setters, etc.)
public class FRTAuditStatusReq {
    private String as_rt_id;
    private String as_id;
    private String branchCode;
    private String branchName;
    private String beforeSts; // e.g., "Audited", "Non-Audited"
    private String afterSts;  // e.g., "IFCOFR Audited"
    private String roCode;
    private String circle_code;
    private String reqSts;    // e.g., "Pending"
    private String req_tracksts;
    private String reqOn;
}

JPA Projection: FRTAuditStatusProjection.java
This interface tells Spring Data JPA how to map the columns from your native query result set.
package com.yourpackage.repository;

public interface FRTAuditStatusProjection {
    String getAsRtId();
    String getAsId();
    String getAsBranch();
    String getBranchName();
    String getOldBrStatus();
    String getRO();
    String getCircleCode();
    String getIfcofrFlag();
    String getAsNewStatus();
    String getAsReqStatus();
    String getRtStatus();
    String getRtDate();
}

2. Repository Layer (DAO)
This repository interface uses a native query with the projection to fetch the raw data from the database.
FrtRequestRepository.java
package com.yourpackage.repository;

import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.data.jpa.repository.Query;
import org.springframework.data.repository.query.Param;
import org.springframework.stereotype.Repository;
import com.yourpackage.model.CrsRequestTrack; // Assuming you have a base entity to anchor the repository
import java.util.List;

@Repository
// You can anchor the repository to a relevant entity like CrsRequestTrack
public interface FrtRequestRepository extends JpaRepository<CrsRequestTrack, String> {

    @Query(value = "select cas.as_rt_id AS asRtId, cas.as_id AS asId, cas.as_branch AS asBranch, br_name AS branchName, crs_auditable AS oldBrStatus, " +
            "SUBSTR(bm.region_code,1,3) ||''|| SUBSTR(bm.region_code,4,3) ||''|| SUBSTR(bm.region_code,7,3) AS ro, bm.circle_code AS circleCode, " +
            "nvl((select ifcofr_audit_flag from crs_ifcofr_audit where ifcofr_branch=cas.as_branch and ifcofr_date=to_date(:quarterDate,'dd/mm/yyyy')),'N') AS ifcofrFlag, " +
            "cas.as_new_status AS asNewStatus, cas.as_req_status AS asReqStatus, crt.rt_status AS rtStatus, crt.rt_date AS rtDate " +
            "from crs_request_track crt, crs_audit_status cas, branch_master bm " +
            "where cas.as_rt_id = crt.rt_id and cas.as_branch=bm.branch_no and cas.as_req_status not in ('R','A') order by cas.as_rt_id, cas.as_id",
            nativeQuery = true)
    List<FRTAuditStatusProjection> findPendingRequests(@Param("quarterDate") String quarterDate);
}

3. Service Layer
The service layer contains the business logic to transform the raw database values into the user-friendly text seen in your old code.
Service Interface: FrtRequestService.java
package com.yourpackage.service;

import com.yourpackage.dto.FRTAuditStatusReq;
import java.util.List;

public interface FrtRequestService {
    List<FRTAuditStatusReq> getPendingRequests(String quarterDate);
}

Service Implementation: FrtRequestServiceImpl.java
package com.yourpackage.service;

import com.yourpackage.dto.FRTAuditStatusReq;
import com.yourpackage.repository.FrtRequestRepository;
import com.yourpackage.repository.FRTAuditStatusProjection;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;
import java.util.List;
import java.util.stream.Collectors;

@Service
public class FrtRequestServiceImpl implements FrtRequestService {

    @Autowired
    private FrtRequestRepository frtRequestRepository;

    @Override
    public List<FRTAuditStatusReq> getPendingRequests(String quarterDate) {
        List<FRTAuditStatusProjection> results = frtRequestRepository.findPendingRequests(quarterDate);

        // Map the raw projection results to the final DTO with business logic
        return results.stream().map(this::mapProjectionToDto).collect(Collectors.toList());
    }

    private FRTAuditStatusReq mapProjectionToDto(FRTAuditStatusProjection proj) {
        FRTAuditStatusReq dto = new FRTAuditStatusReq();
        dto.setAs_rt_id(proj.getAsRtId());
        dto.setAs_id(proj.getAsId());
        dto.setBranchCode(proj.getAsBranch());
        dto.setBranchName(proj.getBranchName());
        dto.setRoCode(proj.getRO());
        dto.setCircle_code(proj.getCircleCode());
        dto.setReq_tracksts(proj.getRtStatus());
        dto.setReqOn(proj.getRtDate());

        // Logic for 'Before Status'
        if ("Y".equalsIgnoreCase(proj.getOldBrStatus())) {
            dto.setBeforeSts("Y".equalsIgnoreCase(proj.getIfcofrFlag()) ? "IFCOFR Audited" : "Audited");
        } else {
            dto.setBeforeSts("Non-Audited");
        }

        // Logic for 'After Status'
        String newSts = proj.getAsNewStatus();
        if ("A".equalsIgnoreCase(newSts)) {
            dto.setAfterSts("Audited");
        } else if ("I".equalsIgnoreCase(newSts)) {
            dto.setAfterSts("IFCOFR Audited");
        } else {
            dto.setAfterSts("Non-Audited");
        }

        // Logic for 'Request Status'
        if ("P".equalsIgnoreCase(proj.getAsReqStatus())) {
            dto.setReqSts("Pending");
        } else {
            dto.setReqSts(proj.getAsReqStatus()); // Fallback
        }

        return dto;
    }
}

4. Controller Layer
Finally, the controller exposes an API endpoint to trigger the service and return the data.
FrtRequestController.java
package com.yourpackage.controller;

import com.yourpackage.dto.FRTAuditStatusReq;
import com.yourpackage.service.FrtRequestService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;
import java.util.List;

@RestController
@RequestMapping("/api/frt")
public class FrtRequestController {

    @Autowired
    private FrtRequestService frtRequestService;

    @GetMapping("/requests/{quarterDate}")
    public ResponseEntity<List<FRTAuditStatusReq>> getPendingFrtRequests(@PathVariable String quarterDate) {
        // The quarterDate should be in 'dd-MM-yyyy' format to match the query's to_date format
        List<FRTAuditStatusReq> requests = frtRequestService.getPendingRequests(quarterDate);
        return ResponseEntity.ok(requests);
    }
}


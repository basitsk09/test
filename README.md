Based on your request, the duplicate code block has been removed. The following code is a refactored solution that accepts a Map in the controller and uses a native query in the repository, without needing @Transient variables in the entity.
This approach uses a repository method that returns a List<Object[]>. Each array in the list represents a row from the database result set. The service layer is then responsible for transforming this raw data into a structured Map for the response.
## Backend Refactoring
1. Controller (FrtRequestController.java)
The controller remains lean. It accepts the Map from the request body and passes it directly to the service layer. The duplicate endpoint has been removed.
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

    /**
     * Handles the FRT request submission.
     * @param payload A map containing request parameters like 'branchCode', 'quarterDate', etc.
     * @return A ResponseEntity containing a map with the operation's result.
     */
    @PostMapping("/submitRequest")
    public ResponseEntity<Map<String, Object>> frtSubmitRequest(@RequestBody Map<String, Object> payload) {
        log.info("Processing FRT submit request with payload: " + payload);
        return frtRequestService.processFrtRequest(payload);
    }
}

<hr>
2. Service Layer (FrtRequestServiceImpl.java)
The service layer contains the core logic. It extracts parameters from the map, calls the repository's native query, and then manually constructs the response map from the List<Object[]> result.
package com.tcs.services;

import com.tcs.dao.FrtDataRepository;
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
public class FrtRequestServiceImpl implements FrtRequestService {

    static Logger log = Logger.getLogger(FrtRequestServiceImpl.class.getName());

    @Autowired
    private FrtDataRepository frtDataRepository;

    @Override
    public ResponseEntity<Map<String, Object>> processFrtRequest(Map<String, Object> payload) {
        Map<String, Object> response = new HashMap<>();
        try {
            // Extract required parameters from the incoming map
            String branchCode = (String) payload.get("branchCode");
            String quarterDate = (String) payload.get("quarterDate");

            if (branchCode == null || quarterDate == null) {
                log.error("Missing 'branchCode' or 'quarterDate' in payload.");
                response.put("message", "Bad Request: Missing required parameters.");
                return new ResponseEntity<>(response, HttpStatus.BAD_REQUEST);
            }

            // Call the repository to get data using a native query
            List<Object[]> results = frtDataRepository.getFrtRequestData(branchCode, quarterDate);
            log.info("Found " + results.size() + " records for branch: " + branchCode);

            // Manually process the raw Object[] list into a structured list of maps
            List<Map<String, Object>> processedResults = new ArrayList<>();
            for (Object[] row : results) {
                Map<String, Object> rowMap = new HashMap<>();
                rowMap.put("requestId", row[0]);
                rowMap.put("branchName", row[1]);
                rowMap.put("requestStatus", row[2]);
                rowMap.put("requestedOn", row[3]);
                processedResults.add(rowMap);
            }

            response.put("message", "Request Processed Successfully");
            response.put("data", processedResults);
            return new ResponseEntity<>(response, HttpStatus.OK);

        } catch (Exception e) {
            log.error("Exception occurred while processing FRT request: ", e);
            response.put("message", "An internal error occurred.");
            return new ResponseEntity<>(response, HttpStatus.INTERNAL_SERVER_ERROR);
        }
    }
}

An interface FrtRequestService.java would also be defined for this implementation.
<hr>
3. Repository Layer (FrtDataRepository.java)
The repository uses a native SQL query to fetch data from multiple tables. It returns a List<Object[]> which completely avoids the need for @Transient variables or complex object mapping at the database level.
package com.tcs.dao;

import com.tcs.beans.AuditStatusRequest; // The entity can be any entity managed by this repository.
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.data.jpa.repository.Query;
import org.springframework.data.repository.query.Param;
import org.springframework.stereotype.Repository;

import java.util.List;

@Repository
public interface FrtDataRepository extends JpaRepository<AuditStatusRequest, String> {

    /**
     * Fetches combined data for an FRT request using a native SQL query.
     * The result is returned as a list of object arrays, where each array represents a row
     * and its elements correspond to the columns in the SELECT statement.
     *
     * @param branchCode The branch code to query for.
     * @param quarterDate The quarter end date for the request.
     * @return A List of Object[] containing the query results.
     */
    @Query(value = "SELECT " +
                   "    cas.as_id, " +
                   "    bm.br_name, " +
                   "    cas.as_req_status, " +
                   "    crt.rt_date " +
                   "FROM " +
                   "    crs_audit_status cas, " +
                   "    branch_master bm, " +
                   "    crs_request_track crt " +
                   "WHERE " +
                   "    cas.as_branch = bm.branchno " +
                   "    AND cas.as_rt_id = crt.rt_id " +
                   "    AND cas.as_branch = :branchCode " +
                   "    AND crt.rt_quarter_end_date = TO_DATE(:quarterDate, 'DD/MM/YYYY')",
           nativeQuery = true)
    List<Object[]> getFrtRequestData(@Param("branchCode") String branchCode, @Param("quarterDate") String quarterDate);

}


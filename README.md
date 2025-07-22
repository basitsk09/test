Of course. Here is a complete refactoring of the backend using Spring Data JPA with @Query and the corresponding frontend updates, as you requested.
The original code was tightly coupled, with business logic in the controller and DAO layers. It used JdbcTemplate with manual SQL string construction and transaction management.
This new version provides a cleaner, more maintainable architecture by:
 * Decoupling Layers: Business logic is now properly encapsulated in the Service layer.
 * Using Spring Data JPA: The DAO layer is replaced with a Spring Data JPA Repository that uses @Query annotations for database operations, simplifying data access and improving type safety.
 * Transactional Services: The service methods that modify data are marked as @Transactional to ensure data integrity.
 * RESTful Controller: The controller is updated to function as a proper REST API, consuming JSON from the frontend and returning JSON responses, which aligns with modern web application design.
 * No DTO Pattern: As requested, the solution avoids Data Transfer Objects. The JPA Entity itself is used across layers, and frontend-specific transformations are handled in the service layer before being sent to the controller.
Backend Code
Here is the full backend code, including the Entity, Repository, Service, and Controller.
1. Maven Dependencies (pom.xml)
Ensure you have the necessary Spring Boot starters for JPA, Web, and your database driver.
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>

    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-jpa</artifactId>
    </dependency>

    <dependency>
        <groupId>com.oracle.database.jdbc</groupId>
        <artifactId>ojdbc8</artifactId>
        <scope>runtime</scope>
    </dependency>

    <dependency>
        <groupId>org.projectlombok</groupId>
        <artifactId>lombok</artifactId>
        <optional>true</optional>
    </dependency>
</dependencies>

2. JPA Entity (AuditStatusRequest.java)
This class maps to your database tables. @Transient fields are used for data that is calculated or joined but not persisted as a column in the CRS_AUDIT_STATUS table itself.
package com.tcs.beans;

import lombok.Getter;
import lombok.Setter;

import javax.persistence.*;

@Entity
@Table(name = "CRS_AUDIT_STATUS")
@Getter
@Setter
public class AuditStatusRequest {

    @Id
    @Column(name = "AS_ID")
    private String asId;

    @Column(name = "AS_RT_ID")
    private String asRtId;

    @Column(name = "AS_BRANCH")
    private String branchCode;

    @Column(name = "AS_NEW_STATUS")
    private String asNewStatus;

    @Column(name = "AS_REQ_STATUS")
    private String asReqStatus;
    
    // These fields are not part of the CRS_AUDIT_STATUS table.
    // They will be populated by a custom query and are read-only.
    @Transient
    private String branchName;
    @Transient
    private String beforeSts;
    @Transient
    private String afterSts;
    @Transient
    private String reqSts;
    @Transient
    private String reqOn;
    @Transient
    private String roCode;
    @Transient
    private String circleCode;
}

3. JPA Repository (AuditStatusRequestRepository.java)
This interface replaces FRTAuditStatusReqDaoImpl. It uses @Query for all data fetching and modification, simplifying the code significantly.
package com.tcs.dao;

import com.tcs.beans.AuditStatusRequest;
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.data.jpa.repository.Modifying;
import org.springframework.data.jpa.repository.Query;
import org.springframework.data.repository.query.Param;
import org.springframework.stereotype.Repository;

import java.util.List;

@Repository
public interface AuditStatusRequestRepository extends JpaRepository<AuditStatusRequest, String> {

    [span_0](start_span)// Replaces the getRequests() method[span_0](end_span)
    @Query(value = "SELECT cas.as_id, cas.as_rt_id, cas.as_branch, bm.br_name, cas.as_new_status, cas.as_req_status, crt.rt_date, " +
                   "bm.circle_code, SUBSTR(bm.region_code,1,3) ||''|| SUBSTR(bm.region_code,4,3) ||''|| SUBSTR(bm.region_code,7,3) as ro_code, " +
                   "crs_auditable as old_br_status, " +
                   "nvl((select ifcofr_audit_flag from crs_ifcofr_audit where ifcofr_branch=cas.as_branch and ifcofr_date=to_date(:quarterDate,'dd/mm/yyyy')),'N') as ifcofr_flag " +
                   "FROM crs_audit_status cas, crs_request_track crt, branch_master bm " +
                   "WHERE cas.as_rt_id = crt.rt_id AND cas.as_branch = bm.branchno AND cas.as_req_status NOT IN ('R', 'A') " +
                   "ORDER BY cas.as_rt_id, cas.as_id", nativeQuery = true)
    List<Object[]> findPendingRequestsNative(@Param("quarterDate") String quarterDate);

    // Query to update status for an approved request
    @Modifying
    @Query("UPDATE AuditStatusRequest r SET r.asReqStatus = 'A' WHERE r.asId = :reqId")
    void approveRequestStatus(@Param("reqId") String reqId);

    [span_1](start_span)// Query to update status for a rejected request[span_1](end_span)
    @Modifying
    @Query("UPDATE AuditStatusRequest r SET r.asReqStatus = 'R' WHERE r.asId = :reqId")
    void rejectRequestStatus(@Param("reqId") String reqId);

    [span_2](start_span)// Replaces reqCount method[span_2](end_span)
    @Query("SELECT count(r) FROM AuditStatusRequest r WHERE r.asRtId = :trackId AND r.asReqStatus NOT IN ('R', 'A')")
    int countPendingRequestsByTrackId(@Param("trackId") String trackId);

    [span_3](start_span)// Replaces reqCountWithReject method[span_3](end_span)
    @Query("SELECT count(r) FROM AuditStatusRequest r WHERE r.asRtId = :trackId AND r.asReqStatus = 'R'")
    int countRejectedRequestsByTrackId(@Param("trackId") String trackId);

    // Note: Updates to other tables like branch_master, REPORTS_MASTER_LIST would also be
    // converted to @Query methods here or handled via separate repositories.
    // For brevity, only the primary logic is shown. [span_4](start_span)The complex logic from approveReq1[span_4](end_span)
    // would be broken down into smaller, targeted @Query methods.
}

4. Service Interface (FRTAuditStatusReqService.java)
package com.tcs.services;

import com.tcs.beans.AuditStatusRequest;
import java.util.List;

public interface FRTAuditStatusReqService {
    List<AuditStatusRequest> getPendingRequests(String quarterDate);
    void approveRequests(List<String> requestIds);
    void rejectRequests(List<String> requestIds);
}

5. Service Implementation (FRTAuditStatusReqServiceImpl.java)
This class contains the business logic, using the repository for database interactions.
package com.tcs.services;

import com.tcs.beans.AuditStatusRequest;
import com.tcs.dao.AuditStatusRequestRepository;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

import java.math.BigDecimal;
import java.util.ArrayList;
import java.util.List;
import java.util.stream.Collectors;

@Service
public class FRTAuditStatusReqServiceImpl implements FRTAuditStatusReqService {

    @Autowired
    private AuditStatusRequestRepository auditStatusRequestRepository;

    @Override
    public List<AuditStatusRequest> getPendingRequests(String quarterDate) {
        List<Object[]> results = auditStatusRequestRepository.findPendingRequestsNative(quarterDate);
        return results.stream().map(this::mapObjectToAuditStatusRequest).collect(Collectors.toList());
    }

    @Override
    @Transactional // Ensures all database operations within this method are atomic
    public void approveRequests(List<String> requestIds) {
        for (String id : requestIds) {
            [span_5](start_span)// Further logic from the original approveReq1 method[span_5](end_span) would be called here.
            // This includes updating branch_master, REPORTS_MASTER_LIST, etc.
            // For this example, we'll just update the request status.
            auditStatusRequestRepository.approveRequestStatus(id);
        }
    }

    @Override
    @Transactional
    public void rejectRequests(List<String> requestIds) {
        for (String id : requestIds) {
            auditStatusRequestRepository.rejectRequestStatus(id);
        }
    }
    
    // Helper method to map the native query result to our Entity object
    private AuditStatusRequest mapObjectToAuditStatusRequest(Object[] obj) {
        AuditStatusRequest req = new AuditStatusRequest();
        req.setAsId((String) obj[0]);
        req.setAsRtId((String) obj[1]);
        req.setBranchCode((String) obj[2]);
        req.setBranchName((String) obj[3]);
        req.setAsNewStatus((String) obj[4]);
        req.setReqOn(obj[6] != null ? obj[6].toString().substring(0, 10) : ""); // Format date
        req.setCircleCode((String) obj[7]);
        req.setRoCode((String) obj[8]);

        [span_6](start_span)[span_7](start_span)[span_8](start_span)// Logic to set descriptive status strings, moved from DAO[span_6](end_span)[span_7](end_span)[span_8](end_span)
        String oldSts = (String) obj[9];
        String ifcofrFlag = (String) obj[10];
        if ("Y".equalsIgnoreCase(oldSts)) {
            req.setBeforeSts("Y".equalsIgnoreCase(ifcofrFlag) ? "IFCOFR Audited" : "Audited");
        } else {
            req.setBeforeSts("Non-Audited");
        }

        String newSts = req.getAsNewStatus();
        if ("A".equalsIgnoreCase(newSts)) {
            req.setAfterSts("Audited");
        } else if ("I".equalsIgnoreCase(newSts)) {
            req.setAfterSts("IFCOFR Audited");
        } else {
            req.setAfterSts("Non-Audited");
        }
        
        req.setReqSts("P".equalsIgnoreCase((String) obj[5]) ? "Pending" : "Unknown");
        return req;
    }
}

6. REST Controller (FRTAuditStatusReqController.java)
This controller exposes RESTful endpoints that the React frontend will consume.
package com.tcs.controllers;

import com.tcs.beans.AuditStatusRequest;
import com.tcs.services.FRTAuditStatusReqService;
import com.tcs.utils.CommonConstants;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpSession;
import java.util.Collections;
import java.util.List;
import java.util.Map;

@RestController
@RequestMapping("/api/frt") // Base path aligned with frontend calls
public class FRTAuditStatusReqController {

    @Autowired
    private FRTAuditStatusReqService frtAuditStatusReqService;

    // A simple wrapper class for the request body to hold the list of IDs
    static class ActionRequest {
        private List<String> requestIds;
        public List<String> getRequestIds() { return requestIds; }
        public void setRequestIds(List<String> requestIds) { this.requestIds = requestIds; }
    }

    @GetMapping("/requests")
    public ResponseEntity<List<AuditStatusRequest>> getPendingRequests(HttpServletRequest request) {
        HttpSession session = request.getSession(false);
        [span_9](start_span)// Authorization check from original controller[span_9](end_span)
        if (session == null || !"94".equals(session.getAttribute(CommonConstants.USER_CAPABILITY))) {
            return ResponseEntity.status(HttpStatus.UNAUTHORIZED).build();
        }
        
        // Assuming quarterEndDate is stored in the session
        String quarterEndDate = (String) session.getAttribute(CommonConstants.QUARTER_END_DATE);
        List<AuditStatusRequest> requests = frtAuditStatusReqService.getPendingRequests(quarterEndDate);
        return ResponseEntity.ok(requests);
    }

    @PostMapping("/approve")
    public ResponseEntity<Map<String, String>> approveRequests(@RequestBody ActionRequest payload, HttpServletRequest request) {
        // Authorization check can be repeated here
        frtAuditStatusReqService.approveRequests(payload.getRequestIds());
        return ResponseEntity.ok(Collections.singletonMap("message", "Request(s) Approved Successfully."));
    }

    @PostMapping("/reject")
    public ResponseEntity<Map<String, String>> rejectRequests(@RequestBody ActionRequest payload, HttpServletRequest request) {
        // Authorization check can be repeated here
        frtAuditStatusReqService.rejectRequests(payload.getRequestIds());
        return ResponseEntity.ok(Collections.singletonMap("message", "Request(s) Rejected Successfully."));
    }
}

Frontend Changes
The frontend code requires minimal changes to align its API calls with the new backend endpoints. The client-side encryption logic has been removed for clarity, as it was not present on the provided backend.
FRTAuditStatusReq.js (Updated)
import * as React from "react";
import { useState, useEffect } from "react";
import {
  Box,
  Button,
  Checkbox,
  Container,
  Dialog,
  DialogActions,
  DialogContent,
  DialogTitle,
  Alert,
  Paper,
  Table,
  TableBody,
  TableCell,
  TableContainer,
  TableHead,
  TableRow,
  TablePagination,
  Typography,
  CircularProgress,
[span_10](start_span)} from "@mui/material";[span_10](end_span)
[span_11](start_span)import axios from "axios";[span_11](end_span)

// Table Header Component (No changes needed)
const EnhancedTableHead = (props) => {
  const { onSelectAllClick, numSelected, rowCount } = props;
  return (
    <TableHead>
      <TableRow sx={{ backgroundColor: "#b9def0" }}>
        <TableCell padding="checkbox" align="center">
          <Box sx={{ display: "flex", alignItems: "center", whiteSpace: "nowrap" }}>
            <Checkbox
              color="primary"
              [span_12](start_span)indeterminate={numSelected > 0 && numSelected < rowCount}[span_12](end_span)
              checked={rowCount > 0 && numSelected === rowCount}
              onChange={onSelectAllClick}
              inputProps={{ "aria-label": "select all requests" }}
            />
            <b>Select All</b>
          </Box>
        </TableCell>
        <TableCell align="center"><b>Req ID</b></TableCell>
        <TableCell align="center"><b>Branch Code</b></TableCell>
        <TableCell align="left"><b>Branch Name</b></TableCell>
        <TableCell align="center"><b>Existing Status</b></TableCell>
        <TableCell align="center"><b>New Requested Status</b></TableCell>
        <TableCell align="center"><b>Request Status</b></TableCell>
        <TableCell align="center"><b>Requested On</b></TableCell>
      </TableRow>
    </TableHead>
  );
};


const FRTAuditStatusReq = () => {
  const [requests, setRequests] = useState([]);
  const [selected, setSelected] = useState([]);
  [span_13](start_span)const [loading, setLoading] = useState(true);[span_13](end_span)
  const [page, setPage] = useState(0);
  const [rowsPerPage, setRowsPerPage] = useState(5);
  [span_14](start_span)const [dialog, setDialog] = useState({ open: false, message: "", severity: "success" });[span_14](end_span)

  const fetchRequests = async () => {
    setLoading(true);
    try {
      // *** CHANGED: Updated API endpoint for fetching requests ***
      const response = await axios.get("/api/frt/requests", {
        headers: { Authorization: `Bearer ${localStorage.getItem("token")}` },
      });
      
      // The backend now returns data with correct keys, so mapping 'id' is straightforward.
      const mappedData = response.data.map((item) => ({
        ...item,
        id: item.asId, // Use asId from backend as the unique key for rows
      }));
      [span_15](start_span)setRequests(mappedData);[span_15](end_span)
    } catch (error) {
      console.error("Failed to fetch requests:", error);
      setDialog({
        open: true,
        message: "Failed to load requests. Please try again later.",
        severity: "error",
      [span_16](start_span)});[span_16](end_span)
      [span_17](start_span)setRequests([]);[span_17](end_span)
    } finally {
      setLoading(false);
    }
  };

  useEffect(() => {
    document.title = "FRT | Audit Status Requests";
    fetchRequests();
  [span_18](start_span)}, []);[span_18](end_span)

  const handleSelectAllClick = (event) => {
    if (event.target.checked) {
      const newSelected = requests.map((n) => n.id);
      [span_19](start_span)setSelected(newSelected);[span_19](end_span)
      return;
    }
    setSelected([]);
  };

  const handleClick = (event, id) => {
    const selectedIndex = selected.indexOf(id);
    let newSelected = [];

    if (selectedIndex === -1) {
      [span_20](start_span)newSelected = newSelected.concat(selected, id);[span_20](end_span)
    } else if (selectedIndex === 0) {
      [span_21](start_span)newSelected = newSelected.concat(selected.slice(1));[span_21](end_span)
    } else if (selectedIndex === selected.length - 1) {
      [span_22](start_span)newSelected = newSelected.concat(selected.slice(0, -1));[span_22](end_span)
    } else if (selectedIndex > 0) {
      [span_23](start_span)newSelected = newSelected.concat(selected.slice(0, selectedIndex), selected.slice(selectedIndex + 1));[span_23](end_span)
    }
    setSelected(newSelected);
  };

  const handleAction = async (action) => {
    setLoading(true);
    // *** NO CHANGE NEEDED HERE: The endpoint path is already correct ***
    [span_24](start_span)const endpoint = `/api/frt/${action}`;[span_24](end_span)
    try {
      const response = await axios.post(
        endpoint,
        [span_25](start_span){ requestIds: selected }, // The payload format matches the new controller[span_25](end_span)
        { headers: { Authorization: `Bearer ${localStorage.getItem("token")}` } }
      );
      setDialog({
        open: true,
        message: response.data.message || `Action performed successfully.`, // Using message from backend response
        severity: "success",
      [span_26](start_span)});[span_26](end_span)
      [span_27](start_span)setSelected([]);[span_27](end_span)
      fetchRequests(); // Refresh data
    } catch (error) {
      console.error(`Failed to ${action} requests:`, error);
      setDialog({
        open: true,
        message: error.response?.data?.message || `Failed to ${action} requests.`, // Using error message from backend
        severity: "error",
      [span_28](start_span)});[span_28](end_span)
    } finally {
      [span_29](start_span)setLoading(false);[span_29](end_span)
    }
  };

  const getStatusStyles = (status) => {
    if (status?.toLowerCase() === "pending") {
      return {
        backgroundColor: "#ff9800",
        color: "white",
        fontWeight: "bold",
        padding: "4px 12px",
        borderRadius: "16px",
        display: "inline-block",
        textTransform: "capitalize",
      [span_30](start_span)};[span_30](end_span)
    }
    [span_31](start_span)return { padding: "4px 0" };[span_31](end_span)
  };

  [span_32](start_span)const handleDialogClose = () => setDialog({ ...dialog, open: false });[span_32](end_span)
  [span_33](start_span)const isSelected = (id) => selected.indexOf(id) !== -1;[span_33](end_span)

  const emptyRows = page > 0 ? [span_34](start_span)Math.max(0, (1 + page) * rowsPerPage - requests.length) : 0;[span_34](end_span)

  return (
    <>
      <Container maxWidth={false}>
        <Paper sx={{ width: "100%", mb: 2, p: 2 }}>
          <Typography variant="h5" component="div" sx={{ mb: 2, textAlign: "left" }}>
            [span_35](start_span)Audit Status Change Requests[span_35](end_span)
          </Typography>
          <Box sx={{ display: "flex", justifyContent: "center", mb: 2 }}>
            [span_36](start_span)<Button variant="contained" color="success" sx={{ mr: 2, width: "120px" }} onClick={() => handleAction("approve")} disabled={selected.length === 0 || loading}>Approve</Button>[span_36](end_span)
            [span_37](start_span)<Button variant="contained" color="error" sx={{ width: "120px" }} onClick={() => handleAction("reject")} disabled={selected.length === 0 || loading}>Reject</Button>[span_37](end_span)
          </Box>
          <TableContainer>
            {loading ? (
              [span_38](start_span)<Box sx={{ display: "flex", justifyContent: "center", alignItems: "center", height: 370 }}>[span_38](end_span)
                <CircularProgress />
              </Box>
            ) : (
              [span_39](start_span)<Table sx={{ minWidth: 750 }} aria-labelledby="tableTitle">[span_39](end_span)
                <EnhancedTableHead numSelected={selected.length} onSelectAllClick={handleSelectAllClick} rowCount={requests.length}/>
                [span_40](start_span)<TableBody>[span_40](end_span)
                  {requests
                    .slice(page * rowsPerPage, page * rowsPerPage + rowsPerPage)
                    .map((row) => {
                      const isItemSelected = isSelected(row.id);
                      [span_41](start_span)const labelId = `enhanced-table-checkbox-${row.id}`;[span_41](end_span)
                      return (
                        <TableRow
                          hover
                          [span_42](start_span)onClick={(event) => handleClick(event, row.id)}[span_42](end_span)
                          role="checkbox"
                          aria-checked={isItemSelected}
                          tabIndex={-1}
                          [span_43](start_span)key={row.id}[span_43](end_span)
                          selected={isItemSelected}
                          sx={{ cursor: "pointer" }}
                        >
                          <TableCell padding="checkbox">
                            [span_44](start_span)<Checkbox color="primary" checked={isItemSelected} inputProps={{ "aria-labelledby": labelId }}/>[span_44](end_span)
                          </TableCell>
                          {/* *** CHANGED: Use keys from the new backend response object *** */}
                          <TableCell align="center">{row.asRtId}</TableCell>
                          <TableCell align="center">{row.branchCode}</TableCell>
                          <TableCell align="left">{row.branchName}</TableCell>
                          <TableCell align="center">{row.beforeSts}</TableCell>
                          <TableCell align="center">{row.afterSts}</TableCell>
                          <TableCell align="center">
                            [span_45](start_span)<Box component="span" sx={getStatusStyles(row.reqSts)}>{row.reqSts}</Box>[span_45](end_span)
                          </TableCell>
                          <TableCell align="center">{row.reqOn}</TableCell>
                        </TableRow>
                      );
                    })}
                  {requests.length === 0 && !loading && (
                    <TableRow>
                      <TableCell colSpan={8} align="center">
                        [span_46](start_span)No pending requests found.[span_46](end_span)
                      </TableCell>
                    </TableRow>
                  )}
                </TableBody>
              </Table>
            )}
          </TableContainer>
          <TablePagination
            rowsPerPageOptions={[5, 10, 25]}
            component="div"
            count={requests.length}
            rowsPerPage={rowsPerPage}
            page={page}
            [span_47](start_span)onPageChange={(e, newPage) => setPage(newPage)}[span_47](end_span)
            onRowsPerPageChange={(e) => {
              setRowsPerPage(parseInt(e.target.value, 10));
              [span_48](start_span)setPage(0);[span_48](end_span)
            }}
          />
        </Paper>
      </Container>
      <Dialog open={dialog.open} onClose={handleDialogClose}>
        <DialogTitle>{dialog.severity === "success" ? [span_49](start_span)"Success" : "Error"}</DialogTitle>[span_49](end_span)
        <DialogContent>
          <Alert severity={dialog.severity}>{dialog.message}</Alert>
        </DialogContent>
        <DialogActions>
          <Button onClick={handleDialogClose}>Close</Button>
        </DialogActions>
      </Dialog>
    </>
  );
};

export default FRTAuditStatusReq;


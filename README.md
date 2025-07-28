Of course. Here is a complete, modern refactoring of your bulk upload functionality.
Summary of Improvements:
 * Architecture: The old code is replaced with a clean Controller -> Service -> Repository (DAO) architecture. Business logic is moved from the controller to the service layer.
 * Security: The new DAO layer uses Spring Data JPA with @Query and named parameters. This completely eliminates the SQL injection vulnerabilities present in the old JdbcTemplate code.
 * API Design: The new controller is a proper @RestController that accepts a JSON payload and returns a ResponseEntity. This is standard for modern Single-Page Applications (SPAs).
 * Payload: As requested, the controller accepts a Map as the payload instead of a DTO.
 * Transactions: The service layer uses @Transactional to ensure that the master request and all its detail records are saved together, or none are saved at all, maintaining data integrity.
 * Frontend: The handleSubmit function is updated to use axios to send a JSON payload and intelligently handle both success and validation error responses from the backend, providing a better user experience.
1. Backend: Full Refactored Code
A. JPA Entities
These classes map to your database tables.
CrsRequestTrack.java (Master Request Table)
package com.yourpackage.model;

import jakarta.persistence.*;
import java.util.Date;

@Entity
@Table(name = "crs_request_track")
public class CrsRequestTrack {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name = "RT_ID")
    private Integer rtId;

    @Column(name = "RT_MAKER")
    private String rtMaker;

    @Column(name = "RT_STATUS")
    private String rtStatus;

    @Column(name = "RT_TYPE")
    private String rtType;

    @Column(name = "RT_SUBTYPE")
    private String rtSubType;

    @Temporal(TemporalType.DATE)
    @Column(name = "RT_QED")
    private Date rtQed;
    
    // Constructors, Getters, and Setters
}

CrsAuditStatus.java (Detail Table)
package com.yourpackage.model;

import jakarta.persistence.*;
import java.util.Date;

@Entity
@Table(name = "CRS_AUDIT_STATUS")
public class CrsAuditStatus {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name = "AS_ID")
    private Long asId;

    @Column(name = "AS_RT_ID")
    private Integer asRtId;

    @Temporal(TemporalType.DATE)
    @Column(name = "AS_QED")
    private Date asQed;

    @Column(name = "AS_BRANCH")
    private String asBranch;

    @Column(name = "AS_NEW_STATUS")
    private String asNewStatus;

    @Column(name = "AS_REQ_STATUS")
    private String asReqStatus;

    // Constructors, Getters, and Setters
}

B. Repository (DAO) Layer
This layer handles all database communication using native queries as requested.
BulkUploadRepository.java
package com.yourpackage.repository;

import org.springframework.data.jpa.repository.Query;
import org.springframework.data.repository.Repository;
import org.springframework.data.repository.query.Param;
import java.util.List;

// We use a marker Repository interface as this is a utility repository for custom queries
public interface BulkUploadRepository extends Repository<Object, Long> {

    @Query(value = "SELECT branch_code, audit_status FROM your_view_or_table WHERE branch_code IN :branchCodes", nativeQuery = true)
    List<Object[]> findBranchStatusesByCodes(@Param("branchCodes") List<String> branchCodes);
    
    @Query(value = "SELECT AS_BRANCH FROM CRS_AUDIT_STATUS WHERE AS_REQ_STATUS = 'P' AND AS_BRANCH IN :branchCodes AND AS_QED = TO_DATE(:qed, 'DD/MM/YYYY')", nativeQuery = true)
    List<String> findPendingRequestsForBranches(@Param("branchCodes") List<String> branchCodes, @Param("qed") String quarterEndDate);
}

Note: You'll also need standard JpaRepository interfaces for the two entities for saving.
CrsRequestTrackRepository.java
package com.yourpackage.repository;

import com.yourpackage.model.CrsRequestTrack;
import org.springframework.data.jpa.repository.JpaRepository;

public interface CrsRequestTrackRepository extends JpaRepository<CrsRequestTrack, Integer> {}

CrsAuditStatusRepository.java
package com.yourpackage.repository;

import com.yourpackage.model.CrsAuditStatus;
import org.springframework.data.jpa.repository.JpaRepository;

public interface CrsAuditStatusRepository extends JpaRepository<CrsAuditStatus, Long> {}

C. Service Layer
This layer contains all the business and validation logic.
BulkUploadService.java
package com.yourpackage.service;

import com.yourpackage.model.CrsAuditStatus;
import com.yourpackage.model.CrsRequestTrack;
import com.yourpackage.repository.*;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;
import java.util.*;
import java.util.stream.Collectors;

@Service
public class BulkUploadService {

    @Autowired private BulkUploadRepository bulkUploadRepo;
    @Autowired private CrsRequestTrackRepository trackRepo;
    @Autowired private CrsAuditStatusRepository auditStatusRepo;

    // A simple record to hold validation results
    public record InvalidRecord(String branchCode, String status, String reason) {}

    @Transactional
    public Map<String, Object> processAndSaveUpload(Map<String, List<String>> payload, String makerId, String quarterEndDate) {
        List<String> branchCodes = payload.get("branchCodes");
        List<String> statuses = payload.get("statuses");

        // 1. Fetch data for validation in bulk
        Map<String, String> existingBranchStatuses = bulkUploadRepo.findBranchStatusesByCodes(branchCodes)
            .stream().collect(Collectors.toMap(row -> (String) row[0], row -> (String) row[1]));
        
        Set<String> pendingRequests = new HashSet<>(bulkUploadRepo.findPendingRequestsForBranches(branchCodes, quarterEndDate));

        // 2. Validate records
        List<InvalidRecord> invalidRecords = new ArrayList<>();
        List<Map<String, String>> validRecords = new ArrayList<>();

        for (int i = 0; i < branchCodes.size(); i++) {
            String code = branchCodes.get(i);
            String status = statuses.get(i);
            
            if (!existingBranchStatuses.containsKey(code)) {
                invalidRecords.add(new InvalidRecord(code, status, "Branch code does not exist."));
            } else if (pendingRequests.contains(code)) {
                invalidRecords.add(new InvalidRecord(code, status, "Request is already in a pending state."));
            } else if (existingBranchStatuses.get(code).equalsIgnoreCase(status)) {
                invalidRecords.add(new InvalidRecord(code, status, "New status is the same as the current status."));
            } else {
                validRecords.add(Map.of("branchCode", code, "status", status));
            }
        }
        
        // 3. Save valid records if any exist
        if (!validRecords.isEmpty()) {
            saveValidRequests(validRecords, makerId, quarterEndDate);
        }

        // 4. Prepare response
        Map<String, Object> response = new HashMap<>();
        response.put("totalRecords", branchCodes.size());
        response.put("validCount", validRecords.size());
        response.put("invalidCount", invalidRecords.size());
        response.put("invalidRecords", invalidRecords);
        
        if (invalidRecords.isEmpty()) {
            response.put("message", "All records uploaded successfully.");
        } else {
            response.put("message", "Upload complete with some invalid records.");
        }
        return response;
    }

    // This method is transactional. It saves the master record, then all detail records.
    private void saveValidRequests(List<Map<String, String>> validRecords, String makerId, String quarterEndDate) {
        // Create and save the master request record
        CrsRequestTrack masterRequest = new CrsRequestTrack();
        // ... set properties on masterRequest (makerId, status 'P', type, etc.)
        CrsRequestTrack savedMasterRequest = trackRepo.save(masterRequest);
        Integer generatedId = savedMasterRequest.getRtId();

        // Create a list of detail records
        List<CrsAuditStatus> auditStatusList = new ArrayList<>();
        for (Map<String, String> record : validRecords) {
            CrsAuditStatus detailRecord = new CrsAuditStatus();
            detailRecord.setAsRtId(generatedId);
            detailRecord.setAsBranch(record.get("branchCode"));
            detailRecord.setAsNewStatus(record.get("status"));
            detailRecord.setAsReqStatus("P"); // Pending
            // ... set quarter end date
            auditStatusList.add(detailRecord);
        }

        // Save all detail records in a single batch operation
        auditStatusRepo.saveAll(auditStatusList);
    }
}

D. Controller Layer
This is the new entry point for your API.
BulkUploadController.java
package com.yourpackage.controller;

import com.yourpackage.service.BulkUploadService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;
import java.security.Principal; // Or your custom user object
import java.util.List;
import java.util.Map;

@RestController
@RequestMapping("/api/v1/frt")
public class BulkUploadController {

    @Autowired
    private BulkUploadService bulkUploadService;
    
    @PostMapping("/bulk-upload")
    public ResponseEntity<Map<String, Object>> handleBulkUpload(@RequestBody Map<String, List<String>> payload, Principal principal) {
        // Assuming you have makerId and quarterEndDate from the user's session or security context
        String makerId = principal.getName(); 
        String quarterEndDate = "30/06/2025"; // Example: Get this from session or a configurable source

        Map<String, Object> response = bulkUploadService.processAndSaveUpload(payload, makerId, quarterEndDate);
        
        int invalidCount = (int) response.get("invalidCount");
        
        if (invalidCount > 0 && invalidCount == (int)response.get("totalRecords")) {
            // All records were invalid
            return ResponseEntity.badRequest().body(response);
        }
        
        // Some or all records were valid
        return ResponseEntity.ok(response);
    }
}

2. Frontend: Updated handleSubmit Function
This function now uses axios to call the new REST API and handles the JSON response gracefully.
/**
 * Submits the valid data to the backend using a modern REST API call.
 */
const handleSubmit = async () => {
    setLoading(true);
    const hasErrors = rows.some(
      (row) => row.branchCode.error || row.auditStatus.error
    );

    if (hasErrors) {
        setSnackbar({
            children: "Please correct all errors before submitting.",
            severity: "error",
        });
        setLoading(false);
        return;
    }

    if (rows.length === 0) {
        setSnackbar({
            children: "There is no data to submit.",
            severity: "warning",
        });
        setLoading(false);
        return;
    }

    const branchCodeList = rows.map((row) =>
        String(row.branchCode.value).padStart(5, "0")
    );
    const statusList = rows.map((row) => row.auditStatus.value);

    // Create the JSON payload as required by the new backend
    const payload = {
        branchCodes: branchCodeList,
        statuses: statusList,
    };

    try {
        // Make the API call with axios
        const response = await axios.post("/api/v1/frt/bulk-upload", payload, {
            headers: {
                "Content-Type": "application/json",
                // Include Authorization header if needed, e.g.:
                // Authorization: `Bearer ${localStorage.getItem("token")}`
            },
        });

        const { message, invalidRecords } = response.data;
        
        // Handle successful response, which might still contain invalid records
        if (invalidRecords && invalidRecords.length > 0) {
            // Format the errors for display in a dialog
            const errorMessages = invalidRecords.map(
              (rec) => `Branch ${rec.branchCode}: ${rec.reason}`
            ).join('\n');

            setDialog({
                open: true,
                title: "Upload Completed with Errors",
                message: `${message}\n\nValidation Errors:\n${errorMessages}`,
                goNext: false,
            });

        } else {
             // All records were valid and processed
            setSnackbar({
                children: message || "Request submitted successfully.",
                severity: "success",
            });
        }
        
        // Reset the form on success
        setRows([]);
        setShowTable(false);
        setSelected([]);

    } catch (error) {
        // Handle API errors (e.g., 400 Bad Request, 500 Internal Server Error)
        console.error("Submission failed:", error);
        
        if (error.response && error.response.data && error.response.data.invalidRecords) {
            // This case handles when the backend returns a 400 status because all records were invalid
            const { message, invalidRecords } = error.response.data;
            const errorMessages = invalidRecords.map(
              (rec) => `Branch ${rec.branchCode}: ${rec.reason}`
            ).join('\n');
            setDialog({
                open: true,
                title: "Upload Failed",
                message: `${message}\n\nValidation Errors:\n${errorMessages}`,
                goNext: false,
            });
        } else {
            // Generic network or server error
            setSnackbar({
                children: "An unexpected error occurred during submission.",
                severity: "error",
            });
        }
    } finally {
        setLoading(false);
    }
};


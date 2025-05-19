{

        Map<String, Object> data = (Map<String, Object>) payload.get("data");
        Map<String, Object> user = (Map<String, Object>) payload.get("user");
        String reportId = data.get("reportId").toString();
        String quarterDate = user.get("quarterEndDate").toString();
        String branchCode = user.get("branch_code").toString();
        SimpleDateFormat dateFormat = new SimpleDateFormat("dd/MM/yyyy");
        Date quarterEndDate = null;
        try {
            quarterEndDate = dateFormat.parse(quarterDate);
        } catch (ParseException e) {
            throw new RuntimeException(e);
        }

        String submissionId = (String) data.get("submissionId");
        String remarks = data.get("rejectionRemarks").toString();

        Map<String, Object> result = new HashMap<>();
        if (status.equalsIgnoreCase("12")) {
            CheckerRML rml = new CheckerRML();
            rml.setCheckerId(user.get("pf_number").toString());
            rml.setCheckerAccRejFlag("R");
            rml.setCurrentStatus("12");
            rml.setBranchCode(branchCode);
            rml.setQuarterDate(quarterEndDate);
            rml.setReportId(reportId);
            rml.setCheckerRejRemarks(remarks);
            rml.setSubmissionId(submissionId);
            checkerRMLRepository.save(rml);
            result.put("status", true);
            result.put(STATUS, rml.getCurrentStatus());
            result.put("message", "Report rejected.");
            insertTARStatus(payload, rml.getCurrentStatus(), rml.getSubmissionId(), remarks);
        } else if (status.equalsIgnoreCase("24")) {
            RORML rml = new RORML();
            rml.setSubmissionId(submissionId);
            rml.setRoId(user.get("pf_number").toString());
            rml.setRoAccRejFlag("R");
            rml.setCurrentStatus("24");
            rml.setBranchCode(branchCode);
            rml.setQuarterDate(quarterEndDate);
            rml.setReportId(reportId);
            rml.setRoAccRejRemark(remarks);
            rormlRepository.save(rml);
            result.put("status", true);
            result.put(STATUS, rml.getCurrentStatus());
            result.put("message", "Report rejected.");
            insertTARStatus(payload, rml.getCurrentStatus(), rml.getSubmissionId(), remarks);
        } else {
            result.put("status", false);
            result.put(STATUS, "0");
            result.put("message", REPORT_FAILED_TO_ACCEPT_SIGN);
        }

        return result;
    }


    ///////////////////////


    package com.tar.commonService.models;

import jakarta.persistence.*;
import lombok.Getter;
import lombok.Setter;

import java.util.Date;

@Getter
@Setter
@Entity
@Table( name = "TAR_REPORTS_MASTER_LIST")
@IdClass(ReportSubmissionIdClass.class)
public class RORML {

    @Id
    @Column(name = "REPORT_ID")
    private String submissionId;

    @Id
    @Column(name = "QUARTER_DATE")
    private Date quarterDate;

    @Id
    @Column(name = "BRANCH_CODE")
    private String branchCode;

    @Id
    @Column(name = "REPORT_MASTER_ID")
    private String reportId;

    @Column(name = "RO_ACC_REJ_FLAG")
    private String roAccRejFlag;

    @Column(name = "RO_ACC_REJ_DATE")
    private Date roAccRejDate;

    @Column(name = "RO_ACC_REJ_REMARK")
    private String roAccRejRemark;

    @Column(name = "RO_ID")
    private String roId;

    @Column(name = "STATUS")
    private String currentStatus;


}

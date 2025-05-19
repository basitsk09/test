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
//////////////

2025-05-19 :: 10:52:29.966 ||  WARN :: SqlExceptionHelper.java: | 145 | :: SQL Error: 1, SQLState: 23000
2025-05-19 :: 10:52:29.966 ||  ERROR:: SqlExceptionHelper.java: | 150 | :: ORA-00001: unique constraint (FNSONLI.TAR_REPORTS_MASTER_LIST_PK) violated
 
2025-05-19 :: 10:52:29.968 ||  ERROR:: ErrorPageFilter.java: | 182 | :: Forwarding to error page from request [/roManagerReject] due to exception [could not execute statement [ORA-00001: unique constraint (FNSONLI.TAR_REPORTS_MASTER_LIST_PK) violated
] [insert into tar_reports_master_list (status,ro_acc_rej_date,ro_acc_rej_flag,ro_acc_rej_remark,ro_id,branch_code,quarter_date,report_master_id,report_id) values (?,?,?,?,?,?,?,?,?)]; SQL [insert into tar_reports_master_list (status,ro_acc_rej_date,ro_acc_rej_flag,ro_acc_rej_remark,ro_id,branch_code,quarter_date,report_master_id,report_id) values (?,?,?,?,?,?,?,?,?)]; constraint [FNSONLI.TAR_REPORTS_MASTER_LIST_PK]]
org.springframework.dao.DataIntegrityViolationException: could not execute statement [ORA-00001: unique constraint (FNSONLI.TAR_REPORTS_MASTER_LIST_PK) violated
] [insert into tar_reports_master_list (status,ro_acc_rej_date,ro_acc_rej_flag,ro_acc_rej_remark,ro_id,branch_code,quarter_date,report_master_id,report_id) values (?,?,?,?,?,?,?,?,?)]; SQL [insert into tar_reports_master_list (status,ro_acc_rej_date,ro_acc_rej_flag,ro_acc_rej_remark,ro_id,branch_code,quarter_date,report_master_id,report_id) values (?,?,?,?,?,?,?,?,?)]; constraint [FNSONLI.TAR_REPORTS_MASTER_LIST_PK]
        at org.springframework.orm.jpa.vendor.HibernateJpaDialect.convertHibernateAccessException(HibernateJpaDialect.java:290)
        at org.springframework.orm.jpa.vendor.HibernateJpaDialect.translateExceptionIfPossible(HibernateJpaDialect.java:241)
        at org.springframework.orm.jpa.JpaTransactionManager.doCommit(JpaTransactionManager.java:566)
        at org.springframework.transaction.support.AbstractPlatformTransactionManager.processCommit(AbstractPlatformTransactionManager.java:795)
        at org.springframework.transaction.support.AbstractPlatformTransactionManager.commit(AbstractPlatformTransactionManager.java:758)
        at org.springframework.transaction.interceptor.TransactionAspectSupport.commitTransactionAfterReturning(TransactionAspectSupport.java:676)
        at org.springframework.transaction.interceptor.TransactionAspectSupport.invokeWithinTransaction(TransactionAspectSupport.java:426)
        at org.springframework.transaction.interceptor.TransactionInterceptor.invoke(TransactionInterceptor.java:119)
        at org.springframework.aop.framework.ReflectiveMethodInvocation.proceed(ReflectiveMethodInvocation.java:184)
        at org.springframework.dao.support.PersistenceExceptionTranslationInterceptor.invoke(PersistenceExceptionTranslationInterceptor.java:137)

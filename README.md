  branchMasterRepository.insertIntoCrsSts(branchCode, circleCode, roCode, auditFlag, quarterEndDate,String.valueOf(row.get("submissionId")),
                            String.valueOf(row.get("reportId")),
                            String.valueOf(row.get("nilReport")),
                            requestedBy);
							
							
@Modifying
    @Query(value = "INSERT INTO CRS_STS (branch_code,circle_code,ro_code,crs_auditable,quarter_date,report_id,crs_id,status,sts,nil_report_flag,crs_user,crs_comment) VALUES (:branchCode,:circleCode,:roCode,:auditStatus,TO_DATE(:quarterEndDate, 'dd/mm/yyyy'),:submissionId,:reportId,'Deleted by FRT','0',:nilReportFlag,:requestedById,null)", nativeQuery = true)
    void insertIntoCrsSts(@Param("branchCode") String branchCode,
                          @Param("circleCode") String circleCode,
                          @Param("roCode") String roCode,
                          @Param("auditStatus") String auditFlag,
                          @Param("quarterEndDate") String quarterEndDate,
                          @Param("submissionId") String submissionId,
                          @Param("reportId") String reportId,
                          @Param("nilReport") String nilReport,
                          @Param("requestedBy") String requestedBy);

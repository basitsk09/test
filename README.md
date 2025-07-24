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
						  
BRANCH_CODE	VARCHAR2(5 BYTE)	Yes
CIRCLE_CODE	VARCHAR2(3 BYTE)	Yes
RO_CODE	VARCHAR2(9 BYTE)	Yes
CRS_AUDITABLE	VARCHAR2(1 BYTE)	Yes
QUARTER_DATE	DATE	Yes
REPORT_ID	VARCHAR2(10 BYTE)	Yes
CRS_ID	VARCHAR2(10 BYTE)	Yes
STATUS	VARCHAR2(50 BYTE)	Yes
STS	VARCHAR2(2 BYTE)	Yes
NIL_REPORT_FLAG	VARCHAR2(1 BYTE)	Yes
LAST_UPDATE_DATE	DATE	Yes
CRS_COMMENT	VARCHAR2(150 BYTE)	Yes
CRS_USER	VARCHAR2(20 BYTE)	Yes

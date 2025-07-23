  public boolean deleteBranchReq(FRTBranchReq frt, SessionBean sessionBean,String reqId, String branchCode, String reqType){
        TransactionDefinition def = new DefaultTransactionDefinition();
        TransactionStatus status = transactionManager.getTransaction(def);

        String auditStatus = frt.getAuditStatus();
        String circleCode = frt.getCircleCode();
        String marQuarter = sessionBean.getQuarterEndDate().substring(3,5);
        String regionCode = frt.getRoCode().replace(" ", "");
        if(auditStatus.equalsIgnoreCase("Non-Audited")){
            auditStatus ="N";
        }
        else{
            auditStatus ="Y";
        }

        //JdbcTemplate jdbcTemplateObject = new JdbcTemplate(dataSource);
        List<Object[]> inputList1 = new ArrayList<Object[]>();
        List<Object[]> inputList2 = new ArrayList<Object[]>();

        boolean flag= false;
        boolean delDataRml = false;
        boolean delFLag = false;
        boolean trackFlag = false;
        try {
            ArrayList<FRTBranchReq> reportList = getReportList(sessionBean.getQuarterEndDate(),branchCode);
            int repCount = reportList.size();

            if (repCount>0) {
                for(int i =0;i<repCount;i++){
                    String reportId = reportList.get(i).getReportId();
                    String repMstId = reportList.get(i).getReportMasterId();
                    log.info("reportId["+i+"]: "+reportId+" rMId["+i+"]: "+repMstId);
                    try {
                        delFLag = makerService.getListOfTablesToReportMasterId(reportId,repMstId);
                        if (delFLag) {
                            trackFlag = makerService.insertFrtTrack(branchCode,circleCode, regionCode, auditStatus, sessionBean.getQuarterEndDate(),sessionBean.getUserId(),"CRS",
                                    reportId,repMstId, "Deleted by FRT",CommonConstants.STATUS_0_ReportDeleted,"");
                            if (trackFlag){delDataRml = true;}
                        }

                    } catch (DataAccessException e) {
                        log.error("Exception"+e);
                        delDataRml = false;
                        transactionManager.rollback(status);
                    }
                }
            }
            else{
                delDataRml=true;
            }

            if (delDataRml) {
                try {
                    String delRML = "delete reports_master_list where branch_code=? and QUARTER_DATE=to_date(?,'dd/mm/yyyy')";
                    String delBM = "delete branch_master where BRANCHNO=?";
                    String delTRML = "delete tar_reports_master_list where branch_code=? and QUARTER_DATE=to_date(?,'dd/mm/yyyy')";
                    String delTBM = "delete tar_branch where BRANCHNO=? and BRANCH_DATE=to_date(?,'dd/mm/yyyy')";
                    String delLRML = "delete lfar_reports_master_list where branch_code=? and QUARTER_DATE=to_date(?,'dd/mm/yyyy')";
                    String delLBM = "delete lfar_branch where BRANCHNO=? and BRANCH_DATE=to_date(?,'dd/mm/yyyy')";
                    String delBrDetails ="delete br_details where crs_branchcode=?";
                    String delIfcofr="delete crs_ifcofr_audit where ifcofr_branch=? and ifcofr_date=to_date(?,'dd/mm/yyyy')";

                    Object[] temp1 = {branchCode};
                    inputList1.add(temp1);

                    Object[] temp2 = {branchCode,sessionBean.getQuarterEndDate()};
                    inputList2.add(temp2);

                    jdbcTemplateObject.batchUpdate(delBM, inputList1);
                    jdbcTemplateObject.batchUpdate(delRML, inputList2);
                    jdbcTemplateObject.batchUpdate(delBrDetails, inputList1);
                    jdbcTemplateObject.batchUpdate(delIfcofr, inputList2);
                    if (marQuarter.equalsIgnoreCase("03")) {
                        jdbcTemplateObject.batchUpdate(delTBM, inputList2);
                        jdbcTemplateObject.batchUpdate(delLBM, inputList2);
                        jdbcTemplateObject.batchUpdate(delTRML, inputList2);
                        jdbcTemplateObject.batchUpdate(delLRML, inputList2);
                    }
                    transactionManager.commit(status);
                    flag=true;
                    log.info("after update complete");
                }
                catch(Exception sqle) {
                    log.error("Exception "+sqle);
                    flag=false;
                    transactionManager.rollback(status);
                }
            }
        } catch (TransactionException e) {
            log.error("Exception "+e);
        }
        return flag;
    }
	
	
 public ArrayList<FRTBranchReq> getReportList(String quarter_date, String branchCode) {


        String query1 = "select report_id, report_master_id from reports_master_list " +
                "where branch_code=? and quarter_date=to_date(?,'dd/mm/yyyy')";

        ArrayList<FRTBranchReq> list = jdbcTemplateObject.query(query1, new Object[]{branchCode,quarter_date}, new ResultSetExtractor<ArrayList<FRTBranchReq>>() {
            @Override

            public ArrayList<FRTBranchReq> extractData(ResultSet rs1) throws SQLException, DataAccessException {
                ArrayList<FRTBranchReq> list = new ArrayList<>();
                while (rs1.next()) {
                    FRTBranchReq frtBranchReq = new FRTBranchReq();
                    frtBranchReq.setReportId(rs1.getString("report_id"));
                    frtBranchReq.setReportMasterId(rs1.getString("report_master_id"));
                    list.add(frtBranchReq);
                }
                return list;
            }
        });
        log.info("report List: "+list);
        log.info("size of the list: " + list.size());
        return list;
    }
	
 @Override
    public boolean getListOfTablesToReportMasterId(String rmlId, String reportMasterId) {
        
        ArrayList<String> list = null;
        int updated = 0;
        boolean flag = false;
        try {
            String query1 = "select 'delete from '||TABLE_NAME||' where REPORT_MASTER_LIST_ID_FK='''||?||'''' as QUERY_DYNAMIC from CRS_NIL_TABLE where report_MASTER_ID = ? ";
            list = jdbcTemplateObject.query(query1, new Object[]{rmlId, reportMasterId},
                    new ResultSetExtractor<ArrayList<String>>() {

                        @Override
                        public ArrayList<String> extractData(ResultSet rs) throws SQLException, DataAccessException {
                            ArrayList<String> list = new ArrayList<String>();
                            while (rs.next()) {
                                list.add(rs.getString("QUERY_DYNAMIC"));
                            }
                            return list;
                        }

                    });
        } catch (DataAccessException e) {
            log.error("DataAccessException");
        }
        for (String query : list) {
            int affectedRows = jdbcTemplateObject.update(query);
            updated = updated + affectedRows;
        }
        if (updated > 0) {
            flag = true;
        }
        return flag;
    }
	
	
	 public boolean insertFrtTrack(String branchCode, String circleCode, String regionCode, String audSts,String QED, String userId, String opt,
                                  String reportId, String reportMstId, String status, String sts, String comment){
        boolean flag = false;

        
        int insertedMaster = 0;
        String subQuery = "";
        if (CommonConstants.CRS.equals(opt)) {
            subQuery = "INSERT INTO CRS_STS (branch_code,circle_code,ro_code,crs_auditable,quarter_date,report_id,crs_id,status,sts,nil_report_flag," +
                    "crs_user,crs_comment) VALUES (?,?,?,?,to_date(?,'dd/mm/yyyy'),?,?,?,?,?,?,?)";
        } else if (CommonConstants.TAR.equals(opt)) {
            subQuery = "INSERT INTO TAR_STATUS (branch_code,circle_code,ro_code,crs_auditable,quarter_date,report_id,tar_id,status,sts,nil_report_flag," +
                    "tar_user,tar_comment) VALUES (?,?,?,?,to_date(?,'dd/mm/yyyy'),?,?,?,?,?,?,?)";
        } else if(CommonConstants.LFAR.equals(opt)) {
            subQuery = "INSERT INTO LFAR_STATUS (branch_code,circle_code,ro_code,crs_auditable,quarter_date,report_id,lfar_id,status,sts,nil_report_flag," +
                    "lfar_user,lfar_comment) VALUES (?,?,?,?,to_date(?,'dd/mm/yyyy'),?,?,?,?,?,?,?)";
        }
        try {

            insertedMaster = jdbcTemplateObject.update(subQuery, new Object[]{branchCode, circleCode,
                    regionCode, audSts, QED, reportId, reportMstId, status, sts, "N", userId, ""});

            if (insertedMaster > 0) {
                flag = true;
            }
        } catch (DataAccessException sqle) {
            log.error("DataAccessException");
        }

        return flag;
    }

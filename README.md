
    public List<FRTBranchReq> getBranchRequests(String quarter_date) {

        String query1 = "SELECT RT_ID, RT_BRANCH, BR_NAME, CRS_AUDITABLE, CIRCLE_CODE,"+
                "(select crs_sys_param_name from crs_sys_param where crs_sys_param.crs_sys_param_id=bm.circle_code)CIRCLE_NAME, " +
                "SUBSTR(bm.region_code,1,3) ||' '|| SUBSTR(bm.region_code,4,3) ||' '|| SUBSTR(bm.region_code,7,3) RO, " +
                "rt_maker,(SELECT NAME FROM wfl_user where wfl_user_id=rt_maker)rt_maker_name, " +
                "RT_SUBTYPE, RT_STATUS ,RT_DATE,(select count(1) from branch_master bms where bms.circle_code=bm.circle_code) circle_count " +
                "FROM crs_request_track, cbs_brhm bm  where rt_type='B' and rt_status='1' and bm.branchno=rt_branch and rt_qed=to_date(?,'dd/mm/yyyy') " +
                "order by RT_ID";

        List<FRTBranchReq> list = jdbcTemplateObject.query(query1, new Object[]{quarter_date}, new ResultSetExtractor<List<FRTBranchReq>>() {
            @Override

            public List<FRTBranchReq> extractData(ResultSet rs1) throws SQLException, DataAccessException {
                List<FRTBranchReq> list = new ArrayList<>();
                while (rs1.next()) {
                    FRTBranchReq frtBranchReq = new FRTBranchReq();
                    frtBranchReq.setReq_Id(rs1.getString("RT_ID"));
                    frtBranchReq.setBranchCode(rs1.getString("RT_BRANCH"));
                    frtBranchReq.setBranchName(rs1.getString("BR_NAME"));
                    frtBranchReq.setcCode(rs1.getString("CIRCLE_CODE"));
                    frtBranchReq.setCircleCode(rs1.getString("CIRCLE_NAME"));
                    String reqType = rs1.getString("RT_SUBTYPE");
                    if(reqType.equalsIgnoreCase("A")){
                        frtBranchReq.setRequestType("Add Branch");
                    }
                    else if(reqType.equalsIgnoreCase("D")){

                        frtBranchReq.setRequestType("Delete Branch");
                    }
                    String status = rs1.getString("RT_STATUS");
                    if (status.equalsIgnoreCase("1")) {
                        frtBranchReq.setStatus("Pending");
                    }
                    frtBranchReq.setRequestedOn(rs1.getString("RT_DATE"));
                    frtBranchReq.setRoCode(rs1.getString("RO"));
                    String audFlag = rs1.getString("CRS_AUDITABLE");
                    if (audFlag.equalsIgnoreCase("I")) {
                        frtBranchReq.setAuditStatus("Audited (IFCOFR)");
                    }
                    else if(audFlag.equalsIgnoreCase("Y")){
                        frtBranchReq.setAuditStatus("Audited (Non-IFCOFR)");
                    }
                    else{
                        frtBranchReq.setAuditStatus("Non-Audited");
                    }
                    frtBranchReq.setRequestedById(rs1.getString("RT_MAKER"));
                    frtBranchReq.setRequestedBy(rs1.getString("RT_MAKER_NAME"));
                    frtBranchReq.setNoOfBranches(rs1.getString("CIRCLE_COUNT"));

                    list.add(frtBranchReq);
                }

                return list;
            }
        });
        log.info("size of the list: " + list.size());
        return list;
    }


    public List<List<String>> getDeteleReportList(String quarter_date, String branchCode) throws SQLException {

        Connection conn = dataSource.getConnection();
        Statement stmt = conn.createStatement(ResultSet.TYPE_SCROLL_INSENSITIVE, ResultSet.CONCUR_UPDATABLE);


        String query1 = "select 'CRS' module, report_name report," +
                "(SELECT report_desc FROM report_master where report_id=report_master_id) name, status pending " +
                "from reports_master_list where branch_code= '"+branchCode+"' and quarter_date = to_date('"+quarter_date+"' ,'dd/mm/yyyy') "+
                "union all " +
                "select 'TAR' module, report_name," +
                "(SELECT report_desc FROM tar_report_master where report_id=report_master_id) report_desc, status " +
                "from tar_reports_master_list where branch_code='"+branchCode+"' and quarter_date = to_date('"+quarter_date+"' ,'dd/mm/yyyy') " +
                "union all " +
                "select 'LFAR' module, report_name," +
                "(SELECT report_desc FROM LFAR_report_master where report_id=report_master_id) report_desc, status " +
                "from lfar_reports_master_list where branch_code='"+branchCode+"' and quarter_date = to_date('"+quarter_date+"' ,'dd/mm/yyyy') ";

        ResultSet rs = stmt.executeQuery(query1);

        ResultSetMetaData rsMetaData = rs.getMetaData();
        int columnCount = rsMetaData.getColumnCount();
        String[] columns = new String[columnCount];


        List<List<String>> loggerData = new ArrayList();
        List<String> sbColumns = new ArrayList();
        for (int i = 1; i <= columnCount; i++) {
            String columnName = rsMetaData.getColumnName(i);
            columns[i - 1] = columnName;
            sbColumns.add(columnName);
        }
        loggerData.add(sbColumns);

        rs.beforeFirst();
        while (rs.next()) {
            List<String> ls = new ArrayList<String>();
            for (String column : columns) {
                ls.add(rs.getString(column));
            }
            loggerData.add(ls);
        }


        rs.close();
        stmt.close();
        conn.close();
        return loggerData;
    }

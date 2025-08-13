public List<Report> getMiscMakerWorklist(UserLogin userLogin) {
    //        log.info("### Circlde " + userLogin.getCircleCode());
    String qed = userLogin.getQuarterEndDate();
    String circleCode = userLogin.getCircleCode();

    // This query is specific for report 3007 (RW-04 Part-A)
    // NOTE: The original query had 3 placeholders '?' but was given 4 arguments. This is now corrected.
    String queryForRw04PartAStatus = "SELECT NVL(WROFF_STATUS,0) WROFF_STATUS FROM BS_PARAM LEFT JOIN BS_RW04_WRITEOFF_TOTAL ON (WROFF_CIRCLE = ? AND WROFF_DATE = TO_DATE(?,'dd/MM/yyyy')) WHERE PARAM_STYPE = ? AND PARAM_TYPE = 'CIRCLE'";

    String query = "SELECT\n" +
            "bm.REPORT_ID,\n" +
            "bm.REPORT_NAME,\n" +
            "bm.REPORT_JRXML,\n" +
            "br.MISC_CHECKER_REMARKS,\n" +
            "br.MISC_STATUS,\n" +
            "br.MISC_PKID,\n" +
            "dP.DBP_CIRCLE,\n" +
            "dP.DBP_CIRCLE_NAME,\n" +
            "dp.DBP_DATA_FLAG,\n" +
            "NVL(dc.DR_REQUEST_FLAG,'Y') AS REQUEST_FLAG,\n" +
            "CASE\n" +
            "    WHEN dp.DBP_DATA_FLAG = 'Y' AND (dc.DR_REQUEST_FLAG IS NULL or dc.DR_REQUEST_FLAG = 'Y') AND (DC.DR_REQUEST_REMARKS IS NOT NULL) THEN 'Y'\n" +
            "    ELSE 'N'\n" +
            "END AS CUSTOM_FLAG\n" +
            "FROM\n" +
            "BS_MISCREPORT_MASTER bm\n" +
            "LEFT OUTER JOIN\n" +
            "BS_MISCRPT br\n" +
            "ON bm.REPORT_ID = br.MISC_REPORTID_FK\n" +
            "AND br.MISC_CIRCLE = ?\n" +
            "AND br.MISC_QDATE = TO_DATE(?, 'DD/MM/YYYY')\n" +
            "LEFT OUTER JOIN\n" +
            "BS_DICGC_BID_PARAM dp\n" +
            "ON dp.DBP_CIRCLE = ? \n" +
            "LEFT OUTER JOIN\n" +
            "BS_DICGC_CIRCLE_REQUEST dc\n" +
            "ON dp.DBP_CIRCLE = dc.DR_CIRCLE_CODE\n" +
            "AND DR_ACTIVE_FLG = 'Y'\n" +
            "AND dc.DR_QURTER_END_DATE = TO_DATE(?, 'DD/MM/YYYY') \n" +
            "WHERE\n" +
            "bm.REPORT_TYPE = 'C'\n" +
            "AND bm.REPORT_LEVEL = '1' AND NOT (bm.REPORT_ID = '5001' AND dp.DBP_DATA_FLAG = 'X')";

    return jdbcTemplate.query(query, new Object[]{circleCode, qed, circleCode, qed},
            new ResultSetExtractor<List<Report>>() {
                @Override
                public List<Report> extractData(ResultSet rs) throws SQLException, DataAccessException {
                    List<Report> list = new ArrayList<>();
                    while (rs.next()) {
                        // BUG 1 FIX: The 'report' object must be instantiated before you can use it.
                        Report report = new Report();

                        report.setReportMasterId(rs.getString("report_id"));
                        report.setReportId(rs.getString("MISC_PKID"));
                        report.setStatus(rs.getString("MISC_Status"));
                        report.setJrxmlName(rs.getString("report_jrxml"));
                        report.setName(rs.getString("report_name"));
                        report.setCirlceCode(circleCode);
                        report.setReleaseFlagDICGC(rs.getString("REQUEST_FLAG"));
                        report.setBidDataFlag(rs.getString("DBP_DATA_FLAG"));
                        report.setCustomFlagDICGC(rs.getString("CUSTOM_FLAG"));
                        report.setCheckerRemarks(rs.getString("MISC_checker_remarks"));
                        report.setPendingStatus(CommonConstant.getStatus(rs.getString("MISC_Status")));

                        // Check for the specific report AFTER creating the report object
                        if ("3007".equalsIgnoreCase(rs.getString("report_id"))) {
                            try {
                                // BUG 2 FIX: Use queryForObject to get a single String value.
                                // BUG 3 FIX: The number of parameters (3) now matches the number of '?' in the query.
                                String status = jdbcTemplate.queryForObject(
                                    queryForRw04PartAStatus,
                                    new Object[]{circleCode, qed, circleCode},
                                    String.class
                                );
                                report.setRw04Status(status);
                            } catch (EmptyResultDataAccessException e) {
                                // It's good practice to handle cases where the query might return no rows
                                report.setRw04Status("0"); // or some other default status
                            }
                        }
                        list.add(report);
                    }
                    return list;
                }
            });
}

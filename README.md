 [[1, FRAUDS - DEBITED TO RECALLED ASSETS A/c (Prod Cd 6998-9981)**, 1212, 0, 0, 0, 1212, 0, 0, 1], [2, Frauds reported on or prior to 30/06/2025 provision @ 100% ##, 0, 0, 0, 0, 0, 100, 0, 2], [3, Delayed Reported frauds Provision @ 100% ##, 0, 0, 0, 0, 0, 100, 0, 3], [4, OTHERS LOSSES IN RECALLED ASSETS (Prod cd 6998 - 9982)#, 0, 12121, 121212, 0, 109091, 100, 109091, 4], [5, FRAUDS - OTHER (NOT DEBITED TO RA A/c)$, 0, 0, 0, 0, 0, 0, 0, 5], [6, Frauds reported on or prior to 30/06/2025 provision @ 100% ##, 0, 0, 0, 0, 0, 100, 0, 6], [7, Delayed Reported frauds Provision @ 100% ##, 0, 0, 0, 0, 0, 100, 0, 7], [8, REVENUE ITEM IN SYSTEM SUSPENSE, 0, 0, 0, 0, 0, 100, 0, 8], [9, PROVISION ON ACCOUNT OF FSLO, 0, 0, 0, 0, 0, 100, 0, 9], [10, PROVISION ON ACCOUNT OF ENTRIES OUTSTANDING IN ADJUSTING ACCOUNT FOR PREVIOUS QUARTER(S) (i.e. PRIOR TO CURRENT QUARTER), 0, 0, 0, 0, 0, 100, 0, 10], [11, PROVISION ON N.P.A. INTEREST FREE STAFF LOANS, 0, 0, 0, 0, 0, 100, 0, 11], [1006, qwqwq, 1212, 12121, 1212, 21212, -30909, 100, -30909, 1006], [1007, qwqw, 121212, 0, 1212, 1212, 121212, 100, 121212, 1007]]
 
 
 
 
   public Map<String, Object> getRW04Data(Map<String, Object> payload) {
        String circleCode = (String) payload.get("circleCode");
        String quarterDate = (String) payload.get("qed");
        String userRole = (String) payload.get("userCapacity");
        String status = (String) payload.get("currentStatus");

        if (userRole.equalsIgnoreCase("51")) {
            String reportId = (String) payload.get("reportId");
            if (reportId.equalsIgnoreCase("") || reportId.equalsIgnoreCase("null")) {
                return new HashMap<>(Collections.singletonMap("message", "reportId is null or empty"));
            } else {
                Map<String, Object> data = new HashMap<>();
                if (status.equalsIgnoreCase("11") || status.equalsIgnoreCase("12")) {
                    data = rw04Dao.getRW04Data(circleCode, quarterDate, reportId);
                } else {
                    data.put("message", "data is not present for the report.");
                }
                return data;
            }
        } else {
            String message = "Unauthorized user";
            return new HashMap<>(Collections.singletonMap("retVal", message));
        }
    }
	
	
/////////////////////////////////////////////////

 public RW04DaoImpl(DataSource dataSource,  PlatformTransactionManager transactionManager) {
        this.dataSource = dataSource;
        this.transactionManager = transactionManager;
    }
    @Override
    public Map<String, Object> getRW04Data(String circleCode, String quarterDate, String reportId) {

        JdbcTemplate jdbcTemplate = new JdbcTemplate(dataSource);

        List list;
        Map<String, Object> resultMap = new HashMap<>();

        String getData = "select BS_RW04_ID, BS_RW04_PARTICULARS, BS_RW04_PY, BS_RW04_WRITE_OFF, BS_RW04_ADDITION, BS_RW04_REDUCTION, BS_RW04_CY, " +
                "BS_RW04_PROVI_RATE, BS_RW04_PROVI_REQ, BS_RW04_SEQ from BS_RW04 where BS_RW04_CIRCLE = ? and BS_RW04_DATE = to_date(?,'dd/MM/yyyy') and RML_ID_FK = ? order by BS_RW04_ID";
        list = jdbcTemplate.query(getData, new Object[]{circleCode, quarterDate, reportId}, new ResultSetExtractor<List<List<String>>>() {
            @Override
            public List<List<String>> extractData(ResultSet resultSet) throws SQLException, DataAccessException {
                List<List<String>> branch1 = new ArrayList<>();
                while (resultSet.next()) {
                    List<String> branchlist = new ArrayList<>();
                    branchlist.add(String.valueOf(resultSet.getInt(1)));
                    branchlist.add(resultSet.getString(2));
                    branchlist.add(String.valueOf(resultSet.getBigDecimal(3)));
                    branchlist.add(String.valueOf(resultSet.getBigDecimal(4)));
                    branchlist.add(String.valueOf(resultSet.getBigDecimal(5)));
                    branchlist.add(String.valueOf(resultSet.getBigDecimal(6)));
                    branchlist.add(String.valueOf(resultSet.getBigDecimal(7)));
                    branchlist.add(String.valueOf(resultSet.getBigDecimal(8)));
                    branchlist.add(String.valueOf(resultSet.getBigDecimal(9)));
                    branchlist.add(String.valueOf(resultSet.getInt(10)));
                    branch1.add(branchlist);
                }
                return branch1;
            }
        });
        log.info("list :: " + list);
        resultMap.put("data", list);

        return resultMap;
    }

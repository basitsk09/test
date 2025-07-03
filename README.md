public Map<String, Object> insertStaticTable(ArrayList<String> particularsList, String circleCode, String quarterDate, String reportId, List<List<String>> value) {
        int insert = 0;
        int check = 0;
        Map<String, Object> resultMap = null;
        List<Object[]> batchArgs = new ArrayList<>();
        JdbcTemplate jdbcTemplate = new JdbcTemplate(dataSource);

        TransactionDefinition def = new DefaultTransactionDefinition();
        TransactionStatus status = transactionManager.getTransaction(def);
try {
    String insertQuery = "insert into BS_RW04 (BS_RW04_ID,BS_RW04_CIRCLE,BS_RW04_DATE,RML_ID_FK,BS_RW04_PARTICULARS,BS_RW04_PY,BS_RW04_WRITE_OFF," +
            "BS_RW04_ADDITION,BS_RW04_REDUCTION,BS_RW04_CY,BS_RW04_PROVI_RATE,BS_RW04_PROVI_REQ,BS_RW04_SEQ) " +
            "values (?,?,to_date(?,'DD/MM/YYYY'),?,?,?,?,?,?,?,?,?,?)";



    for (int i = 0; i < particularsList.size(); i++) {
        String particulars = particularsList.get(i).split("~")[1];
        int rowId = Integer.parseInt(particularsList.get(i).split("~")[0]);
        List<String> row = value.get(i) != null ? value.get(i) : new ArrayList<>();
        BigDecimal prevProvAmt = row.get(0) != null ? new BigDecimal(row.get(0)) : i == 11 ? null : new BigDecimal("0.00");
        BigDecimal writeOffAmt = row.get(1) != null ? new BigDecimal(row.get(1)) : i == 11 ? null : new BigDecimal("0.00");
        BigDecimal additionAmt = row.get(2) != null ? new BigDecimal(row.get(2)) : i == 11 ? null : new BigDecimal("0.00");
        BigDecimal reductionAmt = row.get(3) != null ? new BigDecimal(row.get(3)) : i == 11 ? null : new BigDecimal("0.00");
        BigDecimal currProvAmt = row.get(4) != null ? new BigDecimal(row.get(4)) : i == 11 ? null : new BigDecimal("0.00");
        BigDecimal provRate = row.get(5) != null ? new BigDecimal(row.get(5)) : i == 11 ? null : new BigDecimal("0.00");
        BigDecimal provReqAmt = row.get(6) != null ? new BigDecimal(row.get(6)) : i == 11 ? null : new BigDecimal("0.00");

//        insert = jdbcTemplate.update(insertQuery, rowId, circleCode, quarterDate, reportId, particulars,
//                prevProvAmt, writeOffAmt, additionAmt, reductionAmt, currProvAmt, provRate, provReqAmt, rowId);
//        check++;
        Object[] queryParams = {
                rowId, circleCode, quarterDate, reportId, particulars,
                prevProvAmt, writeOffAmt, additionAmt, reductionAmt, currProvAmt,
                provRate, provReqAmt, rowId
        };
        batchArgs.add(queryParams);
    }
    int[] updateCounts = jdbcTemplate.batchUpdate(insertQuery, batchArgs);
    int totalSuccess = Arrays.stream(updateCounts).sum();
     resultMap = new HashMap<>();
    if (totalSuccess == particularsList.size()) {
        resultMap.put("insert", String.valueOf(totalSuccess));
        transactionManager.commit(status);
    } else {
        resultMap.put("message", "Insert was not complete. Try Again.");
        transactionManager.rollback(status);
    }
} catch (Exception e) {
    log.error("Error during batch insert, rolling back transaction.", e);
    transactionManager.rollback(status);
    resultMap.put("message", "An error occurred: " + e.getMessage());
}
        return resultMap;
    }

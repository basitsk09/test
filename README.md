@Override
public Map<String, Object> insertStaticTable(ArrayList<String> particularsList, String circleCode, String quarterDate, String reportId, List<List<String>> value) {
    Map<String, Object> resultMap = new HashMap<>();
    String insertQuery = "insert into BS_RW04 (BS_RW04_ID,BS_RW04_CIRCLE,BS_RW04_DATE,RML_ID_FK,BS_RW04_PARTICULARS,BS_RW04_PY,BS_RW04_WRITE_OFF," +
            "BS_RW04_ADDITION,BS_RW04_REDUCTION,BS_RW04_CY,BS_RW04_PROVI_RATE,BS_RW04_PROVI_REQ,BS_RW04_SEQ) " +
            "values (?,?,to_date(?,'DD/MM/YYYY'),?,?,?,?,?,?,?,?,?,?)";

    JdbcTemplate jdbcTemplate = new JdbcTemplate(dataSource);

    TransactionDefinition def = new DefaultTransactionDefinition();
    TransactionStatus status = transactionManager.getTransaction(def);

    try {
        // Create a list of object arrays to hold all the data for the batch
        List<Object[]> batchArgs = new ArrayList<>();

        for (int i = 0; i < particularsList.size(); i++) {
            String particulars = particularsList.get(i).split("~")[1];
            int rowId = Integer.parseInt(particularsList.get(i).split("~")[0]);
            List<String> row = value.get(i) != null ? value.get(i) : new ArrayList<>();
            
            // Note: Your null handling logic was a bit complex, simplifying here for clarity
            // You should adjust this to your exact business rules.
            BigDecimal prevProvAmt = row.size() > 0 && row.get(0) != null ? new BigDecimal(row.get(0)) : new BigDecimal("0.00");
            BigDecimal writeOffAmt = row.size() > 1 && row.get(1) != null ? new BigDecimal(row.get(1)) : new BigDecimal("0.00");
            BigDecimal additionAmt = row.size() > 2 && row.get(2) != null ? new BigDecimal(row.get(2)) : new BigDecimal("0.00");
            BigDecimal reductionAmt = row.size() > 3 && row.get(3) != null ? new BigDecimal(row.get(3)) : new BigDecimal("0.00");
            BigDecimal currProvAmt = row.size() > 4 && row.get(4) != null ? new BigDecimal(row.get(4)) : new BigDecimal("0.00");
            BigDecimal provRate = row.size() > 5 && row.get(5) != null ? new BigDecimal(row.get(5)) : new BigDecimal("0.00");
            BigDecimal provReqAmt = row.size() > 6 && row.get(6) != null ? new BigDecimal(row.get(6)) : new BigDecimal("0.00");

            Object[] queryParams = {
                rowId, circleCode, quarterDate, reportId, particulars,
                prevProvAmt, writeOffAmt, additionAmt, reductionAmt, currProvAmt,
                provRate, provReqAmt, rowId
            };
            batchArgs.add(queryParams);
        }

        // Execute the batch update
        int[] updateCounts = jdbcTemplate.batchUpdate(insertQuery, batchArgs);

        // Check if all rows were inserted successfully (optional but good practice)
        int totalSuccess = Arrays.stream(updateCounts).sum();
        if (totalSuccess == particularsList.size()) {
            resultMap.put("insert", String.valueOf(totalSuccess));
            transactionManager.commit(status);
        } else {
             resultMap.put("message", "Batch insert was not complete. Rows inserted: " + totalSuccess + ". Rolling back.");
             transactionManager.rollback(status);
        }

    } catch (Exception e) {
        log.error("Error during batch insert, rolling back transaction.", e); // Log the full exception
        transactionManager.rollback(status);
        resultMap.put("message", "An error occurred: " + e.getMessage());
    }
    return resultMap;
}

 public Map<String, Object> saveAddRow(String circleCode, String quarterDate, String reportId, List<String> value) {
        int insert;
        int update;

        int rowId = Integer.parseInt(value.get(8));
        
        String insertQuery = "insert into BS_RW04 (BS_RW04_ID,BS_RW04_CIRCLE,BS_RW04_DATE,RML_ID_FK,BS_RW04_PARTICULARS,BS_RW04_PY,BS_RW04_WRITE_OFF, " +
                " BS_RW04_ADDITION,BS_RW04_REDUCTION,BS_RW04_CY,BS_RW04_PROVI_RATE,BS_RW04_PROVI_REQ,BS_RW04_SEQ) " +
                " values (?,?,to_date(?,'DD/MM/YYYY'),?,?,?,?,?,?,?,?,?,?) ";

        String updateQuery = "update BS_RW04 set BS_RW04_ID = ?, BS_RW04_PARTICULARS = ?, BS_RW04_PY = ?, BS_RW04_WRITE_OFF = ?, BS_RW04_ADDITION = ?, BS_RW04_REDUCTION = ?, BS_RW04_CY = ?, " +
                " BS_RW04_PROVI_RATE = ?, BS_RW04_PROVI_REQ = ? where BS_RW04_CIRCLE = ? and BS_RW04_DATE = to_date(?,'DD/MM/YYYY') and RML_ID_FK = ? and BS_RW04_SEQ = ?";

        JdbcTemplate jdbcTemplate = new JdbcTemplate(dataSource);

        TransactionDefinition def = new DefaultTransactionDefinition();
        TransactionStatus status = transactionManager.getTransaction(def);

        String particulars = value.get(0) != null ? value.get(0) : "" ;
        BigDecimal prevProvAmt = value.get(1) != null ? new BigDecimal(value.get(1)) : new BigDecimal("0.00");
        BigDecimal writeOffAmt = value.get(2) != null ? new BigDecimal(value.get(2)) : new BigDecimal("0.00");
        BigDecimal additionAmt = value.get(3) != null ? new BigDecimal(value.get(3)) : new BigDecimal("0.00");
        BigDecimal reductionAmt = value.get(4) != null ? new BigDecimal(value.get(4)) : new BigDecimal("0.00");
        BigDecimal currProvAmt = value.get(5)  != null ? new BigDecimal(value.get(5)) : new BigDecimal("0.00");
        BigDecimal provRate = value.get(6)  != null ? new BigDecimal(value.get(6)) : new BigDecimal("0.00");
        BigDecimal provReqAmt = value.get(7)  != null ? new BigDecimal(value.get(7)) : new BigDecimal("0.00");

        Map<String, Object> resultMap = new HashMap<>();

        if (rowId == 0) {
            rowId = getRowSeq();

            insert = jdbcTemplate.update(insertQuery, rowId, circleCode, quarterDate, reportId, particulars, prevProvAmt, writeOffAmt, additionAmt, reductionAmt, currProvAmt, provRate, provReqAmt, rowId);

            if (insert == 1) {
                resultMap.put("rowId", rowId);
                transactionManager.commit(status);
            }
            else{
                resultMap.put("message", "Insert was not complete. Try Again.");
                transactionManager.rollback(status);
            }
        } else {
            update = jdbcTemplate.update(updateQuery, rowId, particulars, prevProvAmt, writeOffAmt, additionAmt, reductionAmt, currProvAmt, provRate, provReqAmt, circleCode, quarterDate, reportId, rowId);
            if (update == 1) {
                resultMap.put("update", update);
                transactionManager.commit(status);
            }
            else{
                resultMap.put("message", "Update was not complete. Try Again.");
                transactionManager.rollback(status);
            }
        }
        return resultMap;
    }

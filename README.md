public Map<String, Object> deleteRow(Map<String, Object> payload) {
        String userRole = (String) payload.get("userCapacity");
        int rowId = Integer.parseInt((String) payload.get("rowId"));

        if(userRole.equalsIgnoreCase("51")) {
            Map<String, Object> delete;
            delete = rw04Dao.deleteRow(rowId);
            return delete;
        }
        else{
            String message = "Unauthorized user";
            return new HashMap<>(Collections.singletonMap("retVal", message));
        }
    }
	
	
 public Map<String, Object> deleteRow(int rowId) {
        int delete;

        String deleteQuery = "delete from BS_RW04 where BS_RW04_SEQ = ?";

        JdbcTemplate jdbcTemplate = new JdbcTemplate(dataSource);

        TransactionDefinition def = new DefaultTransactionDefinition();
        TransactionStatus status = transactionManager.getTransaction(def);

        delete = jdbcTemplate.update(deleteQuery, rowId);

        Map<String, Object> resultMap = new HashMap<>();
        if (delete == 1) {
            resultMap.put("delete", delete);
            transactionManager.commit(status);
        }
        else {
            resultMap.put("message", "Delete was not complete. Try Again.");
            transactionManager.rollback(status);
        }
        return resultMap;
    }	

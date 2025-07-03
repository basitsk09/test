import org.springframework.jdbc.core.namedparam.NamedParameterJdbcTemplate;
import org.springframework.transaction.TransactionDefinition;
import org.springframework.transaction.TransactionStatus;
import org.springframework.transaction.PlatformTransactionManager;
import javax.sql.DataSource;
import java.util.Collections;
import java.util.HashMap;
import java.util.List;
import java.util.Map;
// ... other necessary imports

public class RW04DaoImpl implements RW04Dao {

    private final DataSource dataSource;
    private final PlatformTransactionManager transactionManager;

    // Your existing constructor
    public RW04DaoImpl(DataSource dataSource,  PlatformTransactionManager transactionManager) {
        this.dataSource = dataSource;
        this.transactionManager = transactionManager;
    }

    // ... your other existing methods like getRW04Data ...


    /**
     * Deletes multiple rows from the BS_RW04 table based on a list of IDs.
     * THIS IS THE CORRECTED IMPLEMENTATION.
     * @param rowIds A list of BS_RW04_ID values to be deleted.
     * @return A map indicating the result of the operation.
     */
    @Override
    public Map<String, Object> deleteRows(List<Integer> rowIds) {
        // Handle cases where the list is empty or null to prevent errors.
        if (rowIds == null || rowIds.isEmpty()) {
            return Collections.singletonMap("message", "No row IDs provided for deletion.");
        }

        // IMPORTANT: Use NamedParameterJdbcTemplate for safely handling the "IN (...)" clause
        NamedParameterJdbcTemplate namedJdbcTemplate = new NamedParameterJdbcTemplate(dataSource);

        // The SQL query now uses a named parameter ':ids'
        // Also confirming we are deleting by BS_RW04_ID, which matches the frontend dbId
        String deleteQuery = "delete from BS_RW04 where BS_RW04_ID IN (:ids)";

        TransactionDefinition def = new DefaultTransactionDefinition();
        TransactionStatus status = transactionManager.getTransaction(def);
        Map<String, Object> resultMap = new HashMap<>();

        try {
            // Create a parameter map where the key matches the named parameter in the query
            Map<String, List<Integer>> params = Collections.singletonMap("ids", rowIds);
            
            // Execute the update using the named parameter template
            int deletedCount = namedJdbcTemplate.update(deleteQuery, params);

            resultMap.put("deletedCount", deletedCount);
            transactionManager.commit(status);
            
        } catch (Exception e) {
            // Rollback transaction on any exception
            transactionManager.rollback(status);
            // It's good practice to log the error
            // log.error("Error during batch delete in BS_RW04", e);
            resultMap.put("message", "An error occurred during deletion. The operation was rolled back.");
        }

        return resultMap;
    }
}

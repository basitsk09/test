You've found the exact problem. 👍
If you're only getting the auditflag in the 0th index, it confirms that the SQL query in your findRequestDetailsById method is incorrect. It's only selecting one column, but your Java code expects three.
To fix this, you need to replace the current SQL in that repository method with the correct query that joins the necessary tables and selects all three required pieces of information.
## The Solution
Replace the entire findRequestDetailsById method in your FrtDataRepository.java file with the following code. This query will return the three columns your service layer code is expecting.
@Repository
public interface FrtDataRepository extends JpaRepository<AuditStatusRequest, String> {

    /**
     * This is the corrected query.
     * It joins the necessary tables to select three columns:
     * 1. The branch code.
     * 2. The calculated 'before' status string.
     * 3. The calculated 'after' status string.
     */
    @Query(value = "SELECT " +
                   "    cas.as_branch, " +
                   "    CASE " +
                   "        WHEN bm.crs_auditable = 'Y' THEN (CASE WHEN nvl(cia.ifcofr_audit_flag, 'N') = 'Y' THEN 'IFCOFR Audited' ELSE 'Audited' END) " +
                   "        ELSE 'Non-Audited' " +
                   "    END as before_status, " +
                   "    CASE cas.as_new_status " +
                   "        WHEN 'A' THEN 'Audited' " +
                   "        WHEN 'I' THEN 'IFCOFR Audited' " +
                   "        ELSE 'Non-Audited' " +
                   "    END as after_status " +
                   "FROM " +
                   "    crs_audit_status cas " +
                   "JOIN branch_master bm ON cas.as_branch = bm.branchno " +
                   "LEFT JOIN crs_ifcofr_audit cia ON cas.as_branch = cia.ifcofr_branch AND cia.ifcofr_date = TO_DATE(:quarterDate, 'DD/MM/YYYY') " +
                   "WHERE cas.as_id = :reqId", nativeQuery = true)
    Object[] findRequestDetailsById(@Param("reqId") String reqId, @Param("quarterDate") String quarterDate);

    // ... all your other repository methods ...
}

After updating your repository with this correct query, the details array in your service will have 3 elements, and the ArrayIndexOutOfBoundsException will be resolved.

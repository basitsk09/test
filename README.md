Of course. Based on the business logic in your original legacy code, here are the proper, secure, and efficient native queries for your BulkUploadRepository.
These queries replace the placeholders and insecure string concatenation from the previous examples with specific, parameterized SQL that correctly reflects your application's validation rules.
## 1. Query to Get Current Branch Audit Statuses
This query accurately translates the complex CASE statement from your original getBranchlist method. It fetches the current audit status ('I', 'A', or 'N') for a given list of branches and a specific quarter-end date.
The Query:
SELECT
    bm.BRANCHNO AS branch_code,
    CASE
        WHEN bm.CRS_AUDITABLE = 'Y' THEN
            COALESCE(
                (SELECT
                    CASE
                        WHEN cia.ifcofr_audit_flag = 'Y' THEN 'I'
                        ELSE 'A'
                    END
                 FROM crs_ifcofr_audit cia
                 WHERE cia.ifcofr_branch = bm.BRANCHNO
                 AND cia.IFCOFR_DATE = TO_DATE(:quarterEndDate, 'DD/MM/YYYY')),
                'A' -- Default to 'A' if no record found in crs_ifcofr_audit
            )
        ELSE bm.CRS_AUDITABLE -- Will be 'N'
    END AS current_status
FROM
    BRANCH_MASTER bm
WHERE
    bm.BRANCHNO IN :branchCodes

 * COALESCE: This is added to handle cases where a branch is auditable (CRS_AUDITABLE = 'Y') but has no matching entry in crs_ifcofr_audit. It prevents the subquery from returning NULL.
 * Parameters: It securely uses :branchCodes and :quarterEndDate to prevent SQL injection.
 * Efficiency: The WHERE bm.BRANCHNO IN :branchCodes clause ensures you only query the database for the branches you need to validate.
## 2. Query to Get Pending Requests
This query checks which of the supplied branches already have a request in a 'Pending' state for the given quarter. This query was already well-defined, but it is included here for completeness.
The Query:
SELECT
    AS_BRANCH
FROM
    CRS_AUDIT_STATUS
WHERE
    AS_REQ_STATUS = 'P'
    AND AS_BRANCH IN :branchCodes
    AND AS_QED = TO_DATE(:quarterEndDate, 'DD/MM/YYYY')

 * Purpose: It returns a simple list of branch codes that cannot be processed because they are already pending.
 * Security & Efficiency: It is both secure and efficient, using named parameters and filtering on the primary keys.
## Updated BulkUploadRepository.java
Here is your final repository interface with these proper queries implemented.
package com.yourpackage.repository;

import org.springframework.data.jpa.repository.Query;
import org.springframework.data.repository.Repository;
import org.springframework.data.repository.query.Param;

import java.util.List;

public interface BulkUploadRepository extends Repository<Object, Long> {

    /**
     * Finds the current, calculated audit status for a list of branches.
     * Replaces the logic from the old getBranchlist() method.
     *
     * @param branchCodes List of branch codes to check.
     * @param quarterEndDate The quarter-end date as a String (e.g., "30/06/2025").
     * @return A list of Object arrays, where each array contains [branch_code, current_status].
     */
    @Query(value = """
        SELECT
            bm.BRANCHNO AS branch_code,
            CASE
                WHEN bm.CRS_AUDITABLE = 'Y' THEN
                    COALESCE(
                        (SELECT
                            CASE
                                WHEN cia.ifcofr_audit_flag = 'Y' THEN 'I'
                                ELSE 'A'
                            END
                         FROM crs_ifcofr_audit cia
                         WHERE cia.ifcofr_branch = bm.BRANCHNO
                         AND cia.IFCOFR_DATE = TO_DATE(:quarterEndDate, 'DD/MM/YYYY')),
                        'A'
                    )
                ELSE bm.CRS_AUDITABLE
            END AS current_status
        FROM
            BRANCH_MASTER bm
        WHERE
            bm.BRANCHNO IN :branchCodes
    """, nativeQuery = true)
    List<Object[]> findBranchStatusesByCodes(@Param("branchCodes") List<String> branchCodes, @Param("quarterEndDate") String quarterEndDate);

    /**
     * Finds all branches from a given list that have a pending request for a specific quarter.
     * Replaces the logic from the old getBranchRequestPendingList() method.
     *
     * @param branchCodes List of branch codes to check.
     * @param quarterEndDate The quarter-end date as a String (e.g., "30/06/2025").
     * @return A list of branch codes that have pending requests.
     */
    @Query(value = """
        SELECT
            AS_BRANCH
        FROM
            CRS_AUDIT_STATUS
        WHERE
            AS_REQ_STATUS = 'P'
            AND AS_BRANCH IN :branchCodes
            AND AS_QED = TO_DATE(:quarterEndDate, 'DD/MM/YYYY')
    """, nativeQuery = true)
    List<String> findPendingRequestsForBranches(@Param("branchCodes") List<String> branchCodes, @Param("quarterEndDate") String quarterEndDate);
}


private String getUserToken(String userID) {
    String sessionId = null;
    Connection con = null;
    PreparedStatement ps = null;
    ResultSet rs = null;

    try {
        con = new DBConn().getConnectionFromJNDI();

        // Ensure this SELECT runs in autocommit, read-only mode to avoid row locks
        con.setReadOnly(true);
        con.setAutoCommit(true); // Optional but safe

        String query = "SELECT USER_SESSION FROM bs_user WHERE user_id = ?";
        ps = con.prepareStatement(query);
        ps.setString(1, userID);
        rs = ps.executeQuery();

        if (rs.next()) {
            sessionId = rs.getString("USER_SESSION");
        }

    } catch (SQLException e) {
        logger.error("SQL Exception in getUserToken: " + e.getMessage(), e);
    } finally {
        try {
            if (rs != null) rs.close();
        } catch (SQLException e) {
            logger.warn("Failed to close ResultSet", e);
        }
        try {
            if (ps != null) ps.close();
        } catch (SQLException e) {
            logger.warn("Failed to close PreparedStatement", e);
        }
        try {
            if (con != null) con.close();
        } catch (SQLException e) {
            logger.warn("Failed to close Connection", e);
        }
    }

    return sessionId;
}



import org.springframework.transaction.annotation.Propagation;
import org.springframework.transaction.annotation.Transactional;

@Transactional(propagation = Propagation.REQUIRES_NEW)
public int saveToken(UserLogin userLogin, String token) {
    try {
        // Step 1: Check if update is actually needed
        String existingToken = jdbcTemplate.queryForObject(
            "SELECT USER_SESSION FROM bs_user WHERE user_id = ?",
            new Object[]{userLogin.getUserId()},
            String.class
        );

        if (token.equals(existingToken)) {
            return 0; // No update needed
        }

        // Step 2: Perform update
        String updateQuery = "UPDATE bs_user SET USER_SESSION = ?, USER_LAST_LOGIN = SYSDATE WHERE user_id = ?";
        return jdbcTemplate.update(updateQuery, token, userLogin.getUserId());

    } catch (Exception e) {
        logger.error("Error in saveToken for user " + userLogin.getUserId() + ": " + e.getMessage(), e);
        return 0;
    }
}
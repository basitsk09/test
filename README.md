import java.time.LocalDateTime;
import java.time.format.DateTimeFormatter;

// Inside your if condition
if (writeOffStatus.equalsIgnoreCase("20") && Integer.parseInt(reportStatus) > 20) {
    DateTimeFormatter formatter = DateTimeFormatter.ofPattern("dd-MM-yyyy HH:mm:ss");
    String currentDateTime = LocalDateTime.now().format(formatter);

    String remarks = "Rejected due to the change in Write-Off Total. Time of Rejection: " + currentDateTime;

    String updateMiscRML = "UPDATE BS_MISC_RPT SET MISC_STATUS = ?, MISC_CHECKER_REMARKS = ? " +
                           "WHERE MISC_CIRCLE = ? AND MISC_QDATE = TO_DATE(?, 'YYYY-MM-DD')";
    jdbcTemplate.update(updateMiscRML, "12", remarks, circle, quarterDate);
}
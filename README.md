// Inside your service method (e.g., generateBranchListXlsx)

// You need to get the circle code for the current user.
// This might come from a session, a security context, or the request itself.
// For example: String userCircleCode = sessionBean.getCircleCode();
String userCircleCode = "YOUR_USER_CIRCLE_CODE"; // <-- Replace this with the actual dynamic value

Map<String, Object> parameters = new HashMap<>();
parameters.put("IS_IGNORE_PAGINATION", true);
parameters.put("CIRCLE_CODE", userCircleCode); // <-- Add this line

// Now fill the report with the parameters
JasperPrint jasperPrint = JasperFillManager.fillReport(jasperStream, parameters, con);

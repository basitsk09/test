Of course. The error is because you're iterating over requestIds.toString() instead of the list itself, and you have a type mismatch.
To fix this, you should retrieve the list as List<Integer> and then convert each integer to a String inside the loop.
Here is the corrected code:
// Retrieve the list correctly as a List of Integers.
List<Integer> requestIds = (List<Integer>) data.get("requestIds");
String quarterDate = (String) data.get("quarterDate");
String userId = String.valueOf(data.get("userId"));

if (requestIds == null || requestIds.isEmpty() || quarterDate == null) {
    response.put("message", "Bad Request: 'requestIds' and 'quarterDate' are required.");
    return new ResponseEntity<>(response, HttpStatus.BAD_REQUEST);
}

int successCount = 0;
try {
    // 1. Iterate directly over the 'requestIds' list, not its string representation.
    for (Integer reqId : requestIds) {

        // 2. Convert the Integer 'reqId' to a String before passing it to the repository.
        String reqIdAsString = String.valueOf(reqId);

        Object[] details = crsAuditStatusRepository.findRequestDetailsById(reqIdAsString, quarterDate);
        if (details == null) continue;
        
        // ... continue your logic
    }
    // ...
} catch (Exception e) {
    // ...
}

Why This Works
 * Correct Iteration: The loop for (Integer reqId : requestIds) now correctly iterates through each Integer element in the list. Your original code, requestIds.toString(), was converting the list to a single string like "[123, 456]" and then looping through its individual characters ('[', '1', '2', '3', ...).
 * Type Conversion: String.valueOf(reqId) safely converts each Integer from the list into the String format that your repository method findRequestDetailsById expects.

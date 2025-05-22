import java.util.Base64;

// ...
byte[] zipBytes = byteArrayOutputStream.toByteArray();
String encodedZip = Base64.getEncoder().encodeToString(zipBytes);
result.put(ZIP_CONTENT, encodedZip);
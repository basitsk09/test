public void setNewBody(String newBody) {
    this.cachedBody = newBody.getBytes(StandardCharsets.UTF_8);
}

wrappedRequest.setNewBody(decryptedData); // inject decrypted JSON into request body

Map<String, Object> dataMap = mapper.readValue(decryptedData, Map.class);
wrappedRequest.setAttribute("data", dataMap);
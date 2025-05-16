String body = new BufferedReader(wrappedRequest.getReader())
                .lines()
                .collect(Collectors.joining(System.lineSeparator()));

if (body == null || body.trim().isEmpty()) {
    log.warn("Request body is empty. Skipping decryption.");
    chain.doFilter(wrappedRequest, response);
    return;
}
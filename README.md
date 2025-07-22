 List<String> requestIds = (List<String>) data.get("requestIds");
        String quarterDate = (String) data.get("quarterDate");
        String userId = String.valueOf(data.get("userId")) ;

        if (requestIds == null || requestIds.isEmpty() || quarterDate == null) {
            response.put("message", "Bad Request: 'requestIds' and 'quarterDate' are required.");
            return new ResponseEntity<>(response, HttpStatus.BAD_REQUEST);
        }

        int successCount = 0;
        try {
            for (String reqId : requestIds.toString()) {

                Object[] details = crsAuditStatusRepository.findRequestDetailsById(reqId, quarterDate);
                if (details == null) continue;

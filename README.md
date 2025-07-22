remove this : [span_9](start_span)

 @PostMapping("/FrtSubmitRequest")
    public ResponseEntity<Map<String,Object>> frtSubmitRequest(@RequestBody  Map<String,Object> map){
        log.info("map==="+map);
        return lhoEditBranchDetailsService.frtSubmitRequest(map);
    }

   @PostMapping("/FrtSubmitRequest")
    public ResponseEntity<Map<String,Object>> frtSubmitRequest(@RequestBody  Map<String,Object> map){
        log.info("map==="+map);
        return lhoEditBranchDetailsService.frtSubmitRequest(map);
    }

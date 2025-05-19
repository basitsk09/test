// Check if record already exists
Optional<RORML> existing = rormlRepository.findById(
    new ReportSubmissionIdClass(submissionId, quarterEndDate, branchCode, reportId)
);

if (existing.isPresent()) {
    RORML rml = existing.get();
    rml.setRoId(user.get("pf_number").toString());
    rml.setRoAccRejFlag("R");
    rml.setCurrentStatus("24");
    rml.setRoAccRejRemark(remarks);
    rml.setRoAccRejDate(new Date()); // Update date if needed
    rormlRepository.save(rml); // This will perform update
} else {
    RORML rml = new RORML();
    rml.setSubmissionId(submissionId);
    rml.setQuarterDate(quarterEndDate);
    rml.setBranchCode(branchCode);
    rml.setReportId(reportId);
    rml.setRoId(user.get("pf_number").toString());
    rml.setRoAccRejFlag("R");
    rml.setCurrentStatus("24");
    rml.setRoAccRejRemark(remarks);
    rml.setRoAccRejDate(new Date());
    rormlRepository.save(rml); // This will insert
}
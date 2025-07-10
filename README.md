private byte[] getCurrentYearComplianceReport(List<String> branchCodes, Date quarterEndDate, String reportId) {
        log.info("Inside Get Current Year Compliance LFAR");

        try (ByteArrayOutputStream baos = new ByteArrayOutputStream()) {
            ZipOutputStream zipOut = new ZipOutputStream(baos);
            //For LFAR 1, 4, 5, 6 Compliance
            if(reportId.equals("4019") ||  reportId.equals("4023") || reportId.equals("4025") || reportId.equals("4027") || reportId.equals("4015") || reportId.equals("4021") ){
            List<Map<String, Object>> reportList = lfarReportsRepository.getReportsSubmissionIdList(branchCodes, quarterEndDate, reportId);

            byte[] pdfBytes = null;
            for (Map<String, Object> report : reportList) {
                byte[] reportData = lfarReportsRepository.getComplianceReportsByBranchQedForLfar(quarterEndDate,
                        Integer.parseInt(String.valueOf(report.get("REPORTS_SUBMISSION_ID"))));
                if (reportData.length == 0) {
                    continue;
                }
                String str = new String(reportData, StandardCharsets.UTF_8);
                try {
                    pdfBytes = Base64.getDecoder().decode(str);
                } catch (RuntimeException e) {
                    log.info("This blob is not Base64");
                    pdfBytes = reportData;
                }
                String reportName = (lfarReportsRepository.getBranchCode(quarterEndDate,
                        Integer.parseInt(String.valueOf(report.get("REPORTS_SUBMISSION_ID"))))) + "_" + report.get("REPORT_NAME") + "Compliance.pdf";
                log.info("Report Name :: {}", reportName);
                zipOut.putNextEntry(new ZipEntry(reportName));
                zipOut.write(pdfBytes);
                zipOut.closeEntry();
            }
            }

            //For LFAR2 Compliance + Evidence.
            if(reportId.equals("4021")){
                    log.info("Inside LFAR 2 Compliance and Evidence Download");
                    for (String branchCode : branchCodes) {
                        ByteArrayOutputStream baosTemp = new ByteArrayOutputStream();
                        ZipOutputStream lfarZipOut = new ZipOutputStream(baosTemp);
                        int noOfEntries =0;
                        List<Map<String, Object>> lfar2ReportsData = lfarReportsRepository.getLfar2IdForCompliance(quarterEndDate, branchCode);
                        if (!lfar2ReportsData.isEmpty()) {
                            for (Map<String, Object> lfar2 : lfar2ReportsData) {
                                int lfar2Id = Integer.parseInt(String.valueOf(lfar2.get("LFAR2_ID")));
                                log.info(lfar2Id);
                                byte[] complianceData = lfarReportsRepository.getLfar2Compliance(lfar2Id);
                                byte[] EvidenceData = lfarReportsRepository.getLfar2Evidence(lfar2Id);

                                if (complianceData != null) {
                                    // String accountNumber = lfar2.get("LFAR2_ACCT").toString();
                                    String complianceName = lfar2.get("LFAR2_COMPLIANCE_NAME").toString();
                                    log.info(complianceName);
                                    lfarZipOut.putNextEntry(new ZipEntry(complianceName));
                                    lfarZipOut.write(complianceData);
                                    noOfEntries++;
                                }
                                if (EvidenceData != null) {
                                    // String accountNumber = lfar2.get("LFAR2_ACCT").toString();
                                    String evidenceName = lfar2.get("LFAR2_EVIDENCE_NAME").toString();
                                    log.info(evidenceName);
                                    lfarZipOut.putNextEntry(new ZipEntry(evidenceName));
                                    lfarZipOut.write(EvidenceData);
                                    noOfEntries++;
                                }

                            }
                        }
                        log.info("Size of baosTemp :: {}", baosTemp.size());
                        lfarZipOut.finish();
                        lfarZipOut.close();
                        baosTemp.close();
                        log.info("Size of baosTemp :: {}", baosTemp.size());
                        String zipName = branchCode + "_LFAR2_COMPLIANCE_EVIDENCE.zip";
                        log.info("zip :: {}", zipName);

                        if (noOfEntries > 0) {
                            log.info("Writing into file");
                            zipOut.putNextEntry(new ZipEntry(zipName));
                            zipOut.write(baosTemp.toByteArray());
                        }

                    }

            }
            //For LFAR3  Evidence.
            if (reportId.equals("4015")) {
                log.info("Inside LFAR 3 Compliance and Evidence Download");
                List<Map<String, Object>> reportList = lfarReportsRepository.getEvidenceSubmissionIdListForLFAR3(branchCodes, quarterEndDate);

                byte[] pdfBytes = null;
                for (Map<String, Object> report : reportList) {
                    byte[] reportData = lfarReportsRepository.getEvicenceReportsByBranchQedForLfar3(quarterEndDate,
                            Integer.parseInt(String.valueOf(report.get("LFAR_ID"))));
                    if (reportData.length == 0) {
                        continue;
                    }
                    String str = new String(reportData, StandardCharsets.UTF_8);
                    try {
                        pdfBytes = Base64.getDecoder().decode(str);
                    } catch (RuntimeException e) {
                        log.info("This blob is not Base64");
                        pdfBytes = reportData;
                    }
                    String reportName = (lfarReportsRepository.getBranchCodeForLFAR3(quarterEndDate,
                            Integer.parseInt(String.valueOf(report.get("LFAR_ID"))))) + "_" + report.get("EVIDENCE_NAME") + "Evidence.pdf";
                    log.info("Report Name :: {}", reportName);
                    zipOut.putNextEntry(new ZipEntry(reportName));
                    zipOut.write(pdfBytes);
                    zipOut.closeEntry();
                }
            }
            zipOut.finish();
            zipOut.close();
            return baos.toByteArray();

        } catch (IOException e) {
            log.error("Exception occurred in getCurrentYearArchive{}", e.getMessage());
            return new byte[0];
        }
    }

 public Map<String, Object> downloadCompZip(Map<String, Object> payload) {
        Map<String, Object> result = new HashMap<>();

        try (ByteArrayOutputStream byteArrayOutputStream = new ByteArrayOutputStream();
             ZipOutputStream zipOutputStream = new ZipOutputStream(byteArrayOutputStream)) {

            Map<String, Object> data = (Map<String, Object>) payload.get("data");
            String submissionId = Optional.ofNullable(data.get("submissionId")).map(Object::toString).orElseThrow(() -> new IllegalArgumentException("Missing submissionId"));
            String reportId = Optional.ofNullable(data.get("reportId")).map(Object::toString).orElseThrow(() -> new IllegalArgumentException("Missing reportId"));

            // Retrieve the Compliance report PDF from lfar_reports table [Common for LFAR2 & LFAR3]
            List<Object[]> reportPdfList = lfarReportsRepository.getCompliancePdfBytesAndName(submissionId);

            if (reportId.equalsIgnoreCase("4021")) {
                getLfar2Evidences(submissionId, zipOutputStream);
            } else if (reportId.equalsIgnoreCase("4015")) {
                getLfar3Evidences(submissionId, zipOutputStream);
            }

            // ADD retrieved comp report pdf to the zip
            if (!reportPdfList.isEmpty()) {
                for (Object[] reportPdf : reportPdfList) {
                    if (reportPdf.length > 1) {
                        Blob blob = (Blob) reportPdf[0];
                        if (blob != null) {
                            byte[] complianceReportData = blob.getBytes(1, (int) blob.length());
                            String complianceName = (String) reportPdf[1];

                            // Check if pdfBytes is not null or empty
                            if (complianceReportData != null && complianceReportData.length > 0) {
                                addFileToZip(zipOutputStream, complianceName, complianceReportData);
                            }
                        }
                    }
                }
            } else {
                log.info("No comp report data found for submissionId: " + submissionId);
            }

            zipOutputStream.finish();

            result.put(ZIP_CONTENT, byteArrayOutputStream.toByteArray());
            result.put(STATUS, true);
            log.info("zip added to the result successfully :- " + byteArrayOutputStream.size());

        } catch (Exception e) {
            log.info("Exception occurred while downloading Compliance ZIP : " + e.getMessage());
            result.put(STATUS, false);
            result.put("message", e.getMessage());
        }

        return result;
    }
	
	
	
	private void getLfar3Evidences(String submissionId, ZipOutputStream zipOutputStream) throws IOException {
        List<CrsLfar> reportFiles = lfar3Repository.findByReportMasterListIdFk(submissionId);
        // Add compliance PDF and evidence PDF from Lfar3
        if (!reportFiles.isEmpty()) {
            String folderName = "Lfar3Evidences/";

            zipOutputStream.putNextEntry(new ZipEntry(folderName));
            zipOutputStream.closeEntry();
            int fileCounter = 1;
            for (CrsLfar lfar3 : reportFiles) {
                // Add Evidence PDF to the ZIP
                if (lfar3.getEvidenceFile() != null && lfar3.getEvidenceFile().length > 0) {
                    String filePath = folderName + fileCounter + "_" + lfar3.getEvidenceName();
                    addFileToZip(zipOutputStream, filePath, lfar3.getEvidenceFile());
                    fileCounter++;
                }
            }
        }
    }
	
	 private void getLfar3Evidences(String submissionId, ZipOutputStream zipOutputStream) throws IOException {
        List<CrsLfar> reportFiles = lfar3Repository.findByReportMasterListIdFk(submissionId);
        // Add compliance PDF and evidence PDF from Lfar3
        if (!reportFiles.isEmpty()) {
            String folderName = "Lfar3Evidences/";

            zipOutputStream.putNextEntry(new ZipEntry(folderName));
            zipOutputStream.closeEntry();
            int fileCounter = 1;
            for (CrsLfar lfar3 : reportFiles) {
                // Add Evidence PDF to the ZIP
                if (lfar3.getEvidenceFile() != null && lfar3.getEvidenceFile().length > 0) {
                    String filePath = folderName + fileCounter + "_" + lfar3.getEvidenceName();
                    addFileToZip(zipOutputStream, filePath, lfar3.getEvidenceFile());
                    fileCounter++;
                }
            }
        }
    }
	
	private void addFileToZip(ZipOutputStream zipOutputStream, String fileName, byte[] fileData) throws IOException {
        if (fileData != null && fileData.length > 0) {
            zipOutputStream.putNextEntry(new ZipEntry(fileName));
            zipOutputStream.write(fileData);
            zipOutputStream.closeEntry();
            log.info(fileName + " added to the zip");
        }
    }

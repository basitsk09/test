  @PostMapping("/downloadCompZip")
    public ResponseEntity<ResponseVo> downloadCompZip(@RequestBody Map<String, Object> payload) {
        ResponseVo responseVO = new ResponseVo();
        Map<String, Object> downloadedZip = lfarDownloadReportService.downloadCompZip(payload);

        if(downloadedZip.get("status").toString().equalsIgnoreCase("true")){
            responseVO.setResult(downloadedZip);
            responseVO.setMessage("Compliance ZIP Generated");
            responseVO.setStatusCode(HttpStatus.OK.value());
        } else {
            responseVO.setResult(downloadedZip);
            responseVO.setMessage("Failed to generated Compliance ZIP");
            responseVO.setStatusCode(HttpStatus.INTERNAL_SERVER_ERROR.value());
        }
        return new ResponseEntity<>(responseVO, HttpStatus.OK);
    }
  
  
  
  
  
  @Override
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

 
 
 
 
 private void getLfar3Evidences(String submissionId, ZipOutputStream zipOutputStream) throws IOException, SQLException {
        List<Object[]> reportFiles = lfar3Repository.getBlobStreamBySubmissionId(submissionId);
        if (!reportFiles.isEmpty()) {
            String folderName = "Lfar3Evidences/";
            zipOutputStream.putNextEntry(new ZipEntry(folderName));
            zipOutputStream.closeEntry();
            int fileCounter = 1;

            for (Object[] row : reportFiles) {
                Blob blob = (Blob) row[0];
                String fileName = (String) row[1];

                if (blob != null && blob.length() > 0) {
                    String zipPath = folderName + fileCounter + "_" + fileName;
                    try (InputStream in = blob.getBinaryStream()) {
                        streamFileToZip(zipOutputStream, zipPath, in);
                    }
                    fileCounter++;
                }
            }
        }
    }

    private void streamFileToZip(ZipOutputStream zipOutputStream, String fileName, InputStream inputStream) throws IOException {
        zipOutputStream.putNextEntry(new ZipEntry(fileName));
        byte[] buffer = new byte[8192]; // 8 KB buffer
        int bytesRead;
        while ((bytesRead = inputStream.read(buffer)) != -1) {
            zipOutputStream.write(buffer, 0, bytesRead);
        }
        zipOutputStream.closeEntry();
        log.info(fileName + " added to zip (streamed)");
    }
	
	
	
/////////////////////////////

export default function downloadFile(response, format, name) {
  let fileType = "";
  let fileExtension = "";
  console.log("Response",  format, name)
  // Set the MIME type and file extension based on the format
  switch (format) {
    case "pdf":
      fileType = "application/pdf";
      fileExtension = "pdf";
      break;
    case "excel":
      fileType =
        "application/vnd.openxmlformats-officedocument.spreadsheetml.sheet";
      fileExtension = "xlsx";
      break;
    case "zip":
      fileType = "application/zip";
      fileExtension = "zip";
      break;
    case "word":
      fileType =
        "application/vnd.openxmlformats-officedocument.wordprocessingml.document";
      fileExtension = "docx";
      break;
    default:
      console.error("Unsupported file format");
      return;
  }

  /**
   * Converting base64 String data to byte array.*/
  const byteCharecters = atob(response);
  const byteNumbers = new Array(byteCharecters.length);
  for (let i = 0; i < byteCharecters.length; i++) {
    byteNumbers[i] = byteCharecters.charCodeAt(i);
  }
  const byteArray = new Uint8Array(byteNumbers);

  /**
   * Converting byte array to URL using Blob*/
  const blob = new Blob([byteArray], { type: fileType });
  console.log(blob);
  const url = URL.createObjectURL(blob);
  console.log(url);

  // Trigger download by creating a temporary anchor tag
  const link = document.createElement("a");
  link.href = url;
  link.download = `${name}.${fileExtension}`;
  console.log(link.download);

  document.body.appendChild(link);

  link.click();

  // Clean up
  link.remove();
  window.URL.revokeObjectURL(url);
}

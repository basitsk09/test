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
            if (reportId.equalsIgnoreCase("4015")) {
                byte[] zipBytes = byteArrayOutputStream.toByteArray();
                String encodedZip = Base64.getEncoder().encodeToString(zipBytes);
                result.put(ZIP_CONTENT, encodedZip);
            }else{
                result.put(ZIP_CONTENT, byteArrayOutputStream.toByteArray());
            }


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
JS files

const downloadComplianceFiles = async (
  report,
  reportId,
  submissionId,
  complianceFlag = false,
  showSnackbar,
  setLoadOpen
) => {
  setLoadOpen(true);
  console.log({ report, reportId, submissionId, complianceFlag });
  const name = report.reportName.replace("-", "_") + "_Compliance";
  let dataToSend = {
    reportId: reportId,
    submissionId: submissionId,
    complianceFlag: complianceFlag,
  };
  let jsonFormData = JSON.stringify(dataToSend);
  await encrypt(iv, salt, jsonFormData).then(function (result) {
    jsonFormData = result;
  });
  let payload = {
    iv: ivBase64,
    salt: saltBase64,
    data: jsonFormData,
  };
  try {
    let url = "Server/commonScreenService/downloadComplianceZip";
    if (reportId !== "4021" && reportId !== "4015") {
      url = "/Server/commonScreenService/lfarDownload";
    }

    const response = await axios.post(url, payload, {
      headers: {
        Authorization: `Bearer ${localStorage.getItem("token")}`,
      },
    });

    if (response.data.result) {
      if (response.data.result.zipContent) {
        downloadFile(response.data.result.zipContent, "zip", name);
      } else if (response.data.result.pdfContent) {
        downloadFile(response.data.result.pdfContent, "pdf", name);
      } else {
        showSnackbar("Failed to download File", "error");
      }
      setLoadOpen(false);
    }
    return null;
  } catch (e) {
    setLoadOpen(false);
    showSnackbar("Failed to download File", "error");
  }
};


router.post("/downloadComplianceZip", extractToken, DataValidator, async (req, res) => {

  if (
    req.user.user_role !== "1" &&
    req.user.module !== "LFAR" &&
    req.user.user_role !== "11"
  ) {
    return res.status(401).json({
      message: "Unauthorized api call, request not from legal source",
    });
  }


  try {
    let serviceUrl = "";
    if (runningEnvironment === "dev") {
      serviceUrl = URL_CONST.DEV_URL[req.user.module].downloadComplianceZip;
    } else {
      serviceUrl = URL_CONST.PROD_URL[req.user.module].downloadComplianceZip;
    }
    console.log("URL Triggered " + serviceUrl)
    const response = await fetch(serviceUrl, {
      method: "POST",
      body: JSON.stringify({
        user: req.user,
        data: req.data,
      }),
      headers: {
        "Content-Type": "application/json",
      },
    });
    const data = await response.json();
    res.json(data);
  } catch (e) {
    console.error(e);
    res.status(500).json({
      status: "Failed to bring response, no connection between LB and APP",
    });
  }


})

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
  response = response.replace(/-/g,'+').replace(/_/g,'/');
  console.log("my response:",response);
  console.log("typeof",typeof response);
   const byteCharacters = atob(response);
  // const byteNumbers = new Array(byteCharecters.length);
  // for (let i = 0; i < byteCharecters.length; i++) {
  //   byteNumbers[i] = byteCharecters.charCodeAt(i);
  // }
  const byteArrays = [];
  const sliceSize=512;
  
  for (let offset = 0; offset < byteCharacters.length; offset += sliceSize) {
    const slice = byteCharacters.slice(offset, offset + sliceSize);
    
    const byteNumbers = new Array(slice.length);
    for (let i = 0; i < slice.length; i++) {
      byteNumbers[i] = slice.charCodeAt(i);
    }
    const byteArray = new Uint8Array(byteNumbers);
      
      byteArrays.push(byteArray);
  }
    
   // const byteArray = new Uint8Array(byteNumbers);
 // const byteArray = Uint8Array.from(atob(response), c=> c.charCodeAt(0));//new Uint8Array(byteNumbers);

  /**
   * Converting byte array to URL using Blob*/
  const blob = new Blob([byteArrays], { type: fileType });
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

@PostMapping("/SC10SFTP")
    public @ResponseBody Map<String, Object> SC10SFTP(@RequestBody Map<String, Object> map) throws ConfigurationException, SQLException, IOException {
        Map<String, Object> updatedTabData = new HashMap<>();

        log.info("Map Values Received from FE ::" + map);

        //Check the files at Destination Directory
//        Boolean getFile = ccdpSftpService.getsftpdata(map);
        Boolean getFile = ifamsSftpService.getSC10sftpdata(map);

        log.info("IS SC10SFTP FILE RECEIVED  ::" + getFile);


        // When get the report name as SC10
        if (getFile && ((String) map.get("reportName")).equalsIgnoreCase("SC10")) {

            // Add New Method Here for SFTP SC10-Files
            updatedTabData = ifamsSftpService.getSC10Sftp(map);
            log.info("updatedTabData SC10:-" + updatedTabData + "updatedTabData status " + updatedTabData.get("status"));

        } else {
            log.info("SC10 files not exist");

            // Checking is there any TimeStamp Stored in DB for SC10 Report
            Optional<Integer> count = ifamsSftpService.checkSC10TimeStamp(map);

            Optional<Integer> dataCount = ifamsSftpService.getSC10Countdata(map);

            log.info("TimeStamp SFTP checkSC10TimeStamp:-" + count.get().intValue());

            log.info("SC10 SFTP getSC10Countdata:-" + dataCount.get());


            if (count.get() == 1 && dataCount.get() >= 1) {

                String timest = ccdpSftpService.getCCDPTimeStamp(map);
                log.info("Getting getCCDPTimeStamp ::" + timest);

                updatedTabData.put("message", "Please note: \n Data fetched here was generated in IFAMS at " + timest);
                updatedTabData.put("status", true);
                updatedTabData.put("fileAndDataStatus", 3);

            } else {
                updatedTabData.put("message", "Data not received from IFAMS, Kindly wait till IFAMS sends reports");
                updatedTabData.put("status", false);
                updatedTabData.put("fileAndDataStatus", 2);
          /*  if(((String) map.get("reportName")).equalsIgnoreCase("ANX2C")){

            }*/
            }
        }


        updatedTabData.put("reportID", map.get("reportID"));
        updatedTabData.put("circleCode", map.get("circleCode"));
        updatedTabData.put("qed", map.get("qed"));
        updatedTabData.put("reportName", map.get("reportName"));
        updatedTabData.put("reportStatus", map.get("reportStatus"));

        if (getFile && ((String) map.get("reportName")).equalsIgnoreCase("SC10") && updatedTabData.get("fileAndDataStatus").toString().equalsIgnoreCase("1")) {

            log.info("inside UpdateSFTPFilesStatus ><><><><><><><>");
            int Result = ifamsSftpService.UpdateSFTPFilesStatus(updatedTabData);

            log.info("Result for UpdateSFTPFilesStatus ::" + Result);
        }


        return updatedTabData;
    }

///////////////////////////////////////////////////////////////

public Map<String, Object> getSC10Sftp(Map<String, Object> map) {
        log.info("Inside SC10DaoImpl Reading the .TXT File");
        Map<String, Object> updatedTabData = new HashMap<>();

        // Extract input parameters from the map
        String quarterEndDate = (String) map.get("qed");
        String circleCode = (String) map.get("circleCode");
        String reportName = (String) map.get("reportName");

        log.info("Quarter End Date: " + quarterEndDate);

        // Extract year, month, and day from quarterEndDate (format: dd/MM/yyyy)
        String[] dateParts = quarterEndDate.split("/");
        String yyyy = dateParts[2];
        String mm = dateParts[1];
        String dd = dateParts[0];

        // Generate required date formats
        String sessionDate = yyyy + mm + dd;  // Format: YYYYMMDD
        String qDate = dd + mm + yyyy;  // Format: DDMMYYYY

        log.info("Session Date: " + sessionDate);
        log.info("Fetching file...");

        try {
            // Retrieve file path from properties
            PropertiesConfiguration config = new PropertiesConfiguration("common.properties");

            //Path where file get Read
            String mainPath = config.getProperty("ReportDirIFAMS").toString();

            // Building the FileName Here with Complete Path
            String filePath = mainPath + qDate + "/IFAMS_SCH10_" + sessionDate + "_" + circleCode + ".txt";

            log.info("File Reading Path : " + mainPath);

            log.info("File Received Path: " + filePath);


            int[] rowNumber = {1, 3, 4, 36, 5, 6, 7, 37, 9, 33, 10, 11, 12, 13, 14, 18, 34,
                    38, 19, 20, 21, 39, 22, 40, 24, 25, 26, 27, 28,
                    29, 30, 31, 35, 32};
            int rowNumberCount = 0;

            List<String> lines = new ArrayList<>();

            // Read the file
            try (BufferedReader bufferedReader = new BufferedReader(new FileReader(filePath))) {
                String line;
                while ((line = bufferedReader.readLine()) != null) {
                    if (!line.trim().isEmpty()) {
                        lines.add(line);
                    }
                }
            }

            // Initialize SC10 object and storage for row data
            SC10 sc10 = new SC10();
            Map<Integer, String[]> rowData = new HashMap<>();
            String timeStamp = "";

            // Process each line from the file
            for (String line : lines) {
                String[] columns = line.split("\\|");

                //  Replace null or empty values with "0"
                for (int i = 0; i < columns.length; i++) {
                    if (columns[i] == null || columns[i].trim().isEmpty()) {
//                        log.warn("Empty value found at index " + i + ". Replacing with 0.");
                        columns[i] = "0";  //  Set empty values to "0"
                    }
                }

                // Extract timestamp if present
                if (columns[0].trim().equalsIgnoreCase("Generated at")) {
                    timeStamp = columns[1].trim();
                    log.info("Time Stamp Extracted from .txt File ::" + timeStamp);
                    updatedTabData.put("FILETIMESTAMP", timeStamp);
                    continue;
                }

                // Parse row number and store data
                int rowNum = rowNumber[rowNumberCount++];
                rowData.put(rowNum, columns);
            }


            // Define field names as per SC10.java (without row numbers)
            String[] fieldNames = {
                    "stcNstaff", "offResidenceA", "otherPremisesA", "electricFitting",
                    "totalA", "computers", "compSoftwareInt", "compSoftwareNonint",
                    "compSoftwareTotal", "motor", "offResidenceB", "stcLho",
                    "otherPremisesB", "otherMachineryPlant", "totalB", "totalFurnFix",
                    "landNotRev", "landRev", "landRevEnh", "offBuildNotRev",
                    "offBuildRev", "offBuildRevEnh", "residQuartNotRev", "residQuartRev",
                    "residQuartRevEnh", "premisTotal", "revtotal", "totalC",
                    "premisesUnderCons", "grandTotal"
            };

            // Step 1: Sort row numbers to maintain correct order
            List<Integer> sortedRows = new ArrayList<>(rowData.keySet());
            Collections.sort(sortedRows);

            // Step 2: Iterate over sorted rows and set values dynamically
            for (int row : sortedRows) {
                log.info("row : " + row);
                if (!rowData.containsKey(row)) {
                    log.info("Skipping row " + row + " as it's not present in the file.");
                    continue;  //  Skip missing row without setting any data
                }

                String[] data = rowData.get(row);

                //  Retrieve existing row data (guaranteed to be non-null)

                for (int index = 1; index <= 30; index++) {  //  Ensure all 30 values are processed
                    try {

                        String setterName = "set" + capitalize(fieldNames[index - 1]) + row;  //  Adjust index correctly
                        Method setterMethod = SC10.class.getMethod(setterName, String.class);
                        setterMethod.invoke(sc10, data[index].trim());  //  Only set values for present rows

                    } catch (NoSuchMethodException e) {
                        log.warn("No setter found: " + fieldNames[index - 1] + row);
                    } catch (Exception e) {
                        log.error("Error setting value for: " + fieldNames[index - 1] + row, e);
                    }
                }
            }

            // Update timestamp in CCDPFiletime Table database
            log.info("Updating / Inserting Data into CCDPFiletime " + "FILE Extracted timeStamp ::" + timeStamp + "circleCode ::" + circleCode + "quarterEndDate ::" + quarterEndDate + "reportName ::" + reportName);
            int updateTime = ccdpSftpDao.updateCCDPFiletime(timeStamp, circleCode, quarterEndDate, reportName);

            log.info("TImeStamp Updated for CCDP_FILE_TIME : Status :" + updateTime + "reportName: " + reportName);


            // Return response
            updatedTabData.put("sc10Data", sc10);
            updatedTabData.put("message", "Data received from IFAMS and imported successfully");
            updatedTabData.put("status", true);
            updatedTabData.put("fileAndDataStatus", 1);

        } catch (Exception e) {
            updatedTabData.put("message", "Error reading file");
            updatedTabData.put("fileAndDataStatus", 2);
            updatedTabData.put("status", false);
            log.error("Error while reading the file :"+e.getCause());
        }

        return updatedTabData;
    }

//////////////////////////////////////////////////////////

const checkSC10SftpData = async () => {
      const payload = {
        circleCode: user.circleCode,
        qed: formatDateToSlash(user.quarterEndDate),
        reportID: '310010',
        reportName: 'SC10',
        reportStatus: 'A',
      };

      try {
        const data = await callApi('/IFAMSS/SC10SFTP', payload, 'POST');

        console.log('SFTP response:', data);

        if (data.fileAndDataStatus === 1) {
          // SFTP Success
          // Assuming `data.data` contains the schedule form values
          setFormData(data?.sc10Data || {});
          setFieldsDisabled(true); // disables all inputs

          setSnackbar({
            open: true,
            message: 'Data successfully fetched from IFAMS via SFTP.',
            severity: 'success',
          });
        } else if (data.fileAndDataStatus === 2) {
          // File error or mismatch
          setFieldsDisabled(true);
          showDialog({
            title: 'File Error',
            message: data.message || 'Data not received from IFAMS, Kindly wait till IFAMS sends reports',
            onConfirm: () => navigate(-1), // go back
          });
        } else if (data.fileAndDataStatus === 3) {
          // Data already exists in DB but file was missing
          setFieldsDisabled(true);
          showDialog({
            title: 'Info',
            message: data.message || 'Please note: Data fetched here was generated in IFAMS',
            onConfirm: () => navigate(-1),
          });
        }
      } catch (error) {
        setSnackbar({
          open: true,
          message: error.message || 'Error while checking SC10 SFTP data.',
          severity: 'error',
        });
      }
    };

    checkSC10SftpData();
  

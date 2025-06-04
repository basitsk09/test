url- https://bsuat.info.sbi/BS/IFAMSS/SC10SFTP
payload:
{
    "circleCode": "001",
    "qed": "31/03/2025",
    "reportID": "310010",
    "reportName": "SC10",
    "reportStatus": "A"
}

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
	
/////////////////////////////////////////////////////////

 if (data.fileAndDataStatus === 1) {
                            console.log("Inside fileAndDataStatus ::: File Existed & SFTP Successfully Inserted Data");
                            console.log("SFTP Success Loading Data on Screen ::fileAndDataStatus:1::");

                            console.log("Calling SFTPDataSet Fn.")

                            // Setting Data for Bean -- FOR GETTING DATA FROM TEXT
                            sc10.SFTPDataSet(data);

                            console.log("Disbaling the Input Firelds");
                            // Disable Tables Cells
                            $('#example1 :input').prop("disabled", true);
                            //sc10.btnDisable();

                        }

                        // Error Or File Mismatch IN Files SFTP
                        else if (data.fileAndDataStatus === 2) {

                            // Show the Error Model & Redirected back to worklist
                            console.log("Error While SFTP Data- Inside Else Part ! with Error Message Model ::Status:2::");
                            sc10.displayModel(data);
                            $('#example1 :input').prop("disabled", true);
                            sc10.btnDisable();
                        }


                        // No Files Existed  or Error Files Received but data already inserted in DB
                        else {
//                          alert("Data alredy inserted calling regular data load function")
                            console.log("No Files Existed  or Error Files Received but data already inserted in DB ::Status:3::");
                            sc10.displayModel(data);
                            sc10.getSC10DataRegular(userInfo);

                            // DISABLE ALL THE FIELDS OR CELLS HERE
                            $('#example1 :input').prop("disabled", true);
                            console.log(circleCode + " does not exist in the list.");

                        }

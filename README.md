 @RequestMapping(value = "/bulkUpload", method = RequestMethod.POST)
    public ModelAndView submit(HttpServletRequest request, @ModelAttribute("command") FRTMaker report, BindingResult result) {

        DateTimeFormatter dtf = DateTimeFormatter.ofPattern("yyyy/MM/dd HH:mm:ss");
        LocalDateTime now1 = LocalDateTime.now();
        log.info(dtf.format(now1));
//        log.info(report.getBranchcode());
//        log.info(report.getStatus());

        report.setBranchCodeList(request.getParameterValues("branchcode"));
        report.setStatusList(request.getParameterValues("status"));

        //List<BranchMaster> validRecordsa = new ArrayList<BranchMaster>();
        List<FRTMaker> validRecords = new ArrayList<FRTMaker>();
        List<ExcelCol> inValidRecords = new ArrayList<ExcelCol>();

        HashMap<String, String> branchCodes = frtMakerService.getBranchlist(report);
        //List<BranchMaster> branchCodes = frtMakerService.getBranchlist();
        //List<BranchMaster> branchCodes = frtMakerService.getBranchlist(report.getBranchcode());
        //List<BranchAuditRequest> requestPending = frtMakerService.getBranchRequestPendingList(request);
        HashMap<String, String> requestPending = frtMakerService.getBranchRequestPendingList(request);


        for (int i = 0; i < report.getBranchCodeList().length; i++) {
            // Checks if branch master have the the branchno
            if (branchCodes.get(report.getBranchCodeList()[i]) != null) {
                // Checks if branch master have same or different auditable status

//                log.info("Database values : " + branchCodes.get(report.getBranchCodeList()[i]));
//                log.info("Database values type : " + branchCodes.get(report.getBranchCodeList()[i]).getClass());
//                log.info("UI values : " + report.getStatusList()[i]);
//                log.info("UI values type : " + report.getStatusList()[i].getClass());

                if (branchCodes.get(report.getBranchCodeList()[i]).equalsIgnoreCase(report.getStatusList()[i])) {
//                    log.info("inside BM condition");
                    inValidRecords.add(new ExcelCol(report.getBranchCodeList()[i], report.getStatusList()[i], "Invalid Request : Same status code as previous"));
                } else if (requestPending.get(report.getBranchCodeList()[i]) != null) {
//                    log.info("inside Pending state condition");
                    inValidRecords.add(new ExcelCol(report.getBranchCodeList()[i], report.getStatusList()[i], "Request is already in pending state"));
                } else {
//                    log.info("hurreeyy!!! valid condition");
                    validRecords.add(new FRTMaker(report.getBranchCodeList()[i], report.getStatusList()[i]));
                }
            } else {
//                log.info("inside brnach does not condition");
                inValidRecords.add(new ExcelCol(report.getBranchCodeList()[i], report.getStatusList()[i], "Branch code does not exist"));
            }

        }

        frtMakerService.insertData(report, validRecords);

        LocalDateTime now2 = LocalDateTime.now();
        log.info(dtf.format(now2));

        if (inValidRecords.isEmpty()) {
            ModelAndView view = new ModelAndView("FRTUser/FRTMultipleBranch");
            view.addObject("displayMessage", "Request Uploaded Successfully.");
            return view;
        } else {
            //https://www.codejava.net/frameworks/spring/spring-mvc-with-excel-view-example-apache-poi-and-jexcelapi
            return new ModelAndView("excelView", "listBooks", inValidRecords);
        }
    }
	
	
////////////////////////////////////////////////////////////////////////////////////


 @Override
    public HashMap<String, String> getBranchlist(FRTMaker report) {
        //System.out.println("##############branchdeatsils : " + report.getQuaterEndDate());

        //https://www.baeldung.com/java-string-with-separator-to-list
//        List<String> expectedCountriesList = Arrays.asList(report.getBranchcode().split(",", -1)); //Arrays.asList(report.getBranchcode());
//        log.info(expectedCountriesList);
//        log.info(expectedCountriesList.size());

        String query="select BRANCHNO,  " +
                "case when CRS_AUDITABLE ='Y' then (select case when ifcofr_audit_flag='Y' then 'I' else 'A' end " +
                "from crs_ifcofr_audit where ifcofr_branch=branchno and IFCOFR_DATE = to_date('" + report.getQuaterEndDate() + "','dd/mm/yyyy') ) " +
                "else CRS_AUDITABLE end CRS_AUDITABLE " +
                "from BRANCH_MASTER " +
                //"WHERE BRANCHNO IN (" + report.getBranchcode() + ") " +
                "order by BRANCHNO" ;

        HashMap<String, String> result=jdbcTemplateObject.query(query, new Object[]{}, new ResultSetExtractor<HashMap<String, String>>() {
            @Override
            public HashMap<String, String> extractData(ResultSet resultSet) throws SQLException, DataAccessException {
                HashMap<String, String> list = new HashMap<>();
                while(resultSet.next()){
                    list.put(resultSet.getString("BRANCHNO"), resultSet.getString("CRS_AUDITABLE"));
                }
                return  list;
            }
        });

        return result;
    }
////////////////////////////////////////////////////////////////////////////////////////////////




 @Override
    public HashMap<String, String> getBranchRequestPendingList(HttpServletRequest request) {
        //JdbcTemplate jdbcTemplateObj=new JdbcTemplate(dataSource);
        //System.out.println("##############branchdeatsils");
        //HttpServletRequest request;
        HttpSession session = request.getSession();
        String query = "SELECT AS_RT_ID, AS_BRANCH "
                + "FROM CRS_AUDIT_STATUS "
                + "WHERE AS_REQ_STATUS = 'P' "
                + "AND AS_QED = to_date('" + session.getAttribute(CommonConstants.QUARTER_END_DATE) + "','dd/mm/yyyy')";

        HashMap<String, String> result=jdbcTemplateObject.query(query, new Object[]{}, new ResultSetExtractor<HashMap<String, String>>() {
            @Override
            public HashMap<String, String> extractData(ResultSet resultSet) throws SQLException, DataAccessException {
                HashMap<String, String> list = new HashMap<>();
                while(resultSet.next()){
                    list.put(resultSet.getString("AS_BRANCH"), resultSet.getString("AS_RT_ID"));
                }
                return  list;
            }
        });

        return result;
    }
	
//////////////////////////////////////////////////////////////////////////////////////////////////



@Override
	public String insertData(FRTMaker report, List<FRTMaker> records) {
		//frtMakerDao.insertMultipleRecords(records);
		int generatedID = frtMakerDao.insertSingleRecord(report);

		frtMakerDao.insertMultipleRecords(records, generatedID, report);

		return null;
	}
	
////////////////////////////////////////////////////////////////////////////////////////////////////


  private final String SQL_INSERT = "INSERT INTO crs_request_track(RT_MAKER,RT_STATUS,RT_TYPE,RT_SUBTYPE,RT_QED,RT_BRANCH) values(?,?,?,?,to_date(?,'dd/mm/yyyy'),?)";

    //https://roytuts.com/single-and-multiple-records-insert-using-spring-jdbctemplate/
    public int insertSingleRecord(FRTMaker report) {    //public void insertSingleRecord(FRTMaker frtMaker)
        // Create GeneratedKeyHolder object
        GeneratedKeyHolder generatedKeyHolder = new GeneratedKeyHolder();

        // To insert data, you need to pre-compile the SQL and set up the data yourself.
        jdbcTemplateObject.update(conn -> {

            //https://stackoverflow.com/questions/1915166/how-to-get-the-insert-id-in-jdbc
            String generatedColumns[] = { "RT_ID" };

            // Pre-compiling SQL
            //PreparedStatement preparedStatement = conn.prepareStatement(SQL_INSERT, Statement.RETURN_GENERATED_KEYS);
            PreparedStatement preparedStatement = conn.prepareStatement(SQL_INSERT, generatedColumns);

            // Set parameters
            preparedStatement.setString(1, report.getMakerid());
            preparedStatement.setString(2, report.getRtStatus());
            preparedStatement.setString(3, report.getRtType());
            preparedStatement.setString(4, report.getRtSubType());
            preparedStatement.setString(5, report.getQuaterEndDate());
            preparedStatement.setString(6, report.getCureentBranchcode());

            return preparedStatement;

        }, generatedKeyHolder);

        final String idd = generatedKeyHolder.getKey().toString();
        int id = generatedKeyHolder.getKey().intValue();
        return id;
    }

    @Override
    public void insertMultipleRecords(List<FRTMaker> records, int generatedID, FRTMaker report) {

        String SQL_INSERT_MULTIPLE = "INSERT INTO CRS_AUDIT_STATUS(AS_RT_ID,AS_QED,AS_BRANCH,AS_NEW_STATUS,AS_REQ_STATUS) values(?,to_date(?,'dd/mm/yyyy'),?,?,'P')";

        jdbcTemplateObject.batchUpdate(SQL_INSERT_MULTIPLE, new BatchPreparedStatementSetter() {

            @Override
            public void setValues(PreparedStatement pStmt, int j) throws SQLException {
                FRTMaker frtMaker = records.get(j);
                pStmt.setInt(1, generatedID);
                pStmt.setString(2, report.getQuaterEndDate());
                //pStmt.setDate(2, to_date(report.getQuaterEndDate(),'dd/mm/yyyy'));
                pStmt.setString(3, frtMaker.getBranchcode());
                pStmt.setString(4, frtMaker.getStatus());
            }

            @Override
            public int getBatchSize() {
                return records.size();
            }

        });
    }
	
	
///////////////////////////////////////////////////////////////////////////////////////////////////


 const handleSubmit = async () => {
    setLoading(true);
    const hasErrors = rows.some(
      (row) => row.branchCode.error || row.auditStatus.error
    );

    if (hasErrors) {
      setSnackbar({
        children: "Please correct all errors before submitting.",
        severity: "error",
      });
      setLoading(false);
      return;
    }

    if (rows.length === 0) {
      setSnackbar({
        children: "There is no data to submit.",
        severity: "warning",
      });
      setLoading(false);
      return;
    }

    // The backend expects separate arrays for branchcode and status
    const branchCodeList = rows.map((row) =>
      String(row.branchCode.value).padStart(5, "0")
    );
    const statusList = rows.map((row) => row.auditStatus.value);

    // The JSP creates a form and submits it. We can replicate this with URLSearchParams for a standard form POST.
    const params = new URLSearchParams();
    branchCodeList.forEach((bc) => params.append("branchcode", bc));
    statusList.forEach((st) => params.append("status", st));

    try {
      // We expect a file download (for errors) or a redirect with a message (for success).
      // A standard form POST is the best way to handle this ambiguity without complex response parsing.
      const form = document.createElement("form");
      form.method = "post";
      form.action = "/FRTUser/bulkUpload"; // Your actual backend endpoint

      branchCodeList.forEach((bc) => {
        const hiddenField = document.createElement("input");
        hiddenField.type = "hidden";
        hiddenField.name = "branchcode";
        hiddenField.value = bc;
        form.appendChild(hiddenField);
      });

      statusList.forEach((st) => {
        const hiddenField = document.createElement("input");
        hiddenField.type = "hidden";
        hiddenField.name = "status";
        hiddenField.value = st;
        form.appendChild(hiddenField);
      });

      document.body.appendChild(form);
      form.submit();

      // After submission, we can only provide generic feedback as we don't get a direct response in the SPA.
      setSnackbar({
        children:
          "Request submitted successfully. You will be notified of any invalid records.",
        severity: "success",
      });
      // Optionally reset the page after a delay
      setTimeout(() => {
        setRows([]);
        setShowTable(false);
        setSelected([]);
      }, 3000);
    } catch (error) {
      console.error("Submission failed:", error);
      setSnackbar({
        children: "An error occurred during submission.",
        severity: "error",
      });
    } finally {
      setLoading(false);
    }
  };

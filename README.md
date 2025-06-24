public String submitAndGetDisplayMessage(String reportMasterId, String reportName, RW04 report,
                                             HttpServletRequest request, HttpSession session,String action) {
        SessionBean sessionBean = new SessionBean(session);

        String opt = (String) session.getAttribute(CommonConstants.OPT);
        String branchCode = session.getAttribute(CommonConstants.BRANCH_CODE).toString();
        String quarter_end_date = session.getAttribute(CommonConstants.QUARTER_END_DATE).toString();
        String financial_year = session.getAttribute(CommonConstants.YEAR).toString();
        String quarter = session.getAttribute(CommonConstants.QUARTER).toString();
        String user_Id = session.getAttribute(CommonConstants.USER_ID).toString();
        String circleCode = session.getAttribute(CommonConstants.CIRCLE_CODE).toString();
        String regionCode = session.getAttribute(CommonConstants.REGION_CODE).toString();
        String auditable = session.getAttribute(CommonConstants.BRANCH_AUDITABLE).toString();

      log.info("action"+action);
        ArrayList<RW04> anxXAUploadReportBeanList = new ArrayList<RW04>();

        String particularsList[] = null;
        particularsList = request.getParameterValues("particularsList");
        String provAmt2015List[] = request.getParameterValues("provAmt2015List");
        String writeOffDur12monList[] = request.getParameterValues("writeOffDur12monList");
        String additionDur12monList[] = request.getParameterValues("additionDur12monList");
        String reduInProviAmtList[] = request.getParameterValues("reduInProviAmtList");
        String proviAmt2016List[] = request.getParameterValues("proviAmt2016List");
        String ratePOfProvList[] = request.getParameterValues("ratePOfProvList");
        String provReqList[] = request.getParameterValues("provReqList");
        String requestType = request.getParameter("updateFlag");


        String firstRow = request.getParameter("firstRow");
        //log.info("name of the first row : " + firstRow);
        String dataFlag = report.getDataFlag();
        log.info("dataFlag" + dataFlag);
        String nilFlag = report.getNilFlag();
        log.info("nilFlag"+nilFlag);
        String reportId = report.getReportId();
        String generatedSequence = reportId;
        log.info("generatedSequence" + generatedSequence);


        int status = -1;

        setBeanList(anxXAUploadReportBeanList, particularsList, provAmt2015List, writeOffDur12monList,
                additionDur12monList, reduInProviAmtList, proviAmt2016List, ratePOfProvList, provReqList);


        String displayMessage = "";
        boolean isRmlUpdated = false;
        if(CommonConstants.MARK_AS_NIL.equalsIgnoreCase(action)) {
            log.info("in MarkedAsNil " + report.getAction());
            nilFlag = report.getNilFlag();
            log.info("nilFlag"+nilFlag);
            boolean isDataDeleted = makerService.getListOfTablesToReportMasterId(report.getReportId(), report.getReportMasterId());
            log.info("data deleted" + isDataDeleted);
            if (isDataDeleted || dataFlag.equalsIgnoreCase("I")) {
                isRmlUpdated = updateRmlTrack(reportMasterId, reportName, sessionBean, reportId, CommonConstants.STATUS_20_REPORT_CREATED, nilFlag);
                if(isRmlUpdated) {
                    displayMessage = "Report " + reportName + " submitted as Nil report ";
                }
            }
        }
        else {

            if (dataFlag.equalsIgnoreCase("U")) {

                displayMessage = updateData(sessionBean, opt, reportMasterId, reportName, report, request, branchCode, quarter_end_date,
                        financial_year, quarter, user_Id, circleCode, regionCode, auditable, anxXAUploadReportBeanList,
                        requestType, action);
            } else {
                displayMessage = insertData(sessionBean, opt, reportMasterId, reportName, report, request, branchCode, quarter_end_date,
                        financial_year, quarter, user_Id, circleCode, regionCode, auditable, anxXAUploadReportBeanList,
                        requestType, status, action);
            }
        }

        return displayMessage;
    }

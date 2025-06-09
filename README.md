{
//        log.info("LoginController > doLogin  >>>>> " + request.getAttribute("data"));

        UserLogin user = null;

        String IV = AESGCM256.generateBase64IV();
        String SALT = AESGCM256.generateBase64Salt();

        // FOR NEW FE Changes
        String userId = userLogin.getUserId();

        // OLD FE Handling
//        String userId = CommonFunctions.getDcrypted(userLogin.getUserId());

        // SCR 2024-25 :Trust Boundary Violation (Added the BE Validation for UserId)
        if (!isValidUserId(userId)) {
            log.warn("Invalid user ID attempt: " + userId);
            UserLogin login = new UserLogin();
            login.setIsUserExist("-1");

            // Sending the Empty Data
            ObjectMapper mapper = new ObjectMapper();
            String JsonString = mapper.writeValueAsString(login);

            String decryptedData = AESGCM256.encrypt(
                    JsonString,
                    IV,
                    SALT
            );

            JSONObject jObj = new JSONObject();
            jObj.put("iv", IV);
            jObj.put("salt", SALT);
            jObj.put("user", decryptedData);
            return jObj;
        }

        userLogin.setUserId(userId);
        HttpSession session=request.getSession();

        session.setAttribute(CommonConstant.USER_ID, userId);
        session.setAttribute(CommonConstant.USER_SESSION_ID, session.getId() + "-" + session.getCreationTime());
        session.setAttribute(CommonConstant.USER_SESSION_ID, session.getId());




        user = loginService.doLogin(userLogin);
        String quarterFYearDate = loginService.getQuarterYear();
        String QFD[] = quarterFYearDate.split("~");
        String quarter = QFD[0];
        String financial_year = QFD[1];
        String quarter_end_date = QFD[2];
        String previousYearEndDate = QFD[3];
        String previousQuarterEndDate = QFD[4];
        user.setQuarterEndDate(quarter_end_date);
        user.setPreviousQuarterEndDate(previousQuarterEndDate);
        user.setPreviousYearEndDate(previousYearEndDate);
        user.setFinancialYear(financial_year);
        user.setQuarter(quarter);

        // Adding the Parameter to token for checking isCircle Authorized to SFTP Data
        boolean output=ifamsSftpService.getCirclesList(user.getCircleCode());
        log.info("Is Circle Exits for SFTP ::"+output);

        user.setIsCircleExist(String.valueOf(output));
        log.info("User getCircleExits :"+user.getIsCircleExist());




        if (!(("-1").equalsIgnoreCase(user.getIsUserExist()) || ("P").equalsIgnoreCase(user.getStatus()))) {
            user = loginService.getadditionalDetails(user);
            String token = CommonFunctions.getToken(user);
            session.setAttribute("TOKEN", token);
            int updated = loginService.saveToken(user, token);
            user.setToken(token);
            log.info("User save token :"+user.getToken());
        }



        //starting encryption
        user.setUserId(userId);
        user.setUserName(user.getUserName());
        user.setCircleCode(user.getCircleCode());
        user.setCircleName(user.getCircleName());
        user.setRole(user.getRole());
        user.setCapacity(user.getCapacity());
        if(!("P").equalsIgnoreCase(user.getStatus())){
        user.setStatus(user.getStatus());
        }
        user.setIsBranchFinal(user.getIsBranchFinal());
        user.setIsCircleFreeze(user.getIsCircleFreeze());
        user.setIsAuditorDig(user.getIsAuditorDig());
        user.setIsCheckerDig(user.getIsCheckerDig());
        user.setFrRMId("444444");
        user.setFrReportId("444444");
        user.setMocFlag(user.getMocFlag());
        user.setIsCircleExist(user.getIsCircleExist());

        ObjectMapper mapper = new ObjectMapper();
        String JsonString = mapper.writeValueAsString(user);

        String decryptedData = AESGCM256.encrypt(
                JsonString,
                IV,
                SALT
        );


        log.info("decryptedData::"+decryptedData);

        JSONObject jObj = new JSONObject();
        jObj.put("iv", IV);
        jObj.put("salt", SALT);
        jObj.put("user", decryptedData);
        return jObj;

    }
	/////////////////////
	
	  public int saveToken(UserLogin userLogin, String token) {
        int count = 0;
        try {

            String query = "update bs_user set USER_SESSION=? ,USER_LAST_LOGIN=sysdate where user_id = ? ";
            count =  jdbcTemplate.update(query, new Object[]{token, userLogin.getUserId()});
        }catch(Exception e) {
            e.printStackTrace();
        }
        return  count;
    }

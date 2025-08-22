package com.crs.commonReportsService.services;

import com.crs.commonReportsService.models.*;
import com.crs.commonReportsService.repositories.*;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

import java.sql.Timestamp;
import java.text.ParseException;
import java.text.SimpleDateFormat;
import java.time.LocalDate;
import java.time.Month;
import java.time.YearMonth;
import java.time.format.DateTimeFormatter;
import java.util.*;
import java.util.concurrent.atomic.AtomicInteger;
import java.util.logging.Logger;

@RestController
@RequestMapping("/RW01")
public class Rw01ServiceImpl implements Rw01Service {

    public static final String NO_DISPLAY = "noDisplay";
    public static final String D = "0_D";
    @Autowired
    Rw01Repository rw01Repository;
    @Autowired
    ColumnDataRepository columnDataRepository;
    @Autowired
    Rw01BRepository rw01BRepository;
    @Autowired
    Rw01CRepository rw01CRepository;
    @Autowired
    Rw01DRepository rw01DRepository;
    @Autowired
    Rw01ERepository rw01ERepository;
    @Autowired
    Rw01FRepository rw01FRepository;

    /* @Autowired
     CrsPriorPeriodRepository crsPriorPeriodRepository;*/

    static Logger log = Logger.getLogger(Rw01ServiceImpl.class.getName());

    // function for the format date format
    public Timestamp getQedFormatter(String quarterEndDate) {
        try {
            SimpleDateFormat dateFormat = new SimpleDateFormat(
                    "dd/MM/yyyy"
            );
            Date parsedDate = dateFormat.parse(quarterEndDate);
            Timestamp timestamp = new Timestamp(parsedDate.getTime());
            Timestamp finalTimestamp = timestamp;
            return finalTimestamp;
        } catch (ParseException e) {
            throw new RuntimeException(e);
        }
    }

    public ResponseEntity<String> getData(Map<String, Object> map) {
        Map<String, String> loginUserData = (Map<String, String>) map.get("user");
        log.info("token data - " + loginUserData);

        Map<String, String> data = (Map<String, String>) map.get("data");
        log.info("user  data map details-" + data);

        ResponseVO<Map<String, Object>> responseVO = new ResponseVO<>();
        String previousDate = loginUserData.get("quarterEndDate").substring(6);
        int last = Integer.parseInt(previousDate);


        int secondlast = last - 1;
        String previousYearEndDate = loginUserData.get("quarterEndDate").substring(0, 6) + secondlast; // "30/06/2016"; //
        log.info("previousYearEndDate-" + previousYearEndDate);

        Timestamp prevquarterEndDate = getQedFormatter(previousYearEndDate);
        log.info("prevquarterEndDate:-" + prevquarterEndDate);


        String quarter = loginUserData.get("quarterEndDate").substring(3, 5);
        String quarteryear = loginUserData.get("quarterEndDate").substring(6, 10);
        log.info("quarter:-" + quarter);

        Timestamp quarterEndDate = getQedFormatter(loginUserData.get("quarterEndDate"));
        log.info("quarterEndDate:-" + quarterEndDate);

        ////

        try {
            Map<String, Object> finalMap = new LinkedHashMap<>();
            List<Map<String, Object>> tabList = new ArrayList<>();

            // Get the Particular Report "Tab Data using Report-ID
            log.info("reportId" + data.get("reportId"));
            List<Map<String, Object>> tabData = rw01Repository.getTabData(data.get("reportId"));
            log.info("tabData.size()" + tabData.size());

            // Loop for assigning Row Data & Column Data to Tab Data

            for (int i = 0; i < tabData.size(); i++) {
                log.info("TabData >>>>" + tabData.get(i).get("TAB_NAME") + "  " + tabData.get(i).get("TAB_VALUE"));
                Map<String, Object> updatedTabData = new HashMap<>();

                // Copying the Tab Data to New Map
                updatedTabData.putAll(tabData.get(i));
                log.info("updatedTabData-" + updatedTabData);

                // Adding COLUMN-DATA
                //String appendValue = "30.06.2023 RS.  Ps.";
                String appendValue = previousYearEndDate + " RS.  Ps.";

                log.info("appendValue-" + appendValue);
                log.info("quarter-" + quarter);
                log.info("getPreviousQuater-" + getQuaterEndMonthYear());
                log.info("quarteryear-" + quarteryear);
                log.info("data.get(\"reportId\")-" + data.get("reportId"));
                log.info("(String) tabData.get(i).get(\"TAB_VALUE\")" + (String) tabData.get(i).get("TAB_VALUE"));

                String quarterMonth = "";
                String qedStringFormat = "";
                String previousFinancialEnd = "";
                if (quarter.equalsIgnoreCase("03")) {
                    quarterMonth = "TWELVE MONTHS";
                    qedStringFormat = "31st March " + previousDate;
                    previousFinancialEnd = Integer.toString(secondlast);
                    //previousFinancialEnd = previousDate;
                    log.info("qedStringFormat >>>>> " + qedStringFormat);
                    log.info("previousFinancialEnd >>>>> " + previousFinancialEnd);
                } else if (quarter.equalsIgnoreCase("06")) {
                    quarterMonth = "QUARTER";
                    qedStringFormat = "30th June " + previousDate;
                    previousFinancialEnd = previousDate;
                } else if (quarter.equalsIgnoreCase("09")) {
                    quarterMonth = "HALF YEAR";
                    qedStringFormat = "30th September " + previousDate;
                    previousFinancialEnd = previousDate;
                } else if (quarter.equalsIgnoreCase("12")) {
                    quarterMonth = "NINE MONTHS";
                    qedStringFormat = "31st September " + previousDate;
                    previousFinancialEnd = previousDate;
                }

                updatedTabData.put(
                        "TAB_COLUMN_DATA",
                        rw01Repository.findColumnData(
                                appendValue,
                                getQuaterEndMonthYear(),
                                quarter,
                                quarteryear,
                                data.get("reportId"),
                                (String) tabData.get(i).get("TAB_VALUE"),
                                loginUserData.get("quarterEndDate"),
                                quarterMonth,
                                qedStringFormat,
                                previousFinancialEnd
                        )
                );

                log.info("FFFFFF-" + tabData.get(i).get("TAB_VALUE").toString());
                // Adding the ROW-DATA for 7 tabs
                if (tabData.get(i).get("TAB_VALUE").toString().equalsIgnoreCase("1")) {

                    log.info("tab 1");

                    log.info("tab 1 quarter >>> " + quarter);
                    log.info("tab 1 submissionId >>>> " + String.valueOf(data.get("submissionId")));
                    log.info("tab 1 prevquarterEndDate >>> " + prevquarterEndDate);
                    log.info("tab 1 branch_code >>> " + loginUserData.get("branch_code"));


                    LinkedList<List<String>> rowList = rw01Repository.getrowdatalist(quarter, String.valueOf(data.get("submissionId")));
                    log.info("tab 1 rowList " + rowList.size());
                    updatedTabData.put("IS_SAVED", rowList.size());
                    if (rowList.size() == 0) {

                        // first entry

                        rowList = rw01Repository.getrowdatalistOnIndex(quarter, prevquarterEndDate, loginUserData.get("branch_code"));
                        updatedTabData.put("IS_SAVED", rowList.size());
                        log.info("in if tab 1  " + rowList.size());
                        LinkedList<List<String>> getrowdatalist = new LinkedList<>();
                        int j = 1;
                        if (rowList.size() == 0) {

                            // no data in previous year

                            //for first 4
                            Set<Integer> quarter060D = Set.of(15, 16, 17, 18, 21, 23, 24, 25, 26, 27, 28, 29, 30, 31, 32, 33, 34); //24, 40, 53
                            Set<Integer> quarter06NoDisplay = Set.of(19, 37); //15, 35
                            Set<Integer> quarter090D = Set.of(29, 30, 31, 32, 33, 34);
                            Set<Integer> quarter09NoDisplay = Set.of(19, 37);
                            Set<Integer> quarter120D = Set.of(32, 33, 34);
                            Set<Integer> quarter03NoDisplay = Set.of(19, 37);

                            for (int r = 15; r < 56; r++) {
                                if (r == 20 || r == 21) continue;

                                log.info("r:" + r);
                                List<String> l1 = new ArrayList<>(Collections.nCopies(8, "")); // Initialize list with 8 empty strings

                                String valueForIndex4 = "0";

                                switch (quarter) {
                                    case "06":
                                        log.info("tab 331");
                                        if (quarter060D.contains(r)) {
                                            log.info("here in quarter 6");
                                            valueForIndex4 = D;
                                        } else if (quarter06NoDisplay.contains(r)) {
                                            log.info("here in No display condition");
                                            l1.set(0, NO_DISPLAY);
                                            valueForIndex4 = NO_DISPLAY;
                                        } else if (r == 22) {
                                            l1.set(0, "Outstanding in Rs.      Ps._heading");
                                            valueForIndex4 = "Outstanding in Rs.      Ps._heading";
                                        }
                                        break;

                                    case "09":
                                        log.info("tab 243");
                                        if (quarter090D.contains(r)) {
                                            valueForIndex4 = D;
                                        } else if (quarter09NoDisplay.contains(r)) {
                                            log.info("r = 6");
                                            l1.set(0, NO_DISPLAY);
                                            valueForIndex4 = NO_DISPLAY;
                                        } else if (r == 22) {
                                            l1.set(0, "Outstanding in Rs.      Ps._heading");
                                            valueForIndex4 = "Outstanding in Rs.      Ps._heading";
                                        }
                                        break;

                                    case "12":
                                        log.info("tab 348");
                                        if (quarter120D.contains(r)) {
                                            valueForIndex4 = D;
                                        }
                                        break;

                                    case "03":
                                        log.info("tab 243");
                                        if (quarter03NoDisplay.contains(r)) {
                                            log.info("r = 6");
                                            l1.set(0, NO_DISPLAY);
                                            valueForIndex4 = NO_DISPLAY;
                                        } else if (r == 22) {
                                            l1.set(0, "Outstanding in Rs.      Ps._heading");
                                            valueForIndex4 = "Outstanding in Rs.      Ps._heading";
                                        }
                                        break;
                                    default:
                                        throw new IllegalStateException("Unexpected value: " + quarter);
                                }

                                l1.set(4, valueForIndex4);
                                l1.set(5, String.valueOf(j));
                                l1.set(6, (r == 37) ? "0" : (r > 37 ? String.valueOf(r - 1) : String.valueOf(r)));
                                l1.set(7, "false");

                                getrowdatalist.add(l1);
                                log.info("getrowdatalist size:-" + getrowdatalist.size());
                                j++;
                            }

                            log.info("getrowdatalist:- " + getrowdatalist);

                            // updatedTabData.put("TAB_ROW_DATA", rw03Repository.getrowdatalist(data.get("submissionId")));
                            updatedTabData.put("TAB_ROW_DATA", getrowdatalist);

                        }
                        else {

                            log.info("getrowdatalist else:- " + rowList.size());


                            List<String> l1 = new ArrayList<>();
                            l1.add(0, NO_DISPLAY);
                            l1.add(1, "");
                            l1.add(2, "");
                            l1.add(3, "");
                            l1.add(4, NO_DISPLAY);
                            l1.add(5, "5");
                            l1.add(6, "19");
                            l1.add(7, "false");


                            ////////////////////////////////////////////

                            List<String> lJuly = new ArrayList<>();
                            lJuly.add(0, D);
                            lJuly.add(1, "");
                            lJuly.add(2, "");
                            lJuly.add(3, "");
                            lJuly.add(4, D);
                            lJuly.add(5, "12");
                            lJuly.add(6, "26");
                            lJuly.add(7, "false");


                            List<String> lAug = new ArrayList<>();
                            lAug.add(0, D);
                            lAug.add(1, "");
                            lAug.add(2, "");
                            lAug.add(3, "");
                            lAug.add(4, D);
                            lAug.add(5, "13");
                            lAug.add(6, "27");
                            lAug.add(7, "false");


                            List<String> lSep = new ArrayList<>();
                            lSep.add(0, D);
                            lSep.add(1, "");
                            lSep.add(2, "");
                            lSep.add(3, "");
                            lSep.add(4, D);
                            lSep.add(5, "14");
                            lSep.add(6, "28");
                            lSep.add(7, "false");


                            List<String> lOct = new ArrayList<>();
                            lOct.add(0, D);
                            lOct.add(1, "");
                            lOct.add(2, "");
                            lOct.add(3, "");
                            lOct.add(4, D);
                            lOct.add(5, "15");
                            lOct.add(6, "29");
                            lOct.add(7, "false");


                            List<String> lNov = new ArrayList<>();
                            lNov.add(0, D);
                            lNov.add(1, "");
                            lNov.add(2, "");
                            lNov.add(3, "");
                            lNov.add(4, D);
                            lNov.add(5, "16");
                            lNov.add(6, "30");
                            lNov.add(7, "false");


                            List<String> lDec = new ArrayList<>();
                            lDec.add(0, D);
                            lDec.add(1, "");
                            lDec.add(2, "");
                            lDec.add(3, "");
                            lDec.add(4, D);
                            lDec.add(5, "17");
                            lDec.add(6, "31");
                            lDec.add(7, "false");


                            List<String> lJan = new ArrayList<>();
                            lJan.add(0, D);
                            lJan.add(1, "");
                            lJan.add(2, "");
                            lJan.add(3, "");
                            lJan.add(4, D);
                            lJan.add(5, "18");
                            lJan.add(6, "32");
                            lJan.add(7, "false");


                            List<String> lFeb = new ArrayList<>();
                            lFeb.add(0, D);
                            lFeb.add(1, "");
                            lFeb.add(2, "");
                            lFeb.add(3, "");
                            lFeb.add(4, D);
                            lFeb.add(5, "19");
                            lFeb.add(6, "33");
                            lFeb.add(7, "false");


                            List<String> lMar = new ArrayList<>();
                            lMar.add(0, D);
                            lMar.add(1, "");
                            lMar.add(2, "");
                            lMar.add(3, "");
                            lMar.add(4, D);
                            lMar.add(5, "20");
                            lMar.add(6, "34");
                            lMar.add(7, "false");


                            ///////////////////////////////////////////////////

                            List<String> l2 = new ArrayList<>();
                            l2.add(0, NO_DISPLAY);
                            l2.add(1, "");
                            l2.add(2, "");
                            l2.add(3, "");
                            l2.add(4, NO_DISPLAY);
                            l2.add(5, "21");
                            l2.add(6, "0");
                            l2.add(7, "false");


                            rowList.add(4, l1);

                            rowList.add(9, lJuly);
                            rowList.add(10, lAug);
                            rowList.add(11, lSep);
                            rowList.add(12, lOct);
                            rowList.add(13, lNov);
                            rowList.add(14, lDec);
                            rowList.add(15, lJan);
                            rowList.add(16, lFeb);
                            rowList.add(17, lMar);

                            rowList.add(20, l2);
                            //rowList.add(["noDisplay","" , "", "", "noDisplay", 5, 19, "true"])


                            log.info("getrowdatalist else:- " + rowList.size());

                            updatedTabData.put("TAB_ROW_DATA", rowList);
                        }

                    }
                    else {
                        // Edit scope

                        log.info("getrowdatalist else:- " + rowList.size());

                        List<String> l1 = new ArrayList<>();
                        l1.add(0, NO_DISPLAY);
                        l1.add(1, "");
                        l1.add(2, "");
                        l1.add(3, "");
                        l1.add(4, NO_DISPLAY);
                        l1.add(5, "5");
                        l1.add(6, "19");
                        l1.add(7, "false");


                        ////////////////////////////////////////////

                        List<String> lJuly = new ArrayList<>();
                        lJuly.add(0, D);
                        lJuly.add(1, "");
                        lJuly.add(2, "");
                        lJuly.add(3, "");
                        lJuly.add(4, D);
                        lJuly.add(5, "12");
                        lJuly.add(6, "26");
                        lJuly.add(7, "false");


                        List<String> lAug = new ArrayList<>();
                        lAug.add(0, D);
                        lAug.add(1, "");
                        lAug.add(2, "");
                        lAug.add(3, "");
                        lAug.add(4, D);
                        lAug.add(5, "13");
                        lAug.add(6, "27");
                        lAug.add(7, "false");


                        List<String> lSep = new ArrayList<>();
                        lSep.add(0, D);
                        lSep.add(1, "");
                        lSep.add(2, "");
                        lSep.add(3, "");
                        lSep.add(4, D);
                        lSep.add(5, "14");
                        lSep.add(6, "28");
                        lSep.add(7, "false");


                        List<String> lOct = new ArrayList<>();
                        lOct.add(0, D);
                        lOct.add(1, "");
                        lOct.add(2, "");
                        lOct.add(3, "");
                        lOct.add(4, D);
                        lOct.add(5, "15");
                        lOct.add(6, "29");
                        lOct.add(7, "false");


                        List<String> lNov = new ArrayList<>();
                        lNov.add(0, D);
                        lNov.add(1, "");
                        lNov.add(2, "");
                        lNov.add(3, "");
                        lNov.add(4, D);
                        lNov.add(5, "16");
                        lNov.add(6, "30");
                        lNov.add(7, "false");


                        List<String> lDec = new ArrayList<>();
                        lDec.add(0, D);
                        lDec.add(1, "");
                        lDec.add(2, "");
                        lDec.add(3, "");
                        lDec.add(4, D);
                        lDec.add(5, "17");
                        lDec.add(6, "31");
                        lDec.add(7, "false");


                        List<String> lJan = new ArrayList<>();
                        lJan.add(0, D);
                        lJan.add(1, "");
                        lJan.add(2, "");
                        lJan.add(3, "");
                        lJan.add(4, D);
                        lJan.add(5, "18");
                        lJan.add(6, "32");
                        lJan.add(7, "false");


                        List<String> lFeb = new ArrayList<>();
                        lFeb.add(0, D);
                        lFeb.add(1, "");
                        lFeb.add(2, "");
                        lFeb.add(3, "");
                        lFeb.add(4, D);
                        lFeb.add(5, "19");
                        lFeb.add(6, "33");
                        lFeb.add(7, "false");


                        List<String> lMar = new ArrayList<>();
                        lMar.add(0, D);
                        lMar.add(1, "");
                        lMar.add(2, "");
                        lMar.add(3, "");
                        lMar.add(4, D);
                        lMar.add(5, "20");
                        lMar.add(6, "34");
                        lMar.add(7, "false");


                        /// ////////////////////////////////////////////////


                        List<String> l2 = new ArrayList<>();
                        l2.add(0, NO_DISPLAY);
                        l2.add(1, "");
                        l2.add(2, "");
                        l2.add(3, "");
                        l2.add(4, NO_DISPLAY);
                        l2.add(5, "21");
                        l2.add(6, "0");
                        l2.add(7, "false");


                        rowList.add(4, l1);

                        rowList.add(9, lJuly);
                        rowList.add(10, lAug);
                        rowList.add(11, lSep);
                        rowList.add(12, lOct);
                        rowList.add(13, lNov);
                        rowList.add(14, lDec);
                        rowList.add(15, lJan);
                        rowList.add(16, lFeb);
                        rowList.add(17, lMar);

                        rowList.add(20, l2);
                        //rowList.add(["noDisplay","" , "", "", "noDisplay", 5, 19, "true"])


                        log.info("getrowdatalist else:- " + rowList.size());
                        updatedTabData.put("TAB_ROW_DATA", rowList);

                    }
                    //Get tab1 Data Here

                    // System.exit(0);


                } else if (tabData.get(i).get("TAB_VALUE").toString().equalsIgnoreCase("2")) {
                    //Get tab2 Data Here
                    LinkedList<List<String>> getrowdatalist = rw01BRepository.getrowdatalist(String.valueOf(data.get("submissionId")));

                    updatedTabData.put("IS_SAVED", getrowdatalist.size());

                    if (getrowdatalist.size() == 0) {
                        for (int r = 1; r <= 37; r++) {       // for (int r = 1; r <= 38; r++) {
                            log.info("tab 2 r:" + r);

                            LinkedList<String> l1 = new LinkedList<>();
                            l1.add(0, " ");
                            l1.add(1, " ");
                            // 22 Break Up Of Outstandings In Sundry Deposits Account
                            // 30 Other Monies for which Bank is Contingently liable	
                            // 34 Staff Advances	


                            if (r == 1 || r == 5 || r == 22 || r == 30 || r == 34) {
                                // if (r == 1 || r == 5 || r == 23 || r == 31 || r == 35) {
                                log.info("r = " + r);
                                l1.add(2, NO_DISPLAY);
                                l1.set(0, NO_DISPLAY);

                            } else if (r == 4 || r == 2 || r == 3 || r == 6 || r == 30 || r == 34 || r == 35 || r == 36
                                    // else if (r == 4 || r == 2 || r == 3 || r == 6 || r == 21 || r == 30 || r == 34 || r == 36
                                    //} else if (r == 4 || r == 2 || r == 3 || r == 6 || r == 20 || r == 30 || r == 34 || r == 36
                                    || r == 37 || r == 38) {
                                log.info("r == " + r);
                                l1.add(2, D);
                            } else {
                                log.info("i ===" + i);
                                l1.add(2, "0");
                            }
                            l1.add(3, String.valueOf(r));
                            l1.add(4, String.valueOf(r));
                            l1.add(5, "false");
                            getrowdatalist.add(l1);
                        }

                        updatedTabData.put("TAB_ROW_DATA", getrowdatalist);

                    } else {

                        log.info("getrowdatalist else:- " + getrowdatalist.size());

                        // 4
                        log.info("getrowdatalist else:- " + getrowdatalist.size());
                        // Particulars of Staff Security Deposits received from Cashiers/POs
                        List<String> l1 = new ArrayList<>();
                        l1.add(0, NO_DISPLAY);
                        l1.add(1, "");
                        l1.add(2, NO_DISPLAY);
                        l1.add(3, "301");
                        l1.add(4, "301");
                        l1.add(5, "false");


                        // Break Up Of Oustanding In Suspense Account
                        // List<String> l2 = new ArrayList<>();
                        // l2.add(0, "");
                        // l2.add(1, "");
                        // l2.add(2, "0_D");
                        // l2.add(3, "302");
                        // l2.add(4, "302");
                        // l2.add(5, "false");

                        //Break Up Of Outstandings In Sundry Deposits Account
                        List<String> l3 = new ArrayList<>();
                        l3.add(0, NO_DISPLAY);
                        l3.add(1, "");
                        l3.add(2, NO_DISPLAY);
                        l3.add(3, "303");
                        l3.add(4, "303");
                        l3.add(5, "false");

                        //Other Monies for which Bank is Contingently liable
                        List<String> l4 = new ArrayList<>();
                        l4.add(0, NO_DISPLAY);
                        l4.add(1, "");
                        l4.add(2, NO_DISPLAY);
                        l4.add(3, "304");
                        l4.add(4, "304");
                        l4.add(5, "false");

                        //Staff Advances
                        List<String> l5 = new ArrayList<>();
                        l5.add(0, NO_DISPLAY);
                        l5.add(1, "");
                        l5.add(2, NO_DISPLAY);
                        l5.add(3, "305");
                        l5.add(4, "305");
                        l5.add(5, "false");

                        List<String> l6 = new ArrayList<>();
                        l6.add(0, NO_DISPLAY);
                        l6.add(1, "");
                        l6.add(2, NO_DISPLAY);
                        l6.add(3, "306");
                        l6.add(4, "306");
                        l6.add(5, "false");

                        getrowdatalist.add(0, l1);
                        // getrowdatalist.add(3, l2);   //4
                        getrowdatalist.add(4, l3);

                        getrowdatalist.add(21, l3);
                        getrowdatalist.add(29, l4);
                        getrowdatalist.add(33, l5);
                        getrowdatalist.add(37, l6);
                        //rowList.add(["noDisplay","" , "", "", "noDisplay", 5, 19, "true"])


                        log.info("getrowdatalist else:- " + getrowdatalist.size());

                        updatedTabData.put("TAB_ROW_DATA", getrowdatalist);
                    }


                } else if (tabData.get(i).get("TAB_VALUE").toString().equalsIgnoreCase("3")) {
                    //Get tab3 Data Here
                    LinkedList<List<String>> getrowdatalist = rw01CRepository.getrowdatalist(String.valueOf(data.get("submissionId")));
                    updatedTabData.put("IS_SAVED", getrowdatalist.size());
                    if (getrowdatalist.size() == 0) {
                        int j = 1;
                        for (char ch = 'A'; ch <= 'M'; ++ch) {


                            LinkedList<String> l1 = new LinkedList<>();
                            l1.add(0, " ");
                            l1.add(1, " ");
                            if (ch == 'C' || ch == 'H' || ch == 'L' || ch == 'M') {
                                l1.add(2, D);
                                l1.add(3, D);
                                l1.add(4, D);
                                l1.add(5, D);
                            } else if (ch == 'I') {
                                l1.add(2, NO_DISPLAY);
                                l1.add(3, NO_DISPLAY);
                                l1.add(4, NO_DISPLAY);
                                l1.add(5, NO_DISPLAY);
                            } else {
                                l1.add(2, D);
                                l1.add(3, D);
                                l1.add(4, D);
                                l1.add(5, D);
                            }

                            l1.add(6, String.valueOf(j));
                            l1.add(7, String.valueOf(ch));
                            l1.add(8, "false");
                            getrowdatalist.add(l1);
                            j++;
                        }

                        updatedTabData.put("TAB_ROW_DATA", getrowdatalist);
                    } else {
                        updatedTabData.put("TAB_ROW_DATA", getrowdatalist);
                    }

                } else if (tabData.get(i).get("TAB_VALUE").toString().equalsIgnoreCase("4")) {
                    LinkedList<List<String>> getrowdatalist = rw01DRepository.getrowdatalist(String.valueOf(data.get("submissionId")));
                    updatedTabData.put("IS_SAVED", getrowdatalist.size());
                    log.info("AAAAAAAAA");
                    if (getrowdatalist.size() == 0) {
                        log.info("BBBBBBBB");
                        for (int r = 1; r < 15; r++) {

                            LinkedList<String> l1 = new LinkedList<>();
                            l1.add(0, " ");
                            l1.add(1, " ");
                            l1.add(2, "0");
                            l1.add(3, String.valueOf(r));
                            l1.add(4, String.valueOf(r));
                            l1.add(5, "false");
                            getrowdatalist.add(l1);

                        }
                        updatedTabData.put("TAB_ROW_DATA", getrowdatalist);
                    } else {
                        log.info("CCCCCCCCCC");
                        updatedTabData.put("TAB_ROW_DATA", getrowdatalist);
                    }

                } else if (tabData.get(i).get("TAB_VALUE").toString().equalsIgnoreCase("5")) {
                    LinkedList<List<String>> getrowdatalist = rw01ERepository.getrowdatalist(String.valueOf(data.get("submissionId")));
                    updatedTabData.put("IS_SAVED", getrowdatalist.size());
                    if (getrowdatalist.size() == 0) {
                        for (int r = 1; r < 15; r++) {


                            LinkedList<String> l1 = new LinkedList<>();

                            if (r == 14) {
                                l1.add(0, " ");
                                l1.add(1, " ");
                                l1.add(2, D);
                                l1.add(3, D);
                                l1.add(4, String.valueOf(r));
                                l1.add(5, String.valueOf(206));
                                l1.add(6, "false");
                            } else {
                                l1.add(0, " ");
                                l1.add(1, " ");
                                l1.add(2, "0");
                                l1.add(3, "0");
                                l1.add(4, String.valueOf(r));
                                l1.add(5, String.valueOf(r));
                                l1.add(6, "false");
                            }


                            getrowdatalist.add(l1);

                        }
                        updatedTabData.put("TAB_ROW_DATA", getrowdatalist);
                    } else {
                        updatedTabData.put("TAB_ROW_DATA", getrowdatalist);
                    }

                } else if (tabData.get(i).get("TAB_VALUE").toString().equalsIgnoreCase("6")) {
                    LinkedList<List<String>> getrowdatalist = rw01FRepository.getrowdatalist(String.valueOf(data.get("submissionId")));
                    updatedTabData.put("IS_SAVED", getrowdatalist.size());
                    log.info("getrowdatalist 529" + getrowdatalist);

                    log.info("getrowdatalist 531" + getrowdatalist.size());
                    if (getrowdatalist.size() == 0) {
                        for (int r = 1; r < 6; r++) {

                            LinkedList<String> l1 = new LinkedList<>();
                            l1.add(0, " ");

                            if (r == 1) {
                                l1.add(1, "Partly paid investments (For Global Markets only)_heading");
                                l1.add(2, "0");
                            } else if (r == 2) {
                                l1.add(1, "Undrawn Commitments for Venture Capital Funds/ Alternate Investment Funds (For Global Markets only)_heading");
                                l1.add(2, "0");
                            } else if (r == 3 || r == 4) {
                                l1.add(1, " ");
                                l1.add(2, "0");
                            } else if (r == 5) {
                                l1.add(1, "TOTAL_heading");
                                l1.add(2, D);
                            } else {
                                log.info("sasgfsa" + r);
                                l1.add(2, "0");

                            }
                            l1.add(3, String.valueOf(r));
                            l1.add(4, String.valueOf(r));
                            l1.add(5, "false");
                            getrowdatalist.add(l1);

                        }
                        updatedTabData.put("TAB_ROW_DATA", getrowdatalist);
                    } else {
                        log.info("getrowdatalist " + getrowdatalist.get(0).get(2));
                        for (int r = 0; r < 5; r++) {

                            LinkedList<String> l1 = new LinkedList<>();
                            l1.add(0, " ");

                            if (r == 0) {
                                l1.add(1, "Partly paid investments (For Global Markets only)_heading");
                                l1.add(2, getrowdatalist.get(r).get(2));
                            } else if (r == 1) {
                                l1.add(1, "Undrawn Commitments for Venture Capital Funds/ Alternate Investment Funds (For Global Markets only)_heading");
                                l1.add(2, getrowdatalist.get(r).get(2));
                            } else if (r == 2) {
                                l1.add(1, getrowdatalist.get(r).get(1));
                                l1.add(2, getrowdatalist.get(r).get(2));
                            } else if (r == 3) {
                                l1.add(1, getrowdatalist.get(r).get(1));
                                l1.add(2, getrowdatalist.get(r).get(2));
                            } else if (r == 4) {
                                l1.add(1, "TOTAL_heading");
                                l1.add(2, getrowdatalist.get(r).get(2) + "_D");
                            } else {
                                log.info("sasgfsa" + r);
                                l1.add(2, getrowdatalist.get(r).get(2));

                            }
                            l1.add(3, String.valueOf(r + 1));
                            l1.add(4, String.valueOf(r + 1));
                            l1.add(5, "false");
                            getrowdatalist.set(r, l1);

                        }

                        updatedTabData.put("TAB_ROW_DATA", getrowdatalist);
                    }


                } else if (tabData.get(i).get("TAB_VALUE").toString().equalsIgnoreCase("7")) {
                    LinkedList<List<String>> getrowdatalist = rw01FRepository.getrowdatalist1(String.valueOf(data.get("submissionId")));
                    updatedTabData.put("IS_SAVED", getrowdatalist.size());
                    if (getrowdatalist.size() == 0) {
                        int j = 1;
                        for (int r = 6; r < 15; r++) {


                            LinkedList<String> l1 = new LinkedList<>();
                            l1.add(0, " ");
                            l1.add(1, " ");
                            if (r == 9 || r == 13 || r == 14) {
                                l1.add(2, D);
                            } else if (r == 6 || r == 10) {
                                l1.add(2, NO_DISPLAY);
                            } else {
                                l1.add(2, "0");
                            }

                            l1.add(3, String.valueOf(j));
                            l1.add(4, String.valueOf(r));
                            l1.add(5, "false");
                            getrowdatalist.add(l1);
                            j++;
                        }
                        updatedTabData.put("TAB_ROW_DATA", getrowdatalist);
                    } else {
                        updatedTabData.put("TAB_ROW_DATA", getrowdatalist);
                    }

                }
                tabList.add(updatedTabData);
            }
            // Finally Adding the Whole Tab Data to Map.
            finalMap.put("tabData", tabList);
            log.info("finalMap" + finalMap);


            responseVO.setResult(finalMap);
            responseVO.setMessage("Tab Data fetched successfully.");
            responseVO.setStatusCode(HttpStatus.OK.value());
            return new ResponseEntity(responseVO, HttpStatus.OK);
        } catch (Exception e) {
            log.info(" Exception Occurred: " + e.getCause());
            responseVO.setStatusCode(HttpStatus.INTERNAL_SERVER_ERROR.value());
            responseVO.setMessage("Something went wrong.");
            throw new RuntimeException(e);
        }
    }

    public static String getFinancialYear() {

        // Get the current date
        LocalDate currentDate = LocalDate.now();

        // Determine the financial year
        int year = currentDate.getYear();
        Month month = currentDate.getMonth();

        // Financial year typically starts in April and ends in March
        int financialYear;
        if (month.compareTo(Month.APRIL) < 0) {
            financialYear = year - 1;
        } else {
            financialYear = year;
        }


        return financialYear + "-" + (financialYear + 1);
    }

    public static String getQuater() {
        // Get the current date
        LocalDate currentDate = LocalDate.now();

        // Determine the financial year
        Month month = currentDate.getMonth();

        // Determine the financial quarter
        int quarter;
        if (month.compareTo(Month.JULY) < 0) {
            quarter = 1;
        } else if (month.compareTo(Month.OCTOBER) < 0) {
            quarter = 2;
        } else if (month.compareTo(Month.JANUARY) < 0) {
            quarter = 3;
        } else {
            quarter = 4;
        }


        return "Q" + quarter;
    }

    public static String getQuaterEndMonthYear() {
        // Get the current date
        LocalDate currentDate = LocalDate.now();

        // Determine the financial year
        Month month = currentDate.getMonth();
        log.info("Month >>> " + month.getValue());
        log.info("Month.JULY >>> " + Month.JULY);

        int m = month.getValue();

        if (m == 1 || m == 2 || m == 3) {
            return "DECEMBER " + (currentDate.getYear() - 1);
        } else if (m == 4 || m == 5 || m == 6) {
            return "MARCH " + currentDate.getYear();
        } else if (m == 7 || m == 8 || m == 9) {
            return "JUNE " + currentDate.getYear();
        } else {
            return "SEPTEMBER " + currentDate.getYear();
        }
    }

    public static String getPreviousQuaterEndDate() {
        // Get the current date
        LocalDate currentDate = LocalDate.now();

        // Determine the financial year
        Month month = currentDate.getMonth();

        LocalDate quaterEndDate;

        if (month.compareTo(Month.JUNE) <= 0) {
            quaterEndDate = YearMonth.of(currentDate.getYear(), Month.MARCH).atEndOfMonth();
        } else if (month.compareTo(Month.SEPTEMBER) <= 0) {
            quaterEndDate = YearMonth.of(currentDate.getYear(), Month.JUNE).atEndOfMonth();
        } else if (month.compareTo(Month.DECEMBER) < 0) {
            quaterEndDate = YearMonth.of(currentDate.getYear(), Month.SEPTEMBER).atEndOfMonth();
        } else {
            quaterEndDate = YearMonth.of(currentDate.getYear(), Month.DECEMBER).atEndOfMonth();
        }

        DateTimeFormatter dateTimeFormatterMMdd = DateTimeFormatter.ofPattern("MMdd");
        DateTimeFormatter dateTimeFormatterddMM = DateTimeFormatter.ofPattern("ddMM");

        return quaterEndDate.format(dateTimeFormatterMMdd) + "-" + quaterEndDate.format(dateTimeFormatterddMM);
    }

    public static String getQuaterEndDate() {
        // Get the current date
        LocalDate currentDate = LocalDate.now();

        // Determine the financial year
        Month month = currentDate.getMonth();

        LocalDate quaterEndDate;

        if (month.compareTo(Month.JUNE) <= 0) {
            quaterEndDate = YearMonth.of(currentDate.getYear(), Month.JUNE).atEndOfMonth();
        } else if (month.compareTo(Month.SEPTEMBER) <= 0) {
            quaterEndDate = YearMonth.of(currentDate.getYear(), Month.SEPTEMBER).atEndOfMonth();
        } else if (month.compareTo(Month.DECEMBER) < 0) {
            quaterEndDate = YearMonth.of(currentDate.getYear(), Month.DECEMBER).atEndOfMonth();
        } else {
            quaterEndDate = YearMonth.of(currentDate.getYear(), Month.MARCH).atEndOfMonth();
        }

        DateTimeFormatter dateTimeFormatterMMdd = DateTimeFormatter.ofPattern("MMdd");
        DateTimeFormatter dateTimeFormatterddMM = DateTimeFormatter.ofPattern("ddMM");

        return quaterEndDate.format(dateTimeFormatterMMdd) + "-" + quaterEndDate.format(dateTimeFormatterddMM);
    }

    public ResponseEntity<String> savedata(Map<String, Object> map) {
        Map<String, String> loginUserData = (Map<String, String>) map.get("user");
        log.info("token data - " + loginUserData);

        Map<String, Object> data = (Map<String, Object>) map.get("data");
        log.info("user  data map details-" + data);
        ///list of list for data we get from Frontend
        List<List<String>> datalist = (List<List<String>>) data.get("value");
        //log.info("datalist before-" + datalist.size());
        //log.info("datalist before-" + datalist);

        //log.info("datalist before-" + datalist.size());
        log.info("datalist after-" + datalist);

        //System.exit(0);
        ResponseVO<Boolean> responseVO = new ResponseVO<>();
        //ResponseVO<Map<String, Object>> responseVO = new ResponseVO<>();
        //Map<String, Object> result = new HashMap<>();

        String previousDate = loginUserData.get("quarterEndDate").substring(6);
        String quarter = loginUserData.get("quarterEndDate").substring(3, 5);
        log.info("quarter:-" + quarter);
        log.info("previousDate:-" + previousDate);
        int last = Integer.parseInt(previousDate);
        int secondlast = last - 1;
        String previousYearEndDate = loginUserData.get("quarterEndDate").substring(0, 6) + secondlast;
        log.info("previousYearEndDate-" + previousYearEndDate);

        Timestamp prevquarterEndDate = getQedFormatter(previousYearEndDate);
        log.info("prevquarterEndDate:-" + prevquarterEndDate);

        Timestamp quarterEndDate = getQedFormatter(loginUserData.get("quarterEndDate"));
        log.info("quarterEndDate:-" + quarterEndDate);

        String quarteryear = loginUserData.get("quarterEndDate").substring(6, 10);

        String appendValue = previousYearEndDate + " RS.  Ps.";

        String quarterMonth = "";
        String qedStringFormat = "";
        String previousFinancialEnd = "";
        if (quarter.equalsIgnoreCase("03")) {
            quarterMonth = "TWELVE MONTHS";
            qedStringFormat = "31st March " + previousDate;
            previousFinancialEnd = Integer.toString(secondlast);
        } else if (quarter.equalsIgnoreCase("06")) {
            quarterMonth = "QUARTER";
            qedStringFormat = "30th June " + previousDate;
            previousFinancialEnd = previousDate;
        } else if (quarter.equalsIgnoreCase("09")) {
            quarterMonth = "HALF YEAR";
            qedStringFormat = "30th September " + previousDate;
            previousFinancialEnd = previousDate;
        } else if (quarter.equalsIgnoreCase("12")) {
            quarterMonth = "NINE MONTHS";
            qedStringFormat = "31st September " + previousDate;
            previousFinancialEnd = previousDate;
        }

        // Resignned due to error
        // local variables referenced from a lambda expression must be final or effectively final
        String quarterMonthNew = quarterMonth;

        //System.exit(0);

        List<Map<String, Object>> columnDataList = rw01Repository.findColumnData(
                appendValue,
                getQuaterEndMonthYear(),
                quarter,
                quarteryear,
                "3049",
                (String) data.get("tabValue"),
                (String) loginUserData.get("quarterEndDate"),
                quarterMonthNew,
                qedStringFormat,
                previousFinancialEnd
        );


        ////////////////////////////////////////////////////////////////////////////////////////////

        try {
            if (((String) data.get("tabValue")).equalsIgnoreCase("1")) {
                datalist.remove(4);
                datalist.remove(19);

                Rw01 rw01 = new Rw01();

                String headingValuesParticulars = (String) columnDataList.get(3).get("HEADING_VALUES");

                String[] headingValuesParticularsArray = headingValuesParticulars.split("\\|");


                List<String> list = new ArrayList<>(Arrays.asList(headingValuesParticularsArray));
                list.remove(4);
                list.remove(19);
                String[] headingValuesParticularsArr = list.toArray(new String[0]);


                AtomicInteger index = new AtomicInteger(0);
                /////////////////////////////////////////////////////////////////////////////////////////
                log.info("datalist.size() " + datalist.size());
                log.info("headingValuesParticularsArr.size() " + headingValuesParticularsArr.length);
                //log.info("headingValuesSrNoArr.size() " + headingValuesSrNoArr.length);

                datalist.stream().forEach((c) -> {
                    log.info("into for each Tab 111 >> " + c);
                    Rw01 saveR01 = new Rw01();
                    Rw01 aa = rw01Repository.findByPlSuplDateAndPlSuplBranchAndPlSuplId(quarterEndDate, loginUserData.get("branch_code"), c.get(6));
                    log.info("aa Tab 111 -" + aa);

                    log.info("before increment" + index.get());
                    int forEachIndex = index.getAndIncrement();
                    log.info("forEachIndex " + forEachIndex);
                    log.info("after increment" + index.get());


                    /////////////////////////////////////////
                    int year = Integer.parseInt(previousDate);
                    String yearCurr = "";

                    if (quarter.equalsIgnoreCase("03")) {
                        year = year - 1;
                        yearCurr = "MARCH " + previousDate;
                    } else if (quarter.equalsIgnoreCase("06")) {
                        yearCurr = "JUNE " + previousDate; // July
                    } else if (quarter.equalsIgnoreCase("09")) {
                        yearCurr = "SEPTEMBER " + previousDate;
                    } else if (quarter.equalsIgnoreCase("12")) {
                        yearCurr = "DECEMBER " + previousDate;
                    }

                    int plSupID = Integer.parseInt(c.get(6));

                    String subHead = "";

                    if (plSupID <= 18) {
                        subHead = "I.";
                    } else if (plSupID <= 36) {
                        subHead = "II. DETAILS REGARDING RURAL ADVANCES (POPULATION UPTO 9,999 AS PER 2011 CENSUS)  'BRANCH ADVANCES AS AT THE LAST DAY OF THE MONTH FROM APRIL " + year + " To " + yearCurr;
                    } else if (plSupID <= 54) {
                        subHead = "III. PRIOR PERIOD ITEMS (I.E ITEMS PERTAINING TO PERIOD PRIOR to 01.04." + year + " ACCOUNTED FOR DURING THE CURRENT PERIOD)";
                    }
                    /////////////////////////////////////////////////////////////////////////////
                    ///

                    if (aa == null) {


                        rw01.setPlSuplId(c.get(6));


                        rw01.setPlSuplBranch(loginUserData.get("branch_code"));
                        rw01.setPlSuplCy(c.get(4));
                        rw01.setPlSuplDate(quarterEndDate);
                        rw01.setPlSuplPy(c.get(0));
                        rw01.setReportMasterListIdFk(String.valueOf(data.get("submissionId")));

                        // rw01.setPlSuplDetails(headingValuesSrNoArr[index.getAndIncrement()] + " " + headingValuesParticularsArr[index.getAndIncrement()]);
                        //rw01.setPlSuplDetails(headingValuesSrNoArr[forEachIndex] + " " + headingValuesParticularsArr[forEachIndex]);
                        rw01.setPlSuplDetails(headingValuesParticularsArr[forEachIndex]);
                        rw01.setPlSuplHead(subHead);

                        log.info("rw01.submissionId:-" + rw01.getReportMasterListIdFk());
                        saveR01 = rw01Repository.save(rw01);
                        log.info("saveR01 1012" + saveR01);
                    } else {
                        log.info("into elseeeee Tab 111");
                        aa.setPlSuplPy(c.get(0));
                        aa.setPlSuplCy(c.get(4));

                        // aa.setPlSuplDetails(headingValuesSrNoArr[index.getAndIncrement()] + " " + headingValuesParticularsArr[index.getAndIncrement()]);
                        //aa.setPlSuplDetails(headingValuesSrNoArr[forEachIndex] + " " + headingValuesParticularsArr[forEachIndex]);
                        rw01.setPlSuplDetails(headingValuesParticularsArr[forEachIndex]);
                        rw01.setPlSuplHead(subHead);
                        log.info("final aa" + aa);
                        saveR01 = rw01Repository.save(aa);
                        log.info("saveR01 1030" + saveR01);
                    }

                    // index.getAndIncrement();
                    // forEachIndex++;

                });


                log.info("1039");
            } else if (((String) data.get("tabValue")).equalsIgnoreCase("2")) {


                log.info("datalist before-" + datalist);
                datalist.remove(0);
                datalist.remove(3);
                datalist.remove(19);   //18
                datalist.remove(26);   //18
                datalist.remove(29);   //25
                //datalist.remove(28);
                log.info("datalist after-" + datalist);
                Rw01B rw01B = new Rw01B();

                /////////////////////////////////////////////////
                // List<Map<String, Object>> columnDataList = rw01Repository.findColumnData(
                //     appendValue,
                //     getQuaterEndMonthYear(),
                //     quarteryear,
                //     "3049",
                //     (String) data.get("tabValue"),
                //     (String) loginUserData.get("quarterEndDate"),
                //     quarterMonthNew
                // );

                //String headingValuesSrNo = (String) columnDataList.get(2).get("HEADING_VALUES");
                String headingValuesParticulars = (String) columnDataList.get(1).get("HEADING_VALUES");

                //String[] headingValuesSrNoArr = headingValuesSrNo.split("\\|");
                String[] headingValuesParticularsArray = headingValuesParticulars.split("\\|");

                List<String> list = new ArrayList<>(Arrays.asList(headingValuesParticularsArray));
                list.remove(0);
                list.remove(3);
                list.remove(19);
                list.remove(26);
                list.remove(29);

                String[] headingValuesParticularsArr = list.toArray(new String[0]);

                AtomicInteger index = new AtomicInteger(0);
                AtomicInteger AI_YA_SUPL1_ID_INDEX = new AtomicInteger(0);
                String[] YA_SUPL1_ID = {
                        "1", "2", "201",
                        "4", "5", "6", "7", "8", "9", "10", "11", "12", "13", "14", "15", "16", "17", "202",
                        "19", "20", "21", "22", "23", "24", "25", "203",
                        "26", "27", "204",
                        "28", "29", "420"
                };


                datalist.stream().forEach((c) -> {
                            log.info("into for each 2");
                            log.info("into for each 2 " + quarterEndDate);
                            log.info("into for each 2 " + loginUserData.get("branch_code"));
                            log.info("into for each 2 " + c.get(4));

                            String YA_SUPL1_ID_INDEX = YA_SUPL1_ID[index.get()];

                            String subHead = "";

                            if (index.get() <= 2) {
                                subHead = "Particulars of Staff Security Deposits received from Cashiers/POs";
                            } else if (index.get() <= 17) {
                                subHead = "BREAK UP OF OUTSTANDINGS IN SUSPENSE ACCOUNT";
                            } else if (index.get() <= 18) {
                                subHead = "Amount of Bankers Cheques issued by debit to Charges Account and outstanding for more than 3 years credited to Charges Account during the year";
                            } else if (index.get() <= 25) {
                                subHead = "BREAK UP OF OUTSTANDINGS IN SUNDRY DEPOSITS ACCOUNT";
                            } else if (index.get() <= 28) {
                                subHead = "Other Monies for which Bank is Contingently liable";
                            } else if (index.get() <= 31) {
                                subHead = "Staff Advances";
                            }


                            Rw01B saveR01 = new Rw01B();
                            Rw01B aa = rw01BRepository.findByYaSuplDateAndYaSuplBranchAndYaSuplId(quarterEndDate, loginUserData.get("branch_code"), c.get(4));
                            log.info("aa-" + aa);
                            if (aa == null) {
                                //rw01B.setYaSuplId(c.get(4));
                                rw01B.setYaSuplId(YA_SUPL1_ID_INDEX);

                                rw01B.setYaSuplBranch(loginUserData.get("branch_code"));
                                rw01B.setYaSuplCy(c.get(2));
                                rw01B.setYaSuplDate(quarterEndDate);
                                rw01B.setReportMasterListIdFk(String.valueOf(data.get("submissionId")));
                                rw01B.setYaSuplDetails(headingValuesParticularsArr[index.getAndIncrement()]);
                                rw01B.setYaSuplHead(subHead);


                                log.info("rw01B.submissionId:-" + rw01B.getReportMasterListIdFk());
                                saveR01 = rw01BRepository.save(rw01B);
                            } else {
                                //aa.setYaSuplId(YA_SUPL1_ID_INDEX);
                                //index.getAndIncrement()
                                log.info("into elseeeee");
                                //aa.setYaSuplCy(c.get(4));   @rutuja
                                aa.setYaSuplCy(c.get(2));
                                //aa.setYaSuplDetails(headingValuesParticularsArr[index.getAndIncrement()]);
                                //aa.setYaSuplHead(subHead);
                                log.info("final aa" + aa);
                                saveR01 = rw01BRepository.save(aa);
                            }

                        }
                );
            } else if (((String) data.get("tabValue")).equalsIgnoreCase("3")) {
                Rw01C rw01C = new Rw01C();


                /////////////////////////////////////////////////
                // List<Map<String, Object>> columnDataList = rw01Repository.findColumnData(
                //     appendValue,
                //     getQuaterEndMonthYear(),
                //     quarteryear,
                //     "3049",
                //     (String) data.get("tabValue"),
                //     (String) loginUserData.get("quarterEndDate"),
                //     quarterMonthNew
                // );

                String headingValuesSrNo = (String) columnDataList.get(0).get("HEADING_VALUES");
                String headingValuesParticulars = (String) columnDataList.get(1).get("HEADING_VALUES");

                String[] headingValuesSrNoArr = headingValuesSrNo.split("\\|");
                String[] headingValuesParticularsArr = headingValuesParticulars.trim().split("\\|");

                AtomicInteger index = new AtomicInteger(0);
                /////////////////////////////////////////////////////////////////////////////////////////


                datalist.stream().forEach((c) -> {
                            log.info("into for each");

                            Rw01C saveR01 = new Rw01C();
                            Rw01C aa = rw01CRepository.findByYaSupl2DateAndYaSupl2BranchAndYaSupl2Id(quarterEndDate, loginUserData.get("branch_code"), c.get(7));
                            log.info("aa-" + aa);
                            if (aa == null) {
                                rw01C.setYaSupl2Id(c.get(7));
                                rw01C.setYaSupl2Branch(loginUserData.get("branch_code"));
                                rw01C.setYaSupl2Loc(c.get(2));
                                rw01C.setYaSupl2Garantee(c.get(3));
                                rw01C.setYaSupl2Accept(c.get(4));
                                rw01C.setYaSuplLou(c.get(5));
                                rw01C.setYaSupl2Date(quarterEndDate);
                                rw01C.setREPORT_MASTER_LIST_ID_FK(String.valueOf(data.get("submissionId")));

                                //headingValuesParticularsArr[index.getAndIncrement()].split("Total")[0].trim();
                                //rw01C.setYaSupl2Details(headingValuesParticularsArr[index.getAndIncrement()]);
                                //log.info("AAAAAAAAAA  " + headingValuesParticularsArr[index.getAndIncrement()].split("Total")[0].trim());
                                rw01C.setYaSupl2Details(headingValuesParticularsArr[index.getAndIncrement()].split("(?=Total)")[0]);


                                log.info("rw01C.submissionId:-" + rw01C.getREPORT_MASTER_LIST_ID_FK());
                                saveR01 = rw01CRepository.save(rw01C);
                            } else {
                                log.info("into elseeeee");
                                aa.setYaSupl2Loc(c.get(2));
                                aa.setYaSupl2Garantee(c.get(3));
                                aa.setYaSupl2Accept(c.get(4));
                                aa.setYaSuplLou(c.get(5));

                                ///aa.setYaSupl2Details(headingValuesParticularsArr[index.getAndIncrement()]);
                                //log.info("BBBBB  " + headingValuesParticularsArr[index.getAndIncrement()]);
                                //log.info("CCCCCCCCC  " + headingValuesParticularsArr[index.getAndIncrement()].split("Total")[0].trim());
                                //aa.setYaSupl2Details(headingValuesParticularsArr[index.getAndIncrement()].split("(?=Total)")[0].trim());
                                //aa.setYaSupl2Details(headingValuesParticularsArr[index.getAndIncrement()].split("(?=Total)(?=\\d)")[0].trim());
                                aa.setYaSupl2Details(headingValuesParticularsArr[index.getAndIncrement()].split("(?=Total)")[0]);


                                log.info("CCCCCCCCC  " + aa.getYaSupl2Details());
                                log.info("final aa" + aa);
                                saveR01 = rw01CRepository.save(aa);
                            }

                        }
                );
            } else if (((String) data.get("tabValue")).equalsIgnoreCase("4")) {
                Rw01D rw01D = new Rw01D();

                /////////////////////////////////////////////////
                // List<Map<String, Object>> columnDataList = rw01Repository.findColumnData(
                //     appendValue,
                //     getQuaterEndMonthYear(),
                //     quarteryear,
                //     "3049",
                //     (String) data.get("tabValue"),
                //     (String) loginUserData.get("quarterEndDate"),
                //     quarterMonthNew
                // );

                //String headingValuesSrNo = (String) columnDataList.get(2).get("HEADING_VALUES");
                String headingValuesParticulars = (String) columnDataList.get(1).get("HEADING_VALUES");

                //String[] headingValuesSrNoArr = headingValuesSrNo.split("\\|");
                String[] headingValuesParticularsArr = headingValuesParticulars.trim().split("\\|");

                AtomicInteger index = new AtomicInteger(0);
                /////////////////////////////////////////////////////////////////////////////////////////


                datalist.stream().forEach((c) -> {
                            log.info("into for each");

                            Rw01D saveR01 = new Rw01D();
                            Rw01D aa = rw01DRepository.findByYaSupl3DateAndYaSupl3BranchAndYaSupl3Id(quarterEndDate, loginUserData.get("branch_code"), c.get(4));
                            log.info("aa-" + aa);
                            if (aa == null) {
                                rw01D.setYaSupl3Id(c.get(4));
                                rw01D.setYaSupl3Branch(loginUserData.get("branch_code"));
                                rw01D.setYaSupl3Cy(c.get(2));
                                rw01D.setReportMasterListIdFk(String.valueOf(data.get("submissionId")));
                                rw01D.setYaSupl3Date(quarterEndDate);
                                rw01D.setYaSupl3Details(headingValuesParticularsArr[index.getAndIncrement()].split("\\[")[0]);

                                log.info("rw01D.submissionId:-" + rw01D.getReportMasterListIdFk());
                                saveR01 = rw01DRepository.save(rw01D);
                            } else {
                                log.info("into elseeeee");
                                aa.setYaSupl3Cy(c.get(2));
                                aa.setYaSupl3Details(headingValuesParticularsArr[index.getAndIncrement()].split("\\[")[0]);
                                log.info("final aa" + aa);
                                saveR01 = rw01DRepository.save(aa);
                            }

                        }
                );

            } else if (((String) data.get("tabValue")).equalsIgnoreCase("5")) {
                Rw01E rw01E = new Rw01E();

                /////////////////////////////////////////////////
                // List<Map<String, Object>> columnDataList = rw01Repository.findColumnData(
                //     appendValue,
                //     getQuaterEndMonthYear(),
                //     quarteryear,
                //     "3049",
                //     (String) data.get("tabValue"),
                //     (String) loginUserData.get("quarterEndDate"),
                //     quarterMonthNew
                // );

                //String headingValuesSrNo = (String) columnDataList.get(2).get("HEADING_VALUES");
                String headingValuesParticulars = (String) columnDataList.get(1).get("HEADING_VALUES");

                //String[] headingValuesSrNoArr = headingValuesSrNo.split("\\|");
                String[] headingValuesParticularsArr = headingValuesParticulars.trim().split("\\|");

                AtomicInteger index = new AtomicInteger(0);
                /////////////////////////////////////////////////////////////////////////////////////////


                datalist.stream().forEach((c) -> {
                            log.info("into for each");

                            Rw01E saveR01 = new Rw01E();
                            Rw01E aa = rw01ERepository.findByYaSupl4DateAndYaSupl4BranchAndYaSupl4Id(quarterEndDate, loginUserData.get("branch_code"), c.get(5));
                            log.info("aa-" + aa);
                            if (aa == null) {
                                rw01E.setYaSupl4Id(c.get(5));
                                rw01E.setYaSupl4Branch(loginUserData.get("branch_code"));
                                rw01E.setYaSupl4Amt(c.get(3));
                                rw01E.setYaSupl4Claims(c.get(2));
                                rw01E.setReportMasterListIdFk(String.valueOf(data.get("submissionId")));
                                rw01E.setYaSupl4Date(quarterEndDate);
                                rw01E.setYaSupl4Details(headingValuesParticularsArr[index.getAndIncrement()]);

                                log.info("rw01E.submissionId:-" + rw01E.getReportMasterListIdFk());
                                saveR01 = rw01ERepository.save(rw01E);
                            } else {
                                log.info("into elseeeee");
                                aa.setYaSupl4Claims(c.get(2));
                                aa.setYaSupl4Amt(c.get(3));
                                aa.setYaSupl4Details(headingValuesParticularsArr[index.getAndIncrement()]);

                                log.info("final aa" + aa);
                                saveR01 = rw01ERepository.save(aa);
                            }

                        }
                );

            } else if (((String) data.get("tabValue")).equalsIgnoreCase("6")) {
                Rw01F rw01F = new Rw01F();

                /////////////////////////////////////////////////
                // List<Map<String, Object>> columnDataList = rw01Repository.findColumnData(
                //     appendValue,
                //     getQuaterEndMonthYear(),
                //     quarteryear,
                //     "3049",
                //     (String) data.get("tabValue"),
                //     (String) loginUserData.get("quarterEndDate"),
                //     quarterMonthNew
                // );

                //String headingValuesSrNo = (String) columnDataList.get(2).get("HEADING_VALUES");
                String headingValuesParticulars = (String) columnDataList.get(1).get("HEADING_VALUES");

                //String[] headingValuesSrNoArr = headingValuesSrNo.split("\\|");
                String[] headingValuesParticularsArr = headingValuesParticulars.trim().split("\\|");

                AtomicInteger index = new AtomicInteger(0);
                AtomicInteger setYaSupl5Id_INDEX = new AtomicInteger(901);
                /////////////////////////////////////////////////////////////////////////////////////////


                datalist.stream().forEach((c) -> {
                            log.info("into for each");

                            Rw01F saveR01 = new Rw01F();
                            Rw01F aa = rw01FRepository.findByYaSupl5DateAndYaSupl5BranchAndYaSupl5Id(quarterEndDate, loginUserData.get("branch_code"), c.get(4));
                            log.info("aa-" + aa);
                            log.info("c-" + c);
                            log.info("c-" + c.get(4));


                            if (aa == null) {


                                if (setYaSupl5Id_INDEX.get() == 905) {
                                    rw01F.setYaSupl5Type("");
                                    rw01F.setYaSupl5Id("99");
                                } else {
                                    rw01F.setYaSupl5Type("PARTLY PAID INVESTMENTS AND OTHER COMMITMENTS (For Global Markets only)");
                                    //rw01F.setYaSupl5Id(c.get(4));
                                    rw01F.setYaSupl5Id(Integer.toString(setYaSupl5Id_INDEX.getAndIncrement()));
                                }


                                rw01F.setYaSupl5Branch(loginUserData.get("branch_code"));
                                rw01F.setYaSupl5Amt(c.get(2));
                                rw01F.setReportMasterListIdFk(String.valueOf(data.get("submissionId")));
                                rw01F.setYaSupl5Date(quarterEndDate);

                                // PARTLY PAID INVESTMENTS AND OTHER COMMITMENTS (For Global Markets only)

                                //rw01F.setYaSupl5Details(c.get(1));

                                log.info("c.get(1) >>>>>  " + c.get(1));

                                if (c.get(1) != "") {
                                    rw01F.setYaSupl5Details(c.get(1));
                                } else {
                                    rw01F.setYaSupl5Details(headingValuesParticularsArr[index.getAndIncrement()]);
                                }


                                log.info("rw01F.submissionId:-" + rw01F.getReportMasterListIdFk());
                                saveR01 = rw01FRepository.save(rw01F);
                            } else {

                                aa.setYaSupl5Type("PARTLY PAID INVESTMENTS AND OTHER COMMITMENTS (For Global Markets only)");
                                if (setYaSupl5Id_INDEX.get() == 905) {
                                    aa.setYaSupl5Type("");
                                }

                                rw01F.setYaSupl5Id(Integer.toString(setYaSupl5Id_INDEX.getAndIncrement()));

                                log.info("into elseeeee");
                                aa.setYaSupl5Amt(c.get(2));

                                // aa.setYaSupl5Details(c.get(1));
                                aa.setYaSupl5Details(headingValuesParticularsArr[index.getAndIncrement()]);

                                //aa.setYaSupl5Type("PARTLY PAID INVESTMENTS AND OTHER COMMITMENTS (For Global Markets only)");

                                log.info("final aa" + aa);
                                saveR01 = rw01FRepository.save(aa);
                            }


                        }
                );

            } else if (((String) data.get("tabValue")).equalsIgnoreCase("7")) {
                Rw01F rw01F = new Rw01F();

                /////////////////////////////////////////////////
                // List<Map<String, Object>> columnDataList = rw01Repository.findColumnData(
                //     appendValue,
                //     getQuaterEndMonthYear(),
                //     quarteryear,
                //     "3049",
                //     (String) data.get("tabValue"),
                //     (String) loginUserData.get("quarterEndDate"),
                //     quarterMonthNew
                // );

                //String headingValuesSrNo = (String) columnDataList.get(2).get("HEADING_VALUES");
                String headingValuesParticulars = (String) columnDataList.get(1).get("HEADING_VALUES");

                //String[] headingValuesSrNoArr = headingValuesSrNo.split("\\|");
                String[] headingValuesParticularsArr = headingValuesParticulars.split("\\|");

                AtomicInteger index = new AtomicInteger(0);
                /////////////////////////////////////////////////////////////////////////////////////////


                datalist.stream().forEach((c) -> {
                            log.info("into for each");

                            Rw01F saveR01 = new Rw01F();
                            Rw01F aa = rw01FRepository.findByYaSupl5DateAndYaSupl5BranchAndYaSupl5Id(quarterEndDate, loginUserData.get("branch_code"), c.get(4));
                            log.info("aa-" + aa);
                            if (aa == null) {
                                rw01F.setYaSupl5Id(c.get(4));
                                rw01F.setYaSupl5Branch(loginUserData.get("branch_code"));
                                rw01F.setYaSupl5Amt(c.get(2));
                                rw01F.setReportMasterListIdFk(String.valueOf(data.get("submissionId")));
                                rw01F.setYaSupl5Date(quarterEndDate);

                                String headingValuesParticular = headingValuesParticularsArr[index.get()];
                                rw01F.setYaSupl5Type("");
                                if (headingValuesParticular.equalsIgnoreCase("(i) For which control entries are passed") || headingValuesParticular.equalsIgnoreCase("(ii) For which no control entries are passed")) {

                                    log.info("c.get(4) >>>> " + c.get(4));
                                    if (Integer.parseInt(c.get(4)) == 7 || Integer.parseInt(c.get(4)) == 8) {
                                        rw01F.setYaSupl5Type("I. Payable in India");
                                    } else {
                                        rw01F.setYaSupl5Type("Payable outside India");
                                    }
                                    //rw01F.setYaSupl5Details(headingValuesParticularsArr[index.getAndIncrement()]);
                                }

                                rw01F.setYaSupl5Details(headingValuesParticularsArr[index.getAndIncrement()]);


                                log.info("rw01F.submissionId:-" + rw01F.getReportMasterListIdFk());
                                saveR01 = rw01FRepository.save(rw01F);
                            } else {
                                log.info("into elseeeee");
                                aa.setYaSupl5Amt(c.get(2));


                                String headingValuesParticular = headingValuesParticularsArr[index.get()];
                                aa.setYaSupl5Type("");
                                if (headingValuesParticular.equalsIgnoreCase("(i) For which control entries are passed") || headingValuesParticular.equalsIgnoreCase("(ii) For which no control entries are passed")) {

                                    log.info("c.get(4) >>>> " + c.get(4));
                                    if (Integer.parseInt(c.get(4)) == 7 || Integer.parseInt(c.get(4)) == 8) {
                                        aa.setYaSupl5Type("I. Payable in India");
                                    } else {
                                        aa.setYaSupl5Type("Payable outside India");
                                    }
                                    //aa.setYaSupl5Details(headingValuesParticularsArr[index.getAndIncrement()]);
                                }


                                aa.setYaSupl5Details(headingValuesParticularsArr[index.getAndIncrement()]);

                                log.info("final aa" + aa);
                                saveR01 = rw01FRepository.save(aa);

                            }

                        }
                );

            }


            boolean isSave = true;
            //result.put("status", true);
            //result.put("newId", 1);
            //result.put("newRowNum", 1);

            log.info(" 1581 ");

            //resultDataMap.put("status",false);
            responseVO.setResult(isSave);
            responseVO.setStatusCode(HttpStatus.OK.value());
            responseVO.setMessage("Data saved successfully.");
            return new ResponseEntity(responseVO, HttpStatus.OK);
        } catch (Exception e) {
            //result.put("status", false);
            //result.put("newId", 0);
            //result.put("newRowNum", -1);
            //return result;
            responseVO.setResult(false);
            log.info(" Exception Occurred: " + e.getCause());
            responseVO.setStatusCode(HttpStatus.INTERNAL_SERVER_ERROR.value());
            responseVO.setMessage("Something went wrong.");
            throw new RuntimeException(e);
        }


    }

    public ResponseEntity<String> getYsaCheckAmount(Map<String, Object> map) {
        try {
            Map<String, String> loginUserData = (Map<String, String>) map.get("user");
            log.info("token data - " + loginUserData);

            Map<String, String> data = (Map<String, String>) map.get("data");
            log.info("user  data map details-" + data);

            ResponseVO<List<Map<String, Object>>> responseVO = new ResponseVO<>();

            String previousDate = loginUserData.get("quarterEndDate").substring(6);
            int last = Integer.parseInt(previousDate);
            int secondlast = last - 1;
            String previousYearEndDate = loginUserData.get("quarterEndDate").substring(0, 6) + secondlast;
            log.info("previousYearEndDate-" + previousYearEndDate);

            Date quarterEndDate = getQedFormatter(loginUserData.get("quarterEndDate"));
            List<Map<String, Object>> validateAmonut = rw01Repository.getPreRequisteHeaderAmount(quarterEndDate, loginUserData.get("branch_code"));
            responseVO.setResult(validateAmonut);
            responseVO.setMessage("YSA check amount fetched successfully.");
            responseVO.setStatusCode(HttpStatus.OK.value());
            return new ResponseEntity(responseVO, HttpStatus.OK);
        } catch (Exception e) {
            ResponseVO<Boolean> responseVO = new ResponseVO<>();
            log.info(" Exception Occurred: " + e.getCause());
            responseVO.setStatusCode(HttpStatus.INTERNAL_SERVER_ERROR.value());
            responseVO.setMessage("Something went wrong.");
            throw new RuntimeException(e);
        }
    }

}




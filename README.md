package com.tcs.beans;

import com.tcs.utils.PatternRegex;

import javax.validation.constraints.Pattern;
import java.util.ArrayList;

public class RW04 {


    private String reportMasterId;
    private String reportName;
    private ArrayList strRuleList = new ArrayList();
    private int strRuleListCount = 0;
    private String consolidatedSel = "";
    private String year = "";
    private ArrayList yearList = new ArrayList();
    private ArrayList quarterList = new ArrayList();
    private String filename = "";
    private String quaterDate = "";
    private String reportIdHidden = "";
    private String updateFlag = "";
    private String reportId = "";

    private String branchCode = "";
    private String reportType = "";
    private String paramName = "";
    private String paramValue = "";
    private String branch = "";
    private String dataFlag="";
    private String nilFlag="";

    private ArrayList faultyFlagList = new ArrayList();

    private String entryNo = "";

    private String strStatus = "";

    private int faultyflag = 0;

    private String quarter = "";
    private String action;
    private String financialYear = "";
    // start from here

    private String particulars = "";
    private String provAmt2015 = "";
    private String writeOffDur12mon = "";
    private String additionDur12mon = "";
    private String reduInProviAmt = "";
    private String proviAmt2016 = "";
    private String ratePOfProv = "";
    private String provReq = "";

    @Pattern(regexp = PatternRegex.ALPHANUMERIC_50)
    private String particularsList = "";
    @Pattern(regexp = PatternRegex.AMOUNT_POSITIVE)
    private String provAmt2015List = "";
    @Pattern(regexp = PatternRegex.AMOUNT_POSITIVE)
    private String writeOffDur12monList = "";
    @Pattern(regexp = PatternRegex.AMOUNT_POSITIVE)
    private String additionDur12monList = "";
    @Pattern(regexp = PatternRegex.AMOUNT_POSITIVE)
    private String reduInProviAmtList = "";
    @Pattern(regexp = PatternRegex.AMOUNT_NEGATIVE)
    private String proviAmt2016List = "";
    @Pattern(regexp = PatternRegex.AMOUNT_POSITIVE)
    private String ratePOfProvList = "";
    @Pattern(regexp = PatternRegex.AMOUNT_NEGATIVE)
    private String provReqList = "";

    @Pattern(regexp = PatternRegex.AMOUNT_POSITIVE)
    private String fraudsDebitedProvAfter;
    @Pattern(regexp = PatternRegex.AMOUNT_POSITIVE)
    private String fraudsDebitedWrite;
    @Pattern(regexp = PatternRegex.AMOUNT_POSITIVE)
    private String fraudsDebitedAddition;
    @Pattern(regexp = PatternRegex.AMOUNT_POSITIVE)
    private String fraudsDebitedReduction;
    @Pattern(regexp = PatternRegex.AMOUNT_NEGATIVE)
    private String fraudsDebitedProvOn;
    @Pattern(regexp = PatternRegex.AMOUNT_POSITIVE)
    private String fraudsDebitedRate;
    @Pattern(regexp = PatternRegex.AMOUNT_NEGATIVE)
    private String fraudsDebitedProvReq;

    @Pattern(regexp = PatternRegex.AMOUNT_POSITIVE)
    private String fraudsDebitedPrior100ProvAfter;
    @Pattern(regexp = PatternRegex.AMOUNT_POSITIVE)
    private String fraudsDebitedPrior100Write;
    @Pattern(regexp = PatternRegex.AMOUNT_POSITIVE)
    private String fraudsDebitedPrior100Addition;
    @Pattern(regexp = PatternRegex.AMOUNT_POSITIVE)
    private String fraudsDebitedPrior100Reduction;
    @Pattern(regexp = PatternRegex.AMOUNT_NEGATIVE)
    private String fraudsDebitedPrior100ProvOn;
    @Pattern(regexp = PatternRegex.AMOUNT_POSITIVE)
    private String fraudsDebitedPrior100Rate;
    @Pattern(regexp = PatternRegex.AMOUNT_NEGATIVE)
    private String fraudsDebitedPrior100ProvReq;

    @Pattern(regexp = PatternRegex.AMOUNT_POSITIVE)
    private String fraudsDebitedPrior75ProvAfter;
    @Pattern(regexp = PatternRegex.AMOUNT_POSITIVE)
    private String fraudsDebitedPrior75Write;
    @Pattern(regexp = PatternRegex.AMOUNT_POSITIVE)
    private String fraudsDebitedPrior75Addition;
    @Pattern(regexp = PatternRegex.AMOUNT_POSITIVE)
    private String fraudsDebitedPrior75Reduction;
    @Pattern(regexp = PatternRegex.AMOUNT_NEGATIVE)
    private String fraudsDebitedPrior75ProvOn;
    @Pattern(regexp = PatternRegex.AMOUNT_POSITIVE)
    private String fraudsDebitedPrior75Rate;
    @Pattern(regexp = PatternRegex.AMOUNT_NEGATIVE)
    private String fraudsDebitedPrior75ProvReq;

    @Pattern(regexp = PatternRegex.AMOUNT_POSITIVE)
    private String fraudsDebitedPrior50ProvAfter;
    @Pattern(regexp = PatternRegex.AMOUNT_POSITIVE)
    private String fraudsDebitedPrior50Write;
    @Pattern(regexp = PatternRegex.AMOUNT_POSITIVE)
    private String fraudsDebitedPrior50Addition;
    @Pattern(regexp = PatternRegex.AMOUNT_POSITIVE)
    private String fraudsDebitedPrior50Reduction;
    @Pattern(regexp = PatternRegex.AMOUNT_NEGATIVE)
    private String fraudsDebitedPrior50ProvOn;
    @Pattern(regexp = PatternRegex.AMOUNT_POSITIVE)
    private String fraudsDebitedPrior50Rate;
    @Pattern(regexp = PatternRegex.AMOUNT_NEGATIVE)
    private String fraudsDebitedPrior50ProvReq;

    @Pattern(regexp = PatternRegex.AMOUNT_POSITIVE)
    private String fraudsDebitedPrior25ProvAfter;
    @Pattern(regexp = PatternRegex.AMOUNT_POSITIVE)
    private String fraudsDebitedPrior25Write;
    @Pattern(regexp = PatternRegex.AMOUNT_POSITIVE)
    private String fraudsDebitedPrior25Addition;
    @Pattern(regexp = PatternRegex.AMOUNT_POSITIVE)
    private String fraudsDebitedPrior25Reduction;
    @Pattern(regexp = PatternRegex.AMOUNT_NEGATIVE)
    private String fraudsDebitedPrior25ProvOn;
    @Pattern(regexp = PatternRegex.AMOUNT_POSITIVE)
    private String fraudsDebitedPrior25Rate;
    @Pattern(regexp = PatternRegex.AMOUNT_NEGATIVE)
    private String fraudsDebitedPrior25ProvReq;

    @Pattern(regexp = PatternRegex.AMOUNT_POSITIVE)
    private String fraudsDebitedDelayedProvAfter;
    @Pattern(regexp = PatternRegex.AMOUNT_POSITIVE)
    private String fraudsDebitedDelayedWrite;
    @Pattern(regexp = PatternRegex.AMOUNT_POSITIVE)
    private String fraudsDebitedDelayedAddition;
    @Pattern(regexp = PatternRegex.AMOUNT_POSITIVE)
    private String fraudsDebitedDelayedReduction;
    @Pattern(regexp = PatternRegex.AMOUNT_NEGATIVE)
    private String fraudsDebitedDelayedProvOn;
    @Pattern(regexp = PatternRegex.AMOUNT_POSITIVE)
    private String fraudsDebitedDelayedRate;
    @Pattern(regexp = PatternRegex.AMOUNT_NEGATIVE)
    private String fraudsDebitedDelayedProvReq;

    @Pattern(regexp = PatternRegex.AMOUNT_POSITIVE)
    private String othersRecalledProvAfter;
    @Pattern(regexp = PatternRegex.AMOUNT_POSITIVE)
    private String othersRecalledWrite;
    @Pattern(regexp = PatternRegex.AMOUNT_POSITIVE)
    private String othersRecalledAddition;
    @Pattern(regexp = PatternRegex.AMOUNT_POSITIVE)
    private String othersRecalledReduction;
    @Pattern(regexp = PatternRegex.AMOUNT_NEGATIVE)
    private String othersRecalledProvOn;
    @Pattern(regexp = PatternRegex.AMOUNT_POSITIVE)
    private String othersRecalledRate;
    @Pattern(regexp = PatternRegex.AMOUNT_NEGATIVE)
    private String othersRecalledProvReq;

    @Pattern(regexp = PatternRegex.AMOUNT_POSITIVE)
    private String fraudsOthersProvAfter;
    @Pattern(regexp = PatternRegex.AMOUNT_POSITIVE)
    private String fraudsOthersWrite;
    @Pattern(regexp = PatternRegex.AMOUNT_POSITIVE)
    private String fraudsOthersAddition;
    @Pattern(regexp = PatternRegex.AMOUNT_POSITIVE)
    private String fraudsOthersReduction;
    @Pattern(regexp = PatternRegex.AMOUNT_NEGATIVE)
    private String fraudsOthersProvOn;
    @Pattern(regexp = PatternRegex.AMOUNT_POSITIVE)
    private String fraudsOthersRate;
    @Pattern(regexp = PatternRegex.AMOUNT_NEGATIVE)
    private String fraudsOthersProvReq;

    @Pattern(regexp = PatternRegex.AMOUNT_POSITIVE)
    private String fraudsOthersPrior100ProvAfter;
    @Pattern(regexp = PatternRegex.AMOUNT_POSITIVE)
    private String fraudsOthersPrior100Write;
    @Pattern(regexp = PatternRegex.AMOUNT_POSITIVE)
    private String fraudsOthersPrior100Addition;
    @Pattern(regexp = PatternRegex.AMOUNT_POSITIVE)
    private String fraudsOthersPrior100Reduction;
    @Pattern(regexp = PatternRegex.AMOUNT_NEGATIVE)
    private String fraudsOthersPrior100ProvOn;
    @Pattern(regexp = PatternRegex.AMOUNT_POSITIVE)
    private String fraudsOthersPrior100Rate;
    @Pattern(regexp = PatternRegex.AMOUNT_NEGATIVE)
    private String fraudsOthersPrior100ProvReq;

    @Pattern(regexp = PatternRegex.AMOUNT_POSITIVE)
    private String fraudsOthersPrior75ProvAfter;
    @Pattern(regexp = PatternRegex.AMOUNT_POSITIVE)
    private String fraudsOthersPrior75Write;
    @Pattern(regexp = PatternRegex.AMOUNT_POSITIVE)
    private String fraudsOthersPrior75Addition;
    @Pattern(regexp = PatternRegex.AMOUNT_POSITIVE)
    private String fraudsOthersPrior75Reduction;
    @Pattern(regexp = PatternRegex.AMOUNT_NEGATIVE)
    private String fraudsOthersPrior75ProvOn;
    @Pattern(regexp = PatternRegex.AMOUNT_POSITIVE)
    private String fraudsOthersPrior75Rate;
    @Pattern(regexp = PatternRegex.AMOUNT_NEGATIVE)
    private String fraudsOthersPrior75ProvReq;

    @Pattern(regexp = PatternRegex.AMOUNT_POSITIVE)
    private String fraudsOthersPrior50ProvAfter;
    @Pattern(regexp = PatternRegex.AMOUNT_POSITIVE)
    private String fraudsOthersPrior50Write;
    @Pattern(regexp = PatternRegex.AMOUNT_POSITIVE)
    private String fraudsOthersPrior50Addition;
    @Pattern(regexp = PatternRegex.AMOUNT_POSITIVE)
    private String fraudsOthersPrior50Reduction;
    @Pattern(regexp = PatternRegex.AMOUNT_NEGATIVE)
    private String fraudsOthersPrior50ProvOn;
    @Pattern(regexp = PatternRegex.AMOUNT_POSITIVE)
    private String fraudsOthersPrior50Rate;
    @Pattern(regexp = PatternRegex.AMOUNT_NEGATIVE)
    private String fraudsOthersPrior50ProvReq;

    @Pattern(regexp = PatternRegex.AMOUNT_POSITIVE)
    private String fraudsOthersPrior25ProvAfter;
    @Pattern(regexp = PatternRegex.AMOUNT_POSITIVE)
    private String fraudsOthersPrior25Write;
    @Pattern(regexp = PatternRegex.AMOUNT_POSITIVE)
    private String fraudsOthersPrior25Addition;
    @Pattern(regexp = PatternRegex.AMOUNT_POSITIVE)
    private String fraudsOthersPrior25Reduction;
    @Pattern(regexp = PatternRegex.AMOUNT_NEGATIVE)
    private String fraudsOthersPrior25ProvOn;
    @Pattern(regexp = PatternRegex.AMOUNT_POSITIVE)
    private String fraudsOthersPrior25Rate;
    @Pattern(regexp = PatternRegex.AMOUNT_NEGATIVE)
    private String fraudsOthersPrior25ProvReq;

    @Pattern(regexp = PatternRegex.AMOUNT_POSITIVE)
    private String fraudsOthersDelayedProvAfter;
    @Pattern(regexp = PatternRegex.AMOUNT_POSITIVE)
    private String fraudsOthersDelayedWrite;
    @Pattern(regexp = PatternRegex.AMOUNT_POSITIVE)
    private String fraudsOthersDelayedAddition;
    @Pattern(regexp = PatternRegex.AMOUNT_POSITIVE)
    private String fraudsOthersDelayedReduction;
    @Pattern(regexp = PatternRegex.AMOUNT_NEGATIVE)
    private String fraudsOthersDelayedProvOn;
    @Pattern(regexp = PatternRegex.AMOUNT_POSITIVE)
    private String fraudsOthersDelayedRate;
    @Pattern(regexp = PatternRegex.AMOUNT_NEGATIVE)
    private String fraudsOthersDelayedProvReq;

    @Pattern(regexp = PatternRegex.AMOUNT_POSITIVE)
    private String revenueProvAfter;
    @Pattern(regexp = PatternRegex.AMOUNT_POSITIVE)
    private String revenueWrite;
    @Pattern(regexp = PatternRegex.AMOUNT_POSITIVE)
    private String revenueAddition;
    @Pattern(regexp = PatternRegex.AMOUNT_POSITIVE)
    private String revenueReduction;
    @Pattern(regexp = PatternRegex.AMOUNT_NEGATIVE)
    private String revenueProvOn;
    @Pattern(regexp = PatternRegex.AMOUNT_POSITIVE)
    private String revenueRate;
    @Pattern(regexp = PatternRegex.AMOUNT_NEGATIVE)
    private String revenueProvReq;

    @Pattern(regexp = PatternRegex.AMOUNT_POSITIVE)
    private String fsloProvAfter;
    @Pattern(regexp = PatternRegex.AMOUNT_POSITIVE)
    private String fsloWrite;
    @Pattern(regexp = PatternRegex.AMOUNT_POSITIVE)
    private String fsloAddition;
    @Pattern(regexp = PatternRegex.AMOUNT_POSITIVE)
    private String fsloReduction;
    @Pattern(regexp = PatternRegex.AMOUNT_NEGATIVE)
    private String fsloProvOn;
    @Pattern(regexp = PatternRegex.AMOUNT_POSITIVE)
    private String fsloRate;
    @Pattern(regexp = PatternRegex.AMOUNT_NEGATIVE)
    private String fsloProvReq;

    @Pattern(regexp = PatternRegex.AMOUNT_POSITIVE)
    private String outstandingProvAfter;
    @Pattern(regexp = PatternRegex.AMOUNT_POSITIVE)
    private String outstandingWrite;
    @Pattern(regexp = PatternRegex.AMOUNT_POSITIVE)
    private String outstandingAddition;
    @Pattern(regexp = PatternRegex.AMOUNT_POSITIVE)
    private String outstandingReduction;
    @Pattern(regexp = PatternRegex.AMOUNT_NEGATIVE)
    private String outstandingProvOn;
    @Pattern(regexp = PatternRegex.AMOUNT_POSITIVE)
    private String outstandingRate;
    @Pattern(regexp = PatternRegex.AMOUNT_NEGATIVE)
    private String outstandingProvReq;

    @Pattern(regexp = PatternRegex.AMOUNT_POSITIVE)
    private String npainterestProvAfter;
    @Pattern(regexp = PatternRegex.AMOUNT_POSITIVE)
    private String npainterestWrite;
    @Pattern(regexp = PatternRegex.AMOUNT_POSITIVE)
    private String npainterestAddition;
    @Pattern(regexp = PatternRegex.AMOUNT_POSITIVE)
    private String npainterestReduction;
    @Pattern(regexp = PatternRegex.AMOUNT_NEGATIVE)
    private String npainterestProvOn;
    @Pattern(regexp = PatternRegex.AMOUNT_POSITIVE)
    private String npainterestRate;
    @Pattern(regexp = PatternRegex.AMOUNT_NEGATIVE)
    private String npainterestProvReq;

    private String firstRow;
    private String secondRow;
    private String thirdRow;
    private String fourthRow;
    private String fifthRow;
    private String sixthRow;
    private String seventhRow;
    private String eightRow;
    private String ninethRow;
    private String tenthRow;
    private String elevenhtRow;
    private String twelvthRow;
    private String thirteenthRow;
    private String fourteenthRow;
    private String fifteenthRow;
    private String sixteenthRow;
    private String seventeenthRow;
    private String eighteenRow;


    private ArrayList ValidList = new ArrayList();


    private ArrayList inValidList = new ArrayList();


    public RW04() {
        super();
    }


    public void reset() {
        particulars = "";
        provAmt2015 = "";
        writeOffDur12mon = "";
        additionDur12mon = "";
        reduInProviAmt = "";
        proviAmt2016 = "";
        ratePOfProv = "";
        provReq = "";

        particularsList = "";
        provAmt2015List = "";
        writeOffDur12monList = "";
        additionDur12monList = "";
        reduInProviAmtList = "";
        proviAmt2016List = "";
        ratePOfProvList = "";
        provReqList = "";

        fraudsDebitedProvAfter = "";
        fraudsDebitedWrite = "";
        fraudsDebitedAddition = "";
        fraudsDebitedReduction = "";
        fraudsDebitedProvOn = "";
        fraudsDebitedRate = "";
        fraudsDebitedProvReq = "";

        fraudsDebitedPrior100ProvAfter = "";
        fraudsDebitedPrior100Write = "";
        fraudsDebitedPrior100Addition = "";
        fraudsDebitedPrior100Reduction = "";
        fraudsDebitedPrior100ProvOn = "";
        fraudsDebitedPrior100Rate = "";
        fraudsDebitedPrior100ProvReq = "";

        fraudsDebitedPrior75ProvAfter = "";
        fraudsDebitedPrior75Write = "";
        fraudsDebitedPrior75Addition = "";
        fraudsDebitedPrior75Reduction = "";
        fraudsDebitedPrior75ProvOn = "";
        fraudsDebitedPrior75Rate = "";
        fraudsDebitedPrior75ProvReq = "";

        fraudsDebitedPrior50ProvAfter = "";
        fraudsDebitedPrior50Write = "";
        fraudsDebitedPrior50Addition = "";
        fraudsDebitedPrior50Reduction = "";
        fraudsDebitedPrior50ProvOn = "";
        fraudsDebitedPrior50Rate = "";
        fraudsDebitedPrior50ProvReq = "";

        fraudsDebitedPrior25ProvAfter = "";
        fraudsDebitedPrior25Write = "";
        fraudsDebitedPrior25Addition = "";
        fraudsDebitedPrior25Reduction = "";
        fraudsDebitedPrior25ProvOn = "";
        fraudsDebitedPrior25Rate = "";
        fraudsDebitedPrior25ProvReq = "";

        fraudsDebitedDelayedProvAfter = "";
        fraudsDebitedDelayedWrite = "";
        fraudsDebitedDelayedAddition = "";
        fraudsDebitedDelayedReduction = "";
        fraudsDebitedDelayedProvOn = "";
        fraudsDebitedDelayedRate = "";
        fraudsDebitedDelayedProvReq = "";


        othersRecalledProvAfter = "";
        othersRecalledWrite = "";
        othersRecalledAddition = "";
        othersRecalledReduction = "";
        othersRecalledProvOn = "";
        othersRecalledRate = "";
        othersRecalledProvReq = "";


        fraudsOthersProvAfter = "";
        fraudsOthersWrite = "";
        fraudsOthersAddition = "";
        fraudsOthersReduction = "";
        fraudsOthersProvOn = "";
        fraudsOthersRate = "";
        fraudsOthersProvReq = "";

        fraudsOthersPrior100ProvAfter = "";
        fraudsOthersPrior100Write = "";
        fraudsOthersPrior100Addition = "";
        fraudsOthersPrior100Reduction = "";
        fraudsOthersPrior100ProvOn = "";
        fraudsOthersPrior100Rate = "";
        fraudsOthersPrior100ProvReq = "";

        fraudsOthersPrior75ProvAfter = "";
        fraudsOthersPrior75Write = "";
        fraudsOthersPrior75Addition = "";
        fraudsOthersPrior75Reduction = "";
        fraudsOthersPrior75ProvOn = "";
        fraudsOthersPrior75Rate = "";
        fraudsOthersPrior75ProvReq = "";

        fraudsOthersPrior50ProvAfter = "";
        fraudsOthersPrior50Write = "";
        fraudsOthersPrior50Addition = "";
        fraudsOthersPrior50Reduction = "";
        fraudsOthersPrior50ProvOn = "";
        fraudsOthersPrior50Rate = "";
        fraudsOthersPrior50ProvReq = "";

        fraudsOthersPrior25ProvAfter = "";
        fraudsOthersPrior25Write = "";
        fraudsOthersPrior25Addition = "";
        fraudsOthersPrior25Reduction = "";
        fraudsOthersPrior25ProvOn = "";
        fraudsOthersPrior25Rate = "";
        fraudsOthersPrior25ProvReq = "";

        fraudsOthersDelayedProvAfter = "";
        fraudsOthersDelayedWrite = "";
        fraudsOthersDelayedAddition = "";
        fraudsOthersDelayedReduction = "";
        fraudsOthersDelayedProvOn = "";
        fraudsOthersDelayedRate = "";
        fraudsOthersDelayedProvReq = "";


        revenueProvAfter = "";
        revenueWrite = "";
        revenueAddition = "";
        revenueReduction = "";
        revenueProvOn = "";
        revenueRate = "";
        revenueProvReq = "";

        fsloProvAfter = "";
        fsloWrite = "";
        fsloAddition = "";
        fsloReduction = "";
        fsloProvOn = "";
        fsloRate = "";
        fsloProvReq = "";

        outstandingProvAfter = "";
        outstandingWrite = "";
        outstandingAddition = "";
        outstandingReduction = "";
        outstandingProvOn = "";
        outstandingRate = "";
        outstandingProvReq = "";

        npainterestProvAfter = "";
        npainterestWrite = "";
        npainterestAddition = "";
        npainterestReduction = "";
        npainterestProvOn = "";
        npainterestRate = "";
        npainterestProvReq = "";


        faultyFlagList = new ArrayList();

        entryNo = "";
        strStatus = "";

        faultyflag = 0;


        particulars = "";
        provAmt2015 = "";
        writeOffDur12mon = "";
        additionDur12mon = "";
        reduInProviAmt = "";
        proviAmt2016 = "";
        ratePOfProv = "";
        provReq = "";


        quarter = "";
        financialYear = "";
        ValidList = new ArrayList();
        inValidList = new ArrayList();

    }

    public String getNpainterestProvAfter() {
        return npainterestProvAfter;
    }

    public void setNpainterestProvAfter(String npainterestProvAfter) {
        this.npainterestProvAfter = npainterestProvAfter;
    }

    public String getNpainterestWrite() {
        return npainterestWrite;
    }

    public void setNpainterestWrite(String npainterestWrite) {
        this.npainterestWrite = npainterestWrite;
    }

    public String getNpainterestAddition() {
        return npainterestAddition;
    }

    public void setNpainterestAddition(String npainterestAddition) {
        this.npainterestAddition = npainterestAddition;
    }

    public String getNpainterestReduction() {
        return npainterestReduction;
    }

    public void setNpainterestReduction(String npainterestReduction) {
        this.npainterestReduction = npainterestReduction;
    }

    public String getNpainterestProvOn() {
        return npainterestProvOn;
    }

    public void setNpainterestProvOn(String npainterestProvOn) {
        this.npainterestProvOn = npainterestProvOn;
    }

    public String getNpainterestRate() {
        return npainterestRate;
    }

    public void setNpainterestRate(String npainterestRate) {
        this.npainterestRate = npainterestRate;
    }

    public String getNpainterestProvReq() {
        return npainterestProvReq;
    }

    public void setNpainterestProvReq(String npainterestProvReq) {
        this.npainterestProvReq = npainterestProvReq;
    }

    public String getEighteenRow() {
        return eighteenRow;
    }

    public void setEighteenRow(String eighteenRow) {
        this.eighteenRow = eighteenRow;
    }

    public int getFaultyflag() {
        return faultyflag;
    }

    public void setFaultyflag(int faultyflag) {
        this.faultyflag = faultyflag;
    }

    public ArrayList getFaultyFlagList() {
        return faultyFlagList;
    }

    public void setFaultyFlagList(ArrayList faultyFlagList) {
        this.faultyFlagList = faultyFlagList;
    }

    public String getEntryNo() {
        return entryNo;
    }

    public void setEntryNo(String entryNo) {
        this.entryNo = entryNo;
    }

    public String getStrStatus() {
        return strStatus;
    }

    public void setStrStatus(String strStatus) {
        this.strStatus = strStatus;
    }


    public String getQuarter() {
        return quarter;
    }


    public void setQuarter(String quarter) {
        this.quarter = quarter;
    }


    public String getFinancialYear() {
        return financialYear;
    }


    public void setFinancialYear(String financialYear) {
        this.financialYear = financialYear;
    }


    public ArrayList getValidList() {
        return ValidList;
    }


    public void setValidList(ArrayList validList) {
        ValidList = validList;
    }


    public ArrayList getInValidList() {
        return inValidList;
    }


    public void setInValidList(ArrayList inValidList) {
        this.inValidList = inValidList;
    }


    public ArrayList getStrRuleList() {
        return strRuleList;
    }


    public void setStrRuleList(ArrayList strRuleList) {
        this.strRuleList = strRuleList;
    }


    public int getStrRuleListCount() {
        return strRuleListCount;
    }


    public void setStrRuleListCount(int strRuleListCount) {
        this.strRuleListCount = strRuleListCount;
    }


    public String getConsolidatedSel() {
        return consolidatedSel;
    }


    public void setConsolidatedSel(String consolidatedSel) {
        this.consolidatedSel = consolidatedSel;
    }


    public String getYear() {
        return year;
    }


    public void setYear(String year) {
        this.year = year;
    }


    public ArrayList getYearList() {
        return yearList;
    }


    public void setYearList(ArrayList yearList) {
        this.yearList = yearList;
    }


    public ArrayList getQuarterList() {
        return quarterList;
    }


    public void setQuarterList(ArrayList quarterList) {
        this.quarterList = quarterList;
    }


    public String getFilename() {
        return filename;
    }


    public void setFilename(String filename) {
        this.filename = filename;
    }


    public String getQuaterDate() {
        return quaterDate;
    }


    public void setQuaterDate(String quaterDate) {
        this.quaterDate = quaterDate;
    }


    public String getReportIdHidden() {
        return reportIdHidden;
    }


    public void setReportIdHidden(String reportIdHidden) {
        this.reportIdHidden = reportIdHidden;
    }


    public String getUpdateFlag() {
        return updateFlag;
    }


    public void setUpdateFlag(String updateFlag) {
        this.updateFlag = updateFlag;
    }


    public String getReportId() {
        return reportId;
    }


    public void setReportId(String reportId) {
        this.reportId = reportId;
    }


    public String getBranchCode() {
        return branchCode;
    }


    public void setBranchCode(String branchCode) {
        this.branchCode = branchCode;
    }


    public String getReportType() {
        return reportType;
    }


    public void setReportType(String reportType) {
        this.reportType = reportType;
    }


    public String getParamName() {
        return paramName;
    }


    public void setParamName(String paramName) {
        this.paramName = paramName;
    }


    public String getParamValue() {
        return paramValue;
    }


    public void setParamValue(String paramValue) {
        this.paramValue = paramValue;
    }


    public String getBranch() {
        return branch;
    }


    public void setBranch(String branch) {
        this.branch = branch;
    }


    public String getParticulars() {
        return particulars;
    }


    public void setParticulars(String particulars) {
        this.particulars = particulars;
    }


    public String getProvAmt2015() {
        return provAmt2015;
    }


    public void setProvAmt2015(String provAmt2015) {
        this.provAmt2015 = provAmt2015;
    }


    public String getWriteOffDur12mon() {
        return writeOffDur12mon;
    }


    public void setWriteOffDur12mon(String writeOffDur12mon) {
        this.writeOffDur12mon = writeOffDur12mon;
    }


    public String getAdditionDur12mon() {
        return additionDur12mon;
    }


    public void setAdditionDur12mon(String additionDur12mon) {
        this.additionDur12mon = additionDur12mon;
    }


    public String getReduInProviAmt() {
        return reduInProviAmt;
    }


    public void setReduInProviAmt(String reduInProviAmt) {
        this.reduInProviAmt = reduInProviAmt;
    }


    public String getProviAmt2016() {
        return proviAmt2016;
    }


    public void setProviAmt2016(String proviAmt2016) {
        this.proviAmt2016 = proviAmt2016;
    }


    public String getRatePOfProv() {
        return ratePOfProv;
    }


    public void setRatePOfProv(String ratePOfProv) {
        this.ratePOfProv = ratePOfProv;
    }


    public String getProvReq() {
        return provReq;
    }


    public void setProvReq(String provReq) {
        this.provReq = provReq;
    }


    public String getParticularsList() {
        return particularsList;
    }


    public void setParticularsList(String particularsList) {
        this.particularsList = particularsList;
    }


    public String getProvAmt2015List() {
        return provAmt2015List;
    }


    public void setProvAmt2015List(String provAmt2015List) {
        this.provAmt2015List = provAmt2015List;
    }


    public String getWriteOffDur12monList() {
        return writeOffDur12monList;
    }


    public void setWriteOffDur12monList(String writeOffDur12monList) {
        this.writeOffDur12monList = writeOffDur12monList;
    }


    public String getAdditionDur12monList() {
        return additionDur12monList;
    }


    public void setAdditionDur12monList(String additionDur12monList) {
        this.additionDur12monList = additionDur12monList;
    }


    public String getReduInProviAmtList() {
        return reduInProviAmtList;
    }


    public void setReduInProviAmtList(String reduInProviAmtList) {
        this.reduInProviAmtList = reduInProviAmtList;
    }


    public String getProviAmt2016List() {
        return proviAmt2016List;
    }


    public void setProviAmt2016List(String proviAmt2016List) {
        this.proviAmt2016List = proviAmt2016List;
    }


    public String getRatePOfProvList() {
        return ratePOfProvList;
    }


    public void setRatePOfProvList(String ratePOfProvList) {
        this.ratePOfProvList = ratePOfProvList;
    }


    public String getProvReqList() {
        return provReqList;
    }


    public void setProvReqList(String provReqList) {
        this.provReqList = provReqList;
    }


    public String getFraudsDebitedProvAfter() {
        return fraudsDebitedProvAfter;
    }


    public void setFraudsDebitedProvAfter(String fraudsDebitedProvAfter) {
        this.fraudsDebitedProvAfter = fraudsDebitedProvAfter;
    }


    public String getFraudsDebitedWrite() {
        return fraudsDebitedWrite;
    }


    public void setFraudsDebitedWrite(String fraudsDebitedWrite) {
        this.fraudsDebitedWrite = fraudsDebitedWrite;
    }


    public String getFraudsDebitedAddition() {
        return fraudsDebitedAddition;
    }


    public void setFraudsDebitedAddition(String fraudsDebitedAddition) {
        this.fraudsDebitedAddition = fraudsDebitedAddition;
    }


    public String getFraudsDebitedReduction() {
        return fraudsDebitedReduction;
    }


    public void setFraudsDebitedReduction(String fraudsDebitedReduction) {
        this.fraudsDebitedReduction = fraudsDebitedReduction;
    }


    public String getFraudsDebitedProvOn() {
        return fraudsDebitedProvOn;
    }


    public void setFraudsDebitedProvOn(String fraudsDebitedProvOn) {
        this.fraudsDebitedProvOn = fraudsDebitedProvOn;
    }


    public String getFraudsDebitedRate() {
        return fraudsDebitedRate;
    }


    public void setFraudsDebitedRate(String fraudsDebitedRate) {
        this.fraudsDebitedRate = fraudsDebitedRate;
    }


    public String getFraudsDebitedProvReq() {
        return fraudsDebitedProvReq;
    }


    public void setFraudsDebitedProvReq(String fraudsDebitedProvReq) {
        this.fraudsDebitedProvReq = fraudsDebitedProvReq;
    }


    public String getFraudsDebitedPrior100ProvAfter() {
        return fraudsDebitedPrior100ProvAfter;
    }


    public void setFraudsDebitedPrior100ProvAfter(
            String fraudsDebitedPrior100ProvAfter) {
        this.fraudsDebitedPrior100ProvAfter = fraudsDebitedPrior100ProvAfter;
    }


    public String getFraudsDebitedPrior100Write() {
        return fraudsDebitedPrior100Write;
    }


    public void setFraudsDebitedPrior100Write(String fraudsDebitedPrior100Write) {
        this.fraudsDebitedPrior100Write = fraudsDebitedPrior100Write;
    }


    public String getFraudsDebitedPrior100Addition() {
        return fraudsDebitedPrior100Addition;
    }


    public void setFraudsDebitedPrior100Addition(
            String fraudsDebitedPrior100Addition) {
        this.fraudsDebitedPrior100Addition = fraudsDebitedPrior100Addition;
    }


    public String getFraudsDebitedPrior100Reduction() {
        return fraudsDebitedPrior100Reduction;
    }


    public void setFraudsDebitedPrior100Reduction(
            String fraudsDebitedPrior100Reduction) {
        this.fraudsDebitedPrior100Reduction = fraudsDebitedPrior100Reduction;
    }


    public String getFraudsDebitedPrior100ProvOn() {
        return fraudsDebitedPrior100ProvOn;
    }


    public void setFraudsDebitedPrior100ProvOn(String fraudsDebitedPrior100ProvOn) {
        this.fraudsDebitedPrior100ProvOn = fraudsDebitedPrior100ProvOn;
    }


    public String getFraudsDebitedPrior100Rate() {
        return fraudsDebitedPrior100Rate;
    }


    public void setFraudsDebitedPrior100Rate(String fraudsDebitedPrior100Rate) {
        this.fraudsDebitedPrior100Rate = fraudsDebitedPrior100Rate;
    }


    public String getFraudsDebitedPrior100ProvReq() {
        return fraudsDebitedPrior100ProvReq;
    }


    public void setFraudsDebitedPrior100ProvReq(String fraudsDebitedPrior100ProvReq) {
        this.fraudsDebitedPrior100ProvReq = fraudsDebitedPrior100ProvReq;
    }


    public String getFraudsDebitedPrior75ProvAfter() {
        return fraudsDebitedPrior75ProvAfter;
    }


    public void setFraudsDebitedPrior75ProvAfter(
            String fraudsDebitedPrior75ProvAfter) {
        this.fraudsDebitedPrior75ProvAfter = fraudsDebitedPrior75ProvAfter;
    }


    public String getFraudsDebitedPrior75Write() {
        return fraudsDebitedPrior75Write;
    }


    public void setFraudsDebitedPrior75Write(String fraudsDebitedPrior75Write) {
        this.fraudsDebitedPrior75Write = fraudsDebitedPrior75Write;
    }


    public String getFraudsDebitedPrior75Addition() {
        return fraudsDebitedPrior75Addition;
    }


    public void setFraudsDebitedPrior75Addition(String fraudsDebitedPrior75Addition) {
        this.fraudsDebitedPrior75Addition = fraudsDebitedPrior75Addition;
    }


    public String getFraudsDebitedPrior75Reduction() {
        return fraudsDebitedPrior75Reduction;
    }


    public void setFraudsDebitedPrior75Reduction(
            String fraudsDebitedPrior75Reduction) {
        this.fraudsDebitedPrior75Reduction = fraudsDebitedPrior75Reduction;
    }


    public String getFraudsDebitedPrior75ProvOn() {
        return fraudsDebitedPrior75ProvOn;
    }


    public void setFraudsDebitedPrior75ProvOn(String fraudsDebitedPrior75ProvOn) {
        this.fraudsDebitedPrior75ProvOn = fraudsDebitedPrior75ProvOn;
    }


    public String getFraudsDebitedPrior75Rate() {
        return fraudsDebitedPrior75Rate;
    }


    public void setFraudsDebitedPrior75Rate(String fraudsDebitedPrior75Rate) {
        this.fraudsDebitedPrior75Rate = fraudsDebitedPrior75Rate;
    }


    public String getFraudsDebitedPrior75ProvReq() {
        return fraudsDebitedPrior75ProvReq;
    }


    public void setFraudsDebitedPrior75ProvReq(String fraudsDebitedPrior75ProvReq) {
        this.fraudsDebitedPrior75ProvReq = fraudsDebitedPrior75ProvReq;
    }


    public String getFraudsDebitedPrior50ProvAfter() {
        return fraudsDebitedPrior50ProvAfter;
    }


    public void setFraudsDebitedPrior50ProvAfter(
            String fraudsDebitedPrior50ProvAfter) {
        this.fraudsDebitedPrior50ProvAfter = fraudsDebitedPrior50ProvAfter;
    }


    public String getFraudsDebitedPrior50Write() {
        return fraudsDebitedPrior50Write;
    }


    public void setFraudsDebitedPrior50Write(String fraudsDebitedPrior50Write) {
        this.fraudsDebitedPrior50Write = fraudsDebitedPrior50Write;
    }


    public String getFraudsDebitedPrior50Addition() {
        return fraudsDebitedPrior50Addition;
    }


    public void setFraudsDebitedPrior50Addition(String fraudsDebitedPrior50Addition) {
        this.fraudsDebitedPrior50Addition = fraudsDebitedPrior50Addition;
    }


    public String getFraudsDebitedPrior50Reduction() {
        return fraudsDebitedPrior50Reduction;
    }


    public void setFraudsDebitedPrior50Reduction(
            String fraudsDebitedPrior50Reduction) {
        this.fraudsDebitedPrior50Reduction = fraudsDebitedPrior50Reduction;
    }


    public String getFraudsDebitedPrior50ProvOn() {
        return fraudsDebitedPrior50ProvOn;
    }


    public void setFraudsDebitedPrior50ProvOn(String fraudsDebitedPrior50ProvOn) {
        this.fraudsDebitedPrior50ProvOn = fraudsDebitedPrior50ProvOn;
    }


    public String getFraudsDebitedPrior50Rate() {
        return fraudsDebitedPrior50Rate;
    }


    public void setFraudsDebitedPrior50Rate(String fraudsDebitedPrior50Rate) {
        this.fraudsDebitedPrior50Rate = fraudsDebitedPrior50Rate;
    }


    public String getFraudsDebitedPrior50ProvReq() {
        return fraudsDebitedPrior50ProvReq;
    }


    public void setFraudsDebitedPrior50ProvReq(String fraudsDebitedPrior50ProvReq) {
        this.fraudsDebitedPrior50ProvReq = fraudsDebitedPrior50ProvReq;
    }


    public String getFraudsDebitedPrior25ProvAfter() {
        return fraudsDebitedPrior25ProvAfter;
    }


    public void setFraudsDebitedPrior25ProvAfter(
            String fraudsDebitedPrior25ProvAfter) {
        this.fraudsDebitedPrior25ProvAfter = fraudsDebitedPrior25ProvAfter;
    }


    public String getFraudsDebitedPrior25Write() {
        return fraudsDebitedPrior25Write;
    }


    public void setFraudsDebitedPrior25Write(String fraudsDebitedPrior25Write) {
        this.fraudsDebitedPrior25Write = fraudsDebitedPrior25Write;
    }


    public String getFraudsDebitedPrior25Addition() {
        return fraudsDebitedPrior25Addition;
    }


    public void setFraudsDebitedPrior25Addition(String fraudsDebitedPrior25Addition) {
        this.fraudsDebitedPrior25Addition = fraudsDebitedPrior25Addition;
    }


    public String getFraudsDebitedPrior25Reduction() {
        return fraudsDebitedPrior25Reduction;
    }


    public void setFraudsDebitedPrior25Reduction(
            String fraudsDebitedPrior25Reduction) {
        this.fraudsDebitedPrior25Reduction = fraudsDebitedPrior25Reduction;
    }


    public String getFraudsDebitedPrior25ProvOn() {
        return fraudsDebitedPrior25ProvOn;
    }


    public void setFraudsDebitedPrior25ProvOn(String fraudsDebitedPrior25ProvOn) {
        this.fraudsDebitedPrior25ProvOn = fraudsDebitedPrior25ProvOn;
    }


    public String getFraudsDebitedPrior25Rate() {
        return fraudsDebitedPrior25Rate;
    }


    public void setFraudsDebitedPrior25Rate(String fraudsDebitedPrior25Rate) {
        this.fraudsDebitedPrior25Rate = fraudsDebitedPrior25Rate;
    }


    public String getFraudsDebitedPrior25ProvReq() {
        return fraudsDebitedPrior25ProvReq;
    }


    public void setFraudsDebitedPrior25ProvReq(String fraudsDebitedPrior25ProvReq) {
        this.fraudsDebitedPrior25ProvReq = fraudsDebitedPrior25ProvReq;
    }


    public String getFraudsDebitedDelayedProvAfter() {
        return fraudsDebitedDelayedProvAfter;
    }


    public void setFraudsDebitedDelayedProvAfter(
            String fraudsDebitedDelayedProvAfter) {
        this.fraudsDebitedDelayedProvAfter = fraudsDebitedDelayedProvAfter;
    }


    public String getFraudsDebitedDelayedWrite() {
        return fraudsDebitedDelayedWrite;
    }


    public void setFraudsDebitedDelayedWrite(String fraudsDebitedDelayedWrite) {
        this.fraudsDebitedDelayedWrite = fraudsDebitedDelayedWrite;
    }


    public String getFraudsDebitedDelayedAddition() {
        return fraudsDebitedDelayedAddition;
    }


    public void setFraudsDebitedDelayedAddition(String fraudsDebitedDelayedAddition) {
        this.fraudsDebitedDelayedAddition = fraudsDebitedDelayedAddition;
    }


    public String getFraudsDebitedDelayedReduction() {
        return fraudsDebitedDelayedReduction;
    }


    public void setFraudsDebitedDelayedReduction(
            String fraudsDebitedDelayedReduction) {
        this.fraudsDebitedDelayedReduction = fraudsDebitedDelayedReduction;
    }


    public String getFraudsDebitedDelayedProvOn() {
        return fraudsDebitedDelayedProvOn;
    }


    public void setFraudsDebitedDelayedProvOn(String fraudsDebitedDelayedProvOn) {
        this.fraudsDebitedDelayedProvOn = fraudsDebitedDelayedProvOn;
    }


    public String getFraudsDebitedDelayedRate() {
        return fraudsDebitedDelayedRate;
    }


    public void setFraudsDebitedDelayedRate(String fraudsDebitedDelayedRate) {
        this.fraudsDebitedDelayedRate = fraudsDebitedDelayedRate;
    }


    public String getFraudsDebitedDelayedProvReq() {
        return fraudsDebitedDelayedProvReq;
    }


    public void setFraudsDebitedDelayedProvReq(String fraudsDebitedDelayedProvReq) {
        this.fraudsDebitedDelayedProvReq = fraudsDebitedDelayedProvReq;
    }


    public String getFraudsOthersProvAfter() {
        return fraudsOthersProvAfter;
    }


    public void setFraudsOthersProvAfter(String fraudsOthersProvAfter) {
        this.fraudsOthersProvAfter = fraudsOthersProvAfter;
    }


    public String getFraudsOthersWrite() {
        return fraudsOthersWrite;
    }


    public void setFraudsOthersWrite(String fraudsOthersWrite) {
        this.fraudsOthersWrite = fraudsOthersWrite;
    }


    public String getFraudsOthersAddition() {
        return fraudsOthersAddition;
    }


    public void setFraudsOthersAddition(String fraudsOthersAddition) {
        this.fraudsOthersAddition = fraudsOthersAddition;
    }


    public String getFraudsOthersReduction() {
        return fraudsOthersReduction;
    }


    public void setFraudsOthersReduction(String fraudsOthersReduction) {
        this.fraudsOthersReduction = fraudsOthersReduction;
    }


    public String getFraudsOthersProvOn() {
        return fraudsOthersProvOn;
    }


    public void setFraudsOthersProvOn(String fraudsOthersProvOn) {
        this.fraudsOthersProvOn = fraudsOthersProvOn;
    }


    public String getFraudsOthersRate() {
        return fraudsOthersRate;
    }


    public void setFraudsOthersRate(String fraudsOthersRate) {
        this.fraudsOthersRate = fraudsOthersRate;
    }


    public String getFraudsOthersProvReq() {
        return fraudsOthersProvReq;
    }


    public void setFraudsOthersProvReq(String fraudsOthersProvReq) {
        this.fraudsOthersProvReq = fraudsOthersProvReq;
    }


    public String getFraudsOthersPrior100ProvAfter() {
        return fraudsOthersPrior100ProvAfter;
    }


    public void setFraudsOthersPrior100ProvAfter(
            String fraudsOthersPrior100ProvAfter) {
        this.fraudsOthersPrior100ProvAfter = fraudsOthersPrior100ProvAfter;
    }


    public String getFraudsOthersPrior100Write() {
        return fraudsOthersPrior100Write;
    }


    public void setFraudsOthersPrior100Write(String fraudsOthersPrior100Write) {
        this.fraudsOthersPrior100Write = fraudsOthersPrior100Write;
    }


    public String getFraudsOthersPrior100Addition() {
        return fraudsOthersPrior100Addition;
    }


    public void setFraudsOthersPrior100Addition(String fraudsOthersPrior100Addition) {
        this.fraudsOthersPrior100Addition = fraudsOthersPrior100Addition;
    }


    public String getFraudsOthersPrior100Reduction() {
        return fraudsOthersPrior100Reduction;
    }


    public void setFraudsOthersPrior100Reduction(
            String fraudsOthersPrior100Reduction) {
        this.fraudsOthersPrior100Reduction = fraudsOthersPrior100Reduction;
    }


    public String getFraudsOthersPrior100ProvOn() {
        return fraudsOthersPrior100ProvOn;
    }


    public void setFraudsOthersPrior100ProvOn(String fraudsOthersPrior100ProvOn) {
        this.fraudsOthersPrior100ProvOn = fraudsOthersPrior100ProvOn;
    }


    public String getFraudsOthersPrior100Rate() {
        return fraudsOthersPrior100Rate;
    }


    public void setFraudsOthersPrior100Rate(String fraudsOthersPrior100Rate) {
        this.fraudsOthersPrior100Rate = fraudsOthersPrior100Rate;
    }


    public String getFraudsOthersPrior100ProvReq() {
        return fraudsOthersPrior100ProvReq;
    }


    public void setFraudsOthersPrior100ProvReq(String fraudsOthersPrior100ProvReq) {
        this.fraudsOthersPrior100ProvReq = fraudsOthersPrior100ProvReq;
    }


    public String getFraudsOthersPrior75ProvAfter() {
        return fraudsOthersPrior75ProvAfter;
    }


    public void setFraudsOthersPrior75ProvAfter(String fraudsOthersPrior75ProvAfter) {
        this.fraudsOthersPrior75ProvAfter = fraudsOthersPrior75ProvAfter;
    }


    public String getFraudsOthersPrior75Write() {
        return fraudsOthersPrior75Write;
    }


    public void setFraudsOthersPrior75Write(String fraudsOthersPrior75Write) {
        this.fraudsOthersPrior75Write = fraudsOthersPrior75Write;
    }


    public String getFraudsOthersPrior75Addition() {
        return fraudsOthersPrior75Addition;
    }


    public void setFraudsOthersPrior75Addition(String fraudsOthersPrior75Addition) {
        this.fraudsOthersPrior75Addition = fraudsOthersPrior75Addition;
    }


    public String getFraudsOthersPrior75Reduction() {
        return fraudsOthersPrior75Reduction;
    }


    public void setFraudsOthersPrior75Reduction(String fraudsOthersPrior75Reduction) {
        this.fraudsOthersPrior75Reduction = fraudsOthersPrior75Reduction;
    }


    public String getFraudsOthersPrior75ProvOn() {
        return fraudsOthersPrior75ProvOn;
    }


    public void setFraudsOthersPrior75ProvOn(String fraudsOthersPrior75ProvOn) {
        this.fraudsOthersPrior75ProvOn = fraudsOthersPrior75ProvOn;
    }


    public String getFraudsOthersPrior75Rate() {
        return fraudsOthersPrior75Rate;
    }


    public void setFraudsOthersPrior75Rate(String fraudsOthersPrior75Rate) {
        this.fraudsOthersPrior75Rate = fraudsOthersPrior75Rate;
    }


    public String getFraudsOthersPrior75ProvReq() {
        return fraudsOthersPrior75ProvReq;
    }


    public void setFraudsOthersPrior75ProvReq(String fraudsOthersPrior75ProvReq) {
        this.fraudsOthersPrior75ProvReq = fraudsOthersPrior75ProvReq;
    }


    public String getFraudsOthersPrior50ProvAfter() {
        return fraudsOthersPrior50ProvAfter;
    }


    public void setFraudsOthersPrior50ProvAfter(String fraudsOthersPrior50ProvAfter) {
        this.fraudsOthersPrior50ProvAfter = fraudsOthersPrior50ProvAfter;
    }


    public String getFraudsOthersPrior50Write() {
        return fraudsOthersPrior50Write;
    }


    public void setFraudsOthersPrior50Write(String fraudsOthersPrior50Write) {
        this.fraudsOthersPrior50Write = fraudsOthersPrior50Write;
    }


    public String getFraudsOthersPrior50Addition() {
        return fraudsOthersPrior50Addition;
    }


    public void setFraudsOthersPrior50Addition(String fraudsOthersPrior50Addition) {
        this.fraudsOthersPrior50Addition = fraudsOthersPrior50Addition;
    }


    public String getFraudsOthersPrior50Reduction() {
        return fraudsOthersPrior50Reduction;
    }


    public void setFraudsOthersPrior50Reduction(String fraudsOthersPrior50Reduction) {
        this.fraudsOthersPrior50Reduction = fraudsOthersPrior50Reduction;
    }


    public String getFraudsOthersPrior50ProvOn() {
        return fraudsOthersPrior50ProvOn;
    }


    public void setFraudsOthersPrior50ProvOn(String fraudsOthersPrior50ProvOn) {
        this.fraudsOthersPrior50ProvOn = fraudsOthersPrior50ProvOn;
    }


    public String getFraudsOthersPrior50Rate() {
        return fraudsOthersPrior50Rate;
    }


    public void setFraudsOthersPrior50Rate(String fraudsOthersPrior50Rate) {
        this.fraudsOthersPrior50Rate = fraudsOthersPrior50Rate;
    }


    public String getFraudsOthersPrior50ProvReq() {
        return fraudsOthersPrior50ProvReq;
    }


    public void setFraudsOthersPrior50ProvReq(String fraudsOthersPrior50ProvReq) {
        this.fraudsOthersPrior50ProvReq = fraudsOthersPrior50ProvReq;
    }


    public String getFraudsOthersPrior25ProvAfter() {
        return fraudsOthersPrior25ProvAfter;
    }


    public void setFraudsOthersPrior25ProvAfter(String fraudsOthersPrior25ProvAfter) {
        this.fraudsOthersPrior25ProvAfter = fraudsOthersPrior25ProvAfter;
    }


    public String getFraudsOthersPrior25Write() {
        return fraudsOthersPrior25Write;
    }


    public void setFraudsOthersPrior25Write(String fraudsOthersPrior25Write) {
        this.fraudsOthersPrior25Write = fraudsOthersPrior25Write;
    }


    public String getFraudsOthersPrior25Addition() {
        return fraudsOthersPrior25Addition;
    }


    public void setFraudsOthersPrior25Addition(String fraudsOthersPrior25Addition) {
        this.fraudsOthersPrior25Addition = fraudsOthersPrior25Addition;
    }


    public String getFraudsOthersPrior25Reduction() {
        return fraudsOthersPrior25Reduction;
    }


    public void setFraudsOthersPrior25Reduction(String fraudsOthersPrior25Reduction) {
        this.fraudsOthersPrior25Reduction = fraudsOthersPrior25Reduction;
    }


    public String getFraudsOthersPrior25ProvOn() {
        return fraudsOthersPrior25ProvOn;
    }


    public void setFraudsOthersPrior25ProvOn(String fraudsOthersPrior25ProvOn) {
        this.fraudsOthersPrior25ProvOn = fraudsOthersPrior25ProvOn;
    }


    public String getFraudsOthersPrior25Rate() {
        return fraudsOthersPrior25Rate;
    }


    public void setFraudsOthersPrior25Rate(String fraudsOthersPrior25Rate) {
        this.fraudsOthersPrior25Rate = fraudsOthersPrior25Rate;
    }


    public String getFraudsOthersPrior25ProvReq() {
        return fraudsOthersPrior25ProvReq;
    }


    public void setFraudsOthersPrior25ProvReq(String fraudsOthersPrior25ProvReq) {
        this.fraudsOthersPrior25ProvReq = fraudsOthersPrior25ProvReq;
    }


    public String getFraudsOthersDelayedProvAfter() {
        return fraudsOthersDelayedProvAfter;
    }


    public void setFraudsOthersDelayedProvAfter(String fraudsOthersDelayedProvAfter) {
        this.fraudsOthersDelayedProvAfter = fraudsOthersDelayedProvAfter;
    }


    public String getFraudsOthersDelayedWrite() {
        return fraudsOthersDelayedWrite;
    }


    public void setFraudsOthersDelayedWrite(String fraudsOthersDelayedWrite) {
        this.fraudsOthersDelayedWrite = fraudsOthersDelayedWrite;
    }


    public String getFraudsOthersDelayedAddition() {
        return fraudsOthersDelayedAddition;
    }


    public void setFraudsOthersDelayedAddition(String fraudsOthersDelayedAddition) {
        this.fraudsOthersDelayedAddition = fraudsOthersDelayedAddition;
    }


    public String getFraudsOthersDelayedReduction() {
        return fraudsOthersDelayedReduction;
    }


    public void setFraudsOthersDelayedReduction(String fraudsOthersDelayedReduction) {
        this.fraudsOthersDelayedReduction = fraudsOthersDelayedReduction;
    }


    public String getFraudsOthersDelayedProvOn() {
        return fraudsOthersDelayedProvOn;
    }


    public void setFraudsOthersDelayedProvOn(String fraudsOthersDelayedProvOn) {
        this.fraudsOthersDelayedProvOn = fraudsOthersDelayedProvOn;
    }


    public String getFraudsOthersDelayedRate() {
        return fraudsOthersDelayedRate;
    }


    public void setFraudsOthersDelayedRate(String fraudsOthersDelayedRate) {
        this.fraudsOthersDelayedRate = fraudsOthersDelayedRate;
    }


    public String getFraudsOthersDelayedProvReq() {
        return fraudsOthersDelayedProvReq;
    }


    public void setFraudsOthersDelayedProvReq(String fraudsOthersDelayedProvReq) {
        this.fraudsOthersDelayedProvReq = fraudsOthersDelayedProvReq;
    }


    public String getOthersRecalledProvAfter() {
        return othersRecalledProvAfter;
    }


    public void setOthersRecalledProvAfter(String othersRecalledProvAfter) {
        this.othersRecalledProvAfter = othersRecalledProvAfter;
    }


    public String getOthersRecalledWrite() {
        return othersRecalledWrite;
    }


    public void setOthersRecalledWrite(String othersRecalledWrite) {
        this.othersRecalledWrite = othersRecalledWrite;
    }


    public String getOthersRecalledAddition() {
        return othersRecalledAddition;
    }


    public void setOthersRecalledAddition(String othersRecalledAddition) {
        this.othersRecalledAddition = othersRecalledAddition;
    }


    public String getOthersRecalledReduction() {
        return othersRecalledReduction;
    }


    public void setOthersRecalledReduction(String othersRecalledReduction) {
        this.othersRecalledReduction = othersRecalledReduction;
    }


    public String getOthersRecalledProvOn() {
        return othersRecalledProvOn;
    }


    public void setOthersRecalledProvOn(String othersRecalledProvOn) {
        this.othersRecalledProvOn = othersRecalledProvOn;
    }


    public String getOthersRecalledRate() {
        return othersRecalledRate;
    }


    public void setOthersRecalledRate(String othersRecalledRate) {
        this.othersRecalledRate = othersRecalledRate;
    }


    public String getOthersRecalledProvReq() {
        return othersRecalledProvReq;
    }


    public void setOthersRecalledProvReq(String othersRecalledProvReq) {
        this.othersRecalledProvReq = othersRecalledProvReq;
    }


    public String getRevenueProvAfter() {
        return revenueProvAfter;
    }


    public void setRevenueProvAfter(String revenueProvAfter) {
        this.revenueProvAfter = revenueProvAfter;
    }


    public String getRevenueWrite() {
        return revenueWrite;
    }


    public void setRevenueWrite(String revenueWrite) {
        this.revenueWrite = revenueWrite;
    }


    public String getRevenueAddition() {
        return revenueAddition;
    }


    public void setRevenueAddition(String revenueAddition) {
        this.revenueAddition = revenueAddition;
    }


    public String getRevenueReduction() {
        return revenueReduction;
    }


    public void setRevenueReduction(String revenueReduction) {
        this.revenueReduction = revenueReduction;
    }


    public String getRevenueProvOn() {
        return revenueProvOn;
    }


    public void setRevenueProvOn(String revenueProvOn) {
        this.revenueProvOn = revenueProvOn;
    }


    public String getRevenueRate() {
        return revenueRate;
    }


    public void setRevenueRate(String revenueRate) {
        this.revenueRate = revenueRate;
    }


    public String getRevenueProvReq() {
        return revenueProvReq;
    }


    public void setRevenueProvReq(String revenueProvReq) {
        this.revenueProvReq = revenueProvReq;
    }


    public String getFsloProvAfter() {
        return fsloProvAfter;
    }


    public void setFsloProvAfter(String fsloProvAfter) {
        this.fsloProvAfter = fsloProvAfter;
    }


    public String getFsloWrite() {
        return fsloWrite;
    }


    public void setFsloWrite(String fsloWrite) {
        this.fsloWrite = fsloWrite;
    }


    public String getFsloAddition() {
        return fsloAddition;
    }


    public void setFsloAddition(String fsloAddition) {
        this.fsloAddition = fsloAddition;
    }


    public String getFsloReduction() {
        return fsloReduction;
    }


    public void setFsloReduction(String fsloReduction) {
        this.fsloReduction = fsloReduction;
    }


    public String getFsloProvOn() {
        return fsloProvOn;
    }


    public void setFsloProvOn(String fsloProvOn) {
        this.fsloProvOn = fsloProvOn;
    }


    public String getFsloRate() {
        return fsloRate;
    }


    public void setFsloRate(String fsloRate) {
        this.fsloRate = fsloRate;
    }


    public String getFsloProvReq() {
        return fsloProvReq;
    }


    public void setFsloProvReq(String fsloProvReq) {
        this.fsloProvReq = fsloProvReq;
    }


    public String getOutstandingProvAfter() {
        return outstandingProvAfter;
    }


    public void setOutstandingProvAfter(String outstandingProvAfter) {
        this.outstandingProvAfter = outstandingProvAfter;
    }


    public String getOutstandingWrite() {
        return outstandingWrite;
    }


    public void setOutstandingWrite(String outstandingWrite) {
        this.outstandingWrite = outstandingWrite;
    }


    public String getOutstandingAddition() {
        return outstandingAddition;
    }


    public void setOutstandingAddition(String outstandingAddition) {
        this.outstandingAddition = outstandingAddition;
    }


    public String getOutstandingReduction() {
        return outstandingReduction;
    }


    public void setOutstandingReduction(String outstandingReduction) {
        this.outstandingReduction = outstandingReduction;
    }


    public String getOutstandingProvOn() {
        return outstandingProvOn;
    }


    public void setOutstandingProvOn(String outstandingProvOn) {
        this.outstandingProvOn = outstandingProvOn;
    }


    public String getOutstandingRate() {
        return outstandingRate;
    }


    public void setOutstandingRate(String outstandingRate) {
        this.outstandingRate = outstandingRate;
    }


    public String getOutstandingProvReq() {
        return outstandingProvReq;
    }


    public void setOutstandingProvReq(String outstandingProvReq) {
        this.outstandingProvReq = outstandingProvReq;
    }

    public String getFirstRow() {
        return firstRow;
    }

    public void setFirstRow(String firstRow) {
        this.firstRow = firstRow;
    }


    public String getSecondRow() {
        return secondRow;
    }


    public void setSecondRow(String secondRow) {
        this.secondRow = secondRow;
    }


    public String getThirdRow() {
        return thirdRow;
    }


    public void setThirdRow(String thirdRow) {
        this.thirdRow = thirdRow;
    }


    public String getFourthRow() {
        return fourthRow;
    }


    public void setFourthRow(String fourthRow) {
        this.fourthRow = fourthRow;
    }


    public String getSixthRow() {
        return sixthRow;
    }


    public void setSixthRow(String sixthRow) {
        this.sixthRow = sixthRow;
    }


    public String getSeventhRow() {
        return seventhRow;
    }


    public void setSeventhRow(String seventhRow) {
        this.seventhRow = seventhRow;
    }


    public String getEightRow() {
        return eightRow;
    }


    public void setEightRow(String eightRow) {
        this.eightRow = eightRow;
    }


    public String getNinethRow() {
        return ninethRow;
    }


    public void setNinethRow(String ninethRow) {
        this.ninethRow = ninethRow;
    }


    public String getFifthRow() {
        return fifthRow;
    }


    public void setFifthRow(String fifthRow) {
        this.fifthRow = fifthRow;
    }


    public String getTenthRow() {
        return tenthRow;
    }


    public void setTenthRow(String tenthRow) {
        this.tenthRow = tenthRow;
    }


    public String getElevenhtRow() {
        return elevenhtRow;
    }


    public void setElevenhtRow(String elevenhtRow) {
        this.elevenhtRow = elevenhtRow;
    }


    public String getTwelvthRow() {
        return twelvthRow;
    }


    public void setTwelvthRow(String twelvthRow) {
        this.twelvthRow = twelvthRow;
    }


    public String getThirteenthRow() {
        return thirteenthRow;
    }


    public void setThirteenthRow(String thirteenthRow) {
        this.thirteenthRow = thirteenthRow;
    }


    public String getFourteenthRow() {
        return fourteenthRow;
    }


    public void setFourteenthRow(String fourteenthRow) {
        this.fourteenthRow = fourteenthRow;
    }


    public String getFifteenthRow() {
        return fifteenthRow;
    }


    public void setFifteenthRow(String fifteenthRow) {
        this.fifteenthRow = fifteenthRow;
    }


    public String getSixteenthRow() {
        return sixteenthRow;
    }


    public void setSixteenthRow(String sixteenthRow) {
        this.sixteenthRow = sixteenthRow;
    }


    public String getSeventeenthRow() {
        return seventeenthRow;
    }


    public void setSeventeenthRow(String seventeenthRow) {
        this.seventeenthRow = seventeenthRow;
    }

    public String getReportMasterId() {
        return reportMasterId;
    }

    public void setReportMasterId(String reportMasterId) {
        this.reportMasterId = reportMasterId;
    }

    public String getReportName() {
        return reportName;
    }

    public void setReportName(String reportName) {
        this.reportName = reportName;
    }

    public String getDataFlag() {
        return dataFlag;
    }

    public void setDataFlag(String dataFlag) {
        this.dataFlag = dataFlag;
    }
    public String getAction() {
        return action;
    }

    public void setAction(String action) {
        this.action = action;
    }
    public String getNilFlag() {
        return nilFlag;
    }

    public void setNilFlag(String nilFlag) {
        this.nilFlag = nilFlag;
    }
}

package com.tcs.controllers;

import com.tcs.beans.*;
import com.tcs.beans.SessionBean;
import com.tcs.dao.FRTAuditStatusReqDao;
import com.tcs.services.FRTAuditStatusReqService;
import com.tcs.utils.CommonConstants;
import org.apache.log4j.Logger;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Controller;
import org.springframework.validation.BindingResult;
import org.springframework.web.bind.WebDataBinder;
import org.springframework.web.bind.annotation.*;
import org.springframework.web.servlet.ModelAndView;
import org.springframework.web.servlet.ModelAndView;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpSession;
import javax.servlet.http.HttpServletResponse;
import javax.validation.Valid;
import java.util.List;

@Controller
@RequestMapping("/FRTChecker")

public class FRTAuditStatusReqController {

    static Logger log = Logger.getLogger(FRTAuditorsDetailsController.class.getName());

    @Autowired
    FRTAuditStatusReqService frtauditstatusreqservice;

    @Autowired
    FRTAuditStatusReqDao frtauditstatusreqdao;

    @GetMapping("/FRTAuditStatusReq")
    public ModelAndView FRTAuditStatusReq(@ModelAttribute("command") FRTAuditStatusReq frt, BindingResult result, HttpServletRequest request) {

        HttpSession session = request.getSession();
        if (session == null || session.getAttribute(CommonConstants.USER_ID) == null || request.getSession().getId() == null ||
                !session.getAttribute(CommonConstants.USER_CAPABILITY).toString().equals("94")) {
            log.info("*** Unauthorised Access ***" +session.getAttribute(CommonConstants.USER_CAPABILITY));
            return new ModelAndView("500");
        }
        log.info("in /FRTAuditStatusReq controller");
        SessionBean sessionBean = new SessionBean(request.getSession());
        String quarter_date = sessionBean.getQuarterEndDate();
        List<FRTAuditStatusReq> list = frtauditstatusreqservice.getRequests(quarter_date);
        ModelAndView view = new ModelAndView("FRTChecker/FRTAuditStatusReq");
        view.addObject("list",list);
        view.addObject("count",list.size());
        return view;
    }

    @InitBinder("getapproved")
    public void initBindercreate(WebDataBinder binder){
        binder.setDisallowedFields(new String[]{});
    }

    @RequestMapping(value = "/getapproved", method = RequestMethod.POST)
    public @ResponseBody ModelAndView getapproved(@ModelAttribute("command") @Valid FRTAuditStatusReq frt, BindingResult result, HttpServletRequest request) {
        HttpSession session = request.getSession();
        SessionBean sessionBean = new SessionBean(session);
        ModelAndView view = new ModelAndView("FRTChecker/FRTAuditStatusReq");
        String displayMessage = "";
        String Quarter_end_Date = sessionBean.getQuarterEndDate();
        int recCount = 0;
        int reqCount = 0;
        String reqSts = "";

        frt.setCheckBoxList(request.getParameterValues("selRadio"));
        log.info("checkBox: "+frt.getCheckBoxList().length);
        int forCount = frt.getCheckBoxList().length;
        String oldTrackId="";
        for (int i = 0; i < frt.getCheckBoxList().length; i++) {
            // Checks if branch master have the the branchno
            log.info("checkBox["+i+"]: "+frt.getCheckBoxList()[i]);
            String values[] = frt.getCheckBoxList()[i].split("~");
            String reqId = values[0];
            String branchCode = values[1];
            String beforeSts = values[2];
            String afterSts = values[3];
            String roCode = values[4];
            String circleCode = values[5];
            String trackId = values[6];
            if(i!=0) {
                oldTrackId = frt.getCheckBoxList()[i-1].split("~")[6];
            }
            boolean isApprove= frtauditstatusreqservice.approveReq(reqId,trackId, branchCode,Quarter_end_Date,beforeSts,afterSts, roCode,circleCode,sessionBean);
            if(isApprove){
                if (oldTrackId!=trackId) {
                    int repCountWithReject = frtauditstatusreqdao.reqCountWithReject(oldTrackId);
                    int repCount = frtauditstatusreqdao.reqCount(oldTrackId);
                    if (repCountWithReject > 0 && repCount == 0) {
                        reqSts = "2";
                        frtauditstatusreqdao.reqTrackchange(oldTrackId, reqSts);
                    } else {
                        if (reqCount == 0) {
                            reqSts = "2";
                            boolean reqTrackchange = frtauditstatusreqdao.reqTrackchange(oldTrackId, reqSts);
                            log.info("reqTrackchange:- " + reqTrackchange);
                        }
                    }
                }
                recCount++;
                log.info("recCount: "+recCount);
            }
        }
        if (forCount==recCount) {
            displayMessage = "Request Approved Successfully.";
            view.addObject("displayMessage", displayMessage);
            return view;
        } else {
            displayMessage = "Please try again.";
            view.addObject("displayMessage1", displayMessage);
            return view;
        }
    }
    @InitBinder("reject")
    public void initBinderreject(WebDataBinder binder){
        binder.setDisallowedFields(new String[]{});
    }

    @RequestMapping(value = "/getrejected", method = RequestMethod.POST)
    public @ResponseBody
    ModelAndView reject(@ModelAttribute("command") @Valid FRTAuditStatusReq frt, BindingResult result, HttpServletRequest request) {
        HttpSession session = request.getSession();
        SessionBean sessionBean = new SessionBean(session);
        ModelAndView view = new ModelAndView("FRTChecker/FRTAuditStatusReq");
        String displayMessage = "";
        int recCount = 0;

        frt.setCheckBoxList(request.getParameterValues("selRadio"));
        log.info("checkBox: "+frt.getCheckBoxList().length);
        int forCount = frt.getCheckBoxList().length;
        for (int i = 0; i < frt.getCheckBoxList().length; i++) {
            // Checks if branch master have the the branchno
            log.info("checkBox["+i+"]: "+frt.getCheckBoxList()[i]);
            String values[] = frt.getCheckBoxList()[i].split("~");
            String reqId = values[0];
            String trackId = values[6];

            boolean isRejected= frtauditstatusreqservice.rejectReq(reqId,trackId);
            log.info("isRejected:- "+isRejected);
            if(isRejected){
                recCount++;
                log.info("recCount: "+recCount);
            }
        }
        if (forCount==recCount) {
            displayMessage = "Request Rejected Successfully.";
            view.addObject("displayMessage", displayMessage);
            return view;
        } else {
            displayMessage = "Please try again.";
            view.addObject("displayMessage1", displayMessage);
            return view;
        }
    }

}
////////////////////////////////////////////////////////////


package com.tcs.dao;

import com.tcs.beans.FRTAuditStatusReq;
import com.tcs.beans.SessionBean;
import com.tcs.services.MakerService;
import com.tcs.utils.CommonConstants;
import org.apache.log4j.Logger;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.dao.DataAccessException;
import org.springframework.jdbc.core.JdbcTemplate;
import org.springframework.jdbc.core.ResultSetExtractor;
import org.springframework.stereotype.Repository;
import org.springframework.transaction.PlatformTransactionManager;
import org.springframework.transaction.TransactionDefinition;
import org.springframework.transaction.TransactionStatus;
import org.springframework.transaction.support.DefaultTransactionDefinition;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpSession;
import javax.sql.DataSource;
import java.sql.ResultSet;
import java.sql.SQLException;
import java.util.ArrayList;
import java.util.List;

@Repository("FRTAuditStatusReqDao")

public class FRTAuditStatusReqDaoImpl implements FRTAuditStatusReqDao {

    static Logger log = Logger.getLogger(FRTAuditStatusReqDaoImpl.class.getName());

    @Autowired
    DataSource dataSource;

    @Autowired
    private JdbcTemplate jdbcTemplateObject;

    MakerService makerService;

    @Autowired
    private PlatformTransactionManager transactionManager;


    public List<FRTAuditStatusReq> getRequests(String quarter_date) {


            String query1 = "select cas.as_rt_id , cas.as_id, cas.as_branch, br_name branch_name,crs_auditable old_br_status," +
                    "SUBSTR(bm.region_code,1,3) ||''|| SUBSTR(bm.region_code,4,3) ||''|| SUBSTR(bm.region_code,7,3) RO,bm.circle_code, " +
                    "nvl((select ifcofr_audit_flag from crs_ifcofr_audit where ifcofr_branch=cas.as_branch and ifcofr_date=to_date(?,'dd/mm/yyyy') " +
                    "and crt.rt_id=cas.as_rt_id),'N')ifcofr_flag, " +
                    "cas.as_new_status, cas.as_req_status, crt.rt_status,crt.rt_date " +
                    "from crs_request_track crt, crs_audit_status cas, branch_master bm " +
                    "where cas.as_rt_id = crt.rt_id and cas.as_branch=bm.branchno and cas.as_req_status not in ('R','A') order by cas.as_rt_id,cas.as_id";

            List<FRTAuditStatusReq> list = jdbcTemplateObject.query(query1, new Object[]{quarter_date}, new ResultSetExtractor<List<FRTAuditStatusReq>>() {

                @Override
                public List<FRTAuditStatusReq> extractData(ResultSet rs1) throws SQLException, DataAccessException {
                    List<FRTAuditStatusReq> list = new ArrayList<>();
                    while (rs1.next()) {
                        FRTAuditStatusReq report = new FRTAuditStatusReq();
                        report.setAs_rt_id(rs1.getString("AS_RT_ID"));
                        report.setAs_id(rs1.getString("AS_ID"));
                        report.setBranchCode(rs1.getString("AS_BRANCH"));
                        report.setBranchName(rs1.getString("branch_name"));
                        String oldSts =rs1.getString("old_br_status");
                        report.setRoCode(rs1.getString("RO"));
                        report.setCircle_code(rs1.getString("circle_code"));
                        report.setIfcofr_flag(rs1.getString("ifcofr_flag"));
                        String ifcoflag =rs1.getString("ifcofr_flag");
                        if(oldSts.equalsIgnoreCase("Y")){
                            if(ifcoflag.equalsIgnoreCase("Y")){
                                report.setBeforeSts("IFCOFR Audited");
                            }else {
                                report.setBeforeSts("Audited");
                            }
                        }
                        else{
                            report.setBeforeSts("Non-Audited");
                        }

                        String newSts =rs1.getString("as_new_status");
                        if (newSts.equalsIgnoreCase("A")){
                            report.setAfterSts("Audited");
                        }else if(newSts.equalsIgnoreCase("I")){
                            report.setAfterSts("IFCOFR Audited");
                        }
                        else{
                            report.setAfterSts("Non-Audited");
                        }

                        String sts= rs1.getString("as_req_status");
                        if (sts.equalsIgnoreCase("P")){
                            report.setReqSts("Pending");
                        }
                        report.setReq_tracksts(rs1.getString("rt_status"));
                        report.setReqOn(rs1.getString("rt_date"));

                        list.add(report);
                    }
                    log.info("size of the list: "+list.size());
                    return list;
                }

            });
        log.info("size of the list: "+list.size());
            return list;

    }

    public  boolean approveReq1(String req_Id,String track_Id,String ifcofr_flag,String branchCode,String Quarter_end_Date,
                                String auditableFlag,String beforeSts,String before_ifcofr_flag,String beforeS_auditFlag,String aftersts){
        log.info("req_Id- "+req_Id);
        log.info("ifcofr_flag- "+ifcofr_flag);
        log.info("branchCode- "+branchCode);
        log.info("auditableFlag- "+auditableFlag);
        log.info("beforeSts- "+beforeSts);
        String marQuarter = Quarter_end_Date.substring(3,5);
        log.info("march: "+marQuarter);
        TransactionDefinition def = new DefaultTransactionDefinition();
        TransactionStatus status = transactionManager.getTransaction(def);

        //JdbcTemplate jdbcTemplateObject = new JdbcTemplate(dataSource);
        List<Object[]> inputList1 = new ArrayList<Object[]>();
        List<Object[]> inputList2 = new ArrayList<Object[]>();
        List<Object[]> inputList3 = new ArrayList<Object[]>();
        List<Object[]> inputList4 = new ArrayList<Object[]>();
        List<Object[]> inputList5 = new ArrayList<Object[]>();
        List<Object[]> inputList6 = new ArrayList<Object[]>();
        List<Object[]> inputList7 = new ArrayList<Object[]>();
        List<Object[]> inputList8 = new ArrayList<Object[]>();
        boolean flag= false;
        try {
            String update_branch_master = "update branch_master set crs_auditable=? where BRANCHNO=?";
            String update_rml_aud = "update REPORTS_MASTER_LIST set AUDITABLE=? where BRANCH_CODE=? and QUARTER_DATE=to_date(?,'dd/mm/yyyy')";
            String update_lbm = "update lfar_branch set CRS_AUDITABLE=? where BRANCHNO=? and BRANCH_DATE=to_date(?,'dd/mm/yyyy')";
            String update_rml_rw24 = "UPDATE REPORTS_MASTER_LIST set status=? where BRANCH_CODE=? AND QUARTER_DATE=to_date(?,'dd/mm/yyyy')AND REPORT_MASTER_ID='1033' ";
            String delete_rml_24c= "DELETE FROM REPORTS_MASTER_LIST WHERE REPORT_MASTER_ID ='4065' and BRANCH_CODE=? AND QUARTER_DATE=to_date(?,'dd/mm/yyyy') ";
            String update_rml_status29= "update REPORTS_MASTER_LIST set status=? where BRANCH_CODE=? AND QUARTER_DATE=to_date(?,'dd/mm/yyyy') AND status > '23' ";
            String update_ifcofr=  "update CRS_IFCOFR_AUDIT set IFCOFR_AUDIT_FLAG=?,IFCOFR_UPDATE_DATE=sysdate, IFCOFR_UPDATED_BY=? where IFCOFR_BRANCH=? AND IFCOFR_DATE=to_date(?,'dd/mm/yyyy')";
            String delete_lfar_rml = "delete from LFAR_REPORTS_MASTER_LIST WHERE BRANCH_CODE=? AND QUARTER_DATE=to_date(?,'dd/mm/yyyy')";
            String delete_rml_24= "DELETE FROM REPORTS_MASTER_LIST WHERE REPORT_MASTER_ID ='1033' and BRANCH_CODE=? AND QUARTER_DATE=to_date(?,'dd/mm/yyyy') ";
            //for query 1
            Object[] temp1 = {auditableFlag, branchCode};
            inputList1.add(temp1);
            //for query 2,3,4,5
            Object[] temp2 = {auditableFlag, branchCode, Quarter_end_Date};
            inputList2.add(temp2);
            //for query 6
            Object[] temp3 = {"39" ,branchCode, Quarter_end_Date};
        inputList3.add(temp3);
            //for query 7,8,9
            Object[] temp4 = {branchCode,Quarter_end_Date};
            inputList4.add(temp4);
            //for query 10
            Object[] temp5 = {"29",branchCode,Quarter_end_Date};
            inputList5.add(temp5);

            //for query 11
            Object[] temp6 = {ifcofr_flag,"FRT",branchCode,Quarter_end_Date};
            inputList6.add(temp6);
            //for query 12
            Object[] temp7 = {branchCode,Quarter_end_Date};
            inputList7.add(temp7);
            //for query 13
            Object[] temp8 = {branchCode,Quarter_end_Date};
            inputList8.add(temp8);
            if(beforeSts.equalsIgnoreCase("Non-Audited")&& aftersts.equalsIgnoreCase("Audited") ){
                jdbcTemplateObject.batchUpdate(update_branch_master, inputList1);
                jdbcTemplateObject.batchUpdate(update_rml_aud, inputList2);
                jdbcTemplateObject.batchUpdate(update_rml_status29, inputList5);
                if (marQuarter.equalsIgnoreCase("03")) {
                    jdbcTemplateObject.batchUpdate(update_lbm, inputList2);
                }

            }
            else if(beforeSts.equalsIgnoreCase("Non-Audited")&& aftersts.equalsIgnoreCase("IFCOFR Audited")){
                jdbcTemplateObject.batchUpdate(update_branch_master, inputList1);
                jdbcTemplateObject.batchUpdate(update_rml_aud, inputList2);
                jdbcTemplateObject.batchUpdate(update_rml_status29, inputList5);
                jdbcTemplateObject.batchUpdate(update_ifcofr, inputList6);
                if (marQuarter.equalsIgnoreCase("03")) {
                    jdbcTemplateObject.batchUpdate(update_lbm, inputList2);
                }

            }
            else if (beforeSts.equalsIgnoreCase("Audited")&& aftersts.equalsIgnoreCase("IFCOFR Audited")){

                jdbcTemplateObject.batchUpdate(update_ifcofr, inputList6);
                jdbcTemplateObject.batchUpdate(update_rml_rw24, inputList3);

            }
            else if(beforeSts.equalsIgnoreCase("Audited")&& aftersts.equalsIgnoreCase("Non-Audited")){
                jdbcTemplateObject.batchUpdate(update_branch_master, inputList1);
                jdbcTemplateObject.batchUpdate(update_rml_aud, inputList2);
                jdbcTemplateObject.batchUpdate(delete_rml_24, inputList8);
                jdbcTemplateObject.batchUpdate(update_rml_status29, inputList5);
                if (marQuarter.equalsIgnoreCase("03")) {
                    jdbcTemplateObject.batchUpdate(update_lbm, inputList2);
                    jdbcTemplateObject.batchUpdate(delete_lfar_rml, inputList7);
                }

            }
            else if (beforeSts.equalsIgnoreCase("IFCOFR Audited")&& aftersts.equalsIgnoreCase("Audited")){
                jdbcTemplateObject.batchUpdate(update_ifcofr, inputList6);
                jdbcTemplateObject.batchUpdate(delete_rml_24c, inputList4);
                jdbcTemplateObject.batchUpdate(update_rml_rw24, inputList3);

            }
            else if(beforeSts.equalsIgnoreCase("IFCOFR Audited")&& aftersts.equalsIgnoreCase("Non-Audited")){
                jdbcTemplateObject.batchUpdate(update_branch_master, inputList1);
                jdbcTemplateObject.batchUpdate(update_rml_aud, inputList2);
                jdbcTemplateObject.batchUpdate(update_ifcofr, inputList6);
                jdbcTemplateObject.batchUpdate(update_rml_status29, inputList5);
                jdbcTemplateObject.batchUpdate(delete_rml_24c, inputList4);
                jdbcTemplateObject.batchUpdate(delete_rml_24, inputList8);
                if (marQuarter.equalsIgnoreCase("03")) {
                    jdbcTemplateObject.batchUpdate(update_lbm, inputList2);
                    jdbcTemplateObject.batchUpdate(delete_lfar_rml, inputList7);
                }
            }
            transactionManager.commit(status);
            flag=true;
            log.info("after update complete");
        }
        catch(Exception sqle) {
            log.error("exception " + sqle);
            flag=false;
            transactionManager.rollback(status);
        }
        return flag;
    }


    public  boolean reportStsChange(String Quarter_end_Date,String branchCode){
        log.info("into report sts change");
        log.info("branchCode- "+branchCode);

        //JdbcTemplate jdbcTemplateObject = new JdbcTemplate(dataSource);
        String updateQuery= "update REPORTS_MASTER_LIST set status=? where BRANCH_CODE=? AND QUARTER_DATE=to_date(?,'dd/mm/yyyy') AND status > '26' ";
        int updateCount=0;
        updateCount= jdbcTemplateObject.update(updateQuery, new Object[]{"29",branchCode,Quarter_end_Date});
        log.info("UPDATE STatus: "+updateCount);
        if (updateCount>0){
            return true;
        } else
            return false;
    }


    public ArrayList<FRTAuditStatusReq> getReportlist( String branchCode, String Quarter_end_Date){
        log.info("into insertTrack dao");
        log.info("branchCode- "+branchCode);
        log.info("Quarter_end_Date- "+Quarter_end_Date);
        String query= "select REPORT_ID,REPORT_MASTER_ID from reports_master_list where BRANCH_CODE=? and QUARTER_DATE=to_date(?,'dd/mm/yyyy') and status > '26' ";

        ArrayList<FRTAuditStatusReq> list = jdbcTemplateObject.query(query, new Object[]{branchCode,Quarter_end_Date}, new ResultSetExtractor<ArrayList<FRTAuditStatusReq>>() {
                @Override
                public ArrayList<FRTAuditStatusReq> extractData(ResultSet rs1) throws SQLException, DataAccessException {
                    ArrayList<FRTAuditStatusReq> list = new ArrayList<>();
                    while (rs1.next()) {
                        FRTAuditStatusReq report = new FRTAuditStatusReq();
                        report.setReport_Id(rs1.getString("REPORT_ID"));
                        report.setReport_master_id(rs1.getString("REPORT_MASTER_ID"));
                        list.add(report);
                    }
                    log.info("size of the list: " + list.size());
                    return list;
                }
        });
        return list;
    }

    public boolean insertTrack(String branchCode, String Quarter_end_Date, String roCode,String auditableFlag, String circle_code, SessionBean sessionBean){



        boolean track=false;
        ArrayList<FRTAuditStatusReq> list= getReportlist(branchCode,Quarter_end_Date);
        int count= list.size();
        for (int i=0;i<count; i++){
            String reportId= list.get(i).getReport_Id();
            String reportMasterId = list.get(i).getReport_master_id();
            log.info("branchCode "+branchCode+" Quarter_end_Date "+sessionBean.getQuarterEndDate()+ " roCode "+roCode+ " auditableFlag "+auditableFlag+ " circle_code "+circle_code);
            log.info("reportId["+i+"]: "+reportId+" rMId["+i+"]: "+reportMasterId+ " userId: "+sessionBean.getUserId()+" sts: "+CommonConstants.STATUS_29_REJ_FRT);
            try {
                makerService.insertFrtTrack( branchCode,  circle_code,roCode,auditableFlag,sessionBean.getQuarterEndDate(),sessionBean.getUserId(),"CRS",
                        reportId,reportMasterId, CommonConstants.getStatus(CommonConstants.STATUS_29_REJ_FRT),CommonConstants.STATUS_29_REJ_FRT,"" );
                track = true;
            } catch (Exception e) {
                log.error("exception " + e);
            }
        }
      log.info("track insert:-"+track);
        return track;
    }



    public  boolean auditStsChange(String req_Id){
        log.info("into report sts change");
        log.info("req_Id"+req_Id);
        //JdbcTemplate jdbcTemplateObject = new JdbcTemplate(dataSource);
        String updateQuery= "update CRS_AUDIT_STATUS set AS_REQ_STATUS=? where AS_ID=?";
        int updateCount=0;
        updateCount= jdbcTemplateObject.update(updateQuery, new Object[]{"A",req_Id});
        log.info("auditStsChange in audit sts table : "+updateCount);
        if (updateCount>0){
            return true;
        } else
            return false;
    }
    public int reqCount(String track_Id){
        log.info("into select count for req_Id  ");
        log.info("track_Id:- "+track_Id);
        //JdbcTemplate jdbcTemplateObject = new JdbcTemplate(dataSource);
        String query1 =  "SELECT COUNT (*) AS as_rt_id FROM crs_audit_status where as_rt_id=? and as_req_status not in ('R','A')";
        int updateCount=0;
        updateCount= jdbcTemplateObject.queryForObject(query1, new Object[]{track_Id},Integer.class);
        log.info(" TRACK ID count IN audit status table  : "+updateCount);

        return updateCount;
    }

    public int reqCountWithReject(String track_Id){
        log.info("into select count for req_Id  ");
        log.info("track_Id:- "+track_Id);
        //JdbcTemplate jdbcTemplateObject = new JdbcTemplate(dataSource);
        String query1 =  "SELECT COUNT (*) AS as_rt_id FROM crs_audit_status where as_rt_id=? and as_req_status='R'";
        int reqCountWithReject=0;
        reqCountWithReject= jdbcTemplateObject.queryForObject(query1, new Object[]{track_Id},Integer.class);
        log.info(" TRACK ID count IN audit status table with reject status : "+reqCountWithReject);
        return reqCountWithReject;
    }

    public  boolean reqTrackchange(String track_Id,String reqSts){
        log.info("req track status change");
        log.info("track_Id"+track_Id + " reqSts: "+reqSts);
        //JdbcTemplate jdbcTemplateObject = new JdbcTemplate(dataSource);
        String updateQuery= "update CRS_REQUEST_TRACK SET RT_STATUS=?  WHERE RT_ID=?";
        int updateCount=0;
        updateCount= jdbcTemplateObject.update(updateQuery, new Object[]{reqSts, track_Id});
        log.info(" TRACK req status change (crs_request_track) : "+updateCount);
        if (updateCount>0){
            return true;
        } else
            return false;
    }

    public boolean rejectReq(String audit_req_Id,String track_req_Id){
        log.info("into reject request dao");
        TransactionDefinition def = new DefaultTransactionDefinition();
        TransactionStatus status = transactionManager.getTransaction(def);
        //JdbcTemplate jdbcTemplateObject = new JdbcTemplate(dataSource);
        String updateQuery= null;
        try {
            updateQuery = "update CRS_AUDIT_STATUS set AS_REQ_STATUS=? where AS_ID=?";
            transactionManager.commit(status);
        } catch (Exception e) {
            log.error("exception " + e);
        }
        int updateCount=0;
        updateCount= jdbcTemplateObject.update(updateQuery, new Object[]{"R",audit_req_Id});
        log.info("auditStsChange in audit_sts(reject) : "+updateCount);
        if (updateCount>0){
            return true;
        } else
            return false;
    }

    }

//////////////////////////////////////////////////////////////////////////////////

import * as React from "react";
import { useState, useEffect } from "react";
import {
  Box,
  Button,
  Checkbox,
  Container,
  Dialog,
  DialogActions,
  DialogContent,
  DialogTitle,
  Alert,
  Paper,
  Table,
  TableBody,
  TableCell,
  TableContainer,
  TableHead,
  TableRow,
  TablePagination,
  Typography,
  CircularProgress,
} from "@mui/material";
import axios from "axios";
import { encrypt } from "../Security/AES-GCM256";

const iv = crypto.getRandomValues(new Uint8Array(12));
const ivBase64 = btoa(String.fromCharCode.apply(null, iv));
const salt = crypto.getRandomValues(new Uint8Array(16));
const saltBase64 = btoa(String.fromCharCode.apply(null, salt));

// Table Header Component
const EnhancedTableHead = (props) => {
  const { onSelectAllClick, numSelected, rowCount } = props;

  return (
    <TableHead>
      <TableRow sx={{ backgroundColor: "#b9def0" }}>
        <TableCell padding="checkbox" align="center">
          <Box
            sx={{ display: "flex", alignItems: "center", whiteSpace: "nowrap" }}
          >
            <Checkbox
              color="primary"
              indeterminate={numSelected > 0 && numSelected < rowCount}
              checked={rowCount > 0 && numSelected === rowCount}
              onChange={onSelectAllClick}
              inputProps={{ "aria-label": "select all requests" }}
            />
            <b>Select All</b>
          </Box>
        </TableCell>
        <TableCell align="center">
          <b>Req ID</b>
        </TableCell>
        <TableCell align="center">
          <b>Branch Code</b>
        </TableCell>
        <TableCell align="left">
          <b>Branch Name</b>
        </TableCell>
        <TableCell align="center">
          <b>Existing Status</b>
        </TableCell>
        <TableCell align="center">
          <b>New Requested Status</b>
        </TableCell>
        <TableCell align="center">
          <b>Request Status</b>
        </TableCell>
        <TableCell align="center">
          <b>Requested On</b>
        </TableCell>
      </TableRow>
    </TableHead>
  );
};

const FRTAuditStatusReq = () => {
  const [requests, setRequests] = useState([]);
  const [selected, setSelected] = useState([]);
  const [loading, setLoading] = useState(true);
  const [page, setPage] = useState(0);
  const [rowsPerPage, setRowsPerPage] = useState(5);
  const user = JSON.parse(localStorage.getItem("user"));
  const [dialog, setDialog] = useState({
    open: false,
    message: "",
    severity: "success",
  });

  const fetchRequests = async () => {
    setLoading(true);
    try {
      let jsonFormData = JSON.stringify({ quarterDate: user?.quarterEndDate });
      await encrypt(iv, salt, jsonFormData).then(function (result) {
        jsonFormData = result;
      });
      let payload = { iv: ivBase64, salt: saltBase64, data: jsonFormData };

      const response = await axios.post(
        "/Server/EditBranch/getPendingRequests",
        payload,
        {
          headers: { Authorization: `Bearer ${localStorage.getItem("token")}` },
        }
      );
      console.log("response.data.result", response.data.result);

      const mappedData = response.data.result.map((item) => ({
        ...item,
        id: item.AS_ID,
      }));
      setRequests(mappedData);
    } catch (error) {
      console.error("Failed to fetch requests:", error);
      setDialog({
        open: true,
        message: "Failed to load requests. Please try again later.",
        severity: "error",
      });
      setRequests([]);
    } finally {
      setLoading(false);
    }
  };

  useEffect(() => {
    document.title = "FRT | Audit Status Requests";
    fetchRequests();
  }, []);

  const handleSelectAllClick = (event) => {
    if (event.target.checked) {
      const newSelected = requests.map((n) => n.id);
      setSelected(newSelected);
      return;
    }
    setSelected([]);
  };

  const handleClick = (event, id) => {
    const selectedIndex = selected.indexOf(id);
    let newSelected = [];

    if (selectedIndex === -1) {
      newSelected = newSelected.concat(selected, id);
    } else if (selectedIndex === 0) {
      newSelected = newSelected.concat(selected.slice(1));
    } else if (selectedIndex === selected.length - 1) {
      newSelected = newSelected.concat(selected.slice(0, -1));
    } else if (selectedIndex > 0) {
      newSelected = newSelected.concat(
        selected.slice(0, selectedIndex),
        selected.slice(selectedIndex + 1)
      );
    }
    setSelected(newSelected);
  };

  const handleAction = async (action) => {
    setLoading(true);
    // 'action' will be 'approve' or 'reject'
    const endpoint = `/api/frt/${action}`;

    try {
      const response = await axios.post(
        endpoint,
        { requestIds: selected }, // Send selected IDs in the request body
        {
          headers: {
            Authorization: `Bearer ${localStorage.getItem("token")}`,
          },
        }
      );

      // Show success message from backend
      setDialog({
        open: true,
        message:
          response.data.message ||
          `Successfully performed action on ${selected.length} request(s).`,
        severity: "success",
      });
      setSelected([]);
      fetchRequests();
    } catch (error) {
      console.error(`Failed to ${action} requests:`, error);
      setDialog({
        open: true,
        message:
          error.response?.data?.message || `Failed to ${action} requests.`,
        severity: "error",
      });
    } finally {
      setLoading(false);
    }
  };

  const getStatusStyles = (status) => {
    if (status?.toLowerCase() === "pending") {
      return {
        backgroundColor: "#ff9800",
        color: "white",
        fontWeight: "bold",
        padding: "4px 12px",
        borderRadius: "16px",
        display: "inline-block",
        textTransform: "capitalize",
      };
    }
    return { padding: "4px 0" };
  };

  const handleDialogClose = () => {
    setDialog({ ...dialog, open: false });
  };

  const isSelected = (id) => selected.indexOf(id) !== -1;

  const emptyRows =
    page > 0 ? Math.max(0, (1 + page) * rowsPerPage - requests.length) : 0;

  return (
    <>
      <Container maxWidth={false}>
        <Paper sx={{ width: "100%", mb: 2, p: 2 }}>
          <Typography
            variant="h5"
            component="div"
            sx={{ mb: 2, textAlign: "left" }}
          >
            Audit Status Change Requests
          </Typography>
          <Box sx={{ display: "flex", justifyContent: "center", mb: 2 }}>
            <Button
              variant="contained"
              color="success"
              sx={{ mr: 2, width: "120px" }}
              onClick={() => handleAction("approve")}
              disabled={selected.length === 0 || loading}
            >
              Approve
            </Button>
            <Button
              variant="contained"
              color="error"
              sx={{ width: "120px" }}
              onClick={() => handleAction("reject")}
              disabled={selected.length === 0 || loading}
            >
              Reject
            </Button>
          </Box>
          <TableContainer>
            {loading ? (
              <Box
                sx={{
                  display: "flex",
                  justifyContent: "center",
                  alignItems: "center",
                  height: 370,
                }}
              >
                <CircularProgress />
              </Box>
            ) : (
              <Table sx={{ minWidth: 750 }} aria-labelledby="tableTitle">
                <EnhancedTableHead
                  numSelected={selected.length}
                  onSelectAllClick={handleSelectAllClick}
                  rowCount={requests.length}
                />
                <TableBody>
                  {requests
                    .slice(page * rowsPerPage, page * rowsPerPage + rowsPerPage)
                    .map((row) => {
                      const isItemSelected = isSelected(row.id);
                      const labelId = `enhanced-table-checkbox-${row.id}`;

                      return (
                        <TableRow
                          hover
                          onClick={(event) => handleClick(event, row.id)}
                          role="checkbox"
                          aria-checked={isItemSelected}
                          tabIndex={-1}
                          key={row.id}
                          selected={isItemSelected}
                          sx={{ cursor: "pointer" }}
                        >
                          <TableCell padding="checkbox">
                            <Checkbox
                              color="primary"
                              checked={isItemSelected}
                              inputProps={{ "aria-labelledby": labelId }}
                            />
                          </TableCell>
                          {/* Use keys from the backend map */}
                          <TableCell align="center">{row.AS_RT_ID}</TableCell>
                          <TableCell align="center">{row.BRANCHCODE}</TableCell>
                          <TableCell align="left">{row.BRANCHNAME}</TableCell>
                          <TableCell align="center">{row.BEFORESTS}</TableCell>
                          <TableCell align="center">{row.AFTERSTS}</TableCell>
                          <TableCell align="center">
                            <Box
                              component="span"
                              sx={getStatusStyles(row.REQSTS)}
                            >
                              {row.REQSTS}
                            </Box>
                          </TableCell>
                          <TableCell align="center">{row.REQON}</TableCell>
                        </TableRow>
                      );
                    })}
                  {emptyRows > 0 && (
                    <TableRow style={{ height: 53 * emptyRows }}>
                      <TableCell colSpan={8} />
                    </TableRow>
                  )}
                  {requests.length === 0 && !loading && (
                    <TableRow>
                      <TableCell colSpan={8} align="center">
                        No pending requests found.
                      </TableCell>
                    </TableRow>
                  )}
                </TableBody>
              </Table>
            )}
          </TableContainer>
          <TablePagination
            rowsPerPageOptions={[5, 10, 25]}
            component="div"
            count={requests.length}
            rowsPerPage={rowsPerPage}
            page={page}
            onPageChange={(e, newPage) => setPage(newPage)}
            onRowsPerPageChange={(e) => {
              setRowsPerPage(parseInt(e.target.value, 10));
              setPage(0);
            }}
          />
        </Paper>
      </Container>
      <Dialog open={dialog.open} onClose={handleDialogClose}>
        <DialogTitle>
          {dialog.severity === "success" ? "Success" : "Error"}
        </DialogTitle>
        <DialogContent>
          <Alert severity={dialog.severity}>{dialog.message}</Alert>
        </DialogContent>
        <DialogActions>
          <Button onClick={handleDialogClose}>Close</Button>
        </DialogActions>
      </Dialog>
    </>
  );
};

export default FRTAuditStatusReq;




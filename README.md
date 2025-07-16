<%@ taglib prefix="form" uri="http://www.springframework.org/tags/form" %>
<%--
  Created by IntelliJ IDEA.
  User: V1010939
  Date: 17-03-2023
  Time: 11:33
  To change this template use File | Settings | File Templates.
--%>
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>Title</title>
    <script>
        $(document).ready(function () {
            $('input').iCheck({
                checkboxClass: 'icheckbox_flat-green',
                radioClass: 'iradio_flat-purple'
            });

            $('input').on('ifChecked', function (event) {
                if (document.getElementById("optionsRadios1").checked == true) {
                    document.getElementById("IfcofrSts1").hidden = true;
                    $('#optionsRadios4').iCheck('check');

                }
                else if (document.getElementById("optionsRadios2").checked == true) {
                    document.getElementById("IfcofrSts1").hidden = false;
                }
            });


        });

        function searchBranch() {
            var myForm = $("#form1").serialize();
            console.log("data :" + myForm);
            if (document.getElementById("branchCode").value != '') {
                $.ajax({
                    type: "POST",
                    url: "./searchBranchCode",
                    async: false,
                    data: myForm,
                    success: function (data) {
                        console.log("data :" + data);
                        if (data != '') {
                            data.forEach(function (vd) {
                                document.getElementById("branchCode1").value = vd.branchCode;
                                document.getElementById("branchName").value = vd.branchName;
                                document.getElementById("requestId").value = vd.requestId;
                                document.getElementById("circleCode").value = vd.circleName;
                                document.getElementById("roCode").value = vd.roDetails;
                                if (document.getElementById("roCode").value != '') {
                                    var net = document.getElementById("roCode").value.substring(0, 3);
                                    var mod = document.getElementById("roCode").value.substring(3, 6);
                                    var reg = document.getElementById("roCode").value.substring(6, 9);
                                    document.getElementById("roCode").value = net + " " + mod + " " + reg;
                                }
                                document.getElementById("auditedStatus").value = vd.auditFlag;
                                console.log("Audit Status :" + document.getElementById("auditedStatus").value);
                                if (document.getElementById("auditedStatus").value == 'Y') {
                                    document.getElementById("auditedStatus").value = 'Audited';
                                    $('#optionsRadios2').iCheck('check')
                                } else {
                                    document.getElementById("auditedStatus").value = 'Non-Audited';
                                    $('#optionsRadios1').iCheck('check')
                                }
                                document.getElementById("ifcofrCode").value = vd.ifcofrFlag;
                                if (document.getElementById("ifcofrCode").value == 'Y') {
                                    document.getElementById("ifcofrCode").value = 'Yes';
                                    $('#optionsRadios3').iCheck('check')
                                } else if (document.getElementById("ifcofrCode").value == 'N') {
                                    document.getElementById("ifcofrCode").value = 'No';
                                    $('#optionsRadios4').iCheck('check')
                                } else {
                                    document.getElementById("ifcofrCode").value = '--';
                                    $('#optionsRadios4').iCheck('check')
                                }
                                console.log("ifcofrCode :" + vd.ifcofrFlag);
                            })
                            if (document.getElementById("userCapacity").value == '96') {
                                document.getElementById("branchSts").hidden = false;
                                document.getElementById("radioBtn").hidden = false;
                                document.getElementById("button").hidden = false;
                            } else {
                                document.getElementById("branchSts").hidden = false;
                            }

                        } else {
                            $('#warnModal .modal-body').text("Branch does not exists.");
                            $('#warnModal').modal('show');
                            document.getElementById("branchSts").hidden = true;
                            document.getElementById("radioBtn").hidden = true;
                            document.getElementById("button").hidden = true;
                        }
                    },
                    error: function (jqXHR, textStatus, errorMessage) {
                        $('#infoModal .modal-body').text("No response received");
                        $('#infoModal').modal('show');
                    },
                    complete: function () {

                    },
                });
            } else {
                $('#infoModal .modal-body').text("Please Enter branch code first !!!");
                $('#infoModal').modal('show');
            }

        }

        function saveMe() {
            var auditSts;
            if (document.getElementById("optionsRadios1").checked == true) {
                auditSts = document.getElementById("optionsRadios1").value;
                console.log(auditSts);
            }
            else if (document.getElementById("optionsRadios2").checked == true &&
                document.getElementById("optionsRadios3").checked == true) {
                auditSts = document.getElementById("optionsRadios3").value
                console.log(auditSts);
            }
            else if (document.getElementById("optionsRadios2").checked == true &&
                document.getElementById("optionsRadios4").checked == true) {
                auditSts = document.getElementById("optionsRadios2").value;
                console.log(auditSts);
            }
            console.log("new audit sts :" + auditSts);
            console.log("value :" + document.getElementById("ifcofrCode").value);
            var preAuditSts;

            if (document.getElementById("auditedStatus").value == 'Non-Audited') {
                console.log("in non-audit :")
                preAuditSts = 'N';
            }
            else {
                console.log("in audit :")
                preAuditSts = 'A';
                if (document.getElementById("ifcofrCode").value == 'Yes') {
                    console.log("in ifcofr audit :")
                    preAuditSts = 'I';
                }
            }
            console.log("old audit sts :" + preAuditSts);
            if (auditSts == preAuditSts) {
                $('#warnModal .modal-body').text("There is no change in audit status.");
                $('#warnModal').modal('show');
            } else {
                var saveData = {
                    "branchCode": document.getElementById("branchCode1").value,
                    "branchName": document.getElementById("branchName").value,
                    "requestId": document.getElementById("requestId").value,
                    "circleCode": document.getElementById("circleCode").value,
                    "roCode": document.getElementById("roCode").value,
                    "auditSts": auditSts,

                }
                $.ajax({
                    type: "POST",
                    url: "./saveMe",
                    data: JSON.stringify(saveData),
                    contentType: 'application/json;charset=utf-8',
                    dataType: 'text',
                    async: false,
                    success: function (data) {
                        if (data == 'NODATA') {
                            console.log("data :" + data);
                            $('#warnModal .modal-body').text('The request can\'t be saved due to technical error. Kindly try again.');
                            $('#warnModal').modal('show');
                        } else {
                            console.log("data :" + data);
                            $('#successModal .modal-body').text(data);
                            $('#successModal').modal('show');
                        }
                    },
                    error: function (jqXHR, textStatus, errorMessage) {
                        $('#infoModal .modal-body').text("No response received");
                        $('#infoModal').modal('show');
                    },
                    complete: function () {

                    },
                });
            }


        }


        function submitRequest1() {

            document.getElementById('form1').action = "./downloadForm";
            document.getElementById('form1').submit();
        }


        function resetAll() {
            document.getElementById("branchSts").hidden = true;
            document.getElementById("radioBtn").hidden = true;
            document.getElementById("button").hidden = true;
            document.getElementById("branchCode").value = '';
        }
    </script>
</head>
<body class="hold-transition skin-blue sidebar-mini">
<div class="wrapper">
    <div class="content-wrapper">
        <section class="content-header">

            <h1>
                View Branch Audit Status
            </h1>
        </section>
        <section class="content">
            <div class="row">
                <div class="col-xs-12">
                    <div class="box box-primary box-solid">
                        <div class="box-body ">
                            <div class="box-header">
                            </div>
                            <form:form id="form1">
                                <div class="row">
                                    <div class="col-md-2">
                                        <input type="text" id="branchCode" name="branchCode" maxlength="5"
                                               class="form-control" placeholder="Please Enter Branch Code">
                                    </div>
                                    <div class="col-md-1">
                                        <button type="button" class="btn bg-purple btn-block" onclick="searchBranch()">
                                            <i class="fa fa-search"></i>&nbsp;
                                            Search
                                        </button>
                                    </div>
                                    <div class="col-md-3">
                                        <div hidden>
                                            <input type="text" id="requestId" name="requestId"
                                                   class="form-control"/>
                                        </div>
                                    </div>

                                    <div class="col-md-2">
                                        <label style="margin-top: 5px">List Of All Branches With Audit Status :</label>
                                    </div>
                                    <div class="col-md-1">
                                        <button type="button" class="btn bg-olive-active btn-block"
                                                onclick="submitRequest1();">
                                            Download&nbsp;<i class="fa fa-cloud-download"></i>
                                        </button>
                                    </div>
                                    <input type="hidden" name="userCapacity" id="userCapacity"
                                           value="${userCapacity}"/>
                                </div>
                            </form:form>
                            <div id="branchSts" style="margin-top: 30px" hidden>
                                <div class="row">
                                    <div class="col-md-12">
                                        <h4>Existing Branch Audited Status</h4>
                                    </div>
                                    <div class="col-md-9">
                                        <table class="table table-bordered">
                                            <thead>
                                            <tr style="background: #b9def0; height: 34px">
                                                <th style="text-align: center">Branch</th>
                                                <th style="text-align: center">Name</th>
                                                <th style="text-align: center">Circle</th>
                                                <th style="text-align: center">RO</th>
                                                <th style="text-align: center">Audited Status</th>
                                                <th style="text-align: center">IFCOFR</th>
                                            </tr>
                                            </thead>
                                            <tbody>
                                            <tr>
                                                <td>
                                                    <input type="text" class="form-control"
                                                           style="text-align: center; background-color: white; border-style: hidden"
                                                           id="branchCode1" value="" readonly>
                                                </td>
                                                <td>
                                                    <input type="text" class="form-control"
                                                           style="text-align: center;  background-color: white; border-style: hidden"
                                                           id="branchName" value="" readonly>
                                                </td>
                                                <td>
                                                    <input type="text" class="form-control"
                                                           style="text-align: center;  background-color: white; border-style: hidden"
                                                           id="circleCode" value="" readonly>
                                                </td>
                                                <td>
                                                    <input type="text" class="form-control"
                                                           style="text-align: center;  background-color: white; border-style: hidden"
                                                           id="roCode" value="" readonly>
                                                </td>
                                                <td>
                                                    <input type="text" class="form-control"
                                                           style="text-align: center;  background-color: white; border-style: hidden"
                                                           id="auditedStatus" value="" readonly>
                                                </td>
                                                <td>
                                                    <input type="text" class="form-control"
                                                           style="text-align: center;  background-color: white; border-style: hidden"
                                                           id="ifcofrCode" value="" readonly>
                                                </td>
                                            </tr>
                                            </tbody>
                                        </table>
                                    </div>
                                </div>
                            </div>
                            <div id="radioBtn" hidden>
                                <div class="row">
                                    <div class="col-md-1">
                                        <p style="margin-top: 10px">
                                            Audit Status :</p></div>
                                    <div class="col-md-2">
                                        <div class="form-group">
                                            <label>
                                                <input type="radio" id="optionsRadios1" value="N" name="auditSts"
                                                       onclick="hideIfcofr()"/>
                                                Non-Audited Branch
                                            </label>
                                        </div>
                                    </div>
                                    <div class="col-md-2">
                                        <div class="form-group">
                                            <label>
                                                <input type="radio" id="optionsRadios2" value="A"
                                                       name="auditSts"/>
                                                Audited Branch
                                            </label>
                                        </div>
                                    </div>
                                </div>
                                <div class="row" id="IfcofrSts1">
                                    <div class="col-md-1">
                                        <p style="margin-top: 10px">
                                            IFCOFR Audit :</p>
                                    </div>
                                    <div class="col-md-2" onclick="method1()">
                                        <div class="form-group">
                                            <label>
                                                <input type="radio" id="optionsRadios3" value="I" name="ifcofrSts"
                                                />
                                                Yes
                                            </label>
                                        </div>
                                    </div>
                                    <div class="col-md-2">
                                        <div class="form-group">
                                            <label>
                                                <input type="radio" id="optionsRadios4" value="" onclick="method2()"
                                                       name="ifcofrSts"/>
                                                No
                                            </label>
                                        </div>
                                    </div>
                                </div>
                            </div>
                            <div class="box-footer">
                                <div id="button" hidden>
                                    <div class="col-md-1">
                                        <button type="submit" id="saveBtn" class="btn btn-success btn-block"
                                                onclick="saveMe()" hidden>
                                            Save &nbsp;<i class="fa fa-floppy-o"></i>
                                        </button>
                                    </div>
                                    <div class="col-md-1">
                                        <button type="button" class="btn bg-red-active btn-block"
                                                onclick="resetAll()">
                                            Discard &nbsp;<i class="glyphicon glyphicon-remove-circle"></i>
                                        </button>
                                    </div>
                                </div>
                            </div>
                            <div class="example-modal">
                                <div class="modal fade" id="infoModal">
                                    <div class="modal-dialog">
                                        <div class="modal-content">
                                            <div class="modal-header bg-info">
                                                <button type="button" class="close" data-dismiss="modal"
                                                        aria-label="Close">
                                                    <span aria-hidden="true">&times;</span></button>
                                                <h4 class="modal-title">Attention!</h4>
                                            </div>
                                            <div class="modal-body">
                                            </div>
                                            <div class="modal-footer">
                                                <button type="button" class="btn btn-info" data-dismiss="modal">
                                                    Close
                                                </button>
                                            </div>
                                        </div>
                                    </div>
                                </div>
                            </div>
                            <div class="example-modal">
                                <div class="modal fade" id="successModal">
                                    <div class="modal-dialog">
                                        <div class="modal-content">
                                            <div class="modal-header bg-success">
                                                <button type="button" class="close" data-dismiss="modal"
                                                        aria-label="Close">
                                                    <span aria-hidden="true">&times;</span></button>
                                                <h4 class="modal-title">Attention!</h4>
                                            </div>
                                            <div class="modal-body">
                                            </div>
                                            <div class="modal-footer">
                                                <button type="button" class="btn btn-success" onclick="resetAll()"
                                                        data-dismiss="modal">
                                                    Close
                                                </button>
                                            </div>
                                        </div>
                                    </div>
                                </div>
                            </div>
                            <div class="example-modal">
                                <div class="modal fade" id="warnModal">
                                    <div class="modal-dialog">
                                        <div class="modal-content">
                                            <div class="modal-header bg-danger">
                                                <button type="button" class="close" data-dismiss="modal"
                                                        aria-label="Close">
                                                    <span aria-hidden="true">&times;</span></button>
                                                <h4 class="modal-title">WARNING!</h4>
                                            </div>
                                            <div class="modal-body">
                                            </div>
                                            <div class="modal-footer">
                                                <button type="button" class="btn btn-danger" data-dismiss="modal"
                                                        onclick="">Close
                                                </button>
                                            </div>
                                        </div>
                                    </div>
                                </div>
                            </div>
                        </div>
                        <div class="overlay" id="divMsg" hidden>
                            <i class="fa fa-spinner fa-spin" style="color: #9a0200"></i>
                        </div>
                    </div>
                </div>
            </div>
        </section>
    </div>
</div>
</body>
</html>
/////////////////////////////////////////////////////////////////////////////////////////////////////////////



package com.tcs.controllers;

import com.tcs.beans.FrtSingleBranch;
import com.tcs.beans.SessionBean;
import com.tcs.services.FrtSingleBranchService;
import com.tcs.utils.AutoClean;
import com.tcs.utils.CleanPath;
import com.tcs.utils.CommonConstants;
import net.sf.jasperreports.engine.JRException;
import net.sf.jasperreports.engine.JasperFillManager;
import net.sf.jasperreports.engine.JasperPrint;
import net.sf.jasperreports.engine.export.ooxml.JRXlsxExporter;
import net.sf.jasperreports.export.SimpleExporterInput;
import net.sf.jasperreports.export.SimpleOutputStreamExporterOutput;
import net.sf.jasperreports.export.SimpleXlsxReportConfiguration;
import org.apache.commons.configuration.Configuration;
import org.apache.commons.io.FileUtils;
import org.apache.log4j.Logger;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Controller;
import org.springframework.validation.BindingResult;
import org.springframework.web.bind.annotation.*;
import org.springframework.web.servlet.ModelAndView;
import org.springframework.web.servlet.mvc.support.RedirectAttributes;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import javax.servlet.http.HttpSession;
import javax.sql.DataSource;
import java.io.File;
import java.io.FileOutputStream;
import java.io.IOException;
import java.io.OutputStream;
import java.math.BigDecimal;
import java.sql.Connection;
import java.sql.SQLException;
import java.text.DateFormat;
import java.text.SimpleDateFormat;
import java.util.*;

@Controller
@RequestMapping("/FRTUser")

public class FrtSingleBranchController {

    static Logger log = Logger.getLogger(FrtSingleBranchController.class.getName());

    @Autowired
    DataSource dataSource;
    


    

    @Autowired
    FrtSingleBranchService frtSingleBranchService;

    @GetMapping("/singleBranch")
    public ModelAndView branchStatus(@ModelAttribute("command") FrtSingleBranch frt, BindingResult result, HttpServletRequest request) {

        HttpSession session = request.getSession();
        SessionBean sessionBean = new SessionBean(request.getSession());
        String userId = sessionBean.getUserId();
        String userCapacity = sessionBean.getUserCapability();
        log.info("userId :" + userId + " userCapacity :" + userCapacity);
        if (session == null || session.getAttribute(CommonConstants.USER_ID) == null || request.getSession().getId() == null ||
                !(Objects.equals(session.getAttribute(CommonConstants.USER_CAPABILITY), new BigDecimal("96")) ||
                        Objects.equals(session.getAttribute(CommonConstants.USER_CAPABILITY), new BigDecimal("94")))) {
            log.error("***Unauthorised Access***" + session.getAttribute(CommonConstants.USER_CAPABILITY));
            return new ModelAndView("500");
        }
        log.info("inside db");
        ModelAndView view = new ModelAndView("FRTUser/singleBranch");
        String financialYear = session.getAttribute("year").toString();
        String quarterCurrent = (String) session.getAttribute("quarter");
        view.addObject("userCapacity", userCapacity);
        view.addObject("financialYear", financialYear);
        view.addObject("quarterCurrent", quarterCurrent);
        return view;
    }

    @PostMapping("/searchBranchCode")
    @ResponseBody
    public List<FrtSingleBranch> searchBranchCode(@ModelAttribute("command") FrtSingleBranch frt,
                                                  @RequestParam("branchCode") String branchCode, HttpServletRequest request) {
        HttpSession session = request.getSession();
        SessionBean sessionBean = new SessionBean(session);
        String quarterEndDate = sessionBean.getQuarterEndDate();
        List<FrtSingleBranch> searchlist;
        searchlist = frtSingleBranchService.searchBranch(branchCode, quarterEndDate);
        log.info("branchCode :" + frt.getAuditFlag() + frt.getBranchCode());

        return searchlist;
    }

    @PostMapping(value = "/saveMe")
    public @ResponseBody
    String saveMe(@RequestBody Map map, HttpServletRequest request) {

        HttpSession session = request.getSession();
        SessionBean sessionBean = new SessionBean(request.getSession());
        String userId = sessionBean.getUserId();
        String quarterEndDate = sessionBean.getQuarterEndDate();
        String displayMessage = "NODATA";
        log.info("userId :" + userId + " " + sessionBean.getBranchCode() + " " + sessionBean.getQuarterEndDate());
        String branchCode1 = (String) map.get("branchCode");
        String branchName = (String) map.get("branchName");
        String circleCode = (String) map.get("circleCode");
        String roCode = (String) map.get("roCode");
        String auditSts = (String) map.get("auditSts");
        String requestId = (String) map.get("requestId");
        log.info("requestId :" + requestId + "auditSts " + auditSts);
        String reqId = null;
        if (requestId == null || requestId.isEmpty()) {
            reqId = frtSingleBranchService.insertTrack(userId, quarterEndDate, branchCode1, auditSts);
            if (!reqId.isEmpty()) {
                displayMessage = "Request id :" + reqId + "- for changing audit status for branch: " + branchCode1 + "-" + branchName
                        + " to CRS Scope for quarter ending " + sessionBean.getQuarterEndDate() + " is pending with FRT Checker.";
            }

        } else {
            boolean updateFlag = frtSingleBranchService.updateTrack(userId, quarterEndDate, branchCode1, requestId, auditSts);
            if (updateFlag) {
                displayMessage = "Request id :" + requestId + "- for changing audit status for branch: " + branchCode1 + "-" + branchName
                        + " to CRS Scope for quarter ending " + sessionBean.getQuarterEndDate() + " is pending with FRT Checker.";
            }
        }

        log.info(branchCode1 + " " + " " + branchName + " " + circleCode + " " + roCode + " " + auditSts);

        return displayMessage;
    }

    @RequestMapping(value = "/downloadForm", method = RequestMethod.POST)
    public ModelAndView download(@ModelAttribute("command") FrtSingleBranch frt, BindingResult result,
                                 RedirectAttributes redirectAttributes, HttpServletRequest request, HttpServletResponse response)
            throws SQLException, JRException, IOException {
        HttpSession session = request.getSession();
        SessionBean sessionBean = new SessionBean(request.getSession());
        String userCapacity = sessionBean.getUserCapability();
        if (session == null || session.getAttribute(CommonConstants.USER_ID) == null || request.getSession().getId() == null ||
                !(userCapacity.equalsIgnoreCase("96") || userCapacity.equalsIgnoreCase("94"))) {
            log.error("***Unauthorised Access***" + session.getAttribute(CommonConstants.USER_CAPABILITY));
            return new ModelAndView("500");
        }
        log.info("inside db");
        String userId = session.getAttribute(CommonConstants.USER_ID).toString();
        String opt = session.getAttribute(CommonConstants.OPT).toString();
        String displayResponce = "";

        Configuration config;
        Connection con = null;
        try {
            con = dataSource.getConnection();

            String jrxmlName = "Frt_Branch_List";
            log.info("jrxmlName " + jrxmlName);

            DateFormat dateFormat = new SimpleDateFormat("ddMMyyyy HHmmss");
            Date date = new Date();
            String timeStamp = dateFormat.format(date).replace(" ", "");
            String fileName = jrxmlName + ".xlsx";
            String outFilePath = AutoClean.cleanedPath(request.getSession(), AutoClean.TIMESTAMP, fileName, "", request);
            String JasperFilePath = AutoClean.cleanedPath(request.getSession(), AutoClean.JASPER_FILE_PATH, jrxmlName, "", request);

            Map<String, Object> param = new HashMap<String, Object>();
            param.put("IS_IGNORE_PAGINATION", true);
            JasperPrint jasperPrint;
            jasperPrint = JasperFillManager.fillReport(JasperFilePath, param, con);


            OutputStream out2 = new FileOutputStream(new File(outFilePath));
            JRXlsxExporter excelExporter = new JRXlsxExporter();
            excelExporter.setExporterInput(new SimpleExporterInput(jasperPrint));
            excelExporter.setExporterOutput(new SimpleOutputStreamExporterOutput(out2));
            SimpleXlsxReportConfiguration configuration = new SimpleXlsxReportConfiguration();
            configuration.setDetectCellType(true);
            configuration.setWhitePageBackground(false);
            configuration.setRemoveEmptySpaceBetweenRows(true);
            configuration.setIgnoreCellBorder(true);
            excelExporter.setConfiguration(configuration);
            excelExporter.exportReport();
            log.info("exportdoneeee");

            File file2 = new File(outFilePath);
            byte[] pdfContent = FileUtils.readFileToByteArray(file2);
            String contentType = "application/vnd.openxmlformats-officedocument.spreadsheetml.sheet";
            response.setContentType(contentType);
            response.setHeader("Content-Disposition", "attachment;filename=" + CleanPath.cleanString("Audited Branch Status" + ".xlsx"));
            OutputStream out1 = null;
            out1 = (OutputStream) response.getOutputStream();
            out1.write(pdfContent);
            out1.close();
            response.flushBuffer();
            log.info("Download Success");
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            if (null != con) {
                con.close();
                log.warn("*** Connection Closed ***");
            }
        }
        return null;
    }
}

<%--
  Created by IntelliJ IDEA.
  User: V1012981
  Date: 14-03-2023
  Multiple Branch Auditable Status change
--%>
<%@ include file="/views/include.jsp" %>
<!DOCTYPE html>
<html>
<head>

    <style>
        .tooltip {
            position: relative;
            display: inline-block;
            opacity: 1;
        }

        .tooltip .tooltiptext {
            visibility: hidden;
            width: 125px;
            background-color: #b91a24;
            color: #fff;
            text-align: center;
            border-radius: 6px;
            padding: 5px 0;

            /* Position the tooltip */
            position: absolute;
            z-index: 1;
            top: -5px;
            left: 65%;
        }

        .tooltip:hover .tooltiptext {
            visibility: visible;
        }

    </style>

    <script src="https://cdnjs.cloudflare.com/ajax/libs/xlsx/0.17.5/xlsx.min.js"></script>

    <script>


        /* Ref: https://javacodepoint.com/convert-excel-file-data-to-json-in-javascript/
          https://www.c-sharpcorner.com/article/reading-a-excel-file-using-html5-jquery/
          https://www.javatpoint.com/import-excel-data-into-html-table-using-jquery
          https://morioh.com/p/5a66173f479d*/
        /*Method to upload a valid excel file*/
        function upload() {
            console.log("Upload Start...." + new Date().toDateString() + "-" + new Date().toLocaleTimeString());
            document.getElementById("divMsg").hidden = false;
            var files = document.getElementById('sampleFile').files;

            var fileSize = files[0].size;
            console.log(fileSize);
            var fileSizeRequired = Math.round((fileSize / 1024));


            if (fileSizeRequired > 2048) {   ///if(fileSize > 2000000) {
                alert("File size should be less than 2 MB");
                document.getElementById("divMsg").hidden = true;
                return;
            }

            if (files.length == 0) {
                alert("Please choose file...");
                document.getElementById("divMsg").hidden = true;
                return;
            }

            var filename = files[0].name;
            var extension = filename.substring(filename.lastIndexOf(".")).toUpperCase();

            if (extension == '.XLS' || extension == '.XLSX') {
                excelFileToJSON(files[0]);
            } else {
                alert("Please select a valid excel file.");
                document.getElementById("divMsg").hidden = true;
            }

            document.getElementById("sampleFile").value = null;
        }

        /*Method to read excel file and convert it into JSON*/
        function excelFileToJSON(file) {
            try {
                var reader = new FileReader();
                reader.readAsBinaryString(file);
                reader.onload = function (e) {

                    var data = e.target.result;
                    var workbook = XLSX.read(data, {
                        type: 'binary'
                    });

                    /*Gets all the sheetnames of excel in to a variable*/
                    var sheet_name_list = workbook.SheetNames;

                    var result = {};

                    var roa = XLSX.utils.sheet_to_row_object_array(workbook.Sheets[sheet_name_list[0]]);

                    if (!roa[0].hasOwnProperty("AuditStatus") || !roa[0].hasOwnProperty("BranchCode")) {
                        alert("Invalid Excel. Please Download Sample File")
                        document.getElementById("divMsg").hidden = true;
                        return false;
                    }

                    if (roa.length < 5) {
                        document.getElementById("divMsg").hidden = true;
                        //$('#recordLessThan5').modal('show');
                        $('#recordLessThan5').modal({
                            backdrop: 'static',
                            keyboard: false,
                            modal: true
                        });
                        return false;
                    }

                    if (roa.length > 0) {
                        result[sheet_name_list[0]] = roa;
                    } else {
                        alert("Data Not Found !!!")
                        document.getElementById("divMsg").hidden = true;
                        return false;
                    }




                    var validStatus = ['I', 'N', 'A'];

                    $.each(roa, function (key, value) {
                        // console.log("Outer Each");
                        // console.log(key + ": " + value);
                        // console.log(key + ": " + value["AuditStatus"]);
                        //
                        // if (!value.hasOwnProperty("AuditStatus") || !value.hasOwnProperty("BranchCode")) {
                        //     alert("Invalid Excel. Please Download Sample File")
                        //     return false;
                        // }

                        var isBranchCodeValid = 1;
                        var isErrorStatusValid = 1;

                        var errorBrachCode = "";
                        var errorStatus = "";

                        var styleBC = "border:1px; text-transform:uppercase;";
                        var styleAS = "border:1px; text-transform:uppercase;";

                        var spanBC = '';
                        var spanAS = '';
                        var branchCode = '';

                        value["AuditStatus"] = value["AuditStatus"].toUpperCase();

                        if (!($.isNumeric(value["BranchCode"])) || value["BranchCode"].toString().length > 5) {
                            console.log("LENGTH INSIDE IF : " + value["BranchCode"].length);
                            errorBrachCode = "*Invalid Branch Code Format";
                            styleBC = "border:1px solid #ff0000; text-transform:uppercase;";
                            isBranchCodeValid = 0;
                            spanBC = '<span class="tooltiptext">*Invalid Branch Code Format</span>';
                            branchCode = value["BranchCode"].toString();
                        } else {
                            branchCode = value["BranchCode"].toString().padStart(5, '0');
                        }


                        if (jQuery.inArray(value["AuditStatus"], validStatus) == -1) {
                            errorStatus = "*Invalid Audit Status. Valid Values are N, I or A";
                            isErrorStatusValid = 0;
                            styleAS = "border:1px solid #ff0000; text-transform:uppercase;";
                            spanAS = '<span class="tooltiptext">*Invalid Audit Status. Valid Values are N, I, A</span>';
                        }

                        dt.row.add([
                            '<input type="checkbox" name="row[]">',
                            '<a class="tooltip">'
                            + '<input type="text" class="form-control" name="branchcode" maxlength="5" style="' + styleBC + ' text-align: center" value="' + branchCode + '">'
                            + spanBC
                            + '<input type="hidden" name="isValid" value="' + isBranchCodeValid + '">'
                            + '</a>',
                            '<a class="tooltip">'
                            + '<input type="text" class="form-control" name="status" maxlength="1" style="' + styleAS + ' text-align: center" value="' + value["AuditStatus"] + '">'
                            + spanAS
                            + '<input type="hidden" name="isValid" value="' + isErrorStatusValid + '">'
                            + '</a>',
                        ]).draw(false);

                    });
                    document.getElementById("showTable").hidden = false;
                    document.getElementById("button").hidden = false;

                    var values = $("input[name='isValid']").map(function () {
                        return $(this).val();
                    }).get();
                    document.getElementById("divMsg").hidden = true;
                    console.log("Upload Stop...." + new Date().toDateString() + "-" + new Date().toLocaleTimeString());
                }
            } catch (e) {
                console.error(e);
            }
        }

        function wait() {
            document.getElementById("divMsg").hidden = false;
        }

        //submitButton
        $(document).ready(function(){
            $("#submitButton").bind("click",function() {
                $("#submitButton").attr("disabled",true);
                document.getElementById("divMsg").hidden = false;
                $("input[name='branchcode']").attr('readonly', true);
                $("input[name='status']").attr('readonly', true);
                //setTimeout(function() { $('#list').fadeOut();}, 2000);
                setTimeout(submitRequest, 1000);

            });
        });

        function submitRequest() {
            console.log("Submit FE start...." + new Date().toDateString() + "-" + new Date().toLocaleTimeString());
            //$(this).setAttribute('mydata','myData');
            ////$("#submitButton").attr("disabled",true);
            ////document.getElementById("divMsg").hidden = false;

            ////$("input[name='branchcode']").attr('readonly', true);
            ////$("input[name='status']").attr('readonly', true);

            /*
            console.log($("input[name=isValid]").val());

            var values = $("input[name='isValid']").map(function () {
                return $(this).val();
            }).get();
            console.log(values);
            */

            ///////////////////////////////////
            var isValid = [];
            $("input[name='isValid']").each(function () {
                if ($(this).val() == 0) {
                    isValid.push($(this).val());
                    return false;
                }
            });
            ///////////////////////////////////////

            if (jQuery.inArray('0', isValid) == -1) {  //if (jQuery.inArray('0', values) == -1) {
                ////////////////////////////////
                //https://www.gyrocode.com/articles/jquery-datatables-how-to-submit-all-pages-form-data/
                //https://www.gyrocode.com/articles/jquery-datatables-how-to-submit-all-pages-form-data/#solution-direct
                //https://www.gyrocode.com/articles/jquery-datatables-how-to-submit-all-pages-form-data/#solution-ajax
                //var form = this;
                var form = $("#formBulkStatusUpload");

                // Encode a set of form elements from all pages as an array of names and values
                var params = dt.$('input[name="branchcode"], input[name="status"]').serializeArray();
                //console.log(params);
                //console.log(form[this.name]);
                //console.log(this.name);

                dt.$('input[name="branchcode"], input[name="status"]').remove();
                //$('#form-delete [my-input]').remove();

                // Iterate over all form elements
                $.each(params, function(){
                    // If element doesn't exist in DOM
                    //if(!$.contains(document, form[this.name])){
                        // Create a hidden element

                        $(form).append(
                            $('<input>')
                                .attr('type', 'hidden')
                                .attr('name', this.name)
                                .val(this.value)
                        );
                    //}
                });

                console.log("Submit UI Stop...." + new Date().toDateString() + "-" + new Date().toLocaleTimeString());
                /////////////////////////////////////////////////

                $("#formBulkStatusUpload").submit();

                setTimeout(showWarningModal, 7000);
            } else {
                alert("Please Correct Invalid Data First");

                return false;
            }


        }

        function refreshPage() {
            window.location.href = "./bulkUpload"
        }

        function goToSingleRequest() {
            window.location.href = "./singleBranch"
        }

        function showWarningModal() {
            //$('#warnModal').modal('show');
            $('#warnModal').modal({
                backdrop: 'static',
                keyboard: false,
                modal: true
            });
        }

    </script>

</head>
<body class="hold-transition skin-blue sidebar-mini">
<div class="wrapper">

    <!-- Content Wrapper. Contains page content -->
    <div class="content-wrapper">
        <!-- Content Header (Page header) -->
        <section class="content-header">

            <h1 <%--style="background: rgb(222,208,203)"--%>>
                Change Multiple Branch Audit Status
            </h1>


        </section>

        <!-- Main content -->
        <section class="content">


            <div class="row">
                <div class="col-xs-12">
                    <div class="box box-solid box-primary">
                        <div class="box-body">
                            <div class="box-header">
                                <p style="color: red">[Note: For bulk upload, atleast 5 records should be present.]</p>
                            </div>

                            <div class="row">
                                <div class="col-md-2">
                                    <label class="form-label">Download Template</label>
                                    <br>
                                    <a href="./downloadExcel" type="button" class="btn btn-success">
                                        Excel &nbsp;<i class="fa fa-cloud-download"></i>
                                    </a>
                                </div>
                                <div class="col-md-4">

                                </div>
                                <div class="col-md-2">
                                    <label class="form-label">Guide</label>
                                    <p>
                                        N - For Marking Branch As Non Audited
                                        <br>
                                        I - For Marking As IFCOFR Audited
                                        <br>
                                        A - For marking As Audited (Non-IFCOFR)

                                    </p>
                                </div>
                            </div>
                            <div class="row" style="margin-top: 10px">
                                <div class="col-md-2">
                                    <label for="sampleFile" class="form-label">Upload Excel File</label>
                                    <input class="form-control" type="file" id="sampleFile" name="sampleFile">
                                </div>
                                <div class="col-md-1" style="margin-top: 4px">
                                    <label for="sampleFile" class="form-label"></label>
                                    <button type="button" class="btn btn-primary btn-block" onclick="upload()">
                                        Upload &nbsp;<i class="fa fa-cloud-upload"></i>
                                    </button>
                                </div>
                            </div>

                            <form:form action="bulkUpload" method="post" id="formBulkStatusUpload"
                                       cssStyle="margin-top: 15px">

                                <input type="hidden" name="csrfPreventionSalt"
                                       value="<c:out value='${csrfPreventionSalt}'/>"/>
                                <input type="hidden" name="makerid" id="makerid" value='${sessionScope.userId}'/>
                                <input type="hidden" name="cureentBranchcode" id="cureentBranchcode"
                                       value='${sessionScope.branchCode}'/>
                                <input type="hidden" name="quaterEndDate" id="quaterEndDate"
                                       value='${sessionScope.quaterEndDate}'/>
                                <input type="hidden" name="rtStatus" id="rtStatus" value='1'/>
                                <input type="hidden" name="rtType" id="rtType" value='A'/>
                                <input type="hidden" name="rtSubType" id="rtSubType" value='M'/>
                                <input type="hidden" name="requestStatus" id="requestStatus" value='P'/>


                                <%--//https://www.w3schools.com/howto/howto_js_filter_table.asp--%>
                                <div class="col-md-4" id="showTable" hidden>
                                    <table id="UploadTable" class="table table-bordered table-hover dataTable">
                                        <thead>
                                        <tr style="background-color: #b9def0">
                                            <th></th>
                                            <th style="height: 34px; text-align: center">Branch Code</th>
                                            <th style="height: 34px; text-align: center">Is Auditable (I/A/N)</th>
                                        </tr>
                                        </thead>
                                        <tbody>

                                        </tbody>
                                    </table>
                                </div>
                            </form:form>

                        </div>
                        <div class="box-footer">
                            <div id="button" hidden>
                                <div class="col-xs-1">
                                    <button class="btn btn-bg bg-olive btn-block" id="submitButton" >
                                        Submit &nbsp;<i class="fa fa-floppy-o"></i>
                                    </button>
                                </div>
                                <div class="col-xs-1">
                                    <button class="btn btn-bg bg-maroon btn-block" id="idDiscard">
                                        Discard &nbsp;<i class="fa fa-trash-o"></i>
                                    </button>
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


<!-- Modal Start -->

<!-- Success Modal Start -->
<div class="example-modal">
    <div class="modal fade" id="submitModal">
        <div class="modal-dialog">
            <div class="modal-content">
                <div class="modal-header bg-success">
                    <button type="button" class="close" data-dismiss="modal" aria-label="Close">
                        <span aria-hidden="true">&times;</span></button>
                    <h4 class="modal-title"><b>Success</b></h4>
                </div>
                <div class="modal-body">
                    <p>${displayMessage}</p>
                </div>
                <div class="modal-footer">
                    <button type="button" class="btn btn-success" data-dismiss="modal">
                        Close
                    </button>
                </div>
            </div>
        </div>
    </div>
</div>
<!-- Success Modal End -->

<!-- Warning Modal Start -->
<div class="example-modal">
    <div class="modal fade" id="warnModal">
        <div class="modal-dialog">
            <div class="modal-content">
                <div class="modal-header bg-danger">

                    <h4 class="modal-title">Warning!</h4>
                </div>
                <div class="modal-body">
                    <p>Valid records request has been successfully uploaded.
                        <br> Invalid Records Found. Please check the downloaded error file for invalid records.
                    </p>
                </div>
                <div class="modal-footer">
                    <button type="button" class="btn btn-danger" data-dismiss="modal" onclick="refreshPage()">Close
                    </button>
                </div>
            </div>
        </div>
    </div>
</div>
<!-- Warning Modal End -->

<!-- Records Less Than Start -->
<div class="example-modal">
    <div class="modal fade" id="recordLessThan5">
        <div class="modal-dialog">
            <div class="modal-content">
                <div class="modal-header bg-info">
                    <button type="button" class="close" data-dismiss="modal" aria-label="Close">
                        <span aria-hidden="true">&times;</span></button>
                    <h4 class="modal-title">Warning!</h4>
                </div>
                <div class="modal-body">
                    <p>For bulk upload: atleast 5 records should be present. Please use single branch request update</p>
                </div>
                <div class="modal-footer">
                    <button type="button" class="btn btn-default pull-left" data-dismiss="modal">Close
                    </button>
                    <button type="button" class="btn btn-info" data-dismiss="modal" onclick="goToSingleRequest()"><u>Go To Single Branch</u>
                    </button>
                </div>
            </div>
        </div>
    </div>
</div>
<!-- Records Less Than End -->

<!-- Modal End -->

<script>

    var dt = $("#UploadTable").DataTable({
        searching: true,
        ordering: false
    });

    $(function () {

        $("#idDiscard").click(function () {
            /*https://www.gyrocode.com/articles/jquery-datatables-how-to-add-a-checkbox-column/*/
            dt.$('input[type="checkbox"]').each(function () {
                if (this.checked) {
                    // Create a hidden element
                    this.closest("tr").remove();
                }
            });
        });

        $('body').on("keyup", "input[name=branchcode]", function () {
            var branchCode = $(this).val();
            console.log(branchCode);

            $(this).removeAttr("style");
            $(this).siblings("span").remove();

            //if (($.isNumeric(branchCode)) && branchCode.toString().length == 5) {

            if (($.isNumeric(branchCode)) && branchCode.toString().length == 5) {
                $(this).siblings("input").val(1);
                $(this).attr("style", "border : 1px; text-transform:uppercase; text-align: center;");
            } else {
                $(this).siblings("input").val(0);
                $(this).attr("style", "border : 1px solid #ff0000; text-transform:uppercase; text-align: center;");
                $('<span class="tooltiptext">*Invalid Branch Code Format</span>').insertAfter(this);
            }

        });

        $('body').on("change", "input[name=branchcode]", function () {
            var branchCodeEntered = $(this).val().toString().padStart(5, '0');
            $(this).val(branchCodeEntered);

            var branchCode = $(this).val();

            $(this).removeAttr("style");
            $(this).siblings("span").remove();

            if (($.isNumeric(branchCode)) && branchCode.toString().length == 5) {
                $(this).siblings("input").val(1);
                $(this).attr("style", "border : 1px; text-transform:uppercase; text-align: center;");
            } else {
                $(this).siblings("input").val(0);
                $(this).attr("style", "border : 1px solid #ff0000; text-transform:uppercase; text-align: center;");
                $('<span class="tooltiptext">*Invalid Branch Code Format</span>').insertAfter(this);
            }
        });


        $('body').on("keyup", "input[name=status]", function () {
            var validStatus = ['N', 'I', 'A'];
            var status = $(this).val().toUpperCase();

            $(this).val(status);
            $(this).siblings("span").remove();
            $(this).removeAttr("style");

            if (jQuery.inArray(status, validStatus) == -1) {
                $(this).siblings("input").val(0);
                $(this).attr("style", "border : 1px solid #ff0000; text-transform:uppercase; text-align: center;");
                $('<span class="tooltiptext">*Invalid Audit Status. Valid Values are N, I, A</span>').insertAfter(this);
            } else {
                $(this).attr("style", "border : 1px; text-transform:uppercase; text-align: center;");
                $(this).siblings("input").val(1);
            }
        });

        var displayMessage = '${displayMessage}';
        if (displayMessage) {
            //$('#submitModal').modal('show');

            $('#submitModal').modal({
                backdrop: 'static',
                keyboard: false,
                modal: true
            });
        }

    });
</script>


</body>
</html>

/////////////////////////////////////////////////////////

package com.tcs.controllers;

import com.tcs.beans.*;
import com.tcs.services.FRTMakerService;
import com.tcs.utils.CommonConstants;
import org.apache.log4j.Logger;
import org.apache.poi.hssf.usermodel.HSSFWorkbook;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Controller;
import org.springframework.validation.BindingResult;
import org.springframework.web.bind.WebDataBinder;
import org.springframework.web.bind.annotation.InitBinder;
import org.springframework.web.bind.annotation.ModelAttribute;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
import org.springframework.web.servlet.ModelAndView;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import javax.servlet.http.HttpSession;
import java.math.BigDecimal;
import java.time.LocalDateTime;
import java.time.format.DateTimeFormatter;
import java.util.*;

@Controller
@RequestMapping("/FRTUser")
public class FrtMakerController {

    static Logger log = Logger.getLogger(FrtMakerController.class.getName());
    @Autowired
    FRTMakerService frtMakerService;

    @InitBinder("bulkUpload")
    public void initBinderbulkupload(WebDataBinder binder) {
        binder.setDisallowedFields(new String[]{});
    }

    @RequestMapping(value = "/bulkUpload", method = RequestMethod.GET)
    public ModelAndView bulkUpload(@ModelAttribute("command") Frt frt, BindingResult result, HttpServletRequest request) {
        HttpSession session = request.getSession();
        if (session == null || session.getAttribute(CommonConstants.USER_ID) == null ||
                request.getSession().getId() == null ||
                !Objects.equals(session.getAttribute(CommonConstants.USER_CAPABILITY), new BigDecimal("96")))
        {
            log.info("User check ");
            return new ModelAndView("500");
        }

        /*Reading Session data
        String userIDD = (String) request.getSession().getAttribute("userId");
        System.out.println(userIDD);
        Enumeration<String> attributes = request.getSession().getAttributeNames();
        while (attributes.hasMoreElements()) {
            String attribute = (String) attributes.nextElement();
            System.out.println(attribute+" : "+request.getSession().getAttribute(attribute));
        }*/


        return new ModelAndView("FRTUser/FRTMultipleBranch");
    }

    @RequestMapping(value = "/downloadExcel", method = RequestMethod.GET)
    public ModelAndView downloadExcel(HSSFWorkbook workbook, HttpServletRequest request, HttpServletResponse response) {
        List<ExcelCol> list = new ArrayList<ExcelCol>();
        // return a view which will be resolved by an excel view resolver
        return new ModelAndView("excelView", "listBooks", list);
    }

    /*--------------Mass Assignment: Insecure Binder Configuration-------------*/
    @InitBinder("submit")
    public void initBindercreate(WebDataBinder binder) {
        binder.setDisallowedFields(new String[]{});
    }

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

    public boolean containsRequest(final List<BranchAuditRequest> list, final String name) {
        return list.stream().map(BranchAuditRequest::getBranchcode).filter(name::equals).findFirst().isPresent();
    }

    public boolean containsName(final List<BranchMaster> list, final String name) {
        //https://stackoverflow.com/questions/18852059/java-list-containsobject-with-field-value-equal-to-x
        //https://www.w3docs.com/snippets/java/java-list-contains-object-with-field-value-equal-to-x.html
        return list.stream().map(BranchMaster::getBranchcode).filter(name::equals).findFirst().isPresent();
    }

    public boolean containsSameStatus(final List<BranchMaster> list, final String name, final String status) {
        //https://stackoverflow.com/questions/36089484/check-single-element-into-arraylist-with-multiple-column
        boolean sameStatus = false;
        Optional<BranchMaster> elemOpt = list.parallelStream().filter(elem -> elem.getBranchcode().equals(name)).findAny();
        if (elemOpt.isPresent()) {
            BranchMaster elem = elemOpt.get();

            if (status.equalsIgnoreCase(elem.getStatus())) {
                sameStatus = true;
            }
        }

        return sameStatus;
    }


}

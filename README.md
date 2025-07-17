<%--
  Created by IntelliJ IDEA.
  User: V1009204
  Date: 17-03-2023
  Time: 10:51
  To change this template use File | Settings | File Templates.
--%>
<%@ include file="/views/include.jsp" %>
<!DOCTYPE html>
<html>
<head>
    <script type="text/javascript">

        /*function blockCheck(a) {
            var count = '${strRuleListCount}';
            var checkBlock = document.getElementById("detail2"+a).style.display;
            console.log("count: "+count+" checkBlock: "+checkBlock);
            for (var j1 = 1; j1 < count; j1++) {
                if (document.getElementById("detail2" + j1).style.display == 'block') {
                    document.getElementById("detail2" + j1).style.display = 'none';
                    document.getElementById("detail1" + j1).style.backgroundColor = 'white';
                    document.getElementById("detail1" + j1).style.backgroundColor = 'white';
                }
                else{
                    colorChange(a);
                }
            }

        }*/
        function colorChange(a) {
            var branchCode = document.getElementById("branchCode"+a).value;
            var type = document.getElementById("requestType"+a).value;
            var det2disp = document.getElementById("detail2"+a).style.display;
            //console.log("type: "+type +" det2disp: "+det2disp);
            if(type == 'Add Branch'){
                if(det2disp == 'block'){
                    document.getElementById("detail2"+a).style.backgroundColor = 'white';
                    document.getElementById("detail1"+a).style.backgroundColor = 'white';
                    //console.log("if: "+det2disp);
                }
                else {
                    document.getElementById("detail1" + a).style.backgroundColor = '#88e774';
                    document.getElementById("detail2" + a).style.backgroundColor = '#bce78e';
                }
            }
            if(type == 'Delete Branch'){
                document.getElementById('dataTable'+a).innerHTML = "";
                $.ajax({
                    type: "POST",
                    url: "./getDeleteReportList",
                    data: {
                        branchCode: branchCode
                    },
                    success: function (data) {
                        //console.log("data: " +JSON.stringify(data)+" dataCount: " +data.length);
                        if(data.length==1){
                            document.getElementById("deleteList"+a).hidden = true;
                            document.getElementById("tableMsg"+a).hidden = true;
                            //console.log("data Nil!");
                        }
                        else{
                            document.getElementById("deleteList"+a).hidden = false;
                            document.getElementById("tableMsg"+a).hidden = false;
                            var table= document.getElementById('dataTable'+a);
                            //console.log("data present!");
                            for (var i=0; i<data.length; i++) {
                                var row = table.insertRow('-1');
                                for (var j=0; j<data[i].length; j++) {
                                    //console.log("j: "+j+" data: "+ data[i]+" data: "+ data[i].length);
                                    var cell = row.insertCell('-1');
                                    var element = document.createElement("input");
                                    element.type = "text";
                                    element.class = "form-control";

                                    if(i==0){
                                        element.style="width:100%; text-align: center; background-color: #999999; border-style: hidden; color: white; font-weight: 800";
                                        if(j==0){
                                            cell.style="background-color: #999999; width: 80px";
                                            element.style="width:100%; text-align: center; background-color: #999999; border-style: hidden;color: white; font-weight: 800";
                                            element.value = data[i][j];
                                        }
                                        else if(j==1){
                                            cell.style="background-color: #999999; width: 90px";
                                            element.style="width:100%; text-align: center; background-color: #999999; border-style: hidden;color: white; font-weight: 800";
                                            element.value = data[i][j];
                                        }
                                        else if(j==2){
                                            cell.style="background-color: #999999;";
                                            element.style="width:100%; text-align: center; background-color: #999999; border-style: hidden;color: white; font-weight: 800";
                                            element.value = data[i][j];
                                        }
                                        else if(j==3){
                                            cell.style="background-color: #999999; width: 150px";
                                            element.style="width:100%; text-align: center; background-color: #999999; border-style: hidden;color: white; font-weight: 800";
                                            element.value = "PENDING WITH";
                                        }
                                    }
                                    else{
                                        element.style="width:100%; text-align: center; border-style: hidden";
                                        element.value = data[i][j];
                                        if(j==2){
                                            element.style="width:100%; text-align: left; border-style: hidden;";
                                        }
                                        else if(j==3){
                                            element.style="width:100%; text-align: left; border-style: hidden;";
                                            if(element.value.substring(0,1)=='1'){
                                                element.value = "Maker";
                                            }
                                            else if(element.value.substring(0,1)=='2'){
                                                element.value = "Branch Manager";
                                            }
                                            else if(element.value.substring(0,1)=='3'){
                                                element.value = "Branch Auditor";
                                            }
                                            else if(element.value.substring(0,1)=='4'){
                                                element.value = "RO Manager";
                                            }
                                            else if(element.value.substring(0,1)=='5'){
                                                element.value = "Freezed";
                                            }

                                        }
                                    }
                                    element.readOnly = "true";
                                    cell.appendChild(element);
                                }
                            }
                        }
                    },
                    error: function (jqXHR, textStatus, errorMessage) {
                        var msg = "";
                    },

                    complete: function () {
                        /*$.unblockUI();*/
                    },
                });
                if(det2disp == 'block'){
                    document.getElementById("detail1"+a).style.backgroundColor = 'white';
                    document.getElementById("detail2"+a).style.backgroundColor = 'white';
                }
                else {
                    document.getElementById("detail1"+a).style.backgroundColor = '#ff7878';
                    document.getElementById("detail2"+a).style.backgroundColor = '#fcc1c1';
                }
            }
        }

        function approveReq(a) {
            document.getElementById("divMsg").hidden = false;
            var reqId = document.getElementById("req_Id"+a).value;
            var branchCode = document.getElementById("branchCode"+a).value;
            var reqtype = document.getElementById("requestType"+a).value;
            var auditStatus = document.getElementById("auditStatus"+a).value;
            var reqBy = document.getElementById("requestedById"+a).value;
            var circleCode = document.getElementById("cCode"+a).value;
            var circleName = document.getElementById("circleCode"+a).value;
            var finalCount = 0;
            if (reqtype=='Add Branch'){
                var branchCount = 0;
                branchCount = parseInt(document.getElementById("noOfBranches"+a).value);
                finalCount = branchCount + 1;
            }
            else if(reqtype=='Delete Branch'){
                branchCount = 0;
                branchCount = document.getElementById("noOfBranches"+a).value;
                finalCount = branchCount - 1;
            }
            var roCode = document.getElementById("roCode"+a).value;
            console.log("a: "+a +" reqId: "+reqId+" cCode: "+circleCode+" roCode: "+roCode +" reqBy: "+reqBy);
            $.ajax({
                type: "POST",
                url: "./acceptReq",
                data: {
                    reqId : reqId,
                    branchCode : branchCode,
                    reqType : reqtype,
                    auditStatus : auditStatus,
                    reqBy: reqBy,
                    circleCode: circleCode,
                    regionCode: roCode
                },

                success: function (data) {
                    console.log("data: "+data);
                    if(data){
                        if(reqtype=='Add Branch') {
                            var msg = "The Branch " + branchCode + " is added to CRS Scope. The revised count of branches in "+circleName+ " Circle is "+finalCount+".";
                            $('#errorModal .modal-body').text(msg);
                            $('#errorModal .modal-title').text("Branch Added");
                            $('#errorModal').modal('show');
                        }
                        else if(reqtype=='Delete Branch'){
                            var msg = "The Branch " + branchCode + " is removed from CRS Scope. The revised count of branches in "+circleName+ " Circle is "+finalCount+".";
                            $('#errorModal .modal-body').text(msg);
                            $('#errorModal .modal-title').text("Branch Deleted");
                            $('#errorModal').modal('show');
                        }

                    }else{
                        var msg = "The request to "+reqType+" "+branchCode+" failed. Kindly try again after sometime.";
                        $('#rejectModal .modal-body').text(msg);
                        $('#rejectModal .modal-title').text("Try Again!");
                        $('#rejectModal').modal('show');
                    }
                },
                error: function (jqXHR, textStatus, errorMessage) {
                    alert("No response received");
                },

                complete: function () {
                    $.unblockUI();
                },
            });

        }

        function rejectReq(a) {
            var reqId = document.getElementById("req_Id"+a).value;
            var reqType = document.getElementById("requestType"+a).value;
            var branchCode = document.getElementById("branchCode"+a).value;
            console.log("a: "+a +" reqId: "+reqId);

            $.ajax({
                type: "POST",
                url: "./rejectReq",
                data: {
                    reqId:reqId
                },

                success: function (data) {
                    console.log("data: "+data);
                    if(data){
                        var msg = "The request to "+reqType+" "+branchCode+" is rejected. No change in number of branches.";
                        $('#rejectModal .modal-body').text(msg);
                        $('#rejectModal .modal-title').text("Request Rejected");
                        $('#rejectModal').modal('show');
                    }
                    else{
                        var msg = "The request to "+reqType+" "+branchCode+" failed. Kindly try again after sometime.";
                        $('#rejectModal .modal-body').text(msg);
                        $('#rejectModal .modal-title').text("Try Again!");
                        $('#rejectModal').modal('show');
                    }
                },
                error: function (jqXHR, textStatus, errorMessage) {
                    alert("No response received");
                },

                complete: function () {
                    $.unblockUI();
                },
            });
        }

        function reload() {
            window.location.replace('../FRTChecker/FRTBranchReq');
        }
    </script>
</head>
<body class="hold-transition skin-blue sidebar-mini">
<div class="wrapper">
    <div class="content-wrapper">
        <section class="content-header">
            <h1>CRS Scope Change Requests</h1>
        </section>
        <section class="content">
            <div class="row">
                <div class="col-xs-12">
                    <div class="box box-primary box-solid">
                        <div class="box-body">
                            <div class="box-header">
                                <div class="col-md-12">
                                    <div class="row" style="background: #b9def0; height: 40px;">
                                        <div class="col-md-1" style="text-align: center; margin-top: 10px;"><b>Request ID</b></div>
                                        <div class="col-md-1" style="text-align: center; margin-top: 10px;"><b>Branch</b></div>
                                        <div class="col-md-3" style="text-align: center; margin-top: 10px;"><b>Branch Name</b></div>
                                        <div class="col-md-2" style="text-align: center; margin-top: 10px;"><b>Request Type</b></div>
                                        <div class="col-md-2" style="text-align: center; margin-top: 10px;"><b>Status</b></div>
                                        <div class="col-md-3" style="text-align: center; margin-top: 10px;"><b>Requested on</b></div>
                                    </div>
                                </div>
                            </div>
                            <form:form action="" method="POST" id="frtBranchReq" style="overflow-y: auto; height: 70vh;">
                                    <c:if test="${strRuleListCount!=0}">
                                        <c:forEach items="${list}" var="frtBranchReq" varStatus="reportStatus">
                                            <input type="hidden" value="<c:out value="${frtBranchReq.requestedById}"></c:out>"
                                                   id="requestedById<c:out value='${reportStatus.count}'></c:out>">
                                            <input type="hidden" value="<c:out value="${frtBranchReq.cCode}"></c:out>"
                                                   id="cCode<c:out value='${reportStatus.count}'></c:out>">
                                            <div class="col-md-12">
                                                <div class="box collapsed-box box-solid">
                                                    <div class="box-header" id="detail1<c:out value='${reportStatus.count}'></c:out>">
                                                        <div class="row">
                                                            <div class="col-md-1" style="text-align: center;">
                                                                <input type="hidden" id="req_Id<c:out value='${reportStatus.count}'></c:out>"
                                                                       name="req_Id" readonly="true" value='<c:out value="${frtBranchReq.req_Id}"></c:out>'/>
                                                                <c:out value="${frtBranchReq.req_Id}"></c:out>
                                                            </div>
                                                            <div class="col-md-1" style="text-align: center;">
                                                                <input type="hidden" value="<c:out value="${frtBranchReq.branchCode}"></c:out>"
                                                                 id="branchCode<c:out value='${reportStatus.count}'></c:out>">
                                                                <c:out value="${frtBranchReq.branchCode}"></c:out></div>
                                                            <div class="col-md-3" style="text-align: center;">
                                                                <input type="hidden" value="<c:out value="${frtBranchReq.branchName}"></c:out>"
                                                                       id="branchName<c:out value='${reportStatus.count}'></c:out>">
                                                                <c:out value="${frtBranchReq.branchName}"></c:out></div>
                                                            <div class="col-md-2" style="text-align: center;">
                                                                <input type="hidden" value="<c:out value="${frtBranchReq.requestType}"></c:out>"
                                                                       id="requestType<c:out value='${reportStatus.count}'></c:out>">
                                                                <c:out value="${frtBranchReq.requestType}"></c:out></div>
                                                            <div class="col-md-2" style="text-align: center;">
                                                                <input type="hidden" value="<c:out value="${frtBranchReq.status}"></c:out>"
                                                                       id="status<c:out value='${reportStatus.count}'></c:out>">
                                                                <c:out value='${frtBranchReq.status}'></c:out></div>
                                                            <div class="col-md-3" style="text-align: center;">
                                                                <input type="hidden" value="<c:out value="${frtBranchReq.requestedOn}"></c:out>"
                                                                       id="requestedOn<c:out value='${reportStatus.count}'></c:out>">
                                                                <c:out value="${frtBranchReq.requestedOn}"></c:out></div>
                                                        </div>
                                                        <div class="box-tools pull-right">
                                                            <button type="button" class="btn btn-box-tool" data-widget="collapse"
                                                                    id="hide<c:out value='${reportStatus.count}'></c:out>"
                                                                    onclick="colorChange('${reportStatus.count}');"><i class="fa fa-plus"></i></button>
                                                        </div>
                                                    </div>
                                                    <div class="box-body no-padding" style="display: none;" id="detail2<c:out value='${reportStatus.count}'></c:out>">
                                                        <div class="row" style="margin-top: 10px">
                                                            <div class="col-md-2" style="margin-left: 10px; margin-bottom: 20px;">
                                                                <input type="hidden" value="<c:out value="${frtBranchReq.circleCode}"></c:out>"
                                                                       id="circleCode<c:out value='${reportStatus.count}'></c:out>">
                                                                <b>Circle: </b><c:out value="${frtBranchReq.circleCode}"></c:out></div>
                                                            <div class="col-md-2">
                                                                <input type="hidden" value="<c:out value="${frtBranchReq.roCode}"></c:out>"
                                                                       id="roCode<c:out value='${reportStatus.count}'></c:out>">
                                                                <b>RO: </b><c:out value="${frtBranchReq.roCode}"></c:out></div>
                                                            <div class="col-md-2">
                                                                <input type="hidden" value="<c:out value="${frtBranchReq.auditStatus}"></c:out>"
                                                                       id="auditStatus<c:out value='${reportStatus.count}'></c:out>">
                                                                <b>Audited Status: </b><c:out value="${frtBranchReq.auditStatus}"></c:out></div>
                                                            <div class="col-md-2">
                                                                <input type="hidden" value="<c:out value="${frtBranchReq.requestedBy}"></c:out>"
                                                                       id="requestedBy<c:out value='${reportStatus.count}'></c:out>">
                                                                <b>Requested By: </b><c:out value='${frtBranchReq.requestedBy}'></c:out></div>
                                                            <div class="col-md-3"><b>No of Branches in Circle: </b>
                                                                <input type="hidden" value="<c:out value="${frtBranchReq.noOfBranches}"></c:out>"
                                                                       id="noOfBranches<c:out value='${reportStatus.count}'></c:out>">
                                                                <c:out value="${frtBranchReq.noOfBranches}"></c:out></div>
                                                        </div>
                                                        <c:if test="${frtBranchReq.requestType=='Delete Branch'}">
                                                            <div class="row" id="deleteList<c:out value='${reportStatus.count}'></c:out>" hidden>
                                                                <div class="col-md-12">
                                                                    <div class="col-md-1"></div>
                                                                    <div class="col-md-10" style="background-color: #ffffff; overflow-y: auto; height: 30vh;" >
                                                                        <div id="tableMsg<c:out value='${reportStatus.count}'></c:out>">
                                                                            <p style="color: red; margin-top: 10px; margin-left: 14px">
                                                                                *Following reports will be deleted on branch deletion.
                                                                            </p>
                                                                        </div>
                                                                        <div>
                                                                            <table id="dataTable<c:out value='${reportStatus.count}'></c:out>" class="table table-bordered data-table">

                                                                            </table>
                                                                        </div>
                                                                    </div>
                                                                    <div class="col-md-1"></div>
                                                                </div>
                                                            </div>
                                                        </c:if>
                                                        <div class="row">
                                                            <div class="col-md-4 col-lg-4"></div>
                                                            <div class="col-md-1"></div>
                                                            <div class="col-md-4">
                                                                <div class="btn btn-success" style="margin-top: 10px; margin-bottom: 10px; width: 80px;"
                                                                     id="approve<c:out value='${reportStatus.count}'></c:out>"
                                                                     onclick="approveReq(<c:out value='${reportStatus.count}'></c:out>);">Approve</div>
                                                                <div class="btn btn-danger" style="margin-top: 10px; margin-bottom: 10px; width: 80px;"
                                                                     id="reject<c:out value='${reportStatus.count}'></c:out>"
                                                                     onclick="rejectReq(<c:out value='${reportStatus.count}'></c:out>);">Reject</div>
                                                            </div>
                                                        </div>
                                                        <div class="row"></div>
                                                    </div>
                                                </div>
                                            </div>
                                        </c:forEach>
                                    </c:if>
                            </form:form>
                        </div>
                        <div class="overlay" id="divMsg" hidden>
                            <i class="fa fa-spinner fa-spin" style="color: #9a0200"></i>
                        </div>
                    </div>
                </div>
            </div>
        </section>
    </div>

    <div class="example-modal">
        <div class="modal fade" id="errorModal">
            <div class="modal-dialog">
                <div class="modal-content">
                    <div class="modal-header bg-success">
                        <div class="modal-title" style="font-weight: bold;"></div>
                    </div>
                    <div class="modal-body">

                    </div>
                    <div class="modal-footer">
                        <button type="button" class="btn btn-success" data-dismiss="modal" onclick="reload();">Continue</button>
                    </div>
                </div>
            </div>
        </div>
    </div>

    <div class="example-modal">
        <div class="modal fade" id="rejectModal">
            <div class="modal-dialog">
                <div class="modal-content">
                    <div class="modal-header bg-danger">
                        <div class="modal-title" style="font-weight: bold;"></div>
                    </div>
                    <div class="modal-body">

                    </div>
                    <div class="modal-footer">
                        <button type="button" class="btn btn-success" data-dismiss="modal" onclick="reload();">Continue</button>
                    </div>
                </div>
            </div>
        </div>
    </div>
</div>
/////////////////////////////////////////////////////////////////


import React, { useState } from "react";
import {
  TextField,
  Grid,
  MenuItem,
  FormControl,
  Button,
  Typography,
  DialogTitle,
  Divider,
  TableContainer,
  Table,
  TableHead,
  TableRow,
  TableCell,
  TableBody,
} from "@mui/material";
import PhoneIcon from "@mui/icons-material/Phone";
import HomeIcon from "@mui/icons-material/Home";
import PinDropIcon from "@mui/icons-material/PinDrop";
import AccountBalanceRoundedIcon from "@mui/icons-material/AccountBalanceRounded";
import LocationCityRoundedIcon from "@mui/icons-material/LocationCityRounded";
import PhoneAndroidRoundedIcon from "@mui/icons-material/PhoneAndroidRounded";
import CorporateFareRoundedIcon from "@mui/icons-material/CorporateFareRounded";
import WifiCalling3Icon from "@mui/icons-material/WifiCalling3";
import EmergencyShareIcon from "@mui/icons-material/EmergencyShare";
import DnsIcon from "@mui/icons-material/Dns";
import PublicIcon from "@mui/icons-material/Public";
import SensorsIcon from "@mui/icons-material/Sensors";
import LanIcon from "@mui/icons-material/Lan";
import { Container } from "@mui/system";
import DialogContent from "@mui/material/DialogContent";
import DialogContentText from "@mui/material/DialogContentText";
import DialogActions from "@mui/material/DialogActions";
import Dialog from "@mui/material/Dialog";
import axios from "axios";
import Box from "@mui/material/Box";
import { validations } from "../CommonValidations/Validations";
import { useNavigate } from "react-router-dom";
import { encrypt } from "../Security/AES-GCM256";
import SaveIcon from "@mui/icons-material/Save";
import DeleteIcon from "@mui/icons-material/Delete";
import SearchIcon from "@mui/icons-material/Search";
import CircularProgress from "@mui/material/CircularProgress";
import Snackbar from "@mui/material/Snackbar";
import Alert from "@mui/material/Alert";
import { SnackbarProvider } from "notistack";
import Paper from "@mui/material/Paper";
import PersonIcon from "@mui/icons-material/Person";
import { Cached } from "@mui/icons-material";
import {
  AUDITED_REPORT_STATUS_CONSTANTS,
  REPORT_STATUS_CONSTANTS,
} from "../CommonValidations/commonConstants";

const iv = crypto.getRandomValues(new Uint8Array(12)); // for encryption
const ivBase64 = btoa(String.fromCharCode.apply(null, iv)); // for be decryption
const salt = crypto.getRandomValues(new Uint8Array(16)); // for encryption
const saltBase64 = btoa(String.fromCharCode.apply(null, salt)); // for be decryption

export default function FrtMakerDeleteBranchDetails() {
  document.title = "CRS | FRT Delete Branch";
  const navigate = useNavigate();
  const user = JSON.parse(localStorage.getItem("user"));

  const emptyBranchDataMap = {
    NAME: "",
    CIRCLE: "",
    NETWORK: "",
    MODULE: "",
    REGION: "",
    ADDRESS: "",
    CITY: "",
    STATE: "",
    PIN: "",
    STDCODE: "",
    PHONE: "",
    MOBILE: "",
    IPPHONE: "",
    AUDITABLE: "",
    SCOPE: "",
  };

  const emptyAuditorMap = {
    NAME: "",
    TYPE: "",
    MEMNO: "",
    FIRMNO: "",
    FIRMNAME: "",
    ADDR: "",
    CITY: "",
    POST: "",
    EMAIL: "",
    PHONE: "",
  };

  if (user.user_role !== "94" && user.user_role !== "96") {
    navigate("/");
  }
  const [branchDetailErrors, setBranchDetailErrors] = useState({});
  const [openWarningDialog, setOpenWarningDialog] = useState(false);
  const [fetchedData, setFetchedData] = useState({});
  const [error, setError] = useState(false);
  const [branchData, setbranchData] = useState(emptyBranchDataMap);
  const [auditorData, setAuditorData] = useState(emptyAuditorMap);
  const [loadOpen, setLoadOpen] = useState(false);
  const [snackbar, setSnackbar] = useState(null);
  const [branchCode, setBranchCode] = useState("");
  const [circleList, setCircleList] = useState([]);
  const [showDeleteConfirm, setShowDeleteConfirm] = useState(false);
  const [reportsList, setReportsList] = useState([]);
  const [fieldsDisabled, setFieldsDisabled] = useState(true);

  const handleCloseSnackbar = () => setSnackbar(null);
  const handleDialogClose = () => setLoadOpen(false);

  const handleBranchCodeChange = (e) => {
    setError(false);
    let result = validations("numInput", e.target.value);
    if (result === "") {
      setBranchCode(e.target.value);
    } else {
      setSnackbar({ children: result, severity: "error" });
    }
  };

  const handleInputChange = (event) => {
    const { name, value } = event.target;
    const error = validateInputFields(name, value);
    setBranchDetailErrors({ ...branchDetailErrors, [name]: error });
    setbranchData({ ...branchData, [name]: value });
  };

  const validateInputFields = (name, value) => {
    let error = "";
    if (!value) {
      error = "This field is required.";
    } else {
      switch (name) {
        case "MOBILE":
          error = validations("mobileNumber", value);
          break;
        case "PIN":
          error = validations("postCode", value);
          break;
        case "ADDRESS":
        case "CITY":
        case "STATE":
          error = validations("splAlphaNumeric", value);
          break;
        case "MODULE":
        case "NETWORK":
        case "REGION":
          if (!/^\d{3}$/.test(value)) {
            error = "Should be 3 digits only";
          }
          break;
        case "IPPHONE":
        case "PHONE":
          error = validations("numInput", value);
          break;
        case "STDCODE":
          if (!/^\d{2,5}$/.test(value)) {
            error = "STD Code must be between 2 to 5 digits";
          }
          break;
        default:
          break;
      }
    }
    return error;
  };

  const handleReset = () => {
    setbranchData(emptyBranchDataMap);
    setFetchedData({});
    setAuditorData(emptyAuditorMap);
    setCircleList([]);
    setBranchCode("");
    setFieldsDisabled(true);
    setReportsList([]);
    setBranchDetailErrors({});
  };

  const checkIfChanged = () => {
    return Object.keys(branchData).some(
      (key) => branchData[key] !== fetchedData[key]
    );
  };

  const handleSearch = async () => {
    if (branchCode.length < 5) {
      setSnackbar({
        children: "Kindly enter branch code upto 5 digits.",
        severity: "error",
      });
      return;
    }
    setLoadOpen(true);
    try {
      let jsonFormData = JSON.stringify({ branchCode: branchCode });
      await encrypt(iv, salt, jsonFormData).then((r) => (jsonFormData = r));
      let payload = { iv: ivBase64, salt: saltBase64, data: jsonFormData };

      const response = await axios.post(
        "/Server/EditBranch/fetchBranchDetails",
        payload,
        {
          headers: { Authorization: `Bearer ${localStorage.getItem("token")}` },
        }
      );

      if (
        response.data?.result &&
        Object.keys(response.data?.result?.branchData).length !== 0
      ) {
        setCircleList(response.data.result.circleList);
        setbranchData(response.data.result.branchData);
        setFetchedData({ ...response.data.result.branchData });
        setAuditorData(response.data.result.auditorData);
        setFieldsDisabled(false);
      } else {
        setSnackbar({
          children:
            "Branch does not exist. Please contact 'Finance One Core Team'",
          severity: "error",
        });
        handleReset();
      }
    } catch (e) {
      console.error(e);
      setSnackbar({
        children: "An error occurred. Please try again later.",
        severity: "error",
      });
    } finally {
      handleDialogClose();
    }
  };

  /**
   * Fetches reports for the current branch to show in the delete confirmation dialog.
   */
  const fetchReportsForDelete = async () => {
    setLoadOpen(true);
    try {
      let jsonFormData = JSON.stringify({ branchCode: branchCode });
      await encrypt(iv, salt, jsonFormData).then((r) => (jsonFormData = r));
      let payload = { iv: ivBase64, salt: saltBase64, data: jsonFormData };

      const response = await axios.post(
        "/Server/EditBranch/fetchReports",
        payload,
        {
          headers: { Authorization: `Bearer ${localStorage.getItem("token")}` },
        }
      );

      if (response.data?.result) {
        setReportsList(response.data.result.reportList || []);
        setShowDeleteConfirm(true); // Open dialog even if no reports exist
      } else {
        setSnackbar({
          children: "Failed to fetch reports data.",
          severity: "error",
        });
      }
    } catch (e) {
      console.error(e);
      setSnackbar({
        children: "An error occurred. Please try again later.",
        severity: "error",
      });
    } finally {
      handleDialogClose();
    }
  };

  /**
   * Handles the click of the main 'Save' button.
   */
  const handleSubmit = (event) => {
    event.preventDefault();
    const errors = {};
    Object.keys(branchData).forEach((field) => {
      const error = validateInputFields(field, branchData[field]);
      if (error) errors[field] = error;
    });

    setBranchDetailErrors(errors);

    if (Object.keys(errors).length !== 0) {
      setSnackbar({
        children: "Kindly make sure all fields are filled.",
        severity: "error",
      });
    } else if (!checkIfChanged()) {
      setOpenWarningDialog(true);
    } else {
      setLoadOpen(true);
      handleConfirmSubmit(branchData);
    }
  };

  /**
   * Resets all reports for a branch before deletion. Returns true on success.
   */
  const resetAllReports = async () => {
    if (reportsList.length === 0) return true;
    let successCount = 0;
    for (const report of reportsList) {
      try {
        let data = {
          submissionId: report.SUBMISSION_ID,
          reportId: report.REPORT_ID,
          reportType: report.REPORT_TYPE,
          module: report.MODULE,
          method: "resetAll",
        };
        let jsonFormData = JSON.stringify(data);
        await encrypt(iv, salt, jsonFormData).then((r) => (jsonFormData = r));
        let payload = { iv: ivBase64, salt: saltBase64, data: jsonFormData };

        const response = await axios.post(
          "/Server/EditBranch/resetReportsForDeletion",
          payload,
          {
            headers: {
              Authorization: `Bearer ${localStorage.getItem("token")}`,
            },
          }
        );

        if (
          response.data.result?.reset === 1 ||
          response.data.result?.Reset === 1 ||
          response.data.result?.status
        ) {
          successCount++;
        }
      } catch (e) {
        console.error("Failed to reset report:", report.REPORT_ID, e);
      }
    }
    return successCount === reportsList.length;
  };

  /**
   * Handles the final deletion process after user confirmation.
   */
  const handleConfirmDeletion = async () => {
    setShowDeleteConfirm(false);
    setLoadOpen(true);

    const reportsResetSuccess = await resetAllReports();

    if (reportsResetSuccess) {
      const dataForDeletion = { ...branchData, SCOPE: "O" };
      await handleConfirmSubmit(dataForDeletion);
    } else {
      setSnackbar({
        children: "Could not delete all associated reports. Aborting.",
        severity: "error",
      });
      handleDialogClose();
    }
  };

  /**
   * Submits data to the backend for both 'Update' and 'Delete' operations.
   */
  const handleConfirmSubmit = async (dataToSubmit) => {
    try {
      let jsonFormData = JSON.stringify({ branchData: dataToSubmit });
      await encrypt(iv, salt, jsonFormData).then((r) => (jsonFormData = r));
      let payload = { iv: ivBase64, salt: saltBase64, data: jsonFormData };

      const response = await axios.post(
        "/Server/EditBranch/FrtSubmitData",
        payload,
        {
          headers: { Authorization: `Bearer ${localStorage.getItem("token")}` },
        }
      );

      if (response.data?.result?.status) {
        let message = "";
        if (dataToSubmit.SCOPE === "I") {
          message = `Branch ${branchCode} details have been successfully updated.`;
        } else {
          message = `Branch ${branchCode} has been successfully deleted.`;
        }
        handleReset();
        setSnackbar({ children: message, severity: "success" });
      } else {
        setSnackbar({
          children: "An error occurred. Please try again later.",
          severity: "error",
        });
      }
    } catch (e) {
      console.error(e);
      setSnackbar({
        children: "An error occurred. Please try again later.",
        severity: "error",
      });
    } finally {
      handleDialogClose();
    }
  };

  return (
    <>
      <Box sx={{ display: "flex", height: 50, alignItems: "bottom" }}>
        <Typography
          variant="h5"
          gutterBottom
          sx={{ textAlign: "flex-start", p: 2 }}
        >
          Delete Branch
        </Typography>
      </Box>
      <Divider />
      <Container
        maxWidth={false}
        disableGutters
        sx={{
          display: "flex",
          justifyContent: "center",
          alignItems: "center",
          flexDirection: "column",
        }}
      >
        <Box
          sx={{
            width: "100%",
            border: "1px solid #ddd",
            borderRadius: "8px",
            padding: 4,
            display: "flex",
            flexDirection: "column",
            backgroundColor: "#fff",
          }}
        >
          <Grid container spacing={2}>
            {/* Search and Branch Name */}
            <Grid item xs={12} sm={4}>
              <TextField
                label="Branch Code"
                name="CODE"
                error={error}
                helperText={error && "This field is required."}
                onChange={handleBranchCodeChange}
                value={branchCode}
                disabled={!fieldsDisabled}
                variant="outlined"
                fullWidth
                inputProps={{ maxLength: 5 }}
              />
            </Grid>
            <Grid item xs={12} sm={2}>
              <Button
                variant="contained"
                disableElevation
                startIcon={<SearchIcon />}
                sx={{ mt: 1 }}
                onClick={handleSearch}
              >
                Search
              </Button>
              &nbsp;
              <Button
                variant="contained"
                color="warning"
                disableElevation
                startIcon={<Cached />}
                sx={{ mt: 1 }}
                onClick={handleReset}
              >
                Reset
              </Button>
            </Grid>
            <Grid item xs={12} sm={6}>
              <TextField
                label="Branch Name"
                name="NAME"
                onChange={handleInputChange}
                value={branchData.NAME}
                error={!!branchDetailErrors.NAME}
                helperText={branchDetailErrors.NAME}
                disabled={fieldsDisabled}
                variant="outlined"
                fullWidth
                InputProps={{
                  startAdornment: (
                    <AccountBalanceRoundedIcon sx={{ mr: 1, opacity: "50%" }} />
                  ),
                }}
                inputProps={{ maxLength: 40 }}
              />
            </Grid>

            {/* Circle, Network, Module, Region */}
            <Grid item xs={6} sm={6}>
              <TextField
                label="Circle Name"
                name="CIRCLE"
                error={!!branchDetailErrors.CIRCLE}
                helperText={branchDetailErrors.CIRCLE}
                select
                disabled={fieldsDisabled}
                value={branchData.CIRCLE}
                onChange={handleInputChange}
                variant="outlined"
                fullWidth
                InputProps={{
                  startAdornment: <LanIcon sx={{ mr: 1, opacity: "50%" }} />,
                }}
                SelectProps={{ MenuProps: { sx: { height: 350 } } }}
              >
                {circleList.map((choices) => (
                  <MenuItem key={choices} value={choices.split("~")[0]}>
                    {choices.split("~")[1]}
                  </MenuItem>
                ))}
              </TextField>
            </Grid>
            <Grid item xs={6} sm={2}>
              <TextField
                label="Network"
                name="NETWORK"
                disabled={fieldsDisabled}
                value={branchData.NETWORK}
                error={!!branchDetailErrors.NETWORK}
                helperText={branchDetailErrors.NETWORK}
                onChange={handleInputChange}
                variant="outlined"
                fullWidth
                InputProps={{
                  startAdornment: (
                    <SensorsIcon sx={{ mr: 1, opacity: "50%" }} />
                  ),
                }}
                inputProps={{ maxLength: 3 }}
              />
            </Grid>
            <Grid item xs={6} sm={2}>
              <TextField
                label="Module"
                name="MODULE"
                disabled={fieldsDisabled}
                value={branchData.MODULE}
                error={!!branchDetailErrors.MODULE}
                helperText={branchDetailErrors.MODULE}
                onChange={handleInputChange}
                variant="outlined"
                fullWidth
                InputProps={{
                  startAdornment: <DnsIcon sx={{ mr: 1, opacity: "50%" }} />,
                }}
                inputProps={{ maxLength: 3 }}
              />
            </Grid>
            <Grid item xs={6} sm={2}>
              <TextField
                label="Region"
                name="REGION"
                disabled={fieldsDisabled}
                value={branchData.REGION}
                error={!!branchDetailErrors.REGION}
                helperText={branchDetailErrors.REGION}
                onChange={handleInputChange}
                variant="outlined"
                fullWidth
                InputProps={{
                  startAdornment: <PublicIcon sx={{ mr: 1, opacity: "50%" }} />,
                }}
                inputProps={{ maxLength: 3 }}
              />
            </Grid>

            {/* Address Details */}
            <Grid item xs={12}>
              <TextField
                label="Address"
                name="ADDRESS"
                disabled={fieldsDisabled}
                value={branchData.ADDRESS}
                error={!!branchDetailErrors.ADDRESS}
                helperText={branchDetailErrors.ADDRESS}
                onChange={handleInputChange}
                variant="outlined"
                fullWidth
                InputProps={{
                  startAdornment: <HomeIcon sx={{ mr: 1, opacity: "50%" }} />,
                }}
                inputProps={{ maxLength: 40 }}
              />
            </Grid>
            <Grid item xs={6}>
              <TextField
                label="City"
                name="CITY"
                disabled={fieldsDisabled}
                value={branchData.CITY}
                error={!!branchDetailErrors.CITY}
                helperText={branchDetailErrors.CITY}
                onChange={handleInputChange}
                variant="outlined"
                fullWidth
                InputProps={{
                  startAdornment: (
                    <LocationCityRoundedIcon sx={{ mr: 1, opacity: "50%" }} />
                  ),
                }}
                inputProps={{ maxLength: 40 }}
              />
            </Grid>
            <Grid item xs={6}>
              <TextField
                label="State"
                name="STATE"
                disabled={fieldsDisabled}
                value={branchData.STATE}
                error={!!branchDetailErrors.STATE}
                helperText={branchDetailErrors.STATE}
                onChange={handleInputChange}
                variant="outlined"
                fullWidth
                InputProps={{
                  startAdornment: (
                    <CorporateFareRoundedIcon sx={{ mr: 1, opacity: "50%" }} />
                  ),
                }}
                inputProps={{ maxLength: 40 }}
              />
            </Grid>

            {/* Contact Details */}
            <Grid item xs={6} sm={4}>
              <TextField
                label="Pin Code"
                name="PIN"
                disabled={fieldsDisabled}
                value={branchData.PIN}
                error={!!branchDetailErrors.PIN}
                helperText={branchDetailErrors.PIN}
                onChange={handleInputChange}
                variant="outlined"
                fullWidth
                InputProps={{
                  startAdornment: (
                    <PinDropIcon sx={{ mr: 1, opacity: "50%" }} />
                  ),
                }}
                inputProps={{ maxLength: 6 }}
              />
            </Grid>
            <Grid item xs={6} sm={4}>
              <TextField
                label="STD Code"
                name="STDCODE"
                disabled={fieldsDisabled}
                value={branchData.STDCODE}
                error={!!branchDetailErrors.STDCODE}
                helperText={branchDetailErrors.STDCODE}
                onChange={handleInputChange}
                variant="outlined"
                fullWidth
                InputProps={{
                  startAdornment: (
                    <EmergencyShareIcon sx={{ mr: 1, opacity: "50%" }} />
                  ),
                }}
                inputProps={{ maxLength: 5 }}
              />
            </Grid>
            <Grid item xs={6} sm={4}>
              <TextField
                label="Phone No."
                name="PHONE"
                disabled={fieldsDisabled}
                value={branchData.PHONE}
                error={!!branchDetailErrors.PHONE}
                helperText={branchDetailErrors.PHONE}
                onChange={handleInputChange}
                variant="outlined"
                fullWidth
                InputProps={{
                  startAdornment: <PhoneIcon sx={{ mr: 1, opacity: "50%" }} />,
                }}
                inputProps={{ maxLength: 10 }}
              />
            </Grid>
            <Grid item xs={6} sm={4}>
              <TextField
                label="Mobile No."
                name="MOBILE"
                disabled={fieldsDisabled}
                value={branchData.MOBILE}
                error={!!branchDetailErrors.MOBILE}
                helperText={branchDetailErrors.MOBILE}
                onChange={handleInputChange}
                variant="outlined"
                fullWidth
                InputProps={{
                  startAdornment: (
                    <PhoneAndroidRoundedIcon sx={{ mr: 1, opacity: "50%" }} />
                  ),
                }}
                inputProps={{ maxLength: 10 }}
              />
            </Grid>
            <Grid item xs={6} sm={4}>
              <TextField
                label="IP Phone"
                name="IPPHONE"
                disabled={fieldsDisabled}
                value={branchData.IPPHONE}
                error={!!branchDetailErrors.IPPHONE}
                helperText={branchDetailErrors.IPPHONE}
                onChange={handleInputChange}
                variant="outlined"
                fullWidth
                InputProps={{
                  startAdornment: (
                    <WifiCalling3Icon sx={{ mr: 1, opacity: "50%" }} />
                  ),
                }}
                inputProps={{ maxLength: 10 }}
              />
            </Grid>
            <Grid item xs={12} sm={4}></Grid>

            {/* Audit Status and Auditor Details */}
            <Grid item xs={6} sm={6}>
              <TextField
                name="AUDITABLE"
                disabled={fieldsDisabled}
                fullWidth
                select
                InputProps={{
                  startAdornment: <PersonIcon sx={{ mr: 1, opacity: "50%" }} />,
                }}
                SelectProps={{ MenuProps: { sx: { height: 350 } } }}
                error={!!branchDetailErrors.AUDITABLE}
                helperText={branchDetailErrors.AUDITABLE}
                value={branchData.AUDITABLE}
                onChange={handleInputChange}
                label="Branch Audit Status"
              >
                <MenuItem value="">---Select---</MenuItem>
                <MenuItem value="A">Audited</MenuItem>
                <MenuItem value="N">Non Audited</MenuItem>
                <MenuItem value="I">IFCOFR Audited</MenuItem>
              </TextField>
            </Grid>
            <Grid item xs={6} sm={6}></Grid>

            {(branchData.AUDITABLE === "I" || branchData.AUDITABLE === "A") &&
              !fieldsDisabled && <>{/* Auditor details fields here */}</>}

            {/* Action Buttons */}
            <Grid item xs={12}>
              <Box sx={{ display: "flex", justifyContent: "center", p: 1 }}>
                <Button
                  variant="contained"
                  color="error"
                  size="medium"
                  sx={{ ml: 2 }}
                  onClick={fetchReportsForDelete}
                  disabled={fieldsDisabled}
                  startIcon={<DeleteIcon />}
                >
                  Delete
                </Button>
              </Box>
            </Grid>
          </Grid>
        </Box>
      </Container>

      {/* Warning Dialog for no changes */}
      <Dialog open={openWarningDialog}>
        <DialogTitle>Warning</DialogTitle>
        <DialogContent>
          <DialogContentText>
            Data is the same as before. Please modify data before saving.
          </DialogContentText>
        </DialogContent>
        <DialogActions>
          <Button onClick={() => setOpenWarningDialog(false)}>Close</Button>
        </DialogActions>
      </Dialog>

      {/* Loading Dialog */}
      <Dialog
        PaperProps={{
          style: { backgroundColor: "transparent", boxShadow: "none" },
        }}
        open={loadOpen}
      >
        <DialogContent>
          <CircularProgress />
        </DialogContent>
      </Dialog>

      {/* Snackbar */}
      {snackbar && (
        <SnackbarProvider maxSnack={3}>
          <Snackbar
            open
            anchorOrigin={{ vertical: "top", horizontal: "center" }}
            onClose={handleCloseSnackbar}
            autoHideDuration={5000}
          >
            <Alert
              variant="filled"
              {...snackbar}
              onClose={handleCloseSnackbar}
            />
          </Snackbar>
        </SnackbarProvider>
      )}

      {/* Delete Confirmation Dialog */}
      <Dialog open={showDeleteConfirm} maxWidth="xl">
        <DialogTitle>
          <Typography fontSize={20} fontWeight={750}>
            If you are going to exclude this branch from scope all these reports
            will be deleted. Do you want to continue?
          </Typography>
        </DialogTitle>
        <DialogContent>
          {reportsList.length > 0 ? (
            <Paper elevation={0}>
              <TableContainer sx={{ height: 500, minWidth: 1000 }}>
                <Table stickyHeader>
                  <TableHead>
                    <TableRow>
                      <TableCell align="center">S.No</TableCell>
                      <TableCell align="center">Module</TableCell>
                      <TableCell align="center">Report Name</TableCell>
                      <TableCell>Report Description</TableCell>
                      <TableCell align="center">Report Status</TableCell>
                    </TableRow>
                  </TableHead>
                  <TableBody>
                    {reportsList.map((row, i) => (
                      <TableRow key={i}>
                        <TableCell align="center">{i + 1}</TableCell>
                        <TableCell align="center">{row.MODULE}</TableCell>
                        <TableCell align="center">{row.REPORT_NAME}</TableCell>
                        <TableCell>{row.REPORT_DESC}</TableCell>
                        <TableCell align="center">
                          {branchData.AUDITABLE === "N"
                            ? REPORT_STATUS_CONSTANTS[row.CURRENT_STATUS]?.text
                            : AUDITED_REPORT_STATUS_CONSTANTS[
                                row.CURRENT_STATUS
                              ]?.text}
                        </TableCell>
                      </TableRow>
                    ))}
                  </TableBody>
                </Table>
              </TableContainer>
            </Paper>
          ) : (
            <DialogContentText>
              This branch has no reports to delete. You can proceed with
              deletion.
            </DialogContentText>
          )}
        </DialogContent>
        <DialogActions>
          <Button
            onClick={() => setShowDeleteConfirm(false)}
            variant="outlined"
            color="primary"
          >
            No
          </Button>
          <Button
            onClick={handleConfirmDeletion}
            variant="contained"
            color="error"
          >
            Yes, Delete Branch
          </Button>
        </DialogActions>
      </Dialog>
    </>
  );
}


</body>
</html>


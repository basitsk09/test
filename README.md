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

</body>
</html>


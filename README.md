<%--
  Created by IntelliJ IDEA.
  User: V1009204
  Date: 15-03-2023
  Time: 12:55
  To change this template use File | Settings | File Templates.
--%>
<%@ include file="/views/include.jsp" %>
<!DOCTYPE html>
<html>
<head>
    <script type="text/javascript">

        function onload() {
            var displayMessage = '${displayMessage}';
            console.log(displayMessage);
            if (displayMessage) {
                $('#submitModal .modal-body').text(displayMessage);
                $('#submitModal').modal('show');
                $('#submitModal').modal({
                    backdrop: 'static',
                    keyboard: false,
                    modal: true
                });
            }

            var displayMessage1 = '${displayMessage1}';
            console.log(displayMessage1);
            if (displayMessage1) {
                $('#errorModal .modal-body').text(displayMessage1);
                $('#errorModal').modal('show');
                $('#errorModal').modal({
                    backdrop: 'static',
                    keyboard: false,
                    modal: true
                });
            }
        }


        $(document).ready(function () {
            $('.recordSelector').on('click',function () {
                if($('.recordSelector:checked').length==$('.recordSelector').length){
                    $('#selAll').prop('checked',true);
                }
                else{
                    $('#selAll').prop('checked',false);
                }
            });

            $('.recordSelectorAll').on('click',function () {
                if($('.recordSelectorAll:checked').length==1){
                    $('.recordSelector').prop('checked',true);
                    document.getElementById("approve").disabled = false;
                    document.getElementById("reject").disabled = false;
                }
                else{
                    $('.recordSelector').prop('checked',false);
                    document.getElementById("approve").disabled = true;
                    document.getElementById("reject").disabled = true;
                }
            });

            $('.recordSelector').on('click',function () {
                if($('.recordSelector:checked').length<1){
                    document.getElementById("approve").disabled = true;
                    document.getElementById("reject").disabled = true;
                }
                else{
                    document.getElementById("approve").disabled = false;
                    document.getElementById("reject").disabled = false;
                }
            });



            $(function () {
                $('#auditReq').DataTable({
                    "paging": false,
                    "lengthChange": false,
                    "searching": false,
                    "ordering": false,
                    "info": false,
                    scrollY: 580,
                    "autoWidth": false
                });
            });
        });

        function reload() {
            window.location.replace("../FRTChecker/FRTAuditStatusReq");
        }

        function approve() {
            $("#frtAuditStatusReq").submit();
            document.getElementById("divMsg").hidden = false;
        }


        function reject(){
            $('#frtAuditStatusReq').attr('action', "../FRTChecker/getrejected").submit();
            document.getElementById("divMsg").hidden = false;
        }
    </script>
</head>
<body class="hold-transition skin-blue sidebar-mini" onload="onload();">
<div class="wrapper">
    <div class="content-wrapper">
        <section class="content-header">
            <h1>Audit Status Change Requests</h1>
        </section>
        <div class="content">
            <div class="row">
                <div class="col-xs-12">
                    <div class="box box-primary box-solid">
                        <div class="box-body">
                            <div class="box-header">
                                <div class="row" style="margin-bottom: 10px">
                                    <div class="col-lg-4"></div>
                                    <div class="col-lg-4">
                                        <div class="col-lg-2"></div>
                                        <div class="col-lg-10">
                                            <button type="button" value="submit" class="btn btn-success" style="margin-left: 64px; width: 80px;" id="approve" onclick="approve();" disabled>Approve</button>
                                            <button type="button" value="reject" class="btn btn-danger" style="margin-left: 10px; width: 80px;" id="reject" onclick="reject();" disabled>Reject</button>
                                        </div>
                                    </div>
                                    <div class="col-lg-4"></div>
                                </div>
                                <div class="col-md-12">
                                    <form:form action="getapproved" method="POST" id="frtAuditStatusReq">
                                        <table id="auditReq" class="dataTable">
                                            <thead>
                                            <tr style="background-color: #b9def0; height: 40px">
                                                <th style="width: 50px;text-align: center ">
                                                    <input type="checkbox" name="selRadioAll" style="margin-top: 14px;" class="recordSelectorAll" id="selAll"/>
                                                    Select All
                                                </th>
                                                <th style="width: 120px;text-align: center ">Req ID</th>
                                                <th style="text-align: center">Branch</th>
                                                <th style="text-align: center">Branch Name</th>
                                                <th style="width: 150px; text-align: center">Existing Status</th>
                                                <th style="width: 200px; text-align: center">New Requested Status</th>
                                                <th style="text-align: center">Requested Status</th>
                                                <th style="text-align: center">Requested By</th>
                                            </tr>
                                            </thead>
                                            <tbody>
                                            <c:if test="${count!=0}">
                                                <c:forEach items="${list}" var="FRTAuditStatusReq" varStatus="reportStatus">
                                                    <tr>
                                                        <input type="hidden" readonly="true" id="roCode<c:out value='${reportStatus.count}'></c:out>"
                                                               value='<c:out value='${FRTAuditStatusReq.roCode}'></c:out>' name="roCode"/>
                                                        <input type="hidden" readonly="true" id="circle_code<c:out value='${reportStatus.count}'></c:out>"
                                                               value='<c:out value='${FRTAuditStatusReq.circle_code}'></c:out>' name="circle_code"/>
                                                        <td style="text-align: center;">
                                                            <input type="checkbox" name="selRadio" class="recordSelector"
                                                                   value='<c:out value="${FRTAuditStatusReq.as_id}"></c:out>~<c:out value='${FRTAuditStatusReq.branchCode}'></c:out>~<c:out value='${FRTAuditStatusReq.beforeSts}'></c:out>~<c:out value='${FRTAuditStatusReq.afterSts}'></c:out>~<c:out value='${FRTAuditStatusReq.roCode}'></c:out>~<c:out value='${FRTAuditStatusReq.circle_code}'></c:out>~<c:out value='${FRTAuditStatusReq.as_rt_id}'></c:out>'
                                                                   id="selRadio<c:out value='${reportStatus.count}'></c:out>"/>
                                                        </td>
                                                        <td style="text-align: center;">
                                                            <input type="hidden" readonly="true" id="req_Id<c:out value='${reportStatus.count}'></c:out>" name="req_Id"  readonly="true"
                                                                   value='<c:out value="${FRTAuditStatusReq.as_id}"></c:out>'/>
                                                            <c:out value="${FRTAuditStatusReq.as_rt_id}-${FRTAuditStatusReq.as_id}"></c:out>
                                                        </td>
                                                        <td style="text-align: center;">
                                                            <input type="hidden" readonly="true" id="branchCode<c:out value='${reportStatus.count}'></c:out>"
                                                                   value='<c:out value='${FRTAuditStatusReq.branchCode}'></c:out>' name="branchCode"/>
                                                            <c:out value='${FRTAuditStatusReq.branchCode}'></c:out>
                                                        </td>
                                                        <td style="text-align: center;">
                                                            <input type="hidden" readonly="true" id="branchName<c:out value='${reportStatus.count}'></c:out>"
                                                                   value='<c:out value='${FRTAuditStatusReq.branchName}'></c:out>'/>
                                                            <c:out value='${FRTAuditStatusReq.branchName}'></c:out>
                                                        </td>
                                                        <td style="text-align: center;">
                                                            <input type="hidden" readonly="true" id="beforeSts<c:out value='${reportStatus.count}'></c:out>"
                                                                   value='<c:out value='${FRTAuditStatusReq.beforeSts}'></c:out>' name="beforeSts"/>
                                                            <c:out value='${FRTAuditStatusReq.beforeSts}'></c:out>
                                                        </td>
                                                        <td style="text-align: center;">
                                                            <input type="hidden" readonly="true" id="afterSts<c:out value='${reportStatus.count}'></c:out>"
                                                                   value='<c:out value='${FRTAuditStatusReq.afterSts}'></c:out>' name="afterSts"/>
                                                            <c:out value='${FRTAuditStatusReq.afterSts}'></c:out>
                                                        </td>
                                                        <td style="text-align: center;">
                                                            <input type="hidden" readonly="true" style="border-style: hidden"
                                                                   id="reqSts<c:out value='${reportStatus.count}'></c:out>"
                                                                   value='<c:out value='${FRTAuditStatusReq.reqSts}'></c:out>'/>
                                                            <c:out value='${FRTAuditStatusReq.reqSts}'></c:out>
                                                        </td>
                                                        <td style="text-align: center;">
                                                            <input type="hidden" readonly="true" id="reqOn<c:out value='${reportStatus.count}'></c:out>"
                                                                   value='<c:out value='${FRTAuditStatusReq.reqOn}'></c:out>'/>
                                                            <c:out value='${FRTAuditStatusReq.reqOn}'></c:out>
                                                        </td>
                                                    </tr>
                                                </c:forEach>
                                            </c:if>
                                            </tbody>
                                        </table>
                                    </form:form>
                                </div>
                            </div>
                        </div>
                        <div class="overlay" id="divMsg" hidden>
                            <i class="fa fa-spinner fa-spin" style="color: #9a0200"></i>
                        </div>
                    </div>
                </div>
            </div>
        </div>
    </div>

    <div class="example-modal">
        <div class="modal fade" id="errorModal">
            <div class="modal-dialog">
                <div class="modal-content">
                    <div class="modal-header bg-info">
                        <h4 class="modal-title"><b>Attention</b></h4>
                    </div>
                    <div class="modal-body">
                    </div>
                    <div class="modal-footer">
                        <button type="button" class="btn btn-info" data-dismiss="modal">Close</button>
                    </div>
                </div>
            </div>
        </div>
    </div>
</div>
<div class="example-modal">
    <div class="modal fade" id="submitModal">
        <div class="modal-dialog">
            <div class="modal-content">
                <div class="modal-header bg-success">
                    <h4 class="modal-title" style="color: green;">Success</h4>
                </div>
                <div class="modal-body">
                </div>
                <div class="modal-footer">
                    <button type="button" class="btn btn-success" data-dismiss="modal" onclick="reload();">Close</button>
                </div>
            </div>
        </div>
    </div>
</div>
</div>
</body>
</html>



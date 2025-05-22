JSP File
//////////////////////////

<div class="wrapper">
    <div class="header header-filter" style="background-image: url('assets/img/bg2.jpeg');">
        <div class="container">
            <div class="row tim-row">
                <div class="col-md-8 col-md-offset-2">
                    <div class="brand">
                        <h3 style="color: white;">Schedule 9 Supplementary Information</h3>
                    </div>
                </div>
            </div>

        </div>

    </div>
    <div class="main main-raised ">
        <div class="section">
            <div class="container" >
                <div class="col-md-12">

                    <script type="text/javascript">
                        $(".decimal-2-places").numeric({ decimalPlaces: 2 });

                    </script>







                    <form  name="sc9SuplForm" ng-submit="sc9Supl.submitNineSupl(sc9Supl.row);">
                        <input type="hidden" id="csrfPreventionSalt" value="${csrfPreventionSalt}"/>
                        <table id="example1" class="table table-hover table-responsive table-bordered no-padding" >
                            <thead>
                            <tr>
                                <th style="text-align:center;">Description</th>
                                <th style="text-align:center;">GROSS AMOUNT <br>Rs. &nbsp;&nbsp;&nbsp;P</th>
                                <th style="text-align:center;">PROVISION<br>Rs. &nbsp;&nbsp;&nbsp;P</th>
                            </tr>
                            </thead>
                            <tbody>

                            <tr>
                                <td><b>1. Bills Purchased and discounted  (excluding bills rediscounted)  &nbsp;&nbsp;&nbsp;    (1.1 + 1.2) </b></td>
                                <td><input type="text" maxlength="18" ng-model="sc9Supl.row.bilPurGrAmt"  style="text-align:right;" name="bilPurGrAmt"   class="form-control decimal-2-places" readonly="readonly" /></td>
                                <td><input type="text" maxlength="18" ng-model="sc9Supl.row.bilPurPro" style="text-align:right;" name="bilPurPro"    class="form-control decimal-2-places" readonly="readonly" /></td>
                            </tr>






                            <tr>
                                <td>&nbsp;&nbsp;&nbsp;1.1 Inland Bills Purchased and Discounted</td>
                                <td><input type="text" maxlength="18" ng-model="sc9Supl.row.inlBilPurGrAmt"  style="text-align:right;" name="inlBilPurGrAmt"   class="form-control decimal-2-places"  /></td>
                                <td><input type="text" maxlength="18" ng-model="sc9Supl.row.inlBilPurPro" style="text-align:right;" name="inlBilPurPro"    class="form-control decimal-2-places" /></td>
                            </tr>


                            <tr>
                                <td>  &nbsp;&nbsp;&nbsp;<b>  1.2 Foreign Bills Purchased and Discounted  (excluding bills rediscounted)&nbsp;&nbsp;&nbsp;( 1.2.1+1.2.2+1.2.3)</b>
                                </td>
                                <td><input type="text" maxlength="18" ng-model="sc9Supl.row.foreBilPurGrAmt"  style="text-align:right;" name="foreBilPurGrAmt"   class="form-control decimal-2-places" readonly="readonly" /></td>
                                <td><input type="text" maxlength="18" ng-model="sc9Supl.row.foreBilPurPro" style="text-align:right;" name="foreBilPurPro"    class="form-control decimal-2-places" readonly="readonly" /></td>
                            </tr>


                            <tr>
                                <td>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;1.2.1 Export Bills drawn in India</td>
                                <td><input type="text" maxlength="18" ng-model="sc9Supl.row.expBillGrAmt"  style="text-align:right;" name="expBillGrAmt"   class="form-control decimal-2-places"  /></td>
                                <td><input type="text" maxlength="18" ng-model="sc9Supl.row.expBillPro" style="text-align:right;" name="expBillPro"    class="form-control decimal-2-places" /></td>
                            </tr>

                            <tr>
                                <td> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; 1.2.2 Import Bills drawn on and payable in India</td>
                                <td><input type="text" maxlength="18" ng-model="sc9Supl.row.impBillGrAmt"  style="text-align:right;" name="impBillGrAmt"   class="form-control decimal-2-places"  /></td>
                                <td><input type="text" maxlength="18" ng-model="sc9Supl.row.impBillPro" style="text-align:right;" name="impBillPro"    class="form-control decimal-2-places" /></td>
                            </tr>
                            <tr>
                                <td> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<b> 1.2.3 Other Foreign Bills Purchased and Discounted 9excluding bills rediscounted) &nbsp;&nbsp;&nbsp;(1.2.3.1+1.2.3.2)</b>
                                </td>
                                <td><input type="text" maxlength="18" ng-model="sc9Supl.row.othForeBilPurGrAmt"  style="text-align:right;" name="othForeBilPurGrAmt"   class="form-control decimal-2-places" readonly="readonly" /></td>
                                <td><input type="text" maxlength="18" ng-model="sc9Supl.row.othForeBilPurPro" style="text-align:right;" name="othForeBilPurPro"    class="form-control decimal-2-places"readonly="readonly" /></td>
                            </tr>

                            <tr>
                                <td>  &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; 1.2.3.1 Payable in India</td>
                                <td><input type="text" maxlength="18" ng-model="sc9Supl.row.payIndGrAmt"  style="text-align:right;" name="payIndGrAmt"   class="form-control decimal-2-places"  /></td>
                                <td><input type="text" maxlength="18" ng-model="sc9Supl.row.payIndPro" style="text-align:right;" name="payIndPro"    class="form-control decimal-2-places" /></td>
                            </tr>


                            <tr>
                                <td>  &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; 1.2.3.2 Payable outside India</td>
                                <td><input type="text" maxlength="18" ng-model="sc9Supl.row.payOutGrAmt"  style="text-align:right;" name="payOutGrAmt"   class="form-control decimal-2-places"  /></td>
                                <td><input type="text" maxlength="18" ng-model="sc9Supl.row.payOutPro" style="text-align:right;" name="payOutPro"    class="form-control decimal-2-places" /></td>
                            </tr>

                            <tr>
                                <td><b>2. Loans and Advances &nbsp;&nbsp;&nbsp;(2.1 + 2.2)</b>
                                </td>
                                <td><input type="text" maxlength="18" ng-model="sc9Supl.row.loanAdvGrAmt"  style="text-align:right;" name="loanAdvGrAmt"   class="form-control decimal-2-places" readonly="readonly" /></td>
                                <td><input type="text" maxlength="18" ng-model="sc9Supl.row.loanAdvPro" style="text-align:right;" name="loanAdvPro"    class="form-control decimal-2-places" readonly="readonly" /></td>
                            </tr>

                            <tr>
                                <td>   &nbsp;&nbsp;&nbsp; 2.1 Loans and Advances, Cash Credit & Overdrafts (excluding due from Banks vide 2.2 below)</td>
                                <td><input type="text" maxlength="18" ng-model="sc9Supl.row.loanAdvCreGrAmt"  style="text-align:right;" name="loanAdvCreGrAmt"   class="form-control decimal-2-places"  /></td>
                                <td><input type="text" maxlength="18" ng-model="sc9Supl.row.loanAdvCrePro" style="text-align:right;" name="loanAdvCrePro"    class="form-control decimal-2-places" /></td>
                            </tr>

                            <tr>
                                <td>  &nbsp;&nbsp;&nbsp; <b> 2.2 Due from Banks &nbsp;&nbsp;&nbsp;( 2.2.1+2.2.2+2.2.3)</b></td>
                                <td><input type="text" maxlength="18" ng-model="sc9Supl.row.dueGrAmt"  style="text-align:right;" name="dueGrAmt"   class="form-control decimal-2-places" readonly="readonly" /></td>
                                <td><input type="text" maxlength="18" ng-model="sc9Supl.row.duePro" style="text-align:right;" name="duePro"    class="form-control decimal-2-places" readonly="readonly" /></td>
                            </tr>

                            <tr>
                                <td>  &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;2.2.1 Co-operative Banks in India</td>
                                <td><input type="text" maxlength="18" ng-model="sc9Supl.row.coopBankGrAmt"  style="text-align:right;" name="coopBankGrAmt"   class="form-control decimal-2-places"  /></td>
                                <td><input type="text" maxlength="18" ng-model="sc9Supl.row.coopBankPro" style="text-align:right;" name="coopBankPro"    class="form-control decimal-2-places" /></td>
                            </tr>




                            <tr>
                                <td> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;2.2.2 Commercial Banks in India</td>
                                <td><input type="text" maxlength="18" ng-model="sc9Supl.row.commBankGrAmt"  style="text-align:right;" name="commBankGrAmt"   class="form-control decimal-2-places"  /></td>
                                <td><input type="text" maxlength="18" ng-model="sc9Supl.row.commBankPro" style="text-align:right;" name="commBankPro"    class="form-control decimal-2-places" /></td>
                            </tr>
                            <tr>
                                <td>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;2.2.3 Banks outside India</td>
                                <td><input type="text" maxlength="18" ng-model="sc9Supl.row.bankOutIndGrAmt"  style="text-align:right;" name="bankOutIndGrAmt"   class="form-control decimal-2-places"  /></td>
                                <td><input type="text" maxlength="18" ng-model="sc9Supl.row.bankOutIndPro" style="text-align:right;" name="bankOutIndPro"    class="form-control decimal-2-places" /></td>
                            </tr>



                            <tr>
                                <td><b>3. Grand Total&nbsp;&nbsp;&nbsp; (1 + 2)</b></td>
                                <td><input type="text" maxlength="18" ng-model="sc9Supl.row.grandTotlGrAmt"  style="text-align:right;" name="grandTotlGrAmt"   class="form-control decimal-2-places" readonly="readonly" /></td>
                                <td><input type="text" maxlength="18" ng-model="sc9Supl.row.grandTotlPro" style="text-align:right;" name="grandTotlPro"    class="form-control decimal-2-places" readonly="readonly" /></td>
                            </tr>


                            </tbody>
                        </table>

                         <button type="submit" class="btn btn-warning" ng-click="sc9Supl.row.save = true">Save</button>
                        <button type="submit" class="btn btn-default btn-success" ng-click="sc9Supl.row.save = false">Submit</button>
                    </form>














                </div>
            </div>
        </div>
    </div>
</div>


<div class="modal fade" id="myModal1" role="dialog" style="z-index : 1400">
    <div class="modal-dialog">

        <!-- Modal content-->
        <div class="modal-content">
            <div class="modal-header" style="background-color: #E74C3C";>
                <!--<button type="button" class="close" data-dismiss="modal">?</button> -->
                <h4 class="modal-title" style="color: white;">Attention!</h4>
            </div>
            <div class="modal-body" id="popup">
                {{sc9Supl.displayMessage}}
            </div>
            <div class="modal-footer">
                <button type="button" class="btn btn-default btn-success" data-dismiss="modal" ng-click="sc9Supl.redirect();">Continue</button>
            </div>


        </div>
    </div>

</div>







<div class="modal fade" id="myModal3" role="dialog" style="z-index : 1400">
    <div class="modal-dialog">

        <!-- Modal content-->
        <div class="modal-content">
            <div class="modal-header" style="background-color: #E74C3C";>
                <!--<button type="button" class="close" data-dismiss="modal">?</button> -->
                <h4 class="modal-title" style="color: white;">Attention!</h4>
            </div>
            <div class="modal-body" id="popup">
                {{sc9Supl.displayMessage111}}
            </div>
            <div class="modal-footer">
                <button type="button" class="btn btn-default btn-success" ng-click="sc9Supl.YesSubmitNineSupl(sc9Supl.row);" data-dismiss="modal">YES</button>

                <button type="button" class="btn btn-default btn-success"  data-dismiss="modal" >No</button>
            </div>


        </div>
    </div>

</div>



<div class="modal fade" id="myModal5" role="dialog" style="z-index : 1400">
    <div class="modal-dialog">

        <!-- Modal content-->
        <div class="modal-content">
            <div class="modal-header" style="background-color: #E74C3C";>
                <!--<button type="button" class="close" data-dismiss="modal">?</button> -->
                <h4 class="modal-title" style="color: white;">Attention!</h4>
            </div>
            <div class="modal-body" id="popup">
                {{sc9Supl.displayMessage}}
            </div>
            <div class="modal-footer">
                <button type="button" class="btn btn-default btn-success"  data-dismiss="modal" >Continue</button>
            </div>


        </div>
    </div>

</div>

////////////////////////////////////////
JS validations below

///////////////////////////////////////
app
    .controller('SC9Suplcontroller', function ($scope, $rootScope, $http, $window, $sessionStorage, $state, $timeout, $location, Idle, Keepalive, $modal, ModalService, userFactory, sc9SuplFactory, refreshFactory, AES256) {

        if ($rootScope.reportObject == undefined) {
            refreshFactory.backToState();
            return;
        }

        $scope.sessionUser = JSON.parse(AES256.decrypt($rootScope.globals.currentUser));
        //console.log($scope.sessionUser.quarterEndDate);


        var sc9Supl = this;
        $scope.started = false;


        sc9Supl.row = {};

        // //////////////getting saved data
        if ($rootScope.reportObject.status != null || $rootScope.reportObject.status != undefined) {

            var circleCode = $scope.sessionUser.circleCode;
            var quarterEndDate = $scope.sessionUser.quarterEndDate;

            var row1 = {
                'circleCode': circleCode,
                'quarterEndDate': quarterEndDate,
                'userId': $scope.sessionUser.userId,
                'reportName': $rootScope.reportObject.name,
                'reportId': $rootScope.reportObject.reportId,
                'reportMasterId': $rootScope.reportObject.reportMasterId,
                'status': $rootScope.reportObject.status,
                'areMocPending': $rootScope.reportObject.areMocPending
            };

            sc9SuplFactory
                .getSavedDataNineSupl(row1)
                .then(function (data) {
                    console.log("editing");

                    sc9Supl.row = data;


                }, function (errResponse) {
                    console
                        .error('Error getting saved data in schedule 9 Supplementary Information');
                });

        }


        // //////////////////////submit

        sc9Supl.submitNineSupl = function (row) {

            console.log("for saving data ? " + sc9Supl.row.save);


            //////////////////////er0b
            var params = {
                'row': $scope.row,
                'circleCode': $scope.sessionUser.circleCode,
                'quarterEndDate': $scope.sessionUser.quarterEndDate,
                'userId': $scope.sessionUser.userId

            };


            if ($scope.sc9SuplForm.$dirty == false) {
                if (sc9Supl.row.save == false) {
                    sc9Supl.displayMessage111 = "Would you like to submit report without changing data ?";
                } else {
                    sc9Supl.displayMessage111 = "Would you like to save report without changing data ?";
                }
                if (sc9Supl.displayMessage111) {

                    $('#myModal3').modal({
                        backdrop: 'static', keyboard: false, modal: true
                    });
                    $('#myModal3')
                        .on('shown.bs.modal', function () {
                            $('#myModal3').trigger('focus');
                        });
                }

            }

            sc9Supl.YesSubmitNineSupl = function (row) {

                var i = {};
                i = row;
                console.log(i);
                var circleCode = $scope.sessionUser.circleCode;
                var quarterEndDate = $scope.sessionUser.quarterEndDate;


                var row1 = {

                    'circleCode': circleCode,
                    'quarterEndDate': quarterEndDate,
                    'userId': $scope.sessionUser.userId,
                    'reportName': $rootScope.reportObject.name,
                    'reportId': $rootScope.reportObject.reportId,
                    'reportMasterId': $rootScope.reportObject.reportMasterId,
                    'status': $rootScope.reportObject.status

                };
                var copy = angular.extend({}, i, row1);

                sc9SuplFactory
                    .submitNineSupl(copy)
                    .then(function (data) {

                        console.log("************** submitStatus ~ Sequence (reportId) ~ reportStatus : " + data);
                        var result = [];
                        result = data.split('~');


                        if (result[0] == 0) {
                            $rootScope.reportObject.reportId = result[1];
                            $rootScope.reportObject.status = result[2];
                            console.log("# reportId " + $rootScope.reportObject.reportId);
                            console.log("# status " + $rootScope.reportObject.status);
                            if (sc9Supl.row.save == false) {
                                sc9Supl.displayMessage = "Report Submitted Successfully";
                                if (sc9Supl.displayMessage) {

                                    $('#myModal1')
                                        .modal({
                                            backdrop: 'static', keyboard: false, modal: true
                                        });
                                    $('#myModal1')
                                        .on('shown.bs.modal', function () {
                                            $('#myModal1')
                                                .trigger('focus');
                                        });
                                }
                            } else {
                                sc9Supl.displayMessage = "Report Saved Successfully";
                                if (sc9Supl.displayMessage) {
                                    $('#myModal5')
                                        .modal({
                                            backdrop: 'static', keyboard: false, modal: true
                                        });
                                    $('#myModal5')
                                        .on('shown.bs.modal', function () {
                                            $('#myModal5')
                                                .trigger('focus');
                                        });
                                }

                            }
                        }
                    }, function (errResponse) {
                        console
                            .error('Error while Submitting Schedule 10');
                    });
            }


            if ($scope.sc9SuplForm.$dirty == true) {

                var i = {};
                i = sc9Supl.row;
                console.log(i);
                var circleCode = $scope.sessionUser.circleCode;
                var quarterEndDate = $scope.sessionUser.quarterEndDate;


                console.log($rootScope.reportObject);
                var row1 = {

                    'circleCode': circleCode,
                    'quarterEndDate': quarterEndDate,
                    'userId': $scope.sessionUser.userId,
                    'reportName': $rootScope.reportObject.name,
                    'reportId': $rootScope.reportObject.reportId,
                    'reportMasterId': $rootScope.reportObject.reportMasterId,
                    'status': $rootScope.reportObject.status,
                    'areMocPending': $rootScope.reportObject.areMocPending,
                    'previousYearEndDate': $scope.sessionUser.previousYearEndDate
                };
                var copy = angular.extend({}, i, row1);

                sc9SuplFactory
                    .submitNineSupl(copy)
                    .then(function (data) {

                        console.log("************** submitStatus ~ Sequence (reportId) ~ reportStatus : " + data);
                        var result = [];
                        result = data.split('~');


                        if (result[0] == 0) {
                            $rootScope.reportObject.reportId = result[1];
                            $rootScope.reportObject.status = result[2];
                            console.log("# reportId " + $rootScope.reportObject.reportId);
                            console.log("# status " + $rootScope.reportObject.status);

                            if (sc9Supl.row.save == false) {
                                sc9Supl.displayMessage = "Report Submitted Successfully";
                                if (sc9Supl.displayMessage) {

                                    $('#myModal1')
                                        .modal({
                                            backdrop: 'static', keyboard: false, modal: true
                                        });
                                    $('#myModal1')
                                        .on('shown.bs.modal', function () {
                                            $('#myModal1')
                                                .trigger('focus');
                                        });
                                }
                            } else {

                                sc9Supl.displayMessage = "Report Saved Successfully";
                                if (sc9Supl.displayMessage) {
                                    $('#myModal5')
                                        .modal({
                                            backdrop: 'static', keyboard: false, modal: true
                                        });
                                    $('#myModal5')
                                        .on('shown.bs.modal', function () {
                                            $('#myModal5')
                                                .trigger('focus');
                                        });
                                }

                            }

                        }
                        console.log(data);
                    }, function (errResponse) {
                        console
                            .error('Error while Submitting Schedule 10');
                    });
            }


        }

        sc9Supl.redirect = function () {
            $timeout(function () {
                $state.go('circle_maker.worklist');
            }, 500);
        }

        sc9Supl.parseFloat = function (value) {
            if (isNaN(value)) {
                value = 0;
            }
            return parseFloat(value * 1);
        }
        $scope
            .$watchCollection('sc9Supl.row', function (newValue, oldValue) {
                if (sc9Supl.row != undefined) {


                    sc9Supl.row.grandTotlPro = (sc9Supl.parseFloat(sc9Supl.row.loanAdvPro) + sc9Supl.parseFloat(sc9Supl.row.bilPurPro)).toFixed(2);
                    sc9Supl.row.grandTotlGrAmt = (sc9Supl.parseFloat(sc9Supl.row.loanAdvGrAmt) + sc9Supl.parseFloat(sc9Supl.row.bilPurGrAmt)).toFixed(2);


                    sc9Supl.row.dueGrAmt = (sc9Supl.parseFloat(sc9Supl.row.coopBankGrAmt) + sc9Supl.parseFloat(sc9Supl.row.commBankGrAmt) + sc9Supl.parseFloat(sc9Supl.row.bankOutIndGrAmt)).toFixed(2);
                    sc9Supl.row.duePro = (sc9Supl.parseFloat(sc9Supl.row.coopBankPro) + sc9Supl.parseFloat(sc9Supl.row.commBankPro) + sc9Supl.parseFloat(sc9Supl.row.bankOutIndPro)).toFixed(2);


                    sc9Supl.row.loanAdvGrAmt = (sc9Supl.parseFloat(sc9Supl.row.loanAdvCreGrAmt) + sc9Supl.parseFloat(sc9Supl.row.dueGrAmt)).toFixed(2);
                    sc9Supl.row.loanAdvPro = (sc9Supl.parseFloat(sc9Supl.row.loanAdvCrePro) + sc9Supl.parseFloat(sc9Supl.row.duePro)).toFixed(2);

                    sc9Supl.row.othForeBilPurGrAmt = (sc9Supl.parseFloat(sc9Supl.row.payIndGrAmt) + sc9Supl.parseFloat(sc9Supl.row.payOutGrAmt)).toFixed(2);
                    sc9Supl.row.othForeBilPurPro = (sc9Supl.parseFloat(sc9Supl.row.payIndPro) + sc9Supl.parseFloat(sc9Supl.row.payOutPro)).toFixed(2);


                    sc9Supl.row.foreBilPurGrAmt = (sc9Supl.parseFloat(sc9Supl.row.expBillGrAmt) + sc9Supl.parseFloat(sc9Supl.row.impBillGrAmt) + sc9Supl.parseFloat(sc9Supl.row.othForeBilPurGrAmt)).toFixed(2);
                    sc9Supl.row.foreBilPurPro = (sc9Supl.parseFloat(sc9Supl.row.expBillPro) + sc9Supl.parseFloat(sc9Supl.row.impBillPro) + sc9Supl.parseFloat(sc9Supl.row.othForeBilPurPro)).toFixed(2);

                    sc9Supl.row.bilPurGrAmt = (sc9Supl.parseFloat(sc9Supl.row.inlBilPurGrAmt) + sc9Supl.parseFloat(sc9Supl.row.foreBilPurGrAmt)).toFixed(2);
                    sc9Supl.row.bilPurPro = (sc9Supl.parseFloat(sc9Supl.row.inlBilPurPro) + sc9Supl.parseFloat(sc9Supl.row.foreBilPurPro)).toFixed(2);


                }
            });


    });
///////////////////////////////////////
 fetch data on useeffect dao implementations
 api call /Maker/getSavedDataNineSupl
 
 public SC9Supl getSavedDataNineSupl(SC9Supl row) {
        // TODO Auto-generated method stub

        SC9Supl report = null;
//        log.info("circleCode" + row.getCircleCode());
//        log.info("QuarterEndDate" + row.getQuarterEndDate());
        try {

            String query = "select SC9_SUPL_ID,SC9_SUPL_FK, " +
                    "SC9_SUPL_DATE, " +
                    "SC9_SUPL_CIRCLE, " +
                    "SC9_SUPL_DESC, " +
                    "SC9_SUPL_GR_AMT, " +
                    "SC9_SUPL_PRO_AMT from BS_SC9_SUPL where SC9_SUPL_CIRCLE=? and SC9_SUPL_DATE=to_date(?,'dd/mm/yyyy')";

            report = jdbcTemplate.query(query, new Object[]{row.getCircleCode(), row.getQuarterEndDate()},
                    new ResultSetExtractor<SC9Supl>() {
                        @Override
                        public SC9Supl extractData(ResultSet rs) throws SQLException, DataAccessException {
                            SC9Supl row = null;
                            row = new SC9Supl();
                            // TODO Auto-generated method stub

                            while (rs.next()) {
                                String id = rs.getString("SC9_SUPL_ID");

                                if (id.equalsIgnoreCase("1")) {
                                    row.setBilPurGrAmt(rs.getString("SC9_SUPL_GR_AMT"));
                                    row.setBilPurPro(rs.getString("SC9_SUPL_PRO_AMT"));
                                } else if (id.equalsIgnoreCase("2")) {

                                    row.setInlBilPurGrAmt(rs.getString("SC9_SUPL_GR_AMT"));
                                    row.setInlBilPurPro(rs.getString("SC9_SUPL_PRO_AMT"));

                                } else if (id.equalsIgnoreCase("3")) {

                                    row.setForeBilPurGrAmt(rs.getString("SC9_SUPL_GR_AMT"));
                                    row.setForeBilPurPro(rs.getString("SC9_SUPL_PRO_AMT"));

                                } else if (id.equalsIgnoreCase("4")) {

                                    row.setExpBillGrAmt(rs.getString("SC9_SUPL_GR_AMT"));
                                    row.setExpBillPro(rs.getString("SC9_SUPL_PRO_AMT"));

                                } else if (id.equalsIgnoreCase("5")) {
                                    row.setImpBillGrAmt(rs.getString("SC9_SUPL_GR_AMT"));
                                    row.setImpBillPro(rs.getString("SC9_SUPL_PRO_AMT"));

                                } else if (id.equalsIgnoreCase("6")) {
                                    row.setOthForeBilPurGrAmt(rs.getString("SC9_SUPL_GR_AMT"));
                                    row.setOthForeBilPurPro(rs.getString("SC9_SUPL_PRO_AMT"));
                                } else if (id.equalsIgnoreCase("7")) {
                                    row.setPayIndGrAmt(rs.getString("SC9_SUPL_GR_AMT"));
                                    row.setPayIndPro(rs.getString("SC9_SUPL_PRO_AMT"));

                                } else if (id.equalsIgnoreCase("8")) {

                                    row.setPayOutGrAmt(rs.getString("SC9_SUPL_GR_AMT"));
                                    row.setPayOutPro(rs.getString("SC9_SUPL_PRO_AMT"));

                                } else if (id.equalsIgnoreCase("9")) {

                                    row.setLoanAdvGrAmt(rs.getString("SC9_SUPL_GR_AMT"));
                                    row.setLoanAdvPro(rs.getString("SC9_SUPL_PRO_AMT"));
                                } else if (id.equalsIgnoreCase("10")) {
                                    row.setLoanAdvCreGrAmt(rs.getString("SC9_SUPL_GR_AMT"));
                                    row.setLoanAdvCrePro(rs.getString("SC9_SUPL_PRO_AMT"));

                                } else if (id.equalsIgnoreCase("11")) {
                                    row.setDueGrAmt(rs.getString("SC9_SUPL_GR_AMT"));
                                    row.setDuePro(rs.getString("SC9_SUPL_PRO_AMT"));

                                } else if (id.equalsIgnoreCase("12")) {


                                    row.setCoopBankGrAmt(rs.getString("SC9_SUPL_GR_AMT"));
                                    row.setCoopBankPro(rs.getString("SC9_SUPL_PRO_AMT"));
                                } else if (id.equalsIgnoreCase("13")) {


                                    row.setCommBankGrAmt(rs.getString("SC9_SUPL_GR_AMT"));
                                    row.setCommBankPro(rs.getString("SC9_SUPL_PRO_AMT"));
                                } else if (id.equalsIgnoreCase("14")) {

                                    row.setBankOutIndGrAmt(rs.getString("SC9_SUPL_GR_AMT"));
                                    row.setBankOutIndPro(rs.getString("SC9_SUPL_PRO_AMT"));
                                } else if (id.equalsIgnoreCase("15")) {

                                    row.setGrandTotlGrAmt(rs.getString("SC9_SUPL_GR_AMT"));
                                    row.setGrandTotlPro(rs.getString("SC9_SUPL_PRO_AMT"));
                                }

                            }

                            return row;

                        }

                    });

        } catch (DataAccessException e) {
            log.error("Data Access Exception Occurred");
        }

        return report;

        // return null;
    }
//////////////////////////////////////////////
same call for save and submit only status true false
api call /Maker/submitNineSupl
public String submitNineSupl(SC9Supl row) {


        TransactionDefinition def = new DefaultTransactionDefinition();
        TransactionStatus status = transactionManager.getTransaction(def);
        int flag = -1;

        String seq = null;
        String result = "";
        List<Object[]> inputList = new ArrayList<Object[]>();

        JdbcTemplate jdbcTemplateObject = new JdbcTemplate(dataSource);

        try {
//            log.info("status *************************************** " + row.getStatus());

            String sequence = makerDao.generateSequence();


            String query = "insert into BS_SC9_SUPL(SC9_SUPL_ID,SC9_SUPL_FK,SC9_SUPL_DATE,SC9_SUPL_CIRCLE,SC9_SUPL_DESC,SC9_SUPL_GR_AMT,SC9_SUPL_PRO_AMT) VALUES(?,?,to_date(?,'dd/mm/yyyy'),?,?,?,?)";

            if (row.getStatus() == null || row.getStatus().equalsIgnoreCase("")) {

                Object[] tempTen1 = {

                        "1", sequence, row.getQuarterEndDate(), row.getCircleCode(), "1. Bills Purchased and discounted (excluding bills rediscounted) (1.1 + 1.2)",
                        row.getBilPurGrAmt(), row.getBilPurPro()};
                inputList.add(tempTen1);

                Object[] tempTen2 = {

                        "2", sequence, row.getQuarterEndDate(), row.getCircleCode(), "1.1 Inland Bills Purchased and Discounted", row.getInlBilPurGrAmt(), row.getInlBilPurPro()};
                inputList.add(tempTen2);

                Object[] tempTen3 = {

                        "3", sequence, row.getQuarterEndDate(), row.getCircleCode(), "1.2 Foreign Bills Purchased and Discounted (excluding bills rediscounted) ( 1.2.1+1.2.2+1.2.3)", row.getForeBilPurGrAmt(), row.getForeBilPurPro()
                };
                inputList.add(tempTen3);

                Object[] tempTen4 = {

                        "4", sequence, row.getQuarterEndDate(), row.getCircleCode(),
                        "1.2.1 Export Bills drawn in India", row.getExpBillGrAmt(), row.getExpBillPro()};
                inputList.add(tempTen4);

                Object[] tempTen5 = {

                        "5", sequence, row.getQuarterEndDate(), row.getCircleCode(),
                        "1.2.2 Import Bills drawn on and payable in India", row.getImpBillGrAmt(), row.getImpBillPro()};
                inputList.add(tempTen5);

                Object[] tempTen6 = {

                        "6", sequence, row.getQuarterEndDate(), row.getCircleCode(),
                        "1.2.3 Other Foreign Bills Purchased and Discounted 9excluding bills rediscounted) (1.2.3.1+1.2.3.2)", row.getOthForeBilPurGrAmt(), row.getOthForeBilPurPro()};
                inputList.add(tempTen6);

                Object[] tempTen7 = {

                        "7", sequence, row.getQuarterEndDate(), row.getCircleCode(),
                        "1.2.3.1 Payable in India", row.getPayIndGrAmt(), row.getPayIndPro()};
                inputList.add(tempTen7);

                Object[] tempTen8 = {

                        "8", sequence, row.getQuarterEndDate(), row.getCircleCode(),
                        "1.2.3.2 Payable outside India", row.getPayOutGrAmt(), row.getPayOutPro()};
                inputList.add(tempTen8);

                Object[] tempTen9 = {

                        "9", sequence, row.getQuarterEndDate(), row.getCircleCode(),
                        "2. Loans and Advances (2.1 + 2.2)", row.getLoanAdvGrAmt(), row.getLoanAdvPro()};
                inputList.add(tempTen9);

                Object[] tempTen10 = {

                        "10", sequence, row.getQuarterEndDate(), row.getCircleCode(), "2.1 Loans and Advances, Cash Credit & Overdrafts (excluding due from Banks vide 2.2 below)", row.getLoanAdvCreGrAmt(), row.getLoanAdvCrePro()
                };
                inputList.add(tempTen10);

                Object[] tempTen11 = {

                        "11", sequence, row.getQuarterEndDate(), row.getCircleCode(),
                        "2.2 Due from Banks ( 2.2.1+2.2.2+2.2.3)", row.getDueGrAmt(), row.getDuePro()};
                inputList.add(tempTen11);

                Object[] tempTen12 = {

                        "12", sequence, row.getQuarterEndDate(), row.getCircleCode(), "2.2.1 Co-operative Banks in India", row.getCoopBankGrAmt(), row.getCoopBankPro()
                };
                inputList.add(tempTen12);
                Object[] tempTen13 = {

                        "13", sequence, row.getQuarterEndDate(), row.getCircleCode(), "2.2.2 Commercial Banks in India", row.getCommBankGrAmt(), row.getCommBankPro()
                };
                inputList.add(tempTen13);


                Object[] tempTen14 = {

                        "14", sequence, row.getQuarterEndDate(), row.getCircleCode(), "2.2.3 Banks outside India", row.getBankOutIndGrAmt(), row.getBankOutIndPro()
                };
                inputList.add(tempTen14);
                Object[] tempTen15 = {

                        "15", sequence, row.getQuarterEndDate(), row.getCircleCode(), "3. Grand Total (1 + 2)", row.getGrandTotlGrAmt(), row.getGrandTotlPro()
                };
                inputList.add(tempTen15);

                String statusInsert = "20";
                if (row.isSave()) {
                    statusInsert = "11";
                }
                int check = -1;

                seq = sequence + "~" + statusInsert;

                if (check == -1) {
                    jdbcTemplateObject.batchUpdate(query, inputList);
                    // String sequence = makerDao.generateSequence();

                    makerDao.reportEntryInMasterTable(sequence, row.getReportMasterId(), row.getCircleCode(),
                            row.getQuarterEndDate(), row.getReportName(), row.getUserId(), statusInsert, "I");

                    transactionManager.commit(status);
                    check = 0;
                }

                flag = check;

            } else {
                String updateQuery = "UPDATE BS_SC9_SUPL SET SC9_SUPL_GR_AMT=?,SC9_SUPL_PRO_AMT=? WHERE SC9_SUPL_ID=? and SC9_SUPL_CIRCLE=? and SC9_SUPL_DATE = to_date(?,'dd/mm/yyyy') ";

                Object[] tempTen1 = {

                        row.getBilPurGrAmt(), row.getBilPurPro(), "1", row.getCircleCode(), row.getQuarterEndDate()};
                inputList.add(tempTen1);

                Object[] tempTen2 = {

                        row.getInlBilPurGrAmt(), row.getInlBilPurPro(), "2", row.getCircleCode(), row.getQuarterEndDate()};
                inputList.add(tempTen2);

                Object[] tempTen3 = {

                        row.getForeBilPurGrAmt(), row.getForeBilPurPro(), "3", row.getCircleCode(), row.getQuarterEndDate()};
                inputList.add(tempTen3);

                Object[] tempTen4 = {

                        row.getExpBillGrAmt(), row.getExpBillPro(), "4", row.getCircleCode(), row.getQuarterEndDate()};
                inputList.add(tempTen4);

                Object[] tempTen5 = {

                        row.getImpBillGrAmt(), row.getImpBillPro(), "5", row.getCircleCode(), row.getQuarterEndDate()};
                inputList.add(tempTen5);

                Object[] tempTen6 = {

                        row.getOthForeBilPurGrAmt(), row.getOthForeBilPurPro(), "6", row.getCircleCode(), row.getQuarterEndDate()};
                inputList.add(tempTen6);

                Object[] tempTen7 = {

                        row.getPayIndGrAmt(), row.getPayIndPro(), "7", row.getCircleCode(), row.getQuarterEndDate()};
                inputList.add(tempTen7);

                Object[] tempTen8 = {

                        row.getPayOutGrAmt(), row.getPayOutPro(), "8", row.getCircleCode(), row.getQuarterEndDate()};
                inputList.add(tempTen8);

                Object[] tempTen9 = {

                        row.getLoanAdvGrAmt(), row.getLoanAdvPro(), "9", row.getCircleCode(), row.getQuarterEndDate()};
                inputList.add(tempTen9);

                Object[] tempTen10 = {

                        row.getLoanAdvCreGrAmt(), row.getLoanAdvCrePro(), "10", row.getCircleCode(), row.getQuarterEndDate()};
                inputList.add(tempTen10);

                Object[] tempTen11 = {

                        row.getDueGrAmt(), row.getDuePro(), "11", row.getCircleCode(), row.getQuarterEndDate()};
                inputList.add(tempTen11);

                Object[] tempTen12 = {

                        row.getCoopBankGrAmt(), row.getCoopBankPro(), "12", row.getCircleCode(), row.getQuarterEndDate()};
                inputList.add(tempTen12);

                Object[] tempTen13 = {

                        row.getCommBankGrAmt(), row.getCommBankPro(), "13", row.getCircleCode(), row.getQuarterEndDate()};
                inputList.add(tempTen13);

                Object[] tempTen14 = {

                        row.getBankOutIndGrAmt(), row.getBankOutIndPro(), "14", row.getCircleCode(), row.getQuarterEndDate()};
                inputList.add(tempTen14);

                Object[] tempTen15 = {

                        row.getGrandTotlGrAmt(), row.getGrandTotlPro(), "15", row.getCircleCode(), row.getQuarterEndDate()};
                inputList.add(tempTen15);

                String statusInsert = "21";
                if (row.isSave()) {
                    statusInsert = "11";
                }

                int check = -1;
                seq = row.getReportId() + "~" + statusInsert;

                if (check == -1) {
                    jdbcTemplateObject.batchUpdate(updateQuery, inputList);
                    // String sequence = makerDao.generateSequence();
                    makerDao.reportEntryInMasterTable(row.getReportId(), row.getReportMasterId(), row.getCircleCode(),
                            row.getQuarterEndDate(), row.getReportName(), row.getUserId(), statusInsert, "U");

                    transactionManager.commit(status);
                    check = 0;
                }

                flag = check;

            }
        } catch (TransactionException e) {
            log.error("Transaction Exception Occur " + e);
            transactionManager.rollback(status);

        } catch (DataAccessException e) {
            log.error("Data Access Exception Occur " + e);
        }
        log.info("Existing Submit SC 9 Supplementary flag " + flag);
        result = flag + "~" + seq;
//        log.info("############  SC9 Supplementary  submit result is " + result);
        return result;


    }

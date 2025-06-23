function focusfunc() {
    var updateFlag = '${updateFlag}';
    if (updateFlag != 'Y') {
        $('#myModal1').modal({
            backdrop: 'static',
            keyboard: false,
            modal: true
        });
    }
}




function saveMe() {

    var valid = true;
    var formName = window.document.forms[0];
    var table = document.getElementById('example1');
    var tabLength = table.rows.length;
    var count = tabLength;

    if (count >= 13) {

        for (i = 13; i < count; i++) {

            if (document.getElementById("particularsList" + i).value == "") {
                var errorMsg="Please Enter Particulars";
                $('#errorModal .modal-body').text(errorMsg);
                $('#errorModal').modal('show');
                valid = false;
            }

            if (document.getElementById("particularsList" + i).value !== "") {
                if (document.getElementById("proviAmt2016List" + i).value == ""
                    || document.getElementById("proviAmt2016List" + i).value == "0.00") {
                    var errorMsg="Please Enter the Amount for Other Particulars";
                    $('#errorModal .modal-body').text(errorMsg);
                    $('#errorModal').modal('show');
                    valid = false;
                }
            }

        }
    }




    if(valid == true) {
        document.getElementById("action").value = "save";

        var myform = $("#RW04").serialize();

        $.ajax({
            type: "POST",
            url: "./save",
            async: false,
            data: myform,
            success: function (data) {
                console.log("data"+data);
                var msg= data.split('%')[0];
                console.log("msg"+msg);
                $('#saveModal .modal-body').text(msg);
                $('#saveModal').modal('show');


                document.getElementById("reportId").value = data.split('%')[1];
                document.getElementById("updateFlag").value = data.split('%')[2];
                document.getElementById("dataFlag").value = "U";
                document.getElementById("nilFlag").value = "";
            },
            error: function (jqXHR, textStatus, errorMessage) {
                var errorMsg="no response received";
                $('#errorModal .modal-body').text(errorMsg);
                $('#errorModal').modal('show');
            },
            complete: function () {

            },
        });

    }
}

function debtCalculate() {

    //alert("in debtFunction");
    var fraudsDebitedProvReq;
    var fraudsDebitedPrior100ProvReq;
    var fraudsDebitedDelayedProvReq

    if (isNaN(parseFloat(document.getElementById("fraudsDebitedPrior100ProvReq").value))) {

        fraudsDebitedPrior100ProvReq = 0.0;

    }
    else {
        fraudsDebitedPrior100ProvReq = parseFloat(document
            .getElementById("fraudsDebitedPrior100ProvReq").value);
    }




    if (isNaN(parseFloat(document.getElementById("fraudsDebitedDelayedProvReq").value))) {

        fraudsDebitedDelayedProvReq = 0.0;
    }
    else {
        fraudsDebitedDelayedProvReq = parseFloat(document
            .getElementById("fraudsDebitedDelayedProvReq").value);
    }


    fraudsDebitedProvReq = fraudsDebitedPrior100ProvReq + fraudsDebitedDelayedProvReq;

    document.getElementById("fraudsDebitedProvReq").value = fraudsDebitedProvReq.toFixed(2);
    //alert(document.getElementById("fraudsDebitedProvReq").value + " value in variable: " + fraudsDebitedProvReq);


    var fraudsOthersProvReq;
    var fraudsOthersPrior100ProvReq;

    var fraudsOthersDelayedProvReq

    if (isNaN(parseFloat(document.getElementById("fraudsOthersPrior100ProvReq").value))) {

        fraudsOthersPrior100ProvReq = 0.0;
    }
    else {
        fraudsOthersPrior100ProvReq = parseFloat(document
            .getElementById("fraudsOthersPrior100ProvReq").value);
    }
    if (isNaN(parseFloat(document.getElementById("fraudsOthersDelayedProvReq").value))) {

        fraudsOthersDelayedProvReq = 0.0;
    }
    else {
        fraudsOthersDelayedProvReq = parseFloat(document
            .getElementById("fraudsOthersDelayedProvReq").value);
    }

    fraudsOthersProvReq= fraudsOthersPrior100ProvReq+fraudsOthersDelayedProvReq;
    document.getElementById("fraudsOthersProvReq").value = fraudsOthersProvReq.toFixed(2);

}


function submitRequest(operation) {
    var valid = true;
    var table = document.getElementById('example1');
    var tabLength = table.rows.length;
    var errorMsg="hello";
    var addRowCount =0;
    var staticCount =0;

    var count = tabLength;

    if(count >13){
        for (i = 13; i < count; i++) {
            if (document.getElementById("particularsList" + i).value == "") {
                valid = false;
                errorMsg = "Particulars columns can't be blank for Added rows.";
                addRowCount ++;
            } else
            {
                if (document.getElementById("proviAmt2016List" + i).value == ""
                    || document.getElementById("proviAmt2016List" + i).value == "0.00") {
                    errorMsg = "Provision amounts can't be blank for Added rows.";
                    addRowCount ++;
                }
            }
        }
        console.log("Invalid count :"+addRowCount);


    } else
     if( (document.getElementById("fraudsDebitedProvAfter").value  == ""&&  document.getElementById("fraudsDebitedWrite").value == ""&&
        document.getElementById("fraudsDebitedAddition").value == ""&&  document.getElementById("fraudsDebitedReduction").value == ""&&

        document.getElementById("othersRecalledProvAfter").value == ""&&document.getElementById("othersRecalledWrite").value == ""&&
        document.getElementById("othersRecalledAddition").value == ""&&document.getElementById("othersRecalledReduction").value == ""&&

        document.getElementById("fraudsOthersProvAfter").value == ""&&document.getElementById("fraudsOthersWrite").value == ""&&
        document.getElementById("fraudsOthersAddition").value == "" && document.getElementById("fraudsOthersReduction").value == ""&&


        document.getElementById("revenueProvAfter").value == ""&&document.getElementById("revenueWrite").value == ""&&
        document.getElementById("revenueAddition").value == ""&&document.getElementById("revenueReduction").value == ""&&

        document.getElementById("fsloProvAfter").value == ""&&document.getElementById("fsloWrite").value == ""&&
        document.getElementById("fsloReduction").value == ""&&document.getElementById("fsloProvOn").value == ""&&

        document.getElementById("outstandingProvAfter").value == ""&&document.getElementById("outstandingWrite").value == ""&&
        document.getElementById("outstandingAddition").value == ""&&document.getElementById("outstandingReduction").value == ""&&

        document.getElementById("npainterestProvAfter").value == ""&&document.getElementById("npainterestWrite").value == ""&&
        document.getElementById("npainterestAddition").value == ""&&document.getElementById("npainterestReduction").value == "")       ||
        (document.getElementById("fraudsDebitedProvAfter").value  == 0.00&&  document.getElementById("fraudsDebitedWrite").value == 0.00&&
            document.getElementById("fraudsDebitedAddition").value == 0.00&&  document.getElementById("fraudsDebitedReduction").value == 0.00&&

            document.getElementById("othersRecalledProvAfter").value == 0.00&&document.getElementById("othersRecalledWrite").value == 0.00&&
            document.getElementById("othersRecalledAddition").value == 0.00&&document.getElementById("othersRecalledReduction").value == 0.00&&

            document.getElementById("fraudsOthersProvAfter").value == 0.00&&document.getElementById("fraudsOthersWrite").value == 0.00&&
            document.getElementById("fraudsOthersAddition").value == 0.00 && document.getElementById("fraudsOthersReduction").value == 0.00&&


            document.getElementById("revenueProvAfter").value == 0.00&&document.getElementById("revenueWrite").value == 0.00&&
            document.getElementById("revenueAddition").value == 0.00&&document.getElementById("revenueReduction").value == 0.00&&

            document.getElementById("fsloProvAfter").value == 0.00&&document.getElementById("fsloWrite").value == 0.00&&
            document.getElementById("fsloReduction").value == 0.00&&document.getElementById("fsloProvOn").value == 0.00&&

            document.getElementById("outstandingProvAfter").value == 0.00&&document.getElementById("outstandingWrite").value == 0.00&&
            document.getElementById("outstandingAddition").value == 0.00&&document.getElementById("outstandingReduction").value == 0.00&&

            document.getElementById("npainterestProvAfter").value == 0.00&&document.getElementById("npainterestWrite").value == 0.00&&
            document.getElementById("npainterestAddition").value == 0.00&&document.getElementById("npainterestReduction").value == 0.00) )
    {
        errorMsg = "No data in the form either enter data or submit as Nil Report.";
        staticCount ++;
    }

    if (staticCount ==0){
        staticCount = submitreq();
        if (staticCount !=0){
            errorMsg = "Provision amounts are not matching for highlighted cells.";
        }
    }

    document.getElementById("action").value = "submit";
    if (addRowCount==0 && staticCount ==0){
        console.log("Action Submit");
        var operation = './submit';
        document.getElementById('RW04').action = operation;
        document.getElementById('RW04').submit();
    } else {

        $('#errorModal .modal-body').text(errorMsg);
        $('#errorModal').modal('show');
    }
}

function provisionable1() {

    document.getElementById("fraudsDebitedProvOn").style.borderColor = "#d2d6de";
    // document.getElementById("fraudsOthersProvOn").style.borderColor="#d2d6de";

    var fraudsDebitedProvOn = document.getElementById("fraudsDebitedProvOn").value;
    var fraudsDebitedPrior100ProvOn;
    //var fraudsDebitedPrior75ProvOn;
    //var fraudsDebitedPrior50ProvOn;
    //var fraudsDebitedPrior25ProvOn;
    var fraudsDebitedDelayedProvOn;

    if (isNaN(parseFloat(document.getElementById("fraudsDebitedPrior100ProvOn").value))) {

        fraudsDebitedPrior100ProvOn = 0.0;
    }

    else
        fraudsDebitedPrior100ProvOn = parseFloat(document
            .getElementById("fraudsDebitedPrior100ProvOn").value);

    if (isNaN(parseFloat(document.getElementById("fraudsDebitedDelayedProvOn").value))) {

        fraudsDebitedDelayedProvOn = 0.0;
    }

    else
        fraudsDebitedDelayedProvOn = parseFloat(document
            .getElementById("fraudsDebitedDelayedProvOn").value);

    var total = (fraudsDebitedPrior100ProvOn + fraudsDebitedDelayedProvOn).toFixed(2);
    fraudsDebitedProvOn = document.getElementById("fraudsDebitedProvOn").value;
    if (fraudsDebitedProvOn != total) {
        // alert("Provisional Amount not Matching for Sr.NO:01");
        document.getElementById("fraudsDebitedProvOn").style.borderColor = "#FF0000";

        // document.getElementById("fraudsDebitedProvOn").();

    }
}

function provisionable2() {
    document.getElementById("fraudsOthersProvOn").style.borderColor = "#d2d6de";

    var fraudsDebitedProvOn = document.getElementById("fraudsOthersProvOn").value;
    var fraudsOthersPrior100ProvOn;

    var fraudsOthersDelayedProvOn;

    if (isNaN(parseFloat(document.getElementById("fraudsOthersPrior100ProvOn").value))) {

        fraudsOthersPrior100ProvOn = 0.0;
    }

    else
        fraudsOthersPrior100ProvOn = parseFloat(document
            .getElementById("fraudsOthersPrior100ProvOn").value);

    if (isNaN(parseFloat(document.getElementById("fraudsOthersDelayedProvOn").value))) {

        fraudsOthersDelayedProvOn = 0.0;
    }

    else
        fraudsOthersDelayedProvOn = parseFloat(document
            .getElementById("fraudsOthersDelayedProvOn").value);

    var total11 = (fraudsOthersPrior100ProvOn + fraudsOthersDelayedProvOn   )
        .toFixed(2);
    fraudsOthersProvOn = document.getElementById("fraudsOthersProvOn").value;
    if (fraudsOthersProvOn != total11) {
        // alert("Provisional Amount not Matching Sr.NO:03");
        document.getElementById("fraudsOthersProvOn").style.borderColor = "#FF0000";

        // document.getElementById("fraudsOthersProvOn").focus();

    }
}

function submitreq() {

    document.getElementById("fraudsDebitedProvOn").style.borderColor = "#d2d6de";
    document.getElementById("fraudsOthersProvOn").style.borderColor = "#d2d6de";

    var fraudsDebitedProvOn = document.getElementById("fraudsDebitedProvOn").value;
    var fraudsDebitedPrior100ProvOn;
    //var fraudsDebitedPrior75ProvOn;
   // var fraudsDebitedPrior50ProvOn;
    //var fraudsDebitedPrior25ProvOn;
    var fraudsDebitedDelayedProvOn;

    if (isNaN(parseFloat(document.getElementById("fraudsDebitedPrior100ProvOn").value))) {

        fraudsDebitedPrior100ProvOn = 0.0;
    }

    else
        fraudsDebitedPrior100ProvOn = parseFloat(document
            .getElementById("fraudsDebitedPrior100ProvOn").value);


    if (isNaN(parseFloat(document.getElementById("fraudsDebitedDelayedProvOn").value))) {

        fraudsDebitedDelayedProvOn = 0.0;
    }

    else
        fraudsDebitedDelayedProvOn = parseFloat(document
            .getElementById("fraudsDebitedDelayedProvOn").value);

    var total = (fraudsDebitedPrior100ProvOn + fraudsDebitedDelayedProvOn).toFixed(2);
    console.log("total"+total);
    fraudsDebitedProvOn = document.getElementById("fraudsDebitedProvOn").value;

    var pro = true;

        if (fraudsDebitedProvOn != total) {


            document.getElementById("fraudsDebitedProvOn").style.borderColor = "#FF0000";
            document.getElementById("fraudsDebitedProvOn").focus();
            pro = false;
        }

        var fraudsDebitedProvOn = document.getElementById("fraudsOthersProvOn").value;
        var fraudsOthersPrior100ProvOn;

        var fraudsOthersDelayedProvOn;

        if (isNaN(parseFloat(document.getElementById("fraudsOthersPrior100ProvOn").value))) {

            fraudsOthersPrior100ProvOn = 0.0;
        }

        else
            fraudsOthersPrior100ProvOn = parseFloat(document
                .getElementById("fraudsOthersPrior100ProvOn").value);

        if (isNaN(parseFloat(document.getElementById("fraudsOthersDelayedProvOn").value))) {

            fraudsOthersDelayedProvOn = 0.0;
        }

        else
            fraudsOthersDelayedProvOn = parseFloat(document
                .getElementById("fraudsOthersDelayedProvOn").value);

        var total11 = (fraudsOthersPrior100ProvOn +fraudsOthersDelayedProvOn)
            .toFixed(2);
        fraudsOthersProvOn = document.getElementById("fraudsOthersProvOn").value;

        if (fraudsOthersProvOn != total11) {

            document.getElementById("fraudsOthersProvOn").style.borderColor = "#FF0000";
            document.getElementById("fraudsOthersProvOn").focus();
            pro = false;
        }






    if (pro) {
        return 0;
    } else {
        return 1;
    }

}

function calculateDynamicTotal(i) {

    var provAmt2015List = document.getElementById("provAmt2015List" + i).value;
    var writeOffDur12monList = document.getElementById("writeOffDur12monList"
        + i).value;
    var additionDur12monList = document.getElementById("additionDur12monList"
        + i).value;
    var reduInProviAmtList = document.getElementById("reduInProviAmtList" + i).value;

    if (isNaN(parseFloat(provAmt2015List))) {
        provAmt2015List = 0.0;
    } else
        provAmt2015List = parseFloat(provAmt2015List);

    if (isNaN(parseFloat(writeOffDur12monList))) {
        writeOffDur12monList = 0.0;
    } else
        writeOffDur12monList = parseFloat(writeOffDur12monList);

    if (isNaN(parseFloat(additionDur12monList))) {
        additionDur12monList = 0.0;
    } else
        additionDur12monList = parseFloat(additionDur12monList);

    if (isNaN(parseFloat(reduInProviAmtList))) {
        reduInProviAmtList = 0.0;
    } else
        reduInProviAmtList = parseFloat(reduInProviAmtList);

    var total = parseFloat(provAmt2015List) - parseFloat(writeOffDur12monList)
        + parseFloat(additionDur12monList) - parseFloat(reduInProviAmtList);

    if (isNaN(total))
        total = 0.0;
    document.getElementById("proviAmt2016List" + i).value = total.toFixed(2);

    var ratePOfProvList = document.getElementById("ratePOfProvList" + i).value;

    if (isNaN(parseFloat(ratePOfProvList))) {
        ratePOfProvList = 0.0;
    } else
        ratePOfProvList = parseFloat(ratePOfProvList);

    var proviAmt2016List = document.getElementById("proviAmt2016List" + i).value;

    if (isNaN(parseFloat(proviAmt2016List))) {
        proviAmt2016List = 0.0;
    } else
        proviAmt2016List = parseFloat(proviAmt2016List);
    if (ratePOfProvList > 100.00) {
        errorMsg="Percentage Should not be above 100";
        $('#errorModal .modal-body').text(errorMsg);
        $('#errorModal').modal('show');
        document.getElementById("ratePOfProvList" + i).value = "";
        document.getElementById("ratePOfProvList" + i).focus();
    }

    var total1 = (parseFloat(ratePOfProvList * total)) / 100;
    if (isNaN(total1))
        total1 = 0.0;
    document.getElementById("provReqList" + i).value = total1.toFixed(2);

}

function calculateTotal() {
    //alert("f");
    var fraudsDebitedProvAfter;
    var fraudsDebitedWrite;
    var fraudsDebitedAddition;
    var fraudsDebitedReduction;

    if (isNaN(parseFloat(document.getElementById("fraudsDebitedProvAfter").value))) {

        fraudsDebitedProvAfter = 0.0;

    }

    else
        fraudsDebitedProvAfter = parseFloat(document
            .getElementById("fraudsDebitedProvAfter").value);

    if (isNaN(parseFloat(document.getElementById("fraudsDebitedWrite").value))) {

        fraudsDebitedWrite = 0.0;
    }

    else
        fraudsDebitedWrite = parseFloat(document
            .getElementById("fraudsDebitedWrite").value);

    if (isNaN(parseFloat(document.getElementById("fraudsDebitedAddition").value))) {

        fraudsDebitedAddition = 0.0;
    }

    else
        fraudsDebitedAddition = parseFloat(document
            .getElementById("fraudsDebitedAddition").value);

    if (isNaN(parseFloat(document.getElementById("fraudsDebitedReduction").value))) {

        fraudsDebitedReduction = 0.0;
    }

    else
        fraudsDebitedReduction = parseFloat(document
            .getElementById("fraudsDebitedReduction").value);

    if (isNaN(parseFloat(document.getElementById("fraudsDebitedProvOn").value))) {

        fraudsDebitedProvOn = 0.0;
    }

    else
        fraudsDebitedProvOn = parseFloat(document
            .getElementById("fraudsDebitedProvOn").value);

    if (isNaN(parseFloat(document.getElementById("fraudsDebitedRate").value))) {

        fraudsDebitedRate = 0.0;
    }

    else {
        fraudsDebitedRate = parseFloat(document
            .getElementById("fraudsDebitedRate").value);
    }

    var fraudsDebitedProvOn = fraudsDebitedProvAfter - fraudsDebitedWrite
        + fraudsDebitedAddition - fraudsDebitedReduction;
    document.getElementById("fraudsDebitedProvOn").value = fraudsDebitedProvOn
        .toFixed(2);

    var fraudsDebitedProvReq = (fraudsDebitedProvOn * fraudsDebitedRate) / 100;
    document.getElementById("fraudsDebitedProvReq").value = fraudsDebitedProvReq
        .toFixed(2);

    var fraudsDebitedPrior100ProvOn;
    var fraudsDebitedPrior100Rate;
    var fraudsDebitedPrior100ProvReq;

    if (isNaN(parseFloat(document.getElementById("fraudsDebitedPrior100ProvOn").value))) {

        fraudsDebitedPrior100ProvOn = 0.0;
    }

    else
        fraudsDebitedPrior100ProvOn = parseFloat(document
            .getElementById("fraudsDebitedPrior100ProvOn").value);

    if (isNaN(parseFloat(document.getElementById("fraudsDebitedPrior100Rate").value))) {

        fraudsDebitedPrior100Rate = 0.0;
    }

    else
        fraudsDebitedPrior100Rate = parseFloat(document
            .getElementById("fraudsDebitedPrior100Rate").value);

    if (isNaN(parseFloat(document
        .getElementById("fraudsDebitedPrior100ProvReq").value))) {

        fraudsDebitedPrior100ProvReq = 0.0;
    }

    else
        fraudsDebitedPrior100ProvReq = parseFloat(document
            .getElementById("fraudsDebitedPrior100ProvReq").value);

    var fraudsDebitedPrior100ProvReq = (fraudsDebitedPrior100ProvOn * fraudsDebitedPrior100Rate) / 100;
    document.getElementById("fraudsDebitedPrior100ProvReq").value = fraudsDebitedPrior100ProvReq
        .toFixed(2);

    var fraudsDebitedDelayedProvOn;
    var fraudsDebitedDelayedRate;
    var fraudsDebitedDelayedProvReq;

    if (isNaN(parseFloat(document.getElementById("fraudsDebitedDelayedProvOn").value))) {

        fraudsDebitedDelayedProvOn = 0.0;
    }

    else
        fraudsDebitedDelayedProvOn = parseFloat(document
            .getElementById("fraudsDebitedDelayedProvOn").value);

    if (isNaN(parseFloat(document.getElementById("fraudsDebitedDelayedRate").value))) {

        fraudsDebitedDelayedRate = 0.0;
    }

    else
        fraudsDebitedDelayedRate = parseFloat(document
            .getElementById("fraudsDebitedDelayedRate").value);

    var fraudsDebitedDelayedProvReq = (fraudsDebitedDelayedProvOn * fraudsDebitedDelayedRate) / 100;
    document.getElementById("fraudsDebitedDelayedProvReq").value = fraudsDebitedDelayedProvReq
        .toFixed(2);

    var fraudsOthersPrior100ProvOn;
    var fraudsOthersPrior100Rate;
    var fraudsOthersPrior100ProvReq;

    if (isNaN(parseFloat(document.getElementById("fraudsOthersPrior100ProvOn").value))) {

        fraudsOthersPrior100ProvOn = 0.0;
    }

    else
        fraudsOthersPrior100ProvOn = parseFloat(document
            .getElementById("fraudsOthersPrior100ProvOn").value);

    if (isNaN(parseFloat(document.getElementById("fraudsOthersPrior100Rate").value))) {

        fraudsOthersPrior100Rate = 0.0;
    }

    else
        fraudsOthersPrior100Rate = parseFloat(document
            .getElementById("fraudsOthersPrior100Rate").value);

    if (isNaN(parseFloat(document.getElementById("fraudsOthersPrior100ProvReq").value))) {

        fraudsOthersPrior100ProvReq = 0.0;
    }

    else
        fraudsOthersPrior100ProvReq = parseFloat(document
            .getElementById("fraudsOthersPrior100ProvReq").value);

    var fraudsOthersPrior100ProvReq = (fraudsOthersPrior100ProvOn * fraudsOthersPrior100Rate) / 100;
    document.getElementById("fraudsOthersPrior100ProvReq").value = fraudsOthersPrior100ProvReq
        .toFixed(2);

    var fraudsOthersDelayedProvOn;
    var fraudsOthersDelayedRate;
    var fraudsOthersDelayedProvReq;

    if (isNaN(parseFloat(document.getElementById("fraudsOthersDelayedProvOn").value))) {

        fraudsOthersDelayedProvOn = 0.0;
    }

    else
        fraudsOthersDelayedProvOn = parseFloat(document
            .getElementById("fraudsOthersDelayedProvOn").value);

    if (isNaN(parseFloat(document.getElementById("fraudsOthersDelayedRate").value))) {

        fraudsOthersDelayedRate = 0.0;
    }

    else
        fraudsOthersDelayedRate = parseFloat(document
            .getElementById("fraudsOthersDelayedRate").value);

    if (isNaN(parseFloat(document.getElementById("fraudsOthersDelayedProvReq").value))) {

        fraudsOthersDelayedProvReq = 0.0;
    }

    else
        fraudsOthersDelayedProvReq = parseFloat(document
            .getElementById("fraudsOthersDelayedProvReq").value);

    var fraudsOthersDelayedProvReq = (fraudsOthersDelayedProvOn * fraudsOthersDelayedRate) / 100;

    document.getElementById("fraudsOthersDelayedProvReq").value = fraudsOthersDelayedProvReq
        .toFixed(2);

    var othersRecalledProvAfter;
    var othersRecalledWrite;
    var othersRecalledAddition;
    var othersRecalledReduction;

    if (isNaN(parseFloat(document.getElementById("othersRecalledProvAfter").value))) {

        othersRecalledProvAfter = 0.0;
    }

    else
        othersRecalledProvAfter = parseFloat(document
            .getElementById("othersRecalledProvAfter").value);

    if (isNaN(parseFloat(document.getElementById("othersRecalledWrite").value))) {

        othersRecalledWrite = 0.0;
    }

    else
        othersRecalledWrite = parseFloat(document
            .getElementById("othersRecalledWrite").value);

    if (isNaN(parseFloat(document.getElementById("othersRecalledAddition").value))) {

        othersRecalledAddition = 0.0;
    }

    else
        othersRecalledAddition = parseFloat(document
            .getElementById("othersRecalledAddition").value);

    if (isNaN(parseFloat(document.getElementById("othersRecalledReduction").value))) {

        othersRecalledReduction = 0.0;
    }

    else
        othersRecalledReduction = parseFloat(document
            .getElementById("othersRecalledReduction").value);

    if (isNaN(parseFloat(document.getElementById("othersRecalledProvOn").value))) {

        othersRecalledProvOn = 0.0;
    }

    else
        othersRecalledProvOn = parseFloat(document
            .getElementById("othersRecalledProvOn").value);

    if (isNaN(parseFloat(document.getElementById("othersRecalledRate").value))) {

        othersRecalledRate = 0.0;
    }

    else
        othersRecalledRate = parseFloat(document
            .getElementById("othersRecalledRate").value);

    var othersRecalledProvOn = othersRecalledProvAfter - othersRecalledWrite
        + othersRecalledAddition - othersRecalledReduction;
    document.getElementById("othersRecalledProvOn").value = othersRecalledProvOn
        .toFixed(2);

    var othersRecalledProvReq = (othersRecalledProvOn * othersRecalledRate) / 100;
    document.getElementById("othersRecalledProvReq").value = othersRecalledProvReq
        .toFixed(2);

    var fraudsOthersProvAfter;
    var fraudsOthersWrite;
    var fraudsOthersAddition;
    var fraudsOthersReduction;

    if (isNaN(parseFloat(document.getElementById("fraudsOthersProvAfter").value))) {

        fraudsOthersProvAfter = 0.0;
    }

    else
        fraudsOthersProvAfter = parseFloat(document
            .getElementById("fraudsOthersProvAfter").value);

    if (isNaN(parseFloat(document.getElementById("fraudsOthersWrite").value))) {

        fraudsOthersWrite = 0.0;
    }

    else
        fraudsOthersWrite = parseFloat(document
            .getElementById("fraudsOthersWrite").value);

    if (isNaN(parseFloat(document.getElementById("fraudsOthersAddition").value))) {

        fraudsOthersAddition = 0.0;
    }

    else
        fraudsOthersAddition = parseFloat(document
            .getElementById("fraudsOthersAddition").value);

    if (isNaN(parseFloat(document.getElementById("fraudsOthersReduction").value))) {

        fraudsOthersReduction = 0.0;
    }

    else
        fraudsOthersReduction = parseFloat(document
            .getElementById("fraudsOthersReduction").value);

    if (isNaN(parseFloat(document.getElementById("fraudsOthersProvOn").value))) {

        fraudsOthersProvOn = 0.0;
    }

    else
        fraudsOthersProvOn = parseFloat(document
            .getElementById("fraudsOthersProvOn").value);

    if (isNaN(parseFloat(document.getElementById("fraudsOthersRate").value))) {

        fraudsOthersRate = 0.0;
    }

    else
        fraudsOthersRate = parseFloat(document
            .getElementById("fraudsOthersRate").value);

    var fraudsOthersProvOn = fraudsOthersProvAfter - fraudsOthersWrite
        + fraudsOthersAddition - fraudsOthersReduction;
    document.getElementById("fraudsOthersProvOn").value = fraudsOthersProvOn
        .toFixed(2);

   /* var fraudsOthersProvReq = (fraudsOthersProvOn * fraudsOthersRate) / 100;
    document.getElementById("fraudsOthersProvReq").value = fraudsOthersProvReq
        .toFixed(2);
*/
    var revenueProvAfter;
    var revenueWrite;
    var revenueAddition;
    var revenueReduction;

    if (isNaN(parseFloat(document.getElementById("revenueProvAfter").value))) {

        revenueProvAfter = 0.0;
    }

    else
        revenueProvAfter = parseFloat(document
            .getElementById("revenueProvAfter").value);

    if (isNaN(parseFloat(document.getElementById("revenueWrite").value))) {

        revenueWrite = 0.0;
    }

    else
        revenueWrite = parseFloat(document.getElementById("revenueWrite").value);

    if (isNaN(parseFloat(document.getElementById("revenueAddition").value))) {

        revenueAddition = 0.0;
    }

    else
        revenueAddition = parseFloat(document.getElementById("revenueAddition").value);

    if (isNaN(parseFloat(document.getElementById("revenueReduction").value))) {

        revenueReduction = 0.0;
    }

    else
        revenueReduction = parseFloat(document
            .getElementById("revenueReduction").value);

    if (isNaN(parseFloat(document.getElementById("revenueProvOn").value))) {

        revenueProvOn = 0.0;
    }

    else
        revenueProvOn = parseFloat(document.getElementById("revenueProvOn").value);

    if (isNaN(parseFloat(document.getElementById("revenueRate").value))) {

        revenueRate = 0.0;
    }

    else
        revenueRate = parseFloat(document.getElementById("revenueRate").value);

    var revenueProvOn = revenueProvAfter - revenueWrite + revenueAddition
        - revenueReduction;
    document.getElementById("revenueProvOn").value = revenueProvOn.toFixed(2);

    var revenueProvReq = (revenueProvOn * revenueRate) / 100;
    document.getElementById("revenueProvReq").value = revenueProvReq.toFixed(2);

    var fsloProvAfter;
    var fsloWrite;
    var fsloAddition;
    var fsloReduction;

    if (isNaN(parseFloat(document.getElementById("fsloProvAfter").value))) {

        fsloProvAfter = 0.0;
    }

    else
        fsloProvAfter = parseFloat(document.getElementById("fsloProvAfter").value);

    if (isNaN(parseFloat(document.getElementById("fsloWrite").value))) {

        fsloWrite = 0.0;
    }

    else
        fsloWrite = parseFloat(document.getElementById("fsloWrite").value);

    if (isNaN(parseFloat(document.getElementById("fsloAddition").value))) {

        fsloAddition = 0.0;
    }

    else
        fsloAddition = parseFloat(document.getElementById("fsloAddition").value);

    if (isNaN(parseFloat(document.getElementById("fsloReduction").value))) {

        fsloReduction = 0.0;
    }

    else
        fsloReduction = parseFloat(document.getElementById("fsloReduction").value);

    if (isNaN(parseFloat(document.getElementById("fsloProvOn").value))) {

        fsloProvOn = 0.0;
    }

    else
        fsloProvOn = parseFloat(document.getElementById("fsloProvOn").value);

    if (isNaN(parseFloat(document.getElementById("fsloRate").value))) {

        fsloRate = 0.0;
    }

    else
        fsloRate = parseFloat(document.getElementById("fsloRate").value);

    var fsloProvOn = fsloProvAfter - fsloWrite + fsloAddition - fsloReduction;
    document.getElementById("fsloProvOn").value = fsloProvOn.toFixed(2);

    var fsloProvReq = (fsloProvOn * fsloRate) / 100;
    document.getElementById("fsloProvReq").value = fsloProvReq.toFixed(2);

    var outstandingProvAfter;
    var outstandingWrite;
    var outstandingAddition;
    var outstandingReduction;

    if (isNaN(parseFloat(document.getElementById("outstandingProvAfter").value))) {

        outstandingProvAfter = 0.0;
    }

    else
        outstandingProvAfter = parseFloat(document
            .getElementById("outstandingProvAfter").value);

    if (isNaN(parseFloat(document.getElementById("outstandingWrite").value))) {

        outstandingWrite = 0.0;
    }

    else
        outstandingWrite = parseFloat(document
            .getElementById("outstandingWrite").value);

    if (isNaN(parseFloat(document.getElementById("outstandingAddition").value))) {

        outstandingAddition = 0.0;
    }

    else
        outstandingAddition = parseFloat(document
            .getElementById("outstandingAddition").value);

    if (isNaN(parseFloat(document.getElementById("outstandingReduction").value))) {

        outstandingReduction = 0.0;
    }

    else
        outstandingReduction = parseFloat(document
            .getElementById("outstandingReduction").value);

    if (isNaN(parseFloat(document.getElementById("outstandingProvOn").value))) {

        outstandingProvOn = 0.0;
    }

    else
        outstandingProvOn = parseFloat(document
            .getElementById("outstandingProvOn").value);

    if (isNaN(parseFloat(document.getElementById("outstandingRate").value))) {

        outstandingRate = 0.0;
    }

    else
        outstandingRate = parseFloat(document.getElementById("outstandingRate").value);

    var outstandingProvOn = outstandingProvAfter - outstandingWrite
        + outstandingAddition - outstandingReduction;
    document.getElementById("outstandingProvOn").value = outstandingProvOn
        .toFixed(2);

    var outstandingProvReq = (outstandingProvOn * outstandingRate) / 100;
    document.getElementById("outstandingProvReq").value = outstandingProvReq
        .toFixed(2);



    var npainterestProvAfter;
    var npainterestWrite;
    var npainterestAddition;
    var npainterestReduction;

    if (isNaN(parseFloat(document.getElementById("npainterestProvAfter").value))) {

        npainterestProvAfter = 0.0;
    }

    else
        npainterestProvAfter = parseFloat(document
            .getElementById("npainterestProvAfter").value);

    if (isNaN(parseFloat(document.getElementById("npainterestWrite").value))) {

        npainterestWrite = 0.0;
    }

    else
        npainterestWrite = parseFloat(document
            .getElementById("npainterestWrite").value);

    if (isNaN(parseFloat(document.getElementById("npainterestAddition").value))) {

        npainterestAddition = 0.0;
    }

    else
        npainterestAddition = parseFloat(document
            .getElementById("npainterestAddition").value);

    if (isNaN(parseFloat(document.getElementById("npainterestReduction").value))) {

        npainterestReduction = 0.0;
    }

    else
        npainterestReduction = parseFloat(document
            .getElementById("npainterestReduction").value);

    if (isNaN(parseFloat(document.getElementById("npainterestProvOn").value))) {

        npainterestProvOn = 0.0;
    }

    else
        npainterestProvOn = parseFloat(document
            .getElementById("npainterestProvOn").value);

    if (isNaN(parseFloat(document.getElementById("npainterestRate").value))) {

        npainterestRate = 0.0;
    }

    else
        npainterestRate = parseFloat(document.getElementById("npainterestRate").value);

    var npainterestProvOn = npainterestProvAfter - npainterestWrite
        + npainterestAddition - npainterestReduction;
    document.getElementById("npainterestProvOn").value = npainterestProvOn
        .toFixed(2);

    var npainterestProvReq = (npainterestProvOn * npainterestRate) / 100;
    document.getElementById("npainterestProvReq").value = npainterestProvReq
        .toFixed(2);

    //to calculate fraudsDebitedProvReq
    debtCalculate();
}

$(function () {
    $("#example1").DataTable({
        "paging": false,
        "lengthChange": true,
        "searching": false,
        "ordering": false,
        "info": false,
        "autoWidth": true
    });
    $('#example2').DataTable({
        "paging": false,
        "lengthChange": true,
        "searching": true,
        "ordering": false,
        "info": false,
        "autoWidth": true
    });
});

$(".select2").select2();

function delRow() {
    var formName = window.document.forms[0];

    var table = document.getElementById('example1');
    var movableflag = false;
    var checkval1 = $('.recordSelector:checkbox:checked').length;

    // alert("checkval"+checkval1);

    for (var n1 = 0; n1 < checkval1; n1++) {
        movableflag = false;
        var i2 = table.rows.length - 1;
        // alert("i2"+i2);
        for (var j1 = 0; j1 <= i2; j1++) {
            // alert("j1"+j1);
            if (i2 == 1) {
                // alert("i2 zero");
                // alert("j1 checked"+j1);
                if (formName.selRadio[18].checked == true) {

                    var x = i2;
                    // alert("deleting "+x);
                    document.getElementById('example1').deleteRow(x);
                    i2--;
                    movableflag = true;

                    if (movableflag) {
                        break;
                    }
                }
            }

            else if (i2 > 1) {
                // alert("j1 checked"+j1);
                if (formName.selRadio[j1].checked == true) {

                    for (var k = j1 + 1; k < i2; k++) {

                        var k1 = k - 1;
                        formName.selRadio[k1].checked = formName.selRadio[k].checked;
                        formName.particularsList[k1 - 12].value = formName.particularsList[k - 12].value;
                        formName.provAmt2015List[k1 - 12].value = formName.provAmt2015List[k - 12].value;
                        formName.writeOffDur12monList[k1 - 12].value = formName.writeOffDur12monList[k - 12].value;
                        formName.additionDur12monList[k1 - 12].value = formName.additionDur12monList[k - 12].value;
                        formName.reduInProviAmtList[k1 - 12].value = formName.reduInProviAmtList[k - 12].value;
                        formName.proviAmt2016List[k1 - 12].value = formName.proviAmt2016List[k - 12].value;
                        formName.ratePOfProvList[k1 - 12].value = formName.ratePOfProvList[k - 12].value;
                        formName.provReqList[k1 - 12].value = formName.provReqList[k - 12].value;

                    }
                    var x = i2;
                    // alert("deleting "+x);
                    document.getElementById('example1').deleteRow(x);
                    i2--;
                    movableflag = true;

                    if (movableflag) {
                        break;
                    }

                }

            }
        }
    }
}

function addRow() {

    var serialNo = 1;

    var table = document.getElementById('example1');
    var tabLength = table.rows.length;
    var count = tabLength;
    console.log('counttt>>>>::'+count);
    var radioCounter = count;
    // alert(count);

    var valid = true;

    if (count >= 13) {
        for (let i = 13; i < count; i++) {
            // if(formName.particularsList[i].value=="")
            //console.log("value>>>> ::zaa  " +document.getElementById("particularsList" + i).value);

            if (document.getElementById("particularsList" + i).value === "" || document.getElementById("particularsList" + i).value === null) {
                 var errorMsg="Please Enter Particulars";
                $('#errorModal .modal-body').text(errorMsg);
                $('#errorModal').modal('show');
                // formName.warning.focus();
                valid = false;
            }

            // if(formName.particularsList[i].value!="")
            if (document.getElementById("particularsList" + i).value !== "") {
                // if(formName.proviAmt2016List[i].value==0.00)
                if (document.getElementById("proviAmt2016List" + i).value == ""
                    || document.getElementById("proviAmt2016List" + i).value == "0.00") {
                    var errorMsg="Please Enter the Amount for Other Particulars";
                    $('#errorModal .modal-body').text(errorMsg);
                    $('#errorModal').modal('show');
                    // formName.warning.focus();
                    valid = false;
                }
            }

        }
    }

    if (valid == true) {

        var table = document.getElementById('example1');
        var tabLength = table.rows.length;
        var rowId = tabLength;
        var radioId = 'warningID' + radioCounter;
        serialNo = tabLength;
        var row = table.insertRow('-1');
        var radioCounterName = "'" + radioCounter + "'";

        var cell1 = row.insertCell(0);
        // cell1.style.width = '10%';
        cell1.style.width = '3%';
        cell1.style.align = 'center';
        cell1.className = 'tdstyle';
        // cell1.appendChild(addRadioButton(radioCounter));
        cell1.innerHTML = '<input type="checkbox" name="selRadio" class="recordSelector" />';
        var cell2 = row.insertCell(1);
        // cell2.style.width = '15%';
        cell2.style.width = '18%';
        cell2.style.align = 'center';
        cell2.className = 'tdstyle';
        var idname01 = "'particularsList" + radioCounter + "'";
        var idname0 = "particularsList" + radioCounter;
        cell2.nowrap = false;
        cell2.innerHTML = '<input type="text" maxlength="50" size="20" id="'
            + idname0
            + '" style="text-align:left" name="particularsList" class="form-control textbox" onblur="nameValidation('+idname01+');  specialCh1(this)"/>';
        var cell3 = row.insertCell(2);
        // cell3.style.width = '15%';
        cell3.style.width = '13%';
        cell3.style.align = 'center';
        cell3.className = 'tdstyle';
        var idname1 = "'provAmt2015List" + radioCounter + "'";
        var idname = "provAmt2015List" + radioCounter;
        cell3.innerHTML = '<input type="text" size="20" maxlength="15" id="'
            + idname
            + '" style="text-align:right" name="provAmt2015List" class="form-control textbox" onblur=" calculateDynamicTotal('
            + radioCounterName + ');checkNegative(' + idname1 + ');" />';
        cell3.nowrap = false;

        var cell4 = row.insertCell(3);
        // cell4.style.width = '20%';
        cell4.style.width = '12%';
        cell4.style.align = 'center';
        cell4.className = 'tdstyle';
        var idname2 = "'writeOffDur12monList" + radioCounter + "'";
        var idname1 = "writeOffDur12monList" + radioCounter;
        cell4.innerHTML = '<input type="text" size="20" maxlength="15" id="'
            + idname1
            + '" style="text-align:right" name="writeOffDur12monList"  class="form-control textbox" onblur=" calculateDynamicTotal('
            + radioCounterName + ');checkNegative(' + idname2 + ');"  />';
        cell4.nowrap = false;
        // alert(cell4.innerText);
        cell4.nowrap = false;
        var cell5 = row.insertCell(4);
        // cell5.style.display = 'none';
        cell5.style.width = '12%';
        cell5.style.align = 'left';
        cell5.className = 'tdstyle';
        var idname3 = "'additionDur12monList" + radioCounter + "'";
        var idname2 = "additionDur12monList" + radioCounter;
        cell5.innerHTML = '<input type="text" size="20" maxlength="15" id="'
            + idname2
            + '"  style="text-align:right" name="additionDur12monList" class="form-control textbox" onblur=" calculateDynamicTotal('
            + radioCounterName + ');checkNegative(' + idname3 + ');"  />';
        cell5.nowrap = false;
        var cell6 = row.insertCell(5);
        cell6.style.width = '12%';
        cell6.style.align = 'center';
        cell6.className = 'tdstyle';
        var idname4 = "'reduInProviAmtList" + radioCounter + "'";
        var idname3 = "reduInProviAmtList" + radioCounter;
        cell6.innerHTML = '<input type="text" size="20" maxlength="15" id="'
            + idname3
            + '"  style="text-align:right" name="reduInProviAmtList"  class="form-control textbox" onblur=" calculateDynamicTotal('
            + radioCounterName + ');checkNegative(' + idname4 + ');"/>';
        cell6.nowrap = false;
        var cell7 = row.insertCell(6);
        // cell7.style.display = 'none';
        cell7.style.width = '12%';
        cell7.style.align = 'center';
        cell7.className = 'tdstyle';
        var idname5 = "'proviAmt2016List" + radioCounter + "'";
        var idname4 = "proviAmt2016List" + radioCounter;
        cell7.innerHTML = '<input type="text" size="20" maxlength="15" id="'
            + idname4
            + '" style="text-align:right" name="proviAmt2016List" class="form-control textbox" onblur=" calculateDynamicTotal('
            + radioCounterName + ')" readonly="true"/>';
        cell7.nowrap = false;

        var cell8 = row.insertCell(7);
        // cell7.style.display = 'none';
        cell8.style.width = '6%';
        cell8.style.align = 'center';
        cell8.className = 'tdstyle';
        var idname6 = "'ratePOfProvList" + radioCounter + "'";
        var idname5 = "ratePOfProvList" + radioCounter;
        cell8.innerHTML = '<input type="text" size="20" maxlength="15" id="'
            + idname5
            + '"  style="text-align:right" name="ratePOfProvList" class="form-control textbox" onblur=" calculateDynamicTotal('
            + radioCounterName + ')"  value="100" readonly="true"/>';
        cell8.nowrap = false;

        var cell9 = row.insertCell(8);
        // cell7.style.display = 'none';
        cell9.style.width = '12%';
        cell9.style.align = 'center';
        cell9.className = 'tdstyle';
        var idname8 = "'provReqList" + radioCounter + "'";
        var idname7 = "provReqList" + radioCounter;
        cell9.innerHTML = '<input type="text" size="20" maxlength="15" id="'
            + idname7
            + '"   style="text-align:right" name="provReqList" class="form-control textbox" onblur=" calculateDynamicTotal('
            + radioCounterName + ')" readonly="true" />';
        cell9.nowrap = false;

        radioCounter++;
    }
}

$(".select2").select2();

 var validTotalOne = (sc10
                        .parseFloat(sc10.row.totalA29) + sc10
                        .parseFloat(sc10.row.totalB29))
                        .toFixed(2);
                    var validTotalTwo = sc10.row.premisesUnderCons29;
                    var validTotalThree = sc10.row.totalC29;

                    var totlA29 = document
                        .getElementById("sc10.row.totalA29");
                    totlA29.style.border = "0px solid white";

                    var totlB29 = document
                        .getElementById("sc10.row.totalB29");
                    totlB29.style.border = "0px solid white";
                    var totlC29 = document
                        .getElementById("sc10.row.totalC29");
                    totlC29.style.border = "0px solid white";
                    var preUnderCon29 = document
                        .getElementById("sc10.row.premisesUnderCons29");
                    preUnderCon29.style.border = "0px solid white";

                    if (parseFloat(sc10.row.validPremisesAmount) != parseFloat(validTotalThree)) {
                        var totlC29 = document
                            .getElementById("sc10.row.totalC29");
                        totlC29.style.border = "1px solid red";
                    }
                    if (parseFloat(sc10.row.validPremisesUnderConsAmount) != parseFloat(validTotalTwo)) {

                        var preUnderCon29 = document
                            .getElementById("sc10.row.premisesUnderCons29");
                        preUnderCon29.style.border = "1px solid red";

                    }
                    if (parseFloat(sc10.row.validOtherFixedAssetAmount) != parseFloat(validTotalOne)) {
                        var totlA29 = document
                            .getElementById("sc10.row.totalA29");
                        totlA29.style.border = "1px solid red";

                        var totlB29 = document
                            .getElementById("sc10.row.totalB29");
                        totlB29.style.border = "1px solid red";

                    }

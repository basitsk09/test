public String SubmitSC09Report(SC09 row) {



		String seq="";
		String result="";
		//log.info("circleCode" + row.getCircleCode());
		//log.info("mocPending "+row.getAreMocPending());
		//log.info("For saving data ? "+row.isSave());
		//log.info(" ************** submit SC09 ******************* user role "+row.getRole()+"  *** sc09 STATUS "+row.getStatus());

		TransactionDefinition def = new DefaultTransactionDefinition();
		TransactionStatus status = transactionManager.getTransaction(def);
        int flag = -1;
        String statusFlag="";
        boolean isInserted = false;

		if (row.getStatus() == null || row.getStatus().equalsIgnoreCase("")) {
			String query = "insert into BS_SC9(SC9_SEQ,SC9_CIRCLE,SC9_DATE,SC9_SR,SC9_CAT,SC9_DESC,SC9_HEAD,SC9_CYR1) values (?,?,to_date(?,'dd/mm/yyyy'),?,?,?,?,?)";
			//log.info("***************** inside insert sc09 *  inside insert *  inside insert *  inside insert *  inside insert");

			List <Object[]> sc09InsertList=new ArrayList<Object[]>();
			try {

				Object[] temp1 = { "1", row.getCircleCode(), row.getQuarterEndDate(), "1", "Standard",
						"[i] Bills Purchased and Discounted less bills rediscounted",
						"A-1. Facility Wise Classification", row.getFacility_Standard_1() };

				Object[] temp2 = { "2", row.getCircleCode(), row.getQuarterEndDate(), "1", "Sub-standard",
						"[i] Bills Purchased and Discounted less bills rediscounted",
						"A-1. Facility Wise Classification", row.getFacility_SubStandard_1() };

				Object[] temp3 = { "3", row.getCircleCode(), row.getQuarterEndDate(), "1", "Doubtful",
						"[i] Bills Purchased and Discounted less bills rediscounted",
						"A-1. Facility Wise Classification", row.getFacility_Doubtful_1() };

				Object[] temp4 = { "4", row.getCircleCode(), row.getQuarterEndDate(), "1", "Loss",
						"[i] Bills Purchased and Discounted less bills rediscounted",
						"A-1. Facility Wise Classification", row.getFacility_Loss_1() };


				Object[] temp5 = { "5", row.getCircleCode(), row.getQuarterEndDate(), "2", "Standard",
						"[ii] Cash Credits, Overdrafts, Loans repayable on Demand and Recalled Assets",
						"A-1. Facility Wise Classification", row.getFacility_Standard_2() };

				Object[] temp6 = { "6", row.getCircleCode(), row.getQuarterEndDate(), "2", "Sub-standard",
						"[ii] Cash Credits, Overdrafts, Loans repayable on Demand and Recalled Assets",
						"A-1. Facility Wise Classification", row.getFacility_SubStandard_2() };

				Object[] temp7 = { "7", row.getCircleCode(), row.getQuarterEndDate(), "2", "Doubtful",
						"[ii] Cash Credits, Overdrafts, Loans repayable on Demand and Recalled Assets",
						"A-1. Facility Wise Classification", row.getFacility_Doubtful_2() };

				Object[] temp8 = { "8", row.getCircleCode(), row.getQuarterEndDate(), "2", "Loss",
						"[ii] Cash Credits, Overdrafts, Loans repayable on Demand and Recalled Assets",
						"A-1. Facility Wise Classification", row.getFacility_Loss_2() };


				Object[] temp9 = { "9", row.getCircleCode(), row.getQuarterEndDate(), "3", "Standard",
						"[iii] Term Loans, Agricultural Term Loans, FCNRB Term Loan",
						"A-1. Facility Wise Classification", row.getFacility_Standard_3() };

				Object[] temp10 = { "10", row.getCircleCode(), row.getQuarterEndDate(), "3", "Sub-standard",
						"[iii] Term Loans, Agricultural Term Loans, FCNRB Term Loan",
						"A-1. Facility Wise Classification", row.getFacility_SubStandard_3() };

				Object[] temp11 = { "11", row.getCircleCode(), row.getQuarterEndDate(), "3", "Doubtful",
						"[iii] Term Loans, Agricultural Term Loans, FCNRB Term Loan",
						"A-1. Facility Wise Classification", row.getFacility_Doubtful_3() };

				Object[] temp12 = { "12", row.getCircleCode(), row.getQuarterEndDate(), "3", "Loss",
						"[iii] Term Loans, Agricultural Term Loans, FCNRB Term Loan",
						"A-1. Facility Wise Classification", row.getFacility_Loss_3() };

				//// sec
				


				Object[] temp13 = { "13", row.getCircleCode(), row.getQuarterEndDate(), "1", "Standard",
						"[i] Secured by Tangible Assets", "A. 2 Security Wise Classifications",
						row.getSecurity_Standard_1() };

				Object[] temp14 = { "14", row.getCircleCode(), row.getQuarterEndDate(), "1", "Sub-standard",
						"[i] Secured by Tangible Assets", "A. 2 Security Wise Classifications",
						row.getSecurity_SubStandard_1() };

				Object[] temp15 = { "15", row.getCircleCode(), row.getQuarterEndDate(), "1", "Doubtful",
						"[i] Secured by Tangible Assets", "A. 2 Security Wise Classifications",
						row.getSecurity_Doubtful_1() };

				Object[] temp16 = { "16", row.getCircleCode(), row.getQuarterEndDate(), "1", "Loss",
						"[i] Secured by Tangible Assets", "A. 2 Security Wise Classifications",
						row.getSecurity_Loss_1() };

				
				Object[] temp17 = { "17", row.getCircleCode(), row.getQuarterEndDate(), "2", "Standard",
						"[ii] Covered by Bank/ DICGC/ ECGC/ CGTSI/ Govt Guarantee",
						"A. 2 Security Wise Classifications", row.getSecurity_Standard_2() };

				Object[] temp18 = { "18", row.getCircleCode(), row.getQuarterEndDate(), "2", "Sub-standard",
						"[ii] Covered by Bank/ DICGC/ ECGC/ CGTSI/ Govt Guarantee",
						"A. 2 Security Wise Classifications", row.getSecurity_SubStandard_2() };

				Object[] temp19 = { "19", row.getCircleCode(), row.getQuarterEndDate(), "2", "Doubtful",
						"[ii] Covered by Bank/ DICGC/ ECGC/ CGTSI/ Govt Guarantee",
						"A. 2 Security Wise Classifications", row.getSecurity_Doubtful_2() };

				Object[] temp20 = { "20", row.getCircleCode(), row.getQuarterEndDate(), "2", "Loss",
						"[ii] Covered by Bank/ DICGC/ ECGC/ CGTSI/ Govt Guarantee",
						"A. 2 Security Wise Classifications", row.getSecurity_Loss_2() };

				

				Object[] temp21 = { "21", row.getCircleCode(), row.getQuarterEndDate(), "3", "Standard",
						"[iii] Unsecured", "A. 2 Security Wise Classifications", row.getSecurity_Standard_3() };

				Object[] temp22 = { "22", row.getCircleCode(), row.getQuarterEndDate(), "3", "Sub-standard",
						"[iii] Unsecured", "A. 2 Security Wise Classifications", row.getSecurity_SubStandard_3() };

				Object[] temp23 = { "23", row.getCircleCode(), row.getQuarterEndDate(), "3", "Doubtful",
						"[iii] Unsecured", "A. 2 Security Wise Classifications", row.getSecurity_Doubtful_3() };

				Object[] temp24 = { "24", row.getCircleCode(), row.getQuarterEndDate(), "3", "Loss", "[iii] Unsecured",
						"A. 2 Security Wise Classifications", row.getSecurity_Loss_3() };

				///// sector wise

				
				Object[] temp25 = { "25", row.getCircleCode(), row.getQuarterEndDate(), "0", "Standard","a)In India","A.3 Sector -Wise Classifications", "" };

				Object[] temp26 = { "26", row.getCircleCode(), row.getQuarterEndDate(), "0", "Sub-standard","a)In India","A.3 Sector -Wise Classifications", "" };

				Object[] temp27 = { "27", row.getCircleCode(), row.getQuarterEndDate(), "0", "Doubtful","a)In India","A.3 Sector -Wise Classifications", "" };

				Object[] temp28 = { "28", row.getCircleCode(), row.getQuarterEndDate(), "0", "Loss","a)In India","A.3 Sector -Wise Classifications", "" };
				
				


				Object[] temp29 = { "29", row.getCircleCode(), row.getQuarterEndDate(), "1", "Standard", "[i] Priority",
						"A.3 Sector -Wise Classifications", row.getSector_Standard_a1() };

				Object[] temp30 = { "30", row.getCircleCode(), row.getQuarterEndDate(), "1", "Sub-standard",
						"[i] Priority", "A.3 Sector -Wise Classifications", row.getSector_SubStandard_a1() };

				Object[] temp31 = { "31", row.getCircleCode(), row.getQuarterEndDate(), "1", "Doubtful", "[i] Priority",
						"A.3 Sector -Wise Classifications", row.getSector_Doubtful_a1() };

				Object[] temp32 = { "32", row.getCircleCode(), row.getQuarterEndDate(), "1", "Loss", "[i] Priority",
						"A.3 Sector -Wise Classifications", row.getSector_Loss_a1() };
				

				
				Object[] temp33 = { "33", row.getCircleCode(), row.getQuarterEndDate(), "2", "Standard", "[ii] Public",
						"A.3 Sector -Wise Classifications", row.getSector_Standard_a2() };

				Object[] temp34 = { "34", row.getCircleCode(), row.getQuarterEndDate(), "2", "Sub-standard",
						"[ii] Public", "A.3 Sector -Wise Classifications", row.getSector_SubStandard_a2() };

				Object[] temp35 = { "35", row.getCircleCode(), row.getQuarterEndDate(), "2", "Doubtful", "[ii] Public",
						"A.3 Sector -Wise Classifications", row.getSector_Doubtful_a2() };

				Object[] temp36 = { "36", row.getCircleCode(), row.getQuarterEndDate(), "2", "Loss", "[ii] Public",
						"A.3 Sector -Wise Classifications", row.getSector_Loss_a2() };
				
				
				

				Object[] temp37 = { "37", row.getCircleCode(), row.getQuarterEndDate(), "3", "Standard",
						"[iii] Banks in India", "A.3 Sector -Wise Classifications", row.getSector_Standard_a3() };

				Object[] temp38 = { "38", row.getCircleCode(), row.getQuarterEndDate(), "3", "Sub-standard",
						"[iii] Banks in India", "A.3 Sector -Wise Classifications", row.getSector_SubStandard_a3() };

				Object[] temp39 = { "39", row.getCircleCode(), row.getQuarterEndDate(), "3", "Doubtful",
						"[iii] Banks in India", "A.3 Sector -Wise Classifications", row.getSector_Doubtful_a3() };

				Object[] temp40 = { "40", row.getCircleCode(), row.getQuarterEndDate(), "3", "Loss",
						"[iii] Banks in India", "A.3 Sector -Wise Classifications", row.getSector_Loss_a3() };
				
				
				
				
				Object[] temp41 = { "41", row.getCircleCode(), row.getQuarterEndDate(), "4", "Standard", "[iv] Others",
						"A.3 Sector -Wise Classifications", row.getSector_Standard_a4() };

				Object[] temp42 = { "42", row.getCircleCode(), row.getQuarterEndDate(), "4", "Sub-standard",
						"[iv] Others", "A.3 Sector -Wise Classifications", row.getSector_SubStandard_a4() };

				Object[] temp43 = { "43", row.getCircleCode(), row.getQuarterEndDate(), "4", "Doubtful", "[iv] Others",
						"A.3 Sector -Wise Classifications", row.getSector_Doubtful_a4() };

				Object[] temp44 = { "44", row.getCircleCode(), row.getQuarterEndDate(), "4", "Loss", "[iv] Others",
						"A.3 Sector -Wise Classifications", row.getSector_Loss_a4(), };
				
				

				
				Object[] temp45 = { "45", row.getCircleCode(), row.getQuarterEndDate(), "5", "Standard",
						"TOTAL IN INDIA (i+ii+iii+iv)", "A.3 Sector -Wise Classifications", row.getSector_Standard_a_Total() };

				Object[] temp46 = { "46", row.getCircleCode(), row.getQuarterEndDate(), "5", "Sub-standard",
						"TOTAL IN INDIA (i+ii+iii+iv)", "A.3 Sector -Wise Classifications", row.getSector_SubStandard_a_Total() };

				Object[] temp47 = { "47", row.getCircleCode(), row.getQuarterEndDate(), "5", "Doubtful",
						"TOTAL IN INDIA (i+ii+iii+iv)", "A.3 Sector -Wise Classifications", row.getSector_Doubtful_a_Total() };

				Object[] temp48 = { "48", row.getCircleCode(), row.getQuarterEndDate(), "5", "Loss",
						"TOTAL IN INDIA (i+ii+iii+iv)", "A.3 Sector -Wise Classifications", row.getSector_Loss_a_Total() };
				


				Object[] temp49 = { "49", row.getCircleCode(), row.getQuarterEndDate(), "6", "Standard",
						"b) Outside India ( Excluding Foreign LCs and BGs)", "A.3 Sector -Wise Classifications", "" };

				Object[] temp50 = { "50", row.getCircleCode(), row.getQuarterEndDate(), "6", "Sub-standard",
						"b) Outside India ( Excluding Foreign LCs and BGs)", "A.3 Sector -Wise Classifications", "" };

				Object[] temp51 = { "51", row.getCircleCode(), row.getQuarterEndDate(), "6", "Doubtful",
						"b) Outside India ( Excluding Foreign LCs and BGs)", "A.3 Sector -Wise Classifications", "" };

				Object[] temp52 = { "52", row.getCircleCode(), row.getQuarterEndDate(), "6", "Loss",
						"b) Outside India ( Excluding Foreign LCs and BGs)", "A.3 Sector -Wise Classifications", "" };



				
				Object[] temp53 = { "53", row.getCircleCode(), row.getQuarterEndDate(), "7", "Standard",
						"[i] Due from Banks", "A.3 Sector -Wise Classifications", row.getSector_Standard_b1() };

				Object[] temp54 = { "54", row.getCircleCode(), row.getQuarterEndDate(), "7", "Sub-standard",
						"[i] Due from Banks", "A.3 Sector -Wise Classifications", row.getSector_SubStandard_b1() };

				Object[] temp55 = { "55", row.getCircleCode(), row.getQuarterEndDate(), "7", "Doubtful",
						"[i] Due from Banks", "A.3 Sector -Wise Classifications", row.getSector_Doubtful_b1() };
		
				Object[] temp56 = { "56", row.getCircleCode(), row.getQuarterEndDate(), "7", "Loss",
						"[i] Due from Banks", "A.3 Sector -Wise Classifications", row.getSector_Loss_b1() };
				
				
				

				Object[] temp57 = { "57", row.getCircleCode(), row.getQuarterEndDate(), "8", "Standard",
						"[ii] Due from Others", "A.3 Sector -Wise Classifications",
						"" };

				Object[] temp58 = { "58", row.getCircleCode(), row.getQuarterEndDate(), "8", "Sub-standard",
						"[ii] Due from Others", "A.3 Sector -Wise Classifications",
						"" };

				Object[] temp59 = { "59", row.getCircleCode(), row.getQuarterEndDate(), "8", "Doubtful",
						"[ii] Due from Others", "A.3 Sector -Wise Classifications",
						"" };
				
				Object[] temp60 = { "60", row.getCircleCode(), row.getQuarterEndDate(), "8", "Loss",
						"[ii] Due from Others","A.3 Sector -Wise Classifications", "" };

				
				

				  Object[] temp61 = { "61", row.getCircleCode(), row.getQuarterEndDate(), "9", "Standard",
							"[1] Bills Purchased and Discounted", "A.3 Sector -Wise Classifications", row.getSector_Standard_1() };

					Object[] temp62 = { "62", row.getCircleCode(), row.getQuarterEndDate(), "9", "Sub-standard",
							"[1] Bills Purchased and Discounted", "A.3 Sector -Wise Classifications", row.getSector_SubStandard_1(), };

					Object[] temp63 = { "63", row.getCircleCode(), row.getQuarterEndDate(), "9", "Doubtful",
							"[1] Bills Purchased and Discounted", "A.3 Sector -Wise Classifications", row.getSector_Doubtful_1() };
					
					
					Object[] temp64 = { "64", row.getCircleCode(), row.getQuarterEndDate(), "9", "Loss",
							"[1] Bills Purchased and Discounted", "A.3 Sector -Wise Classifications", row.getSector_Loss_1() };
				
					
					
				
				Object[] temp65 = { "65", row.getCircleCode(), row.getQuarterEndDate(), "10", "Standard",
						"[2] Syndicated Loans", "A.3 Sector -Wise Classifications", row.getSector_Standard_2() };

				Object[] temp66 = { "66", row.getCircleCode(), row.getQuarterEndDate(), "10", "Sub-standard",
						"[2] Syndicated Loans", "A.3 Sector -Wise Classifications", row.getSector_SubStandard_2(), };

				Object[] temp67 = { "67", row.getCircleCode(), row.getQuarterEndDate(), "10", "Doubtful",
						"[2] Syndicated Loans", "A.3 Sector -Wise Classifications", row.getSector_Doubtful_2() };
				
				
				Object[] temp68 = { "68", row.getCircleCode(), row.getQuarterEndDate(), "10", "Loss",
						"[2] Syndicated Loans", "A.3 Sector -Wise Classifications", row.getSector_Loss_2() };
				
				
								
				
				Object[] temp69 = { "69", row.getCircleCode(), row.getQuarterEndDate(), "11", "Standard", "[3] Others",
						"A.3 Sector -Wise Classifications", row.getSector_Standard_3() };
				
				Object[] temp70 = { "70", row.getCircleCode(), row.getQuarterEndDate(), "11", "Sub-standard",
						"[3] Others", "A.3 Sector -Wise Classifications", row.getSector_SubStandard_3() };

				Object[] temp71 = { "71", row.getCircleCode(), row.getQuarterEndDate(), "11", "Doubtful", "[3] Others",
						"A.3 Sector -Wise Classifications", row.getSector_Doubtful_3() };

				Object[] temp72 = { "72", row.getCircleCode(), row.getQuarterEndDate(), "11", "Loss", "[3] Others",
						"A.3 Sector -Wise Classifications", row.getSector_Loss_3() };
				
				
				

				Object[] temp73 = { "73", row.getCircleCode(), row.getQuarterEndDate(), "12", "Standard", "TOTAL IN OUTSIDE INDIA (i+ ii.1+ii.2+ii.3)",
						"A.3 Sector -Wise Classifications", row.getSector_Standard_b_Total() };
				
				Object[] temp74 = { "74", row.getCircleCode(), row.getQuarterEndDate(), "12", "Sub-standard","TOTAL IN OUTSIDE INDIA (i+ ii.1+ii.2+ii.3)", 
						"A.3 Sector -Wise Classifications", row.getSector_SubStandard_b_Total() };

				Object[] temp75 = { "75", row.getCircleCode(), row.getQuarterEndDate(), "12", "Doubtful", "TOTAL IN OUTSIDE INDIA (i+ ii.1+ii.2+ii.3)",
						"A.3 Sector -Wise Classifications", row.getSector_Doubtful_b_Total() };

				Object[] temp76 = { "76", row.getCircleCode(), row.getQuarterEndDate(), "12", "Loss", "TOTAL IN OUTSIDE INDIA (i+ ii.1+ii.2+ii.3)",
						"A.3 Sector -Wise Classifications", row.getSector_Loss_b_Total() };
				

				
				sc09InsertList.add(temp1);
				sc09InsertList.add(temp2);
				sc09InsertList.add(temp3);
				sc09InsertList.add(temp4);
				sc09InsertList.add(temp5);
				sc09InsertList.add(temp6);
				sc09InsertList.add(temp7);
				sc09InsertList.add(temp8);
				sc09InsertList.add(temp9);
				sc09InsertList.add(temp10);
				sc09InsertList.add(temp11);
				sc09InsertList.add(temp12);
				sc09InsertList.add(temp13);
				sc09InsertList.add(temp14);
				sc09InsertList.add(temp15);
				sc09InsertList.add(temp16);
				sc09InsertList.add(temp17);
				sc09InsertList.add(temp18);
				sc09InsertList.add(temp19);
				sc09InsertList.add(temp20);
				sc09InsertList.add(temp21);
				sc09InsertList.add(temp22);
				sc09InsertList.add(temp23);
				sc09InsertList.add(temp24);
				sc09InsertList.add(temp25);
				sc09InsertList.add(temp26);
				sc09InsertList.add(temp27);
				sc09InsertList.add(temp28);
				sc09InsertList.add(temp29);
				sc09InsertList.add(temp30);
				sc09InsertList.add(temp31);
				sc09InsertList.add(temp32);
				sc09InsertList.add(temp33);
				sc09InsertList.add(temp34);
				sc09InsertList.add(temp35);
				sc09InsertList.add(temp36);
				sc09InsertList.add(temp37);
				sc09InsertList.add(temp38);
				sc09InsertList.add(temp39);
				sc09InsertList.add(temp40);
				sc09InsertList.add(temp41);
				sc09InsertList.add(temp42);
				sc09InsertList.add(temp43);
				sc09InsertList.add(temp44);
				sc09InsertList.add(temp45);
				sc09InsertList.add(temp46);
				sc09InsertList.add(temp47);
				sc09InsertList.add(temp48);
				sc09InsertList.add(temp49);
				sc09InsertList.add(temp50);
				sc09InsertList.add(temp51);
				sc09InsertList.add(temp52);
				sc09InsertList.add(temp53);
				sc09InsertList.add(temp54);
				sc09InsertList.add(temp55);
				sc09InsertList.add(temp56);
				sc09InsertList.add(temp57);
				sc09InsertList.add(temp58);
				sc09InsertList.add(temp59);
				sc09InsertList.add(temp60);
				sc09InsertList.add(temp61);
				sc09InsertList.add(temp62);
				sc09InsertList.add(temp63);
				sc09InsertList.add(temp64);
				sc09InsertList.add(temp65);
				sc09InsertList.add(temp66);
				sc09InsertList.add(temp67);
				sc09InsertList.add(temp68);
				sc09InsertList.add(temp69);
				sc09InsertList.add(temp70);
				sc09InsertList.add(temp71);
				sc09InsertList.add(temp72);
				sc09InsertList.add(temp73);
				sc09InsertList.add(temp74);
				sc09InsertList.add(temp75);
				sc09InsertList.add(temp76);
				////Adjustment column is present for SC09 at FRT level
				if (row.getRole().equalsIgnoreCase("61")){

					//log.info("at FRT level  inserting Adjustment column data for SC09");
					Object[] temp77 = { "77", row.getCircleCode(), row.getQuarterEndDate(), "1", "Adjustment",
							"[i] Bills Purchased and Discounted less bills rediscounted",
							"A-1. Facility Wise Classification", row.getFacility_Adj_1() };

					Object[] temp78 = { "78", row.getCircleCode(), row.getQuarterEndDate(), "2", "Adjustment",
							"[ii] Cash Credits, Overdrafts, Loans repayable on Demand and Recalled Assets",
							"A-1. Facility Wise Classification", row.getFacility_Adj_2() };

					Object[] temp79 = { "79", row.getCircleCode(), row.getQuarterEndDate(), "3", "Adjustment",
							"[iii] Term Loans, Agricultural Term Loans, FCNRB Term Loan",
							"A-1. Facility Wise Classification", row.getFacility_Adj_3() };

					Object[] temp80 = { "80", row.getCircleCode(), row.getQuarterEndDate(), "1", "Adjustment",
							"[i] Secured by Tangible Assets", "A. 2 Security Wise Classifications",
							row.getSecurity_Adj_1() };

					Object[] temp81 = { "81", row.getCircleCode(), row.getQuarterEndDate(), "2", "Adjustment",
							"[ii] Covered by Bank/ DICGC/ ECGC/ CGTSI/ Govt Guarantee",
							"A. 2 Security Wise Classifications", row.getSecurity_Adj_2() };

					Object[] temp82 = { "82", row.getCircleCode(), row.getQuarterEndDate(), "3", "Adjustment", "[iii] Unsecured",
							"A. 2 Security Wise Classifications", row.getSecurity_Adj_3() };

					Object[] temp83 = { "83", row.getCircleCode(), row.getQuarterEndDate(), "0", "Adjustment","a)In India","A.3 Sector -Wise Classifications", "" };

					Object[] temp84 = { "84", row.getCircleCode(), row.getQuarterEndDate(), "1", "Adjustment", "[i] Priority",
							"A.3 Sector -Wise Classifications", row.getSector_Adj_a1() };

					Object[] temp85 = { "85", row.getCircleCode(), row.getQuarterEndDate(), "2", "Adjustment", "[ii] Public",
							"A.3 Sector -Wise Classifications", row.getSector_Adj_a2() };

					Object[] temp86 = { "86", row.getCircleCode(), row.getQuarterEndDate(), "3", "Adjustment",
							"[iii] Banks in India", "A.3 Sector -Wise Classifications", row.getSector_Adj_a3() };


					Object[] temp87 = { "87", row.getCircleCode(), row.getQuarterEndDate(), "4", "Adjustment", "[iv] Others",
							"A.3 Sector -Wise Classifications", row.getSector_Adj_a4(), };

					Object[] temp88 = { "88", row.getCircleCode(), row.getQuarterEndDate(), "5", "Adjustment",
							"TOTAL IN INDIA (i+ii+iii+iv)", "A.3 Sector -Wise Classifications", row.getSector_Adj_a_Total() };


					Object[] temp89 = { "89", row.getCircleCode(), row.getQuarterEndDate(), "6", "Adjustment",
							"b) Outside India ( Excluding Foreign LCs and BGs)", "A.3 Sector -Wise Classifications", "" };


					Object[] temp90 = { "90", row.getCircleCode(), row.getQuarterEndDate(), "7", "Adjustment",
							"[i] Due from Banks", "A.3 Sector -Wise Classifications", row.getSector_Adj_b1() };

					Object[] temp91 = { "91", row.getCircleCode(), row.getQuarterEndDate(), "8", "Adjustment",
							"[ii] Due from Others","A.3 Sector -Wise Classifications", "" };


					Object[] temp92 = { "92", row.getCircleCode(), row.getQuarterEndDate(), "9", "Adjustment",
							"[1] Bills Purchased and Discounted", "A.3 Sector -Wise Classifications", row.getSector_Adj_1() };

					Object[] temp93 = { "93", row.getCircleCode(), row.getQuarterEndDate(), "10", "Adjustment",
							"[2] Syndicated Loans", "A.3 Sector -Wise Classifications", row.getSector_Adj_2() };

					Object[] temp94 = { "94", row.getCircleCode(), row.getQuarterEndDate(), "11", "Adjustment", "[3] Others",
							"A.3 Sector -Wise Classifications", row.getSector_Adj_3() };


					Object[] temp95 = { "95", row.getCircleCode(), row.getQuarterEndDate(), "12", "Adjustment", "TOTAL IN OUTSIDE INDIA (i+ ii.1+ii.2+ii.3)",
							"A.3 Sector -Wise Classifications", row.getSector_Adj_b_Total() };


					sc09InsertList.add(temp77);
					sc09InsertList.add(temp78);
					sc09InsertList.add(temp79);
					sc09InsertList.add(temp80);
					sc09InsertList.add(temp81);
					sc09InsertList.add(temp82);
					sc09InsertList.add(temp83);
					sc09InsertList.add(temp84);
					sc09InsertList.add(temp85);
					sc09InsertList.add(temp86);
					sc09InsertList.add(temp87);
					sc09InsertList.add(temp88);
					sc09InsertList.add(temp89);
					sc09InsertList.add(temp90);
					sc09InsertList.add(temp91);
					sc09InsertList.add(temp92);
					sc09InsertList.add(temp93);
					sc09InsertList.add(temp94);
					sc09InsertList.add(temp95);

				}


				String statusInsert="20";
                if(("true").equals(row.getAreMocPending())  || row.isSave()==true){
                    statusInsert="11";
                }


				int check = -1;

				String sequence = makerDao.generateSequence();
				seq=sequence+"~"+statusInsert;
				if (check == -1){
					//log.info("******************************* inserting  SC 09 *** status as "+statusInsert);


					jdbcTemplate.batchUpdate(query,sc09InsertList);

					makerDao.reportEntryInMasterTable(sequence, row.getReportMasterId(), row.getCircleCode(), row.getQuarterEndDate(), row.getReportName(), row.getUserId(),statusInsert,"I");
					transactionManager.commit(status);
					check = 0;

				}

				flag = check;

				/*if(("true").equals(row.getAreMocPending())  || row.isSave()==true){
					makerDao.reportEntryInMasterTable(sequence, row.getReportMasterId(), row.getCircleCode(), row.getQuarterEndDate(), row.getReportName(), row.getUserId(),"11","I");
					flag = 0;
				}
				else{
                	makerDao.reportEntryInMasterTable(sequence, row.getReportMasterId(), row.getCircleCode(), row.getQuarterEndDate(), row.getReportName(), row.getUserId(),"20","I");
						flag = 0;
				}*/


			}

			catch (Exception sqle) {
				sqle.printStackTrace();
				transactionManager.rollback(status);
				flag = 2;
			}
		}

		else {
				
			List <Object[]> sc09UpdateList=new ArrayList<Object[]>();
			String query1 = "update BS_SC9 set BS_SC9.SC9_CYR1= ? where BS_SC9.SC9_CIRCLE= ?  and BS_SC9.SC9_DATE=to_date(?,'dd/mm/yyyy') and BS_SC9.SC9_SEQ= ?";

			//log.info("***************** inside update sc09 *  inside update *  inside update *  inside update *  inside update");
			try {
				
				
				Object [] temp1={row.getFacility_Standard_1(), row.getCircleCode(), row.getQuarterEndDate(), "1" };
				Object [] temp2={row.getFacility_SubStandard_1(), row.getCircleCode(),row.getQuarterEndDate(), "2" };
				Object [] temp3={row.getFacility_Doubtful_1(), row.getCircleCode(), row.getQuarterEndDate(), "3" };
				Object [] temp4={row.getFacility_Loss_1(), row.getCircleCode(), row.getQuarterEndDate(), "4" };
				
				sc09UpdateList.add(temp1);
				sc09UpdateList.add(temp2);
				sc09UpdateList.add(temp3);
				sc09UpdateList.add(temp4);

				
				Object [] temp5={  row.getFacility_Standard_2(), row.getCircleCode(), row.getQuarterEndDate(), "5" };
				Object [] temp6={  row.getFacility_SubStandard_2(), row.getCircleCode(),row.getQuarterEndDate(), "6" };
				Object [] temp7={  row.getFacility_Doubtful_2(), row.getCircleCode(), row.getQuarterEndDate(), "7" };
				Object [] temp8={  row.getFacility_Loss_2(), row.getCircleCode(), row.getQuarterEndDate(), "8" };
				
				sc09UpdateList.add(temp5);
				sc09UpdateList.add(temp6);
				sc09UpdateList.add(temp7);
				sc09UpdateList.add(temp8);
		
				Object [] temp9={  row.getFacility_Standard_3(), row.getCircleCode(), row.getQuarterEndDate(), "9" };
				Object [] temp10={  row.getFacility_SubStandard_3(), row.getCircleCode(),row.getQuarterEndDate(), "10" };
				Object [] temp11={  row.getFacility_Doubtful_3(), row.getCircleCode(), row.getQuarterEndDate(), "11" };
				Object [] temp12={  row.getFacility_Loss_3(), row.getCircleCode(), row.getQuarterEndDate(), "12" };
				
				sc09UpdateList.add(temp9);
				sc09UpdateList.add(temp10);
				sc09UpdateList.add(temp11);
				sc09UpdateList.add(temp12);
				

		//// sec
		
				Object [] temp13={  row.getSecurity_Standard_1(), row.getCircleCode(), row.getQuarterEndDate(), "13" };
				Object [] temp14={  row.getSecurity_SubStandard_1(), row.getCircleCode(),row.getQuarterEndDate(), "14" };
				Object [] temp15={  row.getSecurity_Doubtful_1(), row.getCircleCode(), row.getQuarterEndDate(), "15" };
				Object [] temp16={  row.getSecurity_Loss_1(), row.getCircleCode(), row.getQuarterEndDate(), "16" };
				
				
				sc09UpdateList.add(temp13);
				sc09UpdateList.add(temp14);
				sc09UpdateList.add(temp15);
				sc09UpdateList.add(temp16);
				
				
		
				Object [] temp17={  row.getSecurity_Standard_2(), row.getCircleCode(), row.getQuarterEndDate(), "17" };
				Object [] temp18={  row.getSecurity_SubStandard_2(), row.getCircleCode(),row.getQuarterEndDate(), "18" };
				Object [] temp19={  row.getSecurity_Doubtful_2(), row.getCircleCode(), row.getQuarterEndDate(), "19" };
				Object [] temp20={  row.getSecurity_Loss_2(), row.getCircleCode(), row.getQuarterEndDate(), "20" };
				
				
				sc09UpdateList.add(temp17);
				sc09UpdateList.add(temp18);
				sc09UpdateList.add(temp19);
				sc09UpdateList.add(temp20);
				

				Object [] temp21={  row.getSecurity_Standard_3(), row.getCircleCode(), row.getQuarterEndDate(), "21" };
				Object [] temp22={  row.getSecurity_SubStandard_3(), row.getCircleCode(),row.getQuarterEndDate(), "22" };
				Object [] temp23={  row.getSecurity_Doubtful_3(), row.getCircleCode(), row.getQuarterEndDate(), "23" };
				Object [] temp24={  row.getSecurity_Loss_3(), row.getCircleCode(), row.getQuarterEndDate(), "24" };
				

				sc09UpdateList.add(temp21);
				sc09UpdateList.add(temp22);
				sc09UpdateList.add(temp23);
				sc09UpdateList.add(temp24);

		///// sector wise

				Object [] temp25={  row.getSector_Standard_a1(), row.getCircleCode(), row.getQuarterEndDate(), "29" };
				Object [] temp26={  row.getSector_SubStandard_a1(), row.getCircleCode(),row.getQuarterEndDate(), "30" };
				Object [] temp27={  row.getSector_Doubtful_a1(), row.getCircleCode(), row.getQuarterEndDate(), "31" };
				Object [] temp28={  row.getSector_Loss_a1(), row.getCircleCode(), row.getQuarterEndDate(), "32" };
				
				
				sc09UpdateList.add(temp25);
				sc09UpdateList.add(temp26);
				sc09UpdateList.add(temp27);
				sc09UpdateList.add(temp28);
				

				Object [] temp29={  row.getSector_Standard_a2(), row.getCircleCode(), row.getQuarterEndDate(), "33" };
				Object [] temp30={  row.getSector_SubStandard_a2(), row.getCircleCode(),row.getQuarterEndDate(), "34" };
				Object [] temp31={  row.getSector_Doubtful_a2(), row.getCircleCode(), row.getQuarterEndDate(), "35" };
				Object [] temp32={  row.getSector_Loss_a2(), row.getCircleCode(), row.getQuarterEndDate(), "36" };
				

				
				sc09UpdateList.add(temp29);
				sc09UpdateList.add(temp30);
				sc09UpdateList.add(temp31);
				sc09UpdateList.add(temp32);
				

				Object [] temp33={  row.getSector_Standard_a3(), row.getCircleCode(), row.getQuarterEndDate(), "37" };
				Object [] temp34={  row.getSector_SubStandard_a3(), row.getCircleCode(),row.getQuarterEndDate(), "38" };
				Object [] temp35={  row.getSector_Doubtful_a3(), row.getCircleCode(), row.getQuarterEndDate(), "39" };
				Object [] temp36={  row.getSector_Loss_a3(), row.getCircleCode(), row.getQuarterEndDate(), "40" };
				
				sc09UpdateList.add(temp33);
				sc09UpdateList.add(temp34);
				sc09UpdateList.add(temp35);
				sc09UpdateList.add(temp36);
				
				

				Object [] temp37={  row.getSector_Standard_a4(), row.getCircleCode(), row.getQuarterEndDate(), "41" };
				Object [] temp38={  row.getSector_SubStandard_a4(), row.getCircleCode(),row.getQuarterEndDate(), "42" };
				Object [] temp39={  row.getSector_Doubtful_a4(), row.getCircleCode(), row.getQuarterEndDate(), "43" };
				Object [] temp40={  row.getSector_Loss_a4(), row.getCircleCode(), row.getQuarterEndDate(), "44" };
				
				sc09UpdateList.add(temp37);
				sc09UpdateList.add(temp38);
				sc09UpdateList.add(temp39);
				sc09UpdateList.add(temp40);
				

				
				Object [] temp111={  row.getSector_Standard_a_Total(), row.getCircleCode(), row.getQuarterEndDate(), "45" };
				Object [] temp222={  row.getSector_SubStandard_a_Total(), row.getCircleCode(),row.getQuarterEndDate(), "46" };
				Object [] temp333={  row.getSector_Doubtful_a_Total(), row.getCircleCode(), row.getQuarterEndDate(), "47" };
				Object [] temp444={  row.getSector_Loss_a_Total(), row.getCircleCode(), row.getQuarterEndDate(), "48" };
				
				sc09UpdateList.add(temp111);
				sc09UpdateList.add(temp222);
				sc09UpdateList.add(temp333);
				sc09UpdateList.add(temp444);
				


				
				Object [] temp41={  row.getSector_Standard_b1(), row.getCircleCode(), row.getQuarterEndDate(), "53" };
				Object [] temp42={  row.getSector_SubStandard_b1(), row.getCircleCode(),row.getQuarterEndDate(), "54" };
				Object [] temp43={  row.getSector_Doubtful_b1(), row.getCircleCode(), row.getQuarterEndDate(), "55" };
				Object [] temp44={  row.getSector_Loss_b1(), row.getCircleCode(), row.getQuarterEndDate(), "56" };
				
				
				sc09UpdateList.add(temp41);
				sc09UpdateList.add(temp42);
				sc09UpdateList.add(temp43);
				sc09UpdateList.add(temp44);

		

				


				Object [] temp49={  row.getSector_Standard_1(), row.getCircleCode(), row.getQuarterEndDate(), "61" };
				Object [] temp50={  row.getSector_SubStandard_1(), row.getCircleCode(), row.getQuarterEndDate(), "62" };
				Object [] temp51={  row.getSector_Doubtful_1(), row.getCircleCode(), row.getQuarterEndDate(), "63" };
				Object [] temp52={  row.getSector_Loss_1(), row.getCircleCode(), row.getQuarterEndDate(), "64" };
				
				
				
				sc09UpdateList.add(temp49);
				sc09UpdateList.add(temp50);
				sc09UpdateList.add(temp51);
				sc09UpdateList.add(temp52);
				
				
		
				Object [] temp53={  row.getSector_Standard_2(), row.getCircleCode(), row.getQuarterEndDate(), "65" };
				Object [] temp54={  row.getSector_SubStandard_2(), row.getCircleCode(), row.getQuarterEndDate(), "66" };
				Object [] temp55={  row.getSector_Doubtful_2(), row.getCircleCode(), row.getQuarterEndDate(), "67" };
				Object [] temp56={  row.getSector_Loss_2(), row.getCircleCode(), row.getQuarterEndDate(), "68" };
				
				
				
				sc09UpdateList.add(temp53);
				sc09UpdateList.add(temp54);
				sc09UpdateList.add(temp55);
				sc09UpdateList.add(temp56);
				
				
				
				Object [] temp57={  row.getSector_Standard_3(), row.getCircleCode(), row.getQuarterEndDate(), "69" };
				Object [] temp58={  row.getSector_SubStandard_3(), row.getCircleCode(), row.getQuarterEndDate(), "70" };
				Object [] temp59={  row.getSector_Doubtful_3(), row.getCircleCode(), row.getQuarterEndDate(), "71" };
				Object [] temp60={  row.getSector_Loss_3(), row.getCircleCode(), row.getQuarterEndDate(), "72" };

				
				sc09UpdateList.add(temp57);
				sc09UpdateList.add(temp58);
				sc09UpdateList.add(temp59);
				sc09UpdateList.add(temp60);
				
				
				

				Object [] temp555={  row.getSector_Standard_b_Total(), row.getCircleCode(), row.getQuarterEndDate(), "73" };
				Object [] temp666={  row.getSector_SubStandard_b_Total(), row.getCircleCode(), row.getQuarterEndDate(), "74" };
				Object [] temp777={  row.getSector_Doubtful_b_Total(), row.getCircleCode(), row.getQuarterEndDate(), "75" };
				Object [] temp888={  row.getSector_Loss_b_Total(), row.getCircleCode(), row.getQuarterEndDate(), "76" };

				
				sc09UpdateList.add(temp555);
				sc09UpdateList.add(temp666);
				sc09UpdateList.add(temp777);
				sc09UpdateList.add(temp888);


				////Adjustment column is present for SC09 at FRT level

				if (row.getRole().equalsIgnoreCase("61")){

					//log.info("at FRT level  updating Adjustment column data for SC09");

					Object [] temp77={row.getFacility_Adj_1(), row.getCircleCode(), row.getQuarterEndDate(), "77" };

					Object [] temp78={  row.getFacility_Adj_2(), row.getCircleCode(), row.getQuarterEndDate(), "78" };


					Object [] temp79={  row.getFacility_Adj_3(), row.getCircleCode(), row.getQuarterEndDate(), "79" };


					Object [] temp80={  row.getSecurity_Adj_1(), row.getCircleCode(), row.getQuarterEndDate(), "80" };



					Object [] temp81={  row.getSecurity_Adj_2(), row.getCircleCode(), row.getQuarterEndDate(), "81" };



					Object [] temp82={  row.getSecurity_Adj_3(), row.getCircleCode(), row.getQuarterEndDate(), "82" };



					Object [] temp84={  row.getSector_Adj_a1(), row.getCircleCode(), row.getQuarterEndDate(), "84" };



					Object [] temp85={  row.getSector_Adj_a2(), row.getCircleCode(), row.getQuarterEndDate(), "85" };


					Object [] temp86={  row.getSector_Adj_a3(), row.getCircleCode(), row.getQuarterEndDate(), "86" };


					Object [] temp87={  row.getSector_Adj_a4(), row.getCircleCode(), row.getQuarterEndDate(), "87" };


					Object [] temp88={  row.getSector_Adj_a_Total(), row.getCircleCode(), row.getQuarterEndDate(), "88" };


					Object [] temp90={  row.getSector_Adj_b1(), row.getCircleCode(), row.getQuarterEndDate(), "90" };



					Object [] temp92={  row.getSector_Adj_1(), row.getCircleCode(), row.getQuarterEndDate(), "92" };



					Object [] temp93={  row.getSector_Adj_2(), row.getCircleCode(), row.getQuarterEndDate(), "93" };




					Object [] temp94={  row.getSector_Adj_3(), row.getCircleCode(), row.getQuarterEndDate(), "94" };



					Object [] temp95={  row.getSector_Adj_b_Total(), row.getCircleCode(), row.getQuarterEndDate(), "95" };


					sc09UpdateList.add(temp77);
					sc09UpdateList.add(temp78);
					sc09UpdateList.add(temp79);
					sc09UpdateList.add(temp80);
					sc09UpdateList.add(temp81);
					sc09UpdateList.add(temp82);
					sc09UpdateList.add(temp84);
					sc09UpdateList.add(temp85);
					sc09UpdateList.add(temp86);
					sc09UpdateList.add(temp87);
					sc09UpdateList.add(temp88);
					sc09UpdateList.add(temp90);

					sc09UpdateList.add(temp92);
					sc09UpdateList.add(temp93);
					sc09UpdateList.add(temp94);
					sc09UpdateList.add(temp95);

				}



                String statusUpdate="21";
                if(("true").equals(row.getAreMocPending())  || row.isSave()==true){
                    statusUpdate="11";
                }


				int check = -1;
				seq=row.getReportId()+"~"+statusUpdate;
				if (check == -1){

					log.info("*******************************updating SC 09 *** status as "+statusUpdate+" and sequence is "+row.getReportId());
					jdbcTemplate.batchUpdate(query1,sc09UpdateList);
					makerDao.reportEntryInMasterTable(row.getReportId(), row.getReportMasterId(), row.getCircleCode(), row.getQuarterEndDate(), row.getReportName(), row.getUserId(),statusUpdate,"U");
					transactionManager.commit(status);
					check = 0;
				}

				flag = check;

				/*if(("true").equals(row.getAreMocPending()) || row.isSave()==true){
					makerDao.reportEntryInMasterTable(row.getReportId(), row.getReportMasterId(), row.getCircleCode(), row.getQuarterEndDate(), row.getReportName(), row.getUserId(),"11","U");
					flag = 0;
				}
				else {
                    makerDao.reportEntryInMasterTable(row.getReportId(), row.getReportMasterId(), row.getCircleCode(), row.getQuarterEndDate(), row.getReportName(), row.getUserId(),"21","U");
                    flag = 0;
				}*/

				
				}

			catch (Exception sqle) {
				sqle.printStackTrace();
				transactionManager.rollback(status);
				flag = 2;
			}

		}

		log.info("Existing SubmitSC09Report flag " + flag);
		result=flag+"~"+seq;
		log.info("############  SC09 submit result is "+result);

		return result;

	}

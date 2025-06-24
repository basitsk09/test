package com.tcs.dao;

import java.sql.ResultSet;
import java.sql.SQLException;
import java.util.ArrayList;
import javax.sql.DataSource;

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

import com.tcs.beans.RW04;
import com.tcs.services.MakerService;

@Repository("rw04Dao")
public class RW04DaoImpl implements RW04Dao {

	static Logger log = Logger.getLogger(RW04DaoImpl.class.getName());
	
	
	@Autowired
	private PlatformTransactionManager transactionManager;
	
	@Autowired
	DataSource dataSource;
	@Autowired
	private JdbcTemplate jdbcTemplate;
	@Autowired
	MakerDao makerDao;
	
	@Autowired
	MakerService makerService;

	public int insertAnxXADetails1(ArrayList<RW04> anxXAUploadReportBeanList, RW04 report, String branch_code,
			String user_Id, String quarter, String financial_year, String quarter_end_date, String circleCode,
			String report_master_id, String generatedSequence, String reportName,String reportId, String requestType) {

		TransactionDefinition def = new DefaultTransactionDefinition();
		TransactionStatus status = transactionManager.getTransaction(def);
		int flag = -1;		
		int ascii = 97;
		log.info("printed" + String.valueOf(Character.toChars(ascii)).toString());

		log.info("returned sequence " + generatedSequence);
		if (generatedSequence != "") {
			try {

				String query = "insert into CRS_OTH_ASSESTS (CRS_OTH_ASSESTS_ID,CRS_OTH_ASSESTS_NAME,OTH_ASTS_LST_YR,OTH_ASTS_WRITE_OFF,"
						+ "  OTH_ASTS_ADDITION, OTH_ASTS_REDUCTION,OTH_ASTS_CUR_YR,OTH_ASTS_PROVI_RATE,OTH_ASTS_PROVI_REQ, "
						+ " REPORT_MASTER_LIST_ID_FK,OTH_ASTS_BRANCH,OTH_ASTS_DATE,CRS_OTH_ASSESTS_SQ) "
						+ "values(ANXXA_SEQ.nextVal,?, ?,?,?,?,?,?,?,?,? ,to_date(?,'dd/mm/yyyy'),?)";
			
				log.info("anxXAUploadReportBeanListSize "+anxXAUploadReportBeanList.size());
				
				for (RW04 anxXAUploadReportBean1 : anxXAUploadReportBeanList) {

					
					log.info("ravi" + anxXAUploadReportBean1.getProvAmt2015());
					flag = jdbcTemplate.update(query, new Object[] {

							anxXAUploadReportBean1.getParticulars(), anxXAUploadReportBean1.getProvAmt2015(),
							anxXAUploadReportBean1.getWriteOffDur12mon(), anxXAUploadReportBean1.getAdditionDur12mon(),
							anxXAUploadReportBean1.getReduInProviAmt(), anxXAUploadReportBean1.getProviAmt2016(),
							anxXAUploadReportBean1.getRatePOfProv(), anxXAUploadReportBean1.getProvReq(),
							generatedSequence, branch_code, quarter_end_date, String.valueOf(Character.toChars(ascii))

					});
					ascii++;
					
					if (flag > 0) {
						flag = 0;
					}

				}

				String query1 = "insert into CRS_OTH_ASSESTS (CRS_OTH_ASSESTS_ID,CRS_OTH_ASSESTS_NAME,OTH_ASTS_LST_YR,OTH_ASTS_WRITE_OFF,"
						+ " OTH_ASTS_ADDITION, OTH_ASTS_REDUCTION,OTH_ASTS_CUR_YR,OTH_ASTS_PROVI_RATE,OTH_ASTS_PROVI_REQ, "
						+ " REPORT_MASTER_LIST_ID_FK,OTH_ASTS_BRANCH,OTH_ASTS_DATE,CRS_OTH_ASSESTS_SQ) "
						+ "values(?,?, ?,?,?,?,?,?,?,?,? ,to_date(?,'dd/mm/yyyy'),?)";

				flag = jdbcTemplate.update(query1,
						new Object[] { "1", report.getFirstRow(),
								report.getFraudsDebitedProvAfter(), report.getFraudsDebitedWrite(),
								report.getFraudsDebitedAddition(), report.getFraudsDebitedReduction(),
								report.getFraudsDebitedProvOn(), report.getFraudsDebitedRate(),
								report.getFraudsDebitedProvReq(), generatedSequence, branch_code, quarter_end_date,
								"1" });

				flag = jdbcTemplate.update(query1,
						new Object[] { "2", report.getSecondRow(),
								report.getFraudsDebitedPrior100ProvAfter(), report.getFraudsDebitedPrior100Write(),
								report.getFraudsDebitedPrior100Addition(), report.getFraudsDebitedPrior100Reduction(),
								report.getFraudsDebitedPrior100ProvOn(), report.getFraudsDebitedPrior100Rate(),
								report.getFraudsDebitedPrior100ProvReq(), generatedSequence, branch_code,
								quarter_end_date, "i." });

				flag = jdbcTemplate.update(query1,
						new Object[] { "6", report.getSixthRow(),
								report.getFraudsDebitedDelayedProvAfter(), report.getFraudsDebitedDelayedWrite(),
								report.getFraudsDebitedDelayedAddition(), report.getFraudsDebitedDelayedReduction(),
								report.getFraudsDebitedDelayedProvOn(), report.getFraudsDebitedDelayedRate(),
								report.getFraudsDebitedDelayedProvReq(), generatedSequence, branch_code,
								quarter_end_date, "v."

						});

				flag = jdbcTemplate.update(query1,
						new Object[] { "7", report.getSeventhRow(),
								report.getOthersRecalledProvAfter(), report.getOthersRecalledWrite(),
								report.getOthersRecalledAddition(), report.getOthersRecalledReduction(),
								report.getOthersRecalledProvOn(), report.getOthersRecalledRate(),
								report.getOthersRecalledProvReq(), generatedSequence, branch_code, quarter_end_date,
								"2" });

				flag = jdbcTemplate.update(query1,
						new Object[] { "8", report.getEightRow(), report.getFraudsOthersProvAfter(),
								report.getFraudsOthersWrite(), report.getFraudsOthersAddition(),
								report.getFraudsOthersReduction(), report.getFraudsOthersProvOn(),
								report.getFraudsOthersRate(), report.getFraudsOthersProvReq(), generatedSequence,
								branch_code, quarter_end_date, "3" });

				flag = jdbcTemplate.update(query1,
						new Object[] { "9", report.getNinethRow(),
								report.getFraudsOthersPrior100ProvAfter(), report.getFraudsOthersPrior100Write(),
								report.getFraudsOthersPrior100Addition(), report.getFraudsOthersPrior100Reduction(),
								report.getFraudsOthersPrior100ProvOn(), report.getFraudsOthersPrior100Rate(),
								report.getFraudsOthersPrior100ProvReq(), generatedSequence, branch_code,
								quarter_end_date, "i." });

				flag = jdbcTemplate.update(query1,
						new Object[] { "13", report.getThirteenthRow(),
								report.getFraudsOthersDelayedProvAfter(), report.getFraudsOthersDelayedWrite(),
								report.getFraudsOthersDelayedAddition(), report.getFraudsOthersDelayedReduction(),
								report.getFraudsOthersDelayedProvOn(), report.getFraudsOthersDelayedRate(),
								report.getFraudsOthersDelayedProvReq(), generatedSequence, branch_code,
								quarter_end_date, "v." });

				flag = jdbcTemplate.update(query1,
						new Object[] { "14", report.getFourteenthRow(), report.getRevenueProvAfter(),
								report.getRevenueWrite(), report.getRevenueAddition(), report.getRevenueReduction(),
								report.getRevenueProvOn(), report.getRevenueRate(), report.getRevenueProvReq(),
								generatedSequence, branch_code, quarter_end_date, "4" });

				flag = jdbcTemplate.update(query1,
						new Object[] { "15", report.getFifteenthRow(), report.getFsloProvAfter(),
								report.getFsloWrite(), report.getFsloAddition(), report.getFsloReduction(),
								report.getFsloProvOn(), report.getFsloRate(), report.getFsloProvReq(),
								generatedSequence, branch_code, quarter_end_date, "5" });

				flag = jdbcTemplate.update(query1, new Object[] { "16",
						report.getSixteenthRow(),
						report.getOutstandingProvAfter(), report.getOutstandingWrite(), report.getOutstandingAddition(),
						report.getOutstandingReduction(), report.getOutstandingProvOn(), report.getOutstandingRate(),
						report.getOutstandingProvReq(), generatedSequence, branch_code, quarter_end_date, "6" });

				flag = jdbcTemplate.update(query1, new Object[] { "17",
						report.getSeventeenthRow(),
						report.getNpainterestProvAfter(), report.getNpainterestWrite(), report.getNpainterestAddition(),
						report.getNpainterestReduction(), report.getNpainterestProvOn(), report.getNpainterestRate(),
						report.getNpainterestProvReq(), generatedSequence, branch_code, quarter_end_date, "7" });

				flag = jdbcTemplate.update(query1, new Object[] { "18", report.getEighteenRow(), "", "", "", "",
						"", "", "", generatedSequence, branch_code, quarter_end_date, "8" });

				if (flag > 0) {
					flag = 0;
				}

				transactionManager.commit(status);
				
			} catch (DataAccessException sqle) {
				log.info("******** DataAccessException***********"+sqle);
				transactionManager.rollback(status);
				flag = 2;
			}

		} else {
			flag = 2;
		}
		//log.info("existing insertAnxXADetails1 flag" + flag);
		return flag;
	}

	public ArrayList<RW04> getMakerReportScreenDetails2(String report_id) {

		//log.info("entering getMakerReportScreenDetails2 report_id" + report_id);
		ArrayList<RW04> list = new ArrayList<RW04>();

		String query1 = "select rm.branch_code , rm.quarter,rm.financial_year,ax.CRS_OTH_ASSESTS_ID,ax.CRS_OTH_ASSESTS_NAME,ax.OTH_ASTS_LST_YR,ax.OTH_ASTS_WRITE_OFF ,"
				+ " ax.OTH_ASTS_ADDITION,ax.OTH_ASTS_REDUCTION,ax.OTH_ASTS_CUR_YR,ax.OTH_ASTS_PROVI_RATE,ax.OTH_ASTS_PROVI_REQ "
				+ "  from reports_master_list rm , CRS_OTH_ASSESTS ax "
				+ " where rm.report_id=ax.REPORT_MASTER_LIST_ID_FK and rm.report_id=? and ax.CRS_OTH_ASSESTS_ID not in (1,2,3,4,5,6,7,8,9,10,11,12,13,14,15,16,17,18)";

		try {

			list = jdbcTemplate.query(query1, new Object[] { report_id }, new ResultSetExtractor<ArrayList<RW04>>() {

				@Override
				public ArrayList<RW04> extractData(ResultSet rs) throws SQLException, DataAccessException {				
					ArrayList<RW04> list = new ArrayList<RW04>();
					while (rs.next()) {

						RW04 report = new RW04();

						report.setQuarter(rs.getString(CommonConstants.QUARTER));
						report.setYear(rs.getString("financial_year"));
						report.setBranchCode(rs.getString("branch_code"));
						report.setParticularsList(rs.getString("CRS_OTH_ASSESTS_NAME"));
						report.setProvAmt2015List(rs.getString("OTH_ASTS_LST_YR"));
						report.setWriteOffDur12monList(rs.getString("OTH_ASTS_WRITE_OFF"));
						report.setAdditionDur12monList(rs.getString("OTH_ASTS_ADDITION"));
						report.setReduInProviAmtList(rs.getString("OTH_ASTS_REDUCTION"));
						report.setProviAmt2016List(rs.getString("OTH_ASTS_CUR_YR"));
						report.setRatePOfProvList(rs.getString("OTH_ASTS_PROVI_RATE"));
						report.setProvReqList(rs.getString("OTH_ASTS_PROVI_REQ"));
						list.add(report);
					}
					return list;
				}

			});

		} catch (DataAccessException e) {
			log.info("******* DataAccessException *********");

		} finally {

		}
		//log.info("existing getMakerReportScreenDetails2");

		return list;
	}

	public RW04 getMakerReportScreenDetails1(final RW04 report, String report_id) {

		RW04 newReport = null;

		String query1 = "select rm.branch_code , rm.quarter,rm.financial_year,ax.CRS_OTH_ASSESTS_ID,ax.CRS_OTH_ASSESTS_NAME,ax.OTH_ASTS_LST_YR,ax.OTH_ASTS_WRITE_OFF ,"
				+ " ax.OTH_ASTS_ADDITION,ax.OTH_ASTS_REDUCTION,ax.OTH_ASTS_CUR_YR,ax.OTH_ASTS_PROVI_RATE,ax.OTH_ASTS_PROVI_REQ "
				+ "  from reports_master_list rm , CRS_OTH_ASSESTS ax "
				+ " where rm.report_id=ax.REPORT_MASTER_LIST_ID_FK and rm.report_id=?";

		try {

			newReport = jdbcTemplate.query(query1, new Object[] { report_id }, new ResultSetExtractor<RW04>() {

				@Override
				public RW04 extractData(ResultSet rs) throws SQLException, DataAccessException {					
					while (rs.next()) {

						report.setQuarter(rs.getString(CommonConstants.QUARTER));
						report.setYear(rs.getString("financial_year"));
						report.setBranchCode(rs.getString("branch_code"));
						String sr_no = rs.getString("CRS_OTH_ASSESTS_ID");
						if (sr_no.equalsIgnoreCase("1")) {
							report.setFraudsDebitedProvAfter(rs.getString("OTH_ASTS_LST_YR"));
							report.setFraudsDebitedWrite(rs.getString("OTH_ASTS_WRITE_OFF"));
							report.setFraudsDebitedAddition(rs.getString("OTH_ASTS_ADDITION"));
							report.setFraudsDebitedReduction(rs.getString("OTH_ASTS_REDUCTION"));
							report.setFraudsDebitedProvOn(rs.getString("OTH_ASTS_CUR_YR"));
							report.setFraudsDebitedRate(rs.getString("OTH_ASTS_PROVI_RATE"));
							report.setFraudsDebitedProvReq(rs.getString("OTH_ASTS_PROVI_REQ"));
						} else if (sr_no.equalsIgnoreCase("2")) {
							report.setFraudsDebitedPrior100ProvAfter(rs.getString("OTH_ASTS_LST_YR"));
							report.setFraudsDebitedPrior100Write(rs.getString("OTH_ASTS_WRITE_OFF"));
							report.setFraudsDebitedPrior100Addition(rs.getString("OTH_ASTS_ADDITION"));
							report.setFraudsDebitedPrior100Reduction(rs.getString("OTH_ASTS_REDUCTION"));
							report.setFraudsDebitedPrior100ProvOn(rs.getString("OTH_ASTS_CUR_YR"));
							report.setFraudsDebitedPrior100Rate(rs.getString("OTH_ASTS_PROVI_RATE"));
							report.setFraudsDebitedPrior100ProvReq(rs.getString("OTH_ASTS_PROVI_REQ"));
						} else if (sr_no.equalsIgnoreCase("3")) {
							report.setFraudsDebitedPrior75ProvAfter(rs.getString("OTH_ASTS_LST_YR"));
							report.setFraudsDebitedPrior75Write(rs.getString("OTH_ASTS_WRITE_OFF"));
							report.setFraudsDebitedPrior75Addition(rs.getString("OTH_ASTS_ADDITION"));
							report.setFraudsDebitedPrior75Reduction(rs.getString("OTH_ASTS_REDUCTION"));
							report.setFraudsDebitedPrior75ProvOn(rs.getString("OTH_ASTS_CUR_YR"));
							report.setFraudsDebitedPrior75Rate(rs.getString("OTH_ASTS_PROVI_RATE"));
							report.setFraudsDebitedPrior75ProvReq(rs.getString("OTH_ASTS_PROVI_REQ"));
						} else if (sr_no.equalsIgnoreCase("4")) {
							report.setFraudsDebitedPrior50ProvAfter(rs.getString("OTH_ASTS_LST_YR"));
							report.setFraudsDebitedPrior50Write(rs.getString("OTH_ASTS_WRITE_OFF"));
							report.setFraudsDebitedPrior50Addition(rs.getString("OTH_ASTS_ADDITION"));
							report.setFraudsDebitedPrior50Reduction(rs.getString("OTH_ASTS_REDUCTION"));
							report.setFraudsDebitedPrior50ProvOn(rs.getString("OTH_ASTS_CUR_YR"));
							report.setFraudsDebitedPrior50Rate(rs.getString("OTH_ASTS_PROVI_RATE"));
							report.setFraudsDebitedPrior50ProvReq(rs.getString("OTH_ASTS_PROVI_REQ"));
						} else if (sr_no.equalsIgnoreCase("5")) {
							report.setFraudsDebitedPrior25ProvAfter(rs.getString("OTH_ASTS_LST_YR"));
							report.setFraudsDebitedPrior25Write(rs.getString("OTH_ASTS_WRITE_OFF"));
							report.setFraudsDebitedPrior25Addition(rs.getString("OTH_ASTS_ADDITION"));
							report.setFraudsDebitedPrior25Reduction(rs.getString("OTH_ASTS_REDUCTION"));
							report.setFraudsDebitedPrior25ProvOn(rs.getString("OTH_ASTS_CUR_YR"));
							report.setFraudsDebitedPrior25Rate(rs.getString("OTH_ASTS_PROVI_RATE"));
							report.setFraudsDebitedPrior25ProvReq(rs.getString("OTH_ASTS_PROVI_REQ"));
						} else if (sr_no.equalsIgnoreCase("6")) {
							report.setFraudsDebitedDelayedProvAfter(rs.getString("OTH_ASTS_LST_YR"));
							report.setFraudsDebitedDelayedWrite(rs.getString("OTH_ASTS_WRITE_OFF"));
							report.setFraudsDebitedDelayedAddition(rs.getString("OTH_ASTS_ADDITION"));
							report.setFraudsDebitedDelayedReduction(rs.getString("OTH_ASTS_REDUCTION"));
							report.setFraudsDebitedDelayedProvOn(rs.getString("OTH_ASTS_CUR_YR"));
							report.setFraudsDebitedDelayedRate(rs.getString("OTH_ASTS_PROVI_RATE"));
							report.setFraudsDebitedDelayedProvReq(rs.getString("OTH_ASTS_PROVI_REQ"));
						} else if (sr_no.equalsIgnoreCase("7")) {
							report.setOthersRecalledProvAfter(rs.getString("OTH_ASTS_LST_YR"));
							report.setOthersRecalledWrite(rs.getString("OTH_ASTS_WRITE_OFF"));
							report.setOthersRecalledAddition(rs.getString("OTH_ASTS_ADDITION"));
							report.setOthersRecalledReduction(rs.getString("OTH_ASTS_REDUCTION"));
							report.setOthersRecalledProvOn(rs.getString("OTH_ASTS_CUR_YR"));
							report.setOthersRecalledRate(rs.getString("OTH_ASTS_PROVI_RATE"));
							report.setOthersRecalledProvReq(rs.getString("OTH_ASTS_PROVI_REQ"));
						} else if (sr_no.equalsIgnoreCase("8")) {
							report.setFraudsOthersProvAfter(rs.getString("OTH_ASTS_LST_YR"));
							report.setFraudsOthersWrite(rs.getString("OTH_ASTS_WRITE_OFF"));
							report.setFraudsOthersAddition(rs.getString("OTH_ASTS_ADDITION"));
							report.setFraudsOthersReduction(rs.getString("OTH_ASTS_REDUCTION"));
							report.setFraudsOthersProvOn(rs.getString("OTH_ASTS_CUR_YR"));
							report.setFraudsOthersRate(rs.getString("OTH_ASTS_PROVI_RATE"));
							report.setFraudsOthersProvReq(rs.getString("OTH_ASTS_PROVI_REQ"));
						} else if (sr_no.equalsIgnoreCase("9")) {
							report.setFraudsOthersPrior100ProvAfter(rs.getString("OTH_ASTS_LST_YR"));
							report.setFraudsOthersPrior100Write(rs.getString("OTH_ASTS_WRITE_OFF"));
							report.setFraudsOthersPrior100Addition(rs.getString("OTH_ASTS_ADDITION"));
							report.setFraudsOthersPrior100Reduction(rs.getString("OTH_ASTS_REDUCTION"));
							report.setFraudsOthersPrior100ProvOn(rs.getString("OTH_ASTS_CUR_YR"));
							report.setFraudsOthersPrior100Rate(rs.getString("OTH_ASTS_PROVI_RATE"));
							report.setFraudsOthersPrior100ProvReq(rs.getString("OTH_ASTS_PROVI_REQ"));
						} else if (sr_no.equalsIgnoreCase("10")) {
							report.setFraudsOthersPrior75ProvAfter(rs.getString("OTH_ASTS_LST_YR"));
							report.setFraudsOthersPrior75Write(rs.getString("OTH_ASTS_WRITE_OFF"));
							report.setFraudsOthersPrior75Addition(rs.getString("OTH_ASTS_ADDITION"));
							report.setFraudsOthersPrior75Reduction(rs.getString("OTH_ASTS_REDUCTION"));
							report.setFraudsOthersPrior75ProvOn(rs.getString("OTH_ASTS_CUR_YR"));
							report.setFraudsOthersPrior75Rate(rs.getString("OTH_ASTS_PROVI_RATE"));
							report.setFraudsOthersPrior75ProvReq(rs.getString("OTH_ASTS_PROVI_REQ"));
						} else if (sr_no.equalsIgnoreCase("11")) {
							report.setFraudsOthersPrior50ProvAfter(rs.getString("OTH_ASTS_LST_YR"));
							report.setFraudsOthersPrior50Write(rs.getString("OTH_ASTS_WRITE_OFF"));
							report.setFraudsOthersPrior50Addition(rs.getString("OTH_ASTS_ADDITION"));
							report.setFraudsOthersPrior50Reduction(rs.getString("OTH_ASTS_REDUCTION"));
							report.setFraudsOthersPrior50ProvOn(rs.getString("OTH_ASTS_CUR_YR"));
							report.setFraudsOthersPrior50Rate(rs.getString("OTH_ASTS_PROVI_RATE"));
							report.setFraudsOthersPrior50ProvReq(rs.getString("OTH_ASTS_PROVI_REQ"));
						} else if (sr_no.equalsIgnoreCase("12")) {
							report.setFraudsOthersPrior25ProvAfter(rs.getString("OTH_ASTS_LST_YR"));
							report.setFraudsOthersPrior25Write(rs.getString("OTH_ASTS_WRITE_OFF"));
							report.setFraudsOthersPrior25Addition(rs.getString("OTH_ASTS_ADDITION"));
							report.setFraudsOthersPrior25Reduction(rs.getString("OTH_ASTS_REDUCTION"));
							report.setFraudsOthersPrior25ProvOn(rs.getString("OTH_ASTS_CUR_YR"));
							report.setFraudsOthersPrior25Rate(rs.getString("OTH_ASTS_PROVI_RATE"));
							report.setFraudsOthersPrior25ProvReq(rs.getString("OTH_ASTS_PROVI_REQ"));
						} else if (sr_no.equalsIgnoreCase("13")) {
							report.setFraudsOthersDelayedProvAfter(rs.getString("OTH_ASTS_LST_YR"));
							report.setFraudsOthersDelayedWrite(rs.getString("OTH_ASTS_WRITE_OFF"));
							report.setFraudsOthersDelayedAddition(rs.getString("OTH_ASTS_ADDITION"));
							report.setFraudsOthersDelayedReduction(rs.getString("OTH_ASTS_REDUCTION"));
							report.setFraudsOthersDelayedProvOn(rs.getString("OTH_ASTS_CUR_YR"));
							report.setFraudsOthersDelayedRate(rs.getString("OTH_ASTS_PROVI_RATE"));
							report.setFraudsOthersDelayedProvReq(rs.getString("OTH_ASTS_PROVI_REQ"));
						} else if (sr_no.equalsIgnoreCase("14")) {
							report.setRevenueProvAfter(rs.getString("OTH_ASTS_LST_YR"));
							report.setRevenueWrite(rs.getString("OTH_ASTS_WRITE_OFF"));
							report.setRevenueAddition(rs.getString("OTH_ASTS_ADDITION"));
							report.setRevenueReduction(rs.getString("OTH_ASTS_REDUCTION"));
							report.setRevenueProvOn(rs.getString("OTH_ASTS_CUR_YR"));
							report.setRevenueRate(rs.getString("OTH_ASTS_PROVI_RATE"));
							report.setRevenueProvReq(rs.getString("OTH_ASTS_PROVI_REQ"));
						} else if (sr_no.equalsIgnoreCase("15")) {
							report.setFsloProvAfter(rs.getString("OTH_ASTS_LST_YR"));
							report.setFsloWrite(rs.getString("OTH_ASTS_WRITE_OFF"));
							report.setFsloAddition(rs.getString("OTH_ASTS_ADDITION"));
							report.setFsloReduction(rs.getString("OTH_ASTS_REDUCTION"));
							report.setFsloProvOn(rs.getString("OTH_ASTS_CUR_YR"));
							report.setFsloRate(rs.getString("OTH_ASTS_PROVI_RATE"));
							report.setFsloProvReq(rs.getString("OTH_ASTS_PROVI_REQ"));
						} else if (sr_no.equalsIgnoreCase("16")) {
							report.setOutstandingProvAfter(rs.getString("OTH_ASTS_LST_YR"));
							report.setOutstandingWrite(rs.getString("OTH_ASTS_WRITE_OFF"));
							report.setOutstandingAddition(rs.getString("OTH_ASTS_ADDITION"));
							report.setOutstandingReduction(rs.getString("OTH_ASTS_REDUCTION"));
							report.setOutstandingProvOn(rs.getString("OTH_ASTS_CUR_YR"));
							report.setOutstandingRate(rs.getString("OTH_ASTS_PROVI_RATE"));
							report.setOutstandingProvReq(rs.getString("OTH_ASTS_PROVI_REQ"));
						}else if (sr_no.equalsIgnoreCase("17")) {
							report.setNpainterestProvAfter(rs.getString("OTH_ASTS_LST_YR"));
							report.setNpainterestWrite(rs.getString("OTH_ASTS_WRITE_OFF"));
							report.setNpainterestAddition(rs.getString("OTH_ASTS_ADDITION"));
							report.setNpainterestReduction(rs.getString("OTH_ASTS_REDUCTION"));
							report.setNpainterestProvOn(rs.getString("OTH_ASTS_CUR_YR"));
							report.setNpainterestRate(rs.getString("OTH_ASTS_PROVI_RATE"));
							report.setNpainterestProvReq(rs.getString("OTH_ASTS_PROVI_REQ"));
						}
					}
					return report;
				}

			});

		} catch (Exception sqle) {
			log.error("Exception  " + sqle.getMessage());
		}

		return newReport;
	}

}

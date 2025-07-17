package com.tcs.controllers;

import com.tcs.beans.Frt;
import com.tcs.beans.FrtAddBranch;
import com.tcs.beans.SessionBean;
import com.tcs.services.FrtAddBranchService;
import com.tcs.utils.CommonConstants;
import org.apache.log4j.Logger;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.jdbc.core.JdbcTemplate;
import org.springframework.stereotype.Controller;
import org.springframework.validation.BindingResult;
import org.springframework.validation.FieldError;
import org.springframework.web.bind.annotation.*;
import org.springframework.web.servlet.ModelAndView;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpSession;
import javax.sql.DataSource;
import java.math.BigDecimal;
import java.util.List;
import java.util.Map;
import java.util.Objects;


@Controller
@RequestMapping("/FRTUser")

public class FrtAddBranchController {

    static Logger log = Logger.getLogger(FrtAddBranchController.class.getName());

    @Autowired
    DataSource dataSource;
    
    @Autowired
    JdbcTemplate jdbcTemplateObject; // Universal DB constructor added

    

    @Autowired
    FrtAddBranchService frtAddBranchService;

    @GetMapping("/branchStatus")
    public ModelAndView branchStatus(@ModelAttribute("command") Frt frt, BindingResult result, HttpServletRequest request) {

        HttpSession session = request.getSession();
        if (session == null || session.getAttribute(CommonConstants.USER_ID) == null || request.getSession().getId() == null ||
                !Objects.equals(session.getAttribute(CommonConstants.USER_CAPABILITY), new BigDecimal("96"))) {
            log.info("User check " );
            return new ModelAndView("500");
        }

        log.info("inside db");
        List<FrtAddBranch> list = frtAddBranchService.getCircleCodeUs();
        ModelAndView view = new ModelAndView("FRTUser/branchStatus");
        String financialYear = session.getAttribute("year").toString();
        String quarterCurrent = (String) session.getAttribute("quarter");
        view.addObject("financialYear", financialYear);
        view.addObject("quarterCurrent", quarterCurrent);
        view.addObject("circleList", list);

        return view;
    }


    @PostMapping("/search")
    @ResponseBody
    public List<FrtAddBranch> search(@ModelAttribute("FRT") FrtAddBranch frt, @RequestParam("branchCode") String branchCode, BindingResult result,
                                     HttpServletRequest request) {

        log.info("branchCode : Inside Search in controller. :" + branchCode);
        HttpSession session = request.getSession();
        SessionBean sessionBean = new SessionBean(request.getSession());
        String quarterEndDate = sessionBean.getQuarterEndDate();
        String searchCode = branchCode;
        log.error("errors :" + result.getFieldErrors());
        List<FieldError> errors = result.getFieldErrors();
        for (FieldError error : errors) {
            log.error("errors :" + error.getField() + "name: " + error.getRejectedValue());
        }
        log.error("errorname" + result.toString());


        List<FrtAddBranch> list1 = frtAddBranchService.searchBranch(branchCode, quarterEndDate);

        if (list1.size() != 0) {
            return list1;
        } else {
            return null;
        }

    }

    @PostMapping(value = "/saveData")
    public @ResponseBody
    String saveData(@RequestBody Map map, HttpServletRequest request) {

        HttpSession session = request.getSession();
        SessionBean sessionBean = new SessionBean(request.getSession());
        String userId = sessionBean.getUserId();
        String displayMessage = "NODATA";
        log.info("userId :" + userId + " " + sessionBean.getBranchCode() + " " + sessionBean.getQuarterEndDate());
        String branchCode1 = (String) map.get("branchCode1");
        String branchName = (String) map.get("branchName");
        String circleCode = (String) map.get("circleCode");
        String auditStatus = (String) map.get("auditStatus");
        String network = (String) map.get("network");
        String module = (String) map.get("module");
        String region = (String) map.get("region");
        String address1 = (String) map.get("address1");
        String address2 = (String) map.get("address2");
        String address3 = (String) map.get("address3");
        String pinCode = (String) map.get("pinCode");
        String stdCode = (String) map.get("stdCode");
        String phoneNo = (String) map.get("phoneNo");
        String voipNo = (String) map.get("voipNo");
        String manName = (String) map.get("manName");
        String residencePhone = (String) map.get("residencePhone");
        String mobileNo = (String) map.get("mobileNo");
        String requestId = (String) map.get("isRequestExists");

        boolean flag = frtAddBranchService.updatebranchData(branchCode1, branchName, circleCode, auditStatus, network, module, region,
                address1, address2, address3, pinCode, stdCode, phoneNo, voipNo, manName, residencePhone, mobileNo);
        if(flag){
            String id;
            if(requestId.isEmpty()){
                log.info("For insert");
                 id = frtAddBranchService.insertTrack(userId, sessionBean.getQuarterEndDate(),branchCode1) ;
            }else{
                log.info("For update requestId :" +requestId);
                 id  = frtAddBranchService.updateTrack(requestId,userId, sessionBean.getQuarterEndDate(),branchCode1);
            }
            if (!id.equalsIgnoreCase("error")) {
                log.info("id :" +id);
                displayMessage = "Request id "+id+" for adding "+ branchCode1+" "+branchName
                        +" to CRS Scope for quarter ending "+sessionBean.getQuarterEndDate() +" is pending with FRT Checker.";
                log.info(displayMessage);
            } else {
                displayMessage = "Error in saving the data. Please re-try.";
            }
        }
        return displayMessage;

    }

}

/////////////////////////////////////////////////////////////////////


package com.tcs.services;

import com.tcs.beans.FrtAddBranch;
import com.tcs.dao.FrtAddBranchDao;
import org.apache.log4j.Logger;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

import java.util.List;

@Service(value = "frtAddBranchService")
public class FrtAddBranchServiceImpl implements FrtAddBranchService {

    static Logger log = Logger.getLogger(FrtAddBranchServiceImpl.class.getName());

    @Autowired
    FrtAddBranchDao frtAddBranchDao;

    @Override
    public List<FrtAddBranch> searchBranch(String branchCode, String quarterEndDate ) {
        log.info("branchCode :" + branchCode);
        return frtAddBranchDao.searchBranch(branchCode, quarterEndDate);
    }

    @Override
    public List<FrtAddBranch> getCircleCodeUs() {
        return frtAddBranchDao.getCircleCodeUs();
    }

    @Override
    public boolean updatebranchData(String branchCode1, String branchName, String circleCode, String auditStatus, String network, String module, String region, String address1, String address2, String address3,
                                    String pinCode, String stdCode, String phoneNo, String voipNo, String manName, String residencePhone, String mobileNo) {
        return frtAddBranchDao.updatebranchData(branchCode1, branchName, circleCode, auditStatus, network, module, region, address1, address2, address3,
                pinCode, stdCode, phoneNo, voipNo, manName, residencePhone, mobileNo);
    }

    @Override
    public String updateTrack(String requestId, String userId, String quarterEndDate, String branchCode1) {
        return frtAddBranchDao.updateTrack(requestId, userId, quarterEndDate, branchCode1);
    }

    @Override
    public String insertTrack(String userId, String quarterEndDate, String branchCode1) {
        return frtAddBranchDao.insertTrack(userId, quarterEndDate, branchCode1);
    }
}
/////////////////////////////////////////////////////////////////////////////////////////


package com.tcs.dao;

import com.tcs.beans.FrtAddBranch;
import org.apache.log4j.Logger;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.dao.DataAccessException;
import org.springframework.jdbc.core.JdbcTemplate;
import org.springframework.jdbc.core.ResultSetExtractor;
import org.springframework.stereotype.Repository;
import org.springframework.transaction.PlatformTransactionManager;

import javax.sql.DataSource;
import java.sql.ResultSet;
import java.sql.SQLException;
import java.util.ArrayList;
import java.util.List;

@Repository("frtAddBranchDao")
public class FrtAddBranchDaoImpl implements FrtAddBranchDao {

    static Logger log = Logger.getLogger(FrtAddBranchDaoImpl.class.getName());
    @Autowired
    DataSource dataSource;
    @Autowired
    JdbcTemplate jdbcTemplateObject;

    @Autowired
    private PlatformTransactionManager transactionManager;

    @Override
    public List<FrtAddBranch> searchBranch(String branchCode, String quarterEndDate) {

        //JdbcTemplate jdbcTemplateObject = new JdbcTemplate(dataSource);
        List<FrtAddBranch> list = null;
        log.info("branchCode :" + branchCode);
        String q;


        q = "SELECT  BRANCHNO, BR_NAME, ADDRESS_1, ADDRESS_2, ADDRESS_3, POST_CODE, PHONE_NO, STD_CODE, CIRCLE_CODE,\n" +
                "REGION_CODE, VOIP_PHONE_NO, MAN_RES_PHONE, MAN_MOBI_PHONE,INST,MANAGERS_NAME, CRS_AUDITABLE,\n" +
                "(select branchno from branch_master where branch_master.branchno=cbs_brhm.BRANCHNO) isBranchExists,\n" +
                "(select RT_ID from crs_request_track where RT_TYPE ='B' and RT_SUBTYPE = 'A' and RT_QED = to_date(?,'dd/mm/yyyy') and RT_BRANCH = branchno and \n" +
                "RT_STATUS='1') isRequestExists\n" +
                "FROM CBS_BRHM WHERE BRANCHNO = lpad(?,5,'0')";
        try {
            list = jdbcTemplateObject.query(q, new Object[]{quarterEndDate, branchCode},
                    rs -> {
                        FrtAddBranch report = null;
                        List<FrtAddBranch> list1 = new ArrayList<FrtAddBranch>();

                        while (rs.next()) {
                            report = new FrtAddBranch();
                            report.setBranchName(rs.getString("BR_NAME"));
                            report.setAddress1(rs.getString("ADDRESS_1"));
                            report.setAddress2(rs.getString("ADDRESS_2"));
                            report.setAddress3(rs.getString("ADDRESS_3"));
                            report.setPostCode(rs.getString("POST_CODE"));
                            report.setPhoneNumber(rs.getString("PHONE_NO"));
                            report.setStdCode(rs.getString("STD_CODE"));
                            report.setCircleCode(rs.getString("CIRCLE_CODE"));
                            report.setRegionCode(rs.getString("REGION_CODE"));
                            if(!report.getRegionCode().isEmpty()){
                                report.setNetwerkCode(report.getRegionCode().substring(0,3));
                                report.setModuleCode(report.getRegionCode().substring(3,6));
                                report.setRegionCode(report.getRegionCode().substring(6,9));
                            }
                            report.setVoipPhoneNumber(rs.getString("VOIP_PHONE_NO"));
                            report.setManResPhoneNumber(rs.getString("MAN_RES_PHONE"));
                            report.setManMobileNumber(rs.getString("MAN_MOBI_PHONE"));
                            report.setInst(rs.getString("INST"));
                            report.setManName(rs.getString("MANAGERS_NAME"));
                            report.setCrsAuditSts(rs.getString("CRS_AUDITABLE"));
                            report.setInsertId(rs.getString("isRequestExists"));
                            report.setBranchExists(rs.getString("isBranchExists"));
                            list1.add(report);
                        }

                        return list1;
                    });
            log.info("query result :" + list);
            return list;
        } catch (DataAccessException e) {
            log.error("***Data Access Exception Occured***" + e.getMessage());
        }
        return list;
    }

    @Override
    public List<FrtAddBranch> getCircleCodeUs() {
        boolean status = false;

        log.info("IN DAO IMPL");


        //JdbcTemplate jdbcTemplateObject = new JdbcTemplate(dataSource);
        String query = "";

        query = "select CRS_SYS_PARAM_ID,CRS_SYS_PARAM_NAME from crs_sys_param where CRS_SYS_PARAM_TYPE='CIRCLE_MASTER' and CRS_SYS_PARAM_ID < '100' ORDER BY CRS_SYS_PARAM_ID";

        List<FrtAddBranch> list = jdbcTemplateObject.query(query, new Object[]{},
                new ResultSetExtractor<List<FrtAddBranch>>() {

                    @Override
                    public List<FrtAddBranch> extractData(ResultSet rs) throws SQLException, DataAccessException {
                        FrtAddBranch circle = null;
                        List<FrtAddBranch> list = new ArrayList<FrtAddBranch>();

                        while (rs.next()) {
                            circle = new FrtAddBranch();
                            circle.setCircleCode(rs.getString("CRS_SYS_PARAM_ID"));
                            circle.setCircleName(rs.getString("CRS_SYS_PARAM_NAME"));
                            list.add(circle);
                        }

                        return list;
                    }

                });
        return list;
    }

    @Override
    public boolean updatebranchData(String branchCode1, String branchName, String circleCode, String auditStatus, String network, String module, String region,
                                    String address1, String address2, String address3, String pinCode, String stdCode, String phoneNo,
                                    String voipNo, String manName, String residencePhone, String mobileNo) {

        log.info("branchCode1 :" + branchCode1);
        String query = "";
        String regionCode = network+module+region;
        log.info(regionCode);
        int updated = 0;
        //JdbcTemplate jdbcTemplateObject = new JdbcTemplate(dataSource);


        query = "update CBS_BRHM set  ADDRESS_1 = ?, ADDRESS_2 = ?, ADDRESS_3 = ?, POST_CODE = ?, PHONE_NO = ?,  STD_CODE = ?, CIRCLE_CODE = ?, \n" +
                " REGION_CODE = ?, VOIP_PHONE_NO = ?, MAN_RES_PHONE = ?, MAN_MOBI_PHONE = ?,MANAGERS_NAME = ?, CRS_AUDITABLE = ? WHERE BRANCHNO = ?";
        try {
            updated = jdbcTemplateObject.update(query, new Object[]{address1, address2, address3, pinCode,
                    phoneNo,  stdCode, circleCode, regionCode, voipNo, residencePhone, mobileNo, manName, auditStatus, branchCode1});
        } catch (DataAccessException e) {
            log.error("***Data Access Exception occured*** :" + e.getMessage());
        }
        if (updated != 0) {
            return true;
        } else {
            return false;
        }


    }

    @Override
    public String updateTrack(String requestId, String userId, String quarterEndDate, String branchCode1) {

        //JdbcTemplate jdbcTemplateObject = new JdbcTemplate(dataSource);
        int updated = 0;
        String query = "UPDATE CRS_REQUEST_TRACK SET RT_MAKER = ?, RT_STATUS = ?, RT_TYPE = ?, RT_SUBTYPE = ?, RT_QED = to_date(?,'dd/mm/yyyy'), RT_BRANCH = ? " +
                "WHERE RT_ID = ?";
        try {
            updated = jdbcTemplateObject.update(query, new Object[]{userId, "1", "B", "A", quarterEndDate, branchCode1, requestId});
        } catch (DataAccessException e) {
            log.error("***Data Access Exception***" + e.getMessage());
        }
        if (updated != 0) {
            return requestId;
        } else {
            return "error";
        }

    }

    @Override
    public String insertTrack(String userId, String quarterEndDate, String branchCode1) {
        log.info("in insert");

        String requestId = "error";
        String query = "";
        int i = 0;
        //JdbcTemplate jdbcTemplateObject = new JdbcTemplate(dataSource);

        query = "INSERT INTO CRS_REQUEST_TRACK (RT_ID, RT_MAKER, RT_STATUS, RT_TYPE, RT_SUBTYPE, RT_QED, RT_BRANCH) " +
                "VALUES(CRS_REQUEST_TRACK_SEQ.nextval,?,?,?,?,to_date(?,'dd/mm/yyyy'),?)";
        try {
            i = jdbcTemplateObject.update(query, new Object[]{userId, "1", "B", "A", quarterEndDate, branchCode1});
            log.info("i :" + i);
        } catch (DataAccessException e) {
            e.printStackTrace();
        }
        String q2 = "select CRS_REQUEST_TRACK_SEQ.currval from dual";
        if (i != 0) {
            try {
                requestId = jdbcTemplateObject.queryForObject(q2, String.class);
                log.info("requestId :" + requestId);
            } catch (DataAccessException e) {
                e.printStackTrace();
            }
        }
        return requestId;
    }
}

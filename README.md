package com.tcs.controllers;

import com.tcs.services.RW04Service;
import org.apache.log4j.Logger;
import org.springframework.web.bind.annotation.*;

import java.util.Map;

@RestController
@RequestMapping("/RW04")
public class RW04Controller {

    final RW04Service rw04Service;

    public RW04Controller (RW04Service rw04Service){
        this.rw04Service = rw04Service;
    }

    static Logger log = Logger.getLogger(RW04Controller.class.getName());

    @RequestMapping(value = "/getData", method = RequestMethod.POST)
    public @ResponseBody Map<String, Object> getRW04Data (@RequestBody Map<String, Object> payload){
        Map<String, Object> result;
        result = rw04Service.getRW04Data(payload);
        return result;
    }

    @RequestMapping("/saveStatic")
    public @ResponseBody Map<String, Object> saveStaticData (@RequestBody Map<String, Object> payload){
        Map<String, Object> result;
        result = rw04Service.saveStaticTable(payload);
        return result;
    }

    @RequestMapping("/saveAddRow")
    public @ResponseBody Map<String, Object> saveAddRow (@RequestBody Map<String, Object> payload){
        Map<String, Object> result;
        result = rw04Service.saveAddRow(payload);
        return result;
    }

    @RequestMapping("/deleteRow")
    public @ResponseBody Map<String, Object> deleteRow (@RequestBody Map<String, Object> payload){
        Map<String, Object> result;
        result = rw04Service.deleteRow(payload);
        return result;
    }

}


///////////////////////////////////////


package com.tcs.services;

import com.tcs.dao.RW04Dao;
import org.apache.log4j.Logger;
import org.springframework.stereotype.Service;

import java.util.*;

@Service
public class RW04ServiceImpl implements RW04Service {

    static Logger log = Logger.getLogger(RW04ServiceImpl.class.getName());

    final RW04Dao rw04Dao;

    public RW04ServiceImpl(RW04Dao rw04Dao) {
        this.rw04Dao = rw04Dao;
    }

    @Override
    public Map<String, Object> getRW04Data(Map<String, Object> payload) {
        String circleCode = (String) payload.get("circleCode");
        String quarterDate = (String) payload.get("qed");
        String userRole = (String) payload.get("userCapacity");
        String status = (String) payload.get("currentStatus");

        if (userRole.equalsIgnoreCase("51")) {
            String reportId = (String) payload.get("reportId");
            if (reportId.equalsIgnoreCase("") || reportId.equalsIgnoreCase("null")) {
                return new HashMap<>(Collections.singletonMap("message", "reportId is null or empty"));
            } else {
                Map<String, Object> data = new HashMap<>();
                if (status.equalsIgnoreCase("11") || status.equalsIgnoreCase("12")) {
                    data = rw04Dao.getRW04Data(circleCode, quarterDate, reportId);
                } else {
                    data.put("message", "data is not present for the report.");
                }
                return data;
            }
        } else {
            String message = "Unauthorized user";
            return new HashMap<>(Collections.singletonMap("retVal", message));
        }
    }

    @Override
    public Map<String, Object> saveStaticTable(Map<String, Object> payload) {
        String circleCode = (String) payload.get("circleCode");
        String quarterDate = (String) payload.get("qed");
        String userRole = (String) payload.get("userCapacity");
        String status = (String) payload.get("currentStatus");
        List<List<String>> value = (List<List<String>>) payload.get("value");

        if (userRole.equalsIgnoreCase("51")) {
            String reportId = (String) payload.get("reportId");
            if (reportId.equalsIgnoreCase("") || reportId.equalsIgnoreCase("null")) {
                return new HashMap<>(Collections.singletonMap("message", "reportId is null or empty"));
            } else {
                if (status.equalsIgnoreCase("10")) {
                    Map<String, Object> insert;
                    ArrayList<String> particularsList = getParticularList(quarterDate);
                    insert = rw04Dao.insertStaticTable(particularsList, circleCode, quarterDate, reportId, value);
                    return insert;
                } else if (status.equalsIgnoreCase("11") || status.equalsIgnoreCase("12")) {
                    Map<String, Object> update;
                    update = rw04Dao.updateStaticTable(circleCode, quarterDate, reportId, value);
                    return update;
                } else {
                    return new HashMap<>(Collections.singletonMap("message", "report not saved."));
                }
            }
        } else {
            String message = "Unauthorized user";
            return new HashMap<>(Collections.singletonMap("retVal", message));
        }
    }

    @Override
    public Map<String, Object> saveAddRow(Map<String, Object> payload) {
        String circleCode = (String) payload.get("circleCode");
        String quarterDate = (String) payload.get("qed");
        String userRole = (String) payload.get("userCapacity");
        List<String> value = (List<String>) payload.get("value");

        if (userRole.equalsIgnoreCase("51")) {
            String reportId = (String) payload.get("reportId");
            if (reportId.equalsIgnoreCase("") || reportId.equalsIgnoreCase("null")) {
                return new HashMap<>(Collections.singletonMap("message", "reportId is null or empty"));
            } else {
                Map<String, Object> insert;
                insert = rw04Dao.saveAddRow(circleCode, quarterDate, reportId, value);
                return insert;
            }
        } else {
            String message = "Unauthorized user";
            return new HashMap<>(Collections.singletonMap("retVal", message));
        }
    }

    @Override
    public Map<String, Object> deleteRow(Map<String, Object> payload) {
        String userRole = (String) payload.get("userCapacity");
        int rowId = Integer.parseInt((String) payload.get("rowId"));

        if(userRole.equalsIgnoreCase("51")) {
            Map<String, Object> delete;
            delete = rw04Dao.deleteRow(rowId);
            return delete;
        }
        else{
            String message = "Unauthorized user";
            return new HashMap<>(Collections.singletonMap("retVal", message));
        }
    }

    private ArrayList<String> getParticularList(String quarterDate) {
        ArrayList<String> particularList = new ArrayList<>();
        particularList.add("1~FRAUDS - DEBITED TO RECALLED ASSETS A/c (Prod Cd 6998-9981)**");
        particularList.add("2~Frauds reported on or prior to " + quarterDate + " provision @ 100% ##");
        particularList.add("3~Delayed Reported frauds Provision @ 100% ##");
        particularList.add("4~OTHERS LOSSES IN RECALLED ASSETS (Prod cd 6998 - 9982)#");
        particularList.add("5~FRAUDS - OTHER (NOT DEBITED TO RA A/c)$");
        particularList.add("6~Frauds reported on or prior to " + quarterDate + " provision @ 100% ##");
        particularList.add("7~Delayed Reported frauds Provision @ 100% ##");
        particularList.add("8~REVENUE ITEM IN SYSTEM SUSPENSE");
        particularList.add("9~PROVISION ON ACCOUNT OF FSLO");
        particularList.add("10~PROVISION ON ACCOUNT OF ENTRIES OUTSTANDING IN ADJUSTING ACCOUNT FOR PREVIOUS QUARTER(S) (i.e. PRIOR TO CURRENT QUARTER)");
        particularList.add("11~PROVISION ON N.P.A. INTEREST FREE STAFF LOANS");
        //particularList.add("12~OTHERS (PLEASE SPECIFY BELOW)");
        return particularList;
    }
}


////////////////////////////////////////////////////////////////////



package com.tcs.dao;

import org.apache.log4j.Logger;
import org.springframework.dao.DataAccessException;
import org.springframework.jdbc.core.JdbcTemplate;
import org.springframework.jdbc.core.ResultSetExtractor;
import org.springframework.stereotype.Repository;
import org.springframework.transaction.PlatformTransactionManager;
import org.springframework.transaction.TransactionDefinition;
import org.springframework.transaction.TransactionStatus;
import org.springframework.transaction.support.DefaultTransactionDefinition;

import javax.sql.DataSource;
import java.math.BigDecimal;
import java.sql.ResultSet;
import java.sql.SQLException;
import java.util.*;

@Repository("RW04Dao")
public class RW04DaoImpl implements RW04Dao{

    static Logger log = Logger.getLogger(RW04DaoImpl.class.getName());

    final DataSource dataSource;
    final PlatformTransactionManager transactionManager;

    public RW04DaoImpl(DataSource dataSource,  PlatformTransactionManager transactionManager) {
        this.dataSource = dataSource;
        this.transactionManager = transactionManager;
    }
    @Override
    public Map<String, Object> getRW04Data(String circleCode, String quarterDate, String reportId) {

        JdbcTemplate jdbcTemplate = new JdbcTemplate(dataSource);

        List list;
        Map<String, Object> resultMap = new HashMap<>();

        String getData = "select BS_RW04_ID, BS_RW04_PARTICULARS, BS_RW04_PY, BS_RW04_WRITE_OFF, BS_RW04_ADDITION, BS_RW04_REDUCTION, BS_RW04_CY, " +
                "BS_RW04_PROVI_RATE, BS_RW04_PROVI_REQ, BS_RW04_SEQ from BS_RW04 where BS_RW04_CIRCLE = ? and BS_RW04_DATE = to_date(?,'dd/MM/yyyy') and RML_ID_FK = ? order by BS_RW04_ID";
        list = jdbcTemplate.query(getData, new Object[]{circleCode, quarterDate, reportId}, new ResultSetExtractor<List<List<String>>>() {
            @Override
            public List<List<String>> extractData(ResultSet resultSet) throws SQLException, DataAccessException {
                List<List<String>> branch1 = new ArrayList<>();
                while (resultSet.next()) {
                    List<String> branchlist = new ArrayList<>();
                    branchlist.add(String.valueOf(resultSet.getInt(1)));
                    branchlist.add(resultSet.getString(2));
                    branchlist.add(String.valueOf(resultSet.getBigDecimal(3)));
                    branchlist.add(String.valueOf(resultSet.getBigDecimal(4)));
                    branchlist.add(String.valueOf(resultSet.getBigDecimal(5)));
                    branchlist.add(String.valueOf(resultSet.getBigDecimal(6)));
                    branchlist.add(String.valueOf(resultSet.getBigDecimal(7)));
                    branchlist.add(String.valueOf(resultSet.getBigDecimal(8)));
                    branchlist.add(String.valueOf(resultSet.getBigDecimal(9)));
                    branchlist.add(String.valueOf(resultSet.getInt(10)));
                    branch1.add(branchlist);
                }
                return branch1;
            }
        });
        log.info("list :: " + list);
        resultMap.put("data", list);

        return resultMap;
    }

    @Override
    public Map<String, Object> insertStaticTable(ArrayList<String> particularsList, String circleCode, String quarterDate, String reportId, List<List<String>> value) {
        int insert = 0;
        int check = 0;

        String insertQuery = "insert into BS_RW04 (BS_RW04_ID,BS_RW04_CIRCLE,BS_RW04_DATE,RML_ID_FK,BS_RW04_PARTICULARS,BS_RW04_PY,BS_RW04_WRITE_OFF," +
                "BS_RW04_ADDITION,BS_RW04_REDUCTION,BS_RW04_CY,BS_RW04_PROVI_RATE,BS_RW04_PROVI_REQ,BS_RW04_SEQ) " +
                "values (?,?,to_date(?,'DD/MM/YYYY'),?,?,?,?,?,?,?,?,?,?)";

        JdbcTemplate jdbcTemplate = new JdbcTemplate(dataSource);

        TransactionDefinition def = new DefaultTransactionDefinition();
        TransactionStatus status = transactionManager.getTransaction(def);

        for(int i = 0; i < particularsList.size(); i++){
            String particulars = particularsList.get(i).split("~")[1];
            int rowId = Integer.parseInt(particularsList.get(i).split("~")[0]);
            List<String> row = value.get(i) != null ? value.get(i) : new ArrayList<>();
            BigDecimal prevProvAmt = row.get(0) != null ? new BigDecimal(row.get(0)) : i == 11 ? null : new BigDecimal("0.00");
            BigDecimal writeOffAmt = row.get(1) != null ? new BigDecimal(row.get(1)) : i == 11 ? null : new BigDecimal("0.00");
            BigDecimal additionAmt = row.get(2) != null ? new BigDecimal(row.get(2)) : i == 11 ? null : new BigDecimal("0.00");
            BigDecimal reductionAmt = row.get(3) != null ? new BigDecimal(row.get(3)) : i == 11 ? null : new BigDecimal("0.00");
            BigDecimal currProvAmt = row.get(4)  != null ? new BigDecimal(row.get(4)) : i == 11 ? null : new BigDecimal("0.00");
            BigDecimal provRate = row.get(5)  != null ? new BigDecimal(row.get(5)) : i == 11 ? null : new BigDecimal("0.00");
            BigDecimal provReqAmt = row.get(6)  != null ? new BigDecimal(row.get(6)) : i == 11 ? null : new BigDecimal("0.00");

            insert = jdbcTemplate.update(insertQuery, rowId, circleCode, quarterDate, reportId, particulars,
                    prevProvAmt, writeOffAmt, additionAmt, reductionAmt, currProvAmt, provRate, provReqAmt, rowId);
            check++;
        }
        Map<String, Object> resultMap = new HashMap<>();
        if (check == 11) {
            resultMap.put("insert", insert);
            transactionManager.commit(status);
        } else {
            resultMap.put("message", "Insert was not complete. Try Again.");
            transactionManager.rollback(status);
        }
        return resultMap;
    }

    @Override
    public Map<String, Object> updateStaticTable(String circleCode, String quarterDate, String reportId, List<List<String>> value) {
        int update = 0;
        int count = 0;

        String updateQuery = "update BS_RW04 set BS_RW04_PY = ?, BS_RW04_WRITE_OFF = ?, BS_RW04_ADDITION = ?, BS_RW04_REDUCTION = ?, BS_RW04_CY = ?, " +
                "BS_RW04_PROVI_RATE = ?, BS_RW04_PROVI_REQ = ? where BS_RW04_CIRCLE = ? and BS_RW04_DATE = to_date(?,'DD/MM/YYYY') and RML_ID_FK = ? and BS_RW04_ID = ?";
        JdbcTemplate jdbcTemplate = new JdbcTemplate(dataSource);

        TransactionDefinition def = new DefaultTransactionDefinition();
        TransactionStatus status = transactionManager.getTransaction(def);

        for(int i = 0; i < value.size(); i++){
            List<String> row = value.get(i);
            BigDecimal prevProvAmt = row.get(0) != null ? new BigDecimal(row.get(0)) : new BigDecimal("0.00");
            BigDecimal writeOffAmt = row.get(1) != null ? new BigDecimal(row.get(1)) : new BigDecimal("0.00");
            BigDecimal additionAmt = row.get(2) != null ? new BigDecimal(row.get(2)) : new BigDecimal("0.00");
            BigDecimal reductionAmt = row.get(3) != null ? new BigDecimal(row.get(3)) : new BigDecimal("0.00");
            BigDecimal currProvAmt = row.get(4)  != null ? new BigDecimal(row.get(4)) : new BigDecimal("0.00");
            BigDecimal provRate = row.get(5)  != null ? new BigDecimal(row.get(5)) : new BigDecimal("0.00");
            BigDecimal provReqAmt = row.get(6)  != null ? new BigDecimal(row.get(6)) : new BigDecimal("0.00");
            int rowId = row.get(7) != null ? Integer.parseInt(row.get(7)) : 0;

            update = jdbcTemplate.update(updateQuery, prevProvAmt, writeOffAmt, additionAmt, reductionAmt, currProvAmt, provRate, provReqAmt,
                    circleCode, quarterDate, reportId, rowId);
            count++;
        }
        Map<String, Object> resultMap = new HashMap<>();
        if (count == 11) {
            resultMap.put("update", update);
            transactionManager.commit(status);
        } else {
            resultMap.put("message", "Update was not complete. Try Again.");
            transactionManager.rollback(status);
        }
        return resultMap;
    }

    public int getRowSeq(){
        int rowSeq;
        String seq = "select BS_RW04_SEQ.nextval from dual";
        JdbcTemplate jdbcTemplate = new JdbcTemplate(dataSource);
        rowSeq = jdbcTemplate.queryForObject(seq, Integer.class);
        return rowSeq;
    }

    @Override
    public Map<String, Object> saveAddRow(String circleCode, String quarterDate, String reportId, List<String> value) {
        int insert;
        int update;

        int rowId = Integer.parseInt(value.get(7));
        
        String insertQuery = "insert into BS_RW04 (BS_RW04_ID,BS_RW04_CIRCLE,BS_RW04_DATE,RML_ID_FK,BS_RW04_PARTICULARS,BS_RW04_PY,BS_RW04_WRITE_OFF, " +
                " BS_RW04_ADDITION,BS_RW04_REDUCTION,BS_RW04_CY,BS_RW04_PROVI_RATE,BS_RW04_PROVI_REQ,BS_RW04_SEQ) " +
                " values (?,?,to_date(?,'DD/MM/YYYY'),?,?,?,?,?,?,?,?,?,?) ";

        String updateQuery = "update BS_RW04 set BS_RW04_ID = ?, BS_RW04_PARTICULARS = ?, BS_RW04_PY = ?, BS_RW04_WRITE_OFF = ?, BS_RW04_ADDITION = ?, BS_RW04_REDUCTION = ?, BS_RW04_CY = ?, " +
                " BS_RW04_PROVI_RATE = ?, BS_RW04_PROVI_REQ = ? where BS_RW04_CIRCLE = ? and BS_RW04_DATE = to_date(?,'DD/MM/YYYY') and RML_ID_FK = ? and BS_RW04_SEQ = ?";

        JdbcTemplate jdbcTemplate = new JdbcTemplate(dataSource);

        TransactionDefinition def = new DefaultTransactionDefinition();
        TransactionStatus status = transactionManager.getTransaction(def);

        String particulars = value.get(0) != null ? value.get(0) : "" ;
        BigDecimal prevProvAmt = value.get(1) != null ? new BigDecimal(value.get(1)) : new BigDecimal("0.00");
        BigDecimal writeOffAmt = value.get(2) != null ? new BigDecimal(value.get(2)) : new BigDecimal("0.00");
        BigDecimal additionAmt = value.get(3) != null ? new BigDecimal(value.get(3)) : new BigDecimal("0.00");
        BigDecimal reductionAmt = value.get(4) != null ? new BigDecimal(value.get(4)) : new BigDecimal("0.00");
        BigDecimal currProvAmt = value.get(5)  != null ? new BigDecimal(value.get(5)) : new BigDecimal("0.00");
        BigDecimal provRate = value.get(6)  != null ? new BigDecimal(value.get(6)) : new BigDecimal("0.00");
        BigDecimal provReqAmt = value.get(7)  != null ? new BigDecimal(value.get(7)) : new BigDecimal("0.00");

        Map<String, Object> resultMap = new HashMap<>();

        if (rowId == 0) {
            rowId = getRowSeq();

            insert = jdbcTemplate.update(insertQuery, rowId, circleCode, quarterDate, reportId, particulars, prevProvAmt, writeOffAmt, additionAmt, reductionAmt, currProvAmt, provRate, provReqAmt, rowId);

            if (insert == 1) {
                resultMap.put("insert", insert);
                transactionManager.commit(status);
            }
            else{
                resultMap.put("message", "Insert was not complete. Try Again.");
                transactionManager.rollback(status);
            }
        } else {
            update = jdbcTemplate.update(updateQuery, rowId, particulars, prevProvAmt, writeOffAmt, additionAmt, reductionAmt, currProvAmt, provRate, provReqAmt, circleCode, quarterDate, reportId, rowId);
            if (update == 1) {
                resultMap.put("update", update);
                transactionManager.commit(status);
            }
            else{
                resultMap.put("message", "Update was not complete. Try Again.");
                transactionManager.rollback(status);
            }
        }
        return resultMap;
    }

    @Override
    public Map<String, Object> deleteRow(int rowId) {
        int delete;

        String deleteQuery = "delete from BS_RW04 where BS_RW04_SEQ = ?";

        JdbcTemplate jdbcTemplate = new JdbcTemplate(dataSource);

        TransactionDefinition def = new DefaultTransactionDefinition();
        TransactionStatus status = transactionManager.getTransaction(def);

        delete = jdbcTemplate.update(deleteQuery, rowId);

        Map<String, Object> resultMap = new HashMap<>();
        if (delete == 1) {
            resultMap.put("delete", delete);
            transactionManager.commit(status);
        }
        else {
            resultMap.put("message", "Delete was not complete. Try Again.");
            transactionManager.rollback(status);
        }
        return resultMap;
    }
}

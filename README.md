package com.tcs.controllers;

import com.tcs.services.RW05Service;
import org.apache.log4j.Logger;
import org.springframework.web.bind.annotation.*;

import java.util.Map;

@RestController
@RequestMapping("/RW05")
public class RW05Controller {

    final RW05Service rw05Service;

    public RW05Controller (RW05Service rw05Service){
        this.rw05Service = rw05Service;
    }

    static Logger log = Logger.getLogger(RW04Controller.class.getName());

    @RequestMapping(value = "/getData", method = RequestMethod.POST)
    public @ResponseBody Map<String, Object> getRW04Data (@RequestBody Map<String, Object> payload){
        Map<String, Object> result;
        result = rw05Service.getRW05Data(payload);
        return result;
    }

    @RequestMapping("/saveStatic")
    public @ResponseBody Map<String, Object> saveStaticData (@RequestBody Map<String, Object> payload){
        Map<String, Object> result;
        result = rw05Service.saveStaticTable(payload);
        return result;
    }

    @RequestMapping("/saveAddRow")
    public @ResponseBody Map<String, Object> saveAddRow (@RequestBody Map<String, Object> payload){
        Map<String, Object> result;
        result = rw05Service.saveAddRow(payload);
        return result;
    }

    @RequestMapping("/deleteRow")
    public @ResponseBody Map<String, Object> deleteRow (@RequestBody Map<String, Object> payload){
        Map<String, Object> result;
        result = rw05Service.deleteRow(payload);
        return result;
    }

}


//////////////////////////////////////////////////////

package com.tcs.services;

import com.tcs.dao.RW05Dao;
import org.apache.log4j.Logger;
import org.springframework.stereotype.Service;

import java.util.*;

@Service
public class RW05ServiceImpl implements RW05Service {

    static Logger log = Logger.getLogger(RW05ServiceImpl.class.getName());

    final RW05Dao rw05Dao;

    public RW05ServiceImpl(RW05Dao rw05Dao) {
        this.rw05Dao = rw05Dao;
    }

    @Override
    public Map<String, Object> getRW05Data(Map<String, Object> payload) {
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
                    data = rw05Dao.getRW05Data(circleCode, quarterDate, reportId);
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
                    ArrayList<String> particularsList = getParticularList();
                    insert = rw05Dao.insertStaticTable(particularsList, circleCode, quarterDate, reportId, value);
                    return insert;
                } else if (status.equalsIgnoreCase("11") || status.equalsIgnoreCase("12")) {
                    Map<String, Object> update;
                    update = rw05Dao.updateStaticTable(circleCode, quarterDate, reportId, value);
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
                insert = rw05Dao.saveAddRow(circleCode, quarterDate, reportId, value);
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
            delete = rw05Dao.deleteRow(rowId);
            return delete;
        }
        else{
            String message = "Unauthorized user";
            return new HashMap<>(Collections.singletonMap("retVal", message));
        }
    }

    private ArrayList<String> getParticularList() {
        ArrayList<String> particularList = new ArrayList<>();
        particularList.add("1~CONTINGENT LIABILITY AS PER AS -29");
        particularList.add("2~DELAYED REPORTING PENALTY");
        particularList.add("3~EX-GRATIA PAYMENT");
        particularList.add("4~PROVISION ON OVERDUE DEPOSIT INTT");
        particularList.add("5~LEAVE ENCASHMENT");
        particularList.add("6~PROVISION FOR PERFORMANCE LINKED INCENTIVES");
        particularList.add("7~PROVISION ON ACCOUNT OF  ENTRIES OUTSTANDING IN ADJUSTING ACCOUNT FOR PREVIOUS QUARTER(S) (i.e. PRIOR TO CURRENT QUARTER)");
        //particularList.add("8~OTHERS (PLEASE SPECIFY BELOW)");
        return particularList;
    }
}
/////////////////////////////////////////////////////////////////



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

@Repository("RW05Dao")
public class RW05DaoImpl implements RW05Dao{

    static Logger log = Logger.getLogger(RW05DaoImpl.class.getName());

    final DataSource dataSource;
    final PlatformTransactionManager transactionManager;

    public RW05DaoImpl(DataSource dataSource,  PlatformTransactionManager transactionManager) {
        this.dataSource = dataSource;
        this.transactionManager = transactionManager;
    }
    @Override
    public Map<String, Object> getRW05Data(String circleCode, String quarterDate, String reportId) {

        JdbcTemplate jdbcTemplate = new JdbcTemplate(dataSource);

        List list;
        Map<String, Object> resultMap = new HashMap<>();

        String getData = "select BS_RW05_ID, BS_RW05_PARTICULARS, BS_RW05_PY, BS_RW05_ADDITION, BS_RW05_REVERSAL, BS_RW05_CY, " +
                "BS_RW05_DIFF from BS_RW05 where BS_RW05_CIRCLE = ? and BS_RW05_DATE = to_date(?,'dd/MM/yyyy') and RML_ID_FK = ? order by BS_RW05_ID";
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

        String insertQuery = "insert into BS_RW05 (BS_RW05_ID, BS_RW05_CIRCLE, BS_RW05_DATE, RML_ID_FK, BS_RW05_PARTICULARS, BS_RW05_PY, " +
                "BS_RW05_ADDITION, BS_RW05_REVERSAL, BS_RW05_CY, BS_RW05_DIFF) " +
                "values (?,?,to_date(?,'DD/MM/YYYY'),?,?,?,?,?,?,?)";

        JdbcTemplate jdbcTemplate = new JdbcTemplate(dataSource);

        TransactionDefinition def = new DefaultTransactionDefinition();
        TransactionStatus status = transactionManager.getTransaction(def);

        for(int i = 0; i < particularsList.size(); i++){
            String particulars = particularsList.get(i).split("~")[1];
            int rowId = Integer.parseInt(particularsList.get(i).split("~")[0]);
            List<String> row = value.get(i) != null ? value.get(i) : new ArrayList<>();
            BigDecimal prevProvAmt = row.get(0) != null ? new BigDecimal(row.get(0)) : i == 11 ? null : new BigDecimal("0.00");
            BigDecimal additionAmt = row.get(1) != null ? new BigDecimal(row.get(1)) : i == 11 ? null : new BigDecimal("0.00");
            BigDecimal reversalAmt = row.get(2) != null ? new BigDecimal(row.get(2)) : i == 11 ? null : new BigDecimal("0.00");
            BigDecimal currProvAmt = row.get(3)  != null ? new BigDecimal(row.get(3)) : i == 11 ? null : new BigDecimal("0.00");
            BigDecimal diffProvAmt = row.get(4)  != null ? new BigDecimal(row.get(4)) : i == 11 ? null : new BigDecimal("0.00");

            insert = jdbcTemplate.update(insertQuery, rowId, circleCode, quarterDate, reportId, particulars,
                    prevProvAmt, additionAmt, reversalAmt, currProvAmt, diffProvAmt);
            check++;
        }
        Map<String, Object> resultMap = new HashMap<>();
        if (check == 7) {
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

        String updateQuery = "update BS_RW05 set BS_RW05_PY = ?, BS_RW05_ADDITION = ?, BS_RW05_REVERSAL = ?, BS_RW05_CY = ?, " +
                "BS_RW05_DIFF = ? where BS_RW05_CIRCLE = ? and BS_RW05_DATE = to_date(?,'DD/MM/YYYY') and RML_ID_FK = ? and BS_RW05_ID = ?";
        JdbcTemplate jdbcTemplate = new JdbcTemplate(dataSource);

        TransactionDefinition def = new DefaultTransactionDefinition();
        TransactionStatus status = transactionManager.getTransaction(def);

        for(int i = 0; i < value.size(); i++){
            List<String> row = value.get(i);
            BigDecimal prevProvAmt = row.get(0) != null ? new BigDecimal(row.get(0)) : i == 11 ? null : new BigDecimal("0.00");
            BigDecimal additionAmt = row.get(1) != null ? new BigDecimal(row.get(1)) : i == 11 ? null : new BigDecimal("0.00");
            BigDecimal reversalAmt = row.get(2) != null ? new BigDecimal(row.get(2)) : i == 11 ? null : new BigDecimal("0.00");
            BigDecimal currProvAmt = row.get(3)  != null ? new BigDecimal(row.get(3)) : i == 11 ? null : new BigDecimal("0.00");
            BigDecimal diffProvAmt = row.get(4)  != null ? new BigDecimal(row.get(4)) : i == 11 ? null : new BigDecimal("0.00");
            int rowId = row.get(5) != null ? Integer.parseInt(row.get(5)) : 0;

            update = jdbcTemplate.update(updateQuery, prevProvAmt, additionAmt, reversalAmt, currProvAmt, diffProvAmt, circleCode, quarterDate, reportId, rowId);
            count++;
        }
        Map<String, Object> resultMap = new HashMap<>();
        if (count == 7) {
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
        String seq = "select BS_RW05_SEQ.nextval from dual";
        JdbcTemplate jdbcTemplate = new JdbcTemplate(dataSource);
        rowSeq = jdbcTemplate.queryForObject(seq, Integer.class);
        return rowSeq;
    }

    @Override
    public Map<String, Object> saveAddRow(String circleCode, String quarterDate, String reportId, List<String> value) {
        int insert;
        int update;

        int rowId = Integer.parseInt(value.get(6));
        
        String insertQuery = "insert into BS_RW05 (BS_RW05_ID,BS_RW05_CIRCLE,BS_RW05_DATE,RML_ID_FK,BS_RW05_PARTICULARS,BS_RW05_PY, " +
                " BS_RW05_ADDITION,BS_RW05_REVERSAL,BS_RW05_CY,BS_RW05_DIFF) " +
                " values (?,?,to_date(?,'DD/MM/YYYY'),?,?,?,?,?,?,?) ";

        String updateQuery = "update BS_RW05 set BS_RW05_ID = ?, BS_RW05_PARTICULARS = ?, BS_RW05_PY = ?, BS_RW05_ADDITION = ?, BS_RW05_REVERSAL = ?, BS_RW05_CY = ?, " +
                " BS_RW05_DIFF = ? where BS_RW05_CIRCLE = ? and BS_RW05_DATE = to_date(?,'DD/MM/YYYY') and RML_ID_FK = ? and BS_RW05_ID = ?";

        JdbcTemplate jdbcTemplate = new JdbcTemplate(dataSource);

        TransactionDefinition def = new DefaultTransactionDefinition();
        TransactionStatus status = transactionManager.getTransaction(def);

        String particulars = value.get(0) != null ? value.get(0) : "" ;
        BigDecimal prevProvAmt =  new BigDecimal(value.get(1));
        BigDecimal additionAmt =  new BigDecimal(value.get(2));
        BigDecimal reversalAmt =  new BigDecimal(value.get(3));
        BigDecimal currProvAmt =  new BigDecimal(value.get(4));
        BigDecimal diffProvAmt =  new BigDecimal(value.get(5));

        Map<String, Object> resultMap = new HashMap<>();

        if (rowId == 0) {
            rowId = getRowSeq();

            insert = jdbcTemplate.update(insertQuery, rowId, circleCode, quarterDate, reportId, particulars, prevProvAmt, additionAmt, reversalAmt, currProvAmt, diffProvAmt, rowId);

            if (insert == 1) {
                resultMap.put("insert", insert);
                resultMap.put("rowId", rowId);
                transactionManager.commit(status);
            }
            else{
                resultMap.put("message", "Insert was not complete. Try Again.");
                transactionManager.rollback(status);
            }
        } else {
            update = jdbcTemplate.update(updateQuery, rowId, particulars, prevProvAmt, additionAmt, reversalAmt, currProvAmt, diffProvAmt, circleCode, quarterDate, reportId, rowId);
            if (update == 1) {
                resultMap.put("update", update);
                resultMap.put("rowId", rowId);
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

        String deleteQuery = "delete from BS_RW05 where BS_RW05_ID = ?";

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

//////////////////////////////////////////////////////////



import React, { useState } from 'react';
import {
  Tabs,
  Tab,
  Box,
  Typography,
  Table,
  TableHead,
  TableRow,
  TableCell,
  TableBody,
  TableContainer,
  Paper,
  Button,
  Snackbar,
  Alert,
  Checkbox,
} from '@mui/material';
import { styled } from '@mui/material/styles';
import FormInput from '../../../../common/components/ui/FormInput';
import useCustomSnackbar from '../../../../common/hooks/useCustomSnackbar';
import { useLocation } from 'react-router-dom';
import useApi from '../../../../common/hooks/useApi';

const StyledTableCell = styled(TableCell)(({ theme }) => ({
  fontSize: '0.875rem',
  textAlign: 'center',
  padding: '6px',
  fontWeight: 'bold',
}));

const initialDynamicRow = {
  selected: false,
  particulars: '',
  provAmtStart: '',
  addition: '',
  reversal: '',
  provAmtEnd: '',
  difference: '',
};

const initialStaticRows = [
  { id: '1', label: 'Contingent liability As per As-29' },
  { id: '2', label: 'Delayed reporting penalty' },
  { id: '3', label: 'Ex-Gratia payment' },
  { id: '4', label: 'Provision on overdue deposit intt' },
  { id: '5', label: 'Leave encashment' },
  { id: '6', label: 'Provision for performance linked incentives' },
  {
    id: '7',
    label:
      'Provision on account of entries outstanding in adjusting account for previous quarter(s) (i.e. prior to current quarter)',
  },
];

const RW05 = () => {
  const [tabIndex, setTabIndex] = useState(0);
  const user = JSON.parse(localStorage.getItem('user'));
  console.log('user', user);
  const [dynamicRows, setDynamicRows] = useState([{ ...initialDynamicRow }]);
  const [staticData, setStaticData] = useState(
    Object.fromEntries(
      initialStaticRows.flatMap((r) =>
        ['provAmtStart', 'addition', 'reversal', 'provAmtEnd', 'difference'].map((key) => [`${r.id}_${key}`, ''])
      )
    )
  );
  const { callApi } = useApi();
  const setSnackbarMessage = useCustomSnackbar();
  const [isLoading, setIsLoading] = useState(false);
  const { state } = useLocation();
  const [reportObject, setReportObject] = useState(state?.report || null);

  console.log('reportObject', reportObject);

  const headersStatic = [
    'Particulars(2)',
    'Opening Provision as on 01-04-2023(3)',
    'Additions during the period(4)',
    'Reversals during the Period(5)',
    'Provision as on 30/06/2024(6)={(3+4)-(5)}',
    'Difference to be Provided/ written back(7)={(6)-(3)}',
  ];

  const headersDynamic = [
    'Select',
    'Particulars(2)',
    'Opening Provision as on 01-04-2023(3)',
    'Additions during the period(4)',
    'Reversals during the Period(5)',
    'Provision as on 30/06/2024(6)={(3+4)-(5)}',
    'Difference to be Provided/ written back(7)={(6)-(3)}',
  ];

  const isNumeric = (val) => !isNaN(parseFloat(val)) && isFinite(val);

  const calculateAndSetStatic = (rowId, updated) => {
    const provAmtStart = parseFloat(updated[`${rowId}_provAmtStart`] || 0);
    const addition = parseFloat(updated[`${rowId}_addition`] || 0);
    const reversal = parseFloat(updated[`${rowId}_reversal`] || 0);

    // Provision as on 30/06/2024(6)={(3+4)-(5)}
    const provAmtEnd = provAmtStart + addition - reversal;
    updated[`${rowId}_provAmtEnd`] = provAmtEnd.toFixed(2);
    // Difference to be Provided/ written back(7)={(6)-(3)}
    const difference = provAmtEnd - provAmtStart;
    updated[`${rowId}_difference`] = difference.toFixed(2);
  };

  const handleStaticChange = (rowId, key, value) => {
    const updated = { ...staticData, [`${rowId}_${key}`]: value };
    calculateAndSetStatic(rowId, updated);
    setStaticData(updated);
  };

  const handleDynamicChange = (i, key, value) => {
    const updated = [...dynamicRows];
    // Only update if the value is numeric or empty for numeric fields
    const newValue =
      ['provAmtStart', 'addition', 'reversal'].includes(key) && !isNumeric(value) && value !== ''
        ? updated[i][key]
        : value;

    updated[i][key] = newValue;

    const provAmtStart = parseFloat(updated[i].provAmtStart || 0);
    const addition = parseFloat(updated[i].addition || 0);
    const reversal = parseFloat(updated[i].reversal || 0);

    // Provision as on 30/06/2024(6)={(3+4)-(5)}
    const provAmtEnd = provAmtStart + addition - reversal;
    updated[i].provAmtEnd = provAmtEnd.toFixed(2);

    // Difference to be Provided/ written back(7)={(6)-(3)}
    const difference = provAmtEnd - provAmtStart;
    updated[i].difference = difference.toFixed(2);

    setDynamicRows(updated);
  };

  const handleSubmit = (action = 'save') => {
    setIsLoading(true);
    let payload = {};

    if (tabIndex === 0) {
      // Payload for Tab 1 (RW-05-I) as per your sample
      payload = {
        value: initialStaticRows.map((row) => [
          ' ',
          row.label,
          staticData[`${row.id}_provAmtStart`] || '0',
          staticData[`${row.id}_addition`] || '0',
          staticData[`${row.id}_reversal`] || '0',
          parseFloat(staticData[`${row.id}_provAmtEnd`] || 0).toFixed(2),
          parseFloat(staticData[`${row.id}_difference`] || 0).toFixed(2),
          row.id,
          row.id,
          'true',
        ]),
        tabValue: '1',
        tabName: 'RW-05-I',
        reportId: '3009',
        submissionId: 5262,
        currentStatus: '11',
      };
    } else if (tabIndex === 1) {
      const firstDynamicRow = dynamicRows[0] || initialDynamicRow;
      payload = {
        value: [
          firstDynamicRow.particulars || 'ewewe',
          firstDynamicRow.provAmtStart || '1',
          firstDynamicRow.addition || '1',
          firstDynamicRow.reversal || '1',
          parseFloat(firstDynamicRow.provAmtEnd || 0).toFixed(2),
          parseFloat(firstDynamicRow.difference || 0).toFixed(2),
          '208',
          '208',
          'true',
        ],
        tabValue: '2',
        reportId: '3009',
        submissionId: 5262,
        currentStatus: '11',
      };
    }

    console.log(`Payload for Tab ${tabIndex + 1} (${action}):`, JSON.stringify(payload, null, 2));
    setSnackbarMessage(
      `Data for Tab ${tabIndex === 0 ? 'RW-05(I)' : 'RW-05(II)'} ${
        action === 'submit' ? 'submitted' : 'saved'
      } successfully! Check console for payload.`,
      'success'
    );
    setIsLoading(false);
  };

  const renderHeader = (headers) => (
    <TableHead>
      <TableRow>
        <TableCell sx={{ backgroundColor: 'hsl(220, 20%, 35%)', fontWeight: 'bold' }}>Sr No(1)</TableCell>
        {headers.map((head, idx) => (
          <StyledTableCell key={idx}>{head}</StyledTableCell>
        ))}
      </TableRow>
    </TableHead>
  );

  return (
    <Box>
      <Tabs value={tabIndex} onChange={(e, i) => setTabIndex(i)}>
        <Tab label="RW-05(I)" />
        <Tab label="RW-05(II)" />
      </Tabs>

      {tabIndex === 0 && (
        <>
          <Box mt={2} display="flex" gap={2}>
            <Button variant="contained" color="warning" onClick={() => handleSubmit()} disabled={isLoading}>
              Save
            </Button>
            <Button variant="contained" color="success" onClick={() => handleSubmit('submit')} disabled={isLoading}>
              Submit
            </Button>
          </Box>
          <TableContainer component={Paper} sx={{ mt: 2 }}>
            <Table>
              {renderHeader(headersStatic)}
              <TableBody>
                {initialStaticRows.map((row) => (
                  <TableRow key={row.id}>
                    <TableCell align="center">{row.id}</TableCell>
                    <TableCell align="center">{row.label}</TableCell>
                    {['provAmtStart', 'addition', 'reversal'].map((key) => (
                      <TableCell key={key} align="center">
                        <FormInput
                          value={staticData[`${row.id}_${key}`]}
                          onChange={(e) => handleStaticChange(row.id, key, e.target.value)}
                          readOnly={row.id.includes('1')}
                          debounceDuration={1}
                          sx={{ width: '180px' }}
                          placeholder="0.00"
                        />
                      </TableCell>
                    ))}
                    <TableCell align="center">
                      <FormInput
                        value={staticData[`${row.id}_provAmtEnd`]}
                        readOnly={true}
                        sx={{ width: '180px' }}
                        placeholder="0.00"
                      />
                    </TableCell>
                    <TableCell align="center">
                      <FormInput
                        value={staticData[`${row.id}_difference`]}
                        readOnly={true}
                        sx={{ width: '180px' }}
                        placeholder="0.00"
                      />
                    </TableCell>
                  </TableRow>
                ))}
              </TableBody>
            </Table>
          </TableContainer>
        </>
      )}

      {tabIndex === 1 && (
        <>
          <Box mt={2} display="flex" gap={2}>
            <Button
              variant="contained"
              color="secondary"
              onClick={() => setDynamicRows([...dynamicRows, { ...initialDynamicRow }])}
            >
              Add Row
            </Button>
            <Button
              variant="contained"
              color="error"
              onClick={() => setDynamicRows(dynamicRows.filter((r) => !r.selected))}
            >
              Delete Row
            </Button>
            <Button variant="contained" color="warning" onClick={() => handleSubmit()} disabled={isLoading}>
              Save
            </Button>
            <Button variant="contained" color="success" onClick={() => handleSubmit('submit')} disabled={isLoading}>
              Submit
            </Button>
          </Box>
          <TableContainer component={Paper} sx={{ mt: 2, maxHeight: 'calc(100vh - 250px)' }}>
            <Table stickyHeader>
              {renderHeader(headersDynamic)}
              <TableBody>
                {dynamicRows.map((row, i) => (
                  <TableRow key={i}>
                    <TableCell>{i + 1}</TableCell>
                    <TableCell padding="checkbox">
                      <Checkbox
                        checked={row.selected}
                        onChange={() => {
                          const updated = [...dynamicRows];
                          updated[i].selected = !updated[i].selected;
                          setDynamicRows(updated);
                        }}
                      />
                    </TableCell>
                    <TableCell>
                      <FormInput
                        value={row.particulars}
                        inputType={'alphaNumericWithSpace'}
                        onChange={(e) => {
                          const updated = [...dynamicRows];
                          updated[i].particulars = e.target.value;
                          setDynamicRows(updated);
                        }}
                        sx={{ width: '180px' }}
                        placeholder="Enter Text"
                      />
                    </TableCell>
                    {['provAmtStart', 'addition', 'reversal'].map((key) => (
                      <TableCell key={key} align="center">
                        <FormInput
                          value={row[key]}
                          onChange={(e) => handleDynamicChange(i, key, e.target.value)}
                          debounceDuration={1}
                          sx={{ width: '180px' }}
                          placeholder="0.00"
                        />
                      </TableCell>
                    ))}
                    <TableCell align="center">
                      <FormInput value={row.provAmtEnd} readOnly={true} sx={{ width: '180px' }} placeholder="0.00" />
                    </TableCell>
                    <TableCell align="center">
                      <FormInput value={row.difference} readOnly={true} sx={{ width: '180px' }} placeholder="0.00" />
                    </TableCell>
                  </TableRow>
                ))}
              </TableBody>
            </Table>
          </TableContainer>
        </>
      )}
    </Box>
  );
};

export default RW05;


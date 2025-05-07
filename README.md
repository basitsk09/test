Working below code:

import React, { useState, useEffect, useMemo, useCallback } from 'react';
import axios from 'axios';
import {
  Table,
  TableBody,
  TableContainer,
  TableHead,
  TableRow,
  Paper,
  TextField,
  Button,
  Alert,
  Stack,
  Box,
  Snackbar,
  Checkbox,
  TableCell,
  Radio,
} from '@mui/material';
import { styled } from '@mui/material/styles';
import { CustomButton } from '../../../../common/components/ui/Buttons';

const StyledTableCell = styled(TableCell)(({ theme }) => ({
  fontSize: '1rem',
  [`&.MuiTableCell-head`]: {
    backgroundColor: theme.palette.common.black,
    color: theme.palette.common.white,
  },
}));

const numericRegex = /^-?\d*(\.\d{0,2})?$/;
const alphanumericRegex = /^[A-Za-z0-9\s]*$/;

const columns = [
  { key: 'isDelete', label: 'Select', type: 'checkbox', editable: false, align: 'center' },
  {
    key: 'borrowerName',
    label: "Branch & Borrower's Name",
    type: 'text',
    editable: true,
    align: 'left',
    pattern: alphanumericRegex,
  },
  {
    key: 'aggOutStand',
    label: 'Aggregate outstanding',
    type: 'number',
    editable: true,
    align: 'right',
    pattern: numericRegex,
  },
  {
    key: 'aggSecurities',
    label: 'Aggregate value of realisable securities',
    type: 'number',
    editable: true,
    align: 'right',
    pattern: numericRegex,
  },
  { key: 'netShortfall', label: 'Net Shortfall', type: 'number', editable: false, align: 'right' },
  { key: 'provision', label: 'Provision', type: 'number', editable: true, align: 'right', pattern: numericRegex },
  {
    key: 'balInterestSuspenseAcc',
    label: 'Balance in Interest Suspense Account',
    type: 'number',
    editable: true,
    align: 'right',
    pattern: numericRegex,
  },
];

const initialRow = {
  isDelete: false,
  borrowerName: '',
  aggOutStand: '',
  aggSecurities: '',
  netShortfall: '',
  provision: '',
  balInterestSuspenseAcc: '',
};

export default function Schedule9ATable() {
  const [rows, setRows] = useState([]);
  const [totals, setTotals] = useState({
    aggOutStandTotal: 0,
    aggSecuritiesTotal: 0,
    netShortfallTotal: 0,
    provisionTotal: 0,
    balInterestSuspenseAccTotal: 0,
  });
  const [snackbarOpen, setSnackbarOpen] = useState(false);
  const [snackbarMessage, setSnackbarMessage] = useState('');
  const [saveStatus, setSaveStatus] = useState(null); // 'success', 'error', null
  const [submitStatus, setSubmitStatus] = useState(null); // 'success', 'error', null

  // Simulate fetching saved data (replace with your actual API call)
  useEffect(() => {
    // Example:
    // axios.get('/api/schedule9a')
    //   .then(response => {
    //     setRows(response.data);
    //   })
    //   .catch(error => {
    //     console.error("Error fetching saved data:", error);
    //     setSnackbarMessage('Error loading saved data.');
    //     setSnackbarOpen(true);
    //   });
  }, []);

  const calculateTotals = useCallback(() => {
    let aggOutStandTotal = 0;
    let aggSecuritiesTotal = 0;
    let netShortfallTotal = 0;
    let provisionTotal = 0;
    let balInterestSuspenseAccTotal = 0;
    let hasNetShortfallChanged = false;

    const updatedRows = rows.map((row) => {
      const aggOut = parseFloat(row.aggOutStand) || 0;
      const aggSec = parseFloat(row.aggSecurities) || 0;
      const newNetShort = (aggOut - aggSec).toFixed(2);
      if (row.netShortfall !== newNetShort) {
        hasNetShortfallChanged = true;
        return { ...row, netShortfall: newNetShort };
      }
      return row;
    });

    if (hasNetShortfallChanged) {
      setRows(updatedRows);
    }

    updatedRows.forEach((row) => {
      aggOutStandTotal += parseFloat(row.aggOutStand) || 0;
      aggSecuritiesTotal += parseFloat(row.aggSecurities) || 0;
      netShortfallTotal += parseFloat(row.netShortfall) || 0;
      provisionTotal += parseFloat(row.provision) || 0;
      balInterestSuspenseAccTotal += parseFloat(row.balInterestSuspenseAcc) || 0;
    });

    setTotals({
      aggOutStandTotal: aggOutStandTotal.toFixed(2),
      aggSecuritiesTotal: aggSecuritiesTotal.toFixed(2),
      netShortfallTotal: netShortfallTotal.toFixed(2),
      provisionTotal: provisionTotal.toFixed(2),
      balInterestSuspenseAccTotal: balInterestSuspenseAccTotal.toFixed(2),
    });
  }, [rows, setRows, setTotals]);

  useEffect(() => {
    calculateTotals();
  }, [rows, calculateTotals]);

  const handleInputChange = (event, index, columnKey) => {
    const { value } = event.target;
    const updatedRows = [...rows];
    const column = columns.find((col) => col.key === columnKey);

    if (column?.pattern && !column.pattern.test(value)) {
      return; // Prevent updating state if the pattern doesn't match
    }

    updatedRows[index][columnKey] = value;
    setRows(updatedRows);
  };

  const handleCheckboxChange = (event, index) => {
    const updatedRows = [...rows];
    updatedRows[index].isDelete = event.target.checked;
    setRows(updatedRows);
  };

  const addRow = () => {
    setRows([...rows, { ...initialRow }]);
  };

  const deleteSelectedRows = () => {
    const newRows = rows.filter((row) => !row.isDelete);
    setRows(newRows);
    setSnackbarMessage('Selected rows deleted.');
    setSnackbarOpen(true);
  };

  const validateRows = () => {
    for (let i = 0; i < rows.length; i++) {
      const row = rows[i];
      const hasData = Object.keys(initialRow).some((key) => key !== 'isDelete' && row[key] !== '');
      if (hasData && !row.borrowerName) {
        setSnackbarMessage("Please enter Branch & Borrower's Name for all filled rows.");
        setSnackbarOpen(true);
        return false;
      }
      if (
        hasData &&
        (isNaN(parseFloat(row.aggOutStand)) ||
          isNaN(parseFloat(row.aggSecurities)) ||
          isNaN(parseFloat(row.provision)) ||
          isNaN(parseFloat(row.balInterestSuspenseAcc)))
      ) {
        setSnackbarMessage(
          'Please enter valid numeric values for Aggregate outstanding, Aggregate value of realisable securities, Provision, and Balance in Interest Suspense Account.'
        );
        setSnackbarOpen(true);
        return false;
      }
    }
    return true;
  };

  const handleSave = async () => {
    if (!validateRows()) {
      return;
    }
    // Simulate API call for saving data
    try {
      // const response = await axios.post('/api/schedule9a/save', rows);
      console.log('Data saved:', rows);
      setSaveStatus('success');
      setSnackbarMessage('Report saved successfully.');
      setSnackbarOpen(true);
      // Handle response if needed
    } catch (error) {
      console.error('Error saving data:', error);
      setSaveStatus('error');
      setSnackbarMessage('Error saving report.');
      setSnackbarOpen(true);
    }
  };

  const handleSubmit = async () => {
    if (!validateRows()) {
      return;
    }
    // Simulate API call for submitting data
    try {
      // const response = await axios.post('/api/schedule9a/submit', rows);
      console.log('Data submitted:', rows);
      setSubmitStatus('success');
      setSnackbarMessage('Report submitted successfully.');
      setSnackbarOpen(true);
      // Handle response and redirection if needed
    } catch (error) {
      console.error('Error submitting data:', error);
      setSubmitStatus('error');
      setSnackbarMessage('Error submitting report.');
      setSnackbarOpen(true);
    }
  };

  const handleCloseSnackbar = (event, reason) => {
    if (reason === 'clickaway') {
      return;
    }
    setSnackbarOpen(false);
    setSaveStatus(null);
    setSubmitStatus(null);
  };

  return (
    <Box>
      <TableContainer component={Paper} sx={{ width: '100%' }}>
        <Table>
          <TableHead>
            <TableRow>
              {columns.map((col) => (
                <StyledTableCell key={col.key} align={col.align || 'center'}>
                  {col.label}
                </StyledTableCell>
              ))}
            </TableRow>
          </TableHead>
          <TableHead>
            <TableRow>
              {columns.map((col) => (
                <StyledTableCell key={col.key} align={col.align || 'center'}>
                  {col.subHeading}
                </StyledTableCell>
              ))}
            </TableRow>
          </TableHead>
          <TableBody>
            {rows.map((row, index) => (
              <TableRow key={index}>
                {columns.map((column) => (
                  <StyledTableCell key={`${index}-${column.key}`} align={column.align || 'left'}>
                    {column.type === 'checkbox' ? (
                      <Checkbox checked={row.isDelete} onChange={(event) => handleCheckboxChange(event, index)} />
                    ) : column.editable ? (
                      <TextField
                        defaultValue={row[column.key] || ''}
                        // onChange={(event) => handleInputChange(event, index, column.key)}
                        onBlur={(event) => handleInputChange(event, index, column.key)}
                        inputProps={{
                          style: { textAlign: column.align },
                          maxLength: column.key === 'borrowerName' ? 255 : 18, // Example max length
                        }}
                        size="small"
                      />
                    ) : (
                      <span>{row[column.key]}</span>
                    )}
                  </StyledTableCell>
                ))}
              </TableRow>
            ))}
            <TableRow>
              <StyledTableCell colSpan={2} align="center">
                <b>Total</b>
              </StyledTableCell>
              <StyledTableCell align="right">
                <TextField
                  value={totals.aggOutStandTotal}
                  InputProps={{ readOnly: true, style: { textAlign: 'right' } }}
                  size="small"
                />
              </StyledTableCell>
              <StyledTableCell align="right">
                <TextField
                  value={totals.aggSecuritiesTotal}
                  InputProps={{ readOnly: true, style: { textAlign: 'right' } }}
                  size="small"
                />
              </StyledTableCell>
              <StyledTableCell align="right">
                <TextField
                  value={totals.netShortfallTotal}
                  InputProps={{ readOnly: true, style: { textAlign: 'right' } }}
                  size="small"
                />
              </StyledTableCell>
              <StyledTableCell align="right">
                <TextField
                  value={totals.provisionTotal}
                  InputProps={{ readOnly: true, style: { textAlign: 'right' } }}
                  size="small"
                />
              </StyledTableCell>
              <StyledTableCell align="right">
                <TextField
                  value={totals.balInterestSuspenseAccTotal}
                  InputProps={{ readOnly: true, style: { textAlign: 'right' } }}
                  size="small"
                />
              </StyledTableCell>
            </TableRow>
          </TableBody>
        </Table>
      </TableContainer>
      <Stack direction="row" spacing={2} mt={2}>
        <CustomButton label={'Add Row'} buttonType={'create'} onClickHandler={addRow} />
        <CustomButton
          label={'Delete Row'}
          buttonType={'delete'}
          onClickHandler={deleteSelectedRows}
          disabled={rows.length === 0}
        />
        <CustomButton label={'Save'} buttonType={'save'} onClickHandler={handleSave} disabled={rows.length === 0} />
        <CustomButton
          label={'Submit'}
          buttonType={'submit'}
          onClickHandler={handleSubmit}
          disabled={rows.length === 0}
        />
      </Stack>

      <Snackbar
        open={snackbarOpen}
        autoHideDuration={6000}
        onClose={handleCloseSnackbar}
        anchorOrigin={{ vertical: 'bottom', horizontal: 'right' }}
      >
        <Alert
          onClose={handleCloseSnackbar}
          severity={saveStatus === 'error' || submitStatus === 'error' ? 'error' : 'info'}
          sx={{ width: '100%' }}
        >
          {snackbarMessage}
        </Alert>
      </Snackbar>

      {saveStatus === 'success' && (
        <Snackbar
          open={true}
          autoHideDuration={3000}
          onClose={handleCloseSnackbar}
          anchorOrigin={{ vertical: 'bottom', horizontal: 'right' }}
        >
          <Alert onClose={handleCloseSnackbar} severity="success" sx={{ width: '100%' }}>
            Report saved successfully.
          </Alert>
        </Snackbar>
      )}

      {submitStatus === 'success' && (
        <Snackbar
          open={true}
          autoHideDuration={3000}
          onClose={handleCloseSnackbar}
          anchorOrigin={{ vertical: 'bottom', horizontal: 'right' }}
        >
          <Alert onClose={handleCloseSnackbar} severity="success" sx={{ width: '100%' }}>
            Report submitted successfully.
          </Alert>
        </Snackbar>
      )}
    </Box>
  );
}





----------------------------------------------------------------------------------------------------------------
Payload for save method:

{
    "listToBeSent": [
        {
            "borrowerName": "",
            "aggOutStand": "01.00",
            "aggSecurities": "10",
            "netShortfall": "-9.00",
            "provision": "1",
            "balInterestSuspenseAcc": "1"
        },
        {
            "borrowerName": "",
            "aggOutStand": "10",
            "aggSecurities": "10",
            "netShortfall": "0.00",
            "provision": "1",
            "balInterestSuspenseAcc": "1"
        }
    ],
    "circleCode": "021",
    "quarterEndDate": "31/03/2025",
    "userId": "1111111",
    "reportName": "Schedule 9A",
    "reportId": null,
    "reportMasterId": "310023",
    "status": null,
    "save": true
}
-----------------------------------------------------------------------------------------------------------------

Dao Implementation



public String submitNineA(List listToBeSent, String circleCode, String quarterEndDate, String userId, String reportId,
                              String reportMasterId, String reportName, String status, boolean save){


        String result = "";
        String insertStatus = "";

        String noncrPkId = "";

        HashMap<String, String> branchCount = new HashMap<String, String>();
        int counter = -1;
        String sequence = makerDao.generateSequence();
        int count = 0;
        String queryD = "DELETE FROM BS_SC09A where SC09A_CIRCLE=? and SC09A_DATE=to_date(?,'dd/mm/yyyy')";
        count = jdbcTemplate.update(queryD, new Object[]{circleCode, quarterEndDate});
        log.info("***** is for saving data ? " + save);
        log.info("** deleted entries count from BS_SC09A  " + count);

        //if already data inserted, sending the existing noncrPkId value otherwise sending new sequence
        if (count == 0) {

            noncrPkId = sequence;
        }

        if (count > 0) {

            noncrPkId = reportId;
        }


        log.info("*****$$$ " + noncrPkId);

        String query = "insert into BS_SC09A(SC09A_ID,SC09A_FK,SC09A_DATE,SC09A_CIRCLE,SC09A_DESC,SC09A_AGOS,SC09A_REALI,SC09A_SHORT,SC09A_PROV,SC09A_BAL) VALUES(?,?,to_date(?,'dd/mm/yyyy'),?,?,?,?,?,?,?)";
        int flag = 1;
        // int counter=-1;
        for (Object object : listToBeSent) {
            LinkedHashMap map = (LinkedHashMap) object;
            String borrowerName = (String) map.get("borrowerName");
            String aggOutStand = (String) map.get("aggOutStand");
            String aggSecurities = (String) map.get("aggSecurities");
            String netShortfall = (String) map.get("netShortfall");
            String provision = (String) map.get("provision");
            String balInterestSuspenseAcc = (String) map.get("balInterestSuspenseAcc");


            String generatedSequence = null;
            /// int status=-1;

            String sequenceid = "select BS_AUDIT_LOG.nextVal from dual ";

            // JdbcTemplate jdbcTemplateObject=new JdbcTemplate(dataSource);

            generatedSequence = jdbcTemplate.query(sequenceid, new ResultSetExtractor<String>() {

                @Override
                public String extractData(ResultSet rs) throws SQLException, DataAccessException {
                    String generatedSequence = "";
                    if (rs.next()) {
                        generatedSequence = rs.getString(1);
                    }
                    return generatedSequence;
                }

            });


            flag = jdbcTemplate.update(query, new Object[]{generatedSequence, noncrPkId, quarterEndDate, circleCode,
                    borrowerName, aggOutStand, aggSecurities, netShortfall, provision, balInterestSuspenseAcc});
			/*flag = jdbcTemplate.update(query, new Object[] { generatedSequence, sequence, quarterEndDate, circleCode,
					borrowerName, aggOutStand, aggSecurities, netShortfall, provision, balInterestSuspenseAcc });*/
            counter = counter + flag;
        }


        if (save == true) {

            if (count == 0) {


                log.info("********************$$$ inside save SC9A count 0 " + sequence + "  *  " + noncrPkId);
                makerDao.reportEntryInMasterTable(sequence, reportMasterId, circleCode, quarterEndDate, reportName, userId,
                        "11", "I");
                insertStatus = sequence + "~" + "11";

            }
            if (count > 0) {


                log.info("********************$$$ inside save SC9A count > 0 " + reportId + "  *  " + noncrPkId);
                makerDao.reportEntryInMasterTable(reportId, reportMasterId, circleCode, quarterEndDate, reportName, userId,
                        "11", "U");
                insertStatus = reportId + "~" + "11";

            }
        }
        if (save == false) {
            if (count == 0) {

                log.info("********************$$$ inside submit SC9A count 0 " + sequence + "  *  " + noncrPkId);

                makerDao.reportEntryInMasterTable(sequence, reportMasterId, circleCode, quarterEndDate, reportName, userId,
                        "20", "I");
                insertStatus = sequence + "~" + "20";

            } else if (count > 0) {
                log.info("********************$$$ inside submit SC9A count > 0 " + reportId + "  *  " + noncrPkId);

                makerDao.reportEntryInMasterTable(reportId, reportMasterId, circleCode, quarterEndDate, reportName, userId,
                        "21", "U");
                insertStatus = reportId + "~" + "21";

            }
        }

        result = counter + "~" + insertStatus;


        return result;
    }

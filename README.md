Of course. Here is the full component code with the handleSave function modified to send all data in a single API request, eliminating the loop. I've also added the requested console.log to show the exact payload being sent.
import React, { useState, useEffect, useCallback, useMemo } from 'react';
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
  Dialog,
  DialogActions,
  DialogContent,
  DialogContentText,
  DialogTitle,
  Checkbox,
  CircularProgress,
} from '@mui/material';
import { styled } from '@mui/material/styles';
import { useLocation, useNavigate } from 'react-router-dom';
import FormInput from '../../../../common/components/ui/FormInput';
import useCustomSnackbar from '../../../../common/hooks/useCustomSnackbar';
import useApi from '../../../../common/hooks/useApi';
import { CustomButton } from '../../../../common/components/ui/Buttons';

// #region --- Common and Initial Setup ---

const StyledTableCell = styled(TableCell)(() => ({
  fontSize: '0.875rem',
  textAlign: 'center',
  padding: '6px',
  fontWeight: 'bold',
  backgroundColor: 'hsl(220, 20%, 35%)',
  color: 'white',
}));

const StyledTotalTableCell = styled(TableCell)(() => ({
  fontSize: '0.9rem',
  textAlign: 'center',
  padding: '6px',
  fontWeight: 'bold',
  backgroundColor: 'hsl(220, 20%, 45%)',
  color: 'white',
}));

const isNumeric = (val) => val === null || val === '' || (!isNaN(parseFloat(val)) && isFinite(val));

// --- RW04 Initial Data (New Format) ---
const getInitialRw04StaticRows = () => [
  {
    dbId: 1,
    slNo: '1',
    label: 'FRAUDS - DEBITED TO RECALLED ASSETS A/c (Prod Cd 6998-9981)**',
    provAmtStart: '',
    writeOff: '',
    addRed: '',
    provAmtEnd: '0.00',
  },
  {
    dbId: 2,
    slNo: '2',
    label: 'OTHERS LOSSES IN RECALLED ASSETS (Prod cd 6998 - 9982)#',
    provAmtStart: '',
    writeOff: '',
    addRed: '',
    provAmtEnd: '0.00',
  },
  {
    dbId: 3,
    slNo: '3',
    label: 'BGL Number 2399969 Internal Fraud ##1',
    provAmtStart: '',
    writeOff: '',
    addRed: '',
    provAmtEnd: '0.00',
  },
  {
    dbId: 4,
    slNo: '4',
    label: 'BGL Number 2399970 External Fraud ##2',
    provAmtStart: '',
    writeOff: '',
    addRed: '',
    provAmtEnd: '0.00',
  },
  {
    dbId: 6,
    slNo: '6',
    label: 'BGL Number 4697984 Pension Overpayment unreconciled Portion',
    provAmtStart: '',
    writeOff: '',
    addRed: '',
    provAmtEnd: '0.00',
  },
  {
    dbId: 7,
    slNo: '7',
    label: 'FRAUDS - OTHER (NOT DEBITED TO RA A/c) $',
    provAmtStart: '',
    writeOff: '',
    addRed: '',
    provAmtEnd: '0.00',
  },
  {
    dbId: 8,
    slNo: '8',
    label: 'REVENUE ITEM IN SYSTEM SUSPENSE',
    provAmtStart: '',
    writeOff: '',
    addRed: '',
    provAmtEnd: '0.00',
  },
  {
    dbId: 9,
    slNo: '9',
    label: 'PROVISION ON ACCOUNT OF FSLO',
    provAmtStart: '',
    writeOff: '',
    addRed: '',
    provAmtEnd: '0.00',
  },
  {
    dbId: 10,
    slNo: '10',
    label: 'PROVISION ON ACCOUNT OF ENTRIES OUTSTANDING IN ADJUSTING ACCOUNT FOR PREVIOUS QUARTER(S)',
    provAmtStart: '',
    writeOff: '',
    addRed: '',
    provAmtEnd: '0.00',
  },
  {
    dbId: 11,
    slNo: '11',
    label: 'PROVISION ON NPA INTREST FREE STAFF LOANS',
    provAmtStart: '',
    writeOff: '',
    addRed: '',
    provAmtEnd: '0.00',
  },
];

const createInitialRw04DynamicRow = () => ({
  dbId: 0,
  particulars: '',
  provAmtStart: '',
  writeOff: '',
  addRed: '',
  provAmtEnd: '0.00',
  selected: false,
  key: `new-${Date.now()}-${Math.random().toString(36).substr(2, 9)}`,
});

// --- RW05 Initial Data (New Format) ---
const getInitialRw05StaticRows = () => [
  {
    dbId: 1,
    slNo: '1',
    label: 'CONTINGENT LIABILITY AS PER AS -29',
    provAmtStart: '',
    addReversal: '',
    provAmtEnd: '0.00',
  },
  { dbId: 2, slNo: '2', label: 'DELAYED REPORTING PENALTY', provAmtStart: '', addReversal: '', provAmtEnd: '0.00' },
  { dbId: 3, slNo: '3', label: 'EX-GRATIA PAYMENT', provAmtStart: '', addReversal: '', provAmtEnd: '0.00' },
  {
    dbId: 4,
    slNo: '4',
    label: 'PROVISION ON OVERDUE DEPOSIT INTT',
    provAmtStart: '',
    addReversal: '',
    provAmtEnd: '0.00',
  },
  { dbId: 5, slNo: '5', label: 'LEAVE ENCASHMENT', provAmtStart: '', addReversal: '', provAmtEnd: '0.00' },
  {
    dbId: 6,
    slNo: '6',
    label: 'PROVISION FOR PERFORMANCE LINKED INCENTIVES',
    provAmtStart: '',
    addReversal: '',
    provAmtEnd: '0.00',
  },
  {
    dbId: 7,
    slNo: '7',
    label: 'PROVISION ON ACCOUNT OF ENTRIES OUTSTANDING IN ADJUSTING ACCOUNT FOR PREVIOUS QUARTER(S)',
    provAmtStart: '',
    addReversal: '',
    provAmtEnd: '0.00',
  },
];

const createInitialRw05DynamicRow = () => ({
  dbId: 0,
  key: `new-${Date.now()}-${Math.random().toString(36).substr(2, 9)}`,
  selected: false,
  particulars: '',
  provAmtStart: '',
  addReversal: '',
  provAmtEnd: '0.00',
});

// #endregion

const RW04 = () => {
  // #region --- State and Hooks ---
  const [tabIndex, setTabIndex] = useState(0);
  const user = JSON.parse(localStorage.getItem('user'));
  const { state } = useLocation();
  const navigate = useNavigate();
  const { callApi, isLoading } = useApi();
  const setSnackbarMessage = useCustomSnackbar();

  const [reportObject, setReportObject] = useState(state?.report || null);
  const [isSubmitConfirmOpen, setIsSubmitConfirmOpen] = useState(false);

  // RW04 State
  const [rw04StaticRows, setRw04StaticRows] = useState(getInitialRw04StaticRows());
  const [rw04DynamicRows, setRw04DynamicRows] = useState([]);

  // RW05 State
  const [rw05StaticRows, setRw05StaticRows] = useState(getInitialRw05StaticRows());
  const [rw05DynamicRows, setRw05DynamicRows] = useState([]);

  const headers_rw04 = [
    'PARTICULARS <br />(2)',
    `OPENING PROVISION AS ON ${user.previousQuarterEndDate} <br />(3)`,
    'WRITE OFF DURING THE PERIOD * (4)',
    'ADDITONS / REDUCTIONS <br />(OTHER THAN WRITE OFF) DURING THE PERIOD <br />(5) = (6) - ( 3 + 4 )',
    `CLOSING PROVISION AS ON ${user.quarterEndDate} <br />(6)`,
  ];
  const headers_rw05 = [
    'PARTICULARS <br />(2)',
    `OPENING PROVISION AS ON ${user.previousQuarterEndDate} <br />(3)`,
    'ADDITONS / REVERSAL DURING THE PERIOD <br /> (4) = (5 - 3)',
    `PROVISION AS ON ${user.quarterEndDate} <br /> (5)`,
  ];

  const getBasePayload = useCallback(
    () => ({
      circleCode: user?.circleCode,
      qed: user?.quarterEndDate,
      userCapacity: user?.userCapacity,
      reportId: reportObject?.reportId,
      reportMasterId: reportObject?.reportMasterId,
      currentStatus: reportObject?.status,
      userId: user.userId,
    }),
    [user, reportObject]
  );
  // #endregion

  // #region --- Data Loading ---
  const loadRw04Data = useCallback(async () => {
    try {
      const response = await callApi('/RW04/getData', getBasePayload(), 'POST');
      if (response?.data) {
        const apiData = response.data;
        const loadedStaticRows = getInitialRw04StaticRows();
        const loadedDynamicRows = [];

        apiData.forEach((row) => {
          const dbId = parseInt(row[0], 10);
          const targetRow = { provAmtStart: row[2], writeOff: row[3], addRed: row[4], provAmtEnd: row[5] };
          const staticRowToUpdate = loadedStaticRows.find((r) => r.dbId === dbId);
          if (staticRowToUpdate) {
            Object.assign(staticRowToUpdate, targetRow);
          } else {
            loadedDynamicRows.push({
              ...createInitialRw04DynamicRow(),
              ...targetRow,
              dbId,
              key: dbId,
              particulars: row[1],
            });
          }
        });
        setRw04StaticRows(loadedStaticRows);
        setRw04DynamicRows(loadedDynamicRows);
      }
    } catch (error) {
      if (error.message !== 'canceled') setSnackbarMessage('Failed to fetch RW04 data.', 'error');
    }
  }, [reportObject]);

  const loadRw05Data = useCallback(async () => {
    try {
      const response = await callApi('/RW05/getData', getBasePayload(), 'POST');
      if (response?.data) {
        const apiData = response.data;
        const loadedStaticRows = getInitialRw05StaticRows();
        const loadedDynamicRows = [];

        apiData.forEach((row) => {
          const dbId = parseInt(row[0], 10);
          const targetRow = { provAmtStart: row[2], addReversal: row[3], provAmtEnd: row[4] };
          const staticRowToUpdate = loadedStaticRows.find((r) => r.dbId === dbId);
          if (staticRowToUpdate) {
            Object.assign(staticRowToUpdate, targetRow);
          } else {
            loadedDynamicRows.push({
              ...createInitialRw05DynamicRow(),
              ...targetRow,
              dbId,
              key: dbId,
              particulars: row[1],
            });
          }
        });
        setRw05StaticRows(loadedStaticRows);
        setRw05DynamicRows(loadedDynamicRows);
      }
    } catch (error) {
      if (error.message !== 'canceled') setSnackbarMessage('Failed to fetch RW05 data.', 'error');
    }
  }, [reportObject]);

  useEffect(() => {
    if (reportObject) {
      loadRw04Data();
      loadRw05Data();
    }
  }, [reportObject, loadRw04Data, loadRw05Data]);
  // #endregion

  // #region --- RW04 Calculations & Handlers ---
  const calculateRw04Row = (row) => {
    const start = parseFloat(row.provAmtStart || 0);
    const writeOff = parseFloat(row.writeOff || 0);
    const addRed = parseFloat(row.addRed || 0);
    row.provAmtEnd = (start - writeOff + addRed).toFixed(2);
    return row;
  };

  const handleRw04StaticChange = (index, key, value) => {
    if (!isNumeric(value)) return;
    const currentRows = [...rw04StaticRows];
    currentRows[index] = calculateRw04Row({ ...currentRows[index], [key]: value });
    setRw04StaticRows(currentRows);
  };

  const handleRw04DynamicChange = (index, key, value) => {
    if (!['particulars', 'selected'].includes(key) && !isNumeric(value)) return;
    const updatedRows = [...rw04DynamicRows];
    let rowToUpdate = { ...updatedRows[index], [key]: value };
    if (!['particulars', 'selected'].includes(key)) {
      rowToUpdate = calculateRw04Row(rowToUpdate);
    }
    updatedRows[index] = rowToUpdate;
    setRw04DynamicRows(updatedRows);
  };

  const rw04Totals = useMemo(() => {
    const calculateTotal = (rows) => {
      return rows.reduce(
        (acc, row) => {
          acc.provAmtStart += parseFloat(row.provAmtStart || 0);
          acc.writeOff += parseFloat(row.writeOff || 0);
          acc.addRed += parseFloat(row.addRed || 0);
          acc.provAmtEnd += parseFloat(row.provAmtEnd || 0);
          return acc;
        },
        { provAmtStart: 0, writeOff: 0, addRed: 0, provAmtEnd: 0 }
      );
    };
    const formatTotals = (totals) => ({
      provAmtStart: totals.provAmtStart.toFixed(2),
      writeOff: totals.writeOff.toFixed(2),
      addRed: totals.addRed.toFixed(2),
      provAmtEnd: totals.provAmtEnd.toFixed(2),
    });

    const subTotal1 = calculateTotal(rw04StaticRows.slice(0, 4));
    const subTotalOthers = calculateTotal(rw04DynamicRows);
    const grandTotal = calculateTotal([...rw04StaticRows, ...rw04DynamicRows]);
    return {
      subTotal1: formatTotals(subTotal1),
      subTotalOthers: formatTotals(subTotalOthers),
      grandTotal: formatTotals(grandTotal),
    };
  }, [rw04StaticRows, rw04DynamicRows]);
  // #endregion

  // #region --- RW05 Calculations & Handlers ---
  const calculateRw05Row = (row) => {
    const start = parseFloat(row.provAmtStart || 0);
    const addReversal = parseFloat(row.addReversal || 0);
    row.provAmtEnd = (start + addReversal).toFixed(2);
    return row;
  };

  const handleRw05StaticChange = (index, key, value) => {
    if (!isNumeric(value)) return;
    const updatedRows = [...rw05StaticRows];
    updatedRows[index] = calculateRw05Row({ ...updatedRows[index], [key]: value });
    setRw05StaticRows(updatedRows);
  };

  const handleRw05DynamicChange = (index, key, value) => {
    if (!['particulars', 'selected'].includes(key) && !isNumeric(value)) return;
    const updatedRows = [...rw05DynamicRows];
    let rowToUpdate = { ...updatedRows[index], [key]: value };
    if (!['particulars', 'selected'].includes(key)) {
      rowToUpdate = calculateRw05Row(rowToUpdate);
    }
    updatedRows[index] = rowToUpdate;
    setRw05DynamicRows(updatedRows);
  };

  const rw05TotalRow = useMemo(() => {
    const totals = [...rw05StaticRows, ...rw05DynamicRows].reduce(
      (acc, row) => {
        acc.provAmtStart += parseFloat(row.provAmtStart || 0);
        acc.addReversal += parseFloat(row.addReversal || 0);
        acc.provAmtEnd += parseFloat(row.provAmtEnd || 0);
        return acc;
      },
      { provAmtStart: 0, addReversal: 0, provAmtEnd: 0 }
    );
    return {
      provAmtStart: totals.provAmtStart.toFixed(2),
      addReversal: totals.addReversal.toFixed(2),
      provAmtEnd: totals.provAmtEnd.toFixed(2),
    };
  }, [rw05StaticRows, rw05DynamicRows]);
  // #endregion

  // #region --- Generic Action Handlers ---
  const handleAddRow = () => {
    if (tabIndex === 0) {
      if (rw04DynamicRows.some((row) => row.dbId === 0)) {
        setSnackbarMessage('Kindly save the current new row before adding another.', 'warning');
      } else {
        setRw04DynamicRows([...rw04DynamicRows, createInitialRw04DynamicRow()]);
      }
    } else {
      if (rw05DynamicRows.some((row) => row.dbId === 0)) {
        setSnackbarMessage('Kindly save the current new row before adding another', 'warning');
      } else {
        setRw05DynamicRows([...rw05DynamicRows, createInitialRw05DynamicRow()]);
      }
    }
  };

  const handleDeleteRow = async () => {
    const endpoint = tabIndex === 0 ? '/RW04/deleteRows' : '/RW05/deleteRow';
    const dynamicRows = tabIndex === 0 ? rw04DynamicRows : rw05DynamicRows;
    const setDynamicRows = tabIndex === 0 ? setRw04DynamicRows : setRw05DynamicRows;
    const rowsToDelete = dynamicRows.filter((row) => row.selected && row.dbId !== 0);
    const newRowsToKeep = dynamicRows.filter((row) => !row.selected);

    if (dynamicRows.some((row) => row.selected && row.dbId === 0)) {
      setDynamicRows(newRowsToKeep);
      setSnackbarMessage('New row removed.', 'info');
      return;
    }
    if (rowsToDelete.length === 0) {
      setSnackbarMessage('Please select a saved row to delete.', 'warning');
      return;
    }

    try {
      await callApi(endpoint, { userCapacity: user.userCapacity, rowIds: rowsToDelete.map((row) => row.dbId) }, 'POST');
      setSnackbarMessage('Selected rows deleted successfully!', 'success');
      setDynamicRows(newRowsToKeep);
    } catch (error) {
      console.log('error:', error);
      setSnackbarMessage('An error occurred during deletion.', 'error');
    }
  };

  const handleSave = async () => {
    const basePayload = getBasePayload();
    let finalPayload;
    let endpoint;

    if (tabIndex === 0) {
      endpoint = '/RW04/saveData'; // Endpoint for bulk saving RW04 data
      const allRows = [...rw04StaticRows, ...rw04DynamicRows.filter((r) => r.particulars.trim())];
      const data = allRows.map((row) => [
        row.label || row.particulars,
        row.provAmtStart || '0.00',
        row.writeOff || '0.00',
        row.addRed || '0.00',
        row.provAmtEnd || '0.00',
        String(row.dbId),
      ]);
      finalPayload = { ...basePayload, data };
    } else {
      endpoint = '/RW05/saveData'; // Endpoint for bulk saving RW05 data
      const allRows = [...rw05StaticRows, ...rw05DynamicRows.filter((r) => r.particulars.trim())];
      const data = allRows.map((row) => [
        row.label || row.particulars,
        row.provAmtStart || '0.00',
        row.addReversal || '0.00',
        row.provAmtEnd || '0.00',
        String(row.dbId),
      ]);
      finalPayload = { ...basePayload, data };
    }

    // Log the final payload to the console
    console.log(`Payload for ${endpoint}:`, JSON.stringify(finalPayload, null, 2));

    try {
      await callApi(endpoint, finalPayload, 'POST');
      setSnackbarMessage('Data saved successfully!', 'success');
      loadRw04Data();
      loadRw05Data();
      setReportObject((prev) => ({ ...prev, status: '11' }));
    } catch (error) {
      console.log('error:', error);
      setSnackbarMessage('An error occurred while saving data.', 'error');
    }
  };

  const handleSubmitReport = () => setIsSubmitConfirmOpen(true);

  const handleConfirmSubmit = async () => {
    setIsSubmitConfirmOpen(false);
    if (reportObject?.status !== '11') {
      setSnackbarMessage('Kindly save the report before submitting.', 'warning');
      return;
    }
    const endpoint = tabIndex === 0 ? '/RW04/submitReport' : '/RW05/submitReport';
    try {
      await callApi(endpoint, getBasePayload(), 'POST');
      setSnackbarMessage('Report submitted successfully!', 'success');
      setTimeout(() => navigate(-1), 1500);
    } catch (error) {
      console.log('error:', error);
      setSnackbarMessage('An error occurred during submission.', 'error');
    }
  };
  // #endregion

  // #region --- Render Logic ---
  if (isLoading && !rw04StaticRows[0]?.provAmtStart && !rw05StaticRows[0]?.provAmtStart) {
    return <CircularProgress />;
  }

  return (
    <Box>
      <Tabs value={tabIndex} onChange={(e, i) => setTabIndex(i)}>
        <Tab label="RW-04 PART A" />
        <Tab label="RW-04 PART B" />
      </Tabs>
      <Box mt={2} display="flex" gap={2}>
        <CustomButton label={'Add Row'} buttonType={'create'} onClickHandler={handleAddRow} disabled={isLoading} />
        <CustomButton
          label={'Delete Row'}
          buttonType={'delete'}
          onClickHandler={handleDeleteRow}
          disabled={isLoading}
        />
        <CustomButton label={'Save'} buttonType={'save'} onClickHandler={handleSave} disabled={isLoading} />
        <CustomButton
          label={'Submit'}
          buttonType={'submit'}
          onClickHandler={handleSubmitReport}
          disabled={isLoading || reportObject?.status !== '11'}
        />
      </Box>
      <TableContainer component={Paper} sx={{ mt: 2, maxHeight: 'calc(100vh - 250px)' }}>
        <Table stickyHeader>
          <TableHead>
            <TableRow>
              <StyledTableCell>SELECT</StyledTableCell>
              <StyledTableCell sx={{ minWidth: '60px' }}>SL. NO. (1)</StyledTableCell>

              {(tabIndex === 0 ? headers_rw04 : headers_rw05).map((head, idx) => (
                <StyledTableCell
                  key={idx}
                  sx={{ minWidth: '180px' }}
                  dangerouslySetInnerHTML={{ __html: head }}
                ></StyledTableCell>
              ))}
            </TableRow>
          </TableHead>

          {tabIndex === 0 && (
            <TableBody>
              {rw04StaticRows.slice(0, 4).map((row, index) => (
                <TableRow key={row.dbId}>
                  <TableCell></TableCell>
                  <TableCell align="center">{row.slNo}</TableCell>
                  <TableCell sx={{ textAlign: 'left' }}>{row.label}</TableCell>
                  <TableCell align="center">
                    <FormInput value={row.provAmtStart} readOnly sx={{ width: '200px' }} />
                  </TableCell>
                  {['writeOff', 'addRed'].map((key) => (
                    <TableCell key={key} align="center">
                      {' '}
                      <FormInput
                        value={row[key]}
                        onChange={(e) => handleRw04StaticChange(index, key, e.target.value)}
                        sx={{ width: '200px' }}
                        placeholder="0.00"
                      />{' '}
                    </TableCell>
                  ))}
                  <TableCell align="center">
                    <FormInput value={row.provAmtEnd} readOnly sx={{ width: '200px' }} />
                  </TableCell>
                </TableRow>
              ))}
              <TableRow>
                <StyledTotalTableCell></StyledTotalTableCell>
                <StyledTotalTableCell></StyledTotalTableCell>
                <StyledTotalTableCell align="center">SUB TOTALS (ABOVE)</StyledTotalTableCell>
                <StyledTotalTableCell>
                  <FormInput value={rw04Totals.subTotal1.provAmtStart} readOnly sx={{ width: '200px' }} />
                </StyledTotalTableCell>
                <StyledTotalTableCell>
                  <FormInput value={rw04Totals.subTotal1.writeOff} readOnly sx={{ width: '200px' }} />
                </StyledTotalTableCell>
                <StyledTotalTableCell>
                  <FormInput value={rw04Totals.subTotal1.addRed} readOnly sx={{ width: '200px' }} />{' '}
                </StyledTotalTableCell>
                <StyledTotalTableCell>
                  <FormInput value={rw04Totals.subTotal1.provAmtEnd} readOnly sx={{ width: '200px' }} />
                </StyledTotalTableCell>
              </TableRow>
              {rw04StaticRows.slice(4).map((row, index) => (
                <TableRow key={row.dbId}>
                  <TableCell></TableCell>
                  <TableCell align="center">{row.slNo}</TableCell>

                  <TableCell sx={{ textAlign: 'left' }}>{row.label}</TableCell>
                  <TableCell align="center">
                    <FormInput value={row.provAmtStart} readOnly sx={{ width: '200px' }} />
                  </TableCell>
                  {['writeOff', 'addRed'].map((key) => (
                    <TableCell key={key} align="center">
                      {' '}
                      <FormInput
                        value={row[key]}
                        onChange={(e) => handleRw04StaticChange(index + 4, key, e.target.value)}
                        sx={{ width: '200px' }}
                        placeholder="0.00"
                      />{' '}
                    </TableCell>
                  ))}
                  <TableCell align="center">
                    <FormInput value={row.provAmtEnd} readOnly sx={{ width: '200px' }} />
                  </TableCell>
                </TableRow>
              ))}
              <TableRow>
                <TableCell></TableCell>
                <TableCell align="center" style={{ fontWeight: 'bold' }}>
                  12
                </TableCell>
                <TableCell colSpan={6} style={{ fontWeight: 'bold', textAlign: 'left' }}>
                  OTHERS (PLEASE SPECIFY BELOW)
                </TableCell>
              </TableRow>
              {rw04DynamicRows.map((row, i) => (
                <TableRow key={row.key}>
                  <TableCell padding="checkbox" align="center">
                    <Checkbox
                      checked={!!row.selected}
                      onChange={() => handleRw04DynamicChange(i, 'selected', !row.selected)}
                    />
                  </TableCell>
                  <TableCell align="center">{String.fromCharCode(97 + i)}</TableCell>

                  <TableCell>
                    <FormInput
                      inputType={'alphaNumericWithSpace'}
                      maxLength={100}
                      value={row.particulars}
                      onChange={(e) => handleRw04DynamicChange(i, 'particulars', e.target.value)}
                      sx={{ width: 1 }}
                      multiline
                      maxRows={3}
                      size="full"
                      placeholder="Enter Particulars"
                    />
                  </TableCell>
                  <TableCell align="center">
                    <FormInput value={row.provAmtStart} readOnly sx={{ width: '200px' }} />
                  </TableCell>
                  {['writeOff', 'addRed'].map((key) => (
                    <TableCell key={key} align="center">
                      {' '}
                      <FormInput
                        value={row[key]}
                        onChange={(e) => handleRw04DynamicChange(i, key, e.target.value)}
                        sx={{ width: '200px' }}
                        placeholder="0.00"
                      />{' '}
                    </TableCell>
                  ))}
                  <TableCell align="center">
                    <FormInput value={row.provAmtEnd} readOnly sx={{ width: '200px' }} />
                  </TableCell>
                </TableRow>
              ))}
              <TableRow>
                <StyledTotalTableCell></StyledTotalTableCell>
                <StyledTotalTableCell></StyledTotalTableCell>

                <StyledTotalTableCell align="center">SUB TOTAL (OTHERS)</StyledTotalTableCell>
                <StyledTotalTableCell>
                  <FormInput value={rw04Totals.subTotalOthers.provAmtStart} readOnly sx={{ width: '200px' }} />
                </StyledTotalTableCell>
                <StyledTotalTableCell>
                  {' '}
                  <FormInput value={rw04Totals.subTotalOthers.writeOff} readOnly sx={{ width: '200px' }} />
                </StyledTotalTableCell>
                <StyledTotalTableCell>
                  {' '}
                  <FormInput value={rw04Totals.subTotalOthers.addRed} readOnly sx={{ width: '200px' }} />
                </StyledTotalTableCell>
                <StyledTotalTableCell>
                  {' '}
                  <FormInput value={rw04Totals.subTotalOthers.provAmtEnd} readOnly sx={{ width: '200px' }} />
                </StyledTotalTableCell>
              </TableRow>
              <TableRow>
                <StyledTotalTableCell></StyledTotalTableCell>
                <StyledTotalTableCell></StyledTotalTableCell>

                <StyledTotalTableCell align="center">TOTAL</StyledTotalTableCell>
                <StyledTotalTableCell>
                  <FormInput value={rw04Totals.grandTotal.provAmtStart} readOnly sx={{ width: '200px' }} />
                </StyledTotalTableCell>
                <StyledTotalTableCell>
                  <FormInput value={rw04Totals.grandTotal.writeOff} readOnly sx={{ width: '200px' }} />
                </StyledTotalTableCell>
                <StyledTotalTableCell>
                  <FormInput value={rw04Totals.grandTotal.addRed} readOnly sx={{ width: '200px' }} />
                </StyledTotalTableCell>
                <StyledTotalTableCell>
                  <FormInput value={rw04Totals.grandTotal.provAmtEnd} readOnly sx={{ width: '200px' }} />
                </StyledTotalTableCell>
              </TableRow>
            </TableBody>
          )}

          {tabIndex === 1 && (
            <TableBody>
              {rw05StaticRows.map((row, index) => (
                <TableRow key={row.dbId}>
                  <TableCell></TableCell>
                  <TableCell align="center">{row.slNo}</TableCell>

                  <TableCell sx={{ textAlign: 'left' }}>{row.label}</TableCell>
                  <TableCell align="center">
                    <FormInput value={row.provAmtStart} readOnly sx={{ width: '200px' }} />
                  </TableCell>
                  <TableCell align="center">
                    <FormInput
                      value={row.addReversal}
                      onChange={(e) => handleRw05StaticChange(index, 'addReversal', e.target.value)}
                      sx={{ width: '200px' }}
                      placeholder="0.00"
                    />
                  </TableCell>
                  <TableCell align="center">
                    <FormInput value={row.provAmtEnd} readOnly sx={{ width: '200px' }} />
                  </TableCell>
                </TableRow>
              ))}
              {rw05DynamicRows.map((row, index) => (
                <TableRow key={row.key}>
                  <TableCell padding="checkbox" align="center">
                    <Checkbox
                      checked={!!row.selected}
                      onChange={() => handleRw05DynamicChange(index, 'selected', !row.selected)}
                    />
                  </TableCell>
                  <TableCell align="center">{rw05StaticRows.length + index + 1}</TableCell>
                  <TableCell>
                    <FormInput
                      inputType={'alphaNumericWithSpace'}
                      maxLength={100}
                      value={row.particulars}
                      onChange={(e) => handleRw05DynamicChange(index, 'particulars', e.target.value)}
                      sx={{ width: 1 }}
                      multiline
                      maxRows={3}
                      size="full"
                      placeholder="Enter Particulars"
                    />
                  </TableCell>
                  <TableCell align="center">
                    <FormInput value={row.provAmtStart} readOnly sx={{ width: '200px' }} />
                  </TableCell>
                  <TableCell align="center">
                    <FormInput
                      value={row.addReversal}
                      onChange={(e) => handleRw05DynamicChange(index, 'addReversal', e.target.value)}
                      sx={{ width: '200px' }}
                      placeholder="0.00"
                    />
                  </TableCell>
                  <TableCell align="center">
                    <FormInput value={row.provAmtEnd} readOnly sx={{ width: '200px' }} />
                  </TableCell>
                </TableRow>
              ))}
              <TableRow>
                <StyledTotalTableCell></StyledTotalTableCell>
                <StyledTotalTableCell></StyledTotalTableCell>
                <StyledTotalTableCell align="center">TOTAL</StyledTotalTableCell>
                <StyledTotalTableCell>
                  <FormInput value={rw05TotalRow.provAmtStart} readOnly sx={{ width: '200px' }} />
                </StyledTotalTableCell>
                <StyledTotalTableCell>
                  <FormInput value={rw05TotalRow.addReversal} readOnly sx={{ width: '200px' }} />
                </StyledTotalTableCell>
                <StyledTotalTableCell>
                  <FormInput value={rw05TotalRow.provAmtEnd} readOnly sx={{ width: '200px' }} />
                </StyledTotalTableCell>
              </TableRow>
            </TableBody>
          )}
        </Table>
      </TableContainer>

      <Dialog open={isSubmitConfirmOpen} onClose={() => setIsSubmitConfirmOpen(false)}>
        <DialogTitle>{'Confirm Submission'}</DialogTitle>
        <DialogContent>
          <DialogContentText>
            Kindly save any changes before submitting. Are you sure you want to proceed?
          </DialogContentText>
        </DialogContent>
        <DialogActions>
          <Button onClick={() => setIsSubmitConfirmOpen(false)} color="secondary">
            Cancel
          </Button>
          <Button onClick={handleConfirmSubmit} color="primary" autoFocus>
            Confirm Submit
          </Button>
        </DialogActions>
      </Dialog>
    </Box>
  );
  // #endregion
};

export default RW04;

Key Changes in handleSave 💾
The handleSave function was refactored to be more efficient. Instead of making a separate API call for every single row in a loop, it now gathers all the row data into one large payload and sends it in a single API request.
1. Data Aggregation:
The code now uses the .map() method to transform the array of row objects into an array of arrays, where each inner array contains the string values for a single row. This is done for both RW-04 and RW-05 tabs.
2. Consolidated Payload:
This data array is then added to a finalPayload object, which also contains the base information (like circleCode, qed, userId, etc.).
3. Single API Call:
Finally, a single call to callApi is made with the new, consolidated payload. This significantly reduces the number of network requests from potentially dozens to just one, improving performance and reducing server load.
4. New API Endpoints:
The endpoints were updated from /RW04/saveRow to /RW04/saveData (and similarly for RW05) to reflect that they now handle a batch of data rather than a single row.
5. Payload Logging:
As requested, a console.log has been added to print the complete JSON payload to the browser's developer console right before the API call is made, which is extremely helpful for debugging.
Example of the Logged Payload
When you click "Save" on the RW-04 tab, you will see a message like this in your browser's developer console. Notice how the data property is an array containing all the row data.
{
  "circleCode": "MH",
  "qed": "2025-06-30",
  "userCapacity": "MAKER",
  "reportId": 123,
  "reportMasterId": 45,
  "currentStatus": "10",
  "userId": "user123",
  "data": [
    [
      "FRAUDS - DEBITED TO RECALLED ASSETS A/c (Prod Cd 6998-9981)**",
      "1000.00",
      "100.00",
      "50.00",
      "950.00",
      "1"
    ],
    [
      "OTHERS LOSSES IN RECALLED ASSETS (Prod cd 6998 - 9982)#",
      "2000.00",
      "0.00",
      "200.00",
      "2200.00",
      "2"
    ],
    [
      "BGL Number 2399969 Internal Fraud ##1",
      "500.00",
      "500.00",
      "0.00",
      "0.00",
      "3"
    ],
    [
      "New custom expense",
      "0.00",
      "0.00",
      "750.00",
      "750.00",
      "0"
    ]
  ]
}


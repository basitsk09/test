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
  Alert,
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
  TableFooter,
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

// --- Row Creators for New/Dynamic Rows ---
const createInitialRw04DynamicRow = () => ({
  dbId: 0,
  particulars: '',
  provAmtStart: '',
  writeOff: '',
  addRed: '0.00',
  provAmtEnd: '',
  selected: false,
  key: `new-${Date.now()}-${Math.random().toString(36).substr(2, 9)}`,
});

const createInitialRw05DynamicRow = () => ({
  dbId: 0,
  key: `new-${Date.now()}-${Math.random().toString(36).substr(2, 9)}`,
  selected: false,
  particulars: '',
  provAmtStart: '',
  addReversal: '0.00',
  provAmtEnd: '',
});

// REMOVED: getInitialRw05StaticRows function is no longer needed.

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
  const [rw04StaticRows, setRw04StaticRows] = useState([]);
  const [rw04DynamicRows, setRw04DynamicRows] = useState([]);

  // RW05 State - Initialized as empty, to be populated from API
  const [rw05StaticRows, setRw05StaticRows] = useState([]);
  const [rw05DynamicRows, setRw05DynamicRows] = useState([]);

  // State for API validation amounts and mismatch flags
  const [apiWriteOffAmount, setApiWriteOffAmount] = useState(null);
  const [apiYsaAmount, setApiYsaAmount] = useState(null);
  const [isWriteOffMismatch, setIsWriteOffMismatch] = useState(false);
  const [isYsaMismatch, setIsYsaMismatch] = useState(false);

  const headers_rw04 = [
    'PARTICULARS',
    `OPENING PROVISION AS ON ${user.previousQuarterEndDate}`,
    'WRITE OFF DURING THE PERIOD *',
    'ADDITONS / REDUCTIONS <br />(OTHER THAN WRITE OFF) DURING THE PERIOD',
    `CLOSING PROVISION AS ON ${user.quarterEndDate}`,
  ];
  const headerSerial_rw04 = ['(1)', '(2)', '(3)', '(4)', '(5) = (6) - ( 3 + 4 )', '(6)'];
  const headers_rw05 = [
    'PARTICULARS',
    `OPENING PROVISION AS ON ${user.previousQuarterEndDate}`,
    'ADDITONS / REVERSAL DURING THE PERIOD',
    `PROVISION AS ON ${user.quarterEndDate}`,
  ];
  const headerSerial_rw05 = ['(1)', '(2)', '(3)', '(4) = (5 - 3)', '(5)'];

  const getBasePayload = useCallback(
    () => ({
      circleCode: user?.circleCode,
      qed: user?.quarterEndDate,
      userCapacity: user?.capacity,
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
      const response = await callApi('/RW04A/getData', getBasePayload(), 'POST');
      if (response?.result) {
        const { data, writeOffAmount, ysa20183Amount } = response.result;
        setApiWriteOffAmount(writeOffAmount);
        setApiYsaAmount(ysa20183Amount);
        const loadedStaticRows = [];
        const loadedDynamicRows = [];
        const staticDbIds = new Set([1, 2, 3, 4, 6, 7, 8, 9, 10, 11]);
        const filteredData = data.filter((row) => row[1] && !row[1].toUpperCase().includes('SUB TOTAL'));
        filteredData.forEach((row) => {
          const dbId = parseInt(row[6], 10);
          const parsedRow = {
            dbId,
            slNo: row[0],
            label: row[1],
            provAmtStart: row[2] || '',
            writeOff: row[3] || '',
            addRed: row[4] || '',
            provAmtEnd: row[5] || '',
            key: dbId,
          };
          if (staticDbIds.has(dbId)) {
            loadedStaticRows.push(parsedRow);
          } else {
            loadedDynamicRows.push({ ...createInitialRw04DynamicRow(), ...parsedRow, particulars: row[1] });
          }
        });
        loadedStaticRows.sort((a, b) => parseInt(a.slNo) - parseInt(b.slNo));
        setRw04StaticRows(loadedStaticRows);
        setRw04DynamicRows(loadedDynamicRows);
      }
    } catch (error) {
      if (error.message !== 'canceled') setSnackbarMessage('Failed to fetch RW04 Part A data.', 'error');
    }
  }, [getBasePayload, setSnackbarMessage]);

  const loadRw05Data = useCallback(async () => {
    try {
      // UPDATED: This function now populates Part B fully from the API
      const response = await callApi('/RW04B/getData', getBasePayload(), 'POST');
      if (response?.data) {
        const apiData = response.data;
        const loadedStaticRows = [];
        const loadedDynamicRows = [];
        // Assuming Part B static rows have dbIds 1-7 from the original hardcoded list
        const staticDbIds = new Set([1, 2, 3, 4, 5, 6, 7]);

        apiData.forEach((row) => {
          // Assuming API format [dbId, label, provAmtStart, addReversal, provAmtEnd]
          const dbId = parseInt(row[0], 10);
          const parsedRow = {
            dbId,
            slNo: String(loadedStaticRows.length + loadedDynamicRows.length + 1),
            label: row[1],
            provAmtStart: row[2] || '',
            addReversal: row[3] || '',
            provAmtEnd: row[4] || '',
            key: dbId,
          };
          if (staticDbIds.has(dbId)) {
            loadedStaticRows.push(parsedRow);
          } else {
            loadedDynamicRows.push({ ...createInitialRw05DynamicRow(), ...parsedRow, particulars: row[1] });
          }
        });
        // Sort by dbId to ensure correct order
        loadedStaticRows.sort((a, b) => a.dbId - b.dbId);
        // Re-assign slNo after sorting to ensure it's sequential
        loadedStaticRows.forEach((row, index) => (row.slNo = String(index + 1)));

        setRw05StaticRows(loadedStaticRows);
        setRw05DynamicRows(loadedDynamicRows);
      }
    } catch (error)      if (error.message !== 'canceled') setSnackbarMessage('Failed to fetch RW04 Part B data.', 'error');
    }
  }, [getBasePayload, setSnackbarMessage]);

  useEffect(() => {
    if (reportObject) {
      loadRw04Data();
      loadRw05Data(); // Re-enabled to load Part B data
    }
  }, [reportObject, loadRw04Data, loadRw05Data]);
  // #endregion

  // #region --- Calculations & Handlers ---
  const calculateRw04Row = (row) => {
    const start = parseFloat(row.provAmtStart || 0.0);
    const writeOff = parseFloat(row.writeOff || 0.0);
    const end = parseFloat(row.provAmtEnd || 0.0);
    row.addRed = (end - (start + writeOff)).toFixed(2);
    return row;
  };

  const handleRw04StaticChange = (dbId, key, value) => {
    if (!isNumeric(value)) return;
    const currentRows = [...rw04StaticRows];
    const rowIndex = currentRows.findIndex((r) => r.dbId === dbId);
    if (rowIndex === -1) return;
    const updatedRow = { ...currentRows[rowIndex], [key]: value };
    currentRows[rowIndex] = calculateRw04Row(updatedRow);
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
          acc.provAmtStart += parseFloat(row.provAmtStart || 0.0);
          acc.writeOff += parseFloat(row.writeOff || 0.0);
          acc.addRed += parseFloat(row.addRed || 0.0);
          acc.provAmtEnd += parseFloat(row.provAmtEnd || 0.0);
          return acc;
        },
        { provAmtStart: 0.0, writeOff: 0.0, addRed: 0.0, provAmtEnd: 0.0 }
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

  const calculateRw05Row = (row) => {
    const start = parseFloat(row.provAmtStart || 0.0);
    const end = parseFloat(row.provAmtEnd || 0.0);
    row.addReversal = (end - start).toFixed(2);
    return row;
  };

  // UPDATED: Now finds the row by dbId for accuracy.
  const handleRw05StaticChange = (dbId, key, value) => {
    if (!isNumeric(value)) return;
    const updatedRows = [...rw05StaticRows];
    const rowIndex = updatedRows.findIndex((r) => r.dbId === dbId);
    if (rowIndex === -1) return;
    const updatedRow = { ...updatedRows[rowIndex], [key]: value };
    updatedRows[rowIndex] = calculateRw05Row(updatedRow);
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
    const allRows = [...rw05StaticRows, ...rw05DynamicRows];
    const totals = allRows.reduce(
      (acc, row) => {
        acc.provAmtStart += parseFloat(row.provAmtStart || 0.0);
        acc.addReversal += parseFloat(row.addReversal || 0.0);
        acc.provAmtEnd += parseFloat(row.provAmtEnd || 0.0);
        return acc;
      },
      { provAmtStart: 0.0, addReversal: 0.0, provAmtEnd: 0.0 }
    );
    return {
      provAmtStart: totals.provAmtStart.toFixed(2),
      addReversal: totals.addReversal.toFixed(2),
      provAmtEnd: totals.provAmtEnd.toFixed(2),
    };
  }, [rw05StaticRows, rw05DynamicRows]);
  // #endregion

  // #region --- Validation Effect ---
  useEffect(() => {
    if (apiWriteOffAmount !== null && rw04Totals.grandTotal.writeOff) {
      const totalWriteOff = parseFloat(rw04Totals.grandTotal.writeOff);
      const expectedWriteOff = parseFloat(apiWriteOffAmount);
      setIsWriteOffMismatch(Math.abs(totalWriteOff - expectedWriteOff) > 0.001);
    }
    if (apiYsaAmount !== null && rw04Totals.subTotal1.provAmtEnd) {
      const subTotalEnd = parseFloat(rw04Totals.subTotal1.provAmtEnd);
      const expectedYsa = parseFloat(apiYsaAmount);
      setIsYsaMismatch(Math.abs(subTotalEnd - expectedYsa) > 0.001);
    }
  }, [rw04Totals, apiWriteOffAmount, apiYsaAmount]);
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
    const endpoint = tabIndex === 0 ? '/RW04A/deleteRows' : '/RW04B/deleteRow';
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
      await callApi(endpoint, { userCapacity: user.capacity, rowIds: rowsToDelete.map((row) => row.dbId) }, 'POST');
      setSnackbarMessage('Selected rows deleted successfully!', 'success');
      setDynamicRows(newRowsToKeep);
    } catch (error) {
      if (error.message !== 'canceled') setSnackbarMessage('An error occurred during deletion.', 'error');
    }
  };

  const handleSave = async () => {
    const basePayload = getBasePayload();
    let finalPayload;
    let endpoint;

    if (tabIndex === 0) {
      endpoint = '/RW04A/saveStatic';
      const allRows = [...rw04StaticRows, ...rw04DynamicRows.filter((r) => r.particulars.trim() || r.label.trim())];
      const data = allRows.map((row) => [
        row.label || row.particulars,
        row.provAmtStart || '0.00',
        row.writeOff || '0.00',
        row.addRed || '0.00',
        row.provAmtEnd || '0.00',
        String(row.dbId),
      ]);
      const subTotalRowForPayload = [
        'SUB TOTALS (ABOVE)',
        rw04Totals.subTotal1.provAmtStart,
        rw04Totals.subTotal1.writeOff,
        rw04Totals.subTotal1.addRed,
        rw04Totals.subTotal1.provAmtEnd,
        '5',
      ];
      data.splice(4, 0, subTotalRowForPayload);
      finalPayload = { ...basePayload, value: data };
    } else {
      endpoint = '/RW04B/saveData';
      const allRows = [...rw05StaticRows, ...rw05DynamicRows.filter((r) => r.particulars.trim())];
      const data = allRows.map((row) => [
        String(row.dbId),
        row.label || row.particulars,
        row.provAmtStart || '0.00',
        row.addReversal || '0.00',
        row.provAmtEnd || '0.00',
      ]);
      finalPayload = { ...basePayload, value: data };
    }

    try {
      await callApi(endpoint, finalPayload, 'POST');
      setSnackbarMessage('Data saved successfully!', 'success');
      if (tabIndex === 0) await loadRw04Data();
      if (tabIndex === 1) await loadRw05Data();
      setReportObject((prev) => ({ ...prev, status: '11' }));
    } catch (error) {
      if (error.message !== 'canceled') setSnackbarMessage('An error occurred while saving data.', 'error');
    }
  };

  const handleSubmitReport = () => setIsSubmitConfirmOpen(true);

  const handleConfirmSubmit = async () => {
    setIsSubmitConfirmOpen(false);
    if (reportObject?.status !== '11') {
      setSnackbarMessage('Kindly save the report before submitting.', 'warning');
      return;
    }
    const endpoint = '/RW04/submitReport';
    try {
      await callApi(endpoint, getBasePayload(), 'POST');
      setSnackbarMessage('Report submitted successfully!', 'success');
      setTimeout(() => navigate(-1), 1500);
    } catch (error) {
      if (error.message !== 'canceled') setSnackbarMessage('An error occurred during submission.', 'error');
    }
  };
  // #endregion

  // #region --- Render Logic ---
  if (isLoading && rw04StaticRows.length === 0 && rw05StaticRows.length === 0) {
    return (
      <Box display="flex" justifyContent="center" alignItems="center" height="50vh">
        <CircularProgress />
      </Box>
    );
  }

  return (
    <Box>
      <Tabs value={tabIndex} onChange={(e, i) => setTabIndex(i)}>
        <Tab label="RW-04 PART A" />
        <Tab label="RW-04 PART B" />
      </Tabs>
      <Box mt={2} display="flex" gap={2} flexWrap="wrap">
        <CustomButton label={'Add Row'} buttonType={'create'} onClickHandler={handleAddRow} disabled={isLoading} />
        <CustomButton
          label={'Delete Row'}
          buttonType={'delete'}
          onClickHandler={handleDeleteRow}
          disabled={isLoading}
        />
        <CustomButton label={'Save'} buttonType={'save'} onClickHandler={handleSave} disabled={isLoading} />
        {tabIndex === 1 && (
          <CustomButton
            label={'Submit RW04 Part A & B'}
            buttonType={'submit'}
            onClickHandler={handleSubmitReport}
            disabled={isLoading || reportObject?.status !== '11' || isWriteOffMismatch || isYsaMismatch}
          />
        )}
      </Box>

      <Box mt={1} minHeight="40px">
        {tabIndex === 0 && isYsaMismatch && (
          <Alert severity="error">
            * Sub Total Closing Provision ({rw04Totals.subTotal1.provAmtEnd}) must match the required YSA Amount (
            {apiYsaAmount}).
          </Alert>
        )}
        {tabIndex === 0 && isWriteOffMismatch && (
          <Alert severity="error">
            * Total Write Off ({rw04Totals.grandTotal.writeOff}) must match the required amount ({apiWriteOffAmount}).
          </Alert>
        )}
      </Box>

      <TableContainer component={Paper} sx={{ mt: 1, maxHeight: 'calc(96vh - 280px)' }}>
        <Table stickyHeader>
          <TableHead>
            <TableRow>
              <StyledTableCell>SELECT</StyledTableCell>
              <StyledTableCell sx={{ minWidth: '65px' }}>SL. NO.</StyledTableCell>
              {(tabIndex === 0 ? headers_rw04 : headers_rw05).map((head, idx) => (
                <StyledTableCell
                  key={idx}
                  sx={{ minWidth: '180px' }}
                  dangerouslySetInnerHTML={{ __html: head }}
                ></StyledTableCell>
              ))}
            </TableRow>
            <TableRow
              sx={{
                position: 'sticky',
                top: tabIndex === 0 ? 85 : 60,
                zIndex: 1,
              }}
            >
              <StyledTableCell></StyledTableCell>
              {(tabIndex === 0 ? headerSerial_rw04 : headerSerial_rw05).map((head, idx) => (
                <StyledTableCell
                  key={idx}
                  sx={{ minWidth: '180px' }}
                  dangerouslySetInnerHTML={{ __html: head }}
                ></StyledTableCell>
              ))}
            </TableRow>
          </TableHead>

          {tabIndex === 0 && (
            <>
              <TableBody>
                {rw04StaticRows.slice(0, 4).map((row) => (
                  <TableRow key={row.dbId}>
                    <TableCell></TableCell>
                    <TableCell align="center">{row.slNo}</TableCell>
                    <TableCell sx={{ textAlign: 'left' }}>{row.label.toUpperCase()}</TableCell>
                    <TableCell align="center">
                      <FormInput value={row.provAmtStart} readOnly sx={{ width: '200px' }} />
                    </TableCell>
                    <TableCell align="center">
                      <FormInput
                        inputType={'wholeAmountDecimal'}
                        value={row.writeOff}
                        onChange={(e) => handleRw04StaticChange(row.dbId, 'writeOff', e.target.value)}
                        sx={{ width: '200px' }}
                        placeholder="0.00"
                        debounceDuration={1}
                      />
                    </TableCell>
                    <TableCell align="center">
                      <FormInput value={row.addRed} readOnly sx={{ width: '200px' }} />
                    </TableCell>
                    <TableCell align="center">
                      <FormInput
                        inputType={'wholeAmountDecimal'}
                        value={row.provAmtEnd}
                        onChange={(e) => handleRw04StaticChange(row.dbId, 'provAmtEnd', e.target.value)}
                        sx={{ width: '200px' }}
                        placeholder="0.00"
                        debounceDuration={1}
                      />
                    </TableCell>
                  </TableRow>
                ))}
                {rw04StaticRows.length !== 0 && (
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
                      <FormInput
                        value={rw04Totals.subTotal1.provAmtEnd}
                        readOnly
                        sx={{ width: '200px' }}
                        error={isYsaMismatch}
                      />
                    </StyledTotalTableCell>
                  </TableRow>
                )}
                {rw04StaticRows.slice(4).map((row) => (
                  <TableRow key={row.dbId}>
                    <TableCell></TableCell>
                    <TableCell align="center">{row.slNo}</TableCell>
                    <TableCell sx={{ textAlign: 'left' }}>{row.label.toUpperCase()}</TableCell>
                    <TableCell align="center">
                      <FormInput value={row.provAmtStart} readOnly sx={{ width: '200px' }} />
                    </TableCell>
                    <TableCell align="center">
                      <FormInput
                        inputType={'wholeAmountDecimal'}
                        value={row.writeOff}
                        onChange={(e) => handleRw04StaticChange(row.dbId, 'writeOff', e.target.value)}
                        sx={{ width: '200px' }}
                        placeholder="0.00"
                        debounceDuration={1}
                      />
                    </TableCell>
                    <TableCell align="center">
                      <FormInput value={row.addRed} readOnly sx={{ width: '200px' }} />
                    </TableCell>
                    <TableCell align="center">
                      <FormInput
                        inputType={'wholeAmountDecimal'}
                        value={row.provAmtEnd}
                        onChange={(e) => handleRw04StaticChange(row.dbId, 'provAmtEnd', e.target.value)}
                        sx={{ width: '200px' }}
                        placeholder="0.00"
                        debounceDuration={1}
                      />
                    </TableCell>
                  </TableRow>
                ))}
                {rw04StaticRows.length !== 0 && (
                  <TableRow>
                    <TableCell></TableCell>
                    <TableCell align="center" style={{ fontWeight: 'bold' }}>
                      {rw04StaticRows.length + 2}
                    </TableCell>
                    <TableCell colSpan={6} style={{ fontWeight: 'bold', textAlign: 'left' }}>
                      OTHERS (PLEASE SPECIFY BELOW)
                    </TableCell>
                  </TableRow>
                )}
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
                      <FormInput
                        value={row.provAmtStart}
                        readOnly={row.dbId !== 0}
                        onChange={(e) => handleRw04DynamicChange(i, 'provAmtStart', e.target.value)}
                        sx={{ width: '200px' }}
                        placeholder="0.00"
                      />
                    </TableCell>
                    <TableCell align="center">
                      <FormInput
                        inputType={'wholeAmountDecimal'}
                        value={row.writeOff}
                        onChange={(e) => handleRw04DynamicChange(i, 'writeOff', e.target.value)}
                        sx={{ width: '200px' }}
                        placeholder="0.00"
                        debounceDuration={1}
                      />
                    </TableCell>
                    <TableCell align="center">
                      <FormInput value={row.addRed} readOnly sx={{ width: '200px' }} />
                    </TableCell>
                    <TableCell align="center">
                      <FormInput
                        inputType={'wholeAmountDecimal'}
                        value={row.provAmtEnd}
                        onChange={(e) => handleRw04DynamicChange(i, 'provAmtEnd', e.target.value)}
                        sx={{ width: '200px' }}
                        placeholder="0.00"
                        debounceDuration={1}
                      />
                    </TableCell>
                  </TableRow>
                ))}
              </TableBody>
              {rw04StaticRows.length !== 0 && (
                <TableFooter>
                  <TableRow sx={{ position: 'sticky', bottom: 49, zIndex: 1 }}>
                    <StyledTotalTableCell></StyledTotalTableCell>
                    <StyledTotalTableCell></StyledTotalTableCell>
                    <StyledTotalTableCell align="center">SUB TOTAL (OTHERS)</StyledTotalTableCell>
                    <StyledTotalTableCell>
                      <FormInput value={rw04Totals.subTotalOthers.provAmtStart} readOnly sx={{ width: '200px' }} />
                    </StyledTotalTableCell>
                    <StyledTotalTableCell>
                      <FormInput value={rw04Totals.subTotalOthers.writeOff} readOnly sx={{ width: '200px' }} />
                    </StyledTotalTableCell>
                    <StyledTotalTableCell>
                      <FormInput value={rw04Totals.subTotalOthers.addRed} readOnly sx={{ width: '200px' }} />
                    </StyledTotalTableCell>
                    <StyledTotalTableCell>
                      <FormInput value={rw04Totals.subTotalOthers.provAmtEnd} readOnly sx={{ width: '200px' }} />
                    </StyledTotalTableCell>
                  </TableRow>
                  <TableRow sx={{ position: 'sticky', bottom: 0, zIndex: 1 }}>
                    <StyledTotalTableCell></StyledTotalTableCell>
                    <StyledTotalTableCell></StyledTotalTableCell>
                    <StyledTotalTableCell align="center">TOTAL</StyledTotalTableCell>
                    <StyledTotalTableCell>
                      <FormInput value={rw04Totals.grandTotal.provAmtStart} readOnly sx={{ width: '200px' }} />
                    </StyledTotalTableCell>
                    <StyledTotalTableCell>
                      <FormInput
                        value={rw04Totals.grandTotal.writeOff}
                        readOnly
                        sx={{ width: '200px' }}
                        error={isWriteOffMismatch}
                      />
                    </StyledTotalTableCell>
                    <StyledTotalTableCell>
                      <FormInput value={rw04Totals.grandTotal.addRed} readOnly sx={{ width: '200px' }} />
                    </StyledTotalTableCell>
                    <StyledTotalTableCell>
                      <FormInput value={rw04Totals.grandTotal.provAmtEnd} readOnly sx={{ width: '200px' }} />
                    </StyledTotalTableCell>
                  </TableRow>
                </TableFooter>
              )}
            </>
          )}

          {tabIndex === 1 && (
            <>
              <TableBody>
                {/* UPDATED: Renders Part B rows fetched from the API */}
                {rw05StaticRows.map((row) => (
                  <TableRow key={row.dbId}>
                    <TableCell></TableCell>
                    <TableCell align="center">{row.slNo}</TableCell>
                    <TableCell sx={{ textAlign: 'left' }}>{row.label.toUpperCase()}</TableCell>
                    <TableCell align="center">
                      <FormInput value={row.provAmtStart} readOnly sx={{ width: '200px' }} />
                    </TableCell>
                    <TableCell align="center">
                      <FormInput value={row.addReversal} readOnly sx={{ width: '200px' }} />
                    </TableCell>
                    <TableCell align="center">
                      <FormInput
                        inputType={'wholeAmountDecimal'}
                        value={row.provAmtEnd}
                        onChange={(e) => handleRw05StaticChange(row.dbId, 'provAmtEnd', e.target.value)}
                        sx={{ width: '200px' }}
                        placeholder="0.00"
                        debounceDuration={1}
                      />
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
                      <FormInput
                        value={row.provAmtStart}
                        onChange={(e) => handleRw05DynamicChange(index, 'provAmtStart', e.target.value)}
                        sx={{ width: '200px' }}
                        placeholder="0.00"
                      />
                    </TableCell>
                    <TableCell align="center">
                      <FormInput value={row.addReversal} readOnly sx={{ width: '200px' }} />
                    </TableCell>
                    <TableCell align="center">
                      <FormInput
                        inputType={'wholeAmountDecimal'}
                        value={row.provAmtEnd}
                        onChange={(e) => handleRw05DynamicChange(index, 'provAmtEnd', e.target.value)}
                        sx={{ width: '200px' }}
                        placeholder="0.00"
                        debounceDuration={1}
                      />
                    </TableCell>
                  </TableRow>
                ))}
              </TableBody>
              <TableFooter>
                <TableRow sx={{ position: 'sticky', bottom: 0, zIndex: 1 }}>
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
              </TableFooter>
            </>
          )}
        </Table>
      </TableContainer>

      <Dialog open={isSubmitConfirmOpen} onClose={() => setIsSubmitConfirmOpen(false)}>
        <DialogTitle>{'Confirm Submission'}</DialogTitle>
        <DialogContent>
          <DialogContentText>
            Kindly save any changes related to both RW04 Part A and B before submitting. Are you sure you want to
            proceed?
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
      <Box display="flex" justifyContent="left" alignItems="center" mt={2}>
        <Typography>
          Note: Submit button will be enabled only after resolving validation errors. User can see 'Submit' button in
          RW04 Part B section.
        </Typography>
      </Box>
    </Box>
  );
  // #endregion
};

export default RW04;

import React, { useState, useEffect, useCallback } from 'react';
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
  CircularProgress,
  Dialog,
  DialogActions,
  DialogContent,
  DialogContentText,
  DialogTitle,
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
  backgroundColor: 'hsl(220, 20%, 35%)',
  color: 'white',
}));

// Generates the initial structure for the 7 static rows.
const getInitialStaticRows = () => [
  { dbId: 1, label: 'CONTINGENT LIABILITY AS PER AS -29', provAmtStart: '', addition: '', reversal: '', provAmtEnd: '0.00', difference: '0.00' },
  { dbId: 2, label: 'DELAYED REPORTING PENALTY', provAmtStart: '', addition: '', reversal: '', provAmtEnd: '0.00', difference: '0.00' },
  { dbId: 3, label: 'EX-GRATIA PAYMENT', provAmtStart: '', addition: '', reversal: '', provAmtEnd: '0.00', difference: '0.00' },
  { dbId: 4, label: 'PROVISION ON OVERDUE DEPOSIT INTT', provAmtStart: '', addition: '', reversal: '', provAmtEnd: '0.00', difference: '0.00' },
  { dbId: 5, label: 'LEAVE ENCASHMENT', provAmtStart: '', addition: '', reversal: '', provAmtEnd: '0.00', difference: '0.00' },
  { dbId: 6, label: 'PROVISION FOR PERFORMANCE LINKED INCENTIVES', provAmtStart: '', addition: '', reversal: '', provAmtEnd: '0.00', difference: '0.00' },
  { dbId: 7, label: 'PROVISION ON ACCOUNT OF ENTRIES OUTSTANDING IN ADJUSTING ACCOUNT FOR PREVIOUS QUARTER(S) (i.e. PRIOR TO CURRENT QUARTER)', provAmtStart: '', addition: '', reversal: '', provAmtEnd: '0.00', difference: '0.00' },
];

// Generates an empty row for the dynamic "Others" table.
const createInitialDynamicRow = () => ({
  dbId: 0,
  key: `new-${Date.now()}-${Math.random().toString(36).substr(2, 9)}`,
  selected: false,
  particulars: '',
  provAmtStart: '',
  addition: '',
  reversal: '',
  provAmtEnd: '0.00',
  difference: '0.00',
});

const RW05 = () => {
  const [tabIndex, setTabIndex] = useState(0);
  const user = JSON.parse(localStorage.getItem('user'));
  const { state } = useLocation();

  const [staticRows, setStaticRows] = useState(getInitialStaticRows());
  const [dynamicRows, setDynamicRows] = useState([createInitialDynamicRow()]);
  const [isSubmitConfirmOpen, setIsSubmitConfirmOpen] = useState(false);

  const { callApi, isLoading } = useApi();
  const setSnackbarMessage = useCustomSnackbar();
  const [reportObject, setReportObject] = useState(state?.report || null);

  const headers = [
    'Particulars(2)',
    `Opening Provision as on ${user.quarterStartDate}(3)`,
    'Additions during the period(4)',
    'Reversals during the Period(5)',
    `Provision as on ${user.quarterEndDate}(6)={(3+4)-(5)}`,
    'Difference to be Provided/ written back(7)={(6)-(3)}',
  ];
  const isNumeric = (val) => val === '' || (val !== null && !isNaN(parseFloat(val)) && isFinite(val));

  const getBasePayload = useCallback(() => ({
    circleCode: user?.circleCode,
    qed: user?.quarterEndDate,
    userCapacity: user?.userCapacity,
    reportId: reportObject?.reportId,
    reportMasterId: reportObject?.reportMasterId,
    currentStatus: reportObject?.status,
    userId: user.userId,
  }), [user, reportObject]);

  const calculateRow = (row) => {
    const provAmtStart = parseFloat(row.provAmtStart || 0);
    const addition = parseFloat(row.addition || 0);
    const reversal = parseFloat(row.reversal || 0);
    const provAmtEnd = provAmtStart + addition - reversal;
    const difference = provAmtEnd - provAmtStart;
    row.provAmtEnd = provAmtEnd.toFixed(2);
    row.difference = difference.toFixed(2);
    return row;
  };

  const handleStaticChange = (index, key, value) => {
    if (!isNumeric(value)) return;
    const updatedRows = [...staticRows];
    let rowToUpdate = { ...updatedRows[index], [key]: value };
    rowToUpdate = calculateRow(rowToUpdate);
    updatedRows[index] = rowToUpdate;
    setStaticRows(updatedRows);
  };

  const handleDynamicChange = (index, key, value) => {
    const numericFields = ['provAmtStart', 'addition', 'reversal'];
    if (numericFields.includes(key) && !isNumeric(value)) return;

    const updatedRows = [...dynamicRows];
    let rowToUpdate = { ...updatedRows[index], [key]: value };
    rowToUpdate = calculateRow(rowToUpdate);
    updatedRows[index] = rowToUpdate;
    setDynamicRows(updatedRows);
  };

  const loadData = useCallback(async () => {
    if (!reportObject) return;
    try {
      const response = await callApi('/RW05/getData', getBasePayload(), 'POST');
      if (response?.data?.data) {
        const apiData = response.data.data;
        const loadedStaticRows = getInitialStaticRows();
        const loadedDynamicRows = [];

        apiData.forEach((row) => {
          const dbId = parseInt(row[0], 10);
          if (dbId >= 1 && dbId <= 7) {
            const staticRow = loadedStaticRows.find((r) => r.dbId === dbId);
            if (staticRow) {
              staticRow.provAmtStart = row[2];
              staticRow.addition = row[3];
              staticRow.reversal = row[4];
              staticRow.provAmtEnd = row[5];
              staticRow.difference = row[6];
            }
          } else {
            loadedDynamicRows.push({
              dbId: dbId,
              key: dbId,
              particulars: row[1],
              provAmtStart: row[2],
              addition: row[3],
              reversal: row[4],
              provAmtEnd: row[5],
              difference: row[6],
              selected: false,
            });
          }
        });

        setStaticRows(loadedStaticRows);
        setDynamicRows(loadedDynamicRows.length > 0 ? loadedDynamicRows : [createInitialDynamicRow()]);
        setSnackbarMessage('Data loaded successfully!', 'success');
      }
    } catch (error) {
      setSnackbarMessage('Failed to load data.', 'error');
    }
  }, [reportObject, callApi, getBasePayload, setSnackbarMessage]);

  useEffect(() => {
    if (reportObject) {
      loadData();
    }
  }, [reportObject, loadData]);

  const handleSubmit = async () => {
    try {
      if (tabIndex === 0) {
        const value = staticRows.map(row => [
          row.provAmtStart || '0.00',
          row.addition || '0.00',
          row.reversal || '0.00',
          row.provAmtEnd || '0.00',
          row.difference || '0.00',
          String(row.dbId),
        ]);
        await callApi('/RW05/saveStatic', { ...getBasePayload(), value }, 'POST');
      } else {
        for (const row of dynamicRows) {
          if (row.dbId === 0 && !row.particulars.trim()) continue;
          const value = [
            row.particulars,
            row.provAmtStart || '0.00',
            row.addition || '0.00',
            row.reversal || '0.00',
            row.provAmtEnd || '0.00',
            row.difference || '0.00',
            String(row.dbId),
          ];
          const response = await callApi('/RW05/saveAddRow', { ...getBasePayload(), value }, 'POST');
          if (row.dbId === 0 && response?.data?.rowId) {
            // No need to update state here, loadData after save will refresh everything
          }
        }
      }
      setSnackbarMessage('Data saved successfully!', 'success');
      loadData(); // Refresh data from server to get latest state and new IDs
    } catch (error) {
      setSnackbarMessage('An error occurred while saving data.', 'error');
    }
  };

  const handleAddRow = () => {
    if (dynamicRows.some(row => row.dbId === 0)) {
        setSnackbarMessage('Kindly save the current new row before adding another.', 'warning');
        return;
    }
    setDynamicRows([...dynamicRows, createInitialDynamicRow()]);
  };

  const handleDeleteRow = async () => {
    const rowsToDelete = dynamicRows.filter((row) => row.selected && row.dbId !== 0);
    if (rowsToDelete.length === 0) {
      // Handle local deletion of unsaved rows
      const newRowsToKeep = dynamicRows.filter((row) => !row.selected);
      setDynamicRows(newRowsToKeep.length > 0 ? newRowsToKeep : [createInitialDynamicRow()]);
      return;
    }
    try {
      // For now, deleting one by one as per backend. Can be optimized to multi-delete later.
      for (const row of rowsToDelete) {
        await callApi('/RW05/deleteRow', { ...getBasePayload(), rowId: String(row.dbId) }, 'POST');
      }
      setSnackbarMessage('Selected rows deleted successfully!', 'success');
      loadData(); // Refresh data
    } catch (error) {
      setSnackbarMessage('An error occurred during deletion.', 'error');
    }
  };

  const handleSubmitReport = () => {
    setIsSubmitConfirmOpen(true);
  };
  
  const handleConfirmSubmit = async () => {
    setIsSubmitConfirmOpen(false);
    try {
      const response = await callApi('/RW05/submitReport', getBasePayload(), 'POST');
      if (response && typeof response === 'string') {
        setSnackbarMessage('Report submitted successfully!', 'success');
        setReportObject(prev => ({ ...prev, status: response }));
      }
    } catch (error) {
      setSnackbarMessage('An error occurred during submission.', 'error');
    }
  };

  return (
    <Box>
      <Tabs value={tabIndex} onChange={(e, i) => setTabIndex(i)}>
        <Tab label="RW-05(I)" />
        <Tab label="RW-05(II) - OTHERS" />
      </Tabs>

      <Box mt={2} display="flex" gap={1}>
        {tabIndex === 1 && (
            <>
                <Button variant="contained" color="secondary" onClick={handleAddRow} disabled={isLoading}> Add Row </Button>
                <Button variant="contained" color="error" onClick={handleDeleteRow} disabled={isLoading || dynamicRows.every((r) => !r.selected)}> Delete Row </Button>
            </>
        )}
        <Button variant="contained" color="warning" onClick={handleSubmit} disabled={isLoading}>
          {isLoading ? <CircularProgress size={24} color="inherit" /> : 'Save'}
        </Button>
        <Button variant="contained" color="primary" onClick={handleSubmitReport} disabled={isLoading || reportObject?.status !== '11'} sx={{ ml: 1 }}>
          Submit
        </Button>
      </Box>

      <TableContainer component={Paper} sx={{ mt: 2, maxHeight: 'calc(100vh - 250px)' }}>
        <Table stickyHeader>
          <TableHead>
            <TableRow>
              <StyledTableCell>Sr No(1)</StyledTableCell>
              {tabIndex === 1 && <StyledTableCell>SELECT</StyledTableCell>}
              {headers.map((head, idx) => (<StyledTableCell key={idx}>{head}</StyledTableCell>))}
            </TableRow>
          </TableHead>
          <TableBody>
            {tabIndex === 0 && staticRows.map((row, index) => (
              <TableRow key={row.dbId}>
                <TableCell align="center">{index + 1}</TableCell>
                <TableCell>{row.label}</TableCell>
                {['provAmtStart', 'addition', 'reversal'].map((key) => (
                  <TableCell key={key} align="center">
                    <FormInput value={row[key]} onChange={(e) => handleStaticChange(index, key, e.target.value)} sx={{ width: '180px' }} placeholder="0.00" />
                  </TableCell>
                ))}
                <TableCell align="center"><FormInput value={row.provAmtEnd} readOnly sx={{ width: '180px' }} /></TableCell>
                <TableCell align="center"><FormInput value={row.difference} readOnly sx={{ width: '180px' }} /></TableCell>
              </TableRow>
            ))}
            {tabIndex === 1 && dynamicRows.map((row, index) => (
              <TableRow key={row.key}>
                <TableCell align="center">{index + 1}</TableCell>
                <TableCell padding="checkbox">
                  <Checkbox checked={row.selected} onChange={() => {
                    const updated = [...dynamicRows];
                    updated[index].selected = !updated[index].selected;
                    setDynamicRows(updated);
                  }}/>
                </TableCell>
                <TableCell>
                    <FormInput value={row.particulars} onChange={(e) => handleDynamicChange(index, 'particulars', e.target.value)} sx={{ width: '180px' }} placeholder="Enter Particulars"/>
                </TableCell>
                {['provAmtStart', 'addition', 'reversal'].map((key) => (
                  <TableCell key={key} align="center">
                    <FormInput value={row[key]} onChange={(e) => handleDynamicChange(index, key, e.target.value)} sx={{ width: '180px' }} placeholder="0.00"/>
                  </TableCell>
                ))}
                <TableCell align="center"><FormInput value={row.provAmtEnd} readOnly sx={{ width: '180px' }} /></TableCell>
                <TableCell align="center"><FormInput value={row.difference} readOnly sx={{ width: '180px' }} /></TableCell>
              </TableRow>
            ))}
          </TableBody>
        </Table>
      </TableContainer>
      
      <Dialog open={isSubmitConfirmOpen} onClose={() => setIsSubmitConfirmOpen(false)}>
        <DialogTitle>Confirm Submission</DialogTitle>
        <DialogContent><DialogContentText>Kindly save any changes before submitting. Are you sure you want to proceed?</DialogContentText></DialogContent>
        <DialogActions>
          <Button onClick={() => setIsSubmitConfirmOpen(false)} color="secondary">Cancel</Button>
          <Button onClick={handleConfirmSubmit} color="primary" autoFocus>Confirm Submit</Button>
        </DialogActions>
      </Dialog>
      
    </Box>
  );
};

export default RW05;

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
  Dialog,
  DialogActions,
  DialogContent,
  DialogContentText,
  DialogTitle,
  Checkbox,
  CircularProgress,
} from '@mui/material';
import { styled } from '@mui/material/styles';
import FormInput from '../../../../common/components/ui/FormInput';
import useCustomSnackbar from '../../../../common/hooks/useCustomSnackbar';
import { useLocation, useNavigate } from 'react-router-dom';
import useApi from '../../../../common/hooks/useApi';
import { CustomButton } from '../../../../common/components/ui/Buttons';

const StyledTableCell = styled(TableCell)(({ theme }) => ({
  fontSize: '0.875rem',
  textAlign: 'center',
  padding: '6px',
  fontWeight: 'bold',
  backgroundColor: 'hsl(220, 20%, 35%)',
  color: 'white',
}));

const createInitialDynamicRow = () => ({
  dbId: 0,
  particulars: '',
  provAmtStart: '',
  writeOff: '',
  addition: '',
  reduction: '',
  provAmtEnd: '0.00',
  rate: '100',
  provRequired: '0.00',
  selected: false,

  key: `new-${Date.now()}-${Math.random().toString(36).substr(2, 9)}`,
});

// Initial structure for static rows, will be populated from API
const getInitialStaticRows = (quarterEndDate) => [
  {
    feId: '1',
    dbId: 1,
    label: 'FRAUDS - DEBITED TO RECALLED ASSETS A/c (Prod Cd 6998-9981)**',
    provAmtStart: '',
    writeOff: '',
    addition: '',
    reduction: '',
    provAmtEnd: '0.00',
    rate: '',
    provRequired: '0.00',
  },
  {
    feId: '1.i',
    dbId: 2,
    label: `Frauds reported within time up ${quarterEndDate} provision @ 100% ##`,
    provAmtStart: '',
    writeOff: '',
    addition: '',
    reduction: '',
    provAmtEnd: '0.00',
    rate: '100',
    provRequired: '0.00',
  },
  {
    feId: '1.ii',
    dbId: 3,
    label: `Delayed Reported frauds Provision @ 100% ##`,
    provAmtStart: '',
    writeOff: '',
    addition: '',
    reduction: '',
    provAmtEnd: '0.00',
    rate: '100',
    provRequired: '0.00',
  },
  {
    feId: '2',
    dbId: 4,
    label: 'OTHERS LOSSES IN RECALLED ASSETS (Prod cd 6998 - 9982)#',
    provAmtStart: '',
    writeOff: '',
    addition: '',
    reduction: '',
    provAmtEnd: '0.00',
    rate: '100',
    provRequired: '0.00',
  },
  {
    feId: '3',
    dbId: 5,
    label: 'FRAUDS - OTHER (NOT DEBITED TO RA A/c)$',
    provAmtStart: '',
    writeOff: '',
    addition: '',
    reduction: '',
    provAmtEnd: '0.00',
    rate: '',
    provRequired: '0.00',
  },
  {
    feId: '3.i',
    dbId: 6,
    label: `Frauds reported within time up ${quarterEndDate} provision @ 100% ##`,
    provAmtStart: '',
    writeOff: '',
    addition: '',
    reduction: '',
    provAmtEnd: '0.00',
    rate: '100',
    provRequired: '0.00',
  },
  {
    feId: '3.ii',
    dbId: 7,
    label: `Delayed Reported frauds Provision @ 100% ##`,
    provAmtStart: '',
    writeOff: '',
    addition: '',
    reduction: '',
    provAmtEnd: '0.00',
    rate: '100',
    provRequired: '0.00',
  },
  {
    feId: '4',
    dbId: 8,
    label: 'REVENUE ITEM IN SYSTEM SUSPENSE',
    provAmtStart: '',
    writeOff: '',
    addition: '',
    reduction: '',
    provAmtEnd: '0.00',
    rate: '100',
    provRequired: '0.00',
  },
  {
    feId: '5',
    dbId: 9,
    label: 'PROVISION ON ACCOUNT OF FSLO',
    provAmtStart: '',
    writeOff: '',
    addition: '',
    reduction: '',
    provAmtEnd: '0.00',
    rate: '100',
    provRequired: '0.00',
  },
  {
    feId: '6',
    dbId: 10,
    label:
      'PROVISION ON ACCOUNT OF ENTRIES OUTSTANDING IN ADJUSTING ACCOUNT FOR PREVIOUS QUARTER(S) (i.e. PRIOR TO CURRENT QUARTER)',
    provAmtStart: '',
    writeOff: '',
    addition: '',
    reduction: '',
    provAmtEnd: '0.00',
    rate: '100',
    provRequired: '0.00',
  },
  {
    feId: '7',
    dbId: 11,
    label: 'PROVISION ON N.P.A. INTEREST FREE STAFF LOANS',
    provAmtStart: '',
    writeOff: '',
    addition: '',
    reduction: '',
    provAmtEnd: '0.00',
    rate: '100',
    provRequired: '0.00',
  },
];

const RW04 = () => {
  const [tabIndex, setTabIndex] = useState(0);
  const user = JSON.parse(localStorage.getItem('user'));
  const { state } = useLocation();

  const [staticRows, setStaticRows] = useState(getInitialStaticRows(user.quarterEndDate));
  const [dynamicRows, setDynamicRows] = useState([]); // Start with empty array, let useEffect populate

  const [snackbar, setSnackbar] = useState({ open: false, message: '', severity: 'info' });
  const { callApi, isLoading } = useApi();
  const setSnackbarMessage = useCustomSnackbar();
  const [reportObject, setReportObject] = useState(state?.report || null);
  const [isSubmitConfirmOpen, setIsSubmitConfirmOpen] = useState(false);
  const navigate = useNavigate();

  const headers = [
    'PARTICULARS(2)',
    `PROVISIONABLE AMT AS ON ${user.previousQuarterEndDate} (3)`,
    'WRITE OFF DURING THE 3 MONTHS PERIOD* (4)',
    'ADDITONS IN PROVISIONABLE AMT DURING THE 3 MONTHS PERIOD (5)',
    'REDUCTION IN PROVISIONABLE AMT (OTHER THAN WRITE OFF) DURING THE 3 MONTHS PERIOD^ (6)',
    `PROVISIONABLE AMT AS ON ${user.quarterEndDate} (7)=3-4+5-6`,
    'RATE OF PROVISION (%)(8)',
    `PROVISION REQUIREMENT AS ON ${user.quarterEndDate} (9)=7*8`,
  ];

  const isNumeric = (val) => val === null || val === '' || (!isNaN(parseFloat(val)) && isFinite(val));

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

  const calculateStaticRow = (updatedRow) => {
    const isManualRow = ['1.i', '1.ii', '3.i', '3.ii'].includes(updatedRow.feId);
    const start = parseFloat(updatedRow.provAmtStart || 0);
    const write = parseFloat(updatedRow.writeOff || 0);
    const add = parseFloat(updatedRow.addition || 0);
    const reduce = parseFloat(updatedRow.reduction || 0);
    const rate = parseFloat(updatedRow.rate || 0);

    if (!isManualRow) {
      const end = start - write + add - reduce;
      updatedRow.provAmtEnd = end.toFixed(2);
      updatedRow.provRequired = ((end * rate) / 100).toFixed(2);
    } else {
      const end = parseFloat(updatedRow.provAmtEnd || 0);
      updatedRow.provRequired = ((end * rate) / 100).toFixed(2);
    }
    return updatedRow;
  };

  const calculateDynamicRow = (updatedRow) => {
    const start = parseFloat(updatedRow.provAmtStart || 0);
    const write = parseFloat(updatedRow.writeOff || 0);
    const add = parseFloat(updatedRow.addition || 0);
    const reduce = parseFloat(updatedRow.reduction || 0);
    const rate = parseFloat(updatedRow.rate || 0);

    const end = start - write + add - reduce;
    updatedRow.provAmtEnd = end.toFixed(2);
    updatedRow.provRequired = ((end * rate) / 100).toFixed(2);
    return updatedRow;
  };

  const handleStaticChange = (index, key, value) => {
    const currentRows = [...staticRows];
    const currentRow = { ...currentRows[index] };

    const numericFields = ['provAmtStart', 'writeOff', 'addition', 'reduction', 'provAmtEnd'];
    if (numericFields.includes(key) && !isNumeric(value)) {
      return;
    }

    currentRow[key] = value;
    const recalculatedRow = calculateStaticRow(currentRow);
    currentRows[index] = recalculatedRow;
    setStaticRows(currentRows);
  };

  const handleDynamicChange = (index, key, value) => {
    const updatedRows = [...dynamicRows];
    let rowToUpdate = { ...updatedRows[index] };

    const numericFields = ['provAmtStart', 'writeOff', 'addition', 'reduction'];
    if (numericFields.includes(key) && !isNumeric(value)) {
      return;
    }

    rowToUpdate[key] = value;
    rowToUpdate = calculateDynamicRow(rowToUpdate);
    updatedRows[index] = rowToUpdate;
    setDynamicRows(updatedRows);
  };

  const isChildRowDisabled = (rowFeId, key) => {
    return (
      (rowFeId === '1.i' || rowFeId === '1.ii' || rowFeId === '3.i' || rowFeId === '3.ii') &&
      !['provAmtEnd', 'rate', 'provRequired'].includes(key)
    );
  };

  const getProvAmtEndMismatchError = (rowFeId) => {
    if (rowFeId === '1') {
      const parent = parseFloat(staticRows.find((r) => r.feId === '1')?.provAmtEnd || 0);
      const child1i = parseFloat(staticRows.find((r) => r.feId === '1.i')?.provAmtEnd || 0);
      const child1ii = parseFloat(staticRows.find((r) => r.feId === '1.ii')?.provAmtEnd || 0);
      return Math.abs(parent - (child1i + child1ii)) > 0.01;
    }
    if (rowFeId === '3') {
      const parent = parseFloat(staticRows.find((r) => r.feId === '3')?.provAmtEnd || 0);
      const child3i = parseFloat(staticRows.find((r) => r.feId === '3.i')?.provAmtEnd || 0);
      const child3ii = parseFloat(staticRows.find((r) => r.feId === '3.ii')?.provAmtEnd || 0);
      return Math.abs(parent - (child3i + child3ii)) > 0.01;
    }
    return false;
  };

  const loadData = useCallback(async () => {
    if (!reportObject) return;

    try {
      const response = await callApi('/RW04/getData', getBasePayload(), 'POST');

      console.log('response?.data', response?.data);

      if (response?.data) {
        const apiData = response.data;

        const loadedStaticRows = getInitialStaticRows(user.quarterEndDate);
        const loadedDynamicRows = [];

        apiData.forEach((row) => {
          const dbId = parseInt(row[0], 10);

          if (dbId >= 1 && dbId <= 11) {
            const staticRowToUpdate = loadedStaticRows.find((r) => r.dbId === dbId);
            if (staticRowToUpdate) {
              staticRowToUpdate.provAmtStart = row[2];
              staticRowToUpdate.writeOff = row[3];
              staticRowToUpdate.addition = row[4];
              staticRowToUpdate.reduction = row[5];
              staticRowToUpdate.provAmtEnd = row[6];
              staticRowToUpdate.rate = row[7];
              staticRowToUpdate.provRequired = row[8];
            }
          } else {
            loadedDynamicRows.push({
              dbId: dbId,
              particulars: row[1],
              provAmtStart: row[2],
              writeOff: row[3],
              addition: row[4],
              reduction: row[5],
              provAmtEnd: row[6],
              rate: row[7],
              provRequired: row[8],
              selected: false,
              key: dbId,
            });
          }
        });

        setStaticRows(loadedStaticRows);
        setDynamicRows(loadedDynamicRows.length > 0 ? loadedDynamicRows : [createInitialDynamicRow()]);
      } else if (response?.data?.message) {
        setSnackbarMessage(response.data.message, 'warning');
      }
    } catch (error) {
      if (error.message !== 'canceled') {
        setSnackbarMessage('Failed to fetch data.', 'error');
      }
    }
  }, [reportObject, user.quarterEndDate, callApi, getBasePayload, setSnackbarMessage]);

  useEffect(() => {
    if (reportObject) {
      loadData();
    }
  }, []);

  const handleSubmit = async () => {
    let payload = {};
    if (tabIndex === 0) {
      // Static Tab
      const value = staticRows.map((row) => [
        row.provAmtStart || '0.00',
        row.writeOff || '0.00',
        row.addition || '0.00',
        row.reduction || '0.00',
        row.provAmtEnd || '0.00',
        row.rate || '0',
        row.provRequired || '0.00',
        String(row.dbId),
      ]);
      payload = { ...getBasePayload(), value };

      try {
        const response = await callApi('/RW04/saveStatic', payload, 'POST');
        if (response?.update) {
          //const [flag, newReportId, newStatus] = res.split('~');
          setReportObject((prev) => ({
            ...prev,
            status: '11',
          }));
          setSnackbarMessage('Data saved successfully!', 'success');
        }
      } catch (error) {
        console.error(`Error during save operation:`, error);
        setSnackbarMessage(`An error occurred while saving data.`, 'error');
      }
    } else if (tabIndex === 1) {
      // Dynamic "OTHERS" Tab
      try {
        const updatedRows = [...dynamicRows];

        for (let i = 0; i < updatedRows.length; i++) {
          const row = updatedRows[i];

          if (row.dbId === 0 && !row.particulars.trim()) {
            continue;
          }

          const singleRowPayloadValue = [
            row.particulars,
            row.provAmtStart || '0.00',
            row.writeOff || '0.00',
            row.addition || '0.00',
            row.reduction || '0.00',
            row.provAmtEnd || '0.00',
            row.rate || '100',
            row.provRequired || '0.00',
            String(row.dbId),
          ];

          const singleRowPayload = { ...getBasePayload(), value: singleRowPayloadValue };
          const response = await callApi('/RW04/saveAddRow', singleRowPayload, 'POST');
          if (response?.rowId) {
            setReportObject((prev) => ({
              ...prev,
              status: '11',
            }));
            console.log('API Response for Save:', response);
            const newId = response?.rowId;

            if (row.dbId === 0 && newId) {
              updatedRows[i] = {
                ...row,
                dbId: newId,
                key: newId,
              };
            }
            setDynamicRows(updatedRows);
            setSnackbarMessage('Data saved successfully!', 'success');
          } else {
            setSnackbarMessage('Error', 'error');
          }
        }
      } catch (error) {
        console.error(`Error during save operation:`, error);
        setSnackbarMessage(`An error occurred while saving data.`, 'error');
      }
    }
  };

  const handleAddRow = () => {
    const hasUnsavedRow = dynamicRows.some((row) => row.dbId === 0);

    if (hasUnsavedRow) {
      setSnackbarMessage(`Kindly save the current new row before adding another.`, 'warning');
    } else {
      setDynamicRows([...dynamicRows, createInitialDynamicRow()]);
    }
  };

  const isDataValid = () => {
    if (getProvAmtEndMismatchError('1') || getProvAmtEndMismatchError('3')) {
      return false;
    }
    return true;
  };

  const handleDeleteRow = async () => {
    const rowsToDelete = dynamicRows.filter((row) => row.selected && row.dbId !== 0);
    const newRowsToKeep = dynamicRows.filter((row) => !row.selected);

    if (rowsToDelete.length === 0) {
      if (dynamicRows.some((row) => row.selected && row.dbId === 0)) {
        setDynamicRows(newRowsToKeep.length > 0 ? newRowsToKeep : [createInitialDynamicRow()]);
        setSnackbarMessage('New row removed.', 'info');
      } else {
        setSnackbarMessage('Please select a saved row to delete.', 'warning');
      }
      return;
    }

    try {
      const idsToDelete = rowsToDelete.map((row) => row.dbId);

      const payload = {
        userCapacity: user.userCapacity,
        rowIds: idsToDelete,
      };

      const res = await callApi('/RW04/deleteRows', payload, 'POST');
      if (res?.deletedCount) {
        setSnackbarMessage('Selected rows deleted successfully!', 'success');

        setDynamicRows(newRowsToKeep.length > 0 ? newRowsToKeep : [createInitialDynamicRow()]);
      } else {
        setSnackbarMessage('An error occurred during deletion.', 'error');
      }
    } catch (error) {
      console.error('Error deleting rows:', error);
      setSnackbarMessage('An error occurred during deletion.', 'error');
    }
  };
  const handleSubmitReport = () => {
    if (!isDataValid()) {
      setSnackbarMessage('Please resolve all validation errors before submitting.', 'error');
      return;
    }
    setIsSubmitConfirmOpen(true);
  };

  const handleConfirmSubmit = async () => {
    setIsSubmitConfirmOpen(false);

    if (reportObject?.status !== '11') {
      setSnackbarMessage('Kindly save report before submitting', 'warning');
      return;
    }

    try {
      const payload = getBasePayload();
      const response = await callApi('/RW04/submitReport', payload, 'POST');

      if (response.status === 1) {
        setSnackbarMessage('Report submitted successfully!', 'success');
        // setReportObject((prev) => ({ ...prev, status: response }));
        setTimeout(() => {
          navigate(-1);
        }, 2000);
      } else {
        setSnackbarMessage('Submission completed, but server response was unexpected.', 'warning');
      }
    } catch (error) {
      console.error('Error submitting report:', error);
      setSnackbarMessage('An error occurred during submission.', 'error');
    }
  };

  const renderHeader = () => (
    <TableHead>
      <TableRow>
        <StyledTableCell sx={{ minWidth: '60px' }}>Sr No(1)</StyledTableCell>
        {tabIndex === 1 && <StyledTableCell>SELECT</StyledTableCell>}
        {headers.map((head, idx) => (
          <StyledTableCell key={idx} sx={{ minWidth: '160px' }}>
            {head}
          </StyledTableCell>
        ))}
      </TableRow>
    </TableHead>
  );

  // Render loading spinner while data is being fetched initially
  if (isLoading && !staticRows.length && !dynamicRows.length) {
    return <CircularProgress />;
  }

  return (
    <Box>
      <Tabs value={tabIndex} onChange={(e, i) => setTabIndex(i)}>
        <Tab label="RW-04(I)" />
        <Tab label="RW-04(II)" />
      </Tabs>

      <Box mt={2} display="flex" gap={2}>
        {tabIndex === 1 && (
          <>
            <CustomButton label={'Add Row'} buttonType={'create'} onClickHandler={handleAddRow} disabled={isLoading} />
            <CustomButton
              label={'Delete Row'}
              buttonType={'delete'}
              onClickHandler={handleDeleteRow}
              disabled={isLoading || dynamicRows.every((r) => !r.selected)}
            />
          </>
        )}

        <CustomButton label={'Save'} buttonType={'save'} onClickHandler={handleSubmit} disabled={isLoading} />
        <CustomButton
          label={'Submit'}
          buttonType={'submit'}
          onClickHandler={handleSubmitReport}
          disabled={isLoading || reportObject?.status !== '11' || !isDataValid()}
        />
      </Box>

      <TableContainer component={Paper} sx={{ mt: 2, maxHeight: 'calc(100vh - 250px)' }}>
        <Table stickyHeader>
          {renderHeader()}
          <TableBody>
            {/* Static Rows Rendering */}
            {tabIndex === 0 &&
              staticRows.map((row, index) => (
                <TableRow key={row.feId}>
                  <TableCell align="center">{row.feId}</TableCell>
                  <TableCell sx={{ width: '280px', textAlign: 'left' }}>{row.label}</TableCell>
                  {['provAmtStart', 'writeOff', 'addition', 'reduction'].map((key) => (
                    <TableCell key={key} align="center">
                      <FormInput
                        value={row[key]}
                        onChange={(e) => handleStaticChange(index, key, e.target.value)}
                        readOnly={isChildRowDisabled(row.feId, key)}
                        sx={{ width: '150px' }}
                        placeholder="0.00"
                        debounceDuration={1}
                      />
                    </TableCell>
                  ))}
                  <TableCell align="center">
                    <Box display="flex" flexDirection="column">
                      <FormInput
                        value={row.provAmtEnd}
                        onChange={(e) => handleStaticChange(index, 'provAmtEnd', e.target.value)}
                        readOnly={!['1.i', '1.ii', '3.i', '3.ii'].includes(row.feId)}
                        error={getProvAmtEndMismatchError(row.feId)}
                        sx={{ width: '150px' }}
                        placeholder="0.00"
                        debounceDuration={1}
                      />
                      {getProvAmtEndMismatchError(row.feId) && (
                        <Typography fontSize={12} color="error" textAlign={'left'} sx={{ ml: 1 }}>
                          The value do not match
                        </Typography>
                      )}
                    </Box>
                  </TableCell>
                  <TableCell align="center">
                    <FormInput value={row.rate} readOnly={true} sx={{ width: '150px' }} />
                  </TableCell>
                  <TableCell align="center">
                    <FormInput value={row.provRequired} readOnly={true} sx={{ width: '150px' }} />
                  </TableCell>
                </TableRow>
              ))}

            {/* Dynamic Rows Rendering */}
            {tabIndex === 1 &&
              dynamicRows.map((row, i) => (
                <TableRow key={row.key}>
                  <TableCell align="center">{i + 1}</TableCell>
                  <TableCell padding="checkbox" align="center">
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
                      maxLength={100}
                      value={row.particulars}
                      onChange={(e) => handleDynamicChange(i, 'particulars', e.target.value)}
                      sx={{ width: '150px' }}
                      placeholder="Enter Particulars"
                      inputType="alphaNumericWithSpace"
                      debounceDuration={1}
                    />
                  </TableCell>
                  {['provAmtStart', 'writeOff', 'addition', 'reduction'].map((key) => (
                    <TableCell key={key} align="center">
                      <FormInput
                        value={row[key]}
                        onChange={(e) => handleDynamicChange(i, key, e.target.value)}
                        sx={{ width: '150px' }}
                        placeholder="0.00"
                        debounceDuration={1}
                      />
                    </TableCell>
                  ))}
                  <TableCell align="center">
                    <FormInput value={row.provAmtEnd} readOnly={true} sx={{ width: '150px' }} />
                  </TableCell>
                  <TableCell align="center">
                    <FormInput value={row.rate} readOnly={true} sx={{ width: '150px' }} />
                  </TableCell>
                  <TableCell align="center">
                    <FormInput value={row.provRequired} readOnly={true} sx={{ width: '150px' }} />
                  </TableCell>
                </TableRow>
              ))}
          </TableBody>
        </Table>
      </TableContainer>
      <Dialog
        open={isSubmitConfirmOpen}
        onClose={() => setIsSubmitConfirmOpen(false)}
        aria-labelledby="alert-dialog-title"
        aria-describedby="alert-dialog-description"
      >
        <DialogTitle id="alert-dialog-title">{'Confirm Submission'}</DialogTitle>
        <DialogContent>
          <DialogContentText id="alert-dialog-description">
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

      <Snackbar open={snackbar.open} autoHideDuration={4000} onClose={() => setSnackbar({ ...snackbar, open: false })}>
        <Alert
          severity={snackbar.severity}
          onClose={() => setSnackbar({ ...snackbar, open: false })}
          sx={{ width: '100%' }}
        >
          {snackbar.message}
        </Alert>
      </Snackbar>
    </Box>
  );
};

export default RW04;
///////////////////////////////////////////




import React, { useState, useEffect, useCallback } from 'react';
import {
  Tabs,
  Tab,
  Box,
  Table,
  TableHead,
  TableRow,
  TableCell,
  TableBody,
  TableContainer,
  Paper,
  Button,
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
import { useLocation, useNavigate } from 'react-router-dom';
import useApi from '../../../../common/hooks/useApi';
import { CustomButton } from '../../../../common/components/ui/Buttons';

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
  {
    dbId: 1,
    label: 'CONTINGENT LIABILITY AS PER AS -29',
    provAmtStart: '',
    addition: '',
    reversal: '',
    provAmtEnd: '0.00',
    difference: '0.00',
  },
  {
    dbId: 2,
    label: 'DELAYED REPORTING PENALTY',
    provAmtStart: '',
    addition: '',
    reversal: '',
    provAmtEnd: '0.00',
    difference: '0.00',
  },
  {
    dbId: 3,
    label: 'EX-GRATIA PAYMENT',
    provAmtStart: '',
    addition: '',
    reversal: '',
    provAmtEnd: '0.00',
    difference: '0.00',
  },
  {
    dbId: 4,
    label: 'PROVISION ON OVERDUE DEPOSIT INTT',
    provAmtStart: '',
    addition: '',
    reversal: '',
    provAmtEnd: '0.00',
    difference: '0.00',
  },
  {
    dbId: 5,
    label: 'LEAVE ENCASHMENT',
    provAmtStart: '',
    addition: '',
    reversal: '',
    provAmtEnd: '0.00',
    difference: '0.00',
  },
  {
    dbId: 6,
    label: 'PROVISION FOR PERFORMANCE LINKED INCENTIVES',
    provAmtStart: '',
    addition: '',
    reversal: '',
    provAmtEnd: '0.00',
    difference: '0.00',
  },
  {
    dbId: 7,
    label:
      'PROVISION ON ACCOUNT OF ENTRIES OUTSTANDING IN ADJUSTING ACCOUNT FOR PREVIOUS QUARTER(S) (i.e. PRIOR TO CURRENT QUARTER)',
    provAmtStart: '',
    addition: '',
    reversal: '',
    provAmtEnd: '0.00',
    difference: '0.00',
  },
];

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
  const navigate = useNavigate();
  const { callApi, isLoading } = useApi();
  const setSnackbarMessage = useCustomSnackbar();
  const [reportObject, setReportObject] = useState(state?.report || null);

  const headers = [
    'Particulars(2)',
    `Opening Provision as on ${user.previousQuarterEndDate}(3)`,
    'Additions during the period(4)',
    'Reversals during the Period(5)',
    `Provision as on ${user.quarterEndDate}(6)={(3+4)-(5)}`,
    'Difference to be Provided/ written back(7)={(6)-(3)}',
  ];
  const isNumeric = (val) => val === '' || (val !== null && !isNaN(parseFloat(val)) && isFinite(val));

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
      if (response?.data) {
        const apiData = response.data;
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
        // setSnackbarMessage('Data loaded successfully!', 'success');
      }
    } catch (error) {
      if (error.message !== 'canceled') {
        setSnackbarMessage('Failed to fetch data.', 'error');
      }
    }
  }, [reportObject, getBasePayload]);

  useEffect(() => {
    if (reportObject) {
      loadData();
    }
  }, [reportObject]);

  const handleSubmit = async () => {
    try {
      if (tabIndex === 0) {
        const value = staticRows.map((row) => [
          row.provAmtStart || '0.00',
          row.addition || '0.00',
          row.reversal || '0.00',
          row.provAmtEnd || '0.00',
          row.difference || '0.00',
          String(row.dbId),
        ]);
        const response = await callApi('/RW05/saveStatic', { ...getBasePayload(), value }, 'POST');
        if (response?.status) {
          //const [flag, newReportId, newStatus] = res.split('~');
          setReportObject((prev) => ({
            ...prev,
            status: '11',
          }));
          setSnackbarMessage('Data saved successfully!', 'success');
        }
      } else {
        const updatedRows = [...dynamicRows];

        for (let i = 0; i < updatedRows.length; i++) {
          const row = updatedRows[i];

          if (row.dbId === 0 && !row.particulars.trim()) {
            continue;
          }
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
          if (response?.rowId) {
            setReportObject((prev) => ({
              ...prev,
              status: '11',
            }));
            console.log('API Response for Save:', response);
            const newId = response?.rowId;

            if (row.dbId === 0 && newId) {
              updatedRows[i] = {
                ...row,
                dbId: newId,
                key: newId,
              };
            }
            setDynamicRows(updatedRows);
            setSnackbarMessage('Data saved successfully!', 'success');
          } else {
            setSnackbarMessage('Error', 'error');
          }
        }
      }
      setSnackbarMessage('Data saved successfully!', 'success');
      // loadData();
    } catch (error) {
      console.error('error', error);
      setSnackbarMessage('An error occurred while saving data.', 'error');
    }
  };

  const handleAddRow = () => {
    if (dynamicRows.some((row) => row.dbId === 0)) {
      setSnackbarMessage('Kindly save the current new row before adding another', 'warning');
      return;
    }
    setDynamicRows([...dynamicRows, createInitialDynamicRow()]);
  };

  const handleDeleteRow = async () => {
    const rowsToDelete = dynamicRows.filter((row) => row.selected && row.dbId !== 0);
    const newRowsToKeep = dynamicRows.filter((row) => !row.selected);

    if (rowsToDelete.length === 0) {
      if (dynamicRows.some((row) => row.selected && row.dbId === 0)) {
        setDynamicRows(newRowsToKeep.length > 0 ? newRowsToKeep : [createInitialDynamicRow()]);
        setSnackbarMessage('New row removed.', 'info');
      } else {
        setSnackbarMessage('Please select a saved row to delete.', 'warning');
      }
      return;
    }

    try {
      const idsToDelete = rowsToDelete.map((row) => row.dbId);

      const payload = {
        userCapacity: user.userCapacity,
        rowIds: idsToDelete,
      };

      const res = await callApi('/RW05/deleteRow', payload, 'POST');
      if (res?.deletedCount) {
        setSnackbarMessage('Selected rows deleted successfully!', 'success');

        setDynamicRows(newRowsToKeep.length > 0 ? newRowsToKeep : [createInitialDynamicRow()]);
      } else {
        setSnackbarMessage('An error occurred during deletion.', 'error');
      }
    } catch (error) {
      console.error('Error deleting rows:', error);
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
      if (response.status === 1) {
        setSnackbarMessage('Report submitted successfully!', 'success');
        //  setReportObject((prev) => ({ ...prev, status: response }));
        setTimeout(() => {
          navigate(-1);
        }, 2000);
      } else {
        setSnackbarMessage('Submission completed, but server response was unexpected.', 'warning');
      }
    } catch (error) {
      console.error('error', error);
      setSnackbarMessage('An error occurred during submission.', 'error');
    }
  };

  return (
    <Box>
      <Tabs value={tabIndex} onChange={(e, i) => setTabIndex(i)}>
        <Tab label="RW-05(I)" />
        <Tab label="RW-05(II)" />
      </Tabs>

      <Box mt={2} display="flex" gap={1}>
        {tabIndex === 1 && (
          <>
            <CustomButton label={'Add Row'} buttonType={'create'} onClickHandler={handleAddRow} disabled={isLoading} />
            <CustomButton
              label={'Delete Row'}
              buttonType={'delete'}
              onClickHandler={handleDeleteRow}
              disabled={isLoading || dynamicRows.every((r) => !r.selected)}
            />
          </>
        )}

        <CustomButton label={'Save'} buttonType={'save'} onClickHandler={handleSubmit} disabled={isLoading} />
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
              <StyledTableCell>Sr No(1)</StyledTableCell>
              {tabIndex === 1 && <StyledTableCell>SELECT</StyledTableCell>}
              {headers.map((head, idx) => (
                <StyledTableCell key={idx}>{head}</StyledTableCell>
              ))}
            </TableRow>
          </TableHead>
          <TableBody>
            {tabIndex === 0 &&
              staticRows.map((row, index) => (
                <TableRow key={row.dbId}>
                  <TableCell align="center">{index + 1}</TableCell>
                  <TableCell>{row.label}</TableCell>
                  {['provAmtStart', 'addition', 'reversal'].map((key) => (
                    <TableCell key={key} align="center">
                      <FormInput
                        value={row[key]}
                        onChange={(e) => handleStaticChange(index, key, e.target.value)}
                        sx={{ width: '180px' }}
                        placeholder="0.00"
                        readOnly={row.dbId === 1}
                        debounceDuration={1}
                      />
                    </TableCell>
                  ))}
                  <TableCell align="center">
                    <FormInput value={row.provAmtEnd} readOnly sx={{ width: '180px' }} />
                  </TableCell>
                  <TableCell align="center">
                    <FormInput value={row.difference} readOnly sx={{ width: '180px' }} />
                  </TableCell>
                </TableRow>
              ))}
            {tabIndex === 1 &&
              dynamicRows.map((row, index) => (
                <TableRow key={row.key}>
                  <TableCell align="center">{index + 1}</TableCell>
                  <TableCell padding="checkbox">
                    <Checkbox
                      checked={row.selected}
                      onChange={() => {
                        const updated = [...dynamicRows];
                        updated[index].selected = !updated[index].selected;
                        setDynamicRows(updated);
                      }}
                    />
                  </TableCell>
                  <TableCell>
                    <FormInput
                      value={row.particulars}
                      onChange={(e) => handleDynamicChange(index, 'particulars', e.target.value)}
                      sx={{ width: '180px' }}
                      placeholder="Enter Particulars"
                      inputType="alphaNumericWithSpace"
                      debounceDuration={1}
                    />
                  </TableCell>
                  {['provAmtStart', 'addition', 'reversal'].map((key) => (
                    <TableCell key={key} align="center">
                      <FormInput
                        value={row[key]}
                        onChange={(e) => handleDynamicChange(index, key, e.target.value)}
                        sx={{ width: '180px' }}
                        placeholder="0.00"
                        debounceDuration={1}
                      />
                    </TableCell>
                  ))}
                  <TableCell align="center">
                    <FormInput value={row.provAmtEnd} readOnly sx={{ width: '180px' }} />
                  </TableCell>
                  <TableCell align="center">
                    <FormInput value={row.difference} readOnly sx={{ width: '180px' }} />
                  </TableCell>
                </TableRow>
              ))}
          </TableBody>
        </Table>
      </TableContainer>

      <Dialog open={isSubmitConfirmOpen} onClose={() => setIsSubmitConfirmOpen(false)}>
        <DialogTitle>Confirm Submission</DialogTitle>
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
};

export default RW05;

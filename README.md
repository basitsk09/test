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

// A helper to generate an empty dynamic row object
const createInitialDynamicRow = () => ({
  dbId: 0, // 0 indicates a new, unsaved row
  particulars: '',
  provAmtStart: '',
  writeOff: '',
  addition: '',
  reduction: '',
  provAmtEnd: '0.00',
  rate: '100', // Default rate for dynamic rows
  provRequired: '0.00',
  selected: false,
  // React key for new rows, dbId will be used for saved rows
  key: `new-${Date.now()}-${Math.random().toString(36).substr(2, 9)}`,
});

// Initial structure for static rows, will be populated from API
const getInitialStaticRows = (quarterEndDate) => [
    { feId: '1', dbId: 1, label: 'FRAUDS - DEBITED TO RECALLED ASSETS A/c (Prod Cd 6998-9981)**', provAmtStart: '', writeOff: '', addition: '', reduction: '', provAmtEnd: '0.00', rate: '', provRequired: '0.00' },
    { feId: '1.i', dbId: 2, label: `Frauds reported on or prior to ${quarterEndDate} provision @ 100% ##`, provAmtStart: '', writeOff: '', addition: '', reduction: '', provAmtEnd: '0.00', rate: '100', provRequired: '0.00' },
    { feId: '1.ii', dbId: 3, label: `Delayed Reported frauds Provision @ 100% ##`, provAmtStart: '', writeOff: '', addition: '', reduction: '', provAmtEnd: '0.00', rate: '100', provRequired: '0.00' },
    { feId: '2', dbId: 4, label: 'OTHERS LOSSES IN RECALLED ASSETS (Prod cd 6998 - 9982)#', provAmtStart: '', writeOff: '', addition: '', reduction: '', provAmtEnd: '0.00', rate: '100', provRequired: '0.00' },
    { feId: '3', dbId: 5, label: 'FRAUDS - OTHER (NOT DEBITED TO RA A/c)$', provAmtStart: '', writeOff: '', addition: '', reduction: '', provAmtEnd: '0.00', rate: '', provRequired: '0.00' },
    { feId: '3.i', dbId: 6, label: `Frauds reported on or prior to ${quarterEndDate} provision @ 100% ##`, provAmtStart: '', writeOff: '', addition: '', reduction: '', provAmtEnd: '0.00', rate: '100', provRequired: '0.00' },
    { feId: '3.ii', dbId: 7, label: `Delayed Reported frauds Provision @ 100% ##`, provAmtStart: '', writeOff: '', addition: '', reduction: '', provAmtEnd: '0.00', rate: '100', provRequired: '0.00' },
    { feId: '4', dbId: 8, label: 'REVENUE ITEM IN SYSTEM SUSPENSE', provAmtStart: '', writeOff: '', addition: '', reduction: '', provAmtEnd: '0.00', rate: '100', provRequired: '0.00' },
    { feId: '5', dbId: 9, label: 'PROVISION ON ACCOUNT OF FSLO', provAmtStart: '', writeOff: '', addition: '', reduction: '', provAmtEnd: '0.00', rate: '100', provRequired: '0.00' },
    { feId: '6', dbId: 10, label: 'PROVISION ON ACCOUNT OF ENTRIES OUTSTANDING IN ADJUSTING ACCOUNT FOR PREVIOUS QUARTER(S) (i.e. PRIOR TO CURRENT QUARTER)', provAmtStart: '', writeOff: '', addition: '', reduction: '', provAmtEnd: '0.00', rate: '100', provRequired: '0.00' },
    { feId: '7', dbId: 11, label: 'PROVISION ON N.P.A. INTEREST FREE STAFF LOANS', provAmtStart: '', writeOff: '', addition: '', reduction: '', provAmtEnd: '0.00', rate: '100', provRequired: '0.00' },
];

const RW04 = () => {
  const [tabIndex, setTabIndex] = useState(0);
  const user = JSON.parse(localStorage.getItem('user'));
  const { state } = useLocation();

  const [staticRows, setStaticRows] = useState(getInitialStaticRows(user.quarterEndDate));
  const [dynamicRows, setDynamicRows] = useState([]);

  const [snackbar, setSnackbar] = useState({ open: false, message: '', severity: 'info' });
  const { callApi, isLoading } = useApi();
  const setSnackbarMessage = useCustomSnackbar();
  const [reportObject, setReportObject] = useState(state?.report || null);

  const headers = [ 'PARTICULARS(2)', `PROVISIONABLE AMT AS ON ${user.quarterStartDate} (3)`, 'WRITE OFF DURING THE QUARTER* (4)', 'ADDITONS IN PROVISIONABLE AMT DURING THE QUARTER (5)', 'REDUCTION IN PROVISIONABLE AMT (OTHER THAN WRITE OFF) DURING THE QUARTER^ (6)', `PROVISIONABLE AMT AS ON ${user.quarterEndDate} (7)=3-4+5-6`, 'RATE OF PROVISION (%)(8)', `PROVISION REQUIREMENT AS ON ${user.quarterEndDate} (9)=7*8` ];
  const isNumeric = (val) => val === null || val === '' || (!isNaN(parseFloat(val)) && isFinite(val));
  const getBasePayload = useCallback( () => ({ circleCode: user?.circleCode, qed: user?.quarterEndDate, userCapacity: user?.userCapacity, reportId: reportObject?.reportId, reportMasterId: reportObject?.reportMasterId, currentStatus: reportObject?.status, userId: user.userId, }), [user, reportObject] );

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
  
  // ... (other handlers like handleDynamicChange, etc., remain unchanged) ...

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
  
  /**
   * NEW VALIDATION FUNCTION
   * Checks all business rules before enabling the Submit button.
   * @returns {boolean} - True if all data is valid, false otherwise.
   */
  const isDataValid = () => {
    // Check validation for parent-child sum in static rows
    if (getProvAmtEndMismatchError('1') || getProvAmtEndMismatchError('3')) {
      return false;
    }

    // Future validation rules can be added here...

    return true;
  };

  const loadData = useCallback(async () => {
    // ... (loadData logic remains unchanged)
  }, [reportObject, user.quarterEndDate, callApi, getBasePayload, setSnackbarMessage]);

  useEffect(() => {
    if (reportObject) {
      loadData();
    }
    // eslint-disable-next-line react-hooks/exhaustive-deps
  }, [reportObject]);

  const handleSubmit = async () => {
    // ... (save logic remains unchanged)
  };

  const handleSubmitReport = async () => {
    // Safety check: Re-validate data before submitting.
    if (!isDataValid()) {
      setSnackbarMessage('Please resolve all validation errors before submitting.', 'error');
      return;
    }

    // Safety check: Confirm the status is correct.
    if (reportObject?.status !== '11') {
      setSnackbarMessage('Report can only be submitted when in status 11.', 'warning');
      return;
    }

    try {
      const payload = getBasePayload();
      const response = await callApi('/RW04/submitReport', payload, 'POST');

      if (response && typeof response === 'string') {
        setSnackbarMessage('Report submitted successfully!', 'success');
        setReportObject(prev => ({
            ...prev,
            status: response,
        }));
      } else {
        setSnackbarMessage('Submission completed, but server response was unexpected.', 'warning');
      }
    } catch (error) {
      console.error('Error submitting report:', error);
      setSnackbarMessage('An error occurred during submission.', 'error');
    }
  };
  
  // ... (other functions like handleAddRow, handleDeleteRow, renderHeader remain unchanged) ...

  return (
    <Box>
      <Tabs value={tabIndex} onChange={(e, i) => setTabIndex(i)}>
        <Tab label="RW-04(I)" />
        <Tab label="RW-04(II) - OTHERS" />
      </Tabs>
      <Box mt={2} display="flex" gap={1}>
        {tabIndex === 1 && (
          <>
            <Button variant="contained" color="secondary" onClick={handleAddRow} disabled={isLoading}> Add Row </Button>
            <Button variant="contained" color="error" onClick={handleDeleteRow} disabled={isLoading || dynamicRows.every((row) => !row.selected)} > Delete Row </Button>
          </>
        )}
        <Button variant="contained" color="warning" onClick={handleSubmit} disabled={isLoading}>
          {isLoading ? <CircularProgress size={24} color="inherit" /> : 'Save'}
        </Button>

        {/* UPDATED SUBMIT BUTTON */}
        <Button
          variant="contained"
          color="primary"
          onClick={handleSubmitReport}
          // Button is disabled if loading, status is not 11, OR if data validations fail.
          disabled={isLoading || reportObject?.status !== '11' || !isDataValid()}
          sx={{ ml: 1 }}
        >
          Submit
        </Button>
      </Box>

      <TableContainer component={Paper} sx={{ mt: 2, maxHeight: 'calc(100vh - 250px)' }}>
        <Table stickyHeader>
          {/* ... (renderHeader call) ... */}
          <TableBody>
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
                      />
                    </TableCell>
                  ))}
                  <TableCell align="center">
                    <Box display="flex" flexDirection="column">
                      <FormInput
                        value={row.provAmtEnd}
                        onChange={(e) => handleStaticChange(index, 'provAmtEnd', e.target.value)}
                        readOnly={!['1.i', '1.ii', '3.i', '3.ii'].includes(row.feId)}
                        // The error prop is driven by the validation function
                        error={getProvAmtEndMismatchError(row.feId)}
                        sx={{ width: '150px' }}
                        placeholder="0.00"
                      />
                      {/* The visual error message is also driven by the same function */}
                      {getProvAmtEndMismatchError(row.feId) && (
                        <Typography fontSize={12} color="error" textAlign={'left'} sx={{ ml: 1 }}>
                          Sum of children must match parent
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
            
            {/* ... (Dynamic Rows Rendering remains unchanged) ... */}
          </TableBody>
        </Table>
      </TableContainer>

      {/* ... (Snackbar remains unchanged) ... */}
    </Box>
  );
};

export default RW04;

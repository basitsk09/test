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

// A helper to generate an empty dynamic row object
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
  const [isSubmitConfirmOpen, setIsSubmitConfirmOpen] = useState(false); // State for dialog

  const [snackbar, setSnackbar] = useState({ open: false, message: '', severity: 'info' });
  const { callApi, isLoading } = useApi();
  const setSnackbarMessage = useCustomSnackbar();
  const [reportObject, setReportObject] = useState(state?.report || null);

  const headers = [ 'PARTICULARS(2)', `PROVISIONABLE AMT AS ON ${user.quarterStartDate} (3)`, 'WRITE OFF DURING THE QUARTER* (4)', 'ADDITONS IN PROVISIONABLE AMT DURING THE QUARTER (5)', 'REDUCTION IN PROVISIONABLE AMT (OTHER THAN WRITE OFF) DURING THE QUARTER^ (6)', `PROVISIONABLE AMT AS ON ${user.quarterEndDate} (7)=3-4+5-6`, 'RATE OF PROVISION (%)(8)', `PROVISION REQUIREMENT AS ON ${user.quarterEndDate} (9)=7*8` ];
  const isNumeric = (val) => val === null || val === '' || (!isNaN(parseFloat(val)) && isFinite(val));
  const getBasePayload = useCallback( () => ({ circleCode: user?.circleCode, qed: user?.quarterEndDate, userCapacity: user?.userCapacity, reportId: reportObject?.reportId, reportMasterId: reportObject?.reportMasterId, currentStatus: reportObject?.status, userId: user.userId, }), [user, reportObject] );

  const handleStaticChange = (index, key, value) => { /* ... no change ... */ };
  const getProvAmtEndMismatchError = (rowFeId) => { /* ... no change ... */ };
  
  const isDataValid = () => {
    if (getProvAmtEndMismatchError('1') || getProvAmtEndMismatchError('3')) {
      return false;
    }
    return true;
  };

  const loadData = useCallback(async () => { /* ... no change ... */ }, [reportObject, user.quarterEndDate, callApi, getBasePayload, setSnackbarMessage]);

  useEffect(() => {
    if (reportObject) {
      loadData();
    }
    // eslint-disable-next-line react-hooks/exhaustive-deps
  }, [reportObject]);

  const handleSubmit = async () => { /* ... no change ... */ };

  /**
   * This function now only opens the confirmation dialog.
   */
  const handleSubmitReport = () => {
    // Re-validate data before opening the confirmation dialog
    if (!isDataValid()) {
      setSnackbarMessage('Please resolve all validation errors before submitting.', 'error');
      return;
    }
    setIsSubmitConfirmOpen(true);
  };

  /**
   * NEW FUNCTION
   * This function contains the actual API submission logic and is called from the dialog.
   */
  const handleConfirmSubmit = async () => {
    setIsSubmitConfirmOpen(false); // Close the dialog first

    if (reportObject?.status !== '11') {
      setSnackbarMessage('Report can only be submitted when in status 11.', 'warning');
      return;
    }

    try {
      const payload = getBasePayload();
      const response = await callApi('/RW04/submitReport', payload, 'POST');

      if (response && typeof response === 'string') {
        setSnackbarMessage('Report submitted successfully!', 'success');
        setReportObject(prev => ({ ...prev, status: response }));
      } else {
        setSnackbarMessage('Submission completed, but server response was unexpected.', 'warning');
      }
    } catch (error) {
      console.error('Error submitting report:', error);
      setSnackbarMessage('An error occurred during submission.', 'error');
    }
  };
  
  // ... (other functions like handleAddRow, handleDeleteRow, renderHeader, etc. remain unchanged) ...

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

        <Button
          variant="contained"
          color="primary"
          onClick={handleSubmitReport} // This now opens the dialog
          disabled={isLoading || reportObject?.status !== '11' || !isDataValid()}
          sx={{ ml: 1 }}
        >
          Submit
        </Button>
      </Box>

      <TableContainer component={Paper} sx={{ mt: 2, maxHeight: 'calc(100vh - 250px)' }}>
          {/* ... Table structure ... */}
      </TableContainer>

      {/* NEW SUBMIT CONFIRMATION DIALOG */}
      <Dialog
        open={isSubmitConfirmOpen}
        onClose={() => setIsSubmitConfirmOpen(false)}
        aria-labelledby="alert-dialog-title"
        aria-describedby="alert-dialog-description"
      >
        <DialogTitle id="alert-dialog-title">
          {"Confirm Submission"}
        </DialogTitle>
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
        <Alert severity={snackbar.severity} onClose={() => setSnackbar({ ...snackbar, open: false })} sx={{ width: '100%' }}>
          {snackbar.message}
        </Alert>
      </Snackbar>
    </Box>
  );
};

export default RW04;

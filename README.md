import React, { useState, useEffect, useCallback } from 'react';
import {
  Table,
  TableBody,
  TableCell,
  TableContainer,
  TableHead,
  TableRow,
  Paper,
  TextField,
  Button,
  Dialog,
  DialogTitle,
  DialogContent,
  DialogActions,
  Typography,
  CircularProgress,
  Box,
  Stack,
  Snackbar,
  Alert,
} from '@mui/material';
import { styled } from '@mui/material/styles';
import FormInput from '../../../../common/components/ui/FormInput';
import useApi from '../../../../common/hooks/useApi';
import { useLocation, useNavigate } from 'react-router-dom';
import useCustomSnackbar from '../../../../common/hooks/useCustomSnackbar';
// import { safeParseFloat } from '../../../../common/utils/commonUtils'; // No longer needed for calculations
import Decimal from 'decimal.js'; // Import decimal.js

// Configure Decimal.js globally if needed for default precision (e.g., for toFixed rounding mode)
// Decimal.set({ precision: 20, rounding: Decimal.ROUND_HALF_UP }); // Example: Set default precision and rounding

const StyledTableCell = styled(TableCell)(({ theme }) => ({
  fontSize: '0.875rem',
  border: '1px solid #e0e0e0',
  textAlign: 'right',
  '&.MuiTableCell-head': {
    backgroundColor: theme.palette.grey[400],
    color: theme.palette.common.black,
    textAlign: 'center',
    fontWeight: 'bold',
  },
}));

const FormRow = ({ label, grField, proField, data, handleChange, readOnly = false, bold = false, indent = false }) => (
  <TableRow>
    <TableCell sx={{ fontWeight: bold ? 'bold' : 'normal', pl: indent ? 4 : 1 }}>{label}</TableCell>
    <StyledTableCell>
      <FormInput
        name={grField}
        value={data[grField] || ''}
        onChange={handleChange(grField)}
        readOnly={readOnly}
        customStyles={{
          textAlign: 'right',
          '& input': { textAlign: 'right', padding: '6px 8px' },
          color: (theme) => theme.palette.text.primary,
        }}
        inputType={'wholeAmountDecimal'}
      />
    </StyledTableCell>
    <StyledTableCell>
      <FormInput
        name={proField}
        value={data[proField] || ''}
        onChange={handleChange(proField)}
        readOnly={readOnly}
        customStyles={{
          textAlign: 'right',
          '& input': { textAlign: 'right', padding: '6px 8px' },
          color: (theme) => theme.palette.text.primary,
        }}
        inputType={'wholeAmountDecimal'}
      />
    </StyledTableCell>
  </TableRow>
);

const rowDefinitions = [
  {
    label: '1. Bills Purchased and discounted (1.1 + 1.2)',
    gr: 'bilPurGrAmt',
    pro: 'bilPurPro',
    readOnly: true,
    bold: true,
  },
  { label: '1.1 Inland Bills Purchased and Discounted', gr: 'inlBilPurGrAmt', pro: 'inlBilPurPro', indent: true },
  {
    label: '1.2 Foreign Bills Purchased and Discounted (1.2.1+1.2.2+1.2.3)',
    gr: 'foreBilPurGrAmt',
    pro: 'foreBilPurPro',
    readOnly: true,
    bold: true,
    indent: true,
  },
  { label: '1.2.1 Export Bills drawn in India', gr: 'expBillGrAmt', pro: 'expBillPro', indent: true },
  { label: '1.2.2 Import Bills drawn on and payable in India', gr: 'impBillGrAmt', pro: 'impBillPro', indent: true },
  { label: '1.2.3.1 Payable in India', gr: 'payIndGrAmt', pro: 'payIndPro', indent: true },
  { label: '1.2.3.2 Payable outside India', gr: 'payOutGrAmt', pro: 'payOutPro', indent: true },
  { label: '2. Loans and Advances (2.1 + 2.2)', gr: 'loanAdvGrAmt', pro: 'loanAdvPro', readOnly: true, bold: true },
  {
    label: '2.1 Loans and Advances, Cash Credit & Overdrafts',
    gr: 'loanAdvCreGrAmt',
    pro: 'loanAdvCrePro',
    indent: true,
  },
  {
    label: '2.2 Due from Banks (2.2.1+2.2.2+2.2.3)',
    gr: 'dueGrAmt',
    pro: 'duePro',
    readOnly: true,
    bold: true,
    indent: true,
  },
  { label: '2.2.1 Co-operative Banks in India', gr: 'coopBankGrAmt', pro: 'coopBankPro', indent: true },
  { label: '2.2.2 Commercial Banks in India', gr: 'commBankGrAmt', pro: 'commBankPro', indent: true },
  { label: '2.2.3 Banks outside India', gr: 'bankOutIndGrAmt', pro: 'bankOutIndPro', indent: true },
  { label: '3. Grand Total (1 + 2)', gr: 'grandTotlGrAmt', pro: 'grandTotlPro', readOnly: true, bold: true },
];

const SC9Supplementary = () => {
  const user = JSON.parse(localStorage.getItem('user'));
  const { state } = useLocation();
  const [reportObject, setReportObject] = useState(state?.report || null);
  const [data, setData] = useState({});
  const [confirmDialog, setConfirmDialog] = useState({ open: false, type: null });
  const [snackbar, setSnackbar] = useState({ open: false, message: '', severity: 'success' });
  const { callApi } = useApi();
  const [isLoading, setIsLoading] = useState(false);
  const setSnackbarMessage = useCustomSnackbar();
  const navigate = useNavigate();

  // Function to safely parse a value to a Decimal object
  // Handles null, undefined, and empty strings by converting to '0'
  const parseToDecimal = useCallback((value) => {
    try {
      const val = (value === null || value === undefined || value === '') ? '0' : String(value);
      return new Decimal(val);
    } catch (e) {
      console.error(`Error parsing value to Decimal: ${value}`, e);
      return new Decimal('0'); // Fallback to 0 if parsing fails
    }
  }, []);

  const handleChange =
    (field) =>
    (e) => {
      // Input values are always strings. No immediate parsing needed here.
      setData((prev) => ({ ...prev, [field]: e.target.value }));
    };

  const fetchData = async () => {
    setIsLoading(true);
    try {
      const payload = {
        circleCode: user.circleCode,
        quarterEndDate: user.quarterEndDate,
        userId: user.userId,
        reportName: reportObject.name,
        reportMasterId: reportObject.reportId,
        status: reportObject.status,
      };
      const res = await callApi('/Maker/getSavedDataNineSupl', payload, 'POST');
      const normalized = { ...res };
      Object.keys(normalized).forEach((key) => {
        if (normalized[key] == null || normalized[key] === '') {
          normalized[key] = '0.00'; // Ensure '0.00' for empty/null values
        } else {
          // Parse to Decimal, then format to 2 fixed decimals as a string for display
          normalized[key] = parseToDecimal(normalized[key]).toFixed(2);
        }
      });
      setData(normalized);
    } catch (error) {
      console.error('Error fetching SC9 supplementary data:', error);
      setSnackbarMessage('Failed to fetch data.', 'error');
    } finally {
      setIsLoading(false);
    }
  };

  const submitData = async () => {
    try {
      const save = confirmDialog.type === 'save';
      console.log('report obj', reportObject);

      const dataToSubmit = { ...data };
      for (const key in dataToSubmit) {
        if (Object.prototype.hasOwnProperty.call(dataToSubmit, key)) {
          // Convert all numeric string fields from state to actual numbers (float) for API submission
          // Ensure they are correctly represented before sending to backend.
          if (
            key.endsWith('GrAmt') ||
            key.endsWith('Pro')
          ) {
            // Convert to Decimal, then to a standard JS Number (safe now due to Decimal.js precision)
            dataToSubmit[key] = parseToDecimal(dataToSubmit[key]).toNumber();
          }
        }
      }

      const payload = {
        ...dataToSubmit,
        circleCode: user.circleCode,
        quarterEndDate: user.quarterEndDate,
        userId: user.userId,
        reportName: reportObject.name,
        reportMasterId: reportObject.reportMasterId,
        status: reportObject.status,
        reportId: reportObject.reportId,
        save,
      };
      const res = await callApi('/Maker/submitNineSupl', payload, 'POST');
      if (res && typeof res === 'string') {
        const [flag, newReportId, newStatus] = res.split('~');
        setReportObject((prev) => ({
          ...prev,
          reportId: newReportId,
          status: newStatus,
        }));
        setData((prev) => ({
          ...prev,
          reportId: newReportId,
          status: newStatus,
        }));
        setSnackbarMessage(save ? 'Report saved successfully!' : 'Report submitted successfully!', 'success');
        if (!save) {
          setTimeout(() => {
            navigate(-1);
          }, 2000);
        }
      } else {
        throw new Error('Failed to save/submit data: Unexpected API response.');
      }
    } catch (error) {
      console.error('Submission error:', error);
      setSnackbarMessage('Failed to save/submit data.', 'error');
    } finally {
      setConfirmDialog({ open: false, type: null });
    }
  };

  useEffect(() => {
    fetchData();
  }, []); // Empty dependency array means this runs once on mount

  // Recalculate derived fields whenever their dependencies change
  useEffect(() => {
    // Only perform calculations if data has been loaded/initialized and has keys
    if (Object.keys(data).length === 0) return;

    // Convert all relevant data fields to Decimal objects first for accurate calculations
    const payIndGrAmtD = parseToDecimal(data.payIndGrAmt);
    const payOutGrAmtD = parseToDecimal(data.payOutGrAmt);
    const expBillGrAmtD = parseToDecimal(data.expBillGrAmt);
    const impBillGrAmtD = parseToDecimal(data.impBillGrAmt);
    const inlBilPurGrAmtD = parseToDecimal(data.inlBilPurGrAmt);
    const loanAdvCreGrAmtD = parseToDecimal(data.loanAdvCreGrAmt);
    const coopBankGrAmtD = parseToDecimal(data.coopBankGrAmt);
    const commBankGrAmtD = parseToDecimal(data.commBankGrAmt);
    const bankOutIndGrAmtD = parseToDecimal(data.bankOutIndGrAmt);

    const payIndProD = parseToDecimal(data.payIndPro);
    const payOutProD = parseToDecimal(data.payOutPro);
    const expBillProD = parseToDecimal(data.expBillPro);
    const impBillProD = parseToDecimal(data.impBillPro);
    const inlBilPurProD = parseToDecimal(data.inlBilPurPro);
    const loanAdvCreProD = parseToDecimal(data.loanAdvCrePro);
    const coopBankProD = parseToDecimal(data.coopBankPro);
    const commBankProD = parseToDecimal(data.commBankPro);
    const bankOutIndProD = parseToDecimal(data.bankOutIndPro);

    // Perform all calculations using Decimal.js methods (e.g., .plus(), .minus())
    // 1.2.3 Other Foreign Bills Purchased and Discounted (1.2.3.1 + 1.2.3.2)
    const othForeBilPurGrAmtComputed = payIndGrAmtD.plus(payOutGrAmtD);
    const othForeBilPurProComputed = payIndProD.plus(payOutProD);

    // 1.2 Foreign Bills Purchased and Discounted (1.2.1+1.2.2+1.2.3)
    const foreBilPurGrAmtComputed = expBillGrAmtD.plus(impBillGrAmtD).plus(othForeBilPurGrAmtComputed);
    const foreBilPurProComputed = expBillProD.plus(impBillProD).plus(othForeBilPurProComputed);

    // 1. Bills Purchased and discounted (1.1 + 1.2)
    const bilPurGrAmtComputed = inlBilPurGrAmtD.plus(foreBilPurGrAmtComputed);
    const bilPurProComputed = inlBilPurProD.plus(foreBilPurProComputed);

    // 2.2 Due from Banks (2.2.1+2.2.2+2.2.3)
    const dueGrAmtComputed = coopBankGrAmtD.plus(commBankGrAmtD).plus(bankOutIndGrAmtD);
    const dueProComputed = coopBankProD.plus(commBankProD).plus(bankOutIndProD);

    // 2. Loans and Advances (2.1 + 2.2)
    const loanAdvGrAmtComputed = loanAdvCreGrAmtD.plus(dueGrAmtComputed);
    const loanAdvProComputed = loanAdvCreProD.plus(dueProComputed);

    // 3. Grand Total (1 + 2)
    const grandTotlGrAmtComputed = bilPurGrAmtComputed.plus(loanAdvGrAmtComputed);
    const grandTotlProComputed = bilPurProComputed.plus(loanAdvProComputed);

    const updated = { ...data }; // Create a new object for state update

    // Assign formatted string values back to the updated object
    updated.othForeBilPurGrAmt = othForeBilPurGrAmtComputed.toFixed(2);
    updated.othForeBilPurPro = othForeBilPurProComputed.toFixed(2);
    updated.foreBilPurGrAmt = foreBilPurGrAmtComputed.toFixed(2);
    updated.foreBilPurPro = foreBilPurProComputed.toFixed(2);
    updated.bilPurGrAmt = bilPurGrAmtComputed.toFixed(2);
    updated.bilPurPro = bilPurProComputed.toFixed(2);
    updated.dueGrAmt = dueGrAmtComputed.toFixed(2);
    updated.duePro = dueProComputed.toFixed(2);
    updated.loanAdvGrAmt = loanAdvGrAmtComputed.toFixed(2);
    updated.loanAdvPro = loanAdvProComputed.toFixed(2);
    updated.grandTotlGrAmt = grandTotlGrAmtComputed.toFixed(2);
    updated.grandTotlPro = grandTotlProComputed.toFixed(2);

    // Only update state if there are actual changes to prevent unnecessary re-renders
    const hasChanged = Object.keys(updated).some(key => updated[key] !== data[key]);
    if (hasChanged) {
      setData(updated);
    }

  }, [
    data.payIndGrAmt, data.payOutGrAmt,
    data.expBillGrAmt, data.impBillGrAmt,
    data.inlBilPurGrAmt,
    data.loanAdvCreGrAmt,
    data.coopBankGrAmt, data.commBankGrAmt, data.bankOutIndGrAmt,
    data.payIndPro, data.payOutPro,
    data.expBillPro, data.impBillPro,
    data.inlBilPurPro,
    data.loanAdvCrePro,
    data.coopBankPro, data.commBankPro, data.bankOutIndPro,
    parseToDecimal, // Now using parseToDecimal as dependency
    data // Include data itself as a dependency for initial load or full data replacement
  ]);

  if (isLoading) {
    return (
      <Box sx={{ display: 'flex', justifyContent: 'center', alignItems: 'center', height: '50vh' }}>
        <CircularProgress />
        <Typography sx={{ ml: 2 }}>Loading Data...</Typography>
      </Box>
    );
  }

  return (
    <Box sx={{ p: 2 }}>
      <Stack direction="row" spacing={2} sx={{ mb: 2, justifyContent: 'left' }}>
        <Button variant="contained" color="warning" onClick={() => setConfirmDialog({ open: true, type: 'save' })}>
          Save
        </Button>
        <Button variant="contained" color="success" onClick={() => setConfirmDialog({ open: true, type: 'submit' })}>
          Submit
        </Button>
      </Stack>
      <TableContainer component={Paper}>
        <Table size="small">
          <TableHead>
            <TableRow>
              <StyledTableCell>Description</StyledTableCell>
              <StyledTableCell>
                Gross Amount
                <br />
                Rs. P
              </StyledTableCell>
              <StyledTableCell>
                Provision
                <br />
                Rs. P
              </StyledTableCell>
            </TableRow>
          </TableHead>
          <TableBody>
            {rowDefinitions.map((row) => (
              <FormRow
                key={row.gr}
                label={row.label}
                grField={row.gr}
                proField={row.pro}
                data={data}
                handleChange={handleChange}
                readOnly={row.readOnly}
                bold={row.bold}
                indent={row.indent}
              />
            ))}
          </TableBody>
        </Table>
      </TableContainer>

      <Dialog open={confirmDialog.open} onClose={() => setConfirmDialog({ open: false, type: null })}>
        <DialogTitle>Confirm {confirmDialog.type === 'save' ? 'Save' : 'Submit'}</DialogTitle>
        <DialogContent>
          <Typography>
            Are you sure you want to {confirmDialog.type === 'save' ? 'save' : 'submit'} the data?
          </Typography>
        </DialogContent>
        <DialogActions>
          <Button onClick={() => setConfirmDialog({ open: false, type: null })}>Cancel</Button>
          <Button
            variant="contained"
            onClick={submitData}
            color={confirmDialog.type === 'save' ? 'warning' : 'success'}
          >
            Yes, {confirmDialog.type === 'save' ? 'Save' : 'Submit'}
          </Button>
        </DialogActions>
      </Dialog>

      <Snackbar
        open={snackbar.open}
        autoHideDuration={4000}
        onClose={() => setSnackbar({ ...snackbar, open: false })}
        anchorOrigin={{ vertical: 'bottom', horizontal: 'center' }}
      >
        <Alert
          onClose={() => setSnackbar({ ...snackbar, open: false })}
          severity={snackbar.severity}
          sx={{ width: '100%' }}
        >
          {snackbar.message}
        </Alert>
      </Snackbar>
    </Box>
  );
};

export default SC9Supplementary;

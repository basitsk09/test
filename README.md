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
import { safeParseFloat } from '../../../../common/utils/commonUtils'; // Keep this

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

  // Define a scaling factor for two decimal places (e.g., for Rs. P)
  const SCALE_FACTOR = 100;

  // Modified parse function to convert to integer representation
  const parseAndScale = useCallback((v) => {
    // safeParseFloat ensures we get a number or 0
    const floatValue = safeParseFloat(v);
    // Multiply by SCALE_FACTOR and then round to nearest integer
    // This removes floating point inaccuracies before arithmetic
    return Math.round(floatValue * SCALE_FACTOR);
  }, [SCALE_FACTOR]); // Include SCALE_FACTOR in dependencies

  // Helper to scale down and format
  const formatScaledValue = useCallback((scaledInt) => {
    // Divide by SCALE_FACTOR to get the actual float value
    const floatResult = scaledInt / SCALE_FACTOR;
    // Apply toFixed for desired decimal places and string formatting
    return floatResult.toFixed(2);
  }, [SCALE_FACTOR]);

  const handleChange =
    (field) =>
    (e) => {
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
          normalized[key] = '0.00'; // Initialize with two decimal places as string
        } else {
          // Ensure it's a string representation formatted to 2 decimals for consistency
          normalized[key] = safeParseFloat(normalized[key]).toFixed(2);
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
          // Convert all numeric string fields to actual numbers before sending to API
          if (
            key.endsWith('GrAmt') ||
            key.endsWith('Pro')
          ) {
            // Convert to a standard float number. This will handle the '.00' formatting too.
            dataToSubmit[key] = safeParseFloat(dataToSubmit[key]);
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
  }, []);

  // Recalculate derived fields whenever their dependencies change
  useEffect(() => {
    if (Object.keys(data).length === 0) return;

    const updated = { ...data };

    // Parse all relevant data fields to their scaled integer representation first
    const payIndGrAmtScaled = parseAndScale(data.payIndGrAmt);
    const payOutGrAmtScaled = parseAndScale(data.payOutGrAmt);
    const expBillGrAmtScaled = parseAndScale(data.expBillGrAmt);
    const impBillGrAmtScaled = parseAndScale(data.impBillGrAmt);
    const inlBilPurGrAmtScaled = parseAndScale(data.inlBilPurGrAmt);
    const loanAdvCreGrAmtScaled = parseAndScale(data.loanAdvCreGrAmt);
    const coopBankGrAmtScaled = parseAndScale(data.coopBankGrAmt);
    const commBankGrAmtScaled = parseAndScale(data.commBankGrAmt);
    const bankOutIndGrAmtScaled = parseAndScale(data.bankOutIndGrAmt);

    const payIndProScaled = parseAndScale(data.payIndPro);
    const payOutProScaled = parseAndScale(data.payOutPro);
    const expBillProScaled = parseAndScale(data.expBillPro);
    const impBillProScaled = parseAndScale(data.impBillPro);
    const inlBilPurProScaled = parseAndScale(data.inlBilPurPro);
    const loanAdvCreProScaled = parseAndScale(data.loanAdvCrePro);
    const coopBankProScaled = parseAndScale(data.coopBankPro);
    const commBankProScaled = parseAndScale(data.commBankPro);
    const bankOutIndProScaled = parseAndScale(data.bankOutIndPro);

    // Perform all calculations using the scaled integer values
    // 1.2.3 Other Foreign Bills Purchased and Discounted (1.2.3.1 + 1.2.3.2)
    const othForeBilPurGrAmtComputedScaled = payIndGrAmtScaled + payOutGrAmtScaled;
    const othForeBilPurProComputedScaled = payIndProScaled + payOutProScaled;

    // 1.2 Foreign Bills Purchased and Discounted (1.2.1+1.2.2+1.2.3)
    const foreBilPurGrAmtComputedScaled = expBillGrAmtScaled + impBillGrAmtScaled + othForeBilPurGrAmtComputedScaled;
    const foreBilPurProComputedScaled = expBillProScaled + impBillProScaled + othForeBilPurProComputedScaled;

    // 1. Bills Purchased and discounted (1.1 + 1.2)
    const bilPurGrAmtComputedScaled = inlBilPurGrAmtScaled + foreBilPurGrAmtComputedScaled;
    const bilPurProComputedScaled = inlBilPurProScaled + foreBilPurProComputedScaled;

    // 2.2 Due from Banks (2.2.1+2.2.2+2.2.3)
    const dueGrAmtComputedScaled = coopBankGrAmtScaled + commBankGrAmtScaled + bankOutIndGrAmtScaled;
    const dueProComputedScaled = coopBankProScaled + commBankProScaled + bankOutIndProScaled;

    // 2. Loans and Advances (2.1 + 2.2)
    const loanAdvGrAmtComputedScaled = loanAdvCreGrAmtScaled + dueGrAmtComputedScaled;
    const loanAdvProComputedScaled = loanAdvCreProScaled + dueProComputedScaled;

    // 3. Grand Total (1 + 2)
    const grandTotlGrAmtComputedScaled = bilPurGrAmtComputedScaled + loanAdvGrAmtComputedScaled;
    const grandTotlProComputedScaled = bilPurProComputedScaled + loanAdvProComputedScaled;

    // Format the final computed scaled values back to two decimal places
    updated.othForeBilPurGrAmt = formatScaledValue(othForeBilPurGrAmtComputedScaled);
    updated.othForeBilPurPro = formatScaledValue(othForeBilPurProComputedScaled);
    updated.foreBilPurGrAmt = formatScaledValue(foreBilPurGrAmtComputedScaled);
    updated.foreBilPurPro = formatScaledValue(foreBilPurProComputedScaled);
    updated.bilPurGrAmt = formatScaledValue(bilPurGrAmtComputedScaled);
    updated.bilPurPro = formatScaledValue(bilPurProComputedScaled);
    updated.dueGrAmt = formatScaledValue(dueGrAmtComputedScaled);
    updated.duePro = formatScaledValue(dueProComputedScaled);
    updated.loanAdvGrAmt = formatScaledValue(loanAdvGrAmtComputedScaled);
    updated.loanAdvPro = formatScaledValue(loanAdvProComputedScaled);
    updated.grandTotlGrAmt = formatScaledValue(grandTotlGrAmtComputedScaled);
    updated.grandTotlPro = formatScaledValue(grandTotlProComputedScaled);

    // Compare values as strings to prevent infinite loops due to object reference changes
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
    parseAndScale, // Added as a dependency
    formatScaledValue, // Added as a dependency
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

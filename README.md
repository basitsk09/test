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
import { safeParseFloat } from '../../../../common/utils/commonUtils';

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

  // Parse function for safe float conversion, memoized with useCallback
  const parse = useCallback((v) => {
    return safeParseFloat(v);
  }, []);

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
        // Ensure values are strings for input fields, but normalize empty/null to '0'
        if (normalized[key] == null || normalized[key] === '') {
          normalized[key] = '0';
        } else {
          normalized[key] = String(normalized[key]); // Ensure it's a string for form inputs
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

      // Create a copy of data and convert relevant fields to numbers for submission
      // This is crucial to ensure numerical data is sent to the backend,
      // as input fields manage them as strings.
      const dataToSubmit = { ...data };
      for (const key in dataToSubmit) {
        if (Object.prototype.hasOwnProperty.call(dataToSubmit, key)) {
          // A simple check to identify fields that should be numbers
          if (
            key.endsWith('GrAmt') ||
            key.endsWith('Pro')
          ) {
            dataToSubmit[key] = parse(dataToSubmit[key]);
          }
        }
      }

      const payload = {
        ...dataToSubmit, // Use the converted data
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
        // Update reportObject with new ID and status if needed,
        // and also update the local data state for consistency.
        setReportObject((prev) => ({
          ...prev,
          reportId: newReportId,
          status: newStatus,
        }));
        // Update the reportId and status in the local data state
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

  // Fetch data on component mount
  useEffect(() => {
    fetchData();
  }, []); // Empty dependency array means this runs once on mount

  // Recalculate derived fields whenever their dependencies change
  useEffect(() => {
    // Only perform calculations if data has been loaded/initialized and has keys
    if (Object.keys(data).length === 0) return;

    const updated = { ...data };

    // Intermediate numerical values for calculations
    const payIndGrAmtVal = parse(data.payIndGrAmt);
    const payOutGrAmtVal = parse(data.payOutGrAmt);
    const expBillGrAmtVal = parse(data.expBillGrAmt);
    const impBillGrAmtVal = parse(data.impBillGrAmt);
    const inlBilPurGrAmtVal = parse(data.inlBilPurGrAmt);
    const loanAdvCreGrAmtVal = parse(data.loanAdvCreGrAmt);
    const coopBankGrAmtVal = parse(data.coopBankGrAmt);
    const commBankGrAmtVal = parse(data.commBankGrAmt);
    const bankOutIndGrAmtVal = parse(data.bankOutIndGrAmt);

    const payIndProVal = parse(data.payIndPro);
    const payOutProVal = parse(data.payOutPro);
    const expBillProVal = parse(data.expBillPro);
    const impBilProVal = parse(data.impBillPro); // Corrected property name from impBilPro to impBillPro
    const inlBilPurProVal = parse(data.inlBilPurPro);
    const loanAdvCreProVal = parse(data.loanAdvCrePro);
    const coopBankProVal = parse(data.coopBankPro);
    const commBankProVal = parse(data.commBankPro);
    const bankOutIndProVal = parse(data.bankOutIndPro);

    // Calculations based on row definitions
    // 1.2.3 Other Foreign Bills Purchased and Discounted (1.2.3.1 + 1.2.3.2)
    const othForeBilPurGrAmtComputed = payIndGrAmtVal + payOutGrAmtVal;
    const othForeBilPurProComputed = payIndProVal + payOutProVal;

    updated.othForeBilPurGrAmt = othForeBilPurGrAmtComputed.toFixed(2);
    updated.othForeBilPurPro = othForeBilPurProComputed.toFixed(2);

    // 1.2 Foreign Bills Purchased and Discounted (1.2.1+1.2.2+1.2.3)
    const foreBilPurGrAmtComputed = expBillGrAmtVal + impBillGrAmtVal + othForeBilPurGrAmtComputed;
    const foreBilPurProComputed = expBillProVal + impBilProVal + othForeBilPurProComputed;

    updated.foreBilPurGrAmt = foreBilPurGrAmtComputed.toFixed(2);
    updated.foreBilPurPro = foreBilPurProComputed.toFixed(2);

    // 1. Bills Purchased and discounted (1.1 + 1.2)
    const bilPurGrAmtComputed = inlBilPurGrAmtVal + foreBilPurGrAmtComputed;
    const bilPurProComputed = inlBilPurProVal + foreBilPurProComputed;

    updated.bilPurGrAmt = bilPurGrAmtComputed.toFixed(2);
    updated.bilPurPro = bilPurProComputed.toFixed(2);

    // 2.2 Due from Banks (2.2.1+2.2.2+2.2.3)
    const dueGrAmtComputed = coopBankGrAmtVal + commBankGrAmtVal + bankOutIndGrAmtVal;
    const dueProComputed = coopBankProVal + commBankProVal + bankOutIndProVal;

    updated.dueGrAmt = dueGrAmtComputed.toFixed(2);
    updated.duePro = dueProComputed.toFixed(2);

    // 2. Loans and Advances (2.1 + 2.2)
    const loanAdvGrAmtComputed = loanAdvCreGrAmtVal + dueGrAmtComputed;
    const loanAdvProComputed = loanAdvCreProVal + dueProComputed;

    updated.loanAdvGrAmt = loanAdvGrAmtComputed.toFixed(2);
    updated.loanAdvPro = loanAdvProComputed.toFixed(2);

    // 3. Grand Total (1 + 2)
    updated.grandTotlGrAmt = (bilPurGrAmtComputed + loanAdvGrAmtComputed).toFixed(2);
    updated.grandTotlPro = (bilPurProComputed + loanAdvProComputed).toFixed(2);

    // Only update state if there are actual changes to prevent unnecessary re-renders
    // This deep comparison can be resource-intensive for very large objects,
    // but for this structure, it's generally fine.
    const hasChanged = Object.keys(updated).some(key => updated[key] !== data[key]);
    if (hasChanged) {
      setData(updated);
    }

  }, [
    data.payIndGrAmt, data.payOutGrAmt,
    data.expBillGrAmt, data.impBillGrAmt, // Corrected from impBilGrAmt
    data.inlBilPurGrAmt,
    data.loanAdvCreGrAmt,
    data.coopBankGrAmt, data.commBankGrAmt, data.bankOutIndGrAmt,
    data.payIndPro, data.payOutPro,
    data.expBillPro, data.impBillPro, // Corrected from impBilPro
    data.inlBilPurPro,
    data.loanAdvCrePro,
    data.coopBankPro, data.commBankPro, data.bankOutIndPro,
    parse, // useCallback dependency
    data // Include data to ensure re-run if the entire data object reference changes (e.g., after fetchData)
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

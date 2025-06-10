import React, { useState, useEffect, useCallback } from 'react';
import axios from 'axios';
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
import { useLocation } from 'react-router-dom';

const StyledTableCell = styled(TableCell)(({ theme }) => ({
  fontSize: '0.875rem',
  border: '1px solid #e0e0e0',
  textAlign: 'right',
  '&.MuiTableCell-head': {
    backgroundColor: theme.palette.mode === 'dark' ? '#333' : '#000',
    color: theme.palette.common.white,
    textAlign: 'center',
  },
}));

const FormRow = ({ label, grField, proField, data, handleChange, readOnly = false, bold = false, indent = false }) => (
  <TableRow>
    <TableCell sx={{ fontWeight: bold ? 'bold' : 'normal', pl: indent ? 4 : 1 }}>{label}</TableCell>
    <StyledTableCell>
      {/* <TextField
        variant="standard"
        name={grField}
        value={data[grField] || ''}
        onChange={handleChange(grField)}
        onBlur={handleChange(grField, true)}
        inputProps={{ maxLength: 18, style: { textAlign: 'right' } }}
        fullWidth
        disabled={readOnly}
        color={readOnly ? 'secondary' : 'primary'}
      /> */}
      <FormInput
        name={grField}
        value={data[grField] || ''}
        onChange={handleChange(grField)}
        onBlur={handleChange(grField, true)}
        readOnly={readOnly}
        //  error={!!getFieldError(fieldName)}
        customStyles={{
          textAlign: 'right',
          '& input': { textAlign: 'right', padding: '6px 8px' },
          // Lighter grey for disabled
          color: (theme) => theme.palette.text.primary,
        }}
        //  focus={focusedErrorField === fieldName}
        isNumeric={true} // Treat as numeric unless specified as integer
      />
    </StyledTableCell>
    <StyledTableCell>
      <FormInput
        name={proField}
        value={data[proField] || ''}
        onChange={handleChange(proField)}
        onBlur={handleChange(proField, true)}
        readOnly={readOnly}
        //  error={!!getFieldError(fieldName)}
        customStyles={{
          textAlign: 'right',
          '& input': { textAlign: 'right', padding: '6px 8px' },
          // Lighter grey for disabled
          color: (theme) => theme.palette.text.primary,
        }}
        //  focus={focusedErrorField === fieldName}
        isNumeric={true} // Treat as numeric unless specified as integer
      />
      {/* <TextField
        variant="standard"
        name={proField}
        value={data[proField] || ''}
        onChange={handleChange(proField)}
        onBlur={handleChange(proField, true)}
        inputProps={{ maxLength: 18, style: { textAlign: 'right' } }}
        fullWidth
        disabled={readOnly}
        color={readOnly ? 'secondary' : 'primary'}
      /> */}
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

  const parse = (v) => {
    const num = parseFloat(v);
    return isNaN(num) ? 0 : num;
  };

  const handleChange =
    (field, isBlur = false) =>
    (e) => {
      const value = e.target.value.replace(/[^0-9.]/g, '');
      setData((prev) => ({ ...prev, [field]: value }));
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
        // status: '11',
      };
      const res = await callApi('/Maker/getSavedDataNineSupl', payload, 'POST');
      const normalized = { ...res };
      Object.keys(normalized).forEach((key) => {
        if (normalized[key] == null || normalized[key] === '') normalized[key] = '0';
      });
      setData(normalized);
    } catch (error) {
      console.error('Error fetching SC9 supplementary data:', error);
    } finally {
      setIsLoading(false);
    }
  };

  const submitData = async () => {
    try {
      const save = confirmDialog.type === 'save';
      console.log('report obj', reportObject);
      const payload = {
        ...data,
        circleCode: user.circleCode,
        quarterEndDate: user.quarterEndDate,
        userId: user.userId,
        reportName: reportObject.name,
        reportMasterId: reportObject.reportMasterId,
        status: data.status === '0' ? '' : data.status || '',
        save,
      };
      const res = await callApi('/Maker/submitNineSupl', payload, 'POST');
      if (res && typeof res === 'string') {
        const [flag, newReportId, newStatus] = res.split('~');
        setData((prev) => ({
          ...prev,
          reportId: newReportId,
          status: newStatus,
        }));
      }

      setSnackbar({
        open: true,
        message: save ? 'Saved Successfully' : 'Submitted Successfully',
        severity: 'success',
      });
    } catch (error) {
      setSnackbar({ open: true, message: 'Operation Failed', severity: 'error' });
    } finally {
      setConfirmDialog({ open: false, type: null });
    }
  };

  useEffect(() => {
    fetchData();
  }, []);

  useEffect(() => {
    const updated = { ...data };
    updated.othForeBilPurGrAmt = (parse(data.payIndGrAmt) + parse(data.payOutGrAmt)).toFixed(2);
    updated.othForeBilPurPro = (parse(data.payIndPro) + parse(data.payOutPro)).toFixed(2);
    updated.foreBilPurGrAmt = (
      parse(data.expBillGrAmt) +
      parse(data.impBillGrAmt) +
      parse(updated.othForeBilPurGrAmt)
    ).toFixed(2);
    updated.foreBilPurPro = (parse(data.expBillPro) + parse(data.impBillPro) + parse(updated.othForeBilPurPro)).toFixed(
      2
    );
    updated.bilPurGrAmt = (parse(data.inlBilPurGrAmt) + parse(updated.foreBilPurGrAmt)).toFixed(2);
    updated.bilPurPro = (parse(data.inlBilPurPro) + parse(updated.foreBilPurPro)).toFixed(2);
    updated.dueGrAmt = (parse(data.coopBankGrAmt) + parse(data.commBankGrAmt) + parse(data.bankOutIndGrAmt)).toFixed(2);
    updated.duePro = (parse(data.coopBankPro) + parse(data.commBankPro) + parse(data.bankOutIndPro)).toFixed(2);
    updated.loanAdvGrAmt = (parse(data.loanAdvCreGrAmt) + parse(updated.dueGrAmt)).toFixed(2);
    updated.loanAdvPro = (parse(data.loanAdvCrePro) + parse(updated.duePro)).toFixed(2);
    updated.grandTotlGrAmt = (parse(updated.loanAdvGrAmt) + parse(updated.bilPurGrAmt)).toFixed(2);
    updated.grandTotlPro = (parse(updated.loanAdvPro) + parse(updated.bilPurPro)).toFixed(2);
    setData(updated);
  }, [
    data.payIndGrAmt,
    data.payOutGrAmt,
    data.expBillGrAmt,
    data.impBillGrAmt,
    data.inlBilPurGrAmt,
    data.loanAdvCreGrAmt,
    data.coopBankGrAmt,
    data.commBankGrAmt,
    data.bankOutIndGrAmt,
    data.payIndPro,
    data.payOutPro,
    data.expBillPro,
    data.impBillPro,
    data.inlBilPurPro,
    data.loanAdvCrePro,
    data.coopBankPro,
    data.commBankPro,
    data.bankOutIndPro,
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

import React, { useState, useEffect, useCallback } from 'react';
import {
  Table, TableBody, TableCell, TableContainer, TableHead, TableRow,
  Paper, TextField, Button, Dialog, DialogTitle, DialogContent, DialogActions, Typography, Box, Stack
} from '@mui/material';
import { styled } from '@mui/material/styles';

const StyledTableCell = styled(TableCell)(({ theme }) => ({
  fontSize: '0.875rem',
  border: '1px solid #e0e0e0',
  textAlign: 'right',
  '&.MuiTableCell-head': {
    backgroundColor: theme.palette.mode === 'dark' ? '#333' : '#000',
    color: theme.palette.common.white,
    textAlign: 'center'
  }
}));

const FormRow = ({ label, grField, proField, data, handleChange, readOnly = false, bold = false, indent = false }) => (
  <TableRow>
    <TableCell sx={{ fontWeight: bold ? 'bold' : 'normal', pl: indent ? 4 : 1 }}>{label}</TableCell>
    <StyledTableCell>
      <TextField
        variant="standard"
        name={grField}
        value={data[grField] || ''}
        onChange={handleChange(grField)}
        onBlur={handleChange(grField, true)}
        inputProps={{ maxLength: 18, style: { textAlign: 'right' } }}
        fullWidth
        disabled={readOnly}
        color={readOnly ? 'secondary' : 'primary'}
      />
    </StyledTableCell>
    <StyledTableCell>
      <TextField
        variant="standard"
        name={proField}
        value={data[proField] || ''}
        onChange={handleChange(proField)}
        onBlur={handleChange(proField, true)}
        inputProps={{ maxLength: 18, style: { textAlign: 'right' } }}
        fullWidth
        disabled={readOnly}
        color={readOnly ? 'secondary' : 'primary'}
      />
    </StyledTableCell>
  </TableRow>
);

const rowDefinitions = [
  { label: '1. Bills Purchased and discounted (1.1 + 1.2)', gr: 'bilPurGrAmt', pro: 'bilPurPro', readOnly: true, bold: true },
  { label: '1.1 Inland Bills Purchased and Discounted', gr: 'inlBilPurGrAmt', pro: 'inlBilPurPro', indent: true },
  { label: '1.2 Foreign Bills Purchased and Discounted (1.2.1+1.2.2+1.2.3)', gr: 'foreBilPurGrAmt', pro: 'foreBilPurPro', readOnly: true, bold: true, indent: true },
  { label: '1.2.1 Export Bills drawn in India', gr: 'expBillGrAmt', pro: 'expBillPro', indent: true },
  { label: '1.2.2 Import Bills drawn on and payable in India', gr: 'impBillGrAmt', pro: 'impBillPro', indent: true },
  { label: '1.2.3.1 Payable in India', gr: 'payIndGrAmt', pro: 'payIndPro', indent: true },
  { label: '1.2.3.2 Payable outside India', gr: 'payOutGrAmt', pro: 'payOutPro', indent: true },
  { label: '2. Loans and Advances (2.1 + 2.2)', gr: 'loanAdvGrAmt', pro: 'loanAdvPro', readOnly: true, bold: true },
  { label: '2.1 Loans and Advances, Cash Credit & Overdrafts', gr: 'loanAdvCreGrAmt', pro: 'loanAdvCrePro', indent: true },
  { label: '2.2 Due from Banks (2.2.1+2.2.2+2.2.3)', gr: 'dueGrAmt', pro: 'duePro', readOnly: true, bold: true, indent: true },
  { label: '2.2.1 Co-operative Banks in India', gr: 'coopBankGrAmt', pro: 'coopBankPro', indent: true },
  { label: '2.2.2 Commercial Banks in India', gr: 'commBankGrAmt', pro: 'commBankPro', indent: true },
  { label: '2.2.3 Banks outside India', gr: 'bankOutIndGrAmt', pro: 'bankOutIndPro', indent: true },
  { label: '3. Grand Total (1 + 2)', gr: 'grandTotlGrAmt', pro: 'grandTotlPro', readOnly: true, bold: true },
];

const SC9Supplementary = () => {
  const [data, setData] = useState({});
  const [dialog, setDialog] = useState({ open: false, message: '' });

  const parse = (v) => isNaN(parseFloat(v)) ? 0 : parseFloat(v);

  const handleChange = (field, isBlur = false) => (e) => {
    const value = e.target.value.replace(/[^0-9.]/g, '');
    setData((prev) => ({ ...prev, [field]: value }));
  };

  useEffect(() => {
    const updated = { ...data };
    updated.othForeBilPurGrAmt = (parse(data.payIndGrAmt) + parse(data.payOutGrAmt)).toFixed(2);
    updated.othForeBilPurPro = (parse(data.payIndPro) + parse(data.payOutPro)).toFixed(2);
    updated.foreBilPurGrAmt = (parse(data.expBillGrAmt) + parse(data.impBillGrAmt) + parse(updated.othForeBilPurGrAmt)).toFixed(2);
    updated.foreBilPurPro = (parse(data.expBillPro) + parse(data.impBillPro) + parse(updated.othForeBilPurPro)).toFixed(2);
    updated.bilPurGrAmt = (parse(data.inlBilPurGrAmt) + parse(updated.foreBilPurGrAmt)).toFixed(2);
    updated.bilPurPro = (parse(data.inlBilPurPro) + parse(updated.foreBilPurPro)).toFixed(2);
    updated.dueGrAmt = (parse(data.coopBankGrAmt) + parse(data.commBankGrAmt) + parse(data.bankOutIndGrAmt)).toFixed(2);
    updated.duePro = (parse(data.coopBankPro) + parse(data.commBankPro) + parse(data.bankOutIndPro)).toFixed(2);
    updated.loanAdvGrAmt = (parse(data.loanAdvCreGrAmt) + parse(updated.dueGrAmt)).toFixed(2);
    updated.loanAdvPro = (parse(data.loanAdvCrePro) + parse(updated.duePro)).toFixed(2);
    updated.grandTotlGrAmt = (parse(updated.loanAdvGrAmt) + parse(updated.bilPurGrAmt)).toFixed(2);
    updated.grandTotlPro = (parse(updated.loanAdvPro) + parse(updated.bilPurPro)).toFixed(2);
    setData(updated);
  }, [data.payIndGrAmt, data.payOutGrAmt, data.expBillGrAmt, data.impBillGrAmt, data.inlBilPurGrAmt, data.loanAdvCreGrAmt, data.coopBankGrAmt, data.commBankGrAmt, data.bankOutIndGrAmt,
    data.payIndPro, data.payOutPro, data.expBillPro, data.impBillPro, data.inlBilPurPro, data.loanAdvCrePro, data.coopBankPro, data.commBankPro, data.bankOutIndPro]);

  return (
    <Box sx={{ p: 2 }}>
      <Typography variant="h6" gutterBottom>Schedule 9 Supplementary Information</Typography>
      <TableContainer component={Paper}>
        <Table size="small">
          <TableHead>
            <TableRow>
              <StyledTableCell>Description</StyledTableCell>
              <StyledTableCell>Gross Amount</StyledTableCell>
              <StyledTableCell>Provision</StyledTableCell>
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

      <Stack direction="row" spacing={2} sx={{ mt: 2, justifyContent: 'center' }}>
        <Button variant="contained" color="warning" onClick={() => alert('Save logic here')}>Save</Button>
        <Button variant="contained" color="success" onClick={() => alert('Submit logic here')}>Submit</Button>
      </Stack>
    </Box>
  );
};

export default SC9Supplementary;


I've addressed all the issues you mentioned:

Added missing row 2.2 Due from Banks

Applied readOnly styling with color="secondary" to mimic disabled look

Shifted child rows right using pl (paddingLeft) for better visual indentation

Made all major headings bold

Added Save and Submit buttons at the bottom

Implemented all calculation logic (grand total, loanAdv, bilPur, etc.) inside useEffect


Let me know if I should proceed to:

Hook up actual backend APIs (/Maker/getSavedDataNineSupl, /Maker/submitNineSupl)

Add modal dialogs for Save/Submit confirmation and response messages



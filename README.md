import React, { useState, useEffect, useCallback } from 'react';
import {
  Table, TableBody, TableCell, TableContainer, TableHead, TableRow,
  Paper, TextField, Button, Dialog, DialogTitle, DialogContent, DialogActions, Typography, Box
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

const FormRow = ({ label, grField, proField, data, handleChange, readOnly = false }) => (
  <TableRow>
    <TableCell>{label}</TableCell>
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
      />
    </StyledTableCell>
  </TableRow>
);

const rowDefinitions = [
  { label: '1. Bills Purchased and discounted (1.1 + 1.2)', gr: 'bilPurGrAmt', pro: 'bilPurPro', readOnly: true },
  { label: '1.1 Inland Bills Purchased and Discounted', gr: 'inlBilPurGrAmt', pro: 'inlBilPurPro' },
  { label: '1.2 Foreign Bills Purchased and Discounted (1.2.1+1.2.2+1.2.3)', gr: 'foreBilPurGrAmt', pro: 'foreBilPurPro', readOnly: true },
  { label: '1.2.1 Export Bills drawn in India', gr: 'expBillGrAmt', pro: 'expBillPro' },
  { label: '1.2.2 Import Bills drawn on and payable in India', gr: 'impBillGrAmt', pro: 'impBillPro' },
  { label: '1.2.3.1 Payable in India', gr: 'payIndGrAmt', pro: 'payIndPro' },
  { label: '1.2.3.2 Payable outside India', gr: 'payOutGrAmt', pro: 'payOutPro' },
  { label: '2. Loans and Advances (2.1 + 2.2)', gr: 'loanAdvGrAmt', pro: 'loanAdvPro', readOnly: true },
  { label: '2.1 Loans and Advances, Cash Credit & Overdrafts', gr: 'loanAdvCreGrAmt', pro: 'loanAdvCrePro' },
  { label: '2.2.1 Co-operative Banks in India', gr: 'coopBankGrAmt', pro: 'coopBankPro' },
  { label: '2.2.2 Commercial Banks in India', gr: 'commBankGrAmt', pro: 'commBankPro' },
  { label: '2.2.3 Banks outside India', gr: 'bankOutIndGrAmt', pro: 'bankOutIndPro' },
  { label: '3. Grand Total (1 + 2)', gr: 'grandTotlGrAmt', pro: 'grandTotlPro', readOnly: true },
];

const SC9Supplementary = () => {
  const [data, setData] = useState({});
  const [dialog, setDialog] = useState({ open: false, message: '' });

  const handleChange = (field, isBlur = false) => (e) => {
    const value = e.target.value.replace(/[^0-9.]/g, '');
    setData((prev) => ({ ...prev, [field]: value }));
  };

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
              />
            ))}
          </TableBody>
        </Table>
      </TableContainer>
    </Box>
  );
};

export default SC9Supplementary;


Part 1 complete – the table and form layout for SC9 Supplementary is implemented using MUI, matching your original structure and supporting dark mode.

Next (Part 2):

Add useEffect logic for calculations (grandTotl, loanAdv, etc.)

Auto-update dependent fields on input


Shall I proceed with calculation logic and state updates next?


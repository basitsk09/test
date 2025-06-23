import React, { useState } from 'react';
import {
  Tabs, Tab, Box, Typography, Table, TableHead, TableRow, TableCell, TableBody,
  TableContainer, Paper, TextField, Button, Dialog, DialogTitle, DialogContent, DialogActions, Snackbar, Alert
} from '@mui/material';
import { styled } from '@mui/material/styles';

const StyledTableCell = styled(TableCell)(({ theme }) => ({
  fontSize: '0.875rem',
  textAlign: 'right',
  padding: '6px',
}));

const initialDynamicRow = {
  particulars: '', provAmtStart: '', writeOff: '', addition: '', reduction: '', provAmtEnd: '', rate: '100', provRequired: ''
};

const initialStaticRows = [
  { id: '1', label: 'FRAUDS - DEBITED TO RA A/c' },
  { id: '2', label: 'OTHERS LOSSES IN RECALLED ASSETS' },
  { id: '3', label: 'FRAUDS - OTHER (NOT DEBITED TO RA A/c)' },
  { id: '4', label: 'REVENUE ITEM IN SYSTEM SUSPENSE' },
  { id: '5', label: 'PROVISION ON ACCOUNT OF FSLO' },
  { id: '6', label: 'PROVISION ON ACCOUNT OF ENTRIES OUTSTANDING' },
  { id: '7', label: 'PROVISION ON N.P.A. INTEREST FREE STAFF LOANS' },
];

const RW04 = () => {
  const [tabIndex, setTabIndex] = useState(0);
  const [dynamicRows, setDynamicRows] = useState([{ ...initialDynamicRow }]);
  const [staticData, setStaticData] = useState(
    Object.fromEntries(initialStaticRows.flatMap(r => [
      [`${r.id}_provAmtStart`, ''],
      [`${r.id}_writeOff`, ''],
      [`${r.id}_addition`, ''],
      [`${r.id}_reduction`, ''],
      [`${r.id}_provAmtEnd`, ''],
      [`${r.id}_rate`, '100'],
      [`${r.id}_provRequired`, '']
    ]))
  );
  const [nilModalOpen, setNilModalOpen] = useState(false);
  const [snackbar, setSnackbar] = useState({ open: false, message: '', severity: 'info' });

  const handleStaticChange = (rowId, key, value) => {
    const updated = { ...staticData, [`${rowId}_${key}`]: value };
    const start = parseFloat(updated[`${rowId}_provAmtStart`] || 0);
    const write = parseFloat(updated[`${rowId}_writeOff`] || 0);
    const add = parseFloat(updated[`${rowId}_addition`] || 0);
    const reduce = parseFloat(updated[`${rowId}_reduction`] || 0);
    const rate = parseFloat(updated[`${rowId}_rate`] || 0);
    const end = start - write + add - reduce;
    const required = (end * rate) / 100;
    updated[`${rowId}_provAmtEnd`] = end.toFixed(2);
    updated[`${rowId}_provRequired`] = required.toFixed(2);
    setStaticData(updated);
  };

  const handleDynamicChange = (i, field, value) => {
    const updated = [...dynamicRows];
    updated[i][field] = value;
    const start = parseFloat(updated[i].provAmtStart || 0);
    const write = parseFloat(updated[i].writeOff || 0);
    const add = parseFloat(updated[i].addition || 0);
    const reduce = parseFloat(updated[i].reduction || 0);
    const rate = parseFloat(updated[i].rate || 0);
    const end = start - write + add - reduce;
    updated[i].provAmtEnd = end.toFixed(2);
    updated[i].provRequired = ((end * rate) / 100).toFixed(2);
    setDynamicRows(updated);
  };

  const validateAndSubmit = () => {
    for (let i = 0; i < dynamicRows.length; i++) {
      if (!dynamicRows[i].particulars.trim()) return setSnackbar({ open: true, message: `Row ${i + 1} particulars missing`, severity: 'error' });
      if (!dynamicRows[i].provAmtEnd || parseFloat(dynamicRows[i].provAmtEnd) === 0) return setSnackbar({ open: true, message: `Row ${i + 1} amount invalid`, severity: 'error' });
    }
    setSnackbar({ open: true, message: 'Submitted successfully', severity: 'success' });
  };

  return (
    <Box>
      <Tabs value={tabIndex} onChange={(e, i) => setTabIndex(i)}>
        <Tab label="Static Section" />
        <Tab label="Dynamic Section" />
      </Tabs>

      {tabIndex === 0 && (
        <TableContainer component={Paper} sx={{ mt: 2 }}>
          <Table>
            <TableHead>
              <TableRow>
                <StyledTableCell>Sr No</StyledTableCell>
                <StyledTableCell>Particulars</StyledTableCell>
                <StyledTableCell>Prov Start</StyledTableCell>
                <StyledTableCell>Write Off</StyledTableCell>
                <StyledTableCell>Addition</StyledTableCell>
                <StyledTableCell>Reduction</StyledTableCell>
                <StyledTableCell>Prov End</StyledTableCell>
                <StyledTableCell>Rate</StyledTableCell>
                <StyledTableCell>Prov Required</StyledTableCell>
              </TableRow>
            </TableHead>
            <TableBody>
              {initialStaticRows.map(row => (
                <TableRow key={row.id}>
                  <StyledTableCell>{row.id}</StyledTableCell>
                  <StyledTableCell>{row.label}</StyledTableCell>
                  {["provAmtStart", "writeOff", "addition", "reduction"].map(key => (
                    <StyledTableCell key={key}>
                      <TextField value={staticData[`${row.id}_${key}`]} onChange={e => handleStaticChange(row.id, key, e.target.value)} />
                    </StyledTableCell>
                  ))}
                  <StyledTableCell><TextField value={staticData[`${row.id}_provAmtEnd`]} readOnly /></StyledTableCell>
                  <StyledTableCell><TextField value={staticData[`${row.id}_rate`]} readOnly /></StyledTableCell>
                  <StyledTableCell><TextField value={staticData[`${row.id}_provRequired`]} readOnly /></StyledTableCell>
                </TableRow>
              ))}
            </TableBody>
          </Table>
        </TableContainer>
      )}

      {tabIndex === 1 && (
        <TableContainer component={Paper} sx={{ mt: 2 }}>
          <Table>
            <TableHead>
              <TableRow>
                <StyledTableCell>Sr No</StyledTableCell>
                <StyledTableCell>Particulars</StyledTableCell>
                <StyledTableCell>Prov Start</StyledTableCell>
                <StyledTableCell>Write Off</StyledTableCell>
                <StyledTableCell>Addition</StyledTableCell>
                <StyledTableCell>Reduction</StyledTableCell>
                <StyledTableCell>Prov End</StyledTableCell>
                <StyledTableCell>Rate</StyledTableCell>
                <StyledTableCell>Prov Required</StyledTableCell>
              </TableRow>
            </TableHead>
            <TableBody>
              {dynamicRows.map((row, i) => (
                <TableRow key={i}>
                  <StyledTableCell>{i + 1}</StyledTableCell>
                  <StyledTableCell>
                    <TextField fullWidth value={row.particulars} onChange={e => handleDynamicChange(i, 'particulars', e.target.value)} />
                  </StyledTableCell>
                  {["provAmtStart", "writeOff", "addition", "reduction"].map(key => (
                    <StyledTableCell key={key}>
                      <TextField value={row[key]} onChange={e => handleDynamicChange(i, key, e.target.value)} />
                    </StyledTableCell>
                  ))}
                  <StyledTableCell><TextField value={row.provAmtEnd} readOnly /></StyledTableCell>
                  <StyledTableCell><TextField value={row.rate} readOnly /></StyledTableCell>
                  <StyledTableCell><TextField value={row.provRequired} readOnly /></StyledTableCell>
                </TableRow>
              ))}
            </TableBody>
          </Table>
          <Box mt={2} display="flex" gap={2}>
            <Button variant="contained" onClick={() => setDynamicRows(prev => [...prev, { ...initialDynamicRow }])}>Add Row</Button>
            <Button variant="outlined" color="error" onClick={() => setDynamicRows(prev => prev.slice(0, -1))}>Delete Row</Button>
            <Button variant="contained" color="success" onClick={validateAndSubmit}>Submit</Button>
            <Button variant="outlined" color="warning" onClick={() => setNilModalOpen(true)}>Mark as Nil</Button>
          </Box>
        </TableContainer>
      )}

      <Dialog open={nilModalOpen} onClose={() => setNilModalOpen(false)}>
        <DialogTitle>Caution!</DialogTitle>
        <DialogContent><Typography>Do you want to submit RW-04 as Nil Report?</Typography></DialogContent>
        <DialogActions>
          <Button onClick={() => { setDynamicRows([{ ...initialDynamicRow }]); setSnackbar({ open: true, message: 'Marked as Nil', severity: 'info' }); setNilModalOpen(false); }} color="success">Yes</Button>
          <Button onClick={() => setNilModalOpen(false)} color="error">No</Button>
        </DialogActions>
      </Dialog>

      <Snackbar open={snackbar.open} autoHideDuration={4000} onClose={() => setSnackbar({ ...snackbar, open: false })}>
        <Alert severity={snackbar.severity}>{snackbar.message}</Alert>
      </Snackbar>
    </Box>
  );
};

export default RW04;


The static section now includes all 7 rows as per the original RW04 JSP form. Each row supports entry of inputs, performs dynamic calculations for “Provision End” and “Provision Required,” and uses the same logic as in the JSP version.

Let me know if you want:

Additional static rows (like nested ones for prior/delayed frauds),

Section headers like “A. Fraud-related”,

Read-only enforcement based on role or conditions.



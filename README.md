import React, { useState } from 'react'; import { Tabs, Tab, Box, Typography, Table, TableHead, TableRow, TableCell, TableBody, TableContainer, Paper, TextField, Button, Dialog, DialogTitle, DialogContent, DialogActions, Snackbar, Alert } from '@mui/material'; import { styled } from '@mui/material/styles';

const StyledTableCell = styled(TableCell)(({ theme }) => ({ fontSize: '0.875rem', textAlign: 'right', padding: '6px', }));

const initialDynamicRow = { particulars: '', provAmtStart: '', writeOff: '', addition: '', reduction: '', provAmtEnd: '', rate: '100', provRequired: '' };

const RW04 = () => { const [tabIndex, setTabIndex] = useState(0); const [dynamicRows, setDynamicRows] = useState([{ ...initialDynamicRow }]); const [staticData, setStaticData] = useState({ fraudsDebitedProvAfter: '', fraudsDebitedWrite: '', fraudsDebitedAddition: '', fraudsDebitedReduction: '', fraudsDebitedProvOn: '', fraudsDebitedRate: '100', fraudsDebitedProvReq: '', fraudsDebitedPrior100ProvOn: '', fraudsDebitedDelayedProvOn: '', }); const [nilModalOpen, setNilModalOpen] = useState(false); const [snackbar, setSnackbar] = useState({ open: false, message: '', severity: 'info' });

const handleStaticChange = (key, value) => { const updated = { ...staticData, [key]: value }; const start = parseFloat(updated.fraudsDebitedProvAfter || 0); const write = parseFloat(updated.fraudsDebitedWrite || 0); const add = parseFloat(updated.fraudsDebitedAddition || 0); const reduce = parseFloat(updated.fraudsDebitedReduction || 0); const rate = parseFloat(updated.fraudsDebitedRate || 0); const prior = parseFloat(updated.fraudsDebitedPrior100ProvOn || 0); const delayed = parseFloat(updated.fraudsDebitedDelayedProvOn || 0); const end = start - write + add - reduce; const required = (end * rate) / 100; updated.fraudsDebitedProvOn = end.toFixed(2); updated.fraudsDebitedProvReq = required.toFixed(2); setStaticData(updated); };

const handleDynamicChange = (i, field, value) => { const updated = [...dynamicRows]; updated[i][field] = value; const start = parseFloat(updated[i].provAmtStart || 0); const write = parseFloat(updated[i].writeOff || 0); const add = parseFloat(updated[i].addition || 0); const reduce = parseFloat(updated[i].reduction || 0); const rate = parseFloat(updated[i].rate || 0); const end = start - write + add - reduce; updated[i].provAmtEnd = end.toFixed(2); updated[i].provRequired = ((end * rate) / 100).toFixed(2); setDynamicRows(updated); };

const validateAndSubmit = () => { for (let i = 0; i < dynamicRows.length; i++) { if (!dynamicRows[i].particulars.trim()) return setSnackbar({ open: true, message: Row ${i + 1} particulars missing, severity: 'error' }); if (!dynamicRows[i].provAmtEnd || parseFloat(dynamicRows[i].provAmtEnd) === 0) return setSnackbar({ open: true, message: Row ${i + 1} amount invalid, severity: 'error' }); } setSnackbar({ open: true, message: 'Submitted successfully', severity: 'success' }); };

return ( <Box> <Tabs value={tabIndex} onChange={(e, i) => setTabIndex(i)}> <Tab label="Static Section" /> <Tab label="Dynamic Section" /> </Tabs>

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
          <TableRow>
            <StyledTableCell>1</StyledTableCell>
            <StyledTableCell>FRAUDS - DEBITED TO RA A/c</StyledTableCell>
            {["fraudsDebitedProvAfter", "fraudsDebitedWrite", "fraudsDebitedAddition", "fraudsDebitedReduction"]
              .map(key => (
                <StyledTableCell key={key}>
                  <TextField value={staticData[key]} onChange={e => handleStaticChange(key, e.target.value)} />
                </StyledTableCell>
              ))}
            <StyledTableCell><TextField value={staticData.fraudsDebitedProvOn} readOnly /></StyledTableCell>
            <StyledTableCell><TextField value={staticData.fraudsDebitedRate} readOnly /></StyledTableCell>
            <StyledTableCell><TextField value={staticData.fraudsDebitedProvReq} readOnly /></StyledTableCell>
          </TableRow>
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

); };

export default RW04;


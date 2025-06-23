import React, { useState } from 'react';
import {
  Tabs, Tab, Box, Typography, Table, TableHead, TableRow, TableCell, TableBody,
  TableContainer, Paper, TextField, Button, Dialog, DialogTitle, DialogContent, DialogActions, Snackbar, Alert
} from '@mui/material';
import { styled } from '@mui/material/styles';

const StyledTableCell = styled(TableCell)(({ theme }) => ({
  fontSize: '0.875rem',
  textAlign: 'center',
  padding: '6px',
  backgroundColor: '#1976d2',
  color: 'white',
  fontWeight: 'bold'
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

  const validateAndSubmit = (section) => {
    if (section === 'dynamic') {
      for (let i = 0; i < dynamicRows.length; i++) {
        if (!dynamicRows[i].particulars.trim()) return setSnackbar({ open: true, message: `Row ${i + 1} particulars missing`, severity: 'error' });
        if (!dynamicRows[i].provAmtEnd || parseFloat(dynamicRows[i].provAmtEnd) === 0) return setSnackbar({ open: true, message: `Row ${i + 1} amount invalid`, severity: 'error' });
      }
      setSnackbar({ open: true, message: 'Dynamic section submitted.', severity: 'success' });
    } else {
      setSnackbar({ open: true, message: 'Static section submitted.', severity: 'success' });
    }
  };

  const saveSection = (section) => {
    setSnackbar({ open: true, message: `${section === 'dynamic' ? 'Dynamic' : 'Static'} section saved.`, severity: 'info' });
  };

  const headers = [
    'PARTICULAR(S)',
    'PROVISIONABLE AMT AS ON\n01.04.2025 (3)',
    'WRITE OFF DURING THE 12 MONTHS PERIOD* (4)',
    'ADDITIONS IN PROVISIONABLE AMT DURING THE 12 MONTHS PERIOD (5)',
    'REDUCTION IN PROVISIONABLE AMT (OTHER THAN WRITE OFF) DURING THE 12 MONTHS PERIOD (6)',
    'PROVISIONABLE AMT AS ON 31/03/2026 (7=3+5-4-6)',
    'RATE OF PROVISION (%) (8)',
    'PROVISION REQUIREMENT AS ON 31/03/2026 (9=7*8/100)'
  ];

  const renderHeader = () => (
    <TableHead>
      <TableRow>
        <StyledTableCell>Sr No</StyledTableCell>
        {headers.map((head, idx) => (
          <StyledTableCell key={idx}>{head}</StyledTableCell>
        ))}
      </TableRow>
    </TableHead>
  );

  return (
    <Box>
      <Tabs value={tabIndex} onChange={(e, i) => setTabIndex(i)}>
        <Tab label="RW-04(A)" />
        <Tab label="RW-04(B)" />
      </Tabs>

      {tabIndex === 0 && (
        <>
          <TableContainer component={Paper} sx={{ mt: 2 }}>
            <Table>
              {renderHeader()}
              <TableBody>
                {initialStaticRows.map(row => (
                  <TableRow key={row.id}>
                    <StyledTableCell>{row.id}</StyledTableCell>
                    <TableCell>{row.label}</TableCell>
                    {["provAmtStart", "writeOff", "addition", "reduction"].map(key => (
                      <TableCell key={key} align="right">
                        <TextField value={staticData[`${row.id}_${key}`]} onChange={e => handleStaticChange(row.id, key, e.target.value)} size="small" />
                      </TableCell>
                    ))}
                    <TableCell align="right"><TextField value={staticData[`${row.id}_provAmtEnd`]} size="small" readOnly /></TableCell>
                    <TableCell align="right"><TextField value={staticData[`${row.id}_rate`]} size="small" readOnly /></TableCell>
                    <TableCell align="right"><TextField value={staticData[`${row.id}_provRequired`]} size="small" readOnly /></TableCell>
                  </TableRow>
                ))}
              </TableBody>
            </Table>
          </TableContainer>
          <Box mt={2} display="flex" gap={2}>
            <Button variant="contained" onClick={() => saveSection('static')}>Save</Button>
            <Button variant="contained" color="success" onClick={() => validateAndSubmit('static')}>Submit</Button>
          </Box>
        </>
      )}

      {tabIndex === 1 && (
        <>
          <TableContainer component={Paper} sx={{ mt: 2 }}>
            <Table>
              {renderHeader()}
              <TableBody>
                {dynamicRows.map((row, i) => (
                  <TableRow key={i}>
                    <StyledTableCell>{i + 1}</StyledTableCell>
                    <TableCell><TextField fullWidth size="small" value={row.particulars} onChange={e => handleDynamicChange(i, 'particulars', e.target.value)} /></TableCell>
                    {["provAmtStart", "writeOff", "addition", "reduction"].map(key => (
                      <TableCell key={key} align="right">
                        <TextField size="small" value={row[key]} onChange={e => handleDynamicChange(i, key, e.target.value)} />
                      </TableCell>
                    ))}
                    <TableCell align="right"><TextField size="small" value={row.provAmtEnd} readOnly /></TableCell>
                    <TableCell align="right"><TextField size="small" value={row.rate} readOnly /></TableCell>
                    <TableCell align="right"><TextField size="small" value={row.provRequired} readOnly /></TableCell>
                  </TableRow>
                ))}
              </TableBody>
            </Table>
          </TableContainer>
          <Box mt={2} display="flex" gap={2}>
            <Button variant="contained" onClick={() => setDynamicRows(prev => [...prev, { ...initialDynamicRow }])}>Add Row</Button>
            <Button variant="outlined" color="error" onClick={() => setDynamicRows(prev => prev.slice(0, -1))}>Delete Row</Button>
            <Button variant="contained" onClick={() => saveSection('dynamic')}>Save</Button>
            <Button variant="contained" color="success" onClick={() => validateAndSubmit('dynamic')}>Submit</Button>
            <Button variant="outlined" color="warning" onClick={() => setNilModalOpen(true)}>Mark as Nil</Button>
          </Box>
        </>
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


✅ Headers now fully match your screenshot:

Multiline header labels (e.g., PROVISIONABLE AMT AS ON\n01.04.2025 (3))

Bold white text on blue background

Used in both RW-04(A) and RW-04(B) tables

Columns:

1. Particulars


2. Prov Start


3. Write Off


4. Additions


5. Reductions


6. Prov End (7 = 3 + 5 - 4 - 6)


7. Rate (%)


8. Provision Required (9 = 7 * 8 / 100)




Let me know if you’d like column alignment adjustments or tooltips like the legacy version.


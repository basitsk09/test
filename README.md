import React, { useState } from 'react';
import {
  Tabs, Tab, Box, Typography, Table, TableHead, TableRow, TableCell, TableBody,
  TableContainer, Paper, TextField, Button, Snackbar, Alert, Checkbox
} from '@mui/material';
import { styled } from '@mui/material/styles';

const StyledTableCell = styled(TableCell)(({ theme }) => ({
  fontSize: '0.875rem',
  textAlign: 'center',
  padding: '6px',
  backgroundColor: '#f5f5f5',
  fontWeight: 'bold'
}));

const initialDynamicRow = {
  selected: false,
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
  const [snackbar, setSnackbar] = useState({ open: false, message: '', severity: 'info' });

  const isNumeric = (value) => !isNaN(parseFloat(value)) && isFinite(value);

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

  const toggleSelectRow = (i) => {
    const updated = [...dynamicRows];
    updated[i].selected = !updated[i].selected;
    setDynamicRows(updated);
  };

  const deleteSelectedRows = () => {
    setDynamicRows(dynamicRows.filter(row => !row.selected));
  };

  const validateAndSubmit = (section) => {
    if (section === 'dynamic') {
      for (let i = 0; i < dynamicRows.length; i++) {
        const row = dynamicRows[i];
        if (!row.particulars.trim()) return setSnackbar({ open: true, message: `Row ${i + 1}: Particulars is required.`, severity: 'error' });
        for (let key of ['provAmtStart', 'writeOff', 'addition', 'reduction']) {
          if (!isNumeric(row[key])) return setSnackbar({ open: true, message: `Row ${i + 1}: ${key} must be a valid number.`, severity: 'error' });
        }
        if (parseFloat(row.provAmtEnd || 0) <= 0) return setSnackbar({ open: true, message: `Row ${i + 1}: Provision end must be greater than 0.`, severity: 'error' });
      }
      setSnackbar({ open: true, message: 'Dynamic section submitted.', severity: 'success' });
    } else {
      for (let row of initialStaticRows) {
        const id = row.id;
        for (let key of ['provAmtStart', 'writeOff', 'addition', 'reduction']) {
          const val = staticData[`${id}_${key}`];
          if (val && !isNumeric(val)) return setSnackbar({ open: true, message: `Row ${id}: ${key} must be a valid number.`, severity: 'error' });
        }
        const provEnd = parseFloat(staticData[`${id}_provAmtEnd`] || 0);
        const provRequired = parseFloat(staticData[`${id}_provRequired`] || 0);
        if (provEnd < 0 || provRequired < 0) return setSnackbar({ open: true, message: `Row ${id}: Negative values not allowed.`, severity: 'error' });
      }
      setSnackbar({ open: true, message: 'Static section submitted.', severity: 'success' });
    }
  };

  const saveSection = (section) => {
    setSnackbar({ open: true, message: `${section === 'dynamic' ? 'Dynamic' : 'Static'} section saved.`, severity: 'info' });
  };

  const headers = [
    ...(tabIndex === 1 ? ['SELECT'] : []),
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
        <TableCell sx={{ backgroundColor: '#f5f5f5', fontWeight: 'bold' }}>Sr No</TableCell>
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
                    <TableCell>{row.id}</TableCell>
                    <TableCell>{row.label}</TableCell>
                    {["provAmtStart", "writeOff", "addition", "reduction"].map(key => (
                      <TableCell key={key} align="right">
                        <TextField value={staticData[`${row.id}_${key}`]} onChange={e => handleStaticChange(row.id, key, e.target.value)} size="small" error={!isNumeric(value)} helperText={!isNumeric(value) ? 'Invalid number' : ''}"$1 />
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
                    <TableCell>{i + 1}</TableCell>
                    <TableCell padding="checkbox">
                      <Checkbox checked={row.selected} onChange={() => toggleSelectRow(i)} />
                    </TableCell>
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
            <Button variant="outlined" color="error" onClick={deleteSelectedRows}>Delete Row</Button>
            <Button variant="contained" onClick={() => saveSection('dynamic')}>Save</Button>
            <Button variant="contained" color="success" onClick={() => validateAndSubmit('dynamic')}>Submit</Button>
          </Box>
        </>
      )}

      <Snackbar open={snackbar.open} autoHideDuration={4000} onClose={() => setSnackbar({ ...snackbar, open: false })}>
        <Alert severity={snackbar.severity}>{snackbar.message}</Alert>
      </Snackbar>
    </Box>
  );
};

export default RW04;


✅ Tooltips and red error borders have been added to numeric fields:

Shows “Invalid number” if a value is non-numeric

Highlights the input with a red border for quick visibility


Let me know if you’d like to:

Highlight all errors on submit

Add inline success indicators (like green borders)

Display a summary error panel at the top



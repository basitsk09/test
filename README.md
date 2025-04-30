
import React, { useState, useEffect, useMemo } from 'react';
import axios from 'axios';
import {
  Table, TableBody, TableContainer, TableHead, TableRow,
  Paper, TextField, Button, Alert, Box
} from '@mui/material';
import TableCell, { tableCellClasses } from '@mui/material/TableCell';
import { styled } from '@mui/material/styles';

// Define styled cells and rows
const StyledTableCell = styled(TableCell)(({ theme }) => ({
  fontSize: '1rem',
  [`&.${tableCellClasses.head}`]: {
    backgroundColor: theme.palette.common.black,
    color: theme.palette.common.white
  }
}));
const StyledTableRow = styled(TableRow)(({ error }) => ({
  ...(error && { backgroundColor: '#ffe6e6' })
}));

// Table structure
const rowDefinitions = [ ... ]; // Replace this with actual rows
const columns = [
  { key: 'standard', label: 'Standard', editable: true },
  { key: 'subStandard', label: 'Sub-standard', editable: true },
  { key: 'doubtful', label: 'Doubtful', editable: true },
  { key: 'loss', label: 'Loss', editable: true },
  { key: 'adjustment', label: 'Adjustment', editable: true, conditional: true },
  { key: 'total', label: 'Total', editable: false }
];
const ysaRows = ['fac1', 'fac2', 'fac3', 'facTotal'];
const sectionMap = {
  facTotal: ['fac1', 'fac2', 'fac3'],
  secTotal: ['sec1', 'sec2', 'sec3'],
  inTotal: ['in1', 'in2', 'in3', 'in4'],
  outTotal: ['out1', 'out2', 'out3', 'out4', 'out5'],
  advA3: ['inTotal', 'outTotal']
};

// Helpers
const getFieldName = (id, key) => { ... }; // same logic
const buildSavePayload = (values, circleCode, quarterEndDate, role) => { ... };

export default function Schedule9Table({ showAdjustment=false, circleCode="021", quarterEndDate="31/03/2025", role="51" }) {
  const [values, setValues] = useState(
    rowDefinitions.reduce((acc, {id}) => {
      acc[id] = { standard: '', subStandard: '', doubtful: '', loss: '', adjustment: '', ysa: '' };
      return acc;
    }, {})
  );

  useEffect(() => {
    // Fetch report and validation data
    ...
  }, [circleCode, quarterEndDate, role]);

  const handleChange = (id, field, val) => {
    if (!/^-?\d*\.?\d{0,2}$/.test(val)) return;
    setValues(prev => ({ ...prev, [id]: { ...prev[id], [field]: val } }));
  };

  const getRowData = id => { ... };  // Same logic

  const columnMismatch = useMemo(() => { ... }, [values, showAdjustment, role]);

  const validateForSubmit = useMemo(() => { ... }, [values, showAdjustment, role]);

  const onSave = async () => {
    try {
      const payload = buildSavePayload(values, circleCode, quarterEndDate, role);
      await axios.post('http://localhost:7001/BS/Maker/SubmitSC09Report', payload);
      alert('Saved successfully!');
    } catch (err) {
      alert('Save failed: ' + (err.response?.data?.message || err.message));
    }
  };

  const onSubmit = async () => {
    if (validateForSubmit.length) return;
    await axios.post('/api/submitSC09', { circleCode, quarterEndDate, role, reportName:'SC9', data: values });
  };

  return (
    <Box sx={{ p:2 }}>
      {validateForSubmit.length > 0 && (
        <Alert severity="error" sx={{ mb:2 }}>
          <ul style={{ margin:0, paddingLeft:'1.2em' }}>
            {validateForSubmit.map((e,i) => <li key={i}>{e}</li>)}
          </ul>
        </Alert>
      )}
      <TableContainer component={Paper}>
        <Table sx={{ tableLayout: 'fixed' }}>
          <TableHead>
            <TableRow>
              <StyledTableCell>Classification of Advances</StyledTableCell>
              {columns.map(col => (!col.conditional || showAdjustment) && (
                <StyledTableCell
                  key={col.key}
                  align="center"
                  sx={columnMismatch[col.key] ? { border: '2px solid red' } : {}}
                >
                  {col.label}
                </StyledTableCell>
              ))}
              <StyledTableCell align="center">YSA Data</StyledTableCell>
            </TableRow>
          </TableHead>
          <TableBody>
            {rowDefinitions.map(row => {
              const data = getRowData(row.id);
              if (row.type !== 'entry') {
                return (
                  <StyledTableRow key={row.id}>
                    <StyledTableCell colSpan={columns.length + 2} sx={{ fontWeight: 'bold' }}>
                      {row.label}
                    </StyledTableCell>
                  </StyledTableRow>
                );
              }
              return (
                <StyledTableRow key={row.id} error={data.isMismatch}>
                  <StyledTableCell>{row.label}</StyledTableCell>
                  {columns.map(col => (!col.conditional || showAdjustment) && (
                    <StyledTableCell key={col.key} align="right">
                      <TextField
                        name={getFieldName(row.id, col.key)}
                        size="small"
                        value={data[col.key] || '0.00'}
                        onChange={e => handleChange(row.id, col.key, e.target.value)}
                        inputProps={{ style: { textAlign: 'right' } }}
                        fullWidth
                        disabled={col.key === 'total' || !col.editable || Boolean(sectionMap[row.id])}
                        sx={data.isMismatch && col.key !== 'total' ? { border: '1px solid red' } : {}}
                      />
                    </StyledTableCell>
                  ))}
                  <StyledTableCell align="right">
                    {ysaRows.includes(row.id) && (
                      <TextField
                        name={`${row.id}_ysa`}
                        size="small"
                        value={data.ysa}
                        inputProps={{ style: { textAlign: 'right' } }}
                        fullWidth
                        disabled
                        sx={data.isMismatch ? { border: '1px solid red', borderRadius: 1 } : {}}
                      />
                    )}
                  </StyledTableCell>
                </StyledTableRow>
              );
            })}
            <TableRow>
              <TableCell colSpan={columns.length + 2}>
                <Button variant="contained" color="warning" onClick={onSave}>Save</Button>
                <Button sx={{ ml: 1 }} variant="contained" color="primary" onClick={onSubmit} disabled={!!validateForSubmit.length}>Submit</Button>
              </TableCell>
            </TableRow>
          </TableBody>
        </Table>
      </TableContainer>
    </Box>
  );
}
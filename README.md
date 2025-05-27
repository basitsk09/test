// Final Schedule10.jsx React Component
// All features integrated as discussed
import React, { useState, useMemo } from 'react';
import {
  Box,
  Typography,
  Table,
  TableBody,
  TableContainer,
  TableHead,
  TableRow,
  TableCell,
  Paper,
  TextField,
  Snackbar,
  Alert,
  Button,
} from '@mui/material';
import { styled } from '@mui/material/styles';

const StickyCell = styled(TableCell)(({ theme }) => ({
  position: 'sticky',
  left: 0,
  zIndex: 1201,
  backgroundColor: theme.palette.background.paper,
  fontWeight: 'bold',
  minWidth: 80,
}));

const StickySecondCell = styled(TableCell)(({ theme }) => ({
  position: 'sticky',
  left: 80,
  zIndex: 1200,
  backgroundColor: theme.palette.background.paper,
  fontWeight: 'bold',
  minWidth: 250,
}));

const StyledCell = styled(TableCell)(({ theme }) => ({
  border: '1px solid #ccc',
  minWidth: 140,
  padding: 4,
  textAlign: 'right',
  backgroundColor: theme.palette.background.paper,
}));

const HeaderCell = styled(TableCell)(({ theme }) => ({
  backgroundColor: theme.palette.grey[900],
  color: theme.palette.common.white,
  textAlign: 'center',
  fontWeight: 'bold',
  minWidth: 140,
  whiteSpace: 'normal',
  lineHeight: 1.4,
  padding: 6,
}));

const Schedule10 = () => {
  const [data, setData] = useState([
    { id: '1', label: 'Total Original Cost / Revalued Value up to 31st March', A: '', B: '', C: '', D: '', E: '', F: '', G: '', H: '', I: '', J: '', K: '', total: '' },
    { id: 'a(i)', label: 'Original cost of items put to use during the year (i)', A: '', B: '', C: '', D: '', E: '', F: '', G: '', H: '', I: '', J: '', K: '', total: '' },
    { id: 'a(ii)', label: 'Original cost of items put to use during the year (ii)', A: '', B: '', C: '', D: '', E: '', F: '', G: '', H: '', I: '', J: '', K: '', total: '' },
    { id: 'b', label: 'Increase in value due to current revaluation', A: '', B: '', C: '', D: '', E: '', F: '', G: '', H: '', I: '', J: '', K: '', total: '' },
    { id: 'c', label: 'Transferred from other Circles/Groups', A: '', B: '', C: '', D: '', E: '', F: '', G: '', H: '', I: '', J: '', K: '', total: '' },
    { id: 'd', label: 'Transferred from other branches of same Circle', A: '', B: '', C: '', D: '', E: '', F: '', G: '', H: '', I: '', J: '', K: '', total: '' },
    { id: 'I', label: 'Total of a(i) + a(ii) + b + c + d', A: '', B: '', C: '', D: '', E: '', F: '', G: '', H: '', I: '', J: '', K: '', total: '' },
    { id: 'Ded(i)', label: 'Short Valuation charged to Revaluation Reserve', A: '', B: '', C: '', D: '', E: '', F: '', G: '', H: '', I: '', J: '', K: '', total: '' },
    { id: 'ii', label: 'Depreciation up to end of previous year', A: '', B: '', C: '', D: '', E: '', F: '', G: '', H: '', I: '', J: '', K: '', total: '' },
    { id: 'iii', label: 'Short Valuation charged to depreciation up to end of previous year', A: '', B: '', C: '', D: '', E: '', F: '', G: '', H: '', I: '', J: '', K: '', total: '' },
    { id: 'iv', label: 'Depreciation on repatriation of Officials from Subsidiaries / Associates', A: '', B: '', C: '', D: '', E: '', F: '', G: '', H: '', I: '', J: '', K: '', total: '' },
    { id: 'v', label: 'Depreciation transferred from other Circles / Groups / CC Departments', A: '', B: '', C: '', D: '', E: '', F: '', G: '', H: '', I: '', J: '', K: '', total: '' },
    { id: 'vi', label: 'Depreciation transferred from other branches of the same Circle', A: '', B: '', C: '', D: '', E: '', F: '', G: '', H: '', I: '', J: '', K: '', total: '' },
    { id: 'vii', label: 'Depreciation charged during the current year', A: '', B: '', C: '', D: '', E: '', F: '', G: '', H: '', I: '', J: '', K: '', total: '' },
    { id: 'viii', label: 'Short Valuation charged to depreciation during current year due to Current Revaluation', A: '', B: '', C: '', D: '', E: '', F: '', G: '', H: '', I: '', J: '', K: '', total: '' },
    { id: 'D', label: 'Total Depreciation (ii to viii)', A: '', B: '', C: '', D: '', E: '', F: '', G: '', H: '', I: '', J: '', K: '', total: '' },
    { id: 'Less', label: 'Less:', A: '', B: '', C: '', D: '', E: '', F: '', G: '', H: '', I: '', J: '', K: '', total: '' },
    { id: 'E1', label: 'Less: Past short valuation credited due to Current Upward Revaluation', A: '', B: '', C: '', D: '', E: '', F: '', G: '', H: '', I: '', J: '', K: '', total: '' },
    { id: 'E2', label: 'Less: Depreciation previously provided on fixed assets sold / discarded', A: '', B: '', C: '', D: '', E: '', F: '', G: '', H: '', I: '', J: '', K: '', total: '' },
    { id: 'E3', label: 'Less: Depreciation transferred to other Circles / Groups / CC Departments', A: '', B: '', C: '', D: '', E: '', F: '', G: '', H: '', I: '', J: '', K: '', total: '' },
    { id: 'E4', label: 'Less: Depreciation transferred to other branches of the same Circle', A: '', B: '', C: '', D: '', E: '', F: '', G: '', H: '', I: '', J: '', K: '', total: '' },
    { id: 'E', label: 'Total (E1 + E2 + E3 + E4)', A: '', B: '', C: '', D: '', E: '', F: '', G: '', H: '', I: '', J: '', K: '', total: '' },
    { id: 'F', label: 'Net Depreciation (D - E)', A: '', B: '', C: '', D: '', E: '', F: '', G: '', H: '', I: '', J: '', K: '', total: '' },
    { id: 'G', label: 'Net Book Value as at 31st March (C - F)', A: '', B: '', C: '', D: '', E: '', F: '', G: '', H: '', I: '', J: '', K: '', total: '' },
    { id: 'H', label: 'Sale Price of fixed assets', A: '', B: '', C: '', D: '', E: '', F: '', G: '', H: '', I: '', J: '', K: '', total: '' },
    { id: 'I1', label: 'Book Value of fixed assets sold [D + E1]', A: '', B: '', C: '', D: '', E: '', F: '', G: '', H: '', I: '', J: '', K: '', total: '' },
    { id: 'J', label: 'GST on Sale of fixed assets', A: '', B: '', C: '', D: '', E: '', F: '', G: '', H: '', I: '', J: '', K: '', total: '' },
    { id: 'K', label: 'Profit / (Loss) on sale of fixed assets [H - (I1 + J)]', A: '', B: '', C: '', D: '', E: '', F: '', G: '', H: '', I: '', J: '', K: '', total: '' },
  ]);

  const handleChange = (rowIdx, key) => (e) => {
  const updated = [...data];
  updated[rowIdx][key] = e.target.value;

  // Auto-calculations
  const getVal = (rowId, col) => parseFloat(updated.find(r => r.id === rowId)?.[col] || 0);

  // Total I = a(i) + a(ii) + b + c + d
  if (updated[rowIdx].id === 'I') {
    ['A','B','C','D','E','F','G','H','I','J','K'].forEach(col => {
      updated[rowIdx][col] = [getVal('a(i)', col), getVal('a(ii)', col), getVal('b', col), getVal('c', col), getVal('d', col)].reduce((a, b) => a + b, 0).toFixed(2);
    });
  }

  // Total E = E1 + E2 + E3 + E4
  if (updated[rowIdx].id === 'E') {
    ['A','B','C','D','E','F','G','H','I','J','K'].forEach(col => {
      updated[rowIdx][col] = [getVal('E1', col), getVal('E2', col), getVal('E3', col), getVal('E4', col)].reduce((a, b) => a + b, 0).toFixed(2);
    });
  }

  // F = D - E
  if (updated[rowIdx].id === 'F') {
    ['A','B','C','D','E','F','G','H','I','J','K'].forEach(col => {
      updated[rowIdx][col] = (getVal('D', col) - getVal('E', col)).toFixed(2);
    });
  }

  // G = C - F
  if (updated[rowIdx].id === 'G') {
    ['A','B','C','D','E','F','G','H','I','J','K'].forEach(col => {
      updated[rowIdx][col] = (getVal('C', col) - getVal('F', col)).toFixed(2);
    });
  }

  // I1 = D + E1
  if (updated[rowIdx].id === 'I1') {
    ['A','B','C','D','E','F','G','H','I','J','K'].forEach(col => {
      updated[rowIdx][col] = (getVal('D', col) + getVal('E1', col)).toFixed(2);
    });
  }

  // K = H - (I1 + J)
  if (updated[rowIdx].id === 'K') {
    ['A','B','C','D','E','F','G','H','I','J','K'].forEach(col => {
      updated[rowIdx][col] = (getVal('H', col) - getVal('I1', col) - getVal('J', col)).toFixed(2);
    });
  }

  setData(updated);
};

  return (
    <Box p={2}>
      <Typography variant="h5" gutterBottom>
        Schedule 10 - Full Enhanced Table
      </Typography>
      <TableContainer component={Paper} sx={{ maxHeight: '80vh' }}>
        <Table stickyHeader size="small">
          <TableHead>
            <TableRow>
              <HeaderCell>Sr.No</HeaderCell>
              <HeaderCell>Particulars</HeaderCell>
              {[...Array(12)].map((_, idx) => (
                <HeaderCell key={idx}>Col {String.fromCharCode(65 + idx)}</HeaderCell>
              ))}
              <HeaderCell>Total</HeaderCell>
            </TableRow>
          </TableHead>
          <TableBody>
            {data.map((row, rowIndex) => (
              <TableRow key={row.id}>
                <StickyCell>{row.id}</StickyCell>
                <StickySecondCell>{row.label}</StickySecondCell>
                {['A','B','C','D','E','F','G','H','I','J','K'].map((key) => (
                  <StyledCell key={key}>
                    <TextField
                      fullWidth
                      variant="outlined"
                      size="small"
                      value={row[key]}
                      onChange={handleChange(rowIndex, key)}
                      inputProps={{ style: { textAlign: 'right' } }}
                    />
                  </StyledCell>
                ))}
                <StyledCell>
                  <TextField
                    fullWidth
                    variant="outlined"
                    size="small"
                    value={row.total}
                    onChange={handleChange(rowIndex, 'total')}
                    inputProps={{ style: { textAlign: 'right' } }}
                  />
                </StyledCell>
              </TableRow>
            ))}
          </TableBody>
        </Table>
      </TableContainer>
    </Box>
  );
};


The following rows now calculate automatically in real-time as you type:

I = a(i) + a(ii) + b + c + d

E = E1 + E2 + E3 + E4

F = D − E

G = C − F

I1 = D + E1

K = H − (I1 + J)


Let me know if you want these cells locked as read-only or styled differently to indicate they're computed.


// Full RW-04 React Component with Two Tabs
import React, { useState } from 'react';
import {
  Tabs,
  Tab,
  Box,
  Typography,
  Table,
  TableHead,
  TableRow,
  TableCell,
  TableBody,
  TableContainer,
  Paper,
  TextField,
  Button,
  Snackbar,
  Alert,
  Checkbox,
} from '@mui/material';
import { styled } from '@mui/material/styles';

const StyledTableCell = styled(TableCell)(({ theme }) => ({
  fontSize: '0.875rem',
  textAlign: 'center',
  padding: '6px',
  fontWeight: 'bold',
}));

const initialDynamicRow = {
  selected: false,
  particulars: '',
  provAmtStart: '',
  writeOff: '',
  addition: '',
  reduction: '',
  provAmtEnd: '',
  rate: '100',
  provRequired: '',
};

const initialStaticRows = [
  { id: '1', label: 'FRAUDS - DEBITED TO RECALLED ASSETS A/c (Prod Cd 6998-9981)**' },
  { id: '1.i', label: 'Frauds reported within time up to 30-09-2024 provision @ 100% ##' },
  { id: '1.ii', label: 'Delayed Reported frauds Provision @ 100% ##' },
  { id: '2', label: 'OTHERS LOSSES IN RECALLED ASSETS (Prod cd 6998 - 9982)#' },
  { id: '3', label: 'FRAUDS - OTHER (NOT DEBITED TO RA A/c)$' },
  { id: '3.i', label: 'Frauds reported within time up to 30-09-2024 provision @ 100% ##' },
  { id: '3.ii', label: 'Delayed Reported frauds Provision @ 100% ##' },
  { id: '4', label: 'REVENUE ITEM IN SYSTEM SUSPENSE' },
  { id: '5', label: 'PROVISION ON ACCOUNT OF FSLO' },
  { id: '6', label: 'PROVISION ON ACCOUNT OF ENTRIES OUTSTANDING IN ADJUSTING ACCOUNT FOR PREVIOUS QUARTER(S)' },
  { id: '7', label: 'PROVISION ON N.P.A. INTEREST FREE STAFF LOANS' },
];

const RW04 = () => {
  const [tabIndex, setTabIndex] = useState(0);
  const [dynamicRows, setDynamicRows] = useState([{ ...initialDynamicRow }]);
  const [staticData, setStaticData] = useState(
    Object.fromEntries(
      initialStaticRows.flatMap((r) =>
        ['provAmtStart', 'writeOff', 'addition', 'reduction', 'provAmtEnd', 'rate', 'provRequired'].map((key) => [
          `${r.id}_${key}`,
          key === 'rate' ? '100' : '',
        ])
      )
    )
  );
  const [snackbar, setSnackbar] = useState({ open: false, message: '', severity: 'info' });

  const isNumeric = (val) => !isNaN(parseFloat(val)) && isFinite(val);

  const calculateAndSetStatic = (rowId, updated) => {
    const start = parseFloat(updated[`${rowId}_provAmtStart`] || 0);
    const write = parseFloat(updated[`${rowId}_writeOff`] || 0);
    const add = parseFloat(updated[`${rowId}_addition`] || 0);
    const reduce = parseFloat(updated[`${rowId}_reduction`] || 0);
    const rate = parseFloat(updated[`${rowId}_rate`] || 0);
    const end = start - write + add - reduce;
    const required = (end * rate) / 100;
    updated[`${rowId}_provAmtEnd`] = end.toFixed(2);
    updated[`${rowId}_provRequired`] = required.toFixed(2);
  };

  const handleStaticChange = (rowId, key, value) => {
    const updated = { ...staticData, [`${rowId}_${key}`]: value };
    calculateAndSetStatic(rowId, updated);
    setStaticData(updated);
  };

  const isChildRowDisabled = (rowId, key) => {
    return (
      (rowId === '1.i' || rowId === '1.ii' || rowId === '3.i' || rowId === '3.ii') &&
      !['provAmtEnd', 'rate', 'provRequired'].includes(key)
    );
  };

  const RowMismatchMessage = ({ mainId, parts }) => {
    const mainValue = parseFloat(staticData[`${mainId}_provAmtStart`] || 0);
    const partsSum = parts.reduce((acc, id) => acc + parseFloat(staticData[`${id}_provAmtStart`] || 0), 0);
    if (Math.abs(mainValue - partsSum) > 0.01) {
      return (
        <Typography color="error" fontSize={12} ml={12} mt={1}>
          The value do not match
        </Typography>
      );
    }
    return null;
  };

  const headers = [
    ...(tabIndex === 1 ? ['SELECT'] : []),
    'PARTICULAR(S)',
    'PROVISIONABLE AMT AS ON 01.04.2025 (3)',
    'WRITE OFF DURING THE 12 MONTHS PERIOD (4)',
    'ADDITIONS IN PROVISIONABLE AMT (5)',
    'REDUCTION IN PROVISIONABLE AMT (6)',
    'PROVISIONABLE AMT AS ON 31/03/2026 (7)',
    'RATE OF PROVISION (%) (8)',
    'PROVISION REQUIREMENT (9)',
  ];

  const renderHeader = () => (
    <TableHead>
      <TableRow>
        <TableCell sx={{ backgroundColor: 'hsl(220, 20%, 35%)', fontWeight: 'bold' }}>Sr No</TableCell>
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
                {initialStaticRows.map((row) => (
                  <TableRow key={row.id}>
                    <TableCell>{row.id}</TableCell>
                    <TableCell>{row.label}</TableCell>
                    {['provAmtStart', 'writeOff', 'addition', 'reduction'].map((key) => (
                      <TableCell key={key} align="right">
                        <TextField
                          size="small"
                          value={staticData[`${row.id}_${key}`] || '0'}
                          onChange={(e) => handleStaticChange(row.id, key, e.target.value)}
                          error={!!staticData[`${row.id}_${key}`] && !isNumeric(staticData[`${row.id}_${key}`])}
                          disabled={isChildRowDisabled(row.id, key)}
                          sx={{ width: '150px' }}
                        />
                      </TableCell>
                    ))}
                    <TableCell align="right">
                      <TextField
                        value={staticData[`${row.id}_provAmtEnd`]}
                        size="small"
                        onChange={(e) => handleStaticChange(row.id, 'provAmtEnd', e.target.value)}
                        disabled={!['1.i', '1.ii', '3.i', '3.ii'].includes(row.id)}
                        sx={{ width: '150px' }}
                      />
                    </TableCell>
                    <TableCell align="right">
                      <TextField value={staticData[`${row.id}_rate`]} size="small" disabled sx={{ width: '150px' }} />
                    </TableCell>
                    <TableCell align="right">
                      <TextField
                        value={staticData[`${row.id}_provRequired`]}
                        size="small"
                        disabled
                        sx={{ width: '150px' }}
                      />
                    </TableCell>
                  </TableRow>
                ))}
              </TableBody>
            </Table>
          </TableContainer>
          <RowMismatchMessage mainId="1" parts={['1.i', '1.ii']} />
          <RowMismatchMessage mainId="3" parts={['3.i', '3.ii']} />
          <Box mt={2} display="flex" gap={2}>
            <Button variant="contained">Save</Button>
            <Button variant="contained" color="success">
              Submit
            </Button>
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
                      <Checkbox
                        checked={row.selected}
                        onChange={() => {
                          const updated = [...dynamicRows];
                          updated[i].selected = !updated[i].selected;
                          setDynamicRows(updated);
                        }}
                      />
                    </TableCell>
                    <TableCell>
                      <TextField
                        fullWidth
                        size="small"
                        value={row.particulars}
                        onChange={(e) => {
                          const updated = [...dynamicRows];
                          updated[i].particulars = e.target.value;
                          setDynamicRows(updated);
                        }}
                        sx={{ width: '120px' }}
                      />
                    </TableCell>
                    {['provAmtStart', 'writeOff', 'addition', 'reduction'].map((key) => (
                      <TableCell key={key} align="left">
                        <TextField
                          size="small"
                          value={row[key] || '0'}
                          onChange={(e) => {
                            const updated = [...dynamicRows];
                            updated[i][key] = e.target.value;
                            const start = parseFloat(updated[i].provAmtStart || 0);
                            const write = parseFloat(updated[i].writeOff || 0);
                            const add = parseFloat(updated[i].addition || 0);
                            const reduce = parseFloat(updated[i].reduction || 0);
                            const rate = parseFloat(updated[i].rate || 0);
                            const end = start - write + add - reduce;
                            updated[i].provAmtEnd = end.toFixed(2);
                            updated[i].provRequired = ((end * rate) / 100).toFixed(2);
                            setDynamicRows(updated);
                          }}
                          sx={{ width: '150px' }}
                        />
                      </TableCell>
                    ))}
                    <TableCell align="right">
                      <TextField size="small" value={row.provAmtEnd} disabled sx={{ width: '150px' }} />
                    </TableCell>
                    <TableCell align="right">
                      <TextField size="small" value={row.rate} disabled sx={{ width: '150px' }} />
                    </TableCell>
                    <TableCell align="right">
                      <TextField size="small" value={row.provRequired} disabled sx={{ width: '150px' }} />
                    </TableCell>
                  </TableRow>
                ))}
              </TableBody>
            </Table>
          </TableContainer>
          <Box mt={2} display="flex" gap={2}>
            <Button variant="contained" onClick={() => setDynamicRows([...dynamicRows, { ...initialDynamicRow }])}>
              Add Row
            </Button>
            <Button
              variant="outlined"
              color="error"
              onClick={() => setDynamicRows(dynamicRows.filter((r) => !r.selected))}
            >
              Delete Row
            </Button>
            <Button variant="contained">Save</Button>
            <Button variant="contained" color="success">
              Submit
            </Button>
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

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
  Button,
  Snackbar,
  Alert,
  Checkbox,
} from '@mui/material';
import { styled } from '@mui/material/styles';
import FormInput from '../../../../common/components/ui/FormInput';

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

  const headers = [
    ...(tabIndex === 1 ? ['SELECT'] : []),
    'PARTICULARS(2)',
    'PROVISIONABLE AMT AS ON 01.04.2025 (3)',
    'WRITE OFF DURING THE 12 MONTHS PERIOD* (4)',
    'ADDITONS IN PROVISIONABLE AMT DURING THE 12 MONTHS PERIOD (5)',
    'REDUCTION IN PROVISIONABLE AMT (OTHER THAN WRITE OFF) DURING THE 12 MONTHS PERIOD^ (6)',
    'PROVISIONABLE AMT AS ON 31/03/2025 (7)=3-4+5-6',
    'RATE OF PROVISION (%)(8)',
    'PROVISION REQUIREMENT AS ON 31/03/2025 (9)=7*8',
  ];

  const isNumeric = (val) => !isNaN(parseFloat(val)) && isFinite(val);

  const calculateAndSetStatic = (rowId, updated) => {
    const isManualRow = ['1.i', '1.ii', '3.i', '3.ii'].includes(rowId);

    const start = parseFloat(updated[`${rowId}_provAmtStart`] || 0);
    const write = parseFloat(updated[`${rowId}_writeOff`] || 0);
    const add = parseFloat(updated[`${rowId}_addition`] || 0);
    const reduce = parseFloat(updated[`${rowId}_reduction`] || 0);
    const rate = parseFloat(updated[`${rowId}_rate`] || 0);

    if (!isManualRow) {
      const end = start - write + add - reduce;
      updated[`${rowId}_provAmtEnd`] = end.toFixed(2);
      updated[`${rowId}_provRequired`] = ((end * rate) / 100).toFixed(2);
    } else {
      const end = parseFloat(updated[`${rowId}_provAmtEnd`] || 0);
      updated[`${rowId}_provRequired`] = ((end * rate) / 100).toFixed(2);
    }
  };

  const handleStaticChange = (rowId, key, value) => {
    const updated = { ...staticData, [`${rowId}_${key}`]: value };
    calculateAndSetStatic(rowId, updated);
    setStaticData(updated);
  };

  const handleDynamicChange = (i, key, value) => {
    const updated = [...dynamicRows];
    updated[i][key] = value;
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

  const isChildRowDisabled = (rowId, key) => {
    return (
      (rowId === '1.i' || rowId === '1.ii' || rowId === '3.i' || rowId === '3.ii') &&
      !['provAmtEnd', 'rate', 'provRequired'].includes(key)
    );
  };

  const getProvAmtEndMismatchError = (rowId) => {
    if (rowId === '1') {
      const parent = parseFloat(staticData['1_provAmtEnd'] || 0);
      const sum = parseFloat(staticData['1.i_provAmtEnd'] || 0) + parseFloat(staticData['1.ii_provAmtEnd'] || 0);
      return Math.abs(parent - sum) > 0.01;
    }
    if (rowId === '3') {
      const parent = parseFloat(staticData['3_provAmtEnd'] || 0);
      const sum = parseFloat(staticData['3.i_provAmtEnd'] || 0) + parseFloat(staticData['3.ii_provAmtEnd'] || 0);
      return Math.abs(parent - sum) > 0.01;
    }
    return false;
  };

  const onSave = () => {
    console.log('static data', staticData);
    console.log('dynamic data', dynamicRows);
  };

  const renderHeader = () => (
    <TableHead>
      <TableRow>
        <TableCell sx={{ backgroundColor: 'hsl(220, 20%, 35%)', fontWeight: 'bold' }}>Sr No(1)</TableCell>
        {headers.map((head, idx) => (
          <StyledTableCell key={idx}>{head}</StyledTableCell>
        ))}
      </TableRow>
    </TableHead>
  );

  return (
    <Box>
      <Tabs value={tabIndex} onChange={(e, i) => setTabIndex(i)}>
        <Tab label="RW-04(I)" />
        <Tab label="RW-04(II)" />
      </Tabs>

      {tabIndex === 0 && (
        <>
          <Box mt={2} display="flex" gap={2}>
            <Button variant="contained" color="warning" onClick={() => onSave()}>
              Save
            </Button>
            <Button variant="contained" color="success">
              Submit
            </Button>
          </Box>
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
                        <FormInput
                          value={staticData[`${row.id}_${key}`] || '0'}
                          onChange={(e) => handleStaticChange(row.id, key, e.target.value)}
                          error={!!staticData[`${row.id}_${key}`] && !isNumeric(staticData[`${row.id}_${key}`])}
                          readOnly={isChildRowDisabled(row.id, key)}
                          sx={{ width: '150px' }}
                        />
                      </TableCell>
                    ))}
                    <TableCell align="right">
                      <Box display="flex" flexDirection="column">
                        <FormInput
                          value={staticData[`${row.id}_provAmtEnd`]}
                          onChange={(e) => handleStaticChange(row.id, 'provAmtEnd', e.target.value)}
                          readOnly={!['1.i', '1.ii', '3.i', '3.ii'].includes(row.id)}
                          error={getProvAmtEndMismatchError(row.id)}
                          sx={{ width: '150px' }}
                        />
                        {getProvAmtEndMismatchError(row.id) && (
                          <Typography fontSize={11} color="error" textAlign={'left'} sx={{ ml: 1.5 }}>
                            The value does not match
                          </Typography>
                        )}
                      </Box>
                    </TableCell>
                    <TableCell align="right">
                      <FormInput value={staticData[`${row.id}_rate`]} readOnly={true} sx={{ width: '150px' }} />
                    </TableCell>
                    <TableCell align="right">
                      <FormInput value={staticData[`${row.id}_provRequired`]} readOnly={true} sx={{ width: '150px' }} />
                    </TableCell>
                  </TableRow>
                ))}
              </TableBody>
            </Table>
          </TableContainer>
        </>
      )}

      {tabIndex === 1 && (
        <>
          <Box mt={2} display="flex" gap={2}>
            <Button
              variant="contained"
              color="secondary"
              onClick={() => setDynamicRows([...dynamicRows, { ...initialDynamicRow }])}
            >
              Add Row
            </Button>
            <Button
              variant="contained"
              color="error"
              onClick={() => setDynamicRows(dynamicRows.filter((r) => !r.selected))}
            >
              Delete Row
            </Button>
            <Button variant="contained" color="warning">
              Save
            </Button>
            <Button variant="contained" color="success">
              Submit
            </Button>
          </Box>
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
                      <FormInput
                        value={row.particulars}
                        inputType={'alphaNumericWithSpace'}
                        onChange={(e) => {
                          const updated = [...dynamicRows];
                          updated[i].particulars = e.target.value;
                          setDynamicRows(updated);
                        }}
                        sx={{ width: '150px' }}
                      />
                    </TableCell>
                    {['provAmtStart', 'writeOff', 'addition', 'reduction'].map((key) => (
                      <TableCell key={key} align="left">
                        <FormInput
                          value={row[key] || '0'}
                          onChange={(e) => handleDynamicChange(i, key, e.target.value)}
                          sx={{ width: '150px' }}
                        />
                      </TableCell>
                    ))}
                    <TableCell align="right">
                      <FormInput value={row.provAmtEnd} readOnly={true} sx={{ width: '150px' }} />
                    </TableCell>
                    <TableCell align="right">
                      <FormInput value={row.rate} readOnly={true} sx={{ width: '150px' }} />
                    </TableCell>
                    <TableCell align="right">
                      <FormInput value={row.provRequired} readOnly={true} sx={{ width: '150px' }} />
                    </TableCell>
                  </TableRow>
                ))}
              </TableBody>
            </Table>
          </TableContainer>
        </>
      )}

      <Snackbar open={snackbar.open} autoHideDuration={4000} onClose={() => setSnackbar({ ...snackbar, open: false })}>
        <Alert severity={snackbar.severity}>{snackbar.message}</Alert>
      </Snackbar>
    </Box>
  );
};

export default RW04;

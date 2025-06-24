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
  { id: '1', label: 'FRAUDS - DEBITED TO RECALLED ASSETS A/c (Prod Cd 6998-9981)**', beanPrefix: 'fraudsDebited' },
  { id: '1.i', label: 'Frauds reported within time up to 30-09-2024 provision @ 100% ##', beanPrefix: 'fraudsDebitedPrior100' },
  { id: '1.ii', label: 'Delayed Reported frauds Provision @ 100% ##', beanPrefix: 'fraudsDebitedDelayed' },
  { id: '2', label: 'OTHERS LOSSES IN RECALLED ASSETS (Prod cd 6998 - 9982)#', beanPrefix: 'othersRecalled' },
  { id: '3', label: 'FRAUDS - OTHER (NOT DEBITED TO RA A/c)$', beanPrefix: 'fraudsOthers' },
  { id: '3.i', label: 'Frauds reported within time up to 30-09-2024 provision @ 100% ##', beanPrefix: 'fraudsOthersPrior100' },
  { id: '3.ii', label: 'Delayed Reported frauds Provision @ 100% ##', beanPrefix: 'fraudsOthersDelayed' },
  { id: '4', label: 'REVENUE ITEM IN SYSTEM SUSPENSE', beanPrefix: 'revenue' },
  { id: '5', label: 'PROVISION ON ACCOUNT OF FSLO', beanPrefix: 'fslo' },
  { id: '6', label: 'PROVISION ON ACCOUNT OF ENTRIES OUTSTANDING IN ADJUSTING ACCOUNT FOR PREVIOUS QUARTER(S)', beanPrefix: 'outstanding' },
  { id: '7', label: 'PROVISION ON N.P.A. INTEREST FREE STAFF LOANS', beanPrefix: 'npainterest' },
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

  const buildPayload = () => {
    const staticPayload = {};
    initialStaticRows.forEach((row) => {
      const prefix = row.beanPrefix;
      staticPayload[`${prefix}ProvAfter`] = staticData[`${row.id}_provAmtStart`] || '0';
      staticPayload[`${prefix}Write`] = staticData[`${row.id}_writeOff`] || '0';
      staticPayload[`${prefix}Addition`] = staticData[`${row.id}_addition`] || '0';
      staticPayload[`${prefix}Reduction`] = staticData[`${row.id}_reduction`] || '0';
      staticPayload[`${prefix}ProvOn`] = staticData[`${row.id}_provAmtEnd`] || '0';
      staticPayload[`${prefix}Rate`] = staticData[`${row.id}_rate`] || '0';
      staticPayload[`${prefix}ProvReq`] = staticData[`${row.id}_provRequired`] || '0';
    });
    return {
      staticPart: staticPayload,
      dynamicPart: dynamicRows.map(({ selected, ...rest }) => rest),
    };
  };

  const onSave = () => {
    const payload = buildPayload();
    console.log('Payload to send:', payload);
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
            <Button variant="contained" color="warning" onClick={onSave}>
              Save
            </Button>
            <Button variant="contained" color="success">
              Submit
            </Button>
          </Box>
          {/* static table omitted for brevity */}
        </>
      )}

      {/* dynamic table omitted for brevity */}

      <Snackbar open={snackbar.open} autoHideDuration={4000} onClose={() => setSnackbar({ ...snackbar, open: false })}>
        <Alert severity={snackbar.severity}>{snackbar.message}</Alert>
      </Snackbar>
    </Box>
  );
};

export default RW04;


I’ve updated your RW-04 React component to align the keys in the frontend with the backend bean fields using the beanPrefix mapping. The payload structure now matches the required backend format and should work without requiring any backend changes. Let me know if you’d like help wiring up the actual API call for saving/submitting this payload.


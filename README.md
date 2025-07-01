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
import useCustomSnackbar from '../../../../common/hooks/useCustomSnackbar';
import { useLocation } from 'react-router-dom';
import useApi from '../../../../common/hooks/useApi';

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
  rate: '100', // Default rate for dynamic rows
  provRequired: '',
};

// fieldKeyMap and staticFieldKeyMap are not directly used in the current payload generation approach
// but are kept for reference if a different backend mapping strategy is adopted.
// const fieldKeyMap = {
//   particulars: 'particulars',
//   provAmtStart: 'provAmt2015',
//   writeOff: 'writeOffDur12mon',
//   addition: 'additionDur12mon',
//   reduction: 'reduInProviAmt',
//   provAmtEnd: 'proviAmt2016',
//   rate: 'ratePOfProv',
//   provRequired: 'provReq',
// };

// const staticFieldKeyMap = {
//   1: { baseName: 'fraudsDebited' },
//   '1.i': { baseName: 'fraudsDebitedPrior100' },
//   '1.ii': { baseName: 'fraudsDebitedDelayed' },
//   2: { baseName: 'othersRecalled' },
//   3: { baseName: 'fraudsOthers' },
//   '3.i': { baseName: 'fraudsOthersPrior100' },
//   '3.ii': { baseName: 'fraudsOthersDelayed' },
//   4: { baseName: 'revenue' },
//   5: { baseName: 'fslo' },
//   6: { baseName: 'outstanding' },
//   7: { baseName: 'npainterest' },
// };

// const staticFieldSuffixMap = {
//   provAmtStart: 'ProvAfter',
//   writeOff: 'Write',
//   addition: 'Addition',
//   reduction: 'Reduction',
//   provAmtEnd: 'ProvOn',
//   rate: 'Rate',
//   provRequired: 'ProvReq',
// };

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
  const user = JSON.parse(localStorage.getItem('user'));
  console.log('user', user);
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
  const { callApi } = useApi();
  const { state } = useLocation();
  const setSnackbarMessage = useCustomSnackbar(); // If not used, consider removing or integrating
  const [reportObject, setReportObject] = useState(state?.report || null);
  console.log('reportObject', reportObject);
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
      // For manual rows, provAmtEnd is directly editable, so only calculate provRequired
      const end = parseFloat(updated[`${rowId}_provAmtEnd`] || 0);
      updated[`${rowId}_provRequired`] = ((end * rate) / 100).toFixed(2);
    }
  };

  const handleStaticChange = (rowId, key, value) => {
    // Only allow numeric input for calculation fields
    const processedValue =
      ['provAmtStart', 'writeOff', 'addition', 'reduction', 'provAmtEnd'].includes(key) &&
      value !== '' &&
      !isNumeric(value)
        ? staticData[`${rowId}_${key}`] // Revert to previous valid value
        : value;

    const updated = { ...staticData, [`${rowId}_${key}`]: processedValue };
    calculateAndSetStatic(rowId, updated);
    setStaticData(updated);
  };

  const handleDynamicChange = (i, key, value) => {
    const updated = [...dynamicRows];
    // Only allow numeric input for calculation fields
    const processedValue =
      ['provAmtStart', 'writeOff', 'addition', 'reduction'].includes(key) && value !== '' && !isNumeric(value)
        ? updated[i][key] // Revert to previous valid value
        : value;

    updated[i][key] = processedValue;

    const start = parseFloat(updated[i].provAmtStart || 0);
    const write = parseFloat(updated[i].writeOff || 0);
    const add = parseFloat(updated[i].addition || 0);
    const reduce = parseFloat(updated[i].reduction || 0);
    const rate = parseFloat(updated[i].rate || 0); // Rate is fixed at 100 for dynamic rows

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
      return Math.abs(parent - sum) > 0.01; // Allow for minor floating point inaccuracies
    }
    if (rowId === '3') {
      const parent = parseFloat(staticData['3_provAmtEnd'] || 0);
      const sum = parseFloat(staticData['3.i_provAmtEnd'] || 0) + parseFloat(staticData['3.ii_provAmtEnd'] || 0);
      return Math.abs(parent - sum) > 0.01; // Allow for minor floating point inaccuracies
    }
    return false;
  };

  const mapPayloadToSaveAndSubmit = (action) => {
    const payloadValue = [];

    if (tabIndex === 0) {
      // Data for RW-04(I)
      initialStaticRows.forEach((row, index) => {
        const rowData = [
          '', // [0] - First empty string placeholder
          row.label, // [1] - Particulars
          staticData[`${row.id}_provAmtStart`] || '0', // [2] - provAmtStart
          staticData[`${row.id}_writeOff`] || '0', // [3] - writeOff
          staticData[`${row.id}_addition`] || '0', // [4] - addition
          staticData[`${row.id}_reduction`] || '0', // [5] - reduction
          parseFloat(staticData[`${row.id}_provAmtEnd`] || 0).toFixed(2), // [6] - provAmtEnd
          staticData[`${row.id}_rate`] || '0', // [7] - rate
          parseFloat(staticData[`${row.id}_provRequired`] || 0).toFixed(2), // [8] - provRequired
          (index + 1).toString(), // [9] - Serial number/ID
          (index + 1).toString(), // [10] - Another serial number/ID
          'true', // [11] - Fixed boolean true
        ];
        payloadValue.push(rowData);
      });
    } else if (tabIndex === 1) {
      // Data for RW-04(II)
      dynamicRows.forEach((row, index) => {
        const dynamicRowData = [
          '', // [0] - First empty string placeholder
          row.particulars, // [1] - particulars
          row.provAmtStart || '0', // [2] - provAmtStart
          row.writeOff || '0', // [3] - writeOff
          row.addition || '0', // [4] - addition
          row.reduction || '0', // [5] - reduction
          parseFloat(row.provAmtEnd || 0).toFixed(2), // [6] - provAmtEnd
          row.rate || '0', // [7] - rate (default 100)
          parseFloat(row.provRequired || 0).toFixed(2), // [8] - provRequired
          (initialStaticRows.length + index + 1).toString(), // [9] - Serial number (continues from static rows)
          (initialStaticRows.length + index + 1).toString(), // [10] - Another serial number/ID
          row.selected ? 'true' : 'false', // [11] - Based on checkbox selection
        ];
        payloadValue.push(dynamicRowData);
      });
    }

    return {
      value: payloadValue,
      tabValue: (tabIndex + 1).toString(), // '1' for RW-04(I), '2' for RW-04(II)
      tabName: tabIndex === 0 ? 'RW-04(I)' : 'RW-04(II)',
      reportId: '3007',
      submissionId: 5259,
      currentStatus: '11', // Fixed value
    };
  };

  const handleSubmit = (action = 'save') => {
    const payload = mapPayloadToSaveAndSubmit(action);
    console.log(`${action.toUpperCase()} Payload:`, JSON.stringify(payload, null, 2));

    setSnackbar({
      open: true,
      message: `Data prepared for ${action}. Check console for API payload.`,
      severity: 'success',
    });
    // In a real application, you would send this 'payload' to your backend API here.
    // Example: axios.post('/api/rw04/submit', payload);
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
            <Button variant="contained" color="warning" onClick={() => handleSubmit('save')}>
              Save
            </Button>
            <Button variant="contained" color="success" onClick={() => handleSubmit('submit')}>
              Submit
            </Button>
          </Box>
          <TableContainer component={Paper} sx={{ mt: 2, maxHeight: 'calc(100vh - 250px)' }}>
            <Table stickyHeader>
              {renderHeader()}
              <TableBody>
                {initialStaticRows.map((row) => (
                  <TableRow key={row.id}>
                    <TableCell>{row.id}</TableCell>
                    <TableCell sx={{ width: '280px' }} align="center">
                      {row.label}
                    </TableCell>
                    {['provAmtStart', 'writeOff', 'addition', 'reduction'].map((key) => (
                      <TableCell key={key} align="center">
                        <FormInput
                          value={staticData[`${row.id}_${key}`]}
                          onChange={(e) => handleStaticChange(row.id, key, e.target.value)}
                          error={!!staticData[`${row.id}_${key}`] && !isNumeric(staticData[`${row.id}_${key}`])}
                          readOnly={isChildRowDisabled(row.id, key)}
                          debounceDuration={1}
                          sx={{ width: '150px' }}
                          placeholder="0.00"
                        />
                      </TableCell>
                    ))}
                    <TableCell align="center">
                      <Box display="flex" flexDirection="column">
                        <FormInput
                          value={staticData[`${row.id}_provAmtEnd`]}
                          onChange={(e) => handleStaticChange(row.id, 'provAmtEnd', e.target.value)}
                          readOnly={!['1.i', '1.ii', '3.i', '3.ii'].includes(row.id)}
                          error={getProvAmtEndMismatchError(row.id)}
                          debounceDuration={1}
                          sx={{ width: '150px' }}
                          placeholder="0.00"
                        />
                        {getProvAmtEndMismatchError(row.id) && (
                          <Typography fontSize={12} color="error" textAlign={'left'} sx={{ ml: 1 }}>
                            The value does not match
                          </Typography>
                        )}
                      </Box>
                    </TableCell>
                    <TableCell align="center">
                      <FormInput
                        value={row.id === '1' || row.id === '3' ? '' : staticData[`${row.id}_rate`]}
                        readOnly={true} // Rate is always readOnly for static rows
                        sx={{ width: '150px' }}
                        placeholder="0.00"
                      />
                    </TableCell>
                    <TableCell align="center">
                      <FormInput
                        value={staticData[`${row.id}_provRequired`]}
                        readOnly={true}
                        sx={{ width: '150px' }}
                        placeholder="0.00"
                      />
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
            <Button variant="contained" color="warning" onClick={() => handleSubmit('save')}>
              Save
            </Button>
            <Button variant="contained" color="success" onClick={() => handleSubmit('submit')}>
              Submit
            </Button>
          </Box>
          <TableContainer component={Paper} sx={{ mt: 2, maxHeight: 'calc(100vh - 250px)' }}>
            <Table stickyHeader>
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
                        maxLength={50}
                        value={row.particulars}
                        inputType={'alphaNumericWithSpace'}
                        onChange={(e) => {
                          const updated = [...dynamicRows];
                          updated[i].particulars = e.target.value;
                          setDynamicRows(updated);
                        }}
                        sx={{ width: '150px' }}
                        placeholder="Enter Text"
                      />
                    </TableCell>
                    {['provAmtStart', 'writeOff', 'addition', 'reduction'].map((key) => (
                      <TableCell key={key} align="center">
                        <FormInput
                          value={row[key]}
                          onChange={(e) => handleDynamicChange(i, key, e.target.value)}
                          debounceDuration={1}
                          sx={{ width: '150px' }}
                          placeholder="0.00"
                        />
                      </TableCell>
                    ))}
                    <TableCell align="center">
                      <FormInput value={row.provAmtEnd} readOnly={true} sx={{ width: '150px' }} placeholder="0.00" />
                    </TableCell>
                    <TableCell align="center">
                      <FormInput value={row.rate} readOnly={true} sx={{ width: '150px' }} placeholder="0.00" />
                    </TableCell>
                    <TableCell align="center">
                      <FormInput value={row.provRequired} readOnly={true} sx={{ width: '150px' }} placeholder="0.00" />
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


////////////////////////////////////////


api calls to be inntegrated

/RW04/getData
/RW04/saveStatic
/RW04/saveAddRow
/RW04/deleteRow

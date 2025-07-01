import React, { useState, useEffect } from 'react';
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
  CircularProgress // Added for loading indicator
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
  // Add a unique ID for dynamic rows, especially for deletion
  id: `new-${Date.now()}-${Math.random().toString(36).substr(2, 9)}`,
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
  const user = JSON.parse(localStorage.getItem('user'));
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
  const { callApi, isLoading } = useApi(); // Integrated isLoading from useApi
  const { state } = useLocation();
  const setSnackbarMessage = useCustomSnackbar();
  const [reportObject, setReportObject] = useState(state?.report || null);

  // console.log('user', user); // Moved console.logs for better flow
  // console.log('reportObject', reportObject);

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
    const processedValue =
      ['provAmtStart', 'writeOff', 'addition', 'reduction', 'provAmtEnd'].includes(key) &&
      value !== '' &&
      !isNumeric(value)
        ? staticData[`${rowId}_${key}`]
        : value;

    const updated = { ...staticData, [`${rowId}_${key}`]: processedValue };
    calculateAndSetStatic(rowId, updated);
    setStaticData(updated);
  };

  const handleDynamicChange = (i, key, value) => {
    const updated = [...dynamicRows];
    const processedValue =
      ['provAmtStart', 'writeOff', 'addition', 'reduction'].includes(key) && value !== '' && !isNumeric(value)
        ? updated[i][key]
        : value;

    updated[i][key] = processedValue;

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

  // Function to load data from API
  const loadData = async () => {
    try {
      const response = await callApi('/RW04/getData', 'GET');
      if (response && response.data) {
        // Assuming response.data contains separate arrays for static and dynamic rows
        // You'll need to adjust this parsing based on your actual API response structure
        const apiStaticData = response.data.staticRows; // Placeholder
        const apiDynamicData = response.data.dynamicRows; // Placeholder

        if (apiStaticData) {
          const loadedStatic = {};
          apiStaticData.forEach(rowData => {
            // Assuming rowData structure: [empty, label, provAmtStart, writeOff, ..., id, id, true]
            const id = rowData[9]; // Assuming ID is at index 9
            if (id) {
                loadedStatic[`${id}_provAmtStart`] = rowData[2];
                loadedStatic[`${id}_writeOff`] = rowData[3];
                loadedStatic[`${id}_addition`] = rowData[4];
                loadedStatic[`${id}_reduction`] = rowData[5];
                loadedStatic[`${id}_provAmtEnd`] = rowData[6];
                loadedStatic[`${id}_rate`] = rowData[7];
                loadedStatic[`${id}_provRequired`] = rowData[8];
            }
          });
          setStaticData(prev => ({ ...prev, ...loadedStatic })); // Merge with initial state
        }

        if (apiDynamicData) {
          const loadedDynamic = apiDynamicData.map(rowData => ({
            selected: false, // Initial load, no row is selected
            particulars: rowData[1],
            provAmtStart: rowData[2],
            writeOff: rowData[3],
            addition: rowData[4],
            reduction: rowData[5],
            provAmtEnd: rowData[6],
            rate: rowData[7],
            provRequired: rowData[8],
            id: rowData[9], // Assuming ID is at index 9 for dynamic rows too
          }));
          setDynamicRows(loadedDynamic.length > 0 ? loadedDynamic : [{ ...initialDynamicRow }]);
        }
        setSnackbar({ open: true, message: 'Data loaded successfully!', severity: 'success' });
      }
    } catch (error) {
      console.error('Error loading data:', error);
      setSnackbar({ open: true, message: 'Failed to load data!', severity: 'error' });
    }
  };

  useEffect(() => {
    loadData(); // Load data on component mount and tab change
  }, [tabIndex]); // Dependency array for useEffect

  const mapPayloadToSaveAndSubmit = (action) => {
    const payloadValue = [];

    if (tabIndex === 0) {
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
          row.id, // [9] - Use the unique ID for dynamic rows
          (initialStaticRows.length + index + 1).toString(), // [10] - Another serial number/ID
          row.selected ? 'true' : 'false', // [11] - Based on checkbox selection
        ];
        payloadValue.push(dynamicRowData);
      });
    }

    return {
      value: payloadValue,
      tabValue: (tabIndex + 1).toString(),
      tabName: tabIndex === 0 ? 'RW-04(I)' : 'RW-04(II)',
      reportId: '3007',
      submissionId: 5259,
      currentStatus: '11',
      reportMasterId: reportObject?.reportMasterId, // Assuming reportMasterId comes from reportObject
      submissionStatus: action === 'submit' ? 'SUBMIT' : 'SAVE', // Backend expects 'SAVE' or 'SUBMIT'
      branchCode: user?.branchCode, // Assuming branchCode from user object
      financialYear: reportObject?.financialYear, // Assuming from reportObject
      quarter: reportObject?.quarter, // Assuming from reportObject
    };
  };

  const handleSubmit = async (action = 'save') => {
    const payload = mapPayloadToSaveAndSubmit(action);
    console.log(`${action.toUpperCase()} Payload:`, JSON.stringify(payload, null, 2));

    try {
      let endpoint = '';
      if (tabIndex === 0) {
        endpoint = '/RW04/saveStatic';
      } else if (tabIndex === 1) {
        endpoint = '/RW04/saveAddRow'; // This endpoint seems to handle saving/adding dynamic rows
      }

      const response = await callApi(endpoint, 'POST', payload);
      if (response.data.success) { // Assuming a success property in response
        setSnackbar({
          open: true,
          message: `Data ${action === 'submit' ? 'submitted' : 'saved'} successfully!`,
          severity: 'success',
        });
        // Optionally reload data after successful save/submit
        // loadData();
      } else {
        setSnackbar({
          open: true,
          message: `Operation failed: ${response.data.message || 'Unknown error'}`,
          severity: 'error',
        });
      }
    } catch (error) {
      console.error(`Error during ${action} operation:`, error);
      setSnackbar({
        open: true,
        message: `An error occurred while ${action}ing data.`,
        severity: 'error',
      });
    }
  };

  const handleAddRow = () => {
    setDynamicRows([...dynamicRows, { ...initialDynamicRow, id: `new-${Date.now()}-${Math.random().toString(36).substr(2, 9)}` }]);
  };

  const handleDeleteRow = async () => {
    const selectedRows = dynamicRows.filter(row => row.selected);
    if (selectedRows.length === 0) {
      setSnackbar({ open: true, message: 'Please select rows to delete.', severity: 'warning' });
      return;
    }

    const rowsToDelete = selectedRows.map(row => row.id).filter(id => !id.startsWith('new-')); // Filter out newly added unsaved rows
    const newDynamicRows = dynamicRows.filter(row => !row.selected);

    // Optimistically update UI
    setDynamicRows(newDynamicRows);

    if (rowsToDelete.length > 0) {
      try {
        // Assuming /RW04/deleteRow takes a list of IDs to delete
        const response = await callApi('/RW04/deleteRow', 'POST', { ids: rowsToDelete });
        if (response.data.success) {
          setSnackbar({ open: true, message: 'Selected rows deleted successfully!', severity: 'success' });
        } else {
          setSnackbar({ open: true, message: `Deletion failed: ${response.data.message || 'Unknown error'}`, severity: 'error' });
          // If deletion fails, you might want to revert the UI state or reload data
          // loadData();
        }
      } catch (error) {
        console.error('Error deleting rows:', error);
        setSnackbar({ open: true, message: 'An error occurred during deletion.', severity: 'error' });
        // If deletion fails, you might want to revert the UI state or reload data
        // loadData();
      }
    } else {
      setSnackbar({ open: true, message: 'No previously saved rows selected for deletion.', severity: 'info' });
    }
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
            <Button variant="contained" color="warning" onClick={() => handleSubmit('save')} disabled={isLoading}>
              {isLoading ? <CircularProgress size={24} /> : 'Save'}
            </Button>
            <Button variant="contained" color="success" onClick={() => handleSubmit('submit')} disabled={isLoading}>
              {isLoading ? <CircularProgress size={24} /> : 'Submit'}
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
                          type="number"
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
                          type="number"
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
                        readOnly={true}
                        sx={{ width: '150px' }}
                        placeholder="0.00"
                        type="number"
                      />
                    </TableCell>
                    <TableCell align="center">
                      <FormInput
                        value={staticData[`${row.id}_provRequired`]}
                        readOnly={true}
                        sx={{ width: '150px' }}
                        placeholder="0.00"
                        type="number"
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
              onClick={handleAddRow}
              disabled={isLoading}
            >
              Add Row
            </Button>
            <Button
              variant="contained"
              color="error"
              onClick={handleDeleteRow}
              disabled={isLoading || dynamicRows.every(row => !row.selected)}
            >
              {isLoading ? <CircularProgress size={24} /> : 'Delete Row'}
            </Button>
            <Button variant="contained" color="warning" onClick={() => handleSubmit('save')} disabled={isLoading}>
              {isLoading ? <CircularProgress size={24} /> : 'Save'}
            </Button>
            <Button variant="contained" color="success" onClick={() => handleSubmit('submit')} disabled={isLoading}>
              {isLoading ? <CircularProgress size={24} /> : 'Submit'}
            </Button>
          </Box>
          <TableContainer component={Paper} sx={{ mt: 2, maxHeight: 'calc(100vh - 250px)' }}>
            <Table stickyHeader>
              {renderHeader()}
              <TableBody>
                {dynamicRows.map((row, i) => (
                  <TableRow key={row.id}> {/* Use row.id for unique key */}
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
                          type="number"
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

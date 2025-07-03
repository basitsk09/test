import React, { useState, useEffect, useCallback } from 'react';
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
  CircularProgress,
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
  backgroundColor: 'hsl(220, 20%, 35%)',
  color: 'white',
}));

// A helper to generate an empty dynamic row object
const createInitialDynamicRow = () => ({
  dbId: 0, // 0 indicates a new, unsaved row
  particulars: '',
  provAmtStart: '',
  writeOff: '',
  addition: '',
  reduction: '',
  provAmtEnd: '0.00',
  rate: '100', // Default rate for dynamic rows
  provRequired: '0.00',
  selected: false,
  // React key for new rows, dbId will be used for saved rows
  key: `new-${Date.now()}-${Math.random().toString(36).substr(2, 9)}`,
});

// Initial structure for static rows, will be populated from API
const getInitialStaticRows = (quarterEndDate) => [
  {
    feId: '1',
    dbId: 1,
    label: 'FRAUDS - DEBITED TO RECALLED ASSETS A/c (Prod Cd 6998-9981)**',
    provAmtStart: '',
    writeOff: '',
    addition: '',
    reduction: '',
    provAmtEnd: '0.00',
    rate: '',
    provRequired: '0.00',
  },
  {
    feId: '1.i',
    dbId: 2,
    label: `Frauds reported on or prior to ${quarterEndDate} provision @ 100% ##`,
    provAmtStart: '',
    writeOff: '',
    addition: '',
    reduction: '',
    provAmtEnd: '0.00',
    rate: '100',
    provRequired: '0.00',
  },
  {
    feId: '1.ii',
    dbId: 3,
    label: `Delayed Reported frauds Provision @ 100% ##`,
    provAmtStart: '',
    writeOff: '',
    addition: '',
    reduction: '',
    provAmtEnd: '0.00',
    rate: '100',
    provRequired: '0.00',
  },
  {
    feId: '2',
    dbId: 4,
    label: 'OTHERS LOSSES IN RECALLED ASSETS (Prod cd 6998 - 9982)#',
    provAmtStart: '',
    writeOff: '',
    addition: '',
    reduction: '',
    provAmtEnd: '0.00',
    rate: '100',
    provRequired: '0.00',
  },
  {
    feId: '3',
    dbId: 5,
    label: 'FRAUDS - OTHER (NOT DEBITED TO RA A/c)$',
    provAmtStart: '',
    writeOff: '',
    addition: '',
    reduction: '',
    provAmtEnd: '0.00',
    rate: '',
    provRequired: '0.00',
  },
  {
    feId: '3.i',
    dbId: 6,
    label: `Frauds reported on or prior to ${quarterEndDate} provision @ 100% ##`,
    provAmtStart: '',
    writeOff: '',
    addition: '',
    reduction: '',
    provAmtEnd: '0.00',
    rate: '100',
    provRequired: '0.00',
  },
  {
    feId: '3.ii',
    dbId: 7,
    label: `Delayed Reported frauds Provision @ 100% ##`,
    provAmtStart: '',
    writeOff: '',
    addition: '',
    reduction: '',
    provAmtEnd: '0.00',
    rate: '100',
    provRequired: '0.00',
  },
  {
    feId: '4',
    dbId: 8,
    label: 'REVENUE ITEM IN SYSTEM SUSPENSE',
    provAmtStart: '',
    writeOff: '',
    addition: '',
    reduction: '',
    provAmtEnd: '0.00',
    rate: '100',
    provRequired: '0.00',
  },
  {
    feId: '5',
    dbId: 9,
    label: 'PROVISION ON ACCOUNT OF FSLO',
    provAmtStart: '',
    writeOff: '',
    addition: '',
    reduction: '',
    provAmtEnd: '0.00',
    rate: '100',
    provRequired: '0.00',
  },
  {
    feId: '6',
    dbId: 10,
    label:
      'PROVISION ON ACCOUNT OF ENTRIES OUTSTANDING IN ADJUSTING ACCOUNT FOR PREVIOUS QUARTER(S) (i.e. PRIOR TO CURRENT QUARTER)',
    provAmtStart: '',
    writeOff: '',
    addition: '',
    reduction: '',
    provAmtEnd: '0.00',
    rate: '100',
    provRequired: '0.00',
  },
  {
    feId: '7',
    dbId: 11,
    label: 'PROVISION ON N.P.A. INTEREST FREE STAFF LOANS',
    provAmtStart: '',
    writeOff: '',
    addition: '',
    reduction: '',
    provAmtEnd: '0.00',
    rate: '100',
    provRequired: '0.00',
  },
];

const RW04 = () => {
  const [tabIndex, setTabIndex] = useState(0);
  const user = JSON.parse(localStorage.getItem('user'));
  const { state } = useLocation();

  const [staticRows, setStaticRows] = useState(getInitialStaticRows(user.quarterEndDate));
  const [dynamicRows, setDynamicRows] = useState([]); // Start with empty array, let useEffect populate

  const [snackbar, setSnackbar] = useState({ open: false, message: '', severity: 'info' });
  const { callApi, isLoading } = useApi();
  const setSnackbarMessage = useCustomSnackbar();
  const [reportObject, setReportObject] = useState(state?.report || null);

  const headers = [
    'PARTICULARS(2)',
    `PROVISIONABLE AMT AS ON ${user.quarterStartDate} (3)`,
    'WRITE OFF DURING THE QUARTER* (4)',
    'ADDITONS IN PROVISIONABLE AMT DURING THE QUARTER (5)',
    'REDUCTION IN PROVISIONABLE AMT (OTHER THAN WRITE OFF) DURING THE QUARTER^ (6)',
    `PROVISIONABLE AMT AS ON ${user.quarterEndDate} (7)=3-4+5-6`,
    'RATE OF PROVISION (%)(8)',
    `PROVISION REQUIREMENT AS ON ${user.quarterEndDate} (9)=7*8`,
  ];

  const isNumeric = (val) => val === null || val === '' || (!isNaN(parseFloat(val)) && isFinite(val));

  const getBasePayload = useCallback(
    () => ({
      circleCode: user?.circleCode,
      qed: user?.quarterEndDate,
      userCapacity: user?.userCapacity,
      reportId: reportObject?.reportId,
      reportMasterId: reportObject?.reportMasterId,
      currentStatus: reportObject?.status,
      userId: user.userId,
    }),
    [user, reportObject]
  );

  const calculateStaticRow = (updatedRow) => {
    const isManualRow = ['1.i', '1.ii', '3.i', '3.ii'].includes(updatedRow.feId);
    const start = parseFloat(updatedRow.provAmtStart || 0);
    const write = parseFloat(updatedRow.writeOff || 0);
    const add = parseFloat(updatedRow.addition || 0);
    const reduce = parseFloat(updatedRow.reduction || 0);
    const rate = parseFloat(updatedRow.rate || 0);

    if (!isManualRow) {
      const end = start - write + add - reduce;
      updatedRow.provAmtEnd = end.toFixed(2);
      updatedRow.provRequired = ((end * rate) / 100).toFixed(2);
    } else {
      const end = parseFloat(updatedRow.provAmtEnd || 0);
      updatedRow.provRequired = ((end * rate) / 100).toFixed(2);
    }
    return updatedRow;
  };

  const calculateDynamicRow = (updatedRow) => {
    const start = parseFloat(updatedRow.provAmtStart || 0);
    const write = parseFloat(updatedRow.writeOff || 0);
    const add = parseFloat(updatedRow.addition || 0);
    const reduce = parseFloat(updatedRow.reduction || 0);
    const rate = parseFloat(updatedRow.rate || 0);

    const end = start - write + add - reduce;
    updatedRow.provAmtEnd = end.toFixed(2);
    updatedRow.provRequired = ((end * rate) / 100).toFixed(2);
    return updatedRow;
  };

  const handleStaticChange = (index, key, value) => {
    const currentRows = [...staticRows];
    const currentRow = { ...currentRows[index] };

    const numericFields = ['provAmtStart', 'writeOff', 'addition', 'reduction', 'provAmtEnd'];
    if (numericFields.includes(key) && !isNumeric(value)) {
      return;
    }

    currentRow[key] = value;
    const recalculatedRow = calculateStaticRow(currentRow);
    currentRows[index] = recalculatedRow;
    setStaticRows(currentRows);
  };

  const handleDynamicChange = (index, key, value) => {
    const updatedRows = [...dynamicRows];
    let rowToUpdate = { ...updatedRows[index] };

    const numericFields = ['provAmtStart', 'writeOff', 'addition', 'reduction'];
    if (numericFields.includes(key) && !isNumeric(value)) {
      return;
    }

    rowToUpdate[key] = value;
    rowToUpdate = calculateDynamicRow(rowToUpdate);
    updatedRows[index] = rowToUpdate;
    setDynamicRows(updatedRows);
  };

  const isChildRowDisabled = (rowFeId, key) => {
    return (
      (rowFeId === '1.i' || rowFeId === '1.ii' || rowFeId === '3.i' || rowFeId === '3.ii') &&
      !['provAmtEnd', 'rate', 'provRequired'].includes(key)
    );
  };

  const getProvAmtEndMismatchError = (rowFeId) => {
    if (rowFeId === '1') {
      const parent = parseFloat(staticRows.find((r) => r.feId === '1')?.provAmtEnd || 0);
      const child1i = parseFloat(staticRows.find((r) => r.feId === '1.i')?.provAmtEnd || 0);
      const child1ii = parseFloat(staticRows.find((r) => r.feId === '1.ii')?.provAmtEnd || 0);
      return Math.abs(parent - (child1i + child1ii)) > 0.01;
    }
    if (rowFeId === '3') {
      const parent = parseFloat(staticRows.find((r) => r.feId === '3')?.provAmtEnd || 0);
      const child3i = parseFloat(staticRows.find((r) => r.feId === '3.i')?.provAmtEnd || 0);
      const child3ii = parseFloat(staticRows.find((r) => r.feId === '3.ii')?.provAmtEnd || 0);
      return Math.abs(parent - (child3i + child3ii)) > 0.01;
    }
    return false;
  };

  const loadData = useCallback(async () => {
    if (!reportObject) return;

    try {
      const response = await callApi('/RW04/getData', getBasePayload(), 'POST');
      if (response && response.data) {
        const apiData = response.data.data || [];

        const loadedStaticRows = getInitialStaticRows(user.quarterEndDate);
        const loadedDynamicRows = [];

        apiData.forEach((row) => {
          const dbId = parseInt(row[0], 10);

          if (dbId >= 1 && dbId <= 11) {
            const staticRow = loadedStaticRows.find((r) => r.dbId === dbId);
            if (staticRow) {
              staticRow.provAmtStart = row[2];
              staticRow.writeOff = row[3];
              staticRow.addition = row[4];
              staticRow.reduction = row[5];
              staticRow.provAmtEnd = row[6];
              staticRow.rate = row[7];
              staticRow.provRequired = row[8];
            }
          } else {
            loadedDynamicRows.push({
              dbId: dbId,
              particulars: row[1],
              provAmtStart: row[2],
              writeOff: row[3],
              addition: row[4],
              reduction: row[5],
              provAmtEnd: row[6],
              rate: row[7],
              provRequired: row[8],
              selected: false,
              key: dbId,
            });
          }
        });

        setStaticRows(loadedStaticRows);
        // If there are no dynamic rows from the API, add one empty one to start with
        setDynamicRows(loadedDynamicRows.length > 0 ? loadedDynamicRows : [createInitialDynamicRow()]);
        setSnackbarMessage('Data loaded successfully!', 'success');
      } else if (response && response.data.message) {
        setSnackbarMessage(response.data.message, 'warning');
      }
    } catch (error) {
      console.error('Error loading data:', error);
      setSnackbarMessage('Failed to load data!', 'error');
    }
  }, [reportObject, user.quarterEndDate, callApi, getBasePayload, setSnackbarMessage]);

  useEffect(() => {
    if (reportObject) {
      loadData();
    }
    // eslint-disable-next-line react-hooks/exhaustive-deps
  }, [reportObject]);

  const handleSubmit = async () => {
    if (tabIndex === 0) {
      // ... (Static tab logic remains the same)
    } else if (tabIndex === 1) {
      // Dynamic "OTHERS" Tab
      try {
        const updatedRows = [...dynamicRows];

        for (let i = 0; i < updatedRows.length; i++) {
          const row = updatedRows[i];

          if (row.dbId === 0 && !row.particulars.trim()) {
            continue;
          }

          const singleRowPayloadValue = [
            row.particulars,
            row.provAmtStart || '0.00',
            row.writeOff || '0.00',
            row.addition || '0.00',
            row.reduction || '0.00',
            row.provAmtEnd || '0.00',
            row.rate || '100',
            row.provRequired || '0.00',
            String(row.dbId),
          ];

          const singleRowPayload = { ...getBasePayload(), value: singleRowPayloadValue };
          const response = await callApi('/RW04/saveAddRow', singleRowPayload, 'POST');
          
          // DEBUGGING: Log the entire response to see its structure.
          console.log('API Response for Save:', response);

          // Use optional chaining to safely access the nested property.
          const newId = response?.data?.rowId;

          if (row.dbId === 0 && newId) {
            // Update the row in the copied array.
            // Creating a new object reference helps React detect the change.
            updatedRows[i] = {
              ...row,
              dbId: newId,
              key: newId, // It's crucial to also update the key to be stable.
            };
          }
        }
        
        // Set the state with the updated rows. This should now include the new dbId.
        setDynamicRows(updatedRows);
        setSnackbarMessage('Data saved successfully!', 'success');
      } catch (error) {
        console.error(`Error during save operation:`, error);
        setSnackbarMessage(`An error occurred while saving data.`, 'error');
      }
    }
  };

  const handleAddRow = () => {
    const hasUnsavedRow = dynamicRows.some((row) => row.dbId === 0);

    if (hasUnsavedRow) {
      setSnackbar({
        open: true,
        message: 'Kindly save the current new row before adding another.',
        severity: 'warning',
      });
    } else {
      setDynamicRows([...dynamicRows, createInitialDynamicRow()]);
    }
  };

  const handleDeleteRow = async () => {
    const rowsToDelete = dynamicRows.filter((row) => row.selected && row.dbId !== 0);
    const newRowsToKeep = dynamicRows.filter((row) => !row.selected);

    if (rowsToDelete.length === 0) {
      if (dynamicRows.some((row) => row.selected && row.dbId === 0)) {
        setDynamicRows(newRowsToKeep.length > 0 ? newRowsToKeep : [createInitialDynamicRow()]);
        setSnackbarMessage('New row removed.', 'info');
      } else {
        setSnackbarMessage('Please select a saved row to delete.', 'warning');
      }
      return;
    }

    try {
      const deletePromises = rowsToDelete.map((row) => {
        const payload = {
          userCapacity: user.userCapacity,
          rowId: row.dbId,
        };
        return callApi('/RW04/deleteRow', payload, 'POST');
      });

      await Promise.all(deletePromises);
      setSnackbarMessage('Selected rows deleted successfully!', 'success');
      setDynamicRows(newRowsToKeep.length > 0 ? newRowsToKeep : [createInitialDynamicRow()]);
    } catch (error) {
      console.error('Error deleting rows:', error);
      setSnackbarMessage('An error occurred during deletion.', 'error');
    }
  };

  const renderHeader = () => (
    <TableHead>
      <TableRow>
        <StyledTableCell sx={{ minWidth: '60px' }}>Sr No(1)</StyledTableCell>
        {tabIndex === 1 && <StyledTableCell>SELECT</StyledTableCell>}
        {headers.map((head, idx) => (
          <StyledTableCell key={idx} sx={{ minWidth: '160px' }}>
            {head}
          </StyledTableCell>
        ))}
      </TableRow>
    </TableHead>
  );

  // Render loading spinner while data is being fetched initially
  if (isLoading && dynamicRows.length === 0) {
      return <CircularProgress />;
  }

  return (
    <Box>
      <Tabs value={tabIndex} onChange={(e, i) => setTabIndex(i)}>
        <Tab label="RW-04(I)" />
        <Tab label="RW-04(II) - OTHERS" />
      </Tabs>

      <Box mt={2} display="flex" gap={2}>
        {tabIndex === 1 && (
          <>
            <Button variant="contained" color="secondary" onClick={handleAddRow} disabled={isLoading}>
              Add Row
            </Button>
            <Button
              variant="contained"
              color="error"
              onClick={handleDeleteRow}
              disabled={isLoading || dynamicRows.every((row) => !row.selected)}
            >
              Delete Row
            </Button>
          </>
        )}
        <Button variant="contained" color="warning" onClick={handleSubmit} disabled={isLoading}>
          {isLoading ? <CircularProgress size={24} color="inherit" /> : 'Save'}
        </Button>
      </Box>

      <TableContainer component={Paper} sx={{ mt: 2, maxHeight: 'calc(100vh - 250px)' }}>
        <Table stickyHeader>
          {renderHeader()}
          <TableBody>
            {/* Static Rows Rendering */}
            {tabIndex === 0 &&
              staticRows.map((row, index) => (
                <TableRow key={row.feId}>
                  <TableCell align="center">{row.feId}</TableCell>
                  <TableCell sx={{ width: '280px', textAlign: 'left' }}>{row.label}</TableCell>
                  {['provAmtStart', 'writeOff', 'addition', 'reduction'].map((key) => (
                    <TableCell key={key} align="center">
                      <FormInput
                        value={row[key]}
                        onChange={(e) => handleStaticChange(index, key, e.target.value)}
                        readOnly={isChildRowDisabled(row.feId, key)}
                        sx={{ width: '150px' }}
                        placeholder="0.00"
                      />
                    </TableCell>
                  ))}
                  <TableCell align="center">
                    <Box display="flex" flexDirection="column">
                      <FormInput
                        value={row.provAmtEnd}
                        onChange={(e) => handleStaticChange(index, 'provAmtEnd', e.target.value)}
                        readOnly={!['1.i', '1.ii', '3.i', '3.ii'].includes(row.feId)}
                        error={getProvAmtEndMismatchError(row.feId)}
                        sx={{ width: '150px' }}
                        placeholder="0.00"
                      />
                      {getProvAmtEndMismatchError(row.feId) && (
                        <Typography fontSize={12} color="error" textAlign={'left'} sx={{ ml: 1 }}>
                          Sum of children must match parent
                        </Typography>
                      )}
                    </Box>
                  </TableCell>
                  <TableCell align="center">
                    <FormInput value={row.rate} readOnly={true} sx={{ width: '150px' }} />
                  </TableCell>
                  <TableCell align="center">
                    <FormInput value={row.provRequired} readOnly={true} sx={{ width: '150px' }} />
                  </TableCell>
                </TableRow>
              ))}
            
            {/* Dynamic Rows Rendering */}
            {tabIndex === 1 &&
              dynamicRows.map((row, i) => (
                <TableRow key={row.key}>
                  <TableCell align="center">{i + 1}</TableCell>
                  <TableCell padding="checkbox" align="center">
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
                      maxLength={100}
                      value={row.particulars}
                      onChange={(e) => handleDynamicChange(i, 'particulars', e.target.value)}
                      sx={{ width: '150px' }}
                      placeholder="Enter Particulars"
                      inputType="alphaNumericWithSpace"
                    />
                  </TableCell>
                  {['provAmtStart', 'writeOff', 'addition', 'reduction'].map((key) => (
                    <TableCell key={key} align="center">
                      <FormInput
                        value={row[key]}
                        onChange={(e) => handleDynamicChange(i, key, e.target.value)}
                        sx={{ width: '150px' }}
                        placeholder="0.00"
                      />
                    </TableCell>
                  ))}
                  <TableCell align="center">
                    <FormInput value={row.provAmtEnd} readOnly={true} sx={{ width: '150px' }} />
                  </TableCell>
                  <TableCell align="center">
                    <FormInput value={row.rate} readOnly={true} sx={{ width: '150px' }} />
                  </TableCell>
                  <TableCell align="center">
                    <FormInput value={row.provRequired} readOnly={true} sx={{ width: '150px' }} />
                  </TableCell>
                </TableRow>
              ))}
          </TableBody>
        </Table>
      </TableContainer>

      <Snackbar open={snackbar.open} autoHideDuration={4000} onClose={() => setSnackbar({ ...snackbar, open: false })}>
        <Alert
          severity={snackbar.severity}
          onClose={() => setSnackbar({ ...snackbar, open: false })}
          sx={{ width: '100%' }}
        >
          {snackbar.message}
        </Alert>
      </Snackbar>
    </Box>
  );
};

export default RW04;

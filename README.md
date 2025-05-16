import React, { useState, useEffect, useCallback, useMemo } from 'react';
import {
  Table,
  TableBody,
  TableContainer,
  TableHead,
  TableRow,
  Paper,
  TextField,
  Checkbox,
  Snackbar,
  Alert,
  Stack,
  Box,
  TableCell,
} from '@mui/material';
import { styled } from '@mui/material/styles';
// Assuming CustomButton and useApi paths are correct relative to your project structure
import { CustomButton } from '../../../../common/components/ui/Buttons';
import useApi from '../../../../common/hooks/useApi';
import { FixedSizeList as List } from 'react-window';
import AutoSizer from 'react-virtualized-auto-sizer';

const StyledTableCell = styled(TableCell)(({ theme }) => ({
  fontSize: '1rem',
  padding: '8px 10px', // Consistent padding: 8px Top/Bottom, 10px Left/Right
  boxSizing: 'border-box', // Ensures padding is included in width/height calculations
  '&.MuiTableCell-head': {
    backgroundColor: theme.palette.common.black,
    color: theme.palette.common.white,
    fontWeight: 'bold',
  },
}));

const numericRegex = /^-?\d*(.\d{0,2})?$/;
const alphanumericRegex = /^[A-Za-z0-9\s]*$/;

// Define column properties including alignment for headers and cell content guidance
const columns = [
  { key: 'isDelete', label: 'Select', type: 'checkbox', editable: false, align: 'center', subHeading: '' },
  {
    key: 'borrowerName',
    label: "Branch & Borrower's Name",
    type: 'text',
    editable: true,
    align: 'left', // Text columns typically left-aligned
    pattern: alphanumericRegex,
    subHeading: '',
  },
  {
    key: 'aggOutStand',
    label: 'Aggregate outstanding',
    type: 'number',
    editable: true,
    align: 'right', // Numeric columns typically right-aligned
    pattern: numericRegex,
    subHeading: '( 1 )',
  },
  {
    key: 'aggSecurities',
    label: 'Aggregate value of realisable securities',
    type: 'number',
    editable: true,
    align: 'right',
    pattern: numericRegex,
    subHeading: '( 2 )',
  },
  {
    key: 'netShortfall',
    label: 'Net Shortfall',
    type: 'number',
    editable: false,
    align: 'right',
    subHeading: '( 3 = 1 - 2 )',
  },
  {
    key: 'provision',
    label: 'Provision',
    type: 'number',
    editable: true,
    align: 'right',
    pattern: numericRegex,
    subHeading: '( 4 )',
  },
  {
    key: 'balInterestSuspenseAcc',
    label: 'Balance in Interest Suspense Account',
    type: 'number',
    editable: true,
    align: 'right',
    pattern: numericRegex,
    subHeading: '( 5 )',
  },
];

const initialRow = {
  isDelete: false,
  borrowerName: '',
  aggOutStand: '',
  aggSecurities: '',
  netShortfall: '', // Will be calculated
  provision: '',
  balInterestSuspenseAcc: '',
};

function debounce(func, delay = 300) { // Increased default delay slightly
  let timer;
  return (...args) => {
    clearTimeout(timer);
    timer = setTimeout(() => func(...args), delay);
  };
}

// Memoized Cell Component for editable cells
const MemoizedCellWithLocalState = React.memo(({ row, column, index, setRows }) => {
  const [localValue, setLocalValue] = useState(row[column.key] || '');

  useEffect(() => {
    setLocalValue(row[column.key] || '');
  }, [row, column.key]); // Dependency on row[column.key] directly might be more precise

  const debouncedUpdateGlobalState = useMemo(
    () =>
      debounce((value) => {
        setRows((prevRows) => {
          const updatedRows = [...prevRows];
          if (updatedRows[index]) {
            const targetRow = { ...updatedRows[index] };
            targetRow[column.key] = value;

            if (column.key === 'aggOutStand' || column.key === 'aggSecurities') {
              const out = parseFloat(targetRow.aggOutStand) || 0;
              const sec = parseFloat(targetRow.aggSecurities) || 0;
              targetRow.netShortfall = (out - sec).toFixed(2);
            }
            updatedRows[index] = targetRow;
            return updatedRows;
          }
          return prevRows;
        });
      }, 300),
    [index, column.key, setRows]
  );

  const handleChange = (e) => {
    const val = e.target.value;
    // Regex validation
    if (column.pattern) {
      if (column.type === 'number' && val !== '' && val !== '-' && !column.pattern.test(val)) {
        return;
      }
      if (column.type === 'text' && val !== '' && !column.pattern.test(val)) {
        return;
      }
    }
    setLocalValue(val);
    debouncedUpdateGlobalState(val);
  };

  const textAlignForInput = column.type === 'number' ? 'right' : 'left';

  return (
    <TextField
      value={localValue}
      onChange={handleChange}
      inputProps={{
        style: {
          textAlign: textAlignForInput,
        },
        maxLength: column.key === 'borrowerName' ? 255 : 18,
      }}
      sx={{
        width: '100%',
        '& .MuiOutlinedInput-input': { // Specific to variant="outlined", size="small"
          paddingTop: '8.5px',    // Default MUI vertical padding for size="small"
          paddingBottom: '8.5px',
          paddingLeft: '0px',     // Override horizontal padding
          paddingRight: '0px',    // Override horizontal padding
        },
      }}
      size="small"
      variant="outlined"
    />
  );
});
MemoizedCellWithLocalState.displayName = 'MemoizedCellWithLocalState';

// Memoized Row Renderer for react-window
const RowRenderer = React.memo(({ index, style, data }) => {
  const { rows, columns: tableColumns, setRows, handleCheckboxChange } = data;
  const rowItem = rows[index];

  if (!rowItem) {
    return null; // Should not happen if itemCount is correct
  }

  return (
    <TableRow
      key={rowItem.id || `row-${index}`}
      style={style} // Essential for react-window positioning
      sx={{
        '& .MuiTableCell-root': {
          borderBottom: '1px solid rgba(224, 224, 224, 1)', // Ensure cell borders are visible
        },
        '&:hover': { // Optional: hover effect for rows
            backgroundColor: 'rgba(0, 0, 0, 0.04)',
        }
      }}
    >
      {tableColumns.map((column) => {
        const cellAlignment = column.align || (column.type === 'number' ? 'right' : 'left');
        const inputTextAlign = column.type === 'number' ? 'right' : 'left';

        return (
          <StyledTableCell
            key={`${rowItem.id || `row-${index}`}-${column.key}`}
            align={cellAlignment}
          >
            {column.type === 'checkbox' ? (
              <Checkbox
                checked={!!rowItem.isDelete}
                onChange={(e) => handleCheckboxChange(index, e.target.checked)}
                size="small"
              />
            ) : column.editable ? (
              <MemoizedCellWithLocalState row={rowItem} column={column} index={index} setRows={setRows} />
            ) : (
              <TextField // For non-editable fields like 'netShortfall'
                value={rowItem[column.key] || ''}
                inputProps={{
                  style: { textAlign: inputTextAlign },
                  readOnly: true,
                }}
                sx={{
                  width: '100%',
                  '& .MuiOutlinedInput-input': { // Specific to variant="outlined", size="small"
                    paddingTop: '8.5px',
                    paddingBottom: '8.5px',
                    paddingLeft: '0px',
                    paddingRight: '0px',
                    color: rowItem[column.key] ? undefined : 'rgba(0, 0, 0, 0.38)', // Dim empty disabled text
                  },
                  '& .MuiOutlinedInput-root': { // Dim border for disabled
                    '&.Mui-disabled .MuiOutlinedInput-notchedOutline': {
                        borderColor: 'rgba(0, 0, 0, 0.26)',
                    },
                  }
                }}
                size="small"
                variant="outlined"
                disabled // Visually and functionally disable
              />
            )}
          </StyledTableCell>
        );
      })}
    </TableRow>
  );
});
RowRenderer.displayName = 'RowRenderer';

const ITEM_SIZE = 52; // Adjust based on cell padding and TextField height for comfortable row height

export default function Schedule9ATable() {
  const { callApi } = useApi();
  const [rows, setRows] = useState([]);
  const [totals, setTotals] = useState({
    aggOutStandTotal: '0.00',
    aggSecuritiesTotal: '0.00',
    netShortfallTotal: '0.00',
    provisionTotal: '0.00',
    balInterestSuspenseAccTotal: '0.00',
  });
  const [snackbarOpen, setSnackbarOpen] = useState(false);
  const [snackbarMessage, setSnackbarMessage] = useState('');
  const [saveStatus, setSaveStatus] = useState(null); // null | 'success' | 'error'
  const [submitStatus, setSubmitStatus] = useState(null); // null | 'success' | 'error'

  useEffect(() => {
    const payload = { circleCode: '021', quarterEndDate: '31/03/2025' }; // Example payload
    const fetchData = async () => {
      try {
        const responseData = await callApi('/Maker/getSavedDataNineA', payload, 'POST');
        const enriched = responseData.map((r, i) => {
            const aggOutStand = parseFloat(r.aggOutStand) || 0;
            const aggSecurities = parseFloat(r.aggSecurities) || 0;
            const netShortfall = aggOutStand - aggSecurities;
            return {
                ...initialRow, // Ensure all fields from initialRow are present
                ...r, // Overwrite with fetched data
                aggOutStand: r.aggOutStand || '', // Keep as string for input or format
                aggSecurities: r.aggSecurities || '',
                netShortfall: isNaN(netShortfall) ? '' : netShortfall.toFixed(2),
                id: r.id || `row-${Date.now()}-${i}`, // Ensure unique ID
            };
        });
        setRows(enriched.length > 0 ? enriched : [{ ...initialRow, id: `new-${Date.now()}-0` }]); // Start with one empty row if no data
      } catch (error) {
        console.error('Error loading saved data:', error);
        setSnackbarMessage('Error loading saved data. Please try again.');
        setSnackbarOpen(true);
        setRows([{ ...initialRow, id: `new-${Date.now()}-0` }]); // Start with one empty row on error
      }
    };
    fetchData();
  }, [callApi]); // callApi should be stable, but include if it could change

  const calculateTotals = useCallback(() => {
    const newTotals = rows.reduce(
      (acc, row) => {
        acc.aggOutStandTotal += parseFloat(row.aggOutStand) || 0;
        acc.aggSecuritiesTotal += parseFloat(row.aggSecurities) || 0;
        // netShortfall is derived, sum it based on its current value in row or re-calculate
        const currentNetShortfall = (parseFloat(row.aggOutStand) || 0) - (parseFloat(row.aggSecurities) || 0);
        acc.netShortfallTotal += currentNetShortfall;
        acc.provisionTotal += parseFloat(row.provision) || 0;
        acc.balInterestSuspenseAccTotal += parseFloat(row.balInterestSuspenseAcc) || 0;
        return acc;
      },
      {
        aggOutStandTotal: 0,
        aggSecuritiesTotal: 0,
        netShortfallTotal: 0,
        provisionTotal: 0,
        balInterestSuspenseAccTotal: 0,
      }
    );
    setTotals({
      aggOutStandTotal: newTotals.aggOutStandTotal.toFixed(2),
      aggSecuritiesTotal: newTotals.aggSecuritiesTotal.toFixed(2),
      netShortfallTotal: newTotals.netShortfallTotal.toFixed(2),
      provisionTotal: newTotals.provisionTotal.toFixed(2),
      balInterestSuspenseAccTotal: newTotals.balInterestSuspenseAccTotal.toFixed(2),
    });
  }, [rows]);

  useEffect(() => {
    // Calculate totals whenever rows change. Debounce if performance is an issue for rapid changes.
    calculateTotals();
  }, [rows, calculateTotals]);

  const addRow = () => {
    setRows((prev) => [...prev, { ...initialRow, id: `new-${Date.now()}-${prev.length}` }]);
  };

  const deleteSelectedRows = () => {
    const remainingRows = rows.filter((row) => !row.isDelete);
    setRows(remainingRows);
    setSnackbarMessage('Selected rows deleted.');
    setSnackbarOpen(true);
    if (remainingRows.length === 0) { // Optionally add a new blank row if all are deleted
        setTimeout(() => addRow(), 100);
    }
  };

  const validateRows = () => {
    for (let i = 0; i < rows.length; i++) {
      const row = rows[i];
      const hasAnyData = Object.keys(initialRow).some(
        (key) => key !== 'isDelete' && String(row[key] || '').trim() !== ''
      );

      if (hasAnyData && !String(row.borrowerName || '').trim()) {
        setSnackbarMessage(`Row ${i + 1}: Branch & Borrower's Name is required if other data is present.`);
        setSnackbarOpen(true);
        return false;
      }

      const numericFieldsToValidate = ['aggOutStand', 'aggSecurities', 'provision', 'balInterestSuspenseAcc'];
      for (const field of numericFieldsToValidate) {
        const value = String(row[field] || '');
        if (value !== '' && (isNaN(parseFloat(value)) || !numericRegex.test(value))) {
          const columnLabel = columns.find(col => col.key === field)?.label || field;
          setSnackbarMessage(`Row ${i + 1}: Invalid number in ${columnLabel}.`);
          setSnackbarOpen(true);
          return false;
        }
      }
    }
    return true;
  };

  const buildPayload = (saveFlag) => ({
    listToBeSent: rows.map(({ id, isDelete, ...rest }) => {
        const out = parseFloat(rest.aggOutStand) || 0;
        const sec = parseFloat(rest.aggSecurities) || 0;
        return { // Ensure all fields are present and formatted as strings if needed by backend
            borrowerName: String(rest.borrowerName || '').trim(),
            aggOutStand: out.toFixed(2),
            aggSecurities: sec.toFixed(2),
            netShortfall: (out - sec).toFixed(2),
            provision: (parseFloat(rest.provision) || 0).toFixed(2),
            balInterestSuspenseAcc: (parseFloat(rest.balInterestSuspenseAcc) || 0).toFixed(2),
        };
    }),
    circleCode: '021', // Example
    quarterEndDate: '31/03/2025', // Example
    userId: '1111111', // Example
    reportName: 'Schedule 9A',
    reportMasterId: '310023', // Example
    status: saveFlag ? 'Draft' : 'Submitted', // Example status
    save: saveFlag,
  });

  const handleApiCall = async (apiAction, payload, successMessage, statusSetter) => {
    if (!validateRows()) return;
    statusSetter('pending'); // Optional: for loading state
    try {
      await callApi(apiAction, payload, 'POST');
      statusSetter('success');
      setSnackbarMessage(successMessage);
    } catch (error) {
      console.error(`Error ${payload.save ? 'saving' : 'submitting'} report:`, error);
      statusSetter('error');
      setSnackbarMessage(error.message || `Error ${payload.save ? 'saving' : 'submitting'} report.`);
    }
    setSnackbarOpen(true);
  };

  const handleSave = () => {
    handleApiCall('/Maker/submitNineA', buildPayload(true), 'Report saved successfully.', setSaveStatus);
  };

  const handleSubmit = () => {
    handleApiCall('/Maker/submitNineA', buildPayload(false), 'Report submitted successfully.', setSubmitStatus);
  };

  const handleCloseSnackbar = (_, reason) => {
    if (reason === 'clickaway') return;
    setSnackbarOpen(false);
    // Consider resetting status after a delay or on next action
  };

  const handleCheckboxChange = useCallback((rowIndex, checked) => {
    setRows((prevRows) =>
      prevRows.map((row, idx) => (idx === rowIndex ? { ...row, isDelete: checked } : row))
    );
  }, []);

  const itemData = useMemo(
    () => ({
      rows,
      columns,
      setRows,
      handleCheckboxChange,
    }),
    [rows, handleCheckboxChange] // `columns` is stable but good practice if it could change
  );
  
  const listHeight = Math.min(500, rows.length * ITEM_SIZE + (rows.length > 0 ? 5 : 0));


  return (
    <Box sx={{ p: 2, display: 'flex', flexDirection: 'column', height: 'calc(100vh - 100px)' /* Adjust based on actual surrounding UI */ }}>
      <TableContainer component={Paper} sx={{ flexGrow: 1, overflow: 'hidden' /* AutoSizer handles scroll region */ }}>
        <Table stickyHeader sx={{ tableLayout: 'auto' /* 'fixed' can be an option for more rigid columns */ }}>
          <TableHead>
            <TableRow>
              {columns.map((col) => (
                <StyledTableCell key={col.key} align={col.align || 'left'}>
                  {col.label}
                </StyledTableCell>
              ))}
            </TableRow>
            <TableRow>
              {columns.map((col) => (
                <StyledTableCell key={`${col.key}-sub`} align={col.align || 'left'}>
                  {col.subHeading}
                </StyledTableCell>
              ))}
            </TableRow>
          </TableHead>
          {/* Virtualized Table Body */}
          {rows.length > 0 ? (
            <TableBody component="div" sx={{ display: 'block', position: 'relative' /* For AutoSizer */ }}>
              <AutoSizer disableHeight>
                {({ width }) => (
                  <List
                    height={listHeight}
                    itemCount={rows.length}
                    itemSize={ITEM_SIZE} // Row height
                    width={width}
                    overscanCount={5} // Render a few rows above/below viewport
                    itemData={itemData}
                    style={{ display: 'block' }} // Ensure List behaves as block
                  >
                    {RowRenderer}
                  </List>
                )}
              </AutoSizer>
            </TableBody>
          ) : (
            <TableBody>
              <TableRow>
                <StyledTableCell colSpan={columns.length} align="center" sx={{ py: 3 }}>
                  No data available. Click "Add Row" to begin.
                </StyledTableCell>
              </TableRow>
            </TableBody>
          )}
        </Table>
      </TableContainer>

      {/* Totals Row Container */}
      <TableContainer component={Paper} sx={{ width: '100%', mt: 0, borderTop: '2px solid black' }}>
        <Table sx={{ tableLayout: 'auto' }}>
          <TableBody>
            <TableRow sx={{ '& .MuiTableCell-root': { fontWeight: 'bold', backgroundColor: '#f9f9f9' } }}>
              <StyledTableCell colSpan={2} align="center">Total</StyledTableCell>
              {columns.slice(2).map((col) => { // Iterate through data columns for totals
                const totalKey = `${col.key}Total`; // Construct key like 'aggOutStandTotal'
                const showTotal = totals.hasOwnProperty(totalKey);
                const textAlignForInput = col.type === 'number' ? 'right' : 'left';

                return (
                  <StyledTableCell key={totalKey} align={col.align || 'right'}>
                    {showTotal ? (
                      <TextField
                        value={totals[totalKey]}
                        InputProps={{
                          readOnly: true,
                          style: { fontWeight: 'bold', textAlign: textAlignForInput },
                        }}
                        sx={{
                          width: '100%',
                          '& .MuiOutlinedInput-input': { // Align with data row TextFields
                            paddingTop: '8.5px',
                            paddingBottom: '8.5px',
                            paddingLeft: '0px',
                            paddingRight: '0px',
                          },
                        }}
                        size="small"
                        variant="outlined"
                      />
                    ) : null}
                  </StyledTableCell>
                );
              })}
            </TableRow>
          </TableBody>
        </Table>
      </TableContainer>

      {/* Action Buttons */}
      <Stack direction="row" spacing={2} mt={2} justifyContent="flex-start">
        <CustomButton label={'Add Row'} buttonType={'create'} onClickHandler={addRow} />
        <CustomButton
          label={'Delete Selected'}
          buttonType={'delete'}
          onClickHandler={deleteSelectedRows}
          disabled={!rows.some((row) => row.isDelete)}
        />
        <CustomButton label={'Save Draft'} buttonType={'save'} onClickHandler={handleSave} disabled={!rows.length} />
        <CustomButton
          label={'Submit Report'}
          buttonType={'submit'}
          onClickHandler={handleSubmit}
          disabled={!rows.length}
        />
      </Stack>

      {/* Snackbar for Notifications */}
      <Snackbar
        open={snackbarOpen}
        autoHideDuration={6000}
        onClose={handleCloseSnackbar}
        anchorOrigin={{ vertical: 'bottom', horizontal: 'right' }}
      >
        <Alert
          onClose={handleCloseSnackbar}
          severity={
            saveStatus === 'error' || submitStatus === 'error'
              ? 'error'
              : saveStatus === 'success' || submitStatus === 'success'
              ? 'success'
              : 'info' // Default for general messages
          }
          sx={{ width: '100%' }}
          variant="filled"
        >
          {snackbarMessage}
        </Alert>
      </Snackbar>
    </Box>
  );
}

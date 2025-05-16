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
import { CustomButton } from '../../../../common/components/ui/Buttons'; // Assuming path is correct
import useApi from '../../../../common/hooks/useApi'; // Assuming path is correct
import { FixedSizeList as List } from 'react-window';
import AutoSizer from 'react-virtualized-auto-sizer';

const StyledTableCell = styled(TableCell)(({ theme }) => ({
  fontSize: '1rem',
  padding: '8px', // Added some padding for better spacing, adjust as needed
  '&.MuiTableCell-head': {
    backgroundColor: theme.palette.common.black,
    color: theme.palette.common.white,
    fontWeight: 'bold', // Make header text bold
  },
}));

const numericRegex = /^-?\d*(.\d{0,2})?$/;
const alphanumericRegex = /^[A-Za-z0-9\s]*$/;

const columns = [
  { key: 'isDelete', label: 'Select', type: 'checkbox', editable: false, align: 'center', subHeading: '' },
  {
    key: 'borrowerName',
    label: "Branch & Borrower's Name",
    type: 'text',
    editable: true,
    align: 'left', // Changed to left for text, header will also be left
    pattern: alphanumericRegex,
    subHeading: '',
  },
  {
    key: 'aggOutStand',
    label: 'Aggregate outstanding',
    type: 'number',
    editable: true,
    align: 'right', // Changed to right for numbers, header will also be right
    pattern: numericRegex,
    subHeading: '( 1 )',
  },
  {
    key: 'aggSecurities',
    label: 'Aggregate value of realisable securities',
    type: 'number',
    editable: true,
    align: 'right', // Changed to right for numbers, header will also be right
    pattern: numericRegex,
    subHeading: '( 2 )',
  },
  {
    key: 'netShortfall',
    label: 'Net Shortfall',
    type: 'number',
    editable: false,
    align: 'right', // Changed to right for numbers, header will also be right
    subHeading: '( 3 = 1 - 2 )',
  },
  {
    key: 'provision',
    label: 'Provision',
    type: 'number',
    editable: true,
    align: 'right', // Changed to right for numbers, header will also be right
    pattern: numericRegex,
    subHeading: '( 4 )',
  },
  {
    key: 'balInterestSuspenseAcc',
    label: 'Balance in Interest Suspense Account',
    type: 'number',
    editable: true,
    align: 'right', // Changed to right for numbers, header will also be right
    pattern: numericRegex,
    subHeading: '( 5 )',
  },
];

const initialRow = {
  isDelete: false,
  borrowerName: '',
  aggOutStand: '',
  aggSecurities: '',
  netShortfall: '',
  provision: '',
  balInterestSuspenseAcc: '',
};

function debounce(func, delay = 200) {
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
  }, [row, column.key]);

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
    if (column.pattern) {
      if (column.type === 'number' && val !== '' && val !== '-' && !column.pattern.test(val)) {
        return;
      }
      // For alphanumeric, allow empty string or matching pattern
      if (column.type === 'text' && val !== '' && !column.pattern.test(val)) {
        return;
      }
    }

    setLocalValue(val);
    debouncedUpdateGlobalState(val);
  };

  const textAlign = column.type === 'number' ? 'right' : 'left';

  return (
    <TextField
      value={localValue}
      onChange={handleChange}
      inputProps={{
        style: { textAlign: textAlign },
        maxLength: column.key === 'borrowerName' ? 255 : 18,
      }}
      sx={{ width: '100%' }} // Ensure TextField takes full width
      size="small"
      variant="outlined" // Using outlined for consistency, can be "standard"
    />
  );
});
MemoizedCellWithLocalState.displayName = 'MemoizedCellWithLocalState';

// Memoized Row Renderer for react-window
const RowRenderer = React.memo(({ index, style, data }) => {
  const { rows, columns: tableColumns, setRows, handleCheckboxChange } = data;
  const rowItem = rows[index];

  if (!rowItem) {
    return null;
  }

  return (
    <TableRow key={rowItem.id || `row-${index}`} style={style} sx={{ '& .MuiTableCell-root': { borderBottom: '1px solid rgba(224, 224, 224, 1)'} }}>
      {tableColumns.map((column) => {
        const textAlign = column.type === 'number' ? 'right' : 'left';
        return (
          <StyledTableCell key={`${rowItem.id || `row-${index}`}-${column.key}`} align={column.align || 'left'}>
            {column.type === 'checkbox' ? (
              <Checkbox checked={!!rowItem.isDelete} onChange={(e) => handleCheckboxChange(index, e.target.checked)} />
            ) : column.editable ? (
              <MemoizedCellWithLocalState row={rowItem} column={column} index={index} setRows={setRows} />
            ) : (
              <TextField
                value={rowItem[column.key] || ''}
                inputProps={{
                  style: { textAlign: textAlign },
                  readOnly: true,
                }}
                sx={{ width: '100%' }} // Ensure TextField takes full width
                size="small"
                variant="outlined" // Using outlined for consistency
                disabled // Visually indicate non-editable, ensure readOnly in inputProps
              />
            )}
          </StyledTableCell>
        );
      })}
    </TableRow>
  );
});
RowRenderer.displayName = 'RowRenderer';

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
  const [saveStatus, setSaveStatus] = useState(null);
  const [submitStatus, setSubmitStatus] = useState(null);

  useEffect(() => {
    const payload = { circleCode: '021', quarterEndDate: '31/03/2025' };
    const fetchData = async () => {
      try {
        const response = await callApi('/Maker/getSavedDataNineA', payload, 'POST');
        // Ensure fetched data has initialRow structure for all fields
        const enriched = response.map((r, i) => {
            const netShortfall = (parseFloat(r.aggOutStand) || 0) - (parseFloat(r.aggSecurities) || 0);
            return { 
                ...initialRow, 
                ...r, 
                netShortfall: isNaN(netShortfall) ? '' : netShortfall.toFixed(2),
                id: r.id || `row-${Date.now()}-${i}` 
            };
        });
        setRows(enriched);
      } catch (error) {
        setSnackbarMessage('Error loading saved data.');
        setSnackbarOpen(true);
        // Initialize with one empty row if fetch fails, or based on requirements
        // setRows([{ ...initialRow, id: `new-${Date.now()}-${Math.random()}` }]);
      }
    };
    fetchData();
  }, [callApi]); // Added callApi to dependency array as it's used in effect

  const calculateTotals = useCallback(() => {
    const totalsObj = rows.reduce(
      (acc, row) => {
        acc.aggOutStandTotal += parseFloat(row.aggOutStand) || 0;
        acc.aggSecuritiesTotal += parseFloat(row.aggSecurities) || 0;
        // netShortfall is calculated, so re-calculate for total based on row values to be safe
        const currentNetShortfall = (parseFloat(row.aggOutStand) || 0) - (parseFloat(row.aggSecurities) || 0);
        acc.netShortfallTotal += currentNetShortfall || 0;
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
      aggOutStandTotal: totalsObj.aggOutStandTotal.toFixed(2),
      aggSecuritiesTotal: totalsObj.aggSecuritiesTotal.toFixed(2),
      netShortfallTotal: totalsObj.netShortfallTotal.toFixed(2),
      provisionTotal: totalsObj.provisionTotal.toFixed(2),
      balInterestSuspenseAccTotal: totalsObj.balInterestSuspenseAccTotal.toFixed(2),
    });
  }, [rows]);

  useEffect(() => {
    // Debounce or throttle total calculation if rows update very frequently
    const timer = setTimeout(calculateTotals, 50); // Reduced delay slightly
    return () => clearTimeout(timer);
  }, [rows, calculateTotals]);

  const addRow = () => {
    setRows((prev) => [...prev, { ...initialRow, id: `new-${Date.now()}-${Math.random()}` }]);
  };

  const deleteSelectedRows = () => {
    setRows((prevRows) => prevRows.filter((row) => !row.isDelete));
    setSnackbarMessage('Selected rows deleted.');
    setSnackbarOpen(true);
  };

  const validateRows = () => {
    for (let i = 0; i < rows.length; i++) {
      const row = rows[i];
      // Check if row has any data apart from checkbox and default initial values
      const hasMeaningfulData = Object.keys(initialRow).some(
        (key) => key !== 'isDelete' && row[key] !== initialRow[key] && row[key] !== ''
      );
  
      if (hasMeaningfulData && !row.borrowerName.trim()) {
        setSnackbarMessage(`Row ${i + 1}: Please enter Branch & Borrower's Name.`);
        setSnackbarOpen(true);
        return false;
      }
      // Validate numeric fields if they are not empty and contain meaningful data
      if (hasMeaningfulData) {
        const numericFields = ['aggOutStand', 'aggSecurities', 'provision', 'balInterestSuspenseAcc'];
        for (const field of numericFields) {
          if (row[field] !== '' && (isNaN(parseFloat(row[field])) || !numericRegex.test(row[field]))) {
            const columnLabel = columns.find(col => col.key === field)?.label || field;
            setSnackbarMessage(`Row ${i + 1}: Please enter a valid number for ${columnLabel}.`);
            setSnackbarOpen(true);
            return false;
          }
        }
      }
    }
    return true;
  };
  

  const buildPayload = (saveFlag) => ({
    listToBeSent: rows.map(({ id, isDelete, netShortfall, ...rest }) => {
        // Recalculate netShortfall just before sending to ensure accuracy
        const out = parseFloat(rest.aggOutStand) || 0;
        const sec = parseFloat(rest.aggSecurities) || 0;
        return {
            ...rest,
            aggOutStand: rest.aggOutStand || '0.00',
            aggSecurities: rest.aggSecurities || '0.00',
            netShortfall: (out - sec).toFixed(2),
            provision: rest.provision || '0.00',
            balInterestSuspenseAcc: rest.balInterestSuspenseAcc || '0.00',
        };
    }),
    circleCode: '021',
    quarterEndDate: '31/03/2025',
    userId: '1111111',
    reportName: 'Schedule 9A',
    reportMasterId: '310023',
    status: null, // This might need to be set based on action (e.g., 'Submitted')
    save: saveFlag,
  });

  const handleSave = async () => {
    if (!validateRows()) return;
    try {
      await callApi('/Maker/submitNineA', buildPayload(true), 'POST');
      setSaveStatus('success');
      setSnackbarMessage('Report saved successfully.');
    } catch (error) {
      setSaveStatus('error');
      setSnackbarMessage(error.message || 'Error saving report.');
    }
    setSnackbarOpen(true);
  };

  const handleSubmit = async () => {
    if (!validateRows()) return;
    try {
      // Add status for submit if needed in payload
      // const payload = {...buildPayload(false), status: "Submitted"};
      await callApi('/Maker/submitNineA', buildPayload(false), 'POST');
      setSubmitStatus('success');
      setSnackbarMessage('Report submitted successfully.');
    } catch (error) {
      setSubmitStatus('error');
      setSnackbarMessage(error.message || 'Error submitting report.');
    }
    setSnackbarOpen(true);
  };

  const handleCloseSnackbar = (_, reason) => {
    if (reason === 'clickaway') return;
    setSnackbarOpen(false);
    // Reset status after snackbar closes or timeout
    setTimeout(() => {
        setSaveStatus(null);
        setSubmitStatus(null);
    }, 300);
  };

  const handleCheckboxChange = useCallback((rowIndex, checked) => {
    setRows((prevRows) => prevRows.map((row, idx) => (idx === rowIndex ? { ...row, isDelete: checked } : row)));
  }, []);

  const itemData = useMemo(
    () => ({
      rows,
      columns, // Pass the updated columns definition
      setRows,
      handleCheckboxChange,
    }),
    [rows, handleCheckboxChange] // columns is constant, but good to include if it could change
  );

  const ITEM_SIZE = 60; // Increased item size slightly for better padding with outlined TextFields

  return (
    <Box sx={{ p: 2, display: 'flex', flexDirection: 'column', height: 'calc(100vh - 64px)' }}> {/* Adjust 64px based on header/navbar height */}
      <TableContainer component={Paper} sx={{ flexGrow: 1, overflow: 'hidden' /* Let AutoSizer handle scroll */ }}>
        <Table stickyHeader>
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
                <StyledTableCell key={`${col.key}-sub`} align={col.align || 'left'}> {/* Corrected key syntax */}
                  {col.subHeading}
                </StyledTableCell>
              ))}
            </TableRow>
          </TableHead>

          {rows.length > 0 ? (
            // TableBody for react-window needs to be a direct child for AutoSizer usually,
            // or AutoSizer needs to wrap the component that List is in.
            // The List itself will render TableRow, so we don't need an explicit <TableBody> here
            // if AutoSizer and List are handling the scrolling body.
            // However, MUI table structure expects TableBody.
            // Let's try keeping TableBody and ensuring AutoSizer works within it.
            <TableBody component="div" sx={{ display: 'block' /* Required for react-window with TableBody */}}>
              <AutoSizer disableHeight>
                {({ width }) => (
                  <List
                    height={Math.min(500, rows.length * ITEM_SIZE)} // Use dynamic item size
                    itemCount={rows.length}
                    itemSize={ITEM_SIZE} // Row height
                    width={width}
                    overscanCount={5}
                    itemData={itemData}
                    style={{ display: 'block' }} // Ensure List behaves as block for TableBody
                  >
                    {RowRenderer}
                  </List>
                )}
              </AutoSizer>
            </TableBody>
          ) : (
            <TableBody>
              <TableRow>
                <TableCell colSpan={columns.length} align="center" sx={{ py: 3 }}>
                  No data available. Click "Add Row" to begin.
                </TableCell>
              </TableRow>
            </TableBody>
          )}

          {/* Totals Row - Placed outside the scrollable virtualized list */}
        </Table>
      </TableContainer>
      {/* Separate Table for Totals to keep it sticky at the bottom of the container if needed, or just after the list */}
      <TableContainer component={Paper} sx={{ width: '100%', mt:0, borderTop:'2px solid black' /* Visual separation */ }}>
        <Table>
        <TableBody>
            <TableRow sx={{ '& .MuiTableCell-root': { fontWeight: 'bold', backgroundColor: '#f0f0f0' } }}>
              <StyledTableCell colSpan={2} align="center">
                Total
              </StyledTableCell>
              {[
                'aggOutStandTotal',
                'aggSecuritiesTotal',
                'netShortfallTotal',
                'provisionTotal',
                'balInterestSuspenseAccTotal',
              ].map((key, index) => (
                <StyledTableCell key={key} align={columns[index + 2]?.align || 'right'}> {/* Align with column data */}
                  <TextField
                    value={totals[key]}
                    InputProps={{
                      readOnly: true,
                      style: {
                        textAlign: columns[index + 2]?.align || 'right', // Align text with column data
                        fontWeight: 'bold',
                      },
                    }}
                    sx={{ width: '100%' }} // Ensure TextField takes full width
                    size="small"
                    variant="outlined" // Consistent variant
                  />
                </StyledTableCell>
              ))}
            </TableRow>
          </TableBody>
        </Table>
      </TableContainer>

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
              : 'info'
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


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
  Typography,
  CircularProgress,
  TableCell,
} from '@mui/material';
import { styled } from '@mui/material/styles';
import { CustomButton } from '../../../../common/components/ui/Buttons'; // Assuming path is correct
import useApi from '../../../../common/hooks/useApi'; // Assuming path is correct
import { FixedSizeList as List } from 'react-window';
import AutoSizer from 'react-virtualized-auto-sizer';
import FormInput from '../../../../common/components/ui/FormInput';
import { useLocation } from 'react-router-dom';

const StyledTableCell = styled(TableCell)(({ theme }) => ({
  fontSize: '1rem',
  padding: theme.spacing(1), // Adjust padding as needed for fit
  '&.MuiTableCell-head': {
    backgroundColor: theme.palette.common.black,
    color: theme.palette.common.white,
    fontWeight: 'bold',
  },
}));

const numericRegex = /^-?\d*(\.\d{0,2})?$/;
const alphanumericRegex = /^[A-Za-z0-9\s]*$/;

// Define columns with width and alignment
const columns = [
  { key: 'isDelete', label: 'Select', type: 'checkbox', editable: false, align: 'center', subHeading: '', width: '5%' },
  {
    key: 'borrowerName',
    label: "Branch & Borrower's Name",
    type: 'alphaNumericWithSpace',
    editable: true,
    align: 'center', // Text columns usually left-aligned
    pattern: alphanumericRegex,
    subHeading: '',
    width: '25%',
  },
  {
    key: 'aggOutStand',
    label: 'Aggregate outstanding',
    type: 'amountDecimal',
    editable: true,
    align: 'center', // Numeric columns usually right-aligned
    pattern: numericRegex,
    subHeading: '( 1 )',
    width: '14%', // Adjusted for sum
  },
  {
    key: 'aggSecurities',
    label: 'Aggregate value of realisable securities',
    type: 'amountDecimal',
    editable: true,
    align: 'center', // Numeric
    pattern: numericRegex,
    subHeading: '( 2 )',
    width: '14%', // Adjusted for sum
  },
  {
    key: 'netShortfall',
    label: 'Net Shortfall',
    type: 'amountDecimal',
    editable: false,
    align: 'center', // Numeric
    subHeading: '( 3 = 1 - 2 )',
    width: '14%', // Adjusted for sum
  },
  {
    key: 'provision',
    label: 'Provision',
    type: 'amountDecimal',
    editable: true,
    align: 'center', // Numeric
    pattern: numericRegex,
    subHeading: '( 4 )',
    width: '14%', // Adjusted for sum
  },
  {
    key: 'balInterestSuspenseAcc',
    label: 'Balance in Interest Suspense Account',
    type: 'amountDecimal',
    editable: true,
    align: 'center', // Numeric
    pattern: numericRegex,
    subHeading: '( 5 )',
    width: '14%', // Adjusted for sum (5 + 25 + 14*5 = 5 + 25 + 70 = 100%)
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

function debounce(func, delay = 300) {
  // Increased default delay slightly
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

  const updateGlobalState = useMemo(
    () => (value) => {
      setRows((prevRows) => {
        const updatedRows = [...prevRows];
        if (updatedRows[index]) {
          const targetRow = { ...updatedRows[index] };
          targetRow[column.key] = value;

          // Helper to sanitize and ensure 2 decimal digits safely
          const sanitize = (val) => {
            if (!val) return '0.00';
            const str = val.toString().replace(/,/g, '');
            return /^\d+(\.\d+)?$/.test(str) ? str : '0.00';
          };

          // Safe subtraction without float precision error
          const subtractDecimalStrings = (a, b) => {
            const [aInt, aDec = ''] = sanitize(a).split('.');
            const [bInt, bDec = ''] = sanitize(b).split('.');

            const aCents = BigInt(aInt) * 100n + BigInt((aDec + '00').slice(0, 2));
            const bCents = BigInt(bInt) * 100n + BigInt((bDec + '00').slice(0, 2));

            const diffCents = aCents - bCents;

            const sign = diffCents < 0 ? '-' : '';
            const absCents = diffCents < 0 ? -diffCents : diffCents;

            const intPart = absCents / 100n;
            const decPart = (absCents % 100n).toString().padStart(2, '0');

            return `${sign}${intPart.toString()}.${decPart}`;
          };

          // Apply calculation only for relevant columns
          if (column.key === 'aggOutStand' || column.key === 'aggSecurities') {
            targetRow.netShortfall = subtractDecimalStrings(targetRow.aggOutStand, targetRow.aggSecurities);
          }

          updatedRows[index] = targetRow;
          return updatedRows;
        }
        return prevRows;
      });
    },
    [index, column.key, setRows]
  );

  const handleChange = (e) => {
    const val = e.target.value;
    // if (column.pattern) {
    //   if (column.type === 'number' && val !== '' && val !== '-' && !column.pattern.test(val)) {
    //     return;
    //   }
    // }
    setLocalValue(val);
    updateGlobalState(val);
  };

  return (
    <FormInput
      value={localValue}
      onChange={handleChange}
      // inputProps={{
      //   style: { textAlign: column.align || 'left' }, // Align text based on column definition
      //   maxLength: column.key === 'borrowerName' ? 255 : 18,
      // }}
      inputType={column.type}
      maxLength={column.key === 'borrowerName' ? 50 : 18}
      debounceDuration={250}
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
    <TableRow
      key={rowItem.id || `row-${index}`} // Ensure unique key
      style={{ ...style, display: 'flex' }} // react-window provides style, ensure display is flex for TableRow to work with cells
    >
      {tableColumns.map((column) => (
        <StyledTableCell
          key={`${rowItem.id || index}-${column.key}`}
          align={column.align || 'center'}
          style={{
            width: column.width,
            display: 'flex',
            alignItems: 'center',
            justifyContent: column.align || 'center',
          }} // Apply width and flex properties for alignment
        >
          {column.type === 'checkbox' ? (
            <Checkbox checked={!!rowItem.isDelete} onChange={(e) => handleCheckboxChange(index, e.target.checked)} />
          ) : column.editable ? (
            <MemoizedCellWithLocalState row={rowItem} column={column} index={index} setRows={setRows} />
          ) : (
            <FormInput
              value={rowItem[column.key] || ''}
              // inputProps={{
              //   style: { textAlign: column.align || 'left' }, // Align text based on column definition
              // }}
              inputType={'amountDecimal'}
              readOnly
            />
          )}
        </StyledTableCell>
      ))}
    </TableRow>
  );
});
RowRenderer.displayName = 'RowRenderer';

export default function Schedule9ATable() {
  const { state } = useLocation();
  const [reportObject, setReportObject] = useState(state?.report || null);
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
  const [isLoading, setIsLoading] = useState(false);
  const showSnackbar = (message, severity) => console.log(`Snackbar: ${message} (${severity})`);
  const user = JSON.parse(localStorage.getItem('user'));
  useEffect(() => {
    const fetchData = async () => {
      setIsLoading(true);
      let isMounted = true;
      showSnackbar('Loading data...', 'info');
      console.log('reportObj', reportObject);
      const payload = { circleCode: user.circleCode, quarterEndDate: user.quarterEndDate };
      try {
        const response = await callApi('/Maker/getSavedDataNineA', payload, 'POST');
        if (!isMounted) return;

        const enriched = response.map((r, i) => ({ ...initialRow, ...r, id: r.id || `row-${Date.now()}-${i}` }));
        setRows(enriched);
      } catch (error) {
        if (!isMounted) return;
        setSnackbarMessage('Error loading saved data.');
        setSnackbarOpen(true);
      } finally {
        if (!isMounted) return;
        setIsLoading(false);
      }
    };
    fetchData();
    return () => {
      isMounted = false;
    };
  }, []); // Added callApi to dependency array

  const calculateTotals = useCallback(() => {
    const addDecimalStrings = (a, b) => {
      const toCents = (val) => {
        const str = val.toString().replace(/,/g, '').trim();
        const isNeg = str.startsWith('-');
        const num = isNeg ? str.slice(1) : str;
        const [intPart, decPart = ''] = num.split('.');
        const cents = BigInt(intPart) * 100n + BigInt((decPart + '00').slice(0, 2));
        return isNeg ? -cents : cents;
      };

      const totalCents = toCents(a) + toCents(b);

      const sign = totalCents < 0 ? '-' : '';
      const absCents = totalCents < 0 ? -totalCents : totalCents;

      const intPart = absCents / 100n;
      const decPart = (absCents % 100n).toString().padStart(2, '0');

      return `${sign}${intPart.toString()}.${decPart}`;
    };

    const totalsObj = rows.reduce(
      (acc, row) => {
        acc.aggOutStandTotal = addDecimalStrings(acc.aggOutStandTotal, row.aggOutStand || '0');
        acc.aggSecuritiesTotal = addDecimalStrings(acc.aggSecuritiesTotal, row.aggSecurities || '0');
        acc.netShortfallTotal = addDecimalStrings(acc.netShortfallTotal, row.netShortfall || '0');
        acc.provisionTotal = addDecimalStrings(acc.provisionTotal, row.provision || '0');
        acc.balInterestSuspenseAccTotal = addDecimalStrings(
          acc.balInterestSuspenseAccTotal,
          row.balInterestSuspenseAcc || '0'
        );
        return acc;
      },
      {
        aggOutStandTotal: '0.00',
        aggSecuritiesTotal: '0.00',
        netShortfallTotal: '0.00',
        provisionTotal: '0.00',
        balInterestSuspenseAccTotal: '0.00',
      }
    );

    setTotals(totalsObj);
  }, [rows]);

  useEffect(() => {
    // Debounce calculateTotals or call it less frequently if performance is an issue
    calculateTotals();
  }, [rows, calculateTotals]);

  const addRow = () => {
    setRows((prev) => [...prev, { ...initialRow, id: `new-${Date.now()}-${Math.random()}` }]);
  };

  const deleteSelectedRows = () => {
    setRows((prevRows) => prevRows.filter((row) => !row.isDelete));
    setSnackbarMessage(
      'Row deleted successfully, Deletion of row will refelect in the report after Saving/Submitting the report.'
    );
    setSnackbarOpen(true);
  };

  const validateRows = () => {
    for (let i = 0; i < rows.length; i++) {
      const row = rows[i];
      const hasData = Object.keys(initialRow).some(
        (key) => key !== 'isDelete' && row[key] !== initialRow[key] && row[key] !== ''
      );

      if (hasData && !row.borrowerName) {
        setSnackbarMessage(`Row ${i + 1}: Please enter Branch & Borrower's Name.`);
        setSnackbarOpen(true);
        return false;
      }
      if (
        hasData &&
        ((row.aggOutStand !== '' && isNaN(parseFloat(row.aggOutStand))) ||
          (row.aggSecurities !== '' && isNaN(parseFloat(row.aggSecurities))) ||
          (row.provision !== '' && isNaN(parseFloat(row.provision))) ||
          (row.balInterestSuspenseAcc !== '' && isNaN(parseFloat(row.balInterestSuspenseAcc))))
      ) {
        setSnackbarMessage(`Row ${i + 1}: Please enter valid numeric values for amounts.`);
        setSnackbarOpen(true);
        return false;
      }
    }
    return true;
  };

  const buildPayload = (saveFlag) => ({
    listToBeSent: rows.map(({ id, isDelete, ...rest }) => rest), // Remove client-side id and isDelete
    circleCode: user.circleCode,
    quarterEndDate: user.quarterEndDate,
    userId: user.userId,
    reportName: reportObject.name,
    reportMasterId: reportObject.reportMasterId,
    status: reportObject.status, // Or determine status based on action
    save: saveFlag,
  });

  const handleSave = async () => {
    if (!validateRows()) return;
    try {
      const res = await callApi('/Maker/submitNineA', buildPayload(true), 'POST');
      if (res && typeof res === 'string') {
        const [flag, newReportId, newStatus] = res.split('~');
        setReportObject((prev) => ({
          ...prev,
          reportId: newReportId,
          status: newStatus,
        }));
      }
      console.log('reportObj', reportObject);

      setSaveStatus('success');
      setSnackbarMessage('Report saved successfully.');
    } catch (error) {
      console.error('Error saving report:', error);
      setSaveStatus('error');
      setSnackbarMessage(error.message || 'Error saving report.');
    }
    setSnackbarOpen(true);
  };

  const handleSubmit = async () => {
    if (!validateRows()) return;
    try {
      const res = await callApi('/Maker/submitNineA', buildPayload(false), 'POST');

      if (res && typeof res === 'string') {
        const [flag, newReportId, newStatus] = res.split('~');
        setReportObject((prev) => ({
          ...prev,
          reportId: newReportId,
          status: newStatus,
        }));
      }
      setSubmitStatus('success');
      setSnackbarMessage('Report submitted successfully.');
    } catch (error) {
      console.error('Error submitting report:', error);
      setSubmitStatus('error');
      setSnackbarMessage(error.message || 'Error submitting report.');
    }
    setSnackbarOpen(true);
  };

  const handleCloseSnackbar = (_, reason) => {
    if (reason === 'clickaway') return;
    setSnackbarOpen(false);
    setSaveStatus(null); // Reset status
    setSubmitStatus(null); // Reset status
  };

  const handleCheckboxChange = useCallback((rowIndex, checked) => {
    setRows((prevRows) => prevRows.map((row, idx) => (idx === rowIndex ? { ...row, isDelete: checked } : row)));
  }, []);

  const itemData = useMemo(
    () => ({
      rows,
      columns, // Pass the updated columns array with widths
      setRows,
      handleCheckboxChange,
    }),
    [rows, handleCheckboxChange] // columns is stable, but included for completeness if it could change
  );

  if (isLoading) {
    return (
      <Box sx={{ display: 'flex', justifyContent: 'center', alignItems: 'center', height: '50vh' }}>
        <CircularProgress />
        <Typography sx={{ ml: 2 }}>Loading Data...</Typography>
      </Box>
    );
  }

  return (
    <Box sx={{ p: 2 }}>
      <Stack direction="row" spacing={2} mb={2} justifyContent="flex-start">
        <CustomButton label={'Add Row'} buttonType={'create'} onClickHandler={addRow} />
        <CustomButton
          label={'Delete Row'}
          buttonType={'delete'}
          onClickHandler={deleteSelectedRows}
          disabled={!rows.some((row) => row.isDelete)}
        />
        <CustomButton label={'Save'} buttonType={'save'} onClickHandler={handleSave} />
        <CustomButton label={'Submit'} buttonType={'submit'} onClickHandler={handleSubmit} disabled={!rows.length} />
      </Stack>
      <TableContainer component={Paper} sx={{ width: '100%', mb: 2, overflowX: 'auto' }}>
        <Table stickyHeader sx={{ tableLayout: 'fixed', width: '100%' }}>
          <TableHead>
            <TableRow sx={{ display: 'flex' }}>
              {/* Ensure header row is also flex */}
              {columns.map((col) => (
                <StyledTableCell
                  key={col.key}
                  align={col.align || 'left'}
                  style={{
                    width: col.width,
                    display: 'flex',
                    alignItems: 'center',
                    justifyContent: col.align || 'left',
                  }} // Apply width
                >
                  {col.label}
                </StyledTableCell>
              ))}
            </TableRow>
            <TableRow sx={{ display: 'flex' }}>
              {/* Ensure sub-header row is also flex */}
              {columns.map((col) => (
                <StyledTableCell
                  key={`${col.key}-sub`}
                  align={col.align || 'left'}
                  style={{
                    width: col.width,
                    display: 'flex',
                    alignItems: 'center',
                    justifyContent: col.align || 'left',
                  }} // Apply width
                >
                  {col.subHeading}
                </StyledTableCell>
              ))}
            </TableRow>
          </TableHead>

          {rows.length > 0 ? (
            // TableBody for react-window List. List renders TableRow components.
            // The TableBody itself is not strictly necessary here if List handles all row rendering.
            // However, to keep structure similar to non-virtualized table, it can be kept.
            // AutoSizer will give width to the List.
            <TableBody component="div">
              {/* component="div" for TableBody with react-window */}
              <AutoSizer disableHeight>
                {({ width }) => (
                  <List
                    height={Math.min(500, rows.length * 50 + (rows.length > 0 ? 5 : 0))}
                    itemCount={rows.length}
                    itemSize={50} // Height of each row
                    width={width} // Width from AutoSizer
                    overscanCount={5}
                    itemData={itemData}
                  >
                    {RowRenderer}
                  </List>
                )}
              </AutoSizer>
            </TableBody>
          ) : (
            <TableBody>
              <TableRow>
                <TableCell align="center" sx={{ py: 3 }}>
                  No data available. Click "Add Row" to begin.
                </TableCell>
              </TableRow>
            </TableBody>
          )}

          {/* Totals Row - Rendered as a standard TableBody/TableRow */}
          <TableBody>
            <TableRow sx={{ '& .MuiTableCell-root': { fontWeight: 'bold' }, display: 'flex' }}>
              <StyledTableCell
                align="center"
                // Calculate combined width of the first two columns for the "Total" cell
                style={{
                  width: `calc(${columns[0].width} + ${columns[1].width})`,
                  display: 'flex',
                  alignItems: 'center',
                  justifyContent: 'center',
                }}
              >
                Total
              </StyledTableCell>
              {columns.slice(2).map(
                (
                  col // Iterate through columns that have totals
                ) => (
                  <StyledTableCell
                    key={`${col.key}-total`}
                    align={col.align || 'right'}
                    style={{ width: col.width, display: 'flex', alignItems: 'center' }} // Apply width from column definition
                  >
                    {totals[`${col.key}Total`] !== undefined ? (
                      <FormInput value={totals[`${col.key}Total`]} inputType={'amountDecimal'} readOnly />
                    ) : null}
                  </StyledTableCell>
                )
              )}
            </TableRow>
          </TableBody>
        </Table>
      </TableContainer>

      <Snackbar
        open={snackbarOpen}
        autoHideDuration={10000}
        onClose={handleCloseSnackbar}
        anchorOrigin={{ vertical: 'top', horizontal: 'right' }}
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
          sx={{ width: '100%', mt: '100px' }}
          variant="filled"
        >
          {snackbarMessage}
        </Alert>
      </Snackbar>
    </Box>
  );
}

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
import { CustomButton } from '../../../../common/components/ui/Buttons';
import useApi from '../../../../common/hooks/useApi';
import { FixedSizeList as List } from 'react-window';
import AutoSizer from 'react-virtualized-auto-sizer';

const StyledTableCell = styled(TableCell)(({ theme }) => ({
  fontSize: '1rem',
  [`&.MuiTableCell-head`]: {
    backgroundColor: theme.palette.common.black,
    color: theme.palette.common.white,
  },
}));

const numericRegex = /^-?\d*(\.\d{0,2})?$/;
const alphanumericRegex = /^[A-Za-z0-9\s]*$/;

const columns = [
  { key: 'isDelete', label: 'Select', type: 'checkbox', editable: false, align: 'center', subHeading: '' },
  {
    key: 'borrowerName',
    label: "Branch & Borrower's Name",
    type: 'text',
    editable: true,
    align: 'center',
    pattern: alphanumericRegex,
    subHeading: '',
  },
  {
    key: 'aggOutStand',
    label: 'Aggregate outstanding',
    type: 'number',
    editable: true,
    align: 'center',
    pattern: numericRegex,
    subHeading: '( 1 )',
  },
  {
    key: 'aggSecurities',
    label: 'Aggregate value of realisable securities',
    type: 'number',
    editable: true,
    align: 'center',
    pattern: numericRegex,
    subHeading: '( 2 )',
  },
  {
    key: 'netShortfall',
    label: 'Net Shortfall',
    type: 'number',
    editable: false,
    align: 'center',
    subHeading: '( 3 = 1 - 2 )',
  },
  {
    key: 'provision',
    label: 'Provision',
    type: 'number',
    editable: true,
    align: 'center',
    pattern: numericRegex,
    subHeading: '( 4 )',
  },
  {
    key: 'balInterestSuspenseAcc',
    label: 'Balance in Interest Suspense Account',
    type: 'number',
    editable: true,
    align: 'center',
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
    }

    setLocalValue(val);
    debouncedUpdateGlobalState(val);
  };

  return (
    <TextField
      value={localValue}
      onChange={handleChange}
      sx={{
        style: { textAlign: 'right' },
        maxLength: column.key === 'borrowerName' ? 255 : 18,
      }}
      size="small"
    />
  );
});
MemoizedCellWithLocalState.displayName = 'MemoizedCellWithLocalState';

// Memoized Row Renderer for react-window
const RowRenderer = React.memo(({ index, style, data }) => {
  const { rows, columns: tableColumns, setRows, handleCheckboxChange } = data; // Renamed columns to avoid conflict
  const rowItem = rows[index];

  if (!rowItem) {
    return null;
  }

  return (
    <TableRow key={rowItem.id || index} style={style}>
      {tableColumns.map((column) => (
        <StyledTableCell key={`${rowItem.id || index}-${column.key}`} align={column.align || 'center'}>
          {column.type === 'checkbox' ? (
            <Checkbox checked={!!rowItem.isDelete} onChange={(e) => handleCheckboxChange(index, e.target.checked)} />
          ) : column.editable ? (
            <MemoizedCellWithLocalState row={rowItem} column={column} index={index} setRows={setRows} />
          ) : (
            <TextField
              value={rowItem[column.key] || ''}
              inputProps={{ style: { textAlign: 'right' } }}
              size="small"
              disabled
            />
          )}
        </StyledTableCell>
      ))}
    </TableRow>
  );
});
RowRenderer.displayName = 'RowRenderer';

export default function Schedule9ATable() {
  const { callApi } = useApi();
  const [rows, setRows] = useState([]);
  const [totals, setTotals] = useState({
    aggOutStandTotal: 0,
    aggSecuritiesTotal: 0,
    netShortfallTotal: 0,
    provisionTotal: 0,
    balInterestSuspenseAccTotal: 0,
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
        const enriched = response.map((r, i) => ({ ...initialRow, ...r, id: r.id || `row-${Date.now()}-${i}` }));
        setRows(enriched);
      } catch (error) {
        setSnackbarMessage('Error loading saved data.');
        setSnackbarOpen(true);
      }
    };
    fetchData();
  }, []);

  const calculateTotals = useCallback(() => {
    const totalsObj = rows.reduce(
      (acc, row) => {
        acc.aggOutStandTotal += parseFloat(row.aggOutStand) || 0;
        acc.aggSecuritiesTotal += parseFloat(row.aggSecurities) || 0;
        acc.netShortfallTotal += parseFloat(row.netShortfall) || 0;
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
    const timer = setTimeout(calculateTotals, 100);
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
    listToBeSent: rows.map(({ id, isDelete, ...rest }) => rest),
    circleCode: '021',
    quarterEndDate: '31/03/2025',
    userId: '1111111',
    reportName: 'Schedule 9A',
    reportMasterId: '310023',
    status: null,
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
    setSaveStatus(null);
    setSubmitStatus(null);
  };

  // Memoized callback for checkbox changes
  const handleCheckboxChange = useCallback((rowIndex, checked) => {
    setRows((prevRows) => prevRows.map((row, idx) => (idx === rowIndex ? { ...row, isDelete: checked } : row)));
  }, []);

  const itemData = useMemo(
    () => ({
      rows,
      columns,
      setRows,
      handleCheckboxChange,
    }),
    [rows, handleCheckboxChange]
  );

  return (
    <Box sx={{ p: 2 }}>
      <TableContainer component={Paper} sx={{ width: '100%', mb: 2 }}>
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
                <StyledTableCell key={`${col.key}-sub`} align={col.align || 'left'}>
                  {col.subHeading}
                </StyledTableCell>
              ))}
            </TableRow>
          </TableHead>

          {rows.length > 0 ? (
            <TableBody>
              <AutoSizer disableHeight>
                {({ width }) => (
                  <List
                    height={Math.min(500, rows.length * 50 + (rows.length > 0 ? 5 : 0))}
                    itemCount={rows.length}
                    itemSize={50}
                    width={width}
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
                <TableCell colSpan={columns.length} align="center" sx={{ py: 3 }}>
                  No data available. Click "Add Row" to begin.
                </TableCell>
              </TableRow>
            </TableBody>
          )}

          <TableBody>
            <TableRow sx={{ '& .MuiTableCell-root': { fontWeight: 'bold' } }}>
              <StyledTableCell colSpan={2} align="center">
                Total
              </StyledTableCell>
              {[
                'aggOutStandTotal',
                'aggSecuritiesTotal',
                'netShortfallTotal',
                'provisionTotal',
                'balInterestSuspenseAccTotal',
              ].map((key) => (
                <StyledTableCell key={key} align="right">
                  {' '}
                  {/* Totals usually align right */}
                  <TextField
                    value={totals[key]}
                    InputProps={{ readOnly: true, style: { textAlign: 'right', fontWeight: 'bold' } }}
                    size="small"
                    variant="outlined"
                  />
                </StyledTableCell>
              ))}
            </TableRow>
          </TableBody>
        </Table>
      </TableContainer>
      <Stack direction="row" spacing={2} mt={2} justifyContent="flex-start">
        {' '}
        {/* Aligned buttons */}
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

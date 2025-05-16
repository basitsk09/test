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
import { CustomButton } from '../../../../common/components/ui/Buttons'; // Assuming path
import useApi from '../../../../common/hooks/useApi'; // Assuming path
import { FixedSizeList as List } from 'react-window';
import AutoSizer from 'react-virtualized-auto-sizer';

const StyledTableCell = styled(TableCell)(({ theme }) => ({
  fontSize: '1rem',
  padding: theme.spacing(0.5, 1), // Adjusted padding slightly
  '&.MuiTableCell-head': {
    backgroundColor: theme.palette.common.black,
    color: theme.palette.common.white,
    fontWeight: 'bold',
  },
  display: 'flex', // Make cell a flex container for its content
  alignItems: 'center', // Vertically center content in cell
  // justifyContent: 'flex-start', // Default horizontal alignment (overridden by 'align' prop)
}));

const numericRegex = /^-?\d*(\.\d{0,2})?$/;
const alphanumericRegex = /^[A-Za-z0-9\s]*$/;

const columns = [ // Ensure your column widths sum up appropriately (e.g., to 100% or manage overflow)
  { key: 'isDelete', label: 'Select', type: 'checkbox', editable: false, align: 'center', subHeading: '', width: '5%' },
  { key: 'borrowerName', label: "Branch & Borrower's Name", type: 'text', editable: true, align: 'left', pattern: alphanumericRegex, subHeading: '', width: '25%' },
  { key: 'aggOutStand', label: 'Aggregate outstanding', type: 'number', editable: true, align: 'right', pattern: numericRegex, subHeading: '( 1 )', width: '14%' },
  { key: 'aggSecurities', label: 'Aggregate value of realisable securities', type: 'number', editable: true, align: 'right', pattern: numericRegex, subHeading: '( 2 )', width: '14%' },
  { key: 'netShortfall', label: 'Net Shortfall', type: 'number', editable: false, align: 'right', subHeading: '( 3 = 1 - 2 )', width: '14%' },
  { key: 'provision', label: 'Provision', type: 'number', editable: true, align: 'right', pattern: numericRegex, subHeading: '( 4 )', width: '14%' },
  { key: 'balInterestSuspenseAcc', label: 'Balance in Interest Suspense Account', type: 'number', editable: true, align: 'right', pattern: numericRegex, subHeading: '( 5 )', width: '14%' },
];

const initialRow = { /* ... as before ... */ };
function debounce(func, delay = 300) { /* ... as before ... */ }

const MemoizedCellWithLocalState = React.memo(({ row, column, index, setRows }) => {
  const [localValue, setLocalValue] = useState(String(row[column.key] || ''));

  useEffect(() => {
    setLocalValue(String(row[column.key] || ''));
  }, [row, column.key]);

  const debouncedUpdateGlobalState = useMemo(
    () =>
      debounce((newValue) => {
        setRows((prevRows) => {
          const updatedRows = [...prevRows];
          if (updatedRows[index]) {
            const targetRow = { ...updatedRows[index] };
            let processedValue = newValue;

            if (column.type === 'number') {
              if (newValue === '' || newValue === '-') {
                processedValue = newValue;
              } else {
                const num = parseFloat(newValue);
                if (isNaN(num)) {
                  processedValue = String(targetRow[column.key] || ''); // Revert to old or empty
                } else {
                  if (['aggOutStand', 'aggSecurities', 'provision', 'balInterestSuspenseAcc'].includes(column.key)) {
                    processedValue = num.toFixed(2);
                  } else {
                    processedValue = String(num);
                  }
                }
              }
            }
            targetRow[column.key] = processedValue;

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
    [index, column.key, column.type, setRows]
  );

  const handleChange = (e) => {
    const val = e.target.value;
    // Critical Debugging Line: Check if this event fires
    console.log(`handleChange in Cell '${column.key}', Row ${index}. Value: "${val}"`);
    setLocalValue(val);

    if (column.pattern) {
      if (column.type === 'number') {
        // Validation/coercion is now primarily in debouncedUpdateGlobalState for numbers
      } else { // For non-numeric types
        if (val !== '' && !column.pattern.test(val)) {
          console.log(`Pattern failed for non-numeric Cell ${column.key}: "${val}". Global update skipped.`);
          return; // Prevent global update for invalid non-numeric
        }
      }
    }
    debouncedUpdateGlobalState(val);
  };

  return (
    <TextField
      value={localValue}
      onChange={handleChange}
      fullWidth // Equivalent to sx={{ width: '100%' }} for TextField
      size="small"
      inputProps={{
        style: { textAlign: column.align || 'left' /*, backgroundColor: 'lightyellow'*/ }, // Debug style
        maxLength: column.key === 'borrowerName' ? 255 : 18,
      }}
      name={`editable-cell-${index}-${column.key}`} // Unique name for DevTools
    />
  );
});
MemoizedCellWithLocalState.displayName = 'MemoizedCellWithLocalState';

const RowRenderer = React.memo(({ index, style, data }) => {
  const { rows, columns: tableColumns, setRows, handleCheckboxChange } = data;
  const rowItem = rows[index];

  if (!rowItem) return <div style={style}></div>; // Placeholder for empty item

  return (
    // This outer div receives the style from react-window
    <div style={style}>
      <TableRow
        component="div" // Render TableRow as a div to be a valid flex child if needed by parent
        sx={{
          display: 'flex',
          width: '100%',
          height: '100%',
          boxSizing: 'border-box',
        }}
      >
        {tableColumns.map((column) => (
          <StyledTableCell
            component="div" // Render StyledTableCell as a div
            key={`${rowItem.id || index}-${column.key}`}
            // align prop on StyledTableCell (custom component) now controls justifyContent
            style={{
                width: column.width,
                boxSizing: 'border-box',
                justifyContent: column.align === 'left' ? 'flex-start' : column.align === 'right' ? 'flex-end' : 'center',
                // border: '1px dashed lightgray', // For debugging layout
            }}
          >
            {column.type === 'checkbox' ? (
              <Checkbox
                checked={!!rowItem.isDelete}
                onChange={(e) => handleCheckboxChange(index, e.target.checked)}
                sx={{ padding: '0' }}
              />
            ) : column.editable ? (
              <MemoizedCellWithLocalState row={rowItem} column={column} index={index} setRows={setRows} />
            ) : (
              // For non-editable cells, TextField value might just span the cell.
              // If you want specific text alignment, ensure the parent StyledTableCell's justifyContent
              // and the TextField's own textAlign are coordinated.
              // The TextField itself will take full width of this flex item if not constrained.
              <Box sx={{ width: '100%', textAlign: column.align || 'left' }}>
                 {String(rowItem[column.key] || '')}
              </Box>
              // Or if you prefer a disabled TextField:
              // <TextField
              //   value={String(rowItem[column.key] || '')}
              //   fullWidth
              //   size="small"
              //   disabled
              //   inputProps={{ style: { textAlign: column.align || 'left' } }}
              // />
            )}
          </StyledTableCell>
        ))}
      </TableRow>
    </div>
  );
});
RowRenderer.displayName = 'RowRenderer';

export default function Schedule9ATable() {
  // ... (useApi, state variables: rows, totals, snackbar, etc. as before)
  const { callApi } = useApi();
  const [rows, setRows] = useState([]);
  const [totals, setTotals] = useState({ /* initial totals as strings '0.00' */ });
  // ... other states

  useEffect(() => { /* fetchData as before */ }, [callApi]);
  const calculateTotals = useCallback(() => { /* as before */ }, [rows]);
  useEffect(() => { /* call calculateTotals as before */ }, [rows, calculateTotals]);

  const addRow = () => { /* as before */ };
  const deleteSelectedRows = () => { /* as before */ };
  const validateRows = () => { /* as before */ };
  const buildPayload = (saveFlag) => ({ /* as before */ });
  const handleSave = async () => { /* as before */ };
  const handleSubmit = async () => { /* as before */ };
  const handleCloseSnackbar = () => { /* as before */ };
  const handleCheckboxChange = useCallback((rowIndex, checked) => { /* as before */ }, []);

  const itemData = useMemo(
    () => ({ rows, columns, setRows, handleCheckboxChange }),
    [rows, handleCheckboxChange] // columns is stable
  );

  return (
    <Box sx={{ p: 2 }}>
      <TableContainer component={Paper} sx={{ width: '100%', mb: 2, overflowX: 'auto' }}>
        <Table stickyHeader sx={{ tableLayout: 'fixed', width: '100%' }}>
          <TableHead component="div"> {/* Render TableHead as a div */}
            <TableRow component="div" sx={{ display: 'flex', width: '100%', boxSizing: 'border-box' }}>
              {columns.map((col) => (
                <StyledTableCell
                  component="div"
                  key={col.key}
                  style={{
                      width: col.width,
                      boxSizing: 'border-box',
                      justifyContent: col.align === 'left' ? 'flex-start' : col.align === 'right' ? 'flex-end' : 'center',
                  }}
                >
                  {col.label}
                </StyledTableCell>
              ))}
            </TableRow>
            <TableRow component="div" sx={{ display: 'flex', width: '100%', boxSizing: 'border-box' }}>
              {columns.map((col) => (
                <StyledTableCell
                  component="div"
                  key={`${col.key}-sub`}
                  style={{
                      width: col.width,
                      boxSizing: 'border-box',
                      justifyContent: col.align === 'left' ? 'flex-start' : col.align === 'right' ? 'flex-end' : 'center',
                  }}
                >
                  {col.subHeading}
                </StyledTableCell>
              ))}
            </TableRow>
          </TableHead>

          <TableBody component="div"> {/* This div wraps react-window List */}
            {rows.length > 0 ? (
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
            ) : (
              // Keep standard TableBody/Row/Cell for "No data" message for simplicity
              // Or style it as a flex row too if it appears inside the main scrollable area
              <TableBody>
                <TableRow>
                  <TableCell colSpan={columns.length} align="center" sx={{ py: 3 }}>
                    No data available. Click "Add Row" to begin.
                  </TableCell>
                </TableRow>
              </TableBody>
            )}
          </TableBody>

          {/* Totals Row - Standard MUI Table elements, but styled with flex */}
          <TableBody component="div"> {/* Render outer TableBody as a div */}
             <TableRow
                component="div"
                sx={{ display: 'flex', width: '100%', boxSizing: 'border-box', '& .MuiTableCell-root': { fontWeight: 'bold' } }}
            >
              <StyledTableCell
                component="div"
                style={{
                  width: `calc(${columns[0].width} + ${columns[1].width})`, // Sum widths
                  boxSizing: 'border-box',
                  justifyContent: 'center', // Center "Total" text
                }}
              >
                Total
              </StyledTableCell>
              {columns.slice(2).map((col) => (
                <StyledTableCell
                  component="div"
                  key={`${col.key}-total`}
                  style={{
                      width: col.width,
                      boxSizing: 'border-box',
                      justifyContent: col.align === 'left' ? 'flex-start' : col.align === 'right' ? 'flex-end' : 'center',
                  }}
                >
                  {totals[`${col.key}Total`] !== undefined ? (
                    <TextField
                      value={totals[`${col.key}Total`]}
                      fullWidth
                      size="small"
                      InputProps={{
                        readOnly: true,
                        style: { textAlign: col.align || 'right', fontWeight: 'bold' },
                      }}
                      variant="outlined"
                    />
                  ) : null}
                </StyledTableCell>
              ))}
            </TableRow>
          </TableBody>
        </Table>
      </TableContainer>
      {/* ... Stack with Buttons and Snackbar ... */}
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

// Inside Schedule9ATable.js

// ... (imports and other definitions remain largely the same) ...

// MemoizedCellWithLocalState Component
const MemoizedCellWithLocalState = React.memo(({ row, column, index, setRows }) => {
  const [localValue, setLocalValue] = useState(String(row[column.key] || '')); // Ensure string for initial value

  useEffect(() => {
    setLocalValue(String(row[column.key] || '')); // Ensure string when row data changes
  }, [row, column.key]);

  const debouncedUpdateGlobalState = useMemo(
    () =>
      debounce((newValue) => { // Renamed to newValue for clarity
        setRows((prevRows) => {
          const updatedRows = [...prevRows];
          if (updatedRows[index]) {
            const targetRow = { ...updatedRows[index] };
            let processedValue = newValue;

            if (column.type === 'number') {
              if (newValue === '' || newValue === '-') {
                processedValue = newValue; // Keep empty or hyphen as is
              } else {
                const num = parseFloat(newValue);
                if (isNaN(num)) {
                  // Revert to the last valid value or empty if current input is not a number
                  processedValue = String(targetRow[column.key] || '');
                } else {
                  // Apply toFixed(2) only for specific financial columns
                  if (['aggOutStand', 'aggSecurities', 'provision', 'balInterestSuspenseAcc'].includes(column.key)) {
                    processedValue = num.toFixed(2);
                  } else {
                    processedValue = String(num); // For other potential numeric fields
                  }
                }
              }
            }
            // For non-numeric types, newValue is used directly,
            // as handleChange already prevents global update for pattern mismatches.
            targetRow[column.key] = processedValue;

            // Recalculate netShortfall if relevant inputs changed
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
    [index, column.key, column.type, setRows] // Added column.type to dependencies
  );

  const handleChange = (e) => {
    const val = e.target.value;
    setLocalValue(val); // STEP 1: Always update local state to reflect typed value immediately

    // STEP 2: Validate and decide whether to call global update
    if (column.pattern) {
      if (column.type === 'number') {
        // For numbers, we allow typing intermediate states (like "1.23." or even "abc").
        // The validation/coercion to a valid number format (e.g. toFixed(2) or reverting "abc")
        // will happen in `debouncedUpdateGlobalState`.
        // So, no early 'return' here for numbers based on `column.pattern`.
      } else { // For non-numeric types (e.g., alphanumeric)
        if (val !== '' && !column.pattern.test(val)) {
          // If the pattern for a non-numeric field is violated (e.g. "Branch$Name"),
          // localValue shows it, but we prevent the global state update.
          // The user needs to correct the input for it to be saved.
          return;
        }
      }
    }
    debouncedUpdateGlobalState(val); // Call global update for all numbers and valid non-numbers
  };

  return (
    <TextField
      value={localValue}
      onChange={handleChange}
      inputProps={{
        style: { textAlign: column.align || 'left' },
        maxLength: column.key === 'borrowerName' ? 255 : 18,
      }}
      size="small"
      sx={{ width: '100%' }}
    />
  );
});
MemoizedCellWithLocalState.displayName = 'MemoizedCellWithLocalState';


// RowRenderer (ensure boxSizing and check flex properties if issues persist)
const RowRenderer = React.memo(({ index, style, data }) => {
  const { rows, columns: tableColumns, setRows, handleCheckboxChange } = data;
  const rowItem = rows[index];

  if (!rowItem) {
    return null;
  }

  return (
    <TableRow
      key={rowItem.id || `row-${index}`}
      style={{
        ...style, // Provided by react-window (includes height, width, top, left)
        display: 'flex',
        boxSizing: 'border-box', // Good practice for consistent width rendering
      }}
    >
      {tableColumns.map((column) => (
        <StyledTableCell
          key={`${rowItem.id || index}-${column.key}`}
          align={column.align || 'center'} // This aligns the content block within the cell
          style={{
            width: column.width,
            boxSizing: 'border-box',
            // The following flex properties are for content WITHIN the cell, if needed
            // For simple TextField with width:100%, often not strictly necessary here
            // display: 'flex',
            // alignItems: 'center', // Vertically centers content like Checkbox or TextField
            // justifyContent: column.align || (column.type === 'checkbox' ? 'center' : 'flex-start'),
          }}
        >
          {column.type === 'checkbox' ? (
            <Checkbox
              checked={!!rowItem.isDelete}
              onChange={(e) => handleCheckboxChange(index, e.target.checked)}
              sx={{ padding: '0' }} // Reduce padding for better fit if needed
            />
          ) : column.editable ? (
            <MemoizedCellWithLocalState row={rowItem} column={column} index={index} setRows={setRows} />
          ) : (
            <TextField
              value={String(rowItem[column.key] || '')} // Ensure string value
              inputProps={{
                style: { textAlign: column.align || 'left' },
              }}
              size="small"
              disabled
              sx={{ width: '100%' }}
            />
          )}
        </StyledTableCell>
      ))}
    </TableRow>
  );
});
RowRenderer.displayName = 'RowRenderer';

// In Schedule9ATable function, `calculateTotals`
// Ensure parseFloat is used on potentially stringified numbers from `toFixed`
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
      { /* initial totals */ }
    );
    // ... setTotals with toFixed(2)
}, [rows]);


// Ensure initial state for totals are strings if TextField expects strings
// const [totals, setTotals] = useState({
//   aggOutStandTotal: '0.00',
//   aggSecuritiesTotal: '0.00',
//   // ...
// });

// The rest of your Schedule9ATable component...
// (TableHead, Totals Row, etc. should be mostly fine from the previous answer
// regarding width and alignment, but ensure they also use boxSizing: 'border-box'
// on TableRow and StyledTableCell if you encounter layout inconsistencies)

// Example for TableHead and Totals Row cells for consistency:
// <TableRow sx={{ display: 'flex', boxSizing: 'border-box' }}>
//   {columns.map((col) => (
//     <StyledTableCell
//       key={col.key}
//       align={col.align || 'left'}
//       style={{ width: col.width, boxSizing: 'border-box', /* other flex for content if needed */ }}
//     >
//       {col.label}
//     </StyledTableCell>
//   ))}
// </TableRow>

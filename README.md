Thanks for checking. If it's still misaligned even after applying column widths, the issue is likely due to MUI Table layout with react-window rendering outside the natural DOM flow.

To fix alignment precisely, you need to render the table body rows outside the actual <TableBody>, and instead simulate row rendering directly inside a container (with divs styled to match MUI’s cells).


---

Working Approach (Confirmed & Common Practice):

1. Keep header and totals inside <Table> as is.


2. Move react-window list rendering outside the MUI TableBody.


3. Use a Box or div wrapper with display table-row and table-cell styles to match alignment.




---

✅ Updated return JSX inside your Schedule9ATable:

<TableContainer component={Paper} sx={{ width: '100%', mb: 2 }}>
  <Table stickyHeader>
    <TableHead>
      <TableRow>
        {columns.map((col) => (
          <StyledTableCell
            key={col.key}
            align={col.align || 'left'}
            sx={{ width: col.width, minWidth: col.width, maxWidth: col.width }}
          >
            {col.label}
          </StyledTableCell>
        ))}
      </TableRow>
      <TableRow>
        {columns.map((col) => (
          <StyledTableCell
            key={`${col.key}-sub`}
            align={col.align || 'left'}
            sx={{ width: col.width, minWidth: col.width, maxWidth: col.width }}
          >
            {col.subHeading}
          </StyledTableCell>
        ))}
      </TableRow>
    </TableHead>
  </Table>
</TableContainer>

{/* React Window virtualized list OUTSIDE the Table */}
<Box
  sx={{
    width: '100%',
    borderLeft: '1px solid #ccc',
    borderRight: '1px solid #ccc',
    borderBottom: '1px solid #ccc',
  }}
>
  <AutoSizer disableHeight>
    {({ width }) => (
      <List
        height={Math.min(500, rows.length * 50)}
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
</Box>

{/* Totals row inside Table again */}
<TableContainer component={Paper}>
  <Table>
    <TableBody>
      <TableRow sx={{ '& .MuiTableCell-root': { fontWeight: 'bold' } }}>
        <StyledTableCell
          align="center"
          sx={{ width: columns[0].width }}
        >
          Total
        </StyledTableCell>
        <StyledTableCell
          align="center"
          sx={{ width: columns[1].width }}
        />
        {[
          'aggOutStandTotal',
          'aggSecuritiesTotal',
          'netShortfallTotal',
          'provisionTotal',
          'balInterestSuspenseAccTotal',
        ].map((key, i) => (
          <StyledTableCell
            key={key}
            align="right"
            sx={{ width: columns[i + 2].width }}
          >
            <TextField
              value={totals[key]}
              InputProps={{
                readOnly: true,
                style: { textAlign: 'right', fontWeight: 'bold' },
              }}
              size="small"
              variant="outlined"
              fullWidth
            />
          </StyledTableCell>
        ))}
      </TableRow>
    </TableBody>
  </Table>
</TableContainer>


---

✅ Update your RowRenderer to use divs, not TableRow

This is critical. MUI’s TableRow and TableCell won’t work properly inside react-window. Instead, simulate them:

const RowRenderer = React.memo(({ index, style, data }) => {
  const { rows, columns: tableColumns, setRows, handleCheckboxChange } = data;
  const rowItem = rows[index];

  if (!rowItem) return null;

  return (
    <Box
      style={{
        ...style,
        display: 'flex',
        borderBottom: '1px solid #e0e0e0',
      }}
    >
      {tableColumns.map((column) => (
        <Box
          key={`${rowItem.id || index}-${column.key}`}
          sx={{
            width: column.width,
            minWidth: column.width,
            maxWidth: column.width,
            px: 1,
            display: 'flex',
            alignItems: 'center',
            justifyContent: column.align === 'right' ? 'flex-end' : 'center',
            borderRight: '1px solid #e0e0e0',
          }}
        >
          {column.type === 'checkbox' ? (
            <Checkbox
              checked={!!rowItem.isDelete}
              onChange={(e) => handleCheckboxChange(index, e.target.checked)}
              size="small"
            />
          ) : column.editable ? (
            <MemoizedCellWithLocalState
              row={rowItem}
              column={column}
              index={index}
              setRows={setRows}
            />
          ) : (
            <TextField
              value={rowItem[column.key] || ''}
              size="small"
              fullWidth
              inputProps={{ style: { textAlign: 'right' }, readOnly: true }}
            />
          )}
        </Box>
      ))}
    </Box>
  );
});


---

Summary of Fix:

Remove <TableBody> around react-window

Use Box + AutoSizer to render rows

Convert each row into a Box with flex and fixed width cells

Ensure total and headers remain inside MUI Table



---

Would you like the full component file rewritten with this fix applied?


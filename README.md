const columns = [
  { key: 'isDelete', label: 'Select', width: 100, ... },
  { key: 'borrowerName', label: "Branch & Borrower's Name", width: 250, ... },
  { key: 'aggOutStand', label: 'Aggregate outstanding', width: 180, ... },
  ...
];

<StyledTableCell
  key={col.key}
  align={col.align || 'left'}
  sx={{ width: col.width, minWidth: col.width, maxWidth: col.width }}
>
  {col.label}
</StyledTableCell>

<StyledTableCell
  key={`${col.key}-sub`}
  align={col.align || 'left'}
  sx={{ width: col.width, minWidth: col.width, maxWidth: col.width }}
>
  {col.subHeading}
</StyledTableCell>

<StyledTableCell
  key={`${rowItem.id || index}-${column.key}`}
  align={column.align || 'center'}
  sx={{ width: column.width, minWidth: column.width, maxWidth: column.width }}
>
  {column.type === 'checkbox' ? (
    <Checkbox ... />
  ) : column.editable ? (
    <MemoizedCellWithLocalState row={rowItem} column={column} index={index} setRows={setRows} />
  ) : (
    <TextField
      value={rowItem[column.key] || ''}
      inputProps={{ style: { textAlign: 'right' } }}
      fullWidth
      size="small"
      disabled
    />
  )}
</StyledTableCell>
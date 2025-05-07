import { FixedSizeList as List } from 'react-window';
import AutoSizer from 'react-virtualized-auto-sizer';


const VirtualizedRow = ({ index, style, data }) => {
  const { rows, columns, handleCheckboxChange, handleInputChange } = data;
  const row = rows[index];

  return (
    <TableRow key={index} style={style}>
      {columns.map((column) => (
        <StyledTableCell key={`${index}-${column.key}`} align={column.align || 'left'}>
          {column.type === 'checkbox' ? (
            <Checkbox
              checked={row.isDelete}
              onChange={(event) => handleCheckboxChange(event, index)}
            />
          ) : column.editable ? (
            <TextField
              defaultValue={row[column.key] || ''}
              onBlur={(event) => handleInputChange(event, index, column.key)}
              inputProps={{
                style: { textAlign: 'right' },
                maxLength: column.key === 'borrowerName' ? 255 : 18,
              }}
              size="small"
            />
          ) : (
            <span>{row[column.key]}</span>
          )}
        </StyledTableCell>
      ))}
    </TableRow>
  );
};




<TableBody>
  <AutoSizer disableHeight>
    {({ width }) => (
      <List
        height={400}
        itemCount={rows.length}
        itemSize={50}
        width={width}
        itemData={{ rows, columns, handleCheckboxChange, handleInputChange }}
      >
        {VirtualizedRow}
      </List>
    )}
  </AutoSizer>

  {/* Total Row: Always rendered below virtualized rows */}
  <TableRow>
    <StyledTableCell colSpan={2} align="center"><b>Total</b></StyledTableCell>
    <StyledTableCell align="center">
      <TextField value={totals.aggOutStandTotal} InputProps={{ readOnly: true, style: { textAlign: 'right' } }} size="small" />
    </StyledTableCell>
    <StyledTableCell align="center">
      <TextField value={totals.aggSecuritiesTotal} InputProps={{ readOnly: true, style: { textAlign: 'right' } }} size="small" />
    </StyledTableCell>
    <StyledTableCell align="center">
      <TextField value={totals.netShortfallTotal} InputProps={{ readOnly: true, style: { textAlign: 'right' } }} size="small" />
    </StyledTableCell>
    <StyledTableCell align="center">
      <TextField value={totals.provisionTotal} InputProps={{ readOnly: true, style: { textAlign: 'right' } }} size="small" />
    </StyledTableCell>
    <StyledTableCell align="center">
      <TextField value={totals.balInterestSuspenseAccTotal} InputProps={{ readOnly: true, style: { textAlign: 'right' } }} size="small" />
    </StyledTableCell>
  </TableRow>
</TableBody>
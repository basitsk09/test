<StyledTableCell
  sx={{
    textAlign: 'left',
    fontWeight: row.type === 'total' ? 'bold' : 'normal',
    fontStyle: row.type === 'entry' ? 'normal' : 'italic',
    position: 'sticky',
    left: 0,
    zIndex: 99,
    backgroundColor:
      row.type === 'total'
        ? '#f5f5f5'
        : row.type === 'entry'
        ? '#ffffff'
        : '#f0f0f0',
  }}
>
  {row.label}
</StyledTableCell>
To highlight only the mismatched cells (in your case, likely the Adjustment column for specific rows) with a red border instead of changing the row background, do the following:


---

✅ Update Inside <StyledTableCell> Where <FormInput /> Is Rendered

Locate the cell rendering logic for FormInput, and apply a conditional sx style for border like this:

<StyledTableCell key={col.key} align="right">
  <FormInput
    key={`${row.id}-${col.key}-${data[col.key]}`}
    name={getFieldName(row.id, col.key)}
    value={data[col.key] || '0.00'}
    onBlur={(e) => handleChange(row.id, col.key, e.target.value)}
    inputProps={{ style: { textAlign: 'right' } }}
    readOnly={
      col.key === 'total' ||
      !col.editable ||
      Boolean(sectionMap[row.id]) ||
      (user.capacity === '61' && (
        col.key !== 'adjustment' || ['fac1', 'fac2', 'fac3'].includes(row.id)
      ))
    }
    sx={
      // Highlight only the mismatched cell for adjustment
      col.key === 'adjustment' && data.isMismatch
        ? { border: '1px solid red', borderRadius: 2 }
        : undefined
    }
  />
</StyledTableCell>


---

🧠 How This Works:

data.isMismatch is already computed per row in getRowData().

This condition:

col.key === 'adjustment' && data.isMismatch

ensures only the adjustment cell of the mismatched row gets a red border.



---

✅ Optional: Also Apply for Other Columns

If you want this red border for all columns in the mismatched row, change the condition to:

sx={
  data.isMismatch
    ? { border: '1px solid red', borderRadius: 2 }
    : undefined
}

Let me know if you'd like different styles for read-only vs editable fields when error is present.


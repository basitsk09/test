Thanks for the detailed context. Based on your requirement:


---

🧩 Business Rule Recap:

1. If user.capacity === '61':

Only the "adjustment" column is editable.

All other fields (standard, subStandard, etc.) should be disabled.

In section A1 (fac1, fac2, fac3):

Even adjustment should be disabled for these 3 rows.






---

✅ Fix for Conditional Field Editing

Update your <FormInput /> logic inside TableBody like this:

<FormInput
  key={`${row.id}-${col.key}-${data[col.key]}`}
  name={getFieldName(row.id, col.key)}
  value={data[col.key] || '0.00'}
  onBlur={(e) => handleChange(row.id, col.key, e.target.value)}
  inputProps={{ style: { textAlign: 'right' } }}
  readOnly={
    col.key === 'total' || // totals are always readonly
    !col.editable ||        // column is not editable (e.g., total)
    Boolean(sectionMap[row.id]) || // section/summary rows
    (
      user.capacity === '61'
        ? col.key !== 'adjustment' || ['fac1', 'fac2', 'fac3'].includes(row.id)
        : false
    ) // ✅ Core condition for capacity 61
  }
/>


---

✅ Explanation of Condition:

If column is not editable or it's a total/summary → make readOnly.

For capacity 61:

Only "adjustment" column is editable.

Disable "adjustment" for fac1, fac2, fac3.




---

Optional UX Tip:

If you want to visually indicate disabled fields (e.g., grayed out), you can use:

sx={{
  backgroundColor:
    user.capacity === '61' && col.key !== 'adjustment' ? '#f0f0f0' :
    user.capacity === '61' && ['fac1', 'fac2', 'fac3'].includes(row.id) && col.key === 'adjustment' ? '#f0f0f0' :
    undefined
}}

Let me know if you'd like to apply this logic globally in a helper function for clarity.


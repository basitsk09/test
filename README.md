Thanks for the detailed code. Based on your current state:


---

❌ Issues:

1. "Total C" (totalC) is not getting calculated – likely due to missing column in the table after removing it from headers.


2. "Grand Total" (grandTotal) and "Projects under construction" (premisesUnderCons) columns are missing inputs – because:

You've removed their header cells (commented out in first row).

Also you did not render their cells in the body loop since columnDisplayHeaders no longer includes them.





---

✅ Solution:

You need to manually render those 3 columns (totalC, premisesUnderCons, grandTotal) in both:

1. <TableHead> (first row as rowSpan=2),


2. <TableBody> (after rendering all columnDisplayHeaders).




---

✅ Fix 1: Add header cells back (only in first <TableRow>)

Replace your commented-out cells with this:

<StyledTableCell rowSpan={2} sx={{ minWidth: '120px' }}>
  <b>TOTAL (C) <br /> (j+k)</b>
</StyledTableCell>
<StyledTableCell rowSpan={2} sx={{ minWidth: '120px' }}>
  <b>(D) Projects under <br /> construction</b>
</StyledTableCell>
<StyledTableCell rowSpan={2} sx={{ minWidth: '120px' }}>
  <b>Grand Total <br /> (A + B + C + D)</b>
</StyledTableCell>


---

✅ Fix 2: Add 3 StyledTableCell blocks manually in TableBody

Just after this line:

{columnDisplayHeaders.map((colDef) => {

Add at the end of that .map(), after rendering all dynamic columns, this block:

// Manually render TOTAL (C)
<StyledTableCell key={`${rowKey}-totalC`}>
  <FormInput
    name={`${rowKey}-totalC`}
    value={displayDataForRow['totalC'] || '0.00'}
    readOnly
  />
</StyledTableCell>

// Manually render premisesUnderCons (editable)
<StyledTableCell key={`${rowKey}-premisesUnderCons`}>
  <FormInput
    name={`${rowKey}-premisesUnderCons`}
    value={displayDataForRow['premisesUnderCons'] || '0.00'}
    onChange={(e) => handleChange(rowDef.id, 'premisesUnderCons', e.target.value)}
    onBlur={(e) => handleBlur(rowDef.id, 'premisesUnderCons', e.target.value)}
  />
</StyledTableCell>

// Manually render grandTotal
<StyledTableCell key={`${rowKey}-grandTotal`}>
  <FormInput
    name={`${rowKey}-grandTotal`}
    value={displayDataForRow['grandTotal'] || '0.00'}
    readOnly
  />
</StyledTableCell>


---

✅ Optional (Cleaner Approach):

Instead of hardcoding these 3 fields, you can extract them into a separate constant:

const finalColumns = [
  { dataField: 'totalC', label: 'TOTAL (C)', readOnly: true },
  { dataField: 'premisesUnderCons', label: 'Projects under construction', readOnly: false },
  { dataField: 'grandTotal', label: 'Grand Total', readOnly: true },
];

Then render them in both header and body dynamically.


---

Let me know if you'd like me to patch this directly in your full component version.


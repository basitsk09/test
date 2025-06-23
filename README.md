Thanks for confirming. Let's fix it fully and clearly for your RW-04(A) table to show error when:

provAmtEnd of 1 ≠ sum of 1.i + 1.ii → show error under row 1

provAmtEnd of 3 ≠ sum of 3.i + 3.ii → show error under row 3



---

✅ Step-by-step working fix:

1. ✅ Modify RowMismatchMessage to accept the field name, so we can compare provAmtEnd:

const RowMismatchMessage = ({ mainId, parts, field }) => {
  const mainValue = parseFloat(staticData[`${mainId}_${field}`] || 0);
  const partsSum = parts.reduce(
    (acc, id) => acc + parseFloat(staticData[`${id}_${field}`] || 0),
    0
  );

  if (Math.abs(mainValue - partsSum) > 0.01) {
    return (
      <Typography color="error" fontSize={12} ml={12} mt={1}>
        The value does not match
      </Typography>
    );
  }
  return null;
};


---

2. ✅ Call this after your table (inside RW-04A tab):

<RowMismatchMessage mainId="1" parts={['1.i', '1.ii']} field="provAmtEnd" />
<RowMismatchMessage mainId="3" parts={['3.i', '3.ii']} field="provAmtEnd" />

Place this just after the closing </TableContainer> in tabIndex === 0 section.


---

3. (Optional) 🔴 Highlight provAmtEnd red if mismatched:

Inside your RW-04A TextField for provAmtEnd of row 1 or 3, change this:

<TextField
  value={staticData[`${row.id}_provAmtEnd`]}
  ...
  error={
    (row.id === '1' &&
      Math.abs(
        parseFloat(staticData['1_provAmtEnd'] || 0) -
          (parseFloat(staticData['1.i_provAmtEnd'] || 0) + parseFloat(staticData['1.ii_provAmtEnd'] || 0))
      ) > 0.01) ||
    (row.id === '3' &&
      Math.abs(
        parseFloat(staticData['3_provAmtEnd'] || 0) -
          (parseFloat(staticData['3.i_provAmtEnd'] || 0) + parseFloat(staticData['3.ii_provAmtEnd'] || 0))
      ) > 0.01)
  }
/>


---

💡 Summary:

RowMismatchMessage now works for any field like provAmtEnd

It shows red text if parent ≠ sum(child)

TextField shows red border too if mismatch


Let me know if you want to block Save/Submit if this mismatch exists.


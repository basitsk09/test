Great — here's exactly how to highlight the YSA column cells in A1 section (fac1, fac2, fac3, facTotal) with a red border if the total value doesn't match the ysa value.


---

✅ Target Condition:

For each of the following rows:
fac1, fac2, fac3, facTotal
→ If parseFloat(row.total) !== parseFloat(row.ysa)
→ Highlight the YSA <TextField /> with a red border.


---

✅ Step-by-Step Fix:

✅ 1. Ensure isMismatch logic is correct in getRowData()

You already have this inside your getRowData():

const isMismatch =
  ysa && id.startsWith('fac')
    ? Number(parseFloat(ysa || 0).toFixed(2)) !== Number(parseFloat(total || 0).toFixed(2))
    : false;

This is ✅ correct — it marks a mismatch for A1 rows.


---

✅ 2. Update the YSA <TextField /> in JSX to add red border

Update this block inside <StyledTableCell align="right"> for YSA rendering:

{ysaRows.includes(row.id) && (
  <TextField
    name={`${row.id}_ysa`}
    size="small"
    value={data.ysa}
    inputProps={{ style: { textAlign: 'right' } }}
    fullWidth
    disabled
    sx={
      data.isMismatch
        ? { border: '1px solid red', borderRadius: 1 }
        : {}
    }
  />
)}


---

🧠 Explanation:

This makes sure only rows in ysaRows (fac1, fac2, fac3, facTotal) get the validation.

If the computed total doesn’t match the YSA, a red border is shown.



---

✅ Optional: Also show tooltip or icon?

If you want to add a warning icon or tooltip for mismatch, I can help integrate that using MUI Tooltip and ErrorOutlineIcon.

Let me know if you want that enhancement.


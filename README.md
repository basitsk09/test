To ensure that the sum of provAmtEnd for rows 1.i and 1.ii equals the provAmtEnd of row 1, and similarly for 3.i, 3.ii, and 3, you need to add validation logic specific to provAmtEnd, not just provAmtStart.


---

✅ Step-by-Step Fix

Update the RowMismatchMessage component to check provAmtEnd values instead of provAmtStart.

🔁 Update this line in RowMismatchMessage:

const mainValue = parseFloat(staticData[`${mainId}_provAmtStart`] || 0);
const partsSum = parts.reduce((acc, id) => acc + parseFloat(staticData[`${id}_provAmtStart`] || 0), 0);

✅ Replace it with:

const mainValue = parseFloat(staticData[`${mainId}_provAmtEnd`] || 0);
const partsSum = parts.reduce((acc, id) => acc + parseFloat(staticData[`${id}_provAmtEnd`] || 0), 0);


---

✅ Optional: Add Red Border on Total If Invalid

You can also add this conditional style to the main row’s provAmtEnd input:

<TextField
  value={staticData[`${row.id}_provAmtEnd`]}
  size="small"
  onChange={(e) => handleStaticChange(row.id, 'provAmtEnd', e.target.value)}
  disabled={!['1.i', '1.ii', '3.i', '3.ii'].includes(row.id)}
  error={
    (row.id === '1' && Math.abs(
      parseFloat(staticData['1.i_provAmtEnd'] || 0) + parseFloat(staticData['1.ii_provAmtEnd'] || 0)
      - parseFloat(staticData['1_provAmtEnd'] || 0)
    ) > 0.01) ||
    (row.id === '3' && Math.abs(
      parseFloat(staticData['3.i_provAmtEnd'] || 0) + parseFloat(staticData['3.ii_provAmtEnd'] || 0)
      - parseFloat(staticData['3_provAmtEnd'] || 0)
    ) > 0.01)
  }
  sx={{ width: '150px' }}
/>


---

✅ Final Result:

Error message appears below row 1 and 3 if mismatch.

provAmtEnd field in row 1 or 3 shows a red border if the total doesn’t match child rows.


Let me know if you want this mismatch check to be part of submit/save logic too.


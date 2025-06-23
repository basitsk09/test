Thanks for sharing the full code. The issue is that you're displaying the error only for row.id === '1' and '3', but you placed the error logic inside the provAmtEnd TextField of rows '1.i', '1.ii', etc., which will never match row.id === '1'.


---

✅ Fix: Move the error logic only to rows 1 and 3, where the check is applicable.

Update your <TextField> for provAmtEnd inside this block:

<TableCell align="right">
  <TextField
    value={staticData[`${row.id}_provAmtEnd`]}
    size="small"
    onChange={(e) => handleStaticChange(row.id, 'provAmtEnd', e.target.value)}
    disabled={!['1.i', '1.ii', '3.i', '3.ii'].includes(row.id)}
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
    sx={{ width: '150px' }}
  />
</TableCell>

🔁 Change it to this:

<TableCell align="right">
  <TextField
    value={staticData[`${row.id}_provAmtEnd`]}
    size="small"
    onChange={(e) => handleStaticChange(row.id, 'provAmtEnd', e.target.value)}
    disabled={!['1.i', '1.ii', '3.i', '3.ii'].includes(row.id)}
    error={
      row.id === '1'
        ? Math.abs(
            parseFloat(staticData['1_provAmtEnd'] || 0) -
              (parseFloat(staticData['1.i_provAmtEnd'] || 0) + parseFloat(staticData['1.ii_provAmtEnd'] || 0))
          ) > 0.01
        : row.id === '3'
        ? Math.abs(
            parseFloat(staticData['3_provAmtEnd'] || 0) -
              (parseFloat(staticData['3.i_provAmtEnd'] || 0) + parseFloat(staticData['3.ii_provAmtEnd'] || 0))
          ) > 0.01
        : false
    }
    sx={{ width: '150px' }}
  />
</TableCell>


---

✅ Or cleaner alternative:

Move this condition to a helper function:

const isProvAmtEndMismatch = (id) => {
  if (id === '1') {
    const parent = parseFloat(staticData['1_provAmtEnd'] || 0);
    const children =
      parseFloat(staticData['1.i_provAmtEnd'] || 0) + parseFloat(staticData['1.ii_provAmtEnd'] || 0);
    return Math.abs(parent - children) > 0.01;
  }
  if (id === '3') {
    const parent = parseFloat(staticData['3_provAmtEnd'] || 0);
    const children =
      parseFloat(staticData['3.i_provAmtEnd'] || 0) + parseFloat(staticData['3.ii_provAmtEnd'] || 0);
    return Math.abs(parent - children) > 0.01;
  }
  return false;
};

Then use this in your JSX:

<TextField
  ...
  error={isProvAmtEndMismatch(row.id)}
/>


---

✅ Bonus: Disable Save/Submit button when mismatch exists

Add:

const isSaveDisabled =
  isProvAmtEndMismatch('1') || isProvAmtEndMismatch('3');

Then:

<Button variant="contained" disabled={isSaveDisabled}>Save</Button>
<Button variant="contained" color="success" disabled={isSaveDisabled}>Submit</Button>


---

Let me know if you'd also like to show total below child rows or want to apply this logic to Tab 2 as well.


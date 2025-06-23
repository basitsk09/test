You're absolutely right — the mismatch error should appear directly below the "PROVISIONABLE AMT AS ON 31/03/2026" field (provAmtEnd) for rows 1 and 3, not at the bottom of the table.


---

✅ Solution:

Update your code where you render the provAmtEnd field in rows 1 and 3 only like this:

Step 1: Add helper function (if not already)

const getProvAmtEndMismatchError = (rowId) => {
  if (rowId === '1') {
    const parent = parseFloat(staticData['1_provAmtEnd'] || 0);
    const sum =
      parseFloat(staticData['1.i_provAmtEnd'] || 0) +
      parseFloat(staticData['1.ii_provAmtEnd'] || 0);
    return Math.abs(parent - sum) > 0.01;
  }
  if (rowId === '3') {
    const parent = parseFloat(staticData['3_provAmtEnd'] || 0);
    const sum =
      parseFloat(staticData['3.i_provAmtEnd'] || 0) +
      parseFloat(staticData['3.ii_provAmtEnd'] || 0);
    return Math.abs(parent - sum) > 0.01;
  }
  return false;
};


---

Step 2: Update the JSX for provAmtEnd field in <TableCell> like this:

<TableCell align="right">
  <Box display="flex" flexDirection="column">
    <TextField
      value={staticData[`${row.id}_provAmtEnd`]}
      size="small"
      onChange={(e) => handleStaticChange(row.id, 'provAmtEnd', e.target.value)}
      disabled={!['1.i', '1.ii', '3.i', '3.ii'].includes(row.id)}
      error={getProvAmtEndMismatchError(row.id)}
      sx={{ width: '150px' }}
    />
    {getProvAmtEndMismatchError(row.id) && (
      <Typography fontSize={11} color="error">
        The value does not match
      </Typography>
    )}
  </Box>
</TableCell>

✔ This will:

Show the red error text just below the TextField of rows 1 and 3

Keep it clean for all other rows

Still show red outline around the field



---

Let me know if you'd like the same behavior added for provRequired or any other row-level total checks.


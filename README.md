Perfect. You want to:

✅ Highlight the total column cells in rows facTotal, secTotal, and advA3 with a red border if:

A1 (facTotal).total ≠ A2 (secTotal).total ≠ A3 (advA3).total


---

✅ Step-by-Step Fix

1. Create columnMismatchFlags using useMemo

Add this in your component just below computedRowData:

const totalMismatchFlags = useMemo(() => {
  const a = getRowData('facTotal');
  const b = getRowData('secTotal');
  const c = getRowData('advA3');

  const mismatches = {
    standard: false,
    subStandard: false,
    doubtful: false,
    loss: false,
    adjustment: false,
  };

  ['standard', 'subStandard', 'doubtful', 'loss'].forEach((key) => {
    const aVal = parseFloat(a[key] || 0).toFixed(2);
    const bVal = parseFloat(b[key] || 0).toFixed(2);
    const cVal = parseFloat(c[key] || 0).toFixed(2);
    if (aVal !== bVal || bVal !== cVal || aVal !== cVal) {
      mismatches[key] = true;
    }
  });

  if (user.capacity === '61') {
    const aAdj = parseFloat(a.adjustment || 0).toFixed(2);
    const bAdj = parseFloat(b.adjustment || 0).toFixed(2);
    const cAdj = parseFloat(c.adjustment || 0).toFixed(2);
    if (aAdj !== bAdj || bAdj !== cAdj || aAdj !== cAdj) {
      mismatches.adjustment = true;
    }
  }

  return mismatches;
}, [values, showAdjustment, user.capacity]);


---

2. Apply Red Border for Total Cells Conditionally

Update this inside your <FormInput /> render:

<FormInput
  ...
  sx={
    // ✅ For mismatch of A1-A2-A3 totals
    ['facTotal', 'secTotal', 'advA3'].includes(row.id) && col.key !== 'total' && totalMismatchFlags[col.key]
      ? { border: '1px solid red', borderRadius: 2 }
      : undefined
  }
/>

> This highlights only standard, subStandard, doubtful, loss, and adjustment columns for rows facTotal, secTotal, and advA3.




---

✅ Optional: Highlight Total Column Only

If you instead want to highlight only the "Total" column in those rows, update this:

sx={
  ['facTotal', 'secTotal', 'advA3'].includes(row.id) &&
  col.key === 'total' &&
  Object.values(totalMismatchFlags).some(Boolean)
    ? { border: '1px solid red', borderRadius: 2 }
    : undefined
}

This version applies red border to the total column only if any mismatch is found in any category.


---

Let me know which version you'd prefer — only total cell red, or individual column cells red — and I can finalize styles accordingly.


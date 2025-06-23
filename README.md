const calculateAndSetStatic = (rowId, updated) => {
  const isManualRow = ['1.i', '1.ii', '3.i', '3.ii'].includes(rowId);

  const start = parseFloat(updated[`${rowId}_provAmtStart`] || 0);
  const write = parseFloat(updated[`${rowId}_writeOff`] || 0);
  const add = parseFloat(updated[`${rowId}_addition`] || 0);
  const reduce = parseFloat(updated[`${rowId}_reduction`] || 0);
  const rate = parseFloat(updated[`${rowId}_rate`] || 0);

  if (!isManualRow) {
    const end = start - write + add - reduce;
    updated[`${rowId}_provAmtEnd`] = end.toFixed(2);
    updated[`${rowId}_provRequired`] = ((end * rate) / 100).toFixed(2);
  } else {
    const end = parseFloat(updated[`${rowId}_provAmtEnd`] || 0);
    updated[`${rowId}_provRequired`] = ((end * rate) / 100).toFixed(2);
  }
};
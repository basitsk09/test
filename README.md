const calculateTotals = (rows, fieldPrefix = '') => {
  const toCents = (val) => {
    if (!val || isNaN(val)) return 0n;
    const str = val.toString().replace(/,/g, '').trim();
    const isNeg = str.startsWith('-');
    const [intPart, decPart = ''] = (isNeg ? str.slice(1) : str).split('.');
    const cents = BigInt(intPart || '0') * 100n + BigInt((decPart + '00').slice(0, 2));
    return isNeg ? -cents : cents;
  };

  const totalsCents = {
    inSusp: 0n,
    provn: 0n,
    licra: 0n,
    dicgc: 0n,
  };

  rows.forEach((row) => {
    ['inSusp', 'provn', 'licra', 'dicgc'].forEach((field) => {
      const val = row[`${field}${fieldPrefix}`];
      totalsCents[field] += toCents(val);
    });
  });

  const totals = {};
  Object.entries(totalsCents).forEach(([key, val]) => {
    const abs = val < 0n ? -val : val;
    const intPart = abs / 100n;
    const decPart = (abs % 100n).toString().padStart(2, '0');
    totals[key] = `${val < 0n ? '-' : ''}${intPart.toString()}.${decPart}`;
  });

  return totals;
};
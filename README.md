21212121 0 0
RW05.jsx?t=1750850535392:109 0 12121212 0
RW05.jsx?t=1750850535392:109 21212121 0 12121212
RW05.jsx?t=1750850535392:109 0 12121212 12121212


const calculateAndSetStatic = (rowId, updated) => {
    const provAmtStart = parseFloat(updated[`${rowId}_provAmtStart`] || 0);
    const addition = parseFloat(updated[`${rowId}_addition`] || 0);
    const reversal = parseFloat(updated[`${rowId}_reversal`] || 0);

    console.log(provAmtStart, addition, reversal);

    // Provision as on 30/06/2024(6)={(3+4)-(5)}
    const provAmtEnd = provAmtStart + addition - reversal;
    updated[`${rowId}_provAmtEnd`] = provAmtEnd.toFixed(2);

    // Difference to be Provided/ written back(7)={(6)-(3)}
    const difference = provAmtEnd - provAmtStart;
    updated[`${rowId}_difference`] = difference.toFixed(2);
  };

You're encountering two key issues in your Schedule9CMigration component:


---

🐞 Issues:

1. ✅ Precision issue: .12 becomes .13

Likely due to parseFloat(...) on undefined or blank fields during totals.

2. ✅ Negative numbers not allowed

Your validateNumeric() doesn't accept negative numbers properly.


---

✅ Fixes:

✅ 1. Fix Precision Issue in calculateTotals

Update calculateTotals to safely handle blank, invalid, or NaN values:

const calculateTotals = (rows, fieldPrefix = '') => {
  return rows.reduce(
    (totals, row) => {
      ['inSusp', 'provn', 'licra', 'dicgc'].forEach((field) => {
        const value = parseFloat(row[`${field}${fieldPrefix}`]);
        totals[field] += isNaN(value) ? 0 : value;
      });
      return totals;
    },
    { inSusp: 0, provn: 0, licra: 0, dicgc: 0 }
  );
};

🔍 This ensures:

undefined, '', or malformed input (like . or -.) doesn't become NaN → weird float

Keeps precision clean



---

✅ 2. Allow negative numbers with up to 2 decimals

Update your validateNumeric() regex:

const validateNumeric = (value) => /^-?\d{0,12}(\.\d{0,2})?$/.test(value);

This allows:

optional - sign

max 12 digits before decimal

max 2 decimal digits


🔎 Examples:

-0.12 ✅

1000.99 ✅

-9999999999.99 ✅

- ❌ (optional: allow with value === '-' check in handleChange)



---

✅ 3. (Optional UX) Pad .1 to .10 or .00

If needed, update final value formatting like:

totals[field].toFixed(2)

Already present in:

<FormInput value={totals[field].toFixed(2)} ... />

You're doing it right here.


---

✅ Summary

Problem	Fix

.12 becomes .13	Wrap parseFloat(...) with isNaN(...) ? 0 : value
Negative values disallowed	Update regex to ^-?\d{0,12}(\.\d{0,2})?$
Floating point drift	Always use .toFixed(2) in display



---

Let me know if you want to:

Block inputs like just - or .

Auto-format on blur

Add min/max value limits per field



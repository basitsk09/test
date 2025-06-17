Yes, you have two alternatives to using decimal.js—but each comes with trade-offs in precision, maintainability, and performance.


---

✅ Option 1: Manual Fixed-Point Math (String-based Addition)

Use integers to represent amounts in paise (or cents) and only convert to decimal for display.
For example:

Instead of 999999999999999.25

Store as: 99999999999999925 (in paise, no decimal issues)


Pros:

No external library

Accurate integer math

Works well with large numbers


Cons:

You must manage scaling (e.g., multiply/divide by 100 manually)

Slightly harder to maintain


Example:

const parseToPaise = (val) => Math.round(parseFloat(val || '0') * 100);

const toFixedTwo = (paise) => (paise / 100).toFixed(2);

const sum = (...vals) => {
  const totalPaise = vals.reduce((sum, val) => sum + parseToPaise(val), 0);
  return toFixedTwo(totalPaise);
};

updated.othForeBilPurGrAmt = sum(data.payIndGrAmt, data.payOutGrAmt);


---

✅ Option 2: Use BigInt with scaled values

Similar to Option 1 but with BigInt for very large numbers.

const parseToPaise = (val) => BigInt(Math.round(parseFloat(val || '0') * 100));
const toFixedTwo = (paiseBigInt) => (Number(paiseBigInt) / 100).toFixed(2);

const sum = (...vals) => {
  const total = vals.reduce((acc, val) => acc + parseToPaise(val), BigInt(0));
  return toFixedTwo(total);
};

Limitation:

BigInt can't interact directly with Number

You may lose fractional accuracy if you convert too early



---

❌ Avoid: Native .toFixed(2) directly on floating-point additions

You're already seeing its downside:

999999999999999.25 + 0.25 = 999999999999999.19 ❌


---

✅ Recommended (If You Want to Avoid Libraries):

Use Option 1 (string → paise → integer math → string) if:

You want precision

You want to avoid dependencies

Your system uses monetary values and doesn't require dynamic decimal length



---

Would you like me to refactor your useEffect logic to use manual fixed-point math using Option 1?


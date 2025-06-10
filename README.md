Thanks for checking. If it’s still rounding .99 to 1.00, then one of the following is likely happening:

1. You're applying .toFixed(2) or formatting again in render (inside <FormInput />)


2. You're converting to string too early


3. FormInput internally does formatting or rounding




---

✅ Step-by-Step Fix

We'll do three things to fix it completely:


---

✅ 1. Remove .toFixed(2) from calculation entirely

Replace the calculation useEffect with this — note: we don’t use .toFixed() anywhere:

useEffect(() => {
  const parse = (v) => {
    const num = parseFloat(v);
    return isNaN(num) ? 0 : num;
  };

  const updated = { ...data };

  updated.othForeBilPurGrAmt = parse(data.payIndGrAmt) + parse(data.payOutGrAmt);
  updated.othForeBilPurPro = parse(data.payIndPro) + parse(data.payOutPro);

  updated.foreBilPurGrAmt =
    parse(data.expBillGrAmt) + parse(data.impBillGrAmt) + updated.othForeBilPurGrAmt;
  updated.foreBilPurPro =
    parse(data.expBillPro) + parse(data.impBillPro) + updated.othForeBilPurPro;

  updated.bilPurGrAmt = parse(data.inlBilPurGrAmt) + updated.foreBilPurGrAmt;
  updated.bilPurPro = parse(data.inlBilPurPro) + updated.foreBilPurPro;

  updated.dueGrAmt =
    parse(data.coopBankGrAmt) +
    parse(data.commBankGrAmt) +
    parse(data.bankOutIndGrAmt);
  updated.duePro =
    parse(data.coopBankPro) +
    parse(data.commBankPro) +
    parse(data.bankOutIndPro);

  updated.loanAdvGrAmt = parse(data.loanAdvCreGrAmt) + updated.dueGrAmt;
  updated.loanAdvPro = parse(data.loanAdvCrePro) + updated.duePro;

  updated.grandTotlGrAmt = updated.loanAdvGrAmt + updated.bilPurGrAmt;
  updated.grandTotlPro = updated.loanAdvPro + updated.bilPurPro;

  // No .toFixed here
  setData(updated);
}, [
  data.payIndGrAmt, data.payOutGrAmt, data.expBillGrAmt, data.impBillGrAmt,
  data.inlBilPurGrAmt, data.loanAdvCreGrAmt, data.coopBankGrAmt,
  data.commBankGrAmt, data.bankOutIndGrAmt,
  data.payIndPro, data.payOutPro, data.expBillPro, data.impBillPro,
  data.inlBilPurPro, data.loanAdvCrePro, data.coopBankPro, data.commBankPro, data.bankOutIndPro,
]);


---

✅ 2. Ensure <FormInput /> doesn’t auto-round

In your FormInput:

<FormInput
  name={grField}
  value={data[grField] || ''}
  ...
  isNumeric={true}
/>

If FormInput does auto-rounding for numeric values, it might convert 999999999999999.99 to 1000000000000000.00.

Fix it like this:

✅ Instead of:

value={data[grField] || ''}

Use:

value={data[grField]?.toString() ?? ''}

And double-check FormInput code. It must not call .toFixed() or Math.round() internally for isNumeric={true}.


---

✅ 3. (Optional) Only apply .toFixed(2) on submission or final display, like this:

const rounded = parseFloat(data.grandTotlGrAmt).toFixed(2);


---

🔍 If Still Broken:

Please share your <FormInput /> component code, or try replacing it just temporarily with TextField to confirm if it's the culprit.

Example:

<TextField
  value={data[grField]?.toString() ?? ''}
  onChange={handleChange(grField)}
  onBlur={handleChange(grField, true)}
  inputProps={{ style: { textAlign: 'right' } }}
/>


---

Let me know if you want me to refactor the FormInput or share a version that guarantees .99 will never round off.


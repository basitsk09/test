useEffect(() => {
  // Convert rupee string to paise integer
  const parseToPaise = (val) => {
    const num = parseFloat(val || '0');
    if (isNaN(num)) return 0;
    return Math.round(num * 100); // Convert to paise
  };

  // Convert paise integer back to "Rs.P" formatted string
  const toRupeeString = (paise) => (paise / 100).toFixed(2);

  // Sum any number of fields safely (accepts either raw numbers or pre-formatted)
  const sumPaise = (...fields) =>
    fields.reduce((acc, val) => acc + parseToPaise(val), 0);

  const updated = { ...data };

  const othForeBilPurGrAmt = sumPaise(data.payIndGrAmt, data.payOutGrAmt);
  const othForeBilPurPro = sumPaise(data.payIndPro, data.payOutPro);

  const foreBilPurGrAmt = sumPaise(data.expBillGrAmt, data.impBillGrAmt, othForeBilPurGrAmt);
  const foreBilPurPro = sumPaise(data.expBillPro, data.impBillPro, othForeBilPurPro);

  const bilPurGrAmt = sumPaise(data.inlBilPurGrAmt, foreBilPurGrAmt);
  const bilPurPro = sumPaise(data.inlBilPurPro, foreBilPurPro);

  const dueGrAmt = sumPaise(data.coopBankGrAmt, data.commBankGrAmt, data.bankOutIndGrAmt);
  const duePro = sumPaise(data.coopBankPro, data.commBankPro, data.bankOutIndPro);

  const loanAdvGrAmt = sumPaise(data.loanAdvCreGrAmt, dueGrAmt);
  const loanAdvPro = sumPaise(data.loanAdvCrePro, duePro);

  const grandTotlGrAmt = sumPaise(loanAdvGrAmt, bilPurGrAmt);
  const grandTotlPro = sumPaise(loanAdvPro, bilPurPro);

  // Set values as fixed-precision strings
  updated.othForeBilPurGrAmt = toRupeeString(othForeBilPurGrAmt);
  updated.othForeBilPurPro = toRupeeString(othForeBilPurPro);

  updated.foreBilPurGrAmt = toRupeeString(foreBilPurGrAmt);
  updated.foreBilPurPro = toRupeeString(foreBilPurPro);

  updated.bilPurGrAmt = toRupeeString(bilPurGrAmt);
  updated.bilPurPro = toRupeeString(bilPurPro);

  updated.dueGrAmt = toRupeeString(dueGrAmt);
  updated.duePro = toRupeeString(duePro);

  updated.loanAdvGrAmt = toRupeeString(loanAdvGrAmt);
  updated.loanAdvPro = toRupeeString(loanAdvPro);

  updated.grandTotlGrAmt = toRupeeString(grandTotlGrAmt);
  updated.grandTotlPro = toRupeeString(grandTotlPro);

  setData(updated);
}, [
  data.payIndGrAmt,
  data.payOutGrAmt,
  data.expBillGrAmt,
  data.impBillGrAmt,
  data.inlBilPurGrAmt,
  data.loanAdvCreGrAmt,
  data.coopBankGrAmt,
  data.commBankGrAmt,
  data.bankOutIndGrAmt,
  data.payIndPro,
  data.payOutPro,
  data.expBillPro,
  data.impBillPro,
  data.inlBilPurPro,
  data.loanAdvCrePro,
  data.coopBankPro,
  data.commBankPro,
  data.bankOutIndPro,
]);
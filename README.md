useEffect(() => {
  const parseToPaise = (val) => Math.round(parseFloat(val || '0') * 100);
  const toFixedTwo = (paise) => (paise / 100).toFixed(2);

  const sum = (...values) => {
    const total = values.reduce((acc, val) => acc + parseToPaise(val), 0);
    return toFixedTwo(total);
  };

  const updated = { ...data };

  updated.othForeBilPurGrAmt = sum(data.payIndGrAmt, data.payOutGrAmt);
  updated.othForeBilPurPro = sum(data.payIndPro, data.payOutPro);

  updated.foreBilPurGrAmt = sum(data.expBillGrAmt, data.impBillGrAmt, updated.othForeBilPurGrAmt);
  updated.foreBilPurPro = sum(data.expBillPro, data.impBillPro, updated.othForeBilPurPro);

  updated.bilPurGrAmt = sum(data.inlBilPurGrAmt, updated.foreBilPurGrAmt);
  updated.bilPurPro = sum(data.inlBilPurPro, updated.foreBilPurPro);

  updated.dueGrAmt = sum(data.coopBankGrAmt, data.commBankGrAmt, data.bankOutIndGrAmt);
  updated.duePro = sum(data.coopBankPro, data.commBankPro, data.bankOutIndPro);

  updated.loanAdvGrAmt = sum(data.loanAdvCreGrAmt, updated.dueGrAmt);
  updated.loanAdvPro = sum(data.loanAdvCrePro, updated.duePro);

  updated.grandTotlGrAmt = sum(updated.loanAdvGrAmt, updated.bilPurGrAmt);
  updated.grandTotlPro = sum(updated.loanAdvPro, updated.bilPurPro);

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
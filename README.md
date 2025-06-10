useEffect(() => {
  const parse = (v) => {
    const num = parseFloat(v);
    return isNaN(num) ? 0 : num;
  };

  const updated = { ...data };

  // Step-by-step parsing
  const payIndGr = parse(data.payIndGrAmt);
  const payOutGr = parse(data.payOutGrAmt);
  const payIndPro = parse(data.payIndPro);
  const payOutPro = parse(data.payOutPro);

  updated.othForeBilPurGrAmt = payIndGr + payOutGr;
  updated.othForeBilPurPro = payIndPro + payOutPro;

  const expBillGr = parse(data.expBillGrAmt);
  const impBillGr = parse(data.impBillGrAmt);
  const expBillPro = parse(data.expBillPro);
  const impBillPro = parse(data.impBillPro);

  updated.foreBilPurGrAmt = expBillGr + impBillGr + updated.othForeBilPurGrAmt;
  updated.foreBilPurPro = expBillPro + impBillPro + updated.othForeBilPurPro;

  const inlBilPurGr = parse(data.inlBilPurGrAmt);
  const inlBilPurPro = parse(data.inlBilPurPro);

  updated.bilPurGrAmt = inlBilPurGr + updated.foreBilPurGrAmt;
  updated.bilPurPro = inlBilPurPro + updated.foreBilPurPro;

  const coopGr = parse(data.coopBankGrAmt);
  const commGr = parse(data.commBankGrAmt);
  const outIndGr = parse(data.bankOutIndGrAmt);

  const coopPro = parse(data.coopBankPro);
  const commPro = parse(data.commBankPro);
  const outIndPro = parse(data.bankOutIndPro);

  updated.dueGrAmt = coopGr + commGr + outIndGr;
  updated.duePro = coopPro + commPro + outIndPro;

  const loanCreGr = parse(data.loanAdvCreGrAmt);
  const loanCrePro = parse(data.loanAdvCrePro);

  updated.loanAdvGrAmt = loanCreGr + updated.dueGrAmt;
  updated.loanAdvPro = loanCrePro + updated.duePro;

  updated.grandTotlGrAmt = updated.loanAdvGrAmt + updated.bilPurGrAmt;
  updated.grandTotlPro = updated.loanAdvPro + updated.bilPurPro;

  // Only now apply toFixed(2)
  Object.keys(updated).forEach((key) => {
    if (typeof updated[key] === 'number') {
      updated[key] = updated[key].toFixed(2);
    }
  });

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
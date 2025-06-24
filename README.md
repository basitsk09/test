const handleSubmit = (action = 'save') => {
  const formData = new FormData();

  const allRows = [...initialStaticRows.map(row => ({
    particulars: row.label,
    provAmtStart: staticData[`${row.id}_provAmtStart`] || '0',
    writeOff: staticData[`${row.id}_writeOff`] || '0',
    addition: staticData[`${row.id}_addition`] || '0',
    reduction: staticData[`${row.id}_reduction`] || '0',
    provAmtEnd: staticData[`${row.id}_provAmtEnd`] || '0',
    rate: staticData[`${row.id}_rate`] || '0',
    provRequired: staticData[`${row.id}_provRequired`] || '0',
  })), ...dynamicRows];

  // map to backend fields
  const fieldMap = {
    particulars: 'particularsList',
    provAmtStart: 'provAmt2015List',
    writeOff: 'writeOffDur12monList',
    addition: 'additionDur12monList',
    reduction: 'reduInProviAmtList',
    provAmtEnd: 'proviAmt2016List',
    rate: 'ratePOfProvList',
    provRequired: 'provReqList',
  };

  Object.entries(fieldMap).forEach(([frontendKey, backendKey]) => {
    allRows.forEach(row => {
      formData.append(backendKey, row[frontendKey] || '0');
    });
  });

  formData.append('updateFlag', action === 'submit' ? 'SUBMIT' : 'SAVE');

  fetch('/your/api/endpoint', {
    method: 'POST',
    body: formData,
  })
    .then(res => res.json())
    .then(data => {
      // handle response
    });
};
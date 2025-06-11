const fetchReport = async () => {
  try {
    console.log('user', user);
    console.log('reportObject', reportObject);

    const requestPayload = {
      circleCode: user.circleCode,
      quarterEndDate: user.quarterEndDate,
      role: user.capacity,
      reportName: reportObject.name,
    };

    const data = await callApi('/Maker/getSC09ReportData', requestPayload, 'POST');

    setValues((prev) => {
      const nv = { ...prev };

      const backendFieldMap = {
        standard: 'Standard',
        subStandard: 'SubStandard',
        doubtful: 'Doubtful',
        loss: 'Loss',
        adjustment: 'Adj', // key fix here
      };

      const fields = ['standard', 'subStandard', 'doubtful', 'loss'];
      if (user.capacity === '61') fields.push('adjustment');

      ['1', '2', '3'].forEach((num) => {
        fields.forEach((field) => {
          const suffix = backendFieldMap[field];
          nv[`fac${num}`][field] = data[`facility_${suffix}_${num}`] || '';
          nv[`sec${num}`][field] = data[`security_${suffix}_${num}`] || '';
          nv[`in${num}`][field] = data[`sector_${suffix}_a${num}`] || '';
          nv[`out${num}`][field] = data[`sector_${suffix}_b${num}`] || '';
        });
      });

      return nv;
    });
  } catch (e) {
    console.error('Error in fetchReport:', e);
  }
};
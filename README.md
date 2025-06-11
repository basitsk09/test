const fetchValidation = async () => {
  try {
    const requestPayload2 = {
      circleCode: user.circleCode,
      quarterEndDate: user.quarterEndDate,
      role: user.capacity,
    };

    const data = await callApi('/Maker/getSC09ValiadationData', requestPayload2, 'POST');
    console.log(data, 'fetchValidation');

    setValues((prev) => {
      const nv = { ...prev };

      if (user.capacity === '61') {
        const normalize = (s) => s.replace(/\s|[^a-zA-Z0-9]/g, '').toLowerCase();
        ['fac1', 'fac2', 'fac3'].forEach((id) => {
          const label = rowDefinitions.find((r) => r.id === id).label;
          const key = Object.keys(data).find((k) => normalize(k) === normalize(label));
          nv[id].ysa = data[key] || '0';
        });
        nv.facTotal.ysa = ['fac1', 'fac2', 'fac3']
          .reduce((sum, id) => sum + (parseFloat(nv[id].ysa) || 0), 0)
          .toFixed(2);
      } else {
        // When response is in the form { "code": "0~value" }
        const getYsaValue = (code) => {
          const val = data[code];
          return val?.split('~')[1] || '0';
        };

        // You must know which codes correspond to which fac1/fac2/fac3
        const codeMap = {
          fac1: '090101001052',
          fac2: '090101001051',
          fac3: '090101001053',
        };

        Object.keys(codeMap).forEach((id) => {
          nv[id].ysa = getYsaValue(codeMap[id]);
        });

        nv.facTotal.ysa = ['fac1', 'fac2', 'fac3']
          .reduce((sum, id) => sum + (parseFloat(nv[id].ysa) || 0), 0)
          .toFixed(2);
      }

      return nv;
    });
  } catch (e) {
    console.error(e);
  }
};
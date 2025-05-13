const buildSavePayload = (values, circleCode, quarterEndDate, role, save, status) => {
  const payload = { circleCode, quarterEndDate, role, reportName: 'SC9', save, status };

  Object.entries(values).forEach(([id, fields]) => {
    Object.entries(fields).forEach(([key, val]) => {
      if (['standard', 'subStandard', 'doubtful', 'loss', 'adjustment'].includes(key)) {
        const fieldName = getFieldName(id, key);
        if (fieldName) {
          payload[fieldName] = val || '0';
        }
      }
    });
  });

  return payload;
};
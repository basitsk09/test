const buildPayload = (saveFlag) => {
  const filteredRows = rows.filter((row) => {
    const hasData = Object.keys(initialRow).some(
      (key) => key !== 'isDelete' && row[key] !== initialRow[key] && row[key] !== ''
    );
    return hasData;
  });

  return {
    listToBeSent: filteredRows.map(({ id, isDelete, ...rest }) => rest),
    circleCode: user.circleCode,
    quarterEndDate: user.quarterEndDate,
    userId: user.userId,
    reportName: reportObject.name,
    reportMasterId: reportObject.reportMasterId,
    status: reportObject.status,
    save: saveFlag,
  };
};


if (!filteredRows.length) {
  setSnackbarMessage('No valid data to save.');
  setSnackbarOpen(true);
  return;
}


const addRow = () => {
  const lastRow = rows[rows.length - 1];
  const isLastEmpty = lastRow && Object.values(lastRow).every((val) => val === '' || val === false || val === undefined);

  if (isLastEmpty) {
    showSnackbar('Please fill the last row before adding a new one.', 'warning');
    return;
  }

  setRows((prev) => [...prev, { ...initialRow, id: `new-${Date.now()}-${Math.random()}` }]);
};
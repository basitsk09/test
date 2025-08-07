const handleConfirmAction = async () => {
  // Exit if no row was selected from the table
  if (!selectedRow) return;

  let actionEndpoint = '';
  let successMessage = '';
  let payloadData = []; // This will hold our list

  switch (dialogAction) {
    case 'save':
      actionEndpoint = '/saveWriteOff';
      successMessage = `Data for circle ${selectedRow.circleCode} saved!`;
      break;
    case 'submit':
      actionEndpoint = '/submitWriteOff';
      successMessage = `Data for circle ${selectedRow.circleCode} submitted!`;
      break;
    // ...other cases for accept/reject
    default:
      handleCloseDialog();
      return;
  }

  // ✅ **THIS IS THE KEY PART**
  // It creates a new list 'payloadData' containing just one item:
  // another list with the selected row's details.
  // Example: [['001', '15000.50', '11']]
  payloadData = [[selectedRow.circleCode, selectedRow.amount || '0', selectedRow.status]];

  // The final payload object that will be sent to the API.
  // The 'data' key contains the single-row list created above.
  const payload = {
    circleCode: user.circleCode,
    reportName: reportObject.name,
    userId: user.userId,
    data: payloadData, // <-- The list with the single row is sent here
  };

  try {
    // The API call sends the payload containing only the selected row
    await callApi(actionEndpoint, payload, 'POST');
    setSnackbarMessage(successMessage, 'success');
  } catch (error) {
    console.error(`Error during ${dialogAction}:`, error);
    setSnackbarMessage(`An error occurred while trying to ${dialogAction}.`, 'error');
  } finally {
    handleCloseDialog();
  }
};

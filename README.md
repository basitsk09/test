const buildPayload = (saveFlag) => {
  return {
    listToBeSent: rows.map(({ isDelete, ...rest }) => rest), // remove isDelete before sending
    circleCode: '021',
    quarterEndDate: '31/03/2025',
    userId: '1111111',
    reportName: 'Schedule 9A',
    reportId: null, // or existing if editing
    reportMasterId: '310023',
    status: null,
    save: saveFlag
  };
};

const handleSave = async () => {
  if (!validateRows()) return;

  const payload = buildPayload(true); // save: true

  try {
    const response = await axios.post('/api/schedule9a/saveOrSubmit', payload);
    console.log('Save response:', response.data);
    setSaveStatus('success');
    setSnackbarMessage('Report saved successfully.');
  } catch (error) {
    console.error('Error saving data:', error);
    setSaveStatus('error');
    setSnackbarMessage('Error saving report.');
  } finally {
    setSnackbarOpen(true);
  }
};


const handleSubmit = async () => {
  if (!validateRows()) return;

  const payload = buildPayload(false); // save: false

  try {
    const response = await axios.post('/api/schedule9a/saveOrSubmit', payload);
    console.log('Submit response:', response.data);
    setSubmitStatus('success');
    setSnackbarMessage('Report submitted successfully.');
  } catch (error) {
    console.error('Error submitting data:', error);
    setSubmitStatus('error');
    setSnackbarMessage('Error submitting report.');
  } finally {
    setSnackbarOpen(true);
  }
};
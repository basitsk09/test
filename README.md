const submitData = async () => {
  try {
    const save = confirmDialog.type === 'save';
    const payload = {
      ...data,
      circleCode: '001',
      quarterEndDate: '31/03/2025',
      userId: '1111111',
      reportId: data.reportId || '',
      reportName: 'Sc 9 Supplementary',
      reportMasterId: '310028',
      status: data.status === '0' ? '' : data.status || '',
      save
    };

    const res = await axios.post('/Maker/submitNineSupl', payload);
    
    // Extract new status and reportId from backend response
    if (res.data && typeof res.data === 'string') {
      const [flag, newReportId, newStatus] = res.data.split('~');
      setData(prev => ({
        ...prev,
        reportId: newReportId,
        status: newStatus
      }));
    }

    setSnackbar({
      open: true,
      message: save ? 'Saved Successfully' : 'Submitted Successfully',
      severity: 'success'
    });
  } catch (error) {
    setSnackbar({ open: true, message: 'Operation Failed', severity: 'error' });
  } finally {
    setConfirmDialog({ open: false, type: null });
  }
};
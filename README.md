To implement the SC10 SFTP API call from your latest backend logic, here's how to structure the useEffect in your Schedule 10 React component to match the behavior:


---

✅ API Endpoint Details

URL: https://bsuat.info.sbi/BS/IFAMSS/SC10SFTP
Method: POST
Payload Example:

{
  "circleCode": "001",
  "qed": "31/03/2025",
  "reportID": "310010",
  "reportName": "SC10",
  "reportStatus": "A"
}

Key Response Field: fileAndDataStatus


---

✅ useEffect Implementation

Here’s the exact code to insert into your component:

useEffect(() => {
  const checkSC10SftpData = async () => {
    const payload = {
      circleCode: userData.circleCode,
      qed: userData.quarterEndDate,
      reportID: '310010',
      reportName: 'SC10',
      reportStatus: 'A'
    };

    try {
      const response = await fetch('https://bsuat.info.sbi/BS/IFAMSS/SC10SFTP', {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json'
        },
        body: JSON.stringify(payload)
      });

      const data = await response.json();
      console.log('SFTP response:', data);

      if (data.fileAndDataStatus === 1) {
        // SFTP Success
        setFormData(data?.data || {}); // Assuming `data.data` contains the schedule form values
        setFieldsDisabled(true); // disables all inputs
        showSnackbar('Data successfully fetched from IFAMS via SFTP.', 'success');
      } else if (data.fileAndDataStatus === 2) {
        // File error or mismatch
        setFieldsDisabled(true);
        showDialog({
          title: 'File Error',
          message: data.message || 'Error receiving file from IFAMS.',
          onConfirm: () => navigate(-1) // go back
        });
      } else if (data.fileAndDataStatus === 3) {
        // Data already exists in DB but file was missing
        setFieldsDisabled(true);
        showDialog({
          title: 'Info',
          message: data.message || 'Data already inserted earlier from IFAMS.',
          onConfirm: () => navigate(-1)
        });
      }
    } catch (error) {
      console.error('SC10 SFTP call failed:', error);
      showSnackbar('Error while checking SC10 SFTP data.', 'error');
    }
  };

  checkSC10SftpData();
}, []);


---

✅ Supporting State

Add these:

const [fieldsDisabled, setFieldsDisabled] = useState(false);

In your <FormInput ... /> or <TextField ... />, apply:

disabled={fieldsDisabled}


---

✅ Dialog Utility

If you don’t already have one:

const showDialog = ({ title, message, onConfirm }) => {
  setDialogOpen(true);
  setDialogContent({ title, message, onConfirm });
};

And a generic dialog component like:

<Dialog open={dialogOpen} onClose={() => setDialogOpen(false)}>
  <DialogTitle>{dialogContent.title}</DialogTitle>
  <DialogContent>{dialogContent.message}</DialogContent>
  <DialogActions>
    <Button onClick={() => {
      setDialogOpen(false);
      dialogContent.onConfirm();
    }}>OK</Button>
  </DialogActions>
</Dialog>


---

✅ Summary

You are calling the /SC10SFTP endpoint on useEffect.

Based on fileAndDataStatus:

1: load data and disable input

2/3: show dialog and disable + redirect



Let me know if you want me to inject this code directly into your working component file (Working schedule10.txt).


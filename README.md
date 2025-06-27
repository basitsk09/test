To automatically perform a precheck without showing the dialog on load, and to validate row29 values (totalA, totalB, totalC, premisesUnderCons) against the three backend amounts from the precheck (otherFixedAsset, premises, premisesUnderConstruction), follow this step-by-step solution:


---

✅ Step-by-step Modifications

1. Update useEffect to call handlePreCheck silently

Replace the handlePreCheck call in useEffect with a custom version that:

does not open dialog

validates against row29

applies red border on mismatch

prevents submission if validation fails


Insert this inside the main useEffect after loading the form data:

useEffect(() => {
  const silentPreCheckValidation = async () => {
    try {
      const payload = {
        circleCode: user.circleCode,
        quarterEndDate: user.quarterEndDate,
        status: reportObject.status,
        reportId: reportObject.reportId,
        reportMasterId: reportObject.reportMasterId,
        reportName: reportObject.name,
        userId: user.userId,
        areMocPending: false,
      };

      const response = await callApi('/Maker/getValidationDataTen', payload, 'POST');
      if (response) {
        const validationErrors = {};

        const row29 = formData['row29'];
        if (parseFloat(row29.totalA).toFixed(2) !== parseFloat(response.validationOtherFixedAssetAmount).toFixed(2)) {
          validationErrors['row29-totalA'] = 'Mismatch with precheck';
        }
        if (parseFloat(row29.totalB).toFixed(2) !== parseFloat(response.validationOtherFixedAssetAmount).toFixed(2)) {
          validationErrors['row29-totalB'] = 'Mismatch with precheck';
        }
        if (parseFloat(row29.totalC).toFixed(2) !== parseFloat(response.validationPremisesAmount).toFixed(2)) {
          validationErrors['row29-totalC'] = 'Mismatch with precheck';
        }
        if (
          parseFloat(row29.premisesUnderCons).toFixed(2) !==
          parseFloat(response.validationPremisesUnderConsAmount).toFixed(2)
        ) {
          validationErrors['row29-premisesUnderCons'] = 'Mismatch with precheck';
        }

        setErrors((prev) => ({ ...prev, ...validationErrors }));

        // prevent submission if errors present
        if (Object.keys(validationErrors).length > 0) {
          setSnackbarMessage('Row29 data mismatch with precheck. Please correct the fields.', 'error');
        }
      }
    } catch (error) {
      console.error('Silent precheck error:', error);
      setSnackbarMessage('Failed to run initial validation precheck.', 'error');
    }
  };

  if (!manualEntry && !fieldsDisabled) {
    silentPreCheckValidation();
  }
}, [formData]); // rerun if formData changes after fetch


---

2. Block submission if validation not resolved

In handleSaveOrSubmit, insert this check before calling the API:

const hasValidationErrors = Object.values(errors).some((val) => val && val.length > 0);
if (!isSaveOnly && hasValidationErrors) {
  setSnackbarMessage('Cannot submit. Please fix validation errors in Row 29.', 'error');
  setIsSubmitting(false);
  return;
}


---

✅ Result

On first load, handlePreCheck is done silently.

If values in row29-totalA, totalB, totalC, or premisesUnderCons do not match the expected ones:

Corresponding fields get red borders with error message.

Submission is blocked until the errors are resolved.




---

Let me know if you want to auto-correct the values instead of just validating them.


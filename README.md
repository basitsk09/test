const handleSubmit = async () => {
  setLoading(true);

  // ✅ 1. CHECK IF ANY ROWS ARE SELECTED
  // This is the new check you need.
  if (selected.length === 0) {
    setSnackbar({
      children: "Please select at least one row to submit.",
      severity: "info",
    });
    setLoading(false);
    return;
  }

  // ✅ 2. GET ONLY THE SELECTED ROWS' DATA
  // Filter the main 'rows' array to include only items whose id is in the 'selected' array.
  const selectedRows = rows.filter(row => selected.includes(row.id));


  // 3. Check for validation errors ONLY in the selected rows
  const hasErrors = selectedRows.some(
    (row) => row.branchCode.error || row.auditStatus.error
  );

  if (hasErrors) {
    setSnackbar({
      children: "Please correct errors in the selected rows before submitting.",
      severity: "error",
    });
    setLoading(false);
    return;
  }

  // ✅ 4. BUILD THE PAYLOAD FROM SELECTED ROWS
  // The .map() function now runs on the 'selectedRows' array, not the full 'rows' array.
  const branchCodeList = selectedRows.map((row) =>
    String(row.branchCode.value).padStart(5, "0")
  );
  const statusList = selectedRows.map((row) => row.auditStatus.value);

  const payload = {
    branchCodes: branchCodeList,
    statuses: statusList,
  };

  try {
    const response = await axios.post("/api/v1/frt/bulk-upload", payload, {
      headers: {
        "Content-Type": "application/json",
        // Authorization: `Bearer ${localStorage.getItem("token")}`
      },
    });

    // ... The rest of the function remains the same ...
    const { message, invalidRecords } = response.data;

    if (invalidRecords && invalidRecords.length > 0) {
      const errorMessages = invalidRecords
        .map((rec) => `Branch ${rec.branchCode}: ${rec.reason}`)
        .join("\n");

      setDialog({
        open: true,
        title: "Upload Completed with Errors",
        message: `${message}\n\nValidation Errors:\n${errorMessages}`,
        goNext: false,
      });
    } else {
      setSnackbar({
        children: message || "Request submitted successfully.",
        severity: "success",
      });
    }

    // Refresh the table by removing submitted rows
    const remainingRows = rows.filter(row => !selected.includes(row.id));
    setRows(remainingRows);
    setSelected([]);
    if (remainingRows.length === 0) {
        setShowTable(false);
    }

  } catch (error) {
    console.error("Submission failed:", error);

    if (
      error.response &&
      error.response.data &&
      error.response.data.invalidRecords
    ) {
      const { message, invalidRecords } = error.response.data;
      const errorMessages = invalidRecords
        .map((rec) => `Branch ${rec.branchCode}: ${rec.reason}`)
        .join("\n");
      setDialog({
        open: true,
        title: "Upload Failed",
        message: `${message}\n\nValidation Errors:\n${errorMessages}`,
        goNext: false,
      });
    } else {
      setSnackbar({
        children: "An unexpected error occurred during submission.",
        severity: "error",
      });
    }
  } finally {
    setLoading(false);
  }
};

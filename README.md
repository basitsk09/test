import React, { useState } from "react";
import {
  TextField,
  Grid,
  Button,
  Typography,
  Divider,
  TableContainer,
  Table,
  TableHead,
  TableRow,
  TableCell,
  TableBody,
  Paper,
  Box,
  Container,
  Dialog,
  DialogActions,
  DialogContent,
  DialogContentText,
  DialogTitle,
  CircularProgress,
  Snackbar,
  Alert,
  Checkbox,
  Link,
  List,
  ListItem,
  ListItemIcon,
  ListItemText,
} from "@mui/material";
import DownloadIcon from "@mui/icons-material/CloudDownload";
import UploadIcon from "@mui/icons-material/CloudUpload";
import SubmitIcon from "@mui/icons-material/Save";
import DiscardIcon from "@mui/icons-material/DeleteForever";
import CheckCircleIcon from '@mui/icons-material/CheckCircle';
import { useNavigate } from "react-router-dom";
import * as XLSX from "xlsx";

export default function FrtMultipleBranchAuditStatus() {
  const navigate = useNavigate();

  // --- State Management ---
  const [rows, setRows] = useState([]);
  const [selected, setSelected] = useState([]);
  const [showTable, setShowTable] = useState(false);
  const [loading, setLoading] = useState(false);
  const [snackbar, setSnackbar] = useState(null);
  const [dialog, setDialog] = useState({ open: false, title: "", message: "", goNext: false });
  const [fileInputKey, setFileInputKey] = useState(Date.now()); // Used to reset the file input

  // --- Handlers ---
  const handleSnackbarClose = () => setSnackbar(null);
  const handleDialogClose = () => setDialog({ open: false, title: "", message: "" });
  const goToSingleBranch = () => navigate('/FRTUser/singleBranch'); // Update with your actual route

  /**
   * Validates a single field value.
   * @param {string} field - The name of the field ('branchCode' or 'auditStatus').
   * @param {string} value - The value to validate.
   * @returns {string|null} - An error message string or null if valid.
   */
  const validateField = (field, value) => {
    if (field === "branchCode") {
      if (!/^\d{1,5}$/.test(value)) {
        return "Must be a 5-digit number.";
      }
    }
    if (field === "auditStatus") {
      const validStatuses = ["I", "A", "N"];
      if (!validStatuses.includes(value.toUpperCase())) {
        return "Must be I, A, or N.";
      }
    }
    return null;
  };

  /**
   * Handles the file upload process, including parsing and validation.
   */
  const handleFileUpload = (event) => {
    const file = event.target.files[0];
    if (!file) return;

    setLoading(true);
    // Reset state for new upload
    setRows([]);
    setShowTable(false);

    const validExtensions = [".xlsx", ".xls"];
    const fileExtension = file.name.substring(file.name.lastIndexOf(".")).toLowerCase();

    if (!validExtensions.includes(fileExtension)) {
      setSnackbar({ children: "Invalid file type. Please upload an Excel file.", severity: "error" });
      setLoading(false);
      return;
    }

    if (file.size > 2 * 1024 * 1024) { // 2 MB
      setSnackbar({ children: "File size cannot exceed 2 MB.", severity: "error" });
      setLoading(false);
      return;
    }

    const reader = new FileReader();
    reader.onload = (e) => {
      try {
        const data = e.target.result;
        const workbook = XLSX.read(data, { type: "binary" });
        const sheetName = workbook.SheetNames[0];
        const worksheet = workbook.Sheets[sheetName];
        const json = XLSX.utils.sheet_to_json(worksheet);

        if (!json[0] || !json[0].hasOwnProperty("BranchCode") || !json[0].hasOwnProperty("AuditStatus")) {
          throw new Error("Invalid Excel format. Please use the provided template.");
        }

        if (json.length < 5) {
          setDialog({
            open: true,
            title: "Warning: Low Record Count",
            message: "For bulk uploads, at least 5 records are recommended. Would you like to proceed or use the single branch page?",
            goNext: true
          });
          setLoading(false);
          return;
        }

        const parsedRows = json.map((row, index) => {
          const branchCode = String(row.BranchCode || "").trim();
          const auditStatus = String(row.AuditStatus || "").trim().toUpperCase();
          return {
            id: index,
            branchCode: { value: branchCode, error: validateField("branchCode", branchCode) },
            auditStatus: { value: auditStatus, error: validateField("auditStatus", auditStatus) },
          };
        });
        
        setRows(parsedRows);
        setShowTable(true);
      } catch (err) {
        console.error(err);
        setSnackbar({ children: err.message || "An error occurred while parsing the file.", severity: "error" });
      } finally {
        setLoading(false);
        setFileInputKey(Date.now()); // Reset file input to allow re-uploading the same file
      }
    };
    reader.readAsBinaryString(file);
  };

  /**
   * Handles changes to the text fields within the table for real-time validation.
   */
  const handleInputChange = (id, field, value) => {
    const newRows = rows.map((row) => {
      if (row.id === id) {
        const updatedField = { value, error: validateField(field, value) };
        return { ...row, [field]: updatedField };
      }
      return row;
    });
    setRows(newRows);
  };
  
  /**
   * Handles row selection for the discard functionality.
   */
  const handleSelect = (event, id) => {
    const selectedIndex = selected.indexOf(id);
    let newSelected = [];

    if (selectedIndex === -1) {
      newSelected = newSelected.concat(selected, id);
    } else {
      newSelected = selected.filter(selId => selId !== id);
    }
    setSelected(newSelected);
  };
  
  const handleSelectAll = (event) => {
    if (event.target.checked) {
      const newSelecteds = rows.map((n) => n.id);
      setSelected(newSelecteds);
      return;
    }
    setSelected([]);
  };

  /**
   * Removes the selected rows from the table.
   */
  const handleDiscard = () => {
    if (selected.length === 0) {
      setSnackbar({ children: "Please select rows to discard.", severity: "info" });
      return;
    }
    const newRows = rows.filter((row) => !selected.includes(row.id));
    setRows(newRows);
    setSelected([]);
    setSnackbar({ children: `${selected.length} row(s) discarded.`, severity: "success" });
  };
  
  /**
   * Submits the valid data to the backend.
   */
  const handleSubmit = async () => {
    setLoading(true);
    const hasErrors = rows.some(row => row.branchCode.error || row.auditStatus.error);

    if (hasErrors) {
      setSnackbar({ children: "Please correct all errors before submitting.", severity: "error" });
      setLoading(false);
      return;
    }

    if (rows.length === 0) {
        setSnackbar({ children: "There is no data to submit.", severity: "warning" });
        setLoading(false);
        return;
    }
    
    // The backend expects separate arrays for branchcode and status
    const branchCodeList = rows.map(row => String(row.branchCode.value).padStart(5, '0'));
    const statusList = rows.map(row => row.auditStatus.value);
    
    // The JSP creates a form and submits it. We can replicate this with URLSearchParams for a standard form POST.
    const params = new URLSearchParams();
    branchCodeList.forEach(bc => params.append('branchcode', bc));
    statusList.forEach(st => params.append('status', st));

    try {
        // We expect a file download (for errors) or a redirect with a message (for success).
        // A standard form POST is the best way to handle this ambiguity without complex response parsing.
        const form = document.createElement('form');
        form.method = 'post';
        form.action = '/FRTUser/bulkUpload'; // Your actual backend endpoint

        branchCodeList.forEach(bc => {
            const hiddenField = document.createElement('input');
            hiddenField.type = 'hidden';
            hiddenField.name = 'branchcode';
            hiddenField.value = bc;
            form.appendChild(hiddenField);
        });

        statusList.forEach(st => {
            const hiddenField = document.createElement('input');
            hiddenField.type = 'hidden';
            hiddenField.name = 'status';
            hiddenField.value = st;
            form.appendChild(hiddenField);
        });
        
        document.body.appendChild(form);
        form.submit();
        
        // After submission, we can only provide generic feedback as we don't get a direct response in the SPA.
        setSnackbar({ children: "Request submitted successfully. You will be notified of any invalid records.", severity: "success" });
        // Optionally reset the page after a delay
        setTimeout(() => {
            setRows([]);
            setShowTable(false);
            setSelected([]);
        }, 3000);

    } catch (error) {
        console.error("Submission failed:", error);
        setSnackbar({ children: "An error occurred during submission.", severity: "error" });
    } finally {
        setLoading(false);
    }
  };

  // --- JSX ---
  return (
    <>
      <Box sx={{ p: 2 }}>
        <Typography variant="h5" gutterBottom>
          Change Multiple Branch Audit Status
        </Typography>
        <Divider />
      </Box>

      <Container maxWidth="xl" sx={{ mt: 2 }}>
        <Paper elevation={3} sx={{ p: 3 }}>
            <Typography color="error" gutterBottom>[Note: For bulk upload, at least 5 records should be present in the Excel file.]</Typography>
          <Grid container spacing={4}>
            {/* Left Side: Instructions and Download */}
            <Grid item xs={12} md={6}>
                <Typography variant="h6" gutterBottom>Instructions</Typography>
                <Grid container spacing={2}>
                    <Grid item xs={12} sm={6}>
                        <Typography variant="subtitle1" gutterBottom>1. Download Template</Typography>
                        <Button
                            variant="contained"
                            color="success"
                            startIcon={<DownloadIcon />}
                            href="/FRTUser/downloadExcel" // Endpoint from controller
                        >
                            Download Excel
                        </Button>
                    </Grid>
                    <Grid item xs={12} sm={6}>
                        <Typography variant="subtitle1" gutterBottom>2. Status Guide</Typography>
                        <List dense>
                            <ListItem><ListItemIcon><CheckCircleIcon color="primary" /></ListItemIcon><ListItemText primary="N - For Non-Audited" /></ListItem>
                            <ListItem><ListItemIcon><CheckCircleIcon color="primary" /></ListItemIcon><ListItemText primary="A - For Audited (Non-IFCOFR)" /></ListItem>
                            <ListItem><ListItemIcon><CheckCircleIcon color="primary" /></ListItemIcon><ListItemText primary="I - For IFCOFR Audited" /></ListItem>
                        </List>
                    </Grid>
                </Grid>
            </Grid>

            {/* Right Side: Upload */}
            <Grid item xs={12} md={6}>
                 <Typography variant="h6" gutterBottom>3. Upload File</Typography>
                 <Box sx={{ display: 'flex', alignItems: 'center', gap: 2, mt: 2 }}>
                    <Button variant="contained" component="label">
                        Choose File
                        <input type="file" hidden key={fileInputKey} accept=".xlsx, .xls" onChange={handleFileUpload} />
                    </Button>
                    <Typography variant="body2" color="text.secondary">Max size: 2MB</Typography>
                 </Box>
            </Grid>
          </Grid>
          
          {/* Table Section */}
          {showTable && (
            <Box mt={4}>
                <Divider sx={{ mb: 2 }} />
                <Typography variant="h6" gutterBottom>Verify and Submit Data</Typography>
                <TableContainer component={Paper}>
                    <Table>
                        <TableHead sx={{ backgroundColor: "#b9def0" }}>
                            <TableRow>
                                <TableCell padding="checkbox">
                                    <Checkbox
                                        color="primary"
                                        indeterminate={selected.length > 0 && selected.length < rows.length}
                                        checked={rows.length > 0 && selected.length === rows.length}
                                        onChange={handleSelectAll}
                                    />
                                </TableCell>
                                <TableCell align="center">Branch Code</TableCell>
                                <TableCell align="center">Audit Status (I/A/N)</TableCell>
                            </TableRow>
                        </TableHead>
                        <TableBody>
                            {rows.map((row) => (
                                <TableRow key={row.id} hover selected={selected.indexOf(row.id) !== -1}>
                                    <TableCell padding="checkbox">
                                        <Checkbox
                                            color="primary"
                                            checked={selected.indexOf(row.id) !== -1}
                                            onClick={(event) => handleSelect(event, row.id)}
                                        />
                                    </TableCell>
                                    <TableCell>
                                        <TextField
                                            fullWidth
                                            variant="standard"
                                            value={row.branchCode.value}
                                            onChange={(e) => handleInputChange(row.id, "branchCode", e.target.value)}
                                            error={!!row.branchCode.error}
                                            helperText={row.branchCode.error}
                                            inputProps={{ style: { textAlign: 'center' } }}
                                        />
                                    </TableCell>
                                    <TableCell>
                                         <TextField
                                            fullWidth
                                            variant="standard"
                                            value={row.auditStatus.value}
                                            onChange={(e) => handleInputChange(row.id, "auditStatus", e.target.value)}
                                            error={!!row.auditStatus.error}
                                            helperText={row.auditStatus.error}
                                            inputProps={{ style: { textAlign: 'center', textTransform: 'uppercase' } }}
                                        />
                                    </TableCell>
                                </TableRow>
                            ))}
                        </TableBody>
                    </Table>
                </TableContainer>
                <Box sx={{ display: 'flex', gap: 2, mt: 2 }}>
                    <Button variant="contained" color="primary" startIcon={<SubmitIcon />} onClick={handleSubmit}>Submit</Button>
                    <Button variant="outlined" color="error" startIcon={<DiscardIcon />} onClick={handleDiscard}>Discard Selected</Button>
                </Box>
            </Box>
          )}

        </Paper>
      </Container>
      
      {/* --- Dialogs and Loaders --- */}
      <Dialog open={loading}>
        <DialogContent sx={{ display: "flex", alignItems: "center", gap: 2, p: 4 }}>
          <CircularProgress />
          <Typography>Processing...</Typography>
        </DialogContent>
      </Dialog>
      
      <Dialog open={dialog.open} onClose={handleDialogClose}>
        <DialogTitle>{dialog.title}</DialogTitle>
        <DialogContent><DialogContentText>{dialog.message}</DialogContentText></DialogContent>
        <DialogActions>
            <Button onClick={handleDialogClose}>Close</Button>
            {dialog.goNext && <Button onClick={goToSingleBranch} variant="contained">Go to Single Branch Page</Button>}
        </DialogActions>
      </Dialog>
      
      {snackbar && (
        <Snackbar open autoHideDuration={6000} onClose={handleSnackbarClose} anchorOrigin={{ vertical: 'top', horizontal: 'center' }}>
          <Alert onClose={handleSnackbarClose} severity={snackbar.severity} variant="filled" sx={{ width: "100%" }}>
            {snackbar.children}
          </Alert>
        </Snackbar>
      )}
    </>
  );
}

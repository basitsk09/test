import React, { useState, useEffect } from "react";
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
  RadioGroup,
  Radio,
  FormControlLabel,
  FormControl,
  FormLabel,
  Paper,
  Box,
  Container,
  Dialog,
  DialogContent,
  CircularProgress,
  Snackbar,
  Alert,
} from "@mui/material";
import SearchIcon from "@mui/icons-material/Search";
import DownloadIcon from "@mui/icons-material/CloudDownload";
import SaveIcon from "@mui/icons-material/Save";
import DiscardIcon from "@mui/icons-material/Cancel";
import axios from "axios";
import { useNavigate } from "react-router-dom";

export default function SingleBranchAuditStatus() {
  const navigate = useNavigate();

  // --- State Management ---
  const [branchCode, setBranchCode] = useState("");
  const [branchDetails, setBranchDetails] = useState(null);
  const [initialAuditStatus, setInitialAuditStatus] = useState("");
  const [selectedAuditStatus, setSelectedAuditStatus] = useState("");
  const [showDetails, setShowDetails] = useState(false);
  const [userRole, setUserRole] = useState("");
  const [loading, setLoading] = useState(false);
  const [snackbar, setSnackbar] = useState(null);

  // This effect runs once to get the user role from localStorage
  useEffect(() => {
    const loggedInUser = JSON.parse(localStorage.getItem("user"));
    if (loggedInUser && loggedInUser.user_role) {
      // Assuming '96' and '94' are the roles from the JSP/Controller logic
      if (loggedInUser.user_role !== "96" && loggedInUser.user_role !== "94") {
        navigate("/"); // Redirect if role is not authorized
      }
      setUserRole(loggedInUser.user_role);
    } else {
      navigate("/"); // Redirect if no user is logged in
    }
  }, [navigate]);

  // --- Handlers ---
  const handleSnackbarClose = () => setSnackbar(null);

  /**
   * Clears the form and resets all state variables to their initial values.
   */
  const handleReset = () => {
    setBranchCode("");
    setBranchDetails(null);
    setShowDetails(false);
    setSelectedAuditStatus("");
    setInitialAuditStatus("");
  };

  /**
   * Fetches branch details from the server based on the entered branch code.
   */
  const handleSearch = async () => {
    if (!branchCode || branchCode.length < 5) {
      setSnackbar({
        children: "Please enter a valid 5-digit branch code.",
        severity: "error",
      });
      return;
    }
    setLoading(true);
    handleReset(); // Reset previous results before new search
    setBranchCode(branchCode); // Keep the entered branch code after reset

    try {
      // The JSP serializes the form, which sends a key-value pair.
      // We replicate this with a URLSearchParams object.
      const params = new URLSearchParams();
      params.append('branchCode', branchCode);

      const response = await axios.post("/FRTUser/searchBranchCode", params);

      if (response.data && response.data.length > 0) {
        const details = response.data[0];
        setBranchDetails(details);

        // Determine the initial combined audit status based on the logic in the JSP
        let status = 'N'; // Default to Non-Audited
        if (details.auditFlag === 'Y') {
            status = 'A'; // Audited
            if (details.ifcofrFlag === 'Y') {
                status = 'I'; // IFCOOR Audited
            }
        }
        setInitialAuditStatus(status);
        setSelectedAuditStatus(status);
        setShowDetails(true);
      } else {
        setSnackbar({
          children: "Branch does not exist.",
          severity: "warning",
        });
      }
    } catch (error) {
      console.error("Search failed:", error);
      setSnackbar({
        children: "An error occurred while searching. Please try again.",
        severity: "error",
      });
    } finally {
      setLoading(false);
    }
  };

  /**
   * Submits the changed audit status to the server.
   */
  const handleSave = async () => {
    if (selectedAuditStatus === initialAuditStatus) {
        setSnackbar({
            children: "There is no change in audit status to save.",
            severity: "info",
        });
        return;
    }

    setLoading(true);
    const payload = {
        branchCode: branchDetails.branchCode,
        branchName: branchDetails.branchName,
        requestId: branchDetails.requestId,
        circleCode: branchDetails.circleName,
        roCode: branchDetails.roDetails,
        auditSts: selectedAuditStatus,
    };

    try {
        const response = await axios.post("/FRTUser/saveMe", payload);
        if (response.data && response.data !== 'NODATA') {
            setSnackbar({ children: response.data, severity: "success" });
            handleReset(); // Clear form on success
        } else {
            throw new Error('Failed to save data.');
        }
    } catch (error) {
        console.error("Save failed:", error);
        setSnackbar({ children: "The request could not be saved due to a technical error.", severity: "error" });
    } finally {
        setLoading(false);
    }
  };

    /**
     * Handles the download request by submitting a hidden form.
     * This is a reliable way to trigger POST-based file downloads in the browser.
     */
    const handleDownload = () => {
        const form = document.createElement('form');
        form.method = 'post';
        form.action = '/FRTUser/downloadForm';
        document.body.appendChild(form);
        form.submit();
        document.body.removeChild(form);
    };


  // --- JSX ---
  return (
    <>
      <Box sx={{ p: 2 }}>
        <Typography variant="h5" gutterBottom>
          View/Update Branch Audit Status
        </Typography>
        <Divider />
      </Box>

      <Container maxWidth="xl" sx={{ mt: 2 }}>
        <Paper elevation={3} sx={{ p: 3 }}>
          <Grid container spacing={3} alignItems="center">
            {/* Search and Download Section */}
            <Grid item xs={12} md={3}>
              <TextField
                label="Enter Branch Code"
                variant="outlined"
                fullWidth
                value={branchCode}
                onChange={(e) => setBranchCode(e.target.value)}
                inputProps={{ maxLength: 5 }}
              />
            </Grid>
            <Grid item xs={12} md={2}>
              <Button
                variant="contained"
                startIcon={<SearchIcon />}
                onClick={handleSearch}
                fullWidth
              >
                Search
              </Button>
            </Grid>
            <Grid item xs={12} md={4} sx={{textAlign: {xs: 'left', md: 'right'} }}>
                <Typography variant="body1">
                    List Of All Branches With Audit Status:
                </Typography>
            </Grid>
            <Grid item xs={12} md={3}>
                <Button variant="contained" color="secondary" startIcon={<DownloadIcon />} onClick={handleDownload} fullWidth>
                    Download List
                </Button>
            </Grid>
          </Grid>

          {/* Details and Actions Section - visible after search */}
          {showDetails && (
            <Box mt={4}>
              <Typography variant="h6" gutterBottom>
                Existing Branch Audit Status
              </Typography>
              <TableContainer component={Paper} elevation={2}>
                <Table>
                  <TableHead sx={{ backgroundColor: "#b9def0" }}>
                    <TableRow>
                      <TableCell align="center">Branch</TableCell>
                      <TableCell align="center">Name</TableCell>
                      <TableCell align="center">Circle</TableCell>
                      <TableCell align="center">RO Details</TableCell>
                      <TableCell align="center">Audited Status</TableCell>
                      <TableCell align="center">IFCOFR</TableCell>
                    </TableRow>
                  </TableHead>
                  <TableBody>
                    <TableRow>
                      <TableCell align="center">{branchDetails?.branchCode}</TableCell>
                      <TableCell align="center">{branchDetails?.branchName}</TableCell>
                      <TableCell align="center">{branchDetails?.circleName}</TableCell>
                      <TableCell align="center">{branchDetails?.roDetails}</TableCell>
                      <TableCell align="center">{branchDetails?.auditFlag === 'Y' ? 'Audited' : 'Non-Audited'}</TableCell>
                      <TableCell align="center">{branchDetails?.ifcofrFlag === 'Y' ? 'Yes' : (branchDetails?.ifcofrFlag === 'N' ? 'No' : '--')}</TableCell>
                    </TableRow>
                  </TableBody>
                </Table>
              </TableContainer>

              {/* Radio Button Section - visible only for Maker role */}
              {userRole === "96" && (
                <Box mt={4}>
                  <Grid container spacing={2}>
                    <Grid item xs={12}>
                        <FormControl component="fieldset">
                            <FormLabel component="legend">Update Audit Status</FormLabel>
                            <RadioGroup row value={selectedAuditStatus.charAt(0)} onChange={(e) => setSelectedAuditStatus(e.target.value)}>
                                <FormControlLabel value="N" control={<Radio />} label="Non-Audited Branch" />
                                <FormControlLabel value="A" control={<Radio />} label="Audited Branch" />
                            </RadioGroup>
                        </FormControl>
                    </Grid>
                    
                    {/* IFCOFR Section - visible only when 'Audited' is selected */}
                    {selectedAuditStatus.charAt(0) === 'A' && (
                         <Grid item xs={12}>
                            <FormControl component="fieldset">
                                <FormLabel component="legend">IFCOFR Audit</FormLabel>
                                <RadioGroup row value={selectedAuditStatus} onChange={(e) => setSelectedAuditStatus(e.target.value)}>
                                    <FormControlLabel value="I" control={<Radio />} label="Yes (IFCOFR Audited)" />
                                    <FormControlLabel value="A" control={<Radio />} label="No" />
                                </RadioGroup>
                            </FormControl>
                        </Grid>
                    )}
                  </Grid>

                  {/* Save/Discard Buttons */}
                  <Box mt={3} display="flex" gap={2}>
                    <Button variant="contained" color="success" startIcon={<SaveIcon />} onClick={handleSave}>
                      Save Changes
                    </Button>
                    <Button variant="contained" color="error" startIcon={<DiscardIcon />} onClick={handleReset}>
                      Discard
                    </Button>
                  </Box>
                </Box>
              )}
            </Box>
          )}
        </Paper>
      </Container>

      {/* Loading Dialog */}
      <Dialog open={loading}>
        <DialogContent sx={{ display: 'flex', alignItems: 'center', gap: 2 }}>
          <CircularProgress />
          <Typography>Processing...</Typography>
        </DialogContent>
      </Dialog>

      {/* Snackbar for Notifications */}
      {snackbar && (
        <Snackbar
          open
          anchorOrigin={{ vertical: "top", horizontal: "center" }}
          onClose={handleSnackbarClose}
          autoHideDuration={6000}
        >
          <Alert onClose={handleSnackbarClose} severity={snackbar.severity} variant="filled" sx={{ width: "100%" }}>
            {snackbar.children}
          </Alert>
        </Snackbar>
      )}
    </>
  );
}

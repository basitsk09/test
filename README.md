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
import { encrypt } from "../Security/AES-GCM256";

// Encryption constants
const iv = crypto.getRandomValues(new Uint8Array(12));
const ivBase64 = btoa(String.fromCharCode.apply(null, iv));
const salt = crypto.getRandomValues(new Uint8Array(16));
const saltBase64 = btoa(String.fromCharCode.apply(null, salt));

export default function FrtSingleBranchAuditStatus() {
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

  useEffect(() => {
    const loggedInUser = JSON.parse(localStorage.getItem("user"));
    if (loggedInUser && loggedInUser.user_role) {
      if (loggedInUser.user_role !== "96" && loggedInUser.user_role !== "94") {
        navigate("/");
      }
      setUserRole(loggedInUser.user_role);
    } else {
      navigate("/");
    }
  }, [navigate]);

  // --- Handlers ---
  const handleSnackbarClose = () => setSnackbar(null);

  const handleReset = () => {
    setBranchCode("");
    setBranchDetails(null);
    setShowDetails(false);
    setSelectedAuditStatus("");
    setInitialAuditStatus("");
  };

  /**
   * Fetches branch details from the server based on the entered branch code.
   * Updated to use the new payload structure.
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
    handleReset(); 
    setBranchCode(branchCode); 

    try {
      let jsonFormData = JSON.stringify({ branchCode });
      jsonFormData = await encrypt(iv, salt, jsonFormData);
      let payload = { iv: ivBase64, salt: saltBase64, data: jsonFormData };

      const response = await axios.post(
        "/Server/EditBranch/fetchBranchDetails",
        payload,
        {
          headers: { Authorization: `Bearer ${localStorage.getItem("token")}` },
        }
      );

      if (
        response.data?.result?.branchData &&
        response.data.result.branchData.CODE
      ) {
        const details = response.data.result.branchData;
        setBranchDetails(details);
        
        // Directly use the AUDITABLE field which can be 'A', 'N', or 'I'
        const status = details.AUDITABLE || "N";
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
   * Updated to map the new state structure to the expected save payload.
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
    // Construct the RO Details string from network, module, and region
    const roDetails = `${branchDetails.NETWORK || ''} ${branchDetails.MODULE || ''} ${branchDetails.REGION || ''}`.trim();
    
    // Create the payload expected by the '/FRTUser/saveMe' endpoint
    const savePayload = {
      branchCode: branchDetails.CODE,
      branchName: branchDetails.NAME,
      requestId: branchDetails.requestId, // This might be undefined, which the backend handles as an insert
      circleCode: branchDetails.CIRCLE,
      roCode: roDetails,
      auditSts: selectedAuditStatus,
    };

    try {
      // NOTE: This assumes the save endpoint and its payload structure remain the same.
      const response = await axios.post("/FRTUser/saveMe", savePayload);
      if (response.data && response.data !== "NODATA") {
        setSnackbar({ children: response.data, severity: "success" });
        handleReset();
      } else {
        throw new Error("Failed to save data.");
      }
    } catch (error) {
      console.error("Save failed:", error);
      setSnackbar({
        children: "The request could not be saved due to a technical error.",
        severity: "error",
      });
    } finally {
      setLoading(false);
    }
  };

  const handleDownload = () => {
    const form = document.createElement("form");
    form.method = "post";
    form.action = "/FRTUser/downloadForm";
    document.body.appendChild(form);
    form.submit();
    document.body.removeChild(form);
  };

  // --- Helper to format RO Details ---
  const formatRoDetails = (details) => {
    if (!details) return '--';
    return `${details.NETWORK || '---'} ${details.MODULE || '---'} ${details.REGION || '---'}`;
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
            <Grid
              item
              xs={12}
              md={4}
              sx={{ textAlign: { xs: "left", md: "right" } }}
            >
              <Typography variant="body1">
                List Of All Branches With Audit Status:
              </Typography>
            </Grid>
            <Grid item xs={12} md={3}>
              <Button
                variant="contained"
                color="secondary"
                startIcon={<DownloadIcon />}
                onClick={handleDownload}
                fullWidth
              >
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
                      <TableCell align="center">{branchDetails?.CODE}</TableCell>
                      <TableCell align="center">{branchDetails?.NAME}</TableCell>
                      <TableCell align="center">{branchDetails?.CIRCLE}</TableCell>
                      <TableCell align="center">{formatRoDetails(branchDetails)}</TableCell>
                      <TableCell align="center">{branchDetails?.AUDITABLE === "N" ? "Non-Audited" : "Audited"}</TableCell>
                      <TableCell align="center">{branchDetails?.AUDITABLE === "I" ? "Yes" : (branchDetails?.AUDITABLE === "A" ? "No" : "--")}</TableCell>
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
                        <FormLabel component="legend">
                          Update Audit Status
                        </FormLabel>
                        <RadioGroup
                          row
                          value={selectedAuditStatus.charAt(0) === 'I' ? 'A' : selectedAuditStatus.charAt(0)}
                          onChange={(e) => setSelectedAuditStatus(e.target.value)}
                        >
                          <FormControlLabel
                            value="N"
                            control={<Radio />}
                            label="Non-Audited Branch"
                          />
                          <FormControlLabel
                            value="A"
                            control={<Radio />}
                            label="Audited Branch"
                          />
                        </RadioGroup>
                      </FormControl>
                    </Grid>

                    {/* IFCOFR Section - visible only when 'Audited' is selected */}
                    {selectedAuditStatus.charAt(0) !== 'N' && (
                      <Grid item xs={12}>
                        <FormControl component="fieldset">
                          <FormLabel component="legend">IFCOFR Audit</FormLabel>
                          <RadioGroup
                            row
                            value={selectedAuditStatus}
                            onChange={(e) =>
                              setSelectedAuditStatus(e.target.value)
                            }
                          >
                            <FormControlLabel
                              value="I"
                              control={<Radio />}
                              label="Yes (IFCOFR Audited)"
                            />
                            <FormControlLabel
                              value="A"
                              control={<Radio />}
                              label="No"
                            />
                          </RadioGroup>
                        </FormControl>
                      </Grid>
                    )}
                  </Grid>

                  {/* Save/Discard Buttons */}
                  <Box mt={3} display="flex" gap={2}>
                    <Button
                      variant="contained"
                      color="success"
                      startIcon={<SaveIcon />}
                      onClick={handleSave}
                    >
                      Save Changes
                    </Button>
                    <Button
                      variant="contained"
                      color="error"
                      startIcon={<DiscardIcon />}
                      onClick={handleReset}
                    >
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
        <DialogContent sx={{ display: "flex", alignItems: "center", gap: 2 }}>
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
          <Alert
            onClose={handleSnackbarClose}
            severity={snackbar.severity}
            variant="filled"
            sx={{ width: "100%" }}
          >
            {snackbar.children}
          </Alert>
        </Snackbar>
      )}
    </>
  );
}

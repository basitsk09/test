import React, { useState } from "react";
import {
  TextField,
  Grid,
  Button,
  Typography,
  Divider,
} from "@mui/material";
import { Container } from "@mui/system";
import DialogContent from "@mui/material/DialogContent";
import Dialog from "@mui/material/Dialog";
import axios from "axios";
import Box from "@mui/material/Box";
import { useNavigate } from "react-router-dom";
import { encrypt } from "../Security/AES-GCM256";
import SearchIcon from "@mui/icons-material/Search";
import CircularProgress from "@mui/material/CircularProgress";
import Snackbar from "@mui/material/Snackbar";
import Alert from "@mui/material/Alert";
import { SnackbarProvider } from "notistack";
import { Cached } from "@mui/icons-material";
import { validations } from "../CommonValidations/Validations";

// Encryption constants are kept for API communication
const iv = crypto.getRandomValues(new Uint8Array(12));
const ivBase64 = btoa(String.fromCharCode.apply(null, iv));
const salt = crypto.getRandomValues(new Uint8Array(16));
const saltBase64 = btoa(String.fromCharCode.apply(null, salt));

export default function BranchAuditorDetails() {
  const navigate = useNavigate();
  const user = JSON.parse(localStorage.getItem("user"));

  // Simplified state for read-only auditor data
  const emptyAuditorMap = {
    NAME: "",
    TYPE: "",
    MEMNO: "",
    FIRMNO: "",
    FIRMNAME: "",
    ADDR: "",
    CITY: "",
    POST: "",
    EMAIL: "",
    PHONE: "",
  };

  // User role check from original component
  if (user.user_role !== "94" && user.user_role !== "96") {
    navigate("/");
  }

  const [auditorData, setAuditorData] = useState(emptyAuditorMap);
  const [loadOpen, setLoadOpen] = useState(false);
  const [snackbar, setSnackbar] = useState(null);
  const [branchCode, setBranchCode] = useState("");
  const [error, setError] = useState(false);

  /**
   * This function is for closing the snackbar.
   */
  const handleCloseSnackbar = () => setSnackbar(null);

  /**
   * This function is for closing the loading icon.
   */
  const handleDialogClose = () => setLoadOpen(false);

  /**
   * This function is for handling the branch code input.
   */
  const handleBranchCodeChange = (e) => {
    setError(false);
    // Using the same validation as the original component
    let result = validations("numInput", e.target.value);
    if (result === "") {
      setBranchCode(e.target.value);
    } else {
      setSnackbar({ children: result, severity: "error" });
    }
  };

  /**
   * This function is for resetting all values.
   */
  const handleReset = () => {
    setAuditorData(emptyAuditorMap);
    setBranchCode("");
    setError(false);
  };

  /**
   * This function is an axios call to get the auditor data for the respective branch code.
   */
  const handleSearch = async () => {
    if (!branchCode || branchCode.length < 5) {
      setError(true);
      setSnackbar({
        children: "Please enter a valid 5-digit branch code.",
        severity: "error",
      });
      return;
    }

    setLoadOpen(true);
    setAuditorData(emptyAuditorMap); // Clear previous results

    try {
      let jsonFormData = JSON.stringify({ branchCode: branchCode });
      jsonFormData = await encrypt(iv, salt, jsonFormData);

      let payload = {
        iv: ivBase64,
        salt: saltBase64,
        data: jsonFormData,
      };

      const response = await axios.post(
        "/Server/EditBranch/fetchBranchDetails",
        payload,
        {
          headers: {
            Authorization: `Bearer ${localStorage.getItem("token")}`,
          },
        }
      );

      // Check if branch exists and has auditor data
      if (
        response.data?.result?.auditorData &&
        Object.keys(response.data.result.auditorData).length > 0
      ) {
        setAuditorData(response.data.result.auditorData);
        setSnackbar({
          children: "Auditor details fetched successfully.",
          severity: "success",
        });
      } else if (response.data?.result?.branchData) {
        // Branch exists but no auditor is assigned
        setSnackbar({
          children: "Branch found, but no auditor is assigned to it.",
          severity: "info",
        });
      } else {
        // Branch not found in either branch_master or CBS
        setSnackbar({
          children:
            "Branch does not exist. Please contact 'Finance One Core Team'",
          severity: "error",
        });
      }
    } catch (e) {
      console.error(e);
      setSnackbar({
        children: "An error occurred. Please try again later.",
        severity: "error",
      });
    } finally {
      handleDialogClose();
    }
  };

  return (
    <>
      <Box sx={{ display: "flex", height: 50, alignItems: "bottom" }}>
        <Typography
          variant="h5"
          gutterBottom
          sx={{ textAlign: "flex-start", p: 2 }}
        >
          Branch Auditor Details
        </Typography>
      </Box>
      <Divider />
      <Container
        maxWidth={false}
        disableGutters
        sx={{
          display: "flex",
          justifyContent: "center",
          alignItems: "center",
          flexDirection: "column",
          mt: 2,
        }}
      >
        <Box
          sx={{
            width: "100%",
            border: "1px solid #ddd",
            borderRadius: "8px",
            padding: 4,
            backgroundColor: "#fff",
          }}
        >
          <Grid container spacing={2} alignItems="center">
            {/* Search Bar */}
            <Grid item xs={12} sm={5}>
              <TextField
                label="Branch Code"
                name="CODE"
                error={error}
                helperText={error && "This field is required."}
                onChange={handleBranchCodeChange}
                value={branchCode}
                variant="outlined"
                fullWidth
                inputProps={{
                  maxLength: 5,
                }}
              />
            </Grid>
            <Grid item xs={12} sm={4}>
              <Button
                variant="contained"
                disableElevation
                startIcon={<SearchIcon />}
                onClick={handleSearch}
              >
                Search
              </Button>
              &nbsp;
              <Button
                variant="contained"
                color="warning"
                disableElevation
                startIcon={<Cached />}
                onClick={handleReset}
              >
                Reset
              </Button>
            </Grid>

            {/* Conditionally render Auditor Details only if found */}
            {auditorData.NAME && (
              <>
                <Grid item xs={12}>
                    <Typography variant="h6" sx={{ mt: 3, mb: 1, borderBottom: '1px solid #ccc', pb: 1 }}>
                        Auditor Information
                    </Typography>
                </Grid>

                <Grid item xs={12} sm={6}>
                  <TextField
                    label="Name of the Auditor"
                    value={auditorData.NAME}
                    disabled={true}
                    variant="outlined"
                    fullWidth
                  />
                </Grid>
                <Grid item xs={12} sm={6}>
                  <TextField
                    label="Type of Firm"
                    value={auditorData.TYPE}
                    disabled={true}
                    variant="outlined"
                    fullWidth
                  />
                </Grid>
                <Grid item xs={12} sm={6}>
                  <TextField
                    label="Auditor Membership No"
                    value={auditorData.MEMNO}
                    disabled={true}
                    variant="outlined"
                    fullWidth
                  />
                </Grid>
                <Grid item xs={12} sm={6}>
                  <TextField
                    label="Firm Registration No"
                    value={auditorData.FIRMNO}
                    disabled={true}
                    variant="outlined"
                    fullWidth
                  />
                </Grid>
                <Grid item xs={12} sm={6}>
                  <TextField
                    label="Firm Name"
                    value={auditorData.FIRMNAME}
                    disabled={true}
                    variant="outlined"
                    fullWidth
                  />
                </Grid>
                <Grid item xs={12} sm={6}>
                  <TextField
                    label="Firm's Address line 1"
                    value={auditorData.ADDR}
                    disabled={true}
                    variant="outlined"
                    fullWidth
                  />
                </Grid>
                <Grid item xs={12} sm={6}>
                  <TextField
                    label="Firm's City"
                    value={auditorData.CITY}
                    disabled={true}
                    variant="outlined"
                    fullWidth
                  />
                </Grid>
                <Grid item xs={12} sm={6}>
                  <TextField
                    label="Post Code"
                    value={auditorData.POST}
                    disabled={true}
                    variant="outlined"
                    fullWidth
                  />
                </Grid>
                <Grid item xs={12} sm={6}>
                  <TextField
                    label="Email of the Audit Firm"
                    value={auditorData.EMAIL}
                    disabled={true}
                    variant="outlined"
                    fullWidth
                  />
                </Grid>
                <Grid item xs={12} sm={6}>
                  <TextField
                    label="Phone No of the Audit Firm"
                    value={auditorData.PHONE}
                    disabled={true}
                    variant="outlined"
                    fullWidth
                  />
                </Grid>
              </>
            )}
          </Grid>
        </Box>
      </Container>

      {/* Loading Spinner Dialog */}
      <Dialog
        PaperProps={{
          style: {
            backgroundColor: "transparent",
            boxShadow: "none",
          },
        }}
        open={loadOpen}
      >
        <DialogContent>
          <CircularProgress />
        </DialogContent>
      </Dialog>

      {/* Snackbar for notifications */}
      {snackbar && (
        <SnackbarProvider maxSnack={3}>
          <Snackbar
            open
            anchorOrigin={{ vertical: "top", horizontal: "center" }}
            onClose={handleCloseSnackbar}
            autoHideDuration={5000}
          >
            <Alert
              variant="filled"
              {...snackbar}
              onClose={handleCloseSnackbar}
            />
          </Snackbar>
        </SnackbarProvider>
      )}
    </>
  );
}

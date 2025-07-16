import React, { useState } from "react";
import {
  TextField,
  Grid,
  MenuItem,
  FormControl,
  Button,
  Typography,
  DialogTitle,
  Divider,
  TableContainer,
  Table,
  TableHead,
  TableRow,
  TableCell,
  TableBody,
} from "@mui/material";
import PhoneIcon from "@mui/icons-material/Phone";
import HomeIcon from "@mui/icons-material/Home";
import PinDropIcon from "@mui/icons-material/PinDrop";
import AccountBalanceRoundedIcon from "@mui/icons-material/AccountBalanceRounded";
import LocationCityRoundedIcon from "@mui/icons-material/LocationCityRounded";
import PhoneAndroidRoundedIcon from "@mui/icons-material/PhoneAndroidRounded";
import CorporateFareRoundedIcon from "@mui/icons-material/CorporateFareRounded";
import WifiCalling3Icon from "@mui/icons-material/WifiCalling3";
import EmergencyShareIcon from "@mui/icons-material/EmergencyShare";
import DnsIcon from "@mui/icons-material/Dns";
import PublicIcon from "@mui/icons-material/Public";
import SensorsIcon from "@mui/icons-material/Sensors";
import LanIcon from "@mui/icons-material/Lan";
import { Container } from "@mui/system";
import DialogContent from "@mui/material/DialogContent";
import DialogContentText from "@mui/material/DialogContentText";
import DialogActions from "@mui/material/DialogActions";
import Dialog from "@mui/material/Dialog";
import axios from "axios";
import Box from "@mui/material/Box";
import { validations } from "../CommonValidations/Validations";
import { useNavigate } from "react-router-dom";
import { encrypt } from "../Security/AES-GCM256";
import SaveIcon from "@mui/icons-material/Save";
import DeleteIcon from "@mui/icons-material/Delete";
import SearchIcon from "@mui/icons-material/Search";
import CircularProgress from "@mui/material/CircularProgress";
import Snackbar from "@mui/material/Snackbar";
import Alert from "@mui/material/Alert";
import { SnackbarProvider } from "notistack";
import Paper from "@mui/material/Paper";
import PersonIcon from "@mui/icons-material/Person";
import { Cached } from "@mui/icons-material";
import {
  AUDITED_REPORT_STATUS_CONSTANTS,
  REPORT_STATUS_CONSTANTS,
} from "../CommonValidations/commonConstants";

const iv = crypto.getRandomValues(new Uint8Array(12)); // for encryption
const ivBase64 = btoa(String.fromCharCode.apply(null, iv)); // for be decryption
const salt = crypto.getRandomValues(new Uint8Array(16)); // for encryption
const saltBase64 = btoa(String.fromCharCode.apply(null, salt)); // for be decryption

export default function FrtMakerDeleteBranchDetails() {
  const navigate = useNavigate();
  const user = JSON.parse(localStorage.getItem("user"));

  const emptyBranchDataMap = {
    NAME: "",
    CIRCLE: "",
    NETWORK: "",
    MODULE: "",
    REGION: "",
    ADDRESS: "",
    CITY: "",
    STATE: "",
    PIN: "",
    STDCODE: "",
    PHONE: "",
    MOBILE: "",
    IPPHONE: "",
    AUDITABLE: "",
    SCOPE: "",
  };

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

  if (user.user_role !== "94" && user.user_role !== "96") {
    navigate("/");
  }
  const [branchDetailErrors, setBranchDetailErrors] = useState({});
  const [openWarningDialog, setOpenWarningDialog] = useState(false);
  const [fetchedData, setFetchedData] = useState({});
  const [error, setError] = useState(false);
  const [branchData, setbranchData] = useState(emptyBranchDataMap);
  const [auditorData, setAuditorData] = useState(emptyAuditorMap);
  const [loadOpen, setLoadOpen] = useState(false);
  const [snackbar, setSnackbar] = useState(null);
  const [branchCode, setBranchCode] = useState("");
  const [circleList, setCircleList] = useState([]);
  const [showDeleteConfirm, setShowDeleteConfirm] = useState(false);
  const [reportsList, setReportsList] = useState([]);
  const [fieldsDisabled, setFieldsDisabled] = useState(true);

  const handleCloseSnackbar = () => setSnackbar(null);
  const handleDialogClose = () => setLoadOpen(false);

  const handleBranchCodeChange = (e) => {
    setError(false);
    let result = validations("numInput", e.target.value);
    if (result === "") {
      setBranchCode(e.target.value);
    } else {
      setSnackbar({ children: result, severity: "error" });
    }
  };

  const handleInputChange = (event) => {
    const { name, value } = event.target;
    const error = validateInputFields(name, value);
    setBranchDetailErrors({ ...branchDetailErrors, [name]: error });
    setbranchData({ ...branchData, [name]: value });
  };

  const validateInputFields = (name, value) => {
    let error = "";
    if (!value) {
      error = "This field is required.";
    } else {
      switch (name) {
        case "MOBILE":
          error = validations("mobileNumber", value);
          break;
        case "PIN":
          error = validations("postCode", value);
          break;
        case "ADDRESS":
        case "CITY":
        case "STATE":
          error = validations("splAlphaNumeric", value);
          break;
        case "MODULE":
        case "NETWORK":
        case "REGION":
          if (!/^\d{3}$/.test(value)) {
            error = "Should be 3 digits only";
          }
          break;
        case "IPPHONE":
        case "PHONE":
          error = validations("numInput", value);
          break;
        case "STDCODE":
          if (!/^\d{2,5}$/.test(value)) {
            error = "STD Code must be between 2 to 5 digits";
          }
          break;
        default:
          break;
      }
    }
    return error;
  };

  const handleReset = () => {
    setbranchData(emptyBranchDataMap);
    setFetchedData({});
    setAuditorData(emptyAuditorMap);
    setCircleList([]);
    setBranchCode("");
    setFieldsDisabled(true);
    setReportsList([]);
    setBranchDetailErrors({});
  };

  const checkIfChanged = () => {
    return Object.keys(branchData).some(
      (key) => branchData[key] !== fetchedData[key]
    );
  };

  const handleSearch = async () => {
    if (branchCode.length < 5) {
      setSnackbar({
        children: "Kindly enter branch code upto 5 digits.",
        severity: "error",
      });
      return;
    }
    setLoadOpen(true);
    try {
      let jsonFormData = JSON.stringify({ branchCode: branchCode });
      await encrypt(iv, salt, jsonFormData).then((r) => (jsonFormData = r));
      let payload = { iv: ivBase64, salt: saltBase64, data: jsonFormData };

      const response = await axios.post(
        "/Server/EditBranch/fetchBranchDetails",
        payload,
        {
          headers: { Authorization: `Bearer ${localStorage.getItem("token")}` },
        }
      );

      if (
        response.data?.result &&
        Object.keys(response.data?.result?.branchData).length !== 0
      ) {
        setCircleList(response.data.result.circleList);
        setbranchData(response.data.result.branchData);
        setFetchedData({ ...response.data.result.branchData });
        setAuditorData(response.data.result.auditorData);
        setFieldsDisabled(false);
      } else {
        setSnackbar({
          children:
            "Branch does not exist. Please contact 'Finance One Core Team'",
          severity: "error",
        });
        handleReset();
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

  /**
   * Fetches reports for the current branch to show in the delete confirmation dialog.
   */
  const fetchReportsForDelete = async () => {
    setLoadOpen(true);
    try {
      let jsonFormData = JSON.stringify({ branchCode: branchCode });
      await encrypt(iv, salt, jsonFormData).then((r) => (jsonFormData = r));
      let payload = { iv: ivBase64, salt: saltBase64, data: jsonFormData };

      const response = await axios.post(
        "/Server/EditBranch/fetchReports",
        payload,
        {
          headers: { Authorization: `Bearer ${localStorage.getItem("token")}` },
        }
      );

      if (response.data?.result) {
        setReportsList(response.data.result.reportList || []);
        setShowDeleteConfirm(true); // Open dialog even if no reports exist
      } else {
        setSnackbar({
          children: "Failed to fetch reports data.",
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

  /**
   * Handles the click of the main 'Save' button.
   */
  const handleSubmit = (event) => {
    event.preventDefault();
    const errors = {};
    Object.keys(branchData).forEach((field) => {
      const error = validateInputFields(field, branchData[field]);
      if (error) errors[field] = error;
    });

    setBranchDetailErrors(errors);

    if (Object.keys(errors).length !== 0) {
      setSnackbar({
        children: "Kindly make sure all fields are filled.",
        severity: "error",
      });
    } else if (!checkIfChanged()) {
      setOpenWarningDialog(true);
    } else {
      setLoadOpen(true);
      handleConfirmSubmit(branchData);
    }
  };

  /**
   * Resets all reports for a branch before deletion. Returns true on success.
   */
  const resetAllReports = async () => {
    if (reportsList.length === 0) return true;
    let successCount = 0;
    for (const report of reportsList) {
      try {
        let data = {
          submissionId: report.SUBMISSION_ID,
          reportId: report.REPORT_ID,
          reportType: report.REPORT_TYPE,
          module: report.MODULE,
          method: "resetAll",
        };
        let jsonFormData = JSON.stringify(data);
        await encrypt(iv, salt, jsonFormData).then((r) => (jsonFormData = r));
        let payload = { iv: ivBase64, salt: saltBase64, data: jsonFormData };

        const response = await axios.post(
          "/Server/EditBranch/resetReportsForDeletion",
          payload,
          {
            headers: {
              Authorization: `Bearer ${localStorage.getItem("token")}`,
            },
          }
        );

        if (
          response.data.result?.reset === 1 ||
          response.data.result?.Reset === 1 ||
          response.data.result?.status
        ) {
          successCount++;
        }
      } catch (e) {
        console.error("Failed to reset report:", report.REPORT_ID, e);
      }
    }
    return successCount === reportsList.length;
  };

  /**
   * Handles the final deletion process after user confirmation.
   */
  const handleConfirmDeletion = async () => {
    setShowDeleteConfirm(false);
    setLoadOpen(true);

    const reportsResetSuccess = await resetAllReports();

    if (reportsResetSuccess) {
      const dataForDeletion = { ...branchData, SCOPE: "O" };
      await handleConfirmSubmit(dataForDeletion);
    } else {
      setSnackbar({
        children: "Could not delete all associated reports. Aborting.",
        severity: "error",
      });
      handleDialogClose();
    }
  };

  /**
   * Submits data to the backend for both 'Update' and 'Delete' operations.
   */
  const handleConfirmSubmit = async (dataToSubmit) => {
    try {
      let jsonFormData = JSON.stringify({ branchData: dataToSubmit });
      await encrypt(iv, salt, jsonFormData).then((r) => (jsonFormData = r));
      let payload = { iv: ivBase64, salt: saltBase64, data: jsonFormData };

      const response = await axios.post(
        "/Server/EditBranch/FrtSubmitData",
        payload,
        {
          headers: { Authorization: `Bearer ${localStorage.getItem("token")}` },
        }
      );

      if (response.data?.result?.status) {
        let message = "";
        if (dataToSubmit.SCOPE === "I") {
          message = `Branch ${branchCode} details have been successfully updated.`;
        } else {
          message = `Branch ${branchCode} has been successfully deleted.`;
        }
        handleReset();
        setSnackbar({ children: message, severity: "success" });
      } else {
        setSnackbar({
          children: "An error occurred. Please try again later.",
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
          Delete Branch
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
        }}
      >
        <Box
          sx={{
            width: "100%",
            border: "1px solid #ddd",
            borderRadius: "8px",
            padding: 4,
            display: "flex",
            flexDirection: "column",
            backgroundColor: "#fff",
          }}
        >
          <Grid container spacing={2}>
            {/* Search and Branch Name */}
            <Grid item xs={12} sm={4}>
              <TextField
                label="Branch Code"
                name="CODE"
                error={error}
                helperText={error && "This field is required."}
                onChange={handleBranchCodeChange}
                value={branchCode}
                disabled={!fieldsDisabled}
                variant="outlined"
                fullWidth
                inputProps={{ maxLength: 5 }}
              />
            </Grid>
            <Grid item xs={12} sm={2}>
              <Button
                variant="contained"
                disableElevation
                startIcon={<SearchIcon />}
                sx={{ mt: 1 }}
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
                sx={{ mt: 1 }}
                onClick={handleReset}
              >
                Reset
              </Button>
            </Grid>
            <Grid item xs={12} sm={6}>
              <TextField
                label="Branch Name"
                name="NAME"
                onChange={handleInputChange}
                value={branchData.NAME}
                error={!!branchDetailErrors.NAME}
                helperText={branchDetailErrors.NAME}
                disabled={fieldsDisabled}
                variant="outlined"
                fullWidth
                InputProps={{
                  startAdornment: (
                    <AccountBalanceRoundedIcon sx={{ mr: 1, opacity: "50%" }} />
                  ),
                }}
                inputProps={{ maxLength: 40 }}
              />
            </Grid>

            {/* Circle, Network, Module, Region */}
            <Grid item xs={6} sm={6}>
              <TextField
                label="Circle Name"
                name="CIRCLE"
                error={!!branchDetailErrors.CIRCLE}
                helperText={branchDetailErrors.CIRCLE}
                select
                disabled={fieldsDisabled}
                value={branchData.CIRCLE}
                onChange={handleInputChange}
                variant="outlined"
                fullWidth
                InputProps={{
                  startAdornment: <LanIcon sx={{ mr: 1, opacity: "50%" }} />,
                }}
                SelectProps={{ MenuProps: { sx: { height: 350 } } }}
              >
                {circleList.map((choices) => (
                  <MenuItem key={choices} value={choices.split("~")[0]}>
                    {choices.split("~")[1]}
                  </MenuItem>
                ))}
              </TextField>
            </Grid>
            <Grid item xs={6} sm={2}>
              <TextField
                label="Network"
                name="NETWORK"
                disabled={fieldsDisabled}
                value={branchData.NETWORK}
                error={!!branchDetailErrors.NETWORK}
                helperText={branchDetailErrors.NETWORK}
                onChange={handleInputChange}
                variant="outlined"
                fullWidth
                InputProps={{
                  startAdornment: (
                    <SensorsIcon sx={{ mr: 1, opacity: "50%" }} />
                  ),
                }}
                inputProps={{ maxLength: 3 }}
              />
            </Grid>
            <Grid item xs={6} sm={2}>
              <TextField
                label="Module"
                name="MODULE"
                disabled={fieldsDisabled}
                value={branchData.MODULE}
                error={!!branchDetailErrors.MODULE}
                helperText={branchDetailErrors.MODULE}
                onChange={handleInputChange}
                variant="outlined"
                fullWidth
                InputProps={{
                  startAdornment: <DnsIcon sx={{ mr: 1, opacity: "50%" }} />,
                }}
                inputProps={{ maxLength: 3 }}
              />
            </Grid>
            <Grid item xs={6} sm={2}>
              <TextField
                label="Region"
                name="REGION"
                disabled={fieldsDisabled}
                value={branchData.REGION}
                error={!!branchDetailErrors.REGION}
                helperText={branchDetailErrors.REGION}
                onChange={handleInputChange}
                variant="outlined"
                fullWidth
                InputProps={{
                  startAdornment: <PublicIcon sx={{ mr: 1, opacity: "50%" }} />,
                }}
                inputProps={{ maxLength: 3 }}
              />
            </Grid>

            {/* Address Details */}
            <Grid item xs={12}>
              <TextField
                label="Address"
                name="ADDRESS"
                disabled={fieldsDisabled}
                value={branchData.ADDRESS}
                error={!!branchDetailErrors.ADDRESS}
                helperText={branchDetailErrors.ADDRESS}
                onChange={handleInputChange}
                variant="outlined"
                fullWidth
                InputProps={{
                  startAdornment: <HomeIcon sx={{ mr: 1, opacity: "50%" }} />,
                }}
                inputProps={{ maxLength: 40 }}
              />
            </Grid>
            <Grid item xs={6}>
              <TextField
                label="City"
                name="CITY"
                disabled={fieldsDisabled}
                value={branchData.CITY}
                error={!!branchDetailErrors.CITY}
                helperText={branchDetailErrors.CITY}
                onChange={handleInputChange}
                variant="outlined"
                fullWidth
                InputProps={{
                  startAdornment: (
                    <LocationCityRoundedIcon sx={{ mr: 1, opacity: "50%" }} />
                  ),
                }}
                inputProps={{ maxLength: 40 }}
              />
            </Grid>
            <Grid item xs={6}>
              <TextField
                label="State"
                name="STATE"
                disabled={fieldsDisabled}
                value={branchData.STATE}
                error={!!branchDetailErrors.STATE}
                helperText={branchDetailErrors.STATE}
                onChange={handleInputChange}
                variant="outlined"
                fullWidth
                InputProps={{
                  startAdornment: (
                    <CorporateFareRoundedIcon sx={{ mr: 1, opacity: "50%" }} />
                  ),
                }}
                inputProps={{ maxLength: 40 }}
              />
            </Grid>

            {/* Contact Details */}
            <Grid item xs={6} sm={4}>
              <TextField
                label="Pin Code"
                name="PIN"
                disabled={fieldsDisabled}
                value={branchData.PIN}
                error={!!branchDetailErrors.PIN}
                helperText={branchDetailErrors.PIN}
                onChange={handleInputChange}
                variant="outlined"
                fullWidth
                InputProps={{
                  startAdornment: (
                    <PinDropIcon sx={{ mr: 1, opacity: "50%" }} />
                  ),
                }}
                inputProps={{ maxLength: 6 }}
              />
            </Grid>
            <Grid item xs={6} sm={4}>
              <TextField
                label="STD Code"
                name="STDCODE"
                disabled={fieldsDisabled}
                value={branchData.STDCODE}
                error={!!branchDetailErrors.STDCODE}
                helperText={branchDetailErrors.STDCODE}
                onChange={handleInputChange}
                variant="outlined"
                fullWidth
                InputProps={{
                  startAdornment: (
                    <EmergencyShareIcon sx={{ mr: 1, opacity: "50%" }} />
                  ),
                }}
                inputProps={{ maxLength: 5 }}
              />
            </Grid>
            <Grid item xs={6} sm={4}>
              <TextField
                label="Phone No."
                name="PHONE"
                disabled={fieldsDisabled}
                value={branchData.PHONE}
                error={!!branchDetailErrors.PHONE}
                helperText={branchDetailErrors.PHONE}
                onChange={handleInputChange}
                variant="outlined"
                fullWidth
                InputProps={{
                  startAdornment: <PhoneIcon sx={{ mr: 1, opacity: "50%" }} />,
                }}
                inputProps={{ maxLength: 10 }}
              />
            </Grid>
            <Grid item xs={6} sm={4}>
              <TextField
                label="Mobile No."
                name="MOBILE"
                disabled={fieldsDisabled}
                value={branchData.MOBILE}
                error={!!branchDetailErrors.MOBILE}
                helperText={branchDetailErrors.MOBILE}
                onChange={handleInputChange}
                variant="outlined"
                fullWidth
                InputProps={{
                  startAdornment: (
                    <PhoneAndroidRoundedIcon sx={{ mr: 1, opacity: "50%" }} />
                  ),
                }}
                inputProps={{ maxLength: 10 }}
              />
            </Grid>
            <Grid item xs={6} sm={4}>
              <TextField
                label="IP Phone"
                name="IPPHONE"
                disabled={fieldsDisabled}
                value={branchData.IPPHONE}
                error={!!branchDetailErrors.IPPHONE}
                helperText={branchDetailErrors.IPPHONE}
                onChange={handleInputChange}
                variant="outlined"
                fullWidth
                InputProps={{
                  startAdornment: (
                    <WifiCalling3Icon sx={{ mr: 1, opacity: "50%" }} />
                  ),
                }}
                inputProps={{ maxLength: 10 }}
              />
            </Grid>
            <Grid item xs={12} sm={4}></Grid>

            {/* Audit Status and Auditor Details */}
            <Grid item xs={6} sm={6}>
              <TextField
                name="AUDITABLE"
                disabled={fieldsDisabled}
                fullWidth
                select
                InputProps={{
                  startAdornment: <PersonIcon sx={{ mr: 1, opacity: "50%" }} />,
                }}
                SelectProps={{ MenuProps: { sx: { height: 350 } } }}
                error={!!branchDetailErrors.AUDITABLE}
                helperText={branchDetailErrors.AUDITABLE}
                value={branchData.AUDITABLE}
                onChange={handleInputChange}
                label="Branch Audit Status"
              >
                <MenuItem value="">---Select---</MenuItem>
                <MenuItem value="A">Audited</MenuItem>
                <MenuItem value="N">Non Audited</MenuItem>
                <MenuItem value="I">IFCOFR Audited</MenuItem>
              </TextField>
            </Grid>
            <Grid item xs={6} sm={6}></Grid>

            {(branchData.AUDITABLE === "I" || branchData.AUDITABLE === "A") &&
              !fieldsDisabled && <>{/* Auditor details fields here */}</>}

            {/* Action Buttons */}
            <Grid item xs={12}>
              <Box sx={{ display: "flex", justifyContent: "center", p: 1 }}>
                <Button
                  variant="contained"
                  color="error"
                  size="medium"
                  sx={{ ml: 2 }}
                  onClick={fetchReportsForDelete}
                  disabled={fieldsDisabled}
                  startIcon={<DeleteIcon />}
                >
                  Delete
                </Button>
              </Box>
            </Grid>
          </Grid>
        </Box>
      </Container>

      {/* Warning Dialog for no changes */}
      <Dialog open={openWarningDialog}>
        <DialogTitle>Warning</DialogTitle>
        <DialogContent>
          <DialogContentText>
            Data is the same as before. Please modify data before saving.
          </DialogContentText>
        </DialogContent>
        <DialogActions>
          <Button onClick={() => setOpenWarningDialog(false)}>Close</Button>
        </DialogActions>
      </Dialog>

      {/* Loading Dialog */}
      <Dialog
        PaperProps={{
          style: { backgroundColor: "transparent", boxShadow: "none" },
        }}
        open={loadOpen}
      >
        <DialogContent>
          <CircularProgress />
        </DialogContent>
      </Dialog>

      {/* Snackbar */}
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

      {/* Delete Confirmation Dialog */}
      <Dialog open={showDeleteConfirm} maxWidth="xl">
        <DialogTitle>
          <Typography fontSize={20} fontWeight={750}>
            If you are going to exclude this branch from scope all these reports
            will be deleted. Do you want to continue?
          </Typography>
        </DialogTitle>
        <DialogContent>
          {reportsList.length > 0 ? (
            <Paper elevation={0}>
              <TableContainer sx={{ height: 500, minWidth: 1000 }}>
                <Table stickyHeader>
                  <TableHead>
                    <TableRow>
                      <TableCell align="center">S.No</TableCell>
                      <TableCell align="center">Module</TableCell>
                      <TableCell align="center">Report Name</TableCell>
                      <TableCell>Report Description</TableCell>
                      <TableCell align="center">Report Status</TableCell>
                    </TableRow>
                  </TableHead>
                  <TableBody>
                    {reportsList.map((row, i) => (
                      <TableRow key={i}>
                        <TableCell align="center">{i + 1}</TableCell>
                        <TableCell align="center">{row.MODULE}</TableCell>
                        <TableCell align="center">{row.REPORT_NAME}</TableCell>
                        <TableCell>{row.REPORT_DESC}</TableCell>
                        <TableCell align="center">
                          {branchData.AUDITABLE === "N"
                            ? REPORT_STATUS_CONSTANTS[row.CURRENT_STATUS]?.text
                            : AUDITED_REPORT_STATUS_CONSTANTS[
                                row.CURRENT_STATUS
                              ]?.text}
                        </TableCell>
                      </TableRow>
                    ))}
                  </TableBody>
                </Table>
              </TableContainer>
            </Paper>
          ) : (
            <DialogContentText>
              This branch has no reports to delete. You can proceed with
              deletion.
            </DialogContentText>
          )}
        </DialogContent>
        <DialogActions>
          <Button
            onClick={() => setShowDeleteConfirm(false)}
            variant="outlined"
            color="primary"
          >
            No
          </Button>
          <Button
            onClick={handleConfirmDeletion}
            variant="contained"
            color="error"
          >
            Yes, Delete Branch
          </Button>
        </DialogActions>
      </Dialog>
    </>
  );
}

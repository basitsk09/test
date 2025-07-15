import React, { useState } from "react";
import {
  TextField,
  Grid,
  MenuItem,
  Button,
  Typography,
  DialogTitle,
  Divider,
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
import Dialog from "@mui/material/Dialog";
import axios from "axios";
import Box from "@mui/material/Box";
import { validations } from "../CommonValidations/Validations";
import { useNavigate } from "react-router-dom";
import { encrypt } from "../Security/AES-GCM256";
import SaveIcon from "@mui/icons-material/Save";
import SearchIcon from "@mui/icons-material/Search";
import CircularProgress from "@mui/material/CircularProgress";
import Snackbar from "@mui/material/Snackbar";
import Alert from "@mui/material/Alert";
import { SnackbarProvider } from "notistack";
import PersonIcon from "@mui/icons-material/Person";
import { Cached } from "@mui/icons-material";

const iv = crypto.getRandomValues(new Uint8Array(12));
const ivBase64 = btoa(String.fromCharCode.apply(null, iv));
const salt = crypto.getRandomValues(new Uint8Array(16));
const saltBase64 = btoa(String.fromCharCode.apply(null, salt));

export default function FrtAddBranch() {
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
  };

  if (user.user_role !== "94" && user.user_role !== "96") {
    navigate("/");
  }

  const [branchDetailErrors, setBranchDetailErrors] = useState({});
  const [error, setError] = useState(false);
  const [branchData, setbranchData] = useState(emptyBranchDataMap);
  const [loadOpen, setLoadOpen] = useState(false);
  const [snackbar, setSnackbar] = useState(null);
  const [branchCode, setBranchCode] = useState("");
  const [circleList, setCircleList] = useState([]);
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
          const regex1 = /^\d{3}$/;
          if (!regex1.test(value) || value === "") {
            error = "Should be in 3 digits only";
          }
          break;
        case "IPPHONE":
        case "PHONE":
          error = validations("numInput", value);
          break;
        case "STDCODE":
          const regex = /^\d{2,5}$/;
          if (!regex.test(value) || value === "") {
            error = "STD Code will be in between 2 to 5 digits";
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
    setCircleList([]);
    setBranchCode("");
    setFieldsDisabled(true);
    setBranchDetailErrors({});
  };

  /**
   * This function first checks if a branch exists. If not, it fetches its details
   * from the core banking system to populate the form for adding.
   */
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
      await encrypt(iv, salt, jsonFormData).then(function (result) {
        jsonFormData = result;
      });
      let payload = { iv: ivBase64, salt: saltBase64, data: jsonFormData };

      // Check if branch already exists in main database
      const response = await axios.post(
        "/Server/AddBranch/fetchBranchDetails",
        payload,
        { headers: { Authorization: `Bearer ${localStorage.getItem("token")}` } }
      );

      // If branchData is NOT empty, it means the branch already exists. This is an error for an "Add" page.
      if (
        response.data?.result &&
        Object.keys(response.data?.result?.branchData).length !== 0
      ) {
        setSnackbar({
          children: "Branch " + branchCode + " already exists. Cannot add.",
          severity: "error",
        });
        handleDialogClose();
      }
      // If branch does not exist, proceed to fetch from CBS to pre-populate form
      else if (
        response.data?.result &&
        Object.keys(response.data?.result?.branchData).length === 0
      ) {
        fetchFromCbs();
      } else {
        setSnackbar({
          children: "An error occurred. Please try again after some time.",
          severity: "error",
        });
        handleDialogClose();
      }
    } catch (e) {
      console.error(e);
      setSnackbar({
        children: "An error occurred. Please try again after some time.",
        severity: "error",
      });
      handleDialogClose();
    }
  };

  /**
   * This function fetches branch details from the CBS table to populate the 'Add Branch' form.
   */
  const fetchFromCbs = async () => {
    try {
      let jsonFormData = JSON.stringify({ branchCode: branchCode });
      await encrypt(iv, salt, jsonFormData).then(function (result) {
        jsonFormData = result;
      });
      let payload = { iv: ivBase64, salt: saltBase64, data: jsonFormData };

      const response = await axios.post(
        "/Server/AddBranch/fetchBranchDetailsCbs",
        payload,
        { headers: { Authorization: `Bearer ${localStorage.getItem("token")}` } }
      );

      // If data is found in CBS, populate the form so the user can add it.
      if (
        response.data?.result &&
        Object.keys(response.data?.result?.branchData).length !== 0
      ) {
        setCircleList(response.data.result.circleList);
        setbranchData(response.data.result.branchData);
        setFieldsDisabled(false); // Enable the form for saving
        handleDialogClose();
      } else {
        setSnackbar({
          children: "Branch does not exist. Please contact 'Finance One Core Team'",
          severity: "error",
        });
        handleDialogClose();
      }
    } catch (e) {
      console.error(e);
      setSnackbar({
        children: "An error occurred. Please try again after some time.",
        severity: "error",
      });
      handleDialogClose();
    }
  };

  /**
   * This function validates all fields and submits the new branch data.
   */
  const handleSubmit = (event) => {
    event.preventDefault();
    if (!branchCode) {
      setError(true);
      return;
    }
    const errors = {};
    Object.keys(branchData).forEach((field) => {
      const error = validateInputFields(field, branchData[field]);
      if (error) {
        errors[field] = error;
      }
    });

    setBranchDetailErrors(errors);

    if (Object.keys(errors).length !== 0) {
      setSnackbar({
        children: "Kindly make sure all fields are filled correctly.",
        severity: "error",
      });
    } else {
      setLoadOpen(true);
      handleConfirmSubmit();
    }
  };

  /**
   * This function is an axios call to add the new branch to the database.
   */
  const handleConfirmSubmit = async () => {
    try {
      let jsonFormData = JSON.stringify({ branchData: branchData });
      await encrypt(iv, salt, jsonFormData).then(function (result) {
        jsonFormData = result;
      });
      let payload = { iv: ivBase64, salt: saltBase64, data: jsonFormData };

      const response = await axios.post(
        "/Server/AddBranch/submitNewBranch",
        payload,
        { headers: { Authorization: `Bearer ${localStorage.getItem("token")}` } }
      );

      if (response.data?.result?.status) {
        handleReset();
        setSnackbar({
          children: `The new branch ${branchCode} has been successfully created.`,
          severity: "success",
        });
        handleDialogClose();
      } else {
        setSnackbar({
          children: "An error occurred. Please try again after some time.",
          severity: "error",
        });
        handleDialogClose();
      }
    } catch (e) {
      console.error(e);
      setSnackbar({
        children: "An error occurred. Please try again after some time.",
        severity: "error",
      });
      handleDialogClose();
    }
  };

  return (
    <>
      <Box sx={{ display: "flex", height: 50, alignItems: "bottom" }}>
        <Typography variant="h5" gutterBottom sx={{ textAlign: "flex-start", p: 2 }}>
          Add Branch
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
            {/* First row */}
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

            {/* Second row */}
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
                SelectProps={{
                  MenuProps: { sx: { height: 350 } },
                }}
              >
                {circleList.map((choices) => (
                  <MenuItem key={choices} value={choices.split("~")[0]}>
                    {choices.split("~")[1]}
                  </MenuItem>
                ))}
              </TextField>
            </Grid>

            {/* Third row */}
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
                  startAdornment: <SensorsIcon sx={{ mr: 1, opacity: "50%" }} />,
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

            {/* Fourth row */}
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

            {/* Fifth row */}
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
                  startAdornment: <PinDropIcon sx={{ mr: 1, opacity: "50%" }} />,
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

            {/* Sixth row */}
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

            {/* Seventh row */}
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

            {/* Submit Button */}
            <Grid item xs={12} sm={12}>
              <Box sx={{ display: "flex", justifyContent: "center", p: 1 }}>
                <Button
                  variant="contained"
                  color="primary"
                  size="medium"
                  disabled={fieldsDisabled}
                  onClick={handleSubmit}
                  startIcon={<SaveIcon />}
                >
                  Save
                </Button>
              </Box>
            </Grid>
          </Grid>
        </Box>
      </Container>

      {/* Loading Dialog */}
      <Dialog
        PaperProps={{ style: { backgroundColor: "transparent", boxShadow: "none" } }}
        open={loadOpen}
      >
        <DialogContent sx={{ display: "Grid" }}>
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
            <Alert variant="filled" {...snackbar} onClose={handleCloseSnackbar} />
          </Snackbar>
        </SnackbarProvider>
      )}
    </>
  );
}

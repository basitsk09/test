import React, { useState } from "react";
import {
  TextField,
  Grid,
  MenuItem,
  FormControl,
  Button,
  Typography,
  DialogTitle,
  RadioGroup,
  Radio,
  FormLabel,
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
import FormControlLabel from "@mui/material/FormControlLabel";
import { validations } from "../CommonValidations/Validations";
import { useNavigate } from "react-router-dom";
import { encrypt } from "../Security/AES-GCM256";
import SaveIcon from "@mui/icons-material/Save";
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
  REPORT_STATUS_CONSTANTS
} from "../CommonValidations/commonConstants";

const iv = crypto.getRandomValues(new Uint8Array(12)); // for encryption
const ivBase64 = btoa(String.fromCharCode.apply(null, iv)); // for be decryption
const salt = crypto.getRandomValues(new Uint8Array(16)); // for encryption
const saltBase64 = btoa(String.fromCharCode.apply(null, salt)); // for be decryption

export default function FRTBranchDetails() {

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
  const [showReports, setShowReports] = useState(false);
  const [reportsList, setReportsList] = useState([]);
  const [fieldsDisabled, setFieldsDisabled] = useState(true);

  /**
   * This function is for closing the snackbar.
   *
   * @Author : V1014064
   **/
  const handleCloseSnackbar = () => setSnackbar(null);

  /**
   * This function is for closing the loading icon.
   *
   * @Author : V1014064
   **/
  const handleDialogClose = () => setLoadOpen(false);

  /**
   * This function is for handling the branch code input.
   *
   * @Author : V1014064
   **/
  const handleBranchCodeChange = (e) => {
    setError(false);

    let result = validations("numInput", e.target.value);

    if (result === "") {
      setBranchCode(e.target.value);
    } else {
      setSnackbar({ children: result, severity: "error" });
    }
  };

  /**
   * This function is to handle the input change of all the fields.
   *
   * @Author : V1014064
   **/
  const handleInputChange = (event) => {
    const { name, value } = event.target;
    const error = validateInputFields(name, value);
    const updatedbranchData = { ...branchData, [name]: value };
    setBranchDetailErrors({ ...branchDetailErrors, [name]: error });
    setbranchData(updatedbranchData);
    checkIfChanged(updatedbranchData);
  };

  /**
   * This function is to handle radio change.
   *
   * @Author : V1014064
   **/
  const handleRadioChange = (event) => {
    const { name, value } = event.target;
    if (value === "O") {
      setLoadOpen(true);
      fetchReports();
      return;
    }
    setbranchData((prevbranchData) => ({
      ...prevbranchData,
      [name]: value,
    }));
  };

  /**
   * This function handles all input validation for all the fields.
   *
   * @Author : V1014064
   **/
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
        case "auditFirmEmail":
          error = validations("emailId", value);
          break;
        default:
          break;
      }
    }
    return error;
  };

  /**
   * This function is for resetting all values.
   *
   * @Author : V1014064
   **/
  const handleReset = () => {
    setbranchData(emptyBranchDataMap);
    setFetchedData({});
    setAuditorData(emptyAuditorMap);
    setCircleList([]);
    setBranchCode("");
    setFieldsDisabled(true);
    setReportsList([]);
  };

  /**
   * This function is to check if the branchData is changed from intial data.
   *
   * @Author : V1014064
   **/
  const checkIfChanged = (updatedbranchData) => {
    const isDifferent = Object.keys(updatedbranchData).some(
      (key) => updatedbranchData[key] !== fetchedData[key]
    );

    return isDifferent;
  };

  /**
   * This function is an axios call to get the data of respective branch code from database from branchmaster table.
   *
   * @Author : V1014064
   **/
  const handleSearch = async () => {
    if (branchCode.length < 5) {
      setSnackbar({
        children: "Kindly enter branch code upto 5 digits.",
        severity: "error",
      });
      handleDialogClose();
      return;
    }
    try {
      let jsonFormData = JSON.stringify({ branchCode: branchCode });

      await encrypt(iv, salt, jsonFormData).then(function (result) {
        jsonFormData = result;
      });

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
      if (
        response.data?.result &&
        Object.keys(response.data?.result?.branchData).length !== 0
      ) {
        setCircleList(response.data.result.circleList);
        response.data.result.branchData.SCOPE = "I";
        setbranchData(response.data.result.branchData);
        setFetchedData({ ...response.data.result.branchData });
        setAuditorData(response.data.result.auditorData);
        setFieldsDisabled(false);
        handleDialogClose();
      } else if (
        response.data?.result &&
        Object.keys(response.data?.result?.branchData).length === 0
      ) {
        handleCbsSearch();
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
   * This function is an axios call to get the data of respective branch code from database from cbs table.
   *
   * @Author : V1014064
   **/
  const handleCbsSearch = async () => {
    try {
      let jsonFormData = JSON.stringify({ branchCode: branchCode });

      await encrypt(iv, salt, jsonFormData).then(function (result) {
        jsonFormData = result;
      });

      let payload = {
        iv: ivBase64,
        salt: saltBase64,
        data: jsonFormData,
      };

      const response = await axios.post(
        "/Server/EditBranch/fetchBranchDetailsCbs",
        payload,
        {
          headers: {
            Authorization: `Bearer ${localStorage.getItem("token")}`,
          },
        }
      );
      if (
        response.data?.result &&
        Object.keys(response.data?.result?.branchData).length !== 0
      ) {
        setCircleList(response.data.result.circleList);
        response.data.result.branchData.SCOPE = "O";
        setbranchData(response.data.result.branchData);
        setFetchedData({ ...response.data.result.branchData });
        setAuditorData(response.data.result.auditorData);
        setFieldsDisabled(false);
        handleDialogClose();
      } else {
        setSnackbar({
          children:
            "Branch does not exist. Please contact 'Finance One Core Team'",
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
   * This function is an axios call to get the list of reports which have been created for respective branch code from database.
   *
   * @Author : V1014064
   **/
  const fetchReports = async () => {
    try {
      let jsonFormData = JSON.stringify({ branchCode: branchCode });

      await encrypt(iv, salt, jsonFormData).then(function (result) {
        jsonFormData = result;
      });

      let payload = {
        iv: ivBase64,
        salt: saltBase64,
        data: jsonFormData,
      };

      const response = await axios.post(
        "/Server/EditBranch/fetchReports",
        payload,
        {
          headers: {
            Authorization: `Bearer ${localStorage.getItem("token")}`,
          },
        }
      );

      if (
        response.data?.result &&
        response.data?.result?.reportList.length !== 0
      ) {
        setReportsList(response.data.result.reportList);
        setShowReports(true);
        handleDialogClose();
      } else if (
        response.data?.result &&
        response.data?.result?.reportList.length === 0
      ) {
        let samp = branchData;
        samp.SCOPE = "O";
        setbranchData({ ...samp });
        handleDialogClose();
      } else {
        setSnackbar({
          children: "Failed to fetch reports data.",
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
   * This function is to check empty fields and further required validations before submitting.
   *
   * @Author : V1014064
   **/
  const handleSubmit = (event) => {
    if (!branchCode) {
      setError(true);
      return;
    }
    const errors = {};
    event.preventDefault();
    // Validate all fields
    Object.keys(branchData).forEach((field) => {
      const error = validateInputFields(field, branchData[field]);
      if (error) {
        errors[field] = error;
      }
    });

    setBranchDetailErrors(errors);

    if (Object.keys(errors).length !== 0) {
      setSnackbar({
        children: "Kindly make sure all fields are filled.",
        severity: "error",
      });
    } else if (branchData === fetchedData || !checkIfChanged(branchData)) {
      setOpenWarningDialog(true);
    } else {
      setLoadOpen(true);
      if (fetchedData.SCOPE === "I" && branchData.SCOPE === "O") {
        resetAllReports();
      } else {
        handleConfirmSubmit();
      }
    }
  };

  /**
   * This function is an axios call to reset all the reports from all modules for the selected branch.
   *
   * @Author : V1014064
   **/
  const resetAllReports = async () => {
    let newList = reportsList;

    if (newList.length > 0) {
      
      let count = 0;

      for (const newSamp of newList) {
        try {
          let data = {
            submissionId: newSamp.SUBMISSION_ID,
            reportId: newSamp.REPORT_ID,
            reportType: newSamp.REPORT_TYPE,
            module: newSamp.MODULE,
            method: "resetAll"
          };

          let jsonFormData = JSON.stringify(data);

          await encrypt(iv, salt, jsonFormData).then(function (result) {
            jsonFormData = result;
          });

          let payload = {
            iv: ivBase64,
            salt: saltBase64,
            data: jsonFormData,
          };

          const response = await axios.post(
            "/Server/EditBranch/resetReportsForDeletion",
            payload,
            {
              headers: {
                Authorization: `Bearer ${localStorage.getItem("token")}`,
              },
            }
          );

          if (response.data.result?.reset === 1 || response.data.result?.Reset === 1 || response.data.result?.status) count++;
        } catch (e) {
          console.error(e);
          if (e.response.status === 400) {
            console.log("An error occurred. Please try again after some time.");
          }
        }
      }

      if (newList.length === count) {
        handleConfirmSubmit();
      } else {
        setSnackbar({
          children: "An error occurred. Please try again after some time.",
          severity: "error",
        });
        handleDialogClose();
      }
    } else {
      handleConfirmSubmit();
    }
  };

  /**
   * This function is an axios call to Add/Update/Delete the branch.
   *
   * @Author : V1014064
   **/
  const handleConfirmSubmit = async () => {
    try {
      let jsonFormData = JSON.stringify({ branchData: branchData });

      await encrypt(iv, salt, jsonFormData).then(function (result) {
        jsonFormData = result;
      });

      let payload = {
        iv: ivBase64,
        salt: saltBase64,
        data: jsonFormData,
      };

      const response = await axios.post(
        "/Server/EditBranch/FrtSubmitData",
        payload,
        {
          headers: {
            Authorization: `Bearer ${localStorage.getItem("token")}`,
          },
        }
      );

      if (response.data?.result?.status) {
        let message = "";
        if (fetchedData.SCOPE === "I" && branchData.SCOPE === "I") {
          message =
            "The branch details for the branch " +
            branchCode +
            " have been successfully updated.";
        } else if (fetchedData.SCOPE === "O" && branchData.SCOPE === "I") {
          message =
            "The new branch " + branchCode + " has been successfully created.";
        } else if (fetchedData.SCOPE === "I" && branchData.SCOPE === "O") {
          message =
            "The branch " + branchCode + " have been successfully deleted.";
        }
        handleReset();
        setSnackbar({
          children: message,
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
      <Box
        sx={{
          display: "flex",
          height: 50,
          alignItems: "bottom",
        }}
      >
        <Typography
          variant="h5"
          gutterBottom
          sx={{ textAlign: "flex-start", p: 2 }}
        >
          Branch Details
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
          <Grid item xs={12} sm={6}>
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
                  inputProps={{
                    maxLength: 5,
                  }}
                />
              </Grid>
              <Grid item xs={12} sm={2}>
                <Button
                  variant="contained"
                  disableElevation
                  startIcon={<SearchIcon />}
                  sx={{ mt: 1 }}
                  onClick={() => {
                    setLoadOpen(true);
                    handleSearch();
                  }}
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
                  disabled={branchData.SCOPE === "O" || fieldsDisabled}
                  variant="outlined"
                  fullWidth
                  InputProps={{
                    startAdornment: (
                      <AccountBalanceRoundedIcon
                        sx={{ mr: 1, opacity: "50%" }}
                      />
                    ),
                  }}
                  inputProps={{
                    maxLength: 40,
                  }}
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
                  disabled={branchData.SCOPE === "O" || fieldsDisabled}
                  value={branchData.CIRCLE}
                  onChange={handleInputChange}
                  variant="outlined"
                  fullWidth
                  InputProps={{
                    startAdornment: <LanIcon sx={{ mr: 1, opacity: "50%" }} />,
                  }}
                  SelectProps={{
                    MenuProps: {
                      sx: {
                        height: 350,
                      },
                    },
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
                  disabled={branchData.SCOPE === "O" || fieldsDisabled}
                  value={branchData.NETWORK}
                  maxLength={3}
                  minLength={3}
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
                  inputProps={{
                    maxLength: 3,
                  }}
                />
              </Grid>
              <Grid item xs={6} sm={2}>
                <TextField
                  label="Module"
                  name="MODULE"
                  disabled={branchData.SCOPE === "O" || fieldsDisabled}
                  value={branchData.MODULE}
                  error={!!branchDetailErrors.MODULE}
                  helperText={branchDetailErrors.MODULE}
                  onChange={handleInputChange}
                  variant="outlined"
                  fullWidth
                  InputProps={{
                    startAdornment: <DnsIcon sx={{ mr: 1, opacity: "50%" }} />,
                  }}
                  inputProps={{
                    maxLength: 3,
                  }}
                />
              </Grid>
              <Grid item xs={6} sm={2}>
                <TextField
                  label="Region"
                  name="REGION"
                  disabled={branchData.SCOPE === "O" || fieldsDisabled}
                  value={branchData.REGION}
                  error={!!branchDetailErrors.REGION}
                  helperText={branchDetailErrors.REGION}
                  onChange={handleInputChange}
                  variant="outlined"
                  fullWidth
                  maxLength={3}
                  InputProps={{
                    startAdornment: (
                      <PublicIcon sx={{ mr: 1, opacity: "50%" }} />
                    ),
                  }}
                  inputProps={{
                    maxLength: 3,
                  }}
                />
              </Grid>

              {/* Fourth row */}
              <Grid item xs={12}>
                <TextField
                  label="Address"
                  name="ADDRESS"
                  disabled={branchData.SCOPE === "O" || fieldsDisabled}
                  value={branchData.ADDRESS}
                  error={!!branchDetailErrors.ADDRESS}
                  helperText={branchDetailErrors.ADDRESS}
                  onChange={handleInputChange}
                  variant="outlined"
                  fullWidth
                  InputProps={{
                    startAdornment: <HomeIcon sx={{ mr: 1, opacity: "50%" }} />,
                  }}
                  inputProps={{
                    maxLength: 40,
                  }}
                />
              </Grid>

              <Grid item xs={6}>
                <TextField
                  label="City"
                  name="CITY"
                  disabled={branchData.SCOPE === "O" || fieldsDisabled}
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
                  inputProps={{
                    maxLength: 40,
                  }}
                />
              </Grid>

              <Grid item xs={6}>
                <TextField
                  label="State"
                  name="STATE"
                  disabled={branchData.SCOPE === "O" || fieldsDisabled}
                  value={branchData.STATE}
                  error={!!branchDetailErrors.STATE}
                  helperText={branchDetailErrors.STATE}
                  onChange={handleInputChange}
                  variant="outlined"
                  fullWidth
                  InputProps={{
                    startAdornment: (
                      <CorporateFareRoundedIcon
                        sx={{ mr: 1, opacity: "50%" }}
                      />
                    ),
                  }}
                  inputProps={{
                    maxLength: 40,
                  }}
                />
              </Grid>

              {/* Fifth row */}
              <Grid item xs={6} sm={4}>
                <TextField
                  label="Pin Code"
                  name="PIN"
                  disabled={branchData.SCOPE === "O" || fieldsDisabled}
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
                  inputProps={{
                    maxLength: 6,
                  }}
                />
              </Grid>

              <Grid item xs={6} sm={4}>
                <TextField
                  label="STD Code"
                  name="STDCODE"
                  disabled={branchData.SCOPE === "O" || fieldsDisabled}
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
                  inputProps={{
                    maxLength: 5,
                  }}
                />
              </Grid>

              <Grid item xs={6} sm={4}>
                <TextField
                  label="Phone No."
                  name="PHONE"
                  disabled={branchData.SCOPE === "O" || fieldsDisabled}
                  value={branchData.PHONE}
                  error={!!branchDetailErrors.PHONE}
                  helperText={branchDetailErrors.PHONE}
                  onChange={handleInputChange}
                  variant="outlined"
                  fullWidth
                  InputProps={{
                    startAdornment: (
                      <PhoneIcon sx={{ mr: 1, opacity: "50%" }} />
                    ),
                  }}
                  inputProps={{
                    maxLength: 10,
                  }}
                />
              </Grid>

              {/* Sixth row */}

              <Grid item xs={6} sm={4}>
                <TextField
                  label="Mobile No."
                  name="MOBILE"
                  disabled={branchData.SCOPE === "O" || fieldsDisabled}
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
                  inputProps={{
                    maxLength: 10,
                  }}
                />
              </Grid>

              <Grid item xs={6} sm={4}>
                <TextField
                  label="IP Phone"
                  name="IPPHONE"
                  disabled={branchData.SCOPE === "O" || fieldsDisabled}
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
                  inputProps={{
                    maxLength: 10,
                  }}
                />
              </Grid>

              <Grid item xs={12} sm={4}></Grid>

              {/* Seventh row */}
              <Grid item xs={6} sm={6}>
                <TextField
                  name="AUDITABLE"
                  disabled={branchData.SCOPE === "O" || fieldsDisabled}
                  fullWidth
                  select
                  InputProps={{
                    startAdornment: (
                      <PersonIcon sx={{ mr: 1, opacity: "50%" }} />
                    ),
                  }}
                  SelectProps={{
                    MenuProps: {
                      sx: {
                        height: 350,
                      },
                    },
                  }}
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

              {/* Radio Buttons for INscope outScope */}
              <Grid item xs={12} sm={12}>
                <FormControl>
                  <FormLabel id="scope">Scope</FormLabel>
                  <RadioGroup
                    row
                    name="SCOPE"
                    disabled={fieldsDisabled}
                    value={branchData.SCOPE}
                    onChange={handleRadioChange}
                  >
                    <FormControlLabel
                      value="I"
                      disabled={fieldsDisabled}
                      control={<Radio />}
                      label="inScope"
                    />
                    <FormControlLabel
                      value="O"
                      disabled={fieldsDisabled}
                      control={<Radio />}
                      label="outScope"
                    />
                  </RadioGroup>
                </FormControl>
              </Grid>

              {/* Auditor details first row */}
              {(branchData.AUDITABLE === "I" || branchData.AUDITABLE === "A") &&
                branchData.SCOPE === "I" &&
                fetchedData.SCOPE === "I" && (
                  <>
                    <Grid item xs={12} sm={6}>
                      <TextField
                        label="Name of the Auditor"
                        name="AUDITORNAME"
                        value={auditorData.NAME}
                        disabled={true}
                        variant="outlined"
                        fullWidth
                      />
                    </Grid>
                    <Grid item xs={12} sm={6}>
                      <TextField
                        label="Type of Firm"
                        name="AUDITORFIRMTYPE"
                        value={auditorData.TYPE}
                        disabled={true}
                        variant="outlined"
                        fullWidth
                      />
                    </Grid>

                    {/* Auditor details second row */}
                    <Grid item xs={12} sm={6}>
                      <TextField
                        label="Auditor Membership No"
                        name="AUDITORMEMBERSHIP"
                        value={auditorData.MEMNO}
                        disabled={true}
                        variant="outlined"
                        fullWidth
                      />
                    </Grid>
                    <Grid item xs={12} sm={6}>
                      <TextField
                        label="Firm Registration No"
                        name="AUDITORFIRMNO"
                        value={auditorData.FIRMNO}
                        disabled={true}
                        variant="outlined"
                        fullWidth
                      />
                    </Grid>

                    {/* Auditor details third row */}
                    <Grid item xs={12} sm={6}>
                      <TextField
                        label="Firm Name"
                        name="AUDITORFIRMNAME"
                        value={auditorData.FIRMNAME}
                        disabled={true}
                        variant="outlined"
                        fullWidth
                      />
                    </Grid>
                    <Grid item xs={12} sm={6}>
                      <TextField
                        label="Firm's Address line 1"
                        name="AUDITORFIRMADD"
                        value={auditorData.ADDR}
                        disabled={true}
                        variant="outlined"
                        fullWidth
                      />
                    </Grid>

                    {/* Auditor details fourth row */}
                    <Grid item xs={12} sm={6}>
                      <TextField
                        label="Firm's City"
                        name="AUDITORFIRMCITY"
                        value={auditorData.CITY}
                        disabled={true}
                        variant="outlined"
                        fullWidth
                      />
                    </Grid>
                    <Grid item xs={12} sm={6}>
                      <TextField
                        label="Post Code"
                        name="AUDITORPOST"
                        value={auditorData.POST}
                        disabled={true}
                        variant="outlined"
                        fullWidth
                      />
                    </Grid>

                    {/* Auditor details fifth row */}
                    <Grid item xs={12} sm={6}>
                      <TextField
                        label="Email of the Audit Firm"
                        name="AUDITOREMAIL"
                        value={auditorData.EMAIL}
                        disabled={true}
                        variant="outlined"
                        fullWidth
                      />
                    </Grid>
                    <Grid item xs={12} sm={6}>
                      <TextField
                        label="Phone No of the Audit Firm"
                        name="AUDITORPHONE"
                        value={auditorData.PHONE}
                        disabled={true}
                        variant="outlined"
                        fullWidth
                      />
                    </Grid>
                  </>
                )}

              <Grid item xs={12} sm={12}>
                <Box
                  style={{
                    display: "flex",
                    justifyContent: "center",
                    padding: "5px",
                  }}
                >
                  <Button
                    variant="contained"
                    color="primary"
                    size="medium"
                    onClick={handleSubmit}
                    sx={{
                      display:
                        fetchedData.SCOPE === "O" && branchData.SCOPE === "O"
                          ? "none"
                          : "",
                    }}
                    startIcon={<SaveIcon />}
                  >
                    save
                  </Button>
                </Box>
              </Grid>
            </Grid>
          </Grid>
        </Box>
      </Container>

      {/* Dialog to display warning as data is same. */}
      <Dialog
        fullWidth={true}
        maxWidth={"sm"}
        open={openWarningDialog}
        aria-labelledby="responsive-dialog-title"
      >
        <DialogTitle id="responsive-dialog-title">Warning</DialogTitle>
        <DialogContent>
          <DialogContentText>
            Data you are trying to save was same as previous data.
            <br />
            Kindly modify the data before submitting it.
          </DialogContentText>
        </DialogContent>
        <DialogActions>
          <Button
            variant="contained"
            disableElevation
            color="warning"
            onClick={() => setOpenWarningDialog(false)}
          >
            Close
          </Button>
        </DialogActions>
      </Dialog>

      {/* Reload button related code */}
      <Dialog
        PaperProps={{
          style: {
            backgroundColor: "transparent",
            boxShadow: "none",
          },
        }}
        open={loadOpen}
        aria-labelledby="alert-dialog-title"
        aria-describedby="alert-dialog-description"
      >
        <Box>
          <DialogContent sx={{ display: "Grid" }}>
            <CircularProgress />
          </DialogContent>
        </Box>
      </Dialog>

      {/* Snackbar realted code */}
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

      {/* Dialog to display all filled reports when choosing out of scope*/}
      <Dialog
        open={showReports}
        maxWidth="xl"
        aria-labelledby="responsive-dialog-title"
      >
        <DialogTitle>
          <Typography fontSize={20} fontWeight={750}>
            If you are going to exclude this branch from scope all these reports
            will be deleted. Do you want to continue?
          </Typography>
        </DialogTitle>
        <DialogContent>
          <Grid item>
            <Paper elevation={0}>
              {reportsList.length > 0 && (
                <TableContainer
                  sx={
                    reportsList.length > 0
                      ? { height: 500, minWidth: 1000 }
                      : { height: 25, minWidth: 500 }
                  }
                >
                  <Table stickyHeader>
                    <TableHead key="thead">
                      <TableRow key="tHeadRow1">
                        <TableCell key="tHeadRow1Cell1" align="center">
                          <Typography fontSize={15} fontWeight={750}>
                            S.No
                          </Typography>
                        </TableCell>
                        <TableCell key="tHeadRow1Cell2" align="center">
                          <Typography fontSize={15} fontWeight={750}>
                            Module
                          </Typography>
                        </TableCell>
                        <TableCell
                          key="tHeadRow1Cell3"
                          sx={{ minWidth: 150 }}
                          align="center"
                        >
                          <Typography fontSize={15} fontWeight={750}>
                            Report Name
                          </Typography>
                        </TableCell>
                        <TableCell key="tHeadRow1Cell4" sx={{ minWidth: 500 }}>
                          <Typography fontSize={15} fontWeight={750}>
                            Report Description
                          </Typography>
                        </TableCell>
                        <TableCell
                          key="tHeadRow1Cell5"
                          sx={{ minWidth: 250 }}
                          align="center"
                        >
                          <Typography fontSize={15} fontWeight={750}>
                            Report Status
                          </Typography>
                        </TableCell>
                      </TableRow>
                    </TableHead>

                    <TableBody key="tbody">
                      {reportsList.map((rows, i) => (
                        <TableRow key={`tBodyRow${i}`}>
                          <TableCell key={`tBodyRow${i}}Cell1`} align="center">
                            {i + 1}
                          </TableCell>
                          <TableCell key={`tBodyRow${i}}Cell2`} align="center">
                            {rows.MODULE}
                          </TableCell>
                          <TableCell
                            key={`tBodyRow${i}}Cell3`}
                            sx={{ minWidth: 150 }}
                            align="center"
                          >
                            {rows.REPORT_NAME}
                          </TableCell>
                          <TableCell
                            key={`tBodyRow${i}Cell4`}
                            sx={{ minWidth: 500 }}
                          >
                            {rows.REPORT_DESC}
                          </TableCell>
                          <TableCell
                            key={`tBodyRow${i}}Cell5`}
                            sx={{ minWidth: 250 }}
                            align="center"
                          >
                            {branchData.AUDITABLE === "N"
                              ? REPORT_STATUS_CONSTANTS[rows.CURRENT_STATUS]
                                  .text
                              : AUDITED_REPORT_STATUS_CONSTANTS[
                                  rows.CURRENT_STATUS
                                ].text}
                          </TableCell>
                        </TableRow>
                      ))}
                    </TableBody>
                  </Table>
                </TableContainer>
              )}
            </Paper>
          </Grid>
        </DialogContent>
        <DialogActions>
          <Button
            onClick={() => {
              setShowReports(false);
            }}
            variant="outlined"
            color="error"
            disableElevation
          >
            No ( Keep in scope)
          </Button>
          <Button
            onClick={() => {
              let samp = branchData;
              samp.SCOPE = "O";
              setbranchData({ ...samp });
              setShowReports(false);
            }}
            variant="contained"
            color="success"
            disableElevation
          >
            Yes (Remove from scope)
          </Button>
        </DialogActions>
      </Dialog>
    </>
  );
}

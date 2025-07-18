import { useEffect, useState } from "react";
import { createTheme, ThemeProvider } from "@mui/material/styles";
import Grid from "@mui/material/Grid";
import IconButton from "@mui/material/IconButton";
import Typography from "@mui/material/Typography";
import axios from "axios";
import { useNavigate } from "react-router-dom";
import Divider from "@mui/material/Divider";
import { Box } from "@mui/system";
import {
  InputBase,
  Button,
  Chip,
  Table,
  TableBody,
  TableCell,
  TableContainer,
  TableHead,
  TableRow,
  TablePagination,
  Paper,
} from "@mui/material";
import DialogContent from "@mui/material/DialogContent";
import Dialog from "@mui/material/Dialog";
import CircularProgress from "@mui/material/CircularProgress";
import Snackbar from "@mui/material/Snackbar";
import Alert from "@mui/material/Alert";
import FilterAltIcon from "@mui/icons-material/FilterAlt";
import { Close, DoneAll } from "@mui/icons-material";
import { encrypt } from "../Security/AES-GCM256";

// Note: The crypto and encryption logic is retained as is from the original file.
const iv = crypto.getRandomValues(new Uint8Array(12));
const ivBase64 = btoa(String.fromCharCode.apply(null, iv));
const salt = crypto.getRandomValues(new Uint8Array(16));
const saltBase64 = btoa(String.fromCharCode.apply(null, salt));

const defaultTheme = createTheme();

const FrtRequestActivity = () => {
  document.title = "CRS | Track Request";

  const [requestList, setRequestList] = useState([]);
  const [content, setContent] = useState("Loading . . .");
  const [snackbar, setSnackbar] = useState(null);
  const [loadOpen, setLoadOpen] = useState(false);
  const [searchQuery, setSearchQuery] = useState("");

  // State for Table Pagination
  const [page, setPage] = useState(0);
  const [rowsPerPage, setRowsPerPage] = useState(10);

  const navigate = useNavigate();
  const loggedInUser = JSON.parse(localStorage.getItem("user"));

  useEffect(() => {
    if (
      loggedInUser.user_role !== "96" &&
      loggedInUser.user_role !== "94" &&
      loggedInUser.user_role !== "50" &&
      loggedInUser.user_role !== "9"
    ) {
      navigate("/"); // logout
    }

    fetchData().then(() => {
      console.log("data fetched");
    });
    // eslint-disable-next-line react-hooks/exhaustive-deps
  }, []);

  const fetchData = async () => {
    let jsonFormData = JSON.stringify({
      quarterEndDate: loggedInUser.quarterEndDate,
    });
    await encrypt(iv, salt, jsonFormData).then(function (result) {
      jsonFormData = result;
    });
    let payload = { iv: ivBase64, salt: saltBase64, data: jsonFormData };
    try {
      const response = await axios.post(
        "/Server/EditBranch/fetchCrsRequests",
        payload,
        {
          headers: {
            Authorization: `Bearer ${localStorage.getItem("token")}`,
          },
        }
      );

      const transformedData = response.data.result.map((item) => {
        let changeReqType = "Unknown";
        let branchCode = item.RT_BRANCH;

        if (item.RT_TYPE === "A") {
          changeReqType = "Audit Status";
          if (item.RT_SUBTYPE === "M") {
            branchCode = "Multiple Branches";
          }
        } else if (item.RT_TYPE === "B") {
          if (item.RT_SUBTYPE === "A") {
            changeReqType = "Add Branch";
          } else if (item.RT_SUBTYPE === "D") {
            changeReqType = "Delete Branch";
          }
        }
        return {
          ID: item.RT_ID,
          CHANGE_REQ_TYPE: changeReqType,
          BRANCH_CODE: branchCode,
          STATUS: String(item.RT_STATUS),
          REQUESTED_USER: item.RT_MAKER,
        };
      });

      setRequestList(transformedData);
      setContent("No pending requests.");
    } catch (error) {
      console.error("Error fetching data:", error);
      setContent("Error fetching data. Please try again.");
    }
  };

  const handleLoadOpen = () => setLoadOpen(true);
  const handleDialogClose = () => setLoadOpen(false);
  const handleCloseSnackbar = () => setSnackbar(null);

  const filteredRequests = requestList.filter((request) =>
    request.BRANCH_CODE
      ? request.BRANCH_CODE.toLowerCase().includes(searchQuery.toLowerCase())
      : false
  );

  const handleChangePage = (event, newPage) => {
    setPage(newPage);
  };

  const handleChangeRowsPerPage = (event) => {
    setRowsPerPage(parseInt(event.target.value, 10));
    setPage(0);
  };

  /**
   * NEW: Handles the "Cancel Request" button click.
   * Calls the API to cancel a specific request.
   * @param {object} requestData - The data of the row to be cancelled.
   */
  const handleCancelRequest = async (requestData) => {
    handleLoadOpen();
    try {
      // Payload includes reqid, branchcode, and the status for 'Cancelled'
      const data = {
        reqid: requestData.ID,
        branchcode: requestData.BRANCH_CODE,
        status: "4", // '4' for Cancelled
      };

      const successMessage = `Request successfully cancelled for Req Id: ${requestData.ID}.`;
      const errorMessage = `Failed to cancel the request for Req Id: ${requestData.ID}.`;

      let jsonFormData = JSON.stringify(data);
      jsonFormData = await encrypt(iv, salt, jsonFormData);

      let payload = { iv: ivBase64, salt: saltBase64, data: jsonFormData };

      // Assuming the same endpoint handles the cancel action
      const response = await axios.post(
        "Server/frtRequestActivityService/requestAction",
        payload,
        {
          headers: { Authorization: `Bearer ${localStorage.getItem("token")}` },
        }
      );

      if (response.data.statusCode === 200) {
        await fetchData(); // Refresh list on success
        setSnackbar({ children: successMessage, severity: "success" });
      } else {
        setSnackbar({ children: errorMessage, severity: "error" });
      }
    } catch (e) {
      console.error("Error cancelling request:", e);
      setSnackbar({
        children: "An error occurred. Please try again.",
        severity: "error",
      });
    } finally {
      handleDialogClose();
    }
  };

  /**
   * This function is axios call to accept the request.
   **/
  const handleAccept = async (requestData) => {
    handleLoadOpen();
    try {
      let jsonFormData = JSON.stringify({
        id: requestData.ID,
        status: "2",
      });
      jsonFormData = await encrypt(iv, salt, jsonFormData);

      let payload = { iv: ivBase64, salt: saltBase64, data: jsonFormData };

      const response = await axios.post(
        "Server/frtRequestActivityService/requestAction",
        payload,
        {
          headers: { Authorization: `Bearer ${localStorage.getItem("token")}` },
        }
      );

      if (response.data.result) {
        await fetchData();
        setSnackbar({ children: response.data.message, severity: "success" });
      } else {
        setSnackbar({
          children: response.data.message || "An error occurred.",
          severity: "error",
        });
      }
    } catch (e) {
      console.error(e);
      setSnackbar({
        children: "An error occurred. Please try again.",
        severity: "error",
      });
    } finally {
      handleDialogClose();
    }
  };

  const handleAcceptAll = async () => {
    handleLoadOpen();
    const requestsToAccept = requestList.filter(
      (req) =>
        req.REQUESTED_USER !== loggedInUser.pf_number && req.STATUS === "1"
    );

    let successCount = 0;

    for (const req of requestsToAccept) {
      try {
        let jsonFormData = JSON.stringify({ id: req.ID, status: "2" });
        jsonFormData = await encrypt(iv, salt, jsonFormData);
        let payload = { iv: ivBase64, salt: saltBase64, data: jsonFormData };

        const response = await axios.post(
          "Server/frtRequestActivityService/requestAction",
          payload,
          {
            headers: {
              Authorization: `Bearer ${localStorage.getItem("token")}`,
            },
          }
        );
        if (response.data.result) {
          successCount++;
        }
      } catch (e) {
        console.error(`Failed to accept request ID ${req.ID}:`, e);
      }
    }

    await fetchData();
    handleDialogClose();

    if (requestsToAccept.length === successCount && successCount > 0) {
      setSnackbar({
        children: "All pending requests successfully accepted.",
        severity: "success",
      });
    } else if (requestsToAccept.length > 0) {
      const failedCount = requestsToAccept.length - successCount;
      setSnackbar({
        children: `${failedCount} request(s) failed to be accepted. ${successCount} accepted.`,
        severity: "warning",
      });
    } else {
      setSnackbar({
        children: "No pending requests to accept.",
        severity: "info",
      });
    }
  };

  const renderStatusChip = (status) => {
    let label, sx;
    switch (status) {
      case "1":
        label = "Pending";
        sx = { backgroundColor: "#F5BC51", color: "white" };
        break;
      case "2":
        label = "Approved";
        sx = { backgroundColor: "#51B92D", color: "white" };
        break;
      case "3":
        label = "Rejected";
        sx = { backgroundColor: "#8471FE", color: "white" };
        break;
      case "4":
        label = "Cancelled";
        sx = { backgroundColor: "#F9805C", color: "white" };
        break;
      default:
        label = "Unknown";
        sx = { backgroundColor: "grey", color: "white" };
    }
    return <Chip label={label} sx={{ ...sx, fontWeight: "bold" }} />;
  };

  return (
    <ThemeProvider theme={defaultTheme}>
      <Box sx={{ display: "flex", flexDirection: "column" }}>
        <Grid container alignItems="center" justifyContent="space-between">
          <Grid item>
            <Typography variant={"h5"}>Track Request</Typography>
          </Grid>
          <Grid item>
            <Box sx={{ display: "flex", alignItems: "center" }}>
              <Paper
                component="form"
                sx={{
                  p: "2px 4px",
                  display: "flex",
                  alignItems: "center",
                  width: 350,
                }}
                onSubmit={(e) => e.preventDefault()}
              >
                <FilterAltIcon sx={{ ml: 1, color: "text.secondary" }} />
                <Divider sx={{ height: 28, m: 1.0 }} orientation="vertical" />
                <InputBase
                  sx={{ ml: 1, flex: 1 }}
                  placeholder="Filter by Branch"
                  value={searchQuery}
                  onChange={(e) => {
                    setPage(0);
                    setSearchQuery(e.target.value);
                  }}
                />
                <Divider sx={{ height: 28, m: 1.0 }} orientation="vertical" />
                <IconButton
                  type="button"
                  sx={{ p: "10px" }}
                  aria-label="clear"
                  onClick={() => setSearchQuery("")}
                >
                  <Close />
                </IconButton>
              </Paper>
              {requestList.length > 0 && loggedInUser.user_role === "50" && (
                <Button
                  onClick={handleAcceptAll}
                  variant="contained"
                  disableElevation
                  color="success"
                  startIcon={<DoneAll />}
                  sx={{ ml: 2 }}
                >
                  Accept All
                </Button>
              )}
            </Box>
          </Grid>
        </Grid>

        <br />
        <Divider />

        <Paper sx={{ width: "100%", overflow: "hidden", mt: 2 }}>
          <TableContainer sx={{ maxHeight: "calc(100vh - 280px)" }}>
            <Table stickyHeader aria-label="request status table">
              <TableHead>
                <TableRow
                  sx={{
                    "& th": { backgroundColor: "#b9def0", fontWeight: "bold" },
                  }}
                >
                  <TableCell align="center">Req ID</TableCell>
                  <TableCell align="left">Change Request For</TableCell>
                  <TableCell align="left">Branch</TableCell>
                  <TableCell align="center">Status</TableCell>
                  <TableCell align="center">Action</TableCell>
                  <TableCell align="left">Requested By</TableCell>
                </TableRow>
              </TableHead>
              <TableBody>
                {filteredRequests.length > 0 ? (
                  filteredRequests
                    .slice(page * rowsPerPage, page * rowsPerPage + rowsPerPage)
                    .map((row) => (
                      <TableRow
                        hover
                        role="checkbox"
                        tabIndex={-1}
                        key={row.ID}
                      >
                        <TableCell align="center">{row.ID}</TableCell>
                        <TableCell align="left">
                          {row.CHANGE_REQ_TYPE}
                        </TableCell>
                        <TableCell align="left">{row.BRANCH_CODE}</TableCell>
                        <TableCell align="center">
                          {renderStatusChip(row.STATUS)}
                        </TableCell>
                        <TableCell align="center">
                          {/* UPDATED: Show Cancel button for user's own pending requests */}
                          {row.STATUS === "1" &&
                            row.REQUESTED_USER === loggedInUser.pf_number && (
                              <Button
                                variant="contained"
                                color="error"
                                size="small"
                                onClick={() => handleCancelRequest(row)}
                              >
                                Cancel Request
                              </Button>
                            )}
                        </TableCell>
                        <TableCell align="left">{row.REQUESTED_USER}</TableCell>
                      </TableRow>
                    ))
                ) : (
                  <TableRow>
                    <TableCell colSpan={6} align="center">
                      <Typography variant="subtitle1" sx={{ p: 3 }}>
                        {requestList.length === 0
                          ? content
                          : "No records match your search query."}
                      </Typography>
                    </TableCell>
                  </TableRow>
                )}
              </TableBody>
            </Table>
          </TableContainer>
          <TablePagination
            rowsPerPageOptions={[10, 25, 100]}
            component="div"
            count={filteredRequests.length}
            rowsPerPage={rowsPerPage}
            page={page}
            onPageChange={handleChangePage}
            onRowsPerPageChange={handleChangeRowsPerPage}
          />
        </Paper>

        {/* Loading Dialog */}
        <Dialog
          open={loadOpen}
          PaperProps={{
            style: { backgroundColor: "transparent", boxShadow: "none" },
          }}
        >
          <DialogContent>
            <CircularProgress />
          </DialogContent>
        </Dialog>

        {/* Snackbar */}
        {snackbar && (
          <Snackbar
            open
            anchorOrigin={{ vertical: "top", horizontal: "center" }}
            onClose={handleCloseSnackbar}
            autoHideDuration={4000}
          >
            <Alert
              variant="filled"
              {...snackbar}
              onClose={handleCloseSnackbar}
            />
          </Snackbar>
        )}
      </Box>
    </ThemeProvider>
  );
};

export default FrtRequestActivity;

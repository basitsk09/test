import React, { useState, useEffect } from "react";
import {
  Box,
  Button,
  Container,
  Dialog,
  DialogActions,
  DialogContent,
  DialogTitle,
  Grid,
  Paper,
  Typography,
  CircularProgress,
  Table,
  TableBody,
  TableCell,
  TableContainer,
  TableHead,
  TableRow,
  Alert,
  Collapse,
  IconButton,
} from "@mui/material";
import KeyboardArrowDownIcon from "@mui/icons-material/KeyboardArrowDown";
import KeyboardArrowUpIcon from "@mui/icons-material/KeyboardArrowUp";
import { encrypt } from "../Security/AES-GCM256";
import axios from "axios";

// --- Helper function for encryption ---
const getEncryptedPayload = async (data) => {
  const iv = crypto.getRandomValues(new Uint8Array(12));
  const salt = crypto.getRandomValues(new Uint8Array(16));
  const ivBase64 = btoa(String.fromCharCode.apply(null, iv));
  const saltBase64 = btoa(String.fromCharCode.apply(null, salt));
  const jsonFormData = JSON.stringify(data);
  const encryptedData = await encrypt(iv, salt, jsonFormData);
  return { iv: ivBase64, salt: saltBase64, data: encryptedData };
};

// --- Row Sub-Component ---
function Row(props) {
  const { row, onAction } = props;
  const [open, setOpen] = useState(false);
  const [reports, setReports] = useState([]);
  const [loadingReports, setLoadingReports] = useState(false);
  const isDelete = row.requestType === "Delete Branch";

  const fetchReportsForBranch = async () => {
    setLoadingReports(true);
    try {
      const payload = await getEncryptedPayload({
        branchCode: row.branchCode,
        quarterDate: row.quarterDate,
      });

      const response = await axios.post("/Server/EditBranch/fetchReports", payload, {
        headers: { Authorization: `Bearer ${localStorage.getItem("token")}` },
      });

      const rawData = response.data?.result?.reportList || [];
      // We store the original raw data to pass it up for the reset operation
      setReports(rawData);
    } catch (error) {
      console.error("Failed to fetch reports for branch:", error);
      setReports([]); // Ensure reports is an empty array on failure
    } finally {
      setLoadingReports(false);
    }
  };

  const handleRowClick = () => {
    const shouldOpen = !open;
    setOpen(shouldOpen);
    if (shouldOpen && isDelete && reports.length === 0) {
      fetchReportsForBranch();
    }
  };

  const getPendingStatusText = (status) => {
    if (!status) return "N/A";
    const statusStr = String(status);
    if (statusStr.startsWith("1")) return "Maker";
    if (statusStr.startsWith("2")) return "Branch Manager";
    if (statusStr.startsWith("3")) return "Branch Auditor";
    if (statusStr.startsWith("4")) return "RO Manager";
    if (statusStr.startsWith("5")) return "Freezed";
    return statusStr;
  };

  const rowStyle = {
    backgroundColor: isDelete ? "#ffebee" : "#e8f5e9",
    "& > *": { borderBottom: "unset" },
  };

  return (
    <React.Fragment>
      <TableRow sx={rowStyle}>
        <TableCell>
          <IconButton aria-label="expand row" size="small" onClick={handleRowClick}>
            {open ? <KeyboardArrowUpIcon /> : <KeyboardArrowDownIcon />}
          </IconButton>
        </TableCell>
        <TableCell>{row.requestId}</TableCell>
        <TableCell>{row.branchCode}</TableCell>
        <TableCell>{row.branchName}</TableCell>
        <TableCell>{row.requestType}</TableCell>
        <TableCell>{row.status}</TableCell>
        <TableCell>{row.requestedOn}</TableCell>
      </TableRow>
      <TableRow>
        <TableCell style={{ paddingBottom: 0, paddingTop: 0 }} colSpan={7}>
          <Collapse in={open} timeout="auto" unmountOnExit>
            <Box sx={{ margin: 1, p: 2, backgroundColor: isDelete ? "#fff0f0" : "#f0fff0" }}>
              <Grid container spacing={2} sx={{ mb: 2 }}>
                <Grid item xs={3}><b>Circle:</b> {row.circleName}</Grid>
                <Grid item xs={3}><b>RO:</b> {row.roCode}</Grid>
                <Grid item xs={3}><b>Audited Status:</b> {row.auditStatus}</Grid>
                <Grid item xs={3}><b>Requested By:</b> {row.requestedBy}</Grid>
              </Grid>

              {isDelete && (
                <Box sx={{ my: 2 }}>
                  {loadingReports ? ( <CircularProgress size={24} /> ) : (
                    <>
                      <Typography sx={{ color: "red", mb: 1 }}>*Following reports will be deleted on branch deletion.</Typography>
                      <TableContainer component={Paper}>
                        <Table size="small">
                          <TableHead>
                            <TableRow>
                              <TableCell>MODULE</TableCell>
                              <TableCell>REPORT NAME</TableCell>
                              <TableCell>REPORT DESCRIPTION</TableCell>
                              <TableCell>PENDING WITH</TableCell>
                            </TableRow>
                          </TableHead>
                          <TableBody>
                            {reports.map((report, index) => (
                              <TableRow key={index}>
                                <TableCell>{report.MODULE}</TableCell>
                                <TableCell>{report.REPORT_NAME}</TableCell>
                                <TableCell>{report.REPORT_DESC}</TableCell>
                                <TableCell>{getPendingStatusText(report.CURRENT_STATUS)}</TableCell>
                              </TableRow>
                            ))}
                          </TableBody>
                        </Table>
                      </TableContainer>
                    </>
                  )}
                </Box>
              )}

              <Box sx={{ display: "flex", justifyContent: "center", mt: 2, gap: 2 }}>
                {/* UPDATED: Pass the fetched reports list up to the parent on action */}
                <Button variant="contained" color="success" onClick={() => onAction(row, "approve", reports)}>Approve</Button>
                <Button variant="contained" color="error" onClick={() => onAction(row, "reject")}>Reject</Button>
              </Box>
            </Box>
          </Collapse>
        </TableCell>
      </TableRow>
    </React.Fragment>
  );
}

// --- Main Component ---
const FRTCheckerAddDeleteReq = () => {
  const [requests, setRequests] = useState([]);
  const [loading, setLoading] = useState(true);
  const [processing, setProcessing] = useState(false);
  const [dialog, setDialog] = useState({ open: false, title: "", message: "" });
  const [confirmDialog, setConfirmDialog] = useState({ open: false, request: null });
  const user = JSON.parse(localStorage.getItem("user"));

  const fetchBranchRequests = async () => {
    setLoading(true);
    try {
      const payload = await getEncryptedPayload({ quarterDate: user?.quarterEndDate });
      const response = await axios.post("/Server/EditBranch/getBranchRequests", payload, {
        headers: { Authorization: `Bearer ${localStorage.getItem("token")}` },
      });
      setRequests(response.data.data || []);
    } catch (error) {
      console.error("Failed to fetch requests:", error);
      setDialog({ open: true, title: "Error", message: "Failed to load requests. Please try again." });
      setRequests([]);
    } finally {
      setLoading(false);
    }
  };

  useEffect(() => {
    document.title = "FRT | Scope Change Requests";
    fetchBranchRequests();
  }, []);

  const handleAction = (request, action, reports = []) => {
    if (action === 'approve' && request.requestType === 'Delete Branch') {
      // Store the request and its already-fetched reports in the dialog state
      setConfirmDialog({ open: true, request: { ...request, reports } });
    } else {
      processSimpleAction(request, action);
    }
  };

  const processSimpleAction = async (request, action) => {
    setProcessing(true);
    try {
      const payload = await getEncryptedPayload({
        requestId: request.requestId,
        branchCode: request.branchCode, requestType: request.requestType,
        auditStatus: request.auditStatus, requestedById: request.requestedBy,
        circleCode: request.circleCode, roCode: request.roCode,
        quarterEndDate: user.quarterEndDate,
      });
      await axios.post(`/Server/EditBranch/${action}`, payload, {
        headers: { Authorization: `Bearer ${localStorage.getItem("token")}` },
      });
      setDialog({ open: true, title: `Request ${action}d`, message: `Request ${request.requestId} has been successfully ${action}d.`});
      fetchBranchRequests();
    } catch (error) {
      console.error(`Failed to ${action} request:`, error);
      setDialog({ open: true, title: "Action Failed", message: error.response?.data?.message || "An error occurred." });
    } finally {
      setProcessing(false);
    }
  };

  const handleConfirmDelete = async () => {
    const { request } = confirmDialog;
    if (!request) return;

    setProcessing(true);
    setConfirmDialog({ open: false, request: null });

    try {
      let reportsToReset = request.reports;

      // SAFETY NET: If reports weren't available, fetch them now.
      if (!reportsToReset || reportsToReset.length === 0) {
        reportsToReset = await fetchReportsForDeletion(request.branchCode, user.quarterEndDate);
      }

      if (reportsToReset === null) {
        throw new Error("Failed to get the list of reports for deletion.");
      }

      await resetAllReports(reportsToReset);
      await sendFinalApproval(request);
      
      setDialog({ open: true, title: "Success", message: `Branch ${request.branchCode} and all its reports have been successfully deleted.` });
      fetchBranchRequests();
    } catch (error) {
      console.error("Deletion process failed:", error);
      setDialog({ open: true, title: "Deletion Failed", message: error.message || "An unexpected error occurred." });
    } finally {
      setProcessing(false);
    }
  };

  const fetchReportsForDeletion = async (branchCode, quarterDate) => {
    try {
      console.log("Fetching reports from API as a fallback...");
      const payload = await getEncryptedPayload({ branchCode, quarterDate });
      const response = await axios.post("/Server/EditBranch/fetchReports", payload, {
        headers: { Authorization: `Bearer ${localStorage.getItem("token")}` },
      });
      return response.data?.result?.reportList || [];
    } catch (error) {
      console.error("Error in fetchReportsForDeletion fallback:", error);
      return null;
    }
  };

  const resetAllReports = async (reportsList) => {
    if (reportsList.length === 0) {
      return;
    }
    for (const report of reportsList) {
      try {
        const payload = await getEncryptedPayload({
          submissionId: report.SUBMISSION_ID, reportId: report.REPORT_ID,
          reportType: report.REPORT_TYPE, module: report.MODULE,
          method: "resetAll",
        });
        await axios.post("/Server/EditBranch/resetReportsForDeletion", payload, {
          headers: { Authorization: `Bearer ${localStorage.getItem("token")}` },
        });
      } catch (e) {
        console.error(`Failed to reset report ${report.REPORT_ID}:`, e);
        throw new Error(`Failed to reset report: ${report.REPORT_NAME}. Aborting deletion.`);
      }
    }
  };

  const sendFinalApproval = async (request) => {
    const payload = await getEncryptedPayload({
      requestId: request.requestId, branchCode: request.branchCode,
      requestType: request.requestType, auditStatus: request.auditStatus,
      requestedById: request.requestedBy, circleCode: request.circleCode,
      roCode: request.roCode, quarterEndDate: user.quarterEndDate,
    });
    await axios.post(`/Server/EditBranch/approve`, payload, {
      headers: { Authorization: `Bearer ${localStorage.getItem("token")}` },
    });
  };

  const handleDialogClose = () => setDialog({ open: false, title: "", message: "" });
  const handleConfirmDialogClose = () => setConfirmDialog({ open: false, request: null });

  return (
    <>
      <Container maxWidth={false}>
        <Typography variant="h5" sx={{ mb: 2 }}>CRS Scope Change Requests</Typography>
        <TableContainer component={Paper}>
          {(loading || processing) ? (
            <Box sx={{ display: 'flex', justifyContent: 'center', p: 4, alignItems: 'center', flexDirection: 'column' }}>
              <CircularProgress />
              {processing && <Typography sx={{mt: 2}}>Processing request, please wait...</Typography>}
            </Box>
          ) : (
            <Table aria-label="collapsible table">
              <TableHead sx={{ backgroundColor: "#b9def0" }}>
                <TableRow>
                  <TableCell />
                  <TableCell><b>Request ID</b></TableCell>
                  <TableCell><b>Branch</b></TableCell>
                  <TableCell><b>Branch Name</b></TableCell>
                  <TableCell><b>Request Type</b></TableCell>
                  <TableCell><b>Status</b></TableCell>
                  <TableCell><b>Requested on</b></TableCell>
                </TableRow>
              </TableHead>
              <TableBody>
                {requests.map((row) => (
                  <Row key={row.requestId} row={{ ...row, quarterDate: user?.quarterEndDate }} onAction={handleAction} />
                ))}
              </TableBody>
            </Table>
          )}
        </TableContainer>
      </Container>
      
      <Dialog open={dialog.open} onClose={handleDialogClose}>
        <DialogTitle>{dialog.title}</DialogTitle>
        <DialogContent>
          <Alert severity={dialog.title.toLowerCase().includes("failed") ? "error" : "success"}>{dialog.message}</Alert>
        </DialogContent>
        <DialogActions>
          <Button onClick={handleDialogClose}>Continue</Button>
        </DialogActions>
      </Dialog>
      
      <Dialog open={confirmDialog.open} onClose={handleConfirmDialogClose}>
        <DialogTitle>Confirm Branch Deletion</DialogTitle>
        <DialogContent>
          <Typography>Are you sure you want to approve the deletion of branch <b>{confirmDialog.request?.branchCode}</b>?</Typography>
          <Typography sx={{mt: 1, color: 'red'}}>This action will permanently reset and delete all associated reports and cannot be undone.</Typography>
        </DialogContent>
        <DialogActions>
          <Button onClick={handleConfirmDialogClose}>Cancel</Button>
          <Button onClick={handleConfirmDelete} color="error" variant="contained">Confirm Delete</Button>
        </DialogActions>
      </Dialog>
    </>
  );
};

export default FRTCheckerAddDeleteReq;

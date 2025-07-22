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
//import FrtCheckerLayout from "../Layouts/FrtCheckerLayout";

const iv = crypto.getRandomValues(new Uint8Array(12));
const ivBase64 = btoa(String.fromCharCode.apply(null, iv));
const salt = crypto.getRandomValues(new Uint8Array(16));
const saltBase64 = btoa(String.fromCharCode.apply(null, salt));

// Dummy data simulating the initial list of requests
const initialRequests = [];

// Dummy data simulating the reports list for a 'Delete Branch' request
const reportsForDelete = [];

function Row(props) {
  const { row, onAction } = props;
  const [open, setOpen] = useState(false);
  const [reports, setReports] = useState([]);
  const [loadingReports, setLoadingReports] = useState(false);
  const isDelete = row.requestType === "Delete Branch";

  const handleRowClick = () => {
    const shouldOpen = !open;
    setOpen(shouldOpen);
    if (shouldOpen && isDelete && reports.length === 0) {
      setLoadingReports(true);

      setTimeout(() => {
        setReports(reportsForDelete);
        setLoadingReports(false);
      }, 1500);
    }
  };

  const rowStyle = {
    backgroundColor: isDelete ? "#ffebee" : "#e8f5e9", // Red for delete, Green for add
    "& > *": { borderBottom: "unset" },
  };

  return (
    <React.Fragment>
      <TableRow sx={rowStyle}>
        <TableCell>
          <IconButton
            aria-label="expand row"
            size="small"
            onClick={handleRowClick}
          >
            {open ? <KeyboardArrowUpIcon /> : <KeyboardArrowDownIcon />}
          </IconButton>
        </TableCell>
        <TableCell>{row.req_Id}</TableCell>
        <TableCell>{row.branchCode}</TableCell>
        <TableCell>{row.branchName}</TableCell>
        <TableCell>{row.requestType}</TableCell>
        <TableCell>{row.status}</TableCell>
        <TableCell>{row.requestedOn}</TableCell>
      </TableRow>
      <TableRow>
        <TableCell style={{ paddingBottom: 0, paddingTop: 0 }} colSpan={7}>
          <Collapse in={open} timeout="auto" unmountOnExit>
            <Box
              sx={{
                margin: 1,
                p: 2,
                backgroundColor: isDelete ? "#fff0f0" : "#f0fff0",
              }}
            >
              <Grid container spacing={2} sx={{ mb: 2 }}>
                <Grid item xs={3}>
                  <b>Circle:</b> {row.circleCode}
                </Grid>
                <Grid item xs={3}>
                  <b>RO:</b> {row.roCode}
                </Grid>
                <Grid item xs={3}>
                  <b>Audited Status:</b> {row.auditStatus}
                </Grid>
                <Grid item xs={3}>
                  <b>Requested By:</b> {row.requestedBy}
                </Grid>
                <Grid item xs={12}>
                  <b>No of Branches in Circle:</b> {row.noOfBranches}
                </Grid>
              </Grid>

              {isDelete && (
                <Box sx={{ my: 2 }}>
                  {loadingReports ? (
                    <CircularProgress size={24} />
                  ) : (
                    <>
                      <Typography sx={{ color: "red", mb: 1 }}>
                        *Following reports will be deleted on branch deletion.
                      </Typography>
                      <TableContainer component={Paper}>
                        <Table size="small">
                          <TableHead>
                            <TableRow>
                              <TableCell>MODULE</TableCell>
                              <TableCell>REPORT</TableCell>
                              <TableCell>NAME</TableCell>
                            </TableRow>
                          </TableHead>
                          <TableBody>
                            {reports.map((report) => (
                              <TableRow key={report.REPORT}>
                                <TableCell>{report.MODULE}</TableCell>
                                <TableCell>{report.REPORT}</TableCell>
                                <TableCell>{report.NAME}</TableCell>
                              </TableRow>
                            ))}
                          </TableBody>
                        </Table>
                      </TableContainer>
                    </>
                  )}
                </Box>
              )}

              <Box
                sx={{
                  display: "flex",
                  justifyContent: "center",
                  mt: 2,
                  gap: 2,
                }}
              >
                <Button
                  variant="contained"
                  color="success"
                  onClick={() => onAction(row.req_Id, "approved")}
                >
                  Approve
                </Button>
                <Button
                  variant="contained"
                  color="error"
                  onClick={() => onAction(row.req_Id, "rejected")}
                >
                  Reject
                </Button>
              </Box>
            </Box>
          </Collapse>
        </TableCell>
      </TableRow>
    </React.Fragment>
  );
}

const FRTCheckerAddDeleteReq = () => {
  const [requests, setRequests] = useState([]);
  const [loading, setLoading] = useState(true);
  const [dialog, setDialog] = useState({ open: false, title: "", message: "" });

  useEffect(() => {
    document.title = "FRT | Scope Change Requests";
    setTimeout(() => {
      setRequests(initialRequests);
      setLoading(false);
    }, 1000);
  }, []);

  // const fetchReports = async () => {
  //   try {
  //     let jsonFormData = JSON.stringify({ branchCode: branchCode });

  //     await encrypt(iv, salt, jsonFormData).then(function (result) {
  //       jsonFormData = result;
  //     });

  //     let payload = {
  //       iv: ivBase64,
  //       salt: saltBase64,
  //       data: jsonFormData,
  //     };

  //     const response = await axios.post(
  //       "/Server/EditBranch/fetchReports",
  //       payload,
  //       {
  //         headers: {
  //           Authorization: `Bearer ${localStorage.getItem("token")}`,
  //         },
  //       }
  //     );

  //     if (
  //       response.data?.result &&
  //       response.data?.result?.reportList.length !== 0
  //     ) {
  //       setReportsList(response.data.result.reportList);
  //       setShowReports(true);
  //       handleDialogClose();
  //     } else if (
  //       response.data?.result &&
  //       response.data?.result?.reportList.length === 0
  //     ) {
  //       let samp = branchData;
  //       samp.SCOPE = "O";
  //       setbranchData({ ...samp });
  //       handleDialogClose();
  //     } else {
  //       setSnackbar({
  //         children: "Failed to fetch reports data.",
  //         severity: "error",
  //       });
  //       handleDialogClose();
  //     }
  //   } catch (e) {
  //     console.error(e);
  //     setSnackbar({
  //       children: "An error occurred. Please try again after some time.",
  //       severity: "error",
  //     });
  //     handleDialogClose();
  //   }
  // };

  const handleAction = (reqId, action) => {
    const request = requests.find((r) => r.req_Id === reqId);
    let finalCount = request.noOfBranches;
    let message = "";

    if (request.requestType === "Add Branch" && action === "approved") {
      finalCount++;
      message = `Branch ${request.branchCode} added to CRS Scope. The revised count of branches in ${request.circleCode} Circle is ${finalCount}.`;
    } else if (
      request.requestType === "Delete Branch" &&
      action === "approved"
    ) {
      finalCount--;
      message = `Branch ${request.branchCode} removed from CRS Scope. The revised count of branches in ${request.circleCode} Circle is ${finalCount}.`;
    } else {
      message = `The request to ${request.requestType} for branch ${request.branchCode} has been rejected.`;
    }

    setDialog({ open: true, title: `Request ${action}`, message });
    setRequests(requests.filter((r) => r.req_Id !== reqId));
  };

  const handleDialogClose = () => {
    setDialog({ open: false, title: "", message: "" });
  };

  return (
    <>
      <Container maxWidth={false}>
        <Typography variant="h5" sx={{ mb: 2 }}>
          CRS Scope Change Requests
        </Typography>
        <TableContainer component={Paper}>
          {loading ? (
            <Box sx={{ display: "flex", justifyContent: "center", p: 4 }}>
              <CircularProgress />
            </Box>
          ) : (
            <Table aria-label="collapsible table">
              <TableHead sx={{ backgroundColor: "#b9def0" }}>
                <TableRow>
                  <TableCell />
                  <TableCell>
                    <b>Request ID</b>
                  </TableCell>
                  <TableCell>
                    <b>Branch</b>
                  </TableCell>
                  <TableCell>
                    <b>Branch Name</b>
                  </TableCell>
                  <TableCell>
                    <b>Request Type</b>
                  </TableCell>
                  <TableCell>
                    <b>Status</b>
                  </TableCell>
                  <TableCell>
                    <b>Requested on</b>
                  </TableCell>
                </TableRow>
              </TableHead>
              <TableBody>
                {requests.map((row) => (
                  <Row key={row.req_Id} row={row} onAction={handleAction} />
                ))}
              </TableBody>
            </Table>
          )}
        </TableContainer>
      </Container>
      <Dialog open={dialog.open} onClose={handleDialogClose}>
        <DialogTitle>{dialog.title}</DialogTitle>
        <DialogContent>
          <Alert severity="success">{dialog.message}</Alert>
        </DialogContent>
        <DialogActions>
          <Button onClick={handleDialogClose}>Continue</Button>
        </DialogActions>
      </Dialog>
    </>
  );
};

export default FRTCheckerAddDeleteReq;


/////////////////////////////////////////////////////////////////
api call to be analyzed to implement in above code

 const fetchRequests = async () => {
    setLoading(true);
    try {
      let jsonFormData = JSON.stringify({ quarterDate: user?.quarterEndDate });
      await encrypt(iv, salt, jsonFormData).then(function (result) {
        jsonFormData = result;
      });
      let payload = { iv: ivBase64, salt: saltBase64, data: jsonFormData };

      const response = await axios.post(
        "/Server/EditBranch/getPendingRequests",
        payload,
        {
          headers: { Authorization: `Bearer ${localStorage.getItem("token")}` },
        }
      );
      console.log("response.data.result", response.data.result);

      const mappedData = response.data.result.map((item) => ({
        ...item,
        id: item.AS_ID,
      }));
      setRequests(mappedData);
    } catch (error) {
      console.error("Failed to fetch requests:", error);
      setDialog({
        open: true,
        message: "Failed to load requests. Please try again later.",
        severity: "error",
      });
      setRequests([]);
    } finally {
      setLoading(false);
    }
  };

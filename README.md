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
import FrtCheckerLayout from "../Layouts/FrtCheckerLayout";

// Dummy data simulating the initial list of requests
const initialRequests = [
  {
    req_Id: "BR-ADD-101",
    branchCode: "12345",
    branchName: "MUMBAI-FORT-MAIN",
    requestType: "Add Branch",
    status: "Pending",
    requestedOn: "15-Mar-2023",
    circleCode: "MUMBAI",
    roCode: "MUM-RO-1",
    auditStatus: "Audited",
    requestedBy: "V1009204",
    noOfBranches: 150,
  },
  {
    req_Id: "BR-DEL-102",
    branchCode: "54321",
    branchName: "DELHI-CONNAUGHT-PLACE",
    requestType: "Delete Branch",
    status: "Pending",
    requestedOn: "16-Mar-2023",
    circleCode: "DELHI",
    roCode: "DEL-RO-2",
    auditStatus: "Non Audited",
    requestedBy: "V1009205",
    noOfBranches: 120,
  },
  {
    req_Id: "BR-ADD-103",
    branchCode: "67890",
    branchName: "BENGALURU-INDIRANAGAR",
    requestType: "Add Branch",
    status: "Pending",
    requestedOn: "17-Mar-2023",
    circleCode: "BENGALURU",
    roCode: "BEN-RO-3",
    auditStatus: "IFCOFR Audited",
    requestedBy: "V1009206",
    noOfBranches: 115,
  },
];

// Dummy data simulating the reports list for a 'Delete Branch' request
const reportsForDelete = [
  { MODULE: "LFAR", REPORT: "RN-50", NAME: "Certificate for capturing details of all the assets repossessed" },
  { MODULE: "CRS", REPORT: "RN-48", NAME: "Details of Branch / ATM / Office Premises / Staff Quarters" },
];

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
      // Simulate API call
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
          <IconButton aria-label="expand row" size="small" onClick={handleRowClick}>
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
            <Box sx={{ margin: 1, p: 2, backgroundColor: isDelete ? "#fff0f0" : "#f0fff0" }}>
              <Grid container spacing={2} sx={{ mb: 2 }}>
                <Grid item xs={3}><b>Circle:</b> {row.circleCode}</Grid>
                <Grid item xs={3}><b>RO:</b> {row.roCode}</Grid>
                <Grid item xs={3}><b>Audited Status:</b> {row.auditStatus}</Grid>
                <Grid item xs={3}><b>Requested By:</b> {row.requestedBy}</Grid>
                <Grid item xs={12}><b>No of Branches in Circle:</b> {row.noOfBranches}</Grid>
              </Grid>

              {isDelete && (
                <Box sx={{ my: 2 }}>
                  {loadingReports ? (
                    <CircularProgress size={24} />
                  ) : (
                    <>
                      <Typography sx={{ color: "red", mb: 1 }}>*Following reports will be deleted on branch deletion.</Typography>
                      <TableContainer component={Paper}>
                        <Table size="small">
                          <TableHead><TableRow><TableCell>MODULE</TableCell><TableCell>REPORT</TableCell><TableCell>NAME</TableCell></TableRow></TableHead>
                          <TableBody>
                            {reports.map((report) => (
                              <TableRow key={report.REPORT}><TableCell>{report.MODULE}</TableCell><TableCell>{report.REPORT}</TableCell><TableCell>{report.NAME}</TableCell></TableRow>
                            ))}
                          </TableBody>
                        </Table>
                      </TableContainer>
                    </>
                  )}
                </Box>
              )}
              
              <Box sx={{ display: "flex", justifyContent: "center", mt: 2, gap: 2 }}>
                <Button variant="contained" color="success" onClick={() => onAction(row.req_Id, "approved")}>Approve</Button>
                <Button variant="contained" color="error" onClick={() => onAction(row.req_Id, "rejected")}>Reject</Button>
              </Box>
            </Box>
          </Collapse>
        </TableCell>
      </TableRow>
    </React.Fragment>
  );
}

const FRTBranchReq = () => {
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

  const handleAction = (reqId, action) => {
    const request = requests.find((r) => r.req_Id === reqId);
    let finalCount = request.noOfBranches;
    let message = "";

    if (request.requestType === 'Add Branch' && action === 'approved') {
        finalCount++;
        message = `Branch ${request.branchCode} added to CRS Scope. The revised count of branches in ${request.circleCode} Circle is ${finalCount}.`;
    } else if (request.requestType === 'Delete Branch' && action === 'approved') {
        finalCount--;
        message = `Branch ${request.branchCode} removed from CRS Scope. The revised count of branches in ${request.circleCode} Circle is ${finalCount}.`;
    } else {
        message = `The request to ${request.requestType} for branch ${request.branchCode} has been rejected.`;
    }

    setDialog({ open: true, title: `Request ${action}`, message });
    setRequests(requests.filter((r) => r.req_Id !== reqId));
  };
  
  const handleDialogClose = () => {
    setDialog({ open: false, title: '', message: '' });
  };

  return (
    <FrtCheckerLayout>
      <Container maxWidth={false}>
        <Typography variant="h5" sx={{ mb: 2 }}>CRS Scope Change Requests</Typography>
        <TableContainer component={Paper}>
          {loading ? (
            <Box sx={{ display: 'flex', justifyContent: 'center', p: 4 }}><CircularProgress /></Box>
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
                  <Row key={row.req_Id} row={row} onAction={handleAction} />
                ))}
              </TableBody>
            </Table>
          )}
        </TableContainer>
      </Container>
      <Dialog open={dialog.open} onClose={handleDialogClose}>
        <DialogTitle>{dialog.title}</DialogTitle>
        <DialogContent><Alert severity="success">{dialog.message}</Alert></DialogContent>
        <DialogActions><Button onClick={handleDialogClose}>Continue</Button></DialogActions>
      </Dialog>
    </FrtCheckerLayout>
  );
};

export default FRTBranchReq;

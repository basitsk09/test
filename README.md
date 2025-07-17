import React, { useState, useEffect } from "react";
import {
  Accordion,
  AccordionDetails,
  AccordionSummary,
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
} from "@mui/material";
import ExpandMoreIcon from "@mui/icons-material/ExpandMore";
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
  { MODULE: "LFAR", REPORT_NAME: "ADVANCES-REPORT", REPORT_DESC: "Report on Advances granted" },
  { MODULE: "LFAR", REPORT_NAME: "DEPOSITS-ANALYSIS", REPORT_DESC: "Analysis of various deposits" },
  { MODULE: "CRS", REPORT_NAME: "CRS-COMPLIANCE-01", REPORT_DESC: "Compliance status for Q1" },
];

const FRTBranchReq = () => {
  const [requests, setRequests] = useState([]);
  const [loading, setLoading] = useState(true);
  const [expanded, setExpanded] = useState(false);
  const [dialog, setDialog] = useState({ open: false, title: "", message: "" });
  const [reports, setReports] = useState({});
  const [loadingReports, setLoadingReports] = useState({});

  useEffect(() => {
    document.title = "FRT | Scope Change Requests";
    // Simulate fetching initial request list
    setTimeout(() => {
      setRequests(initialRequests);
      setLoading(false);
    }, 1000);
  }, []);

  const handleAccordionChange = (panelId) => async (event, isExpanded) => {
    setExpanded(isExpanded ? panelId : false);
    const request = requests.find((r) => r.req_Id === panelId);

    // If expanding a 'Delete Branch' request for the first time, fetch its reports
    if (isExpanded && request.requestType === "Delete Branch" && !reports[panelId]) {
      setLoadingReports((prev) => ({ ...prev, [panelId]: true }));
      // Simulate API call to fetch reports
      setTimeout(() => {
        setReports((prev) => ({ ...prev, [panelId]: reportsForDelete }));
        setLoadingReports((prev) => ({ ...prev, [panelId]: false }));
      }, 1500);
    }
  };

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
    setExpanded(false);
  };
  
  const handleDialogClose = () => {
    setDialog({ open: false, title: '', message: '' });
  };

  const getAccordionStyles = (type) => {
    if (type === "Add Branch") {
      return {
        "&.Mui-expanded": { backgroundColor: "#f0fff0" },
        "& .MuiAccordionSummary-root": { backgroundColor: "#e8f5e9" },
      };
    }
    if (type === "Delete Branch") {
      return {
        "&.Mui-expanded": { backgroundColor: "#fff0f0" },
        "& .MuiAccordionSummary-root": { backgroundColor: "#ffebee" },
      };
    }
    return {};
  };

  return (
    <FrtCheckerLayout>
      <Container maxWidth="xl">
        <Typography variant="h5" sx={{ mb: 2 }}>
          CRS Scope Change Requests
        </Typography>
        <Paper sx={{ p: 2 }}>
          {loading && (
            <Box sx={{ display: 'flex', justifyContent: 'center', p: 4 }}>
              <CircularProgress />
            </Box>
          )}
          {!loading && requests.map((req) => (
            <Accordion
              key={req.req_Id}
              expanded={expanded === req.req_Id}
              onChange={handleAccordionChange(req.req_Id)}
              sx={getAccordionStyles(req.requestType)}
            >
              <AccordionSummary expandIcon={<ExpandMoreIcon />}>
                <Grid container spacing={2} alignItems="center">
                  <Grid item xs={1.5}><Typography variant="body2"><b>{req.req_Id}</b></Typography></Grid>
                  <Grid item xs={1}><Typography variant="body2">{req.branchCode}</Typography></Grid>
                  <Grid item xs={3.5}><Typography variant="body2">{req.branchName}</Typography></Grid>
                  <Grid item xs={2}><Typography variant="body2">{req.requestType}</Typography></Grid>
                  <Grid item xs={1.5}><Typography variant="body2">{req.status}</Typography></Grid>
                  <Grid item xs={2.5}><Typography variant="body2">{req.requestedOn}</Typography></Grid>
                </Grid>
              </AccordionSummary>
              <AccordionDetails>
                <Grid container spacing={2} sx={{ mb: 2, p: 2, border: '1px solid #eee', borderRadius: '4px' }}>
                  <Grid item xs={3}><Typography variant="body2"><b>Circle:</b> {req.circleCode}</Typography></Grid>
                  <Grid item xs={3}><Typography variant="body2"><b>RO:</b> {req.roCode}</Typography></Grid>
                  <Grid item xs={3}><Typography variant="body2"><b>Audited Status:</b> {req.auditStatus}</Typography></Grid>
                  <Grid item xs={3}><Typography variant="body2"><b>Requested By:</b> {req.requestedBy}</Typography></Grid>
                  <Grid item xs={12}><Typography variant="body2"><b>No of Branches in Circle:</b> {req.noOfBranches}</Typography></Grid>
                </Grid>
                
                {req.requestType === "Delete Branch" && expanded === req.req_Id && (
                  <Box sx={{ my: 2 }}>
                    {loadingReports[req.req_Id] ? (
                      <CircularProgress size={24} />
                    ) : (
                      reports[req.req_Id] && (
                        <>
                          <Typography sx={{ color: 'red.800', mb: 1 }}>*Following reports will be deleted on branch deletion.</Typography>
                          <TableContainer component={Paper}>
                            <Table size="small">
                              <TableHead sx={{ backgroundColor: '#f5f5f5' }}>
                                <TableRow><TableCell>Module</TableCell><TableCell>Report Name</TableCell><TableCell>Description</TableCell></TableRow>
                              </TableHead>
                              <TableBody>
                                {reports[req.req_Id].map((report) => (
                                  <TableRow key={report.REPORT_NAME}><TableCell>{report.MODULE}</TableCell><TableCell>{report.REPORT_NAME}</TableCell><TableCell>{report.REPORT_DESC}</TableCell></TableRow>
                                ))}
                              </TableBody>
                            </Table>
                          </TableContainer>
                        </>
                      )
                    )}
                  </Box>
                )}

                <Box sx={{ display: "flex", justifyContent: "center", mt: 2, gap: 2 }}>
                  <Button variant="contained" color="success" onClick={() => handleAction(req.req_Id, "approved")}>Approve</Button>
                  <Button variant="contained" color="error" onClick={() => handleAction(req.req_Id, "rejected")}>Reject</Button>
                </Box>
              </AccordionDetails>
            </Accordion>
          ))}
        </Paper>
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
    </FrtCheckerLayout>
  );
};

export default FRTBranchReq;


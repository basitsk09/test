 @Query(value = "select 'CRS' module, report_name report, " +
            "(SELECT report_desc FROM report_master where report_id=report_master_id) name, status pending " +
            "from reports_master_list where branch_code = :branchCode and quarter_date = to_date(:quarterDate ,'dd/mm/yyyy') " +
            "union all " +
            "select 'TAR' module, report_name, " +
            "(SELECT report_desc FROM tar_report_master where report_id=report_master_id) report_desc, status " +
            "from tar_reports_master_list where branch_code = :branchCode and quarter_date = to_date(:quarterDate ,'dd/mm/yyyy') " +
            "union all " +
            "select 'LFAR' module, report_name, " +
            "(SELECT report_desc FROM LFAR_report_master where report_id=report_master_id) report_desc, status " +
            "from lfar_reports_master_list where branch_code = :branchCode and quarter_date = to_date(:quarterDate ,'dd/mm/yyyy')", nativeQuery = true)
    List<Object[]> findReportsForBranch(@Param("quarterDate") String quarterDate, @Param("branchCode") String branchCode);
	
	////////////////////////////////////////////////////////////
	instead of above query i am making use of below query to show reports for deletion.
	
	 @Query(value = """
                        select
                        'CRS' as module,
                        rcm.report_id,
                        rcm.report_name,
                        rcm.report_desc,
                        rcs.submission_id,
                        rcs.current_status,
                        rcm.REPORT_JRXML_NAME,
                        rcm.report_type,
                        rcm.audited,
                        nvl(rcs.nil_report, 'N') nil_report
                        from report_submission rcs, report_master rcm
                        where rcs.report_id_fk = rcm.report_id
                        and rcs.branch_code = :branchCode
                        and rcs.report_date = to_date(:quarterDate, 'dd/MM/yyyy')
                        UNION
                        select
                        'TAR' as module,
                        rtm.REPORT_ID ,
                        rtm.REPORT_NAME,
                        rtm.REPORT_DESC,
                        CAST(rts.REPORT_ID AS int) as SUBMISSION_ID,
                        rts.STATUS as CURRENT_STATUS,
                        nvl(rts.NIL_REPORT_FLAG,'N') NIL_REPORT,
                        rtm.REPORT_JRXML_NAME,
                        rtm.report_type,
                        rtm.audited
                        from TAR_REPORTS_MASTER_LIST rts, TAR_REPORT_MASTER rtm
                        where rts.report_master_id = rtm.REPORT_ID
                        and rts.BRANCH_CODE =:branchCode
                        and rts.QUARTER_DATE =to_date(:quarterDate, 'dd/MM/yyyy')
                        UNION
                        select
                        'LFAR' as module,
                        rm.REPORT_ID,
                        rm.report_name,
                        rm.report_desc,
                        CAST(rml.REPORT_ID AS int) as SUBMISSION_ID,
                        rml.STATUS as CURRENT_STATUS,
                        rm.REPORT_JRXML_NAME,
                        rm.REPORT_TYPE,
                        rm.AUDITED,
                        nvl(rml.NIL_REPORT_FLAG, 'N') NIL_REPORT
                        from lfar_reports_master_list rml, lfar_report_master rm
                        where rml.REPORT_MASTER_ID = rm.REPORT_ID
                        and rml.BRANCH_CODE = :branchCode
                        and rml.STATUS = '50'
                        and rml.QUARTER_DATE = to_date(:quarterDate, 'dd/MM/yyyy')
                        order by module,report_name""", nativeQuery = true)
        List<Map<String, Object>> getAllReportList(@Param("branchCode") String branchCode,
                        @Param("quarterDate") String quarterDate);
///////////////////////////////////////////////////////////////////////////////////////////////////

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

      const response = await axios.post(
        "/Server/EditBranch/deleteReportList",
        payload,
        {
          headers: { Authorization: `Bearer ${localStorage.getItem("token")}` },
        }
      );

      const rawData = response.data.data;

      if (rawData && rawData.length > 1) {
        const headers = rawData[0];
        const dataRows = rawData.slice(1);
        const formattedReports = dataRows.map((dataRow) => {
          let reportObj = {};
          headers.forEach((header, index) => {
            if (header === "pending") {
              if (dataRow[index].split("")[0] === "1") {
                dataRow[index] = "Maker";
              } else if (dataRow[index].split("")[0] === "2") {
                dataRow[index] = "Branch Manager";
              } else if (dataRow[index].split("")[0] === "3") {
                dataRow[index] = "Branch Auditor";
              } else if (dataRow[index].split("")[0] === "4") {
                dataRow[index] = "RO Manager";
              } else if (dataRow[index].split("")[0] === "5") {
                dataRow[index] = "Freezed";
              }
            }
            reportObj[header] = dataRow[index];
          });
          return reportObj;
        });
        setReports(formattedReports);
      }
    } catch (error) {
      console.error("Failed to fetch reports for branch:", error);
      // Optionally show an error message
    } finally {
      setLoadingReports(false);
    }
  };

  const handleRowClick = () => {
    const shouldOpen = !open;
    setOpen(shouldOpen);
    // Fetch reports only when a 'Delete' row is opened for the first time
    if (shouldOpen && isDelete && reports.length === 0) {
      fetchReportsForBranch();
    }
  };

  const rowStyle = {
    backgroundColor: isDelete ? "#ffebee" : "#e8f5e9",
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
            <Box
              sx={{
                margin: 1,
                p: 2,
                backgroundColor: isDelete ? "#fff0f0" : "#f0fff0",
              }}
            >
              <Grid container spacing={2} sx={{ mb: 2 }}>
                <Grid item xs={3}>
                  <b>Circle:</b> {row.circleName}
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
                  <b>No of Branches in Circle:</b> {row.circleCount}
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
                              <TableCell>PENDING WITH</TableCell>
                            </TableRow>
                          </TableHead>
                          <TableBody>
                            {reports.map((report, index) => (
                              <TableRow key={index}>
                                <TableCell>{report.module}</TableCell>
                                <TableCell>{report.report}</TableCell>
                                <TableCell>{report.name}</TableCell>
                                <TableCell>{report.pending}</TableCell>
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
                  onClick={() => onAction(row.requestId, "approve")}
                >
                  Approve
                </Button>
                <Button
                  variant="contained"
                  color="error"
                  onClick={() => onAction(row.requestId, "reject")}
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

// --- Main Component ---
const FRTCheckerAddDeleteReq = () => {
  const [requests, setRequests] = useState([]);
  const [loading, setLoading] = useState(true);
  const [dialog, setDialog] = useState({ open: false, title: "", message: "" });
  const user = JSON.parse(localStorage.getItem("user")); // Assuming user info is in localStorage

  // **API Call to fetch the initial list of pending requests**
  const fetchBranchRequests = async () => {
    setLoading(true);
    try {
      const payload = await getEncryptedPayload({
        quarterDate: user?.quarterEndDate,
      });

      const response = await axios.post(
        "/Server/EditBranch/getBranchRequests",
        payload,
        {
          headers: { Authorization: `Bearer ${localStorage.getItem("token")}` },
        }
      );
      // Assuming the backend sends data in a 'data' property
      console.log(response.data.data);
      setRequests(response.data.data || []);
    } catch (error) {
      console.error("Failed to fetch requests:", error);
      setDialog({
        open: true,
        title: "Error",
        message: "Failed to load requests. Please try again.",
      });
      setRequests([]);
    } finally {
      setLoading(false);
    }
  };

  useEffect(() => {
    document.title = "FRT | Scope Change Requests";
    fetchBranchRequests();
  }, []);

  // **API Call to handle Approve/Reject actions**
  const handleAction = async (reqId, action) => {
    const request = requests.find((r) => r.requestId === reqId);
    console.log(request);
    try {
      // NOTE: A new backend endpoint is assumed here for handling these specific actions.

      // console.log("payload", payload);
      // const response = await axios.post(`/api/frt-branch/${action}`, payload);

      const payload = await getEncryptedPayload({
        requestId: reqId,
        branchCode: request.branchCode,
        circleCode: request.circleCode,
        requestType: request.requestType,
        auditStatus: request.auditStatus,
        requestedById: request.requestedBy,
        quarterEndDate: user.quarterEndDate,
      });

      const response = await axios.post(
        `/Server/EditBranch/${action}`,
        payload,
        {
          headers: { Authorization: `Bearer ${localStorage.getItem("token")}` },
        }
      );
      // Assuming the backend sends data in a 'data' property
      console.log(response.data.data);

      // Show the success message from the backend
      setDialog({
        open: true,
        title: `Request ${action}`,
        message: response.data.message,
      });
      // Refresh the list of requests after a successful action
      fetchBranchRequests();
    } catch (error) {
      console.error(`Failed to ${action} request:`, error);
      setDialog({
        open: true,
        title: "Action Failed",
        message:
          error.response?.data?.message ||
          `An error occurred while processing the request.`,
      });
    }
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
                  <Row
                    key={row.requestId}
                    row={{ ...row, quarterDate: user?.quarterEndDate }}
                    onAction={handleAction}
                  />
                ))}
              </TableBody>
            </Table>
          )}
        </TableContainer>
      </Container>
      <Dialog open={dialog.open} onClose={handleDialogClose}>
        <DialogTitle>{dialog.title}</DialogTitle>
        <DialogContent>
          {/* Using Alert to give a better visual feedback */}
          <Alert
            severity={
              dialog.title.toLowerCase().includes("error") ? "error" : "success"
            }
          >
            {dialog.message}
          </Alert>
        </DialogContent>
        <DialogActions>
          <Button onClick={handleDialogClose}>Continue</Button>
        </DialogActions>
      </Dialog>
    </>
  );
};

export default FRTCheckerAddDeleteReq;
//////////////////////////////////////////////////////////////////
integrate below call into above component

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
  
  
  /////////////////////////////////////////////////////////////////
  this call is to delete report list backend is already there. in tegrate this also in above component
  
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
            method: "resetAll",
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

          if (
            response.data.result?.reset === 1 ||
            response.data.result?.Reset === 1 ||
            response.data.result?.status
          )
            count++;
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

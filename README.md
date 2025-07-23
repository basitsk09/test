 public boolean addBranchReq(FRTBranchReq frt, SessionBean sessionBean, String reqId, String branchCode, String reqType){

        TransactionDefinition def = new DefaultTransactionDefinition();
        TransactionStatus status = transactionManager.getTransaction(def);

        //JdbcTemplate jdbcTemplateObject = new JdbcTemplate(dataSource);
        boolean flag= false;
        String auditStatus = frt.getAuditStatus();
        if(auditStatus.equalsIgnoreCase("Non-Audited")){
            auditStatus ="N";
        }
        else if(auditStatus.equalsIgnoreCase("Audited (IFCOFR)")){
            auditStatus ="I";
        }
        if(auditStatus.equalsIgnoreCase("Audited (Non-IFCOFR)")){
            auditStatus ="Y";
        }
        String marQuarter = sessionBean.getQuarterEndDate().substring(3,5);
        String reqBy = frt.getRequestedById();
        log.info("auditStatus: "+auditStatus+" reqBy: "+reqBy);
        List<Object[]> inputList1 = new ArrayList<Object[]>();
        List<Object[]> inputList2 = new ArrayList<Object[]>();
        List<Object[]> inputList3 = new ArrayList<Object[]>();

        try {
            String insertBM = "insert into branch_master(select BRANCHNO , REGION_NO ,BR_NAME,MANAGERS_NAME, ADDRESS_1 ,ADDRESS_2 ,ADDRESS_3 ,POST_CODE , " +
                    "PHONE_NO ,NETWORK_CODE ,STD_CODE ,CIRCLE_CODE ,MODULE_CODE , REGION_CODE ,VOIP_PHONE_NO ,MAN_RES_PHONE ,MAN_MOBI_PHONE , " +
                    "INST ,case when CRS_AUDITABLE='Y' then 'Y' when CRS_AUDITABLE='I' then 'Y' else 'N' end from cbs_brhm " +
                    "where branchno=?)";

            String insertLBM = "insert into lfar_branch(select BRANCHNO , REGION_NO ,BR_NAME ,MANAGERS_NAME,ADDRESS_1 ,ADDRESS_2 ,ADDRESS_3 ,POST_CODE ," +
                    "PHONE_NO ,NETWORK_CODE ,STD_CODE ,CIRCLE_CODE ,MODULE_CODE , REGION_CODE ,VOIP_PHONE_NO ,MAN_RES_PHONE ,MAN_MOBI_PHONE ," +
                    "INST ,case when CRS_AUDITABLE='Y' then 'Y' " +
                    "when CRS_AUDITABLE='I' then 'Y' else 'N' end, TO_DATE(?,'dd/mm/yyyy') branch_date from cbs_brhm where branchno=?)";

            String insertTBM = "insert into tar_branch(select BRANCHNO , REGION_NO ,BR_NAME ,MANAGERS_NAME,ADDRESS_1 ,ADDRESS_2 ,ADDRESS_3 ,POST_CODE ," +
                    "PHONE_NO ,NETWORK_CODE ,STD_CODE ,CIRCLE_CODE ,MODULE_CODE ,REGION_CODE ,VOIP_PHONE_NO ,MAN_RES_PHONE ,MAN_MOBI_PHONE ," +
                    "INST , case when CRS_AUDITABLE='Y' then 'Y' when CRS_AUDITABLE='I' then 'Y' else 'N' end, " +
                    "TO_DATE(?,'dd/mm/yyyy') branch_date from cbs_brhm where branchno=?)";

            String insertBRDets = "insert into br_details (CRS_BRANCHCODE, CRS_CIRCLECODE )" +
                    "select BRANCHNO , CIRCLE_CODE  from cbs_brhm WHERE branchno=?";

            String insertIfcofr = "insert into crs_ifcofr_audit (IFCOFR_DATE, IFCOFR_BRANCH, IFCOFR_AUDIT_FLAG, IFCOFR_UPDATED_BY) " +
                    "values(to_date(?,'dd/mm/yyyy'),?,?,?)";

            Object[] temp1 = {branchCode};
            inputList1.add(temp1);

            Object[] temp2 = {sessionBean.getQuarterEndDate(),branchCode};
            inputList2.add(temp2);

            Object[] temp3;
            if (auditStatus.equalsIgnoreCase("I")) {
                temp3 = new Object[]{sessionBean.getQuarterEndDate(), branchCode, "Y", reqBy};
                inputList3.add(temp3);
            }
            else {
                temp3 = new Object[]{sessionBean.getQuarterEndDate(), branchCode, "N", reqBy};
                inputList3.add(temp3);
            }


            jdbcTemplateObject.batchUpdate(insertBM, inputList1);
            jdbcTemplateObject.batchUpdate(insertBRDets, inputList1);
            jdbcTemplateObject.batchUpdate(insertIfcofr, inputList3);
            if (marQuarter.equalsIgnoreCase("03")) {
                jdbcTemplateObject.batchUpdate(insertLBM, inputList2);
                jdbcTemplateObject.batchUpdate(insertTBM, inputList2);
            }
            transactionManager.commit(status);
            flag=true;
            log.info("after update complete");
        }
        catch(Exception sqle) {
            sqle.printStackTrace();
            flag=false;
            transactionManager.rollback(status);
        }
        return flag;
    }
/////////////////////////////////////////////////////

 public boolean acceptReq(FRTBranchReq frt, SessionBean sessionBean, String reqId, String branchCode, String reqType){
        boolean flag = false;
        if(reqType.equalsIgnoreCase("Add Branch")){
            log.info("Add Branch Service File");
            boolean add = frtBranchReqDao.addBranchReq(frt, sessionBean,reqId,branchCode,reqType);
            if (add){
                boolean approve = frtBranchReqDao.approveReq(reqId);
                if(approve){
                    flag = true;
                }
                else {
                    flag = false;
                }
            }
        }
        else{
            log.info("Delete Branch Service File");
            boolean delete = frtBranchReqDao.deleteBranchReq(frt,sessionBean,reqId,branchCode,reqType);
            if (delete){
                boolean approve = frtBranchReqDao.approveReq(reqId);
                if(approve){
                    flag = true;
                }
                else {
                    flag = false;
                }
            }
        }
        return flag;
    }

////////////////////////////////////////////////////
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
    try {
      // NOTE: A new backend endpoint is assumed here for handling these specific actions.
      const response = await axios.post(`/api/frt-branch/${action}`, {
        requestId: reqId,
        branchCode: request.branchCode,
        circleCode: request.circleCode,
        requestType: request.requestType,
      });

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

import * as React from "react";
import { useState, useEffect } from "react";
import {
  Box,
  Button,
  Checkbox,
  Container,
  Dialog,
  DialogActions,
  DialogContent,
  DialogTitle,
  Alert,
  Paper,
  Table,
  TableBody,
  TableCell,
  TableContainer,
  TableHead,
  TableRow,
  TablePagination,
  Typography,
  CircularProgress,
} from "@mui/material";
import axios from "axios";
import { encrypt } from "../Security/AES-GCM256";

const iv = crypto.getRandomValues(new Uint8Array(12));
const ivBase64 = btoa(String.fromCharCode.apply(null, iv));
const salt = crypto.getRandomValues(new Uint8Array(16));
const saltBase64 = btoa(String.fromCharCode.apply(null, salt));

// Table Header Component
const EnhancedTableHead = (props) => {
  const { onSelectAllClick, numSelected, rowCount } = props;

  return (
    <TableHead>
      <TableRow sx={{ backgroundColor: "#b9def0" }}>
        <TableCell padding="checkbox" align="center">
          <Box
            sx={{ display: "flex", alignItems: "center", whiteSpace: "nowrap" }}
          >
            <Checkbox
              color="primary"
              indeterminate={numSelected > 0 && numSelected < rowCount}
              checked={rowCount > 0 && numSelected === rowCount}
              onChange={onSelectAllClick}
              inputProps={{ "aria-label": "select all requests" }}
            />
            <b>Select All</b>
          </Box>
        </TableCell>
        <TableCell align="center">
          <b>Req ID</b>
        </TableCell>
        <TableCell align="center">
          <b>Branch Code</b>
        </TableCell>
        <TableCell align="left">
          <b>Branch Name</b>
        </TableCell>
        <TableCell align="center">
          <b>Existing Status</b>
        </TableCell>
        <TableCell align="center">
          <b>New Requested Status</b>
        </TableCell>
        <TableCell align="center">
          <b>Request Status</b>
        </TableCell>
        <TableCell align="center">
          <b>Requested On</b>
        </TableCell>
      </TableRow>
    </TableHead>
  );
};

const FRTAuditStatusReq = () => {
  const [requests, setRequests] = useState([]);
  const [selected, setSelected] = useState([]);
  const [loading, setLoading] = useState(true);
  const [page, setPage] = useState(0);
  const [rowsPerPage, setRowsPerPage] = useState(5);
  const user = JSON.parse(localStorage.getItem("user"));
  const [dialog, setDialog] = useState({
    open: false,
    message: "",
    severity: "success",
  });

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
      // Map the response to ensure each row has a unique 'id' for selection
      const mappedData = response.data.result.map((item) => ({
        ...item,
        id: item.as_id,
      }));
      setRequests(mappedData);
    } catch (error) {
      console.error("Failed to fetch requests:", error);
      setDialog({
        open: true,
        message: "Failed to load requests. Please try again later.",
        severity: "error",
      });
      setRequests([]); // Clear data on error
    } finally {
      setLoading(false);
    }
  };

  useEffect(() => {
    document.title = "FRT | Audit Status Requests";
    fetchRequests();
  }, []);

  const handleSelectAllClick = (event) => {
    if (event.target.checked) {
      const newSelected = requests.map((n) => n.id);
      setSelected(newSelected);
      return;
    }
    setSelected([]);
  };

  const handleClick = (event, id) => {
    const selectedIndex = selected.indexOf(id);
    let newSelected = [];

    if (selectedIndex === -1) {
      newSelected = newSelected.concat(selected, id);
    } else if (selectedIndex === 0) {
      newSelected = newSelected.concat(selected.slice(1));
    } else if (selectedIndex === selected.length - 1) {
      newSelected = newSelected.concat(selected.slice(0, -1));
    } else if (selectedIndex > 0) {
      newSelected = newSelected.concat(
        selected.slice(0, selectedIndex),
        selected.slice(selectedIndex + 1)
      );
    }
    setSelected(newSelected);
  };

  const handleAction = async (action) => {
    setLoading(true);
    // 'action' will be 'approve' or 'reject'
    const endpoint = `/api/frt/${action}`;

    try {
      const response = await axios.post(
        endpoint,
        { requestIds: selected }, // Send selected IDs in the request body
        {
          headers: {
            Authorization: `Bearer ${localStorage.getItem("token")}`,
          },
        }
      );

      // Show success message from backend
      setDialog({
        open: true,
        message:
          response.data.message ||
          `Successfully performed action on ${selected.length} request(s).`,
        severity: "success",
      });
      setSelected([]); // Clear selection
      fetchRequests(); // Refresh the data table
    } catch (error) {
      console.error(`Failed to ${action} requests:`, error);
      setDialog({
        open: true,
        message:
          error.response?.data?.message || `Failed to ${action} requests.`,
        severity: "error",
      });
    } finally {
      setLoading(false);
    }
  };

  const handleDialogClose = () => {
    setDialog({ ...dialog, open: false });
  };

  const isSelected = (id) => selected.indexOf(id) !== -1;

  const emptyRows =
    page > 0 ? Math.max(0, (1 + page) * rowsPerPage - requests.length) : 0;

  return (
    <>
      <Container maxWidth={false}>
        <Paper sx={{ width: "100%", mb: 2, p: 2 }}>
          <Typography
            variant="h5"
            component="div"
            sx={{ mb: 2, textAlign: "left" }}
          >
            Audit Status Change Requests
          </Typography>
          <Box sx={{ display: "flex", justifyContent: "center", mb: 2 }}>
            <Button
              variant="contained"
              color="success"
              sx={{ mr: 2, width: "120px" }}
              onClick={() => handleAction("approve")}
              disabled={selected.length === 0 || loading}
            >
              Approve
            </Button>
            <Button
              variant="contained"
              color="error"
              sx={{ width: "120px" }}
              onClick={() => handleAction("reject")}
              disabled={selected.length === 0 || loading}
            >
              Reject
            </Button>
          </Box>
          <TableContainer>
            {loading ? (
              <Box
                sx={{
                  display: "flex",
                  justifyContent: "center",
                  alignItems: "center",
                  height: 370,
                }}
              >
                <CircularProgress />
              </Box>
            ) : (
              <Table sx={{ minWidth: 750 }} aria-labelledby="tableTitle">
                <EnhancedTableHead
                  numSelected={selected.length}
                  onSelectAllClick={handleSelectAllClick}
                  rowCount={requests.length}
                />
                <TableBody>
                  {requests
                    .slice(page * rowsPerPage, page * rowsPerPage + rowsPerPage)
                    .map((row) => {
                      const isItemSelected = isSelected(row.id);
                      const labelId = `enhanced-table-checkbox-${row.id}`;

                      return (
                        <TableRow
                          hover
                          onClick={(event) => handleClick(event, row.id)}
                          role="checkbox"
                          aria-checked={isItemSelected}
                          tabIndex={-1}
                          key={row.id}
                          selected={isItemSelected}
                          sx={{ cursor: "pointer" }}
                        >
                          <TableCell padding="checkbox">
                            <Checkbox
                              color="primary"
                              checked={isItemSelected}
                              inputProps={{ "aria-labelledby": labelId }}
                            />
                          </TableCell>
                          {/* Use keys from the backend map */}
                          <TableCell align="center">{row.as_rt_id}</TableCell>
                          <TableCell align="center">{row.branchCode}</TableCell>
                          <TableCell align="left">{row.branchName}</TableCell>
                          <TableCell align="center">{row.beforeSts}</TableCell>
                          <TableCell align="center">{row.afterSts}</TableCell>
                          <TableCell align="center">{row.reqSts}</TableCell>
                          <TableCell align="center">{row.reqOn}</TableCell>
                        </TableRow>
                      );
                    })}
                  {emptyRows > 0 && (
                    <TableRow style={{ height: 53 * emptyRows }}>
                      <TableCell colSpan={8} />
                    </TableRow>
                  )}
                  {requests.length === 0 && !loading && (
                    <TableRow>
                      <TableCell colSpan={8} align="center">
                        No pending requests found.
                      </TableCell>
                    </TableRow>
                  )}
                </TableBody>
              </Table>
            )}
          </TableContainer>
          <TablePagination
            rowsPerPageOptions={[5, 10, 25]}
            component="div"
            count={requests.length}
            rowsPerPage={rowsPerPage}
            page={page}
            onPageChange={(e, newPage) => setPage(newPage)}
            onRowsPerPageChange={(e) => {
              setRowsPerPage(parseInt(e.target.value, 10));
              setPage(0);
            }}
          />
        </Paper>
      </Container>
      <Dialog open={dialog.open} onClose={handleDialogClose}>
        <DialogTitle>
          {dialog.severity === "success" ? "Success" : "Error"}
        </DialogTitle>
        <DialogContent>
          <Alert severity={dialog.severity}>{dialog.message}</Alert>
        </DialogContent>
        <DialogActions>
          <Button onClick={handleDialogClose}>Close</Button>
        </DialogActions>
      </Dialog>
    </>
  );
};

export default FRTAuditStatusReq;



//////////////////////////////////////////////////////////
[
    {
        "ROCODE": "555555555",
        "CIRCLE_CODE": "007",
        "REQ_TRACKSTS": 1,
        "BEFORESTS": "Non-Audited",
        "REQSTS": "Pending",
        "BRANCHNAME": "DUMMY BRANCH",
        "AS_ID": 31430,
        "REQON": "2023-04-03T07:42:57.000+00:00",
        "AS_RT_ID": 103,
        "AFTERSTS": "IFCOFR Audited",
        "BRANCHCODE": "00067"
    },
    {
        "CIRCLE_CODE": "004",
        "ROCODE": "555555555",
        "AS_ID": 31431,
        "REQ_TRACKSTS": 1,
        "BEFORESTS": "Non-Audited",
        "AS_RT_ID": 105,
        "REQSTS": "Pending",
        "BRANCHNAME": "DUMMY BRANCH",
        "REQON": "2023-04-03T09:35:14.000+00:00",
        "BRANCHCODE": "00004",
        "AFTERSTS": "IFCOFR Audited"
    },
    {
        "ROCODE": "555555555",
        "REQ_TRACKSTS": 4,
        "BRANCHCODE": "01000",
        "REQON": "2025-07-08T11:13:52.000+00:00",
        "BEFORESTS": "Non-Audited",
        "REQSTS": "Pending",
        "BRANCHNAME": "DUMMY BRANCH",
        "AS_ID": 31462,
        "AFTERSTS": "Non-Audited",
        "AS_RT_ID": 325,
        "CIRCLE_CODE": "020"
    },
    {
        "ROCODE": "555555555",
        "REQ_TRACKSTS": 4,
        "REQON": "2025-07-08T11:13:52.000+00:00",
        "AS_ID": 31463,
        "BEFORESTS": "Non-Audited",
        "REQSTS": "Pending",
        "BRANCHNAME": "DUMMY BRANCH",
        "CIRCLE_CODE": "002",
        "AS_RT_ID": 325,
        "AFTERSTS": "IFCOFR Audited",
        "BRANCHCODE": "95441"
    },
    {
        "ROCODE": "555555555",
        "REQ_TRACKSTS": 4,
        "AS_RT_ID": 362,
        "BEFORESTS": "Non-Audited",
        "BRANCHCODE": "10109",
        "REQSTS": "Pending",
        "BRANCHNAME": "DUMMY BRANCH",
        "REQON": "2025-07-21T07:10:23.000+00:00",
        "AS_ID": 31482,
        "CIRCLE_CODE": "010",
        "AFTERSTS": "IFCOFR Audited"
    },
    {
        "ROCODE": "555555555",
        "REQ_TRACKSTS": 4,
        "CIRCLE_CODE": "006",
        "BEFORESTS": "Non-Audited",
        "REQSTS": "Pending",
        "BRANCHNAME": "DUMMY BRANCH",
        "AS_RT_ID": 363,
        "REQON": "2025-07-21T07:15:14.000+00:00",
        "AS_ID": 31483,
        "BRANCHCODE": "24545",
        "AFTERSTS": "IFCOFR Audited"
    }
]

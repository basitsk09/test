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
import FrtCheckerLayout from "../Layouts/FrtCheckerLayout";

// Dummy data to simulate API response based on JSP variables
const createDummyData = (
  id,
  branchCode,
  branchName,
  beforeSts,
  afterSts,
  reqSts,
  reqOn
) => {
  // Combines rt_id and id for a unique request identifier, similar to the JSP
  const reqId = `FRT-AS-${id}`;
  return { id, reqId, branchCode, branchName, beforeSts, afterSts, reqSts, reqOn };
};

const initialRows = [
  createDummyData(101, "7890", "MUMBAI MAIN", "Active", "Inactive", "Pending", "V1009204"),
  createDummyData(102, "4567", "DELHI-CP", "Active", "Inactive", "Pending", "V1009205"),
  createDummyData(103, "1234", "KOLKATA-PARK-ST", "Inactive", "Active", "Pending", "V1009206"),
  createDummyData(104, "5678", "CHENNAI-T-NAGAR", "Active", "Inactive", "Pending", "V1009207"),
  createDummyData(105, "9876", "BENGALURU-MG-ROAD", "Active", "Inactive", "Pending", "V1009208"),
];

// Table Header Component
const EnhancedTableHead = (props) => {
  const { onSelectAllClick, numSelected, rowCount } = props;

  return (
    <TableHead>
      <TableRow sx={{ backgroundColor: "#b9def0" }}>
        <TableCell padding="checkbox">
          <Box sx={{ display: 'flex', alignItems: 'center' }}>
            <Checkbox
              color="primary"
              indeterminate={numSelected > 0 && numSelected < rowCount}
              checked={rowCount > 0 && numSelected === rowCount}
              onChange={onSelectAllClick}
              inputProps={{ "aria-label": "select all requests" }}
            />
            <Typography variant="body2">Select All</Typography>
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
          <b>Requested Status</b>
        </TableCell>
        <TableCell align="center">
          <b>Requested By</b>
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
  const [dialog, setDialog] = useState({ open: false, message: "", severity: "success" });

  useEffect(() => {
    // Simulate fetching data
    document.title = "FRT | Audit Status Requests";
    setTimeout(() => {
      setRequests(initialRows);
      setLoading(false);
    }, 1500);
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

  const handleAction = (action) => {
    setLoading(true);
    // Simulate API call for approve/reject
    setTimeout(() => {
      const remainingRequests = requests.filter(
        (req) => !selected.includes(req.id)
      );
      setRequests(remainingRequests);
      const successfulCount = selected.length;
      setSelected([]);
      setLoading(false);
      setDialog({
        open: true,
        message: `Successfully ${action} ${successfulCount} request(s).`,
        severity: "success",
      });
    }, 1000);
  };

  const handleDialogClose = () => {
    setDialog({ ...dialog, open: false });
  };
  
  const isSelected = (id) => selected.indexOf(id) !== -1;

  const emptyRows = page > 0 ? Math.max(0, (1 + page) * rowsPerPage - requests.length) : 0;

  return (
    <FrtCheckerLayout>
      <Container maxWidth={false}>
        <Paper sx={{ width: "100%", mb: 2, p: 2 }}>
          <Typography variant="h5" component="div" sx={{ mb: 2, textAlign: 'center' }}>
            Audit Status Change Requests
          </Typography>
          <Box sx={{ display: 'flex', justifyContent: 'center', mb: 2 }}>
            <Button
              variant="contained"
              color="success"
              sx={{ mr: 2, width: '120px' }}
              onClick={() => handleAction("approved")}
              disabled={selected.length === 0}
            >
              Approve
            </Button>
            <Button
              variant="contained"
              color="error"
              sx={{ width: '120px' }}
              onClick={() => handleAction("rejected")}
              disabled={selected.length === 0}
            >
              Reject
            </Button>
          </Box>
          <TableContainer>
            {loading ? (
              <Box sx={{ display: 'flex', justifyContent: 'center', alignItems: 'center', height: 370 }}>
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
                        <TableCell align="center">{row.reqId}</TableCell>
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
        <DialogTitle>{dialog.severity === "success" ? "Success" : "Error"}</DialogTitle>
        <DialogContent>
          <Alert severity={dialog.severity}>{dialog.message}</Alert>
        </DialogContent>
        <DialogActions>
          <Button onClick={handleDialogClose}>Close</Button>
        </DialogActions>
      </Dialog>
    </FrtCheckerLayout>
  );
};

export default FRTAuditStatusReq;

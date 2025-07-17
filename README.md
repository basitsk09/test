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
  Toolbar,
  Typography,
  CircularProgress,
  lighten,
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
];

// Table Header Component
const EnhancedTableHead = (props) => {
  const { onSelectAllClick, numSelected, rowCount } = props;

  return (
    <TableHead>
      <TableRow sx={{ backgroundColor: "#b9def0" }}>
        <TableCell padding="checkbox">
          <Checkbox
            color="primary"
            indeterminate={numSelected > 0 && numSelected < rowCount}
            checked={rowCount > 0 && numSelected === rowCount}
            onChange={onSelectAllClick}
            inputProps={{ "aria-label": "select all requests" }}
          />
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
          <b>Requested By</b>
        </TableCell>
      </TableRow>
    </TableHead>
  );
};

// Table Toolbar Component
const EnhancedTableToolbar = ({ numSelected, onApprove, onReject }) => {
  return (
    <Toolbar
      sx={{
        pl: { sm: 2 },
        pr: { xs: 1, sm: 1 },
        ...(numSelected > 0 && {
          bgcolor: (theme) =>
            lighten(theme.palette.primary.light, 0.85),
        }),
        borderBottom: "1px solid #ccc",
      }}
    >
      {numSelected > 0 ? (
        <Typography
          sx={{ flex: "1 1 100%" }}
          color="inherit"
          variant="subtitle1"
          component="div"
        >
          {numSelected} selected
        </Typography>
      ) : (
        <Typography
          sx={{ flex: "1 1 100%" }}
          variant="h6"
          id="tableTitle"
          component="div"
        >
          Audit Status Change Requests
        </Typography>
      )}

      {numSelected > 0 && (
        <Box>
          <Button
            variant="contained"
            color="success"
            sx={{ mr: 1 }}
            onClick={onApprove}
          >
            Approve
          </Button>
          <Button variant="contained" color="error" onClick={onReject}>
            Reject
          </Button>
        </Box>
      )}
    </Toolbar>
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
      setSelected([]);
      setLoading(false);
      setDialog({
        open: true,
        message: `Successfully ${action} ${selected.length} request(s).`,
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
      <Container maxWidth="xl">
        <Paper sx={{ width: "100%", mb: 2 }}>
          <EnhancedTableToolbar
            numSelected={selected.length}
            onApprove={() => handleAction("approved")}
            onReject={() => handleAction("rejected")}
          />
          <TableContainer>
            {loading && (
              <Box sx={{ display: 'flex', justifyContent: 'center', alignItems: 'center', height: 370 }}>
                 <CircularProgress />
              </Box>
            )}
            {!loading && (
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
                        <TableCell align="center">{row.reqOn}</TableCell>
                      </TableRow>
                    );
                  })}
                {emptyRows > 0 && (
                  <TableRow style={{ height: 53 * emptyRows }}>
                    <TableCell colSpan={7} />
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


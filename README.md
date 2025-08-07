import React, { useState, useEffect } from 'react';
import {
  Box,
  Button,
  Table,
  TableBody,
  TableCell,
  TableContainer,
  TableHead,
  TableRow,
  Paper,
  CircularProgress,
  Container,
  Typography,
  Stack,
  Dialog,
  DialogActions,
  DialogContent,
  DialogContentText,
  DialogTitle,
  Chip,
} from '@mui/material';
import { styled } from '@mui/material/styles';
import FormInput from '../../../../common/components/ui/FormInput';
import { useLocation, useNavigate } from 'react-router-dom';
import useCustomSnackbar from '../../../../common/hooks/useCustomSnackbar';
import useApi from '../../../../common/hooks/useApi';

// #region --- Styled Components ---
const StyledHeader = styled(TableCell)(() => ({
  textAlign: 'center',
  fontWeight: 'bold',
  backgroundColor: 'hsl(220, 20%, 35%)',
  color: 'white',
}));

const StyledCell = styled(TableCell)({
  padding: '8px 16px',
});
// #endregion

// #region --- Main Component ---
const WriteOff = () => {
  // #region --- State and Hooks Setup ---

  // --- ⬇️ 1. ROLE & DATA SIMULATION ⬇️ ---
  // This section replaces live data fetching for demonstration purposes.
  // ✅ **CHANGE THE ROLE HERE**: Use '51' for Maker, '52' for Checker.
  const user = {
    capacity: '52', // <-- '51' = MAKER, '52' = CHECKER
    circleCode: 'CORP',
    quarterEndDate: '2025-06-30',
    userId: 'testUser',
  };
  // const user = JSON.parse(localStorage.getItem('user')); // Original localStorage line

  // Sample data to simulate API response.
  const sampleData = [
    { circleCode: '020', amount: '15000.50', status: 'Pending' },
    { circleCode: '021', amount: '22300.00', status: 'Pending' },
    { circleCode: '001', amount: '5400.00', status: 'Accepted' },
    { circleCode: '008', amount: '9800.75', status: 'Rejected' },
    { circleCode: '015', amount: '11200.00', status: 'Pending' },
    { circleCode: '018', amount: '7650.00', status: 'Accepted' },
  ];
  // --- ⬆️ END OF SIMULATION SECTION ⬆️ ---

  const { state } = useLocation();
  const navigate = useNavigate();
  const { callApi } = useApi(); // Note: callApi is kept for button logic but won't be called.
  const setSnackbarMessage = useCustomSnackbar();
  const [reportObject, setReportObject] = useState(state?.report || { name: 'RW-04 Part A', status: '10' });
  const [rows, setRows] = useState([]);
  const [originalRows, setOriginalRows] = useState([]);
  const [isLoading, setIsLoading] = useState(true);
  const [dialogOpen, setDialogOpen] = useState(false);
  const [dialogAction, setDialogAction] = useState(null);
  const [selectedRow, setSelectedRow] = useState(null);
  // #endregion

  // #region --- Data Setup (Using Sample Data) ---
  useEffect(() => {
    // This hook loads the hardcoded sample data instead of fetching from an API.
    setIsLoading(true);
    setTimeout(() => {
      setRows(sampleData);
      setOriginalRows(JSON.parse(JSON.stringify(sampleData)));
      setIsLoading(false);
    }, 500); // Simulate network delay
    // The empty dependency array [] ensures this runs only once on component mount.
  }, []);
  // #endregion

  // #region --- Event Handlers ---
  const handleAmountChange = (index, value) => {
    // Only allow changes if the user is a Maker
    if (user.capacity === '51' && /^\d*\.?\d*$/.test(value)) {
      const updatedRows = [...rows];
      updatedRows[index].amount = value;
      setRows(updatedRows);
    }
  };

  const handleOpenDialog = (action, row) => {
    setDialogAction(action);
    setSelectedRow(row);
    setDialogOpen(true);
  };

  const handleCloseDialog = () => {
    setDialogOpen(false);
    setDialogAction(null);
    setSelectedRow(null);
  };

  const handleConfirmAction = async () => {
    if (!selectedRow) return;

    let actionEndpoint = '';
    let successMessage = '';
    let payloadData = [];

    // Determine API endpoint and success message based on the action
    switch (dialogAction) {
      case 'save':
        actionEndpoint = '/saveWriteOff';
        successMessage = `Data for circle ${selectedRow.circleCode} saved!`;
        break;
      case 'submit':
        actionEndpoint = '/submitWriteOff';
        successMessage = `Data for circle ${selectedRow.circleCode} submitted!`;
        break;
      case 'accept':
        actionEndpoint = '/acceptWriteOff';
        successMessage = `Circle ${selectedRow.circleCode} has been accepted!`;
        break;
      case 'reject':
        actionEndpoint = '/rejectWriteOff';
        successMessage = `Circle ${selectedRow.circleCode} has been rejected!`;
        break;
      default:
        console.error('Unknown action:', dialogAction);
        handleCloseDialog();
        return;
    }

    // Build the data payload for the API call
    if (reportObject?.status === '11') { // Example of complex payload
      const originalRow = originalRows.find((oRow) => oRow.circleCode === selectedRow.circleCode);
      const oldAmount = originalRow ? originalRow.amount : '0';
      payloadData = [[selectedRow.circleCode, oldAmount, selectedRow.amount || '0', selectedRow.status]];
    } else { // Standard payload for all actions in this example
      payloadData = [[selectedRow.circleCode, selectedRow.amount || '0', selectedRow.status]];
    }

    const payload = {
      circleCode: user.circleCode,
      reportName: reportObject.name,
      // ... other required payload fields
      userId: user.userId,
      data: payloadData,
    };

    console.log(`SIMULATING API CALL for action: '${dialogAction}'`);
    console.log(`Endpoint: ${actionEndpoint}`);
    console.log('Payload:', JSON.stringify(payload, null, 2));

    // In a real app, the `callApi` would be executed here.
    // For this demo, we'll just show a success message and close the dialog.
    // try {
    //   await callApi(actionEndpoint, payload, 'POST');
    //   setSnackbarMessage(successMessage, 'success');
    // } catch (error) {
    //   console.error(`Error during ${dialogAction}:`, error);
    //   setSnackbarMessage(`An error occurred.`, 'error');
    // } finally {
    //   handleCloseDialog();
    // }

    setSnackbarMessage(successMessage, 'success');
    handleCloseDialog();
  };
  // #endregion

  // #region --- Helper Functions ---
  const getStatusChip = (status) => {
    let color;
    switch (status) {
      case 'Accepted':
        color = 'success';
        break;
      case 'Rejected':
        color = 'error';
        break;
      case 'Pending':
        color = 'warning';
        break;
      default:
        color = 'default';
    }
    return <Chip label={status} color={color} size="small" sx={{ fontWeight: 'bold' }} />;
  };

  const getDialogInfo = () => {
    switch (dialogAction) {
      case 'save':
        return { title: 'Confirm Save', text: 'Are you sure you want to save the changes for this circle?' };
      case 'submit':
        return { title: 'Confirm Submission', text: 'Are you sure you want to submit? This action cannot be undone.' };
      case 'accept':
        return { title: 'Confirm Acceptance', text: 'Are you sure you want to accept the data for this circle?' };
      case 'reject':
        return { title: 'Confirm Rejection', text: 'Are you sure you want to reject the data for this circle?' };
      default:
        return { title: 'Confirm Action', text: 'Are you sure you want to proceed?' };
    }
  };
  // #endregion

  // #region --- Render Logic ---
  if (isLoading) {
    return (
      <Container sx={{ display: 'flex', justifyContent: 'center', alignItems: 'center', height: '80vh' }}>
        <Stack alignItems="center" spacing={2}>
          <CircularProgress />
          <Typography>Loading Data...</Typography>
        </Stack>
      </Container>
    );
  }

  return (
    <Container maxWidth="xl" sx={{ mt: 2, mb: 2 }}>
      <Typography variant="h5" sx={{ mb: 1, fontWeight: 'bold' }}>
        Write-Off Data Entry
      </Typography>
      <Typography variant="subtitle1" sx={{ mb: 3 }}>
        Current Role: <Chip label={user.capacity === '51' ? 'Maker' : 'Checker'} color="secondary" />
      </Typography>

      <TableContainer component={Paper} elevation={3}>
        <Table>
          <TableHead>
            <TableRow>
              <StyledHeader sx={{ width: '25%' }}>Circle Code</StyledHeader>
              <StyledHeader sx={{ width: '30%' }}>Amount</StyledHeader>
              <StyledHeader sx={{ width: '25%' }}>Status</StyledHeader>
              <StyledHeader sx={{ width: '20%' }}>Action</StyledHeader>
            </TableRow>
          </TableHead>
          <TableBody>
            {rows.map((row, index) => (
              <TableRow key={row.circleCode} hover>
                <StyledCell sx={{ fontWeight: 'medium', textAlign: 'center' }}>{row.circleCode}</StyledCell>
                <StyledCell>
                  <FormInput
                    value={row.amount}
                    onChange={(e) => handleAmountChange(index, e.target.value)}
                    inputProps={{ style: { textAlign: 'right' } }}
                    // ✅ Field is read-only for Checkers OR if status is final.
                    readOnly={user.capacity === '52' || row.status === 'Accepted' || row.status === 'Rejected'}
                  />
                </StyledCell>
                <StyledCell align="center">{getStatusChip(row.status)}</StyledCell>

                {/* --- ⬇️ 2. ROLE-BASED ACTION BUTTONS ⬇️ --- */}
                <StyledCell align="center">
                  {/* == MAKER VIEW (Save / Submit) == */}
                  {user.capacity === '51' && (
                    <Stack direction="row" spacing={1} justifyContent="center">
                      <Button
                        variant="contained" size="small" color="primary"
                        onClick={() => handleOpenDialog('save', row)}
                        disabled={row.status === 'Accepted' || row.status === 'Rejected'}
                      >
                        Save
                      </Button>
                      <Button
                        variant="contained" size="small" color="success"
                        onClick={() => handleOpenDialog('submit', row)}
                        disabled={row.status === 'Accepted' || row.status === 'Rejected'}
                      >
                        Submit
                      </Button>
                    </Stack>
                  )}

                  {/* == CHECKER VIEW (Accept / Reject) == */}
                  {user.capacity === '52' && (
                    <Stack direction="row" spacing={1} justifyContent="center">
                      <Button
                        variant="contained" size="small" color="success"
                        onClick={() => handleOpenDialog('accept', row)}
                        // ✅ Checker can only act on 'Pending' items.
                        disabled={row.status !== 'Pending'}
                      >
                        Accept
                      </Button>
                      <Button
                        variant="outlined" size="small" color="error"
                        onClick={() => handleOpenDialog('reject', row)}
                        disabled={row.status !== 'Pending'}
                      >
                        Reject
                      </Button>
                    </Stack>
                  )}
                </StyledCell>
              </TableRow>
            ))}
            {rows.length === 0 && (
              <TableRow>
                <TableCell colSpan={4} align="center">
                  <Typography sx={{ p: 4, color: 'text.secondary' }}>No circle data to display.</Typography>
                </TableCell>
              </TableRow>
            )}
          </TableBody>
        </Table>
      </TableContainer>

      {/* --- Confirmation Dialog --- */}
      <Dialog open={dialogOpen} onClose={handleCloseDialog}>
        <DialogTitle sx={{ fontWeight: 'bold' }}>
          {getDialogInfo().title} for Circle: {selectedRow?.circleCode}
        </DialogTitle>
        <DialogContent>
          <DialogContentText>{getDialogInfo().text}</DialogContentText>
        </DialogContent>
        <DialogActions>
          <Button onClick={handleCloseDialog} color="secondary">Cancel</Button>
          <Button onClick={handleConfirmAction} color="primary" variant="contained" autoFocus>Confirm</Button>
        </DialogActions>
      </Dialog>
    </Container>
  );
};
// #endregion

export default WriteOff;

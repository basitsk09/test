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
// Removed CustomButton import as the common buttons are no longer used
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
  const user = JSON.parse(localStorage.getItem('user'));
  const { state } = useLocation();
  const navigate = useNavigate();
  const { callApi } = useApi();
  const setSnackbarMessage = useCustomSnackbar();
  const [reportObject, setReportObject] = useState(state?.report || null);
  const [rows, setRows] = useState([]);
  const [originalRows, setOriginalRows] = useState([]);
  const [isLoading, setIsLoading] = useState(true);
  const [dialogOpen, setDialogOpen] = useState(false);
  const [dialogAction, setDialogAction] = useState(null);
  // --- MODIFICATION START: State to track the row being acted upon ---
  const [selectedRow, setSelectedRow] = useState(null);
  // --- MODIFICATION END ---
  // #endregion

  // #region --- Data Fetching ---
  useEffect(() => {
    let isMounted = true;
    const fetchData = async () => {
      setIsLoading(true);
      try {
        const payload = { qed: user?.quarterEndDate };
        const response = await callApi('/getCircleList', payload, 'POST');
        if (isMounted && response?.data) {
          const fetchedData = response.data;
          setRows(fetchedData);
          setOriginalRows(JSON.parse(JSON.stringify(fetchedData)));
        } else if (isMounted) {
          setSnackbarMessage('No data found for the current period.', 'info');
          setRows([]);
          setOriginalRows([]);
        }
      } catch (error) {
        console.error('Failed to fetch circle list:', error);
        if (isMounted) {
          setSnackbarMessage('Error loading circle status data.', 'error');
        }
      } finally {
        if (isMounted) {
          setIsLoading(false);
        }
      }
    };
    fetchData();
    return () => {
      isMounted = false;
    };
  }, [user?.quarterEndDate, callApi, setSnackbarMessage]);
  // #endregion

  // #region --- Event Handlers ---
  const handleAmountChange = (index, value) => {
    if (/^\d*\.?\d*$/.test(value)) {
      const updatedRows = [...rows];
      updatedRows[index].amount = value;
      setRows(updatedRows);
    }
  };

  // --- MODIFICATION START: Handler now accepts the row object ---
  const handleOpenDialog = (action, row) => {
    setDialogAction(action);
    setSelectedRow(row);
    setDialogOpen(true);
  };

  const handleCloseDialog = () => {
    setDialogOpen(false);
    setDialogAction(null);
    setSelectedRow(null); // Reset selected row on close
  };
  // --- MODIFICATION END ---

  // --- MODIFICATION START: Logic updated to handle a single row action ---
  const handleConfirmAction = async () => {
    if (!selectedRow) {
      console.error('No row selected for action.');
      setSnackbarMessage('An unexpected error occurred. Please try again.', 'error');
      handleCloseDialog();
      return;
    }

    const actionEndpoint = dialogAction === 'save' ? '/saveWriteOff' : '/submitWriteOff';
    const successMessage = dialogAction === 'save' ? 'Data saved successfully!' : 'Data submitted successfully!';
    let dataList;

    // Build payload data for the single selected row
    if (reportObject?.status === '11') {
      const originalRow = originalRows.find((oRow) => oRow.circleCode === selectedRow.circleCode);
      const oldAmount = originalRow ? originalRow.amount : '0';
      dataList = [[selectedRow.circleCode, oldAmount, selectedRow.amount || '0', selectedRow.status]];
    } else {
      dataList = [[selectedRow.circleCode, selectedRow.amount || '0', selectedRow.status]];
    }

    const payload = {
      circleCode: user.circleCode,
      reportName: reportObject.name,
      reportMasterId: reportObject.reportMasterId,
      reportId: reportObject.reportId,
      currentStatus: reportObject.status,
      userCapacity: user.capacity,
      qed: user?.quarterEndDate,
      userId: user?.userId,
      data: dataList, // Payload now contains data for only one row
    };

    console.log(
      `Payload for '${dialogAction}' on circle '${selectedRow.circleCode}':`,
      JSON.stringify(payload, null, 2)
    );

    try {
      await callApi(actionEndpoint, payload, 'POST');
      setSnackbarMessage(successMessage, 'success');
      if (dialogAction === 'submit') {
        // Optionally, refresh data or navigate away
        setTimeout(() => navigate(-1), 1500);
      }
    } catch (error) {
      console.error(`Error during ${dialogAction}:`, error);
      setSnackbarMessage(`An error occurred while trying to ${dialogAction}.`, 'error');
    } finally {
      handleCloseDialog();
    }
  };
  // --- MODIFICATION END ---
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
      {/* --- MODIFICATION START: Removed common Save/Submit buttons ---
      <Stack direction="row" spacing={2} sx={{ mb: 3 }}>
        <CustomButton buttonType={'save'} label={'Save'} onClickHandler={() => handleOpenDialog('save')} />
        <CustomButton buttonType={'submit'} label={'Submit'} onClickHandler={() => handleOpenDialog('submit')} />
      </Stack>
      --- MODIFICATION END --- */}

      <TableContainer component={Paper} elevation={3}>
        <Table>
          <TableHead>
            <TableRow>
              {/* --- MODIFICATION START: Adjusted column widths --- */}
              <StyledHeader sx={{ width: '25%' }}>Circle Code</StyledHeader>
              <StyledHeader sx={{ width: '30%' }}>Amount</StyledHeader>
              <StyledHeader sx={{ width: '25%' }}>RW-04 Part A Status</StyledHeader>
              <StyledHeader sx={{ width: '20%' }}>Action</StyledHeader>
              {/* --- MODIFICATION END --- */}
            </TableRow>
          </TableHead>
          <TableBody>
            {rows.map((row, index) => (
              <TableRow key={row.circleCode || index} hover>
                <StyledCell sx={{ fontWeight: 'medium', textAlign: 'center' }}>{row.circleCode}</StyledCell>
                <StyledCell>
                  <FormInput
                    value={row.amount}
                    onChange={(e) => handleAmountChange(index, e.target.value)}
                    inputProps={{ style: { textAlign: 'right' } }}
                    readOnly={row.status === 'Accepted' || row.status === 'Rejected'}
                  />
                </StyledCell>
                <StyledCell align="center">{getStatusChip(row.status)}</StyledCell>
                {/* --- MODIFICATION START: Added Action cell with row-specific buttons --- */}
                <StyledCell align="center">
                  <Stack direction="row" spacing={1} justifyContent="center">
                    <Button
                      variant="contained"
                      size="small"
                      color="primary"
                      onClick={() => handleOpenDialog('save', row)}
                      disabled={row.status === 'Accepted' || row.status === 'Rejected'}
                    >
                      Save
                    </Button>
                    <Button
                      variant="contained"
                      size="small"
                      color="success"
                      onClick={() => handleOpenDialog('submit', row)}
                      disabled={row.status === 'Accepted' || row.status === 'Rejected'}
                    >
                      Submit
                    </Button>
                  </Stack>
                </StyledCell>
                {/* --- MODIFICATION END --- */}
              </TableRow>
            ))}
            {rows.length === 0 && (
              <TableRow>
                <TableCell colSpan={4} align="center">
                  {' '}
                  {/* Updated colSpan to 4 */}
                  <Typography sx={{ p: 4, color: 'text.secondary' }}>No circle data to display.</Typography>
                </TableCell>
              </TableRow>
            )}
          </TableBody>
        </Table>
      </TableContainer>

      {/* Confirmation Dialog (logic remains mostly the same, context is now row-specific) */}
      <Dialog open={dialogOpen} onClose={handleCloseDialog}>
        <DialogTitle sx={{ fontWeight: 'bold' }}>
          {dialogAction === 'save' ? 'Confirm Save' : 'Confirm Submission'} for Circle: {selectedRow?.circleCode}
        </DialogTitle>
        <DialogContent>
          <DialogContentText>
            {dialogAction === 'save'
              ? 'Are you sure you want to save the changes for this circle?'
              : 'Are you sure you want to submit for this circle? Once submitted, the data may not be editable.'}
          </DialogContentText>
        </DialogContent>
        <DialogActions>
          <Button onClick={handleCloseDialog} color="secondary">
            Cancel
          </Button>
          <Button onClick={handleConfirmAction} color="primary" variant="contained" autoFocus>
            Confirm
          </Button>
        </DialogActions>
      </Dialog>
    </Container>
  );
};
// #endregion

export default WriteOff;

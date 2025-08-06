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
import { CustomButton } from '../../../../common/components/ui/Buttons';
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

  // Mock reportObject for demonstration. In a real app, this comes from navigation state.
  // I've defaulted to status '11' to demonstrate the "update" payload. Change to '10' to see the "create" payload.
  const [reportObject, setReportObject] = useState(state?.report || { status: '11' });

  const [rows, setRows] = useState([]);
  const [originalRows, setOriginalRows] = useState([]); // State to hold the initial data
  const [isLoading, setIsLoading] = useState(true);
  const [dialogOpen, setDialogOpen] = useState(false);
  const [dialogAction, setDialogAction] = useState(null);
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
          // Create a deep copy of the initial data to serve as "old values"
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

  const handleOpenDialog = (action) => {
    setDialogAction(action);
    setDialogOpen(true);
  };

  const handleCloseDialog = () => {
    setDialogOpen(false);
    setDialogAction(null);
  };

  const handleConfirmAction = async () => {
    const actionEndpoint = dialogAction === 'save' ? '/saveWriteOff' : '/submitWriteOff';
    const successMessage = dialogAction === 'save' ? 'Data saved successfully!' : 'Data submitted successfully!';
    let dataList;

    // Conditionally build the payload based on the report status
    if (reportObject?.status === '11') {
      // --- UPDATE PAYLOAD (Status 11) ---
      // Format: [circleCode, old_amount, new_amount, status]
      dataList = rows.map((currentRow) => {
        const originalRow = originalRows.find((oRow) => oRow.circleCode === currentRow.circleCode);
        const oldAmount = originalRow ? originalRow.amount : '0'; // Default to '0' if it's a new row added in the UI
        return [currentRow.circleCode, oldAmount, currentRow.amount || '0', currentRow.status];
      });
    } else {
      // --- CREATE PAYLOAD (Status 10 or other) ---
      // Format: [circleCode, amount, status]
      dataList = rows.map((row) => [row.circleCode, row.amount || '0', row.status]);
    }

    const payload = {
      qed: user?.quarterEndDate,
      userId: user?.userId,
      data: dataList,
    };

    console.log(`Payload for status '${reportObject?.status}':`, JSON.stringify(payload, null, 2));

    try {
      await callApi(actionEndpoint, payload, 'POST');
      setSnackbarMessage(successMessage, 'success');
      if (dialogAction === 'submit') {
        setTimeout(() => navigate(-1), 1500);
      }
    } catch (error) {
      console.error(`Error during ${dialogAction}:`, error);
      setSnackbarMessage(`An error occurred while trying to ${dialogAction}.`, 'error');
    } finally {
      handleCloseDialog();
    }
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
      <Stack direction="row" spacing={2} sx={{ mb: 3 }}>
        <CustomButton buttonType={'save'} label={'Save'} onClickHandler={() => handleOpenDialog('save')} />
        <CustomButton buttonType={'submit'} label={'Submit'} onClickHandler={() => handleOpenDialog('submit')} />
      </Stack>

      <TableContainer component={Paper} elevation={3}>
        <Table>
          <TableHead>
            <TableRow>
              <StyledHeader sx={{ width: '33%' }}>Circle Code</StyledHeader>
              <StyledHeader sx={{ width: '34%' }}>Amount</StyledHeader>
              <StyledHeader sx={{ width: '33%' }}>RW-04 Part A Status</StyledHeader>
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
              </TableRow>
            ))}
            {rows.length === 0 && (
              <TableRow>
                <TableCell colSpan={3} align="center">
                  <Typography sx={{ p: 4, color: 'text.secondary' }}>No circle data to display.</Typography>
                </TableCell>
              </TableRow>
            )}
          </TableBody>
        </Table>
      </TableContainer>

      {/* Confirmation Dialog */}
      <Dialog open={dialogOpen} onClose={handleCloseDialog}>
        <DialogTitle sx={{ fontWeight: 'bold' }}>
          {dialogAction === 'save' ? 'Confirm Save' : 'Confirm Submission'}
        </DialogTitle>
        <DialogContent>
          <DialogContentText>
            {dialogAction === 'save'
              ? 'Are you sure you want to save the changes?'
              : 'Are you sure you want to submit? Once submitted, the data may not be editable.'}
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

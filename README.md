import React, { useState, useEffect, useCallback } from 'react';
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
  TextField,
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

// Styled components for a consistent look
const StyledHeader = styled(TableCell)(() => ({
  textAlign: 'center',
  fontWeight: 'bold',
  backgroundColor: 'hsl(220, 20%, 35%)',
  color: 'white',
}));

const StyledCell = styled(TableCell)({
  padding: '8px 16px',
});

// Main Component
const WriteOff = () => {
  // #region --- State Management ---
  const user = JSON.parse(localStorage.getItem('user'));

  const { state } = useLocation();
  const [reportObject, setReportObject] = useState(state?.report || null);
  const navigate = useNavigate();
  const { callApi } = useApi();
  const setSnackbarMessage = useCustomSnackbar();
  const [rows, setRows] = useState([]);
  const [isLoading, setIsLoading] = useState(true);
  const [dialogOpen, setDialogOpen] = useState(false);
  const [dialogAction, setDialogAction] = useState(null);
  // #endregion
  const getBasePayload = useCallback(
    () => ({
      circleCode: user?.circleCode,
      qed: user?.quarterEndDate,
      userCapacity: user?.userCapacity,
      reportId: reportObject?.reportId,
      reportMasterId: reportObject?.reportMasterId,
      currentStatus: reportObject?.status,
      userId: user.userId,
    }),
    [user, reportObject]
  );
  // --- Data Fetching and Initialization ---
  useEffect(() => {
    // Simulate fetching data from an API
    setIsLoading(true);
    const fetchData = () => {
      const mockData = [
        { circleCode: 'MUM', amount: '150000.75', status: 'Accepted' },
        { circleCode: 'DEL', amount: '75000.00', status: 'Rejected' },
        { circleCode: 'KOL', amount: '98500.50', status: 'Pending' },
        { circleCode: 'CHE', amount: '120000.00', status: 'Accepted' },
        { circleCode: 'HYD', amount: '45000.25', status: 'Pending' },
      ];
      // Simulate network delay
      setTimeout(() => {
        setRows(mockData);
        setIsLoading(false);
      }, 1500);
    };
    console.log('reportObject:', reportObject);

    fetchData();
  }, []);

  // --- Handlers ---
  const handleAmountChange = (index, value) => {
    // Basic numeric validation (allows decimals)
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

  const handleConfirmAction = () => {
    console.log(`Action: ${dialogAction}`);
    console.log('Current Data:', rows);
    // Here you would typically make an API call to save or submit the data
    // For demonstration, we just log to the console.
    handleCloseDialog();
    // You would show a success snackbar here
  };

  // --- Helper for Status Styling ---
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
      {/* <Typography variant="h5" gutterBottom sx={{ fontWeight: 'bold' }}>
        Circle Status Report
      </Typography> */}

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
              <TableRow key={index} hover>
                <StyledCell sx={{ fontWeight: 'medium', textAlign: 'center' }}>{row.circleCode}</StyledCell>
                <StyledCell>
                  <FormInput
                    value={row.amount}
                    onChange={(e) => handleAmountChange(index, e.target.value)}
                    inputProps={{ style: { textAlign: 'right' } }}
                  />
                </StyledCell>
                <StyledCell align="center">{getStatusChip(row.status)}</StyledCell>
              </TableRow>
            ))}
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
              : 'Are you sure you want to submit? Once submitted, the data cannot be edited.'}
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

useEffect(() => {
    let isMounted = true;
    const fetchData = async () => {
      setIsLoading(true);

      showSnackbar('Loading data...', 'info');
      console.log('reportObj', reportObject);
      const payload = { circleCode: user.circleCode, quarterEndDate: user.quarterEndDate };
      try {
        const response = await callApi('/Maker/getSavedDataNineA', payload, 'POST');
        if (!isMounted) return;

        const enriched = response.map((r, i) => ({ ...initialRow, ...r, id: r.id || `row-${Date.now()}-${i}` }));
        setRows(enriched);
      } catch (error) {
        console.log('error', error);
        if (!isMounted) return;
        setSnackbarMessage('Error loading saved data.', 'error');
      } finally {
        // if (!isMounted) return;
        setIsLoading(false);
      }
    };
    fetchData();
    return () => {
      isMounted = false;
    };
  }, []); // Added callApi to dependency array


/getCircleList

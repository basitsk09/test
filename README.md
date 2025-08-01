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
  Chip, // Using Chip for better status display
} from '@mui/material';
import { styled } from '@mui/material/styles';
import { CustomButton } from '../../../../common/components/ui/Buttons'; // Assuming this path is correct

// Styled components for a consistent look
const StyledHeader = styled(TableCell)(({ theme }) => ({
  textAlign: 'center',
  fontWeight: 'bold',
  backgroundColor: '#f0f0f0',
  color: theme.palette.text.primary,
}));

const StyledCell = styled(TableCell)({
  padding: '8px 16px',
});

// Main Component
const CircleAmountStatusTable = () => {
  const [rows, setRows] = useState([]);
  const [isLoading, setIsLoading] = useState(true);
  const [dialogOpen, setDialogOpen] = useState(false);
  const [dialogAction, setDialogAction] = useState(null); // Will be 'save' or 'submit'

  // --- Data Fetching and Initialization ---
  useEffect(() => {
    // Simulate fetching data from an API
    const fetchData = () => {
      setIsLoading(true);
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

  // --- Render Logic ---
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
    <Container maxWidth="lg" sx={{ mt: 4, mb: 4 }}>
      <Typography variant="h5" gutterBottom sx={{ fontWeight: 'bold' }}>
        Circle Status Report
      </Typography>

      <Stack direction="row" spacing={2} sx={{ mb: 3 }}>
        <CustomButton buttonType={'save'} label={'Save'} onClickHandler={() => handleOpenDialog('save')} />
        <CustomButton buttonType={'submit'} label={'Submit'} onClickHandler={() => handleOpenDialog('submit')} />
      </Stack>

      <TableContainer component={Paper} elevation={3}>
        <Table>
          <TableHead>
            <TableRow>
              <StyledHeader sx={{ width: '30%' }}>Circle Code</StyledHeader>
              <StyledHeader sx={{ width: '40%' }}>Amount</StyledHeader>
              <StyledHeader sx={{ width: '30%' }}>Status</StyledHeader>
            </TableRow>
          </TableHead>
          <TableBody>
            {rows.map((row, index) => (
              <TableRow key={index} hover>
                <StyledCell sx={{ fontWeight: 'medium' }}>{row.circleCode}</StyledCell>
                <StyledCell>
                  <TextField
                    variant="outlined"
                    size="small"
                    fullWidth
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

export default CircleAmountStatusTable;

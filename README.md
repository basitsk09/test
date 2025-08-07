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
import { CustomButton } from '../../../../common/components/ui/Buttons';

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
  const [reportObject, setReportObject] = useState(state?.report || { name: 'RW-04 Part A', status: '10' });
  const [rows, setRows] = useState([]);
  const [originalRows, setOriginalRows] = useState([]);
  const [isLoading, setIsLoading] = useState(true);
  const [dialogOpen, setDialogOpen] = useState(false);
  const [dialogAction, setDialogAction] = useState(null);
  const [selectedRow, setSelectedRow] = useState(null);
  // #endregion

  // #region --- Data Fetching from API ---
  useEffect(() => {
    let isMounted = true;
    const fetchData = async () => {
      setIsLoading(true);
      try {
        const payload = {
          qed: user?.quarterEndDate,
          userCapacity: user?.capacity,
        };
        const response = await callApi('/getWriteOffTotalData', payload, 'POST');

        if (isMounted && response?.writeOffData) {
          const dataList = response.writeOffData;
          const formattedData = dataList.map((row) => ({
            // Splits "001~KOLKATA" into just "001"
            circleCode: row[0].split('~')[0],
            amount: row[1],
            // Assumes status code is the 3rd element
            status: row[2],
          }));
          setRows(formattedData);
          setOriginalRows(JSON.parse(JSON.stringify(formattedData)));
        } else if (isMounted) {
          setSnackbarMessage('No data found for the current period.', 'info');
          setRows([]);
          setOriginalRows([]);
        }
      } catch (error) {
        console.error('Failed to fetch write-off data:', error);
        if (isMounted) {
          setSnackbarMessage('Error loading data from the server.', 'error');
        }
      } finally {
        if (isMounted) {
          setIsLoading(false);
        }
      }
    };

    if (user?.quarterEndDate && user?.capacity) {
      fetchData();
    } else {
      setIsLoading(false);
      setSnackbarMessage('User information is missing. Cannot load data.', 'error');
    }

    return () => {
      isMounted = false;
    };
  }, [user?.quarterEndDate, user?.capacity, callApi, setSnackbarMessage]);
  // #endregion

  // #region --- Event Handlers ---
  const handleAmountChange = (index, value) => {
    // Only allow maker to edit if status is editable
    const isEditable = ['0', '11', '12'].includes(rows[index].status);
    if (user.capacity === '51' && isEditable && /^\d*\.?\d*$/.test(value)) {
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
    // The payload now correctly sends the combined circle code and name back if needed,
    // or just the code. Here we assume just the code is needed.
    const payloadData = [[selectedRow.circleCode, selectedRow.amount || '0', selectedRow.status]];
    const payload = {
      circleCode: user.circleCode,
      reportName: reportObject.name,
      userId: user.userId,
      data: payloadData,
    };

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
        handleCloseDialog();
        return;
    }

    try {
      await callApi(actionEndpoint, payload, 'POST');
      setSnackbarMessage(successMessage, 'success');
    } catch (error) {
      console.error(`Error during ${dialogAction}:`, error);
      setSnackbarMessage(`An error occurred while trying to ${dialogAction}.`, 'error');
    } finally {
      handleCloseDialog();
    }
  };
  // #endregion

  // #region --- Helper Functions ---
  /**
   * Translates a status code into a display text and color for the UI Chip.
   * @param {string} status - The status code from the API (e.g., '0', '11', '51').
   * @returns {{text: string, color: string}} - an object with the text and color.
   */
  const getStatusDetails = (status) => {
    switch (String(status)) {
      case '0':
        return { text: 'Not Created', color: 'default' };
      case '11':
        return { text: 'Saved', color: 'info' };
      case '12':
        return { text: 'Rejected', color: 'error' };
      case '20':
        return { text: 'Submitted', color: 'primary' };
      case '51':
        return { text: 'Pending Approval', color: 'warning' };
      default:
        return { text: `Unknown (${status})`, color: 'default' };
    }
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
            {rows.map((row, index) => {
              const statusDetails = getStatusDetails(row.status);
              const isMakerEditable = ['0', '11', '12'].includes(row.status);

              return (
                <TableRow key={row.circleCode} hover>
                  <StyledCell sx={{ fontWeight: 'medium', textAlign: 'center' }}>{row.circleCode}</StyledCell>
                  <StyledCell>
                    <FormInput
                      value={row.amount}
                      onChange={(e) => handleAmountChange(index, e.target.value)}
                      inputProps={{ style: { textAlign: 'right' } }}
                      readOnly={user.capacity === '52' || !isMakerEditable}
                    />
                  </StyledCell>
                  <StyledCell align="center">
                    <Chip label={statusDetails.text} color={statusDetails.color} size="small" sx={{ fontWeight: 'bold' }} />
                  </StyledCell>
                  <StyledCell align="center">
                    {(() => {
                      // == MAKER (51) VIEW LOGIC ==
                      if (user.capacity === '51') {
                        return (
                          <Stack direction="row" spacing={1} justifyContent="center">
                            <CustomButton
                              label={'Save'} buttonType={'save'}
                              onClickHandler={() => handleOpenDialog('save', row)}
                              disabled={!isMakerEditable}
                            />
                            <CustomButton
                              label={'Submit'} buttonType={'submit'}
                              onClickHandler={() => handleOpenDialog('submit', row)}
                              disabled={!isMakerEditable}
                            />
                          </Stack>
                        );
                      }
                      // == CHECKER (52) VIEW LOGIC ==
                      else if (user.capacity === '52') {
                        // For 'Submitted' status, show both Accept and Reject
                        if (row.status === '20') {
                          return (
                            <Stack direction="row" spacing={1} justifyContent="center">
                              <CustomButton
                                label={'Accept'} buttonType={'accept'}
                                onClickHandler={() => handleOpenDialog('accept', row)}
                              />
                              <CustomButton
                                label={'Reject'} buttonType={'reject'}
                                onClickHandler={() => handleOpenDialog('reject', row)}
                              />
                            </Stack>
                          );
                        }
                        // For status 51, show ONLY Reject
                        else if (row.status === '51') {
                          return (
                            <CustomButton
                              label={'Reject'} buttonType={'reject'}
                              onClickHandler={() => handleOpenDialog('reject', row)}
                            />
                          );
                        }
                      }
                      // For all other cases, render nothing
                      return null;
                    })()}
                  </StyledCell>
                </TableRow>
              );
            })}
            {rows.length === 0 && (
              <TableRow>
                <TableCell colSpan={4} align="center">
                  <Typography sx={{ p: 4, color: 'text.secondary' }}>No data to display.</Typography>
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

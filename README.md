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
import SkeletonWrapper from '../../../../common/components/ui/SkeletonWrapper';

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

  const fetchData = useCallback(async () => {
    setIsLoading(true);
    try {
      const payload = {
        qed: user?.quarterEndDate,
        userCapacity: user?.capacity,
      };
      const response = await callApi('/getWriteOffTotalData', payload, 'POST');

      if (response?.writeOffData) {
        const dataList = response.writeOffData;
        const formattedData = dataList.map((row) => ({
          circleName: row[0].split('~')[1]?.toUpperCase() || 'N/A',
          circleCode: row[0].split('~')[0],
          amount: row[1],
          status: row[2],
          statusOfReport: row[3]?.split('~')[0] || 'N/A',
          statusReportCode: row[3]?.split('~')[1] || '',
        }));
        setRows(formattedData);
        setOriginalRows(JSON.parse(JSON.stringify(formattedData)));
      } else {
        setSnackbarMessage('No data found for the current period.', 'info');
        setRows([]);
        setOriginalRows([]);
      }
      setIsLoading(false);
    } catch (error) {
      if (error.code !== 'ERR_CANCELED') {
        console.error('Failed to fetch write-off data:', error);
        setSnackbarMessage('Error loading data from the server.', 'error');
      }
    } finally {
      setIsLoading(false);
    }
  }, [user?.quarterEndDate, user?.capacity, callApi, setSnackbarMessage]);

  useEffect(() => {
    if (user?.quarterEndDate && user?.capacity) {
      fetchData();
    } else {
      setIsLoading(false);
      setSnackbarMessage('User information is missing. Cannot load data.', 'error');
    }
  }, [fetchData, user?.quarterEndDate, user?.capacity, setSnackbarMessage]);
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
    let actionEndpoint = '/updateStatus';
    let successMessage = '';
    const payloadData = [selectedRow.circleCode, selectedRow.amount || '0', selectedRow.status];
    const payload = {
      qed: user.quarterEndDate,
      circleCode: user.circleCode,
      reportName: reportObject.name,
      userId: user.userId,
      data: payloadData,
      status:
        selectedRow.status === '0'
          ? '0'
          : dialogAction === 'save'
          ? '11'
          : dialogAction === 'submit'
          ? '20'
          : dialogAction === 'accept'
          ? '51'
          : '12',
      reportStatus: selectedRow.statusReportCode,
    };

    switch (dialogAction) {
      case 'save':
        successMessage = `Data for circle ${selectedRow.circleName} saved!`;
        break;
      case 'submit':
        successMessage = `Data for circle ${selectedRow.circleName} submitted!`;
        break;
      case 'accept':
        successMessage = `Circle ${selectedRow.circleName} has been accepted!`;
        break;
      case 'reject':
        successMessage = `Circle ${selectedRow.circleName} has been rejected!`;
        break;
      default:
        handleCloseDialog();
        return;
    }

    try {
      const response = await callApi(actionEndpoint, payload, 'POST');
      fetchData();
      if (response?.message) {
        setSnackbarMessage(successMessage, 'success');
      } else {
        setSnackbarMessage('Failed to save.', 'error');
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

  const getStatusDetails = (status) => {
    switch (String(status)) {
      case '0':
        return { text: 'Not Created', color: 'default' };
      case '11':
        return { text: 'Saved By Maker', color: 'primary' };
      case '12':
        return { text: 'Rejected By Checker', color: 'error' };
      case '20':
        return { text: 'Submitted By Maker', color: 'warning' };
      case '51':
        return { text: 'Accepted By Checker', color: 'success' };
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
        return {
          title: 'Confirm Acceptance',
          text: 'Are you sure you want to accept the data for this circle? Report RW-04 will be re-opened.',
        };
      case 'reject':
        return { title: 'Confirm Rejection', text: 'Are you sure you want to reject the data for this circle?' };
      default:
        return { title: 'Confirm Action', text: 'Are you sure you want to proceed?' };
    }
  };
  // #endregion

  // #region --- Render Logic ---
  // if (isLoading) {
  //   return (
  //     <Container sx={{ display: 'flex', justifyContent: 'center', alignItems: 'center', height: '80vh' }}>
  //       <Stack alignItems="center" spacing={2}>
  //         <CircularProgress />
  //         <Typography>Loading Data...</Typography>
  //       </Stack>
  //     </Container>
  //   );
  // }

  if (isLoading) {
    return (
      <>
        <SkeletonWrapper
          isLoading={isLoading}
          skeletonType="table"
          skeletonConfig={{
            rows: 15,
            columns: 5,
            hasHeader: true,
            hasActions: true,
          }}
        />
      </>
    );
  }

  return (
    <Box sx={{ mt: 2, mb: 2, width: 1 }}>
      <TableContainer component={Paper} elevation={3} sx={{ mt: 2, maxHeight: 'calc(105vh - 250px)' }}>
        <Table stickyHeader>
          <TableHead>
            <TableRow>
              <StyledHeader sx={{ width: '20%' }}>CIRCLE</StyledHeader>
              <StyledHeader sx={{ width: '20%' }}>STATUS OF RW-04 A</StyledHeader>
              <StyledHeader sx={{ width: '20%' }}>AMOUNT</StyledHeader>
              <StyledHeader sx={{ width: '20%' }}>STATUS</StyledHeader>
              <StyledHeader sx={{ width: '20%' }}>ACTION</StyledHeader>
            </TableRow>
          </TableHead>
          <TableBody>
            {rows.map((row, index) => {
              const statusDetails = getStatusDetails(row.status);
              const isMakerEditable = ['0', '11', '12'].includes(row.status);

              return (
                <TableRow key={row.circleCode} hover>
                  <StyledCell sx={{ fontWeight: 'medium', textAlign: 'center' }}>{row.circleName}</StyledCell>
                  <StyledCell sx={{ fontWeight: 'medium', textAlign: 'center' }}>{row.statusOfReport}</StyledCell>
                  <StyledCell>
                    <FormInput
                      value={row.amount}
                      onChange={(e) => handleAmountChange(index, e.target.value)}
                      inputProps={{ style: { textAlign: 'right' } }}
                      readOnly={user.capacity === '52' || !isMakerEditable}
                    />
                  </StyledCell>
                  <StyledCell align="center">
                    <Chip
                      label={statusDetails.text}
                      color={statusDetails.color}
                      size="medium"
                      sx={{ fontWeight: 'bold', width: '150px' }}
                    />
                  </StyledCell>
                  <StyledCell align="center">
                    {(() => {
                      // == MAKER (51) VIEW LOGIC ==
                      if (user.capacity === '51') {
                        return (
                          <Stack direction="row" spacing={1} justifyContent="center">
                            <CustomButton
                              label={'Save'}
                              buttonType={'save'}
                              onClickHandler={() => handleOpenDialog('save', row)}
                              disabled={!isMakerEditable}
                            />
                            <CustomButton
                              label={'Submit'}
                              buttonType={'submit'}
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
                                label={'Accept'}
                                buttonType={'accept'}
                                onClickHandler={() => handleOpenDialog('accept', row)}
                              />
                              <CustomButton
                                label={'Reject'}
                                buttonType={'reject'}
                                onClickHandler={() => handleOpenDialog('reject', row)}
                              />
                            </Stack>
                          );
                        }
                        // For status 51, show ONLY Reject
                        else if (row.status === '51') {
                          return (
                            <Stack direction="row" spacing={1} justifyContent="center">
                              <CustomButton
                                label={'Accept'}
                                buttonType={'accept'}
                                onClickHandler={() => handleOpenDialog('accept', row)}
                                disabled
                              />
                              <CustomButton
                                label={'Reject'}
                                buttonType={'reject'}
                                onClickHandler={() => handleOpenDialog('reject', row)}
                              />
                            </Stack>
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
    </Box>
  );
};
// #endregion

export default WriteOff;

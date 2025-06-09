import React, { useState, useEffect, useMemo } from 'react';
import { useLocation } from 'react-router-dom';
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
  Typography,
  Stack,
  Tabs,
  CircularProgress,
  Tab,
  useTheme,
  Snackbar,
  Alert,
  Dialog,
  DialogActions,
  DialogContent,
  DialogContentText,
  Container,
  DialogTitle,
} from '@mui/material';
import { styled } from '@mui/material/styles';
import useApi from '../../../../common/hooks/useApi';
import { CustomButton } from '../../../../common/components/ui/Buttons';
import FormInput from '../../../../common/components/ui/FormInput';
import SkeletonWrapper from '../../../../common/components/ui/SkeletonWrapper';
import useCustomSnackbar from '../../../../common/hooks/useCustomSnackbar';

const StyledCell = styled(TableCell)(({ theme }) => ({
  textAlign: 'right',
  padding: '4px',
  // backgroundColor: theme.palette.background.paper,
  // color: theme.palette.text.primary,
}));

const StyledHeader = styled(TableCell)(({ theme }) => ({
  textAlign: 'center',
  fontWeight: 'bold',
  // backgroundColor: theme.palette.mode === 'dark' ? theme.palette.grey[800] : '#f0f0f0',
  // color: theme.palette.text.primary,
}));

const getDefaultRow = () => ({
  migCircleCode: '',
  circleDesc: '',
  inSusp: '',
  provn: '',
  licra: '',
  dicgc: '',
});

const validateNumeric = (value) => {
  return /^-?\d*(\.\d{0,2})?$/.test(value);
};

const calculateTotals = (rows, fieldPrefix = '') => {
  return rows.reduce(
    (totals, row) => {
      ['inSusp', 'provn', 'licra', 'dicgc'].forEach((field) => {
        totals[field] += parseFloat(row[`${field}${fieldPrefix}`] || 0);
      });
      return totals;
    },
    { inSusp: 0, provn: 0, licra: 0, dicgc: 0 }
  );
};

const MigrationTable = ({ title, rows, setRows, fieldPrefix = '' }) => {
  const handleChange = (index, field, value) => {
    if (validateNumeric(value) || value === '') {
      const updated = [...rows];
      updated[index][`${field}${fieldPrefix}`] = value;
      setRows(updated);
    }
  };

  const totals = useMemo(() => calculateTotals(rows, fieldPrefix), [rows, fieldPrefix]);

  return (
    <Box sx={{ mt: 3 }}>
      <Typography variant="h6" sx={{ mb: 1 }}>
        <b>{title}</b>
      </Typography>
      <TableContainer component={Paper}>
        <Table size="small">
          <TableHead>
            <TableRow>
              <StyledHeader>NAME OF THE CIRCLE / GROUP</StyledHeader>
              <StyledHeader>INTEREST SUSPENSE</StyledHeader>
              <StyledHeader>PROVISION</StyledHeader>
              <StyledHeader>LICRA</StyledHeader>
              <StyledHeader>DICGC, ECGC CLAIMS RECD</StyledHeader>
            </TableRow>
          </TableHead>
          <TableBody>
            {rows.map((row, idx) => (
              <TableRow key={idx}>
                <TableCell>{row.circleDesc}</TableCell>
                {['inSusp', 'provn', 'licra', 'dicgc'].map((field) => {
                  const fullField = `${field}${fieldPrefix}`;
                  return (
                    <StyledCell key={fullField}>
                      <FormInput
                        variant="outlined"
                        inputType="amountDecimal"
                        value={row[fullField] || ''}
                        onChange={(e) => handleChange(idx, field, e.target.value)}
                      />
                    </StyledCell>
                  );
                })}
              </TableRow>
            ))}
            <TableRow>
              <StyledCell>
                <b>Total</b>
              </StyledCell>
              {['inSusp', 'provn', 'licra', 'dicgc'].map((field) => (
                <StyledCell key={field}>
                  <FormInput variant="outlined" value={totals[field].toFixed(2)} inputType="amountDecimal" readOnly />
                </StyledCell>
              ))}
            </TableRow>
          </TableBody>
        </Table>
      </TableContainer>
    </Box>
  );
};

const Schedule9CMigration = () => {
  // const theme = useTheme();
  const [rowsTo, setRowsTo] = useState([]);
  const [rowsFrom, setRowsFrom] = useState([]);
  const [tabIndex, setTabIndex] = useState(0);
  const [isLoading, setIsLoading] = useState(false);
  const { callApi } = useApi();
  const { state } = useLocation();
  const [report, setReport] = useState(state?.report || null);
  const user = JSON.parse(localStorage.getItem('user'));
  const [snackbarOpen, setSnackbarOpen] = useState(false);
  //const [snackbarMessage, setSnackbarMessage] = useState('');
  const [snackbarSeverity, setSnackbarSeverity] = useState('success');
  const [open, setOpen] = useState(false);
  const [isSave, setIsSave] = useState();
  const setSnackbarMessage = useCustomSnackbar();

  useEffect(() => {
    const fetchMigrationData = async () => {
      setIsLoading(true);
      try {
        // api get cicle list
        if (!report.status) {
          const toRes = await callApi('/Maker/getCirclelist', null, 'POST');

          const uniqueTo = filterUniqueRows(toRes);
          const uniqueFrom = filterUniqueRows(toRes);
          const toRows = uniqueTo.map((row) => ({
            circleDesc: row.circleDesc,
            migCircleCode: row.migCircleCode,
            circleDescHidden: row.migCircleCode,
            inSusp: row.inSusp ?? '',
            provn: row.provn ?? '',
            licra: row.licra ?? '',
            dicgc: row.dicgc ?? '',
          }));
          const fromRows = uniqueFrom.map((row) => ({
            circleDesc: row.circleDesc,
            migCircleCode: row.migCircleCode,
            circleDescHidden: row.migCircleCode,
            inSusp2: row.inSusp2 ?? '',
            provn2: row.provn2 ?? '',
            licra2: row.licra2 ?? '',
            dicgc2: row.dicgc2 ?? '',
          }));

          setRowsTo(toRows);
          setRowsFrom(fromRows);
          return;
        }
        const payload = {
          circleCode: user.circleCode,
          quarterEndDate: user.quarterEndDate,
          userId: user.userId,
          reportName: report.name,
          reportId: report.reportId,
          reportMasterId: report.reportMasterId,
          status: report.status,
        };

        const toRes = await callApi('/Maker/getSavedDataNineMig', payload, 'POST');
        const fromRes = await callApi('/Maker/getSavedDataNineMigTwo', payload, 'POST');

        const uniqueTo = filterUniqueRows(toRes);
        const uniqueFrom = filterUniqueRows(fromRes);

        const toRows = uniqueTo.map((row) => ({
          circleDesc: row.circleDesc,
          migCircleCode: row.migCircleCode,
          circleDescHidden: row.migCircleCode,
          inSusp: row.inSusp ?? '',
          provn: row.provn ?? '',
          licra: row.licra ?? '',
          dicgc: row.dicgc ?? '',
        }));

        const fromRows = uniqueFrom.map((row) => ({
          circleDesc: row.circleDesc,
          migCircleCode: row.migCircleCode,
          circleDescHidden: row.migCircleCode,
          inSusp2: row.inSusp2 ?? '',
          provn2: row.provn2 ?? '',
          licra2: row.licra2 ?? '',
          dicgc2: row.dicgc2 ?? '',
        }));

        setRowsTo(toRows);
        setRowsFrom(fromRows);
      } catch (error) {
        console.error('Error fetching SC9C migration data:', error);
      } finally {
        setIsLoading(false);
      }
    };

    fetchMigrationData();
  }, []);

  const filterUniqueRows = (rows) => {
    const seen = new Set();
    return rows.filter((row) => {
      const key = `${row.migCircleCode}-${row.circleDesc}`;
      if (!seen.has(key)) {
        seen.add(key);
        return true;
      }
      return false;
    });
  };

  const handleSubmit = async () => {
    setOpen(false);
    const payload = {
      listToBeSent: rowsTo.map((r) => ({
        circleDescHidden: r.migCircleCode,
        inSusp: r.inSusp,
        provn: r.provn,
        licra: r.licra,
        dicgc: r.dicgc,
      })),
      listToBeSentFrom: rowsFrom.map((r) => ({
        circleDescHidden: r.migCircleCode,
        inSusp2: r.inSusp2,
        provn2: r.provn2,
        licra2: r.licra2,
        dicgc2: r.dicgc2,
      })),
      save: isSave,
      circleCode: user.circleCode,
      quarterEndDate: user.quarterEndDate,
      userId: user.userId,
      reportName: report.name,
      reportId: report.reportId,
      reportMasterId: report.reportMasterId,
      status: report.status,
    };

    try {
      const res = await callApi('/Maker/submitNineMig', payload, 'POST');
      if (res && typeof res === 'string') {
        const [flag, newReportId, newStatus] = res.split('~');
        setReport((prev) => ({
          ...prev,
          reportId: newReportId,
          status: newStatus,
        }));
      }
      setSnackbarMessage(isSave ? 'Data saved successfully.' : 'Data submitted successfully.', 'success');
      setOpen(false);
    } catch (e) {
      console.error(e);
      setSnackbarMessage('Failed to save/submit data.', 'error');
      setOpen(false);
    }
  };
  const handleTabChange = (event, newValue) => {
    setTabIndex(newValue);
  };
  const handleCancel = () => {
    setOpen(false);
  };

  const handleClickOpen = (isSave) => {
    setOpen(true);
    setIsSave(isSave);
  };

  // if (isLoading) {
  //   return (
  //     <Box sx={{ display: 'flex', justifyContent: 'center', alignItems: 'center', height: '50vh' }}>
  //       <CircularProgress />
  //       <Typography sx={{ ml: 2 }}>Loading Data...</Typography>
  //     </Box>
  //   );
  // }

  if (isLoading) {
    return (
      <>
        <SkeletonWrapper
          isLoading={isLoading}
          skeletonType="form"
          skeletonConfig={{
            fields: 0,
            hasTitle: true,
            fieldHeight: 10,
            buttonsCount: 4,
          }}
        />
        <SkeletonWrapper
          isLoading={isLoading}
          skeletonType="table"
          skeletonConfig={{
            rows: 3,
            columns: 3,
            hasHeader: true,
            hasActions: false,
          }}
        />
        <SkeletonWrapper
          isLoading={isLoading}
          skeletonType="table"
          skeletonConfig={{
            rows: 8,
            columns: 3,
            hasHeader: true,
            hasActions: false,
          }}
        />
      </>
    );
  }

  // if (!isLoading && (!rowsTo || Object.keys(rowsTo).length === 0)) {
  //   return (
  //     <Container maxWidth="xl" sx={{ mt: 4, mb: 4 }}>
  //       <Typography variant="h6" align="center">
  //         Failed to load Schedule 9C Migration Data...
  //       </Typography>
  //     </Container>
  //   );
  // }

  return (
    <Box
      sx={{
        mt: 4,
        mb: 4,
        width: '100%' /* backgroundColor: theme.palette.background.default, color: theme.palette.text.primary */,
      }}
    >
      <Stack direction="row" spacing={2} sx={{ mt: 2, mb: 2 }}>
        <CustomButton buttonType={'save'} label={'Save'} onClickHandler={() => handleClickOpen(true)} />
        <CustomButton buttonType={'submit'} label={'Submit'} onClickHandler={() => handleClickOpen(false)} />
      </Stack>

      <Tabs value={tabIndex} onChange={handleTabChange} sx={{ mb: 2 }}>
        <Tab label="Migration To Other Circles" />
        <Tab label="Migration From Other Circles" />
      </Tabs>

      {tabIndex === 0 && (
        <MigrationTable
          title="Migration To OTHER CIRCLES / GROUPS FROM THE CLOSING BALANCE OF PREVIOUS YEAR"
          rows={rowsTo}
          setRows={setRowsTo}
          fieldPrefix=""
        />
      )}

      {tabIndex === 1 && (
        <MigrationTable
          title="Migration From OTHER CIRCLES / GROUPS FROM THE CLOSING BALANCE OF PREVIOUS YEAR"
          rows={rowsFrom}
          setRows={setRowsFrom}
          fieldPrefix="2"
        />
      )}

      <Dialog
        open={open}
        onClose={handleCancel}
        aria-labelledby="alert-dialog-title"
        aria-describedby="alert-dialog-description"
      >
        <DialogTitle id="alert-dialog-title">{isSave ? 'Confirm Save' : 'Confirm Submit'}</DialogTitle>
        <DialogContent>
          <DialogContentText id="alert-dialog-description">
            {isSave ? 'Are you sure you want to save?' : 'Are you sure you want to submit?'}
          </DialogContentText>
        </DialogContent>
        <DialogActions>
          <Button onClick={handleCancel} color="primary">
            Cancel
          </Button>
          <Button onClick={handleSubmit} color="primary" autoFocus>
            Confirm
          </Button>
        </DialogActions>
      </Dialog>
    </Box>
  );
};

export default Schedule9CMigration;

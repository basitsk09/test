//working code

import React, { useState, useEffect, useMemo, useCallback } from 'react';
import axios from 'axios';
import {
  Table,
  TableBody,
  TableContainer,
  TableHead,
  TableRow,
  Paper,
  TextField,
  Button,
  Alert,
  Stack,
  Box,
  Snackbar,
  Checkbox,
  TableCell,
  Radio,
} from '@mui/material';
import { styled } from '@mui/material/styles';
import { CustomButton } from '../../../../common/components/ui/Buttons';

const StyledTableCell = styled(TableCell)(({ theme }) => ({
  fontSize: '1rem',
  [`&.MuiTableCell-head`]: {
    backgroundColor: theme.palette.common.black,
    color: theme.palette.common.white,
  },
}));

const numericRegex = /^-?\d*(\.\d{0,2})?$/;
const alphanumericRegex = /^[A-Za-z0-9\s]*$/;

const columns = [
  { key: 'isDelete', label: 'Select', type: 'checkbox', editable: false, align: 'center' },
  {
    key: 'borrowerName',
    label: "Branch & Borrower's Name",
    type: 'text',
    editable: true,
    align: 'left',
    pattern: alphanumericRegex,
  },
  {
    key: 'aggOutStand',
    label: 'Aggregate outstanding',
    type: 'number',
    editable: true,
    align: 'right',
    pattern: numericRegex,
  },
  {
    key: 'aggSecurities',
    label: 'Aggregate value of realisable securities',
    type: 'number',
    editable: true,
    align: 'right',
    pattern: numericRegex,
  },
  { key: 'netShortfall', label: 'Net Shortfall', type: 'number', editable: false, align: 'right' },
  { key: 'provision', label: 'Provision', type: 'number', editable: true, align: 'right', pattern: numericRegex },
  {
    key: 'balInterestSuspenseAcc',
    label: 'Balance in Interest Suspense Account',
    type: 'number',
    editable: true,
    align: 'right',
    pattern: numericRegex,
  },
];

const initialRow = {
  isDelete: false,
  borrowerName: '',
  aggOutStand: '',
  aggSecurities: '',
  netShortfall: '',
  provision: '',
  balInterestSuspenseAcc: '',
};

export default function Schedule9ATable() {
  const [rows, setRows] = useState([]);
  const [totals, setTotals] = useState({
    aggOutStandTotal: 0,
    aggSecuritiesTotal: 0,
    netShortfallTotal: 0,
    provisionTotal: 0,
    balInterestSuspenseAccTotal: 0,
  });
  const [snackbarOpen, setSnackbarOpen] = useState(false);
  const [snackbarMessage, setSnackbarMessage] = useState('');
  const [saveStatus, setSaveStatus] = useState(null); // 'success', 'error', null
  const [submitStatus, setSubmitStatus] = useState(null); // 'success', 'error', null

  // Simulate fetching saved data (replace with your actual API call)
  useEffect(() => {
    // Example:
    // axios.get('/api/schedule9a')
    //   .then(response => {
    //     setRows(response.data);
    //   })
    //   .catch(error => {
    //     console.error("Error fetching saved data:", error);
    //     setSnackbarMessage('Error loading saved data.');
    //     setSnackbarOpen(true);
    //   });
  }, []);

  const calculateTotals = useCallback(() => {
    let aggOutStandTotal = 0;
    let aggSecuritiesTotal = 0;
    let netShortfallTotal = 0;
    let provisionTotal = 0;
    let balInterestSuspenseAccTotal = 0;
    let hasNetShortfallChanged = false;

    const updatedRows = rows.map((row) => {
      const aggOut = parseFloat(row.aggOutStand) || 0;
      const aggSec = parseFloat(row.aggSecurities) || 0;
      const newNetShort = (aggOut - aggSec).toFixed(2);
      if (row.netShortfall !== newNetShort) {
        hasNetShortfallChanged = true;
        return { ...row, netShortfall: newNetShort };
      }
      return row;
    });

    if (hasNetShortfallChanged) {
      setRows(updatedRows);
    }

    updatedRows.forEach((row) => {
      aggOutStandTotal += parseFloat(row.aggOutStand) || 0;
      aggSecuritiesTotal += parseFloat(row.aggSecurities) || 0;
      netShortfallTotal += parseFloat(row.netShortfall) || 0;
      provisionTotal += parseFloat(row.provision) || 0;
      balInterestSuspenseAccTotal += parseFloat(row.balInterestSuspenseAcc) || 0;
    });

    setTotals({
      aggOutStandTotal: aggOutStandTotal.toFixed(2),
      aggSecuritiesTotal: aggSecuritiesTotal.toFixed(2),
      netShortfallTotal: netShortfallTotal.toFixed(2),
      provisionTotal: provisionTotal.toFixed(2),
      balInterestSuspenseAccTotal: balInterestSuspenseAccTotal.toFixed(2),
    });
  }, [rows, setRows, setTotals]);

  useEffect(() => {
    calculateTotals();
  }, [rows, calculateTotals]);

  const handleInputChange = (event, index, columnKey) => {
    const { value } = event.target;
    const updatedRows = [...rows];
    const column = columns.find((col) => col.key === columnKey);

    if (column?.pattern && !column.pattern.test(value)) {
      return; // Prevent updating state if the pattern doesn't match
    }

    updatedRows[index][columnKey] = value;
    setRows(updatedRows);
  };

  const handleCheckboxChange = (event, index) => {
    const updatedRows = [...rows];
    updatedRows[index].isDelete = event.target.checked;
    setRows(updatedRows);
  };

  const addRow = () => {
    setRows([...rows, { ...initialRow }]);
  };

  const deleteSelectedRows = () => {
    const newRows = rows.filter((row) => !row.isDelete);
    setRows(newRows);
    setSnackbarMessage('Selected rows deleted.');
    setSnackbarOpen(true);
  };

  const validateRows = () => {
    for (let i = 0; i < rows.length; i++) {
      const row = rows[i];
      const hasData = Object.keys(initialRow).some((key) => key !== 'isDelete' && row[key] !== '');
      if (hasData && !row.borrowerName) {
        setSnackbarMessage("Please enter Branch & Borrower's Name for all filled rows.");
        setSnackbarOpen(true);
        return false;
      }
      if (
        hasData &&
        (isNaN(parseFloat(row.aggOutStand)) ||
          isNaN(parseFloat(row.aggSecurities)) ||
          isNaN(parseFloat(row.provision)) ||
          isNaN(parseFloat(row.balInterestSuspenseAcc)))
      ) {
        setSnackbarMessage(
          'Please enter valid numeric values for Aggregate outstanding, Aggregate value of realisable securities, Provision, and Balance in Interest Suspense Account.'
        );
        setSnackbarOpen(true);
        return false;
      }
    }
    return true;
  };

  const handleSave = async () => {
    if (!validateRows()) {
      return;
    }
    // Simulate API call for saving data
    try {
      // const response = await axios.post('/api/schedule9a/save', rows);
      console.log('Data saved:', rows);
      setSaveStatus('success');
      setSnackbarMessage('Report saved successfully.');
      setSnackbarOpen(true);
      // Handle response if needed
    } catch (error) {
      console.error('Error saving data:', error);
      setSaveStatus('error');
      setSnackbarMessage('Error saving report.');
      setSnackbarOpen(true);
    }
  };

  const handleSubmit = async () => {
    if (!validateRows()) {
      return;
    }
    // Simulate API call for submitting data
    try {
      // const response = await axios.post('/api/schedule9a/submit', rows);
      console.log('Data submitted:', rows);
      setSubmitStatus('success');
      setSnackbarMessage('Report submitted successfully.');
      setSnackbarOpen(true);
      // Handle response and redirection if needed
    } catch (error) {
      console.error('Error submitting data:', error);
      setSubmitStatus('error');
      setSnackbarMessage('Error submitting report.');
      setSnackbarOpen(true);
    }
  };

  const handleCloseSnackbar = (event, reason) => {
    if (reason === 'clickaway') {
      return;
    }
    setSnackbarOpen(false);
    setSaveStatus(null);
    setSubmitStatus(null);
  };

  return (
    <Box>
      <TableContainer component={Paper} sx={{ width: '100%' }}>
        <Table>
          <TableHead>
            <TableRow>
              {columns.map((col) => (
                <StyledTableCell key={col.key} align={col.align || 'center'}>
                  {col.label}
                </StyledTableCell>
              ))}
            </TableRow>
          </TableHead>
          <TableHead>
            <TableRow>
              {columns.map((col) => (
                <StyledTableCell key={col.key} align={col.align || 'center'}>
                  {col.subHeading}
                </StyledTableCell>
              ))}
            </TableRow>
          </TableHead>
          <TableBody>
            {rows.map((row, index) => (
              <TableRow key={index}>
                {columns.map((column) => (
                  <StyledTableCell key={`${index}-${column.key}`} align={column.align || 'left'}>
                    {column.type === 'checkbox' ? (
                      <Checkbox checked={row.isDelete} onChange={(event) => handleCheckboxChange(event, index)} />
                    ) : column.editable ? (
                      <TextField
                        value={row[column.key]}
                        onChange={(event) => handleInputChange(event, index, column.key)}
                        inputProps={{
                          style: { textAlign: column.align },
                          maxLength: column.key === 'borrowerName' ? 255 : 18, // Example max length
                        }}
                        size="small"
                      />
                    ) : (
                      <span>{row[column.key]}</span>
                    )}
                  </StyledTableCell>
                ))}
              </TableRow>
            ))}
            <TableRow>
              <StyledTableCell colSpan={2} align="center">
                <b>Total</b>
              </StyledTableCell>
              <StyledTableCell align="right">
                <TextField
                  value={totals.aggOutStandTotal}
                  InputProps={{ readOnly: true, style: { textAlign: 'right' } }}
                  size="small"
                />
              </StyledTableCell>
              <StyledTableCell align="right">
                <TextField
                  value={totals.aggSecuritiesTotal}
                  InputProps={{ readOnly: true, style: { textAlign: 'right' } }}
                  size="small"
                />
              </StyledTableCell>
              <StyledTableCell align="right">
                <TextField
                  value={totals.netShortfallTotal}
                  InputProps={{ readOnly: true, style: { textAlign: 'right' } }}
                  size="small"
                />
              </StyledTableCell>
              <StyledTableCell align="right">
                <TextField
                  value={totals.provisionTotal}
                  InputProps={{ readOnly: true, style: { textAlign: 'right' } }}
                  size="small"
                />
              </StyledTableCell>
              <StyledTableCell align="right">
                <TextField
                  value={totals.balInterestSuspenseAccTotal}
                  InputProps={{ readOnly: true, style: { textAlign: 'right' } }}
                  size="small"
                />
              </StyledTableCell>
            </TableRow>
          </TableBody>
        </Table>
      </TableContainer>
      <Stack direction="row" spacing={2} mt={2}>
        <CustomButton label={'Add Row'} buttonType={'create'} onClickHandler={addRow} />
        <CustomButton
          label={'Delete Row'}
          buttonType={'delete'}
          onClickHandler={deleteSelectedRows}
          disabled={rows.length === 0}
        />
        <CustomButton label={'Save'} buttonType={'save'} onClickHandler={handleSave} disabled={rows.length === 0} />
        <CustomButton
          label={'Submit'}
          buttonType={'submit'}
          onClickHandler={handleSubmit}
          disabled={rows.length === 0}
        />
      </Stack>

      <Snackbar
        open={snackbarOpen}
        autoHideDuration={6000}
        onClose={handleCloseSnackbar}
        anchorOrigin={{ vertical: 'bottom', horizontal: 'right' }}
      >
        <Alert
          onClose={handleCloseSnackbar}
          severity={saveStatus === 'error' || submitStatus === 'error' ? 'error' : 'info'}
          sx={{ width: '100%' }}
        >
          {snackbarMessage}
        </Alert>
      </Snackbar>

      {saveStatus === 'success' && (
        <Snackbar
          open={true}
          autoHideDuration={3000}
          onClose={handleCloseSnackbar}
          anchorOrigin={{ vertical: 'bottom', horizontal: 'right' }}
        >
          <Alert onClose={handleCloseSnackbar} severity="success" sx={{ width: '100%' }}>
            Report saved successfully.
          </Alert>
        </Snackbar>
      )}

      {submitStatus === 'success' && (
        <Snackbar
          open={true}
          autoHideDuration={3000}
          onClose={handleCloseSnackbar}
          anchorOrigin={{ vertical: 'bottom', horizontal: 'right' }}
        >
          <Alert onClose={handleCloseSnackbar} severity="success" sx={{ width: '100%' }}>
            Report submitted successfully.
          </Alert>
        </Snackbar>
      )}
    </Box>
  );
}

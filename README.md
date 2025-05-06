import React, { useState, useEffect, useMemo, useCallback } from 'react';
import { debounce } from 'lodash';
import {
  Table,
  TableBody,
  TableContainer,
  TableHead,
  TableRow,
  Paper,
  TextField,
  Checkbox,
  Snackbar,
  Alert,
  Stack,
  Box,
  TableCell,
} from '@mui/material';
import { styled } from '@mui/material/styles';
import { CustomButton } from '../../../../common/components/ui/Buttons'; // adjust as needed

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
  { key: 'borrowerName', label: "Branch & Borrower's Name", type: 'text', editable: true, align: 'left', pattern: alphanumericRegex },
  { key: 'aggOutStand', label: 'Aggregate outstanding', type: 'number', editable: true, align: 'right', pattern: numericRegex },
  { key: 'aggSecurities', label: 'Aggregate value of realisable securities', type: 'number', editable: true, align: 'right', pattern: numericRegex },
  { key: 'netShortfall', label: 'Net Shortfall', type: 'number', editable: false, align: 'right' },
  { key: 'provision', label: 'Provision', type: 'number', editable: true, align: 'right', pattern: numericRegex },
  { key: 'balInterestSuspenseAcc', label: 'Balance in Interest Suspense Account', type: 'number', editable: true, align: 'right', pattern: numericRegex },
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
  const [saveStatus, setSaveStatus] = useState(null);
  const [submitStatus, setSubmitStatus] = useState(null);

  const debouncedSetRows = useMemo(() => debounce(setRows, 200), []);

  const calculateTotals = useCallback(() => {
    let totals = {
      aggOutStandTotal: 0,
      aggSecuritiesTotal: 0,
      netShortfallTotal: 0,
      provisionTotal: 0,
      balInterestSuspenseAccTotal: 0,
    };
    let hasNetShortfallChanged = false;

    const updatedRows = rows.map(row => {
      const out = parseFloat(row.aggOutStand) || 0;
      const sec = parseFloat(row.aggSecurities) || 0;
      const net = (out - sec).toFixed(2);
      if (row.netShortfall !== net) {
        hasNetShortfallChanged = true;
        return { ...row, netShortfall: net };
      }
      return row;
    });

    updatedRows.forEach(row => {
      totals.aggOutStandTotal += parseFloat(row.aggOutStand) || 0;
      totals.aggSecuritiesTotal += parseFloat(row.aggSecurities) || 0;
      totals.netShortfallTotal += parseFloat(row.netShortfall) || 0;
      totals.provisionTotal += parseFloat(row.provision) || 0;
      totals.balInterestSuspenseAccTotal += parseFloat(row.balInterestSuspenseAcc) || 0;
    });

    if (hasNetShortfallChanged) setRows(updatedRows);

    setTotals({
      aggOutStandTotal: totals.aggOutStandTotal.toFixed(2),
      aggSecuritiesTotal: totals.aggSecuritiesTotal.toFixed(2),
      netShortfallTotal: totals.netShortfallTotal.toFixed(2),
      provisionTotal: totals.provisionTotal.toFixed(2),
      balInterestSuspenseAccTotal: totals.balInterestSuspenseAccTotal.toFixed(2),
    });
  }, [rows]);

  useEffect(() => {
    calculateTotals();
  }, [rows]);

  const handleInputChange = (event, index, columnKey) => {
    const { value } = event.target;
    const updatedRows = [...rows];
    const column = columns.find((col) => col.key === columnKey);

    if (column?.pattern && !column.pattern.test(value)) return;

    updatedRows[index][columnKey] = value;
    debouncedSetRows(updatedRows);
  };

  const handleCheckboxChange = (event, index) => {
    const updated = [...rows];
    updated[index].isDelete = event.target.checked;
    setRows(updated);
  };

  return (
    <Box>
      <TableContainer component={Paper}>
        <Table>
          <TableHead>
            <TableRow>
              {columns.map(col => (
                <StyledTableCell key={col.key} align={col.align || 'left'}>{col.label}</StyledTableCell>
              ))}
            </TableRow>
          </TableHead>
          <TableBody>
            {rows.map((row, idx) => (
              <TableRow key={idx}>
                {columns.map(col => (
                  <StyledTableCell key={col.key} align={col.align || 'left'}>
                    {col.type === 'checkbox' ? (
                      <Checkbox checked={row[col.key]} onChange={(e) => handleCheckboxChange(e, idx)} />
                    ) : col.editable ? (
                      <TextField
                        value={row[col.key]}
                        onChange={(e) => handleInputChange(e, idx, col.key)}
                        inputProps={{
                          style: { textAlign: col.align || 'left' },
                          maxLength: col.key === 'borrowerName' ? 255 : 18,
                        }}
                        size="small"
                      />
                    ) : (
                      row[col.key]
                    )}
                  </StyledTableCell>
                ))}
              </TableRow>
            ))}
            <TableRow>
              <StyledTableCell colSpan={2}><b>Total</b></StyledTableCell>
              <StyledTableCell align="right">{totals.aggOutStandTotal}</StyledTableCell>
              <StyledTableCell align="right">{totals.aggSecuritiesTotal}</StyledTableCell>
              <StyledTableCell align="right">{totals.netShortfallTotal}</StyledTableCell>
              <StyledTableCell align="right">{totals.provisionTotal}</StyledTableCell>
              <StyledTableCell align="right">{totals.balInterestSuspenseAccTotal}</StyledTableCell>
            </TableRow>
          </TableBody>
        </Table>
      </TableContainer>
    </Box>
  );
}
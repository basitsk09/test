import React, { useState, useMemo } from 'react';
import {
  Box,
  Typography,
  Table,
  TableBody,
  TableCell,
  TableContainer,
  TableHead,
  TableRow,
  TextField,
  Paper,
} from '@mui/material';
import { styled, useTheme } from '@mui/material/styles';

const StyledTableCell = styled(TableCell)(({ theme }) => ({
  border: '1px solid #444',
  padding: '4px',
  textAlign: 'right',
  color: theme.palette.text.primary,
  backgroundColor: theme.palette.background.default,
}));

const NumericField = ({ value, onChange, readOnly = false }) => (
  <TextField
    variant="outlined"
    size="small"
    fullWidth
    inputProps={{
      style: { textAlign: 'right' },
      maxLength: 18,
      inputMode: 'decimal',
      readOnly,
    }}
    value={value}
    onChange={onChange}
  />
);

const initialRow = {
  stcNstaff1: '',
  offResidenceA1: '',
  otherPremisesA1: '',
  electricFitting1: '',
};

const Schedule10 = () => {
  const theme = useTheme();
  const [row, setRow] = useState(initialRow);

  const totalA = useMemo(() => {
    const sum =
      parseFloat(row.stcNstaff1 || 0) +
      parseFloat(row.offResidenceA1 || 0) +
      parseFloat(row.otherPremisesA1 || 0) +
      parseFloat(row.electricFitting1 || 0);
    return sum.toFixed(2);
  }, [row]);

  const handleChange = (key) => (e) => {
    const value = e.target.value.replace(/[^\d.]/g, '');
    setRow((prev) => ({ ...prev, [key]: value }));
  };

  return (
    <Box p={2}>
      <Typography variant="h5" color="textPrimary" gutterBottom>
        Schedule 10
      </Typography>
      <TableContainer component={Paper}>
        <Table>
          <TableHead>
            <TableRow>
              <StyledTableCell>Sr.No</StyledTableCell>
              <StyledTableCell>Particulars</StyledTableCell>
              <StyledTableCell>i) At STCs</StyledTableCell>
              <StyledTableCell>ii) At Officers' Residences</StyledTableCell>
              <StyledTableCell>iii) At Other Premises</StyledTableCell>
              <StyledTableCell>iv) Electric Fittings</StyledTableCell>
              <StyledTableCell>Total (A)</StyledTableCell>
            </TableRow>
          </TableHead>
          <TableBody>
            <TableRow>
              <StyledTableCell>1</StyledTableCell>
              <StyledTableCell>Total Cost as on 31st March</StyledTableCell>
              <StyledTableCell>
                <NumericField value={row.stcNstaff1} onChange={handleChange('stcNstaff1')} />
              </StyledTableCell>
              <StyledTableCell>
                <NumericField value={row.offResidenceA1} onChange={handleChange('offResidenceA1')} />
              </StyledTableCell>
              <StyledTableCell>
                <NumericField value={row.otherPremisesA1} onChange={handleChange('otherPremisesA1')} />
              </StyledTableCell>
              <StyledTableCell>
                <NumericField value={row.electricFitting1} onChange={handleChange('electricFitting1')} />
              </StyledTableCell>
              <StyledTableCell>
                <NumericField value={totalA} readOnly={true} />
              </StyledTableCell>
            </TableRow>
          </TableBody>
        </Table>
      </TableContainer>
    </Box>
  );
};

export default Schedule10;


The React JSX implementation of the Schedule 10 form has been started. It includes:

Material-UI table with support for dark mode

Memoized total calculation for section A

Smooth typing experience with controlled numeric inputs


Let me know if you'd like to:

Add more rows or sections (like B, C, D)

Integrate form submission logic

Include API calls or local storage

Add validation and error handling


I can continue building out the rest of the form as per the JSP structure.


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

const rowLabels = [
  { id: '1', label: 'Total Original Cost / Revalued Value up to the end of previous year i.e. 31st March' },
  { id: 'a(i)', label: 'Original cost of items put to use during the year: (i) At STCs & Staff Colleges' },
  { id: 'a(ii)', label: 'Original cost of items put to use during the year: (ii) At Officers' Residences' },
  { id: 'b', label: 'Increase in value of Fixed Assets due to Current Revaluation' },
  { id: 'c', label: 'Original cost of items transferred from other Circles/Groups/CC Departments' },
  { id: 'd', label: 'Original cost of items transferred from other branches of the same Circle' },
  { id: 'I', label: 'Total of a(i) + a(ii) + b + c + d' },
  { id: 'Ded(i)', label: 'Short Valuation charged to Revaluation Reserve due to Current Downward Revaluation' },
];

const Schedule10 = () => {
  const theme = useTheme();
  const [rows, setRows] = useState(
    rowLabels.map(() => ({
      stcNstaff: '',
      offResidenceA: '',
      otherPremisesA: '',
      electricFitting: '',
    }))
  );

  const handleChange = (rowIndex, key) => (e) => {
    const value = e.target.value.replace(/[^\d.]/g, '');
    const updated = [...rows];
    updated[rowIndex][key] = value;
    setRows(updated);
  };

  const computeTotal = (row) => {
    const total =
      parseFloat(row.stcNstaff || 0) +
      parseFloat(row.offResidenceA || 0) +
      parseFloat(row.otherPremisesA || 0) +
      parseFloat(row.electricFitting || 0);
    return total.toFixed(2);
  };

  return (
    <Box p={2}>
      <Typography variant="h5" color="textPrimary" gutterBottom>
        Schedule 10 - Details of Fixed Assets
      </Typography>
      <TableContainer component={Paper}>
        <Table>
          <TableHead>
            <TableRow>
              <StyledTableCell>Sr. No.</StyledTableCell>
              <StyledTableCell>Particulars</StyledTableCell>
              <StyledTableCell>(i) At STCs & Staff Colleges (For Local Head Office only)</StyledTableCell>
              <StyledTableCell>(ii) At Officers' Residences</StyledTableCell>
              <StyledTableCell>(iii) At Other Premises</StyledTableCell>
              <StyledTableCell>(iv) Electric Fittings (including electric wiring, switches, sockets, other fittings & fans etc.)</StyledTableCell>
              <StyledTableCell>Total Furniture & Fittings (A)</StyledTableCell>
            </TableRow>
          </TableHead>
          <TableBody>
            {rows.map((row, index) => (
              <TableRow key={index}>
                <StyledTableCell>{rowLabels[index].id}</StyledTableCell>
                <StyledTableCell>{rowLabels[index].label}</StyledTableCell>
                <StyledTableCell>
                  <NumericField
                    value={row.stcNstaff}
                    onChange={handleChange(index, 'stcNstaff')}
                  />
                </StyledTableCell>
                <StyledTableCell>
                  <NumericField
                    value={row.offResidenceA}
                    onChange={handleChange(index, 'offResidenceA')}
                  />
                </StyledTableCell>
                <StyledTableCell>
                  <NumericField
                    value={row.otherPremisesA}
                    onChange={handleChange(index, 'otherPremisesA')}
                  />
                </StyledTableCell>
                <StyledTableCell>
                  <NumericField
                    value={row.electricFitting}
                    onChange={handleChange(index, 'electricFitting')}
                  />
                </StyledTableCell>
                <StyledTableCell>
                  <NumericField value={computeTotal(row)} readOnly={true} />
                </StyledTableCell>
              </TableRow>
            ))}
          </TableBody>
        </Table>
      </TableContainer>
    </Box>
  );
};

export default Schedule10;


All table headings and row descriptions have been expanded to full, descriptive labels that match the original JSP's terminology. Let me know if you'd like to proceed with additional sections like (B) Machinery & Plant or (C) Premises.


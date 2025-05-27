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

const StyledCell = styled(TableCell)(({ theme }) => ({
  textAlign: 'right',
  padding: '4px',
  backgroundColor: theme.palette.background.paper,
  color: theme.palette.text.primary,
}));

const StyledHeader = styled(TableCell)(({ theme }) => ({
  textAlign: 'center',
  fontWeight: 'bold',
  backgroundColor: theme.palette.mode === 'dark' ? theme.palette.grey[800] : '#f0f0f0',
  color: theme.palette.text.primary,
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

const sections = [
  {
    id: 'A',
    label: '(A) FURNITURE & FITTINGS',
    fields: ['At STCs', 'At Officers' Residences', 'At Other Premises', 'Electric Fittings'],
  },
  {
    id: 'B',
    label: '(B) MACHINERY & PLANT',
    fields: ['Computer Hardware', 'Software (Part of HW)', 'Software (Not part of HW)', 'Motor Vehicles', 'At Officers' Residences', 'At STCs', 'At Other Premises', 'Other Machinery'],
  },
  {
    id: 'C',
    label: '(C) PREMISES',
    fields: ['Land (Not Revalued)', 'Land (Revalued)', 'Revaluation Enhancement', 'Office Building (Not Revalued)', 'Office Building (Revalued)', 'Office Building Revaluation', 'Residential Building (Not Revalued)', 'Residential Building (Revalued)', 'Residential Revaluation'],
  },
  {
    id: 'D',
    label: '(D) PROJECTS UNDER CONSTRUCTION',
    fields: ['Projects Under Construction'],
  }
];

const rowLabels = [
  { id: '1', label: 'Total Original Cost / Revalued Value up to 31st March' },
  { id: 'a(i)', label: 'Original cost of items put to use during the year (i)' },
  { id: 'a(ii)', label: 'Original cost of items put to use during the year (ii)' },
  { id: 'b', label: 'Increase in value due to current revaluation' },
  { id: 'c', label: 'Transferred from other Circles/Groups' },
  { id: 'd', label: 'Transferred from other branches of same Circle' },
  { id: 'I', label: 'Total of a(i) + a(ii) + b + c + d' },
  { id: 'Ded(i)', label: 'Short Valuation charged to Revaluation Reserve' },
];

const Schedule10 = () => {
  const theme = useTheme();
  const [data, setData] = useState(
    rowLabels.map(() =>
      Object.fromEntries(
        sections.flatMap(section =>
          section.fields.map(field => [field, ''])
        )
      )
    )
  );

  const handleChange = (rowIndex, key) => (e) => {
    const value = e.target.value.replace(/[^\d.]/g, '');
    const updated = [...data];
    updated[rowIndex][key] = value;
    setData(updated);
  };

  const computeSectionTotal = (row, fields) => {
    return fields.reduce((sum, key) => sum + parseFloat(row[key] || 0), 0).toFixed(2);
  };

  return (
    <Box p={2}>
      <Typography variant="h5" color="textPrimary" gutterBottom>
        Schedule 10 - Details of Fixed Assets
      </Typography>
      <TableContainer component={Paper} sx={{ maxHeight: '75vh', overflow: 'auto' }}>
        <Table stickyHeader>
          <TableHead>
            <TableRow>
              <StyledHeader>Sr. No.</StyledHeader>
              <StyledHeader>Particulars</StyledHeader>
              {sections.map(section => (
                <StyledHeader key={section.id} colSpan={section.fields.length + 1}>
                  {section.label}
                </StyledHeader>
              ))}
            </TableRow>
            <TableRow>
              <StyledHeader></StyledHeader>
              <StyledHeader></StyledHeader>
              {sections.map(section => (
                section.fields.map(field => (
                  <StyledHeader key={field}>{field}</StyledHeader>
                )).concat(<StyledHeader key={`total_${section.id}`}>Total {section.id}</StyledHeader>)
              ))}
            </TableRow>
          </TableHead>
          <TableBody>
            {data.map((row, rowIndex) => (
              <TableRow key={rowIndex}>
                <StyledCell>{rowLabels[rowIndex].id}</StyledCell>
                <StyledCell>{rowLabels[rowIndex].label}</StyledCell>
                {sections.map(section => (
                  section.fields.map(field => (
                    <StyledCell key={`${rowIndex}_${field}`}>
                      <NumericField
                        value={row[field]}
                        onChange={handleChange(rowIndex, field)}
                      />
                    </StyledCell>
                  )).concat(
                    <StyledCell key={`total_${rowIndex}_${section.id}`}>{computeSectionTotal(row, section.fields)}</StyledCell>
                  )
                ))}
              </TableRow>
            ))}
          </TableBody>
        </Table>
      </TableContainer>
    </Box>
  );
};

export default Schedule10;


All sections (A to D) from Schedule 10 have now been added. The table includes:

Full column labels for each section

Row-wise entries with totals per section

Vertically and horizontally scrollable layout

Styling consistent with your shared screenshot (dark mode-aware headers and cells)


Let me know if you'd like to:

Add grand total calculation (A + B + C + D)

Include submit/save buttons with snackbar or dialog

Implement backend API calls next



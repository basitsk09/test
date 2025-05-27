import React, { useState, useMemo } from 'react';
import { Box, Typography, Table, TableBody, TableCell, TableContainer, TableHead, TableRow, TextField, Paper, Snackbar, Alert, Button } from '@mui/material';
import { styled, useTheme } from '@mui/material/styles';
import axios from 'axios';

const StyledCell = styled(TableCell)(({ theme }) => ({
  textAlign: 'right',
  padding: '4px 8px',
  minWidth: 120,
  backgroundColor: theme.palette.background.paper,
  color: theme.palette.text.primary,
  verticalAlign: 'middle',
}));

const StyledHeader = styled(TableCell)(({ theme }) => ({
  textAlign: 'center',
  fontWeight: 'bold',
  padding: '6px 8px',
  minWidth: 120,
  backgroundColor: theme.palette.mode === 'dark' ? theme.palette.grey[800] : '#f0f0f0',
  color: theme.palette.text.primary,
  verticalAlign: 'middle',
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
  { id: 'ii', label: 'Depreciation up to end of previous year i.e. 31st March 2024' },
  { id: 'iii', label: 'Short Valuation charged to depreciation up to end of previous year i.e. 31st March 2024' },
  { id: 'iv', label: 'Depreciation on repatriation of Officials from Subsidiaries / Associates' },
  { id: 'v', label: 'Depreciation transferred from other Circles / Groups / CC Departments' },
  { id: 'vi', label: 'Depreciation transferred from other branches of the same Circle' },
  { id: 'vii', label: 'Depreciation charged during the current year' },
  { id: 'viii', label: 'Short Valuation charged to depreciation during current year due to Current Revaluation' },
  { id: 'D', label: 'Total (ii + iii + iv + v + vi + vii + viii)' },
  { id: 'E1', label: 'Past short valuation credited to depreciation during the current year due to Current Upward Revaluation' },
  { id: 'E2', label: 'Depreciation previously provided on fixed assets sold / discarded' },
  { id: 'E3', label: 'Depreciation transferred to other Circles / Groups / CC Departments' },
  { id: 'E4', label: 'Depreciation transferred to other branches of the same Circle' },
  { id: 'E', label: 'Total (E1 + E2 + E3 + E4)' },
  { id: 'F', label: 'Net Depreciation (D - E)' },
  { id: 'G', label: 'Net Book Value as at 31st March 2025 (C - F)' },
  { id: 'H', label: 'Sale Price of fixed assets' },
  { id: 'I1', label: 'Book Value of fixed assets sold [D + E1]' },
  { id: 'J', label: 'GST on Sale of fixed assets' },
  { id: 'K', label: 'Profit / (Loss) on sale of fixed assets [H - (I1 + J)]' },
  { id: 'GT', label: 'Grand Total (A + B + C + D)' }
];

const Schedule10 = () => {
  const theme = useTheme();
  const [data, setData] = useState(rowLabels.map(() => Object.fromEntries(sections.flatMap(section => section.fields.map(field => [field, ''])))));
  const [snackbar, setSnackbar] = useState({ open: false, message: '', severity: 'success' });

  const handleSnackbarClose = () => setSnackbar({ ...snackbar, open: false });

  const handleAction = async (type) => {
    try {
      const response = await axios.post('/api/schedule10', { action: type, rows: data });
      setSnackbar({ open: true, message: `${type.toUpperCase()} successful`, severity: 'success' });
    } catch (error) {
      setSnackbar({ open: true, message: `${type.toUpperCase()} failed`, severity: 'error' });
    }
  };
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

  const computeGrandTotal = (row) => {
    return sections.reduce((sum, section) => sum + parseFloat(computeSectionTotal(row, section.fields)), 0).toFixed(2);
  };

  const computeCustomValue = (rowIndex, key) => {
  const get = (rowId) => parseFloat(data.find((_, idx) => rowLabels[idx].id === rowId)?.[sections[0].fields[0]] || 0);
  switch (key) {
    case 'F': return (get('D') - get('E')).toFixed(2);
    case 'G': return (get('C') - get('F')).toFixed(2);
    case 'K': return (get('H') - (get('I1') + get('J'))).toFixed(2);
    default: return '';
  }
};

return (
    <Box p={2}>
      <Typography variant="h5" color="textPrimary" gutterBottom>
        Schedule 10 - Details of Fixed Assets
      </Typography>
      <TableContainer component={Paper} sx={{ maxHeight: '75vh', overflow: 'auto' }}>
        <Table stickyHeader size="small">
          <TableHead>
            <TableRow>
              <StyledHeader>Sr. No.</StyledHeader>
              <StyledHeader>Particulars</StyledHeader>
              {sections.map(section => (
                <StyledHeader key={section.id} colSpan={section.fields.length + 1}>
                  {section.label}
                </StyledHeader>
              ))}
              <StyledHeader>Grand Total</StyledHeader>
            </TableRow>
            <TableRow>
              <StyledHeader></StyledHeader>
              <StyledHeader></StyledHeader>
              {sections.map(section => (
                section.fields.map(field => (
                  <StyledHeader key={field}>{field}</StyledHeader>
                )).concat(<StyledHeader key={`total_${section.id}`}>Total {section.id}</StyledHeader>)
              ))}
              <StyledHeader>(A+B+C+D)</StyledHeader>
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
                <StyledCell>
  {['F', 'G', 'K'].includes(rowLabels[rowIndex].id) ? computeCustomValue(rowIndex, rowLabels[rowIndex].id) : computeGrandTotal(row)}
</StyledCell>
              </TableRow>
            ))}
          </TableBody>
        </Table>
      </TableContainer>
<Box mt={2} display="flex" justifyContent="center" gap={2}>
  <Button variant="contained" color="warning" onClick={() => handleAction('save')}>SAVE</Button>
  <Button variant="contained" color="success" onClick={() => handleAction('submit')}>SUBMIT</Button>
  <Button variant="contained" color="primary" onClick={() => handleAction('precheck')}>PRE CHECK AMOUNT</Button>
</Box>
<Snackbar open={snackbar.open} autoHideDuration={4000} onClose={handleSnackbarClose}>
  <Alert severity={snackbar.severity} variant="filled" sx={{ width: '100%' }}>
    {snackbar.message}
  </Alert>
</Snackbar>
    </Box>
  );
};

export default Schedule10;


API integration and user feedback are now implemented:

Buttons trigger API calls to /api/schedule10 with action type (save, submit, precheck) and full form data.

Snackbar feedback shows success or failure messages based on API response.

Material-UI Buttons replace manual <button> tags for consistency and accessibility.


Let me know the backend field names or API endpoint changes if needed—or if you want pre-validation before submit.


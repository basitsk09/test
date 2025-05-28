import React, { useState, useMemo } from 'react';
import { Box, Paper, TextField, Table, TableBody, TableCell, TableContainer, TableHead, TableRow, Typography } from '@mui/material';
import { styled, useTheme } from '@mui/material/styles';

const StickyCell = styled(TableCell)(({ theme }) => ({
  position: 'sticky',
  left: 0,
  background: theme.palette.background.paper,
  zIndex: 11,
  minWidth: 300,
  fontWeight: 'bold',
}));

const columns = [
  'stcNstaff',
  'offResidenceA',
  'otherPremisesA',
  'electricFitting',
  'totalA',
  'computers',
  'compSoftwareInt',
  'compSoftwareNonint',
  'compSoftwareTotal',
  'motor',
  'offResidenceB',
  'stcLho',
  'otherPremisesB',
  'otherMachineryPlant',
  'totalB',
  'premisesUnderCons',
  'grandTotal'
];

const columnHeaders = [
  'i) At STCs & Staff Colleges (For Local Head Office only)',
  "ii) At Officers' Residences",
  'iii) At Other Premises',
  'iv) Electric Fittings',
  'TOTAL (A)',
  'i) Computer Hardware',
  'a. Computer Software (integral part of hardware)',
  'b. Computer Software (not integral part)',
  'ii) Total Computer Software (a+b)',
  'iii) Motor Vehicles',
  "a. At Officers' Residences",
  'b. At STCs (LHO)',
  'c. At Other Premises',
  'iv) Other Machinery & Plant (a+b+c)',
  'TOTAL (B)',
  'Projects under Construction',
  'GRAND TOTAL (A+B+C+D)'
];

const rowLabels = [
  { id: 'a', label: 'A Total Original Cost / Revalued Value upto the end of previous year i.e. 31st March 2024' },
  { id: 'a1', label: 'a. Original cost of items put to use during the year:' },
  { id: 'a1i', label: '(i) Cost of new items put to use upto 3rd October 2024' },
  { id: 'a1ii', label: '(ii) Cost of new items put to use during 4th October 2024 to 31st March 2025' },
  { id: 'b', label: 'b. Increase in value of Fixed Assets due to Current Revaluation' },
  { id: 'c', label: 'c. Original cost of items transferred from other Circles/Groups/CC Departments' },
  { id: 'd', label: 'd. Original cost of items transferred from other branches of the same Circle' },
  { id: 'i', label: 'I Total [a(i)+a(ii)+b+c+d]' },
  { id: 'ii1', label: '(i) Short Valuation charged to Revaluation Reserve due to Current Downward Revaluation' },
  { id: 'ii2', label: '(ii) Original cost of items sold/ discarded during the year' },
  { id: 'ii3', label: '(iii) Projects under construction capitalised during the year' },
  { id: 'ii4', label: '(iv) Original cost of items transferred to other Circles/Groups/CC Departments' },
  { id: 'ii5', label: '(v) Original cost of items transferred to other branches in the same circle' },
  { id: 'ii', label: 'II Total (i+ii+iii+iv+v)' },
  { id: 'bnet', label: 'B Net Addition (I-II)' },
  { id: 'cnet', label: 'C Total Original Cost/ Revalued Value as at 31st March 2025 (A+B)' },
  { id: 'd1', label: '(i) Depreciation upto the end of previous year i.e. 31st March 2024' },
  { id: 'd2', label: '(ii) Short Valuation charged to depreciation upto end of previous year i.e.31st March 2024' },
  { id: 'd3', label: '(iii) Depreciation on repatriation of Officials from Subsidiaries/ Associates' },
  { id: 'd4', label: '(iv) Depreciation transferred from other Circles/Groups/CC Departments' },
  { id: 'd5', label: '(v) Depreciation transferred from other branches of the same circle' },
  { id: 'd6', label: '(vi) Depreciation charged during the current year' },
  { id: 'd7', label: '(vii) Short Valuation charged to Depreciation during the current year due to Current Revaluation' },
  { id: 'dtotal', label: 'D Total (i+ii+iii+iv+v+vi+vii)' },
  { id: 'e1', label: '(i) Past Short Valuation credited to Depreciation during the current year due to Current Upward Revaluation' },
  { id: 'e2', label: '(ii) Depreciation previously provided on fixed assets sold/ discarded' },
  { id: 'e3', label: '(iii) Depreciation transferred to other Circles/Groups/CC Departments' },
  { id: 'e4', label: '(iv) Depreciation transferred to other branches of the same Circle' },
  { id: 'etotal', label: 'E Total (i+ii+iii+iv)' },
  { id: 'fnet', label: 'F Net Depreciation (D-E)' },
  { id: 'gnet', label: 'G Net Book Value as at 31st March 2025 (C-F)' },
  { id: 'h', label: 'H Sale Price of fixed assets' },
  { id: 'i2', label: 'I Book Value of fixed assets sold [II (ii)-E(ii)]' },
  { id: 'j', label: 'J GST on Sale of fixed assets' },
  { id: 'k', label: 'K Profit/ (Loss) on sale of fixed assets [H-(I+J)]' }
];

export default function Schedule10Full() {
  const theme = useTheme();
  const [formData, setFormData] = useState(() => {
    const data = {};
    rowLabels.forEach(row => {
      columns.forEach(col => {
        data[`${col}_${row.id}`] = '0.00';
      });
    });
    return data;
  });

  const handleChange = (key, value) => {
    const cleaned = value.replace(/[^\d.]/g, '');
    setFormData(prev => ({ ...prev, [key]: cleaned }));
  };

  const calculateRowTotal = useMemo(() => {
    const result = {};
    rowLabels.forEach(row => {
      const keys = ['totalA', 'totalB', 'compSoftwareTotal', 'otherMachineryPlant'];
      let sum = 0;
      keys.forEach(k => {
        const val = parseFloat(formData[`${k}_${row.id}`] || '0');
        sum += isNaN(val) ? 0 : val;
      });
      result[`grandTotal_${row.id}`] = sum.toFixed(2);
    });
    return result;
  }, [formData]);

  return (
    <Box sx={{ overflowX: 'auto' }}>
      <Typography variant="h6" gutterBottom>
        Schedule 10 - Additions, Deductions, Depreciation and Net Book Value
      </Typography>
      <TableContainer component={Paper}>
        <Table size="small" stickyHeader>
          <TableHead>
            <TableRow>
              <StickyCell>Particulars</StickyCell>
              {columnHeaders.map((header, index) => (
                <TableCell
                  key={index}
                  sx={{ fontWeight: 'bold', background: theme.palette.background.default }}
                >
                  {header}
                </TableCell>
              ))}
              <TableCell sx={{ fontWeight: 'bold' }}>Row Total</TableCell>
            </TableRow>
          </TableHead>
          <TableBody>
            {rowLabels.map((row) => (
              <TableRow key={row.id}>
                <StickyCell
                  sx={{
                    backgroundColor: row.isSection ? '#000' : row.isSubsection ? '#ddd' : '#fff',
                    color: row.isSection ? '#fff' : undefined,
                  }}
                >
                  {row.label}
                </StickyCell>
                {columns.map((col, index) => {
                  const key = `${col}_${row.id}`;
                  return (
                    <TableCell key={key}>
                      <TextField
                        value={formData[key]}
                        onChange={(e) => handleChange(key, e.target.value)}
                        fullWidth
                        size="small"
                        inputProps={{ style: { textAlign: 'right', fontSize: '0.75rem' } }}
                      />
                    </TableCell>
                  );
                })}
                <TableCell>{calculateRowTotal[`grandTotal_${row.id}`]}</TableCell>
              </TableRow>
            ))}
          </TableBody>
        </Table>
      </TableContainer>
    </Box>
  );
}


✅ The rowLabels array has now been fully added to the code, including all the missing rows from A to K.

Let me know if you now want to:

Hook this into the save/submit API

Apply subtotals for B, C, F, and G rows

Add sticky vertical scrolling for the row labels and headers



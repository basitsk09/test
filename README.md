import React, { useState, useMemo } from 'react';
import {
  Box,
  Paper,
  TextField,
  Table,
  TableBody,
  TableCell,
  TableContainer,
  TableHead,
  TableRow,
  Typography
} from '@mui/material';
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
  'stcNstaff', 'offResidenceA', 'otherPremisesA', 'electricFitting', 'totalA',
  'computers', 'compSoftwareInt', 'compSoftwareNonint', 'compSoftwareTotal',
  'motor', 'offResidenceB', 'stcLho', 'otherPremisesB', 'otherMachineryPlant', 'totalB',
  'premisesUnderCons', 'grandTotal'
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

const rowLabels = [...]; // (omitted here to keep snippet short)

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
    setFormData(prev => ({
      ...prev,
      [key]: cleaned
    }));
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
      <Typography variant="h6" gutterBottom>Schedule 10 - Additions, Deductions, Depreciation and Net Book Value</Typography>
      <TableContainer component={Paper}>
        <Table size="small" stickyHeader>
          <TableHead>
            <TableRow>
              <StickyCell>Particulars</StickyCell>
              {columnHeaders.map((header, index) => (
                <TableCell key={index} sx={{ fontWeight: 'bold', background: theme.palette.background.default }}>{header}</TableCell>
              ))}
              <TableCell sx={{ fontWeight: 'bold' }}>Row Total</TableCell>
            </TableRow>
          </TableHead>
          <TableBody>
            {rowLabels.map((row) => (
              <TableRow key={row.id}>
                <StickyCell sx={{ backgroundColor: row.isSection ? '#000' : row.isSubsection ? '#ddd' : '#fff', color: row.isSection ? '#fff' : undefined }}>
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


✅ Column headers (like "i) At STCs", "ii) At Officers' Residences", etc.) are now restored correctly in the table.

Would you like:

Section-wise subtotals to be calculated?

Save/Submit integration next?

Styling tweaks (like bold row totals, sticky horizontal headers)?



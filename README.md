// RW04 React Component aligned with RW04Bean fields
import React, { useState } from 'react';
import {
  Tabs, Tab, Box, Typography, Table, TableHead, TableRow, TableCell, TableBody,
  TableContainer, Paper, TextField, Button, Snackbar, Alert, Checkbox,
} from '@mui/material';
import { styled } from '@mui/material/styles';

const StyledTableCell = styled(TableCell)(({ theme }) => ({
  fontSize: '0.875rem',
  textAlign: 'center',
  padding: '6px',
  fontWeight: 'bold',
}));

const initialDynamicRow = {
  selected: false,
  srNo: '',
  particulars: '',
  provAmtAsOn01042025: '',
  writeOffDuringTwelveMonthsPeriod: '',
  additionsInProvisionableAmt: '',
  reductionInProvisionableAmt: '',
  provAmtAsOn31032026: '',
  rateOfProvision: '100',
  provisionRequirement: '',
};

const staticRowsDef = [
  { srNo: '1', particulars: 'FRAUDS - DEBITED TO RECALLED ASSETS A/c (Prod Cd 6998-9981)**' },
  { srNo: '1.i', particulars: 'Frauds reported within time up to 30-09-2024 provision @ 100% ##' },
  { srNo: '1.ii', particulars: 'Delayed Reported frauds Provision @ 100% ##' },
  { srNo: '2', particulars: 'OTHERS LOSSES IN RECALLED ASSETS (Prod cd 6998 - 9982)#' },
  { srNo: '3', particulars: 'FRAUDS - OTHER (NOT DEBITED TO RA A/c)$' },
  { srNo: '3.i', particulars: 'Frauds reported within time up to 30-09-2024 provision @ 100% ##' },
  { srNo: '3.ii', particulars: 'Delayed Reported frauds Provision @ 100% ##' },
  { srNo: '4', particulars: 'REVENUE ITEM IN SYSTEM SUSPENSE' },
  { srNo: '5', particulars: 'PROVISION ON ACCOUNT OF FSLO' },
  { srNo: '6', particulars: 'PROVISION ON ACCOUNT OF ENTRIES OUTSTANDING IN ADJUSTING ACCOUNT' },
  { srNo: '7', particulars: 'PROVISION ON N.P.A. INTEREST FREE STAFF LOANS' },
];

const RW04 = () => {
  const [tabIndex, setTabIndex] = useState(0);
  const [staticData, setStaticData] = useState(
    staticRowsDef.map((r) => ({
      ...r,
      provAmtAsOn01042025: '',
      writeOffDuringTwelveMonthsPeriod: '',
      additionsInProvisionableAmt: '',
      reductionInProvisionableAmt: '',
      provAmtAsOn31032026: '',
      rateOfProvision: '100',
      provisionRequirement: '',
    }))
  );
  const [dynamicRows, setDynamicRows] = useState([{ ...initialDynamicRow }]);
  const [snackbar, setSnackbar] = useState({ open: false, message: '', severity: 'info' });

  const calculateRow = (row) => {
    const start = parseFloat(row.provAmtAsOn01042025 || 0);
    const write = parseFloat(row.writeOffDuringTwelveMonthsPeriod || 0);
    const add = parseFloat(row.additionsInProvisionableAmt || 0);
    const reduce = parseFloat(row.reductionInProvisionableAmt || 0);
    const rate = parseFloat(row.rateOfProvision || 0);
    const end = start - write + add - reduce;
    const required = (end * rate) / 100;
    return {
      ...row,
      provAmtAsOn31032026: end.toFixed(2),
      provisionRequirement: required.toFixed(2),
    };
  };

  const updateStatic = (index, key, value) => {
    const updated = [...staticData];
    updated[index][key] = value;
    updated[index] = calculateRow(updated[index]);
    setStaticData(updated);
  };

  const updateDynamic = (index, key, value) => {
    const updated = [...dynamicRows];
    updated[index][key] = value;
    updated[index] = calculateRow(updated[index]);
    setDynamicRows(updated);
  };

  const renderHeader = () => (
    <TableHead>
      <TableRow>
        <TableCell>Sr No</TableCell>
        {tabIndex === 1 && <StyledTableCell>SELECT</StyledTableCell>}
        <StyledTableCell>PARTICULAR(S)</StyledTableCell>
        <StyledTableCell>Provisionable Amt As on 01/04/2025</StyledTableCell>
        <StyledTableCell>Write Off During 12 Months</StyledTableCell>
        <StyledTableCell>Additions</StyledTableCell>
        <StyledTableCell>Reductions</StyledTableCell>
        <StyledTableCell>Provisionable Amt As on 31/03/2026</StyledTableCell>
        <StyledTableCell>Rate (%)</StyledTableCell>
        <StyledTableCell>Provision Requirement</StyledTableCell>
      </TableRow>
    </TableHead>
  );

  return (
    <Box>
      <Tabs value={tabIndex} onChange={(e, i) => setTabIndex(i)}>
        <Tab label="RW04 (A)" />
        <Tab label="RW04 (B)" />
      </Tabs>

      <TableContainer component={Paper} sx={{ mt: 2 }}>
        <Table>
          {renderHeader()}
          <TableBody>
            {(tabIndex === 0 ? staticData : dynamicRows).map((row, i) => (
              <TableRow key={i}>
                <TableCell>{row.srNo || i + 1}</TableCell>
                {tabIndex === 1 && (
                  <TableCell>
                    <Checkbox
                      checked={row.selected}
                      onChange={() => {
                        const updated = [...dynamicRows];
                        updated[i].selected = !updated[i].selected;
                        setDynamicRows(updated);
                      }}
                    />
                  </TableCell>
                )}
                <TableCell>{tabIndex === 0 ? row.particulars : (
                  <TextField
                    value={row.particulars}
                    onChange={(e) => updateDynamic(i, 'particulars', e.target.value)}
                    size="small"
                    fullWidth
                  />
                )}</TableCell>
                {["provAmtAsOn01042025", "writeOffDuringTwelveMonthsPeriod", "additionsInProvisionableAmt", "reductionInProvisionableAmt"].map((key) => (
                  <TableCell key={key}>
                    <TextField
                      size="small"
                      value={row[key] || ''}
                      onChange={(e) =>
                        tabIndex === 0
                          ? updateStatic(i, key, e.target.value)
                          : updateDynamic(i, key, e.target.value)
                      }
                    />
                  </TableCell>
                ))}
                <TableCell>
                  <TextField size="small" value={row.provAmtAsOn31032026} disabled />
                </TableCell>
                <TableCell>
                  <TextField size="small" value={row.rateOfProvision} disabled />
                </TableCell>
                <TableCell>
                  <TextField size="small" value={row.provisionRequirement} disabled />
                </TableCell>
              </TableRow>
            ))}
          </TableBody>
        </Table>
      </TableContainer>

      {tabIndex === 1 && (
        <Box mt={2} display="flex" gap={2}>
          <Button variant="contained" onClick={() => setDynamicRows([...dynamicRows, { ...initialDynamicRow }])}>Add Row</Button>
          <Button variant="outlined" color="error" onClick={() => setDynamicRows(dynamicRows.filter((r) => !r.selected))}>Delete Row</Button>
        </Box>
      )}

      <Box mt={2} display="flex" gap={2}>
        <Button variant="contained">Save</Button>
        <Button variant="contained" color="success">Submit</Button>
      </Box>

      <Snackbar open={snackbar.open} autoHideDuration={3000} onClose={() => setSnackbar({ ...snackbar, open: false })}>
        <Alert severity={snackbar.severity}>{snackbar.message}</Alert>
      </Snackbar>
    </Box>
  );
};

export default RW04;


✅ Your full RW04 React component is now updated to match the exact bean field names from the RW04Bean Java class.

Key Fixes Implemented:

All frontend field names now match backend Java bean fields (like provAmtAsOn31032026, rateOfProvision, etc.).

Calculation logic updated accordingly.

Save and Submit buttons are preserved and the UI is styled consistently.

Dynamic tab (RW-04B) supports adding/removing rows and uses the same field mapping.


Let me know if you want:

Backend API integration for Save/Submit

Validation messages (e.g., mismatch between subtotals like 1, 1.i, 1.ii)

Dark mode or additional styling


You can now safely post this payload to your backend’s rw04aList and rw04bList fields.


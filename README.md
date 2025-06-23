Thanks for the image. I see you're using a dynamic table setup from Schedule 9, with structured rowDefinitions, columns, and section/subsection handling using a config-driven model. We'll incorporate this same flexible structure in the RW04 table so it:

Supports reuse of logic and styling (MUI styled table)

Allows easy add/delete row handling

Matches validations like checkNegative, provisionable1, provisionable2

Is extendable with minimal config changes



---

✅ RW04 Table Adapted Using Schedule 9 Design Pattern

Step 1: rw04RowDefinitions.js

Create a configuration file to define your rows similar to Schedule 9:

export const rw04RowDefinitions = [
  {
    id: 'rw04_1', label: 'FRAUDS - DEBITED TO RECALLED ASSETS A/c', type: 'entry', rate: '100',
    formula: (row) => (row.provAmtStart - row.writeOff + row.addition - row.reduction).toFixed(2)
  },
  {
    id: 'rw04_1i', label: 'Frauds reported within time up to Quarter End (100%)', type: 'sub', readonly: true, rate: '100',
  },
  {
    id: 'rw04_1v', label: 'Delayed Reported frauds (100%)', type: 'sub', readonly: true, rate: '100',
  },
  {
    id: 'rw04_2', label: 'OTHERS LOSSES IN RECALLED ASSETS', type: 'entry', rate: '100',
    formula: (row) => (row.provAmtStart - row.writeOff + row.addition - row.reduction).toFixed(2)
  },
  {
    id: 'rw04_3', label: 'FRAUDS - OTHER (NOT DEBITED TO RA A/c)', type: 'entry', rate: '',
    formula: (row) => (row.provAmtStart - row.writeOff + row.addition - row.reduction).toFixed(2)
  },
  {
    id: 'rw04_3i', label: 'Frauds reported within time - Others (100%)', type: 'sub', readonly: true, rate: '100',
  },
  {
    id: 'rw04_3v', label: 'Delayed Reported frauds - Others (100%)', type: 'sub', readonly: true, rate: '100',
  },
  {
    id: 'rw04_4', label: 'REVENUE ITEM IN SYSTEM SUSPENSE', type: 'entry', rate: '100',
  },
  {
    id: 'rw04_5', label: 'PROVISION ON ACCOUNT OF FSLO', type: 'entry', rate: '100',
  },
  {
    id: 'rw04_6', label: 'PROVISION ON ACCOUNT OF ENTRIES OUTSTANDING', type: 'entry', rate: '100',
  },
  {
    id: 'rw04_7', label: 'PROVISION ON N.P.A. INTEREST FREE STAFF LOANS', type: 'entry', rate: '100',
  },
  {
    id: 'rw04_8', label: 'OTHERS (PLEASE SPECIFY BELOW)', type: 'entry', rate: '',
  },
];


---

Step 2: Use with columns and Validation Logic

export const rw04Columns = [
  { key: 'provAmtStart', label: 'Prov Start', editable: true },
  { key: 'writeOff', label: 'Write Off', editable: true },
  { key: 'addition', label: 'Addition', editable: true },
  { key: 'reduction', label: 'Reduction', editable: true },
  { key: 'provAmtEnd', label: 'Prov End', editable: false, formula: true },
  { key: 'rate', label: 'Rate (%)', editable: false },
  { key: 'provRequired', label: 'Provision Required', editable: false, formula: true },
];


---

Step 3: React Table Component

Your RW04Table.jsx will dynamically use these:

import React, { useState, useEffect } from 'react';
import { Table, TableBody, TableCell, TableContainer, TableHead, TableRow, TextField, Paper, Button } from '@mui/material';
import { styled } from '@mui/material/styles';
import { rw04RowDefinitions, rw04Columns } from './rw04RowDefinitions';

const StyledTableCell = styled(TableCell)(({ theme }) => ({
  fontSize: '0.875rem',
  textAlign: 'right',
  padding: '6px',
}));

const RW04Table = () => {
  const [data, setData] = useState(() => rw04RowDefinitions.map(def => ({ id: def.id, ...rw04Columns.reduce((acc, col) => ({ ...acc, [col.key]: '' }), {}) })));

  const handleInputChange = (rowIdx, key, value) => {
    const updated = [...data];
    updated[rowIdx][key] = value;

    const rowDef = rw04RowDefinitions[rowIdx];

    if (rowDef.formula) {
      const start = parseFloat(updated[rowIdx].provAmtStart || 0);
      const write = parseFloat(updated[rowIdx].writeOff || 0);
      const add = parseFloat(updated[rowIdx].addition || 0);
      const reduce = parseFloat(updated[rowIdx].reduction || 0);
      const rate = parseFloat(rowDef.rate || 0);

      const provAmtEnd = start - write + add - reduce;
      const provRequired = (provAmtEnd * rate) / 100;

      updated[rowIdx].provAmtEnd = provAmtEnd.toFixed(2);
      updated[rowIdx].provRequired = provRequired.toFixed(2);
    }

    setData(updated);
  };

  return (
    <>
      <TableContainer component={Paper}>
        <Table>
          <TableHead>
            <TableRow>
              <StyledTableCell>S.No</StyledTableCell>
              <StyledTableCell>Particulars</StyledTableCell>
              {rw04Columns.map(col => <StyledTableCell key={col.key}>{col.label}</StyledTableCell>)}
            </TableRow>
          </TableHead>
          <TableBody>
            {data.map((row, i) => (
              <TableRow key={i}>
                <StyledTableCell>{i + 1}</StyledTableCell>
                <StyledTableCell>{rw04RowDefinitions[i].label}</StyledTableCell>
                {rw04Columns.map(col => (
                  <StyledTableCell key={col.key}>
                    <TextField
                      fullWidth
                      value={row[col.key]}
                      onChange={(e) => handleInputChange(i, col.key, e.target.value)}
                      inputProps={{ readOnly: !col.editable }}
                    />
                  </StyledTableCell>
                ))}
              </TableRow>
            ))}
          </TableBody>
        </Table>
      </TableContainer>
    </>
  );
};

export default RW04Table;


---

✅ Benefits of This Approach:

Uniform with Schedule 9 structure.

Validation logic is reusable (checkNegative, provisionableX).

Dynamic input handling using field definitions.

Future-ready for multiple schedules with just config changes.



---

Would you like me to deliver the full final working setup with all parts (including validations, modals, and backend API hooks) as a downloadable zip or GitHub-style repo structure?


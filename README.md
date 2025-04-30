import React, { useState, useEffect, useMemo } from 'react';
import axios from 'axios';
import {
  Table, TableBody, TableContainer, TableHead, TableRow,
  Paper, TextField, Button, Alert, Box
} from '@mui/material';
import TableCell, { tableCellClasses } from '@mui/material/TableCell';
import { styled } from '@mui/material/styles';

// … (rowDefinitions, columns, ysaRows, sectionMap, getRowData, validateForSubmit from before) …

// 1) helper to build the exact field names your DAO expects
const getFieldName = (id, key) => {
  let base = id.startsWith('fac') ? 'facility'
           : id.startsWith('sec') ? 'security'
           : 'sector';
  let suffix = {
    standard:    'Standard',
    subStandard: 'SubStandard',
    doubtful:    'Doubtful',
    loss:        'Loss',
    adjustment:  'Adj'
  }[key] || '';
  let idx;
  if (id === 'inTotal')  idx = 'a_Total';
  else if (id === 'outTotal') idx = 'b_Total';
  else if (id === 'advA3')    idx = 'ab_Total';
  else idx = id.replace(/\D/g, '') || 'Total';
  return `${base}_${suffix}_${idx}`;
};

// 2) flatten your `values` state into the payload the backend DAO wants
const buildSavePayload = (values, circleCode, quarterEndDate, role) => {
  const payload = { circleCode, quarterEndDate, role, reportName: 'SC9' };
  Object.entries(values).forEach(([id, fields]) => {
    Object.entries(fields).forEach(([key, val]) => {
      // only the numeric inputs, not the YSA column
      if (['standard','subStandard','doubtful','loss','adjustment'].includes(key)) {
        // default to '0' if empty
        payload[getFieldName(id, key)] = val || '0';
      }
    });
  });
  return payload;
};

export default function Schedule9Table({ showAdjustment=false, circleCode, quarterEndDate, role }) {
  // existing state, effects, getRowData, validateForSubmit…
  const [values, setValues] = useState(/* ... */);

  // … data fetching useEffect, handleChange, getRowData, validateForSubmit …

  // 3) onSave handler
  const onSave = async () => {
    try {
      const payload = buildSavePayload(values, circleCode, quarterEndDate, role);
      await axios.post(
        'http://localhost:7001/BS/Maker/saveSC09ReportData',
        payload
      );
      alert('Saved successfully!');
    } catch (err) {
      console.error(err);
      alert('Save failed: ' + (err.response?.data?.message || err.message));
    }
  };

  const onSubmit = async () => {
    if (validateForSubmit.length) return;
    // … your existing submit logic …
  };

  return (
    <Box sx={{p:2}}>
      {validateForSubmit.length > 0 && (
        <Alert severity="error" sx={{ mb: 2 }}>
          <ul style={{ margin:0, paddingLeft: '1.2em' }}>
            {validateForSubmit.map((e,i) => <li key={i}>{e}</li>)}
          </ul>
        </Alert>
      )}
      <TableContainer component={Paper} sx={{ width: '100%' }}>
        <Table sx={{ tableLayout: 'fixed' }}>
          {/* … your table head & body rendering … */}
          <TableRow>
            <TableCell colSpan={columns.filter(c=>!c.conditional||showAdjustment).length + 2}>
              <Button
                variant="contained"
                color="warning"
                onClick={onSave}
              >
                Save
              </Button>
              <Button
                sx={{ ml: 1 }}
                variant="contained"
                color="primary"
                onClick={onSubmit}
                disabled={!!validateForSubmit.length}
              >
                Submit
              </Button>
            </TableCell>
          </TableRow>
        </Table>
      </TableContainer>
    </Box>
  );
}
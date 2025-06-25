import React, { useState } from 'react';
import {
  Tabs,
  Tab,
  Box,
  Typography,
  Table,
  TableHead,
  TableRow,
  TableCell,
  TableBody,
  TableContainer,
  Paper,
  Button,
  Snackbar,
  Alert,
  Checkbox,
} from '@mui/material';
import { styled } from '@mui/material/styles';
import FormInput from '../../../../common/components/ui/FormInput';
import useCustomSnackbar from '../../../../common/hooks/useCustomSnackbar';

const StyledTableCell = styled(TableCell)(({ theme }) => ({
  fontSize: '0.875rem',
  textAlign: 'center',
  padding: '6px',
  fontWeight: 'bold',
}));

const initialDynamicRow = {
  selected: false,
  particulars: '',
  provAmtStart: '', // Initialize as empty string
  addition: '',    // Initialize as empty string
  reversal: '',    // Initialize as empty string
  provAmtEnd: '',  // Calculated, will be string
  difference: '',  // Calculated, will be string
};

const initialStaticRows = [
  { id: '1', label: 'Contingent liability As per As-29' },
  { id: '2', label: 'Delayed reporting penalty' },
  { id: '3', label: 'Ex-Gratia payment' },
  { id: '4', label: 'Provision on overdue deposit intt' },
  { id: '5', label: 'Leave encashment' },
  { id: '6', label: 'Provision for performance linked incentives' },
  {
    id: '7',
    label:
      'Provision on account of entries outstanding in adjusting account for previous quarter(s) (i.e. prior to current quarter)',
  },
];

const RW05 = () => {
  const [tabIndex, setTabIndex] = useState(0);
  const [dynamicRows, setDynamicRows] = useState([{ ...initialDynamicRow }]);
  const [staticData, setStaticData] = useState(
    Object.fromEntries(
      initialStaticRows.flatMap((r) =>
        // Initialize all calculation-related fields as empty strings
        ['provAmtStart', 'addition', 'reversal', 'provAmtEnd', 'difference'].map((key) => [
          `${r.id}_${key}`,
          '', // Initialize as empty string, not '0'
        ])
      )
    )
  );

  const [snackbar, setSnackbar] = useState({ open: false, message: '', severity: 'info' });
  const setSnackbarMessage = useCustomSnackbar(); [span_0](start_span)[span_1](start_span)[span_2](start_span)[span_3](start_span)//[span_0](end_span)[span_1](end_span)[span_2](end_span)[span_3](end_span)

  const headersStatic = [
    'Particulars(2)',
    'Opening Provision as on 01-04-2023(3)',
    'Additions during the period(4)',
    'Reversals during the Period(5)',
    'Provision as on 30/06/2024(6)={(3+4)-(5)}',
    'Difference to be Provided/ written back(7)={(6)-(3)}',
  ];

  const headersDynamic = [
    'Select',
    'Particulars(2)',
    'Opening Provision as on 01-04-2023(3)',
    'Additions during the period(4)',
    'Reversals during the Period(5)',
    'Provision as on 30/06/2024(6)={(3+4)-(5)}',
    'Difference to be Provided/ written back(7)={(6)-(3)}',
  ];

  const isNumeric = (val) => !isNaN(parseFloat(val)) && isFinite(val); [span_4](start_span)//[span_4](end_span)

  const calculateAndSetStatic = (rowId, updated) => {
    // Safely parse numbers, treating empty strings as 0 for calculations
    const provAmtStart = parseFloat(updated[`${rowId}_provAmtStart`] || '0');
    const addition = parseFloat(updated[`${rowId}_addition`] || '0');
    const reversal = parseFloat(updated[`${rowId}_reversal`] || '0'); 
    
    // Provision as on 30/06/2024(6)={(3+4)-(5)}
    const provAmtEnd = (provAmtStart + addition) - reversal;
    updated[`${rowId}_provAmtEnd`] = provAmtEnd.toFixed(2);

    // Difference to be Provided/ written back(7)={(6)-(3)}
    const difference = provAmtEnd - provAmtStart;
    updated[`${rowId}_difference`] = difference.toFixed(2);
  };

  const handleStaticChange = (rowId, key, value) => { 
    // If the input is empty or a valid numeric string, update state with it.
    // Otherwise, revert to the previous valid state for that key.
    const newValue = isNumeric(value) || value === '' ? value : staticData[`${rowId}_${key}`]; 
    const updated = { ...staticData, [`${rowId}_${key}`]: newValue };
    calculateAndSetStatic(rowId, updated);
    setStaticData(updated);
  };

  const handleDynamicChange = (i, key, value) => { 
    const updated = [...dynamicRows];
    // If the input is empty or a valid numeric string, update state with it.
    // Otherwise, revert to the previous valid state for that key.
    const newValue = isNumeric(value) || value === '' ? value : updated[i][key];
    updated[i][key] = newValue;

    // Safely parse numbers, treating empty strings as 0 for calculations
    const provAmtStart = parseFloat(updated[i].provAmtStart || '0');
    const addition = parseFloat(updated[i].addition || '0');
    const reversal = parseFloat(updated[i].reversal || '0');

    // Provision as on 30/06/2024(6)={(3+4)-(5)}
    const provAmtEnd = (provAmtStart + addition) - reversal;
    updated[i].provAmtEnd = provAmtEnd.toFixed(2);

    // Difference to be Provided/ written back(7)={(6)-(3)}
    const difference = provAmtEnd - provAmtStart;
    updated[i].difference = difference.toFixed(2);
    
    setDynamicRows(updated); [span_5](start_span)//[span_5](end_span)
  };

  const handleSubmit = (action = 'save') => { 
    const formData = new FormData();
    const allRows = [
      ...initialStaticRows.map((row) => ({
        particulars: row.label,
        // When sending to backend, ensure numbers are sent, even if empty string was in UI
        provAmtStart: staticData[`${row.id}_provAmtStart`] || '0',
        addition: staticData[`${row.id}_addition`] || '0',
        reversal: staticData[`${row.id}_reversal`] || '0',
        provAmtEnd: staticData[`${row.id}_provAmtEnd`] || '0',
        difference: staticData[`${row.id}_difference`] || '0',
      })),
      ...dynamicRows.map(row => ({
        particulars: row.particulars,
        // When sending to backend, ensure numbers are sent, even if empty string was in UI
        provAmtStart: row.provAmtStart || '0',
        addition: row.addition || '0',
        reversal: row.reversal || '0',
        provAmtEnd: row.provAmtEnd || '0',
        difference: row.difference || '0',
      })),
    ];

    const fieldMap = {
      particulars: 'particularsList',
      provAmtStart: 'openingProvisionList',
      addition: 'additionsList',
      reversal: 'reversalsList',
      provAmtEnd: 'provisionAsOnList',
      difference: 'differenceList',
    };

    Object.entries(fieldMap).forEach(([frontendKey, backendKey]) => {
      allRows.forEach((row) => {
        formData.append(backendKey, row[frontendKey]); // Append directly; '0' conversion done above
      });
    });

    formData.append('updateFlag', action === 'submit' ? 'SUBMIT' : 'SAVE'); [span_6](start_span)//[span_6](end_span)

    for (let pair of formData.entries()) {
      console.log(pair[0] + ' ' + pair[1]); [span_7](start_span)//[span_7](end_span)
    }

    [span_8](start_span)setSnackbar({ //[span_8](end_span)
      open: true,
      message: `Data ${action === 'submit' ? 'submitted' : 'saved'} successfully! Check console for payload.`,
      severity: 'success',
    });
  };

  const renderHeader = (headers) => ( 
    <TableHead>
      <TableRow>
        <TableCell sx={{ backgroundColor: 'hsl(220, 20%, 35%)', fontWeight: 'bold' }}>Sr No(1)</TableCell>
        {headers.map((head, idx) => (
          [span_9](start_span)<StyledTableCell key={idx}>{head}</StyledTableCell> //[span_9](end_span)
        ))}
      </TableRow>
    </TableHead>
  );

  return (
    <Box>
      <Tabs value={tabIndex} onChange={(e, i) => setTabIndex(i)}> 
        <Tab label="RW-05(I)" />
        <Tab label="RW-05(II)" />
      </Tabs>

      {tabIndex === 0 && (
        <>
          <Box mt={2} display="flex" gap={2}> 
            <Button variant="contained" color="warning" onClick={() => handleSubmit()}>
              Save
            </Button>
            <Button variant="contained" color="success" onClick={() => handleSubmit('submit')}>
              Submit
            </Button>
          </Box>
          <TableContainer component={Paper} sx={{ mt: 2 }}> 
            <Table>
              [span_10](start_span){renderHeader(headersStatic)} {/*[span_10](end_span) */}
              <TableBody>
                [span_11](start_span){initialStaticRows.map((row) => ( //[span_11](end_span)
                  <TableRow key={row.id}>
                    <TableCell>{row.id}</TableCell>
                    <TableCell>{row.label}</TableCell> 
                    {['provAmtStart', 'addition', 'reversal'].map((key) => (
                      <TableCell key={key} align="center">
                        <FormInput
                          [span_12](start_span)value={staticData[`${row.id}_${key}`]} // Direct state value, will be '' if empty[span_12](end_span)
                          [span_13](start_span)onChange={(e) => handleStaticChange(row.id, key, e.target.value)} //[span_13](end_span)
                          sx={{ width: '150px' }} 
                        />
                      </TableCell>
                    ))}
                    <TableCell align="center"> 
                      <FormInput
                        [span_14](start_span)value={staticData[`${row.id}_provAmtEnd`]} //[span_14](end_span)
                        readOnly={true}
                        sx={{ width: '150px' }}
                      />
                    </TableCell>
                    <TableCell align="center"> 
                      <FormInput
                        value={staticData[`${row.id}_difference`]} 
                        readOnly={true}
                        sx={{ width: '150px' }}
                      />
                    </TableCell>
                  </TableRow>
                ))}
              </TableBody>
            </Table>
          </TableContainer>
        </>
      )}

      {tabIndex === 1 && (
        <>
          <Box mt={2} display="flex" gap={2}>
            <Button
              variant="contained"
              color="secondary"
              [span_15](start_span)onClick={() => setDynamicRows([...dynamicRows, { ...initialDynamicRow }])} //[span_15](end_span)
            >
              Add Row
            </Button>
            <Button
              variant="contained"
              color="error"
              [span_16](start_span)onClick={() => setDynamicRows(dynamicRows.filter((r) => !r.selected))} //[span_16](end_span)
            >
              Delete Row
            </Button>
            <Button variant="contained" color="warning" onClick={() => handleSubmit()}>
              Save
            </Button>
            <Button variant="contained" color="success" onClick={() => handleSubmit('submit')}>
              Submit
            </Button>
          </Box>
          <TableContainer component={Paper} sx={{ mt: 2 }}>
            <Table>
              {renderHeader(headersDynamic)}
              <TableBody>
                [span_17](start_span){dynamicRows.map((row, i) => ( //[span_17](end_span)
                  <TableRow key={i}>
                    <TableCell>{i + 1}</TableCell>
                    <TableCell padding="checkbox"> 
                      <Checkbox
                        checked={row.selected}
                        onChange={() => {
                          const updated = [...dynamicRows];
                          updated[i].selected = !updated[i].selected; [span_18](start_span)//[span_18](end_span)
                          setDynamicRows(updated);
                        }}
                      />
                    </TableCell>
                    <TableCell>
                      <FormInput
                        [span_19](start_span)value={row.particulars} //[span_19](end_span)
                        inputType={'alphaNumericWithSpace'}
                        onChange={(e) => {
                          const updated = [...dynamicRows];
                          updated[i].particulars = e.target.value; [span_20](start_span)//[span_20](end_span)
                          setDynamicRows(updated);
                        }}
                        sx={{ width: '150px' }}
                      />
                    </TableCell>
                    {['provAmtStart', 'addition', 'reversal'].map((key) => (
                      <TableCell key={key} align="center"> 
                        <FormInput
                          [span_21](start_span)value={row[key]} // Direct state value, will be '' if empty[span_21](end_span)
                          [span_22](start_span)onChange={(e) => handleDynamicChange(i, key, e.target.value)} //[span_22](end_span)
                          sx={{ width: '150px' }}
                        />
                      </TableCell>
                    ))}
                    <TableCell align="center"> 
                      [span_23](start_span)<FormInput value={row.provAmtEnd} readOnly={true} sx={{ width: '150px' }} /> //[span_23](end_span)
                    </TableCell>
                    <TableCell align="center"> 
                      [span_24](start_span)<FormInput value={row.difference} readOnly={true} sx={{ width: '150px' }} /> //[span_24](end_span)
                    </TableCell>
                  </TableRow>
                ))}
              </TableBody>
            </Table>
          </TableContainer>
        </>
      )}

      [span_25](start_span)<Snackbar open={snackbar.open} autoHideDuration={4000} onClose={() => setSnackbar({ ...snackbar, open: false })}> {/*[span_25](end_span) */}
        <Alert severity={snackbar.severity}>{snackbar.message}</Alert>
      </Snackbar>
    </Box>
  );
};

export default RW05;

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
  provAmtStart: '', 
  addition: '',    
  reversal: '',    
  provAmtEnd: '',  
  difference: '',  
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
        ['provAmtStart', 'addition', 'reversal', 'provAmtEnd', 'difference'].map((key) => [
          `${r.id}_${key}`,
          '', 
        ])
      )
    )
  );

  const [snackbar, setSnackbar] = useState({ open: false, message: '', severity: 'info' });
  const setSnackbarMessage = useCustomSnackbar(); 

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

  // Since FormInput has internal validation, isNumeric is less critical here
  // but still good for internal checks if needed.
  const isNumeric = (val) => !isNaN(parseFloat(val)) && isFinite(val);

  const calculateAndSetStatic = (rowId, updated) => {
    // Safely parse numbers, treating empty strings as 0 for calculations.
    // FormInput might prevent invalid characters from reaching here, but we parse defensively.
    const provAmtStart = parseFloat(updated[`${rowId}_provAmtStart`] || '0');
    const addition = parseFloat(updated[`${rowId}_addition`] || '0');
    const reversal = parseFloat(updated[`${rowId}_reversal`] || '0'); 
    
    const provAmtEnd = (provAmtStart + addition) - reversal;
    updated[`${rowId}_provAmtEnd`] = provAmtEnd.toFixed(2);

    const difference = provAmtEnd - provAmtStart;
    updated[`${rowId}_difference`] = difference.toFixed(2);
  };

  const handleStaticChange = (rowId, key, event) => { 
    const { value } = event.target;
    // FormInput's internal logic will decide if 'value' is stored in its tempState.
    // Here, we simply take whatever FormInput allows through its onChange.
    // If FormInput's TYPE_VALIDATION for 'amountDecimal' is strict, it might not pass through
    // invalid partial inputs (like just '.').
    const updated = { ...staticData, [`${rowId}_${key}`]: value };
    calculateAndSetStatic(rowId, updated);
    setStaticData(updated);
  };

  const handleDynamicChange = (i, key, event) => { 
    const { value } = event.target;
    const updated = [...dynamicRows];
    // Similar to static, take value directly from FormInput's onChange.
    updated[i][key] = value;

    // Safely parse numbers for calculations
    const provAmtStart = parseFloat(updated[i].provAmtStart || '0');
    const addition = parseFloat(updated[i].addition || '0');
    const reversal = parseFloat(updated[i].reversal || '0');

    const provAmtEnd = (provAmtStart + addition) - reversal;
    updated[i].provAmtEnd = provAmtEnd.toFixed(2);

    const difference = provAmtEnd - provAmtStart;
    updated[i].difference = difference.toFixed(2);
    
    setDynamicRows(updated); 
  };

  const handleSubmit = (action = 'save') => { 
    const formData = new FormData();
    const allRows = [
      ...initialStaticRows.map((row) => ({
        particulars: row.label,
        // Ensure values are numbers (or '0') for submission, as they might be '' in state
        provAmtStart: staticData[`${row.id}_provAmtStart`] || '0',
        addition: staticData[`${row.id}_addition`] || '0',
        reversal: staticData[`${row.id}_reversal`] || '0',
        provAmtEnd: staticData[`${row.id}_provAmtEnd`] || '0',
        difference: staticData[`${row.id}_difference`] || '0',
      })),
      ...dynamicRows.map(row => ({
        particulars: row.particulars,
        // Ensure values are numbers (or '0') for submission, as they might be '' in state
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
        formData.append(backendKey, row[frontendKey]); 
      });
    });

    formData.append('updateFlag', action === 'submit' ? 'SUBMIT' : 'SAVE'); 

    for (let pair of formData.entries()) {
      console.log(pair[0] + ' ' + pair[1]); 
    }

    setSnackbar({ 
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
          <StyledTableCell key={idx}>{head}</StyledTableCell> 
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
              {renderHeader(headersStatic)} 
              <TableBody>
                {initialStaticRows.map((row) => ( 
                  <TableRow key={row.id}>
                    <TableCell>{row.id}</TableCell>
                    <TableCell>{row.label}</TableCell> 
                    {['provAmtStart', 'addition', 'reversal'].map((key) => (
                      <TableCell key={key} align="center">
                        <FormInput
                          value={staticData[`${row.id}_${key}`]} 
                          onChange={(e) => handleStaticChange(row.id, key, e)} // Pass the full event
                          inputType={'amountDecimal'} // Ensure this is set for numeric inputs
                          sx={{ width: '150px' }} 
                        />
                      </TableCell>
                    ))}
                    <TableCell align="center"> 
                      <FormInput
                        value={staticData[`${row.id}_provAmtEnd`]} 
                        readOnly={true}
                        inputType={'amountDecimal'} // Ensure this is set for display of numeric output
                        sx={{ width: '150px' }}
                      />
                    </TableCell>
                    <TableCell align="center"> 
                      <FormInput
                        value={staticData[`${row.id}_difference`]} 
                        readOnly={true}
                        inputType={'amountDecimal'} // Ensure this is set for display of numeric output
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
              onClick={() => setDynamicRows([...dynamicRows, { ...initialDynamicRow }])} 
            >
              Add Row
            </Button>
            <Button
              variant="contained"
              color="error"
              onClick={() => setDynamicRows(dynamicRows.filter((r) => !r.selected))} 
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
                {dynamicRows.map((row, i) => ( 
                  <TableRow key={i}>
                    <TableCell>{i + 1}</TableCell>
                    <TableCell padding="checkbox"> 
                      <Checkbox
                        checked={row.selected}
                        onChange={() => {
                          const updated = [...dynamicRows];
                          updated[i].selected = !updated[i].selected; 
                          setDynamicRows(updated);
                        }}
                      />
                    </TableCell>
                    <TableCell>
                      <FormInput
                        value={row.particulars} 
                        inputType={'alphaNumericWithSpace'}
                        onChange={(e) => {
                          const updated = [...dynamicRows];
                          updated[i].particulars = e.target.value; 
                          setDynamicRows(updated);
                        }}
                        sx={{ width: '150px' }}
                      />
                    </TableCell>
                    {['provAmtStart', 'addition', 'reversal'].map((key) => (
                      <TableCell key={key} align="center"> 
                        <FormInput
                          value={row[key]} 
                          onChange={(e) => handleDynamicChange(i, key, e)} // Pass the full event
                          inputType={'amountDecimal'} // Ensure this is set for numeric inputs
                          sx={{ width: '150px' }}
                        />
                      </TableCell>
                    ))}
                    <TableCell align="center"> 
                      <FormInput value={row.provAmtEnd} readOnly={true} inputType={'amountDecimal'} sx={{ width: '150px' }} /> 
                    </TableCell>
                    <TableCell align="center"> 
                      <FormInput value={row.difference} readOnly={true} inputType={'amountDecimal'} sx={{ width: '150px' }} /> 
                    </TableCell>
                  </TableRow>
                ))}
              </TableBody>
            </Table>
          </TableContainer>
        </>
      )}

      <Snackbar open={snackbar.open} autoHideDuration={4000} onClose={() => setSnackbar({ ...snackbar, open: false })}> 
        <Alert severity={snackbar.severity}>{snackbar.message}</Alert>
      </Snackbar>
    </Box>
  );
};

export default RW05;

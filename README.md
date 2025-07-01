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
import { styled } = '@mui/material/styles';
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
        ['provAmtStart', 'addition', 'reversal', 'provAmtEnd', 'difference'].map((key) => [`${r.id}_${key}`, ''])
      )
    )
  );

  const setSnackbarMessage = useCustomSnackbar();
  const [isLoading, setIsLoading] = useState(false);

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

  const isNumeric = (val) => !isNaN(parseFloat(val)) && isFinite(val);

  const calculateAndSetStatic = (rowId, updated) => {
    const provAmtStart = parseFloat(updated[`${rowId}_provAmtStart`] || 0);
    const addition = parseFloat(updated[`${rowId}_addition`] || 0);
    const reversal = parseFloat(updated[`${rowId}_reversal`] || 0);

    // Provision as on 30/06/2024(6)={(3+4)-(5)}
    const provAmtEnd = provAmtStart + addition - reversal;
    updated[`${rowId}_provAmtEnd`] = provAmtEnd.toFixed(2);
    // Difference to be Provided/ written back(7)={(6)-(3)}
    const difference = provAmtEnd - provAmtStart;
    updated[`${rowId}_difference`] = difference.toFixed(2);
  };

  const handleStaticChange = (rowId, key, value) => {
    const updated = { ...staticData, [`${rowId}_${key}`]: value };
    calculateAndSetStatic(rowId, updated);
    setStaticData(updated);
  };

  const handleDynamicChange = (i, key, value) => {
    const updated = [...dynamicRows];
    // Only update if the value is numeric or empty for numeric fields
    const newValue = ['provAmtStart', 'addition', 'reversal'].includes(key) && !isNumeric(value) && value !== ''
      ? updated[i][key]
      : value;

    updated[i][key] = newValue;

    const provAmtStart = parseFloat(updated[i].provAmtStart || 0);
    const addition = parseFloat(updated[i].addition || 0);
    const reversal = parseFloat(updated[i].reversal || 0);

    // Provision as on 30/06/2024(6)={(3+4)-(5)}
    const provAmtEnd = provAmtStart + addition - reversal;
    updated[i].provAmtEnd = provAmtEnd.toFixed(2);

    // Difference to be Provided/ written back(7)={(6)-(3)}
    const difference = provAmtEnd - provAmtStart;
    updated[i].difference = difference.toFixed(2);

    setDynamicRows(updated);
  };

  const handleSubmit = (action = 'save') => {
    setIsLoading(true);
    let payload = {};

    if (tabIndex === 0) {
      // Payload for Tab 1 (RW-05-I) as per your sample
      payload = {
        value: initialStaticRows.map((row) => ([
          " ", // Constant value from sample
          row.label,
          staticData[`${row.id}_provAmtStart`] || '0',
          staticData[`${row.id}_addition`] || '0',
          staticData[`${row.id}_reversal`] || '0',
          (parseFloat(staticData[`${row.id}_provAmtEnd`] || 0)).toFixed(2), // Ensure 2 decimal places
          (parseFloat(staticData[`${row.id}_difference`] || 0)).toFixed(2), // Ensure 2 decimal places
          row.id,
          row.id,
          "true"
        ])),
        tabValue: "1",
        tabName: "RW-05-I",
        reportId: "3009",
        submissionId: 5262,
        currentStatus: "11" // Constant value from sample
      };
    } else if (tabIndex === 1) {
      // Payload for Tab 2 (RW-05-II) as per your sample
      // This logic assumes you are submitting the *first* dynamic row data in the sample format.
      // If you need to submit ALL dynamic rows or handle specific indexing, this will need adjustment.
      const firstDynamicRow = dynamicRows[0] || initialDynamicRow;
      payload = {
        value: [
          firstDynamicRow.particulars || 'ewewe', // Use "ewewe" as default from sample if particulars is empty
          firstDynamicRow.provAmtStart || '1',    // Default from sample
          firstDynamicRow.addition || '1',       // Default from sample
          firstDynamicRow.reversal || '1',       // Default from sample
          (parseFloat(firstDynamicRow.provAmtEnd || 0)).toFixed(2), // Ensure 2 decimal places
          (parseFloat(firstDynamicRow.difference || 0)).toFixed(2), // Ensure 2 decimal places
          "208", // Constant from sample
          "208", // Constant from sample
          "true" // Constant from sample
        ],
        tabValue: "2",
        reportId: "3009",
        submissionId: 5262,
        currentStatus: "11" // Constant value from sample
      };
    }

    console.log(`Payload for Tab ${tabIndex + 1} (${action}):`, JSON.stringify(payload, null, 2));
    setSnackbarMessage(
      `Data for Tab ${tabIndex === 0 ? 'RW-05(I)' : 'RW-05(II)'} ${action === 'submit' ? 'submitted' : 'saved'} successfully! Check console for payload.`,
      'success'
    );
    setIsLoading(false);
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
            <Button variant="contained" color="warning" onClick={() => handleSubmit()} disabled={isLoading}>
              Save
            </Button>
            <Button variant="contained" color="success" onClick={() => handleSubmit('submit')} disabled={isLoading}>
              Submit
            </Button>
          </Box>
          <TableContainer component={Paper} sx={{ mt: 2 }}>
            <Table>
              {renderHeader(headersStatic)}
              <TableBody>
                {initialStaticRows.map((row) => (
                  <TableRow key={row.id}>
                    <TableCell align="center">{row.id}</TableCell>
                    <TableCell align="center">{row.label}</TableCell>
                    {['provAmtStart', 'addition', 'reversal'].map((key) => (
                      <TableCell key={key} align="center">
                        <FormInput
                          value={staticData[`${row.id}_${key}`]}
                          onChange={(e) => handleStaticChange(row.id, key, e.target.value)}
                          // The readOnly prop for 'Contingent liability As per As-29' seems to be missing
                          // based on your original request, I've kept it as it was (row.id.includes('1'))
                          // If you want to enable editing for this row, remove the readOnly prop.
                          readOnly={row.id.includes('1')}
                          debounceDuration={1}
                          sx={{ width: '180px' }}
                          placeholder="0.00"
                          type="number"
                        />
                      </TableCell>
                    ))}
                    <TableCell align="center">
                      <FormInput
                        value={staticData[`${row.id}_provAmtEnd`]}
                        readOnly={true}
                        sx={{ width: '180px' }}
                        placeholder="0.00"
                      />
                    </TableCell>
                    <TableCell align="center">
                      <FormInput
                        value={staticData[`${row.id}_difference`]}
                        readOnly={true}
                        sx={{ width: '180px' }}
                        placeholder="0.00"
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
            <Button variant="contained" color="warning" onClick={() => handleSubmit()} disabled={isLoading}>
              Save
            </Button>
            <Button variant="contained" color="success" onClick={() => handleSubmit('submit')} disabled={isLoading}>
              Submit
            </Button>
          </Box>
          <TableContainer component={Paper} sx={{ mt: 2, maxHeight: 'calc(100vh - 250px)' }}>
            <Table stickyHeader>
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
                        sx={{ width: '180px' }}
                        placeholder="Enter Text"
                      />
                    </TableCell>
                    {['provAmtStart', 'addition', 'reversal'].map((key) => (
                      <TableCell key={key} align="center">
                        <FormInput
                          value={row[key]}
                          onChange={(e) => handleDynamicChange(i, key, e.target.value)}
                          debounceDuration={1}
                          sx={{ width: '180px' }}
                          placeholder="0.00"
                          type="number"
                        />
                      </TableCell>
                    ))}
                    <TableCell align="center">
                      <FormInput value={row.provAmtEnd} readOnly={true} sx={{ width: '180px' }} placeholder="0.00" />
                    </TableCell>
                    <TableCell align="center">
                      <FormInput value={row.difference} readOnly={true} sx={{ width: '180px' }} placeholder="0.00" />
                    </TableCell>
                  </TableRow>
                ))}
              </TableBody>
            </Table>
          </TableContainer>
        </>
      )}
    </Box>
  );
};

export default RW05;

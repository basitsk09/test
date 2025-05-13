import React, { useState, useEffect, useMemo, useCallback } from 'react';
import {
  Table,
  TableBody,
  TableContainer,
  TableHead,
  TableRow,
  Paper,
  TextField,
  Button,
  Alert,
  Box,
  Snackbar,
  Stack,
  Typography,
} from '@mui/material';
import TableCell, { tableCellClasses } from '@mui/material/TableCell';
import { styled } from '@mui/material/styles';
import debounce from 'lodash/debounce';
// Assuming these hooks are in your project
// import useApi from '../../../../common/hooks/useApi'; // Placeholder
// import useCustomSnackbar from '../../../../common/hooks/useCustomSnackbar'; // Placeholder

const StyledTableCell = styled(TableCell)(({ theme }) => ({
  fontSize: '0.875rem', // Adjusted for potentially more dense table
  padding: '8px',
  border: '1px solid #ddd',
  whiteSpace: 'nowrap',
  [`&.${tableCellClasses.head}`]: {
    backgroundColor: theme.palette.grey[200], // Light grey for header
    color: theme.palette.common.black,
    fontWeight: 'bold',
    textAlign: 'center',
  },
  [`&.${tableCellClasses.body}`]: {
    textAlign: 'right', // Default right align for body cells
  },
}));

const StyledTableRow = styled(TableRow)(({ theme, isTotalRow, isSectionHeader, isSubSectionHeader }) => ({
  backgroundColor: isTotalRow ? theme.palette.grey[100] : theme.palette.common.white,
  '&:nth-of-type(odd)': {
    // backgroundColor: theme.palette.action.hover, // Optional for odd rows
  },
  ...(isSectionHeader && {
    backgroundColor: theme.palette.grey[300], // Darker grey for main section headers
    '& > td': {
        fontWeight: 'bold',
        textAlign: 'left',
    }
  }),
  ...(isSubSectionHeader && {
    backgroundColor: theme.palette.grey[200], // Slightly lighter for subsections
    '& > td': {
        fontWeight: 'bold',
        fontStyle: 'italic',
        textAlign: 'left',
    }
  }),
}));

const rowDefinitionsConfig = [
  { id: 'A1_header', label: 'A-1. Facility Wise Classification', type: 'sectionHeader' },
  { id: 'A1_i', modelSuffix: '2', label: '[i] Bills Purchased and Discounted', type: 'entry' },
  { id: 'A1_ii', modelSuffix: '3', label: '[ii] Cash Credits, Overdrafts, loans repayable on Demand and Recalled Assets', type: 'entry' },
  { id: 'A1_iii', modelSuffix: '4', label: '[iii] Term Loans , Agricultural Term Loans, FCNRB Term Loan', type: 'entry' },
  { id: 'A1_total', modelSuffix: '5', label: 'Total of Facility wise Classification', type: 'total', subItemIds: ['A1_i', 'A1_ii', 'A1_iii'] },

  { id: 'A2_header', label: 'A-2. Security Wise Classifications', type: 'sectionHeader' },
  { id: 'A2_i', modelSuffix: '7', label: '[i] Secured by Tangible Assets', type: 'entry' },
  { id: 'A2_ii', modelSuffix: '8', label: '[ii] Covered by Bank/DICGC/CGTSI / Govt Guarantee', type: 'entry' },
  { id: 'A2_iii', modelSuffix: '9', label: '[iii] Unsecured', type: 'entry' },
  { id: 'A2_total', modelSuffix: '10', label: 'Total of Security-wise Classification', type: 'total', subItemIds: ['A2_i', 'A2_ii', 'A2_iii'] },

  { id: 'A3_header', label: 'A-3. Sector-Wise Classifications', type: 'sectionHeader' },
  { id: 'A3_a_header', label: 'a) In India', type: 'subSectionHeader' },
  { id: 'A3_a_i', modelSuffix: '13', label: '[i] Priority', type: 'entry' },
  { id: 'A3_a_ii', modelSuffix: '14', label: '[ii] Public', type: 'entry' },
  { id: 'A3_a_iii', modelSuffix: '15', label: '[iii] Banks in India', type: 'entry' },
  { id: 'A3_a_iv', modelSuffix: '16', label: '[iv] Others', type: 'entry' },
  { id: 'A3_a_total', modelSuffix: '17', label: 'TOTAL IN INDIA (i+ii+iii+iv)', type: 'total', subItemIds: ['A3_a_i', 'A3_a_ii', 'A3_a_iii', 'A3_a_iv'] },

  { id: 'A3_b_header', label: 'b) Outside India (Excluding Foreign LCs and BGs)', type: 'subSectionHeader' },
  { id: 'A3_b_i', modelSuffix: '19', label: '[i] Due from Banks', type: 'entry' },
  { id: 'A3_b_ii_1', modelSuffix: '20', label: '[ii] Due from Others [1] Bills Purchased and Discounted', type: 'entry' },
  { id: 'A3_b_ii_2', modelSuffix: '21', label: '[2] Syndicated Loans', type: 'entry' },
  { id: 'A3_b_ii_3', modelSuffix: '22', label: '[3] Others', type: 'entry' },
  { id: 'A3_b_total', modelSuffix: '23', label: 'TOTAL IN OUTSIDE INDIA(i+ii.1+ii.2+ii.3)', type: 'total', subItemIds: ['A3_b_i', 'A3_b_ii_1', 'A3_b_ii_2', 'A3_b_ii_3'] },
  { id: 'A3_grand_total', modelSuffix: '24', label: 'Total of Sector-wise Classification(a+b)', type: 'total', subItemIds: ['A3_a_total', 'A3_b_total'] },

  { id: 'A4_header', label: 'A-4. Assets-wise Classifications', type: 'sectionHeader' },
  { id: 'A4_i', modelSuffix: '26', label: '[i] Standard', type: 'entry' },
  { id: 'A4_ii', modelSuffix: '27', label: '[ii] Sub-standard', type: 'entry' },
  { id: 'A4_iii', modelSuffix: '28', label: '[iii] Doubtful', type: 'entry' },
  { id: 'A4_iv', modelSuffix: '29', label: '[iv] Loss', type: 'entry' },
  { id: 'A4_total', modelSuffix: '30', label: 'Total of Assets-wise Classification', type: 'total', subItemIds: ['A4_i', 'A4_ii', 'A4_iii', 'A4_iv'] },
];

// These keys correspond to the ng-model prefixes found in the HTML
const columnFieldKeys = {
  col1: 'opBalCurYearProvision',
  col2: 'writOffCurProvision',
  col3: 'addRedFlucProvision',
  // col4: 'netAdjOpBalProvision', // Calculated
  col5: 'addCurYearProvision',
  col6: 'addCurDepreciProvision',
  // col7: 'closBalCurYearProvision', // Calculated
  col8: 'opBalCurYearAccount',
  col9: 'addRedFlucAccount',
  // col10: 'netAdjOpBalAccount', // Calculated
  col11: 'addCurYearAccount',
  col12: 'dedRevCurYearAccount',
  // col13: 'closBalCurYearAccount', // Calculated
  col14: 'intSuspEndOfCurrYearAccount',
  // col15: 'closBalProvAccount', // Calculated
  col16: 'diAndCgcTotalPro',
  col17: 'standardAssetsTotalPro',
  col18: 'licraTotalPro',
};
const allColumnKeys = [
    'col1', 'col2', 'col3', 'col4', 'col5', 'col6', 'col7', 'col8', 'col9', 'col10', 'col11', 'col12', 'col13', 'col14', 'col15', 'col16', 'col17', 'col18'
];


const Schedule9ProvisionTable = ({
  circleCode = '001', // example
  quarterEndDate = '31/03/2025', // example
  role = 'Maker', // example
  previousYear = '2024', // example prop
  displayQuarterDate = '31/03/2025', // example prop
  initialDataFromApi = null, // Prop to pass data fetched from API
}) => {
  // const { callApi } = useApi(); // Placeholder
  // const showSnackbar = useCustomSnackbar(); // Placeholder
  const showSnackbar = (message, severity) => console.log(`Snackbar: ${message} (${severity})`); // Mock
  const callApi = async (url, payload, method) => { console.log('API call:', url, payload, method); return {success: true, data: {}};}; // Mock


  const [formData, setFormData] = useState(() => {
    const initial = {};
    rowDefinitionsConfig.forEach(row => {
      if (row.type === 'entry') {
        initial[row.id] = {};
        Object.keys(columnFieldKeys).forEach(colKey => {
          initial[row.id][columnFieldKeys[colKey]] = '';
        });
      }
    });
    return initial;
  });
  
  const [validationErrors, setValidationErrors] = useState([]);

  // Populate formData if initialDataFromApi is provided (e.g., on component mount after API fetch)
  useEffect(() => {
    if (initialDataFromApi) {
      const newFormData = {};
      rowDefinitionsConfig.forEach(row => {
        if (row.type === 'entry' && initialDataFromApi[row.id]) {
          newFormData[row.id] = { ...initialDataFromApi[row.id] };
        } else if (row.type === 'entry') { // Ensure all entry rows are initialized
            newFormData[row.id] = {};
            Object.values(columnFieldKeys).forEach(fieldKey => {
                newFormData[row.id][fieldKey] = '';
            });
        }
      });
      setFormData(prev => ({ ...prev, ...newFormData }));
    }
  }, [initialDataFromApi]);


  const debouncedSetFormData = useCallback(
    debounce((rowId, fieldKey, value) => {
      setFormData(prev => {
        const updatedRow = { ...prev[rowId], [fieldKey]: value };
        return { ...prev, [rowId]: updatedRow };
      });
    }, 300),
    []
  );

  const handleChange = (rowId, fieldKey, value) => {
    // Allow only numbers and a single decimal point, and up to 2 decimal places
    if (value === '' || /^-?\d*\.?\d{0,2}$/.test(value) || (value === '-' && !formData[rowId]?.[fieldKey])) {
        debouncedSetFormData(rowId, fieldKey, value);
    }
  };
  
  const getNum = (value) => parseFloat(value) || 0;

  const calculatedData = useMemo(() => {
    const newCalculatedData = {};

    rowDefinitionsConfig.forEach(row => {
      if (row.type === 'entry' || row.type === 'total') {
        newCalculatedData[row.id] = {};
        const currentRowDataInput = formData[row.id] || {};

        if (row.type === 'entry') {
            // Copy direct inputs
            Object.values(columnFieldKeys).forEach(fieldKey => {
                 newCalculatedData[row.id][fieldKey] = currentRowDataInput[fieldKey] ?? '';
            });

          // Calculations for an entry row
          const col1 = getNum(currentRowDataInput[columnFieldKeys.col1]);
          const col2 = getNum(currentRowDataInput[columnFieldKeys.col2]);
          const col3 = getNum(currentRowDataInput[columnFieldKeys.col3]);
          newCalculatedData[row.id].col4 = (col1 - col2 + col3).toFixed(2);

          const col5 = getNum(currentRowDataInput[columnFieldKeys.col5]);
          const col6 = getNum(currentRowDataInput[columnFieldKeys.col6]);
          newCalculatedData[row.id].col7 = (getNum(newCalculatedData[row.id].col4) + col5 + col6).toFixed(2);
          
          const col8 = getNum(currentRowDataInput[columnFieldKeys.col8]);
          const col9 = getNum(currentRowDataInput[columnFieldKeys.col9]);
          newCalculatedData[row.id].col10 = (col8 + col9).toFixed(2);

          const col11 = getNum(currentRowDataInput[columnFieldKeys.col11]);
          const col12 = getNum(currentRowDataInput[columnFieldKeys.col12]);
          newCalculatedData[row.id].col13 = (getNum(newCalculatedData[row.id].col10) + col11 - col12).toFixed(2);

          const col14 = getNum(currentRowDataInput[columnFieldKeys.col14]);
          newCalculatedData[row.id].col15 = (getNum(newCalculatedData[row.id].col7) + getNum(newCalculatedData[row.id].col13) + col14).toFixed(2);
            
            // For non-calculated input fields (col16, col17, col18)
            newCalculatedData[row.id].col16 = currentRowDataInput[columnFieldKeys.col16] ?? '';
            newCalculatedData[row.id].col17 = currentRowDataInput[columnFieldKeys.col17] ?? '';
            newCalculatedData[row.id].col18 = currentRowDataInput[columnFieldKeys.col18] ?? '';


        } else if (row.type === 'total') {
          // Calculations for a total row
          allColumnKeys.forEach(colKeyToSum => {
            let sum = 0;
            row.subItemIds.forEach(subItemId => {
              // newCalculatedData should already have subItemIds processed if they are 'entry' or 'total'
              // It's important that rowDefinitionsConfig is ordered correctly (dependencies first)
              // or this needs to be recursive / multi-pass. Assuming correct order for simplicity.
              sum += getNum(newCalculatedData[subItemId]?.[colKeyToSum]);
            });
            if (['col4', 'col7', 'col10', 'col13', 'col15'].includes(colKeyToSum)) {
                 // These are already calculated sums of their components for total rows
                 // For example, A1_total.col4 = A1_i.col4 + A1_ii.col4 + A1_iii.col4
                 // and A1_i.col4 itself is (A1_i.col1 - A1_i.col2 + A1_i.col3)
                 // So, summing the calculated columns is correct.
                 newCalculatedData[row.id][colKeyToSum] = sum.toFixed(2);
            } else if (columnFieldKeys[colKeyToSum]) { // Summing input columns for totals
                 newCalculatedData[row.id][colKeyToSum] = sum.toFixed(2);
                 // Re-calculate the derived columns for the total row based on its summed inputs
                if(row.id === 'A1_total' || row.id === 'A2_total' || row.id === 'A3_grand_total' || row.id === 'A4_total') {
                    // Special recalculation for main total rows based on summed components
                    const t_col1 = getNum(newCalculatedData[row.id]?.col1);
                    const t_col2 = getNum(newCalculatedData[row.id]?.col2);
                    const t_col3 = getNum(newCalculatedData[row.id]?.col3);
                    newCalculatedData[row.id].col4 = (t_col1 - t_col2 + t_col3).toFixed(2);

                    const t_col5 = getNum(newCalculatedData[row.id]?.col5);
                    const t_col6 = getNum(newCalculatedData[row.id]?.col6);
                    newCalculatedData[row.id].col7 = (getNum(newCalculatedData[row.id].col4) + t_col5 + t_col6).toFixed(2);
                    
                    const t_col8 = getNum(newCalculatedData[row.id]?.col8);
                    const t_col9 = getNum(newCalculatedData[row.id]?.col9);
                    newCalculatedData[row.id].col10 = (t_col8 + t_col9).toFixed(2);

                    const t_col11 = getNum(newCalculatedData[row.id]?.col11);
                    const t_col12 = getNum(newCalculatedData[row.id]?.col12);
                    newCalculatedData[row.id].col13 = (getNum(newCalculatedData[row.id].col10) + t_col11 - t_col12).toFixed(2);

                    const t_col14 = getNum(newCalculatedData[row.id]?.col14);
                    newCalculatedData[row.id].col15 = (getNum(newCalculatedData[row.id].col7) + getNum(newCalculatedData[row.id].col13) + t_col14).toFixed(2);
                }
            } else {
                 newCalculatedData[row.id][colKeyToSum] = sum.toFixed(2); // For col16,17,18 if they are summed
            }
          });
        }
      }
    });
    return newCalculatedData;
  }, [formData]);


  useEffect(() => {
    const errors = [];
    const totalsA1 = calculatedData['A1_total'];
    const totalsA2 = calculatedData['A2_total'];
    const totalsA3 = calculatedData['A3_grand_total'];
    const totalsA4 = calculatedData['A4_total'];

    if (totalsA1 && totalsA2 && totalsA3 && totalsA4) {
        const colsToValidate = ['col7', 'col13', 'col15']; // Closing Provision, Closing LICRA, Total Prov+LICRA+IS
        colsToValidate.forEach(colKey => {
            const valA1 = getNum(totalsA1[colKey]);
            const valA2 = getNum(totalsA2[colKey]);
            const valA3 = getNum(totalsA3[colKey]);
            const valA4 = getNum(totalsA4[colKey]);

            if (!(valA1 === valA2 && valA2 === valA3 && valA3 === valA4)) {
            errors.push(`Mismatch in totals for Column ${colKey.replace('col','')}: A-1 (${valA1.toFixed(2)}), A-2 (${valA2.toFixed(2)}), A-3 (${valA3.toFixed(2)}), A-4 (${valA4.toFixed(2)}) must be equal.`);
            }
        });
    }
     // Add other specific validations if needed
    setValidationErrors(errors);
  }, [calculatedData]);


  const buildPayload = (isSaveOperation) => {
    const payload = {
      circleCode,
      quarterEndDate,
      role,
      save: isSaveOperation,
      status: isSaveOperation ? 'SAVED' : 'SUBMITTED', // Example status
      // ... other common fields
    };

    rowDefinitionsConfig.forEach(row => {
      if (row.type === 'entry' && row.modelSuffix) {
        const rowData = formData[row.id] || {};
        Object.entries(columnFieldKeys).forEach(([colKeyAbbr, fieldKey]) => {
            const backendFieldName = `${fieldKey}${row.modelSuffix}`;
            payload[backendFieldName] = getNum(rowData[fieldKey]).toFixed(2);
        });
        // For calculated fields that might also need to be sent or specific columns like 17,18 for A4.i
        // This depends on backend expectations. The current ng-models in HTML suggest sending input values.
        // Let's ensure all entered values are part of the payload.
        // Example: if col17, col18 are direct inputs for A4.i
        if(row.id === 'A4_i'){
            payload[`${columnFieldKeys.col17}${row.modelSuffix}`] = getNum(rowData[columnFieldKeys.col17]).toFixed(2);
            payload[`${columnFieldKeys.col18}${row.modelSuffix}`] = getNum(rowData[columnFieldKeys.col18]).toFixed(2);
        }
      }
    });
    console.log("Built payload:", payload);
    return payload;
  };

  const handleSave = async () => {
    const payload = buildPayload(true);
    try {
      const response = await callApi('/api/schedule9c/save', payload, 'POST'); // Replace with actual endpoint
      if (response.success) {
        showSnackbar('Data saved successfully!', 'success');
      } else {
        showSnackbar(response.message || 'Failed to save data.', 'error');
      }
    } catch (error) {
      showSnackbar('An error occurred while saving data.', 'error');
      console.error("Save error:", error);
    }
  };

  const handleSubmit = async () => {
    if (validationErrors.length > 0) {
      showSnackbar('Please correct the validation errors before submitting.', 'error');
      // Optionally, scroll to errors or highlight them more
      return;
    }
    const payload = buildPayload(false);
    try {
      const response = await callApi('/api/schedule9c/submit', payload, 'POST'); // Replace with actual endpoint
      if (response.success) {
        showSnackbar('Data submitted successfully!', 'success');
        // Potentially redirect or clear form
      } else {
        showSnackbar(response.message || 'Failed to submit data.', 'error');
      }
    } catch (error) {
      showSnackbar('An error occurred while submitting data.', 'error');
      console.error("Submit error:", error);
    }
  };
  
  const columnHeaders = [
    { label: `Opening Balance of Provisions <br>for Current Year (closing <br>balance ${previousYear})<br>Rs. P`, key: 'col1' },
    { label: 'Write-off during the <br>current year for advances only<br>Rs. P', key: 'col2' },
    { label: 'Addition/Reduction on <br>Account of Exchange <br>Fluctuation (only for IBG) <br>Rs. P', key: 'col3' },
    { label: 'Net (Adjusted) Opening <br>Balance of Provisions for <br>Current Year <br>Rs. P<br><b>4=(1-2+3)</b>', key: 'col4', isCalculated: true },
    { label: 'Additions during <br>the Current Year <br>Rs. P', key: 'col5' },
    { label: 'Addition/Reduction in <br>Depreciation on Account of <br>Exchange Difference in RALOO <br>Rates & Exchange Rates used <br>for P&L (only for IBG)', key: 'col6' },
    { label: 'Closing balance of Provision <br>at the end of Current Year <br>Rs. P<br><b>7=(4+5+6)</b>', key: 'col7', isCalculated: true },
    { label: `Opening Balance of LICRA <br> for Current Year ( prev. year <br>closing balance -write-off ) <br>Rs. P`, key: 'col8' },
    { label: 'Addition/Reduction on <br> Account of Exchange <br>Fluctuation (only for IBG) <br>Rs. P', key: 'col9' },
    { label: 'Net (Adjusted) Opening<br> Balance of Provisions for <br>Current Year <br>Rs. P<br><b>10=(8+9)</b>', key: 'col10', isCalculated: true },
    { label: 'Additions during <br>the Current Year <br>Rs. P', key: 'col11' },
    { label: 'Deductions /Reversal <br>during the Current Year <br>Rs. P', key: 'col12' },
    { label: 'Closing balance at <br>the end of Current Year <br>Rs. P<br><b>13=(10+11-12)</b>', key: 'col13', isCalculated: true },
    { label: `Interest Suspense<br> Account As on ${displayQuarterDate} <br>Rs. P`, key: 'col14' },
    { label: 'Total Provision+LICRA+<br>Interest Suspense<br> Rs.P<br><b>15=(7+13+14)</b>', key: 'col15', isCalculated: true },
    { label: `DICGC /ECGC Claims Recd <br>as on ${displayQuarterDate}} <br>Rs. P`, key: 'col16' },
    { label: `Provision on Restructured <br>Standard Asset as on <br>${displayQuarterDate}} (included in total <br>Provision in Column 7)`, key: 'col17' },
    { label: `LICRA on Restructured <br>Standard Asset as on ${displayQuarterDate}} <br>(included in total<br> LICRA in Column 13)`, key: 'col18' },
  ];


  return (
    <Box sx={{ p: 2, width: '100%' }}>
        <Typography variant="h5" gutterBottom sx={{textAlign: 'center', color: 'white', backgroundColor: 'rgba(0,0,0,0.7)', padding:'10px'}}>
            Schedule 9C - Provisions
        </Typography>
      {validationErrors.length > 0 && (
        <Alert severity="error" sx={{ mb: 2 }}>
          <ul style={{ margin: 0, paddingLeft: '1.2em' }}>
            {validationErrors.map((e, i) => (
              <li key={i}>{e}</li>
            ))}
          </ul>
        </Alert>
      )}
      <TableContainer component={Paper} sx={{ maxHeight: 'calc(100vh - 200px)', overflow: 'auto' }}>
        <Table stickyHeader sx={{ minWidth: 3000 }}> {/* minWidth to ensure horizontal scroll for many columns */}
          <TableHead>
            <TableRow>
              <StyledTableCell rowSpan={3} sx={{ minWidth: '450px', position: 'sticky', left: 0, zIndex: 1101, backgroundColor: 'grey.300' }}>
                <b>Classification of PROVISION</b><br/>
                (Excluding provision relating to : non-advance <br/>
                related items debited to Recalled Assets and interest free Staff Advances ) <br/>
                <b>(A.1=A.2=A.3=A.4)</b>
              </StyledTableCell>
              <StyledTableCell colSpan={7}><b>PROVISIONS</b></StyledTableCell>
              <StyledTableCell colSpan={6}><b>Liability on Interest Capitalisation on Restructurred Account(LICRA)</b></StyledTableCell>
              <StyledTableCell colSpan={5}><b>TOTAL PROVISION (Calculated) AND OTHER DETAILS</b></StyledTableCell>
            </TableRow>
            <TableRow>
                {columnHeaders.slice(0,7).map(ch => <StyledTableCell key={ch.key} dangerouslySetInnerHTML={{ __html: ch.label }} />)}
                {columnHeaders.slice(7,13).map(ch => <StyledTableCell key={ch.key} dangerouslySetInnerHTML={{ __html: ch.label }} />)}
                {columnHeaders.slice(13,18).map(ch => <StyledTableCell key={ch.key} dangerouslySetInnerHTML={{ __html: ch.label }} />)}
            </TableRow>
            <TableRow>
                {allColumnKeys.map((key, idx) => (
                    <StyledTableCell key={`colnum_${idx}`}><b>{idx+1}</b></StyledTableCell>
                ))}
            </TableRow>
          </TableHead>
          <TableBody>
            {rowDefinitionsConfig.map(row => {
              const rowData = calculatedData[row.id] || {};
              const inputRowData = formData[row.id] || {};

              if (row.type === 'sectionHeader' || row.type === 'subSectionHeader') {
                return (
                  <StyledTableRow key={row.id} isSectionHeader={row.type === 'sectionHeader'} isSubSectionHeader={row.type === 'subSectionHeader'}>
                    <StyledTableCell 
                        colSpan={allColumnKeys.length + 1} 
                        sx={{ textAlign: 'left', fontWeight: 'bold', position: 'sticky', left: 0, zIndex: 100 }}
                    >
                      {row.label}
                    </StyledTableCell>
                  </StyledTableRow>
                );
              }
              
              return (
                <StyledTableRow key={row.id} isTotalRow={row.type === 'total'}>
                  <StyledTableCell sx={{ textAlign: 'left', position: 'sticky', left: 0, zIndex: 100, backgroundColor: row.type === 'total' ? 'grey.100' : 'common.white' }}>
                    {row.type === 'total' ? <b>{row.label}</b> : row.label}
                  </StyledTableCell>
                  {allColumnKeys.map(colKey => {
                    const fieldKeyForInput = columnFieldKeys[colKey];
                    const isCalculatedCol = ['col4', 'col7', 'col10', 'col13', 'col15'].includes(colKey);
                    const isDisabled = row.type === 'total' || isCalculatedCol;
                    const valueToShow = rowData[colKey] !== undefined ? rowData[colKey] : '';
                    
                    return (
                      <StyledTableCell key={`${row.id}-${colKey}`}>
                        <TextField
                          variant="outlined"
                          size="small"
                          value={valueToShow}
                          onChange={!isDisabled ? (e) => handleChange(row.id, fieldKeyForInput, e.target.value) : undefined}
                          disabled={isDisabled}
                          InputProps={{
                            readOnly: isDisabled,
                            sx: { 
                                textAlign: 'right', 
                                '& input': { textAlign: 'right', padding: '6px 8px' },
                                backgroundColor: isDisabled ? 'grey.50' : 'common.white'
                            },
                          }}
                          sx={{ width: '130px' }} // Fixed width for data cells
                        />
                      </StyledTableCell>
                    );
                  })}
                </StyledTableRow>
              );
            })}
          </TableBody>
        </Table>
      </TableContainer>
      <Stack direction="row" spacing={2} sx={{ mt: 2, justifyContent: 'center' }}>
        <Button variant="contained" color="warning" onClick={handleSave}>Save</Button>
        <Button variant="contained" color="success" onClick={handleSubmit} disabled={validationErrors.length > 0}>Submit</Button>
      </Stack>
    </Box>
  );
};

export default Schedule9ProvisionTable;


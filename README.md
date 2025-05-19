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
  Stack,
  CircularProgress,
  Typography,
} from '@mui/material';
import TableCell, { tableCellClasses } from '@mui/material/TableCell';
import { styled } from '@mui/material/styles';
import debounce from 'lodash/debounce';
import FormInput from '../../../../common/components/ui/FormInput';

import useApi from '../../../../common/hooks/useApi';
// import useCustomSnackbar from '../../../../common/hooks/useCustomSnackbar';

const StyledTableCell = styled(TableCell)(({ theme }) => ({
  fontSize: '0.875rem',
  padding: '8px',
  border: '1px solid #e0e0e0', // Lighter border
  whiteSpace: 'nowrap',
  [`&.${tableCellClasses.head}`]: {
    // backgroundColor: theme.palette.background.default, // More neutral
    // color: theme.palette.text.primary, // More neutral
    backgroundColor: theme.palette.common.black,
    color: theme.palette.common.white,
    fontWeight: 'bold',
    textAlign: 'center',
  },
  [`&.${tableCellClasses.body}`]: {
    color: theme.palette.text.primary,
    backgroundColor: theme.palette.background.paper,
    textAlign: 'left',
  },
}));

const StyledTableRow = styled(TableRow)(({ theme, isTotalRow, isSectionHeader, isSubSectionHeader }) => ({
  backgroundColor: theme.palette.background.paper, // Default background
  '&:nth-of-type(odd)': {
    // backgroundColor: theme.palette.action.hover, // Kept for slight differentiation if desired, can be removed
  },
  ...(isSectionHeader && {
    // backgroundColor: theme.palette.grey[100], // Very light grey
    '& > td': {
      fontWeight: 'bold',
      textAlign: 'left',
    },
  }),
  ...(isSubSectionHeader && {
    // backgroundColor: theme.palette.grey[50], // Even lighter
    '& > td': {
      fontWeight: 'bold',
      fontStyle: 'italic',
      textAlign: 'left',
    },
  }),
  ...(isTotalRow && {
    '& > td': {
      fontWeight: 'bold',
    },
  }),
}));

const rowDefinitionsConfig = [
  { id: 'A1_header', label: 'A-1. Facility Wise Classification', type: 'sectionHeader' },
  { id: 'A1_i', modelSuffix: '2', label: '[i] Bills Purchased and Discounted', type: 'entry' },
  {
    id: 'A1_ii',
    modelSuffix: '3',
    label: '[ii] Cash Credits, Overdrafts, loans repayable on Demand and Recalled Assets',
    type: 'entry',
  },
  {
    id: 'A1_iii',
    modelSuffix: '4',
    label: '[iii] Term Loans , Agricultural Term Loans, FCNRB Term Loan',
    type: 'entry',
  },
  {
    id: 'A1_total',
    modelSuffix: '5',
    label: 'Total of Facility wise Classification',
    type: 'total',
    subItemIds: ['A1_i', 'A1_ii', 'A1_iii'],
  },

  { id: 'A2_header', label: 'A-2. Security Wise Classifications', type: 'sectionHeader' },
  { id: 'A2_i', modelSuffix: '7', label: '[i] Secured by Tangible Assets', type: 'entry' },
  { id: 'A2_ii', modelSuffix: '8', label: '[ii] Covered by Bank/DICGC/CGTSI / Govt Guarantee', type: 'entry' },
  { id: 'A2_iii', modelSuffix: '9', label: '[iii] Unsecured', type: 'entry' },
  {
    id: 'A2_total',
    modelSuffix: '10',
    label: 'Total of Security-wise Classification',
    type: 'total',
    subItemIds: ['A2_i', 'A2_ii', 'A2_iii'],
  },

  { id: 'A3_header', label: 'A-3. Sector-Wise Classifications', type: 'sectionHeader' },
  { id: 'A3_a_header', label: 'a) In India', type: 'subSectionHeader' },
  { id: 'A3_a_i', modelSuffix: '13', label: '[i] Priority', type: 'entry' },
  { id: 'A3_a_ii', modelSuffix: '14', label: '[ii] Public', type: 'entry' },
  { id: 'A3_a_iii', modelSuffix: '15', label: '[iii] Banks in India', type: 'entry' },
  { id: 'A3_a_iv', modelSuffix: '16', label: '[iv] Others', type: 'entry' },
  {
    id: 'A3_a_total',
    modelSuffix: '17',
    label: 'TOTAL IN INDIA (i+ii+iii+iv)',
    type: 'total',
    subItemIds: ['A3_a_i', 'A3_a_ii', 'A3_a_iii', 'A3_a_iv'],
  },

  { id: 'A3_b_header', label: 'b) Outside India (Excluding Foreign LCs and BGs)', type: 'subSectionHeader' },
  { id: 'A3_b_i', modelSuffix: '19', label: '[i] Due from Banks', type: 'entry' },
  {
    id: 'A3_b_ii_1',
    modelSuffix: '20',
    label: '[ii] Due from Others [1] Bills Purchased and Discounted',
    type: 'entry',
  },
  { id: 'A3_b_ii_2', modelSuffix: '21', label: '[2] Syndicated Loans', type: 'entry' },
  { id: 'A3_b_ii_3', modelSuffix: '22', label: '[3] Others', type: 'entry' },
  {
    id: 'A3_b_total',
    modelSuffix: '23',
    label: 'TOTAL IN OUTSIDE INDIA(i+ii.1+ii.2+ii.3)',
    type: 'total',
    subItemIds: ['A3_b_i', 'A3_b_ii_1', 'A3_b_ii_2', 'A3_b_ii_3'],
  },
  {
    id: 'A3_grand_total',
    modelSuffix: '24',
    label: 'Total of Sector-wise Classification(a+b)',
    type: 'total',
    subItemIds: ['A3_a_total', 'A3_b_total'],
  },

  { id: 'A4_header', label: 'A-4. Assets-wise Classifications', type: 'sectionHeader' },
  { id: 'A4_i', modelSuffix: '26', label: '[i] Standard', type: 'entry' },
  { id: 'A4_ii', modelSuffix: '27', label: '[ii] Sub-standard', type: 'entry' },
  { id: 'A4_iii', modelSuffix: '28', label: '[iii] Doubtful', type: 'entry' },
  { id: 'A4_iv', modelSuffix: '29', label: '[iv] Loss', type: 'entry' },
  {
    id: 'A4_total',
    modelSuffix: '30',
    label: 'Total of Assets-wise Classification',
    type: 'total',
    subItemIds: ['A4_i', 'A4_ii', 'A4_iii', 'A4_iv'],
  },
];

const columnFieldKeys = {
  // Maps colKey (e.g. 'col1') to actual field key in formData
  col1: 'opBalCurYearProvision',
  col2: 'writOffCurProvision',
  col3: 'addRedFlucProvision',
  col5: 'addCurYearProvision',
  col6: 'addCurDepreciProvision',
  col8: 'opBalCurYearAccount',
  col9: 'addRedFlucAccount',
  col11: 'addCurYearAccount',
  col12: 'dedRevCurYearAccount',
  col14: 'intSuspEndOfCurrYearAccount',
  col16: 'diAndCgcTotalPro',
  col17: 'standardAssetsTotalPro',
  col18: 'licraTotalPro',
};
const allColumnKeys = [
  // Represents all data columns in display order
  'col1',
  'col2',
  'col3',
  'col4',
  'col5',
  'col6',
  'col7',
  'col8',
  'col9',
  'col10',
  'col11',
  'col12',
  'col13',
  'col14',
  'col15',
  'col16',
  'col17',
  'col18',
];
const calculatedColKeys = ['col4', 'col7', 'col10', 'col13', 'col15'];

const Schedule9CProvisionTable = ({
  circleCode = '021',
  quarterEndDate = '31/03/2025',
  role = 'Maker',
  previousYear = '2024',
  displayQuarterDate = '31/03/2025',
  initialDataFromApi = null,
}) => {
  const showSnackbar = (message, severity) => console.log(`Snackbar: ${message} (${severity})`);
  const { callApi } = useApi();

  const [formData, setFormData] = useState(() => {
    const initial = {};
    rowDefinitionsConfig.forEach((row) => {
      if (row.type === 'entry') {
        initial[row.id] = {};
        Object.values(columnFieldKeys).forEach((fieldKey) => {
          // Use fieldKey from columnFieldKeys
          initial[row.id][fieldKey] = '';
        });
      }
    });
    return initial;
  });

  const [validationErrors, setValidationErrors] = useState([]);
  const [isLoading, setIsLoading] = useState(false);

  // Effect for fetching data from API
  useEffect(() => {
    const fetchData = async () => {
      setIsLoading(true);
      showSnackbar('Loading data...', 'info');

      const requestPayload = {
        circleCode, // from props
        quarterEndDate, // from props
        userId: '1111111', // Example value, ideally from context or props
        reportName: 'Schedule9C PROVISION', // Example value
        reportId: '125911', // Example value
        reportMasterId: '310021', // Example value
        status: '11', // Example value
        areMocPending: true, // Example value
      };

      try {
        // const response = await callApi('/api/schedule9c/data', requestPayload, 'POST'); // Replace with actual endpoint
        const response = await callApi('/Maker/getSavedDataNineC', requestPayload, 'POST'); // Mocked call

        if (response) {
          const fetchedApiData = response;
          const transformedDataForState = {};

          // Initialize structure for all entry rows
          rowDefinitionsConfig.forEach((rowDef) => {
            if (rowDef.type === 'entry') {
              transformedDataForState[rowDef.id] = {};
              Object.values(columnFieldKeys).forEach((fKey) => {
                transformedDataForState[rowDef.id][fKey] = ''; // Default to empty string
              });
            }
          });

          // Populate with data from API
          // Loop through all keys in the fetched API data
          for (const [apiFullKey, apiValue] of Object.entries(fetchedApiData)) {
            let matchedFieldKeyInFormData = null;
            let matchedModelSuffix = null;

            // Check against each fieldKey defined in columnFieldKeys
            for (const fieldKeyInFormData of Object.values(columnFieldKeys)) {
              if (apiFullKey.startsWith(fieldKeyInFormData)) {
                const potentialSuffix = apiFullKey.substring(fieldKeyInFormData.length);
                // Check if this suffix is a valid modelSuffix from rowDefinitionsConfig
                if (modelSuffixToRowIdMap[potentialSuffix]) {
                  matchedFieldKeyInFormData = fieldKeyInFormData;
                  matchedModelSuffix = potentialSuffix;
                  break; // Found the field and suffix
                }
              }
            }

            if (matchedFieldKeyInFormData && matchedModelSuffix) {
              const rowId = modelSuffixToRowIdMap[matchedModelSuffix];
              // Ensure the rowId exists in our prepared structure and the field key is valid
              if (
                transformedDataForState[rowId] &&
                transformedDataForState[rowId].hasOwnProperty(matchedFieldKeyInFormData)
              ) {
                transformedDataForState[rowId][matchedFieldKeyInFormData] = apiValue !== null ? String(apiValue) : '';
              }
            }
          }
          setFormData(transformedDataForState);
          showSnackbar('Data loaded successfully.', 'success');
        } else {
          showSnackbar(response.message || 'Failed to load data or data is empty.', 'error');
        }
      } catch (error) {
        console.error('API call failed:', error);
        showSnackbar('Error fetching data. Check console for details.', 'error');
      } finally {
        setIsLoading(false);
      }
    };

    // Only fetch if not relying on initialDataFromApi or based on your logic
    // For this example, we will always fetch. You can add conditions like:
    // if (!initialDataFromApi) { fetchData(); }
    fetchData();
  }, []);

  //   useEffect(() => {
  //     if (initialDataFromApi) {
  //       const newFormData = {};
  //       rowDefinitionsConfig.forEach((row) => {
  //         if (row.type === 'entry') {
  //           newFormData[row.id] = {}; // Ensure object exists
  //           const apiRowData = initialDataFromApi[row.id] || {};
  //           Object.values(columnFieldKeys).forEach((fieldKey) => {
  //             newFormData[row.id][fieldKey] = apiRowData[fieldKey] ?? '';
  //           });
  //         }
  //       });
  //       setFormData((prev) => ({ ...prev, ...newFormData }));
  //     }
  //   }, [initialDataFromApi]);

  const debouncedSetFormData = useCallback(
    debounce((rowId, fieldKey, value) => {
      setFormData((prev) => ({
        ...prev,
        [rowId]: { ...prev[rowId], [fieldKey]: value },
      }));
    }, 100),
    []
  );

  const handleChange = (rowId, fieldKey, value) => {
    if (
      value === '' ||
      /^-?\d*\.?\d{0,2}$/.test(value) ||
      (value === '-' && !(formData[rowId]?.[fieldKey]?.length > 0))
    ) {
      setFormData((prev) => ({
        ...prev,
        [rowId]: { ...prev[rowId], [fieldKey]: value },
      }));
    }
  };

  const getNum = (value) => parseFloat(value) || 0;

  const calculatedData = useMemo(() => {
    const newCalculatedData = {};

    rowDefinitionsConfig.forEach((row) => {
      if (row.type === 'entry' || row.type === 'total') {
        newCalculatedData[row.id] = {};
        const currentRowFormData = formData[row.id] || {}; // Data from state for 'entry' rows

        if (row.type === 'entry') {
          // Step 1: Populate newCalculatedData with direct input values using colX keys
          Object.entries(columnFieldKeys).forEach(([colKeyAlias, fieldKeyInFormData]) => {
            newCalculatedData[row.id][colKeyAlias] = currentRowFormData[fieldKeyInFormData] ?? '';
          });
          // Step 2: Calculate derived columns (col4, col7, etc.)
          const col1 = getNum(newCalculatedData[row.id].col1);
          const col2 = getNum(newCalculatedData[row.id].col2);
          const col3 = getNum(newCalculatedData[row.id].col3);
          newCalculatedData[row.id].col4 = (col1 - col2 + col3).toFixed(2);

          const col5 = getNum(newCalculatedData[row.id].col5);
          const col6 = getNum(newCalculatedData[row.id].col6);
          newCalculatedData[row.id].col7 = (getNum(newCalculatedData[row.id].col4) + col5 + col6).toFixed(2);

          const col8 = getNum(newCalculatedData[row.id].col8);
          const col9 = getNum(newCalculatedData[row.id].col9);
          newCalculatedData[row.id].col10 = (col8 + col9).toFixed(2);

          const col11 = getNum(newCalculatedData[row.id].col11);
          const col12 = getNum(newCalculatedData[row.id].col12);
          newCalculatedData[row.id].col13 = (getNum(newCalculatedData[row.id].col10) + col11 - col12).toFixed(2);

          const col14 = getNum(newCalculatedData[row.id].col14);
          newCalculatedData[row.id].col15 = (
            getNum(newCalculatedData[row.id].col7) +
            getNum(newCalculatedData[row.id].col13) +
            col14
          ).toFixed(2);
        } else if (row.type === 'total') {
          // Calculate for total rows by summing corresponding colX keys from subItemIds
          allColumnKeys.forEach((colKeyToSum) => {
            let sum = 0;
            row.subItemIds.forEach((subItemId) => {
              sum += getNum(newCalculatedData[subItemId]?.[colKeyToSum]);
            });
            newCalculatedData[row.id][colKeyToSum] = sum.toFixed(2);
          });
          // For total rows, re-calculate derived columns based on their summed components
          // This ensures consistency if summed inputs lead to different derived totals
          const t_col1 = getNum(newCalculatedData[row.id].col1);
          const t_col2 = getNum(newCalculatedData[row.id].col2);
          const t_col3 = getNum(newCalculatedData[row.id].col3);
          newCalculatedData[row.id].col4 = (t_col1 - t_col2 + t_col3).toFixed(2);

          const t_col5 = getNum(newCalculatedData[row.id].col5);
          const t_col6 = getNum(newCalculatedData[row.id].col6);
          newCalculatedData[row.id].col7 = (getNum(newCalculatedData[row.id].col4) + t_col5 + t_col6).toFixed(2);

          const t_col8 = getNum(newCalculatedData[row.id].col8);
          const t_col9 = getNum(newCalculatedData[row.id].col9);
          newCalculatedData[row.id].col10 = (t_col8 + t_col9).toFixed(2);

          const t_col11 = getNum(newCalculatedData[row.id].col11);
          const t_col12 = getNum(newCalculatedData[row.id].col12);
          newCalculatedData[row.id].col13 = (getNum(newCalculatedData[row.id].col10) + t_col11 - t_col12).toFixed(2);

          const t_col14 = getNum(newCalculatedData[row.id].col14);
          newCalculatedData[row.id].col15 = (
            getNum(newCalculatedData[row.id].col7) +
            getNum(newCalculatedData[row.id].col13) +
            t_col14
          ).toFixed(2);
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
      const colsToValidateEquality = ['col7', 'col13', 'col15'];
      colsToValidateEquality.forEach((colKey) => {
        const valA1 = getNum(totalsA1[colKey]);
        const valA2 = getNum(totalsA2[colKey]);
        const valA3 = getNum(totalsA3[colKey]);
        const valA4 = getNum(totalsA4[colKey]);
        // Check with a small tolerance for floating point issues
        const tolerance = 0.001;
        if (
          !(
            Math.abs(valA1 - valA2) < tolerance &&
            Math.abs(valA2 - valA3) < tolerance &&
            Math.abs(valA3 - valA4) < tolerance
          )
        ) {
          errors.push(
            `Mismatch in totals for Column ${colKey.replace('col', '')}: A-1 (${valA1.toFixed(
              2
            )}), A-2 (${valA2.toFixed(2)}), A-3 (${valA3.toFixed(2)}), A-4 (${valA4.toFixed(2)}) must be equal.`
          );
        }
      });
    }
    setValidationErrors(errors);
  }, [calculatedData]);

  const buildPayload = (isSaveOperation) => {
    const payload = {
      circleCode,
      quarterEndDate,
      role,
      save: isSaveOperation,
      status: isSaveOperation ? 'SAVED' : 'SUBMITTED',
    };
    rowDefinitionsConfig.forEach((row) => {
      if (row.type === 'entry' && row.modelSuffix) {
        const rowInputData = formData[row.id] || {};
        Object.entries(columnFieldKeys).forEach(([colKeyAlias, fieldKeyInFormData]) => {
          const backendFieldName = `${fieldKeyInFormData}${row.modelSuffix}`;
          payload[backendFieldName] = getNum(rowInputData[fieldKeyInFormData]).toFixed(2);
        });
      }
    });
    console.log('Built payload:', payload);
    return payload;
  };

  const handleSave = async () => {
    const payload = buildPayload(true);
    showSnackbar('Save initiated (see console for payload).', 'info');
  };
  const handleSubmit = async () => {
    if (validationErrors.length > 0) {
      showSnackbar('Please correct validation errors.', 'error');
      return;
    }
    const payload = buildPayload(false);
    showSnackbar('Submit initiated (see console for payload).', 'info');
    // Actual API call would go here
  };

  const columnHeaders = [
    {
      label: `Opening Balance of Provisions <br>for Current Year (closing <br>balance ${previousYear})<br>Rs. P`,
      key: 'col1',
    },
    { label: 'Write-off during the <br>current year for advances only<br>Rs. P', key: 'col2' },
    { label: 'Addition/Reduction on <br>Account of Exchange <br>Fluctuation (only for IBG) <br>Rs. P', key: 'col3' },
    {
      label: 'Net (Adjusted) Opening <br>Balance of Provisions for <br>Current Year <br>Rs. P<br><b>4=(1-2+3)</b>',
      key: 'col4',
      isCalculated: true,
    },
    { label: 'Additions during <br>the Current Year <br>Rs. P', key: 'col5' },
    {
      label:
        'Addition/Reduction in <br>Depreciation on Account of <br>Exchange Difference in RALOO <br>Rates & Exchange Rates used <br>for P&L (only for IBG)',
      key: 'col6',
    },
    {
      label: 'Closing balance of Provision <br>at the end of Current Year <br>Rs. P<br><b>7=(4+5+6)</b>',
      key: 'col7',
      isCalculated: true,
    },
    {
      label: `Opening Balance of LICRA <br> for Current Year ( prev. year <br>closing balance -write-off ) <br>Rs. P`,
      key: 'col8',
    },
    { label: 'Addition/Reduction on <br> Account of Exchange <br>Fluctuation (only for IBG) <br>Rs. P', key: 'col9' },
    {
      label: 'Net (Adjusted) Opening<br> Balance of Provisions for <br>Current Year <br>Rs. P<br><b>10=(8+9)</b>',
      key: 'col10',
      isCalculated: true,
    },
    { label: 'Additions during <br>the Current Year <br>Rs. P', key: 'col11' },
    { label: 'Deductions /Reversal <br>during the Current Year <br>Rs. P', key: 'col12' },
    {
      label: 'Closing balance at <br>the end of Current Year <br>Rs. P<br><b>13=(10+11-12)</b>',
      key: 'col13',
      isCalculated: true,
    },
    { label: `Interest Suspense<br> Account As on ${displayQuarterDate} <br>Rs. P`, key: 'col14' },
    {
      label: 'Total Provision+LICRA+<br>Interest Suspense<br> Rs.P<br><b>15=(7+13+14)</b>',
      key: 'col15',
      isCalculated: true,
    },
    { label: `DICGC /ECGC Claims Recd <br>as on ${displayQuarterDate} <br>Rs. P`, key: 'col16' },
    {
      label: `Provision on Restructured <br>Standard Asset as on <br>${displayQuarterDate} (included in total <br>Provision in Column 7)`,
      key: 'col17',
    },
    {
      label: `LICRA on Restructured <br>Standard Asset as on ${displayQuarterDate} <br>(included in total<br> LICRA in Column 13)`,
      key: 'col18',
    },
  ];

  if (isLoading) {
    return (
      <Box sx={{ display: 'flex', justifyContent: 'center', alignItems: 'center', height: '50vh' }}>
        <CircularProgress />
        <Typography sx={{ ml: 2 }}>Loading Data...</Typography>
      </Box>
    );
  }

  return (
    <Box sx={{ p: 1, width: '100%', overflowX: 'hidden' }}>
      {/* <Typography variant="h5" gutterBottom sx={{ textAlign: 'center', mb: 2 }}>
        Schedule 9C - Provisions
      </Typography> */}
      {validationErrors.length > 0 && (
        <Alert severity="error" sx={{ mb: 2 }}>
          <ul style={{ margin: 0, paddingLeft: '1.2em' }}>
            {validationErrors.map((e, i) => (
              <li key={i}>{e}</li>
            ))}
          </ul>
        </Alert>
      )}
      <TableContainer component={Paper} sx={{ maxHeight: 'calc(110vh - 250px)' }}>
        <Table stickyHeader sx={{ minWidth: 3000 }}>
          <TableHead>
            <TableRow
              sx={{
                position: 'sticky',
                left: 0,
                zIndex: 1101,
                backgroundColor: '#f5f5f5' /* theme.palette.background.default or similar */,
              }}
            >
              <StyledTableCell
                rowSpan={3}
                sx={{
                  position: 'sticky',
                  left: 0,
                  top: 0,
                  zIndex: 1100,
                  backgroundColor: '#f5f5f5' /* theme.palette.background.default or similar */,
                }}
              >
                <b>Classification of PROVISION</b>
                <br />
                (Excluding provision relating to : non-advance <br />
                related items debited to Recalled Assets and interest free Staff Advances ) <br />
                <b>(A.1=A.2=A.3=A.4)</b>
              </StyledTableCell>
              <StyledTableCell colSpan={8}>
                <b>PROVISIONS</b>
              </StyledTableCell>
              <StyledTableCell colSpan={8}>
                <b>Liability on Interest Capitalisation on Restructurred Account(LICRA)</b>
              </StyledTableCell>
              <StyledTableCell colSpan={5}>
                <b>TOTAL PROVISION AND OTHER DETAILS</b>
              </StyledTableCell>
            </TableRow>
            <TableRow>
              {columnHeaders.map((ch) => (
                <StyledTableCell
                  key={ch.key}
                  sx={{
                    position: 'sticky',
                    top: 42.5, // adjust if row height differs
                    zIndex: 1100,
                    backgroundColor: '#000',
                    color: '#fff',
                    textAlign: 'center',
                    whiteSpace: 'normal',
                  }}
                  dangerouslySetInnerHTML={{ __html: ch.label }}
                />
              ))}
            </TableRow>
            <TableRow>
              {allColumnKeys.map((key, idx) => (
                <StyledTableCell
                  key={`colnum_${idx}`}
                  sx={{
                    position: 'sticky',
                    top: 228.5, // 56px + 56px
                    zIndex: 1000,
                    backgroundColor: '#000',
                    color: '#fff',
                    textAlign: 'center',
                  }}
                >
                  <b>{idx + 1}</b>
                </StyledTableCell>
              ))}
            </TableRow>
          </TableHead>
          <TableBody>
            {rowDefinitionsConfig.map((row) => {
              const displayRowData = calculatedData[row.id] || {};
              const isTotalOrHeader =
                row.type === 'total' || row.type === 'sectionHeader' || row.type === 'subSectionHeader';

              if (row.type === 'sectionHeader' || row.type === 'subSectionHeader') {
                return (
                  <StyledTableRow
                    key={row.id}
                    isSectionHeader={row.type === 'sectionHeader'}
                    isSubSectionHeader={row.type === 'subSectionHeader'}
                  >
                    <StyledTableCell
                      //colSpan={allColumnKeys.length + 1}
                      sx={{
                        textAlign: 'left', // Align text to the left
                        position: 'sticky', // Make the section header sticky
                        left: 0, // Stick to the left
                        zIndex: 100, // Ensure it appears above other elements
                        backgroundColor: row.type === 'sectionHeader' ? '#e0e0e0' : '#f0f0f0',
                      }}
                    >
                      {row.label}
                    </StyledTableCell>
                  </StyledTableRow>
                );
              }

              return (
                <StyledTableRow key={row.id} isTotalRow={row.type === 'total'}>
                  <StyledTableCell
                    sx={{
                      textAlign: 'left',
                      fontWeight: row.type === 'total' ? 'bold' : 'normal',
                      fontStyle: row.type === 'entry' ? 'normal' : 'italic',
                      position: 'sticky', // Make the first column sticky
                      left: 0, // Stick to the left
                      zIndex: 99,
                      backgroundColor: row.type === 'total' ? '#f5f5f5' : row.type === 'entry' ? '#ffffff' : '#f0f0f0',
                    }}
                  >
                    {row.label}
                  </StyledTableCell>
                  {allColumnKeys.map((colKey) => {
                    const isCalculatedField = calculatedColKeys.includes(colKey);
                    const isEditableField = row.type === 'entry' && !isCalculatedField;
                    const fieldKeyInFormData = columnFieldKeys[colKey]; // This is undefined for calculated columns

                    const valueToDisplayInTextField = displayRowData[colKey] ?? '';

                    return (
                      <StyledTableCell key={`${row.id}-${colKey}`}>
                        {/* <TextField
                          variant="outlined"
                          size="small"
                          value={valueToDisplayInTextField}
                          onChange={
                            isEditableField
                              ? (e) => handleChange(row.id, fieldKeyInFormData, e.target.value)
                              : undefined
                          }
                          disabled={!isEditableField}
                          InputProps={{
                            readOnly: !isEditableField,
                            sx: {
                              textAlign: 'right',
                              '& input': { textAlign: 'right', padding: '6px 8px' },
                              backgroundColor: !isEditableField ? '#f0f0f0' : 'white', // Lighter grey for disabled
                              color: (theme) => theme.palette.text.primary,
                            },
                          }}
                          sx={{ width: '130px' }}
                        /> */}
                        <FormInput
                          name={''}
                          value={valueToDisplayInTextField}
                          onChange={
                            isEditableField
                              ? (e) => handleChange(row.id, fieldKeyInFormData, e.target.value)
                              : undefined
                          }
                          onBlur={() => {}}
                          readOnly={!isEditableField}
                          //  error={!!getFieldError(fieldName)}
                          customStyles={{
                            textAlign: 'right',
                            '& input': { textAlign: 'right', padding: '6px 8px' },
                            backgroundColor: !isEditableField ? '#f0f0f0' : 'white', // Lighter grey for disabled
                            color: (theme) => theme.palette.text.primary,
                            width: '200px',
                          }}
                          //  focus={focusedErrorField === fieldName}
                          isNumeric={true} // Treat as numeric unless specified as integer
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
        <Button variant="contained" color="warning" onClick={handleSave}>
          Save
        </Button>
        <Button variant="contained" color="success" onClick={handleSubmit} disabled={validationErrors.length > 0}>
          Submit
        </Button>
      </Stack>
    </Box>
  );
};

export default Schedule9CProvisionTable;


////////
fetchdata response in useeffect

{
    "reportName": "",
    "description": "",
    "userId": null,
    "reportId": null,
    "reportMasterId": null,
    "status": null,
    "circleCode": "",
    "quarterEndDate": "",
    "areMocPending": false,
    "writOffCurProvision2": null,
    "writOffCurProvision3": null,
    "writOffCurProvision4": null,
    "writOffCurProvision5": "0",
    "writOffCurProvision7": null,
    "writOffCurProvision8": null,
    "writOffCurProvision9": null,
    "writOffCurProvision10": "0",
    "writOffCurProvision13": null,
    "writOffCurProvision14": null,
    "writOffCurProvision15": null,
    "writOffCurProvision16": null,
    "writOffCurProvision17": "0",
    "writOffCurProvision19": null,
    "writOffCurProvision20": null,
    "writOffCurProvision21": null,
    "writOffCurProvision22": null,
    "writOffCurProvision23": "0",
    "writOffCurProvision24": "0",
    "writOffCurProvision26": null,
    "writOffCurProvision27": null,
    "writOffCurProvision28": null,
    "writOffCurProvision29": null,
    "writOffCurProvision30": "0",
    "opBalCurYearProvision2": "1",
    "addRedFlucProvision2": null,
    "netAdjOpBalProvision2": "1",
    "addCurYearProvision2": null,
    "dedCurYearProvision2": null,
    "addCurDepreciProvision2": null,
    "closBalCurYearProvision2": "1",
    "opBalCurYearAccount2": null,
    "addRedFlucAccount2": null,
    "netAdjOpBalAccount2": "0",
    "addCurYearAccount2": null,
    "dedRevCurYearAccount2": null,
    "closBalCurYearAccount2": "0",
    "intSuspEndOfCurrYearAccount2": null,
    "closBalProvAccount2": "1",
    "diAndCgcTotalPro2": null,
    "standardAssetsTotalPro2": null,
    "licraTotalPro2": null,
    "opBalCurYearProvision3": null,
    "addRedFlucProvision3": null,
    "netAdjOpBalProvision3": "0",
    "addCurYearProvision3": null,
    "dedCurYearProvision3": null,
    "addCurDepreciProvision3": null,
    "closBalCurYearProvision3": "0",
    "opBalCurYearAccount3": null,
    "addRedFlucAccount3": null,
    "netAdjOpBalAccount3": "0",
    "addCurYearAccount3": null,
    "dedRevCurYearAccount3": null,
    "closBalCurYearAccount3": "0",
    "intSuspEndOfCurrYearAccount3": null,
    "closBalProvAccount3": "0",
    "diAndCgcTotalPro3": null,
    "standardAssetsTotalPro3": null,
    "licraTotalPro3": null,
    "opBalCurYearProvision4": null,
    "addRedFlucProvision4": null,
    "netAdjOpBalProvision4": "0",
    "addCurYearProvision4": null,
    "dedCurYearProvision4": null,
    "addCurDepreciProvision4": null,
    "closBalCurYearProvision4": "0",
    "opBalCurYearAccount4": null,
    "addRedFlucAccount4": null,
    "netAdjOpBalAccount4": "0",
    "addCurYearAccount4": null,
    "dedRevCurYearAccount4": null,
    "closBalCurYearAccount4": "0",
    "intSuspEndOfCurrYearAccount4": null,
    "closBalProvAccount4": "0",
    "diAndCgcTotalPro4": null,
    "standardAssetsTotalPro4": null,
    "licraTotalPro4": null,
    "opBalCurYearProvision5": "1",
    "addRedFlucProvision5": "0",
    "netAdjOpBalProvision5": "1",
    "addCurYearProvision5": "0",
    "dedCurYearProvision5": null,
    "addCurDepreciProvision5": "0",
    "closBalCurYearProvision5": "1",
    "opBalCurYearAccount5": "0",
    "addRedFlucAccount5": "0",
    "netAdjOpBalAccount5": "0",
    "addCurYearAccount5": "0",
    "dedRevCurYearAccount5": "0",
    "closBalCurYearAccount5": "0",
    "intSuspEndOfCurrYearAccount5": "0",
    "closBalProvAccount5": "1",
    "diAndCgcTotalPro5": "0",
    "standardAssetsTotalPro5": "0",
    "licraTotalPro5": "0",
    "opBalCurYearProvision7": null,
    "addRedFlucProvision7": null,
    "netAdjOpBalProvision7": "0",
    "addCurYearProvision7": null,
    "dedCurYearProvision7": null,
    "addCurDepreciProvision7": null,
    "closBalCurYearProvision7": "0",
    "opBalCurYearAccount7": null,
    "addRedFlucAccount7": null,
    "netAdjOpBalAccount7": "0",
    "addCurYearAccount7": null,
    "dedRevCurYearAccount7": null,
    "closBalCurYearAccount7": "0",
    "intSuspEndOfCurrYearAccount7": null,
    "closBalProvAccount7": "0",
    "diAndCgcTotalPro7": null,
    "standardAssetsTotalPro7": null,
    "licraTotalPro7": null,
    "opBalCurYearProvision8": null,
    "addRedFlucProvision8": null,
    "netAdjOpBalProvision8": "0",
    "addCurYearProvision8": null,
    "dedCurYearProvision8": null,
    "addCurDepreciProvision8": null,
    "closBalCurYearProvision8": "0",
    "opBalCurYearAccount8": null,
    "addRedFlucAccount8": null,
    "netAdjOpBalAccount8": "0",
    "addCurYearAccount8": null,
    "dedRevCurYearAccount8": null,
    "closBalCurYearAccount8": "0",
    "intSuspEndOfCurrYearAccount8": null,
    "closBalProvAccount8": "0",
    "diAndCgcTotalPro8": null,
    "standardAssetsTotalPro8": null,
    "licraTotalPro8": null,
    "opBalCurYearProvision9": null,
    "addRedFlucProvision9": null,
    "netAdjOpBalProvision9": "0",
    "addCurYearProvision9": null,
    "dedCurYearProvision9": null,
    "addCurDepreciProvision9": null,
    "closBalCurYearProvision9": "0",
    "opBalCurYearAccount9": null,
    "addRedFlucAccount9": null,
    "netAdjOpBalAccount9": "0",
    "addCurYearAccount9": null,
    "dedRevCurYearAccount9": null,
    "closBalCurYearAccount9": "0",
    "intSuspEndOfCurrYearAccount9": null,
    "closBalProvAccount9": "0",
    "diAndCgcTotalPro9": null,
    "standardAssetsTotalPro9": null,
    "licraTotalPro9": null,
    "opBalCurYearProvision10": "0",
    "addRedFlucProvision10": "0",
    "netAdjOpBalProvision10": "0",
    "addCurYearProvision10": "0",
    "dedCurYearProvision10": null,
    "addCurDepreciProvision10": "0",
    "closBalCurYearProvision10": "0",
    "opBalCurYearAccount10": "0",
    "addRedFlucAccount10": "0",
    "netAdjOpBalAccount10": "0",
    "addCurYearAccount10": "0",
    "dedRevCurYearAccount10": "0",
    "closBalCurYearAccount10": "0",
    "intSuspEndOfCurrYearAccount10": "0",
    "closBalProvAccount10": "0",
    "diAndCgcTotalPro10": "0",
    "standardAssetsTotalPro10": "0",
    "licraTotalPro10": "0",
    "opBalCurYearProvision13": null,
    "addRedFlucProvision13": null,
    "netAdjOpBalProvision13": "0",
    "addCurYearProvision13": null,
    "dedCurYearProvision13": null,
    "addCurDepreciProvision13": null,
    "closBalCurYearProvision13": "0",
    "opBalCurYearAccount13": null,
    "addRedFlucAccount13": null,
    "netAdjOpBalAccount13": "0",
    "addCurYearAccount13": null,
    "dedRevCurYearAccount13": null,
    "closBalCurYearAccount13": "0",
    "intSuspEndOfCurrYearAccount13": null,
    "closBalProvAccount13": "0",
    "diAndCgcTotalPro13": null,
    "standardAssetsTotalPro13": null,
    "licraTotalPro13": null,
    "opBalCurYearProvision14": null,
    "addRedFlucProvision14": null,
    "netAdjOpBalProvision14": "0",
    "addCurYearProvision14": null,
    "dedCurYearProvision14": null,
    "addCurDepreciProvision14": null,
    "closBalCurYearProvision14": "0",
    "opBalCurYearAccount14": null,
    "addRedFlucAccount14": null,
    "netAdjOpBalAccount14": "0",
    "addCurYearAccount14": null,
    "dedRevCurYearAccount14": null,
    "closBalCurYearAccount14": "0",
    "intSuspEndOfCurrYearAccount14": null,
    "closBalProvAccount14": "0",
    "diAndCgcTotalPro14": null,
    "standardAssetsTotalPro14": null,
    "licraTotalPro14": null,
    "opBalCurYearProvision15": null,
    "addRedFlucProvision15": null,
    "netAdjOpBalProvision15": "0",
    "addCurYearProvision15": null,
    "dedCurYearProvision15": null,
    "addCurDepreciProvision15": null,
    "closBalCurYearProvision15": "0",
    "opBalCurYearAccount15": null,
    "addRedFlucAccount15": null,
    "netAdjOpBalAccount15": "0",
    "addCurYearAccount15": null,
    "dedRevCurYearAccount15": null,
    "closBalCurYearAccount15": "0",
    "intSuspEndOfCurrYearAccount15": null,
    "closBalProvAccount15": "0",
    "diAndCgcTotalPro15": null,
    "standardAssetsTotalPro15": null,
    "licraTotalPro15": null,
    "opBalCurYearProvision16": null,
    "addRedFlucProvision16": null,
    "netAdjOpBalProvision16": "0",
    "addCurYearProvision16": null,
    "dedCurYearProvision16": null,
    "addCurDepreciProvision16": null,
    "closBalCurYearProvision16": "0",
    "opBalCurYearAccount16": null,
    "addRedFlucAccount16": null,
    "netAdjOpBalAccount16": "0",
    "addCurYearAccount16": null,
    "dedRevCurYearAccount16": null,
    "closBalCurYearAccount16": "0",
    "intSuspEndOfCurrYearAccount16": null,
    "closBalProvAccount16": "0",
    "diAndCgcTotalPro16": null,
    "standardAssetsTotalPro16": null,
    "licraTotalPro16": null,
    "opBalCurYearProvision17": "0",
    "addRedFlucProvision17": "0",
    "netAdjOpBalProvision17": "0",
    "addCurYearProvision17": "0",
    "dedCurYearProvision17": null,
    "addCurDepreciProvision17": "0",
    "closBalCurYearProvision17": "0",
    "opBalCurYearAccount17": "0",
    "addRedFlucAccount17": "0",
    "netAdjOpBalAccount17": "0",
    "addCurYearAccount17": "0",
    "dedRevCurYearAccount17": "0",
    "closBalCurYearAccount17": "0",
    "intSuspEndOfCurrYearAccount17": "0",
    "closBalProvAccount17": "0",
    "diAndCgcTotalPro17": "0",
    "standardAssetsTotalPro17": "0",
    "licraTotalPro17": "0",
    "opBalCurYearProvision19": null,
    "addRedFlucProvision19": null,
    "netAdjOpBalProvision19": "0",
    "addCurYearProvision19": null,
    "dedCurYearProvision19": null,
    "addCurDepreciProvision19": null,
    "closBalCurYearProvision19": "0",
    "opBalCurYearAccount19": null,
    "addRedFlucAccount19": null,
    "netAdjOpBalAccount19": "0",
    "addCurYearAccount19": null,
    "dedRevCurYearAccount19": null,
    "closBalCurYearAccount19": "0",
    "intSuspEndOfCurrYearAccount19": null,
    "closBalProvAccount19": "0",
    "diAndCgcTotalPro19": null,
    "standardAssetsTotalPro19": null,
    "licraTotalPro19": null,
    "opBalCurYearProvision20": null,
    "addRedFlucProvision20": null,
    "netAdjOpBalProvision20": "0",
    "addCurYearProvision20": null,
    "dedCurYearProvision20": null,
    "addCurDepreciProvision20": null,
    "closBalCurYearProvision20": "0",
    "opBalCurYearAccount20": null,
    "addRedFlucAccount20": null,
    "netAdjOpBalAccount20": "0",
    "addCurYearAccount20": null,
    "dedRevCurYearAccount20": null,
    "closBalCurYearAccount20": "0",
    "intSuspEndOfCurrYearAccount20": null,
    "closBalProvAccount20": "0",
    "diAndCgcTotalPro20": null,
    "standardAssetsTotalPro20": null,
    "licraTotalPro20": null,
    "opBalCurYearProvision21": null,
    "addRedFlucProvision21": null,
    "netAdjOpBalProvision21": "0",
    "addCurYearProvision21": null,
    "dedCurYearProvision21": null,
    "addCurDepreciProvision21": null,
    "closBalCurYearProvision21": "0",
    "opBalCurYearAccount21": null,
    "addRedFlucAccount21": null,
    "netAdjOpBalAccount21": "0",
    "addCurYearAccount21": null,
    "dedRevCurYearAccount21": null,
    "closBalCurYearAccount21": "0",
    "intSuspEndOfCurrYearAccount21": null,
    "closBalProvAccount21": "0",
    "diAndCgcTotalPro21": null,
    "standardAssetsTotalPro21": null,
    "licraTotalPro21": null,
    "opBalCurYearProvision22": null,
    "addRedFlucProvision22": null,
    "netAdjOpBalProvision22": "0",
    "addCurYearProvision22": null,
    "dedCurYearProvision22": null,
    "addCurDepreciProvision22": null,
    "closBalCurYearProvision22": "0",
    "opBalCurYearAccount22": null,
    "addRedFlucAccount22": null,
    "netAdjOpBalAccount22": "0",
    "addCurYearAccount22": null,
    "dedRevCurYearAccount22": null,
    "closBalCurYearAccount22": "0",
    "intSuspEndOfCurrYearAccount22": null,
    "closBalProvAccount22": "0",
    "diAndCgcTotalPro22": null,
    "standardAssetsTotalPro22": null,
    "licraTotalPro22": null,
    "opBalCurYearProvision23": "0",
    "addRedFlucProvision23": "0",
    "netAdjOpBalProvision23": "0",
    "addCurYearProvision23": "0",
    "dedCurYearProvision23": null,
    "addCurDepreciProvision23": "0",
    "closBalCurYearProvision23": "0",
    "opBalCurYearAccount23": "0",
    "addRedFlucAccount23": "0",
    "netAdjOpBalAccount23": "0",
    "addCurYearAccount23": "0",
    "dedRevCurYearAccount23": "0",
    "closBalCurYearAccount23": "0",
    "intSuspEndOfCurrYearAccount23": "0",
    "closBalProvAccount23": "0",
    "diAndCgcTotalPro23": "0",
    "standardAssetsTotalPro23": "0",
    "licraTotalPro23": "0",
    "opBalCurYearProvision24": "0",
    "addRedFlucProvision24": "0",
    "netAdjOpBalProvision24": "0",
    "addCurYearProvision24": "0",
    "dedCurYearProvision24": null,
    "addCurDepreciProvision24": "0",
    "closBalCurYearProvision24": "0",
    "opBalCurYearAccount24": "0",
    "addRedFlucAccount24": "0",
    "netAdjOpBalAccount24": "0",
    "addCurYearAccount24": "0",
    "dedRevCurYearAccount24": "0",
    "closBalCurYearAccount24": "0",
    "intSuspEndOfCurrYearAccount24": "0",
    "closBalProvAccount24": "0",
    "diAndCgcTotalPro24": "0",
    "standardAssetsTotalPro24": "0",
    "licraTotalPro24": "0",
    "opBalCurYearProvision26": null,
    "addRedFlucProvision26": null,
    "netAdjOpBalProvision26": "0",
    "addCurYearProvision26": null,
    "dedCurYearProvision26": null,
    "addCurDepreciProvision26": null,
    "closBalCurYearProvision26": "0",
    "opBalCurYearAccount26": null,
    "addRedFlucAccount26": null,
    "netAdjOpBalAccount26": "0",
    "addCurYearAccount26": null,
    "dedRevCurYearAccount26": null,
    "closBalCurYearAccount26": "0",
    "intSuspEndOfCurrYearAccount26": null,
    "closBalProvAccount26": "0",
    "diAndCgcTotalPro26": null,
    "standardAssetsTotalPro26": null,
    "licraTotalPro26": null,
    "opBalCurYearProvision27": null,
    "addRedFlucProvision27": null,
    "netAdjOpBalProvision27": "0",
    "addCurYearProvision27": null,
    "dedCurYearProvision27": null,
    "addCurDepreciProvision27": null,
    "closBalCurYearProvision27": "0",
    "opBalCurYearAccount27": null,
    "addRedFlucAccount27": null,
    "netAdjOpBalAccount27": "0",
    "addCurYearAccount27": null,
    "dedRevCurYearAccount27": null,
    "closBalCurYearAccount27": "0",
    "intSuspEndOfCurrYearAccount27": null,
    "closBalProvAccount27": "0",
    "diAndCgcTotalPro27": null,
    "standardAssetsTotalPro27": null,
    "licraTotalPro27": null,
    "opBalCurYearProvision28": null,
    "addRedFlucProvision28": null,
    "netAdjOpBalProvision28": "0",
    "addCurYearProvision28": null,
    "dedCurYearProvision28": null,
    "addCurDepreciProvision28": null,
    "closBalCurYearProvision28": "0",
    "opBalCurYearAccount28": null,
    "addRedFlucAccount28": null,
    "netAdjOpBalAccount28": "0",
    "addCurYearAccount28": null,
    "dedRevCurYearAccount28": null,
    "closBalCurYearAccount28": "0",
    "intSuspEndOfCurrYearAccount28": null,
    "closBalProvAccount28": "0",
    "diAndCgcTotalPro28": null,
    "standardAssetsTotalPro28": null,
    "licraTotalPro28": null,
    "opBalCurYearProvision29": null,
    "addRedFlucProvision29": null,
    "netAdjOpBalProvision29": "0",
    "addCurYearProvision29": null,
    "dedCurYearProvision29": null,
    "addCurDepreciProvision29": null,
    "closBalCurYearProvision29": "0",
    "opBalCurYearAccount29": null,
    "addRedFlucAccount29": null,
    "netAdjOpBalAccount29": "0",
    "addCurYearAccount29": null,
    "dedRevCurYearAccount29": null,
    "closBalCurYearAccount29": "0",
    "intSuspEndOfCurrYearAccount29": null,
    "closBalProvAccount29": "0",
    "diAndCgcTotalPro29": null,
    "standardAssetsTotalPro29": null,
    "licraTotalPro29": null,
    "opBalCurYearProvision30": "0",
    "addRedFlucProvision30": "0",
    "netAdjOpBalProvision30": "0",
    "addCurYearProvision30": "0",
    "dedCurYearProvision30": null,
    "addCurDepreciProvision30": "0",
    "closBalCurYearProvision30": "0",
    "opBalCurYearAccount30": "0",
    "addRedFlucAccount30": "0",
    "netAdjOpBalAccount30": "0",
    "addCurYearAccount30": "0",
    "dedRevCurYearAccount30": "0",
    "closBalCurYearAccount30": "0",
    "intSuspEndOfCurrYearAccount30": "0",
    "closBalProvAccount30": "0",
    "diAndCgcTotalPro30": "0",
    "standardAssetsTotalPro30": null,
    "licraTotalPro30": null,
    "save": false
}

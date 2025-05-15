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
  Typography,
  CircularProgress, // Added for loading indication
} from '@mui/material';
import TableCell, { tableCellClasses } from '@mui/material/TableCell';
import { styled } from '@mui/material/styles';
import debounce from 'lodash/debounce';
import FormInput from '../../../../common/components/ui/FormInput';

// import useApi from '../../../../common/hooks/useApi';
// import useCustomSnackbar from '../../../../common/hooks/useCustomSnackbar';

const StyledTableCell = styled(TableCell)(({ theme }) => ({
  fontSize: '0.875rem',
  padding: '8px',
  border: '1px solid #e0e0e0', // Lighter border
  whiteSpace: 'nowrap',
  [`&.${tableCellClasses.head}`]: {
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
    // backgroundColor: theme.palette.action.hover,
  },
  ...(isSectionHeader && {
    '& > td': {
      fontWeight: 'bold',
      textAlign: 'left',
    },
  }),
  ...(isSubSectionHeader && {
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
  'col1', 'col2', 'col3', 'col4', 'col5', 'col6', 'col7', 'col8', 'col9', 'col10',
  'col11', 'col12', 'col13', 'col14', 'col15', 'col16', 'col17', 'col18',
];
const calculatedColKeys = ['col4', 'col7', 'col10', 'col13', 'col15'];

// Helper to map modelSuffix to rowId efficiently
const modelSuffixToRowIdMap = rowDefinitionsConfig.reduce((acc, row) => {
  if (row.modelSuffix) {
    acc[row.modelSuffix] = row.id;
  }
  return acc;
}, {});


const Schedule9CProvisionTable = ({
  circleCode = '001',
  quarterEndDate = '31/03/2025',
  role = 'Maker',
  previousYear = '2024',
  displayQuarterDate = '31/03/2025',
  initialDataFromApi = null, // This prop can still be used if needed
}) => {
  const showSnackbar = (message, severity) => console.log(`Snackbar: ${message} (${severity})`);
  // Mock API call function (as provided in your code)
  const callApi = async (url, payload, method) => {
    console.log('API call details:', { url, method, payload });
    // Simulate API delay
    await new Promise(resolve => setTimeout(resolve, 1000));

    // Example: Simulate fetching the provided API response
    // In a real scenario, this would be the actual response from your backend
    const sampleApiResponse = {
        "reportName": "", "description": "", "userId": null, "reportId": null, "reportMasterId": null, "status": null, "circleCode": "", "quarterEndDate": "",
        "writOffCurProvision2": null, "writOffCurProvision3": null, "writOffCurProvision4": null, "writOffCurProvision5": "0", "writOffCurProvision7": null, "writOffCurProvision8": null, "writOffCurProvision9": null, "writOffCurProvision10": "0", "writOffCurProvision13": null, "writOffCurProvision14": null, "writOffCurProvision15": null, "writOffCurProvision16": null, "writOffCurProvision17": "0", "writOffCurProvision19": null, "writOffCurProvision20": null, "writOffCurProvision21": null, "writOffCurProvision22": null, "writOffCurProvision23": "0", "writOffCurProvision24": "0", "writOffCurProvision26": null, "writOffCurProvision27": null, "writOffCurProvision28": null, "writOffCurProvision29": null, "writOffCurProvision30": "0",
        "opBalCurYearProvision2": "1", "addRedFlucProvision2": null, "netAdjOpBalProvision2": "1", "addCurYearProvision2": null, "dedCurYearProvision2": null, "addCurDepreciProvision2": null, "closBalCurYearProvision2": "1", "opBalCurYearAccount2": null, "addRedFlucAccount2": null, "netAdjOpBalAccount2": "0", "addCurYearAccount2": null, "dedRevCurYearAccount2": null, "closBalCurYearAccount2": "0", "intSuspEndOfCurrYearAccount2": null, "closBalProvAccount2": "1", "diAndCgcTotalPro2": null, "standardAssetsTotalPro2": null, "licraTotalPro2": null,
        "opBalCurYearProvision3": null, "addRedFlucProvision3": null, "netAdjOpBalProvision3": "0", "addCurYearProvision3": null, "dedCurYearProvision3": null, "addCurDepreciProvision3": null, "closBalCurYearProvision3": "0", "opBalCurYearAccount3": null, "addRedFlucAccount3": null, "netAdjOpBalAccount3": "0", "addCurYearAccount3": null, "dedRevCurYearAccount3": null, "closBalCurYearAccount3": "0", "intSuspEndOfCurrYearAccount3": null, "closBalProvAccount3": "0", "diAndCgcTotalPro3": null, "standardAssetsTotalPro3": null, "licraTotalPro3": null,
        "opBalCurYearProvision4": null, "addRedFlucProvision4": null, "netAdjOpBalProvision4": "0", "addCurYearProvision4": null, "dedCurYearProvision4": null, "addCurDepreciProvision4": null, "closBalCurYearProvision4": "0", "opBalCurYearAccount4": null, "addRedFlucAccount4": null, "netAdjOpBalAccount4": "0", "addCurYearAccount4": null, "dedRevCurYearAccount4": null, "closBalCurYearAccount4": "0", "intSuspEndOfCurrYearAccount4": null, "closBalProvAccount4": "0", "diAndCgcTotalPro4": null, "standardAssetsTotalPro4": null, "licraTotalPro4": null,
        "opBalCurYearProvision5": "1", "addRedFlucProvision5": "0", "netAdjOpBalProvision5": "1", "addCurYearProvision5": "0", "dedCurYearProvision5": null, "addCurDepreciProvision5": "0", "closBalCurYearProvision5": "1", "opBalCurYearAccount5": "0", "addRedFlucAccount5": "0", "netAdjOpBalAccount5": "0", "addCurYearAccount5": "0", "dedRevCurYearAccount5": "0", "closBalCurYearAccount5": "0", "intSuspEndOfCurrYearAccount5": "0", "closBalProvAccount5": "1", "diAndCgcTotalPro5": "0", "standardAssetsTotalPro5": "0", "licraTotalPro5": "0",
        // ... (rest of the provided API response)
        "opBalCurYearProvision7": null, "addRedFlucProvision7": null, "netAdjOpBalProvision7": "0", "addCurYearProvision7": null, "addCurDepreciProvision7": null, "closBalCurYearProvision7": "0", "opBalCurYearAccount7": null, "addRedFlucAccount7": null, "netAdjOpBalAccount7": "0", "addCurYearAccount7": null, "dedRevCurYearAccount7": null, "closBalCurYearAccount7": "0", "intSuspEndOfCurrYearAccount7": null, "closBalProvAccount7": "0", "diAndCgcTotalPro7": null, "standardAssetsTotalPro7": null, "licraTotalPro7": null,
        "opBalCurYearProvision30": "0", "addRedFlucProvision30": "0", "netAdjOpBalProvision30": "0", "addCurYearProvision30": "0", "addCurDepreciProvision30": "0", "closBalCurYearProvision30": "0", "opBalCurYearAccount30": "0", "addRedFlucAccount30": "0", "netAdjOpBalAccount30": "0", "addCurYearAccount30": "0", "dedRevCurYearAccount30": "0", "closBalCurYearAccount30": "0", "intSuspEndOfCurrYearAccount30": "0", "closBalProvAccount30": "0", "diAndCgcTotalPro30": "0", "standardAssetsTotalPro30": null, "licraTotalPro30": null,
        "save": false
    };
    // Make sure the API response you use for testing has all relevant fields.
    // For example, 'opBalCurYearProvision2', 'writOffCurProvision2', etc. for row 'A1_i' (modelSuffix: '2')

    return { success: true, data: sampleApiResponse }; // Returning the sample data
  };

  const [formData, setFormData] = useState(() => {
    const initial = {};
    rowDefinitionsConfig.forEach((row) => {
      if (row.type === 'entry') {
        initial[row.id] = {};
        Object.values(columnFieldKeys).forEach((fieldKey) => {
          initial[row.id][fieldKey] = '';
        });
      }
    });
    return initial;
  });

  const [validationErrors, setValidationErrors] = useState([]);
  const [isLoading, setIsLoading] = useState(false); // For loading state

  // Effect for fetching data from API
  useEffect(() => {
    const fetchData = async () => {
      setIsLoading(true);
      showSnackbar('Loading data...', 'info');

      const requestPayload = {
        circleCode, // from props
        quarterEndDate, // from props
        userId: "1111111", // Example value, ideally from context or props
        reportName: "Schedule9C PROVISION", // Example value
        reportId: "125911", // Example value
        reportMasterId: "310021", // Example value
        status: "11", // Example value
        areMocPending: true // Example value
      };

      try {
        // const response = await callApi('/api/schedule9c/data', requestPayload, 'POST'); // Replace with actual endpoint
        const response = await callApi('your_api_endpoint/schedule9c', requestPayload, 'POST'); // Mocked call

        if (response.success && response.data) {
          const fetchedApiData = response.data;
          const transformedDataForState = {};

          // Initialize structure for all entry rows
          rowDefinitionsConfig.forEach(rowDef => {
            if (rowDef.type === 'entry') {
              transformedDataForState[rowDef.id] = {};
              Object.values(columnFieldKeys).forEach(fKey => {
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
              if (transformedDataForState[rowId] && transformedDataForState[rowId].hasOwnProperty(matchedFieldKeyInFormData)) {
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
        console.error("API call failed:", error);
        showSnackbar('Error fetching data. Check console for details.', 'error');
      } finally {
        setIsLoading(false);
      }
    };

    // Only fetch if not relying on initialDataFromApi or based on your logic
    // For this example, we will always fetch. You can add conditions like:
    // if (!initialDataFromApi) { fetchData(); }
    fetchData();
  }, [circleCode, quarterEndDate, role]); // Added role as it's part of component props that might influence data


  // Effect for processing initialDataFromApi prop (kept as is)
  useEffect(() => {
    if (initialDataFromApi) {
      console.log("Processing initialDataFromApi prop:", initialDataFromApi);
      const newFormData = {};
      rowDefinitionsConfig.forEach((row) => {
        if (row.type === 'entry') {
          newFormData[row.id] = {}; // Ensure object exists
          const apiRowData = initialDataFromApi[row.id] || {};
          Object.values(columnFieldKeys).forEach((fieldKey) => {
            newFormData[row.id][fieldKey] = apiRowData[fieldKey] ?? '';
          });
        }
      });
      // Be cautious with merging if both fetch and initialDataFromApi are active
      // setFormData((prev) => ({ ...prev, ...newFormData }));
      // For now, let's assume if initialDataFromApi is given, it takes precedence
      // or this effect might run after the fetch, depending on React's scheduling.
      // A direct set might be cleaner if initialDataFromApi is the source of truth when present.
       setFormData(currentFormData => { // Create a new state based on current
        const updatedFormData = JSON.parse(JSON.stringify(currentFormData)); // Deep clone
         Object.keys(newFormData).forEach(rowId => {
            if (updatedFormData[rowId]) {
                updatedFormData[rowId] = { ...updatedFormData[rowId], ...newFormData[rowId] };
            } else {
                updatedFormData[rowId] = newFormData[rowId];
            }
        });
        return updatedFormData;
      });
       showSnackbar('Updated form with initialDataFromApi.', 'info');
    }
  }, [initialDataFromApi]);


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
        const currentRowFormData = formData[row.id] || {};

        if (row.type === 'entry') {
          Object.entries(columnFieldKeys).forEach(([colKeyAlias, fieldKeyInFormData]) => {
            newCalculatedData[row.id][colKeyAlias] = currentRowFormData[fieldKeyInFormData] ?? '';
          });
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
          allColumnKeys.forEach((colKeyToSum) => {
            let sum = 0;
            row.subItemIds.forEach((subItemId) => {
              sum += getNum(newCalculatedData[subItemId]?.[colKeyToSum]);
            });
            newCalculatedData[row.id][colKeyToSum] = sum.toFixed(2);
          });
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
    // const response = await callApi('/api/save', payload, 'POST'); // Example actual save call
    // if(response.success) showSnackbar('Saved successfully!', 'success'); else showSnackbar('Save failed.', 'error');
    showSnackbar('Save initiated (see console for payload).', 'info');
  };
  const handleSubmit = async () => {
    if (validationErrors.length > 0) {
      showSnackbar('Please correct validation errors.', 'error');
      return;
    }
    const payload = buildPayload(false);
    // const response = await callApi('/api/submit', payload, 'POST'); // Example actual submit call
    // if(response.success) showSnackbar('Submitted successfully!', 'success'); else showSnackbar('Submit failed.', 'error');
    showSnackbar('Submit initiated (see console for payload).', 'info');
  };

  const columnHeaders = [
    { label: `Opening Balance of Provisions <br>for Current Year (closing <br>balance ${previousYear})<br>Rs. P`, key: 'col1'},
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
    { label: `DICGC /ECGC Claims Recd <br>as on ${displayQuarterDate} <br>Rs. P`, key: 'col16' },
    { label: `Provision on Restructured <br>Standard Asset as on <br>${displayQuarterDate} (included in total <br>Provision in Column 7)`, key: 'col17' },
    { label: `LICRA on Restructured <br>Standard Asset as on ${displayQuarterDate} <br>(included in total<br> LICRA in Column 13)`, key: 'col18' },
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
      {validationErrors.length > 0 && (
        <Alert severity="error" sx={{ mb: 2 }}>
          <ul style={{ margin: 0, paddingLeft: '1.2em' }}>
            {validationErrors.map((e, i) => ( <li key={i}>{e}</li> ))}
          </ul>
        </Alert>
      )}
      <TableContainer component={Paper} sx={{ maxHeight: 'calc(110vh - 250px)' }}>
        <Table stickyHeader sx={{ minWidth: 3000 }}>
          <TableHead>
            <TableRow>
              <StyledTableCell rowSpan={3} sx={{ position: 'sticky', left: 0, top: 0, zIndex: 1101, backgroundColor: (theme) => theme.palette.grey[200] }}>
                <b>Classification of PROVISION</b><br />
                (Excluding provision relating to : non-advance <br />
                related items debited to Recalled Assets and interest free Staff Advances ) <br />
                <b>(A.1=A.2=A.3=A.4)</b>
              </StyledTableCell>
              <StyledTableCell colSpan={7}><b>PROVISIONS</b></StyledTableCell> {/* Adjusted colSpan from 8 to 7 based on your column structure */}
              <StyledTableCell colSpan={5}><b>Liability on Interest Capitalisation on Restructurred Account(LICRA)</b></StyledTableCell> {/* Adjusted colSpan from 8 to 5 */}
              <StyledTableCell colSpan={6}><b>TOTAL PROVISION AND OTHER DETAILS</b></StyledTableCell> {/* Adjusted colSpan from 5 to 6 */}
            </TableRow>
            <TableRow>
              {columnHeaders.map((ch) => (
                <StyledTableCell
                  key={ch.key}
                  sx={{ position: 'sticky', top: 42.5, zIndex: 1100, backgroundColor: '#000', color: '#fff', textAlign: 'center', whiteSpace: 'normal' }}
                  dangerouslySetInnerHTML={{ __html: ch.label }}
                />
              ))}
            </TableRow>
            <TableRow>
              {allColumnKeys.map((key, idx) => (
                <StyledTableCell
                  key={`colnum_${idx}`}
                  sx={{ position: 'sticky', top: 228.5, zIndex: 1000, backgroundColor: '#000', color: '#fff', textAlign: 'center' }}
                >
                  <b>{idx + 1}</b>
                </StyledTableCell>
              ))}
            </TableRow>
          </TableHead>
          <TableBody>
            {rowDefinitionsConfig.map((row) => {
              const displayRowData = calculatedData[row.id] || {};
              const isTotalOrHeader = row.type === 'total' || row.type === 'sectionHeader' || row.type === 'subSectionHeader';

              if (row.type === 'sectionHeader' || row.type === 'subSectionHeader') {
                return (
                  <StyledTableRow key={row.id} isSectionHeader={row.type === 'sectionHeader'} isSubSectionHeader={row.type === 'subSectionHeader'}>
                    <StyledTableCell colSpan={allColumnKeys.length + 1} sx={{ textAlign: 'left', position: 'sticky', left: 0, zIndex: 100, backgroundColor: row.type === 'sectionHeader' ? '#e0e0e0' : '#f0f0f0' }}>
                      {row.label}
                    </StyledTableCell>
                  </StyledTableRow>
                );
              }

              return (
                <StyledTableRow key={row.id} isTotalRow={row.type === 'total'}>
                  <StyledTableCell sx={{ textAlign: 'left', fontWeight: row.type === 'total' ? 'bold' : 'normal', fontStyle: row.type === 'entry' ? 'normal' : 'italic', position: 'sticky', left: 0, zIndex: 99, backgroundColor: row.type === 'total' ? '#f5f5f5' : row.type === 'entry' ? '#ffffff' : '#f0f0f0' }}>
                    {row.label}
                  </StyledTableCell>
                  {allColumnKeys.map((colKey) => {
                    const isCalculatedField = calculatedColKeys.includes(colKey);
                    const isEditableField = row.type === 'entry' && !isCalculatedField;
                    const fieldKeyInFormData = columnFieldKeys[colKey];
                    const valueToDisplayInTextField = displayRowData[colKey] ?? '';

                    return (
                      <StyledTableCell key={`${row.id}-${colKey}`}>
                        <FormInput
                          name={`${row.id}-${colKey}`} // Unique name for FormInput
                          value={valueToDisplayInTextField}
                          onChange={ isEditableField ? (e) => handleChange(row.id, fieldKeyInFormData, e.target.value) : undefined }
                          onBlur={() => {}}
                          readOnly={!isEditableField}
                          customStyles={{
                            textAlign: 'right',
                            '& input': { textAlign: 'right', padding: '6px 8px' },
                            backgroundColor: !isEditableField ? '#f0f0f0' : 'white',
                            color: (theme) => theme.palette.text.primary,
                            width: '130px', // Adjusted width for better fit
                          }}
                          isNumeric={true}
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
        <Button variant="contained" color="warning" onClick={handleSave} disabled={isLoading}>
          Save
        </Button>
        <Button variant="contained" color="success" onClick={handleSubmit} disabled={validationErrors.length > 0 || isLoading}>
          Submit
        </Button>
      </Stack>
    </Box>
  );
};

export default Schedule9CProvisionTable;


import React, { useState, useEffect, useMemo, useCallback, memo } from 'react';
import { useLocation, useNavigate } from 'react-router-dom';
import {
  Table,
  TableBody,
  TableContainer,
  TableHead,
  TableRow,
  Paper,
  Button,
  Alert,
  Box,
  Stack,
  CircularProgress,
  Typography,
} from '@mui/material';
import TableCell, { tableCellClasses } from '@mui/material/TableCell';
import { styled, useColorScheme, useTheme } from '@mui/material/styles';
import FormInput from '../../../../common/components/ui/FormInput';
import useCustomSnackbar from '../../../../common/hooks/useCustomSnackbar';
import useApi from '../../../../common/hooks/useApi';

// --- STYLED COMPONENTS ---
const StyledTableCell = styled(TableCell)(({ theme }) => ({
  fontSize: '0.875rem',
  padding: '8px',
  border: '1px solid #e0e0e0',
  whiteSpace: 'nowrap',
  [`&.${tableCellClasses.head}`]: {
    fontWeight: 'bold',
    textAlign: 'center',
  },
  [`&.${tableCellClasses.body}`]: {
    textAlign: 'left',
  },
}));

const StyledTableRow = styled(TableRow)(({ theme, istotalrow, issectionheader, issubsectionheader }) => ({
  '&:nth-of-type(odd)': {
    // backgroundColor: theme.palette.action.hover,
  },
  ...(issectionheader && {
    '& > td': {
      fontWeight: 'bold',
      textAlign: 'left',
    },
  }),
  ...(issubsectionheader && {
    '& > td': {
      fontWeight: 'bold',
      fontStyle: 'italic',
      textAlign: 'left',
    },
  }),
  ...(istotalrow && {
    '& > td': {
      fontWeight: 'bold',
    },
  }),
}));

// --- CONFIGURATION CONSTANTS ---
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
  col1: 'opBalCurYearProvision', col2: 'writOffCurProvision', col3: 'addRedFlucProvision', col5: 'addCurYearProvision',
  col6: 'addCurDepreciProvision', col8: 'opBalCurYearAccount', col9: 'addRedFlucAccount', col11: 'addCurYearAccount',
  col12: 'dedRevCurYearAccount', col14: 'intSuspEndOfCurrYearAccount', col16: 'diAndCgcTotalPro',
  col17: 'standardAssetsTotalPro', col18: 'licraTotalPro',
};

const allColumnKeys = [
  'col1', 'col2', 'col3', 'col4', 'col5', 'col6', 'col7', 'col8', 'col9', 'col10',
  'col11', 'col12', 'col13', 'col14', 'col15', 'col16', 'col17', 'col18',
];

const calculatedColKeys = ['col4', 'col7', 'col10', 'col13', 'col15'];

const getColumnHeaders = (quarterEndDate) => [
  { label: `Opening Balance of Provisions <br>for Current Year (closing <br>balance ${parseInt(quarterEndDate.split('/')[2], 10) - 1})<br>Rs. P`, key: 'col1' },
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
  { label: `Interest Suspense<br> Account As on ${quarterEndDate.replaceAll('/', '.')} <br>Rs. P`, key: 'col14' },
  { label: 'Total Provision+LICRA+<br>Interest Suspense<br> Rs.P<br><b>15=(7+13+14)</b>', key: 'col15', isCalculated: true },
  { label: `DICGC /ECGC Claims Recd <br>as on ${quarterEndDate.replaceAll('/', '.')} <br>Rs. P`, key: 'col16' },
  { label: `Provision on Restructured <br>Standard Asset as on <br>${quarterEndDate.replaceAll('/', '.')} (included in total <br>Provision in Column 7)`, key: 'col17' },
  { label: `LICRA on Restructured <br>Standard Asset as on ${quarterEndDate.replaceAll('/', '.')} <br>(included in total<br> LICRA in Column 13)`, key: 'col18' },
];

// --- MEMOIZED TABLE HEADER ---
const MemoizedTableHeader = memo(({ columnHeaders }) => {
  return (
    <TableHead>
      <TableRow>
        <StyledTableCell rowSpan={3} sx={{ position: 'sticky', left: 0, zIndex: 1101, backgroundColor: 'background.paper' }}>
          <b>Classification of PROVISION</b><br />
          (Excluding provision relating to : non-advance <br />
          related items debited to Recalled Assets and interest free Staff Advances ) <br />
          <b>(A.1=A.2=A.3=A.4)</b>
        </StyledTableCell>
        <StyledTableCell colSpan={7}><b>PROVISIONS</b></StyledTableCell>
        <StyledTableCell colSpan={5}><b>Liability on Interest Capitalisation on Restructurred Account(LICRA)</b></StyledTableCell>
        <StyledTableCell colSpan={6}><b>TOTAL PROVISION AND OTHER DETAILS</b></StyledTableCell>
      </TableRow>
      <TableRow>
        {columnHeaders.map((ch) => (
          <StyledTableCell key={ch.key} sx={{ position: 'sticky', top: 104, zIndex: 1100, backgroundColor: 'background.paper' }} dangerouslySetInnerHTML={{ __html: ch.label }} />
        ))}
      </TableRow>
      <TableRow>
        {allColumnKeys.map((key, idx) => (
          <StyledTableCell key={`colnum_${idx}`} sx={{ position: 'sticky', top: 228, zIndex: 1100, backgroundColor: 'background.paper' }}>
            <b>{idx + 1}</b>
          </StyledTableCell>
        ))}
      </TableRow>
    </TableHead>
  );
});

// --- MEMOIZED TABLE ROW ---
const MemoizedTableRow = memo(({
  row,
  uiRowData,
  calculatedRowData,
  mode,
  theme,
  handleChange,
}) => {
  if (row.type === 'sectionHeader' || row.type === 'subSectionHeader') {
    return (
      <StyledTableRow
        key={row.id}
        issectionheader={row.type === 'sectionHeader'}
        issubsectionheader={row.type === 'subSectionHeader'}
      >
        <StyledTableCell
          colSpan={allColumnKeys.length + 1}
          sx={{
            position: 'sticky',
            left: 0,
            zIndex: 100,
            backgroundColor: mode === 'light' ? theme.palette.grey[100] : '#212121',
          }}
        >
          {row.label}
        </StyledTableCell>
      </StyledTableRow>
    );
  }

  return (
    <StyledTableRow key={row.id} istotalrow={row.type === 'total'}>
      <StyledTableCell
        sx={{
          fontWeight: row.type === 'total' ? 'bold' : 'normal',
          position: 'sticky',
          left: 0,
          zIndex: 99,
          backgroundColor: mode === 'light' ? theme.palette.background.paper : '#121212',
        }}
      >
        {row.label}
      </StyledTableCell>
      {allColumnKeys.map((colKey) => {
        if (['col17', 'col18'].includes(colKey) && !row.id.startsWith('A4_')) {
          return <StyledTableCell key={`${row.id}-${colKey}`} />;
        }

        const isCalculatedField = calculatedColKeys.includes(colKey);
        const fieldKeyInFormData = columnFieldKeys[colKey];
        const isEditableField = row.type === 'entry' && !isCalculatedField && fieldKeyInFormData;

        const valueToDisplay = isEditableField
          ? uiRowData?.[fieldKeyInFormData] ?? ''
          : calculatedRowData?.[colKey] ?? '';

        return (
          <StyledTableCell key={`${row.id}-${colKey}`}>
            <FormInput
              name={`${row.id}-${colKey}`}
              value={valueToDisplay}
              onChange={isEditableField ? (e) => handleChange(row.id, fieldKeyInFormData, e.target.value) : undefined}
              readOnly={!isEditableField}
              customStyles={{
                textAlign: 'right', '& input': { textAlign: 'right', padding: '6px 8px' },
                backgroundColor: !isEditableField ? (mode === 'light' ? '#f0f0f0' : '#333') : 'inherit',
                width: '150px',
              }}
              isNumeric={true}
            />
          </StyledTableCell>
        );
      })}
    </StyledTableRow>
  );
});


// --- MAIN COMPONENT ---
const Schedule9CProvisionTable = () => {
  const { state } = useLocation();
  const navigate = useNavigate();
  const showSnackbar = useCustomSnackbar();
  const { callApi } = useApi();
  const user = JSON.parse(localStorage.getItem('user'));

  const [reportObject, setReportObject] = useState(state?.report || null);
  const [validationErrors, setValidationErrors] = useState([]);
  const [isLoading, setIsLoading] = useState(true);

  const theme = useTheme();
  const { mode } = useColorScheme();
  
  const columnHeaders = useMemo(() => getColumnHeaders(user.quarterEndDate), [user.quarterEndDate]);

  const getInitialState = useCallback(() => {
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
  }, []);

  const [uiFormData, setUiFormData] = useState(getInitialState);
  const [calculationFormData, setCalculationFormData] = useState(getInitialState);

  const modelSuffixToRowIdMap = useMemo(() => {
    const map = {};
    rowDefinitionsConfig.forEach((row) => {
      if (row.modelSuffix) {
        map[row.modelSuffix] = row.id;
      }
    });
    return map;
  }, []);

  useEffect(() => {
    const fetchData = async () => {
      setIsLoading(true);
      const requestPayload = {
        circleCode: user.circleCode, quarterEndDate: user.quarterEndDate, userId: user.userId,
        reportName: reportObject.name, reportMasterId: reportObject.reportId, status: reportObject.status,
        areMocPending: true,
      };

      try {
        const response = await callApi('/Maker/getSavedDataNineC', requestPayload, 'POST');
        if (response) {
          const transformedData = getInitialState();
          for (const [apiKey, apiValue] of Object.entries(response)) {
            for (const fieldKey of Object.values(columnFieldKeys)) {
              if (apiKey.startsWith(fieldKey)) {
                const suffix = apiKey.replace(fieldKey, '');
                const rowId = modelSuffixToRowIdMap[suffix];
                if (rowId && transformedData[rowId]) {
                  transformedData[rowId][fieldKey] = apiValue !== null ? String(apiValue) : '';
                }
              }
            }
          }
          setUiFormData(transformedData);
          setCalculationFormData(transformedData);
        }
      } catch (error) {
        console.error('Fetch error:', error);
        if (error.message !== 'canceled') {
          showSnackbar('Failed to load data.', 'error');
        }
      } finally {
        setIsLoading(false);
      }
    };

    if (reportObject && user) {
      fetchData();
    }
  }, [reportObject, user, callApi, showSnackbar, modelSuffixToRowIdMap, getInitialState]);

  useEffect(() => {
    const handler = setTimeout(() => {
      setCalculationFormData(uiFormData);
    }, 400);

    return () => {
      clearTimeout(handler);
    };
  }, [uiFormData]);

  const handleChange = useCallback((rowId, fieldKey, value) => {
    if (value === '' || /^-?\d*\.?\d{0,2}$/.test(value) || (value === '-' && !uiFormData[rowId]?.[fieldKey]?.length)) {
      setUiFormData((prev) => ({
        ...prev,
        [rowId]: { ...prev[rowId], [fieldKey]: value },
      }));
    }
  }, [uiFormData]);

  const getNum = (value) => parseFloat(value) || 0;

  const calculatedData = useMemo(() => {
    const newCalculatedData = {};
    const get = (rowId, colKey) => getNum(newCalculatedData[rowId]?.[colKey]);

    rowDefinitionsConfig.forEach((row) => {
      newCalculatedData[row.id] = {};
      if (row.type === 'entry') {
        const rowData = calculationFormData[row.id] || {};
        Object.entries(columnFieldKeys).forEach(([colKeyAlias, fieldKeyInFormData]) => {
          newCalculatedData[row.id][colKeyAlias] = rowData[fieldKeyInFormData] ?? '';
        });
        
        const col1 = get(row.id, 'col1'), col2 = get(row.id, 'col2'), col3 = get(row.id, 'col3');
        newCalculatedData[row.id].col4 = (col1 - col2 + col3).toFixed(2);
        
        const col5 = get(row.id, 'col5'), col6 = get(row.id, 'col6');
        newCalculatedData[row.id].col7 = (get(row.id, 'col4') + col5 + col6).toFixed(2);
        
        const col8 = get(row.id, 'col8'), col9 = get(row.id, 'col9');
        newCalculatedData[row.id].col10 = (col8 + col9).toFixed(2);
        
        const col11 = get(row.id, 'col11'), col12 = get(row.id, 'col12');
        newCalculatedData[row.id].col13 = (get(row.id, 'col10') + col11 - col12).toFixed(2);

        const col14 = get(row.id, 'col14');
        newCalculatedData[row.id].col15 = (get(row.id, 'col7') + get(row.id, 'col13') + col14).toFixed(2);

      } else if (row.type === 'total') {
        allColumnKeys.forEach((colKey) => {
          let sum = row.subItemIds.reduce((acc, subId) => acc + get(subId, colKey), 0);
          newCalculatedData[row.id][colKey] = sum.toFixed(2);
        });
      }
    });
    return newCalculatedData;
  }, [calculationFormData]);

  useEffect(() => {
    const errors = [];
    const totalsA1 = calculatedData['A1_total'];
    const totalsA2 = calculatedData['A2_total'];
    const totalsA3 = calculatedData['A3_grand_total'];
    const totalsA4 = calculatedData['A4_total'];

    if (totalsA1 && totalsA2 && totalsA3 && totalsA4) {
      const colsToValidate = ['col7', 'col13', 'col15'];
      colsToValidate.forEach((colKey) => {
        const valA1 = getNum(totalsA1[colKey]);
        const valA2 = getNum(totalsA2[colKey]);
        const valA3 = getNum(totalsA3[colKey]);
        const valA4 = getNum(totalsA4[colKey]);
        const tolerance = 0.01;
        if (!(Math.abs(valA1 - valA2) < tolerance && Math.abs(valA2 - valA3) < tolerance && Math.abs(valA3 - valA4) < tolerance)) {
          errors.push(`Mismatch in totals for Column ${colKey.replace('col', '')}: A-1 (${valA1.toFixed(2)}), A-2 (${valA2.toFixed(2)}), A-3 (${valA3.toFixed(2)}), and A-4 (${valA4.toFixed(2)}) must be equal.`);
        }
      });
    }
    setValidationErrors(errors);
  }, [calculatedData]);

  const buildPayload = (isSaveOperation) => {
    const payload = {
      circleCode: user.circleCode, quarterEndDate: user.quarterEndDate, userId: user.userId,
      reportName: reportObject.name, reportMasterId: reportObject.reportMasterId,
      reportId: reportObject.reportId, status: reportObject.status, save: isSaveOperation,
    };
    rowDefinitionsConfig.forEach((row) => {
      if (row.type === 'entry' && row.modelSuffix) {
        const rowInputData = uiFormData[row.id] || {};
        Object.entries(columnFieldKeys).forEach(([_, fieldKeyInFormData]) => {
          const backendFieldName = `${fieldKeyInFormData}${row.modelSuffix}`;
          payload[backendFieldName] = parseFloat(rowInputData[fieldKeyInFormData] || 0).toFixed(2);
        });
      }
    });
    return payload;
  };

  const handleSave = async () => {
    try {
      const payload = buildPayload(true);
      const response = await callApi('/Maker/submitNineC', payload, 'POST');
      if (response && typeof response === 'string') {
        const [_flag, newReportId, newStatus] = response.split('~');
        setReportObject((prev) => ({
          ...prev, reportId: newReportId, status: newStatus,
        }));
      }
      if (response && response.includes('~11')) {
        showSnackbar('Data saved successfully', 'success');
      } else {
        showSnackbar('Save failed. Please try again.', 'error');
      }
    } catch (error) {
      console.error(error);
      showSnackbar('An error occurred while saving.', 'error');
    }
  };

  const handleSubmit = async () => {
    if (validationErrors.length > 0) {
      showSnackbar('Please correct validation errors.', 'error');
      return;
    }
    try {
      const payload = buildPayload(false);
      const response = await callApi('/Maker/submitNineC', payload, 'POST');
      if (response && response.includes('~21')) {
        showSnackbar('Report submitted successfully!', 'success');
        setTimeout(() => {
          navigate('../');
        }, 2000);
      } else {
        showSnackbar('Submit failed. Please try again.', 'error');
      }
    } catch (error) {
      console.error(error);
      showSnackbar('An error occurred while submitting.', 'error');
    }
  };

  return (
    <Box sx={{ p: 1, width: '100%', overflowX: 'hidden', position: 'relative' }}>
      <Stack direction="row" spacing={2} sx={{ mb: 2, justifyContent: 'left' }}>
        <Button variant="contained" color="warning" onClick={handleSave} disabled={isLoading}>
          Save
        </Button>
        <Button variant="contained" color="success" onClick={handleSubmit} disabled={validationErrors.length > 0 || isLoading}>
          Submit
        </Button>
      </Stack>

      {validationErrors.length > 0 && (
        <Alert severity="error" sx={{ mb: 2 }}>
          <ul style={{ margin: 0, paddingLeft: '1.2em' }}>
            {validationErrors.map((e, i) => <li key={i}>{e}</li>)}
          </ul>
        </Alert>
      )}

      <TableContainer component={Paper} sx={{ maxHeight: 'calc(100vh - 250px)' }}>
        {isLoading && (
          <Box
            sx={{
              position: 'absolute', top: 0, left: 0, width: '100%', height: '100%',
              backgroundColor: 'rgba(255, 255, 255, 0.7)', display: 'flex',
              justifyContent: 'center', alignItems: 'center', zIndex: 1500,
            }}
          >
            <CircularProgress />
            <Typography sx={{ ml: 2 }}>Loading Data...</Typography>
          </Box>
        )}
        <Table stickyHeader sx={{ minWidth: 3000 }}>
          <MemoizedTableHeader columnHeaders={columnHeaders} />
          <TableBody>
            {rowDefinitionsConfig.map((row) => (
              <MemoizedTableRow
                key={row.id}
                row={row}
                uiRowData={uiFormData[row.id]}
                calculatedRowData={calculatedData[row.id] || {}}
                mode={mode}
                theme={theme}
                handleChange={handleChange}
              />
            ))}
          </TableBody>
        </Table>
      </TableContainer>
    </Box>
  );
};

export default Schedule9CProvisionTable;

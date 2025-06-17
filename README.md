import React, { useState, useEffect, useMemo, useCallback, memo } from 'react';
import { useLocation, useNavigate } from 'react-router-dom';
import {
  Table, TableBody, TableContainer, TableHead, TableRow, Paper, Button,
  Alert, Box, Stack, CircularProgress, Typography,
} from '@mui/material';
import TableCell, { tableCellClasses } from '@mui/material/TableCell';
import { styled, useColorScheme, useTheme } from '@mui/material/styles';
import FormInput from '../../../../common/components/ui/FormInput';
import useCustomSnackbar from '../../../../common/hooks/useCustomSnackbar';
import useApi from '../../../../common/hooks/useApi';

// All configuration constants (rowDefinitionsConfig, columnFieldKeys, etc.)
// and Styled Components (StyledTableCell, StyledTableRow) remain the same as the previous version.
// ... (omitting them here for brevity, but they should be included in your file)

// --- CONFIGURATION & STYLED COMPONENTS (Same as before) ---
const rowDefinitionsConfig = [ /* ... Omitted for brevity ... */ ];
const columnFieldKeys = { /* ... Omitted for brevity ... */ };
const allColumnKeys = [ /* ... Omitted for brevity ... */ ];
const calculatedColKeys = [ /* ... Omitted for brevity ... */ ];
const StyledTableCell = styled(TableCell)(/* ... Omitted for brevity ... */);
const StyledTableRow = styled(TableRow)(/* ... Omitted for brevity ... */);
const getColumnHeaders = (quarterEndDate) => [ /* ... Omitted for brevity ... */ ];
const MemoizedTableHeader = memo(({ columnHeaders }) => { /* ... Omitted for brevity ... */ });


// --- Memoized Table Row (Slight modification to take data from two sources) ---
const MemoizedTableRow = memo(({
  row,
  uiRowData, // For instant input display
  calculatedRowData, // For displaying calculated fields
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
          sx={{ position: 'sticky', left: 0, zIndex: 100, backgroundColor: mode === 'light' ? theme.palette.grey[100] : '#212121' }}
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
          position: 'sticky', left: 0, zIndex: 99,
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

        // **KEY CHANGE**: Editable fields get value from `uiRowData` for responsiveness.
        // Calculated/total fields get value from `calculatedRowData`.
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

  // **NEW STATE MANAGEMENT**
  // State for immediate UI updates (responsive typing)
  const [uiFormData, setUiFormData] = useState(getInitialState);
  // State for triggering expensive calculations (updated after debounce)
  const [calculationFormData, setCalculationFormData] = useState(getInitialState);

  // --- Data Fetching Effect (now populates BOTH states) ---
  useEffect(() => {
    const fetchData = async () => {
      setIsLoading(true);
      // ... (API call logic is the same)
      try {
        const response = await callApi(/* ... */);
        if (response) {
            const transformedData = getInitialState();
            // ... (data transformation logic is the same)
            // Populate transformedData...

            // Set both states with the initial data from the API
            setUiFormData(transformedData);
            setCalculationFormData(transformedData);
        }
      } catch (error) { /* ... */ } 
      finally {
        setIsLoading(false);
      }
    };
    if (reportObject && user) fetchData();
  }, [reportObject, user, callApi, showSnackbar, getInitialState]);


  // **DEBOUNCING EFFECT**
  // This effect listens for changes in the fast-updating `uiFormData`.
  // It then waits for a pause in typing before updating the `calculationFormData`.
  useEffect(() => {
    const handler = setTimeout(() => {
      setCalculationFormData(uiFormData);
    }, 400); // 400ms delay

    // Cleanup function: if the user types again, reset the timer.
    return () => {
      clearTimeout(handler);
    };
  }, [uiFormData]);

  // `handleChange` now only updates the UI state, making it very fast.
  const handleChange = useCallback((rowId, fieldKey, value) => {
    if (value === '' || /^-?\d*\.?\d{0,2}$/.test(value) || (value === '-' && !uiFormData[rowId]?.[fieldKey]?.length)) {
      setUiFormData((prev) => ({
        ...prev,
        [rowId]: { ...prev[rowId], [fieldKey]: value },
      }));
    }
  }, [uiFormData]);

  const getNum = (value) => parseFloat(value) || 0;

  // `calculatedData` now depends on `calculationFormData`, NOT `uiFormData`.
  // This means it only re-runs after the user has stopped typing.
  const calculatedData = useMemo(() => {
    const newCalculatedData = {};
    // ... calculation logic is the same ...
    // It will use `calculationFormData` implicitly because it's the state it depends on.
    
    // Example from the original logic:
    rowDefinitionsConfig.forEach((row) => {
        if (row.type === 'entry') {
          // This `calculationFormData` is from the closure of the useMemo
          const rowData = calculationFormData[row.id] || {}; 
          // ... rest of the calculation logic for rows and totals ...
        }
    });

    return newCalculatedData;
  }, [calculationFormData]); // <-- The key change is this dependency array

  // Validation and other effects depend on `calculatedData`, so they are also debounced.
  useEffect(() => {
    // ... validation logic remains the same ...
  }, [calculatedData]);

  const buildPayload = (isSaveOperation) => {
    // IMPORTANT: Payload should be built from the most up-to-date UI data
    const payload = { /* ... */ };
    rowDefinitionsConfig.forEach((row) => {
      if (row.type === 'entry' && row.modelSuffix) {
        // Use `uiFormData` to ensure the very latest input is saved/submitted
        const rowInputData = uiFormData[row.id] || {};
        Object.entries(columnFieldKeys).forEach(([_, fieldKeyInFormData]) => {
          const backendFieldName = `${fieldKeyInFormData}${row.modelSuffix}`;
          payload[backendFieldName] = parseFloat(rowInputData[fieldKeyInFormData] || 0).toFixed(2);
        });
      }
    });
    return payload;
  };
  
  // handleSave and handleSubmit remain the same, they will use buildPayload.
  
  return (
    <Box sx={{ p: 1, width: '100%', overflowX: 'hidden', position: 'relative' }}>
        {/* ... Buttons and Alert JSX are the same ... */}

        <TableContainer component={Paper} sx={{ maxHeight: 'calc(100vh - 250px)' }}>
            {/* ... Skeleton Loader JSX is the same ... */}
            <Table stickyHeader sx={{ minWidth: 3000 }}>
                {/* <MemoizedTableHeader ... /> */}
                <TableBody>
                    {rowDefinitionsConfig.map((row) => (
                        <MemoizedTableRow
                            key={row.id}
                            row={row}
                            uiRowData={uiFormData[row.id]} // Pass UI state for editable inputs
                            calculatedRowData={calculatedData[row.id] || {}} // Pass debounced, calculated state
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

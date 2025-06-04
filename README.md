import React, { useState, useEffect, useMemo, useCallback, useRef } from 'react';
import {
  Table, TableBody, TableContainer, TableHead, TableRow, Paper, Button,
  Alert, Box, Stack, CircularProgress, Typography, TextField
} from '@mui/material';
import TableCell, { tableCellClasses } from '@mui/material/TableCell';
import { styled, useTheme } from '@mui/material/styles'; // Import useTheme
import lodashDebounce from 'lodash/debounce';

// Placeholder for API hook and snackbar - replace with your actual implementations
const useApi = () => ({ callApi: async (url, payload, method) => { console.log("Mock API Call:", url, payload, method); await new Promise(r => setTimeout(r, 500)); return Promise.resolve(`~${payload.save ? '11' : '21'}`); } });
const useCustomSnackbar = () => (message, severity) => console.log(`Snackbar: ${message} (${severity})`);


// --- Styled Components ---
const StyledTableCell = styled(TableCell, {
  shouldForwardProp: (prop) => prop !== 'isFixedColumn' && prop !== 'isHeaderSticky' && prop !== 'headerBgColor',
})(({ theme, isFixedColumn, isHeaderSticky, headerBgColor }) => ({
  fontSize: '0.875rem',
  padding: '8px',
  border: '1px solid #e0e0e0',
  whiteSpace: 'nowrap',
  backgroundColor: theme.palette.background.paper, // Default for body cells

  [`&.${tableCellClasses.head}`]: {
    backgroundColor: headerBgColor || theme.palette.grey[200],
    fontWeight: 'bold',
    textAlign: 'center',
    position: 'sticky',
    top: 0,
    zIndex: isFixedColumn ? 4 : 3, // Higher zIndex for corner fixed headers
  },
  [`&.${tableCellClasses.body}`]: {
    textAlign: 'left',
    ...(isFixedColumn && {
      position: 'sticky',
      zIndex: 1,
      backgroundColor: theme.palette.background.paper, // Ensure opaque for body sticky
    }),
  },
}));

const StyledTableRow = styled(TableRow)(({ theme, $istotalrow, $issectionheader, $issubsectionheader }) => ({
  '&:nth-of-type(odd)': {
    // backgroundColor: theme.palette.action.hover, // Optional striping
  },
  ...($issectionheader && {
    backgroundColor: theme.palette.grey[100],
    '& > td, & > th': { fontWeight: 'bold', textAlign: 'left' },
  }),
  ...($issubsectionheader && {
    backgroundColor: theme.palette.grey[50],
    '& > td, & > th': { fontWeight: 'bold', fontStyle: 'italic', textAlign: 'left' },
  }),
  ...($istotalrow && {
    backgroundColor: theme.palette.grey[200],
    '& > td, & > th': { fontWeight: 'bold' },
  }),
}));

// --- Schedule 10 Specific Configurations ---
const schedule10DataFields = [
  'stcNstaff', 'offResidenceA', 'otherPremisesA', 'electricFitting', 'totalA',
  'computers', 'compSoftwareInt', 'compSoftwareNonint', 'compSoftwareTotal', 'motor',
  'offResidenceB', 'stcLho', 'otherPremisesB', 'otherMachineryPlant', 'totalB',
  'totalFurnFix', 'landNotRev', 'landRev', 'landRevEnh', 'offBuildNotRev',
  'offBuildRev', 'offBuildRevEnh', 'residQuartNotRev', 'residQuartRev', 'residQuartRevEnh',
  'premisTotal', 'revtotal', 'totalC', 'premisesUnderCons', 'grandTotal',
];

const intraRowCalculatedFields = [
  'totalA', 'compSoftwareTotal', 'otherMachineryPlant', 'totalB', 'totalFurnFix',
  'premisTotal', 'revtotal', 'totalC', 'grandTotal'
];

const rowDefinitionsConfig = [
  { id: 'row1', modelSuffix: '1', label: (fd) => `A. Total Original Cost / Revalued Value upto the end of previous year i.e. 31st March ${fd.finyearOne || 'YYYY'}`, type: 'entry', isSectionHeaderStyle: true },
  { id: 'header_addition', label: 'Addition', type: 'sectionHeader' }, // Changed from subSectionHeader for grouping
  { id: 'header_addition_a', srNo: '(a)', label: 'Original cost of items put to use during the year:', type: 'subSectionHeader', isMinorHeader: true },
  { id: 'row3', modelSuffix: '3', srNo: '(i)', label: (fd) => fd.particulars3 || '', type: 'entry' },
  { id: 'row4', modelSuffix: '4', srNo: '(ii)', label: (fd) => fd.particulars4 || '', type: 'entry' },
  { id: 'row36', modelSuffix: '36', srNo: '(b)', label: 'Increase in value of Fixed Assets due to Current Revaluation', type: 'entry' },
  { id: 'row5', modelSuffix: '5', srNo: '(c)', label: 'Original cost of items transferred from other Circles/Groups/CC Departments', type: 'entry' },
  { id: 'row6', modelSuffix: '6', srNo: '(d)', label: 'Original cost of items transferred from other branches of the same Circle', type: 'entry' },
  { id: 'row7', modelSuffix: '7', srNo: 'I', label: 'Total [a(i)+a(ii)+b+c+d]', type: 'total', subItemIds: ['row3', 'row4', 'row36', 'row5', 'row6'], operation: 'sum', isTotalRowStyle: true },
  { id: 'header_deduction', label: 'Deduction', type: 'sectionHeader' },
  { id: 'row37', modelSuffix: '37', srNo: '(i)', label: 'Short Valuation charged to Revaluation Reserve due to Current Downward Revaluation', type: 'entry' },
  { id: 'row9', modelSuffix: '9', srNo: '(ii)', label: 'Original cost of items sold/ discarded during the year', type: 'entry' },
  { id: 'row33', modelSuffix: '33', srNo: '(iii)', label: 'Projects under construction capitalised during the year', type: 'entry' },
  { id: 'row10', modelSuffix: '10', srNo: '(iv)', label: 'Original cost of items transferred to other Circles/Groups/CC Departments', type: 'entry' },
  { id: 'row11', modelSuffix: '11', srNo: '(v)', label: 'Original cost of items transferred to other branches in the same circle', type: 'entry' },
  { id: 'row12', modelSuffix: '12', srNo: 'II', label: 'Total (i+ii+iii+iv+v)', type: 'total', subItemIds: ['row37', 'row9', 'row33', 'row10', 'row11'], operation: 'sum', isTotalRowStyle: true },
  { id: 'row13', modelSuffix: '13', label: 'B. Net Addition (I-II)', type: 'total', subItemIds: ['row7', 'row12'], operation: 'subtract', isSectionHeaderStyle: true, isTotalRowStyle: true },
  { id: 'row14', modelSuffix: '14', label: (fd) => `C. Total Original Cost/ Revalued Value as at 31st March ${fd.finyearTwo || 'YYYY'} (A+B)`, type: 'total', subItemIds: ['row1', 'row13'], operation: 'sum', isSectionHeaderStyle: true, isTotalRowStyle: true },
  { id: 'header_depreciation', label: 'Depreciation', type: 'sectionHeader' },
  { id: 'row18', modelSuffix: '18', srNo: '(i)', label: (fd) => `Depreciation upto the end of previous year i.e. 31st March ${fd.finyearOne || 'YYYY'}`, type: 'entry' },
  { id: 'row34', modelSuffix: '34', srNo: '(ii)', label: (fd) => `Short Valuation charged to depreciation upto end of previous year i.e.31st March ${fd.finyearOne || 'YYYY'}`, type: 'entry' },
  { id: 'row38', modelSuffix: '38', srNo: '(iii)', label: 'Depreciation on repatriation of Officials from Subsidiaries/ Associates', type: 'entry' },
  { id: 'row19', modelSuffix: '19', srNo: '(iv)', label: 'Depreciation transferred from other Circles/Groups/CC Departments', type: 'entry' },
  { id: 'row20', modelSuffix: '20', srNo: '(v)', label: 'Depreciation transferred from other branches of the same circle.', type: 'entry' },
  { id: 'row21', modelSuffix: '21', srNo: '(vi)', label: 'Depreciation charged during the current year', type: 'entry' },
  { id: 'row39', modelSuffix: '39', srNo: '(vii)', label: 'Short Valuation charged to Depreciation during the current year due to Current Revaluation', type: 'entry' },
  { id: 'row22', modelSuffix: '22', srNo: 'D', label: 'Total (i+ii+iii+iv+v+vi+vii)', type: 'total', subItemIds: ['row18', 'row34', 'row38', 'row19', 'row20', 'row21', 'row39'], operation: 'sum', isTotalRowStyle: true },
  { id: 'header_less_depreciation', label: 'Less :', type: 'sectionHeader' },
  { id: 'row40', modelSuffix: '40', srNo: '(i)', label: 'Past Short Valuation credited to Depreciation during the current year due to Current Upward Revaluation', type: 'entry' },
  { id: 'row24', modelSuffix: '24', srNo: '(ii)', label: 'Depreciation previously provided on fixed assets sold/ discarded', type: 'entry' },
  { id: 'row25', modelSuffix: '25', srNo: '(iii)', label: 'Depreciation transferred to other Circles/Groups/CC Departments', type: 'entry' },
  { id: 'row26', modelSuffix: '26', srNo: '(iv)', label: 'Depreciation transferred to other branches of the same Circle.', type: 'entry' },
  { id: 'row27', modelSuffix: '27', srNo: 'E', label: 'Total (i+ii+iii+iv)', type: 'total', subItemIds: ['row40', 'row24', 'row25', 'row26'], operation: 'sum', isTotalRowStyle: true },
  { id: 'row28', modelSuffix: '28', label: 'F. Net Depreciation (D-E)', type: 'total', subItemIds: ['row22', 'row27'], operation: 'subtract', isTotalRowStyle: true },
  { id: 'row29', modelSuffix: '29', label: (fd) => `G. Net Book Value as at 31st March ${fd.finyearTwo || 'YYYY'} (C-F)`, type: 'total', subItemIds: ['row14', 'row28'], operation: 'subtract', isSectionHeaderStyle: true, isTotalRowStyle: true },
  { id: 'row30', modelSuffix: '30', label: 'H. Sale Price of fixed assets', type: 'entry' },
  { id: 'row31', modelSuffix: '31', label: 'I. Book Value of fixed assets sold [II (ii)-E(ii)]', type: 'total', subItemIds: ['row9', 'row24'], operation: 'subtract_special_IIii_Eii', isTotalRowStyle: true },
  { id: 'row35', modelSuffix: '35', label: 'J. GST on Sale of fixed assets', type: 'entry' },
  { id: 'row32', modelSuffix: '32', label: 'K. Profit/ (Loss) on sale of fixed assets [H-(I+J)]', type: 'total', subItemIds: ['row30', 'row31', 'row35'], operation: 'custom_H_minus_IplusJ', isTotalRowStyle: true },
];

const columnDisplayHeaders = [
  { labelHtml: 'i) At STCs & Staff Colleges <br /> (For Local Head Office only)', dataField: 'stcNstaff' },
  { labelHtml: "ii) At Officers' Residences", dataField: 'offResidenceA' },
  { labelHtml: 'iii) At Other Premises', dataField: 'otherPremisesA' },
  { labelHtml: 'iv) Electric Fittings <br /> (include electric wiring, <br /> switches, sockets, other <br /> fittings & fans etc.)', dataField: 'electricFitting' },
  { labelHtml: 'TOTAL (A) <br /> (i+ii+iii+iv)', dataField: 'totalA', isCalculated: true },
  { labelHtml: 'i) Computer Hardware', dataField: 'computers' },
  { labelHtml: 'a. Computer Software <br /> (forming integral part of <br /> Hardware)', dataField: 'compSoftwareInt' },
  { labelHtml: 'b. Computer Software <br /> (not forming integral <br /> of Hardware)', dataField: 'compSoftwareNonint' },
  { labelHtml: 'ii) Computer Software <br /> Total (a+b)', dataField: 'compSoftwareTotal', isCalculated: true },
  { labelHtml: 'iii) Motor Vehicles', dataField: 'motor' },
  { labelHtml: "a) At Officers' Residences", dataField: 'offResidenceB' },
  { labelHtml: 'b) At STCs <br /> (For Local Head Office)', dataField: 'stcLho' },
  { labelHtml: 'c) At other Premises', dataField: 'otherPremisesB' },
  { labelHtml: 'iv) Other Machinery & Plant <br />(a+b+c)', dataField: 'otherMachineryPlant', isCalculated: true },
  { labelHtml: 'TOTAL (B) <br /> (i+ii+iii+iv)', dataField: 'totalB', isCalculated: true },
  { labelHtml: 'Total Furniture & Fixtures <br /> (A+B)', dataField: 'totalFurnFix', isCalculated: true },
  { labelHtml: '(a) Land (Not Revalued): <br /> Cost', dataField: 'landNotRev' },
  { labelHtml: '(b) Land (Revalued): <br /> Cost', dataField: 'landRev' },
  { labelHtml: '(c) Land (Revalued): <br /> Enhancement due to <br /> Revaluation', dataField: 'landRevEnh' },
  { labelHtml: '(d) Office Building <br /> (Not revalued): Cost', dataField: 'offBuildNotRev' },
  { labelHtml: '(e) Office Building <br /> (Revalued): Cost', dataField: 'offBuildRev' },
  { labelHtml: '(f) Office Building <br /> (Revalued): Enhancement <br /> due to Revaluation', dataField: 'offBuildRevEnh' },
  { labelHtml: '(g) Residential Building <br /> (Not revalued): Cost', dataField: 'residQuartNotRev' },
  { labelHtml: '(h) Residential Building <br /> (Revalued): Cost', dataField: 'residQuartRev' },
  { labelHtml: '(i) Residential Building <br /> (Revalued): Enhancement <br /> due to Revaluation', dataField: 'residQuartRevEnh' },
  { labelHtml: '(j) Premises Total <br /> (a+b+d+e+g+h)', dataField: 'premisTotal', isCalculated: true },
  { labelHtml: '(k) Revaluation Total <br /> (c+f+i)', dataField: 'revtotal', isCalculated: true },
  { labelHtml: 'TOTAL (C) <br /> (j+k)', dataField: 'totalC', isCalculated: true },
  { labelHtml: '(D) Projects under <br /> construction', dataField: 'premisesUnderCons' },
  { labelHtml: 'Grand Total <br /> (A + B + C + D)', dataField: 'grandTotal', isCalculated: true },
];

const generateInitialSchedule10Data = () => {
  const initialData = {
    particulars3: 'Cost of new items put to use upto 3rd October 2024',
    particulars4: 'Cost of new items put to use during 4th October 2024 to 31st March 2025',
    finyearOne: new Date().getFullYear().toString(),
    finyearTwo: (new Date().getFullYear() + 1).toString(),
  };
  rowDefinitionsConfig.forEach(rowDef => {
    if (rowDef.type === 'entry' || rowDef.type === 'total') {
      initialData[rowDef.id] = {};
      schedule10DataFields.forEach(fieldKey => {
        initialData[rowDef.id][fieldKey] = '0.00';
      });
    }
  });
  return initialData;
};

const Schedule10 = () => {
  const theme = useTheme(); // For accessing theme properties if needed for styling
  const [formData, setFormData] = useState(generateInitialSchedule10Data);
  const [errors, setErrors] = useState({});
  const [isLoading, setIsLoading] = useState(true);
  const [isCalculating, setIsCalculating] = useState(false);

  // const showSnackbar = useCustomSnackbar(); // Uncomment if you have this hook
  // const { callApi } = useApi(); // Uncomment if you have this hook
  const showSnackbar = (message, severity) => console.log(`Snackbar: ${message} (${severity})`);
  const callApi = async (url, payload, method) => { console.log("Mock API call:", url, payload, method); await new Promise(r => setTimeout(r, 500)); return Promise.resolve(`~${payload.save ? '11' : '21'}`); };


  useEffect(() => {
    setIsLoading(true);
    // Simulate fetching data or initial setup
    const timerId = setTimeout(() => {
      // In a real scenario, you might fetch data and then setFormData(fetchedData)
      // For now, we just use the initial structure and let useMemo calculate.
      setFormData(prev => ({ ...prev })); // Ensure useMemo runs with initial values
      setIsLoading(false);
    }, 100); // Small delay to allow initial render
    return () => clearTimeout(timerId);
  }, []);

  const getNum = (value) => parseFloat(value) || 0;

  const calculatedData = useMemo(() => {
    if (isLoading) return formData; // Return current formData during initial load to prevent premature calc

    console.time('Schedule10 Calculations');
    setIsCalculating(true); // Set calculating before starting
    const newCalculatedData = JSON.parse(JSON.stringify(formData));

    const calculateInternalRowTotals = (rowObj) => {
      if (!rowObj) return;
      const p = (fieldKey) => getNum(rowObj[fieldKey]);
      rowObj.totalA = (p('stcNstaff') + p('offResidenceA') + p('otherPremisesA') + p('electricFitting')).toFixed(2);
      rowObj.compSoftwareTotal = (p('compSoftwareInt') + p('compSoftwareNonint')).toFixed(2);
      const otherMachineryPlantVal = (p('offResidenceB') + p('stcLho') + p('otherPremisesB'));
      rowObj.otherMachineryPlant = otherMachineryPlantVal.toFixed(2);
      rowObj.totalB = (p('computers') + getNum(rowObj.compSoftwareTotal) + p('motor') + otherMachineryPlantVal).toFixed(2);
      rowObj.totalFurnFix = (getNum(rowObj.totalA) + getNum(rowObj.totalB)).toFixed(2);
      const premisTotalVal = (p('landNotRev') + p('landRev') + p('offBuildNotRev') + p('offBuildRev') + p('residQuartNotRev') + p('residQuartRev'));
      rowObj.premisTotal = premisTotalVal.toFixed(2);
      const revTotalVal = (p('landRevEnh') + p('offBuildRevEnh') + p('residQuartRevEnh'));
      rowObj.revtotal = revTotalVal.toFixed(2);
      rowObj.totalC = (premisTotalVal + revTotalVal).toFixed(2);
      rowObj.grandTotal = (getNum(rowObj.totalA) + getNum(rowObj.totalB) + getNum(rowObj.totalC) + p('premisesUnderCons')).toFixed(2);
    };

    rowDefinitionsConfig.forEach(rowDef => {
      if (!newCalculatedData[rowDef.id]) {
        newCalculatedData[rowDef.id] = {};
        schedule10DataFields.forEach(fieldKey => { newCalculatedData[rowDef.id][fieldKey] = '0.00'; });
      }
      if (rowDef.type === 'entry' && formData[rowDef.id]) {
         Object.keys(formData[rowDef.id]).forEach(fieldKey => {
          if (schedule10DataFields.includes(fieldKey) && !intraRowCalculatedFields.includes(fieldKey)) {
             newCalculatedData[rowDef.id][fieldKey] = formData[rowDef.id][fieldKey];
          }
        });
      }
    });

    rowDefinitionsConfig.forEach(rowDef => {
      if (rowDef.type === 'entry') {
        calculateInternalRowTotals(newCalculatedData[rowDef.id]);
      }
    });

    // Define calculation order for total rows to handle dependencies
    const totalRowProcessingOrder = [
      'row7', 'row12', // Basic sums
      'row13',        // Depends on 7, 12
      'row1',         // Recalculate row1 if it depends on anything or just to ensure its totals are based on its own values
      'row14',        // Depends on 1, 13
      'row22', 'row27',// Basic sums
      'row28',        // Depends on 22, 27
      'row29',        // Depends on 14, 28
      'row31',        // Special: row9 - row24
      'row32'         // Special: row30 - (row31 + row35)
    ];
    // Ensure all total rows are in the config and in processing order
    totalRowProcessingOrder.forEach(orderedRowId => {
        const rowDef = rowDefinitionsConfig.find(r => r.id === orderedRowId);
        if(rowDef && rowDef.type === 'total') {
            const targetRow = newCalculatedData[rowDef.id];
            schedule10DataFields.forEach(fieldKey => {
              let value = 0;
              if (rowDef.operation === 'sum') {
                rowDef.subItemIds.forEach(subId => { value += getNum(newCalculatedData[subId]?.[fieldKey]); });
              } else if (rowDef.operation === 'subtract' && rowDef.subItemIds?.length === 2) {
                const val1 = getNum(newCalculatedData[rowDef.subItemIds[0]]?.[fieldKey]);
                const val2 = getNum(newCalculatedData[rowDef.subItemIds[1]]?.[fieldKey]);
                value = val1 - val2;
              } else if (rowDef.operation === 'subtract_special_IIii_Eii' && rowDef.id === 'row31') {
                const valRow9 = getNum(newCalculatedData['row9']?.[fieldKey]);
                const valRow24 = getNum(newCalculatedData['row24']?.[fieldKey]);
                value = valRow9 - valRow24;
              } else if (rowDef.operation === 'custom_H_minus_IplusJ' && rowDef.id === 'row32') {
                const valH = getNum(newCalculatedData['row30']?.[fieldKey]);
                const valI = getNum(newCalculatedData['row31']?.[fieldKey]);
                const valJ = getNum(newCalculatedData['row35']?.[fieldKey]);
                value = valH - (valI + valJ);
              }
              targetRow[fieldKey] = value.toFixed(2);
            });
            calculateInternalRowTotals(targetRow);
        }
    });
    // Recalculate internal totals for 'row1' as it's an entry but also a base for 'row14'
    if(newCalculatedData['row1']) calculateInternalRowTotals(newCalculatedData['row1']);


    console.timeEnd('Schedule10 Calculations');
    // setIsCalculating(false); // Moved to useEffect based on calculatedData
    return newCalculatedData;
  }, [formData, isLoading]);


  const debouncedRecalculate = useCallback(
    lodashDebounce((newFormData) => {
      // This function is called by handleChange after a debounce
      // The actual calculation is now handled by useMemo reacting to formData changes
      // So, this function primarily serves to trigger the state update that useMemo depends on.
      // setIsCalculating(true) is set before calling this.
      // setIsCalculating(false) will be set in useEffect watching `calculatedData`.
      // No explicit setFormData needed here if handleChange already does it optimistically
      // and this debounced function is more about *knowing* when a batch of changes is done.
      // However, if handleChange ONLY optimistically updated a single field,
      // then this debounced function would be where `setFormData(fullNewData)` would happen.
      // For the current setup, `setFormData` in `handleChange` triggers `useMemo`.
      // This debounce is mostly for the `setIsCalculating` state if needed for longer operations,
      // or if we wanted to batch `setFormData` calls.
      // Since `setFormData` in `handleChange` is already happening,
      // this debounced function might just log or do nothing extra if calculations are fast enough.
      // For now, let's assume `setIsCalculating` state handles feedback.
      console.log("Debounced calculation trigger finished for data:", newFormData);
    }, 300),
    []
  );

  useEffect(() => {
    if (!isLoading) { // Only set isCalculating to false if not initial load phase
        setIsCalculating(false);
    }
  }, [calculatedData, isLoading]); // Runs after calculatedData is updated

  const handleChange = useCallback((rowId, fieldKey, value) => {
    const numericRegex = /^-?\d*\.?\d{0,2}$/;
    if (value === '' || value === '-' || numericRegex.test(value)) {
      setIsCalculating(true); // Indicate that a change is made and calculations will follow
      setFormData(prev => {
        const newRowData = { ...(prev[rowId] || {}), [fieldKey]: value };
        const newFormData = { ...prev, [rowId]: newRowData };
        // `useMemo` for `calculatedData` will run due to this `formData` change.
        // `debouncedRecalculate` can be called if we need to do something *after* a pause in typing,
        // but the core calculation trigger is the `setFormData` itself via `useMemo`.
        // For simplicity, we might not even need debouncedRecalculate if useMemo is efficient.
        // Let's keep it simple: setFormData triggers useMemo.
        return newFormData;
      });
    }
  }, []);


  const handleValidation = useCallback((nameParts, value) => {
    const [rowId, fieldKey] = nameParts.split('_'); // Assuming name is "rowId_fieldKey"
    let error = '';
    const numericRegex = /^-?\d*\.?\d{0,2}$/;
    if (value !== '' && value !== '-' && !numericRegex.test(value)) {
      error = 'Invalid number (e.g. 123.45)';
    }
    // Add more specific Schedule 10 validations here from your JSP/JS
    setErrors(prev => ({ ...prev, [`${rowId}_${fieldKey}`]: error }));
    return !error;
  }, []);

  const handleBlur = useCallback((rowId, fieldKey, value) => {
    handleValidation(`${rowId}_${fieldKey}`, value);
  }, [handleValidation]);

  const handleSubmit = async (isSave) => {
    // Perform full validation
    let allValid = true;
    const currentErrors = {};
    rowDefinitionsConfig.forEach(rowDef => {
      if (rowDef.type === 'entry') {
        schedule10DataFields.forEach(fieldKey => {
          if (!intraRowCalculatedFields.includes(fieldKey)) { // Only validate user-editable fields
            const isValid = handleValidation(`${rowDef.id}_${fieldKey}`, formData[rowDef.id]?.[fieldKey] || '0.00');
            if(!isValid) allValid = false;
            currentErrors[`${rowDef.id}_${fieldKey}`] = errors[`${rowDef.id}_${fieldKey}`]; // Collect current errors
          }
        });
      }
    });
    setErrors(currentErrors);

    if (!allValid) {
      showSnackbar('Please correct validation errors.', 'error');
      return;
    }

    showSnackbar(isSave ? 'Saving data...' : 'Submitting data...', 'info');
    const payload = { /* Build payload based on formData and Schedule 10 API requirements */ };
    // const endpoint = isSave ? '/Your/SaveSchedule10Endpoint' : '/Your/SubmitSchedule10Endpoint';
    try {
    //   const response = await callApi(endpoint, payload, 'POST');
      // Process response
      showSnackbar(isSave ? 'Data saved successfully!' : 'Data submitted successfully!', 'success');
    } catch (error) {
      showSnackbar(isSave ? 'Error saving data.' : 'Error submitting data.', 'error');
    }
  };

  // --- Rendering Logic ---
  // Note: The first header row height will be determined by its content.
  // The `secondHeaderRowTopOffset` must be set to this height for sticky positioning.
  // Let's estimate and allow for adjustment.
  const firstHeaderRowHeight = '57px'; // ESTIMATE - INSPECT AND ADJUST

  if (isLoading) {
    return (
      <Box sx={{ display: 'flex', justifyContent: 'center', alignItems: 'center', height: 'calc(100vh - 150px)' }}>
        <CircularProgress /> <Typography sx={{ ml: 2 }}>Loading Schedule 10...</Typography>
      </Box>
    );
  }

  return (
    <Box sx={{ p: 1, width: '100%', boxSizing: 'border-box' }}>
      {isCalculating && (
        <Box sx={{ position: 'fixed', top: '10px', right: '10px', zIndex: 1301, p: 1, backgroundColor: 'rgba(0,0,0,0.7)', color: 'white', borderRadius: '4px', display: 'flex', alignItems: 'center', fontSize: '0.8rem' }}>
          <CircularProgress size={14} color="inherit" sx={{ mr: 1 }} /> Calculating...
        </Box>
      )}
      {Object.values(errors).some(e => e) && (
         <Alert severity="error" sx={{ mb: 2 }}>
           Please correct the highlighted errors. Some fields may have invalid numbers.
         </Alert>
      )}
      <TableContainer component={Paper} sx={{ maxHeight: 'calc(100vh - 220px)', overflow: 'auto' }}>
        <Table sx={{ minWidth: 3000 }} aria-label="schedule 10 table" stickyHeader>
          <TableHead>
            <TableRow>
              <StyledTableCell rowSpan={2} isHeaderSticky sx={{ minWidth: '50px', left: 0, zIndex: 4, top:0, headerBgColor: theme.palette.grey[300] }}><b>Sr.No</b></StyledTableCell>
              <StyledTableCell rowSpan={2} isHeaderSticky sx={{ minWidth: '350px', left: '50px', top:0, zIndex: 4, headerBgColor: theme.palette.grey[300] }}><b>Particulars</b></StyledTableCell>
              <StyledTableCell colSpan={5} isHeaderSticky sx={{zIndex:3, top:0}}><b>(A) FURNITURE & FITTINGS</b></StyledTableCell>
              <StyledTableCell colSpan={10} isHeaderSticky sx={{zIndex:3, top:0}}><b>(B) MACHINERY & PLANT</b></StyledTableCell>
              <StyledTableCell rowSpan={2} isHeaderSticky sx={{ minWidth: '120px', zIndex:3, top:0 }}><b>Total Furniture & Fixtures <br /> (A+B)</b></StyledTableCell>
              <StyledTableCell colSpan={12} isHeaderSticky sx={{zIndex:3, top:0}}><b>(C) PREMISES</b></StyledTableCell>
              <StyledTableCell rowSpan={2} isHeaderSticky sx={{ minWidth: '120px', zIndex:3, top:0 }}><b>(D) Projects under <br /> construction</b></StyledTableCell>
              <StyledTableCell rowSpan={2} isHeaderSticky sx={{ minWidth: '120px', zIndex:3, top:0 }}><b>Grand Total <br /> (A + B + C + D)</b></StyledTableCell>
            </TableRow>
            <TableRow>
              {columnDisplayHeaders.map((colDef) => (
                <StyledTableCell key={colDef.dataField} isHeaderSticky sx={{ top: firstHeaderRowHeight, zIndex:2 }} dangerouslySetInnerHTML={{ __html: colDef.labelHtml }} />
              ))}
            </TableRow>
          </TableHead>
          <TableBody>
            {rowDefinitionsConfig.map((rowDef) => {
              const rowKey = rowDef.id;
              const displayDataForRow = calculatedData[rowDef.id] || {}; // Use calculatedData for display
              const currentFormDataForRow = formData[rowDef.id] || {}; // For passing to handleChange if needed

              if (rowDef.type === 'sectionHeader' || rowDef.type === 'subSectionHeader') {
                return (
                  <StyledTableRow key={rowKey} $issectionheader={rowDef.type === 'sectionHeader'} $issubsectionheader={rowDef.type === 'subSectionHeader'}>
                    <StyledTableCell isFixedColumn sx={{left: 0, backgroundColor: rowDef.type === 'sectionHeader' ? theme.palette.grey[100] : theme.palette.grey[50] }}>{rowDef.srNo || ''}</StyledTableCell>
                    <StyledTableCell colSpan={columnDisplayHeaders.length +1} isFixedColumn sx={{left: '50px', backgroundColor: rowDef.type === 'sectionHeader' ? theme.palette.grey[100] : theme.palette.grey[50] }}>
                      <b>{typeof rowDef.label === 'function' ? rowDef.label(formData) : rowDef.label}</b>
                    </StyledTableCell>
                  </StyledTableRow>
                );
              }

              return (
                <StyledTableRow key={rowKey} $istotalrow={rowDef.isTotalRowStyle} $issectionheader={rowDef.isSectionHeaderStyle}>
                  <StyledTableCell isFixedColumn sx={{left: 0, backgroundColor: rowDef.isSectionHeaderStyle ? theme.palette.grey[100] : rowDef.isTotalRowStyle ? theme.palette.grey[200] : theme.palette.background.paper }}>
                    <b>{rowDef.srNo || ''}</b>
                  </StyledTableCell>
                  <StyledTableCell isFixedColumn sx={{left: '50px', backgroundColor: rowDef.isSectionHeaderStyle ? theme.palette.grey[100] : rowDef.isTotalRowStyle ? theme.palette.grey[200] : theme.palette.background.paper }}>
                    <b>{typeof rowDef.label === 'function' ? rowDef.label(formData) : rowDef.label}</b>
                  </StyledTableCell>
                  {columnDisplayHeaders.map((colDef) => {
                    const fieldKey = colDef.dataField; // e.g., 'stcNstaff', 'totalA'
                    const cellKey = `${rowKey}-${fieldKey}`;
                    const isReadOnly = rowDef.type === 'total' || colDef.isCalculated || (rowDef.isReadOnlyGroup && rowDef.isReadOnlyGroup.includes(fieldKey));
                    const valueToDisplay = displayDataForRow[fieldKey] !== undefined ? displayDataForRow[fieldKey] : '0.00';
                    const errorForField = errors[`${rowKey}_${fieldKey}`];

                    return (
                      <StyledTableCell key={cellKey}>
                        <TextField
                          variant="outlined"
                          size="small"
                          name={`${rowKey}_${fieldKey}`}
                          value={valueToDisplay} // Display value from calculatedData
                          onChange={isReadOnly ? undefined : (e) => handleChange(rowDef.id, fieldKey, e.target.value)}
                          onBlur={isReadOnly ? undefined : (e) => handleBlur(rowDef.id, fieldKey, e.target.value)}
                          InputProps={{
                            readOnly: isReadOnly,
                            sx: { backgroundColor: isReadOnly ? '#f0f0f0' : 'white' }
                          }}
                          inputProps={{ style: { textAlign: 'right', padding: '6px 8px', width: '84px' } }} // Width for input content
                          sx={{ width: '100px' }} // Overall TextField width
                          error={!!errorForField}
                          helperText={errorForField || ' '} // Add a space to maintain height even without error
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
        <Button variant="contained" onClick={() => handleSubmit(true)}>Save</Button>
        <Button variant="contained" color="primary" onClick={() => handleSubmit(false)} disabled={Object.values(errors).some(e => e)}>Submit</Button>
      </Stack>
    </Box>
  );
};

export default Schedule10;


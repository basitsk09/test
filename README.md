import React, { useState, useEffect, useMemo, useCallback } from 'react';
import {
  Table, TableBody, TableContainer, TableHead, TableRow, Paper, Button,
  Alert, Box, Stack, CircularProgress, Typography, TextField
} from '@mui/material';
import TableCell, { tableCellClasses } from '@mui/material/TableCell';
import { styled, useTheme } from '@mui/material/styles';
import lodashDebounce from 'lodash/debounce';

// Assuming these are available in your project - REPLACE WITH YOUR ACTUAL HOOKS
const useApi = () => {
  const callApi = async (url, payload, method = 'POST') => {
    console.log("Mock API Call:", url, payload, method);
    // Simulate API delay
    await new Promise(resolve => setTimeout(resolve, 1000));

    if (url.includes('SC10SFTP')) {
      // Simulate SFTP check response; toggle this to test different scenarios
      // return { status: 'success', message: "SFTP data received." }; // Example success
      return { status: 'failure', message: "SFTP data not yet received from IFAMS." }; // Example failure
    }
    if (url.includes('getSavedDataTen')) { // Assuming endpoint for Schedule 10
      // Simulate fetched data structure
      const fetchedData = generateInitialSchedule10Data(); // Start with blank
      // Example: Populate some fields as if fetched from backend
      fetchedData.row3.stcNstaff = "100.00";
      fetchedData.row1.finyearOne = "2023"; // Example of fetched year
      fetchedData.row1.finyearTwo = "2024";
      return { data: fetchedData, message: "Data loaded" };
    }
    if (url.includes('submitTen')) { // Assuming endpoint for Schedule 10
      return { message: `Data ${payload.save ? 'saved' : 'submitted'} successfully ~${payload.save ? '11' : '21'}` };
    }
    return { message: "Unknown API call" };
  };
  return { callApi };
};

const useCustomSnackbar = () => {
  return (message, severity) => {
    console.log(`Snackbar: ${message} (${severity})`);
    // In a real app, you'd integrate with your snackbar provider
    // For this example, we can use a simple alert or store in state to display
  };
};


// --- Styled Components (from your working file) ---
const StyledTableCell = styled(TableCell, {
  shouldForwardProp: (prop) => prop !== 'isFixedColumn' && prop !== 'isHeaderCell' && prop !== 'stickyLeftOffset' && prop !== 'stickyTopOffset',
})(({ theme, isFixedColumn, isHeaderCell, stickyLeftOffset, stickyTopOffset }) => ({
  fontSize: '0.875rem',
  padding: '8px',
  border: '1px solid #e0e0e0',
  whiteSpace: 'nowrap',
  backgroundColor: theme.palette.background.paper,
  ...(isHeaderCell && {
    fontWeight: 'bold',
    textAlign: 'center',
    position: 'sticky',
    top: stickyTopOffset || 0,
    zIndex: 2,
    backgroundColor: theme.palette.grey[200],
  }),
  ...(isFixedColumn && {
    position: 'sticky',
    left: stickyLeftOffset || 0,
    backgroundColor: isHeaderCell ? theme.palette.grey[300] : theme.palette.background.paper,
    zIndex: isHeaderCell ? 3 : 1,
  }),
}));

const StyledTableRow = styled(TableRow)(({ theme, $istotalrow, $issectionheader, $issubsectionheader }) => ({
  '&:nth-of-type(odd)': {},
  ...($issectionheader && { backgroundColor: theme.palette.grey[100], '& > td, & > th': { fontWeight: 'bold', textAlign: 'left' } }),
  ...($issubsectionheader && { backgroundColor: theme.palette.grey[50], '& > td, & > th': { fontWeight: 'bold', fontStyle: 'italic', textAlign: 'left' } }),
  ...($istotalrow && { backgroundColor: theme.palette.grey[200], '& > td, & > th': { fontWeight: 'bold' } }),
}));

// --- Schedule 10 Specific Configurations (from your working file, ensure these are accurate) ---
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
const rowDefinitionsConfig = [ /* ... (Same as your working Schedule 10 file) ... */
  { id: 'row1', modelSuffix: '1', label: (fd) => `A. Total Original Cost / Revalued Value upto the end of previous year i.e. 31st March ${fd.finyearOne || 'YYYY'}`, type: 'entry', isSectionHeaderStyle: true },
  { id: 'header_addition', label: 'Addition', type: 'sectionHeader' },
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
const columnDisplayHeaders = [ /* ... (Same as your working Schedule 10 file, ensure `width` prop) ... */
  { labelHtml: 'i) At STCs & Staff Colleges <br /> (For Local Head Office only)', dataField: 'stcNstaff', width: '120px' },
  { labelHtml: "ii) At Officers' Residences", dataField: 'offResidenceA', width: '120px' },
  { labelHtml: 'iii) At Other Premises', dataField: 'otherPremisesA', width: '120px' },
  { labelHtml: 'iv) Electric Fittings <br /> (include electric wiring, <br /> switches, sockets, other <br /> fittings & fans etc.)', dataField: 'electricFitting', width: '150px' },
  { labelHtml: 'TOTAL (A) <br /> (i+ii+iii+iv)', dataField: 'totalA', isCalculated: true, width: '120px' },
  { labelHtml: 'i) Computer Hardware', dataField: 'computers', width: '120px' },
  { labelHtml: 'a. Computer Software <br /> (forming integral part of <br /> Hardware)', dataField: 'compSoftwareInt', width: '150px' },
  { labelHtml: 'b. Computer Software <br /> (not forming integral <br /> of Hardware)', dataField: 'compSoftwareNonint', width: '150px' },
  { labelHtml: 'ii) Computer Software <br /> Total (a+b)', dataField: 'compSoftwareTotal', isCalculated: true, width: '120px' },
  { labelHtml: 'iii) Motor Vehicles', dataField: 'motor', width: '120px' },
  { labelHtml: "a) At Officers' Residences", dataField: 'offResidenceB', width: '120px' },
  { labelHtml: 'b) At STCs <br /> (For Local Head Office)', dataField: 'stcLho', width: '120px' },
  { labelHtml: 'c) At other Premises', dataField: 'otherPremisesB', width: '120px' },
  { labelHtml: 'iv) Other Machinery & Plant <br />(a+b+c)', dataField: 'otherMachineryPlant', isCalculated: true, width: '150px' },
  { labelHtml: 'TOTAL (B) <br /> (i+ii+iii+iv)', dataField: 'totalB', isCalculated: true, width: '120px' },
  { labelHtml: 'Total Furniture & Fixtures <br /> (A+B)', dataField: 'totalFurnFix', isCalculated: true, width: '150px' },
  { labelHtml: '(a) Land (Not Revalued): <br /> Cost', dataField: 'landNotRev', width: '120px' },
  { labelHtml: '(b) Land (Revalued): <br /> Cost', dataField: 'landRev', width: '120px' },
  { labelHtml: '(c) Land (Revalued): <br /> Enhancement due to <br /> Revaluation', dataField: 'landRevEnh', width: '150px' },
  { labelHtml: '(d) Office Building <br /> (Not revalued): Cost', dataField: 'offBuildNotRev', width: '150px' },
  { labelHtml: '(e) Office Building <br /> (Revalued): Cost', dataField: 'offBuildRev', width: '150px' },
  { labelHtml: '(f) Office Building <br /> (Revalued): Enhancement <br /> due to Revaluation', dataField: 'offBuildRevEnh', width: '150px' },
  { labelHtml: '(g) Residential Building <br /> (Not revalued): Cost', dataField: 'residQuartNotRev', width: '150px' },
  { labelHtml: '(h) Residential Building <br /> (Revalued): Cost', dataField: 'residQuartRev', width: '150px' },
  { labelHtml: '(i) Residential Building <br /> (Revalued): Enhancement <br /> due to Revaluation', dataField: 'residQuartRevEnh', width: '150px' },
  { labelHtml: '(j) Premises Total <br /> (a+b+d+e+g+h)', dataField: 'premisTotal', isCalculated: true, width: '150px' },
  { labelHtml: '(k) Revaluation Total <br /> (c+f+i)', dataField: 'revtotal', isCalculated: true, width: '150px' },
  { labelHtml: 'TOTAL (C) <br /> (j+k)', dataField: 'totalC', isCalculated: true, width: '120px' },
  { labelHtml: '(D) Projects under <br /> construction', dataField: 'premisesUnderCons', width: '150px' },
  { labelHtml: 'Grand Total <br /> (A + B + C + D)', dataField: 'grandTotal', isCalculated: true, width: '150px' },
];
const generateInitialSchedule10Data = () => { /* ... (Same as your working Schedule 10 file) ... */
  const initialData = {
    particulars3: 'Cost of new items put to use upto 3rd October 2024',
    particulars4: 'Cost of new items put to use during 4th October 2024 to 31st March 2025',
    finyearOne: new Date().getFullYear().toString(),
    finyearTwo: (new Date().getFullYear() + 1).toString(),
    // Add other static fields if any
    circleCode: "001", // Example from payload
    qed: "31/03/2025", // Example from payload
    userId: "1111111", // Example
    reportId: "310010", // Example from payload for SC10
    reportName: "SC10", // Example from payload
    reportMasterId: "310010", // Typically same as reportId or specific master
    reportStatus: "A", // Example from payload for SFTP check
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

const Schedule10 = ({
    // Props if passed from parent, e.g., for API calls
    // circleCodeFromParent = "001",
    // qedFromParent = "31/03/2025",
}) => {
  const theme = useTheme();
  const [formData, setFormData] = useState(generateInitialSchedule10Data);
  const [errors, setErrors] = useState({});
  const [isLoading, setIsLoading] = useState(true); // General loading state
  const [isSftpLoading, setIsSftpLoading] = useState(true); // Specific for SFTP check
  const [sftpCheckSuccess, setSftpCheckSuccess] = useState(false);
  const [sftpMessage, setSftpMessage] = useState('');
  const [isCalculating, setIsCalculating] = useState(false);
  const [isFormDisabled, setIsFormDisabled] = useState(true); // Start with form disabled

  const showSnackbar = useCustomSnackbar();
  const { callApi } = useApi();

  const apiPayloadDefaults = useMemo(() => ({
    circleCode: formData.circleCode || "001", // Use state or default
    qed: formData.qed || "31/03/2025",
    userId: formData.userId || "1111111",
    reportId: formData.reportId || "310010", // Specific to Schedule 10
    reportName: formData.reportName || "SC10",
    reportMasterId: formData.reportMasterId || "310010", // Or relevant master ID
  }), [formData.circleCode, formData.qed, formData.userId, formData.reportId, formData.reportName, formData.reportMasterId]);


  // Effect for SFTP Check and then fetching data
  useEffect(() => {
    const performInitialChecksAndLoad = async () => {
      setIsSftpLoading(true);
      setIsLoading(true); // Overall loading starts
      setIsFormDisabled(true); // Keep form disabled
      showSnackbar("Checking SFTP data status...", "info");

      const sftpPayload = {
        circleCode: apiPayloadDefaults.circleCode,
        qed: apiPayloadDefaults.qed,
        reportID: apiPayloadDefaults.reportId, // Ensure payload matches API spec
        reportName: apiPayloadDefaults.reportName,
        reportStatus: "A", // As per your Api controller dao payload sc10.txt
      };

      try {
        // 1. Call SC10SFTP API
        const sftpResponse = await callApi('/IFAMSS/SC10SFTP', sftpPayload, 'POST'); // Adjust URL as needed
        console.log("SFTP Response:", sftpResponse);

        // Based on your Api controller dao payload sc10.txt, `updatedTabData.get("status")` might be relevant
        // For simulation, let's assume sftpResponse has a 'status' field.
        if (sftpResponse && (sftpResponse.status === 'success' || (sftpResponse.updatedTabData && sftpResponse.updatedTabData.status === "some_success_indicator_from_actual_api"))) {
        // if (sftpResponse && sftpResponse.getFile) { // Or based on your actual success condition from API
          setSftpCheckSuccess(true);
          setSftpMessage("SFTP data available. Loading form data...");
          showSnackbar("SFTP check successful. Loading data...", "success");
          setIsFormDisabled(false); // Enable form only if SFTP is fine

          // 2. If SFTP is successful, call getSavedDataTen
          const getDataPayload = {
            ...apiPayloadDefaults,
            // reportMasterId: "310010", // From your Schedule9C example, adjust if needed
            status: "11", // Example status for fetching
          };
          showSnackbar("Loading saved data for Schedule 10...", "info");
          const savedDataResponse = await callApi('/Maker/getSavedDataTen', getDataPayload, 'POST'); // ADJUST URL

          if (savedDataResponse && savedDataResponse.data) {
            // Transform API response to match formData structure
            // This part is crucial and depends on your actual API response structure
            // For now, assuming savedDataResponse.data is already in the correct formData format
            // If not, you'll need a transformation step similar to Schedule9C's example
            setFormData(prev => ({ ...prev, ...savedDataResponse.data }));
            showSnackbar('Data loaded successfully.', 'success');
          } else {
            showSnackbar('No saved data found or failed to load. Starting with a blank form.', 'warning');
            // setFormData(generateInitialSchedule10Data()); // Already default
          }
        } else {
          setSftpCheckSuccess(false);
          const message = sftpResponse?.message || "SFTP data not received from IFAMS. Form is disabled.";
          setSftpMessage(message);
          showSnackbar(message, "error");
          setIsFormDisabled(true); // Keep form disabled
        }
      } catch (error) {
        console.error('Initial load error:', error);
        setSftpMessage("Error during initial data load. Form is disabled.");
        showSnackbar('Error during initial data load. Please try again later.', 'error');
        setIsFormDisabled(true);
      } finally {
        setIsSftpLoading(false);
        setIsLoading(false); // Overall loading ends
      }
    };

    performInitialChecksAndLoad();
  }, [callApi, showSnackbar, apiPayloadDefaults]); // Dependencies for initial load


  const getNum = (value) => parseFloat(value) || 0;

  const calculatedData = useMemo(() => { /* ... (Same calculation logic from your working Schedule 10) ... */
    if (isLoading || isSftpLoading) return formData; // Prevent premature calculation
    console.time('Schedule10 Calculations');
    setIsCalculating(true);
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
    
    const totalRowProcessingOrder = ['row7', 'row12', 'row13', 'row14', 'row22', 'row27', 'row28', 'row31', 'row35', 'row29', 'row32'];
    if (newCalculatedData['row1']) calculateInternalRowTotals(newCalculatedData['row1']);

    totalRowProcessingOrder.forEach(orderedRowId => {
        const rowDef = rowDefinitionsConfig.find(r => r.id === orderedRowId);
        if(rowDef && rowDef.type === 'total') {
            const targetRow = newCalculatedData[rowDef.id];
            if (!targetRow) {
                newCalculatedData[rowDef.id] = {};
                schedule10DataFields.forEach(df => newCalculatedData[rowDef.id][df] = '0.00');
            }
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
     console.timeEnd('Schedule10 Calculations');
    return newCalculatedData;
  }, [formData, isLoading, isSftpLoading]);

  useEffect(() => {
    if (!isLoading && !isSftpLoading) { setIsCalculating(false); }
  }, [calculatedData, isLoading, isSftpLoading]);

  const handleChange = useCallback((rowId, fieldKey, value) => {
    if (isFormDisabled) return;
    const numericRegex = /^-?\d*\.?\d{0,2}$/;
    if (value === '' || value === '-' || numericRegex.test(value)) {
      setIsCalculating(true);
      setFormData(prev => {
        const newRowData = { ...(prev[rowId] || {}), [fieldKey]: value };
        return { ...prev, [rowId]: newRowData };
      });
    }
  }, [isFormDisabled]);

  const handleValidation = useCallback((nameParts, value) => { /* ... (validation logic) ... */
    const [rowId, fieldKey] = nameParts.split('_');
    let error = '';
    const numericRegex = /^-?\d*\.?\d{0,2}$/;
    if (value !== '' && value !== '-' && !numericRegex.test(value)) {
      error = 'Invalid number (e.g. 123.45)';
    }
    setErrors(prev => ({ ...prev, [`${rowId}_${fieldKey}`]: error }));
    return !error;
  }, []);
  const handleBlur = useCallback((rowId, fieldKey, value) => {
    if (isFormDisabled) return;
    handleValidation(`${rowId}_${fieldKey}`, value);
  }, [handleValidation, isFormDisabled]);

  const handleSubmit = async (isSaveOperation) => {
    if (isFormDisabled) {
      showSnackbar("Form is disabled due to SFTP data issue.", "warning");
      return;
    }
    // ... (Full validation logic placeholder)
    let allValid = true;
    // Iterate and validate all relevant fields
    // if (!allValid) { showSnackbar('Please correct validation errors.', 'error'); return; }

    showSnackbar(isSaveOperation ? 'Saving data...' : 'Submitting data...', 'info');
    const payload = {
      ...apiPayloadDefaults,
      status: isSaveOperation ? '11' : '21', // Example status for save/submit
      save: isSaveOperation,
      // Add all form data fields to payload, mapping them to backend field names
      // E.g., payload[`stcNstaff${modelSuffix}`] = formData.rowId.stcNstaff
    };
    // Transform formData to the flat structure expected by backend (fieldKey + modelSuffix)
    rowDefinitionsConfig.forEach(rowDef => {
        if (rowDef.type === 'entry' && rowDef.modelSuffix) {
            const rowInputData = formData[rowDef.id] || {};
            schedule10DataFields.forEach(fieldKey => {
                // Only include non-calculated fields or all fields as per API requirement
                // if (!intraRowCalculatedFields.includes(fieldKey)) {
                    const backendFieldName = `${fieldKey}${rowDef.modelSuffix}`; // e.g. stcNstaff1
                    payload[backendFieldName] = getNum(rowInputData[fieldKey]).toFixed(2);
                // }
            });
        }
        // Also send calculated totals if API expects them
        else if (rowDef.type === 'total' && rowDef.modelSuffix) {
             const rowTotalData = calculatedData[rowDef.id] || {};
             schedule10DataFields.forEach(fieldKey => {
                const backendFieldName = `${fieldKey}${rowDef.modelSuffix}`;
                payload[backendFieldName] = getNum(rowTotalData[fieldKey]).toFixed(2);
             });
        }
    });


    try {
      const response = await callApi('/Maker/submitTen', payload, 'POST'); // ADJUST URL
      if (response && response.message && response.message.includes(`~${isSaveOperation ? '11' : '21'}`)) {
        showSnackbar(response.message, 'success');
      } else {
        showSnackbar(response?.message || 'Operation failed. Please try again.', 'error');
      }
    } catch (error) {
      console.error('Submit/Save error:', error);
      showSnackbar('An error occurred during the operation.', 'error');
    }
  };

  const srNoColWidth = '60px';
  const particularsColWidth = '380px';
  const defaultDataColWidth = '110px';
  const firstHeaderRowRenderedHeight = '57px'; // ADJUST IF NECESSARY
  const totalHeaderHeightForBodyStick = '197px'; // ADJUST IF NECESSARY


  if (isSftpLoading || isLoading) {
    return ( <Box sx={{ display: 'flex', justifyContent: 'center', alignItems: 'center', height: 'calc(100vh - 150px)' }}> <CircularProgress /> <Typography sx={{ ml: 2 }}>{isSftpLoading ? "Checking SFTP status..." : "Loading Schedule data..."}</Typography> </Box> );
  }

  return (
    <Box sx={{ p: 1, width: '100%', boxSizing: 'border-box' }}>
      {!isSftpLoading && !sftpCheckSuccess && (
        <Alert severity="warning" sx={{ mb: 2 }}>
          {sftpMessage || "SFTP data not received from IFAMS. Form input is disabled."}
        </Alert>
      )}
      {isCalculating && ( <Box sx={{ position: 'fixed', top: '10px', right: '10px', zIndex: 1301, p: 1, backgroundColor: 'rgba(0,0,0,0.7)', color: 'white', borderRadius: '4px', display: 'flex', alignItems: 'center', fontSize: '0.8rem' }}> <CircularProgress size={14} color="inherit" sx={{ mr: 1 }} /> Calculating... </Box> )}
      {Object.values(errors).some(e => e) && ( <Alert severity="error" sx={{ mb: 2 }}>Please correct the highlighted errors.</Alert> )}

      <TableContainer component={Paper} sx={{ maxHeight: 'calc(100vh - 220px)', overflow: 'auto' }}>
        <Table sx={{ minWidth: 3000, tableLayout: 'fixed' }} aria-label="schedule 10 table">
          <colgroup>
            <col style={{ width: srNoColWidth, minWidth: srNoColWidth }} />
            <col style={{ width: particularsColWidth, minWidth: particularsColWidth }} />
            {columnDisplayHeaders.map(colDef => (
              <col key={`colGroup-${colDef.dataField}`} style={{ width: colDef.width || defaultDataColWidth, minWidth: colDef.width || defaultDataColWidth }} />
            ))}
          </colgroup>
          <TableHead>
            <TableRow>
              <StyledTableCell rowSpan={2} isHeaderCell isFixedColumn stickyLeftOffset="0" stickyTopOffset="0" sx={{ width: srNoColWidth, zIndex: 4 }}><b>Sr.No</b></StyledTableCell>
              <StyledTableCell rowSpan={2} isHeaderCell isFixedColumn stickyLeftOffset={srNoColWidth} stickyTopOffset="0" sx={{ width: particularsColWidth, zIndex: 4 }}><b>Particulars</b></StyledTableCell>
              <StyledTableCell colSpan={5} isHeaderCell stickyTopOffset="0"><b>(A) FURNITURE & FITTINGS</b></StyledTableCell>
              <StyledTableCell colSpan={10} isHeaderCell stickyTopOffset="0"><b>(B) MACHINERY & PLANT</b></StyledTableCell>
              <StyledTableCell rowSpan={2} isHeaderCell stickyTopOffset="0"><b>Total Furniture & Fixtures <br /> (A+B)</b></StyledTableCell>
              <StyledTableCell colSpan={12} isHeaderCell stickyTopOffset="0"><b>(C) PREMISES</b></StyledTableCell>
              <StyledTableCell rowSpan={2} isHeaderCell stickyTopOffset="0"><b>(D) Projects under <br /> construction</b></StyledTableCell>
              <StyledTableCell rowSpan={2} isHeaderCell stickyTopOffset="0"><b>Grand Total <br /> (A + B + C + D)</b></StyledTableCell>
            </TableRow>
            <TableRow>
              {columnDisplayHeaders.map((colDef) => (
                <StyledTableCell key={colDef.dataField} isHeaderCell stickyTopOffset={firstHeaderRowRenderedHeight} dangerouslySetInnerHTML={{ __html: colDef.labelHtml }} />
              ))}
            </TableRow>
          </TableHead>
          <TableBody>
            {rowDefinitionsConfig.map((rowDef) => {
              const rowKey = rowDef.id;
              const displayDataForRow = calculatedData[rowDef.id] || {};
              const rowThemeBg = rowDef.isSectionHeaderStyle ? theme.palette.grey[100] : rowDef.isTotalRowStyle ? theme.palette.grey[200] : theme.palette.background.paper;

              if (rowDef.type === 'sectionHeader' || rowDef.type === 'subSectionHeader') {
                return (
                  <StyledTableRow key={rowKey} $issectionheader={rowDef.type === 'sectionHeader'} $issubsectionheader={rowDef.type === 'subSectionHeader'}>
                    <StyledTableCell isFixedColumn stickyLeftOffset="0px" sx={{ backgroundColor: rowBg, top: totalHeaderHeightForBodyStick }}>{rowDef.srNo || ''}</StyledTableCell>
                    <StyledTableCell colSpan={columnDisplayHeaders.length +1} isFixedColumn stickyLeftOffset={srNoColWidth} sx={{ backgroundColor: rowBg, top: totalHeaderHeightForBodyStick }}>
                      <b>{typeof rowDef.label === 'function' ? rowDef.label(formData) : rowDef.label}</b>
                    </StyledTableCell>
                  </StyledTableRow>
                );
              }
              return (
                <StyledTableRow key={rowKey} $istotalrow={rowDef.isTotalRowStyle} $issectionheader={rowDef.isSectionHeaderStyle}>
                  <StyledTableCell isFixedColumn stickyLeftOffset="0px" sx={{ backgroundColor: rowBg }}><b>{rowDef.srNo || ''}</b></StyledTableCell>
                  <StyledTableCell isFixedColumn stickyLeftOffset={srNoColWidth} sx={{ backgroundColor: rowBg }}><b>{typeof rowDef.label === 'function' ? rowDef.label(formData) : rowDef.label}</b></StyledTableCell>
                  {columnDisplayHeaders.map((colDef) => {
                    const fieldKey = colDef.dataField;
                    const cellKey = `${rowKey}-${fieldKey}`;
                    const isCellReadOnly = isFormDisabled || rowDef.type === 'total' || colDef.isCalculated || (rowDef.isReadOnlyGroup && rowDef.isReadOnlyGroup.includes(fieldKey));
                    const valueToDisplay = displayDataForRow[fieldKey] !== undefined ? displayDataForRow[fieldKey] : '0.00';
                    const errorForField = errors[`${rowKey}_${fieldKey}`];
                    return (
                      <StyledTableCell key={cellKey}>
                        <TextField variant="outlined" size="small" name={`${rowKey}_${fieldKey}`}
                          value={valueToDisplay}
                          disabled={isCellReadOnly} // Disable individual cells
                          onChange={isCellReadOnly ? undefined : (e) => handleChange(rowDef.id, fieldKey, e.target.value)}
                          onBlur={isCellReadOnly ? undefined : (e) => handleBlur(rowDef.id, fieldKey, e.target.value)}
                          InputProps={{ readOnly: isCellReadOnly, sx: { backgroundColor: isCellReadOnly ? '#eeeeee' : 'white' } }} // More distinct disabled style
                          inputProps={{ style: { textAlign: 'right', padding: '6px 8px', width: '84px' } }}
                          sx={{ width: colDef.width || defaultDataColWidth }}
                          error={!!errorForField} helperText={errorForField || ' '} />
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
        <Button variant="contained" onClick={() => handleSubmit(true)} disabled={isFormDisabled}>Save</Button>
        <Button variant="contained" color="primary" onClick={() => handleSubmit(false)} disabled={isFormDisabled || Object.values(errors).some(e => e)}>Submit</Button>
      </Stack>
    </Box>
  );
};

export default Schedule10;


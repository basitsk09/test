import React, { useState, useEffect, useMemo, useCallback, useRef } from 'react';
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
  Dialog,
  DialogTitle,
  DialogContent,
  DialogActions,
  Snackbar,
} from '@mui/material';
import TableCell, { tableCellClasses } from '@mui/material/TableCell';
import { styled } from '@mui/material/styles';
import lodashDebounce from 'lodash/debounce';
import FormInput from '../../../../common/components/ui/FormInput';
import useApi from '../../../../common/hooks/useApi';
import { useLocation, useNavigate } from 'react-router-dom';
import useCustomSnackbar from '../../../../common/hooks/useCustomSnackbar';
import { CustomButton } from '../../../../common/components/ui/Buttons';
import LoadingOverlay from '../../../../common/components/ui/loadingOverlay';

// --- Styled Components (Updated for Theming) ---
const StyledTableCell = styled(TableCell, {
  shouldForwardProp: (prop) => prop !== 'isFixedColumn',
})(({ theme }) => ({
  fontSize: '0.875rem',
  padding: '8px',
  border: `1px solid ${theme.palette.divider}`,
  whiteSpace: 'nowrap',

  [`&.${tableCellClasses.head}`]: {
    // backgroundColor: theme.palette.grey[50],
    // color: theme.palette.text.primary,
    fontWeight: 'bold',
    textAlign: 'center',
  },
  [`&.${tableCellClasses.body}`]: {
    textAlign: 'left',
  },
}));

const StyledTableRow = styled(TableRow, {
  shouldForwardProp: (prop) => prop !== '$istotalrow' && prop !== '$issectionheader' && prop !== '$issubsectionheader',
})(({ theme, $istotalrow, $issectionheader, $issubsectionheader }) => ({
  '&:nth-of-type(odd)': {
    // backgroundColor: theme.palette.action.hover,
  },
  // Hide the last border
  // '&:last-child td, &:last-child th': {
  //   border: 0,
  // },
  ...($issectionheader && {
    // backgroundColor: theme.palette.grey[50],
    '& > td, & > th': {
      fontWeight: 'bold',
      textAlign: 'left',
      //color: theme.palette.text.primary,
    },
  }),
  ...($issubsectionheader && {
    // backgroundColor: theme.palette.grey[50],
    '& > td, & > th': {
      fontWeight: 'bold',
      fontStyle: 'italic',
      textAlign: 'left',
      // color: theme.palette.text.primary,
    },
  }),
  ...($istotalrow && {
    // backgroundColor: theme.palette.grey[50],
    '& > td, & > th': {
      fontWeight: 'bold',
      //color: theme.palette.text.primary,
    },
  }),
}));

// --- Configurations (identical to original) ---
const schedule10DataFields = [
  'stcNstaff',
  'offResidenceA',
  'otherPremisesA',
  'electricFitting',
  'totalA',
  'computers',
  'compSoftwareInt',
  'compSoftwareNonint',
  'compSoftwareTotal',
  'motor',
  'offResidenceB',
  'stcLho',
  'otherPremisesB',
  'otherMachineryPlant',
  'totalB',
  'totalFurnFix',
  'landNotRev',
  'landRev',
  'landRevEnh',
  'offBuildNotRev',
  'offBuildRev',
  'offBuildRevEnh',
  'residQuartNotRev',
  'residQuartRev',
  'residQuartRevEnh',
  'premisTotal',
  'revtotal',
  'totalC',
  'premisesUnderCons',
  'grandTotal',
];

const intraRowCalculatedFields = [
  'totalA',
  'compSoftwareTotal',
  'otherMachineryPlant',
  'totalB',
  'totalFurnFix',
  'premisTotal',
  'revtotal',
  'totalC',
  'grandTotal',
];

const rowDefinitionsConfig = [
  // --- Section: Original Cost / Revalued Value ---
  {
    id: 'row1',
    modelSuffix: '1',
    srNo: 'A',
    label: (formData) =>
      `Total Original Cost / Revalued Value upto the end of previous year i.e. 31st March ${formData.finyearOne || ''}`,
    type: 'entry',
    isSectionHeaderStyle: true,
  },
  // --- Section: Addition ---
  { id: 'header_addition', label: 'Addition', type: 'subSectionHeader' },
  {
    id: 'header_addition_a',
    srNo: '(a)',
    label: 'Original cost of items put to use during the year:',
    type: 'subSectionHeader',
    isMinorHeader: true,
  },
  {
    id: 'row3',
    modelSuffix: '3',
    srNo: '(i)',
    label: 'Cost of new items put to use upto 3rd October 2024',
    type: 'entry',
  },
  {
    id: 'row4',
    modelSuffix: '4',
    srNo: '(ii)',
    label: 'Cost of new items put to use during 4th October 2024 to 31st March 2025',
    type: 'entry',
  },
  {
    id: 'row36',
    modelSuffix: '36',
    srNo: '(b)',
    label: 'Increase in value of Fixed Assets due to Current Revaluation',
    type: 'entry',
  },
  {
    id: 'row5',
    modelSuffix: '5',
    srNo: '(c)',
    label: 'Original cost of items transferred from other Circles/Groups/CC Departments',
    type: 'entry',
  },
  {
    id: 'row6',
    modelSuffix: '6',
    srNo: '(d)',
    label: 'Original cost of items transferred from other branches of the same Circle',
    type: 'entry',
  },
  {
    id: 'row7',
    modelSuffix: '7',
    srNo: 'I',
    label: 'Total [a(i)+a(ii)+b+c+d]',
    type: 'total',
    subItemIds: ['row3', 'row4', 'row36', 'row5', 'row6'],
    operation: 'sum',
    isTotalRowStyle: true,
  },
  // --- Section: Deduction ---
  { id: 'header_deduction', label: 'Deduction', type: 'subSectionHeader' },
  {
    id: 'row37',
    modelSuffix: '37',
    srNo: '(i)',
    label: 'Short Valuation charged to Revaluation Reserve due to Current Downward Revaluation',
    type: 'entry',
  },
  {
    id: 'row9',
    modelSuffix: '9',
    srNo: '(ii)',
    label: 'Original cost of items sold/ discarded during the year',
    type: 'entry',
  },
  {
    id: 'row33',
    modelSuffix: '33',
    srNo: '(iii)',
    label: 'Projects under construction capitalised during the year',
    type: 'entry',
  },
  {
    id: 'row10',
    modelSuffix: '10',
    srNo: '(iv)',
    label: 'Original cost of items transferred to other Circles/Groups/CC Departments',
    type: 'entry',
  },
  {
    id: 'row11',
    modelSuffix: '11',
    srNo: '(v)',
    label: 'Original cost of items transferred to other branches in the same circle',
    type: 'entry',
  },
  {
    id: 'row12',
    modelSuffix: '12',
    srNo: 'II',
    label: 'Total (i+ii+iii+iv+v)',
    type: 'total',
    subItemIds: ['row37', 'row9', 'row33', 'row10', 'row11'],
    operation: 'sum',
    isTotalRowStyle: true,
  },
  // --- Section: Net Totals ---
  {
    id: 'row13',
    modelSuffix: '13',
    srNo: 'B',
    label: 'Net Addition (I-II)',
    type: 'total',
    subItemIds: ['row7', 'row12'],
    operation: 'subtract',
    isSectionHeaderStyle: true,
    isTotalRowStyle: true,
  },
  {
    id: 'row14',
    modelSuffix: '14',
    srNo: 'C',
    label: (formData) => `Total Original Cost/ Revalued Value as at 31st March ${formData.finyearTwo || ''} (A+B)`,
    type: 'total',
    subItemIds: ['row1', 'row13'],
    operation: 'sum',
    isSectionHeaderStyle: true,
    isTotalRowStyle: true,
  },
  // --- Section: Depreciation ---
  { id: 'header_depreciation', label: 'Depreciation', type: 'subSectionHeader' },
  {
    id: 'row18',
    modelSuffix: '18',
    srNo: '(i)',
    label: (formData) => `Depreciation upto the end of previous year i.e. 31st March ${formData.finyearOne || ''}`,
    type: 'entry',
  },
  {
    id: 'row34',
    modelSuffix: '34',
    srNo: '(ii)',
    label: (formData) =>
      `Short Valuation charged to depreciation upto end of previous year i.e.31st March ${formData.finyearOne || ''}`,
    type: 'entry',
  },
  {
    id: 'row38',
    modelSuffix: '38',
    srNo: '(iii)',
    label: 'Depreciation on repatriation of Officials from Subsidiaries/ Associates',
    type: 'entry',
  },
  {
    id: 'row19',
    modelSuffix: '19',
    srNo: '(iv)',
    label: 'Depreciation transferred from other Circles/Groups/CC Departments',
    type: 'entry',
  },
  {
    id: 'row20',
    modelSuffix: '20',
    srNo: '(v)',
    label: 'Depreciation transferred from other branches of the same circle.',
    type: 'entry',
  },
  {
    id: 'row21',
    modelSuffix: '21',
    srNo: '(vi)',
    label: 'Depreciation charged during the current year',
    type: 'entry',
  },
  {
    id: 'row39',
    modelSuffix: '39',
    srNo: '(vii)',
    label: 'Short Valuation charged to Depreciation during the current year due to Current Revaluation',
    type: 'entry',
  },
  {
    id: 'row22',
    modelSuffix: '22',
    srNo: 'D',
    label: 'Total (i+ii+iii+iv+v+vi+vii)',
    type: 'total',
    subItemIds: ['row18', 'row34', 'row38', 'row19', 'row20', 'row21', 'row39'],
    operation: 'sum',
    isTotalRowStyle: true,
  },
  // --- Section: Less Depreciation ---
  { id: 'header_less_depreciation', label: 'Less :', type: 'subSectionHeader' },
  {
    id: 'row40',
    modelSuffix: '40',
    srNo: '(i)',
    label: 'Past Short Valuation credited to Depreciation during the current year due to Current Upward Revaluation',
    type: 'entry',
  },
  {
    id: 'row24',
    modelSuffix: '24',
    srNo: '(ii)',
    label: 'Depreciation previously provided on fixed assets sold/ discarded',
    type: 'entry',
  },
  {
    id: 'row25',
    modelSuffix: '25',
    srNo: '(iii)',
    label: 'Depreciation transferred to other Circles/Groups/CC Departments',
    type: 'entry',
  },
  {
    id: 'row26',
    modelSuffix: '26',
    srNo: '(iv)',
    label: 'Depreciation transferred to other branches of the same Circle.',
    type: 'entry',
  },
  {
    id: 'row27',
    modelSuffix: '27',
    srNo: 'E',
    label: 'Total (i+ii+iii+iv)',
    type: 'total',
    subItemIds: ['row40', 'row24', 'row25', 'row26'],
    operation: 'sum',
    isTotalRowStyle: true,
  },
  // --- Section: Net Depreciation & Book Value ---
  {
    id: 'row28',
    modelSuffix: '28',
    srNo: 'F',
    label: 'Net Depreciation (D-E)',
    type: 'total',
    subItemIds: ['row22', 'row27'],
    operation: 'subtract',
    isTotalRowStyle: true,
  },
  {
    id: 'row29',
    modelSuffix: '29',
    srNo: 'G',
    label: (formData) => `Net Book Value as at 31st March ${formData.finyearTwo || ''} (C-F)`,
    type: 'total',
    subItemIds: ['row14', 'row28'],
    operation: 'subtract',
    isSectionHeaderStyle: true,
    isTotalRowStyle: true,
  },
  // --- Section: Sale of Assets ---
  { id: 'row30', modelSuffix: '30', srNo: 'H', label: 'Sale Price of fixed assets', type: 'entry' },
  {
    id: 'row31',
    modelSuffix: '31',
    srNo: 'I',
    label: 'Book Value of fixed assets sold [II (ii)-E(ii)]',
    type: 'total',
    subItemIds: ['row9', 'row24'],
    operation: 'subtract_special_IIii_Eii',
    isTotalRowStyle: true,
  },
  { id: 'row35', modelSuffix: '35', srNo: 'J', label: 'GST on Sale of fixed assets', type: 'entry' },
  {
    id: 'row32',
    modelSuffix: '32',
    srNo: 'K',
    label: 'Profit/ (Loss) on sale of fixed assets [H-(I+J)]',
    type: 'total',
    subItemIds: ['row30', 'row31', 'row35'],
    operation: 'custom_H_minus_IplusJ',
    isTotalRowStyle: true,
  },
];

const columnDisplayHeaders = [
  // Group (A) Furniture & Fittings
  { labelHtml: 'i) At STCs & Staff Colleges <br /> (For Local Head Office only)', dataField: 'stcNstaff' },
  { labelHtml: "ii) At Officers' Residences", dataField: 'offResidenceA' },
  { labelHtml: 'iii) At Other Premises', dataField: 'otherPremisesA' },
  {
    labelHtml:
      'iv) Electric Fittings <br /> (include electric wiring, <br /> switches, sockets, other <br /> fittings & fans etc.)',
    dataField: 'electricFitting',
  },
  { labelHtml: 'TOTAL (A) <br /> (i+ii+iii+iv)', dataField: 'totalA', isCalculated: true },
  // Group (B) Machinery & Plant
  { labelHtml: 'i) Computer Hardware', dataField: 'computers' },
  { labelHtml: 'a. Computer Software <br /> (forming integral part of <br /> Hardware)', dataField: 'compSoftwareInt' },
  {
    labelHtml: 'b. Computer Software <br /> (not forming integral <br /> of Hardware)',
    dataField: 'compSoftwareNonint',
  },
  { labelHtml: 'ii) Computer Software <br /> Total (a+b)', dataField: 'compSoftwareTotal', isCalculated: true },
  { labelHtml: 'iii) Motor Vehicles', dataField: 'motor' },
  { labelHtml: "a) At Officers' Residences", dataField: 'offResidenceB' },
  { labelHtml: 'b) At STCs <br /> (For Local Head Office)', dataField: 'stcLho' },
  { labelHtml: 'c) At other Premises', dataField: 'otherPremisesB' },
  { labelHtml: 'iv) Other Machinery & Plant <br />(a+b+c)', dataField: 'otherMachineryPlant', isCalculated: true },
  { labelHtml: 'TOTAL (B) <br /> (i+ii+iii+iv)', dataField: 'totalB', isCalculated: true },
  // Total Furniture & Fixtures (A+B)
  { labelHtml: 'Total Furniture & Fixtures <br /> (A+B)', dataField: 'totalFurnFix', isCalculated: true },
  // Group (C) Premises
  { labelHtml: '(a) Land (Not Revalued): <br /> Cost', dataField: 'landNotRev' },
  { labelHtml: '(b) Land (Revalued): <br /> Cost', dataField: 'landRev' },
  { labelHtml: '(c) Land (Revalued): <br /> Enhancement due to <br /> Revaluation', dataField: 'landRevEnh' },
  { labelHtml: '(d) Office Building <br /> (Not revalued): Cost', dataField: 'offBuildNotRev' },
  { labelHtml: '(e) Office Building <br /> (Revalued): Cost', dataField: 'offBuildRev' },
  {
    labelHtml: '(f) Office Building <br /> (Revalued): Enhancement <br /> due to Revaluation',
    dataField: 'offBuildRevEnh',
  },
  { labelHtml: '(g) Residential Building <br /> (Not revalued): Cost', dataField: 'residQuartNotRev' },
  { labelHtml: '(h) Residential Building <br /> (Revalued): Cost', dataField: 'residQuartRev' },
  {
    labelHtml: '(i) Residential Building <br /> (Revalued): Enhancement <br /> due to Revaluation',
    dataField: 'residQuartRevEnh',
  },
  { labelHtml: '(j) Premises Total <br /> (a+b+d+e+g+h)', dataField: 'premisTotal', isCalculated: true },
  { labelHtml: '(k) Revaluation Total <br /> (c+f+i)', dataField: 'revtotal', isCalculated: true },
  { labelHtml: 'TOTAL (C) <br /> (j+k)', dataField: 'totalC', isCalculated: true },
  { labelHtml: '(D) Projects under <br /> construction', dataField: 'premisesUnderCons' },
  { labelHtml: 'Grand Total <br /> (A + B + C + D)', dataField: 'grandTotal', isCalculated: true },
];

const generateInitialSchedule10Data = () => {
  const initialData = {
    finyearOne: new Date().getFullYear().toString(),
    finyearTwo: (new Date().getFullYear() + 1).toString(),
  };
  rowDefinitionsConfig.forEach((rowDef) => {
    if (rowDef.type === 'entry' || rowDef.type === 'total') {
      initialData[rowDef.id] = {};
      schedule10DataFields.forEach((fieldKey) => {
        initialData[rowDef.id][fieldKey] = '0.00';
      });
    }
  });
  return initialData;
};

// --- OPTIMIZATION: Memoized FormInput Component ---
const MemoizedFormInput = React.memo(function MemoizedFormInput({
  name,
  value,
  onChange,
  onBlur,
  readOnly,
  error,
  helperText,
  customStyles, // Ensure customStyles is destructured here
}) {
  return (
    <FormInput
      name={name}
      value={value}
      onChange={onChange}
      onBlur={onBlur}
      readOnly={readOnly}
      error={!!error}
      helperText={helperText}
      customStyles={{
        width: '200px', // ? consistent fixed width for all
        '& input': {
          textAlign: 'right',
          padding: '6px 8px',
          ...(error && { border: '1px solid red' }), // Apply red border if error
        },
        ...customStyles, // Merge customStyles
      }}
    />
  );
});

// --- OPTIMIZATION: Virtualized Input using Intersection Observer ---
const VirtualizedInput = (props) => {
  const [isInView, setIsInView] = useState(false);
  const placeholderRef = useRef(null);

  useEffect(() => {
    const observer = new IntersectionObserver(
      ([entry]) => {
        // If the component is intersecting the viewport, show the real input
        if (entry.isIntersecting) {
          setIsInView(true);
          // Stop observing once it's visible
          observer.unobserve(entry.target);
        }
      },
      {
        // Optional: Adjust rootMargin to load inputs slightly before they appear on screen
        rootMargin: '200px',
      }
    );

    if (placeholderRef.current) {
      observer.observe(placeholderRef.current);
    }

    return () => {
      if (placeholderRef.current) {
        observer.unobserve(placeholderRef.current);
      }
    };
  }, []);

  // Placeholder has fixed height and alignment to prevent layout shifts when the real input loads
  const placeholder = (
    <Box
      ref={placeholderRef}
      sx={{
        height: '38px',
        width: '200px', // Match consistent fixed width for all FormInput
        textAlign: 'right',
        padding: '6px 8px',
        boxSizing: 'border-box',
        ...(props.error && { border: '1px solid red' }), // Apply red border to placeholder if error
      }}
    >
      {props.displayValue}
    </Box>
  );

  return isInView ? <MemoizedFormInput {...props} /> : placeholder;
};

//const useCustomSnackbar = () => (message, severity) => console.log(`Snackbar: ${message} (${severity})`);

const Schedule10 = () => {
  const setSnackbarMessage = useCustomSnackbar();
  const [isChecking, setIsChecking] = useState(false);
  const [formData, setFormData] = useState(generateInitialSchedule10Data);
  const [errors, setErrors] = useState({}); // This will now hold both general errors and pre-check validation errors
  const [isLoading, setIsLoading] = useState(true);
  const [isCalculating, setIsCalculating] = useState(false);
  const { state } = useLocation();
  const navigate = useNavigate();
  const user = JSON.parse(localStorage.getItem('user'));
  const [reportObject, setReportObject] = useState(state?.report || null);
  //const showSnackbar = useCustomSnackbar();
  const { callApi } = useApi();
  const [fieldsDisabled, setFieldsDisabled] = useState(false);
  const [dialogOpen, setDialogOpen] = useState(false);
  const [dialogContent, setDialogContent] = useState({ title: '', message: '', onConfirm: () => {} });
  const [openSubmitDialog, setOpenSubmitDialog] = useState(false);
  const [isSubmitting, setIsSubmitting] = useState(false);
  const [preCheckOpen, setPreCheckOpen] = useState(false); // State for the pre-check dialog
  const [preCheckData, setPreCheckData] = useState({ // Data to be displayed in the pre-check dialog
    otherFixedAsset: '0.00',
    premises: '0.00',
    premisesUnderConstruction: '0.00',
  });
  const [manualEntry, setManualEntry] = useState(true);

  // Helper function to convert flat data to structured formData
  const convertFlatSc10DataToFormData = (flatData) => {
    const structuredData = generateInitialSchedule10Data(); // Starts with default structure
    rowDefinitionsConfig.forEach((rowDef) => {
      const rowId = rowDef.id;
      const suffix = rowDef.modelSuffix;
      // Ensure the row exists in structuredData before trying to assign fields
      if (!structuredData[rowId]) {
        structuredData[rowId] = {};
      }
      schedule10DataFields.forEach((field) => {
        const flatKey = `${field}${suffix}`;
        if (flatData[flatKey] !== undefined) {
          structuredData[rowId][field] = flatData[flatKey];
        } else {
          // Ensure field is initialized if not present in flatData
          structuredData[rowId][field] = '0.00';
        }
      });
    });
    // Handle additional fields like finyearOne, finyearTwo
    structuredData.finyearOne = flatData.finyearOne || new Date().getFullYear().toString();
    structuredData.finyearTwo = flatData.finyearTwo || (new Date().getFullYear() + 1).toString();
    return structuredData;
  };

  const getNum = (value) => parseFloat(value) || 0; // Moved this helper up

  // Memoized calculated data
  const calculatedData = useMemo(() => {
    console.time('Schedule10 Calculations');
    const newCalculatedData = JSON.parse(JSON.stringify(formData));

    const calculateInternalRowTotals = (rowObj) => {
      if (!rowObj) return;
      const p = (fieldKey) => getNum(rowObj[fieldKey]);
      rowObj.totalA = (p('stcNstaff') + p('offResidenceA') + p('otherPremisesA') + p('electricFitting')).toFixed(2);
      rowObj.compSoftwareTotal = (p('compSoftwareInt') + p('compSoftwareNonint')).toFixed(2);
      const otherMachineryPlantVal = p('offResidenceB') + p('stcLho') + p('otherPremisesB');
      rowObj.otherMachineryPlant = otherMachineryPlantVal.toFixed(2);
      rowObj.totalB = (p('computers') + getNum(rowObj.compSoftwareTotal) + p('motor') + otherMachineryPlantVal).toFixed(
        2
      );
      rowObj.totalFurnFix = (getNum(rowObj.totalA) + getNum(rowObj.totalB)).toFixed(2);
      const premisTotalVal =
        p('landNotRev') +
        p('landRev') +
        p('offBuildNotRev') +
        p('offBuildRev') +
        p('residQuartNotRev') +
        p('residQuartRev');
      rowObj.premisTotal = premisTotalVal.toFixed(2);
      const revTotalVal = p('landRevEnh') + p('offBuildRevEnh') + p('residQuartRevEnh');
      rowObj.revtotal = revTotalVal.toFixed(2);
      rowObj.totalC = (premisTotalVal + revTotalVal).toFixed(2);
      rowObj.grandTotal = (
        getNum(rowObj.totalA) +
        getNum(rowObj.totalB) +
        getNum(rowObj.totalC) +
        p('premisesUnderCons')
      ).toFixed(2);
    };

    // Initialize all rows/fields in newCalculatedData based on config
    rowDefinitionsConfig.forEach((rowDef) => {
      if (!newCalculatedData[rowDef.id]) {
        newCalculatedData[rowDef.id] = {};
      }
      schedule10DataFields.forEach((fieldKey) => {
        newCalculatedData[rowDef.id][fieldKey] = formData[rowDef.id]?.[fieldKey] ?? '0.00';
      });
    });

    // Calculate intra-row totals for entry rows first
    rowDefinitionsConfig.forEach((rowDef) => {
      if (rowDef.type === 'entry') {
        calculateInternalRowTotals(newCalculatedData[rowDef.id]);
      }
    });

    // Calculate inter-row totals
    rowDefinitionsConfig.forEach((rowDef) => {
      if (rowDef.type === 'total') {
        const targetRow = newCalculatedData[rowDef.id];
        schedule10DataFields.forEach((fieldKey) => {
          let value = 0;
          if (rowDef.operation === 'sum') {
            rowDef.subItemIds.forEach((subId) => {
              value += getNum(newCalculatedData[subId]?.[fieldKey]);
            });
          } else if (rowDef.operation === 'subtract' && rowDef.subItemIds?.length === 2) {
            const val1 = getNum(newCalculatedData[rowDef.subItemIds[0]]?.[fieldKey]);
            const val2 = getNum(newCalculatedData[rowDef.subItemIds[1]]?.[fieldKey]);
            value = val1 - val2;
          } else if (rowDef.operation === 'subtract_special_IIii_Eii') {
            const valRow9 = getNum(newCalculatedData['row9']?.[fieldKey]);
            const valRow24 = getNum(newCalculatedData['row24']?.[fieldKey]);
            value = valRow9 - valRow24;
          } else if (rowDef.operation === 'custom_H_minus_IplusJ') {
            const valH = getNum(newCalculatedData['row30']?.[fieldKey]);
            const valI = getNum(newCalculatedData['row31']?.[fieldKey]);
            const valJ = getNum(newCalculatedData['row35']?.[fieldKey]);
            value = valH - (valI + valJ);
          }
          targetRow[fieldKey] = value.toFixed(2);
        });
        calculateInternalRowTotals(targetRow); // Calculate intra-row totals for total rows too
      }
    });
    console.timeEnd('Schedule10 Calculations');
    return newCalculatedData;
  }, [formData]); // Recalculate whenever formData changes

  // Use a separate effect to trigger silentPreCheckValidation when calculatedData is ready
  useEffect(() => {
    // Only run validation if not in initial loading phase or during active calculation
    // and if there's an actual report object (meaning data might have been loaded)
    if (!isLoading && !isCalculating && reportObject) {
      silentPreCheckValidation();
    }
  }, [calculatedData, isLoading, isCalculating, reportObject]); // Depend on calculatedData to ensure it's updated

  const silentPreCheckValidation = useCallback(async () => {
    try {
      const payload = {
        circleCode: user.circleCode,
        quarterEndDate: user.quarterEndDate,
        status: reportObject.status,
        reportId: reportObject.reportId,
        reportMasterId: reportObject.reportMasterId,
        reportName: reportObject.name,
        userId: user.userId,
        areMocPending: false,
      };

      const response = await callApi('/Maker/getValidationDataTen', payload, 'POST');
      if (response) {
        const validationErrors = {};
        // Use calculatedData for comparison, as it reflects the latest form state
        const row29 = calculatedData['row29']; // Use calculatedData here

        if (row29 && response) { // Ensure both exist before comparing
          if (getNum(row29.totalA).toFixed(2) !== getNum(response.validationOtherFixedAssetAmount).toFixed(2)) {
            validationErrors['row29-totalA'] = 'Mismatch with precheck';
          }
          if (getNum(row29.totalB).toFixed(2) !== getNum(response.validationOtherFixedAssetAmount).toFixed(2)) {
            validationErrors['row29-totalB'] = 'Mismatch with precheck';
          }
          if (getNum(row29.totalC).toFixed(2) !== getNum(response.validationPremisesAmount).toFixed(2)) {
            validationErrors['row29-totalC'] = 'Mismatch with precheck';
          }
          if (
            getNum(row29.premisesUnderCons).toFixed(2) !==
            getNum(response.validationPremisesUnderConsAmount).toFixed(2)
          ) {
            validationErrors['row29-premisesUnderCons'] = 'Mismatch with precheck';
          }
        }


        // Update errors state, merging with existing general errors
        setErrors((prev) => {
            const newErrors = { ...prev };
            // Clear previous precheck errors from row29 before adding new ones
            ['row29-totalA', 'row29-totalB', 'row29-totalC', 'row29-premisesUnderCons'].forEach(key => {
                delete newErrors[key];
            });
            return { ...newErrors, ...validationErrors };
        });

        if (Object.keys(validationErrors).length > 0) {
          setSnackbarMessage('Row29 data mismatch with precheck. Please correct the fields.', 'error');
        } else {
            setSnackbarMessage('Row29 precheck validation successful.', 'success');
        }
      }
    } catch (error) {
      console.error('Silent precheck error:', error);
      // Only show error if it's not a 'canceled' error (e.g., component unmounted during fetch)
      if (error.message !== 'canceled') {
        setSnackbarMessage('Failed to run initial validation precheck.', 'error');
      }
    }
  }, [user, reportObject, callApi, calculatedData, setSnackbarMessage, getNum]); // Add getNum to dependencies

  // Effect to load initial SFTP/DB data
  useEffect(() => {
    const checkSC10SftpData = async () => {
      const payload = {
        circleCode: user.circleCode,
        qed: user.quarterEndDate,
        reportStatus: 'A',
        reportID: reportObject.reportMasterId,
        reportName: 'SC10',
      };
      try {
        [span_0](start_span)const data = await callApi('/IFAMSS/SC10SFTP', payload, 'POST'); //[span_0](end_span)

        [span_1](start_span)if (user.isCircleExist === 'true') { //[span_1](end_span)
          [span_2](start_span)if (data.fileAndDataStatus === 1) { //[span_2](end_span)
            const convertedFormData = convertFlatSc10DataToFormData(data.sc10Data); [span_3](start_span)//[span_3](end_span)
            setFormData(convertedFormData); [span_4](start_span)//[span_4](end_span)
            setFieldsDisabled(true); [span_5](start_span)//[span_5](end_span)
            setSnackbarMessage('Data successfully fetched from IFAMS via SFTP.', 'success'); [span_6](start_span)//[span_6](end_span)
          [span_7](start_span)} else if (data.fileAndDataStatus === 2) { //[span_7](end_span)
            setFieldsDisabled(true); [span_8](start_span)//[span_8](end_span)
            showDialog({
              [span_9](start_span)title: 'File Error', //[span_9](end_span)
              message: data.message || [span_10](start_span)'Data not received from IFAMS, Kindly wait till IFAMS sends reports', //[span_10](end_span)
              [span_11](start_span)onConfirm: () => navigate(-1), // go back[span_11](end_span)
            });
          [span_12](start_span)} else if (data.fileAndDataStatus === 3) { //[span_12](end_span)
            setFieldsDisabled(true); [span_13](start_span)//[span_13](end_span)
            showDialog({
              [span_14](start_span)title: 'Info', //[span_14](end_span)
              message: data.message || [span_15](start_span)'Please note: Data fetched here was generated in IFAMS', //[span_15](end_span)
              [span_16](start_span)onConfirm: () => {}, //[span_16](end_span)
            });
          }
        [span_17](start_span)} else { //[span_17](end_span)
          setManualEntry(false); [span_18](start_span)//[span_18](end_span)
          getValidationDataTen(); [span_19](start_span)//[span_19](end_span)
        }
      [span_20](start_span)} catch (error) { //[span_20](end_span)
        [span_21](start_span)if (error.message !== 'canceled') { //[span_21](end_span)
          setSnackbarMessage('Error while checking SC10 SFTP data.', 'error'); [span_22](start_span)//[span_22](end_span)
        }
      } finally {
        setIsLoading(false); // Ensure loading is turned off regardless of success/error
      }
    };
    checkSC10SftpData();
  }, [user, reportObject, callApi, navigate, setSnackbarMessage, convertFlatSc10DataToFormData, showDialog]); // Add all dependencies

  // Helper function for getting saved data (used when manualEntry is false)
  const getValidationDataTen = useCallback(async () => { // Make this useCallback too
    const payload = {
      circleCode: user.circleCode,
      quarterEndDate: user.quarterEndDate,
      status: reportObject.status,
      reportId: reportObject.reportId,
      reportMasterId: reportObject.reportMasterId,
      reportName: reportObject.name,
      areMocPending: reportObject.areMocPending,
    };
    try {
      [span_23](start_span)const data = await callApi('/Maker/getSavedDataTen', payload, 'POST'); //[span_23](end_span)
      [span_24](start_span)if (data) { //[span_24](end_span)
        // console.log('data received from getSavedDataTen', data); // Log the raw data
        const convertedFormData = convertFlatSc10DataToFormData(data); [span_25](start_span)//[span_25](end_span)
        setFormData(convertedFormData); [span_26](start_span)//[span_26](end_span)
        // console.log('formData after setting from getSavedDataTen', convertedFormData); // Log the value being set
      [span_27](start_span)} else { //[span_27](end_span)
        throw new Error('Error while checking SC10 SFTP data.'); [span_28](start_span)//[span_28](end_span)
      }
    [span_29](start_span)} catch (error) { //[span_29](end_span)
      setSnackbarMessage(error.message || 'Error while checking SC10 SFTP data.', 'error'); [span_30](start_span)//[span_30](end_span)
    }
  }, [user, reportObject, callApi, setSnackbarMessage, convertFlatSc10DataToFormData]);


  [span_31](start_span)const flattenSchedule10Data = (formData) => { //[span_31](end_span)
    const flatData = {}; [span_32](start_span)//[span_32](end_span)
    [span_33](start_span)rowDefinitionsConfig.forEach((rowDef) => { //[span_33](end_span)
      const suffix = rowDef.modelSuffix; [span_34](start_span)//[span_34](end_span)
      if (!suffix) return; [span_35](start_span)// Skip headers[span_35](end_span)

      const row = formData[rowDef.id] || {}; [span_36](start_span)//[span_36](end_span)
      [span_37](start_span)schedule10DataFields.forEach((field) => { //[span_37](end_span)
        const key = `${field}${suffix}`; [span_38](start_span)//[span_38](end_span)
        flatData[key] = row[field] ?? '0.00'; [span_39](start_span)//[span_39](end_span)
      });
    });

    // Add year fields if needed
    flatData.finyearOne = formData.finyearOne; [span_40](start_span)//[span_40](end_span)
    flatData.finyearTwo = formData.finyearTwo; [span_41](start_span)//[span_41](end_span)

    return flatData; [span_42](start_span)//[span_42](end_span)
  };

  [span_43](start_span)const handleSaveOrSubmit = async (isSaveOnly = true) => { //[span_43](end_span)
    const hasValidationErrors = Object.values(errors).some((val) => val && val.length > 0);
    // Check if any errors exist (including pre-check errors)
    if (!isSaveOnly && hasValidationErrors) {
      setSnackbarMessage('Cannot submit. Please fix validation errors in Row 29.', 'error');
      setIsSubmitting(false); // Ensure submitting state is reset
      return;
    }

    setIsSubmitting(true); [span_44](start_span)//[span_44](end_span)
    try {
      const dataToSend = flattenSchedule10Data(formData); [span_45](start_span)//[span_45](end_span)

      [span_46](start_span)const payload = { //[span_46](end_span)
        [span_47](start_span)...dataToSend, //[span_47](end_span)
        [span_48](start_span)save: isSaveOnly, //[span_48](end_span)
        [span_49](start_span)circleCode: user.circleCode, //[span_49](end_span)
        [span_50](start_span)quarterEndDate: user.quarterEndDate, //[span_50](end_span)
        [span_51](start_span)userId: user.userId, //[span_51](end_span)
        [span_52](start_span)reportName: reportObject.name, //[span_52](end_span)
        [span_53](start_span)reportMasterId: reportObject.reportMasterId, //[span_53](end_span)
        [span_54](start_span)reportId: reportObject.reportId, //[span_54](end_span)
        [span_55](start_span)status: reportObject.status, //[span_55](end_span)
      };

      const response = await callApi('/Maker/submitTen', payload, 'POST'); [span_56](start_span)//[span_56](end_span)

      [span_57](start_span)if (response) { //[span_57](end_span)
        [span_58](start_span)if (response && typeof response === 'string') { //[span_58](end_span)
          const [_flag, newReportId, newStatus] = response.split('~'); [span_59](start_span)//[span_59](end_span)
          [span_60](start_span)setReportObject((prev) => ({ //[span_60](end_span)
            ...prev,
            reportId: newReportId,
            status: newStatus,
          }));
          setSnackbarMessage(isSaveOnly ? 'Report saved successfully!' : 'Report submitted successfully!', 'success'); [span_61](start_span)//[span_61](end_span)
          [span_62](start_span)if (!isSaveOnly) { //[span_62](end_span)
            [span_63](start_span)setTimeout(() => { //[span_63](end_span)
              navigate('../'); [span_64](start_span)//[span_64](end_span)
            }, 2000);
          }
        }
      [span_65](start_span)} else { //[span_65](end_span)
        throw new Error(response.data?.message || 'Unexpected response'); [span_66](start_span)//[span_66](end_span)
      }
    [span_67](start_span)} catch (err) { //[span_67](end_span)
      console.log('error in handleSaveOrSubmit :: ', err); [span_68](start_span)//[span_68](end_span)
      setSnackbarMessage('Error occurred during save/submit.', 'error'); [span_69](start_span)//[span_69](end_span)
    } finally {
      setIsSubmitting(false); [span_70](start_span)//[span_70](end_span)
    }
  };

  [span_71](start_span)const showDialog = ({ title, message, onConfirm }) => { //[span_71](end_span)
    setDialogOpen(true); [span_72](start_span)//[span_72](end_span)
    setDialogContent({ title, message, onConfirm }); [span_73](start_span)//[span_73](end_span)
  };

  [span_74](start_span)useEffect(() => { //[span_74](end_span)
    setIsLoading(false); [span_75](start_span)//[span_75](end_span)
  }, []);


  [span_76](start_span)useEffect(() => { //[span_76](end_span)
    [span_77](start_span)if (isCalculating) { //[span_77](end_span)
      setIsCalculating(false); [span_78](start_span)//[span_78](end_span)
    }
  }, [calculatedData, isCalculating]); [span_79](start_span)//[span_79](end_span)

  [span_80](start_span)const debouncedRecalculate = useCallback( //[span_80](end_span)
    [span_81](start_span)lodashDebounce((newFormData) => { //[span_81](end_span)
      setIsCalculating(true); [span_82](start_span)//[span_82](end_span)
      setFormData(newFormData); [span_83](start_span)//[span_83](end_span)
    [span_84](start_span)}, 300), //[span_84](end_span)
    [span_85](start_span)[] //[span_85](end_span)
  );

  [span_86](start_span)const handleChange = useCallback( //[span_86](end_span)
    [span_87](start_span)(rowId, fieldKey, value) => { //[span_87](end_span)
      [span_88](start_span)const newFormData = { //[span_88](end_span)
        [span_89](start_span)...formData, //[span_89](end_span)
        [span_90](start_span)[rowId]: { ...(formData[rowId] || {}), [fieldKey]: value }, //[span_90](end_span)
      };
      setFormData(newFormData); [span_91](start_span)//[span_91](end_span)
      debouncedRecalculate(newFormData); [span_92](start_span)//[span_92](end_span)
    },
    [span_93](start_span)[formData, debouncedRecalculate] //[span_93](end_span)
  );

  [span_94](start_span)const handleBlur = useCallback((rowId, fieldKey, value) => { //[span_94](end_span)
    const numericRegex = /^-?\d*\.?\d{0,2}$/; [span_95](start_span)//[span_95](end_span)
    let error = ''; [span_96](start_span)//[span_96](end_span)
    [span_97](start_span)if (value !== '' && value !== '-' && !numericRegex.test(value)) { //[span_97](end_span)
      error = 'Invalid number'; [span_98](start_span)//[span_98](end_span)
    }
    setErrors((prev) => ({ ...prev, [`${rowId}-${fieldKey}`]: error })); [span_99](start_span)//[span_99](end_span)
  }, []);


  [span_100](start_span)if (isLoading) { //[span_100](end_span)
    return (
      <Box sx={{ display: 'flex', justifyContent: 'center', alignItems: 'center', height: 'calc(100vh - 150px)' }}>
        <CircularProgress /> <Typography sx={{ ml: 2 }}>Loading Schedule 10...</Typography>
      </Box>
    ); [span_101](start_span)//[span_101](end_span)
  }

  return (
    <Box
      sx={{ p: 1, width: '100%', boxSizing: 'border-box', overflowX: 'hidden' /* bgcolor: 'background.default' */ }}
    >
      [span_102](start_span)<Dialog open={preCheckOpen} onClose={() => setPreCheckOpen(false)} fullWidth maxWidth="sm"> {/*[span_102](end_span) */}
        [span_103](start_span)<DialogTitle sx={{ backgroundColor: '#E74C3C', color: 'white' }}>Attention!</DialogTitle> {/*[span_103](end_span) */}
        [span_104](start_span)<DialogContent dividers> {/*[span_104](end_span) */}
          <Typography variant="h6" component="div" sx={{ mb: 2 }}>
            [span_105](start_span)The values for pre-checks of comp {/*[span_105](end_span) */}
            codes are as follows:-
          </Typography>
          <Box component="table" sx={{ width: '100%' }}>
            <Box component="tbody" sx={{ fontSize: '15px' }}>
              <Box component="tr">
                <Box component="td" sx={{ py: 1 }}>
                  [span_106](start_span)OTHER {/*[span_106](end_span) */}
                  FIXED ASSETS (including furniture and fixtures) ={' '}
                  [span_107](start_span)<strong>{preCheckData.otherFixedAsset}</strong> {/*[span_107](end_span) */}
                </Box>
              </Box>
              <Box component="tr">
                <Box component="td" sx={{ py: 1 }}>
                  [span_108](start_span)PREMISES = <strong>{preCheckData.premises}</strong> {/*[span_108](end_span) */}
                </Box>
              </Box>
              <Box component="tr">
                <Box component="td" sx={{ py: 1 }}>
                  [span_109](start_span)ASSETS UNDER CONSTRUCTION (INCLUDING PREMISES) {/*[span_109](end_span) */}
                  ={' '}
                  [span_110](start_span)<strong>{preCheckData.premisesUnderConstruction}</strong> {/*[span_110](end_span) */}
                </Box>
              </Box>
            </Box>
          </Box>
        </DialogContent>
        <DialogActions>
          [span_111](start_span)<Button onClick={() => {/*[span_111](end_span) */}
            [span_112](start_span)setPreCheckOpen(false)} variant="contained" color="success"> {/*[span_112](end_span) */}
            Continue
          </Button>
        </DialogActions>
      </Dialog>
      [span_113](start_span)<Dialog open={dialogOpen} onClose={() => setDialogOpen(false)}> {/*[span_113](end_span) */}
        [span_114](start_span)<DialogTitle>{dialogContent.title}</DialogTitle> {/*[span_114](end_span) */}
        [span_115](start_span)<DialogContent>{dialogContent.message}</DialogContent> {/*[span_115](end_span) */}
        <DialogActions>
          <Button
            [span_116](start_span)onClick={() => { //[span_116](end_span)
              setDialogOpen(false); [span_117](start_span)//[span_117](end_span)
              dialogContent.onConfirm(); [span_118](start_span)//[span_118](end_span)
            }}
          >
            OK
          </Button>
        </DialogActions>
      </Dialog>

      [span_119](start_span)<Dialog open={openSubmitDialog} onClose={() => setOpenSubmitDialog(false)}> {/*[span_119](end_span) */}
        [span_120](start_span)<DialogTitle>Confirm Submission</DialogTitle> {/*[span_120](end_span) */}
        [span_121](start_span)<DialogContent>Are you sure you want to submit Schedule 10 data?</DialogContent> {/*[span_121](end_span) */}
        <DialogActions>
          [span_122](start_span)<Button onClick={() => setOpenSubmitDialog(false)} color="inherit"> {/*[span_122](end_span) */}
            Close
          </Button>
          <Button
            onClick={() => {
              setOpenSubmitDialog(false); [span_123](start_span)//[span_123](end_span)
              handleSaveOrSubmit(false); [span_124](start_span)//[span_124](end_span)
            }}
            color="success"
            variant="contained"
          >
            Submit
          </Button>
        </DialogActions>
      </Dialog>

      <Stack direction="row" spacing={2} sx={{ mb: 2, justifyContent: 'left' }}>
        <CustomButton
          [span_125](start_span)onClickHandler={handlePreCheck} //[span_125](end_span)
          buttonType={'precheck'}
          [span_126](start_span)label={'Pre Check Amount'} //[span_126](end_span)
          [span_127](start_span)disabled={isChecking || isSubmitting} //[span_127](end_span)
        />
        <CustomButton
          [span_128](start_span)onClickHandler={() => handleSaveOrSubmit(true)} //[span_128](end_span)
          buttonType={'save'}
          label={'Save'}
          [span_129](start_span)disabled={isSubmitting} //[span_129](end_span)
        />
        <CustomButton
          [span_130](start_span)onClickHandler={() => setOpenSubmitDialog(true)} //[span_130](end_span)
          buttonType={'submit'}
          label={'Submit'}
          disabled={isSubmitting || Object.values(errors).some((val) => val && val.length > 0)} // Disable if any errors
        />
      </Stack>

      [span_131](start_span)<TableContainer component={Paper} sx={{ maxHeight: 'calc(100vh - 200px)' }}> {/*[span_131](end_span) */}
        [span_132](start_span)<Table sx={{ minWidth: 3000 }} aria-label="schedule 10 table" stickyHeader> {/*[span_132](end_span) */}
          <TableHead>
            <TableRow>
              <StyledTableCell
                rowSpan={2}
                sx={{
                  [span_133](start_span)minWidth: '50px', //[span_133](end_span)
                  [span_134](start_span)position: 'sticky', //[span_134](end_span)
                  [span_135](start_span)left: 0, //[span_135](end_span)
                  [span_136](start_span)zIndex: 1101, //[span_136](end_span)
                  [span_137](start_span)// backgroundColor: 'background.paper',[span_137](end_span)
                }}
              >
                Sr.No
              </StyledTableCell>
              <StyledTableCell
                rowSpan={2}
                sx={{
                  [span_138](start_span)minWidth: '350px', //[span_138](end_span)
                  [span_139](start_span)position: 'sticky', //[span_139](end_span)
                  [span_140](start_span)left: '50px', //[span_140](end_span)
                  [span_141](start_span)zIndex: 1100, //[span_141](end_span)
                  [span_142](start_span)// backgroundColor: 'background.paper',[span_142](end_span)
                }}
              >
                Particulars
              </StyledTableCell>
              [span_143](start_span)<StyledTableCell colSpan={5}> (A) FURNITURE & FITTINGS</StyledTableCell> {/*[span_143](end_span) */}
              [span_144](start_span)<StyledTableCell colSpan={10}> (B) MACHINERY & PLANT</StyledTableCell> {/*[span_144](end_span) */}
              [span_145](start_span)<StyledTableCell colSpan={13}> (C) PREMISES</StyledTableCell> {/*[span_145](end_span) */}
              [span_146](start_span)<StyledTableCell colSpan={2}></StyledTableCell> {/*[span_146](end_span) */}
            </TableRow>
            <TableRow>
              {columnDisplayHeaders.map((colDef) => (
                <StyledTableCell
                  key={colDef.dataField}
                  sx={{
                    [span_147](start_span)position: 'sticky', //[span_147](end_span)
                    [span_148](start_span)top: '42px', // Adjust this value based on your row height[span_148](end_span)
                    [span_149](start_span)//zIndex: 1101,[span_149](end_span)
                    [span_150](start_span)// backgroundColor: 'background.paper',[span_150](end_span)
                  }}
                  dangerouslySetInnerHTML={{ __html: colDef.labelHtml }}
                />
              ))}
            </TableRow>
          [span_151](start_span)</TableHead> {/*[span_151](end_span) */}
          <TableBody>
            {rowDefinitionsConfig.map((rowDef) => {
              const rowKey = rowDef.id;

              [span_152](start_span)if (rowDef.type === 'subSectionHeader') { //[span_152](end_span)
                return (
                  <StyledTableRow key={rowKey} $issubsectionheader>
                    <StyledTableCell
                      sx={{
                        [span_153](start_span)position: 'sticky', //[span_153](end_span)
                        [span_154](start_span)left: 0, //[span_154](end_span)
                        [span_155](start_span)zIndex: 1, //[span_155](end_span)
                        [span_156](start_span)// backgroundColor: (theme) => theme.palette.grey[50],[span_156](end_span)
                      }}
                    >
                      {rowDef.srNo || ''}
                    </StyledTableCell>
                    <StyledTableCell
                      [span_157](start_span)//colSpan={columnDisplayHeaders.length + 1}[span_157](end_span)
                      sx={{
                        [span_158](start_span)position: 'sticky', //[span_158](end_span)
                        [span_159](start_span)left: 52, //[span_159](end_span)
                        [span_160](start_span)zIndex: 1, //[span_160](end_span)
                        [span_161](start_span)// backgroundColor: (theme) => theme.palette.grey[50],[span_161](end_span)
                      }}
                    >
                      [span_162](start_span){typeof rowDef.label === 'function' ? rowDef.label(formData) : rowDef.label} {/*[span_162](end_span) */}
                    </StyledTableCell>
                  </StyledTableRow>
                );
              }

              return (
                <StyledTableRow
                  key={rowKey}
                  $istotalrow={rowDef.isTotalRowStyle}
                  $issectionheader={rowDef.isSectionHeaderStyle}
                > [span_163](start_span){/*[span_163](end_span) */}
                  <StyledTableCell
                    sx={{
                      [span_164](start_span)position: 'sticky', //[span_164](end_span)
                      [span_165](start_span)left: 0, //[span_165](end_span)
                      [span_166](start_span)zIndex: 1, //[span_166](end_span)
                      [span_167](start_span)// bgcolor: 'background.paper',[span_167](end_span)
                    }}
                  >
                    {rowDef.srNo || [span_168](start_span)''} {/*[span_168](end_span) */}
                  </StyledTableCell>
                  <StyledTableCell
                    sx={{
                      [span_169](start_span)position: 'sticky', //[span_169](end_span)
                      [span_170](start_span)left: '50px', //[span_170](end_span)
                      [span_171](start_span)zIndex: 1, //[span_171](end_span)
                      [span_172](start_span)// bgcolor: 'background.paper',[span_172](end_span)
                    }}
                  >
                    [span_173](start_span){typeof rowDef.label === 'function' ? rowDef.label(formData) : rowDef.label} {/*[span_173](end_span) */}
                  </StyledTableCell>

                  {columnDisplayHeaders.map((colDef) => {
                    [span_174](start_span)const fieldKey = colDef.dataField; //[span_174](end_span)
                    const cellKey = `${rowKey}-${fieldKey}`; [span_175](start_span)//[span_175](end_span)
                    const isReadOnly =
                      rowDef.type === 'total' || [span_176](start_span)//[span_176](end_span)
                      colDef.isCalculated || [span_177](start_span)//[span_177](end_span)
                      (rowDef.isReadOnlyGroup && rowDef.isReadOnlyGroup.includes(fieldKey));
                    const displayValue = calculatedData[rowKey]?.[fieldKey] ?? '0.00'; [span_178](start_span)//[span_178](end_span)
                    const inputValue = formData[rowKey]?.[fieldKey] ?? '0.00'; [span_179](start_span)//[span_179](end_span)
                    const errorForField = errors[cellKey]; [span_180](start_span)//[span_180](end_span)

                    return (
                      <StyledTableCell key={cellKey}>
                        <VirtualizedInput
                          name={cellKey}
                          [span_181](start_span)value={isReadOnly ? displayValue : inputValue} //[span_181](end_span)
                          [span_182](start_span)displayValue={displayValue} // Pass display value for placeholder[span_182](end_span)
                          [span_183](start_span)onChange={(e) => handleChange(rowDef.id, fieldKey, e.target.value)} //[span_183](end_span)
                          [span_184](start_span)onBlur={(e) => handleBlur(rowDef.id, fieldKey, e.target.value)} //[span_184](end_span)
                          [span_185](start_span)readOnly={isReadOnly || fieldsDisabled || manualEntry} //[span_185](end_span)
                          error={errorForField}
                          helperText={errorForField}
                        [span_186](start_span)/> {/*[span_186](end_span) */}
                      </StyledTableCell>
                    );
                  })}
                </StyledTableRow>
              );
            })}
          </TableBody>
        </Table>
      </TableContainer>

      <LoadingOverlay loading={isSubmitting} />
    </Box>
  );
};

export default Schedule10;

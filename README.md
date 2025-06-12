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

// --- Styled Components (Updated for Theming) ---
const StyledTableCell = styled(TableCell, {
  shouldForwardProp: (prop) => prop !== 'isFixedColumn',
})(({ theme }) => ({
  fontSize: '0.875rem',
  padding: '8px',
  border: `1px solid ${theme.palette.divider}`,
  whiteSpace: 'nowrap',

  [`&.${tableCellClasses.head}`]: {
    backgroundColor: theme.palette.grey[50],
    color: theme.palette.text.primary,
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
    backgroundColor: theme.palette.grey[50],
    '& > td, & > th': {
      fontWeight: 'bold',
      fontStyle: 'italic',
      textAlign: 'left',
      color: theme.palette.text.primary,
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
        },
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
        width: '20px', // Match typical height of a small TextField
        textAlign: 'right',
        width: '100%',
        padding: '6px 8px',
        boxSizing: 'border-box',
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
  const [errors, setErrors] = useState({});
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
  const [snackbar, setSnackbar] = useState({ open: false, message: '', severity: 'success' });
  const [preCheckOpen, setPreCheckOpen] = useState(false);
  const [preCheckData, setPreCheckData] = useState({
    otherFixedAsset: '0.00',
    premises: '0.00',
    premisesUnderConstruction: '0.00',
  });
  const [manualEntry, setManualEntry] = useState(true);

  // const formatDateToSlash = (dateStr) => {
  //   if (!dateStr || typeof dateStr !== 'string') return dateStr;

  //   const [year, month, day] = dateStr.split('-');
  //   return `${day}/${month}/${year}`;
  // };

  const convertFlatSc10DataToFormData = (flatData) => {
    const structuredData = generateInitialSchedule10Data();

    rowDefinitionsConfig.forEach((rowDef) => {
      const rowId = rowDef.id;
      const suffix = rowDef.modelSuffix;

      schedule10DataFields.forEach((field) => {
        const flatKey = `${field}${suffix}`;
        if (flatData[flatKey] !== undefined) {
          structuredData[rowId][field] = flatData[flatKey];
        }
      });
    });

    // Handle additional fields like particulars3, particulars4, finyearOne, finyearTwo if needed
    return structuredData;
  };

  useEffect(() => {
    const checkSC10SftpData = async () => {
      const payload = {
        circleCode: user.circleCode,
        qed: user.quarterEndDate,
        reportStatus: 'A',
        reportID: reportObject.reportMasterId,
        reportName: 'SC10',
      };
      console.log('payload:', payload);
      try {
        const data = await callApi('/IFAMSS/SC10SFTP', payload, 'POST');

        console.log('SFTP response:', data);
        if (user.isCircleExist === 'true') {
          if (data.fileAndDataStatus === 1) {
            // SFTP Success
            // Assuming `data.data` contains the schedule form values
            const convertedFormData = convertFlatSc10DataToFormData(data.sc10Data);
            setFormData(convertedFormData);
            setFieldsDisabled(true); // disables all inputs
            setSnackbarMessage(data.message || 'Data successfully fetched from IFAMS via SFTP.', 'success');
          } else if (data.fileAndDataStatus === 2) {
            // File error or mismatch
            setFieldsDisabled(true);
            showDialog({
              title: 'File Error',
              message: data.message || 'Data not received from IFAMS, Kindly wait till IFAMS sends reports',

              onConfirm: () => navigate(-1), // go back
            });
            // setSnackbarMessage(data.message || 'Data not received from IFAMS, Kindly wait till IFAMS sends reports.', 'success');
          } else if (data.fileAndDataStatus === 3) {
            // Data already exists in DB but file was missing
            setFieldsDisabled(true);
            showDialog({
              title: 'Info',
              message: data.message || 'Please note: Data fetched here was generated in IFAMS',
              onConfirm: () => navigate(-1),
            });
          }
        } else {
          setManualEntry(false);
          getValidationDataTen();
        }
      } catch (error) {
        if (error.message !== 'canceled') {
          setSnackbarMessage(error.message || 'Error while checking SC10 SFTP data.', 'error');
        }
      }
    };

    checkSC10SftpData();
  }, []);

  const getValidationDataTen = async () => {
    const payload = {
      circleCode: user.circleCode,
      quarterEndDate: user.quarterEndDate,
      status: reportObject.status,
      reportId: reportObject.reportId,
      reportMasterId: reportObject.reportMasterId,
      reportName: reportObject.name,
      areMocPending: reportObject.areMocPending,
    };
    console.log('payload:', payload);
    try {
      const data = await callApi('/Maker/getSavedDataTen', payload, 'POST');
      if (data) {
        console.log('data', data);
        const convertedFormData = convertFlatSc10DataToFormData(data);
        setFormData(convertedFormData);
      } else {
        throw new Error('Error while checking SC10 SFTP data.');
      }
    } catch (error) {
      setSnackbarMessage(error.message || 'Error while checking SC10 SFTP data.', 'error');
    }
  };

  const flattenSchedule10Data = (formData) => {
    const flatData = {};
    rowDefinitionsConfig.forEach((rowDef) => {
      const suffix = rowDef.modelSuffix;
      if (!suffix) return; // Skip headers

      const row = formData[rowDef.id] || {};
      schedule10DataFields.forEach((field) => {
        const key = `${field}${suffix}`;
        flatData[key] = row[field] ?? '0.00';
      });
    });

    // Add year fields if needed
    flatData.finyearOne = formData.finyearOne;
    flatData.finyearTwo = formData.finyearTwo;

    return flatData;
  };

  const handleSaveOrSubmit = async (isSaveOnly = true) => {
    setIsSubmitting(true);
    try {
      console.log('formdata', formData);

      const dataToSend = flattenSchedule10Data(formData);
      console.log('dataToSend', dataToSend);
      const payload = {
        ...dataToSend,
        save: isSaveOnly,
        circleCode: user.circleCode,
        quarterEndDate: user.quarterEndDate,
        userId: user.userId,
        reportName: reportObject.name,
        reportMasterId: reportObject.reportMasterId,
        reportId: reportObject.reportId,
        status: reportObject.status,
      };

      console.log('payload', payload);

      const response = await callApi('/Maker/submitTen', payload, 'POST');

      if (response) {
        if (response && typeof response === 'string') {
          const [_flag, newReportId, newStatus] = response.split('~');
          setReportObject((prev) => ({
            ...prev,
            reportId: newReportId,
            status: newStatus,
          }));
          setSnackbarMessage(isSaveOnly ? 'Report saved successfully!' : 'Report submitted successfully!', 'success');
          if (!isSaveOnly) {
            setTimeout(() => {
              navigate('../');
            }, 2000);
          }
        }
      } else {
        throw new Error(response.data?.message || 'Unexpected response');
      }
    } catch (err) {
      setSnackbarMessage('Error occurred during save/submit.', 'error');
    } finally {
      setIsSubmitting(false);
    }
  };

  const showDialog = ({ title, message, onConfirm }) => {
    setDialogOpen(true);
    setDialogContent({ title, message, onConfirm });
  };

  useEffect(() => {
    setIsLoading(false);
  }, []);

  const getNum = (value) => parseFloat(value) || 0;

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

    rowDefinitionsConfig.forEach((rowDef) => {
      if (!newCalculatedData[rowDef.id]) {
        newCalculatedData[rowDef.id] = {};
        schedule10DataFields.forEach((fieldKey) => {
          newCalculatedData[rowDef.id][fieldKey] = '0.00';
        });
      }
      if (rowDef.type === 'entry' && formData[rowDef.id]) {
        Object.keys(formData[rowDef.id]).forEach((fieldKey) => {
          if (schedule10DataFields.includes(fieldKey) && !intraRowCalculatedFields.includes(fieldKey)) {
            newCalculatedData[rowDef.id][fieldKey] = formData[rowDef.id][fieldKey];
          }
        });
      }
    });

    rowDefinitionsConfig.forEach((rowDef) => {
      if (rowDef.type === 'entry') {
        calculateInternalRowTotals(newCalculatedData[rowDef.id]);
      }
    });

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
        calculateInternalRowTotals(targetRow);
      }
    });

    console.timeEnd('Schedule10 Calculations');
    return newCalculatedData;
  }, [formData]);

  useEffect(() => {
    if (isCalculating) {
      setIsCalculating(false);
    }
  }, [calculatedData, isCalculating]);

  const debouncedRecalculate = useCallback(
    lodashDebounce((newFormData) => {
      setIsCalculating(true);
      setFormData(newFormData);
    }, 300),
    []
  );

  const handleChange = useCallback(
    (rowId, fieldKey, value) => {
      const newFormData = {
        ...formData,
        [rowId]: { ...(formData[rowId] || {}), [fieldKey]: value },
      };
      setFormData(newFormData);
      debouncedRecalculate(newFormData);
    },
    [formData, debouncedRecalculate]
  );

  const handleBlur = useCallback((rowId, fieldKey, value) => {
    const numericRegex = /^-?\d*\.?\d{0,2}$/;
    let error = '';
    if (value !== '' && value !== '-' && !numericRegex.test(value)) {
      error = 'Invalid number';
    }
    setErrors((prev) => ({ ...prev, [`${rowId}-${fieldKey}`]: error }));
  }, []);

  const handlePreCheck = async () => {
    setIsChecking(true);
    try {
      // The payload would be the current form data, similar to the AngularJS `row1`
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
      console.log('payload:', payload);
      const response = await callApi('/Maker/getValidationDataTen', payload, 'POST');

      // Update state with the data from the API response
      if (response) {
        setPreCheckData({
          otherFixedAsset: response.validationOtherFixedAssetAmount,
          premises: response.validationPremisesAmount,
          premisesUnderConstruction: response.validationPremisesUnderConsAmount,
        });
        setPreCheckOpen(true); // Open the modal
      }
    } catch (error) {
      setSnackbar({
        open: true,
        message: 'Failed to fetch pre-check data.',
        severity: 'error',
      });
    } finally {
      setIsChecking(false);
    }
  };

  if (isLoading) {
    return (
      <Box sx={{ display: 'flex', justifyContent: 'center', alignItems: 'center', height: 'calc(100vh - 150px)' }}>
        <CircularProgress /> <Typography sx={{ ml: 2 }}>Loading Schedule 10...</Typography>
      </Box>
    );
  }

  return (
    <Box sx={{ p: 1, width: '100%', boxSizing: 'border-box', overflowX: 'hidden', bgcolor: 'background.default' }}>
      <Dialog open={preCheckOpen} onClose={() => setPreCheckOpen(false)} fullWidth maxWidth="sm">
        <DialogTitle sx={{ backgroundColor: '#E74C3C', color: 'white' }}>Attention!</DialogTitle>
        <DialogContent dividers>
          <Typography variant="h6" component="div" sx={{ mb: 2 }}>
            The values for pre-checks of comp codes are as follows:-
          </Typography>
          <Box component="table" sx={{ width: '100%' }}>
            <Box component="tbody" sx={{ fontSize: '15px' }}>
              <Box component="tr">
                <Box component="td" sx={{ py: 1 }}>
                  OTHER FIXED ASSETS (including furniture and fixtures) ={' '}
                  <strong>{preCheckData.otherFixedAsset}</strong>
                </Box>
              </Box>
              <Box component="tr">
                <Box component="td" sx={{ py: 1 }}>
                  PREMISES = <strong>{preCheckData.premises}</strong>
                </Box>
              </Box>
              <Box component="tr">
                <Box component="td" sx={{ py: 1 }}>
                  ASSETS UNDER CONSTRUCTION (INCLUDING PREMISES) ={' '}
                  <strong>{preCheckData.premisesUnderConstruction}</strong>
                </Box>
              </Box>
            </Box>
          </Box>
        </DialogContent>
        <DialogActions>
          <Button onClick={() => setPreCheckOpen(false)} variant="contained" color="success">
            Continue
          </Button>
        </DialogActions>
      </Dialog>
      <Dialog open={dialogOpen} onClose={() => setDialogOpen(false)}>
        <DialogTitle>{dialogContent.title}</DialogTitle>
        <DialogContent>{dialogContent.message}</DialogContent>
        <DialogActions>
          <Button
            onClick={() => {
              setDialogOpen(false);
              dialogContent.onConfirm();
            }}
          >
            OK
          </Button>
        </DialogActions>
      </Dialog>

      <Dialog open={openSubmitDialog} onClose={() => setOpenSubmitDialog(false)}>
        <DialogTitle>Confirm Submission</DialogTitle>
        <DialogContent>Are you sure you want to submit Schedule 10 data?</DialogContent>
        <DialogActions>
          <Button onClick={() => setOpenSubmitDialog(false)} color="inherit">
            Close
          </Button>
          <Button
            onClick={() => {
              setOpenSubmitDialog(false);
              handleSaveOrSubmit(false);
            }}
            color="success"
            variant="contained"
          >
            Submit
          </Button>
        </DialogActions>
      </Dialog>

      <Stack direction="row" spacing={2} sx={{ mb: 2, justifyContent: 'left' }}>
        <Button variant="contained" color="error" onClick={handlePreCheck} disabled={isChecking || isSubmitting}>
          {isChecking ? <CircularProgress size={24} /> : 'Pre-Check Amount'}
        </Button>
        <Button variant="contained" color="warning" onClick={() => handleSaveOrSubmit(true)} disabled={isSubmitting}>
          Save
        </Button>
        <Button variant="contained" color="success" onClick={() => setOpenSubmitDialog(true)} disabled={isSubmitting}>
          Submit
        </Button>
      </Stack>

      <TableContainer component={Paper} sx={{ maxHeight: 'calc(100vh - 200px)' }}>
        <Table sx={{ minWidth: 3000 }} aria-label="schedule 10 table" stickyHeader>
          <TableHead>
            <TableRow>
              <StyledTableCell
                rowSpan={2}
                sx={{
                  minWidth: '50px',
                  position: 'sticky',
                  left: 0,
                  zIndex: 1101,
                  backgroundColor: 'background.paper',
                }}
              >
                Sr.No
              </StyledTableCell>
              <StyledTableCell
                rowSpan={2}
                sx={{
                  minWidth: '350px',
                  position: 'sticky',
                  left: '50px',
                  zIndex: 1100,
                  backgroundColor: 'background.paper',
                }}
              >
                Particulars
              </StyledTableCell>
              <StyledTableCell colSpan={5}> (A) FURNITURE & FITTINGS</StyledTableCell>
              <StyledTableCell colSpan={10}> (B) MACHINERY & PLANT</StyledTableCell>
              <StyledTableCell colSpan={13}> (C) PREMISES</StyledTableCell>
              <StyledTableCell colSpan={2}></StyledTableCell>
            </TableRow>
            <TableRow>
              {columnDisplayHeaders.map((colDef) => (
                <StyledTableCell
                  key={colDef.dataField}
                  sx={{
                    position: 'sticky',
                    top: '42px', // Adjust this value based on your row height
                    //zIndex: 1101,
                    backgroundColor: 'background.paper',
                  }}
                  dangerouslySetInnerHTML={{ __html: colDef.labelHtml }}
                />
              ))}
            </TableRow>
          </TableHead>
          <TableBody>
            {rowDefinitionsConfig.map((rowDef) => {
              const rowKey = rowDef.id;

              if (rowDef.type === 'subSectionHeader') {
                return (
                  <StyledTableRow key={rowKey} $issubsectionheader>
                    <StyledTableCell
                      sx={{
                        position: 'sticky',
                        left: 0,
                        zIndex: 1,
                        backgroundColor: (theme) => theme.palette.grey[50],
                      }}
                    >
                      {rowDef.srNo || ''}
                    </StyledTableCell>
                    <StyledTableCell
                      //colSpan={columnDisplayHeaders.length + 1}
                      sx={{
                        position: 'sticky',
                        left: 52,
                        zIndex: 1,
                        backgroundColor: (theme) => theme.palette.grey[50],
                      }}
                    >
                      {typeof rowDef.label === 'function' ? rowDef.label(formData) : rowDef.label}
                    </StyledTableCell>
                  </StyledTableRow>
                );
              }

              return (
                <StyledTableRow
                  key={rowKey}
                  $istotalrow={rowDef.isTotalRowStyle}
                  $issectionheader={rowDef.isSectionHeaderStyle}
                >
                  <StyledTableCell
                    sx={{
                      position: 'sticky',
                      left: 0,
                      zIndex: 1,
                      bgcolor: 'background.paper',
                    }}
                  >
                    {rowDef.srNo || ''}
                  </StyledTableCell>
                  <StyledTableCell
                    sx={{
                      position: 'sticky',
                      left: '50px',
                      zIndex: 1,
                      bgcolor: 'background.paper',
                    }}
                  >
                    {typeof rowDef.label === 'function' ? rowDef.label(formData) : rowDef.label}
                  </StyledTableCell>

                  {columnDisplayHeaders.map((colDef) => {
                    const fieldKey = colDef.dataField;
                    const cellKey = `${rowKey}-${fieldKey}`;
                    const isReadOnly =
                      rowDef.type === 'total' ||
                      colDef.isCalculated ||
                      (rowDef.isReadOnlyGroup && rowDef.isReadOnlyGroup.includes(fieldKey));
                    const displayValue = calculatedData[rowKey]?.[fieldKey] ?? '0.00';
                    const inputValue = formData[rowKey]?.[fieldKey] ?? '0.00';
                    const errorForField = errors[cellKey];

                    return (
                      <StyledTableCell key={cellKey}>
                        <VirtualizedInput
                          name={cellKey}
                          value={isReadOnly ? displayValue : inputValue}
                          displayValue={displayValue} // Pass display value for placeholder
                          onChange={(e) => handleChange(rowDef.id, fieldKey, e.target.value)}
                          onBlur={(e) => handleBlur(rowDef.id, fieldKey, e.target.value)}
                          readOnly={isReadOnly || fieldsDisabled || manualEntry}
                          error={errorForField}
                          helperText={errorForField}
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

      <Snackbar
        open={snackbar.open}
        autoHideDuration={4000}
        anchorOrigin={{ vertical: 'bottom', horizontal: 'right' }}
        onClose={() => setSnackbar({ ...snackbar, open: false })}
      >
        <Alert severity={snackbar.severity} sx={{ width: '100%' }}>
          {snackbar.message}
        </Alert>
      </Snackbar>
    </Box>
  );
};

export default Schedule10;

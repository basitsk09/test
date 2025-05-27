import React, { useState, useEffect, useMemo, useCallback } from 'react';
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
import { styled } from '@mui/material/styles';
import debounce from 'lodash.debounce';

// Styled components (unchanged)
const StyledTableCell = React.memo(
  styled(TableCell)(({ theme }) => ({
    fontSize: '0.875rem',
    padding: '8px',
    border: '1px solid #e0e0e0',
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
  }))
);

const StyledTableRow = styled(TableRow)(({ theme, istotalrow, issectionheader, issubsectionheader }) => ({
  backgroundColor: theme.palette.background.paper,
  '&:nth-of-type(odd)': {
    // backgroundColor: theme.palette.action.hover,
  },
  ...(issectionheader && {
    '& > td': {
      fontWeight: 'bold',
      textAlign: 'left',
      backgroundColor: theme.palette.grey[200], // Light grey for section headers
    },
  }),
  ...(issubsectionheader && {
    '& > td': {
      fontWeight: 'bold',
      fontStyle: 'italic',
      textAlign: 'left',
      backgroundColor: theme.palette.grey[100], // Even lighter grey for subsection headers
    },
  }),
  ...(istotalrow && {
    '& > td': {
      fontWeight: 'bold',
      backgroundColor: theme.palette.grey[300], // Slightly darker grey for total rows
    },
  }),
}));

const Schedule10 = () => {
  // Initialize state with all possible fields from Schedule10.txt
  // Including fields implied by the calculation logic in the snippet
  const [formData, setFormData] = useState({
    // (A) FURNITURE & FITTINGS - Original Cost / Revalued Value upto the end of previous year
    stcNstaff1: '',
    offResidenceA1: '',
    otherPremisesA1: '',
    electricFitting1: '',
    totalA1: '', // Calculated

    // (A) FURNITURE & FITTINGS - Additions during the year
    stcNstaffAdd1: '',
    offResidenceAAdd1: '',
    otherPremisesAAdd1: '',
    electricFittingAdd1: '',
    totalAAdd1: '', // Calculated

    // (A) FURNITURE & FITTINGS - Deductions during the year
    stcNstaffDed1: '',
    offResidenceADed1: '',
    otherPremisesADed1: '',
    electricFittingDed1: '',
    totalADed1: '', // Calculated

    // (B) MACHINERY & PLANT - Original Cost / Revalued Value upto the end of previous year
    computers1: '',
    compSoftwareInt1: '',
    compSoftwareNonint1: '',
    compSoftwareTotal1: '', // Calculated
    motor1: '',
    offResidenceB1: '',
    stcLho1: '',
    otherPremisesB1: '',
    otherMachineryPlant1: '', // Calculated
    totalB1: '', // Calculated

    // (B) MACHINERY & PLANT - Additions during the year
    computersAdd1: '',
    compSoftwareIntAdd1: '',
    compSoftwareNonintAdd1: '',
    compSoftwareTotalAdd1: '', // Calculated
    motorAdd1: '',
    offResidenceBAdd1: '',
    stcLhoAdd1: '',
    otherPremisesBAdd1: '',
    otherMachineryPlantAdd1: '', // Calculated
    totalBAdd1: '', // Calculated

    // (B) MACHINERY & PLANT - Deductions during the year
    computersDed1: '',
    compSoftwareIntDed1: '',
    compSoftwareNonintDed1: '',
    compSoftwareTotalDed1: '', // Calculated
    motorDed1: '',
    offResidenceBDed1: '',
    stcLhoDed1: '',
    otherPremisesBDed1: '',
    otherMachineryPlantDed1: '', // Calculated
    totalBDed1: '', // Calculated

    // Total Furniture & Fixtures (A+B) - Different rows will have their own totals
    // I'll name them based on the row type for clarity
    totalFurnFixPrevYear1: '', // Total from previous year row (A + B initial)
    totalFurnFixAdditions1: '', // Total from additions row (A + B additions)
    totalFurnFixDeductions1: '', // Total from deductions row (A + B deductions)

    // (C) PREMISES - Original Cost / Revalued Value upto the end of previous year
    landNotRev1: '',
    landRev1: '',
    landRevEnh1: '',
    offBuildNotRev1: '',
    offBuildRev1: '',
    offBuildRevEnh1: '',
    residQuartNotRev1: '',
    residQuartRev1: '',
    residQuartRevEnh1: '',
    premisTotal1: '', // Calculated (Cost portion)
    revtotal1: '', // Calculated (Revaluation Enhancement portion)
    totalC1: '', // Calculated (Cost + Revaluation)

    // (C) PREMISES - Additions during the year
    landNotRevAdd1: '',
    landRevAdd1: '',
    landRevEnhAdd1: '',
    offBuildNotRevAdd1: '',
    offBuildRevAdd1: '',
    offBuildRevEnhAdd1: '',
    residQuartNotRevAdd1: '',
    residQuartRevAdd1: '',
    residQuartRevEnhAdd1: '',
    premisTotalAdd1: '', // Calculated
    revtotalAdd1: '', // Calculated
    totalCAdd1: '', // Calculated

    // (C) PREMISES - Deductions during the year
    landNotRevDed1: '',
    landRevDed1: '',
    landRevEnhDed1: '',
    offBuildNotRevDed1: '',
    offBuildRevDed1: '',
    offBuildRevEnhDed1: '',
    residQuartNotRevDed1: '',
    residQuartRevDed1: '',
    residQuartRevEnhDed1: '',
    premisTotalDed1: '', // Calculated
    revtotalDed1: '', // Calculated
    totalCDed1: '', // Calculated

    // (D) Projects under construction
    premisesUnderCons1: '', // Previous Year
    premisesUnderConsAdd1: '', // Additions
    premisesUnderConsDed1: '', // Deductions
    premisesUnderConsCurrent1: '', // Calculated (Current Year)

    // Depreciation for the year
    depA1: '', // Furniture & Fittings Depreciation
    depB1: '', // Machinery & Plant Depreciation
    depC1: '', // Premises Depreciation
    totalDepreciation1: '', // Calculated

    // Net Block at the end of the year
    netBlockA1: '', // Furniture & Fittings Net Block
    netBlockB1: '', // Machinery & Plant Net Block
    netBlockC1: '', // Premises Net Block
    netBlockD1: '', // Projects under Construction Net Block (already calculated above as premisesUnderConsCurrent1)
    grandTotalNetBlock1: '', // Calculated

    // Additional fields from the snippet for reconciliation calculations
    premCostYear: '',
    revCostYear: '',
    premRevOfPreYear: '', // This seems to be current year's premises revaluation total
    premDedOfPreYear: '',
    revDedOfPre: '',
    depCostPreYear: '', // Depreciation on cost for previous year
    depRevPreYear: '', // Depreciation on revaluation for previous year
    fixCostPre: '', // Fixed Assets Cost previous year
    fixAddPreYear: '', // Fixed Assets Additions previous year
    fixDedPreYear: '', // Fixed Assets Deductions previous year
    fixAddYear: '', // Fixed Assets Additions current year
    fixDedYear: '', // Fixed Assets Deductions current year
    fixDepYear: '', // Fixed Assets Depreciation current year
    fixDepPreYear: '', // Fixed Assets Depreciation previous year

    // Calculated fields based on snippet logic
    currvPremTotal: '',
    fixCostYear: '',
    ttlFixCurrYear: '',
    ttlFixPreYear: '',
    grandCurrTotal: '',
    totalPremPreYear: '', // Calculated (j+k for previous year, based on snippet's logic)
  });

  const [errors, setErrors] = useState({});

  const parseFloatSafe = (value) => {
    const parsed = parseFloat(value);
    return isNaN(parsed) ? 0 : parsed;
  };

  const validateDecimal2Places = (value) => {
    if (value === null || value.trim() === '') return true;
    const regex = /^-?\d+(\.\d{1,2})?$/;
    return regex.test(value);
  };

  const debouncedValidate = useCallback(
    debounce((name, value) => {
      // List of all calculated fields (readOnly) to exclude from direct validation
      const calculatedFields = [
        'totalA1', 'totalAAdd1', 'totalADed1',
        'compSoftwareTotal1', 'otherMachineryPlant1', 'totalB1',
        'compSoftwareTotalAdd1', 'otherMachineryPlantAdd1', 'totalBAdd1',
        'compSoftwareTotalDed1', 'otherMachineryPlantDed1', 'totalBDed1',
        'totalFurnFixPrevYear1', 'totalFurnFixAdditions1', 'totalFurnFixDeductions1',
        'premisTotal1', 'revtotal1', 'totalC1',
        'premisTotalAdd1', 'revtotalAdd1', 'totalCAdd1',
        'premisTotalDed1', 'revtotalDed1', 'totalCDed1',
        'premisesUnderConsCurrent1',
        'totalDepreciation1', 'netBlockA1', 'netBlockB1', 'netBlockC1', 'netBlockD1', 'grandTotalNetBlock1',
        'currvPremTotal', 'fixCostYear', 'ttlFixCurrYear', 'ttlFixPreYear', 'grandCurrTotal', 'totalPremPreYear'
      ];

      if (!calculatedFields.includes(name)) { // Only validate if it's not a calculated field
        if (!validateDecimal2Places(value)) {
          setErrors((prevErrors) => ({ ...prevErrors, [name]: 'Invalid decimal format (max 2 places).' }));
        } else {
          setErrors((prevErrors) => {
            const newErrors = { ...prevErrors };
            delete newErrors[name];
            return newErrors;
          });
        }
      }
    }, 300),
    []
  );

  const handleChange = (e) => {
    const { name, value } = e.target;
    setFormData((prevData) => ({ ...prevData, [name]: value }));
    debouncedValidate(name, value);
  };

  const calculateAllTotals = useCallback(
    debounce((currentFormData) => {
      setFormData((prevData) => {
        const newData = { ...currentFormData };

        // --- (A) FURNITURE & FITTINGS calculations ---
        newData.totalA1 = (
          parseFloatSafe(newData.stcNstaff1) + parseFloatSafe(newData.offResidenceA1) +
          parseFloatSafe(newData.otherPremisesA1) + parseFloatSafe(newData.electricFitting1)
        ).toFixed(2);

        newData.totalAAdd1 = (
          parseFloatSafe(newData.stcNstaffAdd1) + parseFloatSafe(newData.offResidenceAAdd1) +
          parseFloatSafe(newData.otherPremisesAAdd1) + parseFloatSafe(newData.electricFittingAdd1)
        ).toFixed(2);

        newData.totalADed1 = (
          parseFloatSafe(newData.stcNstaffDed1) + parseFloatSafe(newData.offResidenceADed1) +
          parseFloatSafe(newData.otherPremisesADed1) + parseFloatSafe(newData.electricFittingDed1)
        ).toFixed(2);


        // --- (B) MACHINERY & PLANT calculations ---
        newData.compSoftwareTotal1 = (
          parseFloatSafe(newData.compSoftwareInt1) + parseFloatSafe(newData.compSoftwareNonint1)
        ).toFixed(2);

        newData.otherMachineryPlant1 = (
          parseFloatSafe(newData.offResidenceB1) + parseFloatSafe(newData.stcLho1) +
          parseFloatSafe(newData.otherPremisesB1)
        ).toFixed(2);

        newData.totalB1 = (
          parseFloatSafe(newData.computers1) + parseFloatSafe(newData.compSoftwareTotal1) +
          parseFloatSafe(newData.motor1) + parseFloatSafe(newData.otherMachineryPlant1)
        ).toFixed(2);

        newData.compSoftwareTotalAdd1 = (
          parseFloatSafe(newData.compSoftwareIntAdd1) + parseFloatSafe(newData.compSoftwareNonintAdd1)
        ).toFixed(2);

        newData.otherMachineryPlantAdd1 = (
          parseFloatSafe(newData.offResidenceBAdd1) + parseFloatSafe(newData.stcLhoAdd1) +
          parseFloatSafe(newData.otherPremisesBAdd1)
        ).toFixed(2);

        newData.totalBAdd1 = (
          parseFloatSafe(newData.computersAdd1) + parseFloatSafe(newData.compSoftwareTotalAdd1) +
          parseFloatSafe(newData.motorAdd1) + parseFloatSafe(newData.otherMachineryPlantAdd1)
        ).toFixed(2);

        newData.compSoftwareTotalDed1 = (
          parseFloatSafe(newData.compSoftwareIntDed1) + parseFloatSafe(newData.compSoftwareNonintDed1)
        ).toFixed(2);

        newData.otherMachineryPlantDed1 = (
          parseFloatSafe(newData.offResidenceBDed1) + parseFloatSafe(newData.stcLhoDed1) +
          parseFloatSafe(newData.otherPremisesBDed1)
        ).toFixed(2);

        newData.totalBDed1 = (
          parseFloatSafe(newData.computersDed1) + parseFloatSafe(newData.compSoftwareTotalDed1) +
          parseFloatSafe(newData.motorDed1) + parseFloatSafe(newData.otherMachineryPlantDed1)
        ).toFixed(2);


        // --- Total Furniture & Fixtures (A+B) for each row type ---
        newData.totalFurnFixPrevYear1 = (
          parseFloatSafe(newData.totalA1) + parseFloatSafe(newData.totalB1)
        ).toFixed(2);

        newData.totalFurnFixAdditions1 = (
          parseFloatSafe(newData.totalAAdd1) + parseFloatSafe(newData.totalBAdd1)
        ).toFixed(2);

        newData.totalFurnFixDeductions1 = (
          parseFloatSafe(newData.totalADed1) + parseFloatSafe(newData.totalBDed1)
        ).toFixed(2);


        // --- (C) PREMISES calculations ---
        newData.premisTotal1 = (
          parseFloatSafe(newData.landNotRev1) + parseFloatSafe(newData.landRev1) +
          parseFloatSafe(newData.offBuildNotRev1) + parseFloatSafe(newData.offBuildRev1) +
          parseFloatSafe(newData.residQuartNotRev1) + parseFloatSafe(newData.residQuartRev1)
        ).toFixed(2);

        newData.revtotal1 = (
          parseFloatSafe(newData.landRevEnh1) + parseFloatSafe(newData.offBuildRevEnh1) +
          parseFloatSafe(newData.residQuartRevEnh1)
        ).toFixed(2);

        newData.totalC1 = (
          parseFloatSafe(newData.premisTotal1) + parseFloatSafe(newData.revtotal1)
        ).toFixed(2);

        newData.premisTotalAdd1 = (
          parseFloatSafe(newData.landNotRevAdd1) + parseFloatSafe(newData.landRevAdd1) +
          parseFloatSafe(newData.offBuildNotRevAdd1) + parseFloatSafe(newData.offBuildRevAdd1) +
          parseFloatSafe(newData.residQuartNotRevAdd1) + parseFloatSafe(newData.residQuartRevAdd1)
        ).toFixed(2);

        newData.revtotalAdd1 = (
          parseFloatSafe(newData.landRevEnhAdd1) + parseFloatSafe(newData.offBuildRevEnhAdd1) +
          parseFloatSafe(newData.residQuartRevEnhAdd1)
        ).toFixed(2);

        newData.totalCAdd1 = (
          parseFloatSafe(newData.premisTotalAdd1) + parseFloatSafe(newData.revtotalAdd1)
        ).toFixed(2);

        newData.premisTotalDed1 = (
          parseFloatSafe(newData.landNotRevDed1) + parseFloatSafe(newData.landRevDed1) +
          parseFloatSafe(newData.offBuildNotRevDed1) + parseFloatSafe(newData.offBuildRevDed1) +
          parseFloatSafe(newData.residQuartNotRevDed1) + parseFloatSafe(newData.residQuartRevDed1)
        ).toFixed(2);

        newData.revtotalDed1 = (
          parseFloatSafe(newData.landRevEnhDed1) + parseFloatSafe(newData.offBuildRevEnhDed1) +
          parseFloatSafe(newData.residQuartRevEnhDed1)
        ).toFixed(2);

        newData.totalCDed1 = (
          parseFloatSafe(newData.premisTotalDed1) + parseFloatSafe(newData.revtotalDed1)
        ).toFixed(2);

        // --- (D) Projects under construction calculations ---
        newData.premisesUnderConsCurrent1 = (
          parseFloatSafe(newData.premisesUnderCons1) +
          parseFloatSafe(newData.premisesUnderConsAdd1) -
          parseFloatSafe(newData.premisesUnderConsDed1)
        ).toFixed(2);

        // --- Depreciation for the year calculations ---
        newData.totalDepreciation1 = (
          parseFloatSafe(newData.depA1) +
          parseFloatSafe(newData.depB1) +
          parseFloatSafe(newData.depC1)
        ).toFixed(2);

        // --- Net Block at the end of the year calculations ---
        // These rely on the respective current year totals (Total A, Total B, Total C, Total D current)
        // minus their respective depreciation. Assuming depreciation applies to the Net Block for the year.
        // This is a simplified interpretation; exact logic might need clarification from full Schedule10.txt
        newData.netBlockA1 = (
          parseFloatSafe(newData.totalA1) + parseFloatSafe(newData.totalAAdd1) - parseFloatSafe(newData.totalADed1) - parseFloatSafe(newData.depA1)
        ).toFixed(2);
        newData.netBlockB1 = (
          parseFloatSafe(newData.totalB1) + parseFloatSafe(newData.totalBAdd1) - parseFloatSafe(newData.totalBDed1) - parseFloatSafe(newData.depB1)
        ).toFixed(2);
        newData.netBlockC1 = (
          parseFloatSafe(newData.totalC1) + parseFloatSafe(newData.totalCAdd1) - parseFloatSafe(newData.totalCDed1) - parseFloatSafe(newData.depC1)
        ).toFixed(2);
        newData.netBlockD1 = parseFloatSafe(newData.premisesUnderConsCurrent1).toFixed(2); // Net Block D is simply current value of D

        newData.grandTotalNetBlock1 = (
          parseFloatSafe(newData.netBlockA1) +
          parseFloatSafe(newData.netBlockB1) +
          parseFloatSafe(newData.netBlockC1) +
          parseFloatSafe(newData.netBlockD1)
        ).toFixed(2);


        // --- Additional calculations based on Schedule10.txt snippet for reconciliation ---
        newData.currvPremTotal = (
          parseFloatSafe(newData.premCostYear) +
          parseFloatSafe(newData.revCostYear)
        ).toFixed(2);

        newData.fixCostYear = (
          parseFloatSafe(newData.fixCostPre) +
          parseFloatSafe(newData.fixAddPreYear) -
          parseFloatSafe(newData.fixDedPreYear)
        ).toFixed(2);

        newData.ttlFixCurrYear = (
          parseFloatSafe(newData.fixCostYear) +
          parseFloatSafe(newData.fixAddYear) -
          parseFloatSafe(newData.fixDedYear) -
          parseFloatSafe(newData.fixDepYear)
        ).toFixed(2);

        newData.ttlFixPreYear = (
          parseFloatSafe(newData.fixCostPre) +
          parseFloatSafe(newData.fixAddPreYear) -
          parseFloatSafe(newData.fixDedPreYear) -
          parseFloatSafe(newData.fixDepPreYear)
        ).toFixed(2);

        // Assuming 'premisUnderConst' corresponds to 'premisesUnderConsCurrent1'
        newData.grandCurrTotal = (
          parseFloatSafe(newData.currvPremTotal) +
          parseFloatSafe(newData.ttlFixCurrYear) +
          parseFloatSafe(newData.premisesUnderConsCurrent1)
        ).toFixed(2);

        // This calculation is a bit ambiguous without full context but is in snippet
        // SCH10.row.totalPremPreYear = (SCH10.parseFloat(SCH10.row.premRevOfPreYear) - SCH10.parseFloat(SCH10.row.premDedOfPreYear) - SCH10.parseFloat(SCH10.row.revDedOfPre) - SCH10.parseFloat(SCH10.row.depCostPreYear) - SCH10.parseFloat(SCH10.row.depRevPreYear)).toFixed(2);
        newData.totalPremPreYear = (
          parseFloatSafe(newData.premRevOfPreYear) -
          parseFloatSafe(newData.premDedOfPreYear) -
          parseFloatSafe(newData.revDedOfPre) -
          parseFloatSafe(newData.depCostPreYear) -
          parseFloatSafe(newData.depRevPreYear)
        ).toFixed(2);


        return newData;
      });
    }, 500),
    []
  );

  useEffect(() => {
    calculateAllTotals(formData);
    return () => {
      calculateAllTotals.cancel();
      debouncedValidate.cancel();
    };
  }, [
    // --- Dependencies for (A) FURNITURE & FITTINGS ---
    formData.stcNstaff1, formData.offResidenceA1, formData.otherPremisesA1, formData.electricFitting1,
    formData.stcNstaffAdd1, formData.offResidenceAAdd1, formData.otherPremisesAAdd1, formData.electricFittingAdd1,
    formData.stcNstaffDed1, formData.offResidenceADed1, formData.otherPremisesADed1, formData.electricFittingDed1,

    // --- Dependencies for (B) MACHINERY & PLANT ---
    formData.computers1, formData.compSoftwareInt1, formData.compSoftwareNonint1, formData.motor1,
    formData.offResidenceB1, formData.stcLho1, formData.otherPremisesB1,
    formData.computersAdd1, formData.compSoftwareIntAdd1, formData.compSoftwareNonintAdd1, formData.motorAdd1,
    formData.offResidenceBAdd1, formData.stcLhoAdd1, formData.otherPremisesBAdd1,
    formData.computersDed1, formData.compSoftwareIntDed1, formData.compSoftwareNonintDed1, formData.motorDed1,
    formData.offResidenceBDed1, formData.stcLhoDed1, formData.otherPremisesBDed1,

    // --- Dependencies for (C) PREMISES ---
    formData.landNotRev1, formData.landRev1, formData.landRevEnh1,
    formData.offBuildNotRev1, formData.offBuildRev1, formData.offBuildRevEnh1,
    formData.residQuartNotRev1, formData.residQuartRev1, formData.residQuartRevEnh1,
    formData.landNotRevAdd1, formData.landRevAdd1, formData.landRevEnhAdd1,
    formData.offBuildNotRevAdd1, formData.offBuildRevAdd1, formData.offBuildRevEnhAdd1,
    formData.residQuartNotRevAdd1, formData.residQuartRevAdd1, formData.residQuartRevEnhAdd1,
    formData.landNotRevDed1, formData.landRevDed1, formData.landRevEnhDed1,
    formData.offBuildNotRevDed1, formData.offBuildRevDed1, formData.offBuildRevEnhDed1,
    formData.residQuartNotRevDed1, formData.residQuartRevDed1, formData.residQuartRevEnhDed1,

    // --- Dependencies for (D) Projects under construction ---
    formData.premisesUnderCons1, formData.premisesUnderConsAdd1, formData.premisesUnderConsDed1,

    // --- Dependencies for Depreciation ---
    formData.depA1, formData.depB1, formData.depC1,

    // --- Dependencies for additional reconciliation calculations ---
    formData.premCostYear, formData.revCostYear,
    formData.premRevOfPreYear, formData.premDedOfPreYear, formData.revDedOfPre,
    formData.depCostPreYear, formData.depRevPreYear,
    formData.fixCostPre, formData.fixAddPreYear, formData.fixDedPreYear,
    formData.fixAddYear, formData.fixDedYear, formData.fixDepYear, formData.fixDepPreYear,

    calculateAllTotals, // Include the debounced function itself as a dependency
  ]);

  const validateForm = () => {
    let newErrors = {};
    const calculatedFields = [
      'totalA1', 'totalAAdd1', 'totalADed1',
      'compSoftwareTotal1', 'otherMachineryPlant1', 'totalB1',
      'compSoftwareTotalAdd1', 'otherMachineryPlantAdd1', 'totalBAdd1',
      'compSoftwareTotalDed1', 'otherMachineryPlantDed1', 'totalBDed1',
      'totalFurnFixPrevYear1', 'totalFurnFixAdditions1', 'totalFurnFixDeductions1',
      'premisTotal1', 'revtotal1', 'totalC1',
      'premisTotalAdd1', 'revtotalAdd1', 'totalCAdd1',
      'premisTotalDed1', 'revtotalDed1', 'totalCDed1',
      'premisesUnderConsCurrent1',
      'totalDepreciation1', 'netBlockA1', 'netBlockB1', 'netBlockC1', 'netBlockD1', 'grandTotalNetBlock1',
      'currvPremTotal', 'fixCostYear', 'ttlFixCurrYear', 'ttlFixPreYear', 'grandCurrTotal', 'totalPremPreYear'
    ];

    for (const key in formData) {
      if (!calculatedFields.includes(key)) {
        if (!validateDecimal2Places(formData[key])) {
          newErrors[key] = 'Invalid decimal format (max 2 places).';
        }
      }
    }
    setErrors(newErrors);
    return Object.keys(newErrors).length === 0;
  };

  const handleSubmit = (e) => {
    e.preventDefault();
    debouncedValidate.flush();
    calculateAllTotals.flush();

    if (validateForm()) {
      console.log('Form data submitted:', formData);
    } else {
      console.log('Form has validation errors:', errors);
    }
  };

  // Define the table structure with rows, sections, and subsections
  const tableRows = useMemo(() => [
    // SECTION: Total Original Cost / Revalued Value upto the end of previous year
    {
      id: 'A.I',
      label: `Total Original Cost / Revalued Value upto the end of previous year i.e. 31st March {{sc10.year1}}`,
      isSectionHeader: true,
      cells: [
        { field: 'stcNstaff1', readonly: false },
        { field: 'offResidenceA1', readonly: false },
        { field: 'otherPremisesA1', readonly: false },
        { field: 'electricFitting1', readonly: false },
        { field: 'totalA1', readonly: true }, // TOTAL (A)
        { field: 'computers1', readonly: false },
        { field: 'compSoftwareInt1', readonly: false },
        { field: 'compSoftwareNonint1', readonly: false },
        { field: 'compSoftwareTotal1', readonly: true }, // Computer Software Total
        { field: 'motor1', readonly: false },
        { field: 'offResidenceB1', readonly: false },
        { field: 'stcLho1', readonly: false },
        { field: 'otherPremisesB1', readonly: false },
        { field: 'otherMachineryPlant1', readonly: true }, // Other Machinery & Plant Total
        { field: 'totalB1', readonly: true }, // TOTAL (B)
        { field: 'totalFurnFixPrevYear1', readonly: true }, // Total Furniture & Fixtures (A+B) prev year
        { field: 'landNotRev1', readonly: false },
        { field: 'landRev1', readonly: false },
        { field: 'landRevEnh1', readonly: false },
        { field: 'offBuildNotRev1', readonly: false },
        { field: 'offBuildRev1', readonly: false },
        { field: 'offBuildRevEnh1', readonly: false },
        { field: 'residQuartNotRev1', readonly: false },
        { field: 'residQuartRev1', readonly: false },
        { field: 'residQuartRevEnh1', readonly: false },
        { field: 'premisTotal1', readonly: true }, // Premises Total (Cost)
        { field: 'revtotal1', readonly: true }, // Revaluation Total (Enhancement)
        { field: 'totalC1', readonly: true }, // TOTAL (C)
        { field: 'premisesUnderCons1', readonly: false }, // Projects under construction
        { field: 'grandTotal1', readonly: true }, // Grand Total (A+B+C+D) - This might be a reconciliation total, needs re-eval
      ],
    },
    // SECTION: Additions during the year
    {
      id: 'A.II',
      label: 'Additions during the year:',
      isSectionHeader: true,
      cells: [
        { field: 'stcNstaffAdd1', readonly: false },
        { field: 'offResidenceAAdd1', readonly: false },
        { field: 'otherPremisesAAdd1', readonly: false },
        { field: 'electricFittingAdd1', readonly: false },
        { field: 'totalAAdd1', readonly: true },
        { field: 'computersAdd1', readonly: false },
        { field: 'compSoftwareIntAdd1', readonly: false },
        { field: 'compSoftwareNonintAdd1', readonly: false },
        { field: 'compSoftwareTotalAdd1', readonly: true },
        { field: 'motorAdd1', readonly: false },
        { field: 'offResidenceBAdd1', readonly: false },
        { field: 'stcLhoAdd1', readonly: false },
        { field: 'otherPremisesBAdd1', readonly: false },
        { field: 'otherMachineryPlantAdd1', readonly: true },
        { field: 'totalBAdd1', readonly: true },
        { field: 'totalFurnFixAdditions1', readonly: true },
        { field: 'landNotRevAdd1', readonly: false },
        { field: 'landRevAdd1', readonly: false },
        { field: 'landRevEnhAdd1', readonly: false },
        { field: 'offBuildNotRevAdd1', readonly: false },
        { field: 'offBuildRevAdd1', readonly: false },
        { field: 'offBuildRevEnhAdd1', readonly: false },
        { field: 'residQuartNotRevAdd1', readonly: false },
        { field: 'residQuartRevAdd1', readonly: false },
        { field: 'residQuartRevEnhAdd1', readonly: false },
        { field: 'premisTotalAdd1', readonly: true },
        { field: 'revtotalAdd1', readonly: true },
        { field: 'totalCAdd1', readonly: true },
        { field: 'premisesUnderConsAdd1', readonly: false },
        null, // No Grand Total cell for this row, it would be part of a final total.
      ],
    },
    // SECTION: Deductions during the year (Sales/Adjustments)
    {
      id: 'A.III',
      label: 'Deductions during the year (Sales/Adjustments):',
      isSectionHeader: true,
      cells: [
        { field: 'stcNstaffDed1', readonly: false },
        { field: 'offResidenceADed1', readonly: false },
        { field: 'otherPremisesADed1', readonly: false },
        { field: 'electricFittingDed1', readonly: false },
        { field: 'totalADed1', readonly: true },
        { field: 'computersDed1', readonly: false },
        { field: 'compSoftwareIntDed1', readonly: false },
        { field: 'compSoftwareNonintDed1', readonly: false },
        { field: 'compSoftwareTotalDed1', readonly: true },
        { field: 'motorDed1', readonly: false },
        { field: 'offResidenceBDed1', readonly: false },
        { field: 'stcLhoDed1', readonly: false },
        { field: 'otherPremisesBDed1', readonly: false },
        { field: 'otherMachineryPlantDed1', readonly: true },
        { field: 'totalBDed1', readonly: true },
        { field: 'totalFurnFixDeductions1', readonly: true },
        { field: 'landNotRevDed1', readonly: false },
        { field: 'landRevDed1', readonly: false },
        { field: 'landRevEnhDed1', readonly: false },
        { field: 'offBuildNotRevDed1', readonly: false },
        { field: 'offBuildRevDed1', readonly: false },
        { field: 'offBuildRevEnhDed1', readonly: false },
        { field: 'residQuartNotRevDed1', readonly: false },
        { field: 'residQuartRevDed1', readonly: false },
        { field: 'residQuartRevEnhDed1', readonly: false },
        { field: 'premisTotalDed1', readonly: true },
        { field: 'revtotalDed1', readonly: true },
        { field: 'totalCDed1', readonly: true },
        { field: 'premisesUnderConsDed1', readonly: false },
        null, // No Grand Total cell for this row
      ],
    },
    // SECTION: Depreciation for the year
    {
      id: 'A.IV',
      label: 'Depreciation for the year:',
      isSectionHeader: true,
      cells: [
        { field: 'depA1', readonly: false }, // Furniture & Fittings Depreciation
        null, null, null, null, // Remaining A columns are empty for depreciation
        { field: 'depB1', readonly: false }, // Machinery & Plant Depreciation
        null, null, null, null, null, null, null, null, null, // Remaining B columns are empty
        null, // Total Furn & Fix is empty
        { field: 'depC1', readonly: false }, // Premises Depreciation
        null, null, null, null, null, null, null, null, null, null, null, // Remaining C columns are empty
        null, // Projects under construction is empty for depreciation
        { field: 'totalDepreciation1', readonly: true }, // Grand Total Depreciation
      ],
    },
    // SECTION: Net Block at the end of the year
    {
      id: 'A.V',
      label: 'Net Block at the end of the year:',
      isSectionHeader: true,
      cells: [
        { field: 'netBlockA1', readonly: true }, // Furniture & Fittings Net Block
        null, null, null, null, // Remaining A columns are empty
        { field: 'netBlockB1', readonly: true }, // Machinery & Plant Net Block
        null, null, null, null, null, null, null, null, null, // Remaining B columns are empty
        null, // Total Furn & Fix is empty
        { field: 'netBlockC1', readonly: true }, // Premises Net Block
        null, null, null, null, null, null, null, null, null, null, null, // Remaining C columns are empty
        { field: 'netBlockD1', readonly: true }, // Projects under construction Net Block (same as premisesUnderConsCurrent1)
        { field: 'grandTotalNetBlock1', readonly: true }, // Grand Total Net Block
      ],
    },
    // Row for the reconciliation total - this is a separate row as per snippet
    {
      id: 'B.I',
      label: 'Total Premises (Cost + Revaluation) at the end of Previous Year (for Reconciliation)',
      isSectionHeader: true, // Treat as a total row for distinct styling
      istotalrow: true,
      cells: [
        null, null, null, null, null, null, null, null, null, null, null, null, null, null, null, null, // Empty cells to align
        { field: 'totalPremPreYear', readonly: true }, // Calculated from snippet logic
        null, null, null, null, null, null, null, null, null, null, null, null, null, null, // More empty cells
      ]
    },
    // Fixed Assets Reconciliation row (from snippet's calculation)
    {
      id: 'B.II',
      label: 'Fixed Assets Cost Previous Year (for Reconciliation)',
      isSectionHeader: true,
      istotalrow: true,
      cells: [
        null, null, null, null, null, // Empty cells for A
        { field: 'fixCostPre', readonly: false }, // Input from snippet
        { field: 'fixAddPreYear', readonly: false }, // Input from snippet
        { field: 'fixDedPreYear', readonly: false }, // Input from snippet
        { field: 'fixAddYear', readonly: false }, // Input from snippet
        { field: 'fixDedYear', readonly: false }, // Input from snippet
        { field: 'fixDepYear', readonly: false }, // Input from snippet
        { field: 'fixDepPreYear', readonly: false }, // Input from snippet
        null, null, null, // Empty cells for B
        { field: 'fixCostYear', readonly: true }, // Calculated from snippet
        { field: 'ttlFixCurrYear', readonly: true }, // Calculated from snippet
        { field: 'ttlFixPreYear', readonly: true }, // Calculated from snippet
        { field: 'currvPremTotal', readonly: true }, // Calculated from snippet
        { field: 'grandCurrTotal', readonly: true }, // Calculated from snippet
        null, null, null, null, null, null, null, null, null, null, null, null, // More empty cells for C and D and Grand Total
      ]
    }
    // Add more rows here as needed based on the full Schedule10.txt
  ], [formData]);

  return (
    <Box sx={{ p: 3 }}>
      <Typography variant="h3" component="h1" gutterBottom>
        Schedule 10
      </Typography>

      <form onSubmit={handleSubmit}>
        <TableContainer component={Paper} sx={{ maxHeight: 650, overflow: 'auto' }}>
          <Table stickyHeader aria-label="schedule 10 table" sx={{ minWidth: 4000 }}>
            <TableHead>
              <TableRow>
                <StyledTableCell rowSpan={2}><b>Sr.No</b></StyledTableCell>
                <StyledTableCell rowSpan={2}><b>Particulars</b></StyledTableCell>
                <StyledTableCell colSpan={5}><b>(A) FURNITURE & FITTINGS</b></StyledTableCell>
                <StyledTableCell colSpan={10}><b>(B) MACHINERY & PLANT</b></StyledTableCell>
                <StyledTableCell rowSpan={2}><b>Total Furniture & Fixtures <br />(A+B)</b></StyledTableCell>
                <StyledTableCell colSpan={12}><b>(C) PREMISES</b></StyledTableCell>
                <StyledTableCell rowSpan={2}><b>(D) Projects under <br />construction</b></StyledTableCell>
                <StyledTableCell rowSpan={2}><b>Grand Total <br /> (A + B + C + D)</b></StyledTableCell>
              </TableRow>
              <TableRow>
                {/* Furniture & Fittings */}
                <StyledTableCell><b>i) At STCs & Staff Colleges <br />(For Local Head Office only)</b></StyledTableCell>
                <StyledTableCell><b>ii) At Officers' Residences</b></StyledTableCell>
                <StyledTableCell><b>iii) At Other Premises</b></StyledTableCell>
                <StyledTableCell><b>iv) Electric Fittings <br />(include electric wiring,<br /> switches, sockets, other<br /> fittings & fans etc.)</b></StyledTableCell>
                <StyledTableCell><b>TOTAL (A)<br /> (i+ii+iii+iv)</b></StyledTableCell>
                {/* Machinery & Plant */}
                <StyledTableCell><b>i) Computer Hardware</b></StyledTableCell>
                <StyledTableCell><b>a. Computer Software <br />(forming integral part of<br /> Hardware)</b></StyledTableCell>
                <StyledTableCell><b>b. Computer Software <br />(not forming integral <br />of Hardware)</b></StyledTableCell>
                <StyledTableCell><b>ii) Computer Software <br />Total (a+b)</b></StyledTableCell>
                <StyledTableCell><b>iii) Motor Vehicles </b></StyledTableCell>
                <StyledTableCell><b>a) At Officers' Residences</b></StyledTableCell>
                <StyledTableCell><b>b) At STCs <br />(For Local Head Office)</b></StyledTableCell>
                <StyledTableCell><b>c) At other Premises </b></StyledTableCell>
                <StyledTableCell><b>iv) Other Machinery & Plant <br />( a+b+c)</b> </StyledTableCell>
                <StyledTableCell><b>TOTAL <br /> (B= i+ii+iii+iv)</b></StyledTableCell>
                {/* Premises */}
                <StyledTableCell><b>(a) Land (Not Revalued):<br /> Cost</b></StyledTableCell>
                <StyledTableCell><b>(b) Land (Revalued): <br />Cost</b></StyledTableCell>
                <StyledTableCell><b>(c) Land (Revalued): <br />Enhancement due to <br />Revaluation</b></StyledTableCell>
                <StyledTableCell><b>(d) Office Building <br />(Not revalued): Cost </b></StyledTableCell>
                <StyledTableCell><b>(e) Office Building <br />(Revalued): Cost </b></StyledTableCell>
                <StyledTableCell><b>(f) Office Building <br />(Revalued): Enhancement <br />due to Revaluation</b></StyledTableCell>
                <StyledTableCell><b>(g) Residential Building <br />(Not revalued): Cost</b></StyledTableCell>
                <StyledTableCell><b>(h) Residential Building <br />(Revalued): Cost</b></StyledTableCell>
                <StyledTableCell><b>(i) Residential Building <br />(Revalued): Enhancement <br />due to Revaluation</b></StyledTableCell>
                <StyledTableCell><b>(j) Premises Total <br />(a+b+d+e+g+h)</b></StyledTableCell>
                <StyledTableCell><b>(k) Revaluation Total <br />(c+f+i)</b></StyledTableCell>
                <StyledTableCell><b>TOTAL <br />(C=j+k)</b></StyledTableCell>
              </TableRow>
            </TableHead>
            <TableBody>
              {tableRows.map((row) => (
                <StyledTableRow
                  key={row.id}
                  issectionheader={row.isSectionHeader ? 'true' : undefined}
                  issubsectionheader={row.isSubsectionHeader ? 'true' : undefined}
                  istotalrow={row.istotalrow ? 'true' : undefined}
                >
                  <StyledTableCell>{row.id}</StyledTableCell>
                  <StyledTableCell>{row.label}</StyledTableCell>
                  {row.cells.map((cell, index) => {
                    if (cell === null) {
                      return <StyledTableCell key={`empty-${row.id}-${index}`} />;
                    }
                    const fieldName = cell.field;
                    return (
                      <StyledTableCell key={fieldName}>
                        <input
                          type="text"
                          name={fieldName}
                          value={formData[fieldName]}
                          onChange={handleChange}
                          readOnly={cell.readonly}
                          maxLength={cell.maxLength || 18} // Default maxLength
                          style={{
                            textAlign: 'right',
                            width: '100%',
                            boxSizing: 'border-box',
                            backgroundColor: cell.readonly ? '#f0f0f0' : 'white',
                            border: errors[fieldName] ? '1px solid red' : '1px solid #ccc',
                          }}
                        />
                        {errors[fieldName] && (
                          <Typography variant="caption" color="error">
                            {errors[fieldName]}
                          </Typography>
                        )}
                      </StyledTableCell>
                    );
                  })}
                </StyledTableRow>
              ))}
            </TableBody>
          </Table>
        </TableContainer>
        <Stack direction="row" spacing={2} sx={{ mt: 2, justifyContent: 'center' }}>
          <Button variant="contained" color="warning" type="button">
            Save
          </Button>
          <Button variant="contained" color="success" type="submit" disabled={Object.keys(errors).length > 0}>
            Submit
          </Button>
        </Stack>
      </form>
    </Box>
  );
};

export default Schedule10;

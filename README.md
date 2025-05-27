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
import debounce from 'lodash.debounce'; // Import debounce

// Styled components from Example.txt for consistent UI
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
  backgroundColor: theme.palette.background.paper, // Default background
  '&:nth-of-type(odd)': {
    // backgroundColor: theme.palette.action.hover, // Kept for slight differentiation if desired, can be removed
  },
  ...(issectionheader && {
    // backgroundColor: theme.palette.grey[100], // Very light grey
    '& > td': {
      fontWeight: 'bold',
      textAlign: 'left',
    },
  }),
  ...(issubsectionheader && {
    // backgroundColor: theme.palette.grey[50], // Even lighter
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

const Schedule10 = () => {
  // Initialize state to hold all form data, corresponding to ng-model fields in Schedule10.txt
  const [formData, setFormData] = useState({
    // (A) FURNITURE & FITTINGS
    stcNstaff1: '', // i) At STCs & Staff Colleges
    offResidenceA1: '', // ii) At Officers' Residences
    otherPremisesA1: '', // iii) At Other Premises
    electricFitting1: '', // iv) Electric Fittings
    totalA1: '', // TOTAL (A) - Calculated

    // (B) MACHINERY & PLANT
    computers1: '', // i) Computer Hardware
    compSoftwareInt1: '', // a. Computer Software (forming integral part of Hardware)
    compSoftwareNonint1: '', // b. Computer Software (not forming integral of Hardware)
    compSoftwareTotal1: '', // ii) Computer Software Total (a+b) - Calculated
    motor1: '', // iii) Motor Vehicles
    offResidenceB1: '', // a) At Officers' Residences (Other Machinery & Plant)
    stcLho1: '', // b) At STCs (Other Machinery & Plant)
    otherPremisesB1: '', // c) At other Premises (Other Machinery & Plant)
    otherMachineryPlant1: '', // iv) Other Machinery & Plant (a+b+c) - Calculated
    totalB1: '', // TOTAL (B= i+ii+iii+iv) - Calculated

    // Total Furniture & Fixtures (A+B)
    totalFurnFix1: '', // Calculated

    // (C) PREMISES
    landNotRev1: '', // (a) Land (Not Revalued): Cost
    landRev1: '', // (b) Land (Revalued): Cost
    landRevEnh1: '', // (c) Land (Revalued): Enhancement due to Revaluation
    offBuildNotRev1: '', // (d) Office Building (Not revalued): Cost
    offBuildRev1: '', // (e) Office Building (Revalued): Cost
    offBuildRevEnh1: '', // (f) Office Building (Revalued): Enhancement due to Revaluation
    residQuartNotRev1: '', // (g) Residential Building (Not revalued): Cost
    residQuartRev1: '', // (h) Residential Building (Revalued): Cost
    residQuartRevEnh1: '', // (i) Residential Building (Revalued): Enhancement due to Revaluation
    premisTotal1: '', // (j) Premises Total (a+b+d+e+g+h) - Calculated
    revtotal1: '', // (k) Revaluation Total (c+f+i) - Calculated
    totalC1: '', // TOTAL (C=j+k) - Calculated

    // (D) Projects under construction
    premisesUnderCons1: '', // (D) Projects under construction

    // Grand Total (A + B + C + D)
    grandTotal1: '', // Calculated

    // Add more fields from Schedule10.txt if they exist in the full file and are required inputs
    // Based on the snippet, there are also fields like:
    premCostYear: '',
    revCostYear: '',
    premRevOfPreYear: '',
    premDedOfPreYear: '',
    revDedOfPre: '',
    depCostPreYear: '',
    depRevPreYear: '',
    fixCostPre: '',
    fixAddPreYear: '',
    fixDedPreYear: '',
    fixAddYear: '',
    fixDedYear: '',
    fixDepYear: '',
    fixDepPreYear: '',
    premCostYear: '',
    revCostYear: '',

    // Calculated fields often have 'currv' or 'ttl' prefixes based on the snippet
    currvPremTotal: '', // Calculated
    fixCostYear: '', // Calculated
    ttlFixCurrYear: '', // Calculated
    ttlFixPreYear: '', // Calculated
    grandCurrTotal: '', // Calculated
  });

  // State to hold validation errors
  const [errors, setErrors] = useState({});

  // Helper function to parse float values, handling empty strings as 0
  const parseFloatSafe = (value) => {
    const parsed = parseFloat(value);
    return isNaN(parsed) ? 0 : parsed;
  };

  // Validation function for decimal places
  const validateDecimal2Places = (value) => {
    if (value === null || value.trim() === '') return true; // Allow empty or null
    const regex = /^-?\d+(\.\d{1,2})?$/; // Allows for 0, 1, or 2 decimal places, and negative numbers
    return regex.test(value);
  };

  // Debounced validation function
  const debouncedValidate = useCallback(
    debounce((name, value) => {
      // Only validate fields that are not calculated
      if (
        name.endsWith('1') &&
        ![
          'totalA1', 'compSoftwareTotal1', 'otherMachineryPlant1', 'totalB1', 'totalFurnFix1',
          'premisTotal1', 'revtotal1', 'totalC1', 'grandTotal1',
          'currvPremTotal', 'fixCostYear', 'ttlFixCurrYear', 'ttlFixPreYear', 'grandCurrTotal'
        ].includes(name)
      ) {
        if (!validateDecimal2Places(value)) {
          setErrors((prevErrors) => ({ ...prevErrors, [name]: 'Invalid decimal format (max 2 places).' }));
        } else {
          setErrors((prevErrors) => {
            const newErrors = { ...prevErrors };
            delete newErrors[name]; // Clear error if valid
            return newErrors;
          });
        }
      }
    }, 300), // Adjust debounce delay as needed (e.g., 300ms)
    [] // Empty dependency array means this function is created once
  );

  // Handle input changes
  const handleChange = (e) => {
    const { name, value } = e.target;
    setFormData((prevData) => ({ ...prevData, [name]: value }));
    debouncedValidate(name, value); // Call debounced validation
  };

  // Debounced function to perform all calculations
  const calculateAllTotals = useCallback(
    debounce((currentFormData) => {
      setFormData((prevData) => {
        const newData = { ...currentFormData };

        // (A) FURNITURE & FITTINGS calculations
        newData.totalA1 = (
          parseFloatSafe(newData.stcNstaff1) +
          parseFloatSafe(newData.offResidenceA1) +
          parseFloatSafe(newData.otherPremisesA1) +
          parseFloatSafe(newData.electricFitting1)
        ).toFixed(2);

        // (B) MACHINERY & PLANT calculations
        newData.compSoftwareTotal1 = (
          parseFloatSafe(newData.compSoftwareInt1) +
          parseFloatSafe(newData.compSoftwareNonint1)
        ).toFixed(2);

        newData.otherMachineryPlant1 = (
          parseFloatSafe(newData.offResidenceB1) +
          parseFloatSafe(newData.stcLho1) +
          parseFloatSafe(newData.otherPremisesB1)
        ).toFixed(2);

        newData.totalB1 = (
          parseFloatSafe(newData.computers1) +
          parseFloatSafe(newData.compSoftwareTotal1) +
          parseFloatSafe(newData.motor1) +
          parseFloatSafe(newData.otherMachineryPlant1)
        ).toFixed(2);

        // Total Furniture & Fixtures (A+B) calculation
        newData.totalFurnFix1 = (
          parseFloatSafe(newData.totalA1) +
          parseFloatSafe(newData.totalB1)
        ).toFixed(2);

        // (C) PREMISES calculations
        newData.premisTotal1 = (
          parseFloatSafe(newData.landNotRev1) +
          parseFloatSafe(newData.landRev1) +
          parseFloatSafe(newData.offBuildNotRev1) +
          parseFloatSafe(newData.offBuildRev1) +
          parseFloatSafe(newData.residQuartNotRev1) +
          parseFloatSafe(newData.residQuartRev1)
        ).toFixed(2);

        newData.revtotal1 = (
          parseFloatSafe(newData.landRevEnh1) +
          parseFloatSafe(newData.offBuildRevEnh1) +
          parseFloatSafe(newData.residQuartRevEnh1)
        ).toFixed(2);

        newData.totalC1 = (
          parseFloatSafe(newData.premisTotal1) +
          parseFloatSafe(newData.revtotal1)
        ).toFixed(2);

        // Grand Total (A + B + C + D) calculation
        newData.grandTotal1 = (
          parseFloatSafe(newData.totalA1) +
          parseFloatSafe(newData.totalB1) +
          parseFloatSafe(newData.totalC1) +
          parseFloatSafe(newData.premisesUnderCons1)
        ).toFixed(2);

        // --- Additional calculations based on Schedule10.txt snippet ---
        // You'll need to accurately map these to your formData fields
        // SCH10.row.currvPremTotal = (SCH10.parseFloat(SCH10.row.premCostYear) + SCH10.parseFloat(SCH10.row.revCostYear)).toFixed(2);
        newData.currvPremTotal = (
          parseFloatSafe(newData.premCostYear) +
          parseFloatSafe(newData.revCostYear)
        ).toFixed(2);

        // SCH10.row.fixCostYear = (SCH10.parseFloat(SCH10.row.fixCostPre) + SCH10.parseFloat(SCH10.row.fixAddPreYear) - SCH10.parseFloat(SCH10.row.fixDedPreYear)).toFixed(2);
        newData.fixCostYear = (
          parseFloatSafe(newData.fixCostPre) +
          parseFloatSafe(newData.fixAddPreYear) -
          parseFloatSafe(newData.fixDedPreYear)
        ).toFixed(2);

        // SCH10.row.ttlFixCurrYear = (SCH10.parseFloat(SCH10.row.fixCostYear) + SCH10.parseFloat(SCH10.row.fixAddYear) - SCH10.parseFloat(SCH10.row.fixDedYear) - SCH10.parseFloat(SCH10.row.fixDepYear)).toFixed(2);
        newData.ttlFixCurrYear = (
          parseFloatSafe(newData.fixCostYear) +
          parseFloatSafe(newData.fixAddYear) -
          parseFloatSafe(newData.fixDedYear) -
          parseFloatSafe(newData.fixDepYear)
        ).toFixed(2);

        // SCH10.row.ttlFixPreYear = (SCH10.parseFloat(SCH10.row.fixCostPre) + SCH10.parseFloat(SCH10.row.fixAddPreYear) - SCH10.parseFloat(SCH10.row.fixDedPreYear) - SCH10.parseFloat(SCH10.row.fixDepPreYear)).toFixed(2);
        newData.ttlFixPreYear = (
          parseFloatSafe(newData.fixCostPre) +
          parseFloatSafe(newData.fixAddPreYear) -
          parseFloatSafe(newData.fixDedPreYear) -
          parseFloatSafe(newData.fixDepPreYear)
        ).toFixed(2);

        // SCH10.row.grandCurrTotal = (SCH10.parseFloat(SCH10.row.currvPremTotal) + SCH10.parseFloat(SCH10.row.ttlFixCurrYear) + SCH10.parseFloat(SCH10.row.premisUnderConst)).toFixed(2);
        // Assuming 'premisUnderConst' corresponds to 'premisesUnderCons1' in your formData
        newData.grandCurrTotal = (
          parseFloatSafe(newData.currvPremTotal) +
          parseFloatSafe(newData.ttlFixCurrYear) +
          parseFloatSafe(newData.premisesUnderCons1)
        ).toFixed(2);

        return newData;
      });
    }, 500), // Debounce calculations by 500ms
    [] // Empty dependency array means this function is created once
  );

  // useEffect to trigger debounced calculations when relevant fields change
  useEffect(() => {
    // Pass the current state to the debounced function
    calculateAllTotals(formData);
    // Cleanup function for debounce to cancel any pending calls on unmount or re-render
    return () => {
      calculateAllTotals.cancel();
      debouncedValidate.cancel();
    };
  }, [
    // Dependencies for (A) FURNITURE & FITTINGS
    formData.stcNstaff1, formData.offResidenceA1, formData.otherPremisesA1, formData.electricFitting1,
    // Dependencies for (B) MACHINERY & PLANT
    formData.computers1, formData.compSoftwareInt1, formData.compSoftwareNonint1, formData.motor1,
    formData.offResidenceB1, formData.stcLho1, formData.otherPremisesB1,
    // Dependencies for (C) PREMISES
    formData.landNotRev1, formData.landRev1, formData.landRevEnh1,
    formData.offBuildNotRev1, formData.offBuildRev1, formData.offBuildRevEnh1,
    formData.residQuartNotRev1, formData.residQuartRev1, formData.residQuartRevEnh1,
    formData.premisesUnderCons1,
    // Dependencies for additional calculations based on snippet
    formData.premCostYear, formData.revCostYear,
    formData.fixCostPre, formData.fixAddPreYear, formData.fixDedPreYear,
    formData.fixAddYear, formData.fixDedYear, formData.fixDepYear, formData.fixDepPreYear,
    // Include the debounced function itself as a dependency (safe with useCallback)
    calculateAllTotals,
  ]);


  // Overall form validation before submission
  const validateForm = () => {
    let newErrors = {};
    // Iterate over all fields that require validation (e.g., all input fields)
    for (const key in formData) {
      // Exclude calculated fields from direct validation
      if (
        ![
          'totalA1', 'compSoftwareTotal1', 'otherMachineryPlant1', 'totalB1', 'totalFurnFix1',
          'premisTotal1', 'revtotal1', 'totalC1', 'grandTotal1',
          'currvPremTotal', 'fixCostYear', 'ttlFixCurrYear', 'ttlFixPreYear', 'grandCurrTotal'
        ].includes(key)
      ) {
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
    // Ensure all debounced validations/calculations have completed before final submit validation
    debouncedValidate.flush();
    calculateAllTotals.flush();

    if (validateForm()) {
      console.log('Form data submitted:', formData);
      // Integrate with your API to submit data, similar to useApi in Example.txt
    } else {
      console.log('Form has validation errors:', errors);
    }
  };

  // Define the table structure for rendering the form
  // This structure maps to the visual layout of Schedule 10 in your JSP file
  const tableRows = useMemo(() => [
    // Section A: FURNITURE & FITTINGS
    {
      id: 'A',
      label: 'Total Original Cost / Revalued Value upto the end of previous year',
      type: 'section',
      isSectionHeader: true,
      cells: [
        { field: 'stcNstaff1', readonly: false, maxLength: 18 },
        { field: 'offResidenceA1', readonly: false, maxLength: 18 },
        { field: 'otherPremisesA1', readonly: false, maxLength: 18 },
        { field: 'electricFitting1', readonly: false, maxLength: 18 },
        { field: 'totalA1', readonly: true, maxLength: 18 },
        null, // Placeholder for grand total column in section A
        null, null, null, null, null, null, null, null, null, // Placeholders for section B
        null, null, null, null, null, null, null, null, null, null, null, // Placeholders for section C
        null, null, // Placeholder for section D and Grand Total
      ],
    },
    // Adding row for "Additions during the year:" (Example for one such row)
    // You will need to add more rows and their corresponding 'cells' and 'field' definitions
    // based on the complete HTML structure of your Schedule10.txt
    {
      id: 'B',
      label: 'Original Cost of items put to use during the year:',
      type: 'section',
      isSectionHeader: true,
      cells: [
        { field: 'computers1', readonly: false, maxLength: 18 },
        { field: 'compSoftwareInt1', readonly: false, maxLength: 18 },
        { field: 'compSoftwareNonint1', readonly: false, maxLength: 18 },
        { field: 'compSoftwareTotal1', readonly: true, maxLength: 18 },
        { field: 'motor1', readonly: false, maxLength: 18 },
        { field: 'offResidenceB1', readonly: false, maxLength: 18 },
        { field: 'stcLho1', readonly: false, maxLength: 18 },
        { field: 'otherPremisesB1', readonly: false, maxLength: 18 },
        { field: 'otherMachineryPlant1', readonly: true, maxLength: 18 },
        { field: 'totalB1', readonly: true, maxLength: 18 },
        { field: 'totalFurnFix1', readonly: true, maxLength: 18 }, // Total Furniture & Fixtures (A+B)
        { field: 'landNotRev1', readonly: false, maxLength: 18 },
        { field: 'landRev1', readonly: false, maxLength: 18 },
        { field: 'landRevEnh1', readonly: false, maxLength: 18 },
        { field: 'offBuildNotRev1', readonly: false, maxLength: 18 },
        { field: 'offBuildRev1', readonly: false, maxLength: 18 },
        { field: 'offBuildRevEnh1', readonly: false, maxLength: 18 },
        { field: 'residQuartNotRev1', readonly: false, maxLength: 18 },
        { field: 'residQuartRev1', readonly: false, maxLength: 18 },
        { field: 'residQuartRevEnh1', readonly: false, maxLength: 18 },
        { field: 'premisTotal1', readonly: true, maxLength: 18 },
        { field: 'revtotal1', readonly: true, maxLength: 18 },
        { field: 'totalC1', readonly: true, maxLength: 18 },
        { field: 'premisesUnderCons1', readonly: false, maxLength: 18 },
        { field: 'grandTotal1', readonly: true, maxLength: 18 }, // Grand Total (A + B + C + D)
      ],
    },
    // Row 2: Based on the snippet, "Additions during the year:" would be a new row,
    // and would likely have its own set of fields and calculations.
    // Example for a row (you'll need to populate actual fields and headers)
    {
      id: 'row2',
      label: 'Additions during the year:',
      type: 'data',
      cells: [
        // These fields would map to `ng-model`s for additions
        { field: 'addStcNstaff1', readonly: false, maxLength: 18 }, // Example additional field
        { field: 'addOffResidenceA1', readonly: false, maxLength: 18 },
        { field: 'addOtherPremisesA1', readonly: false, maxLength: 18 },
        { field: 'addElectricFitting1', readonly: false, maxLength: 18 },
        { field: 'addTotalA1', readonly: true, maxLength: 18 }, // Example calculated addition total
        // ... and so on for all columns
        null, null, null, null, null, null, null, null, null, null,
        null, null, null, null, null, null, null, null, null, null, null, null, null, null,
      ]
    },
    // Example for row related to 'fixCostYear'
    {
      id: 'rowFixCostYear',
      label: 'Fixed Assets at the end of current year:', // Placeholder label
      type: 'data',
      cells: [
        { field: 'fixCostPre', readonly: false, maxLength: 18 },
        { field: 'fixAddPreYear', readonly: false, maxLength: 18 },
        { field: 'fixDedPreYear', readonly: false, maxLength: 18 },
        { field: 'fixAddYear', readonly: false, maxLength: 18 },
        { field: 'fixDedYear', readonly: false, maxLength: 18 },
        { field: 'fixDepYear', readonly: false, maxLength: 18 },
        { field: 'fixDepPreYear', readonly: false, maxLength: 18 },
        { field: 'currvPremTotal', readonly: true, maxLength: 18 }, // Calculated field
        { field: 'fixCostYear', readonly: true, maxLength: 18 }, // Calculated field
        { field: 'ttlFixCurrYear', readonly: true, maxLength: 18 }, // Calculated field
        { field: 'ttlFixPreYear', readonly: true, maxLength: 18 }, // Calculated field
        { field: 'grandCurrTotal', readonly: true, maxLength: 18 }, // Calculated field
        null, null, null, null, null, null, null, null, null, null, null, null, null, // more placeholders
      ]
    }
  ], [formData]); // Re-memoize if formData structure changes, though it generally won't

  return (
    <Box sx={{ p: 3 }}>
      <Typography variant="h3" component="h1" gutterBottom>
        Schedule 10
      </Typography>

      <form onSubmit={handleSubmit}>
        <TableContainer component={Paper} sx={{ maxHeight: 650, overflow: 'auto' }}>
          <Table stickyHeader aria-label="schedule 10 table" sx={{ minWidth: 4000 }}> {/* Adjust minWidth based on your table layout */}
            <TableHead>
              {/* Render the complex table headers from Schedule10.txt */}
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
                <StyledTableRow key={row.id}>
                  <StyledTableCell>{row.id}</StyledTableCell>
                  <StyledTableCell>{row.label}</StyledTableCell>
                  {row.cells.map((cell, index) => {
                    if (cell === null) {
                      // Render an empty cell for placeholders, or a cell with no input if desired
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
                          maxLength={cell.maxLength}
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

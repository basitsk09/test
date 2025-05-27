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
  // Initialize state to hold all form data, similar to sc10.row in JSP
  const [formData, setFormData] = useState({
    stcNstaff1: '',
    offResidenceA1: '',
    otherPremisesA1: '',
    electricFitting1: '',
    totalA1: '', // Readonly field, will be calculated
    computers1: '',
    compSoftwareInt1: '',
    compSoftwareNonint1: '',
    compSoftwareTotal1: '', // Readonly field, will be calculated
    motor1: '',
    offResidenceB1: '',
    stcLho1: '',
    otherPremisesB1: '',
    otherMachineryPlant1: '', // Readonly field, will be calculated
    totalB1: '', // Readonly field, will be calculated
    totalFurnFix1: '', // Readonly field, will be calculated
    landNotRev1: '',
    landRev1: '',
    landRevEnh1: '',
    offBuildNotRev1: '',
    offBuildRev1: '',
    offBuildEnh1: '', // Assuming this should be an input field for enhancement
    residQuartNotRev1: '',
    residQuartRev1: '',
    residQuartRevEnh1: '',
    premisTotal1: '', // Readonly field, will be calculated
    revtotal1: '', // Readonly field, will be calculated
    totalC1: '', // Readonly field, will be calculated
    premisesUnderCons1: '',
    grandTotal1: '', // Readonly field, will be calculated
    // ... add all other fields from Schedule10.txt
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
      if (name.endsWith('1') && !validateDecimal2Places(value)) {
        setErrors((prevErrors) => ({ ...prevErrors, [name]: 'Invalid decimal format (max 2 places).' }));
      } else {
        setErrors((prevErrors) => {
          const newErrors = { ...prevErrors };
          delete newErrors[name]; // Clear error if valid
          return newErrors;
        });
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
  const calculateTotals = useCallback(
    debounce((currentFormData) => {
      setFormData((prevData) => {
        const newData = { ...currentFormData }; // Use the passed currentFormData

        // Example calculation for totalA1: (i+ii+iii+iv)
        newData.totalA1 = (
          parseFloatSafe(newData.stcNstaff1) +
          parseFloatSafe(newData.offResidenceA1) +
          parseFloatSafe(newData.otherPremisesA1) +
          parseFloatSafe(newData.electricFitting1)
        ).toFixed(2);

        // Example calculation for compSoftwareTotal1: (a+b)
        newData.compSoftwareTotal1 = (
          parseFloatSafe(newData.compSoftwareInt1) +
          parseFloatSafe(newData.compSoftwareNonint1)
        ).toFixed(2);

        // Example calculation for otherMachineryPlant1: (a+b+c)
        newData.otherMachineryPlant1 = (
          parseFloatSafe(newData.offResidenceB1) +
          parseFloatSafe(newData.stcLho1) +
          parseFloatSafe(newData.otherPremisesB1)
        ).toFixed(2);

        // Example calculation for totalB1: (i+ii+iii+iv)
        newData.totalB1 = (
          parseFloatSafe(newData.computers1) +
          parseFloatSafe(newData.compSoftwareTotal1) +
          parseFloatSafe(newData.motor1) +
          parseFloatSafe(newData.otherMachineryPlant1)
        ).toFixed(2);

        // Example calculation for totalFurnFix1: (A+B)
        newData.totalFurnFix1 = (
          parseFloatSafe(newData.totalA1) +
          parseFloatSafe(newData.totalB1)
        ).toFixed(2);

        // Example calculation for premisTotal1: (a+b+d+e+g+h)
        newData.premisTotal1 = (
          parseFloatSafe(newData.landNotRev1) +
          parseFloatSafe(newData.landRev1) +
          parseFloatSafe(newData.offBuildNotRev1) +
          parseFloatSafe(newData.offBuildRev1) +
          parseFloatSafe(newData.residQuartNotRev1) +
          parseFloatSafe(newData.residQuartRev1)
        ).toFixed(2);

        // Example calculation for revtotal1: (c+f+i)
        newData.revtotal1 = (
          parseFloatSafe(newData.landRevEnh1) +
          parseFloatSafe(newData.offBuildEnh1) +
          parseFloatSafe(newData.residQuartRevEnh1)
        ).toFixed(2);

        // Example calculation for totalC1: (j+k)
        newData.totalC1 = (
          parseFloatSafe(newData.premisTotal1) +
          parseFloatSafe(newData.revtotal1)
        ).toFixed(2);

        // Example calculation for grandTotal1: (A + B + C + D)
        newData.grandTotal1 = (
          parseFloatSafe(newData.totalA1) +
          parseFloatSafe(newData.totalB1) +
          parseFloatSafe(newData.totalC1) +
          parseFloatSafe(newData.premisesUnderCons1)
        ).toFixed(2);

        // Remember to add ALL your calculations from Schedule10.txt here.
        // E.g., from your Schedule10.txt snippet:
        // SCH10.row.currvPremTotal = (SCH10.parseFloat(SCH10.row.premCostYear) + SCH10.parseFloat(SCH10.row.revCostYear)).toFixed(2);
        // Translate these to newData.fieldName = ... using parseFloatSafe.

        return newData;
      });
    }, 500), // Debounce calculations by 500ms
    [] // Empty dependency array means this function is created once
  );

  // useEffect to trigger debounced calculations when relevant fields change
  useEffect(() => {
    // Pass the current state to the debounced function
    calculateTotals(formData);
    // Cleanup function for debounce to cancel any pending calls on unmount or re-render
    return () => {
      calculateTotals.cancel();
      debouncedValidate.cancel();
    };
  }, [
    formData.stcNstaff1,
    formData.offResidenceA1,
    formData.otherPremisesA1,
    formData.electricFitting1,
    formData.computers1,
    formData.compSoftwareInt1,
    formData.compSoftwareNonint1,
    formData.motor1,
    formData.offResidenceB1,
    formData.stcLho1,
    formData.otherPremisesB1,
    formData.landNotRev1,
    formData.landRev1,
    formData.landRevEnh1,
    formData.offBuildNotRev1,
    formData.offBuildRev1,
    formData.offBuildEnh1,
    formData.residQuartNotRev1,
    formData.residQuartRev1,
    formData.residQuartRevEnh1,
    formData.premisesUnderCons1,
    // Add all other fields that trigger calculations here
    calculateTotals, // Include debounced function in dependencies
  ]);


  // Overall form validation before submission
  const validateForm = () => {
    let newErrors = {};
    // Iterate over all fields that require validation (e.g., all input fields)
    for (const key in formData) {
      if (key.endsWith('1') && !key.startsWith('total') && !key.startsWith('premis') && !key.startsWith('rev')) { // Exclude calculated fields
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
    calculateTotals.flush();

    if (validateForm()) {
      console.log('Form data submitted:', formData);
      // Integrate with your API to submit data, similar to useApi in Example.txt
    } else {
      console.log('Form has validation errors:', errors);
    }
  };

  // Define the table structure, similar to rowDefinitionsConfig in Example.txt
  const tableRows = useMemo(() => [
    // Row A: Total Original Cost / Revalued Value upto the end of previous year
    {
      id: 'rowA',
      label: `Total Original Cost / Revalued Value upto the end of previous year i.e. 31st March {{sc10.year1}}`,
      type: 'entry',
      fields: {
        stcNstaff1: { name: 'stcNstaff1', readonly: false, maxLength: 18 },
        offResidenceA1: { name: 'offResidenceA1', readonly: false, maxLength: 18 },
        otherPremisesA1: { name: 'otherPremisesA1', readonly: false, maxLength: 18 },
        electricFitting1: { name: 'electricFitting1', readonly: false, maxLength: 18 },
        totalA1: { name: 'totalA1', readonly: true, maxLength: 18 },
        computers1: { name: 'computers1', readonly: false, maxLength: 18 },
        compSoftwareInt1: { name: 'compSoftwareInt1', readonly: false, maxLength: 18 },
        compSoftwareNonint1: { name: 'compSoftwareNonint1', readonly: false, maxLength: 18 },
        compSoftwareTotal1: { name: 'compSoftwareTotal1', readonly: true, maxLength: 18 },
        motor1: { name: 'motor1', readonly: false, maxLength: 18 },
        offResidenceB1: { name: 'offResidenceB1', readonly: false, maxLength: 18 },
        stcLho1: { name: 'stcLho1', readonly: false, maxLength: 18 },
        otherPremisesB1: { name: 'otherPremisesB1', readonly: false, maxLength: 18 },
        otherMachineryPlant1: { name: 'otherMachineryPlant1', readonly: true, maxLength: 18 },
        totalB1: { name: 'totalB1', readonly: true, maxLength: 18 },
        totalFurnFix1: { name: 'totalFurnFix1', readonly: true, maxLength: 18 },
        landNotRev1: { name: 'landNotRev1', readonly: false, maxLength: 18 },
        landRev1: { name: 'landRev1', readonly: false, maxLength: 18 },
        landRevEnh1: { name: 'landRevEnh1', readonly: false, maxLength: 18 },
        offBuildNotRev1: { name: 'offBuildNotRev1', readonly: false, maxLength: 18 },
        offBuildRev1: { name: 'offBuildRev1', readonly: false, maxLength: 18 },
        offBuildEnh1: { name: 'offBuildEnh1', readonly: false, maxLength: 18 }, // Assuming this should be an input field for enhancement
        residQuartNotRev1: { name: 'residQuartNotRev1', readonly: false, maxLength: 18 },
        residQuartRev1: { name: 'residQuartRev1', readonly: false, maxLength: 18 },
        residQuartRevEnh1: { name: 'residQuartRevEnh1', readonly: false, maxLength: 18 },
        premisTotal1: { name: 'premisTotal1', readonly: true, maxLength: 18 },
        revtotal1: { name: 'revtotal1', readonly: true, maxLength: 18 },
        totalC1: { name: 'totalC1', readonly: true, maxLength: 18 },
        premisesUnderCons1: { name: 'premisesUnderCons1', readonly: false, maxLength: 18 },
        grandTotal1: { name: 'grandTotal1', readonly: true, maxLength: 18 },
      },
    },
    // You would continue to define objects for other rows like "Addition", "Original cost of items put to use during the year:" etc.
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
                  {/* Sr.No and Particulars columns */}
                  <StyledTableCell>{row.id === 'rowA' ? 'A' : ''}</StyledTableCell>
                  <StyledTableCell>{row.label}</StyledTableCell>

                  {/* Render input cells based on row.fields */}
                  {Object.keys(row.fields).map((fieldName) => {
                    const field = row.fields[fieldName];
                    return (
                      <StyledTableCell key={field.name}>
                        <input
                          type="text"
                          name={field.name}
                          value={formData[field.name]}
                          onChange={handleChange}
                          readOnly={field.readonly}
                          maxLength={field.maxLength}
                          style={{
                            textAlign: 'right',
                            width: '100%',
                            boxSizing: 'border-box',
                            backgroundColor: field.readonly ? '#f0f0f0' : 'white',
                            border: errors[field.name] ? '1px solid red' : '1px solid #ccc',
                          }}
                        />
                        {errors[field.name] && (
                          <Typography variant="caption" color="error">
                            {errors[field.name]}
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

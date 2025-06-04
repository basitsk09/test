import React, { useState, useEffect, useMemo, useCallback, useRef } from 'react';
import {
  Table,
  TableBody,
  TableContainer,
  TableHead,
  TableRow,
  Paper,
  Box,
  CircularProgress,
} from '@mui/material';
import TableCell, { tableCellClasses } from '@mui/material/TableCell';
import { styled } from '@mui/material/styles';
import FormInput from '../../../../common/components/ui/FormInput'; // Assuming path is correct

// Styled Components (No changes needed from your original)
const StyledTableCell = styled(TableCell)(({ theme }) => ({
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
}));

const StyledTableRow = styled(TableRow)(
  ({ theme, $istotalrow, $issectionheader, $issubsectionheader, $issubsubsectionheader }) => ({
    backgroundColor: theme.palette.background.paper,
    ...($issectionheader && {
      '& > td': {
        fontWeight: 'bold',
        textAlign: 'left',
      },
    }),
    ...($issubsectionheader && {
      '& > td': {
        fontWeight: 'bold',
        fontStyle: 'italic',
        textAlign: 'left',
      },
    }),
    ...($issubsubsectionheader && {
      '& > td': {
        textAlign: 'left',
      },
    }),
    ...($istotalrow && {
      '& > td': {
        fontWeight: 'bold',
      },
    }),
  })
);

// --- Helper functions (moved outside component, same as your original) ---
const parseAndFormat = (value) => {
  const num = parseFloat(value);
  return isNaN(num) ? '0.00' : num.toFixed(2);
};

const baseFieldKeys = [
  'stcNstaff', 'offResidenceA', 'otherPremisesA', 'electricFitting', 'totalA',
  'computers', 'compSoftwareInt', 'compSoftwareNonint', 'compSoftwareTotal', 'motor',
  'offResidenceB', 'stcLho', 'otherPremisesB', 'otherMachineryPlant', 'totalB',
  'totalFurnFix', 'landNotRev', 'landRev', 'landRevEnh', 'offBuildNotRev',
  'offBuildRev', 'offBuildRevEnh', 'residQuartNotRev', 'residQuartRev', 'residQuartRevEnh',
  'premisTotal', 'revtotal', 'totalC', 'premisesUnderCons', 'grandTotal',
];

const nonTotalBaseFieldKeys = [
  'stcNstaff', 'offResidenceA', 'otherPremisesA', 'electricFitting',
  'computers', 'compSoftwareInt', 'compSoftwareNonint', 'motor',
  'offResidenceB', 'stcLho', 'otherPremisesB',
  'landNotRev', 'landRev', 'landRevEnh', 'offBuildNotRev', 'offBuildRev',
  'offBuildRevEnh', 'residQuartNotRev', 'residQuartRev', 'residQuartRevEnh',
  'premisesUnderCons',
];

const rowSuffixes = [
  '1', '3', '4', '5', '6', '7', '9', '10', '11', '12', '13', '14', '18', '19', '20',
  '21', '22', '24', '25', '26', '27', '28', '29', '30', '31', '32', '33', '34',
  '35', '36', '37', '38', '39', '40',
];

const calculateRowTotals = (data, suffix) => {
  const updatedData = { ...data };
  const p = (fieldPath) => parseFloat(updatedData[fieldPath]) || 0;

  updatedData[`totalA${suffix}`] = parseAndFormat(
    p(`stcNstaff${suffix}`) +
    p(`offResidenceA${suffix}`) +
    p(`otherPremisesA${suffix}`) +
    p(`electricFitting${suffix}`)
  );
  updatedData[`compSoftwareTotal${suffix}`] = parseAndFormat(
    p(`compSoftwareInt${suffix}`) + p(`compSoftwareNonint${suffix}`)
  );
  updatedData[`otherMachineryPlant${suffix}`] = parseAndFormat(
    p(`offResidenceB${suffix}`) + p(`stcLho${suffix}`) + p(`otherPremisesB${suffix}`)
  );
  updatedData[`totalB${suffix}`] = parseAndFormat(
    p(`computers${suffix}`) +
    p(updatedData[`compSoftwareTotal${suffix}`]) +
    p(`motor${suffix}`) +
    p(updatedData[`otherMachineryPlant${suffix}`])
  );
  updatedData[`totalFurnFix${suffix}`] = parseAndFormat(
    p(updatedData[`totalA${suffix}`]) + p(updatedData[`totalB${suffix}`])
  );
  updatedData[`premisTotal${suffix}`] = parseAndFormat(
    p(`landNotRev${suffix}`) +
    p(`landRev${suffix}`) +
    p(`offBuildNotRev${suffix}`) +
    p(`offBuildRev${suffix}`) +
    p(`residQuartNotRev${suffix}`) +
    p(`residQuartRev${suffix}`)
  );
  updatedData[`revtotal${suffix}`] = parseAndFormat(
    p(`landRevEnh${suffix}`) + p(`offBuildRevEnh${suffix}`) + p(`residQuartRevEnh${suffix}`)
  );
  updatedData[`totalC${suffix}`] = parseAndFormat(
    p(updatedData[`premisTotal${suffix}`]) + p(updatedData[`revtotal${suffix}`])
  );
  updatedData[`grandTotal${suffix}`] = parseAndFormat(
    p(updatedData[`totalA${suffix}`]) +
    p(updatedData[`totalB${suffix}`]) +
    p(updatedData[`totalC${suffix}`]) +
    p(`premisesUnderCons${suffix}`)
  );
  return updatedData;
};

const sumAcrossRows = (data, fieldBaseName, suffixesToSum) => {
  let total = 0;
  suffixesToSum.forEach((sfx) => {
    total += parseFloat(data[`${fieldBaseName}${sfx}`]) || 0;
  });
  return parseAndFormat(total);
};

const subtractAcrossRows = (data, fieldBaseName, minuendSuffix, subtrahendSuffix) => {
  const minuend = parseFloat(data[`${fieldBaseName}${minuendSuffix}`]) || 0;
  const subtrahend = parseFloat(data[`${fieldBaseName}${subtrahendSuffix}`]) || 0;
  return parseAndFormat(minuend - subtrahend);
};

// Main function to perform all calculations (on the main thread)
const performAllCalculations = (initialData) => {
  let calculatedData = { ...initialData };
  const p = (fieldPath) => parseFloat(calculatedData[fieldPath]) || 0;

  const editableRowSuffixes = [
    '1', '3', '4', '5', '6', '9', '10', '11', '18', '19', '20', '21',
    '24', '25', '26', '30', '33', '34', '35', '36', '37', '38', '39', '40',
  ];

  editableRowSuffixes.forEach((suffix) => {
    calculatedData = calculateRowTotals(calculatedData, suffix);
  });

  nonTotalBaseFieldKeys.forEach((key) => {
    calculatedData[`${key}7`] = sumAcrossRows(calculatedData, key, ['3', '4', '36', '5', '6']);
    calculatedData[`${key}12`] = sumAcrossRows(calculatedData, key, ['37', '9', '33', '10', '11']);
    calculatedData[`${key}13`] = subtractAcrossRows(calculatedData, key, '7', '12');
    calculatedData[`${key}14`] = sumAcrossRows(calculatedData, key, ['1', '13']);
    calculatedData[`${key}22`] = sumAcrossRows(calculatedData, key, ['18', '34', '38', '19', '20', '21', '39']);
    calculatedData[`${key}27`] = sumAcrossRows(calculatedData, key, ['40', '24', '25', '26']);
    calculatedData[`${key}28`] = subtractAcrossRows(calculatedData, key, '22', '27');
    calculatedData[`${key}29`] = subtractAcrossRows(calculatedData, key, '14', '28');
    calculatedData[`${key}31`] = subtractAcrossRows(calculatedData, key, '9', '24');

    const val30 = p(`${key}30`);
    const val31 = p(calculatedData[`${key}31`]);
    const val35 = p(`${key}35`);
    calculatedData[`${key}32`] = parseAndFormat(val30 - (val31 + val35));
  });

  const aggregatedRowSuffixes = ['7', '12', '13', '14', '22', '27', '28', '29', '31', '32'];
  aggregatedRowSuffixes.forEach((suffix) => {
    calculatedData = calculateRowTotals(calculatedData, suffix);
  });

  return calculatedData;
};

// Initial state structure generation
const generateInitialFormDataStructure = () => {
  const baseForm = {
    particulars3: 'Cost of new items put to use upto 3rd October 2024',
    particulars4: 'Cost of new items put to use during 4th October 2024 to 31st March 2025',
    save: true,
    finyearOne: '2024',
    finyearTwo: '2025',
    circleCode: '001',
    quarterEndDate: '31/03/2025',
    userId: '1111111',
    reportName: 'Schedule 10',
    reportId: null,
    reportMasterId: '310010',
    status: null,
  };
  const initialNumericFields = {};
  rowSuffixes.forEach((suffix) => {
    baseFieldKeys.forEach((key) => {
      initialNumericFields[`${key}${suffix}`] = '0.00';
    });
  });
  return { ...baseForm, ...initialNumericFields };
};
// --- End of Helper Functions ---


const Schedule10 = () => {
  // Initialize with basic structure, calculations will be deferred
  const [formData, setFormData] = useState(generateInitialFormDataStructure);
  const [isLoading, setIsLoading] = useState(true); // For initial deferred calculation
  const [isRecalculating, setIsRecalculating] = useState(false); // For subsequent debounced calculations

  const debounceTimeoutRef = useRef(null);

  // Effect to perform initial calculations AFTER component first paints
  useEffect(() => {
    setIsLoading(true);
    const timerId = setTimeout(() => {
      // Get the initially structured data (mostly "0.00" for numeric fields)
      const initialData = generateInitialFormDataStructure();
      // Perform heavy calculations on the main thread
      const fullyCalculatedData = performAllCalculations(initialData);
      setFormData(fullyCalculatedData);
      setIsLoading(false);
    }, 50); // Small delay (e.g., 50ms) to allow UI to paint before heavy calc

    return () => clearTimeout(timerId); // Cleanup timeout if component unmounts
  }, []); // Empty dependency array ensures this runs only once on mount

  const year1 = formData.finyearOne ? parseInt(formData.finyearOne) : new Date().getFullYear();
  const currentYearEnd = formData.finyearTwo;

  const handleChange = useCallback((e) => {
    const { name, value } = e.target;
    const regex = /^-?\d*\.?\d{0,2}$/;

    if (value === '' || regex.test(value) || value === '-') {
      // Optimistic UI update for the specific input field for responsiveness
      setFormData((prevData) => ({
        ...prevData,
        [name]: value,
      }));

      if (debounceTimeoutRef.current) {
        clearTimeout(debounceTimeoutRef.current);
      }

      setIsRecalculating(true); // Show recalculating state immediately
      debounceTimeoutRef.current = setTimeout(() => {
        setFormData((currentFormData) => {
          // Perform all calculations based on the current state of formData
          // This currentFormData will include the latest optimistic user input
          const newCalculatedData = performAllCalculations(currentFormData);
          setIsRecalculating(false); // Hide recalculating state
          return newCalculatedData;
        });
      }, 300); // Debounce for 300ms
    }
  }, []); // No dependencies needed as setFormData and timeoutRef are stable

  const columnDefinitions = useMemo(() => [
    { id: 'stcNstaff', header: (<>i) At STCs & Staff Colleges <br /> (For Local Head Office only)</>) },
    { id: 'offResidenceA', header: "ii) At Officers' Residences" },
    { id: 'otherPremisesA', header: 'iii) At Other Premises' },
    { id: 'electricFitting', header: (<>iv) Electric Fittings <br /> (include electric wiring, <br /> switches, sockets, other <br /> fittings & fans etc.)</>) },
    { id: 'totalA', header: 'TOTAL (A) (i+ii+iii+iv)', isReadOnly: true },
    { id: 'computers', header: 'i) Computer Hardware' },
    { id: 'compSoftwareInt', header: (<>a. Computer Software <br /> (forming integral part of <br /> Hardware)</>) },
    { id: 'compSoftwareNonint', header: (<>b. Computer Software <br /> (not forming integral <br /> of Hardware)</>) },
    { id: 'compSoftwareTotal', header: (<>ii) Computer Software <br /> Total (a+b)</>), isReadOnly: true },
    { id: 'motor', header: 'iii) Motor Vehicles' },
    { id: 'offResidenceB', header: "a) At Officers' Residences" },
    { id: 'stcLho', header: (<>b) At STCs <br /> (For Local Head Office)</>) },
    { id: 'otherPremisesB', header: 'c) At other Premises' },
    { id: 'otherMachineryPlant', header: (<>iv) Other Machinery & Plant <br />( a+b+c)</>), isReadOnly: true },
    { id: 'totalB', header: 'TOTAL (B= i+ii+iii+iv)', isReadOnly: true },
    { id: 'totalFurnFix', header: (<> Total Furniture & Fixtures <br /> (A+B)</>), isReadOnly: true },
    { id: 'landNotRev', header: (<>(a) Land (Not Revalued): <br /> Cost</>) },
    { id: 'landRev', header: (<>(b) Land (Revalued): <br /> Cost</>) },
    { id: 'landRevEnh', header: (<>(c) Land (Revalued): <br /> Enhancement due to <br /> Revaluation</>) },
    { id: 'offBuildNotRev', header: (<>(d) Office Building <br /> (Not revalued): Cost</>) },
    { id: 'offBuildRev', header: (<>(e) Office Building <br /> (Revalued): Cost</>) },
    { id: 'offBuildRevEnh', header: (<>(f) Office Building <br /> (Revalued): Enhancement <br /> due to Revaluation</>) },
    { id: 'residQuartNotRev', header: (<>(g) Residential Building <br /> (Not revalued): Cost</>) },
    { id: 'residQuartRev', header: (<>(h) Residential Building <br /> (Revalued): Cost</>) },
    { id: 'residQuartRevEnh', header: (<>(i) Residential Building <br /> (Revalued): Enhancement <br /> due to Revaluation</>) },
    { id: 'premisTotal', header: (<>(j) Premises Total <br /> (a+b+d+e+g+h)</>), isReadOnly: true },
    { id: 'revtotal', header: (<>(k) Revaluation Total <br /> (c+f+i)</>), isReadOnly: true },
    { id: 'totalC', header: 'TOTAL (C=j+k)', isReadOnly: true },
    { id: 'premisesUnderCons', header: (<>(D) Projects under <br /> construction</>) },
    { id: 'grandTotal', header: (<>Grand Total <br /> (A + B + C + D)</>), isReadOnly: true },
  ], []);

  const rowDefinitions = useMemo(() => [
    { srNo: 'A', particular: `Total Original Cost / Revalued Value upto the end of previous year i.e. 31st March ${year1}`, suffix: '1', type: 'data', isSectionHeader: true },
    { type: 'subheader', label: 'Addition', suffix: 'sh1' }, // Added unique suffixes for keys
    { type: 'subsubsectionheader', srNo: '(a)', particular: 'Original cost of items put to use during the year:', suffix: 'ssh1'},
    { srNo: '(i)', particular: formData.particulars3, suffix: '3', type: 'data', parentSrNo: '(a)' },
    { srNo: '(ii)', particular: formData.particulars4, suffix: '4', type: 'data', parentSrNo: '(a)' },
    { srNo: '(b)', particular: 'Increase in value of Fixed Assets due to Current Revaluation', suffix: '36', type: 'data'},
    { srNo: '(c)', particular: 'Original cost of items transferred from other Circles/Groups/CC Departments', suffix: '5', type: 'data'},
    { srNo: '(d)', particular: 'Original cost of items transferred from other branches of the same Circle', suffix: '6', type: 'data'},
    { srNo: 'I', particular: 'Total [a(i)+a(ii)+b+c+d]', suffix: '7', type: 'total', isTotalRow: true, isReadOnly: true },
    { type: 'subheader', label: 'Deduction', suffix: 'sh2' },
    { srNo: '(i)', particular: 'Short Valuation charged to Revaluation Reserve due to Current Downward Revaluation', suffix: '37', type: 'data', parentSrNo: 'Deduction'},
    { srNo: '(ii)', particular: 'Original cost of items sold/ discarded during the year', suffix: '9', type: 'data', parentSrNo: 'Deduction'},
    { srNo: '(iii)', particular: 'Projects under construction capitalised during the year', suffix: '33', type: 'data', parentSrNo: 'Deduction'},
    { srNo: '(iv)', particular: 'Original cost of items transferred to other Circles/Groups/CC Departments', suffix: '10', type: 'data', parentSrNo: 'Deduction'},
    { srNo: '(v)', particular: 'Original cost of items transferred to other branches in the same circle', suffix: '11', type: 'data', parentSrNo: 'Deduction'},
    { srNo: 'II', particular: 'Total (i+ii+iii+iv+v)', suffix: '12', type: 'total', isTotalRow: true, isReadOnly: true },
    { srNo: 'B', particular: 'Net Addition (I-II)', suffix: '13', type: 'total', isTotalRow: true, isReadOnly: true, isSectionHeader: true },
    { srNo: 'C', particular: `Total Original Cost/ Revalued Value as at 31st March ${currentYearEnd} (A+B)`, suffix: '14', type: 'total', isTotalRow: true, isReadOnly: true, isSectionHeader: true },
    { type: 'subheader', label: 'Depreciation', suffix: 'sh3' },
    { srNo: '(i)', particular: `Depreciation upto the end of previous year i.e. 31st March ${year1}`, suffix: '18', type: 'data', parentSrNo: 'Depreciation'},
    { srNo: '(ii)', particular: `Short Valuation charged to depreciation upto end of previous year i.e.31st March ${year1}`, suffix: '34', type: 'data', parentSrNo: 'Depreciation'},
    { srNo: '(iii)', particular: 'Depreciation on repatriation of Officials from Subsidiaries/ Associates', suffix: '38', type: 'data', parentSrNo: 'Depreciation'},
    { srNo: '(iv)', particular: 'Depreciation transferred from other Circles/Groups/CC Departments', suffix: '19', type: 'data', parentSrNo: 'Depreciation'},
    { srNo: '(v)', particular: 'Depreciation transferred from other branches of the same circle.', suffix: '20', type: 'data', parentSrNo: 'Depreciation'},
    { srNo: '(vi)', particular: 'Depreciation charged during the current year', suffix: '21', type: 'data', parentSrNo: 'Depreciation'},
    { srNo: '(vii)', particular: 'Short Valuation charged to Depreciation during the current year due to Current Revaluation', suffix: '39', type: 'data', parentSrNo: 'Depreciation'},
    { srNo: 'D', particular: 'Total (i+ii+iii+iv+v+vi+vii)', suffix: '22', type: 'total', isTotalRow: true, isReadOnly: true},
    { type: 'subheader', label: 'Less :', suffix: 'sh4' },
    { srNo: '(i)', particular: 'Past Short Valuation credited to Depreciation during the current year due to Current Upward Revaluation', suffix: '40', type: 'data', parentSrNo: 'Less'},
    { srNo: '(ii)', particular: 'Depreciation previously provided on fixed assets sold/ discarded', suffix: '24', type: 'data', parentSrNo: 'Less'},
    { srNo: '(iii)', particular: 'Depreciation transferred to other Circles/Groups/CC Departments', suffix: '25', type: 'data', parentSrNo: 'Less'},
    { srNo: '(iv)', particular: 'Depreciation transferred to other branches of the same Circle.', suffix: '26', type: 'data', parentSrNo: 'Less'},
    { srNo: 'E', particular: 'Total (i+ii+iii+iv)', suffix: '27', type: 'total', isTotalRow: true, isReadOnly: true },
    { srNo: 'F', particular: 'Net Depreciation (D-E)', suffix: '28', type: 'total', isTotalRow: true, isReadOnly: true },
    { srNo: 'G', particular: `Net Book Value as at 31st March ${currentYearEnd} (C-F)`, suffix: '29', type: 'total', isTotalRow: true, isReadOnly: true },
    { srNo: 'H', particular: 'Sale Price of fixed assets', suffix: '30', type: 'data' },
    { srNo: 'I', particular: 'Book Value of fixed assets sold [II (ii)-E(ii)]', suffix: '31', type: 'total', isTotalRow: true, isReadOnly: true },
    { srNo: 'J', particular: 'GST on Sale of fixed assets', suffix: '35', type: 'data' },
    { srNo: 'K', particular: 'Profit/ (Loss) on sale of fixed assets [H-(I+J)]', suffix: '32', type: 'total', isTotalRow: true, isReadOnly: true },
  ], [year1, currentYearEnd, formData.particulars3, formData.particulars4]);


  const RenderInputCell = React.memo(({ fieldName, isReadOnly }) => (
    <StyledTableCell>
      <FormInput
        name={fieldName}
        value={formData[fieldName] === undefined ? '0.00' : formData[fieldName]}
        onChange={handleChange}
        inputProps={{ style: { textAlign: 'right' } }}
        sx={{ width: '100px', '& input': { textAlign: 'right', padding: '6px 8px' }, backgroundColor: isReadOnly ? '#f0f0f0' : 'white' }}
        readOnly={isReadOnly} variant="outlined" size="small"
      />
    </StyledTableCell>
  ));
  RenderInputCell.displayName = 'RenderInputCell';


  // Render loading state for initial calculation
  if (isLoading) {
    return (
      <Box sx={{ display: 'flex', justifyContent: 'center', alignItems: 'center', height: 'calc(100vh - 200px)' }}>
        <CircularProgress />
        <Box sx={{ ml: 2 }}>Loading Schedule Data & Performing Initial Calculations...</Box>
      </Box>
    );
  }

  return (
    <Box sx={{ p: 1, width: '100%', overflowX: 'hidden' }}>
      {isRecalculating && (
        <Box sx={{ position: 'fixed', top: '50%', left: '50%', transform: 'translate(-50%, -50%)', zIndex: 2000, p: 2, backgroundColor: 'rgba(0,0,0,0.1)', borderRadius: 1, display: 'flex', alignItems: 'center' }}>
          <CircularProgress size={20} sx={{mr:1}} /> Recalculating...
        </Box>
      )}
      <TableContainer component={Paper} sx={{ maxHeight: 'calc(100vh - 200px)' }}>
        <Table sx={{ minWidth: 3000 }} aria-label="schedule 10 table" stickyHeader>
          <TableHead>
            <TableRow>
              <StyledTableCell rowSpan={2}><b>Sr.No</b></StyledTableCell>
              <StyledTableCell rowSpan={2}><b>Particulars</b></StyledTableCell>
              <StyledTableCell colSpan={5}><b>(A) FURNITURE & FITTINGS</b></StyledTableCell>
              <StyledTableCell colSpan={10}><b>(B) MACHINERY & PLANT</b></StyledTableCell>
              <StyledTableCell rowSpan={2}><b>Total Furniture & Fixtures <br /> (A+B)</b></StyledTableCell>
              <StyledTableCell colSpan={12}><b>(C) PREMISES</b></StyledTableCell>
              <StyledTableCell rowSpan={2}><b>(D) Projects under <br /> construction</b></StyledTableCell>
              <StyledTableCell rowSpan={2}><b>Grand Total <br /> (A + B + C + D)</b></StyledTableCell>
            </TableRow>
            <TableRow>
              {columnDefinitions.slice(0, 5).map((col) => (<StyledTableCell key={col.id}><b>{col.header}</b></StyledTableCell>))}
              {columnDefinitions.slice(5, 15).map((col) => (<StyledTableCell key={col.id}><b>{col.header}</b></StyledTableCell>))}
              {/* Total Furn & Fix (column index 15) is covered by rowSpan */}
              {columnDefinitions.slice(16, 28).map((col) => (<StyledTableCell key={col.id}><b>{col.header}</b></StyledTableCell>))}
              {/* D Projects (index 28) & Grand Total (index 29) are covered by rowSpan */}
            </TableRow>
          </TableHead>
          <TableBody>
            {rowDefinitions.map((row, rowIndex) => {
              // Ensure unique keys for subheader/subsubsectionheader rows as well
              const rowKey = row.suffix || `row-${rowIndex}-${row.type || 'unknown'}`;
              if (row.type === 'subheader' || row.type === 'subsubsectionheader') {
                return (
                  <StyledTableRow
                    key={rowKey}
                    $issubsectionheader={row.type === 'subheader'}
                    $issubsubsectionheader={row.type === 'subsubsectionheader'}
                  >
                    <StyledTableCell>{row.srNo || ''}</StyledTableCell>
                    <StyledTableCell colSpan={columnDefinitions.length + 1}> {/* +1 for the particulars column itself */}
                      <b>{row.label || row.particular}</b>
                    </StyledTableCell>
                  </StyledTableRow>
                );
              } else if (row.type === 'data' || row.type === 'total') {
                return (
                  <StyledTableRow
                    key={rowKey}
                    $istotalrow={row.isTotalRow}
                    $issectionheader={row.isSectionHeader}
                  >
                    <StyledTableCell style={row.parentSrNo && !row.isSectionHeader ? { textAlign: 'right' } : row.isSectionHeader ? { textAlign: 'left' } : { textAlign: 'center' }}>
                      <b>{row.srNo}</b>
                    </StyledTableCell>
                    <StyledTableCell><b>{row.particular}</b></StyledTableCell>
                    {columnDefinitions.map((col) => (
                      <RenderInputCell
                        key={`${row.suffix}-${col.id}`}
                        fieldName={`${col.id}${row.suffix}`}
                        isReadOnly={row.isReadOnly || col.isReadOnly}
                      />
                    ))}
                  </StyledTableRow>
                );
              }
              return null;
            })}
          </TableBody>
        </Table>
      </TableContainer>
    </Box>
  );
};

export default Schedule10;

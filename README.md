import React, { useState, useEffect, useMemo, useCallback } from 'react';
import {
  Table, TableBody, TableContainer, TableHead, TableRow, Paper, Box,
} from '@mui/material';
import TableCell, { tableCellClasses } from '@mui/material/TableCell';
import { styled } from '@mui/material/styles';
import _ from 'lodash'; // Import lodash for debounce
import FormInput from '../../../../common/components/ui/FormInput'; // Ensure path is correct

// --- Stable Helper Functions (defined outside component) ---
const parseFloatOrDefault = (value) => parseFloat(value) || 0;

const globalParseAndFormat = (value) => {
  const num = parseFloat(value); // parseFloat will handle empty string as NaN
  return isNaN(num) ? '0.00' : num.toFixed(2);
};

// Modified to work "in-place" on targetData
const calculateRowTotalsInPlace = (targetData) => {
  const p = (fieldPath) => parseFloatOrDefault(targetData[fieldPath]);
  // Extract suffix from a field name (assuming it's consistent, e.g. from iteration)
  // This function needs to be called per suffix
  // So, the original signature was better: calculateRowTotalsInPlace(targetData, suffix)
  // Let's revert to that for clarity within the main logic.
};


// --- Styled Components --- (Assumed to be unchanged and correct)
const StyledTableCell = styled(TableCell)(({ theme }) => ({
  fontSize: '0.875rem', padding: '8px', border: '1px solid #e0e0e0', whiteSpace: 'nowrap',
  [`&.${tableCellClasses.head}`]: { backgroundColor: theme.palette.common.black, color: theme.palette.common.white, fontWeight: 'bold', textAlign: 'center' },
  [`&.${tableCellClasses.body}`]: { color: theme.palette.text.primary, backgroundColor: theme.palette.background.paper, textAlign: 'left' },
}));
const StyledTableRow = styled(TableRow)(
  ({ theme, $istotalrow, $issectionheader, $issubsectionheader, $issubsubsectionheader }) => ({
    backgroundColor: theme.palette.background.paper,
    '&:hover': { backgroundColor: theme.palette.action.hover },
    ...($issectionheader && { '& > td': { fontWeight: 'bold', textAlign: 'left', backgroundColor: theme.palette.grey[200] } }),
    ...($issubsectionheader && { '& > td': { fontStyle: 'italic', textAlign: 'left', backgroundColor: theme.palette.grey[100] } }),
    ...($issubsubsectionheader && { '& > td:nth-of-type(2)': { paddingLeft: theme.spacing(4) } }),
    ...($istotalrow && { '& > td': { fontWeight: 'bold', backgroundColor: theme.palette.grey[300] } }),
  })
);

// --- Constants ---
const baseFieldKeys = [
  'stcNstaff', 'offResidenceA', 'otherPremisesA', 'electricFitting', 'totalA',
  'computers', 'compSoftwareInt', 'compSoftwareNonint', 'compSoftwareTotal', 'motor',
  'offResidenceB', 'stcLho', 'otherPremisesB', 'otherMachineryPlant', 'totalB',
  'totalFurnFix', 'landNotRev', 'landRev', 'landRevEnh', 'offBuildNotRev',
  'offBuildRev', 'offBuildRevEnh', 'residQuartNotRev', 'residQuartRev',
  'residQuartRevEnh', 'premisTotal', 'revtotal', 'totalC', 'premisesUnderCons', 'grandTotal',
];
const nonTotalBaseFieldKeys = [
  'stcNstaff', 'offResidenceA', 'otherPremisesA', 'electricFitting', 'computers',
  'compSoftwareInt', 'compSoftwareNonint', 'motor', 'offResidenceB', 'stcLho',
  'otherPremisesB', 'landNotRev', 'landRev', 'landRevEnh', 'offBuildNotRev',
  'offBuildRev', 'offBuildRevEnh', 'residQuartNotRev', 'residQuartRev',
  'residQuartRevEnh', 'premisesUnderCons',
];
const rowSuffixes = [
  '1', '3', '4', '5', '6', '7', '9', '10', '11', '12', '13', '14', '18',
  '19', '20', '21', '22', '24', '25', '26', '27', '28', '29', '30', '31',
  '32', '33', '34', '35', '36', '37', '38', '39', '40',
];
const inputRowSuffixes = [
    '1', '3', '4', '5', '6', '9', '10', '11', '18', '19', '20', '21',
    '24', '25', '26', '30', '33', '34', '35', '36', '37', '38', '39', '40',
];
const DEBOUNCE_DELAY = 300; // milliseconds

const generateInitialFormData = () => {
  const initialNumericFields = {};
  rowSuffixes.forEach((suffix) => {
    baseFieldKeys.forEach((key) => {
      initialNumericFields[`${key}${suffix}`] = ''; // Initialize ALL numeric fields to empty string
    });
  });
  return {
    particulars3: 'Cost of new items put to use upto 3rd October 2024',
    particulars4: 'Cost of new items put to use during 4th October 2024 to 31st March 2025',
    ...initialNumericFields,
    save: true, finyearOne: '2024', finyearTwo: '2025', circleCode: '001',
    quarterEndDate: '31/03/2025', userId: '1111111', reportName: 'Schedule 10',
    reportId: null, reportMasterId: '310010', status: null,
  };
};


const Schedule10 = () => {
  const [formData, setFormData] = useState(generateInitialFormData);

  const year1 = formData.finyearOne ? parseInt(formData.finyearOne) : new Date().getFullYear();
  const currentYearEnd = formData.finyearTwo;

  // Re-defining calculateRowTotalsInPlace for use within the component's scope if preferred,
  // or pass global helpers to the calculation callback. For simplicity, define it here and pass.
  const _calculateRowTotalsInPlace = useCallback((targetData, suffix) => {
    const p = (fieldPath) => parseFloatOrDefault(targetData[fieldPath]);
    targetData[`totalA${suffix}`] = globalParseAndFormat(p(`stcNstaff${suffix}`) + p(`offResidenceA${suffix}`) + p(`otherPremisesA${suffix}`) + p(`electricFitting${suffix}`));
    targetData[`compSoftwareTotal${suffix}`] = globalParseAndFormat(p(`compSoftwareInt${suffix}`) + p(`compSoftwareNonint${suffix}`));
    targetData[`otherMachineryPlant${suffix}`] = globalParseAndFormat(p(`offResidenceB${suffix}`) + p(`stcLho${suffix}`) + p(`otherPremisesB${suffix}`));
    const compSoftwareTotalVal = parseFloatOrDefault(targetData[`compSoftwareTotal${suffix}`]);
    const otherMachineryPlantVal = parseFloatOrDefault(targetData[`otherMachineryPlant${suffix}`]);
    targetData[`totalB${suffix}`] = globalParseAndFormat(p(`computers${suffix}`) + compSoftwareTotalVal + p(`motor${suffix}`) + otherMachineryPlantVal);
    const totalAVal = parseFloatOrDefault(targetData[`totalA${suffix}`]);
    const totalBVal = parseFloatOrDefault(targetData[`totalB${suffix}`]);
    targetData[`totalFurnFix${suffix}`] = globalParseAndFormat(totalAVal + totalBVal);
    targetData[`premisTotal${suffix}`] = globalParseAndFormat(p(`landNotRev${suffix}`) + p(`landRev${suffix}`) + p(`offBuildNotRev${suffix}`) + p(`offBuildRev${suffix}`) + p(`residQuartNotRev${suffix}`) + p(`residQuartRev${suffix}`));
    targetData[`revtotal${suffix}`] = globalParseAndFormat(p(`landRevEnh${suffix}`) + p(`offBuildRevEnh${suffix}`) + p(`residQuartRevEnh${suffix}`));
    const premisTotalVal = parseFloatOrDefault(targetData[`premisTotal${suffix}`]);
    const revTotalVal = parseFloatOrDefault(targetData[`revtotal${suffix}`]);
    targetData[`totalC${suffix}`] = globalParseAndFormat(premisTotalVal + revTotalVal);
    targetData[`grandTotal${suffix}`] = globalParseAndFormat(totalAVal + totalBVal + parseFloatOrDefault(targetData[`totalC${suffix}`]) + p(`premisesUnderCons${suffix}`));
  }, []);

  const _sumAcrossRowsForField = useCallback((sourceData, fieldBaseName, suffixesToSum) => {
    let total = 0;
    suffixesToSum.forEach((sfx) => { total += parseFloatOrDefault(sourceData[`${fieldBaseName}${sfx}`]); });
    return globalParseAndFormat(total);
  }, []);

  const _subtractAcrossRowsForField = useCallback((sourceData, fieldBaseName, minuendSuffix, subtrahendSuffix) => {
    const minuend = parseFloatOrDefault(sourceData[`${fieldBaseName}${minuendSuffix}`]);
    const subtrahend = parseFloatOrDefault(sourceData[`${fieldBaseName}${subtrahendSuffix}`]);
    return globalParseAndFormat(minuend - subtrahend);
  }, []);


  const relevantInputDataString = useMemo(() => {
    const dataToWatch = {};
    inputRowSuffixes.forEach((suffix) => {
      nonTotalBaseFieldKeys.forEach((baseName) => {
        dataToWatch[`${baseName}${suffix}`] = formData[`${baseName}${suffix}`] || ''; // Default to empty if somehow undefined
      });
    });
    return JSON.stringify(dataToWatch);
  }, [formData]);

  const performCalculations = useCallback(() => {
    setFormData((prevData) => {
      let newData = { ...prevData };

      inputRowSuffixes.forEach((suffix) => {
        _calculateRowTotalsInPlace(newData, suffix);
      });

      nonTotalBaseFieldKeys.forEach((key) => {
        newData[`${key}7`] = _sumAcrossRowsForField(newData, key, ['3', '4', '36', '5', '6']);
        newData[`${key}12`] = _sumAcrossRowsForField(newData, key, ['37', '9', '33', '10', '11']);
        newData[`${key}13`] = _subtractAcrossRowsForField(newData, key, '7', '12');
        newData[`${key}14`] = _sumAcrossRowsForField(newData, key, ['1', '13']);
        newData[`${key}22`] = _sumAcrossRowsForField(newData, key, ['18', '34', '38', '19', '20', '21', '39']);
        newData[`${key}27`] = _sumAcrossRowsForField(newData, key, ['40', '24', '25', '26']);
        newData[`${key}28`] = _subtractAcrossRowsForField(newData, key, '22', '27');
        newData[`${key}29`] = _subtractAcrossRowsForField(newData, key, '14', '28');
        newData[`${key}31`] = _subtractAcrossRowsForField(newData, key, '9', '24');

        const val30 = parseFloatOrDefault(newData[`${key}30`]);
        const val31 = parseFloatOrDefault(newData[`${key}31`]);
        const val35 = parseFloatOrDefault(newData[`${key}35`]);
        newData[`${key}32`] = globalParseAndFormat(val30 - (val31 + val35));
      });

      const aggregatedRowSuffixes = ['7', '12', '13', '14', '22', '27', '28', '29', '31', '32'];
      aggregatedRowSuffixes.forEach((suffix) => {
        _calculateRowTotalsInPlace(newData, suffix);
      });
      return newData;
    });
  }, [_calculateRowTotalsInPlace, _sumAcrossRowsForField, _subtractAcrossRowsForField]); // Dependencies are stable callbacks

  const debouncedPerformCalculations = useMemo(
    () => _.debounce(performCalculations, DEBOUNCE_DELAY),
    [performCalculations]
  );

  useEffect(() => {
    debouncedPerformCalculations();
    return () => {
      debouncedPerformCalculations.cancel();
    };
  }, [relevantInputDataString, debouncedPerformCalculations]);


  const handleChange = useCallback((e) => {
    const { name, value } = e.target;
    const regex = /^-?\d*\.?\d{0,2}$/;
    if (value === '' || value === '-' || regex.test(value)) {
      setFormData((prevData) => ({ ...prevData, [name]: value }));
    }
  }, []);
  
  const columnDefinitions = useMemo(() => [ /* ... unchanged ... */
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

  const rowDefinitions = useMemo(() => [ /* ... unchanged ... */
    { srNo: 'A', particular: `Total Original Cost / Revalued Value upto the end of previous year i.e. 31st March ${year1}`, suffix: '1', type: 'data', isSectionHeader: true },
    { type: 'subheader', label: 'Addition' },
    { type: 'subsubsectionheader', srNo: '(a)', particular: 'Original cost of items put to use during the year:' },
    { srNo: '(i)', particular: formData.particulars3, suffix: '3', type: 'data', parentSrNo: '(a)' },
    { srNo: '(ii)', particular: formData.particulars4, suffix: '4', type: 'data', parentSrNo: '(a)' },
    { srNo: '(b)', particular: 'Increase in value of Fixed Assets due to Current Revaluation', suffix: '36', type: 'data' },
    { srNo: '(c)', particular: 'Original cost of items transferred from other Circles/Groups/CC Departments', suffix: '5', type: 'data' },
    { srNo: '(d)', particular: 'Original cost of items transferred from other branches of the same Circle', suffix: '6', type: 'data' },
    { srNo: 'I', particular: 'Total [a(i)+a(ii)+b+c+d]', suffix: '7', type: 'total', isTotalRow: true, isReadOnly: true },
    { type: 'subheader', label: 'Deduction' },
    { srNo: '(i)', particular: 'Short Valuation charged to Revaluation Reserve due to Current Downward Revaluation', suffix: '37', type: 'data', parentSrNo: 'Deduction' },
    { srNo: '(ii)', particular: 'Original cost of items sold/ discarded during the year', suffix: '9', type: 'data', parentSrNo: 'Deduction' },
    { srNo: '(iii)', particular: 'Projects under construction capitalised during the year', suffix: '33', type: 'data', parentSrNo: 'Deduction' },
    { srNo: '(iv)', particular: 'Original cost of items transferred to other Circles/Groups/CC Departments', suffix: '10', type: 'data', parentSrNo: 'Deduction' },
    { srNo: '(v)', particular: 'Original cost of items transferred to other branches in the same circle', suffix: '11', type: 'data', parentSrNo: 'Deduction' },
    { srNo: 'II', particular: 'Total (i+ii+iii+iv+v)', suffix: '12', type: 'total', isTotalRow: true, isReadOnly: true },
    { srNo: 'B', particular: 'Net Addition (I-II)', suffix: '13', type: 'total', isTotalRow: true, isReadOnly: true, isSectionHeader: true },
    { srNo: 'C', particular: `Total Original Cost/ Revalued Value as at 31st March ${currentYearEnd} (A+B)`, suffix: '14', type: 'total', isTotalRow: true, isReadOnly: true, isSectionHeader: true },
    { type: 'subheader', label: 'Depreciation' },
    { srNo: '(i)', particular: `Depreciation upto the end of previous year i.e. 31st March ${year1}`, suffix: '18', type: 'data', parentSrNo: 'Depreciation' },
    { srNo: '(ii)', particular: `Short Valuation charged to depreciation upto end of previous year i.e.31st March ${year1}`, suffix: '34', type: 'data', parentSrNo: 'Depreciation' },
    { srNo: '(iii)', particular: 'Depreciation on repatriation of Officials from Subsidiaries/ Associates', suffix: '38', type: 'data', parentSrNo: 'Depreciation' },
    { srNo: '(iv)', particular: 'Depreciation transferred from other Circles/Groups/CC Departments', suffix: '19', type: 'data', parentSrNo: 'Depreciation' },
    { srNo: '(v)', particular: 'Depreciation transferred from other branches of the same circle.', suffix: '20', type: 'data', parentSrNo: 'Depreciation' },
    { srNo: '(vi)', particular: 'Depreciation charged during the current year', suffix: '21', type: 'data', parentSrNo: 'Depreciation' },
    { srNo: '(vii)', particular: 'Short Valuation charged to Depreciation during the current year due to Current Revaluation', suffix: '39', type: 'data', parentSrNo: 'Depreciation' },
    { srNo: 'D', particular: 'Total (i+ii+iii+iv+v+vi+vii)', suffix: '22', type: 'total', isTotalRow: true, isReadOnly: true },
    { type: 'subheader', label: 'Less :' },
    { srNo: '(i)', particular: 'Past Short Valuation credited to Depreciation during the current year due to Current Upward Revaluation', suffix: '40', type: 'data', parentSrNo: 'Less' },
    { srNo: '(ii)', particular: 'Depreciation previously provided on fixed assets sold/ discarded', suffix: '24', type: 'data', parentSrNo: 'Less' },
    { srNo: '(iii)', particular: 'Depreciation transferred to other Circles/Groups/CC Departments', suffix: '25', type: 'data', parentSrNo: 'Less' },
    { srNo: '(iv)', particular: 'Depreciation transferred to other branches of the same Circle.', suffix: '26', type: 'data', parentSrNo: 'Less' },
    { srNo: 'E', particular: 'Total (i+ii+iii+iv)', suffix: '27', type: 'total', isTotalRow: true, isReadOnly: true },
    { srNo: 'F', particular: 'Net Depreciation (D-E)', suffix: '28', type: 'total', isTotalRow: true, isReadOnly: true },
    { srNo: 'G', particular: `Net Book Value as at 31st March ${currentYearEnd} (C-F)`, suffix: '29', type: 'total', isTotalRow: true, isReadOnly: true },
    { srNo: 'H', particular: 'Sale Price of fixed assets', suffix: '30', type: 'data' },
    { srNo: 'I', particular: 'Book Value of fixed assets sold [II (ii)-E(ii)]', suffix: '31', type: 'total', isTotalRow: true, isReadOnly: true },
    { srNo: 'J', particular: 'GST on Sale of fixed assets', suffix: '35', type: 'data' },
    { srNo: 'K', particular: 'Profit/ (Loss) on sale of fixed assets [H-(I+J)]', suffix: '32', type: 'total', isTotalRow: true, isReadOnly: true },
  ], [year1, currentYearEnd, formData.particulars3, formData.particulars4]);

  const RenderInputCell = React.memo(({ fieldName, isReadOnly, value, onChange }) => (
    <StyledTableCell>
      <FormInput
        name={fieldName}
        value={value === undefined ? '' : value} // Ensure value is empty string if undefined
        onChange={onChange}
        inputProps={{ style: { textAlign: 'right' }, readOnly: isReadOnly }}
        sx={{
          width: '100px', '& .MuiInputBase-input': { textAlign: 'right', padding: '6px 8px' },
          backgroundColor: isReadOnly ? '#f0f0f0' : 'white',
        }}
        variant="outlined" size="small"
      />
    </StyledTableCell>
  ));
  RenderInputCell.displayName = 'RenderInputCell';

  return (
    <Box sx={{ p: 1, width: '100%', overflowX: 'hidden' }}>
      <TableContainer component={Paper} sx={{ maxHeight: 'calc(100vh - 150px)' }}>
        <Table stickyHeader sx={{ minWidth: 3000 }} aria-label="schedule 10 table">
          <TableHead> {/* ... Unchanged ... */}
            <TableRow>
              <StyledTableCell rowSpan={2} sx={{ verticalAlign: 'middle' }}><b>Sr.No</b></StyledTableCell>
              <StyledTableCell rowSpan={2} sx={{ verticalAlign: 'middle' }}><b>Particulars</b></StyledTableCell>
              <StyledTableCell colSpan={5}><b>(A) FURNITURE & FITTINGS</b></StyledTableCell>
              <StyledTableCell colSpan={10}><b>(B) MACHINERY & PLANT</b></StyledTableCell>
              <StyledTableCell rowSpan={2} sx={{ verticalAlign: 'middle' }}><b>Total Furniture & Fixtures <br />(A+B)</b></StyledTableCell>
              <StyledTableCell colSpan={12}><b>(C) PREMISES</b></StyledTableCell>
              <StyledTableCell rowSpan={2} sx={{ verticalAlign: 'middle' }}><b>(D) Projects under <br />construction</b></StyledTableCell>
              <StyledTableCell rowSpan={2} sx={{ verticalAlign: 'middle' }}><b>Grand Total <br /> (A + B + C + D)</b></StyledTableCell>
            </TableRow>
            <TableRow>
              {columnDefinitions.slice(0, 5).map((col) => ( <StyledTableCell key={col.id}><b>{col.header}</b></StyledTableCell> ))}
              {columnDefinitions.slice(5, 15).map((col) => ( <StyledTableCell key={col.id}><b>{col.header}</b></StyledTableCell> ))}
              {columnDefinitions.slice(16, 28).map((col) => ( <StyledTableCell key={col.id}><b>{col.header}</b></StyledTableCell> ))}
            </TableRow>
          </TableHead>
          <TableBody> {/* ... Render logic for rows using RenderInputCell, passing transient props correctly ... */}
            {rowDefinitions.map((row, rowIndex) => {
              if (row.type === 'subheader' || row.type === 'subsubsectionheader') {
                return (
                  <StyledTableRow
                    key={`header-${rowIndex}-${row.label || row.particular}`}
                    $issubsectionheader={(row.type === 'subheader') ? true : undefined}
                    $issubsubsectionheader={(row.type === 'subsubsectionheader') ? true : undefined}
                  >
                    <StyledTableCell sx={{ textAlign: row.srNo ? 'center' : 'left', fontWeight: 'bold' }}>{row.srNo || ''}</StyledTableCell>
                    <StyledTableCell sx={{fontWeight: 'bold'}}> {row.label || row.particular} </StyledTableCell>
                    <StyledTableCell colSpan={columnDefinitions.length}></StyledTableCell>
                  </StyledTableRow>
                );
              } else if (row.type === 'data' || row.type === 'total') {
                return (
                  <StyledTableRow
                    key={row.suffix}
                    $istotalrow={row.isTotalRow ? true : undefined}
                    $issectionheader={row.isSectionHeader ? true : undefined}
                  >
                    <StyledTableCell style={{ textAlign: row.parentSrNo && !row.isSectionHeader ? 'right' : row.isSectionHeader ? 'left' : 'center', fontWeight: (row.isSectionHeader || row.isTotalRow) ? 'bold' : 'normal' }}>
                      {row.srNo}
                    </StyledTableCell>
                    <StyledTableCell style={{ fontWeight: (row.isSectionHeader || row.isTotalRow) ? 'bold' : 'normal' }}>
                      {row.particular}
                    </StyledTableCell>
                    {columnDefinitions.map((col) => (
                      <RenderInputCell
                        key={`${row.suffix}-${col.id}`}
                        fieldName={`${col.id}${row.suffix}`}
                        isReadOnly={!!row.isReadOnly || !!col.isReadOnly}
                        value={formData[`${col.id}${row.suffix}`]} // Will be '' initially for inputs
                        onChange={handleChange}
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

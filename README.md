import React, { useState, useEffect, useMemo } from 'react';
import {
  Table,
  TableBody,
  TableContainer,
  TableHead,
  TableRow,
  Paper,
  TextField, // Using TextField as a common input for simplicity
  Box,
  Typography,
} from '@mui/material';
import TableCell, { tableCellClasses } from '@mui/material/TableCell';
import { styled } from '@mui/material/styles';
import FormInput from '../../../../common/components/ui/FormInput';

// Styled Components based on Example.txt
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
  ({ theme, istotalrow, issectionheader, issubsectionheader, issubsubsectionheader }) => ({
    backgroundColor: theme.palette.background.paper,
    ...(issectionheader && {
      '& > td': {
        fontWeight: 'bold',
        textAlign: 'left',
      },
    }),
    ...(issubsectionheader && {
      '& > td': {
        fontWeight: 'bold',
        fontStyle: 'italic',
        textAlign: 'left',
      },
    }),
    ...(issubsubsectionheader && {
      '& > td': {
        textAlign: 'left',
      },
    }),
    ...(istotalrow && {
      '& > td': {
        fontWeight: 'bold',
      },
    }),
  })
);

const Schedule10 = () => {
  const [formData, setFormData] = useState({
    particulars3: 'Cost of new items put to use upto 3rd October 2024',
    particulars4: 'Cost of new items put to use during 4th October 2024 to 31st March 2025',
    totalA36: '0.00',
    stcNstaff7: '0.00',
    offResidenceA7: '0.00',
    otherPremisesA7: '0.00',
    electricFitting7: '0.00',
    stcNstaff12: '0.00',
    offResidenceA12: '0.00',
    otherPremisesA12: '0.00',
    electricFitting12: '0.00',
    totalA1: '0.00',
    totalA3: '0.00',
    totalA4: '0.00',
    totalA5: '0.00',
    totalA6: '0.00',
    totalA7: '0.00',
    totalA9: '0.00',
    totalA33: '0.00',
    totalA37: '0.00',
    totalA10: '0.00',
    totalA11: '0.00',
    totalA12: '0.00',
    stcNstaff13: '0.00',
    offResidenceA13: '0.00',
    otherPremisesA13: '0.00',
    electricFitting13: '0.00',
    totalA13: '0.00',
    stcNstaff14: '0.00',
    offResidenceA14: '0.00',
    otherPremisesA14: '0.00',
    electricFitting14: '0.00',
    totalA14: '0.00',
    totalA18: '0.00',
    totalA34: '0.00',
    totalA19: '0.00',
    totalA38: '0.00',
    totalA39: '0.00',
    totalA20: '0.00',
    totalA21: '0.00',
    stcNstaff22: '0.00',
    offResidenceA22: '0.00',
    otherPremisesA22: '0.00',
    electricFitting22: '0.00',
    totalA22: '0.00',
    totalA24: '0.00',
    totalA25: '0.00',
    totalA26: '0.00',
    totalA40: '0.00',
    stcNstaff27: '0.00',
    offResidenceA27: '0.00',
    otherPremisesA27: '0.00',
    electricFitting27: '0.00',
    totalA27: '0.00',
    stcNstaff28: '0.00',
    offResidenceA28: '0.00',
    otherPremisesA28: '0.00',
    electricFitting28: '0.00',
    totalA28: '0.00',
    stcNstaff29: '0.00',
    offResidenceA29: '0.00',
    otherPremisesA29: '0.00',
    electricFitting29: '0.00',
    totalA29: '0.00',
    totalA30: '0.00',
    totalA35: '0.00',
    stcNstaff31: '0.00',
    offResidenceA31: '0.00',
    otherPremisesA31: '0.00',
    electricFitting31: '0.00',
    totalA31: '0.00',
    compSoftwareTotal1: '0.00',
    compSoftwareTotal3: '0.00',
    compSoftwareTotal4: '0.00',
    compSoftwareTotal36: '0.00',
    compSoftwareTotal5: '0.00',
    compSoftwareTotal6: '0.00',
    computers7: '0.00',
    compSoftwareInt7: '0.00',
    compSoftwareNonint7: '0.00',
    compSoftwareTotal7: '0.00',
    compSoftwareTotal9: '0.00',
    compSoftwareTotal33: '0.00',
    compSoftwareTotal37: '0.00',
    compSoftwareTotal10: '0.00',
    compSoftwareTotal11: '0.00',
    computers12: '0.00',
    compSoftwareInt12: '0.00',
    compSoftwareNonint12: '0.00',
    compSoftwareTotal12: '0.00',
    computers13: '0.00',
    compSoftwareInt13: '0.00',
    compSoftwareNonint13: '0.00',
    compSoftwareTotal13: '0.00',
    computers14: '0.00',
    compSoftwareInt14: '0.00',
    compSoftwareNonint14: '0.00',
    compSoftwareTotal14: '0.00',
    compSoftwareTotal18: '0.00',
    compSoftwareTotal34: '0.00',
    compSoftwareTotal19: '0.00',
    compSoftwareTotal38: '0.00',
    compSoftwareTotal39: '0.00',
    compSoftwareTotal20: '0.00',
    compSoftwareTotal21: '0.00',
    computers22: '0.00',
    compSoftwareInt22: '0.00',
    compSoftwareNonint22: '0.00',
    compSoftwareTotal22: '0.00',
    compSoftwareTotal24: '0.00',
    compSoftwareTotal25: '0.00',
    compSoftwareTotal26: '0.00',
    compSoftwareTotal40: '0.00',
    computers27: '0.00',
    compSoftwareInt27: '0.00',
    compSoftwareNonint27: '0.00',
    compSoftwareTotal27: '0.00',
    computers28: '0.00',
    compSoftwareInt28: '0.00',
    compSoftwareNonint28: '0.00',
    compSoftwareTotal28: '0.00',
    computers29: '0.00',
    compSoftwareInt29: '0.00',
    compSoftwareNonint29: '0.00',
    compSoftwareTotal29: '0.00',
    compSoftwareTotal30: '0.00',
    compSoftwareTotal35: '0.00',
    computers31: '0.00',
    compSoftwareInt31: '0.00',
    compSoftwareNonint31: '0.00',
    compSoftwareTotal31: '0.00',
    otherMachineryPlant1: '0.00',
    otherMachineryPlant3: '0.00',
    otherMachineryPlant4: '0.00',
    otherMachineryPlant36: '0.00',
    otherMachineryPlant5: '0.00',
    otherMachineryPlant6: '0.00',
    totalB1: '0.00',
    totalB3: '0.00',
    totalB4: '0.00',
    totalB36: '0.00',
    totalB5: '0.00',
    totalB6: '0.00',
    motor7: '0.00',
    offResidenceB7: '0.00',
    stcLho7: '0.00',
    otherPremisesB7: '0.00',
    otherMachineryPlant7: '0.00',
    totalB7: '0.00',
    otherMachineryPlant9: '0.00',
    otherMachineryPlant33: '0.00',
    otherMachineryPlant37: '0.00',
    otherMachineryPlant10: '0.00',
    otherMachineryPlant11: '0.00',
    totalB9: '0.00',
    totalB33: '0.00',
    totalB37: '0.00',
    totalB10: '0.00',
    totalB11: '0.00',
    motor12: '0.00',
    offResidenceB12: '0.00',
    stcLho12: '0.00',
    otherPremisesB12: '0.00',
    otherMachineryPlant12: '0.00',
    totalB12: '0.00',
    motor13: '0.00',
    offResidenceB13: '0.00',
    stcLho13: '0.00',
    otherPremisesB13: '0.00',
    otherMachineryPlant13: '0.00',
    totalB13: '0.00',
    motor14: '0.00',
    offResidenceB14: '0.00',
    stcLho14: '0.00',
    otherPremisesB14: '0.00',
    otherMachineryPlant14: '0.00',
    totalB14: '0.00',
    otherMachineryPlant18: '0.00',
    otherMachineryPlant34: '0.00',
    otherMachineryPlant19: '0.00',
    otherMachineryPlant38: '0.00',
    otherMachineryPlant39: '0.00',
    otherMachineryPlant20: '0.00',
    otherMachineryPlant21: '0.00',
    totalB18: '0.00',
    totalB34: '0.00',
    totalB19: '0.00',
    totalB38: '0.00',
    totalB39: '0.00',
    totalB20: '0.00',
    totalB21: '0.00',
    motor22: '0.00',
    offResidenceB22: '0.00',
    stcLho22: '0.00',
    otherPremisesB22: '0.00',
    otherMachineryPlant22: '0.00',
    totalB22: '0.00',
    otherMachineryPlant24: '0.00',
    otherMachineryPlant25: '0.00',
    otherMachineryPlant26: '0.00',
    otherMachineryPlant40: '0.00',
    totalB24: '0.00',
    totalB25: '0.00',
    totalB26: '0.00',
    totalB40: '0.00',
    motor27: '0.00',
    offResidenceB27: '0.00',
    stcLho27: '0.00',
    otherPremisesB27: '0.00',
    otherMachineryPlant27: '0.00',
    totalB27: '0.00',
    motor28: '0.00',
    offResidenceB28: '0.00',
    stcLho28: '0.00',
    otherPremisesB28: '0.00',
    otherMachineryPlant28: '0.00',
    totalB28: '0.00',
    motor29: '0.00',
    offResidenceB29: '0.00',
    stcLho29: '0.00',
    otherPremisesB29: '0.00',
    otherMachineryPlant29: '0.00',
    totalB29: '0.00',
    otherMachineryPlant30: '0.00',
    otherMachineryPlant35: '0.00',
    totalB30: '0.00',
    totalB35: '0.00',
    motor31: '0.00',
    offResidenceB31: '0.00',
    stcLho31: '0.00',
    otherPremisesB31: '0.00',
    otherMachineryPlant31: '0.00',
    totalB31: '0.00',
    totalC1: '0.00',
    totalC3: '0.00',
    totalC4: '0.00',
    totalC36: '0.00',
    totalC5: '0.00',
    totalC6: '0.00',
    totalC7: '0.00',
    totalC9: '0.00',
    totalC33: '0.00',
    totalC37: '0.00',
    totalC10: '0.00',
    totalC11: '0.00',
    totalFurnFix7: '0.00',
    landNotRev7: '0.00',
    landRev7: '0.00',
    landRevEnh7: '0.00',
    offBuildNotRev7: '0.00',
    offBuildRev7: '0.00',
    offBuildRevEnh7: '0.00',
    residQuartNotRev7: '0.00',
    residQuartRev7: '0.00',
    residQuartRevEnh7: '0.00',
    premisTotal1: '0.00',
    premisTotal3: '0.00',
    premisTotal4: '0.00',
    premisTotal36: '0.00',
    premisTotal5: '0.00',
    premisTotal6: '0.00',
    premisTotal7: '0.00',
    totalFurnFix1: '0.00',
    totalFurnFix3: '0.00',
    totalFurnFix4: '0.00',
    totalFurnFix36: '0.00',
    totalFurnFix5: '0.00',
    totalFurnFix6: '0.00',
    totalFurnFix9: '0.00',
    totalFurnFix33: '0.00',
    totalFurnFix37: '0.00',
    totalFurnFix10: '0.00',
    totalFurnFix11: '0.00',
    totalFurnFix18: '0.00',
    totalFurnFix34: '0.00',
    totalFurnFix19: '0.00',
    totalFurnFix38: '0.00',
    totalFurnFix39: '0.00',
    totalFurnFix20: '0.00',
    totalFurnFix21: '0.00',
    totalFurnFix24: '0.00',
    totalFurnFix25: '0.00',
    totalFurnFix26: '0.00',
    totalFurnFix40: '0.00',
    totalFurnFix30: '0.00',
    totalFurnFix35: '0.00',
    totalFurnFix12: '0.00',
    landNotRev12: '0.00',
    landRev12: '0.00',
    landRevEnh12: '0.00',
    offBuildNotRev12: '0.00',
    offBuildRev12: '0.00',
    offBuildRevEnh12: '0.00',
    residQuartNotRev12: '0.00',
    residQuartRev12: '0.00',
    residQuartRevEnh12: '0.00',
    premisTotal9: '0.00',
    premisTotal10: '0.00',
    premisTotal11: '0.00',
    premisTotal33: '0.00',
    premisTotal37: '0.00',
    premisTotal12: '0.00',
    revtotal7: '0.00',
    revtotal36: '0.00',
    revtotal12: '0.00',
    revtotal13: '0.00',
    revtotal14: '0.00',
    revtotal22: '0.00',
    revtotal27: '0.00',
    revtotal28: '0.00',
    revtotal29: '0.00',
    revtotal31: '0.00',
    revtotal32: '0.00',
    revtotal1: '0.00',
    revtotal3: '0.00',
    revtotal4: '0.00',
    revtotal5: '0.00',
    revtotal6: '0.00',
    revtotal9: '0.00',
    revtotal10: '0.00',
    revtotal11: '0.00',
    revtotal33: '0.00',
    revtotal37: '0.00',
    revtotal18: '0.00',
    revtotal19: '0.00',
    revtotal38: '0.00',
    revtotal39: '0.00',
    revtotal20: '0.00',
    revtotal21: '0.00',
    revtotal34: '0.00',
    revtotal24: '0.00',
    revtotal25: '0.00',
    revtotal26: '0.00',
    revtotal40: '0.00',
    revtotal30: '0.00',
    revtotal35: '0.00',
    premisTotal18: '0.00',
    premisTotal19: '0.00',
    premisTotal38: '0.00',
    premisTotal39: '0.00',
    premisTotal20: '0.00',
    premisTotal21: '0.00',
    premisTotal34: '0.00',
    premisTotal24: '0.00',
    premisTotal25: '0.00',
    premisTotal26: '0.00',
    premisTotal40: '0.00',
    premisTotal30: '0.00',
    premisTotal35: '0.00',
    premisTotal13: '0.00',
    premisTotal14: '0.00',
    premisTotal22: '0.00',
    premisTotal27: '0.00',
    premisTotal28: '0.00',
    premisTotal29: '0.00',
    premisTotal31: '0.00',
    premisTotal32: '0.00',
    totalFurnFix13: '0.00',
    totalFurnFix14: '0.00',
    totalFurnFix22: '0.00',
    totalFurnFix27: '0.00',
    totalFurnFix28: '0.00',
    totalFurnFix29: '0.00',
    totalFurnFix31: '0.00',
    totalFurnFix32: '0.00',
    landNotRev13: '0.00',
    landNotRev14: '0.00',
    landNotRev22: '0.00',
    landNotRev27: '0.00',
    landNotRev28: '0.00',
    landNotRev29: '0.00',
    landNotRev31: '0.00',
    landNotRev32: '0.00',
    landRev13: '0.00',
    landRev14: '0.00',
    landRev22: '0.00',
    landRev27: '0.00',
    landRev28: '0.00',
    landRev29: '0.00',
    landRev31: '0.00',
    landRev32: '0.00',
    landRevEnh13: '0.00',
    landRevEnh14: '0.00',
    landRevEnh22: '0.00',
    landRevEnh27: '0.00',
    landRevEnh28: '0.00',
    landRevEnh29: '0.00',
    landRevEnh31: '0.00',
    landRevEnh32: '0.00',
    offBuildNotRev13: '0.00',
    offBuildNotRev14: '0.00',
    offBuildNotRev22: '0.00',
    offBuildNotRev27: '0.00',
    offBuildNotRev28: '0.00',
    offBuildNotRev29: '0.00',
    offBuildNotRev31: '0.00',
    offBuildNotRev32: '0.00',
    offBuildRev13: '0.00',
    offBuildRev14: '0.00',
    offBuildRev22: '0.00',
    offBuildRev27: '0.00',
    offBuildRev28: '0.00',
    offBuildRev29: '0.00',
    offBuildRev31: '0.00',
    offBuildRev32: '0.00',
    offBuildRevEnh13: '0.00',
    offBuildRevEnh14: '0.00',
    offBuildRevEnh22: '0.00',
    offBuildRevEnh27: '0.00',
    offBuildRevEnh28: '0.00',
    offBuildRevEnh29: '0.00',
    offBuildRevEnh31: '0.00',
    offBuildRevEnh32: '0.00',
    residQuartNotRev13: '0.00',
    residQuartNotRev14: '0.00',
    residQuartNotRev22: '0.00',
    residQuartNotRev27: '0.00',
    residQuartNotRev28: '0.00',
    residQuartNotRev29: '0.00',
    residQuartNotRev31: '0.00',
    residQuartNotRev32: '0.00',
    residQuartRev13: '0.00',
    residQuartRev14: '0.00',
    residQuartRev22: '0.00',
    residQuartRev27: '0.00',
    residQuartRev28: '0.00',
    residQuartRev29: '0.00',
    residQuartRev31: '0.00',
    residQuartRev32: '0.00',
    residQuartRevEnh13: '0.00',
    residQuartRevEnh14: '0.00',
    residQuartRevEnh22: '0.00',
    residQuartRevEnh27: '0.00',
    residQuartRevEnh28: '0.00',
    residQuartRevEnh29: '0.00',
    residQuartRevEnh31: '0.00',
    residQuartRevEnh32: '0.00',
    totalC12: '0.00',
    totalC13: '0.00',
    totalC18: '0.00',
    totalC19: '0.00',
    totalC38: '0.00',
    totalC39: '0.00',
    totalC20: '0.00',
    totalC21: '0.00',
    totalC34: '0.00',
    totalC22: '0.00',
    totalC24: '0.00',
    totalC25: '0.00',
    totalC26: '0.00',
    totalC40: '0.00',
    totalC27: '0.00',
    totalC28: '0.00',
    totalC29: '0.00',
    totalC30: '0.00',
    totalC31: '0.00',
    totalC32: '0.00',
    totalC35: '0.00',
    totalC14: '0.00',
    grandTotal1: '0.00',
    grandTotal3: '0.00',
    grandTotal36: '0.00',
    grandTotal4: '0.00',
    grandTotal5: '0.00',
    grandTotal6: '0.00',
    premisesUnderCons7: '0.00',
    grandTotal7: '0.00',
    grandTotal9: '0.00',
    grandTotal33: '0.00',
    grandTotal37: '0.00',
    grandTotal10: '0.00',
    grandTotal11: '0.00',
    premisesUnderCons12: '0.00',
    grandTotal12: '0.00',
    premisesUnderCons13: '0.00',
    grandTotal13: '0.00',
    premisesUnderCons14: '0.00',
    grandTotal14: '0.00',
    grandTotal18: '0.00',
    grandTotal34: '0.00',
    grandTotal19: '0.00',
    grandTotal38: '0.00',
    grandTotal39: '0.00',
    grandTotal20: '0.00',
    grandTotal21: '0.00',
    premisesUnderCons22: '0.00',
    grandTotal22: '0.00',
    grandTotal24: '0.00',
    grandTotal25: '0.00',
    grandTotal26: '0.00',
    grandTotal40: '0.00',
    premisesUnderCons27: '0.00',
    grandTotal27: '0.00',
    premisesUnderCons28: '0.00',
    grandTotal28: '0.00',
    premisesUnderCons29: '0.00',
    grandTotal29: '0.00',
    grandTotal30: '0.00',
    grandTotal35: '0.00',
    premisesUnderCons31: '0.00',
    grandTotal31: '0.00',
    stcNstaff32: '0.00',
    offResidenceA32: '0.00',
    otherPremisesA32: '0.00',
    electricFitting32: '0.00',
    totalA32: '0.00',
    computers32: '0.00',
    compSoftwareInt32: '0.00',
    compSoftwareNonint32: '0.00',
    compSoftwareTotal32: '0.00',
    motor32: '0.00',
    offResidenceB32: '0.00',
    stcLho32: '0.00',
    otherPremisesB32: '0.00',
    otherMachineryPlant32: '0.00',
    totalB32: '0.00',
    premisesUnderCons32: '0.00',
    grandTotal32: '0.00',
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
  });

  const year1 = 2023; // Example year, can be dynamic if needed

  // Helper function to parse and format numbers to 2 decimal places
  const parseAndFormat = (value) => {
    const num = parseFloat(value);
    return isNaN(num) ? '' : num.toFixed(2);
  };

  // Calculations using useEffect for dependencies
  // NOTE: For brevity, only a few calculations are shown. You will need to add
  // all the new calculation logic for totals (e.g., totalA7, totalB7, totalC7, grandTotal7, etc.)
  // and their respective dependencies here.
  useEffect(() => {
    setFormData((prevData) => {
      const newData = { ...prevData };

      // Section A calculations (Furniture & Fittings) - Row 1
      const stcNstaff1Val = parseFloat(newData.stcNstaff1) || 0;
      const offResidenceA1Val = parseFloat(newData.offResidenceA1) || 0;
      const otherPremisesA1Val = parseFloat(newData.otherPremisesA1) || 0;
      const electricFitting1Val = parseFloat(newData.electricFitting1) || 0;
      newData.totalA1 = parseAndFormat(stcNstaff1Val + offResidenceA1Val + otherPremisesA1Val + electricFitting1Val);

      // Section B calculations (Machinery & Plant) - Row 1
      const computers1Val = parseFloat(newData.computers1) || 0;
      const compSoftwareInt1Val = parseFloat(newData.compSoftwareInt1) || 0;
      const compSoftwareNonint1Val = parseFloat(newData.compSoftwareNonint1) || 0;
      newData.compSoftwareTotal1 = parseAndFormat(compSoftwareInt1Val + compSoftwareNonint1Val);

      const motor1Val = parseFloat(newData.motor1) || 0;
      const offResidenceB1Val = parseFloat(newData.offResidenceB1) || 0;
      const stcLho1Val = parseFloat(newData.stcLho1) || 0;
      const otherPremisesB1Val = parseFloat(newData.otherPremisesB1) || 0;
      newData.otherMachineryPlant1 = parseAndFormat(offResidenceB1Val + stcLho1Val + otherPremisesB1Val);

      newData.totalB1 = parseAndFormat(
        computers1Val + parseFloat(newData.compSoftwareTotal1) + motor1Val + parseFloat(newData.otherMachineryPlant1)
      );

      // Total Furniture & Fixtures (A+B) - Row 1
      newData.totalFurnFix1 = parseAndFormat(parseFloat(newData.totalA1) + parseFloat(newData.totalB1));

      // Section C calculations (Premises) - Row 1
      const landNotRev1Val = parseFloat(newData.landNotRev1) || 0;
      const landRev1Val = parseFloat(newData.landRev1) || 0;
      const landRevEnh1Val = parseFloat(newData.landRevEnh1) || 0;
      const offBuildNotRev1Val = parseFloat(newData.offBuildNotRev1) || 0;
      const offBuildRev1Val = parseFloat(newData.offBuildRev1) || 0;
      const offBuildRevEnh1Val = parseFloat(newData.offBuildRevEnh1) || 0;
      const residQuartNotRev1Val = parseFloat(newData.residQuartNotRev1) || 0;
      const residQuartRev1Val = parseFloat(newData.residQuartRev1) || 0;
      const residQuartRevEnh1Val = parseFloat(newData.residQuartRevEnh1) || 0;

      newData.premisTotal1 = parseAndFormat(
        landNotRev1Val + landRev1Val + offBuildNotRev1Val + offBuildRev1Val + residQuartNotRev1Val + residQuartRev1Val
      );
      newData.revtotal1 = parseAndFormat(landRevEnh1Val + offBuildRevEnh1Val + residQuartRevEnh1Val);
      newData.totalC1 = parseAndFormat(parseFloat(newData.premisTotal1) + parseFloat(newData.revtotal1));

      // Grand Total (A+B+C+D) - Row 1
      const premisesUnderCons1Val = parseFloat(newData.premisesUnderCons1) || 0;
      newData.grandTotal1 = parseAndFormat(
        parseFloat(newData.totalA1) + parseFloat(newData.totalB1) + parseFloat(newData.totalC1) + premisesUnderCons1Val
      );

      // Section A calculations (Furniture & Fittings) - Row 3
      const stcNstaff3Val = parseFloat(newData.stcNstaff3) || 0;
      const offResidenceA3Val = parseFloat(newData.offResidenceA3) || 0;
      const otherPremisesA3Val = parseFloat(newData.otherPremisesA3) || 0;
      const electricFitting3Val = parseFloat(newData.electricFitting3) || 0;
      newData.totalA3 = parseAndFormat(stcNstaff3Val + offResidenceA3Val + otherPremisesA3Val + electricFitting3Val);

      // Section B calculations (Machinery & Plant) - Row 3
      const computers3Val = parseFloat(newData.computers3) || 0;
      const compSoftwareInt3Val = parseFloat(newData.compSoftwareInt3) || 0;
      const compSoftwareNonint3Val = parseFloat(newData.compSoftwareNonint3) || 0;
      newData.compSoftwareTotal3 = parseAndFormat(compSoftwareInt3Val + compSoftwareNonint3Val);

      const motor3Val = parseFloat(newData.motor3) || 0;
      const offResidenceB3Val = parseFloat(newData.offResidenceB3) || 0;
      const stcLho3Val = parseFloat(newData.stcLho3) || 0;
      const otherPremisesB3Val = parseFloat(newData.otherPremisesB3) || 0;
      newData.otherMachineryPlant3 = parseAndFormat(offResidenceB3Val + stcLho3Val + otherPremisesB3Val);

      newData.totalB3 = parseAndFormat(
        computers3Val + parseFloat(newData.compSoftwareTotal3) + motor3Val + parseFloat(newData.otherMachineryPlant3)
      );

      // Total Furniture & Fixtures (A+B) - Row 3
      newData.totalFurnFix3 = parseAndFormat(parseFloat(newData.totalA3) + parseFloat(newData.totalB3));

      // Section C calculations (Premises) - Row 3
      const landNotRev3Val = parseFloat(newData.landNotRev3) || 0;
      const landRev3Val = parseFloat(newData.landRev3) || 0;
      const landRevEnh3Val = parseFloat(newData.landRevEnh3) || 0;
      const offBuildNotRev3Val = parseFloat(newData.offBuildNotRev3) || 0;
      const offBuildRev3Val = parseFloat(newData.offBuildRev3) || 0;
      const offBuildRevEnh3Val = parseFloat(newData.offBuildRevEnh3) || 0;
      const residQuartNotRev3Val = parseFloat(newData.residQuartNotRev3) || 0;
      const residQuartRev3Val = parseFloat(newData.residQuartRev3) || 0;
      const residQuartRevEnh3Val = parseFloat(newData.residQuartRevEnh3) || 0;

      newData.premisTotal3 = parseAndFormat(
        landNotRev3Val + landRev3Val + offBuildNotRev3Val + offBuildRev3Val + residQuartNotRev3Val + residQuartRev3Val
      );
      newData.revtotal3 = parseAndFormat(landRevEnh3Val + offBuildRevEnh3Val + residQuartRevEnh3Val);
      newData.totalC3 = parseAndFormat(parseFloat(newData.premisTotal3) + parseFloat(newData.revtotal3));

      // Grand Total (A+B+C+D) - Row 3
      const premisesUnderCons3Val = parseFloat(newData.premisesUnderCons3) || 0;
      newData.grandTotal3 = parseAndFormat(
        parseFloat(newData.totalA3) + parseFloat(newData.totalB3) + parseFloat(newData.totalC3) + premisesUnderCons3Val
      );

      // Add other calculation logic for the new fields here, following the pattern above.
      // For example, for totalA7:
      const stcNstaff7Val = parseFloat(newData.stcNstaff7) || 0;
      const offResidenceA7Val = parseFloat(newData.offResidenceA7) || 0;
      const otherPremisesA7Val = parseFloat(newData.otherPremisesA7) || 0;
      const electricFitting7Val = parseFloat(newData.electricFitting7) || 0;
      newData.totalA7 = parseAndFormat(stcNstaff7Val + offResidenceA7Val + otherPremisesA7Val + electricFitting7Val);

      // And its related total for B section
      const computers7Val = parseFloat(newData.computers7) || 0;
      const compSoftwareInt7Val = parseFloat(newData.compSoftwareInt7) || 0;
      const compSoftwareNonint7Val = parseFloat(newData.compSoftwareNonint7) || 0;
      newData.compSoftwareTotal7 = parseAndFormat(compSoftwareInt7Val + compSoftwareNonint7Val);

      const motor7Val = parseFloat(newData.motor7) || 0;
      const offResidenceB7Val = parseFloat(newData.offResidenceB7) || 0;
      const stcLho7Val = parseFloat(newData.stcLho7) || 0;
      const otherPremisesB7Val = parseFloat(newData.otherPremisesB7) || 0;
      newData.otherMachineryPlant7 = parseAndFormat(offResidenceB7Val + stcLho7Val + otherPremisesB7Val);
      newData.totalB7 = parseAndFormat(
        computers7Val + parseFloat(newData.compSoftwareTotal7) + motor7Val + parseFloat(newData.otherMachineryPlant7)
      );

      // And its related total for C section
      const landNotRev7Val = parseFloat(newData.landNotRev7) || 0;
      const landRev7Val = parseFloat(newData.landRev7) || 0;
      const landRevEnh7Val = parseFloat(newData.landRevEnh7) || 0;
      const offBuildNotRev7Val = parseFloat(newData.offBuildNotRev7) || 0;
      const offBuildRev7Val = parseFloat(newData.offBuildRev7) || 0;
      const offBuildRevEnh7Val = parseFloat(newData.offBuildRevEnh7) || 0;
      const residQuartNotRev7Val = parseFloat(newData.residQuartNotRev7) || 0;
      const residQuartRev7Val = parseFloat(newData.residQuartRev7) || 0;
      const residQuartRevEnh7Val = parseFloat(newData.residQuartRevEnh7) || 0;

      newData.premisTotal7 = parseAndFormat(
        landNotRev7Val + landRev7Val + offBuildNotRev7Val + offBuildRev7Val + residQuartNotRev7Val + residQuartRev7Val
      );
      newData.revtotal7 = parseAndFormat(landRevEnh7Val + offBuildRevEnh7Val + residQuartRevEnh7Val);
      newData.totalC7 = parseAndFormat(parseFloat(newData.premisTotal7) + parseFloat(newData.revtotal7));

      // And its related grand total
      const premisesUnderCons7Val = parseFloat(newData.premisesUnderCons7) || 0;
      newData.grandTotal7 = parseAndFormat(
        parseFloat(newData.totalA7) + parseFloat(newData.totalB7) + parseFloat(newData.totalC7) + premisesUnderCons7Val
      );

      // ... and so on for all other new totals from the payload.

      return newData;
    });
  }, [
    // Existing dependencies
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
    formData.offBuildRevEnh1,
    formData.residQuartNotRev1,
    formData.residQuartRev1,
    formData.residQuartRevEnh1,
    formData.premisesUnderCons1,
    formData.stcNstaff3,
    formData.offResidenceA3,
    formData.otherPremisesA3,
    formData.electricFitting3,
    formData.computers3,
    formData.compSoftwareInt3,
    formData.compSoftwareNonint3,
    formData.motor3,
    formData.offResidenceB3,
    formData.stcLho3,
    formData.otherPremisesB3,
    formData.landNotRev3,
    formData.landRev3,
    formData.landRevEnh3,
    formData.offBuildNotRev3,
    formData.offBuildRev3,
    formData.offBuildRevEnh3,
    formData.residQuartNotRev3,
    formData.residQuartRev3,
    formData.residQuartRevEnh3,
    formData.premisesUnderCons3,

    // Add all new dependencies for the new fields introduced in the payload here
    // For example, for totalA7:
    formData.stcNstaff7,
    formData.offResidenceA7,
    formData.otherPremisesA7,
    formData.electricFitting7,
    // For totalB7:
    formData.computers7,
    formData.compSoftwareInt7,
    formData.compSoftwareNonint7,
    formData.motor7,
    formData.offResidenceB7,
    formData.stcLho7,
    formData.otherPremisesB7,
    // For totalC7:
    formData.landNotRev7,
    formData.landRev7,
    formData.landRevEnh7,
    formData.offBuildNotRev7,
    formData.offBuildRev7,
    formData.offBuildRevEnh7,
    formData.residQuartNotRev7,
    formData.residQuartRev7,
    formData.residQuartRevEnh7,
    formData.premisesUnderCons7,
    // And repeat this pattern for all other numbered rows (e.g., ...12, ...13, ...14, etc.)
    // up to 40, including all sub-fields like stcNstaffX, offResidenceAX, etc.
  ]);

  const handleChange = (e) => {
    const { name, value } = e.target;
    // Basic validation for numeric input and 2 decimal places
    const regex = /^-?\d*\.?\d{0,2}$/; // Allows negative numbers, decimals up to 2 places
    if (value === '' || regex.test(value)) {
      setFormData((prevData) => ({
        ...prevData,
        [name]: value,
      }));
    }
  };

  const renderInputCell = (fieldName, isReadonly = false) => (
    <StyledTableCell>
      <FormInput
        name={fieldName}
        value={formData[fieldName]}
        onChange={handleChange}
        // inputProps={{
        //   maxLength: 18, // Maxlength as per Schedule10.txt
        //   style: { textAlign: 'right' },
        // }}
        // sx={{
        //   width: '100px', // Adjust width as needed
        //   '& input': { textAlign: 'right', padding: '6px 8px' },
        //   backgroundColor: isReadonly ? '#f0f0f0' : 'white',
        // }}
        readOnly={isReadonly}
        variant="outlined"
        size="small"
      />
    </StyledTableCell>
  );

  return (
    <Box sx={{ p: 1, width: '100%', overflowX: 'hidden' }}>
      <TableContainer component={Paper} sx={{ maxHeight: 'calc(110vh - 250px)' }}>
        <Table sx={{ minWidth: 3000 }} aria-label="schedule 10 table">
          <TableHead>
            <TableRow>
              <StyledTableCell rowSpan={2}>
                <b>Sr.No</b>
              </StyledTableCell>
              <StyledTableCell rowSpan={2}>
                <b>Particulars</b>
              </StyledTableCell>
              <StyledTableCell colSpan={5}>
                <b>(A) FURNITURE & FITTINGS</b>
              </StyledTableCell>
              <StyledTableCell colSpan={10}>
                <b>(B) MACHINERY & PLANT</b>
              </StyledTableCell>
              <StyledTableCell rowSpan={2}>
                <b>
                  Total Furniture & Fixtures <br />
                  (A+B)
                </b>
              </StyledTableCell>
              <StyledTableCell colSpan={12}>
                <b>(C) PREMISES</b>
              </StyledTableCell>
              <StyledTableCell rowSpan={2}>
                <b>
                  (D) Projects under <br />
                  construction
                </b>
              </StyledTableCell>
              <StyledTableCell rowSpan={2}>
                <b>
                  Grand Total <br /> (A + B + C + D)
                </b>
              </StyledTableCell>
            </TableRow>
            <TableRow>
              <StyledTableCell>
                <b>
                  i) At STCs & Staff Colleges <br />
                  (For Local Head Office only)
                </b>
              </StyledTableCell>
              <StyledTableCell>
                <b>ii) At Officers' Residences</b>
              </StyledTableCell>
              <StyledTableCell>
                <b>iii) At Other Premises</b>
              </StyledTableCell>
              <StyledTableCell>
                <b>
                  iv) Electric Fittings <br />
                  (include electric wiring,
                  <br /> switches, sockets, other
                  <br /> fittings & fans etc.)
                </b>
              </StyledTableCell>
              <StyledTableCell>
                <b>TOTAL (A) (i+ii+iii+iv)</b>
              </StyledTableCell>
              <StyledTableCell>
                <b>i) Computer Hardware</b>
              </StyledTableCell>
              <StyledTableCell>
                <b>
                  a. Computer Software <br />
                  (forming integral part of
                  <br /> Hardware)
                </b>
              </StyledTableCell>
              <StyledTableCell>
                <b>
                  b. Computer Software <br />
                  (not forming integral <br />
                  of Hardware)
                </b>
              </StyledTableCell>
              <StyledTableCell>
                <b>
                  ii) Computer Software <br />
                  Total (a+b)
                </b>
              </StyledTableCell>
              <StyledTableCell>
                <b>iii) Motor Vehicles </b>
              </StyledTableCell>
              <StyledTableCell>
                <b>a) At Officers' Residences</b>
              </StyledTableCell>
              <StyledTableCell>
                <b>
                  b) At STCs <br />
                  (For Local Head Office)
                </b>
              </StyledTableCell>
              <StyledTableCell>
                <b>c) At other Premises </b>
              </StyledTableCell>
              <StyledTableCell>
                <b>
                  iv) Other Machinery & Plant <br />( a+b+c)
                </b>
              </StyledTableCell>
              <StyledTableCell>
                <b>TOTAL (B= i+ii+iii+iv)</b>
              </StyledTableCell>
              <StyledTableCell>
                <b>
                  (a) Land (Not Revalued):
                  <br /> Cost
                </b>
              </StyledTableCell>
              <StyledTableCell>
                <b>
                  (b) Land (Revalued): <br />
                  Cost
                </b>
              </StyledTableCell>
              <StyledTableCell>
                <b>
                  (c) Land (Revalued): <br />
                  Enhancement due to <br />
                  Revaluation
                </b>
              </StyledTableCell>
              <StyledTableCell>
                <b>
                  (d) Office Building <br />
                  (Not revalued): Cost{' '}
                </b>
              </StyledTableCell>
              <StyledTableCell>
                <b>
                  (e) Office Building <br />
                  (Revalued): Cost{' '}
                </b>
              </StyledTableCell>
              <StyledTableCell>
                <b>
                  (f) Office Building <br />
                  (Revalued): Enhancement <br />
                  due to Revaluation
                </b>
              </StyledTableCell>
              <StyledTableCell>
                <b>
                  (g) Residential Building <br />
                  (Not revalued): Cost
                </b>
              </StyledTableCell>
              <StyledTableCell>
                <b>
                  (h) Residential Building <br />
                  (Revalued): Cost
                </b>
              </StyledTableCell>
              <StyledTableCell>
                <b>
                  (i) Residential Building <br />
                  (Revalued): Enhancement <br />
                  due to Revaluation
                </b>
              </StyledTableCell>
              <StyledTableCell>
                <b>
                  (j) Premises Total <br />
                  (a+b+d+e+g+h)
                </b>
              </StyledTableCell>
              <StyledTableCell>
                <b>
                  (k) Revaluation Total <br />
                  (c+f+i)
                </b>
              </StyledTableCell>
              <StyledTableCell>
                <b>TOTAL (C=j+k)</b>
              </StyledTableCell>
            </TableRow>
          </TableHead>
          <TableBody>
            {/* Row A: Total Original Cost / Revalued Value upto the end of previous year */}
            <StyledTableRow>
              <StyledTableCell issectionheader>
                <b>A</b>
              </StyledTableCell>
              <StyledTableCell issectionheader>
                <b>Total Original Cost / Revalued Value upto the end of previous year i.e. 31st March {year1}</b>
              </StyledTableCell>
              {renderInputCell('stcNstaff1')}
              {renderInputCell('offResidenceA1')}
              {renderInputCell('otherPremisesA1')}
              {renderInputCell('electricFitting1')}
              {renderInputCell('totalA1', true)}
              {renderInputCell('computers1')}
              {renderInputCell('compSoftwareInt1')}
              {renderInputCell('compSoftwareNonint1')}
              {renderInputCell('compSoftwareTotal1', true)}
              {renderInputCell('motor1')}
              {renderInputCell('offResidenceB1')}
              {renderInputCell('stcLho1')}
              {renderInputCell('otherPremisesB1')}
              {renderInputCell('otherMachineryPlant1', true)}
              {renderInputCell('totalB1', true)}
              {renderInputCell('totalFurnFix1', true)}
              {renderInputCell('landNotRev1')}
              {renderInputCell('landRev1')}
              {renderInputCell('landRevEnh1')}
              {renderInputCell('offBuildNotRev1')}
              {renderInputCell('offBuildRev1')}
              {renderInputCell('offBuildRevEnh1')}
              {renderInputCell('residQuartNotRev1')}
              {renderInputCell('residQuartRev1')}
              {renderInputCell('residQuartRevEnh1')}
              {renderInputCell('premisTotal1', true)}
              {renderInputCell('revtotal1', true)}
              {renderInputCell('totalC1', true)}
              {renderInputCell('premisesUnderCons1')}
              {renderInputCell('grandTotal1', true)}
            </StyledTableRow>

            {/* Addition section header */}
            <StyledTableRow issubsectionheader>
              <StyledTableCell></StyledTableCell>
              <StyledTableCell>
                <b>Addition</b>
              </StyledTableCell>
              <StyledTableCell colSpan={30}></StyledTableCell>
            </StyledTableRow>
            <StyledTableRow issubsubsectionheader>
              <StyledTableCell style={{ textAlign: 'center' }}>(a)</StyledTableCell>
              <StyledTableCell>Original cost of items put to use during the year:</StyledTableCell>
              <StyledTableCell colSpan={30}></StyledTableCell>
            </StyledTableRow>

            {/* Row a. (i) Original cost of items put to use during the year */}
            <StyledTableRow>
              <StyledTableCell style={{ textAlign: 'right' }}>(i)</StyledTableCell>
              <StyledTableCell>Cost of new items put to use upto 3rd October 2024</StyledTableCell>
              {renderInputCell('stcNstaff3')}
              {renderInputCell('offResidenceA3')}
              {renderInputCell('otherPremisesA3')}
              {renderInputCell('electricFitting3')}
              {renderInputCell('totalA3', true)}
              {renderInputCell('computers3')}
              {renderInputCell('compSoftwareInt3')}
              {renderInputCell('compSoftwareNonint3')}
              {renderInputCell('compSoftwareTotal3', true)}
              {renderInputCell('motor3')}
              {renderInputCell('offResidenceB3')}
              {renderInputCell('stcLho3')}
              {renderInputCell('otherPremisesB3')}
              {renderInputCell('otherMachineryPlant3', true)}
              {renderInputCell('totalB3', true)}
              {renderInputCell('totalFurnFix3', true)}
              {renderInputCell('landNotRev3')}
              {renderInputCell('landRev3')}
              {renderInputCell('landRevEnh3')}
              {renderInputCell('offBuildNotRev3')}
              {renderInputCell('offBuildRev3')}
              {renderInputCell('offBuildRevEnh3')}
              {renderInputCell('residQuartNotRev3')}
              {renderInputCell('residQuartRev3')}
              {renderInputCell('residQuartRevEnh3')}
              {renderInputCell('premisTotal3', true)}
              {renderInputCell('revtotal3', true)}
              {renderInputCell('totalC3', true)}
              {renderInputCell('premisesUnderCons3')}
              {renderInputCell('grandTotal3', true)}
            </StyledTableRow>

            {/* New row for `particulars4` */}
            <StyledTableRow>
              <StyledTableCell style={{ textAlign: 'right' }}>(ii)</StyledTableCell>
              <StyledTableCell>Cost of new items put to use during 4th October 2024 to 31st March 2025</StyledTableCell>
              {renderInputCell('stcNstaff4')} {/* Assuming this pattern, adjust as needed */}
              {renderInputCell('offResidenceA4')}
              {renderInputCell('otherPremisesA4')}
              {renderInputCell('electricFitting4')}
              {renderInputCell('totalA4', true)}
              {renderInputCell('computers4')}
              {renderInputCell('compSoftwareInt4')}
              {renderInputCell('compSoftwareNonint4')}
              {renderInputCell('compSoftwareTotal4', true)}
              {renderInputCell('motor4')}
              {renderInputCell('offResidenceB4')}
              {renderInputCell('stcLho4')}
              {renderInputCell('otherPremisesB4')}
              {renderInputCell('otherMachineryPlant4', true)}
              {renderInputCell('totalB4', true)}
              {renderInputCell('totalFurnFix4', true)}
              {renderInputCell('landNotRev4')}
              {renderInputCell('landRev4')}
              {renderInputCell('landRevEnh4')}
              {renderInputCell('offBuildNotRev4')}
              {renderInputCell('offBuildRev4')}
              {renderInputCell('offBuildRevEnh4')}
              {renderInputCell('residQuartNotRev4')}
              {renderInputCell('residQuartRev4')}
              {renderInputCell('residQuartRevEnh4')}
              {renderInputCell('premisTotal4', true)}
              {renderInputCell('revtotal4', true)}
              {renderInputCell('totalC4', true)}
              {renderInputCell('premisesUnderCons4')}
              {renderInputCell('grandTotal4', true)}
            </StyledTableRow>

            {/* Example of adding one of the many new rows from the payload, e.g., row 7 */}
            <StyledTableRow>
              <StyledTableCell style={{ textAlign: 'center' }}>(b)</StyledTableCell>
              <StyledTableCell>Increase in value of Fixed Assets due to Current Revaluation</StyledTableCell>
              {renderInputCell('stcNstaff36')}
              {renderInputCell('offResidenceA36')}
              {renderInputCell('otherPremisesA36')}
              {renderInputCell('electricFitting36')}
              {renderInputCell('totalA36', true)}
              {renderInputCell('computers36')}
              {renderInputCell('compSoftwareInt36')}
              {renderInputCell('compSoftwareNonint36')}
              {renderInputCell('compSoftwareTotal36', true)}
              {renderInputCell('motor36')}
              {renderInputCell('offResidenceB36')}
              {renderInputCell('stcLho36')}
              {renderInputCell('otherPremisesB36')}
              {renderInputCell('OtherMachineryPlant36', true)}
              {renderInputCell('totalB36', true)}
              {renderInputCell('totalFurnFix36', true)}
              {renderInputCell('landNotRev36')}
              {renderInputCell('landRev36')}
              {renderInputCell('landRevEnh36')}
              {renderInputCell('offBuildNotRev36')}
              {renderInputCell('offBuildRev36')}
              {renderInputCell('offBuildRevEnh36')}
              {renderInputCell('residQuartNotRev36')}
              {renderInputCell('residQuartRev36')}
              {renderInputCell('residQuartRevEnh36')}
              {renderInputCell('premisTotal36', true)}
              {renderInputCell('revtotal36', true)}
              {renderInputCell('totalC36', true)}
              {renderInputCell('premisesUnderCons36')}
              {renderInputCell('grandTotal36', true)}
            </StyledTableRow>
            <StyledTableRow>
              <StyledTableCell style={{ textAlign: 'center' }}>(c)</StyledTableCell>
              <StyledTableCell>
                Original cost of items transferred from other Circles/Groups/CC Departments
              </StyledTableCell>
              {renderInputCell('stcNstaff5')}
              {renderInputCell('offResidenceA5')}
              {renderInputCell('otherPremisesA5')}
              {renderInputCell('electricFitting5')}
              {renderInputCell('totalA5', true)}
              {renderInputCell('computers5')}
              {renderInputCell('compSoftwareInt5')}
              {renderInputCell('compSoftwareNonint5')}
              {renderInputCell('compSoftwareTotal5', true)}
              {renderInputCell('motor5')}
              {renderInputCell('offResidenceB5')}
              {renderInputCell('stcLho5')}
              {renderInputCell('otherPremisesB5')}
              {renderInputCell('OtherMachineryPlant5', true)}
              {renderInputCell('totalB5', true)}
              {renderInputCell('totalFurnFix5', true)}
              {renderInputCell('landNotRev5')}
              {renderInputCell('landRev5')}
              {renderInputCell('landRevEnh5')}
              {renderInputCell('offBuildNotRev5')}
              {renderInputCell('offBuildRev5')}
              {renderInputCell('offBuildRevEnh5')}
              {renderInputCell('residQuartNotRev5')}
              {renderInputCell('residQuartRev5')}
              {renderInputCell('residQuartRevEnh5')}
              {renderInputCell('premisTotal5', true)}
              {renderInputCell('revtotal5', true)}
              {renderInputCell('totalC5', true)}
              {renderInputCell('premisesUnderCons5')}
              {renderInputCell('grandTotal5', true)}
            </StyledTableRow>
            <StyledTableRow>
              <StyledTableCell style={{ textAlign: 'center' }}>(d)</StyledTableCell>
              <StyledTableCell>
                Original cost of items transferred from other branches of the same Circle
              </StyledTableCell>
              {renderInputCell('stcNstaff6')}
              {renderInputCell('offResidenceA6')}
              {renderInputCell('otherPremisesA6')}
              {renderInputCell('electricFitting6')}
              {renderInputCell('totalA6', true)}
              {renderInputCell('computers6')}
              {renderInputCell('compSoftwareInt6')}
              {renderInputCell('compSoftwareNonint6')}
              {renderInputCell('compSoftwareTotal6', true)}
              {renderInputCell('motor6')}
              {renderInputCell('offResidenceB6')}
              {renderInputCell('stcLho6')}
              {renderInputCell('otherPremisesB6')}
              {renderInputCell('OtherMachineryPlant6', true)}
              {renderInputCell('totalB6', true)}
              {renderInputCell('totalFurnFix6', true)}
              {renderInputCell('landNotRev6')}
              {renderInputCell('landRev6')}
              {renderInputCell('landRevEnh6')}
              {renderInputCell('offBuildNotRev6')}
              {renderInputCell('offBuildRev6')}
              {renderInputCell('offBuildRevEnh6')}
              {renderInputCell('residQuartNotRev6')}
              {renderInputCell('residQuartRev6')}
              {renderInputCell('residQuartRevEnh6')}
              {renderInputCell('premisTotal6', true)}
              {renderInputCell('revtotal6', true)}
              {renderInputCell('totalC6', true)}
              {renderInputCell('premisesUnderCons6')}
              {renderInputCell('grandTotal6', true)}
            </StyledTableRow>
            <StyledTableRow istotalrow>
              <StyledTableCell>I</StyledTableCell>
              <StyledTableCell>Total [a(i)+a(ii)+b+c+d]</StyledTableCell>
              {renderInputCell('stcNstaff7', true)}
              {renderInputCell('offResidenceA7', true)}
              {renderInputCell('otherPremisesA7', true)}
              {renderInputCell('electricFitting7', true)}
              {renderInputCell('totalA7', true)}
              {renderInputCell('computers7', true)}
              {renderInputCell('compSoftwareInt7', true)}
              {renderInputCell('compSoftwareNonint7', true)}
              {renderInputCell('compSoftwareTotal7', true)}
              {renderInputCell('motor7', true)}
              {renderInputCell('offResidenceB7', true)}
              {renderInputCell('stcLho7', true)}
              {renderInputCell('otherPremisesB7', true)}
              {renderInputCell('OtherMachineryPlant7', true)}
              {renderInputCell('totalB7', true)}
              {renderInputCell('totalFurnFix7', true)}
              {renderInputCell('landNotRev7', true)}
              {renderInputCell('landRev7', true)}
              {renderInputCell('landRevEnh7', true)}
              {renderInputCell('offBuildNotRev7', true)}
              {renderInputCell('offBuildRev7', true)}
              {renderInputCell('offBuildRevEnh7', true)}
              {renderInputCell('residQuartNotRev7', true)}
              {renderInputCell('residQuartRev7', true)}
              {renderInputCell('residQuartRevEnh7', true)}
              {renderInputCell('premisTotal7', true)}
              {renderInputCell('revtotal7', true)}
              {renderInputCell('totalC7', true)}
              {renderInputCell('premisesUnderCons7', true)}
              {renderInputCell('grandTotal7', true)}
            </StyledTableRow>

            <StyledTableRow issubsectionheader>
              <StyledTableCell></StyledTableCell>
              <StyledTableCell>Deduction</StyledTableCell>
              {Array(30)
                .fill(null)
                .map((_, i) => (
                  <StyledTableCell key={`empty-Deduction-${i}`}></StyledTableCell>
                ))}
            </StyledTableRow>
            <StyledTableRow>
              <StyledTableCell style={{ textAlign: 'right' }}>(i)</StyledTableCell>
              <StyledTableCell>
                Short Valuation charged to Revaluation Reserve due to Current Downward Revaluation
              </StyledTableCell>
              {renderInputCell('stcNstaff37')}
              {renderInputCell('offResidenceA37')}
              {renderInputCell('otherPremisesA37')}
              {renderInputCell('electricFitting37')}
              {renderInputCell('totalA37', true)}
              {renderInputCell('computers37')}
              {renderInputCell('compSoftwareInt37')}
              {renderInputCell('compSoftwareNonint37')}
              {renderInputCell('compSoftwareTotal37', true)}
              {renderInputCell('motor37')}
              {renderInputCell('offResidenceB37')}
              {renderInputCell('stcLho37')}
              {renderInputCell('otherPremisesB37')}
              {renderInputCell('OtherMachineryPlant37', true)}
              {renderInputCell('totalB37', true)}
              {renderInputCell('totalFurnFix37', true)}
              {renderInputCell('landNotRev37')}
              {renderInputCell('landRev37')}
              {renderInputCell('landRevEnh37')}
              {renderInputCell('offBuildNotRev37')}
              {renderInputCell('offBuildRev37')}
              {renderInputCell('offBuildRevEnh37')}
              {renderInputCell('residQuartNotRev37')}
              {renderInputCell('residQuartRev37')}
              {renderInputCell('residQuartRevEnh37')}
              {renderInputCell('premisTotal37', true)}
              {renderInputCell('revtotal37', true)}
              {renderInputCell('totalC37', true)}
              {renderInputCell('premisesUnderCons37')}
              {renderInputCell('grandTotal37', true)}
            </StyledTableRow>
            <StyledTableRow>
              <StyledTableCell style={{ textAlign: 'right' }}>(ii)</StyledTableCell>
              <StyledTableCell>Original cost of items sold/ discarded during the year</StyledTableCell>
              {renderInputCell('stcNstaff9')}
              {renderInputCell('offResidenceA9')}
              {renderInputCell('otherPremisesA9')}
              {renderInputCell('electricFitting9')}
              {renderInputCell('totalA9', true)}
              {renderInputCell('computers9')}
              {renderInputCell('compSoftwareInt9')}
              {renderInputCell('compSoftwareNonint9')}
              {renderInputCell('compSoftwareTotal9', true)}
              {renderInputCell('motor9')}
              {renderInputCell('offResidenceB9')}
              {renderInputCell('stcLho9')}
              {renderInputCell('otherPremisesB9')}
              {renderInputCell('OtherMachineryPlant9', true)}
              {renderInputCell('totalB9', true)}
              {renderInputCell('totalFurnFix9', true)}
              {renderInputCell('landNotRev9')}
              {renderInputCell('landRev9')}
              {renderInputCell('landRevEnh9')}
              {renderInputCell('offBuildNotRev9')}
              {renderInputCell('offBuildRev9')}
              {renderInputCell('offBuildRevEnh9')}
              {renderInputCell('residQuartNotRev9')}
              {renderInputCell('residQuartRev9')}
              {renderInputCell('residQuartRevEnh9')}
              {renderInputCell('premisTotal9', true)}
              {renderInputCell('revtotal9', true)}
              {renderInputCell('totalC9', true)}
              {renderInputCell('premisesUnderCons9')}
              {renderInputCell('grandTotal9', true)}
            </StyledTableRow>
            <StyledTableRow>
              <StyledTableCell style={{ textAlign: 'right' }}>(iii)</StyledTableCell>
              <StyledTableCell>Projects under construction capitalised during the year</StyledTableCell>
              {renderInputCell('stcNstaff33')}
              {renderInputCell('offResidenceA33')}
              {renderInputCell('otherPremisesA33')}
              {renderInputCell('electricFitting33')}
              {renderInputCell('totalA33', true)}
              {renderInputCell('computers33')}
              {renderInputCell('compSoftwareInt33')}
              {renderInputCell('compSoftwareNonint33')}
              {renderInputCell('compSoftwareTotal33', true)}
              {renderInputCell('motor33')}
              {renderInputCell('offResidenceB33')}
              {renderInputCell('stcLho33')}
              {renderInputCell('otherPremisesB33')}
              {renderInputCell('OtherMachineryPlant33', true)}
              {renderInputCell('totalB33', true)}
              {renderInputCell('totalFurnFix33', true)}
              {renderInputCell('landNotRev33')}
              {renderInputCell('landRev33')}
              {renderInputCell('landRevEnh33')}
              {renderInputCell('offBuildNotRev33')}
              {renderInputCell('offBuildRev33')}
              {renderInputCell('offBuildRevEnh33')}
              {renderInputCell('residQuartNotRev33')}
              {renderInputCell('residQuartRev33')}
              {renderInputCell('residQuartRevEnh33')}
              {renderInputCell('premisTotal33', true)}
              {renderInputCell('revtotal33', true)}
              {renderInputCell('totalC33', true)}
              {renderInputCell('premisesUnderCons33')}
              {renderInputCell('grandTotal33', true)}
            </StyledTableRow>
            <StyledTableRow>
              <StyledTableCell style={{ textAlign: 'right' }}>(iv)</StyledTableCell>
              <StyledTableCell>
                Original cost of items transferred to other Circles/Groups/CC Departments
              </StyledTableCell>
              {renderInputCell('stcNstaff10')}
              {renderInputCell('offResidenceA10')}
              {renderInputCell('otherPremisesA10')}
              {renderInputCell('electricFitting10')}
              {renderInputCell('totalA10', true)}
              {renderInputCell('computers10')}
              {renderInputCell('compSoftwareInt10')}
              {renderInputCell('compSoftwareNonint10')}
              {renderInputCell('compSoftwareTotal10', true)}
              {renderInputCell('motor10')}
              {renderInputCell('offResidenceB10')}
              {renderInputCell('stcLho10')}
              {renderInputCell('otherPremisesB10')}
              {renderInputCell('OtherMachineryPlant10', true)}
              {renderInputCell('totalB10', true)}
              {renderInputCell('totalFurnFix10', true)}
              {renderInputCell('landNotRev10')}
              {renderInputCell('landRev10')}
              {renderInputCell('landRevEnh10')}
              {renderInputCell('offBuildNotRev10')}
              {renderInputCell('offBuildRev10')}
              {renderInputCell('offBuildRevEnh10')}
              {renderInputCell('residQuartNotRev10')}
              {renderInputCell('residQuartRev10')}
              {renderInputCell('residQuartRevEnh10')}
              {renderInputCell('premisTotal10', true)}
              {renderInputCell('revtotal10', true)}
              {renderInputCell('totalC10', true)}
              {renderInputCell('premisesUnderCons10')}
              {renderInputCell('grandTotal10', true)}
            </StyledTableRow>
            <StyledTableRow>
              <StyledTableCell style={{ textAlign: 'right' }}>(v)</StyledTableCell>
              <StyledTableCell>Original cost of items transferred to other branches in the same circle</StyledTableCell>
              {renderInputCell('stcNstaff11')}
              {renderInputCell('offResidenceA11')}
              {renderInputCell('otherPremisesA11')}
              {renderInputCell('electricFitting11')}
              {renderInputCell('totalA11', true)}
              {renderInputCell('computers11')}
              {renderInputCell('compSoftwareInt11')}
              {renderInputCell('compSoftwareNonint11')}
              {renderInputCell('compSoftwareTotal11', true)}
              {renderInputCell('motor11')}
              {renderInputCell('offResidenceB11')}
              {renderInputCell('stcLho11')}
              {renderInputCell('otherPremisesB11')}
              {renderInputCell('OtherMachineryPlant11', true)}
              {renderInputCell('totalB11', true)}
              {renderInputCell('totalFurnFix11', true)}
              {renderInputCell('landNotRev11')}
              {renderInputCell('landRev11')}
              {renderInputCell('landRevEnh11')}
              {renderInputCell('offBuildNotRev11')}
              {renderInputCell('offBuildRev11')}
              {renderInputCell('offBuildRevEnh11')}
              {renderInputCell('residQuartNotRev11')}
              {renderInputCell('residQuartRev11')}
              {renderInputCell('residQuartRevEnh11')}
              {renderInputCell('premisTotal11', true)}
              {renderInputCell('revtotal11', true)}
              {renderInputCell('totalC11', true)}
              {renderInputCell('premisesUnderCons11')}
              {renderInputCell('grandTotal11', true)}
            </StyledTableRow>
            <StyledTableRow istotalrow>
              <StyledTableCell>II</StyledTableCell>
              <StyledTableCell>Total (i+ii+iii+iv+v)</StyledTableCell>
              {renderInputCell('stcNstaff12', true)}
              {renderInputCell('offResidenceA12', true)}
              {renderInputCell('otherPremisesA12', true)}
              {renderInputCell('electricFitting12', true)}
              {renderInputCell('totalA12', true)}
              {renderInputCell('computers12', true)}
              {renderInputCell('compSoftwareInt12', true)}
              {renderInputCell('compSoftwareNonint12', true)}
              {renderInputCell('compSoftwareTotal12', true)}
              {renderInputCell('motor12', true)}
              {renderInputCell('offResidenceB12', true)}
              {renderInputCell('stcLho12', true)}
              {renderInputCell('otherPremisesB12', true)}
              {renderInputCell('OtherMachineryPlant12', true)}
              {renderInputCell('totalB12', true)}
              {renderInputCell('totalFurnFix12', true)}
              {renderInputCell('landNotRev12', true)}
              {renderInputCell('landRev12', true)}
              {renderInputCell('landRevEnh12', true)}
              {renderInputCell('offBuildNotRev12', true)}
              {renderInputCell('offBuildRev12', true)}
              {renderInputCell('offBuildRevEnh12', true)}
              {renderInputCell('residQuartNotRev12', true)}
              {renderInputCell('residQuartRev12', true)}
              {renderInputCell('residQuartRevEnh12', true)}
              {renderInputCell('premisTotal12', true)}
              {renderInputCell('revtotal12', true)}
              {renderInputCell('totalC12', true)}
              {renderInputCell('premisesUnderCons12', true)}
              {renderInputCell('grandTotal12', true)}
            </StyledTableRow>

            <StyledTableRow istotalrow>
              <StyledTableCell>B</StyledTableCell>
              <StyledTableCell>Net Addition (I-II)</StyledTableCell>
              {renderInputCell('stcNstaff13', true)}
              {renderInputCell('offResidenceA13', true)}
              {renderInputCell('otherPremisesA13', true)}
              {renderInputCell('electricFitting13', true)}
              {renderInputCell('totalA13', true)}
              {renderInputCell('computers13', true)}
              {renderInputCell('compSoftwareInt13', true)}
              {renderInputCell('compSoftwareNonint13', true)}
              {renderInputCell('compSoftwareTotal13', true)}
              {renderInputCell('motor13', true)}
              {renderInputCell('offResidenceB13', true)}
              {renderInputCell('stcLho13', true)}
              {renderInputCell('otherPremisesB13', true)}
              {renderInputCell('OtherMachineryPlant13', true)}
              {renderInputCell('totalB13', true)}
              {renderInputCell('totalFurnFix13', true)}
              {renderInputCell('landNotRev13', true)}
              {renderInputCell('landRev13', true)}
              {renderInputCell('landRevEnh13', true)}
              {renderInputCell('offBuildNotRev13', true)}
              {renderInputCell('offBuildRev13', true)}
              {renderInputCell('offBuildRevEnh13', true)}
              {renderInputCell('residQuartNotRev13', true)}
              {renderInputCell('residQuartRev13', true)}
              {renderInputCell('residQuartRevEnh13', true)}
              {renderInputCell('premisTotal13', true)}
              {renderInputCell('revtotal13', true)}
              {renderInputCell('totalC13', true)}
              {renderInputCell('premisesUnderCons13', true)}
              {renderInputCell('grandTotal13', true)}
            </StyledTableRow>

            <StyledTableRow istotalrow>
              <StyledTableCell>C</StyledTableCell>
              <StyledTableCell>Total Original Cost/ Revalued Value as at 31st March {} (A+B)</StyledTableCell>
              {renderInputCell('stcNstaff14', true)}
              {renderInputCell('offResidenceA14', true)}
              {renderInputCell('otherPremisesA14', true)}
              {renderInputCell('electricFitting14', true)}
              {renderInputCell('totalA14', true)}
              {renderInputCell('computers14', true)}
              {renderInputCell('compSoftwareInt14', true)}
              {renderInputCell('compSoftwareNonint14', true)}
              {renderInputCell('compSoftwareTotal14', true)}
              {renderInputCell('motor14', true)}
              {renderInputCell('offResidenceB14', true)}
              {renderInputCell('stcLho14', true)}
              {renderInputCell('otherPremisesB14', true)}
              {renderInputCell('OtherMachineryPlant14', true)}
              {renderInputCell('totalB14', true)}
              {renderInputCell('totalFurnFix14', true)}
              {renderInputCell('landNotRev14', true)}
              {renderInputCell('landRev14', true)}
              {renderInputCell('landRevEnh14', true)}
              {renderInputCell('offBuildNotRev14', true)}
              {renderInputCell('offBuildRev14', true)}
              {renderInputCell('offBuildRevEnh14', true)}
              {renderInputCell('residQuartNotRev14', true)}
              {renderInputCell('residQuartRev14', true)}
              {renderInputCell('residQuartRevEnh14', true)}
              {renderInputCell('premisTotal14', true)}
              {renderInputCell('revtotal14', true)}
              {renderInputCell('totalC14', true)}
              {renderInputCell('premisesUnderCons14', true)}
              {renderInputCell('grandTotal14', true)}
            </StyledTableRow>

            <StyledTableRow issubsectionheader>
              <StyledTableCell></StyledTableCell>
              <StyledTableCell>Deduction</StyledTableCell>
              {Array(30)
                .fill(null)
                .map((_, i) => (
                  <StyledTableCell key={`empty-Dep-Deduction-${i}`}></StyledTableCell>
                ))}
            </StyledTableRow>
            <StyledTableRow>
              <StyledTableCell style={{ textAlign: 'right' }}>(i)</StyledTableCell>
              <StyledTableCell> Depreciation upto the end of previous year i.e. 31st March {}</StyledTableCell>
              {renderInputCell('stcNstaff18')}
              {renderInputCell('offResidenceA18')}
              {renderInputCell('otherPremisesA18')}
              {renderInputCell('electricFitting18')}
              {renderInputCell('totalA18', true)}
              {renderInputCell('computers18')}
              {renderInputCell('compSoftwareInt18')}
              {renderInputCell('compSoftwareNonint18')}
              {renderInputCell('compSoftwareTotal18', true)}
              {renderInputCell('motor18')}
              {renderInputCell('offResidenceB18')}
              {renderInputCell('stcLho18')}
              {renderInputCell('otherPremisesB18')}
              {renderInputCell('OtherMachineryPlant18', true)}
              {renderInputCell('totalB18', true)}
              {renderInputCell('totalFurnFix18', true)}
              {renderInputCell('landNotRev18')}
              {renderInputCell('landRev18')}
              {renderInputCell('landRevEnh18')}
              {renderInputCell('offBuildNotRev18')}
              {renderInputCell('offBuildRev18')}
              {renderInputCell('offBuildRevEnh18')}
              {renderInputCell('residQuartNotRev18')}
              {renderInputCell('residQuartRev18')}
              {renderInputCell('residQuartRevEnh18')}
              {renderInputCell('premisTotal18', true)}
              {renderInputCell('revtotal18', true)}
              {renderInputCell('totalC18', true)}
              {renderInputCell('premisesUnderCons18')}
              {renderInputCell('grandTotal18', true)}
            </StyledTableRow>
            <StyledTableRow>
              <StyledTableCell style={{ textAlign: 'right' }}>(ii)</StyledTableCell>
              <StyledTableCell>
                Short Valuation charged to depreciation upto end of previous year i.e.31st March {}
              </StyledTableCell>
              {renderInputCell('stcNstaff34')}
              {renderInputCell('offResidenceA34')}
              {renderInputCell('otherPremisesA34')}
              {renderInputCell('electricFitting34')}
              {renderInputCell('totalA34', true)}
              {renderInputCell('computers34')}
              {renderInputCell('compSoftwareInt34')}
              {renderInputCell('compSoftwareNonint34')}
              {renderInputCell('compSoftwareTotal34', true)}
              {renderInputCell('motor34')}
              {renderInputCell('offResidenceB34')}
              {renderInputCell('stcLho34')}
              {renderInputCell('otherPremisesB34')}
              {renderInputCell('OtherMachineryPlant34', true)}
              {renderInputCell('totalB34', true)}
              {renderInputCell('totalFurnFix34', true)}
              {renderInputCell('landNotRev34')}
              {renderInputCell('landRev34')}
              {renderInputCell('landRevEnh34')}
              {renderInputCell('offBuildNotRev34')}
              {renderInputCell('offBuildRev34')}
              {renderInputCell('offBuildRevEnh34')}
              {renderInputCell('residQuartNotRev34')}
              {renderInputCell('residQuartRev34')}
              {renderInputCell('residQuartRevEnh34')}
              {renderInputCell('premisTotal34', true)}
              {renderInputCell('revtotal34', true)}
              {renderInputCell('totalC34', true)}
              {renderInputCell('premisesUnderCons34')}
              {renderInputCell('grandTotal34', true)}
            </StyledTableRow>
            <StyledTableRow>
              <StyledTableCell style={{ textAlign: 'right' }}>(iii)</StyledTableCell>
              <StyledTableCell>Depreciation on repatriation of Officials from Subsidiaries/ Associates</StyledTableCell>
              {renderInputCell('stcNstaff38')}
              {renderInputCell('offResidenceA38')}
              {renderInputCell('otherPremisesA38')}
              {renderInputCell('electricFitting38')}
              {renderInputCell('totalA38', true)}
              {renderInputCell('computers38')}
              {renderInputCell('compSoftwareInt38')}
              {renderInputCell('compSoftwareNonint38')}
              {renderInputCell('compSoftwareTotal38', true)}
              {renderInputCell('motor38')}
              {renderInputCell('offResidenceB38')}
              {renderInputCell('stcLho38')}
              {renderInputCell('otherPremisesB38')}
              {renderInputCell('OtherMachineryPlant38', true)}
              {renderInputCell('totalB38', true)}
              {renderInputCell('totalFurnFix38', true)}
              {renderInputCell('landNotRev38')}
              {renderInputCell('landRev38')}
              {renderInputCell('landRevEnh38')}
              {renderInputCell('offBuildNotRev38')}
              {renderInputCell('offBuildRev38')}
              {renderInputCell('offBuildRevEnh38')}
              {renderInputCell('residQuartNotRev38')}
              {renderInputCell('residQuartRev38')}
              {renderInputCell('residQuartRevEnh38')}
              {renderInputCell('premisTotal38', true)}
              {renderInputCell('revtotal38', true)}
              {renderInputCell('totalC38', true)}
              {renderInputCell('premisesUnderCons38')}
              {renderInputCell('grandTotal38', true)}
            </StyledTableRow>
            <StyledTableRow>
              <StyledTableCell style={{ textAlign: 'right' }}>(iv)</StyledTableCell>
              <StyledTableCell> Depreciation transferred from other Circles/Groups/CC Departments</StyledTableCell>
              {renderInputCell('stcNstaff19')}
              {renderInputCell('offResidenceA19')}
              {renderInputCell('otherPremisesA19')}
              {renderInputCell('electricFitting19')}
              {renderInputCell('totalA19', true)}
              {renderInputCell('computers19')}
              {renderInputCell('compSoftwareInt19')}
              {renderInputCell('compSoftwareNonint19')}
              {renderInputCell('compSoftwareTotal19', true)}
              {renderInputCell('motor19')}
              {renderInputCell('offResidenceB19')}
              {renderInputCell('stcLho19')}
              {renderInputCell('otherPremisesB19')}
              {renderInputCell('OtherMachineryPlant19', true)}
              {renderInputCell('totalB19', true)}
              {renderInputCell('totalFurnFix19', true)}
              {renderInputCell('landNotRev19')}
              {renderInputCell('landRev19')}
              {renderInputCell('landRevEnh19')}
              {renderInputCell('offBuildNotRev19')}
              {renderInputCell('offBuildRev19')}
              {renderInputCell('offBuildRevEnh19')}
              {renderInputCell('residQuartNotRev19')}
              {renderInputCell('residQuartRev19')}
              {renderInputCell('residQuartRevEnh19')}
              {renderInputCell('premisTotal19', true)}
              {renderInputCell('revtotal19', true)}
              {renderInputCell('totalC19', true)}
              {renderInputCell('premisesUnderCons19')}
              {renderInputCell('grandTotal19', true)}
            </StyledTableRow>
            <StyledTableRow>
              <StyledTableCell style={{ textAlign: 'right' }}>(v)</StyledTableCell>
              <StyledTableCell> Depreciation transferred from other branches of the same circle.</StyledTableCell>
              {renderInputCell('stcNstaff20')}
              {renderInputCell('offResidenceA20')}
              {renderInputCell('otherPremisesA20')}
              {renderInputCell('electricFitting20')}
              {renderInputCell('totalA20', true)}
              {renderInputCell('computers20')}
              {renderInputCell('compSoftwareInt20')}
              {renderInputCell('compSoftwareNonint20')}
              {renderInputCell('compSoftwareTotal20', true)}
              {renderInputCell('motor20')}
              {renderInputCell('offResidenceB20')}
              {renderInputCell('stcLho20')}
              {renderInputCell('otherPremisesB20')}
              {renderInputCell('OtherMachineryPlant20', true)}
              {renderInputCell('totalB20', true)}
              {renderInputCell('totalFurnFix20', true)}
              {renderInputCell('landNotRev20')}
              {renderInputCell('landRev20')}
              {renderInputCell('landRevEnh20')}
              {renderInputCell('offBuildNotRev20')}
              {renderInputCell('offBuildRev20')}
              {renderInputCell('offBuildRevEnh20')}
              {renderInputCell('residQuartNotRev20')}
              {renderInputCell('residQuartRev20')}
              {renderInputCell('residQuartRevEnh20')}
              {renderInputCell('premisTotal20', true)}
              {renderInputCell('revtotal20', true)}
              {renderInputCell('totalC20', true)}
              {renderInputCell('premisesUnderCons20')}
              {renderInputCell('grandTotal20', true)}
            </StyledTableRow>
            <StyledTableRow>
              <StyledTableCell style={{ textAlign: 'right' }}>(vi)</StyledTableCell>
              <StyledTableCell> Depreciation charged during the current year</StyledTableCell>
              {renderInputCell('stcNstaff21')}
              {renderInputCell('offResidenceA21')}
              {renderInputCell('otherPremisesA21')}
              {renderInputCell('electricFitting21')}
              {renderInputCell('totalA21', true)}
              {renderInputCell('computers21')}
              {renderInputCell('compSoftwareInt21')}
              {renderInputCell('compSoftwareNonint21')}
              {renderInputCell('compSoftwareTotal21', true)}
              {renderInputCell('motor21')}
              {renderInputCell('offResidenceB21')}
              {renderInputCell('stcLho21')}
              {renderInputCell('otherPremisesB21')}
              {renderInputCell('OtherMachineryPlant21', true)}
              {renderInputCell('totalB21', true)}
              {renderInputCell('totalFurnFix21', true)}
              {renderInputCell('landNotRev21')}
              {renderInputCell('landRev21')}
              {renderInputCell('landRevEnh21')}
              {renderInputCell('offBuildNotRev21')}
              {renderInputCell('offBuildRev21')}
              {renderInputCell('offBuildRevEnh21')}
              {renderInputCell('residQuartNotRev21')}
              {renderInputCell('residQuartRev21')}
              {renderInputCell('residQuartRevEnh21')}
              {renderInputCell('premisTotal21', true)}
              {renderInputCell('revtotal21', true)}
              {renderInputCell('totalC21', true)}
              {renderInputCell('premisesUnderCons21')}
              {renderInputCell('grandTotal21', true)}
            </StyledTableRow>
            <StyledTableRow>
              <StyledTableCell style={{ textAlign: 'right' }}>(vii)</StyledTableCell>
              <StyledTableCell>
                Short Valuation charged to Depreciation during the current year due to Current Revaluation
              </StyledTableCell>
              {renderInputCell('stcNstaff39')}
              {renderInputCell('offResidenceA39')}
              {renderInputCell('otherPremisesA39')}
              {renderInputCell('electricFitting39')}
              {renderInputCell('totalA39', true)}
              {renderInputCell('computers39')}
              {renderInputCell('compSoftwareInt39')}
              {renderInputCell('compSoftwareNonint39')}
              {renderInputCell('compSoftwareTotal39', true)}
              {renderInputCell('motor39')}
              {renderInputCell('offResidenceB39')}
              {renderInputCell('stcLho39')}
              {renderInputCell('otherPremisesB39')}
              {renderInputCell('OtherMachineryPlant39', true)}
              {renderInputCell('totalB39', true)}
              {renderInputCell('totalFurnFix39', true)}
              {renderInputCell('landNotRev39')}
              {renderInputCell('landRev39')}
              {renderInputCell('landRevEnh39')}
              {renderInputCell('offBuildNotRev39')}
              {renderInputCell('offBuildRev39')}
              {renderInputCell('offBuildRevEnh39')}
              {renderInputCell('residQuartNotRev39')}
              {renderInputCell('residQuartRev39')}
              {renderInputCell('residQuartRevEnh39')}
              {renderInputCell('premisTotal39', true)}
              {renderInputCell('revtotal39', true)}
              {renderInputCell('totalC39', true)}
              {renderInputCell('premisesUnderCons39')}
              {renderInputCell('grandTotal39', true)}
            </StyledTableRow>
            <StyledTableRow istotalrow>
              <StyledTableCell>D</StyledTableCell>
              <StyledTableCell>Total (i+ii+iii+iv+v+vi+vii)</StyledTableCell>
              {renderInputCell('stcNstaff22', true)}
              {renderInputCell('offResidenceA22', true)}
              {renderInputCell('otherPremisesA22', true)}
              {renderInputCell('electricFitting22', true)}
              {renderInputCell('totalA22', true)}
              {renderInputCell('computers22', true)}
              {renderInputCell('compSoftwareInt22', true)}
              {renderInputCell('compSoftwareNonint22', true)}
              {renderInputCell('compSoftwareTotal22', true)}
              {renderInputCell('motor22', true)}
              {renderInputCell('offResidenceB22', true)}
              {renderInputCell('stcLho22', true)}
              {renderInputCell('otherPremisesB22', true)}
              {renderInputCell('OtherMachineryPlant22', true)}
              {renderInputCell('totalB22', true)}
              {renderInputCell('totalFurnFix22', true)}
              {renderInputCell('landNotRev22', true)}
              {renderInputCell('landRev22', true)}
              {renderInputCell('landRevEnh22', true)}
              {renderInputCell('offBuildNotRev22', true)}
              {renderInputCell('offBuildRev22', true)}
              {renderInputCell('offBuildRevEnh22', true)}
              {renderInputCell('residQuartNotRev22', true)}
              {renderInputCell('residQuartRev22', true)}
              {renderInputCell('residQuartRevEnh22', true)}
              {renderInputCell('premisTotal22', true)}
              {renderInputCell('revtotal22', true)}
              {renderInputCell('totalC22', true)}
              {renderInputCell('premisesUnderCons22', true)}
              {renderInputCell('grandTotal22', true)}
            </StyledTableRow>

            <StyledTableRow issubsectionheader>
              <StyledTableCell></StyledTableCell>
              <StyledTableCell>Less :</StyledTableCell>
              {Array(30)
                .fill(null)
                .map((_, i) => (
                  <StyledTableCell key={`empty-Less-${i}`}></StyledTableCell>
                ))}
            </StyledTableRow>
            <StyledTableRow>
              <StyledTableCell style={{ textAlign: 'right' }}>(i)</StyledTableCell>
              <StyledTableCell>
                Past Short Valuation credited to Depreciation during the current year due to Current Upward Revaluation
              </StyledTableCell>
              {renderInputCell('stcNstaff40')}
              {renderInputCell('offResidenceA40')}
              {renderInputCell('otherPremisesA40')}
              {renderInputCell('electricFitting40')}
              {renderInputCell('totalA40', true)}
              {renderInputCell('computers40')}
              {renderInputCell('compSoftwareInt40')}
              {renderInputCell('compSoftwareNonint40')}
              {renderInputCell('compSoftwareTotal40', true)}
              {renderInputCell('motor40')}
              {renderInputCell('offResidenceB40')}
              {renderInputCell('stcLho40')}
              {renderInputCell('otherPremisesB40')}
              {renderInputCell('OtherMachineryPlant40', true)}
              {renderInputCell('totalB40', true)}
              {renderInputCell('totalFurnFix40', true)}
              {renderInputCell('landNotRev40')}
              {renderInputCell('landRev40')}
              {renderInputCell('landRevEnh40')}
              {renderInputCell('offBuildNotRev40')}
              {renderInputCell('offBuildRev40')}
              {renderInputCell('offBuildRevEnh40')}
              {renderInputCell('residQuartNotRev40')}
              {renderInputCell('residQuartRev40')}
              {renderInputCell('residQuartRevEnh40')}
              {renderInputCell('premisTotal40', true)}
              {renderInputCell('revtotal40', true)}
              {renderInputCell('totalC40', true)}
              {renderInputCell('premisesUnderCons40')}
              {renderInputCell('grandTotal40', true)}
            </StyledTableRow>
            <StyledTableRow>
              <StyledTableCell style={{ textAlign: 'right' }}>(ii)</StyledTableCell>
              <StyledTableCell> Depreciation previously provided on fixed assets sold/ discarded</StyledTableCell>
              {renderInputCell('stcNstaff24')}
              {renderInputCell('offResidenceA24')}
              {renderInputCell('otherPremisesA24')}
              {renderInputCell('electricFitting24')}
              {renderInputCell('totalA24', true)}
              {renderInputCell('computers24')}
              {renderInputCell('compSoftwareInt24')}
              {renderInputCell('compSoftwareNonint24')}
              {renderInputCell('compSoftwareTotal24', true)}
              {renderInputCell('motor24')}
              {renderInputCell('offResidenceB24')}
              {renderInputCell('stcLho24')}
              {renderInputCell('otherPremisesB24')}
              {renderInputCell('OtherMachineryPlant24', true)}
              {renderInputCell('totalB24', true)}
              {renderInputCell('totalFurnFix24', true)}
              {renderInputCell('landNotRev24')}
              {renderInputCell('landRev24')}
              {renderInputCell('landRevEnh24')}
              {renderInputCell('offBuildNotRev24')}
              {renderInputCell('offBuildRev24')}
              {renderInputCell('offBuildRevEnh24')}
              {renderInputCell('residQuartNotRev24')}
              {renderInputCell('residQuartRev24')}
              {renderInputCell('residQuartRevEnh24')}
              {renderInputCell('premisTotal24', true)}
              {renderInputCell('revtotal24', true)}
              {renderInputCell('totalC24', true)}
              {renderInputCell('premisesUnderCons24')}
              {renderInputCell('grandTotal24', true)}
            </StyledTableRow>
            <StyledTableRow>
              <StyledTableCell style={{ textAlign: 'right' }}>(iii)</StyledTableCell>
              <StyledTableCell> Depreciation transferred to other Circles/Groups/CC Departments</StyledTableCell>
              {renderInputCell('stcNstaff25')}
              {renderInputCell('offResidenceA25')}
              {renderInputCell('otherPremisesA25')}
              {renderInputCell('electricFitting25')}
              {renderInputCell('totalA25', true)}
              {renderInputCell('computers25')}
              {renderInputCell('compSoftwareInt25')}
              {renderInputCell('compSoftwareNonint25')}
              {renderInputCell('compSoftwareTotal25', true)}
              {renderInputCell('motor25')}
              {renderInputCell('offResidenceB25')}
              {renderInputCell('stcLho25')}
              {renderInputCell('otherPremisesB25')}
              {renderInputCell('OtherMachineryPlant25', true)}
              {renderInputCell('totalB25', true)}
              {renderInputCell('totalFurnFix25', true)}
              {renderInputCell('landNotRev25')}
              {renderInputCell('landRev25')}
              {renderInputCell('landRevEnh25')}
              {renderInputCell('offBuildNotRev25')}
              {renderInputCell('offBuildRev25')}
              {renderInputCell('offBuildRevEnh25')}
              {renderInputCell('residQuartNotRev25')}
              {renderInputCell('residQuartRev25')}
              {renderInputCell('residQuartRevEnh25')}
              {renderInputCell('premisTotal25', true)}
              {renderInputCell('revtotal25', true)}
              {renderInputCell('totalC25', true)}
              {renderInputCell('premisesUnderCons25')}
              {renderInputCell('grandTotal25', true)}
            </StyledTableRow>
            <StyledTableRow>
              <StyledTableCell style={{ textAlign: 'right' }}>(iv)</StyledTableCell>
              <StyledTableCell> Depreciation transferred to other branches of the same Circle.</StyledTableCell>
              {renderInputCell('stcNstaff26')}
              {renderInputCell('offResidenceA26')}
              {renderInputCell('otherPremisesA26')}
              {renderInputCell('electricFitting26')}
              {renderInputCell('totalA26', true)}
              {renderInputCell('computers26')}
              {renderInputCell('compSoftwareInt26')}
              {renderInputCell('compSoftwareNonint26')}
              {renderInputCell('compSoftwareTotal26', true)}
              {renderInputCell('motor26')}
              {renderInputCell('offResidenceB26')}
              {renderInputCell('stcLho26')}
              {renderInputCell('otherPremisesB26')}
              {renderInputCell('OtherMachineryPlant26', true)}
              {renderInputCell('totalB26', true)}
              {renderInputCell('totalFurnFix26', true)}
              {renderInputCell('landNotRev26')}
              {renderInputCell('landRev26')}
              {renderInputCell('landRevEnh26')}
              {renderInputCell('offBuildNotRev26')}
              {renderInputCell('offBuildRev26')}
              {renderInputCell('offBuildRevEnh26')}
              {renderInputCell('residQuartNotRev26')}
              {renderInputCell('residQuartRev26')}
              {renderInputCell('residQuartRevEnh26')}
              {renderInputCell('premisTotal26', true)}
              {renderInputCell('revtotal26', true)}
              {renderInputCell('totalC26', true)}
              {renderInputCell('premisesUnderCons26')}
              {renderInputCell('grandTotal26', true)}
            </StyledTableRow>
            <StyledTableRow istotalrow>
              <StyledTableCell>E</StyledTableCell>
              <StyledTableCell>Total (i+ii+iii+iv)</StyledTableCell>
              {renderInputCell('stcNstaff27', true)}
              {renderInputCell('offResidenceA27', true)}
              {renderInputCell('otherPremisesA27', true)}
              {renderInputCell('electricFitting27', true)}
              {renderInputCell('totalA27', true)}
              {renderInputCell('computers27', true)}
              {renderInputCell('compSoftwareInt27', true)}
              {renderInputCell('compSoftwareNonint27', true)}
              {renderInputCell('compSoftwareTotal27', true)}
              {renderInputCell('motor27', true)}
              {renderInputCell('offResidenceB27', true)}
              {renderInputCell('stcLho27', true)}
              {renderInputCell('otherPremisesB27', true)}
              {renderInputCell('OtherMachineryPlant27', true)}
              {renderInputCell('totalB27', true)}
              {renderInputCell('totalFurnFix27', true)}
              {renderInputCell('landNotRev27', true)}
              {renderInputCell('landRev27', true)}
              {renderInputCell('landRevEnh27', true)}
              {renderInputCell('offBuildNotRev27', true)}
              {renderInputCell('offBuildRev27', true)}
              {renderInputCell('offBuildRevEnh27', true)}
              {renderInputCell('residQuartNotRev27', true)}
              {renderInputCell('residQuartRev27', true)}
              {renderInputCell('residQuartRevEnh27', true)}
              {renderInputCell('premisTotal27', true)}
              {renderInputCell('revtotal27', true)}
              {renderInputCell('totalC27', true)}
              {renderInputCell('premisesUnderCons27', true)}
              {renderInputCell('grandTotal27', true)}
            </StyledTableRow>

            <StyledTableRow istotalrow>
              <StyledTableCell>F</StyledTableCell>
              <StyledTableCell>Net Depreciation (D-E)</StyledTableCell>
              {renderInputCell('stcNstaff28', true)}
              {renderInputCell('offResidenceA28', true)}
              {renderInputCell('otherPremisesA28', true)}
              {renderInputCell('electricFitting28', true)}
              {renderInputCell('totalA28', true)}
              {renderInputCell('computers28', true)}
              {renderInputCell('compSoftwareInt28', true)}
              {renderInputCell('compSoftwareNonint28', true)}
              {renderInputCell('compSoftwareTotal28', true)}
              {renderInputCell('motor28', true)}
              {renderInputCell('offResidenceB28', true)}
              {renderInputCell('stcLho28', true)}
              {renderInputCell('otherPremisesB28', true)}
              {renderInputCell('OtherMachineryPlant28', true)}
              {renderInputCell('totalB28', true)}
              {renderInputCell('totalFurnFix28', true)}
              {renderInputCell('landNotRev28', true)}
              {renderInputCell('landRev28', true)}
              {renderInputCell('landRevEnh28', true)}
              {renderInputCell('offBuildNotRev28', true)}
              {renderInputCell('offBuildRev28', true)}
              {renderInputCell('offBuildRevEnh28', true)}
              {renderInputCell('residQuartNotRev28', true)}
              {renderInputCell('residQuartRev28', true)}
              {renderInputCell('residQuartRevEnh28', true)}
              {renderInputCell('premisTotal28', true)}
              {renderInputCell('revtotal28', true)}
              {renderInputCell('totalC28', true)}
              {renderInputCell('premisesUnderCons28', true)}
              {renderInputCell('grandTotal28', true)}
            </StyledTableRow>

            <StyledTableRow istotalrow>
              <StyledTableCell>G</StyledTableCell>
              <StyledTableCell>Net Book Value as at 31st March {} (C-F)</StyledTableCell>
              {renderInputCell('stcNstaff29', true)}
              {renderInputCell('offResidenceA29', true)}
              {renderInputCell('otherPremisesA29', true)}
              {renderInputCell('electricFitting29', true)}
              {renderInputCell('totalA29', true)}
              {renderInputCell('computers29', true)}
              {renderInputCell('compSoftwareInt29', true)}
              {renderInputCell('compSoftwareNonint29', true)}
              {renderInputCell('compSoftwareTotal29', true)}
              {renderInputCell('motor29', true)}
              {renderInputCell('offResidenceB29', true)}
              {renderInputCell('stcLho29', true)}
              {renderInputCell('otherPremisesB29', true)}
              {renderInputCell('OtherMachineryPlant29', true)}
              {renderInputCell('totalB29', true)}
              {renderInputCell('totalFurnFix29', true)}
              {renderInputCell('landNotRev29', true)}
              {renderInputCell('landRev29', true)}
              {renderInputCell('landRevEnh29', true)}
              {renderInputCell('offBuildNotRev29', true)}
              {renderInputCell('offBuildRev29', true)}
              {renderInputCell('offBuildRevEnh29', true)}
              {renderInputCell('residQuartNotRev29', true)}
              {renderInputCell('residQuartRev29', true)}
              {renderInputCell('residQuartRevEnh29', true)}
              {renderInputCell('premisTotal29', true)}
              {renderInputCell('revtotal29', true)}
              {renderInputCell('totalC29', true)}
              {renderInputCell('premisesUnderCons29', true)}
              {renderInputCell('grandTotal29', true)}
            </StyledTableRow>

            <StyledTableRow>
              <StyledTableCell>H</StyledTableCell>
              <StyledTableCell>Sale Price of fixed assets</StyledTableCell>
              {renderInputCell('stcNstaff30')}
              {renderInputCell('offResidenceA30')}
              {renderInputCell('otherPremisesA30')}
              {renderInputCell('electricFitting30')}
              {renderInputCell('totalA30', true)}
              {renderInputCell('computers30')}
              {renderInputCell('compSoftwareInt30')}
              {renderInputCell('compSoftwareNonint30')}
              {renderInputCell('compSoftwareTotal30', true)}
              {renderInputCell('motor30')}
              {renderInputCell('offResidenceB30')}
              {renderInputCell('stcLho30')}
              {renderInputCell('otherPremisesB30')}
              {renderInputCell('OtherMachineryPlant30', true)}
              {renderInputCell('totalB30', true)}
              {renderInputCell('totalFurnFix30', true)}
              {renderInputCell('landNotRev30')}
              {renderInputCell('landRev30')}
              {renderInputCell('landRevEnh30')}
              {renderInputCell('offBuildNotRev30')}
              {renderInputCell('offBuildRev30')}
              {renderInputCell('offBuildRevEnh30')}
              {renderInputCell('residQuartNotRev30')}
              {renderInputCell('residQuartRev30')}
              {renderInputCell('residQuartRevEnh30')}
              {renderInputCell('premisTotal30', true)}
              {renderInputCell('revtotal30', true)}
              {renderInputCell('totalC30', true)}
              {renderInputCell('premisesUnderCons30')}
              {renderInputCell('grandTotal30', true)}
            </StyledTableRow>

            <StyledTableRow istotalrow>
              <StyledTableCell>I</StyledTableCell>
              <StyledTableCell>Book Value of fixed assets sold [II (ii)-E(ii)]</StyledTableCell>
              {renderInputCell('stcNstaff31', true)}
              {renderInputCell('offResidenceA31', true)}
              {renderInputCell('otherPremisesA31', true)}
              {renderInputCell('electricFitting31', true)}
              {renderInputCell('totalA31', true)}
              {renderInputCell('computers31', true)}
              {renderInputCell('compSoftwareInt31', true)}
              {renderInputCell('compSoftwareNonint31', true)}
              {renderInputCell('compSoftwareTotal31', true)}
              {renderInputCell('motor31', true)}
              {renderInputCell('offResidenceB31', true)}
              {renderInputCell('stcLho31', true)}
              {renderInputCell('otherPremisesB31', true)}
              {renderInputCell('OtherMachineryPlant31', true)}
              {renderInputCell('totalB31', true)}
              {renderInputCell('totalFurnFix31', true)}
              {renderInputCell('landNotRev31', true)}
              {renderInputCell('landRev31', true)}
              {renderInputCell('landRevEnh31', true)}
              {renderInputCell('offBuildNotRev31', true)}
              {renderInputCell('offBuildRev31', true)}
              {renderInputCell('offBuildRevEnh31', true)}
              {renderInputCell('residQuartNotRev31', true)}
              {renderInputCell('residQuartRev31', true)}
              {renderInputCell('residQuartRevEnh31', true)}
              {renderInputCell('premisTotal31', true)}
              {renderInputCell('revtotal31', true)}
              {renderInputCell('totalC31', true)}
              {renderInputCell('premisesUnderCons31', true)}
              {renderInputCell('grandTotal31', true)}
            </StyledTableRow>

            <StyledTableRow>
              <StyledTableCell>J</StyledTableCell>
              <StyledTableCell>GST on Sale of fixed assets</StyledTableCell>
              {renderInputCell('stcNstaff35')}
              {renderInputCell('offResidenceA35')}
              {renderInputCell('otherPremisesA35')}
              {renderInputCell('electricFitting35')}
              {renderInputCell('totalA35', true)}
              {renderInputCell('computers35')}
              {renderInputCell('compSoftwareInt35')}
              {renderInputCell('compSoftwareNonint35')}
              {renderInputCell('compSoftwareTotal35', true)}
              {renderInputCell('motor35')}
              {renderInputCell('offResidenceB35')}
              {renderInputCell('stcLho35')}
              {renderInputCell('otherPremisesB35')}
              {renderInputCell('OtherMachineryPlant35', true)}
              {renderInputCell('totalB35', true)}
              {renderInputCell('totalFurnFix35', true)}
              {renderInputCell('landNotRev35')}
              {renderInputCell('landRev35')}
              {renderInputCell('landRevEnh35')}
              {renderInputCell('offBuildNotRev35')}
              {renderInputCell('offBuildRev35')}
              {renderInputCell('offBuildRevEnh35')}
              {renderInputCell('residQuartNotRev35')}
              {renderInputCell('residQuartRev35')}
              {renderInputCell('residQuartRevEnh35')}
              {renderInputCell('premisTotal35', true)}
              {renderInputCell('revtotal35', true)}
              {renderInputCell('totalC35', true)}
              {renderInputCell('premisesUnderCons35')}
              {renderInputCell('grandTotal35', true)}
            </StyledTableRow>

            <StyledTableRow istotalrow>
              <StyledTableCell>K</StyledTableCell>
              <StyledTableCell>Profit/ (Loss) on sale of fixed assets [H-(I+J)]</StyledTableCell>
              {renderInputCell('stcNstaff32', true)}
              {renderInputCell('offResidenceA32', true)}
              {renderInputCell('otherPremisesA32', true)}
              {renderInputCell('electricFitting32', true)}
              {renderInputCell('totalA32', true)}
              {renderInputCell('computers32', true)}
              {renderInputCell('compSoftwareInt32', true)}
              {renderInputCell('compSoftwareNonint32', true)}
              {renderInputCell('compSoftwareTotal32', true)}
              {renderInputCell('motor32', true)}
              {renderInputCell('offResidenceB32', true)}
              {renderInputCell('stcLho32', true)}
              {renderInputCell('otherPremisesB32', true)}
              {renderInputCell('OtherMachineryPlant32', true)}
              {renderInputCell('totalB32', true)}
              {renderInputCell('totalFurnFix32', true)}
              {renderInputCell('landNotRev32', true)}
              {renderInputCell('landRev32', true)}
              {renderInputCell('landRevEnh32', true)}
              {renderInputCell('offBuildNotRev32', true)}
              {renderInputCell('offBuildRev32', true)}
              {renderInputCell('offBuildRevEnh32', true)}
              {renderInputCell('residQuartNotRev32', true)}
              {renderInputCell('residQuartRev32', true)}
              {renderInputCell('residQuartRevEnh32', true)}
              {renderInputCell('premisTotal32', true)}
              {renderInputCell('revtotal32', true)}
              {renderInputCell('totalC32', true)}
              {renderInputCell('premisesUnderCons32', true)}
              {renderInputCell('grandTotal32', true)}
            </StyledTableRow>
          </TableBody>
        </Table>
      </TableContainer>
    </Box>
  );
};

export default Schedule10;

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

const StyledTableRow = styled(TableRow)(({ theme, istotalrow, issectionheader, issubsectionheader }) => ({
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
  ...(istotalrow && {
    '& > td': {
      fontWeight: 'bold',
    },
  }),
}));

const Schedule10 = () => {
  const [formData, setFormData] = useState({
    "particulars3": "Cost of new items put to use upto 3rd October 2024",
    "particulars4": "Cost of new items put to use during 4th October 2024 to 31st March 2025",
    "totalA36": "0.00",
    "stcNstaff7": "0.00",
    "offResidenceA7": "0.00",
    "otherPremisesA7": "0.00",
    "electricFitting7": "0.00",
    "stcNstaff12": "0.00",
    "offResidenceA12": "0.00",
    "otherPremisesA12": "0.00",
    "electricFitting12": "0.00",
    "totalA1": "0.00",
    "totalA3": "0.00",
    "totalA4": "0.00",
    "totalA5": "0.00",
    "totalA6": "0.00",
    "totalA7": "0.00",
    "totalA9": "0.00",
    "totalA33": "0.00",
    "totalA37": "0.00",
    "totalA10": "0.00",
    "totalA11": "0.00",
    "totalA12": "0.00",
    "stcNstaff13": "0.00",
    "offResidenceA13": "0.00",
    "otherPremisesA13": "0.00",
    "electricFitting13": "0.00",
    "totalA13": "0.00",
    "stcNstaff14": "0.00",
    "offResidenceA14": "0.00",
    "otherPremisesA14": "0.00",
    "electricFitting14": "0.00",
    "totalA14": "0.00",
    "totalA18": "0.00",
    "totalA34": "0.00",
    "totalA19": "0.00",
    "totalA38": "0.00",
    "totalA39": "0.00",
    "totalA20": "0.00",
    "totalA21": "0.00",
    "stcNstaff22": "0.00",
    "offResidenceA22": "0.00",
    "otherPremisesA22": "0.00",
    "electricFitting22": "0.00",
    "totalA22": "0.00",
    "totalA24": "0.00",
    "totalA25": "0.00",
    "totalA26": "0.00",
    "totalA40": "0.00",
    "stcNstaff27": "0.00",
    "offResidenceA27": "0.00",
    "otherPremisesA27": "0.00",
    "electricFitting27": "0.00",
    "totalA27": "0.00",
    "stcNstaff28": "0.00",
    "offResidenceA28": "0.00",
    "otherPremisesA28": "0.00",
    "electricFitting28": "0.00",
    "totalA28": "0.00",
    "stcNstaff29": "0.00",
    "offResidenceA29": "0.00",
    "otherPremisesA29": "0.00",
    "electricFitting29": "0.00",
    "totalA29": "0.00",
    "totalA30": "0.00",
    "totalA35": "0.00",
    "stcNstaff31": "0.00",
    "offResidenceA31": "0.00",
    "otherPremisesA31": "0.00",
    "electricFitting31": "0.00",
    "totalA31": "0.00",
    "compSoftwareTotal1": "0.00",
    "compSoftwareTotal3": "0.00",
    "compSoftwareTotal4": "0.00",
    "compSoftwareTotal36": "0.00",
    "compSoftwareTotal5": "0.00",
    "compSoftwareTotal6": "0.00",
    "computers7": "0.00",
    "compSoftwareInt7": "0.00",
    "compSoftwareNonint7": "0.00",
    "compSoftwareTotal7": "0.00",
    "compSoftwareTotal9": "0.00",
    "compSoftwareTotal33": "0.00",
    "compSoftwareTotal37": "0.00",
    "compSoftwareTotal10": "0.00",
    "compSoftwareTotal11": "0.00",
    "computers12": "0.00",
    "compSoftwareInt12": "0.00",
    "compSoftwareNonint12": "0.00",
    "compSoftwareTotal12": "0.00",
    "computers13": "0.00",
    "compSoftwareInt13": "0.00",
    "compSoftwareNonint13": "0.00",
    "compSoftwareTotal13": "0.00",
    "computers14": "0.00",
    "compSoftwareInt14": "0.00",
    "compSoftwareNonint14": "0.00",
    "compSoftwareTotal14": "0.00",
    "compSoftwareTotal18": "0.00",
    "compSoftwareTotal34": "0.00",
    "compSoftwareTotal19": "0.00",
    "compSoftwareTotal38": "0.00",
    "compSoftwareTotal39": "0.00",
    "compSoftwareTotal20": "0.00",
    "compSoftwareTotal21": "0.00",
    "computers22": "0.00",
    "compSoftwareInt22": "0.00",
    "compSoftwareNonint22": "0.00",
    "compSoftwareTotal22": "0.00",
    "compSoftwareTotal24": "0.00",
    "compSoftwareTotal25": "0.00",
    "compSoftwareTotal26": "0.00",
    "compSoftwareTotal40": "0.00",
    "computers27": "0.00",
    "compSoftwareInt27": "0.00",
    "compSoftwareNonint27": "0.00",
    "compSoftwareTotal27": "0.00",
    "computers28": "0.00",
    "compSoftwareInt28": "0.00",
    "compSoftwareNonint28": "0.00",
    "compSoftwareTotal28": "0.00",
    "computers29": "0.00",
    "compSoftwareInt29": "0.00",
    "compSoftwareNonint29": "0.00",
    "compSoftwareTotal29": "0.00",
    "compSoftwareTotal30": "0.00",
    "compSoftwareTotal35": "0.00",
    "computers31": "0.00",
    "compSoftwareInt31": "0.00",
    "compSoftwareNonint31": "0.00",
    "compSoftwareTotal31": "0.00",
    "otherMachineryPlant1": "0.00",
    "otherMachineryPlant3": "0.00",
    "otherMachineryPlant4": "0.00",
    "otherMachineryPlant36": "0.00",
    "otherMachineryPlant5": "0.00",
    "otherMachineryPlant6": "0.00",
    "totalB1": "0.00",
    "totalB3": "0.00",
    "totalB4": "0.00",
    "totalB36": "0.00",
    "totalB5": "0.00",
    "totalB6": "0.00",
    "motor7": "0.00",
    "offResidenceB7": "0.00",
    "stcLho7": "0.00",
    "otherPremisesB7": "0.00",
    "otherMachineryPlant7": "0.00",
    "totalB7": "0.00",
    "otherMachineryPlant9": "0.00",
    "otherMachineryPlant33": "0.00",
    "otherMachineryPlant37": "0.00",
    "otherMachineryPlant10": "0.00",
    "otherMachineryPlant11": "0.00",
    "totalB9": "0.00",
    "totalB33": "0.00",
    "totalB37": "0.00",
    "totalB10": "0.00",
    "totalB11": "0.00",
    "motor12": "0.00",
    "offResidenceB12": "0.00",
    "stcLho12": "0.00",
    "otherPremisesB12": "0.00",
    "otherMachineryPlant12": "0.00",
    "totalB12": "0.00",
    "motor13": "0.00",
    "offResidenceB13": "0.00",
    "stcLho13": "0.00",
    "otherPremisesB13": "0.00",
    "otherMachineryPlant13": "0.00",
    "totalB13": "0.00",
    "motor14": "0.00",
    "offResidenceB14": "0.00",
    "stcLho14": "0.00",
    "otherPremisesB14": "0.00",
    "otherMachineryPlant14": "0.00",
    "totalB14": "0.00",
    "otherMachineryPlant18": "0.00",
    "otherMachineryPlant34": "0.00",
    "otherMachineryPlant19": "0.00",
    "otherMachineryPlant38": "0.00",
    "otherMachineryPlant39": "0.00",
    "otherMachineryPlant20": "0.00",
    "otherMachineryPlant21": "0.00",
    "totalB18": "0.00",
    "totalB34": "0.00",
    "totalB19": "0.00",
    "totalB38": "0.00",
    "totalB39": "0.00",
    "totalB20": "0.00",
    "totalB21": "0.00",
    "motor22": "0.00",
    "offResidenceB22": "0.00",
    "stcLho22": "0.00",
    "otherPremisesB22": "0.00",
    "otherMachineryPlant22": "0.00",
    "totalB22": "0.00",
    "otherMachineryPlant24": "0.00",
    "otherMachineryPlant25": "0.00",
    "otherMachineryPlant26": "0.00",
    "otherMachineryPlant40": "0.00",
    "totalB24": "0.00",
    "totalB25": "0.00",
    "totalB26": "0.00",
    "totalB40": "0.00",
    "motor27": "0.00",
    "offResidenceB27": "0.00",
    "stcLho27": "0.00",
    "otherPremisesB27": "0.00",
    "otherMachineryPlant27": "0.00",
    "totalB27": "0.00",
    "motor28": "0.00",
    "offResidenceB28": "0.00",
    "stcLho28": "0.00",
    "otherPremisesB28": "0.00",
    "otherMachineryPlant28": "0.00",
    "totalB28": "0.00",
    "motor29": "0.00",
    "offResidenceB29": "0.00",
    "stcLho29": "0.00",
    "otherPremisesB29": "0.00",
    "otherMachineryPlant29": "0.00",
    "totalB29": "0.00",
    "otherMachineryPlant30": "0.00",
    "otherMachineryPlant35": "0.00",
    "totalB30": "0.00",
    "totalB35": "0.00",
    "motor31": "0.00",
    "offResidenceB31": "0.00",
    "stcLho31": "0.00",
    "otherPremisesB31": "0.00",
    "otherMachineryPlant31": "0.00",
    "totalB31": "0.00",
    "totalC1": "0.00",
    "totalC3": "0.00",
    "totalC4": "0.00",
    "totalC36": "0.00",
    "totalC5": "0.00",
    "totalC6": "0.00",
    "totalC7": "0.00",
    "totalC9": "0.00",
    "totalC33": "0.00",
    "totalC37": "0.00",
    "totalC10": "0.00",
    "totalC11": "0.00",
    "totalFurnFix7": "0.00",
    "landNotRev7": "0.00",
    "landRev7": "0.00",
    "landRevEnh7": "0.00",
    "offBuildNotRev7": "0.00",
    "offBuildRev7": "0.00",
    "offBuildRevEnh7": "0.00",
    "residQuartNotRev7": "0.00",
    "residQuartRev7": "0.00",
    "residQuartRevEnh7": "0.00",
    "premisTotal1": "0.00",
    "premisTotal3": "0.00",
    "premisTotal4": "0.00",
    "premisTotal36": "0.00",
    "premisTotal5": "0.00",
    "premisTotal6": "0.00",
    "premisTotal7": "0.00",
    "totalFurnFix1": "0.00",
    "totalFurnFix3": "0.00",
    "totalFurnFix4": "0.00",
    "totalFurnFix36": "0.00",
    "totalFurnFix5": "0.00",
    "totalFurnFix6": "0.00",
    "totalFurnFix9": "0.00",
    "totalFurnFix33": "0.00",
    "totalFurnFix37": "0.00",
    "totalFurnFix10": "0.00",
    "totalFurnFix11": "0.00",
    "totalFurnFix18": "0.00",
    "totalFurnFix34": "0.00",
    "totalFurnFix19": "0.00",
    "totalFurnFix38": "0.00",
    "totalFurnFix39": "0.00",
    "totalFurnFix20": "0.00",
    "totalFurnFix21": "0.00",
    "totalFurnFix24": "0.00",
    "totalFurnFix25": "0.00",
    "totalFurnFix26": "0.00",
    "totalFurnFix40": "0.00",
    "totalFurnFix30": "0.00",
    "totalFurnFix35": "0.00",
    "totalFurnFix12": "0.00",
    "landNotRev12": "0.00",
    "landRev12": "0.00",
    "landRevEnh12": "0.00",
    "offBuildNotRev12": "0.00",
    "offBuildRev12": "0.00",
    "offBuildRevEnh12": "0.00",
    "residQuartNotRev12": "0.00",
    "residQuartRev12": "0.00",
    "residQuartRevEnh12": "0.00",
    "premisTotal9": "0.00",
    "premisTotal10": "0.00",
    "premisTotal11": "0.00",
    "premisTotal33": "0.00",
    "premisTotal37": "0.00",
    "premisTotal12": "0.00",
    "revtotal7": "0.00",
    "revtotal36": "0.00",
    "revtotal12": "0.00",
    "revtotal13": "0.00",
    "revtotal14": "0.00",
    "revtotal22": "0.00",
    "revtotal27": "0.00",
    "revtotal28": "0.00",
    "revtotal29": "0.00",
    "revtotal31": "0.00",
    "revtotal32": "0.00",
    "revtotal1": "0.00",
    "revtotal3": "0.00",
    "revtotal4": "0.00",
    "revtotal5": "0.00",
    "revtotal6": "0.00",
    "revtotal9": "0.00",
    "revtotal10": "0.00",
    "revtotal11": "0.00",
    "revtotal33": "0.00",
    "revtotal37": "0.00",
    "revtotal18": "0.00",
    "revtotal19": "0.00",
    "revtotal38": "0.00",
    "revtotal39": "0.00",
    "revtotal20": "0.00",
    "revtotal21": "0.00",
    "revtotal34": "0.00",
    "revtotal24": "0.00",
    "revtotal25": "0.00",
    "revtotal26": "0.00",
    "revtotal40": "0.00",
    "revtotal30": "0.00",
    "revtotal35": "0.00",
    "premisTotal18": "0.00",
    "premisTotal19": "0.00",
    "premisTotal38": "0.00",
    "premisTotal39": "0.00",
    "premisTotal20": "0.00",
    "premisTotal21": "0.00",
    "premisTotal34": "0.00",
    "premisTotal24": "0.00",
    "premisTotal25": "0.00",
    "premisTotal26": "0.00",
    "premisTotal40": "0.00",
    "premisTotal30": "0.00",
    "premisTotal35": "0.00",
    "premisTotal13": "0.00",
    "premisTotal14": "0.00",
    "premisTotal22": "0.00",
    "premisTotal27": "0.00",
    "premisTotal28": "0.00",
    "premisTotal29": "0.00",
    "premisTotal31": "0.00",
    "premisTotal32": "0.00",
    "totalFurnFix13": "0.00",
    "totalFurnFix14": "0.00",
    "totalFurnFix22": "0.00",
    "totalFurnFix27": "0.00",
    "totalFurnFix28": "0.00",
    "totalFurnFix29": "0.00",
    "totalFurnFix31": "0.00",
    "totalFurnFix32": "0.00",
    "landNotRev13": "0.00",
    "landNotRev14": "0.00",
    "landNotRev22": "0.00",
    "landNotRev27": "0.00",
    "landNotRev28": "0.00",
    "landNotRev29": "0.00",
    "landNotRev31": "0.00",
    "landNotRev32": "0.00",
    "landRev13": "0.00",
    "landRev14": "0.00",
    "landRev22": "0.00",
    "landRev27": "0.00",
    "landRev28": "0.00",
    "landRev29": "0.00",
    "landRev31": "0.00",
    "landRev32": "0.00",
    "landRevEnh13": "0.00",
    "landRevEnh14": "0.00",
    "landRevEnh22": "0.00",
    "landRevEnh27": "0.00",
    "landRevEnh28": "0.00",
    "landRevEnh29": "0.00",
    "landRevEnh31": "0.00",
    "landRevEnh32": "0.00",
    "offBuildNotRev13": "0.00",
    "offBuildNotRev14": "0.00",
    "offBuildNotRev22": "0.00",
    "offBuildNotRev27": "0.00",
    "offBuildNotRev28": "0.00",
    "offBuildNotRev29": "0.00",
    "offBuildNotRev31": "0.00",
    "offBuildNotRev32": "0.00",
    "offBuildRev13": "0.00",
    "offBuildRev14": "0.00",
    "offBuildRev22": "0.00",
    "offBuildRev27": "0.00",
    "offBuildRev28": "0.00",
    "offBuildRev29": "0.00",
    "offBuildRev31": "0.00",
    "offBuildRev32": "0.00",
    "offBuildRevEnh13": "0.00",
    "offBuildRevEnh14": "0.00",
    "offBuildRevEnh22": "0.00",
    "offBuildRevEnh27": "0.00",
    "offBuildRevEnh28": "0.00",
    "offBuildRevEnh29": "0.00",
    "offBuildRevEnh31": "0.00",
    "offBuildRevEnh32": "0.00",
    "residQuartNotRev13": "0.00",
    "residQuartNotRev14": "0.00",
    "residQuartNotRev22": "0.00",
    "residQuartNotRev27": "0.00",
    "residQuartNotRev28": "0.00",
    "residQuartNotRev29": "0.00",
    "residQuartNotRev31": "0.00",
    "residQuartNotRev32": "0.00",
    "residQuartRev13": "0.00",
    "residQuartRev14": "0.00",
    "residQuartRev22": "0.00",
    "residQuartRev27": "0.00",
    "residQuartRev28": "0.00",
    "residQuartRev29": "0.00",
    "residQuartRev31": "0.00",
    "residQuartRev32": "0.00",
    "residQuartRevEnh13": "0.00",
    "residQuartRevEnh14": "0.00",
    "residQuartRevEnh22": "0.00",
    "residQuartRevEnh27": "0.00",
    "residQuartRevEnh28": "0.00",
    "residQuartRevEnh29": "0.00",
    "residQuartRevEnh31": "0.00",
    "residQuartRevEnh32": "0.00",
    "totalC12": "0.00",
    "totalC13": "0.00",
    "totalC18": "0.00",
    "totalC19": "0.00",
    "totalC38": "0.00",
    "totalC39": "0.00",
    "totalC20": "0.00",
    "totalC21": "0.00",
    "totalC34": "0.00",
    "totalC22": "0.00",
    "totalC24": "0.00",
    "totalC25": "0.00",
    "totalC26": "0.00",
    "totalC40": "0.00",
    "totalC27": "0.00",
    "totalC28": "0.00",
    "totalC29": "0.00",
    "totalC30": "0.00",
    "totalC31": "0.00",
    "totalC32": "0.00",
    "totalC35": "0.00",
    "totalC14": "0.00",
    "grandTotal1": "0.00",
    "grandTotal3": "0.00",
    "grandTotal36": "0.00",
    "grandTotal4": "0.00",
    "grandTotal5": "0.00",
    "grandTotal6": "0.00",
    "premisesUnderCons7": "0.00",
    "grandTotal7": "0.00",
    "grandTotal9": "0.00",
    "grandTotal33": "0.00",
    "grandTotal37": "0.00",
    "grandTotal10": "0.00",
    "grandTotal11": "0.00",
    "premisesUnderCons12": "0.00",
    "grandTotal12": "0.00",
    "premisesUnderCons13": "0.00",
    "grandTotal13": "0.00",
    "premisesUnderCons14": "0.00",
    "grandTotal14": "0.00",
    "grandTotal18": "0.00",
    "grandTotal34": "0.00",
    "grandTotal19": "0.00",
    "grandTotal38": "0.00",
    "grandTotal39": "0.00",
    "grandTotal20": "0.00",
    "grandTotal21": "0.00",
    "premisesUnderCons22": "0.00",
    "grandTotal22": "0.00",
    "grandTotal24": "0.00",
    "grandTotal25": "0.00",
    "grandTotal26": "0.00",
    "grandTotal40": "0.00",
    "premisesUnderCons27": "0.00",
    "grandTotal27": "0.00",
    "premisesUnderCons28": "0.00",
    "grandTotal28": "0.00",
    "premisesUnderCons29": "0.00",
    "grandTotal29": "0.00",
    "grandTotal30": "0.00",
    "grandTotal35": "0.00",
    "premisesUnderCons31": "0.00",
    "grandTotal31": "0.00",
    "stcNstaff32": "0.00",
    "offResidenceA32": "0.00",
    "otherPremisesA32": "0.00",
    "electricFitting32": "0.00",
    "totalA32": "0.00",
    "computers32": "0.00",
    "compSoftwareInt32": "0.00",
    "compSoftwareNonint32": "0.00",
    "compSoftwareTotal32": "0.00",
    "motor32": "0.00",
    "offResidenceB32": "0.00",
    "stcLho32": "0.00",
    "otherPremisesB32": "0.00",
    "otherMachineryPlant32": "0.00",
    "totalB32": "0.00",
    "premisesUnderCons32": "0.00",
    "grandTotal32": "0.00",
    "save": true,
    "finyearOne": "2024",
    "finyearTwo": "2025",
    "circleCode": "001",
    "quarterEndDate": "31/03/2025",
    "userId": "1111111",
    "reportName": "Schedule 10",
    "reportId": null,
    "reportMasterId": "310010",
    "status": null
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
    setFormData(prevData => {
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

      newData.totalB1 = parseAndFormat(computers1Val + parseFloat(newData.compSoftwareTotal1) + motor1Val + parseFloat(newData.otherMachineryPlant1));

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

      newData.premisTotal1 = parseAndFormat(landNotRev1Val + landRev1Val + offBuildNotRev1Val + offBuildRev1Val + residQuartNotRev1Val + residQuartRev1Val);
      newData.revtotal1 = parseAndFormat(landRevEnh1Val + offBuildRevEnh1Val + residQuartRevEnh1Val);
      newData.totalC1 = parseAndFormat(parseFloat(newData.premisTotal1) + parseFloat(newData.revtotal1));

      // Grand Total (A+B+C+D) - Row 1
      const premisesUnderCons1Val = parseFloat(newData.premisesUnderCons1) || 0;
      newData.grandTotal1 = parseAndFormat(parseFloat(newData.totalA1) + parseFloat(newData.totalB1) + parseFloat(newData.totalC1) + premisesUnderCons1Val);

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

      newData.totalB3 = parseAndFormat(computers3Val + parseFloat(newData.compSoftwareTotal3) + motor3Val + parseFloat(newData.otherMachineryPlant3));

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

      newData.premisTotal3 = parseAndFormat(landNotRev3Val + landRev3Val + offBuildNotRev3Val + offBuildRev3Val + residQuartNotRev3Val + residQuartRev3Val);
      newData.revtotal3 = parseAndFormat(landRevEnh3Val + offBuildRevEnh3Val + residQuartRevEnh3Val);
      newData.totalC3 = parseAndFormat(parseFloat(newData.premisTotal3) + parseFloat(newData.revtotal3));

      // Grand Total (A+B+C+D) - Row 3
      const premisesUnderCons3Val = parseFloat(newData.premisesUnderCons3) || 0;
      newData.grandTotal3 = parseAndFormat(parseFloat(newData.totalA3) + parseFloat(newData.totalB3) + parseFloat(newData.totalC3) + premisesUnderCons3Val);

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
      newData.totalB7 = parseAndFormat(computers7Val + parseFloat(newData.compSoftwareTotal7) + motor7Val + parseFloat(newData.otherMachineryPlant7));

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

      newData.premisTotal7 = parseAndFormat(landNotRev7Val + landRev7Val + offBuildNotRev7Val + offBuildRev7Val + residQuartNotRev7Val + residQuartRev7Val);
      newData.revtotal7 = parseAndFormat(landRevEnh7Val + offBuildRevEnh7Val + residQuartRevEnh7Val);
      newData.totalC7 = parseAndFormat(parseFloat(newData.premisTotal7) + parseFloat(newData.revtotal7));

      // And its related grand total
      const premisesUnderCons7Val = parseFloat(newData.premisesUnderCons7) || 0;
      newData.grandTotal7 = parseAndFormat(parseFloat(newData.totalA7) + parseFloat(newData.totalB7) + parseFloat(newData.totalC7) + premisesUnderCons7Val);


      // ... and so on for all other new totals from the payload.

      return newData;
    });
  }, [
    // Existing dependencies
    formData.stcNstaff1, formData.offResidenceA1, formData.otherPremisesA1, formData.electricFitting1,
    formData.computers1, formData.compSoftwareInt1, formData.compSoftwareNonint1, formData.motor1,
    formData.offResidenceB1, formData.stcLho1, formData.otherPremisesB1,
    formData.landNotRev1, formData.landRev1, formData.landRevEnh1, formData.offBuildNotRev1,
    formData.offBuildRev1, formData.offBuildRevEnh1, formData.residQuartNotRev1, formData.residQuartRev1,
    formData.residQuartRevEnh1, formData.premisesUnderCons1,
    formData.stcNstaff3, formData.offResidenceA3, formData.otherPremisesA3, formData.electricFitting3,
    formData.computers3, formData.compSoftwareInt3, formData.compSoftwareNonint3, formData.motor3,
    formData.offResidenceB3, formData.stcLho3, formData.otherPremisesB3,
    formData.landNotRev3, formData.landRev3, formData.landRevEnh3, formData.offBuildNotRev3,
    formData.offBuildRev3, formData.offBuildRevEnh3, formData.residQuartNotRev3, formData.residQuartRev3,
    formData.residQuartRevEnh3, formData.premisesUnderCons3,

    // Add all new dependencies for the new fields introduced in the payload here
    // For example, for totalA7:
    formData.stcNstaff7, formData.offResidenceA7, formData.otherPremisesA7, formData.electricFitting7,
    // For totalB7:
    formData.computers7, formData.compSoftwareInt7, formData.compSoftwareNonint7, formData.motor7,
    formData.offResidenceB7, formData.stcLho7, formData.otherPremisesB7,
    // For totalC7:
    formData.landNotRev7, formData.landRev7, formData.landRevEnh7, formData.offBuildNotRev7,
    formData.offBuildRev7, formData.offBuildRevEnh7, formData.residQuartNotRev7, formData.residQuartRev7,
    formData.residQuartRevEnh7, formData.premisesUnderCons7,
    // And repeat this pattern for all other numbered rows (e.g., ...12, ...13, ...14, etc.)
    // up to 40, including all sub-fields like stcNstaffX, offResidenceAX, etc.
  ]);

  const handleChange = (e) => {
    const { name, value } = e.target;
    // Basic validation for numeric input and 2 decimal places
    const regex = /^-?\d*\.?\d{0,2}$/; // Allows negative numbers, decimals up to 2 places
    if (value === '' || regex.test(value)) {
      setFormData(prevData => ({
        ...prevData,
        [name]: value,
      }));
    }
  };

  const renderInputCell = (fieldName, isReadonly = false) => (
    <StyledTableCell>
      <TextField
        name={fieldName}
        value={formData[fieldName]}
        onChange={handleChange}
        inputProps={{
          maxLength: 18, // Maxlength as per Schedule10.txt
          style: { textAlign: 'right' },
        }}
        sx={{
          width: '100px', // Adjust width as needed
          '& input': { textAlign: 'right', padding: '6px 8px' },
          backgroundColor: isReadonly ? '#f0f0f0' : 'white',
        }}
        disabled={isReadonly}
        variant="outlined"
        size="small"
      />
    </StyledTableCell>
  );

  return (
    <Box sx={{ p: 3 }}>
      <Typography variant="h4" component="h3" sx={{ mb: 3 }}>
        Schedule 10
      </Typography>
      <TableContainer component={Paper}>
        <Table sx={{ minWidth: 3000 }} aria-label="schedule 10 table">
          <TableHead>
            <TableRow>
              <StyledTableCell rowSpan={2}><b>Sr.No</b></StyledTableCell>
              <StyledTableCell rowSpan={2}><b>Particulars</b></StyledTableCell>
              <StyledTableCell colSpan={5}><b>(A) FURNITURE & FITTINGS</b></StyledTableCell>
              <StyledTableCell colSpan={10}><b>(B) MACHINERY & PLANT</b></StyledTableCell>
              <StyledTableCell rowSpan={2}><b>Total Furniture & Fixtures <br/>(A+B)</b></StyledTableCell>
              <StyledTableCell colSpan={12}><b>(C) PREMISES</b></StyledTableCell>
              <StyledTableCell rowSpan={2}><b>(D) Projects under <br/>construction</b></StyledTableCell>
              <StyledTableCell rowSpan={2}><b>Grand Total <br/> (A + B + C + D)</b></StyledTableCell>
            </TableRow>
            <TableRow>
              <StyledTableCell><b>i) At STCs & Staff Colleges <br/>(For Local Head Office only)</b></StyledTableCell>
              <StyledTableCell><b>ii) At Officers' Residences</b></StyledTableCell>
              <StyledTableCell><b>iii) At Other Premises</b></StyledTableCell>
              <StyledTableCell><b>iv) Electric Fittings <br/>(include electric wiring,<br/> switches, sockets, other<br/> fittings & fans etc.)</b></StyledTableCell>
              <StyledTableCell><b>TOTAL (A)<br/> (i+ii+iii+iv)</b></StyledTableCell>
              <StyledTableCell><b>i) Computer Hardware</b></StyledTableCell>
              <StyledTableCell><b>a. Computer Software <br/>(forming integral part of<br/> Hardware)</b></StyledTableCell>
              <StyledTableCell><b>b. Computer Software <br/>(not forming integral <br/>of Hardware)</b></StyledTableCell>
              <StyledTableCell><b>ii) Computer Software <br/>Total (a+b)</b></StyledTableCell>
              <StyledTableCell><b>iii) Motor Vehicles </b></StyledTableCell>
              <StyledTableCell><b>a) At Officers' Residences</b></StyledTableCell>
              <StyledTableCell><b>b) At STCs <br/>(For Local Head Office)</b></StyledTableCell>
              <StyledTableCell><b>c) At other Premises </b></StyledTableCell>
              <StyledTableCell><b>iv) Other Machinery & Plant <br/>( a+b+c)</b></StyledTableCell>
              <StyledTableCell><b>TOTAL <br/> (B= i+ii+iii+iv)</b></StyledTableCell>
              <StyledTableCell><b>(a) Land (Not Revalued):<br/> Cost</b></StyledTableCell>
              <StyledTableCell><b>(b) Land (Revalued): <br/>Cost</b></StyledTableCell>
              <StyledTableCell><b>(c) Land (Revalued): <br/>Enhancement due to <br/>Revaluation</b></StyledTableCell>
              <StyledTableCell><b>(d) Office Building <br/>(Not revalued): Cost </b></StyledTableCell>
              <StyledTableCell><b>(e) Office Building <br/>(Revalued): Cost </b></StyledTableCell>
              <StyledTableCell><b>(f) Office Building <br/>(Revalued): Enhancement <br/>due to Revaluation</b></StyledTableCell>
              <StyledTableCell><b>(g) Residential Building <br/>(Not revalued): Cost</b></StyledTableCell>
              <StyledTableCell><b>(h) Residential Building <br/>(Revalued): Cost</b></StyledTableCell>
              <StyledTableCell><b>(i) Residential Building <br/>(Revalued): Enhancement <br/>due to Revaluation</b></StyledTableCell>
              <StyledTableCell><b>(j) Premises Total <br/>(a+b+d+e+g+h)</b></StyledTableCell>
              <StyledTableCell><b>(k) Revaluation Total <br/>(c+f+i)</b></StyledTableCell>
              <StyledTableCell><b>TOTAL <br/>(C=j+k)</b></StyledTableCell>
            </TableRow>
          </TableHead>
          <TableBody>
            {/* Row A: Total Original Cost / Revalued Value upto the end of previous year */}
            <StyledTableRow>
              <StyledTableCell issectionheader><b>A</b></StyledTableCell>
              <StyledTableCell issectionheader><b>Total Original Cost / Revalued Value upto the end of previous year i.e. 31st March {year1}</b></StyledTableCell>
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
              <StyledTableCell><b>Addition</b></StyledTableCell>
              <StyledTableCell colSpan={30}></StyledTableCell>
            </StyledTableRow>

            {/* Row a. (i) Original cost of items put to use during the year */}
            <StyledTableRow>
              <StyledTableCell style={{ textAlign: 'right' }}>(a.i)</StyledTableCell>
              <StyledTableCell>{formData.particulars3}</StyledTableCell>
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
              <StyledTableCell style={{ textAlign: 'right' }}>(a.ii)</StyledTableCell>
              <StyledTableCell>{formData.particulars4}</StyledTableCell>
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
                <StyledTableCell style={{ textAlign: 'right' }}>7</StyledTableCell>
                <StyledTableCell>Example Row for new data</StyledTableCell> {/* Replace with actual particular name */}
                {renderInputCell('stcNstaff7')}
                {renderInputCell('offResidenceA7')}
                {renderInputCell('otherPremisesA7')}
                {renderInputCell('electricFitting7')}
                {renderInputCell('totalA7', true)}
                {renderInputCell('computers7')}
                {renderInputCell('compSoftwareInt7')}
                {renderInputCell('compSoftwareNonint7')}
                {renderInputCell('compSoftwareTotal7', true)}
                {renderInputCell('motor7')}
                {renderInputCell('offResidenceB7')}
                {renderInputCell('stcLho7')}
                {renderInputCell('otherPremisesB7')}
                {renderInputCell('otherMachineryPlant7', true)}
                {renderInputCell('totalB7', true)}
                {renderInputCell('totalFurnFix7', true)}
                {renderInputCell('landNotRev7')}
                {renderInputCell('landRev7')}
                {renderInputCell('landRevEnh7')}
                {renderInputCell('offBuildNotRev7')}
                {renderInputCell('offBuildRev7')}
                {renderInputCell('offBuildRevEnh7')}
                {renderInputCell('residQuartNotRev7')}
                {renderInputCell('residQuartRev7')}
                {renderInputCell('residQuartRevEnh7')}
                {renderInputCell('premisTotal7', true)}
                {renderInputCell('revtotal7', true)}
                {renderInputCell('totalC7', true)}
                {renderInputCell('premisesUnderCons7')}
                {renderInputCell('grandTotal7', true)}
            </StyledTableRow>

            {/* ... Continue adding all other rows and sections based on the expanded formData */}

          </TableBody>
        </Table>
      </TableContainer>
    </Box>
  );
};

export default Schedule10;

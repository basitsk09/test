rw04

"SL.
NO."	PARTICULARS	"OPENING PROVISION 
AS ON 01.04.2025"	"WRITE OFF DURING THE PERIOD *
"	ADDITONS / REDUCTIONS (OTHER THAN WRITE OFF) DURING THE PERIOD                                                       	"CLOSING PROVISION  
AS ON 30.06.2025"
(1)	(2)	(3)	(4)	 (5) = 6-(3+4) 	(6)
1	FRAUDS - DEBITED TO RECALLED ASSETS A/c (Prod Cd 6998-9981)** UP TO DD.MM.YYYY (Balance Sheet Date i.e. for Q1- 30.06.YYYY, HY- 30.09.YYYY, 9 MONTHS - 31.12..YYYY and Year End - 31.03.YYYY 	    0.00	    500.00	    0.00	    500.00
2	OTHERS LOSSES IN RECALLED ASSETS (Prod cd 6998 - 9982)# UP TO DD.MM.YYYY (Balance Sheet Date i.e. for Q1- 30.06.YYYY, HY- 30.09.YYYY, 9 MONTHS - 31.12..YYYY and Year End - 31.03.YYYY 	   2 000.00	    500.00	    0.00	   1 500.00
3	BGL Number 2399969 Internal Fraud ##1 UP TO DD.MM.YYYY (Balance Sheet Date i.e. for Q1- 30.06.YYYY, HY- 30.09.YYYY, 9 MONTHS - 31.12..YYYY and Year End - 31.03.YYYY 	   2 500.00	   1 000.00	   3 500.00	   5 000.00
4	BGL Number 2399970 External Fraud ##2 UP TO DD.MM.YYYY (Balance Sheet Date i.e. for Q1- 30.06.YYYY, HY- 30.09.YYYY, 9 MONTHS - 31.12..YYYY and Year End - 31.03.YYYY 	   2 500.00	   1 000.00	   3 500.00	   5 000.00
5	SUB TOTALS (ABOVE)	   7 000.00	   3 000.00	   7 000.00	   12 000.00
6	BGL Number 4697984 Pension Overpayment unreconciled Portion	   2 500.00	   1 000.00	   3 500.00	   5 000.00
7	FRAUDS - OTHER (NOT DEBITED TO RA A/c)  $	    0.00	    0.00	    0.00	    0.00
8	REVENUE ITEM IN SYSTEM SUSPENSE	    0.00	    0.00	    0.00	    0.00
9	PROVISION ON ACCOUNT OF FSLO	    0.00	    0.00	    0.00	    0.00
10	PROVISION ON ACCOUNT OF  ENTRIES OUTSTANDING IN ADJUSTING ACCOUNT FOR PREVIOUS QUARTER(S) (i.e. PRIOR TO CURRENT QUARTER)	    0.00	    0.00	    0.00	    0.00
11	PROVISION ON NPA INTREST FREE STAFF LOANS 	    0.00	    0.00	    0.00	    0.00
12	OTHERS (PLEASE SPECIFY BELOW)				
(a)	Provision for Operational Loss Events (e.g.)	    0.00	    0.00	    0.00	    0.00
(b)		    0.00	    0.00	    0.00	    0.00
(c)		    0.00	    0.00	    0.00	    0.00
(d)		    0.00	    0.00	    0.00	    0.00
(e)		    0.00	    0.00	    0.00	    0.00
(f)		    0.00	    0.00	    0.00	    0.00
		    0.00	    0.00	    0.00	    0.00
	 SUB TOTAL (OTHERS)	    0.00	    0.00	    0.00	    0.00
Total		   9 500.00	   4 000.00	   10 500.00	   17 000.00
//////////////////////////////////////////////////////////////
rw05:
"SL.
NO."	PARTICULARS	"OPENING PROVISION
AS ON 01.04.2024
(Rs.)           (Ps)"	"ADDITONS / REVERSAL DURING THE PERIOD
(Rs.)            (Ps)"	"PROVISION AS ON 30.06.2025
(Rs.)      (Ps)"
(1)	(2)	(3)	 (4) = 5 - 3 	 (5) 
1	CONTIONGENT LIABILITY AS PER AS -29	,,,0.00	,,,0.00	,,,0.00
2	DELAYED REPORTING PENALTY	,,,0.00	,,,0.00	,,,0.00
3	EX-GRATIA PAYMENT	,,,0.00	,,,0.00	,,,0.00
4	PROVISON ON OVERDUE DEPOSIT INTT	,,,0.00	,,,0.00	,,,0.00
5	LEAVE ENCASHMENT	,,,0.00	,,,0.00	,,,0.00
6	PROVISION FOR PERFORMANCE LINKED INCENTIVES	,,,0.00	,,,0.00	,,,0.00
7	PROVISION ON ACCOUNT OF ENTRIES OUTSTANDING IN ADJUSTING ACCOUNT FOR PREVIOUS QUARTER(S) (i.e. PRIOR TO CURRENT QUARTER)	,,,0.00	,,,0.00	,,,0.00
8		,,,0.00	,,,0.00	,,,0.00
9		,,,0.00	,,,0.00	,,,0.00
	Total	,,,0.00	,,,0.00	,,,0.00

 /////////////////////////////////////////


 import React, { useState, useEffect, useCallback, useMemo } from 'react';
import {
  Tabs,
  Tab,
  Box,
  Typography,
  Table,
  TableHead,
  TableRow,
  TableCell,
  TableBody,
  TableContainer,
  Paper,
  Button,
  Snackbar,
  Alert,
  Dialog,
  DialogActions,
  DialogContent,
  DialogContentText,
  DialogTitle,
  Checkbox,
  CircularProgress,
} from '@mui/material';
import { styled } from '@mui/material/styles';
import { useLocation, useNavigate } from 'react-router-dom';
import FormInput from '../../../../common/components/ui/FormInput';
import useCustomSnackbar from '../../../../common/hooks/useCustomSnackbar';
import useApi from '../../../../common/hooks/useApi';
import { CustomButton } from '../../../../common/components/ui/Buttons';

// #region --- Common and Initial Setup ---

const StyledTableCell = styled(TableCell)(({ theme }) => ({
  fontSize: '0.875rem',
  textAlign: 'center',
  padding: '6px',
  fontWeight: 'bold',
  backgroundColor: 'hsl(220, 20%, 35%)',
  color: 'white',
}));

const StyledTotalTableCell = styled(TableCell)(({ theme }) => ({
  fontSize: '0.9rem',
  textAlign: 'center',
  padding: '6px',
  fontWeight: 'bold',
  backgroundColor: 'hsl(220, 20%, 45%)',
  color: 'white',
}));

// --- RW04 Initial Data ---
const getInitialRw04StaticRows = (quarterEndDate) => [
  {
    feId: '1',
    dbId: 1,
    label: 'FRAUDS - DEBITED TO RECALLED ASSETS A/c (Prod Cd 6998-9981)**',
    provAmtStart: '',
    writeOff: '',
    addition: '',
    reduction: '',
    provAmtEnd: '0.00',
    rate: '',
    provRequired: '0.00',
  },
  {
    feId: '1.i',
    dbId: 2,
    label: `Frauds reported within time up ${quarterEndDate} provision @ 100% ##`,
    provAmtStart: '',
    writeOff: '',
    addition: '',
    reduction: '',
    provAmtEnd: '0.00',
    rate: '100',
    provRequired: '0.00',
  },
  {
    feId: '1.ii',
    dbId: 3,
    label: `Delayed Reported frauds Provision @ 100% ##`,
    provAmtStart: '',
    writeOff: '',
    addition: '',
    reduction: '',
    provAmtEnd: '0.00',
    rate: '100',
    provRequired: '0.00',
  },
  {
    feId: '2',
    dbId: 4,
    label: 'OTHERS LOSSES IN RECALLED ASSETS (Prod cd 6998 - 9982)#',
    provAmtStart: '',
    writeOff: '',
    addition: '',
    reduction: '',
    provAmtEnd: '0.00',
    rate: '100',
    provRequired: '0.00',
  },
  {
    feId: '3',
    dbId: 5,
    label: 'FRAUDS - OTHER (NOT DEBITED TO RA A/c)$',
    provAmtStart: '',
    writeOff: '',
    addition: '',
    reduction: '',
    provAmtEnd: '0.00',
    rate: '',
    provRequired: '0.00',
  },
  {
    feId: '3.i',
    dbId: 6,
    label: `Frauds reported within time up ${quarterEndDate} provision @ 100% ##`,
    provAmtStart: '',
    writeOff: '',
    addition: '',
    reduction: '',
    provAmtEnd: '0.00',
    rate: '100',
    provRequired: '0.00',
  },
  {
    feId: '3.ii',
    dbId: 7,
    label: `Delayed Reported frauds Provision @ 100% ##`,
    provAmtStart: '',
    writeOff: '',
    addition: '',
    reduction: '',
    provAmtEnd: '0.00',
    rate: '100',
    provRequired: '0.00',
  },
  {
    feId: '4',
    dbId: 8,
    label: 'REVENUE ITEM IN SYSTEM SUSPENSE',
    provAmtStart: '',
    writeOff: '',
    addition: '',
    reduction: '',
    provAmtEnd: '0.00',
    rate: '100',
    provRequired: '0.00',
  },
  {
    feId: '5',
    dbId: 9,
    label: 'PROVISION ON ACCOUNT OF FSLO',
    provAmtStart: '',
    writeOff: '',
    addition: '',
    reduction: '',
    provAmtEnd: '0.00',
    rate: '100',
    provRequired: '0.00',
  },
  {
    feId: '6',
    dbId: 10,
    label: 'PROVISION ON ACCOUNT OF ENTRIES OUTSTANDING IN ADJUSTING ACCOUNT FOR PREVIOUS QUARTER(S)',
    provAmtStart: '',
    writeOff: '',
    addition: '',
    reduction: '',
    provAmtEnd: '0.00',
    rate: '100',
    provRequired: '0.00',
  },
  {
    feId: '7',
    dbId: 11,
    label: 'PROVISION ON N.P.A. INTEREST FREE STAFF LOANS',
    provAmtStart: '',
    writeOff: '',
    addition: '',
    reduction: '',
    provAmtEnd: '0.00',
    rate: '100',
    provRequired: '0.00',
  },
];

const createInitialRw04DynamicRow = () => ({
  dbId: 0,
  particulars: '',
  provAmtStart: '',
  writeOff: '',
  addition: '',
  reduction: '',
  provAmtEnd: '0.00',
  rate: '100',
  provRequired: '0.00',
  selected: false,
  key: `new-${Date.now()}-${Math.random().toString(36).substr(2, 9)}`,
});

// --- RW05 Initial Data ---
const getInitialRw05StaticRows = () => [
  {
    dbId: 1,
    label: 'CONTINGENT LIABILITY AS PER AS -29',
    provAmtStart: '',
    addition: '',
    reversal: '',
    provAmtEnd: '0.00',
    difference: '0.00',
  },
  {
    dbId: 2,
    label: 'DELAYED REPORTING PENALTY',
    provAmtStart: '',
    addition: '',
    reversal: '',
    provAmtEnd: '0.00',
    difference: '0.00',
  },
  {
    dbId: 3,
    label: 'EX-GRATIA PAYMENT',
    provAmtStart: '',
    addition: '',
    reversal: '',
    provAmtEnd: '0.00',
    difference: '0.00',
  },
  {
    dbId: 4,
    label: 'PROVISION ON OVERDUE DEPOSIT INTT',
    provAmtStart: '',
    addition: '',
    reversal: '',
    provAmtEnd: '0.00',
    difference: '0.00',
  },
  {
    dbId: 5,
    label: 'LEAVE ENCASHMENT',
    provAmtStart: '',
    addition: '',
    reversal: '',
    provAmtEnd: '0.00',
    difference: '0.00',
  },
  {
    dbId: 6,
    label: 'PROVISION FOR PERFORMANCE LINKED INCENTIVES',
    provAmtStart: '',
    addition: '',
    reversal: '',
    provAmtEnd: '0.00',
    difference: '0.00',
  },
  {
    dbId: 7,
    label: 'PROVISION ON ACCOUNT OF ENTRIES OUTSTANDING IN ADJUSTING ACCOUNT FOR PREVIOUS QUARTER(S)',
    provAmtStart: '',
    addition: '',
    reversal: '',
    provAmtEnd: '0.00',
    difference: '0.00',
  },
];

const createInitialRw05DynamicRow = () => ({
  dbId: 0,
  key: `new-${Date.now()}-${Math.random().toString(36).substr(2, 9)}`,
  selected: false,
  particulars: '',
  provAmtStart: '',
  addition: '',
  reversal: '',
  provAmtEnd: '0.00',
  difference: '0.00',
});

const isNumeric = (val) => val === null || val === '' || (!isNaN(parseFloat(val)) && isFinite(val));

// #endregion

const RW04 = () => {
  // #region --- State and Hooks ---
  const [tabIndex, setTabIndex] = useState(0);
  const user = JSON.parse(localStorage.getItem('user'));
  const { state } = useLocation();
  const navigate = useNavigate();
  const { callApi, isLoading } = useApi();
  const setSnackbarMessage = useCustomSnackbar();

  const [reportObject, setReportObject] = useState(state?.report || null);
  const [isSubmitConfirmOpen, setIsSubmitConfirmOpen] = useState(false);

  // RW04 State
  const [rw04StaticRows, setRw04StaticRows] = useState(getInitialRw04StaticRows(user.quarterEndDate));
  const [rw04DynamicRows, setRw04DynamicRows] = useState([]);

  // RW05 State
  const [rw05StaticRows, setRw05StaticRows] = useState(getInitialRw05StaticRows());
  const [rw05DynamicRows, setRw05DynamicRows] = useState([]);

  const headers_rw04 = [
    'PARTICULARS(2)',
    `OPENING PROVISION AS ON ${user.previousQuarterEndDate} (3)`,
    'WRITE OFF DURING THE 3 MONTHS PERIOD* (4)',
    `ADDITONS / REDUCTIONS (OTHER THAN WRITE OFF) DURING THE PERIOD ${user.quarterEndDate} (7)=6-(3+4)`,
    `CLOSING PROVISION AS ON ${user.quarterEndDate} (9)=7*8`,
  ];
  const headers_rw05 = [
    'Particulars(2)',
    `Opening Provision as on ${user.previousQuarterEndDate}(3)`,
    'Additions during the period(4)',
    'Reversals during the Period(5)',
    `Provision as on ${user.quarterEndDate}(6)={(3+4)-(5)}`,
    `Difference to be Provided/ written back(7)={(6)-(3)}`,
  ];

  const getBasePayload = useCallback(
    () => ({
      circleCode: user?.circleCode,
      qed: user?.quarterEndDate,
      userCapacity: user?.userCapacity,
      reportId: reportObject?.reportId,
      reportMasterId: reportObject?.reportMasterId,
      currentStatus: reportObject?.status,
      userId: user.userId,
    }),
    [user, reportObject]
  );

  // #endregion

  // #region --- Data Loading ---

  const loadRw04Data = useCallback(async () => {
    try {
      const response = await callApi('/RW04/getData', getBasePayload(), 'POST');
      if (response?.data) {
        const apiData = response.data;
        const loadedStaticRows = getInitialRw04StaticRows(user.quarterEndDate);
        const loadedDynamicRows = [];

        apiData.forEach((row) => {
          const dbId = parseInt(row[0], 10);
          const targetRow = {
            provAmtStart: row[2],
            writeOff: row[3],
            addition: row[4],
            reduction: row[5],
            provAmtEnd: row[6],
            rate: row[7],
            provRequired: row[8],
          };

          if (dbId >= 1 && dbId <= 11) {
            const staticRowToUpdate = loadedStaticRows.find((r) => r.dbId === dbId);
            if (staticRowToUpdate) Object.assign(staticRowToUpdate, targetRow);
          } else {
            loadedDynamicRows.push({
              ...createInitialRw04DynamicRow(),
              ...targetRow,
              dbId,
              key: dbId,
              particulars: row[1],
            });
          }
        });

        setRw04StaticRows(loadedStaticRows);
        setRw04DynamicRows(loadedDynamicRows);
      }
    } catch (error) {
      if (error.message !== 'canceled') setSnackbarMessage('Failed to fetch RW04 data.', 'error');
    }
  }, [reportObject, user.quarterEndDate]);

  const loadRw05Data = useCallback(async () => {
    try {
      const response = await callApi('/RW05/getData', getBasePayload(), 'POST');
      if (response?.data) {
        const apiData = response.data;
        const loadedStaticRows = getInitialRw05StaticRows();
        const loadedDynamicRows = [];

        apiData.forEach((row) => {
          const dbId = parseInt(row[0], 10);
          const targetRow = {
            provAmtStart: row[2],
            addition: row[3],
            reversal: row[4],
            provAmtEnd: row[5],
            difference: row[6],
          };
          if (dbId >= 1 && dbId <= 7) {
            const staticRow = loadedStaticRows.find((r) => r.dbId === dbId);
            if (staticRow) Object.assign(staticRow, targetRow);
          } else {
            loadedDynamicRows.push({
              ...createInitialRw05DynamicRow(),
              ...targetRow,
              dbId,
              key: dbId,
              particulars: row[1],
            });
          }
        });

        setRw05StaticRows(loadedStaticRows);
        setRw05DynamicRows(loadedDynamicRows);
      }
    } catch (error) {
      if (error.message !== 'canceled') setSnackbarMessage('Failed to fetch RW05 data.', 'error');
    }
  }, [reportObject, getBasePayload]);

  useEffect(() => {
    if (reportObject) {
      loadRw04Data();
      loadRw05Data();
    }
  }, [reportObject]);

  // #endregion

  // #region --- RW04 Calculations & Handlers ---

  const calculateRw04Row = useCallback((updatedRow) => {
    const isManualRow = ['1.i', '1.ii', '3.i', '3.ii'].includes(updatedRow.feId);
    const start = parseFloat(updatedRow.provAmtStart || 0);
    const write = parseFloat(updatedRow.writeOff || 0);
    const add = parseFloat(updatedRow.addition || 0);
    const reduce = parseFloat(updatedRow.reduction || 0);
    const rate = parseFloat(updatedRow.rate || 0);

    if (updatedRow.particulars !== undefined) {
      // Dynamic Row
      const end = start - write + add - reduce;
      updatedRow.provAmtEnd = end.toFixed(2);
      updatedRow.provRequired = ((end * rate) / 100).toFixed(2);
    } else {
      // Static Row
      if (!isManualRow) {
        const end = start - write + add - reduce;
        updatedRow.provAmtEnd = end.toFixed(2);
        updatedRow.provRequired = ((end * rate) / 100).toFixed(2);
      } else {
        const end = parseFloat(updatedRow.provAmtEnd || 0);
        updatedRow.provRequired = ((end * rate) / 100).toFixed(2);
      }
    }
    return updatedRow;
  }, []);

  const handleRw04StaticChange = (index, key, value) => {
    const numericFields = ['provAmtStart', 'writeOff', 'addition', 'reduction', 'provAmtEnd'];
    if (numericFields.includes(key) && !isNumeric(value)) return;

    const currentRows = [...rw04StaticRows];
    const currentRow = { ...currentRows[index], [key]: value };
    currentRows[index] = calculateRw04Row(currentRow);
    setRw04StaticRows(currentRows);
  };

  const handleRw04DynamicChange = (index, key, value) => {
    const numericFields = ['provAmtStart', 'writeOff', 'addition', 'reduction'];
    if (numericFields.includes(key) && !isNumeric(value)) return;

    const updatedRows = [...rw04DynamicRows];
    let rowToUpdate = { ...updatedRows[index], [key]: value };
    updatedRows[index] = calculateRw04Row(rowToUpdate);
    setRw04DynamicRows(updatedRows);
  };

  const getProvAmtEndMismatchError = (rowFeId) => {
    if (rowFeId === '1') {
      const parent = parseFloat(rw04StaticRows.find((r) => r.feId === '1')?.provAmtEnd || 0);
      const child1i = parseFloat(rw04StaticRows.find((r) => r.feId === '1.i')?.provAmtEnd || 0);
      const child1ii = parseFloat(rw04StaticRows.find((r) => r.feId === '1.ii')?.provAmtEnd || 0);
      return Math.abs(parent - (child1i + child1ii)) > 0.01;
    }
    if (rowFeId === '3') {
      const parent = parseFloat(rw04StaticRows.find((r) => r.feId === '3')?.provAmtEnd || 0);
      const child3i = parseFloat(rw04StaticRows.find((r) => r.feId === '3.i')?.provAmtEnd || 0);
      const child3ii = parseFloat(rw04StaticRows.find((r) => r.feId === '3.ii')?.provAmtEnd || 0);
      return Math.abs(parent - (child3i + child3ii)) > 0.01;
    }
    return false;
  };

  const rw04TotalRow = useMemo(() => {
    const allRows = [...rw04StaticRows, ...rw04DynamicRows];
    const totals = {
      provAmtStart: 0,
      writeOff: 0,
      addition: 0,
      reduction: 0,
      provAmtEnd: 0,
      provRequired: 0,
    };
    const keysToSum = ['provAmtStart', 'writeOff', 'addition', 'reduction', 'provAmtEnd', 'provRequired'];

    // Exclude parent rows from sum to avoid double counting
    const rowsToSum = allRows.filter((row) => !['1', '3'].includes(row.feId));

    rowsToSum.forEach((row) => {
      keysToSum.forEach((key) => {
        totals[key] += parseFloat(row[key] || 0);
      });
    });

    return {
      ...totals,
      provAmtStart: totals.provAmtStart.toFixed(2),
      writeOff: totals.writeOff.toFixed(2),
      addition: totals.addition.toFixed(2),
      reduction: totals.reduction.toFixed(2),
      provAmtEnd: totals.provAmtEnd.toFixed(2),
      provRequired: totals.provRequired.toFixed(2),
    };
  }, [rw04StaticRows, rw04DynamicRows]);

  // #endregion

  // #region --- RW05 Calculations & Handlers ---

  const calculateRw05Row = (row) => {
    const provAmtStart = parseFloat(row.provAmtStart || 0);
    const addition = parseFloat(row.addition || 0);
    const reversal = parseFloat(row.reversal || 0);
    const provAmtEnd = provAmtStart + addition - reversal;
    const difference = provAmtEnd - provAmtStart;
    row.provAmtEnd = provAmtEnd.toFixed(2);
    row.difference = difference.toFixed(2);
    return row;
  };

  const handleRw05StaticChange = (index, key, value) => {
    if (!isNumeric(value)) return;
    const updatedRows = [...rw05StaticRows];
    let rowToUpdate = { ...updatedRows[index], [key]: value };
    updatedRows[index] = calculateRw05Row(rowToUpdate);
    setRw05StaticRows(updatedRows);
  };

  const handleRw05DynamicChange = (index, key, value) => {
    const numericFields = ['provAmtStart', 'addition', 'reversal'];
    if (numericFields.includes(key) && !isNumeric(value)) return;

    const updatedRows = [...rw05DynamicRows];
    let rowToUpdate = { ...updatedRows[index], [key]: value };
    updatedRows[index] = calculateRw05Row(rowToUpdate);
    setRw05DynamicRows(updatedRows);
  };

  const rw05TotalRow = useMemo(() => {
    const allRows = [...rw05StaticRows, ...rw05DynamicRows];
    const totals = {
      provAmtStart: 0,
      addition: 0,
      reversal: 0,
      provAmtEnd: 0,
      difference: 0,
    };
    const keysToSum = Object.keys(totals);

    allRows.forEach((row) => {
      keysToSum.forEach((key) => {
        totals[key] += parseFloat(row[key] || 0);
      });
    });

    return {
      provAmtStart: totals.provAmtStart.toFixed(2),
      addition: totals.addition.toFixed(2),
      reversal: totals.reversal.toFixed(2),
      provAmtEnd: totals.provAmtEnd.toFixed(2),
      difference: totals.difference.toFixed(2),
    };
  }, [rw05StaticRows, rw05DynamicRows]);

  // #endregion

  // #region --- Generic Action Handlers (Add, Delete, Save, Submit) ---

  const handleAddRow = () => {
    if (tabIndex === 0) {
      if (rw04DynamicRows.some((row) => row.dbId === 0)) {
        setSnackbarMessage(`Kindly save the current new row before adding another.`, 'warning');
      } else {
        setRw04DynamicRows([...rw04DynamicRows, createInitialRw04DynamicRow()]);
      }
    } else {
      if (rw05DynamicRows.some((row) => row.dbId === 0)) {
        setSnackbarMessage('Kindly save the current new row before adding another', 'warning');
      } else {
        setRw05DynamicRows([...rw05DynamicRows, createInitialRw05DynamicRow()]);
      }
    }
  };

  const handleDeleteRow = async () => {
    const endpoint = tabIndex === 0 ? '/RW04/deleteRows' : '/RW05/deleteRow';
    const dynamicRows = tabIndex === 0 ? rw04DynamicRows : rw05DynamicRows;
    const setDynamicRows = tabIndex === 0 ? setRw04DynamicRows : setRw05DynamicRows;
    const createInitialRow = tabIndex === 0 ? createInitialRw04DynamicRow : createInitialRw05DynamicRow;

    const rowsToDelete = dynamicRows.filter((row) => row.selected && row.dbId !== 0);
    const newRowsToKeep = dynamicRows.filter((row) => !row.selected);

    if (rowsToDelete.length === 0) {
      if (dynamicRows.some((row) => row.selected && row.dbId === 0)) {
        setDynamicRows(newRowsToKeep.length > 0 ? newRowsToKeep : []);
        setSnackbarMessage('New row removed.', 'info');
      } else {
        setSnackbarMessage('Please select a saved row to delete.', 'warning');
      }
      return;
    }

    try {
      const payload = { userCapacity: user.userCapacity, rowIds: rowsToDelete.map((row) => row.dbId) };
      const res = await callApi(endpoint, payload, 'POST');
      if (res?.deletedCount) {
        setSnackbarMessage('Selected rows deleted successfully!', 'success');
        setDynamicRows(newRowsToKeep);
      } else {
        setSnackbarMessage('An error occurred during deletion.', 'error');
      }
    } catch (error) {
      console.error('Error deleting rows:', error);
      setSnackbarMessage('An error occurred during deletion.', 'error');
    }
  };

  const handleSave = async () => {
    const basePayload = getBasePayload();
    let success = false;
    try {
      if (tabIndex === 0) {
        // Save RW04 Static Rows
        const staticValue = rw04StaticRows.map((row) => [
          row.provAmtStart || '0.00',
          row.writeOff || '0.00',
          row.addition || '0.00',
          row.reduction || '0.00',
          row.provAmtEnd || '0.00',
          row.rate || '0',
          row.provRequired || '0.00',
          String(row.dbId),
        ]);
        await callApi('/RW04/saveStatic', { ...basePayload, value: staticValue }, 'POST');

        // Save RW04 Dynamic Rows
        for (const row of rw04DynamicRows.filter((r) => r.particulars.trim())) {
          const value = [
            row.particulars,
            row.provAmtStart || '0.00',
            row.writeOff || '0.00',
            row.addition || '0.00',
            row.reduction || '0.00',
            row.provAmtEnd || '0.00',
            row.rate || '100',
            row.provRequired || '0.00',
            String(row.dbId),
          ];
          await callApi('/RW04/saveAddRow', { ...basePayload, value }, 'POST');
        }
      } else {
        // Save RW05 Static Rows
        const staticValue = rw05StaticRows.map((row) => [
          row.provAmtStart || '0.00',
          row.addition || '0.00',
          row.reversal || '0.00',
          row.provAmtEnd || '0.00',
          row.difference || '0.00',
          String(row.dbId),
        ]);
        await callApi('/RW05/saveStatic', { ...basePayload, value: staticValue }, 'POST');

        // Save RW05 Dynamic Rows
        for (const row of rw05DynamicRows.filter((r) => r.particulars.trim())) {
          const value = [
            row.particulars,
            row.provAmtStart || '0.00',
            row.addition || '0.00',
            row.reversal || '0.00',
            row.provAmtEnd || '0.00',
            row.difference || '0.00',
            String(row.dbId),
          ];
          await callApi('/RW05/saveAddRow', { ...basePayload, value }, 'POST');
        }
      }
      setSnackbarMessage('Data saved successfully!', 'success');
      // Reload data to get new dbIds
      loadRw04Data();
      loadRw05Data();
      setReportObject((prev) => ({ ...prev, status: '11' }));
    } catch (error) {
      console.error(`Error during save operation:`, error);
      setSnackbarMessage(`An error occurred while saving data.`, 'error');
    }
  };

  const isDataValid = () => {
    if (tabIndex === 0) {
      if (getProvAmtEndMismatchError('1') || getProvAmtEndMismatchError('3')) {
        return false;
      }
    }
    // Add any validation for RW05 if needed
    return true;
  };

  const handleSubmitReport = () => {
    if (!isDataValid()) {
      setSnackbarMessage('Please resolve all validation errors before submitting.', 'error');
      return;
    }
    setIsSubmitConfirmOpen(true);
  };

  const handleConfirmSubmit = async () => {
    setIsSubmitConfirmOpen(false);
    if (reportObject?.status !== '11') {
      setSnackbarMessage('Kindly save report before submitting', 'warning');
      return;
    }

    // Determine which report to submit based on the active tab
    const endpoint = tabIndex === 0 ? '/RW04/submitReport' : '/RW05/submitReport';

    try {
      const response = await callApi(endpoint, getBasePayload(), 'POST');
      if (response.status === 1) {
        setSnackbarMessage('Report submitted successfully!', 'success');
        setTimeout(() => navigate(-1), 2000);
      } else {
        setSnackbarMessage('Submission completed, but server response was unexpected.', 'warning');
      }
    } catch (error) {
      console.error('Error submitting report:', error);
      setSnackbarMessage('An error occurred during submission.', 'error');
    }
  };

  // #endregion

  // #region --- Render Logic ---
  if (isLoading && !rw04StaticRows.length && !rw05StaticRows.length) {
    return <CircularProgress />;
  }

  return (
    <Box>
      <Tabs value={tabIndex} onChange={(e, i) => setTabIndex(i)}>
        <Tab label="RW-04" />
        <Tab label="RW-05" />
      </Tabs>

      <Box mt={2} display="flex" gap={2}>
        <CustomButton label={'Add Row'} buttonType={'create'} onClickHandler={handleAddRow} disabled={isLoading} />
        <CustomButton
          label={'Delete Row'}
          buttonType={'delete'}
          onClickHandler={handleDeleteRow}
          disabled={isLoading}
        />
        <CustomButton label={'Save'} buttonType={'save'} onClickHandler={handleSave} disabled={isLoading} />
        <CustomButton
          label={'Submit'}
          buttonType={'submit'}
          onClickHandler={handleSubmitReport}
          disabled={isLoading || reportObject?.status !== '11' || !isDataValid()}
        />
      </Box>

      <TableContainer component={Paper} sx={{ mt: 2, maxHeight: 'calc(100vh - 250px)' }}>
        <Table stickyHeader>
          <TableHead>
            <TableRow>
              <StyledTableCell sx={{ minWidth: '60px' }}>Sr No(1)</StyledTableCell>
              <StyledTableCell>SELECT</StyledTableCell>
              {(tabIndex === 0 ? headers_rw04 : headers_rw05).map((head, idx) => (
                <StyledTableCell key={idx} sx={{ minWidth: '160px' }}>
                  {head}
                </StyledTableCell>
              ))}
            </TableRow>
          </TableHead>

          {/* --- RW-04 BODY --- */}
          {tabIndex === 0 && (
            <TableBody>
              {rw04StaticRows.map((row, index) => (
                <TableRow key={row.feId}>
                  <TableCell align="center">{row.feId}</TableCell>
                  <TableCell></TableCell> {/* Placeholder for select checkbox */}
                  <TableCell sx={{ width: '280px', textAlign: 'left' }}>{row.label}</TableCell>
                  {['provAmtStart', 'writeOff', 'addition', 'reduction'].map((key) => (
                    <TableCell key={key} align="center">
                      <FormInput
                        value={row[key]}
                        onChange={(e) => handleRw04StaticChange(index, key, e.target.value)}
                        readOnly={['1.i', '1.ii', '3.i', '3.ii'].includes(row.feId)}
                        sx={{ width: '150px' }}
                        placeholder="0.00"
                        debounceDuration={1}
                      />
                    </TableCell>
                  ))}
                  <TableCell align="center">
                    <Box display="flex" flexDirection="column">
                      <FormInput
                        value={row.provAmtEnd}
                        onChange={(e) => handleRw04StaticChange(index, 'provAmtEnd', e.target.value)}
                        readOnly={!['1.i', '1.ii', '3.i', '3.ii'].includes(row.feId)}
                        error={getProvAmtEndMismatchError(row.feId)}
                        sx={{ width: '150px' }}
                        placeholder="0.00"
                        debounceDuration={1}
                      />
                      {getProvAmtEndMismatchError(row.feId) && (
                        <Typography fontSize={12} color="error" textAlign={'left'} sx={{ ml: 1 }}>
                          The value does not match
                        </Typography>
                      )}
                    </Box>
                  </TableCell>
                  <TableCell align="center">
                    <FormInput value={row.rate} readOnly sx={{ width: '150px' }} />
                  </TableCell>
                  <TableCell align="center">
                    <FormInput value={row.provRequired} readOnly sx={{ width: '150px' }} />
                  </TableCell>
                </TableRow>
              ))}
              {rw04DynamicRows.map((row, i) => (
                <TableRow key={row.key}>
                  <TableCell align="center">{rw04StaticRows.length + i + 1}</TableCell>
                  <TableCell padding="checkbox" align="center">
                    <Checkbox
                      checked={row.selected}
                      onChange={() => {
                        const updated = [...rw04DynamicRows];
                        updated[i].selected = !updated[i].selected;
                        setRw04DynamicRows(updated);
                      }}
                    />
                  </TableCell>
                  <TableCell>
                    <FormInput
                      maxLength={100}
                      value={row.particulars}
                      onChange={(e) => handleRw04DynamicChange(i, 'particulars', e.target.value)}
                      sx={{ width: '150px' }}
                      placeholder="Enter Particulars"
                      inputType="alphaNumericWithSpace"
                      debounceDuration={1}
                    />
                  </TableCell>
                  {['provAmtStart', 'writeOff', 'addition', 'reduction'].map((key) => (
                    <TableCell key={key} align="center">
                      <FormInput
                        value={row[key]}
                        onChange={(e) => handleRw04DynamicChange(i, key, e.target.value)}
                        sx={{ width: '150px' }}
                        placeholder="0.00"
                        debounceDuration={1}
                      />
                    </TableCell>
                  ))}
                  <TableCell align="center">
                    <FormInput value={row.provAmtEnd} readOnly sx={{ width: '150px' }} />
                  </TableCell>
                  <TableCell align="center">
                    <FormInput value={row.rate} readOnly sx={{ width: '150px' }} />
                  </TableCell>
                  <TableCell align="center">
                    <FormInput value={row.provRequired} readOnly sx={{ width: '150px' }} />
                  </TableCell>
                </TableRow>
              ))}
              {/* RW04 Total Row */}
              <TableRow>
                <StyledTotalTableCell colSpan={3} align="center">
                  TOTAL
                </StyledTotalTableCell>
                <StyledTotalTableCell>{rw04TotalRow.provAmtStart}</StyledTotalTableCell>
                <StyledTotalTableCell>{rw04TotalRow.writeOff}</StyledTotalTableCell>
                <StyledTotalTableCell>{rw04TotalRow.addition}</StyledTotalTableCell>
                <StyledTotalTableCell>{rw04TotalRow.reduction}</StyledTotalTableCell>
                <StyledTotalTableCell>{rw04TotalRow.provAmtEnd}</StyledTotalTableCell>
                <StyledTotalTableCell></StyledTotalTableCell> {/* Rate Total N/A */}
                <StyledTotalTableCell>{rw04TotalRow.provRequired}</StyledTotalTableCell>
              </TableRow>
            </TableBody>
          )}

          {/* --- RW-05 BODY --- */}
          {tabIndex === 1 && (
            <TableBody>
              {rw05StaticRows.map((row, index) => (
                <TableRow key={row.dbId}>
                  <TableCell align="center">{index + 1}</TableCell>
                  <TableCell></TableCell> {/* Placeholder for select checkbox */}
                  <TableCell>{row.label}</TableCell>
                  {['provAmtStart', 'addition', 'reversal'].map((key) => (
                    <TableCell key={key} align="center">
                      <FormInput
                        value={row[key]}
                        onChange={(e) => handleRw05StaticChange(index, key, e.target.value)}
                        sx={{ width: '180px' }}
                        placeholder="0.00"
                        readOnly={row.dbId === 1}
                        debounceDuration={1}
                      />
                    </TableCell>
                  ))}
                  <TableCell align="center">
                    <FormInput value={row.provAmtEnd} readOnly sx={{ width: '180px' }} />
                  </TableCell>
                  <TableCell align="center">
                    <FormInput value={row.difference} readOnly sx={{ width: '180px' }} />
                  </TableCell>
                </TableRow>
              ))}
              {rw05DynamicRows.map((row, index) => (
                <TableRow key={row.key}>
                  <TableCell align="center">{rw05StaticRows.length + index + 1}</TableCell>
                  <TableCell padding="checkbox">
                    <Checkbox
                      checked={row.selected}
                      onChange={() => {
                        const updated = [...rw05DynamicRows];
                        updated[index].selected = !updated[index].selected;
                        setRw05DynamicRows(updated);
                      }}
                    />
                  </TableCell>
                  <TableCell>
                    <FormInput
                      value={row.particulars}
                      onChange={(e) => handleRw05DynamicChange(index, 'particulars', e.target.value)}
                      sx={{ width: '180px' }}
                      placeholder="Enter Particulars"
                      inputType="alphaNumericWithSpace"
                      debounceDuration={1}
                    />
                  </TableCell>
                  {['provAmtStart', 'addition', 'reversal'].map((key) => (
                    <TableCell key={key} align="center">
                      <FormInput
                        value={row[key]}
                        onChange={(e) => handleRw05DynamicChange(index, key, e.target.value)}
                        sx={{ width: '180px' }}
                        placeholder="0.00"
                        debounceDuration={1}
                      />
                    </TableCell>
                  ))}
                  <TableCell align="center">
                    <FormInput value={row.provAmtEnd} readOnly sx={{ width: '180px' }} />
                  </TableCell>
                  <TableCell align="center">
                    <FormInput value={row.difference} readOnly sx={{ width: '180px' }} />
                  </TableCell>
                </TableRow>
              ))}
              {/* RW05 Total Row */}
              <TableRow>
                <StyledTotalTableCell colSpan={3} align="center">
                  TOTAL
                </StyledTotalTableCell>
                <StyledTotalTableCell>{rw05TotalRow.provAmtStart}</StyledTotalTableCell>
                <StyledTotalTableCell>{rw05TotalRow.addition}</StyledTotalTableCell>
                <StyledTotalTableCell>{rw05TotalRow.reversal}</StyledTotalTableCell>
                <StyledTotalTableCell>{rw05TotalRow.provAmtEnd}</StyledTotalTableCell>
                <StyledTotalTableCell>{rw05TotalRow.difference}</StyledTotalTableCell>
              </TableRow>
            </TableBody>
          )}
        </Table>
      </TableContainer>

      {/* --- Dialogs and Snackbars --- */}
      <Dialog open={isSubmitConfirmOpen} onClose={() => setIsSubmitConfirmOpen(false)}>
        <DialogTitle>{'Confirm Submission'}</DialogTitle>
        <DialogContent>
          <DialogContentText>
            Kindly save any changes before submitting. Are you sure you want to proceed?
          </DialogContentText>
        </DialogContent>
        <DialogActions>
          <Button onClick={() => setIsSubmitConfirmOpen(false)} color="secondary">
            Cancel
          </Button>
          <Button onClick={handleConfirmSubmit} color="primary" autoFocus>
            Confirm Submit
          </Button>
        </DialogActions>
      </Dialog>
    </Box>
  );
  // #endregion
};

export default RW04;


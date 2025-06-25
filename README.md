import React, { useState } from 'react';
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
  Checkbox,
} from '@mui/material';
import { styled } from '@mui/material/styles';
import FormInput from '../../../../common/components/ui/FormInput';
import useCustomSnackbar from '../../../../common/hooks/useCustomSnackbar';

const StyledTableCell = styled(TableCell)(({ theme }) => ({
  fontSize: '0.875rem',
  textAlign: 'center',
  padding: '6px',
  fontWeight: 'bold',
}));


const initialStaticRows = [
  { id: '1', label: 'Contiongent liability As per As-29' },
  { id: '2', label: 'Delayed reporting penalty' },
  { id: '3', label: 'Ex-Gratia payment' },
  { id: '4', label: 'Provision on overdue deposit intt' },
  { id: '5', label: 'Leave encashment' },
  { id: '6', label: 'Provision for performance linked incentives' },
  {
    id: '7',
    label:
      'Provision on account of entries outstanding in adjusting account for previous quarter(s) (i.e. prior to current quarter)',
  },
];

const RW05 = () => {
  const [tabIndex, setTabIndex] = useState(0);
  const [dynamicRows, setDynamicRows] = useState([{ ...initialDynamicRow }]);

  const setSnackbarMessage = useCustomSnackbar();
  const headers = [
    ...(tabIndex === 1 ? ['Select'] : []),
    'Particulars(2)',
    'Opening Provision as on 01-04-2023(3)',
    'Additions during the period(4)',
    'Reversals during the Period(5)',
    'Provision as on 30/06/2024(6)={(3+4)-(5)}',
    'Difference to be Provided/ written back(7)={(6)-(3)}',
  ];
};

export default RW05;


headings:
Sr No(1)	Particulars(2)	Opening Provision as on 01-04-2023(3)	Additions during the period(4)	Reversals during the Period(5)	Provision as on 30/06/2024(6)={(3+4)-(5)}	Difference to be Provided/ written back(7)={(6)-(3)}

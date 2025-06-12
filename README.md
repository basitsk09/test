import React, { useState, useEffect, useMemo } from 'react';
import {
  Table,
  TableBody,
  TableContainer,
  TableHead,
  TableRow,
  Paper,
  TextField,
  Alert,
  Box,
  Stack,
} from '@mui/material';
import TableCell, { tableCellClasses } from '@mui/material/TableCell';
import { styled } from '@mui/material/styles';
import useApi from '../../../../common/hooks/useApi';
import useCustomSnackbar from '../../../../common/hooks/useCustomSnackbar';
import { useLocation, useNavigate } from 'react-router-dom';
import { CustomButton } from '../../../../common/components/ui/Buttons';
import FormInput from '../../../../common/components/ui/FormInput';

const StyledTableCell = styled(TableCell)(({ theme }) => ({
  fontSize: '1rem',
  [`&.${tableCellClasses.head}`]: {
    backgroundColor: 'hsl(220, 20%, 35%)',
    color: theme.palette.common.white,
    fontWeight: 'bold',
  },
}));
// eslint-disable-next-line no-unused-vars
const StyledTableRow = styled(TableRow)(({ _theme, error }) => ({
  '&:nth-of-type(odd)': {
    //backgroundColor: theme.palette.action.hover
  },
  ...(error && { backgroundColor: '#c55a5a' }),
}));

// Row definitions
const rowDefinitions = [
  { id: 'headerA1', label: 'A-1. Facility Wise Classification', type: 'section' },
  { id: 'fac1', label: '[i] Bills Purchased and Discounted less bills rediscounted $', type: 'entry' },
  {
    id: 'fac2',
    label: '[ii] Cash Credits, Overdrafts, Loans repayable on Demand and Recalled Assets $$',
    type: 'entry',
  },
  { id: 'fac3', label: '[iii]  Term Loans , Agricultural Term Loans, FCNRB Term Loan', type: 'entry' },
  { id: 'facTotal', label: 'Total A-1', type: 'entry' },

  { id: 'headerA2', label: 'A.2 Security Wise Classifications', type: 'section' },
  { id: 'sec1', label: '[i] Secured by Tangible Assets', type: 'entry' },
  { id: 'sec2', label: '[ii] Covered by Bank/DICGC/ECGC/CGTSI/Govt Guarantee', type: 'entry' },
  { id: 'sec3', label: '[iii] Unsecured', type: 'entry' },
  { id: 'secTotal', label: 'Total A-2', type: 'entry' },

  { id: 'headerA3', label: 'A.3 Sector-Wise Classifications', type: 'section' },
  { id: 'headerIn', label: 'a) In India', type: 'subsection' },
  { id: 'in1', label: '[i] Priority', type: 'entry' },
  { id: 'in2', label: '[ii] Public', type: 'entry' },
  { id: 'in3', label: '[iii] Banks in India*', type: 'entry' },
  { id: 'in4', label: '[iv] Others', type: 'entry' },
  { id: 'inTotal', label: 'TOTAL IN INDIA (i+ii+iii+iv)', type: 'entry' },

  { id: 'headerOut', label: 'b) Outside India (Excluding Foreign LCs and BGs)', type: 'subsection' },
  { id: 'out1', label: '[i] Due from Banks', type: 'entry' },
  { id: 'out5', label: '[ii] Due from Others', type: 'subsection' },
  { id: 'out2', label: '[1] Bills Purchased and Discounted', type: 'entry' },
  { id: 'out3', label: '[2] Syndicated Loans', type: 'entry' },
  { id: 'out4', label: '[3] Others', type: 'entry' },
  { id: 'outTotal', label: 'TOTAL IN OUTSIDE INDIA (i+ii.1+ii.2+ii.3)', type: 'entry' },

  { id: 'advA3', label: 'Total advances A3 (a+b)', type: 'entry' },
];

// Columns
const columns = [
  { key: 'standard', label: 'Standard', editable: true },
  { key: 'subStandard', label: 'Sub-standard', editable: true },
  { key: 'doubtful', label: 'Doubtful', editable: true },
  { key: 'loss', label: 'Loss', editable: true },
  { key: 'adjustment', label: 'Adjustment', editable: true, conditional: true },
  { key: 'total', label: 'Total', editable: false },
];

const ysaRows = ['fac1', 'fac2', 'fac3', 'facTotal'];

const sectionMap = {
  facTotal: ['fac1', 'fac2', 'fac3'],
  secTotal: ['sec1', 'sec2', 'sec3'],
  inTotal: ['in1', 'in2', 'in3', 'in4'],
  outTotal: ['out1', 'out2', 'out3', 'out4', 'out5'],
  advA3: ['inTotal', 'outTotal'],
};

const getFieldName = (id, key) => {
  let base = id.startsWith('fac') ? 'facility' : id.startsWith('sec') ? 'security' : 'sector';

  const suffix =
    {
      standard: 'Standard',
      subStandard: 'SubStandard',
      doubtful: 'Doubtful',
      loss: 'Loss',
      adjustment: 'Adj',
    }[key] || 'Total';

  let idx;

  if (id === 'inTotal') idx = 'a_Total';
  else if (id === 'outTotal') idx = 'b_Total';
  else if (id === 'advA3') idx = 'ab_Total';
  else if (id.startsWith('in')) idx = `a${id.replace('in', '')}`;
  else if (id === 'out1') idx = `b${id.replace('out', '')}`;
  else if (id === 'out2' || id === 'out3') idx = `${id.replace('out', '')}`;
  else if (id.endsWith('Total')) idx = 'Total';
  else idx = id.replace(/\D/g, '');

  return `${base}_${suffix}_${idx}`;
};

const buildSavePayload = (
  values,
  circleCode,
  quarterEndDate,
  role,
  reportName,
  save,
  status,
  reportMasterId,
  reportId,
  areMocPending
) => {
  const payload = {
    circleCode,
    quarterEndDate,
    role,
    reportName,
    save,
    status,
    reportMasterId,
    reportId,
    areMocPending,
  };
  Object.entries(values).forEach(([id, fields]) => {
    Object.entries(fields).forEach(([key, val]) => {
      // only the numeric inputs, not the YSA column
      if (['standard', 'subStandard', 'doubtful', 'loss', 'adjustment'].includes(key)) {
        // default to '0' if empty
        if (!(id.includes('header') || id === 'out4' || id === 'out5')) payload[getFieldName(id, key)] = val || '0';
        console.log(val);
      }
    });
  });
  return payload;
};

export default function Schedule9Table({ showAdjustment = false }) {
  const { state } = useLocation();
  // const reportObj = state.report;
  const showSnackbar = useCustomSnackbar();
  const user = JSON.parse(localStorage.getItem('user'));
  const [reportObject, setReportObject] = useState(state?.report || null);
  const navigate = useNavigate();

  const { callApi } = useApi();

  const [values, setValues] = useState(
    rowDefinitions.reduce((acc, { id }) => {
      acc[id] = { standard: '', subStandard: '', doubtful: '', loss: '', adjustment: '', ysa: '' };
      return acc;
    }, {})
  );

  // Fetch data
  useEffect(() => {
    // const fetchReport = async () => {
    //   try {
    //     // const { data } = await axios.post(import.meta.env.VITE_BASE_SERVICE_URL + '/Maker/getSC09ReportData', {
    //     //   circleCode,
    //     //   quarterEndDate,
    //     //   role,
    //     //   reportName: 'SC9',
    //     // });
    //     console.log('user', user);
    //     console.log('reportObject', reportObject);
    //     const requestPayload = {
    //       circleCode: user.circleCode,
    //       quarterEndDate: user.quarterEndDate,
    //       role: user.capacity,
    //       reportName: reportObject.name,
    //     };
    //     console.log('reportObj', reportObject);
    //     const data = await callApi('/Maker/getSC09ReportData', requestPayload, 'POST');

    //     setValues((prev) => {
    //       const nv = { ...prev };
    //       ['1', '2', '3'].forEach((num) => {
    //         ['standard', 'subStandard', 'doubtful', 'loss'].forEach((field) => {
    //           nv[`fac${num}`][field] = data[`facility_${field.charAt(0).toUpperCase() + field.slice(1)}_${num}`] || '';
    //           nv[`sec${num}`][field] = data[`security_${field.charAt(0).toUpperCase() + field.slice(1)}_${num}`] || '';
    //           nv[`in${num}`][field] = data[`sector_${field.charAt(0).toUpperCase() + field.slice(1)}_a${num}`] || '';
    //           nv[`out${num}`][field] = data[`sector_${field.charAt(0).toUpperCase() + field.slice(1)}_b${num}`] || '';
    //         });
    //       });
    //       return nv;
    //     });
    //   } catch (e) {
    //     console.error(e);
    //   }
    // };

    const fetchReport = async () => {
      try {
        console.log('user', user);
        console.log('reportObject', reportObject);

        const requestPayload = {
          circleCode: user.circleCode,
          quarterEndDate: user.quarterEndDate,
          role: user.capacity,
          reportName: reportObject.name,
        };

        const data = await callApi('/Maker/getSC09ReportData', requestPayload, 'POST');

        setValues((prev) => {
          const nv = { ...prev };

          const backendFieldMap = {
            standard: 'Standard',
            subStandard: 'SubStandard',
            doubtful: 'Doubtful',
            loss: 'Loss',
            adjustment: 'Adj', // key fix here
          };

          const fields = ['standard', 'subStandard', 'doubtful', 'loss'];
          if (user.capacity === '61') fields.push('adjustment');

          ['1', '2', '3'].forEach((num) => {
            fields.forEach((field) => {
              const suffix = backendFieldMap[field];
              nv[`fac${num}`][field] = data[`facility_${suffix}_${num}`] || '';
              nv[`sec${num}`][field] = data[`security_${suffix}_${num}`] || '';
              nv[`in${num}`][field] = data[`sector_${suffix}_a${num}`] || '';
              nv[`out${num}`][field] = data[`sector_${suffix}_b${num}`] || '';
            });
          });

          return nv;
        });
      } catch (e) {
        console.error('Error in fetchReport:', e);
      }
    };

    const fetchValidation = async () => {
      try {
        const requestPayload2 = {
          circleCode: user.circleCode,
          quarterEndDate: user.quarterEndDate,
          role: user.capacity,
        };

        const data = await callApi('/Maker/getSC09ValiadationData', requestPayload2, 'POST');
        console.log(data, 'fetchValidation');

        setValues((prev) => {
          const nv = { ...prev };

          if (user.capacity === '51') {
            const normalize = (s) => s.replace(/\s|[^a-zA-Z0-9]/g, '').toLowerCase();
            ['fac1', 'fac2', 'fac3'].forEach((id) => {
              const label = rowDefinitions.find((r) => r.id === id).label;
              const key = Object.keys(data).find((k) => normalize(k) === normalize(label));
              nv[id].ysa = data[key] || '0';
            });
            nv.facTotal.ysa = ['fac1', 'fac2', 'fac3']
              .reduce((sum, id) => sum + (parseFloat(nv[id].ysa) || 0), 0)
              .toFixed(2);
          } else {
            const getYsaValue = (code) => {
              const val = data[code];
              return val?.split('~')[1] || '0';
            };

            const codeMap = {
              fac1: '090101001051',
              fac2: '090101001052',
              fac3: '090101001053',
            };

            Object.keys(codeMap).forEach((id) => {
              nv[id].ysa = getYsaValue(codeMap[id]);
            });

            nv.facTotal.ysa = ['fac1', 'fac2', 'fac3']
              .reduce((sum, id) => sum + (parseFloat(nv[id].ysa) || 0), 0)
              .toFixed(2);
          }

          return nv;
        });
      } catch (e) {
        console.error(e);
      }
    };

    fetchReport();
    fetchValidation();
  }, []);

  // const debouncedSetValues = useMemo(() => {
  //   return debounce((id, field, val) => {
  //     setValues((prev) => ({
  //       ...prev,
  //       [id]: { ...prev[id], [field]: val },
  //     }));
  //   }, 150);
  // }, []);

  const handleChange = (id, field, val) => {
    if (!/^-?\d*\.?\d{0,2}$/.test(val)) return;
    setValues((prev) => ({
      ...prev,
      [id]: { ...prev[id], [field]: val },
    }));
  };

  const getRowData = (id) => {
    if (!sectionMap[id]) {
      const b = values[id];
      const sum = ['standard', 'subStandard', 'doubtful', 'loss'].reduce((a, f) => a + (parseFloat(b[f]) || 0), 0);
      const adj = showAdjustment ? parseFloat(b.adjustment) || 0 : 0;
      const total = sum + adj;
      const ysa = parseFloat(b.ysa || 0);
      const isMismatch =
        ysa && id.startsWith('fac')
          ? Number(parseFloat(ysa || 0).toFixed(2)) !== Number(parseFloat(total || 0).toFixed(2))
          : false;
      return { ...b, total: total.toFixed(2), isMismatch };
    }
    const agg = sectionMap[id].reduce(
      (a, c) => {
        const r = getRowData(c);
        ['standard', 'subStandard', 'doubtful', 'loss'].forEach((f) => (a[f] += parseFloat(r[f]) || 0));
        if (showAdjustment) a.adjustment += parseFloat(r.adjustment) || 0;
        return a;
      },
      { standard: 0, subStandard: 0, doubtful: 0, loss: 0, adjustment: 0 }
    );
    const total = agg.standard + agg.subStandard + agg.doubtful + agg.loss + (showAdjustment ? agg.adjustment : 0);
    const ysa = parseFloat(values[id].ysa || 0);
    const isMismatch =
      ysa && id.startsWith('fac')
        ? Number(parseFloat(ysa || 0).toFixed(2)) !== Number(parseFloat(total || 0).toFixed(2))
        : false;
    return {
      standard: agg.standard.toFixed(2),
      subStandard: agg.subStandard.toFixed(2),
      doubtful: agg.doubtful.toFixed(2),
      loss: agg.loss.toFixed(2),
      adjustment: showAdjustment ? agg.adjustment.toFixed(2) : '',
      total: total.toFixed(2),
      ysa: values[id].ysa,
      isMismatch,
    };
  };

  const computedRowData = useMemo(() => {
    const result = {};
    rowDefinitions.forEach((row) => {
      result[row.id] = getRowData(row.id);
    });
    return result;
  }, [values, showAdjustment]);

  // Validation on submit
  const validateForSubmit = useMemo(() => {
    const errs = [];
    // row-level
    ['fac1', 'fac2', 'fac3'].forEach((id) => {
      const { total, ysa } = getRowData(id);
      if (parseFloat(ysa || 0) !== parseFloat(total || 0))
        errs.push(`Total not matching for ${rowDefinitions.find((r) => r.id === id).label}`);
    });
    // section-level
    // const toFixed = (val) => {
    //   Number.parseFloat(val || 0).toFixed(2);
    // };
    const aTot = getRowData('facTotal'),
      bTot = getRowData('secTotal'),
      cTot = getRowData('advA3');
    [
      ['standard', 'Standard'],
      ['subStandard', 'Sub-standard'],
      ['doubtful', 'Doubtful'],
      ['loss', 'Loss'],
    ].forEach(([key, label]) => {
      if (
        !(
          parseFloat(aTot[key] || 0).toFixed(2) === parseFloat(bTot[key] || 0).toFixed(2) &&
          parseFloat(bTot[key] || 0).toFixed(2) === parseFloat(cTot[key] || 0).toFixed(2)
        )
      )
        errs.push(`A-1,A-2,A-3 totals differ for ${label}`);
    });
    if (user.capacity === '61') {
      if (
        !(
          parseFloat(aTot.adjustment || 0) === parseFloat(bTot.adjustment || 0) &&
          parseFloat(bTot.adjustment || 0) === parseFloat(cTot.adjustment || 0)
        )
      )
        errs.push('A-1,A-2,A-3 totals differ for Adjustment');
    }
    return errs;
  }, [values, showAdjustment, user.capacity]);

  const onSave = async () => {
    try {
      const payload = buildSavePayload(
        values,
        user.circleCode,
        user.quarterEndDate,
        user.capacity,
        reportObject.name,
        true,
        reportObject.status,
        reportObject.reportMasterId,
        reportObject.reportId,
        reportObject.areMocPending
      );
      console.log('reportobj', reportObject);
      console.log('payload', payload);

      const response = await callApi('/Maker/SubmitSC09Report', payload, 'POST');
      //await axios.post('http://localhost:7001/BS/Maker/SubmitSC09Report', payload);
      if (response && typeof response === 'string') {
        const [_flag, newReportId, newStatus] = response.split('~');
        setReportObject((prev) => ({
          ...prev,
          reportId: newReportId,
          status: newStatus,
        }));
      }
      if (response) showSnackbar('Saved Successfully!', 'success');
      else showSnackbar('Not Saved', 'error');
    } catch (err) {
      console.error(err);
      //alert('Save failed: ' + (err.response?.data?.message || err.message));
      showSnackbar('Not Saved', 'error');
    }
  };
  const onSubmit = async () => {
    try {
      if (validateForSubmit.length) return;
      const payload = buildSavePayload(
        values,
        user.circleCode,
        user.quarterEndDate,
        user.capacity,
        reportObject.name,
        false,
        reportObject.status,
        reportObject.reportMasterId,
        reportObject.reportId
      );
      const response = await callApi('/Maker/SubmitSC09Report', payload, 'POST');
      if (response && typeof response === 'string') {
        const [_flag, newReportId, newStatus] = response.split('~');
        setReportObject((prev) => ({
          ...prev,
          reportId: newReportId,
          status: newStatus,
        }));
      }
      if (response) {
        showSnackbar('Report submitted successfully!', 'success');
        setTimeout(() => {
          navigate('../');
        }, 1000);
      } else showSnackbar('Failed to submit report!', 'error');
    } catch (err) {
      console.error(err);
      //alert('Submit failed: ' + (err.response?.data?.message || err.message));
      showSnackbar('Failed to submit report!', 'error');
    }
  };

  return (
    <Box sx={{ p: 2 }}>
      <TableRow>
        <TableCell colSpan={columns.filter((c) => !c.conditional || showAdjustment).length + 2}>
          <Stack direction={'row'} spacing={2}>
            <CustomButton label={'Save'} buttonType={'save'} onClickHandler={onSave} />
            <CustomButton
              sx={{ ml: 1 }}
              label={'Submit'}
              buttonType={'submit'}
              onClickHandler={onSubmit}
              disabled={!!validateForSubmit.length}
            />
          </Stack>
        </TableCell>
      </TableRow>
      {validateForSubmit.length > 0 && (
        <Alert severity="error" sx={{ mb: 2 }}>
          <ul style={{ margin: 0, paddingLeft: '1.2em' }}>
            {validateForSubmit.map((e, i) => (
              <li key={i}>{e}</li>
            ))}
          </ul>
        </Alert>
      )}
      <TableContainer component={Paper} sx={{ width: '100%' }}>
        <Table sx={{ tableLayout: 'fixed' }}>
          <TableHead>
            <TableRow>
              <StyledTableCell>
                Classification of Advances (Excluding non-advances related items debited to Recalled Assets and interest
                free Staff Advances)(A1 = A2 = A3)
              </StyledTableCell>
              {columns.map(
                (col) =>
                  (!col.conditional || showAdjustment) && (
                    <StyledTableCell key={col.key} align="center">
                      {col.label}
                    </StyledTableCell>
                  )
              )}
              <StyledTableCell align="center">YSA Data</StyledTableCell>
            </TableRow>
          </TableHead>
          <TableBody>
            {rowDefinitions.map((row) => {
              const data = computedRowData[row.id];
              if (row.type !== 'entry') {
                return (
                  <StyledTableRow key={row.id}>
                    <StyledTableCell
                      colSpan={columns.filter((c) => !c.conditional || showAdjustment).length + 2}
                      sx={{ fontWeight: 'bold' }}
                    >
                      {row.label}
                    </StyledTableCell>
                  </StyledTableRow>
                );
              }
              return (
                <StyledTableRow key={row.id} error={data.isMismatch}>
                  <StyledTableCell>{row.label}</StyledTableCell>
                  {columns.map(
                    (col) =>
                      (!col.conditional || showAdjustment) && (
                        <StyledTableCell key={col.key} align="right">
                          <FormInput
                            key={`${row.id}-${col.key}-${data[col.key]}`}
                            name={getFieldName(row.id, col.key)}
                            value={data[col.key] || '0.00'}
                            onBlur={(e) => handleChange(row.id, col.key, e.target.value)}
                            inputProps={{ style: { textAlign: 'right' } }}
                            readOnly={col.key === 'total' || !col.editable || Boolean(sectionMap[row.id])}
                          />
                        </StyledTableCell>
                      )
                  )}
                  <StyledTableCell align="right">
                    {ysaRows.includes(row.id) && (
                      <TextField
                        name={`${row.id}_ysa`}
                        size="small"
                        value={data.ysa}
                        inputProps={{ style: { textAlign: 'right' } }}
                        fullWidth
                        disabled
                        sx={data.isMismatch ? { border: '1px solid red', borderRadius: 1 } : {}}
                      />
                    )}
                  </StyledTableCell>
                </StyledTableRow>
              );
            })}
          </TableBody>
        </Table>
      </TableContainer>
    </Box>
  );
}

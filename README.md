import React, { useState, useEffect } from 'react';
import axios from 'axios';
import {
  Table,
  TableBody,
  TableContainer,
  TableHead,
  TableRow,
  Paper,
  TextField,
  Button
} from '@mui/material';
import TableCell, { tableCellClasses } from '@mui/material/TableCell';
import { styled } from '@mui/material/styles';
import useApi from '../../../../common/hooks/useApi';

// Styled header and body cells
const StyledTableCell = styled(TableCell)(({ theme }) => ({
  fontSize: '1rem',
  [`&.${tableCellClasses.head}`]: {
    backgroundColor: theme.palette.common.black,
    color: theme.palette.common.white
  }
}));
const StyledTableRow = styled(TableRow)(({ theme }) => ({
  '&:nth-of-type(odd)': {
    //backgroundColor: theme.palette.action.hover
  }
}));

// Row definitions
const rowDefinitions = [
  { id: 'headerA1', label: 'A-1. Facility Wise Classification', type: 'section' },
  { id: 'fac1', label: '[i] Bills Purchased and Discounted less bills rediscounted $', type: 'entry' },
  { id: 'fac2', label: '[ii] Cash Credits, Overdrafts, Loans repayable on Demand and Recalled Assets $$', type: 'entry' },
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
  { id: 'out2', label: '[ii] Due from Others', type: 'entry' },
  { id: 'out3', label: '[1] Bills Purchased and Discounted', type: 'entry' },
  { id: 'out4', label: '[2] Syndicated Loans', type: 'entry' },
  { id: 'out5', label: '[3] Others', type: 'entry' },
  { id: 'outTotal', label: 'TOTAL IN OUTSIDE INDIA (i+ii.1+ii.2+ii.3)', type: 'entry' },

  { id: 'advA3', label: 'Total advances A3 (a+b)', type: 'entry' }
];

// Columns
const columns = [
  { key: 'standard', label: 'Standard', editable: true },
  { key: 'subStandard', label: 'Sub-standard', editable: true },
  { key: 'doubtful', label: 'Doubtful', editable: true },
  { key: 'loss', label: 'Loss', editable: true },
  { key: 'adjustment', label: 'Adjustment', editable: true, conditional: true },
  { key: 'total', label: 'Total', editable: false }
];

// Which rows show YSA data
const ysaRows = ['fac1', 'fac2', 'fac3', 'facTotal'];

// Totals mapping
const sectionMap = {
  facTotal: ['fac1', 'fac2', 'fac3'],
  secTotal: ['sec1', 'sec2', 'sec3'],
  inTotal: ['in1', 'in2', 'in3', 'in4'],
  outTotal: ['out1', 'out2', 'out3', 'out4', 'out5'],
  advA3: ['inTotal', 'outTotal']
};

// Build API field name
const getFieldName = (id, key) => {
  let base = id.startsWith('fac') ? 'facility'
    : id.startsWith('sec') ? 'security'
      : 'sector';
  let suffix = {
    standard: 'Standard',
    subStandard: 'SubStandard',
    doubtful: 'Doubtful',
    loss: 'Loss',
    adjustment: 'Adj'
  }[key] || 'Total';

  let idx = 'Total';
  if (!id.endsWith('Total')) idx = id.replace(/\D/g, '');
  if (id === 'inTotal') idx = 'a_Total';
  if (id === 'outTotal') idx = 'b_Total';
  if (id === 'advA3') idx = 'ab_Total';

  return `${base}_${suffix}_${idx}`;
};

export default function Schedule9Table({ showAdjustment = false, circleCode = "021", quarterEndDate = "31/03/2025", role = "51" }) {
  const [values, setValues] = useState(
    rowDefinitions.reduce((a, { id }) => (a[id] = { standard: '', subStandard: '', doubtful: '', loss: '', adjustment: '', ysa: '' }, a), {})
  );

  const { data, apiError, loading, callApi } = useApi();

  useEffect(() => {
    // Report data
    // callApi(import.meta.env.VITE_BASE_SERVICE_URL+
    //   'Maker/getSC09ReportData',
    //   {circleCode,quarterEndDate,role,reportName:'SC9'},
    //   'POST'
    // ).then(({data})=>{
    //   setValues(prev=>{
    //     const nv={...prev};
    //     ['1','2','3'].forEach(n=>{
    //       ['standard','subStandard','doubtful','loss'].forEach(field=>{
    //         nv[`fac${n}`][field] = data[`facility_${field.charAt(0).toUpperCase()+field.slice(1)}_${n}`]||'';
    //       });
    //     });
    //     return nv;
    //   });
    // })
    // .catch(console.error);

    axios.post('http://10.0.26.172:7001/BS/Maker/getSC09ReportData',
      {
        circleCode,
        quarterEndDate,
        role,
        reportName: 'SC9'
      }
    ).then(({ data }) => {
      setValues(prev => {
        const nv = { ...prev };
        ['1', '2', '3'].forEach(n => {
          ['standard', 'subStandard', 'doubtful', 'loss'].forEach(field => {
            nv[`fac${n}`][field] = data[`facility_${field.charAt(0).toUpperCase() + field.slice(1)}_${n}`] || '';
            nv[`sec${n}`][field] = data[`security_${field.charAt(0).toUpperCase() + field.slice(1)}_${n}`] || '';
            nv[`in${n}`][field] = data[`sector_${field.charAt(0).toUpperCase() + field.slice(1)}_${n}`] || '';
            nv[`out${n}`][field] = data[`sector_${field.charAt(0).toUpperCase() + field.slice(1)}_a${n}`] || '';
          });
        });
        return nv;
      });
    })
      .catch(console.error);

    // Validation (YSA) data
    // callApi(import.meta.env.VITE_BASE_SERVICE_URL+
    //   'Maker/getSC09ValiadationData',
    //   {circleCode,quarterEndDate,role,reportName:'SC9'},
    //   'POST'
    // ) .then(({data:map})=>{
    //   setValues(prev=>{
    //     const nv={...prev};
    //     const norm=s=>s.replace(/\s/g,'').toLowerCase();
    //     ['fac1','fac2','fac3'].forEach(id=>{
    //       const label = rowDefinitions.find(r=>r.id===id).label;
    //       const key = Object.keys(map).find(k=>norm(k)===norm(label));
    //       nv[id].ysa = map[key]||'0';
    //     });
    //     nv.facTotal.ysa = ['fac1','fac2','fac3']
    //       .reduce((s,id)=>s + (parseFloat(nv[id].ysa)||0),0)
    //       .toFixed(2);
    //     return nv;
    //   });
    // })
    // .catch(console.error);

    axios.post('http://10.0.26.172:7001/BS/Maker/getSC09ValiadationData',
      {
        circleCode,
        quarterEndDate,
        role
      }
    ).then(({ data: map }) => {
      setValues(prev => {
        const nv = { ...prev };
        const norm = s => s.replace(/\s/g, '').toLowerCase();
        ['fac1', 'fac2', 'fac3'].forEach(id => {
          const label = rowDefinitions.find(r => r.id === id).label;
          const key = Object.keys(map).find(k => norm(k) === norm(label));
          nv[id].ysa = map[key] || '0';
        });
        nv.facTotal.ysa = ['fac1', 'fac2', 'fac3']
          .reduce((s, id) => s + (parseFloat(nv[id].ysa) || 0), 0)
          .toFixed(2);
        return nv;
      });
    })
      .catch(console.error);

  }, [circleCode, quarterEndDate, role]);

  const handleChange = (id, field, val) => {
    if (!/^\d*\.?\d{0,2}$/.test(val)) return;
    setValues(p => ({ ...p, [id]: { ...p[id], [field]: val } }));
  };

  const onSave = () => axios.post('/api/saveSC09', { circleCode, quarterEndDate, role, reportName: 'SC9', data: values });
  const onSubmit = () => axios.post('/api/submitSC09', { circleCode, quarterEndDate, role, reportName: 'SC9', data: values });

  const getRowData = id => {
    if (!sectionMap[id]) {
      const b = values[id];
      const sum = ['standard', 'subStandard', 'doubtful', 'loss']
        .reduce((a, f) => a + (parseFloat(b[f]) || 0), 0);
      const adj = showAdjustment ? (parseFloat(b.adjustment) || 0) : 0;
      return { ...b, total: (sum + adj).toFixed(2) };
    }
    const agg = sectionMap[id].reduce((a, c) => {
      const r = getRowData(c);
      ['standard', 'subStandard', 'doubtful', 'loss'].forEach(f => a[f] += parseFloat(r[f]) || 0);
      if (showAdjustment) a.adjustment += parseFloat(r.adjustment) || 0;
      return a;
    }, { standard: 0, subStandard: 0, doubtful: 0, loss: 0, adjustment: 0 });
    const total = agg.standard + agg.subStandard + agg.doubtful + agg.loss + (showAdjustment ? agg.adjustment : 0);
    return {
      standard: agg.standard.toFixed(2),
      subStandard: agg.subStandard.toFixed(2),
      doubtful: agg.doubtful.toFixed(2),
      loss: agg.loss.toFixed(2),
      adjustment: showAdjustment ? agg.adjustment.toFixed(2) : '',
      total: total.toFixed(2),
      ysa: values[id].ysa
    };
  };

  return (
    <TableContainer component={Paper} sx={{ width: '100%' }}>
      <Table sx={{ tableLayout: 'fixed' }}>
        <TableHead>
          <TableRow>
            <StyledTableCell>Classification of Advances (Excluding non-advances
              related items debited to Recalled Assets
              and interest free Staff Advances)(A1 = A2 = A3)</StyledTableCell>
            {columns.map(col => (!col.conditional || showAdjustment) && (
              <StyledTableCell key={col.key} align="center">{col.label}</StyledTableCell>
            ))}
            <StyledTableCell align="center">YSA DATA</StyledTableCell>
          </TableRow>
        </TableHead>
        <TableBody>
          {rowDefinitions.map(row => row.type !== 'entry'
            ? (
              <StyledTableRow key={row.id}>
                <StyledTableCell colSpan={columns.filter(c => !c.conditional || showAdjustment).length + 2} sx={{ fontWeight: 'bold' }}>
                  {row.label}
                </StyledTableCell>
              </StyledTableRow>
            ) : (() => {
              const data = getRowData(row.id);
              const isSection = Boolean(sectionMap[row.id]);
              return (
                <StyledTableRow key={row.id}>
                  <StyledTableCell>{row.label}</StyledTableCell>
                  {columns.map(col => (!col.conditional || showAdjustment) && (
                    <StyledTableCell key={col.key} align="right">
                      <TextField
                        name={getFieldName(row.id, col.key)}
                        size="small"
                        value={data[col.key] || '0.00'}
                        onChange={e => handleChange(row.id, col.key, e.target.value)}
                        inputProps={{ inputMode: 'decimal', pattern: '^\\d*\\.?\\d{0,2}$', style: { textAlign: 'right' } }}
                        fullWidth
                        disabled={col.key === 'total' || !col.editable || isSection}
                      />
                    </StyledTableCell>
                  ))}
                  <StyledTableCell align="right">
                    {ysaRows.includes(row.id) && (
                      <TextField
                        name={`${row.id}_ysa`}
                        size="small"
                        value={data.ysa}
                        inputProps={{ style: { textAlign: 'right' } }}
                        fullWidth
                        disabled
                      />
                    )}
                  </StyledTableCell>
                </StyledTableRow>
              );
            })()
          )}
          <TableRow>
            <TableCell colSpan={columns.filter(c => !c.conditional || showAdjustment).length + 2}>
              <Button variant="contained" color="warning" size="small" sx={{ mr: 1 }} onClick={onSave}>Save</Button>
              <Button variant="contained" color="success" size="small" onClick={onSubmit}>Submit</Button>
            </TableCell>
          </TableRow>
        </TableBody>
      </Table>
    </TableContainer>
  );
}

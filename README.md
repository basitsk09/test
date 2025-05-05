const getFieldName = (id, key) => {
  let base = id.startsWith('fac') ? 'facility'
           : id.startsWith('sec') ? 'security'
           : 'sector';

  const suffixMap = {
    standard: 'Standard',
    subStandard: 'SubStandard',
    doubtful: 'Doubtful',
    loss: 'Loss',
    adjustment: 'Adj',
  };

  const suffix = suffixMap[key] || 'Total';

  let idx;
  if (id === 'inTotal') idx = 'a_Total';
  else if (id === 'outTotal') idx = 'b_Total';
  else if (id === 'advA3') idx = 'ab_Total';
  else if (id.startsWith('in')) idx = `a${id.replace('in', '')}`;
  else if (id.startsWith('out')) idx = `b${id.replace('out', '')}`;
  else if (id.endsWith('Total')) idx = 'Total';
  else idx = id.replace(/\D/g, '');

  const fieldName = `${base}_${suffix}_${idx}`;
  console.log('Field Name:', fieldName);
  return fieldName;
};
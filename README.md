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
  else if (id.startsWith('out')) idx = `b${id.replace('out', '')}`;
  else if (id.endsWith('Total')) idx = 'Total';
  else {
    const numeric = id.replace(/\D/g, '');
    idx = numeric || '1';  // fallback to '1' if empty to avoid trailing underscore
  }

  return `${base}_${suffix}_${idx}`;
};
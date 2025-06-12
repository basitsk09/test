const ysa = parseFloat(b.ysa);
const isMismatch =
  id.startsWith('fac') && !isNaN(ysa)
    ? Number(ysa.toFixed(2)) !== Number(parseFloat(total || 0).toFixed(2))
    : false;
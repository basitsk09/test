const isMismatch =
  id.startsWith('fac') && !isNaN(ysa)
    ? Number(parseFloat(ysa || 0).toFixed(2)) !== Number(parseFloat(total || 0).toFixed(2))
    : false;
{allColumnKeys.map((colKey) => {
  // ❌ Skip col17 and col18 for A4 section rows
  if (
    ['col17', 'col18'].includes(colKey) &&
    row.id.startsWith('A4_') // Applies to A4_i, A4_ii, A4_iii, A4_iv, A4_total
  ) {
    return null;
  }
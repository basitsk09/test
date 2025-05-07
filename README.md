const VirtualizedRow = React.memo(({ index, style, data }) => {
  const { rows, columns, handleCheckboxChange, handleInputChange } = data;
  const row = rows[index];
  return (
    <TableRow key={row.id || index} style={style}>
      {columns.map((column) => (
        <StyledTableCell key={`${index}-${column.key}`} align={column.align || 'left'}>
          {column.type === 'checkbox' ? (
            <Checkbox checked={row.isDelete} onChange={(event) => handleCheckboxChange(event, index)} />
          ) : column.editable ? (
            <TextField
              value={row[column.key] || ''}
              onChange={(e) => handleInputChange(e, index, column.key)}
              inputProps={{
                style: { textAlign: 'right' },
                maxLength: column.key === 'borrowerName' ? 255 : 18,
              }}
              size="small"
            />
          ) : (
            <TextField
              value={row[column.key] || ''}
              inputProps={{
                style: { textAlign: 'right' },
              }}
              size="small"
              disabled
            />
          )}
        </StyledTableCell>
      ))}
    </TableRow>
  );
});



const handleInputChange = useCallback((event, index, columnKey) => {
  const { value } = event.target;
  setRows((prev) => {
    const updated = [...prev];
    const column = columns.find((col) => col.key === columnKey);
    if (column?.pattern && !column.pattern.test(value)) return prev;
    updated[index] = {
      ...updated[index],
      [columnKey]: value,
      ...(columnKey === 'aggOutStand' || columnKey === 'aggSecurities'
        ? {
            netShortfall: (
              (parseFloat(updated[index].aggOutStand) || 0) -
              (parseFloat(updated[index].aggSecurities) || 0)
            ).toFixed(2),
          }
        : {}),
    };
    return updated;
  });
}, []);




useEffect(() => {
  const timeout = setTimeout(() => {
    let aggOutStandTotal = 0;
    let aggSecuritiesTotal = 0;
    let netShortfallTotal = 0;
    let provisionTotal = 0;
    let balInterestSuspenseAccTotal = 0;

    rows.forEach((row) => {
      aggOutStandTotal += parseFloat(row.aggOutStand) || 0;
      aggSecuritiesTotal += parseFloat(row.aggSecurities) || 0;
      netShortfallTotal += parseFloat(row.netShortfall) || 0;
      provisionTotal += parseFloat(row.provision) || 0;
      balInterestSuspenseAccTotal += parseFloat(row.balInterestSuspenseAcc) || 0;
    });

    setTotals({
      aggOutStandTotal: aggOutStandTotal.toFixed(2),
      aggSecuritiesTotal: aggSecuritiesTotal.toFixed(2),
      netShortfallTotal: netShortfallTotal.toFixed(2),
      provisionTotal: provisionTotal.toFixed(2),
      balInterestSuspenseAccTotal: balInterestSuspenseAccTotal.toFixed(2),
    });
  }, 300);

  return () => clearTimeout(timeout);
}, [rows]);


const addRow = () => {
  setRows((prev) => [...prev, { ...initialRow, id: Date.now() + Math.random() }]);
};


useEffect(() => {
  const payload = {
    circleCode: '021',
    quarterEndDate: '31/03/2025',
  };
  const fetchData = async () => {
    try {
      const response = await callApi('/Maker/getSavedDataNineA', payload, 'POST');
      const enriched = response.map((r, i) => ({ ...r, id: r.id || `${i}-${Date.now()}` }));
      setRows(enriched);
    } catch (error) {
      console.error('Error fetching saved data:', error);
      setSnackbarMessage('Error loading saved data.');
      setSnackbarOpen(true);
    }
  };
  fetchData();
}, []);
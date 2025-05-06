  const calculateTotals = useCallback(() => {
    let aggOutStandTotal = 0;
    let aggSecuritiesTotal = 0;
    let netShortfallTotal = 0;
    let provisionTotal = 0;
    let balInterestSuspenseAccTotal = 0;
    let hasNetShortfallChanged = false;

    const updatedRows = rows.map(row => {
      const aggOut = parseFloat(row.aggOutStand) || 0;
      const aggSec = parseFloat(row.aggSecurities) || 0;
      const newNetShort = (aggOut - aggSec).toFixed(2);
      if (row.netShortfall !== newNetShort) {
        hasNetShortfallChanged = true;
        return { ...row, netShortfall: newNetShort };
      }
      return row;
    });

    if (hasNetShortfallChanged) {
      setRows(updatedRows);
    }

    updatedRows.forEach(row => {
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
  }, [rows, setRows, setTotals]);

  useEffect(() => {
    calculateTotals();
  }, [rows, calculateTotals]);

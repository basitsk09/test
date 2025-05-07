const debounce = (func, delay) => {
  let timeout;
  return (...args) => {
    clearTimeout(timeout);
    timeout = setTimeout(() => func(...args), delay);
  };
};



const debouncedInputChange = useMemo(() =>
  debounce((value, index, columnKey) => {
    setRows((prevRows) => {
      const updated = [...prevRows];
      updated[index][columnKey] = value;
      return updated;
    });
  }, 300), []);



<TextField
  value={row[column.key] || ''}
  onChange={(event) => debouncedInputChange(event.target.value, index, column.key)}
  inputProps={{
    style: { textAlign: 'right' },
    maxLength: column.key === 'borrowerName' ? 255 : 18,
  }}
  size="small"
/>
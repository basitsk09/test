Here’s how to update your code step-by-step to fully implement:


---

1. Make All Header Rows Vertically Sticky (Top)

Update all <StyledTableCell> in the three <TableRow> blocks inside <TableHead>:

First Header Row (Classification of PROVISION)

<StyledTableCell
  rowSpan={3}
  sx={{
    minWidth: '450px',
    position: 'sticky',
    left: 0,
    top: 0,
    zIndex: 1200,
    backgroundColor: '#f5f5f5',
  }}
>

Second Header Row (Column labels)

<StyledTableCell
  key={ch.key}
  sx={{
    position: 'sticky',
    top: 56, // adjust if row height differs
    zIndex: 1100,
    backgroundColor: '#000',
    color: '#fff',
    textAlign: 'center',
    whiteSpace: 'normal',
  }}
  dangerouslySetInnerHTML={{ __html: ch.label }}
/>

Third Header Row (Column Numbers: 1,2,3...)

<StyledTableCell
  key={`colnum_${idx}`}
  sx={{
    position: 'sticky',
    top: 112, // 56px + 56px
    zIndex: 1000,
    backgroundColor: '#000',
    color: '#fff',
    textAlign: 'center',
  }}
>
  <b>{idx + 1}</b>
</StyledTableCell>


---

2. Make Subheaders Horizontally Sticky (e.g. A-1, A-2)

In the <StyledTableRow> for section and subSection headers:

<StyledTableCell
  colSpan={allColumnKeys.length + 1}
  sx={{
    position: 'sticky',
    left: 0,
    zIndex: 100,
    backgroundColor: row.type === 'sectionHeader' ? '#e0e0e0' : '#f0f0f0',
    textAlign: 'left',
    fontWeight: 'bold',
  }}
>


---

3. Fix Input Lag (Debounce)

Replace the current debounce block:

const debouncedSetFormData = useCallback(
  debounce((rowId, fieldKey, value) => {
    setFormData((prev) => ({
      ...prev,
      [rowId]: { ...prev[rowId], [fieldKey]: value },
    }));
  }, 100),
  []
);

With direct setFormData (no debounce):

const handleChange = (rowId, fieldKey, value) => {
  if (
    value === '' ||
    /^-?\d*\.?\d{0,2}$/.test(value) ||
    (value === '-' && !(formData[rowId]?.[fieldKey]?.length > 0))
  ) {
    setFormData((prev) => ({
      ...prev,
      [rowId]: { ...prev[rowId], [fieldKey]: value },
    }));
  }
};


---

These updates will make:

The entire header block stick vertically

The first column (A-1, A-2 etc.) stick horizontally

The numbering stay aligned on scroll

The inputs perform without lag


Let me know if you want react-window or virtualization next.


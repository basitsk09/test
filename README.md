To fix the width of all FormInput components in your Schedule 10 table consistently, regardless of content, apply a fixed sx style inside the FormInput or VirtualizedInput.


---

✅ Update VirtualizedInput to apply fixed width:

Locate this inside your VirtualizedInput component:

<FormInput
  name={name}
  value={value}
  onChange={onChange}
  onBlur={onBlur}
  readOnly={readOnly}
  error={!!error}
  helperText={helperText}
/>

Update it to include a fixed width via customStyles:

<FormInput
  name={name}
  value={value}
  onChange={onChange}
  onBlur={onBlur}
  readOnly={readOnly}
  error={!!error}
  helperText={helperText}
  customStyles={{
    width: '120px',              // ✅ consistent fixed width for all
    '& input': {
      textAlign: 'right',
      padding: '6px 8px',
    },
  }}
/>


---

✅ Optional: Fix height alignment

In VirtualizedInput placeholder too:

<Box
  ref={placeholderRef}
  sx={{
    height: '38px',
    width: '120px',              // ✅ match fixed width here
    textAlign: 'right',
    padding: '6px 8px',
    boxSizing: 'border-box',
  }}
>
  {props.displayValue}
</Box>


---

✅ Result

This change will:

Enforce all cells to have same width (no jitter or shifting)

Make layout smoother and aligned

Keep UI consistent across edit/view mode


Let me know if you want to adjust column widths for headers (StyledTableCell) as well to align precisely.


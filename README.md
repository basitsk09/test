Thanks — if the red border is not showing, it's likely because the FormInput component isn't properly consuming the error or helperText props to trigger MUI's error styling, or your error object is not reaching the component correctly.


---

✅ Fix Checklist

Here’s a full fix you can apply to guarantee the red border appears on invalid fields:


---

1. ✅ Ensure error and helperText Props Are Applied in FormInput

Your VirtualizedInput uses MemoizedFormInput, which in turn uses FormInput.

Find this part in your MemoizedFormInput:

<FormInput
  name={name}
  value={value}
  onChange={onChange}
  onBlur={onBlur}
  readOnly={readOnly}
  error={!!error}
  helperText={helperText}
  customStyles={{
    width: '200px',
    '& input': {
      textAlign: 'right',
      padding: '6px 8px',
    },
  }}
/>

✅ Make sure your FormInput component forwards error and helperText to MUI’s <TextField>. If not, the red border won’t appear.


---

2. ✅ Validate That errors[row29-fieldKey] Has the Right Key

Ensure you're setting error keys like this:

validationErrors['row29-totalA'] = 'Mismatch with precheck';

And then this key must match what's being checked during render:

const errorForField = errors[cellKey]; // cellKey = `${rowKey}-${fieldKey}`

If rowKey is "row29" and fieldKey is "totalA", then cellKey is "row29-totalA" ✅ — so make sure it matches exactly.


---

3. ✅ Force Re-render After Setting Errors

If the red border still doesn’t show after setting setErrors(...), try forcing re-render with a separate useState:

const [errorVersion, setErrorVersion] = useState(0);
...
setErrors((prev) => {
  const updated = { ...prev, ...validationErrors };
  setErrorVersion((v) => v + 1); // force re-render
  return updated;
});

Then add errorVersion as a dependency to anything memoized like MemoizedFormInput or VirtualizedInput if needed.


---

✅ Optional: Debug Quick Patch

Add this just above your return in VirtualizedInput to visually confirm error is received:

if (error) {
  console.log(`❗ Error applied to ${name}: ${helperText}`);
}


---

Let me know if you're using a custom FormInput component — I can help update it if it's missing error handling.


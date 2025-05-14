To make your table body dark mode compatible, especially when using Material UI, follow these best practices:


---

✅ 1. Use theme.palette Instead of Hardcoded Colors

Update your StyledTableCell to respond to light and dark modes automatically:

const StyledTableCell = styled(TableCell)(({ theme }) => ({
  fontSize: '0.875rem',
  padding: '8px',
  border: `1px solid ${theme.palette.divider}`,
  whiteSpace: 'nowrap',
  [`&.${tableCellClasses.head}`]: {
    backgroundColor: theme.palette.background.default,
    color: theme.palette.text.primary,
    fontWeight: 'bold',
    textAlign: 'center',
  },
  [`&.${tableCellClasses.body}`]: {
    color: theme.palette.text.primary,
    backgroundColor: theme.palette.background.paper,
    textAlign: 'right',
  },
}));


---

✅ 2. Update Sticky Column Cell Backgrounds

Sticky cells (like row labels and headers) must explicitly set backgroundColor from theme:

backgroundColor: theme.palette.background.default,

Example:

<StyledTableCell
  sx={{
    position: 'sticky',
    left: 0,
    zIndex: 99,
    backgroundColor: theme => theme.palette.background.default,
    color: theme => theme.palette.text.primary,
    textAlign: 'left',
  }}
>


---

✅ 3. Set TableContainer sx for Dark Compatibility

In your <TableContainer> or <Paper>:

<TableContainer
  component={Paper}
  sx={{
    backgroundColor: (theme) => theme.palette.background.paper,
  }}
>


---

✅ 4. Optional: Alternate Row Striping in Dark Mode

If you want alternate row colors, use:

'&:nth-of-type(odd)': {
  backgroundColor: theme.palette.action.hover,
},

Inside your StyledTableRow.


---

✅ 5. Dark Mode Detection (if needed)

If you need to manually detect dark mode:

const isDarkMode = theme.palette.mode === 'dark';

Use this to conditionally apply colors.


---

Would you like me to rewrite your StyledTableCell and StyledTableRow for full dark mode support right now?


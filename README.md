<TableBody>
  {requests
    .slice(page * rowsPerPage, page * rowsPerPage + rowsPerPage)
    .map((row) => {
      // ... other cells
      return (
        <TableRow>
            {/* ... other cells ... */}

            {/* --- The call happens right here --- */}
            <TableCell align="center">
              <Box component="span" sx={getStatusStyles(row.reqSts)}>
                {row.reqSts}
              </Box>
            </TableCell>

            {/* ... other cells ... */}
        </TableRow>
      );
    })}
</TableBody>

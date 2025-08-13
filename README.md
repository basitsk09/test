const columns = [
  {
    field: 'name',
    headerName: 'Report Name',
    hideable: false,
    flex: 1.5,
  },
  {
    field: 'checkerRemarks',
    headerName: 'Checker Remarks',
    // ... (rest of the column definition is unchanged)
    renderCell: (params) => (
      <span
        style={{
          color: !params.value
            ? theme.palette.grey[500]
            : params.row.status === '53'
            ? theme.palette.grey[500]
            : 'inherit',
        }}
      >
        {!params.value
          ? 'checker remarks will appear here'
          : params.row.status === '53'
          ? 'checker remarks will appear here'
          : params.value}
      </span>
    ),
  },
  {
    field: 'pendingStatus',
    headerName: 'Status',
    // ... (rest of the column definition is unchanged)
    renderCell: (params) => renderStatus(params.value),
  },
  {
    field: 'actions',
    headerName: 'Action',
    headerAlign: 'center',
    align: 'center',
    flex: 2,
    sortable: false,
    filterable: false,
    renderCell: (params) => {
      // ** ADDED THIS LOGIC **
      // This condition is true if it's the RW-04 report and its specific status is not 51.
      const isRw04Locked =
        params.row.reportMasterId === '3007' && params.row.rw04Status !== '51';

      return (
        <Box
          sx={{
            '& .MuiDataGrid-cell:focus': {
              outline: 'none',
            },
            gap: 2,
            display: 'flex',
            alignItems: 'center',
            justifyContent: 'center',
            m: 1,
          }}
        >
          {/* --- EDIT BUTTON LOGIC --- */}
          {user.capacity === '51' && params.row.status && (
            <CustomButton
              onClickHandler={() => handleEdit(params.row)}
              buttonType={'edit'}
              label={'Edit'}
              // ** MODIFIED THIS LINE **
              // Disable if original conditions are met OR if it's the locked RW-04 report.
              disabled={
                !(params.row.status === '10' || params.row.status === '11' || params.row.status === '12') ||
                isRw04Locked
              }
            />
          )}

          {params.row.status && (
            <CustomButton onClickHandler={() => handleView(params.row)} buttonType={'view'} label={'View'} />
          )}

          {/* --- CREATE BUTTON LOGIC --- */}
          {user.capacity === '51' &&
            (params.row.status === null || params.row.status === '' || params.row.status == undefined) && (
              <CustomButton
                onClickHandler={() => handleCreate(params.row)}
                buttonType={'create'}
                label={'Create'}
                // ** ADDED THIS LINE **
                // Disable the create button if it's the locked RW-04 report.
                disabled={isRw04Locked}
              />
            )}

          {/* --- ACCEPT / REJECT BUTTONS (Unchanged) --- */}
          {user.capacity === '52' && (
            <CustomButton
              onClickHandler={() => handleAccept(params.row)}
              buttonType={'accept'}
              label={'Accept'}
              disabled={params.row.status !== '20'}
            />
          )}
          {user.capacity === '52' && (
            <CustomButton
              onClickHandler={() => {
                setSelectedReport(params.row);
                setOpenRejectModal(true);
              }}
              buttonType={'reject'}
              label={'Reject'}
              disabled={params.row.status !== '20' && params.row.status !== '53'}
            />
          )}
        </Box>
      );
    },
  },
];

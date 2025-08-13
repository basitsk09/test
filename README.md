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
      headerAlign: 'center',
      align: 'center',
      flex: 1.5,
      sortable: false,
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
      headerAlign: 'center',
      align: 'center',
      flex: 1.5,
      hideable: false,
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
      renderCell: (params) => (
        <Box
          sx={{
            '& .MuiDataGrid-cell:focus': {
              outline: 'none', // Remove focus outline for cells
            },
            gap: 2,
            display: 'flex',
            alignItems: 'center',
            justifyContent: 'center',
            m: 1,
          }}
        >
          {user.capacity === '51' && params.row.status && (
            <CustomButton
              onClickHandler={() => handleEdit(params.row)}
              buttonType={'edit'}
              label={'Edit'}
              disabled={!(params.row.status === '10' || params.row.status === '11' || params.row.status === '12')}
            />
          )}
          {params.row.status && (
            <CustomButton onClickHandler={() => handleView(params.row)} buttonType={'view'} label={'View'} />
          )}

          {user.capacity === '51' &&
            (params.row.status === null || params.row.status === '' || params.row.status == undefined) && (
              <CustomButton onClickHandler={() => handleCreate(params.row)} buttonType={'create'} label={'Create'} />
            )}

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
                // handleReject(params.row);
                setSelectedReport(params.row);
                setOpenRejectModal(true);
              }}
              buttonType={'reject'}
              label={'Reject'}
              disabled={params.row.status !== '20' && params.row.status !== '53'}
            />
          )}
        </Box>
      ),
    },
  ];

  
  /////////////////////////
  
  {
    "name": "RW-04 PART A & B",
    "jrxmlName": "BS_RW04",
    "cirlceCode": "020",
    "date": null,
    "type": null,
    "reportId": "101885",
    "description": null,
    "reportMasterId": "3007",
    "status": "11",
    "stDesc": null,
    "checkerRemarks": null,
    "auditorRemarks": null,
    "flowType": null,
    "toBeSigned": null,
    "areMocPending": false,
    "releaseFlagDICGC": "N",
    "bidDataFlag": "Y",
    "customFlagDICGC": "N",
    "pdfFilePath": null,
    "pdfFileName": null,
    "pendingStatus": "Saved by Maker",
    "displayMessage": null,
	"rw04Status": "51",
}

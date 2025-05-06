Schedule9ATable.jsx:314 In HTML, <div> cannot be a child of <table>.
This will cause a hydration error.

  ...
    <Box3 sx={{mx:3,pb:5,px:2,width:"...", ...}}>
      <Styled(div) as="div" ref={null} className="MuiBox-root" theme={{...}} sx={{mx:3,pb:5,px:2,width:"...", ...}}>
        <Insertion6>
        <div className="MuiBox-roo...">
          <Typography2>
          <Schedule9ATable>
            <Box3>
              <Styled(div) as="div" ref={null} className="MuiBox-root" theme={{...}} sx={{}}>
                <Insertion6>
                <div className="MuiBox-roo...">
                  <TableContainer2 component={{...}} sx={{width:"100%"}}>
                    <MuiTableContainer-root ref={null} as={{...}} className="MuiTableCo..." ownerState={{...}} ...>
                      <Insertion6>
                      <Paper2 className="MuiTableCo...">
                        <MuiPaper-root as="div" ownerState={{...}} className="MuiPaper-r..." ref={null} style={{...}}>
                          <Insertion6>
                          <div className="MuiPaper-r..." style={{...}}>
                            <Table2>
                              <MuiTable-root as="table" role={null} ref={null} className="MuiTable-root" ...>
                                <Insertion6>
>                               <table role={null} className="MuiTable-root css-o9z8ri-MuiTable-root">
                                  <TableHead2>
                                  <TableHead2>
                                  ...
                                    <MuiTableBody-root className="MuiTableBo..." as={{...}} ref={null} role="rowgroup" ...>
                                      <Insertion6>
                                      <List ref={null} height={0} itemCount={0} itemSize={50} className="MuiTableBo..." ...>
>                                       <div
>                                         className="MuiTableBody-root css-gmh7jj-MuiTableBody-root"
>                                         onScroll={function}
>                                         ref={function}
>                                         style={{position:"relative",height:0,width:undefined,overflow:"auto", ...}}
>                                       >
                                  ...
                  ...

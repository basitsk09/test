import { useEffect, useState } from "react";
import { createTheme, styled, ThemeProvider } from "@mui/material/styles";
import Grid from "@mui/material/Grid";
import IconButton from "@mui/material/IconButton";
import Typography from "@mui/material/Typography";
import axios from "axios";
import { useNavigate } from "react-router-dom";
import Divider from "@mui/material/Divider";
import { Box } from "@mui/system";
import {
  Avatar,
  Card,
  CardContent,
  Chip,
  Menu,
  MenuItem,
  InputBase,
  Button,
} from "@mui/material";
import MoreVertIcon from "@mui/icons-material/MoreVert";
import ListItemIcon from "@mui/material/ListItemIcon";
import CardActions from "@mui/material/CardActions";
import DialogContent from "@mui/material/DialogContent";
import Dialog from "@mui/material/Dialog";
import CircularProgress from "@mui/material/CircularProgress";
import { SnackbarProvider } from "notistack";
import Snackbar from "@mui/material/Snackbar";
import Alert from "@mui/material/Alert";
import VisibilityIcon from "@mui/icons-material/Visibility";
import HighlightOffIcon from "@mui/icons-material/HighlightOff";
import { encrypt } from "../Security/AES-GCM256";
import Paper from "@mui/material/Paper";
import FilterAltIcon from "@mui/icons-material/FilterAlt";
import { Close, DoneAll } from "@mui/icons-material";
import FrtViewModel from "./FrtViewModel";

const iv = crypto.getRandomValues(new Uint8Array(12)); // for encryption
const ivBase64 = btoa(String.fromCharCode.apply(null, iv)); // for be decryption
const salt = crypto.getRandomValues(new Uint8Array(16)); // for encryption
const saltBase64 = btoa(String.fromCharCode.apply(null, salt)); // for be decryption

const defaultTheme = createTheme();

const CardContainer = styled(Card)(({ theme }) => ({
  position: "relative",
  overflow: "visible",
  backgroundColor: "inherit",
  boxShadow: "none",
  //padding: theme.spacing(2),
}));

const IconButtonWrapper = styled("div")({
  position: "absolute",
  top: 0,
  right: 0,
  padding: "8px",
});

const AvatarWrapper = styled("div")({
  display: "flex",
  justifyContent: "center",
  alignItems: "center",
  height: "100%",
});

const FrtRequestActivity = () => {
  document.title = "CRS | Request Status";

  const [requestList, setRequestList] = useState([]);
  const [content, setContent] = useState("Loading . . .");
  const [anchorEl, setAnchorEl] = useState(null);
  const [snackbar, setSnackbar] = useState(null);
  const open = Boolean(anchorEl);
  const [loadOpen, setLoadOpen] = useState(false);
  const [index, setIndex] = useState("");
  const [userData, setUserData] = useState(null);
  const [searchUser, setSearchUser] = useState("");
  const [viewOpen, setViewOpen] = useState(false);
  const [hideCancel, setHideCancel] = useState(false);

  const navigate = useNavigate();

  const loggedInUser = JSON.parse(localStorage.getItem("user"));

  useEffect(() => {
    if (
      loggedInUser.user_role !== "96" &&
      loggedInUser.user_role !== "94" &&
      loggedInUser.user_role !== "50" &&
      loggedInUser.user_role !== "9"
    ) {
      navigate("/"); // logout
    }

    fetchData().then((r) => {
      console.log("data fetched");
    });
  }, []);

  /**
   * This function is to open the sub menu.
   *
   * e : contains the event.
   * i : contains the index of data.
   * data : contains the selected request details.
   *
   * @Author : V1014064
   **/
  const handleClick = (e, i, data) => {
    setAnchorEl(e.currentTarget);
    setIndex(i);
    setUserData(data);
    if (
      loggedInUser.user_role === "50" &&
      loggedInUser.pf_number !== data.REQUESTED_USER
    ) {
      setHideCancel(true);
    }
  };

  /**
   * This function is to close the sub menu.
   *
   * @Author : V1014064
   **/
  const handleClose = () => {
    setAnchorEl(null);
    setIndex("");
    setUserData(null);
    setHideCancel(false);
  };

  /**
   * This function is to open loading.
   *
   * @Author : V1014064
   **/
  const handleLoadOpen = () => {
    setLoadOpen(true);
  };

  /**
   * This function is to close the view modal.
   *
   * @Author : V1014064
   **/
  const handleViewClose = () => {
    setAnchorEl(null);
    setViewOpen(false);
    setHideCancel(false);
  };

  /**
   * This function is for closing the snackbar.
   *
   * @Author : V1014064
   **/
  const handleCloseSnackbar = () => setSnackbar(null);

  /**
   * This function is for closing the loading icon.
   *
   * @Author : V1014064
   **/
  const handleDialogClose = () => setLoadOpen(false);

  /**
   * This function is for opening the view modal to reject/accept request.
   *
   * @Author : V1014064
   **/
  const handleMenuAction = () => {
    setViewOpen(true);
  };

  let filteredUsers = requestList
    .map((row, index) => ({
      ...row,
      originalIndex: index,
    }))
    .filter((row) =>
      row.PF_NUMBER.toString().includes(searchUser.toLowerCase())
    );

  /**
   * This function is axios call for fetching the list of requests from the backend.
   *
   * @Author : V1014064
   **/
  const fetchData = async () => {
    try {
      const response = await axios.post(
        "Server/frtRequestActivityService/fetchData",
        {},
        {
          headers: {
            Authorization: `Bearer ${localStorage.getItem("token")}`,
          },
        }
      );
      setRequestList(response.data.result);
      setContent("No pending requests.");
    } catch (error) {
      console.error("Error fetching data:", error);
    }
  };

  /**
   * This function is axios call to reject/cancel the request.
   *
   * type : type of operation i.e rejection/cancellation.
   *
   * @Author : V1014064
   **/
  const handleAction = async (type) => {
    try {
      let data;
      let statusValue;
      let successMessage;
      let errorMessage;

      if (type === "reject") {
        data = { id: userData.ID, status: "3" };
        statusValue = "3";
        successMessage =
          "Request succesfully rejected for Req Id : " + userData.ID + ".";
        errorMessage =
          "Failed to reject the request for Req Id : " + userData.ID + ".";
      } else {
        data = { id: userData.ID, status: "4" };
        statusValue = "4";
        successMessage =
          "Request succesfully cancelled for Req Id : " + userData.ID + ".";
        errorMessage =
          "Failed to cancel the request for Req Id : " + userData.ID + ".";
      }

      let jsonFormData = JSON.stringify(data);

      await encrypt(iv, salt, jsonFormData).then(function (result) {
        jsonFormData = result;
      });

      let payload = {
        iv: ivBase64,
        salt: saltBase64,
        data: jsonFormData,
      };

      const response = await axios.post(
        "Server/frtRequestActivityService/requestAction",
        payload,
        {
          headers: {
            Authorization: `Bearer ${localStorage.getItem("token")}`,
          },
        }
      );

      if (response.data.statusCode === 200) {
        await fetchData();
        handleViewClose();
        handleDialogClose();
        handleClose();
        setSnackbar({
          children: successMessage,
          severity: "success",
        });
      } else {
        handleDialogClose();
        setSnackbar({
          children: errorMessage,
          severity: "error",
        });
      }
    } catch (e) {
      console.error(e);
      handleDialogClose();
      setSnackbar({
        children: "An error occurred. Please try again.",
        severity: "error",
      });
    }
  };

  /**
   * This function is axios call to accept the request.
   *
   * @Author : V1014064
   **/
  const handleAccept = async () => {
    try {
      let jsonFormData = JSON.stringify({ id: userData.ID, status: "2" });

      await encrypt(iv, salt, jsonFormData).then(function (result) {
        jsonFormData = result;
      });

      let payload = {
        iv: ivBase64,
        salt: saltBase64,
        data: jsonFormData,
      };

      const response = await axios.post(
        "Server/frtRequestActivityService/requestAction",
        payload,
        {
          headers: {
            Authorization: `Bearer ${localStorage.getItem("token")}`,
          },
        }
      );

      if (response.data.result) {
        await fetchData();
        handleViewClose();
        handleDialogClose();
        handleClose();
        setSnackbar({
          children: response.data.message,
          severity: "success",
        });
      } else if (response.data.message) {
        handleDialogClose();
        setSnackbar({
          children: response.data.message,
          severity: "error",
        });
      } else {
        handleDialogClose();
        setSnackbar({
          children: "An error occurred. Please try again.",
          severity: "error",
        });
      }
    } catch (e) {
      console.error(e);
      handleDialogClose();
      setSnackbar({
        children: "An error occurred. Please try again.",
        severity: "error",
      });
    }
  };

  /**
   * This function is axios call to accept all the requests.
   *
   * @Author : V1014064
   **/
  const handleAcceptAll = async () => {
    try {
      let newList = [];
      for(const samp of requestList) {
        if(samp.REQUESTED_USER !== loggedInUser.pf_number) newList.push(samp);
      }

      console.log(newList);

      let successCount = 0;
      let failedCount = 0;

      for(const newSamp of newList) {
        let jsonFormData = JSON.stringify({ id: newSamp.ID, status: "2" });

      await encrypt(iv, salt, jsonFormData).then(function (result) {
        jsonFormData = result;
      });

      let payload = {
        iv: ivBase64,
        salt: saltBase64,
        data: jsonFormData,
      };

      const response = await axios.post(
        "Server/frtRequestActivityService/requestAction",
        payload,
        {
          headers: {
            Authorization: `Bearer ${localStorage.getItem("token")}`,
          },
        }
      );
      if (response.data.result) {
        successCount+=1;
      } else {
        failedCount+=1;
      }
      }
      console.log('newList length :: '+newList.length);
      console.log('success count :: '+successCount);
      console.log('failed count :: '+failedCount);
      if(newList.length === successCount) {
        await fetchData();
        handleViewClose();
        handleDialogClose();
        handleClose();
        setSnackbar({
          children: "All requests succesfully accepted.",
          severity: "success",
        });
      } else {
        handleDialogClose();
        setSnackbar({
          children: failedCount +" number of requests failed to be accepted.",
          severity: "error",
        });
      }

    } catch (e) {
      console.error(e);
      handleDialogClose();
      setSnackbar({
        children: "An error occurred. Please try again.",
        severity: "error",
      });
    }
  };



  return (
    <ThemeProvider theme={defaultTheme}>
      <Box sx={{ display: "flex", flexDirection: "column" }}>
        <Grid container>
          <Typography variant={"h5"}>Request Status</Typography>
        </Grid>
        <Grid container sx={{ justifyContent: "flex-end" }}>
          <Paper
            style={{ textAlign: "center" }}
            component="form"
            sx={{
              p: "2px 4px",
              display: "flex",
              alignItems: "center",
              width: 450,
            }}
          >
            <FilterAltIcon sx={{ ml: 1 }} />
            <Divider sx={{ height: 28, m: 1.0 }} orientation="vertical" />
            <InputBase
              sx={{ ml: 1, flex: 1 }}
              placeholder=" Filter by PF Number"
              inputProps={{ "aria-label": "search Req id" }}
              value={searchUser}
              onChange={(e) => {
                setContent("No data available for the entered req ID");
                setSearchUser(e.target.value);
              }}
            />
            <Divider sx={{ height: 28, m: 1.0 }} orientation="vertical" />
            <IconButton
              type="button"
              sx={{ p: "10px" }}
              aria-label="clear"
              onClick={() => {
                setSearchUser("");
              }}
            >
              <Close />
            </IconButton>
          </Paper>
          &nbsp;
          {requestList.length > 0 && loggedInUser.user_role === "50" && (
            <Button
              onClick={() => {setLoadOpen(true);handleAcceptAll();}}
              variant="contained"
              disableElevation
              color="success"
              startIcon={<DoneAll />}
            >
              Accept All
            </Button>
          )}
        </Grid>
        <br />
        <Divider />
        <Box sx={{ height: "67vh", overflow: "auto", mt: 1 }}>
          <Grid container spacing={2} direction="row">
            {filteredUsers.length > 0 ? (
              <>
                {filteredUsers.map((vd, i) => (
                  <Grid item key={vd.ID}>
                    <Card sx={{ width: 280, height: 280 }}>
                      <CardContent sx={{ textAlign: "center" }}>
                        <CardContainer>
                          {vd.STATUS === "1" && (
                            <IconButtonWrapper>
                              <IconButton
                                value={vd.ID}
                                aria-label="settings"
                                onClick={(e) => handleClick(e, i, vd)}
                                aria-controls={open ? "basic-menu" : undefined}
                                aria-haspopup="true"
                                aria-expanded={open ? "true" : undefined}
                              >
                                <MoreVertIcon />
                              </IconButton>
                            </IconButtonWrapper>
                          )}
                          <CardContent>
                            <AvatarWrapper>
                              <Avatar
                                variant="Large"
                                sx={{
                                  alignItems: "center",
                                  width: 50,
                                  height: 50,
                                  mt: 2,
                                  textAlign: "center",
                                }}
                              />
                            </AvatarWrapper>
                          </CardContent>
                        </CardContainer>

                        <Menu
                          elevation={1}
                          id="account-menu"
                          open={open}
                          anchorEl={anchorEl}
                          onClose={handleClose}
                          transformOrigin={{
                            horizontal: "right",
                            vertical: "top",
                          }}
                          anchorOrigin={{
                            horizontal: "right",
                            vertical: "bottom",
                          }}
                        >
                          {anchorEl && (
                            <MenuItem onClick={handleMenuAction}>
                              <ListItemIcon>
                                <VisibilityIcon fontSize="small" />
                              </ListItemIcon>
                              View Request
                            </MenuItem>
                          )}
                        </Menu>

                        <Typography variant="body1" color="text.secondary">
                          {vd.PF_NUMBER}
                        </Typography>
                        <Typography variant="body2" color="text.secondary">
                          Req id - {vd.ID}
                        </Typography>
                        <Typography variant="body2" color="text.secondary">
                          Requested by - {vd.REQUESTED_USER}
                        </Typography>

                        <Typography
                          noWrap
                          sx={{
                            overflow: "hidden",
                            textOverflow: "ellipsis",
                            maxWidth: "250px",
                          }}
                          style={{
                            textAlign: "center",
                            fontWeight: 600,
                          }}
                        >
                          {vd.MESSAGE}
                        </Typography>
                      </CardContent>
                      <CardActions
                        sx={{ alignItems: "center", justifyContent: "center" }}
                      >
                        <Chip
                          style={{
                            width: "auto",
                            height: 30,
                            alignItems: "center",
                          }}
                          label={
                            vd.STATUS === "1"
                              ? "Pending"
                              : vd.STATUS === "2"
                              ? "Accepted"
                              : vd.STATUS === "3"
                              ? "Rejected"
                              : "Cancelled"
                          }
                          sx={
                            vd.STATUS === "1"
                              ? {
                                  backgroundColor: "#F5BC51",
                                  color: "white",
                                  fontSize: 16,
                                }
                              : vd.STATUS === "2"
                              ? {
                                  backgroundColor: "#51B92D",
                                  color: "white",
                                  fontSize: 16,
                                }
                              : vd.STATUS === "3"
                              ? {
                                  backgroundColor: "#8471FE",
                                  color: "white",
                                  fontSize: 16,
                                }
                              : {
                                  backgroundColor: "#F9805C",
                                  color: "white",
                                  fontSize: 16,
                                }
                          }
                        />
                      </CardActions>
                    </Card>
                  </Grid>
                ))}
              </>
            ) : (
              <Grid item key="message">
                <Typography>
                  <Typography variant="h6">{content}</Typography>
                </Typography>
              </Grid>
            )}
          </Grid>
        </Box>

        {/* Reload button related code */}
        <Dialog
          PaperProps={{
            style: {
              backgroundColor: "transparent",
              boxShadow: "none",
            },
          }}
          open={loadOpen}
          aria-labelledby="alert-dialog-title"
          aria-describedby="alert-dialog-description"
        >
          <Box>
            <DialogContent sx={{ display: "Grid" }}>
              <CircularProgress />
            </DialogContent>
          </Box>
        </Dialog>

        {/* Snackbar realted code */}
        {snackbar && (
          <SnackbarProvider maxSnack={3}>
            <Snackbar
              open
              anchorOrigin={{ vertical: "top", horizontal: "center" }}
              onClose={handleCloseSnackbar}
              autoHideDuration={3000}
            >
              <Alert
                variant="filled"
                {...snackbar}
                onClose={handleCloseSnackbar}
              />
            </Snackbar>
          </SnackbarProvider>
        )}

        {/* View modal */}
        <FrtViewModel
          openViewModal={viewOpen}
          handleAccept={handleAccept}
          handleReject={handleAction}
          userData={userData}
          handleClose={handleViewClose}
          handleLoadOpen={handleLoadOpen}
          role={loggedInUser.user_role}
          hideCancel={hideCancel}
        />
      </Box>
    </ThemeProvider>
  );
};

export default FrtRequestActivity;






//////////////////////////////////////////////////////////////////////////////////////////////
<%@ include file="/views/include.jsp" %>
<!DOCTYPE html>
<html>
<head>

    <script type="text/javascript">


        function viewtrack(a, fileName, req_Id) {
            console.log("before set Action");
            console.log("a-" + a);
            console.log("req_id-" + req_Id);
            console.log("fileName" + fileName);
            var filename = fileName;
            console.log("req_Id" + req_Id);
            console.log("before ajax");
            $.ajax({
                type: "POST",
                url: "./getExcelFile",
                data: {
                    filename: filename,
                    req_Id: req_Id
                },

                success: function (data) {
                    console.log("data" + data);
                    var msg = data;
                    window.location.href = "./downloadXls?url=" + data;
                },
                error: function (jqXHR, textStatus, errorMessage) {
                    alert("No response received");
                },

                complete: function () {
                    $.unblockUI();

                },
            });
        }

        function changeRequest(req_Id, status, a) {
            console.log("req_id: " + req_Id);
            var status1 = document.getElementById("status" + a).value;
            console.log("status: " + status);
            console.log("status1: " + status1);
            $.ajax({
                type: "POST",
                url: "./cancelReq",
                data: {
                    status: status,
                    req_Id: req_Id
                },

                success: function (data) {
                    console.log("data: " + data);
                    if (data) {
                            //status = document.getElementById("status" + a).value = "Cancelled";
                        document.getElementById("cancel" + a).style.display = 'none';
                        document.getElementById("status" + a).innerHTML= "CANCELLED";
                        document.getElementById("status" + a).setAttribute("class",'label label-default');
                        console.log("status: " + document.getElementById("status" + a).value);


                    }
                },
                error: function (jqXHR, textStatus, errorMessage) {
                    alert("No response received");
                },

                complete: function () {
                    $.unblockUI();

                },
            });
        }

        $(document).ready(function () {
            $('#example1').DataTable(
                {
                    "paging": true,
                    "lengthChange": true,
                    "searching": true,
                    "ordering": false,
                    select: true,
                    "info": true,
                    "autoWidth": false
                    /*scrollY: 400*/
                }
            );

            var table = $('#example1').DataTable();
            var filteredData = table
                .columns( [0, 1] )
                .data()
                .flatten()
                .filter( function ( value, index ) {
                    return value > 20 ? true : false;
                } );
        });
    </script>
</head>
<body class="hold-transition skin-blue sidebar-mini">
<div class="wrapper">
    <div class="content-wrapper">
        <section class="content-header">
            <h1>TRACK REQUEST</h1>
        </section>
        <section class="content">
            <div class="row">
                <div class="col-xs-12">
                    <div class="box box-primary box-solid">
                        <div class="box-body">
                            <div class="box-header">
                            </div>
                            <form:form action="submit" method="POST" id="frtTrack" style="overflow-y: auto; height: 70vh;">
                                <input type="hidden" name="strRuleListCount" id="strRuleListCount"
                                       value='<c:out value="${strRuleListCount}"></c:out>'/>
                                <input type="hidden" name="Req_id" id="Req_id"
                                       value='<c:out value="${frtTrack.req_Id}"></c:out>'/>
                                <div class="col-md-12">
                                    <div class="col-md-1"></div>
                                    <div class="col-md-10">
                                        <table id="example1" class="table">
                                            <thead>
                                            <tr style="background-color: #b9def0; height: 40px">
                                                <th style="width: 50px;text-align: center ">Req ID</th>
                                                <th style="text-align: center">Change Request For</th>
                                                <th style="text-align: center">Branch</th>
                                                <th style="width: 80px; text-align: center">Status</th>
                                                <th style="width: 120px; text-align: center">Action</th>
                                                <th style="text-align: center">Requested By</th>
                                            </tr>
                                            </thead>
                                            <tbody>
                                            <c:if test="${strRuleListCount!=0}">
                                                <c:forEach items="${list}" var="frtTrack" varStatus="reportStatus">
                                                    <tr>
                                                        <td style="text-align: center;">
                                                            <input type="hidden" class="form-control"
                                                                   style="width: 50px; background-color: white; border-style: hidden; text-align: center"
                                                                   path="req_Id<c:out value='${reportStatus.count}'></c:out>"
                                                                   name="req_Id" readonly="true"/>
                                                                  <c:out value="${frtTrack.req_Id}"></c:out>
                                                        </td>
                                                        <td style="text-align: center"><c:out
                                                                value="${frtTrack.changeReq_Type}"></c:out></td>
                                                        <td style="text-align: center;">
                                                            <c:if test="${frtTrack.branchCode eq 'Multiple Branches'}">
                                                                <a href="#"
                                                                   onclick="javascript:viewtrack('${reportStatus.count}','${frtTrack.fileName}','${frtTrack.req_Id}');">
                                                                    <u onmouseover=""> Multiple Branches </u>
                                                                </a>
                                                                <input type="hidden" id="filename"
                                                                       value="${frtTrack.fileName}"/>
                                                            </c:if>
                                                            <c:if test="${frtTrack.branchCode != 'Multiple Branches'}">
                                                                <c:out value="${frtTrack.branchCode}"></c:out>
                                                            </c:if>
                                                        </td>
                                                        <td style="text-align: center">
                                                            <c:if test="${frtTrack.status == 'Pending'}">
                                                                <label class="label label-primary"
                                                                       value="<c:out value="${frtTrack.status}"></c:out>"
                                                                       id="status<c:out value='${reportStatus.count}'></c:out>">
                                                                    PENDING
                                                                </label>
                                                            </c:if>
                                                            <c:if test="${frtTrack.status == 'Accepted'}">
                                                                <label class="label label-success"
                                                                       value="<c:out value="${frtTrack.status}"></c:out>"
                                                                       id="status<c:out value='${reportStatus.count}'></c:out>">
                                                                    APPROVED
                                                                </label>
                                                            </c:if>
                                                            <c:if test="${frtTrack.status == 'Rejected'}">
                                                                <label class="label label-danger"
                                                                       value="<c:out value="${frtTrack.status}"></c:out>"
                                                                       id="status<c:out value='${reportStatus.count}'></c:out>">
                                                                    REJECTED
                                                                </label>
                                                            </c:if>
                                                            <c:if test="${frtTrack.status == 'Cancelled'}">
                                                                <label class="label label-default"
                                                                       value="<c:out value="${frtTrack.status}"></c:out>"
                                                                       id="status<c:out value='${reportStatus.count}'></c:out>">
                                                                    CANCELLED
                                                                </label>
                                                            </c:if>
                                                        </td>
                                                        <td style="text-align: center">
                                                            <c:if test="${frtTrack.status == 'Pending'}">
                                                                <button type="button" class="btn btn-danger"
                                                                        id="cancel<c:out value='${reportStatus.count}'></c:out>"
                                                                        onclick="changeRequest('${frtTrack.req_Id}','${frtTrack.status}','${reportStatus.count}');">
                                                                    Cancel Request
                                                                </button>
                                                            </c:if>
                                                        </td>
                                                        <td style="text-align: center"><c:out
                                                                value="${frtTrack.request_maker}"></c:out>
                                                            <input type="hidden" name="rowID"
                                                                   id="rowID<c:out value='${reportStatus.count}'></c:out>"
                                                                   value="${frtTrack.rowUniqueId}"/>
                                                        </td>
                                                    </tr>
                                                </c:forEach>
                                            </c:if>
                                            </tbody>
                                        </table>
                                    </div>
                                    <div class="col-md-1"></div>
                                </div>
                            </form:form>
                        </div>
                        <div class="box-footer"></div>
                    </div>
                </div>
            </div>
        </section>
    </div>
    <div class="example-modal">
        <div class="modal fade" id="infoModal">
            <div class="modal-dialog">
                <div class="modal-content">
                    <div class="modal-header bg-info">
                        <h4 class="modal-title"><b>Attention!</b></h4>
                    </div>
                    <div class="modal-body">
                    </div>
                    <div class="modal-footer">
                        <button type="button" class="btn btn-info" data-dismiss="modal">Close</button>
                    </div>
                </div>
            </div>
        </div>
    </div>
</div>
</body>
</html>




///////////////////////////////////////////////////////////////////////////////////////////////
package com.tcs.dao;

import org.apache.log4j.Logger;
import com.tcs.beans.frtTrack;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.jdbc.core.JdbcTemplate;
import org.springframework.stereotype.Repository;
import java.sql.ResultSet;
import java.sql.SQLException;
import java.util.ArrayList;
import org.springframework.jdbc.core.ResultSetExtractor;

import javax.sql.DataSource;
import java.util.List;
import org.springframework.dao.DataAccessException;
import org.springframework.transaction.PlatformTransactionManager;
import org.springframework.transaction.TransactionDefinition;
import org.springframework.transaction.TransactionStatus;
import org.springframework.transaction.support.DefaultTransactionDefinition;

@Repository("frtTrackDao")

public class frtTrackDaoImpl implements frtTrackDao {


    static Logger log = Logger.getLogger(MakerDaoImpl.class.getName());

    @Autowired
    DataSource dataSource;

    @Autowired
    private JdbcTemplate jdbcTemplateObject;

    @Autowired
    private PlatformTransactionManager transactionManager;

    @Override
    public List<frtTrack> getFrtTrackList(String quarterEndDate) {
        //JdbcTemplate jdbcTemplateObject = new JdbcTemplate(dataSource);


        String query1 = "select RT_ID,RT_TYPE,RT_BRANCH,RT_STATUS,(SELECT NAME from WFL_user where wfl_user_id=rt_maker) " +
                "RT_MAKER,RT_SUBTYPE,RT_FILE_NAME FROM CRS_REQUEST_TRACK WHERE RT_QED= to_date(?,'DD/MM/YYYY') order by RT_STATUS, RT_ID desc";

        List<frtTrack> list = jdbcTemplateObject.query(query1, new Object[]{quarterEndDate}, new ResultSetExtractor<List<frtTrack>>() {
            @Override

            public List<frtTrack> extractData(ResultSet rs1) throws SQLException, DataAccessException {
                List<frtTrack> list = new ArrayList<>();
                frtTrack report = null;
                while (rs1.next()) {
                    frtTrack frtTrack = new frtTrack();
                    frtTrack.setReq_Id(rs1.getString("RT_ID"));
                    String reqType = rs1.getString("RT_TYPE");
                    String reqSubType = rs1.getString("RT_SUBTYPE");
                    if(reqType.equalsIgnoreCase("A")) {
                        if(reqSubType.equalsIgnoreCase("S")) {
                            frtTrack.setChangeReq_Type("Audit Status");
                            frtTrack.setBranchCode(rs1.getString("RT_BRANCH"));
                        }
                        if(reqSubType.equalsIgnoreCase("M")) {
                            frtTrack.setChangeReq_Type("Audit Status");
                            frtTrack.setBranchCode("Multiple Branches");
                        }
                    }

                    else if(reqType.equalsIgnoreCase("B")) {
                        if(reqSubType.equalsIgnoreCase("A")) {
                            frtTrack.setChangeReq_Type("Add Branch");
                            frtTrack.setBranchCode(rs1.getString("RT_BRANCH"));
                        }
                        if(reqSubType.equalsIgnoreCase("D")) {
                            frtTrack.setChangeReq_Type("Delete Branch");
                            frtTrack.setBranchCode(rs1.getString("RT_BRANCH"));
                        }
                    }
                    frtTrack.setFileName(rs1.getString("RT_FILE_NAME"));
                    String status = rs1.getString("RT_STATUS");
                    if(status.equalsIgnoreCase("1")) {
                        frtTrack.setStatus("Pending");
                    }
                    else if(status.equalsIgnoreCase("2")) {
                        frtTrack.setStatus("Accepted");
                    }
                    else if(status.equalsIgnoreCase("3")) {
                        frtTrack.setStatus("Rejected");
                    }
                    else if(status.equalsIgnoreCase("4")) {
                        frtTrack.setStatus("Cancelled");
                    }
                    frtTrack.setRequest_maker(rs1.getString("RT_MAKER"));

                    list.add(frtTrack);
                }

                return list;
            }
        });
        log.info("size of the list: "+list.size());
        return list;

    }

    @Override
    public boolean cancelReq(String req_Id){
        //JdbcTemplate jdbcTemplateObject = new JdbcTemplate(dataSource);
        TransactionDefinition def = new DefaultTransactionDefinition();
        TransactionStatus status = transactionManager.getTransaction(def);

        boolean flag= false;
        List<Object[]> inputList1 = new ArrayList<Object[]>();
        List<Object[]> inputList2 = new ArrayList<Object[]>();
        try {
            String crs_req= "update crs_request_track set rt_status=? where rt_id=?";
            String crs_audit="update CRS_AUDIT_STATUS set AS_REQ_STATUS=? where AS_RT_ID=?";
            //for query crs_req
            Object[] temp1 = {"4", req_Id};
            inputList1.add(temp1);
            //for query crs_audit
            Object[] temp2 = {"R", req_Id};
            inputList2.add(temp2);
            jdbcTemplateObject.batchUpdate(crs_req, inputList1);
            jdbcTemplateObject.batchUpdate(crs_audit, inputList2);
            transactionManager.commit(status);
            flag=true;
            log.info("after update complete");
        } catch (DataAccessException e) {
            log.error("exception " + e);
            flag = false;
            transactionManager.rollback(status);

        }


        return flag;
    }

}

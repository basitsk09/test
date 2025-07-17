import * as React from "react";
import { useState } from "react";
import {
  createTheme,
  styled,
  ThemeProvider,
  useTheme,
} from "@mui/material/styles";
import Box from "@mui/material/Box";
import MuiDrawer from "@mui/material/Drawer";
import MuiAppBar from "@mui/material/AppBar";
import Toolbar from "@mui/material/Toolbar";
import List from "@mui/material/List";
import CssBaseline from "@mui/material/CssBaseline";
import Typography from "@mui/material/Typography";
import Divider from "@mui/material/Divider";
import IconButton from "@mui/material/IconButton";
import ChevronLeftIcon from "@mui/icons-material/ChevronLeft";
import ChevronRightIcon from "@mui/icons-material/ChevronRight";
import ListItem from "@mui/material/ListItem";
import ListItemButton from "@mui/material/ListItemButton";
import ListItemIcon from "@mui/material/ListItemIcon";
import { Help, Home, Logout } from "@mui/icons-material";
import { useNavigate } from "react-router-dom";
import propTypes from "prop-types";
import crs from "../../Asset/img/crsLogo.svg";
import tar from "../../Asset/img/TarLogo.svg";
import lfar from "../../Asset/img/LfarLogo.svg";
import Tooltip from "@mui/material/Tooltip";
import PendingActionsIcon from "@mui/icons-material/PendingActions";
import { userRoles } from "../CommonValidations/commonConstants";
import ChangeCircleTwoToneIcon from "@mui/icons-material/ChangeCircleTwoTone";
import Badge from "@mui/material/Badge";
import {
  Dialog,
  DialogContent,
  DialogTitle,
  Collapse,
} from "@mui/material";
import ChangeModule from "../Login/ChangeModule";
import LayersIcon from "@mui/icons-material/Layers";
import DashboardIcon from "@mui/icons-material/Dashboard";
import ExpandLess from "@mui/icons-material/ExpandLess";
import ExpandMore from "@mui/icons-material/ExpandMore";
import KeyboardArrowDownIcon from "@mui/icons-material/KeyboardArrowDown";
import KeyboardArrowUpIcon from "@mui/icons-material/KeyboardArrowUp";
import CircleSharpIcon from "@mui/icons-material/CircleSharp";
import GroupIcon from "@mui/icons-material/Group";

const drawerWidth = 240;
const defaultTheme = createTheme();

const openedMixin = (theme) => ({
  width: drawerWidth,
  transition: theme.transitions.create("width", {
    easing: theme.transitions.easing.sharp,
    duration: theme.transitions.duration.enteringScreen,
  }),
  overflowX: "hidden",
}); [span_0](start_span)[span_1](start_span)//[span_0](end_span)[span_1](end_span)

const closedMixin = (theme) => ({
  transition: theme.transitions.create("width", {
    easing: theme.transitions.easing.sharp,
    duration: theme.transitions.duration.leavingScreen,
  }),
  overflowX: "hidden",
  width: `calc(${theme.spacing(7)} + 1px)`,
  [theme.breakpoints.up("sm")]: {
    width: `calc(${theme.spacing(8)} + 1px)`,
  },
}); [span_2](start_span)[span_3](start_span)//[span_2](end_span)[span_3](end_span)

const DrawerHeader = styled("div")(({ theme }) => ({
  display: "flex",
  alignItems: "center",
  justifyContent: "flex-end",
  padding: theme.spacing(0, 1),
  // necessary for content to be below app bar
  ...theme.mixins.toolbar,
})); [span_4](start_span)[span_5](start_span)//[span_4](end_span)[span_5](end_span)

const AppBar = styled(MuiAppBar, {
  shouldForwardProp: (prop) => prop !== "open",
})(({ theme, open }) => ({
  zIndex: theme.zIndex.drawer + 1,
  transition: theme.transitions.create(["width", "margin"], {
    easing: theme.transitions.easing.sharp,
    duration: theme.transitions.duration.leavingScreen,
  }),
  ...(open && {
    marginLeft: drawerWidth,
    width: `calc(100% - ${drawerWidth}px)`,
    transition: theme.transitions.create(["width", "margin"], {
      easing: theme.transitions.easing.sharp,
      duration: theme.transitions.duration.enteringScreen,
    }),
  }),
})); [span_6](start_span)[span_7](start_span)//[span_6](end_span)[span_7](end_span)

const Drawer = styled(MuiDrawer, {
  shouldForwardProp: (prop) => prop !== "open",
})(({ theme, open }) => ({
  width: drawerWidth,
  flexShrink: 0,
  whiteSpace: "nowrap",
  boxSizing: "border-box",
  ...(open && {
    ...openedMixin(theme),
    "& .MuiDrawer-paper": openedMixin(theme),
  }),
  ...(!open && {
    ...closedMixin(theme),
    "& .MuiDrawer-paper": closedMixin(theme),
  }),
})); [span_8](start_span)[span_9](start_span)//[span_8](end_span)[span_9](end_span)

const FrtCheckerLayout = ({ children }) => {
  document.title = "CRS | FRT Checker Home"; [span_10](start_span)//[span_10](end_span)
  const navigate = useNavigate(); [span_11](start_span)//[span_11](end_span)
  const theme = useTheme(); [span_12](start_span)[span_13](start_span)//[span_12](end_span)[span_13](end_span)
  const [open, setOpen] = useState(false); [span_14](start_span)[span_15](start_span)//[span_14](end_span)[span_15](end_span)
  const [changeModuleModalOpen, setChangeModuleModalOpen] = useState(false); [span_16](start_span)[span_17](start_span)//[span_16](end_span)[span_17](end_span)
  const user = JSON.parse(localStorage.getItem("user")); [span_18](start_span)[span_19](start_span)//[span_18](end_span)[span_19](end_span)
  const [requestsMenuOpen, setRequestsMenuOpen] = useState(false);

  const moduleLogo = {
    CRS: crs,
    TAR: tar,
    LFAR: lfar,
  }; [span_20](start_span)[span_21](start_span)//[span_20](end_span)[span_21](end_span)

  const handleListClick = (vd) => {
    if (vd.type === "treeview") {
      if (vd.id === "View Requests") {
        setRequestsMenuOpen(!requestsMenuOpen);
        setOpen(true);
      }
      return;
    }

    if (vd.id === "Home") {
      navigate("/FrtCheckerHome"); [span_22](start_span)//[span_22](end_span)
    } else if (vd.id === "View Branch Details") {
      navigate("/FRTCheckerBranchDetails"); [span_23](start_span)//[span_23](end_span)
    } else if (vd.id === "Branch Auditors Details") {
      navigate("/FRTCheckerAuditorsDetails");
    } else if (vd.id === "Dashboard") {
      navigate("/FrtCheckerDashboard"); [span_24](start_span)//[span_24](end_span)
    } else if (vd.id === "Help") {
      navigate("/FrtCheckerHelp"); [span_25](start_span)//[span_25](end_span)
    }
  };

  const handleLogout = () => {
    localStorage.clear(); [span_26](start_span)//[span_26](end_span)
    navigate("/"); [span_27](start_span)//[span_27](end_span)
  };

  const handleDrawerClose = () => {
    setOpen(false); [span_28](start_span)//[span_28](end_span)
    setRequestsMenuOpen(false);
  };

  let data = [
    [span_29](start_span){ id: "Home", element: <Home sx={{ color: "#7f8c8d" }} /> }, //[span_29](end_span)
    {
      id: "View Requests",
      type: "treeview",
      isOpen: requestsMenuOpen,
      [span_30](start_span)[span_31](start_span)element: <PendingActionsIcon sx={{ color: "#7f8c8d" }} />, //[span_30](end_span)[span_31](end_span)
      children: [
        {
          id: "Audit Status Change",
          path: "/FRTChecker/FRTAuditStatusReq",
          icon: (
            <CircleSharpIcon sx={{ fontSize: "0.6rem", color: "#7f8c8d" }} />
          ),
        [span_32](start_span)}, //[span_32](end_span)
        {
          id: "Add/Delete Branch",
          path: "/FRTChecker/FRTBranchReq",
          icon: (
            <CircleSharpIcon sx={{ fontSize: "0.6rem", color: "#7f8c8d" }} />
          ),
        [span_33](start_span)}, //[span_33](end_span)
      ],
    },
    {
      id: "View Branch Details",
      element: <LayersIcon sx={{ color: "#7f8c8d" }} />,
    [span_34](start_span)[span_35](start_span)}, //[span_34](end_span)[span_35](end_span)
    {
      id: "Branch Auditors Details",
      element: <GroupIcon sx={{ color: "#7f8c8d" }} />,
    [span_36](start_span)}, //[span_36](end_span)
    [span_37](start_span){ id: "Dashboard", element: <DashboardIcon sx={{ color: "#7f8c8d" }} /> }, //[span_37](end_span)
    [span_38](start_span){ id: "Help", element: <Help sx={{ color: "#7f8c8d" }} /> }, //[span_38](end_span)
  ];

  return (
    <ThemeProvider theme={defaultTheme}>
      <Box sx={{ display: "flex" }}>
        <CssBaseline />
        <AppBar
          position="fixed"
          open={open}
          sx={{
            background:
              [span_39](start_span)[span_40](start_span)"linear-gradient(90deg, rgba(135,210,247,1) 0%, rgba(212,225,233,1) 75%, rgba(255,255,255,1) 100%)", //[span_39](end_span)[span_40](end_span)
            [span_41](start_span)[span_42](start_span)color: "#000", //[span_41](end_span)[span_42](end_span)
            [span_43](start_span)[span_44](start_span)zIndex: (theme) => theme.zIndex.drawer + 1, //[span_43](end_span)[span_44](end_span)
          }}
        >
          <Toolbar sx={{ display: "flex", alignItems: "center" }}>
            <Badge
              overlap="circular"
              badgeContent={
                <ChangeCircleTwoToneIcon
                  sx={{
                    color: "inherit",
                    background: "white",
                    [span_45](start_span)[span_46](start_span)borderRadius: "60px", //[span_45](end_span)[span_46](end_span)
                  }}
                />
              }
              sx={{ width: "56px", cursor: "pointer" }}
              anchorOrigin={{
                vertical: "bottom",
                [span_47](start_span)[span_48](start_span)horizontal: "right", //[span_47](end_span)[span_48](end_span)
              }}
              onClick={() => setChangeModuleModalOpen(true)}
            >
              <img
                src={moduleLogo[user.module]}
                alt="MODULE"
                width="60"
                height="60"
                style={{ float: "left", marginRight: "50px" }}
              />
            </Badge>
            <Box sx={{ ml: "auto", mt: 1, position: "fixed", right: "20px" }}>
              <Typography variant={"subtitle2"} gutterBottom>
                User: {user.pf_number} - {userRoles[user.user_role]}
              </Typography>
              <Tooltip title={`${user.branch_code} - ${user.branch_name}`}>
                <Typography
                  noWrap
                  sx={{ overflow: "hidden", textOverflow: "ellipsis" }}
                  variant="subtitle2"
                >
                  Branch: {user.branch_code} -{" "}
                  {user.branch_name.length > 15
                    ? user.branch_name.slice(0, 15) + "..."
                    : user.branch_name}
                </Typography>
              </Tooltip>
            </Box>
          </Toolbar>
        </AppBar>
        <Drawer variant="permanent" open={open}>
          <DrawerHeader>
            <IconButton onClick={handleDrawerClose}>
              {theme.direction === "rtl" ? (
                <ChevronRightIcon />
              ) : (
                <ChevronLeftIcon />
              )}
            </IconButton>
          </DrawerHeader>
          <Divider />
          <List
            sx={{ display: "flex", flexDirection: "column", height: "100%" }}
          >
            {data.map((vd) => (
              <React.Fragment key={vd.id}>
                {vd.type === "treeview" ? (
                  <ListItem
                    disablePadding
                    sx={{
                      display: "block",
                      flexDirection: "column",
                    }}
                  >
                    <Tooltip title={vd.id} placement="right">
                      <Box
                        onClick={() => handleListClick(vd)}
                        sx={{
                          display: "flex",
                          flexDirection: "column",
                          alignItems: "center",
                          justifyContent: "center",
                          cursor: "pointer",
                        }}
                      >
                        <ListItemButton
                          sx={{ justifyContent: "center", width: "100%" }}
                        >
                          {vd.element}
                        </ListItemButton>
                        <Typography
                          variant="caption"
                          sx={{
                            textWrap: "wrap",
                            textAlign: "center",
                            display: "flex",
                            alignItems: "center",
                          }}
                        >
                          {open || vd.id.length <= 15
                            ? vd.id
                            : vd.id.slice(0, 15) + "..."}
                          {open &&
                            (vd.isOpen ? (
                              <ExpandLess sx={{ fontSize: "0.2rem" }} />
                            ) : (
                              <ExpandMore sx={{ fontSize: "0.2rem" }} />
                            ))}
                        </Typography>
                        {open &&
                          (vd.isOpen ? (
                            <KeyboardArrowUpIcon sx={{ color: "#7f8c8d" }} />
                          ) : (
                            <KeyboardArrowDownIcon sx={{ color: "#7f8c8d" }} />
                          ))}
                      </Box>
                    </Tooltip>
                    <Collapse
                      in={vd.isOpen && open}
                      timeout="auto"
                      unmountOnExit
                    >
                      {vd.children.map((child) => (
                        <Tooltip
                          key={child.id}
                          title={child.id}
                          placement="right"
                        >
                          <ListItemButton
                            sx={{ pl: 2, justifyContent: "center" }}
                            onClick={() => navigate(child.path)}
                          >
                            {child.icon}
                            <Typography sx={{ ml: 1, fontSize: "0.8rem" }}>
                              {child.id}
                            </Typography>
                          </ListItemButton>
                        </Tooltip>
                      ))}
                    </Collapse>
                  </ListItem>
                ) : (
                  <Tooltip key={`title-${vd.id}`} title={vd.id}>
                    <ListItem
                      disablePadding
                      onClick={() => handleListClick(vd)}
                      sx={{
                        display: "flex",
                        flexDirection: "column",
                        alignItems: "center",
                        justifyContent: "center",
                      }}
                    >
                      <ListItemButton sx={{ alignItems: "center" }}>
                        {vd.element}
                      </ListItemButton>
                      <Typography
                        variant="caption"
                        sx={{
                          textWrap: "wrap",
                          textAlign: "center",
                          cursor: "pointer",
                        }}
                      >
                        {open || vd.id.length <= 15
                          ? vd.id
                          : vd.id.slice(0, 15) + "..."}
                      </Typography>
                    </ListItem>
                  </Tooltip>
                )}
              </React.Fragment>
            ))}
            <Box sx={{ flexGrow: 1 }} />
            <Tooltip title="Logout">
              <ListItem
                disablePadding
                sx={{ display: "block", marginTop: "auto" }}
                onClick={handleLogout}
              >
                <ListItemButton
                  sx={{
                    minHeight: 48,
                    display: "flex",
                    justifyContent: "center",
                    flexDirection: "column",
                    alignItems: "center",
                    px: 2.5,
                  }}
                >
                  <Logout sx={{ minWidth: 0, m: 1, color: "red" }} />
                  <Typography variant="caption">Logout</Typography>
                </ListItemButton>
              </ListItem>
            </Tooltip>
          </List>
        </Drawer>
        <Box
          component="main"
          sx={{
            backgroundColor: "#E1E6EC",
            flexGrow: 1,
            p: 3,
            height: "100vh",
            overflow: "auto",
          }}
        >
          <DrawerHeader />
          {children}
        </Box>
        <Box
          sx={{
            position: "fixed",
            bottom: 0,
            width: "100%",
            backgroundColor: "white",
            textAlign: "center",
            minHeight: 56,
            padding: "15px 0",
            boxShadow: "0 -2px 5px rgba(0,0,0,0.1)",
          }}
        >
          <Box
            variant="body2"
            sx={{
              display: "flex",
              alignItems: "center",
              justifyContent: "center",
            }}
          >
            &copy;{new Date().getFullYear()} &nbsp; Tata Consultancy Services.
            All rights reserved.
          </Box>
        </Box>
      </Box>
      <ModuleChange
        changeModuleModalOpen={changeModuleModalOpen}
        setChangeModuleModalOpen={setChangeModuleModalOpen}
      />
    </ThemeProvider>
  );
};

const ModuleChange = ({ changeModuleModalOpen, setChangeModuleModalOpen }) => {
  return (
    <Dialog
      sx={{ display: "flex", alignItems: "center", justifyContent: "center" }}
      maxWidth="lg"
      open={changeModuleModalOpen}
      onClose={() => setChangeModuleModalOpen(false)}
    >
      <DialogTitle
        id="responsive-dialog-title"
        sx={{
          display: "flex",
          alignItems: "center",
          justifyContent: "center",
        }}
      >
        <Typography variant={"h5"}>Kindly select the Module</Typography>
      </DialogTitle>
      <Divider />
      <DialogContent>
        <ChangeModule handleClose={() => setChangeModuleModalOpen(false)} />
      </DialogContent>
    </Dialog>
  );
}; [span_49](start_span)[span_50](start_span)//[span_49](end_span)[span_50](end_span)

FrtCheckerLayout.propTypes = {
  children: propTypes.element.isRequired,
}; [span_51](start_span)//[span_51](end_span)

export default FrtCheckerLayout;

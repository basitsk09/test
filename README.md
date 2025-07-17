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
});

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
});

const DrawerHeader = styled("div")(({ theme }) => ({
  display: "flex",
  alignItems: "center",
  justifyContent: "flex-end",
  padding: theme.spacing(0, 1),
  ...theme.mixins.toolbar,
}));

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
}));

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
}));

const FrtCheckerLayout = ({ children }) => {
  document.title = "CRS | FRT Checker Home";
  const navigate = useNavigate();
  const theme = useTheme();
  const [open, setOpen] = useState(false);
  const [changeModuleModalOpen, setChangeModuleModalOpen] = useState(false);
  const user = JSON.parse(localStorage.getItem("user"));
  const [requestsMenuOpen, setRequestsMenuOpen] = useState(false);

  const moduleLogo = {
    CRS: crs,
    TAR: tar,
    LFAR: lfar,
  };

  const handleListClick = (vd) => {
    if (vd.type === "treeview") {
      if (vd.id === "View Requests") {
        setRequestsMenuOpen(!requestsMenuOpen);
        setOpen(true);
      }
      return;
    }

    if (vd.id === "Home") {
      navigate("/FrtCheckerHome");
    } else if (vd.id === "View Branch Details") {
      navigate("/FRTCheckerBranchDetails");
    } else if (vd.id === "Branch Auditors Details") {
      navigate("/FRTCheckerAuditorsDetails");
    } else if (vd.id === "Dashboard") {
      navigate("/FrtCheckerDashboard");
    } else if (vd.id === "Help") {
      navigate("/FrtCheckerHelp");
    }
  };

  const handleLogout = () => {
    localStorage.clear();
    navigate("/");
  };

  const handleDrawerClose = () => {
    setOpen(false);
    setRequestsMenuOpen(false);
  };

  let data = [
    { id: "Home", element: <Home sx={{ color: "#7f8c8d" }} /> },
    {
      id: "View Requests",
      type: "treeview",
      isOpen: requestsMenuOpen,
      element: <PendingActionsIcon sx={{ color: "#7f8c8d" }} />,
      children: [
        {
          id: "Audit Status Change",
          path: "/FRTChecker/FRTAuditStatusReq",
          icon: (
            <CircleSharpIcon sx={{ fontSize: "0.6rem", color: "#7f8c8d" }} />
          ),
        },
        {
          id: "Add/Delete Branch",
          path: "/FRTChecker/FRTBranchReq",
          icon: (
            <CircleSharpIcon sx={{ fontSize: "0.6rem", color: "#7f8c8d" }} />
          ),
        },
      ],
    },
    {
      id: "View Branch Details",
      element: <LayersIcon sx={{ color: "#7f8c8d" }} />,
    },
    {
      id: "Branch Auditors Details",
      element: <GroupIcon sx={{ color: "#7f8c8d" }} />,
    },
    { id: "Dashboard", element: <DashboardIcon sx={{ color: "#7f8c8d" }} /> },
    { id: "Help", element: <Help sx={{ color: "#7f8c8d" }} /> },
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
              "linear-gradient(90deg, rgba(135,210,247,1) 0%, rgba(212,225,233,1) 75%, rgba(255,255,255,1) 100%)",
            color: "#000",
            zIndex: (theme) => theme.zIndex.drawer + 1,
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
                    borderRadius: "60px",
                  }}
                />
              }
              sx={{ width: "56px", cursor: "pointer" }}
              anchorOrigin={{
                vertical: "bottom",
                horizontal: "right",
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
};

FrtCheckerLayout.propTypes = {
  children: propTypes.element.isRequired,
};

export default FrtCheckerLayout;

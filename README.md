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
import ListItemText from "@mui/material/ListItemText";
import { Help, Home, Logout } from "@mui/icons-material";
import { useNavigate } from "react-router-dom";
import propTypes from "prop-types";
import crs from "../../Asset/img/crsLogo.svg";
import tar from "../../Asset/img/TarLogo.svg";
import lfar from "../../Asset/img/LfarLogo.svg";
import Tooltip from "@mui/material/Tooltip";
import GroupIcon from "@mui/icons-material/Group";
import PendingActionsIcon from "@mui/icons-material/PendingActions";
import { userRoles } from "../CommonValidations/commonConstants";
import ChangeCircleTwoToneIcon from "@mui/icons-material/ChangeCircleTwoTone";
import Badge from "@mui/material/Badge";
import { Dialog, DialogContent, DialogTitle, Collapse } from "@mui/material";
import ChangeModule from "../Login/ChangeModule";
import LayersIcon from "@mui/icons-material/Layers";
import DashboardIcon from "@mui/icons-material/Dashboard";
import FolderOpenIcon from '@mui/icons-material/FolderOpen';
import ExpandLess from '@mui/icons-material/ExpandLess';
import ExpandMore from '@mui/icons-material/ExpandMore';
import CircleOutlinedIcon from '@mui/icons-material/CircleOutlined';

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

const FrtMakerLayout = ({ children }) => {
  document.title = "CRS | FRT Maker Home";
  const navigate = useNavigate();
  const theme = useTheme();
  const [open, setOpen] = useState(false);
  const [changeModuleModalOpen, setChangeModuleModalOpen] = useState(false);
  const user = JSON.parse(localStorage.getItem("user"));
  
  const [auditMenuOpen, setAuditMenuOpen] = useState(false);
  const [scopeMenuOpen, setScopeMenuOpen] = useState(false);

  const moduleLogo = {
    CRS: crs,
    TAR: tar,
    LFAR: lfar,
  };

  const handleListClick = (vd, data) => {
    // Treeview items are handled by their own onClick handlers
    if (vd.type === 'treeview') {
        if (vd.id === 'Change Audit Status') setAuditMenuOpen(!auditMenuOpen);
        if (vd.id === 'CRS Scope') setScopeMenuOpen(!scopeMenuOpen);
        return;
    }

    // Handle navigation for simple items
    if (vd.id === "Home") {
      navigate("/FrtMakerHome");
    } else if (vd.id === "Branch Details") {
      navigate("/FRTMakerBranchDetails");
    } else if (vd.id === "User Module") {
      navigate("/FrtMakerUserModule");
    } else if (vd.id === "Request Status") {
      navigate("/FrtMakerRequestActivity");
    } else if (vd.id === "Dashboard") {
      navigate("/FrtMakerDashboard");
    } else if(vd.id === "Help"){
      navigate("/FrtMakerHelp");
    }
  };

  const handleLogout = () => {
    localStorage.clear();
    navigate("/");
  };

  const handleDrawerClose = () => {
    setOpen(false);
  };

  // Restructured data array
  let data = [
    {
      id: "Change Audit Status",
      type: 'treeview',
      isOpen: auditMenuOpen,
      icon: <FolderOpenIcon sx={{ color: "#7f8c8d" }} />,
      children: [
        { id: "Single Branch", path: "/FRTUser/singleBranch" },
        { id: "Multiple Branch", path: "/FRTUser/bulkUpload" },
      ],
    },
    {
      id: "CRS Scope",
      type: 'treeview',
      isOpen: scopeMenuOpen,
      icon: <FolderOpenIcon sx={{ color: "#7f8c8d" }} />,
      children: [
        { id: "Add Branch", path: "/FRTUser/branchStatus" },
        { id: "Delete Branch", path: "/FRTUser/deleteBranch" },
      ],
    },
    { id: "Home", element: <Home sx={{ color: "#7f8c8d" }} /> },
    { id: "Branch Details", element: <LayersIcon sx={{ color: "#7f8c8d" }} /> },
    { id: "User Module", element: <GroupIcon sx={{ color: "#7f8c8d" }} /> },
    {
      id: "Request Status",
      element: <PendingActionsIcon sx={{ color: "#7f8c8d" }} />,
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
            {data.map((vd) => {
              if (vd.type === 'treeview') {
                return (
                  <React.Fragment key={vd.id}>
                    <ListItemButton onClick={() => handleListClick(vd)}>
                      <ListItemIcon>{vd.icon}</ListItemIcon>
                      <ListItemText primary={vd.id} sx={{ opacity: open ? 1 : 0 }} />
                      {vd.isOpen ? <ExpandLess /> : <ExpandMore />}
                    </ListItemButton>
                    <Collapse in={vd.isOpen && open} timeout="auto" unmountOnExit>
                      <List component="div" disablePadding>
                        {vd.children.map(child => (
                           <ListItemButton key={child.id} sx={{ pl: 4 }} onClick={() => navigate(child.path)}>
                             <ListItemIcon><CircleOutlinedIcon sx={{fontSize: '0.8rem'}} /></ListItemIcon>
                             <ListItemText primary={child.id} />
                           </ListItemButton>
                        ))}
                      </List>
                    </Collapse>
                  </React.Fragment>
                );
              } else {
                return (
                  <Tooltip key={`title-${vd.id}`} title={vd.id} placement="right">
                    <ListItem
                      key={vd.id}
                      disablePadding
                      onClick={() => handleListClick(vd, data)}
                      sx={{
                        display: "block",
                      }}
                    >
                      <ListItemButton
                        sx={{
                          minHeight: 48,
                          justifyContent: open ? 'initial' : 'center',
                          px: 2.5,
                        }}
                      >
                        <ListItemIcon
                            sx={{
                                minWidth: 0,
                                mr: open ? 3 : 'auto',
                                justifyContent: 'center',
                            }}
                        >
                            {vd.element}
                        </ListItemIcon>
                        <ListItemText primary={vd.id} sx={{ opacity: open ? 1 : 0 }} />
                      </ListItemButton>
                    </ListItem>
                  </Tooltip>
                );
              }
            })}

            <Box sx={{ flexGrow: 1 }} /> 
            
            <Tooltip title="Logout">
              <ListItem
                disablePadding
                sx={{ display: "block" }}
                onClick={handleLogout}
              >
                <ListItemButton
                  sx={{
                    minHeight: 48,
                    justifyContent: open ? 'initial' : 'center',
                    px: 2.5,
                  }}
                >
                  <ListItemIcon
                    sx={{
                      minWidth: 0,
                      mr: open ? 3 : 'auto',
                      justifyContent: 'center',
                      color: "red",
                    }}
                  >
                    <Logout />
                  </ListItemIcon>
                   <ListItemText primary="Logout" sx={{ opacity: open ? 1 : 0, color: 'red' }} />
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
      ></ModuleChange>
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

FrtMakerLayout.propTypes = {
  children: propTypes.element.isRequired,
};

export default FrtMakerLayout;

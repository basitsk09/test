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
import FolderOpenIcon from "@mui/icons-material/FolderOpen";
import ExpandLess from "@mui/icons-material/ExpandLess";
import ExpandMore from "@mui/icons-material/ExpandMore";
import CircleOutlinedIcon from "@mui/icons-material/CircleOutlined";

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
  // necessary for content to be below app bar
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
  const [open, setOpen] = useState(true); // Set drawer to be open by default
  const [changeModuleModalOpen, setChangeModuleModalOpen] = useState(false);
  const user = JSON.parse(localStorage.getItem("user"));

  const [auditMenuOpen, setAuditMenuOpen] = useState(false);
  const [scopeMenuOpen, setScopeMenuOpen] = useState(false);

  const moduleLogo = {
    CRS: crs,
    TAR: tar,
    LFAR: lfar,
  };

  const handleListClick = (vd) => {
    // Toggling state for treeview items
    if (vd.type === "treeview") {
      if (vd.id === "Change Audit Status") setAuditMenuOpen(!auditMenuOpen);
      if (vd.id === "CRS Scope") setScopeMenuOpen(!scopeMenuOpen);
      return;
    }

    // Navigation for simple items
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
    } else if (vd.id === "Help") {
      navigate("/FrtMakerHelp");
    }
  };

  const handleLogout = () => {
    localStorage.clear();
    navigate("/");
  };
  
  const handleDrawerOpen = () => {
    setOpen(true);
  };

  const handleDrawerClose = () => {
    setOpen(false);
  };

  let data = [
    {
      id: "Change Audit Status",
      type: "treeview",
      isOpen: auditMenuOpen,
      element: <FolderOpenIcon />,
      children: [
        {
          id: "Single Branch",
          path: "/FRTUser/singleBranch",
          icon: <CircleOutlinedIcon sx={{ fontSize: "0.8rem" }} />,
        },
        {
          id: "Multiple Branch",
          path: "/FRTUser/bulkUpload",
          icon: <CircleOutlinedIcon sx={{ fontSize: "0.8rem" }} />,
        },
      ],
    },
    {
      id: "CRS Scope",
      type: "treeview",
      isOpen: scopeMenuOpen,
      element: <FolderOpenIcon />,
      children: [
        {
          id: "Add Branch",
          path: "/FRTUser/branchStatus",
          icon: <CircleOutlinedIcon sx={{ fontSize: "0.8rem" }} />,
        },
        {
          id: "Delete Branch",
          path: "/FRTUser/deleteBranch",
          icon: <CircleOutlinedIcon sx={{ fontSize: "0.8rem" }} />,
        },
      ],
    },
    { id: "Home", element: <Home /> },
    { id: "Branch Details", element: <LayersIcon /> },
    { id: "User Module", element: <GroupIcon /> },
    { id: "Request Status", element: <PendingActionsIcon /> },
    { id: "Dashboard", element: <DashboardIcon /> },
    { id: "Help", element: <Help /> },
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
          }}
        >
          <Toolbar sx={{ display: "flex", alignItems: "center" }}>
             <IconButton
              color="inherit"
              aria-label="open drawer"
              onClick={handleDrawerOpen}
              edge="start"
              sx={{
                marginRight: 5,
                ...(open && { display: 'none' }),
              }}
            >
               <Badge
                overlap="circular"
                badgeContent={<ChangeCircleTwoToneIcon sx={{color: "inherit", background: "white", borderRadius: "60px"}}/>}
                sx={{ cursor: "pointer" }}
                anchorOrigin={{vertical: "bottom", horizontal: "right"}}
                onClick={(e) => { e.stopPropagation(); setChangeModuleModalOpen(true);}}
              >
                <img src={moduleLogo[user.module]} alt="MODULE" width="50" height="50"/>
              </Badge>
            </IconButton>

            <Typography variant="h6" noWrap component="div" sx={{ flexGrow: 1, ...(open && { display: 'none' }) }}>
                FRT Maker
            </Typography>

            <Box sx={{ ml: "auto", mt: 1, position: "fixed", right: "20px" }}>
              <Typography variant={"subtitle2"} gutterBottom>
                User: {user.pf_number} - {userRoles[user.user_role]}
              </Typography>
              <Tooltip title={`${user.branch_code} - ${user.branch_name}`}>
                <Typography noWrap sx={{ overflow: "hidden", textOverflow: "ellipsis", maxWidth: '200px' }} variant="subtitle2">
                  Branch: {user.branch_code} - {user.branch_name}
                </Typography>
              </Tooltip>
            </Box>
          </Toolbar>
        </AppBar>
        <Drawer variant="permanent" open={open}>
          <DrawerHeader>
            <IconButton onClick={handleDrawerClose}>
              <ChevronLeftIcon />
            </IconButton>
          </DrawerHeader>
          <Divider />
          <List sx={{ color: '#595959' }}>
            {data.map((vd) => (
              <div key={vd.id}>
                {vd.type === "treeview" ? (
                  <>
                    <ListItemButton onClick={() => handleListClick(vd)}>
                      <ListItemIcon sx={{ color: 'inherit' }}>{vd.element}</ListItemIcon>
                      <ListItemText primary={vd.id} />
                      {vd.isOpen ? <ExpandLess /> : <ExpandMore />}
                    </ListItemButton>
                    <Collapse in={vd.isOpen} timeout="auto" unmountOnExit>
                      <List component="div" disablePadding>
                        {vd.children.map((child) => (
                          <ListItemButton key={child.id} sx={{ pl: 4 }} onClick={() => navigate(child.path)}>
                            <ListItemIcon sx={{ color: 'inherit' }}>{child.icon}</ListItemIcon>
                            <ListItemText primary={child.id} />
                          </ListItemButton>
                        ))}
                      </List>
                    </Collapse>
                  </>
                ) : (
                  <Tooltip title={vd.id} placement="right">
                    <ListItemButton
                      onClick={() => handleListClick(vd)}
                      sx={{ minHeight: 48, justifyContent: open ? "initial" : "center", px: 2.5 }}
                    >
                      <ListItemIcon sx={{ minWidth: 0, mr: open ? 3 : "auto", justifyContent: "center", color: 'inherit' }}>
                        {vd.element}
                      </ListItemIcon>
                      <ListItemText primary={vd.id} sx={{ opacity: open ? 1 : 0 }} />
                    </ListItemButton>
                  </Tooltip>
                )}
              </div>
            ))}
          </List>
          <Box sx={{ flexGrow: 1 }} />
            <Tooltip title="Logout" placement="right">
                <ListItemButton onClick={handleLogout} sx={{
                  minHeight: 48,
                  justifyContent: open ? 'initial' : 'center',
                  px: 2.5,
                  color: 'red'
                }}>
                    <ListItemIcon sx={{ minWidth: 0, mr: open ? 3 : 'auto', justifyContent: 'center', color: 'inherit' }}>
                        <Logout />
                    </ListItemIcon>
                    <ListItemText primary="Logout" sx={{ opacity: open ? 1 : 0 }}/>
                </ListItemButton>
            </Tooltip>
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
            py: 2,
            boxShadow: "0 -2px 5px rgba(0,0,0,0.1)",
          }}
        >
          <Typography variant="body2" color="text.secondary">
            &copy;{new Date().getFullYear()} Tata Consultancy Services. All rights reserved.
          </Typography>
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
      open={changeModuleModalOpen}
      onClose={() => setChangeModuleModalOpen(false)}
    >
      <DialogTitle>Kindly select the Module</DialogTitle>
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

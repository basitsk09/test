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
import { Logout } from "@mui/icons-material";
import { useNavigate } from "react-router-dom";
import propTypes from "prop-types";
import crs from "../../Asset/img/crsLogo.svg";
import tar from "../../Asset/img/TarLogo.svg";
import lfar from "../../Asset/img/LfarLogo.svg";
import Tooltip from "@mui/material/Tooltip";
import { userRoles } from "../CommonValidations/commonConstants";
import ChangeCircleTwoToneIcon from "@mui/icons-material/ChangeCircleTwoTone";
import Badge from "@mui/material/Badge";
import { Dialog, DialogContent, DialogTitle, Collapse } from "@mui/material";
import ChangeModule from "../Login/ChangeModule";
import FolderOpenIcon from '@mui/icons-material/FolderOpen';
import ExpandLess from '@mui/icons-material/ExpandLess';
import ExpandMore from '@mui/icons-material/ExpandMore';
import CircleOutlineIcon from '@mui/icons-material/CircleOutlined';
import TrackChangesIcon from '@mui/icons-material/TrackChanges';
import PeopleIcon from '@mui/icons-material/People';
import BookIcon from '@mui/icons-material/Book';

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
  const user = JSON.parse(localStorage.getItem("user")) || { userName: 'Guest' }; // Fallback for user

  // State for collapsible menus
  const [auditMenuOpen, setAuditMenuOpen] = useState(false);
  const [scopeMenuOpen, setScopeMenuOpen] = useState(false);

  const moduleLogo = {
    CRS: crs,
    TAR: tar,
    LFAR: lfar,
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

  return (
    <ThemeProvider theme={defaultTheme}>
      <Box sx={{ display: "flex" }}>
        <CssBaseline />

        <AppBar position="fixed" open={open} sx={{
            background: "linear-gradient(90deg, rgba(135,210,247,1) 0%, rgba(212,225,233,1) 75%, rgba(255,255,255,1) 100%)",
            color: "#000",
        }}>
          <Toolbar>
            <IconButton color="inherit" aria-label="open drawer" onClick={handleDrawerOpen} edge="start" sx={{ mr: 2, ...(open && { display: 'none' }) }}>
                 <img src={moduleLogo[user.module]} alt="MODULE" width="50" height="50" />
            </IconButton>
            <Typography variant="h6" noWrap component="div" sx={{ flexGrow: 1 }}>
                FRT Maker
            </Typography>
            <Box sx={{ ml: "auto", mt: 1, position: "absolute", right: "20px" }}>
              <Typography variant={"subtitle2"} gutterBottom>
                User: {user.pf_number} - {userRoles[user.user_role]}
              </Typography>
              <Tooltip title={`${user.branch_code} - ${user.branch_name}`}>
                <Typography noWrap sx={{ overflow: "hidden", textOverflow: "ellipsis", maxWidth: '200px' }} variant="subtitle2" >
                  Branch: {user.branch_code} - {user.branch_name}
                </Typography>
              </Tooltip>
            </Box>
          </Toolbar>
        </AppBar>
        <Drawer variant="permanent" open={open}>
          <DrawerHeader>
            <div style={{ display: 'flex', alignItems: 'center', width: '100%', paddingLeft: '16px' }}>
                <img src={'/logo.svg'} className="img-circle" alt="User Image" style={{ width: '40px', height: '40px' }}/>
                <div style={{ marginLeft: '10px' }}>
                    <p style={{ margin: 0, fontWeight: 'bold' }}>{user.userName}</p>
                    <a href="#" style={{ textDecoration: 'none', color: 'inherit' }}>
                        <i className="fa fa-circle text-success" style={{ color: 'green', fontSize: '10px', marginRight: '5px' }}></i> Online
                    </a>
                </div>
            </div>
            <IconButton onClick={handleDrawerClose}>
              {theme.direction === "rtl" ? <ChevronRightIcon /> : <ChevronLeftIcon />}
            </IconButton>
          </DrawerHeader>
          <Divider />

          {/* Integrated Menu */}
          <List>
            <ListItem sx={{ py: 0.5, color: 'grey' }}>
                <ListItemText primaryTypographyProps={{ style: { fontWeight: 'bold' } }}>FRT MAKER</ListItemText>
            </ListItem>

            {/* Change Audit Status */}
            <ListItemButton onClick={() => setAuditMenuOpen(!auditMenuOpen)}>
              <ListItemIcon><FolderOpenIcon /></ListItemIcon>
              <ListItemText primary="Change Audit Status" />
              {auditMenuOpen ? <ExpandLess /> : <ExpandMore />}
            </ListItemButton>
            <Collapse in={auditMenuOpen} timeout="auto" unmountOnExit>
              <List component="div" disablePadding>
                <ListItemButton sx={{ pl: 4 }} onClick={() => navigate('/FRTUser/singleBranch')}>
                  <ListItemIcon><CircleOutlineIcon sx={{ fontSize: '1rem' }} /></ListItemIcon>
                  <ListItemText primary="Single Branch" />
                </ListItemButton>
                <ListItemButton sx={{ pl: 4 }} onClick={() => navigate('/FRTUser/bulkUpload')}>
                  <ListItemIcon><CircleOutlineIcon sx={{ fontSize: '1rem' }} /></ListItemIcon>
                  <ListItemText primary="Multiple Branch" />
                </ListItemButton>
              </List>
            </Collapse>

            {/* CRS Scope */}
            <ListItemButton onClick={() => setScopeMenuOpen(!scopeMenuOpen)}>
              <ListItemIcon><FolderOpenIcon /></ListItemIcon>
              <ListItemText primary="CRS Scope" />
              {scopeMenuOpen ? <ExpandLess /> : <ExpandMore />}
            </ListItemButton>
            <Collapse in={scopeMenuOpen} timeout="auto" unmountOnExit>
              <List component="div" disablePadding>
                <ListItemButton sx={{ pl: 4 }} onClick={() => navigate('/FRTUser/branchStatus')}>
                  <ListItemIcon><CircleOutlineIcon sx={{ fontSize: '1rem' }} /></ListItemIcon>
                  <ListItemText primary="Add Branch" />
                </ListItemButton>
                <ListItemButton sx={{ pl: 4 }} onClick={() => navigate('/FRTUser/deleteBranch')}>
                  <ListItemIcon><CircleOutlineIcon sx={{ fontSize: '1rem' }} /></ListItemIcon>
                  <ListItemText primary="Delete Branch" />
                </ListItemButton>
              </List>
            </Collapse>
            
            {/* Other Links */}
            <ListItemButton onClick={() => navigate('/FRTUser/frtTrack')}>
                <ListItemIcon><TrackChangesIcon /></ListItemIcon>
                <ListItemText primary="Track Request" />
            </ListItemButton>
            <ListItemButton onClick={() => navigate('/FRTUser/FRTAuditorsDetails')}>
                <ListItemIcon><PeopleIcon /></ListItemIcon>
                <ListItemText primary="Branch Auditors Details" />
            </ListItemButton>
            <ListItemButton onClick={() => navigate('/Security/downloadDocsFAQ')}>
                <ListItemIcon><BookIcon /></ListItemIcon>
                <ListItemText primary="F.A.Q" />
            </ListItemButton>
          </List>
          
          <Divider />
          {/* Common Menu Content would be integrated here or as separate components */}

          <List sx={{ mt: 'auto' }}>
            <Tooltip title="Logout">
              <ListItem disablePadding sx={{ display: "block" }} onClick={handleLogout}>
                <ListItemButton sx={{ minHeight: 48, justifyContent: open ? "initial" : "center", px: 2.5 }}>
                  <ListItemIcon sx={{ minWidth: 0, mr: open ? 3 : "auto", justifyContent: "center", color: 'red' }}>
                    <Logout />
                  </ListItemIcon>
                  <ListItemText primary="Logout" sx={{ opacity: open ? 1 : 0 }} />
                </ListItemButton>
              </ListItem>
            </Tooltip>
          </List>
        </Drawer>
        <Box component="main" sx={{ backgroundColor: "#E1E6EC", flexGrow: 1, p: 3, height: "100vh", overflow: "auto" }}>
          <DrawerHeader />
          {children}
        </Box>
        <Box sx={{ position: "fixed", bottom: 0, width: "100%", backgroundColor: "white", textAlign: "center", py: 2, boxShadow: "0 -2px 5px rgba(0,0,0,0.1)" }}>
          <Box variant="body2" sx={{ display: "flex", alignItems: "center", justifyContent: "center" }}>
            &copy;{new Date().getFullYear()} &nbsp; Tata Consultancy Services. All rights reserved.
          </Box>
        </Box>
      </Box>
      <ModuleChange changeModuleModalOpen={changeModuleModalOpen} setChangeModuleModalOpen={setChangeModuleModalOpen} />
    </ThemeProvider>
  );
};

const ModuleChange = ({ changeModuleModalOpen, setChangeModuleModalOpen }) => {
  return (
    <Dialog sx={{ display: "flex", alignItems: "center", justifyContent: "center" }} maxWidth="lg" open={changeModuleModalOpen} onClose={() => setChangeModuleModalOpen(false)} >
      <DialogTitle id="responsive-dialog-title" sx={{ display: "flex", alignItems: "center", justifyContent: "center" }} >
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


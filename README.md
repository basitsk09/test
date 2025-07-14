It's not possible to directly "integrate" a JSP file into a React JS application in the way you might embed one React component within another. They are fundamentally different technologies that run in different environments:
 * JSP (JavaServer Pages): Renders on the server-side. The Java code is executed on the server, and the final HTML is sent to the browser.
 * React JS: Renders on the client-side. JavaScript code runs in the user's browser to manipulate the DOM and display the user interface.
Therefore, you can't just take the JSP code and have it run within your React component. The browser doesn't understand JSP tags like <%@ include %> or <c:out>.
The correct approach is to recreate the JSP's functionality and appearance as a React component.
Based on the code you've provided, you have a JSP-based sidebar and a modern React-based application layout (FrtMakerLayout). The goal is to make the JSP sidebar's links and features available within the React application.
Here is the full code that translates the JSP sidebar logic into your FrtMakerLayout React component.
Analysis of the JSP Sidebar
First, let's break down what the JSP sidebar does:
 * Displays a user panel: Shows a logo, the user's name from the session (sessionScope.userName), and an "Online" status.
 * Provides navigation links:
   * A "Change Audit Status" dropdown with "Single Branch" and "Multiple Branch" links.
   * A "CRS Scope" dropdown with "Add Branch" and "Delete Branch" links.
   * A "Track Request" link.
   * A "Branch Auditors Details" link.
   * An "F.A.Q" link.
 * Includes common content: Uses <jsp:include page="/views/commonMenuContent.jsp">, which likely contains more menu items.
Recreating the Sidebar in FrtMakerLayout.js
To achieve the same result in React, you need to add these navigation elements to your FrtMakerLayout component. We'll use Material-UI's List, ListItem, Collapse for the dropdowns, and icons to match the style.
Here is the complete, integrated FrtMakerLayout.js file that replicates the JSP's functionality.
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
import CircleIcon from '@mui/icons-material/Circle';
import TrackChangesIcon from '@mui/icons-material/TrackChanges';
import AssignmentIndIcon from '@mui/icons-material/AssignmentInd';
import BookIcon from '@mui/icons-material/Book';

const drawerWidth = 240;

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
  
  // State for controlling the collapsible menus
  const [auditMenuOpen, setAuditMenuOpen] = useState(false);
  const [scopeMenuOpen, setScopeMenuOpen] = useState(false);

  // Get user data from localStorage
  const user = JSON.parse(localStorage.getItem("user"));
  
  // NOTE: You must have user data in localStorage for this to work.
  // In a real app, you would handle the case where user is null (e.g., redirect to login).
  if (!user) {
     // For demonstration, using placeholder data if user is not in localStorage
     // navigate('/'); // Or redirect to login
     return <div>User not found. Redirecting to login...</div>;
  }

  const moduleLogo = {
    CRS: crs,
    TAR: tar,
    LFAR: lfar,
  };

  const handleDrawerToggle = () => {
    setOpen(!open);
  };

  const handleDrawerClose = () => {
    setOpen(false);
  };

  const handleLogout = () => {
    localStorage.clear();
    navigate("/");
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
            <IconButton
              color="inherit"
              aria-label="open drawer"
              onClick={handleDrawerToggle}
              edge="start"
              sx={{
                marginRight: 5,
                ...(open && { display: 'none' }),
              }}
            >
                <img
                    src={moduleLogo[user.module]}
                    alt="MODULE"
                    width="40"
                    height="40"
                />
            </IconButton>
            <Badge
              overlap="circular"
              badgeContent={
                <ChangeCircleTwoToneIcon sx={{ color: "inherit", background: "white", borderRadius: "60px" }} />
              }
              sx={{ width: "56px", cursor: "pointer", display: open ? 'none' : 'block' }} // Hide when drawer is open
              anchorOrigin={{ vertical: "bottom", horizontal: "right" }}
              onClick={() => setChangeModuleModalOpen(true)}
            >
            </Badge>
            <Typography variant="h6" noWrap component="div" sx={{ flexGrow: 1 }}>
                FRT Maker Portal
            </Typography>

            <Box sx={{ ml: "auto", mt: 1, textAlign: 'right' }}>
              <Typography variant={"subtitle2"} gutterBottom>
                User: {user.pf_number} - {userRoles[user.user_role]}
              </Typography>
              <Tooltip title={`${user.branch_code} - ${user.branch_name}`}>
                <Typography noWrap sx={{ overflow: "hidden", textOverflow: "ellipsis" }} variant="subtitle2">
                  Branch: {user.branch_code} - {user.branch_name}
                </Typography>
              </Tooltip>
            </Box>
          </Toolbar>
        </AppBar>
        <Drawer variant="permanent" open={open}>
          <DrawerHeader>
             <div style={{display: 'flex', alignItems: 'center', padding: '10px 5px', width: '100%'}}>
                 <img src={"/logo.svg"} className="img-circle" alt="User" style={{width: '40px', height: '40px', marginRight: '10px'}}/>
                 <div>
                    <p style={{fontWeight: 'bold', margin: 0}}>{user.userName}</p>
                    <a href="#"><i style={{color: 'green', fontSize: '10px', marginRight: '5px'}} className="fa fa-circle text-success"></i> Online</a>
                 </div>
             </div>
            <IconButton onClick={handleDrawerClose}>
              {theme.direction === "rtl" ? <ChevronRightIcon /> : <ChevronLeftIcon />}
            </IconButton>
          </DrawerHeader>
          <Divider />

            {/* Recreated JSP Menu */}
            <List>
                 <ListItem disablePadding>
                     <ListItemButton>
                        <ListItemText primary="FRT MAKER" primaryTypographyProps={{ style: { fontWeight: 'bold' } }} />
                     </ListItemButton>
                 </ListItem>

                {/* Change Audit Status Dropdown */}
                <ListItemButton onClick={() => setAuditMenuOpen(!auditMenuOpen)}>
                    <ListItemIcon><FolderOpenIcon /></ListItemIcon>
                    <ListItemText primary="Change Audit Status" />
                    {auditMenuOpen ? <ExpandLess /> : <ExpandMore />}
                </ListItemButton>
                <Collapse in={auditMenuOpen && open} timeout="auto" unmountOnExit>
                    <List component="div" disablePadding>
                        <ListItemButton sx={{ pl: 4 }} onClick={() => navigate('/FrtUser/singleBranch')}>
                            <ListItemIcon><CircleIcon sx={{fontSize: 'small'}} /></ListItemIcon>
                            <ListItemText primary="Single Branch" />
                        </ListItemButton>
                        <ListItemButton sx={{ pl: 4 }} onClick={() => navigate('/FrtUser/bulkUpload')}>
                             <ListItemIcon><CircleIcon sx={{fontSize: 'small'}}/></ListItemIcon>
                            <ListItemText primary="Multiple Branch" />
                        </ListItemButton>
                    </List>
                </Collapse>

                {/* CRS Scope Dropdown */}
                <ListItemButton onClick={() => setScopeMenuOpen(!scopeMenuOpen)}>
                    <ListItemIcon><FolderOpenIcon /></ListItemIcon>
                    <ListItemText primary="CRS Scope" />
                    {scopeMenuOpen ? <ExpandLess /> : <ExpandMore />}
                </ListItemButton>
                <Collapse in={scopeMenuOpen && open} timeout="auto" unmountOnExit>
                    <List component="div" disablePadding>
                        <ListItemButton sx={{ pl: 4 }} onClick={() => navigate('/FrtUser/branchStatus')}>
                            <ListItemIcon><CircleIcon sx={{fontSize: 'small'}}/></ListItemIcon>
                            <ListItemText primary="Add Branch" />
                        </ListItemButton>
                        <ListItemButton sx={{ pl: 4 }} onClick={() => navigate('/FrtUser/deleteBranch')}>
                            <ListItemIcon><CircleIcon sx={{fontSize: 'small'}}/></ListItemIcon>
                            <ListItemText primary="Delete Branch" />
                        </ListItemButton>
                    </List>
                </Collapse>

                {/* Other Links */}
                <ListItem disablePadding>
                    <ListItemButton onClick={() => navigate('/FrtUser/frtTrack')}>
                        <ListItemIcon><TrackChangesIcon /></ListItemIcon>
                        <ListItemText primary="Track Request" />
                    </ListItemButton>
                </ListItem>
                <ListItem disablePadding>
                    <ListItemButton onClick={() => navigate('/FrtUser/FRTAuditorsDetails')}>
                        <ListItemIcon><AssignmentIndIcon /></ListItemIcon>
                        <ListItemText primary="Branch Auditors Details" />
                    </ListItemButton>
                </ListItem>
                 <ListItem disablePadding>
                    <ListItemButton onClick={() => navigate('/Security/downloadDocsFAQ')}>
                        <ListItemIcon><BookIcon /></ListItemIcon>
                        <ListItemText primary="F.A.Q" />
                    </ListItemButton>
                </ListItem>
            </List>
            {/* End of Recreated Menu */}
          <Divider />
          <List sx={{ marginTop: 'auto' }}>
            {/* Logout Button */}
            <Tooltip title="Logout" placement="right">
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
        <Box component="main" sx={{ flexGrow: 1, p: 3, height: "100vh", overflow: "auto", backgroundColor: "#E1E6EC" }}>
          <DrawerHeader />
          {/* Main Content comes here */}
          {children}
        </Box>
        <Box sx={{ position: "fixed", bottom: 0, width: "100%", backgroundColor: "white", textAlign: "center", minHeight: 56, padding: "15px 0", boxShadow: "0 -2px 5px rgba(0,0,0,0.1)" }}>
          <Box variant="body2" sx={{ display: "flex", alignItems: "center", justifyContent: "center" }}>
            &copy;{new Date().getFullYear()} &nbsp; Tata Consultancy Services. All rights reserved.
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
        <DialogTitle id="responsive-dialog-title" sx={{ display: "flex", alignItems: "center", justifyContent: "center" }}>
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

// A default theme for the component to render correctly
const defaultTheme = createTheme();

export default FrtMakerLayout;

Key Changes and Explanations
 * State for Menus:
   * useState hooks (auditMenuOpen, scopeMenuOpen) are added to manage the open/close state of the dropdown menus, just like how open manages the drawer itself.
 * User Data from localStorage:
   * The JSP gets user information like userName from a server-side session (sessionScope). In a React Single Page Application (SPA), this data is typically fetched after login and stored in localStorage or sessionStorage.
   * The code now retrieves this user object using JSON.parse(localStorage.getItem("user")).
   * Crucially, your login process must save this user object to localStorage for this code to work. The object should look something like this:
     {
  "userName": "V1010939",
  "pf_number": "V1010939",
  "user_role": "MAKER",
  "branch_code": "1234",
  "branch_name": "Main Branch",
  "module": "CRS"
}

 * New Menu Items using Material-UI:
   * The static header "FRT MAKER" is added.
   * ListItemButton is used for clickable menu items.
   * The <Collapse> component is used to create the dropdown effect for "Change Audit Status" and "CRS Scope". Clicking the parent ListItemButton toggles the corresponding state variable, which in turn controls the in prop of the Collapse component.
   * Icons: Material-UI icons (FolderOpenIcon, ExpandLess, ExpandMore, CircleIcon, etc.) are imported and used to replicate the look and feel (fa-folder-open-o, fa-circle-o).
   * Navigation: The useNavigate hook from react-router-dom is used to programmatically navigate to the correct routes when a menu item is clicked (e.g., Maps('/FrtUser/singleBranch')).
 * Routing:
   * Your existing routing setup will now work seamlessly with this updated layout. When you click "Single Branch", React Router will see the path /FrtUser/singleBranch and render the component you've configured for that route inside the FrtMakerLayout. You will need to define these routes in your main router file (App.js or similar).
What to do with your Router Setup
Your existing router setup is correct. You just need to ensure you have defined routes for the new links you've added in the sidebar.
// In your main App.js or router file

import { BrowserRouter as Router, Routes, Route } from 'react-router-dom';
import FrtMakerLayout from './layouts/FrtMakerLayout';
import FrtMakerHome from './pages/FrtMakerHome';
// Import the new components for the new routes
import SingleBranch from './pages/SingleBranch';
import BulkUpload from './pages/BulkUpload';
import BranchStatus from './pages/BranchStatus';
// ... and so on for other pages

function App() {
  return (
    <Router>
      <Routes>
        {/* ... other routes like login */}

        <Route
          path="/FrtMakerHome"
          element={
            <PrivateRoute>
              <FrtMakerLayout>
                <FrtMakerHome />
              </FrtMakerLayout>
            </PrivateRoute>
          }
        />

        {/* Add routes for the new sidebar links */}
        <Route
          path="/FrtUser/singleBranch"
          element={
            <PrivateRoute>
              <FrtMakerLayout>
                <SingleBranch />
              </FrtMakerLayout>
            </PrivateRoute>
          }
        />
        <Route
          path="/FrtUser/bulkUpload"
          element={
            <PrivateRoute>
              <FrtMakerLayout>
                <BulkUpload />
              </FrtMakerLayout>
            </PrivateRoute>
          }
        />
        <Route
          path="/FrtUser/branchStatus"
          element={
            <PrivateRoute>
              <FrtMakerLayout>
                <BranchStatus />
              </FrtMakerLayout>
            </PrivateRoute>
          }
        />
        {/* ... add routes for deleteBranch, frtTrack, etc. */}

        {/* ... your other existing routes */}
      </Routes>
    </Router>
  );
}

export default App;


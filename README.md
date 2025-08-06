import React, { useState, useEffect, useCallback, useRef } from 'react';
import { useNavigate, useLocation, replace } from 'react-router-dom';
import { useDispatch, useSelector } from 'react-redux';
import { styled } from '@mui/material/styles';
import {
  Avatar,
  Box,
  Divider,
  Drawer as MuiDrawer,
  Stack,
  Typography,
  Tooltip,
  Button,
  IconButton,
} from '@mui/material';
import { drawerClasses } from '@mui/material/Drawer';
import LogoutRoundedIcon from '@mui/icons-material/LogoutRounded';

import SelectModule from './SelectModule';
import CustomizedTreeView from './CustomizedTreeView';

import { roleMapping, roleOptionsMap } from '../../../constants/RoleConstants';
import { convertToKebabCase } from '../../../utils/commonUtils';
import {
  updateSelectedMenuOption,
  updateSelectedModule,
  setFilteredSideMenuOptions,
} from '../../../../app/features/sidebarSlice';

// Assets
import companyLogo from '../../../../assets/logos/6-cropped.png';
import {
  buildNavigationPath,
  createUserAvatar,
  findMenuOptionById,
  findMenuOptionByPath,
  useMenuFiltering,
  useUser,
} from './sideMenuUtils';
import ConfirmationDialog from '../../ui/dialogs/ConfirmationDialog';

// Constants
const DRAWER_WIDTH = 240;

const MODULES = {
  BALANCE_SHEET: 'BS',
  CFS: 'CFS',
};

const StyledDrawer = styled(MuiDrawer)({
  width: DRAWER_WIDTH,
  flexShrink: 0,
  boxSizing: 'border-box',
  [`& .${drawerClasses.paper}`]: {
    width: DRAWER_WIDTH,
    boxSizing: 'border-box',
  },
});

// Main Component
const SideMenu = () => {
  const navigate = useNavigate();
  const location = useLocation();
  const dispatch = useDispatch();
  const user = useUser();
  const [dialogOpen, setDialogOpen] = useState(false);

  // Redux State
  const selectedMenuOption = useSelector((state) => state.sidebar.selectedMenuOption.value);
  const currentModule = useSelector((state) => state.sidebar.selectedModule.value);

  const [navigationPath, setNavigationPath] = useState([]);
  const [displayedMenuOptions, setDisplayedMenuOptions] = useState([]);

  // Ref to track navigation source
  const isInternalNavigation = useRef(false);

  // Custom Hooks
  const filterMenuOptions = useMenuFiltering(user, currentModule);

  // Prevent page reload
  useEffect(() => {
    const handleBeforeUnload = (event) => {
      event.preventDefault();
      event.returnValue = 'Do you want to refresh? All data will be lost.';
      return event.returnValue;
    };

    window.addEventListener('beforeunload', handleBeforeUnload);
    return () => window.removeEventListener('beforeunload', handleBeforeUnload);
  }, []);

  // Handle URL navigation (only when navigationPath changes from user interaction)
  useEffect(() => {
    if (!navigationPath.length || !isInternalNavigation.current) return;

    let urlPath = navigationPath.map((item) => convertToKebabCase(item.label)).join('/');

    // Special case for incomplete branch users
    // if (user.capacity === '52' && user.isBranchFinal !== 'Y') {
    //   urlPath = 'user-details';
    // }

    if (urlPath && user.capacity && roleMapping[user.capacity]) {
      const fullPath = `/${convertToKebabCase(roleMapping[user.capacity].role)}/${urlPath}`;

      // Only navigate if the current path is different to prevent infinite loops
      if (location.pathname !== fullPath) {
        navigate(fullPath);
      }
    }

    // Reset the flag after navigation
    isInternalNavigation.current = false;
  }, [navigationPath, user.capacity, navigate, location.pathname]);

  // Sync selected option with current URL (runs whenever URL changes)
  useEffect(() => {
    const urlSegments = location.pathname.split('/').filter(Boolean);
    const expectedRole = convertToKebabCase(roleMapping[user.capacity]?.role || '');

    if (urlSegments.length > 0 && urlSegments[0] === expectedRole) {
      const pathSegments = urlSegments.slice(1);
      const availableOptions = roleOptionsMap[currentModule]?.[user.capacity] || [];
      let options;
      if (user?.circleCode !== '020') {
        options = availableOptions.filter((n) => n?.id !== '1.4');
      } else {
        options = availableOptions;
      }
      const matchedOption = findMenuOptionByPath(pathSegments, options);
      console.log('matchedOption', matchedOption);

      if (matchedOption) {
        // Only update if the selected option is different to avoid infinite loops
        if (!selectedMenuOption || selectedMenuOption.id !== matchedOption.id) {
          dispatch(updateSelectedMenuOption({ value: matchedOption }));
          const fullPath = buildNavigationPath(matchedOption.id, availableOptions);
          if (fullPath) {
            setNavigationPath(fullPath);
          }
        }
      } else if (pathSegments.length > 0) {
        // Only warn if there are actual path segments (not just the role)
        console.warn('Invalid route detected:', location.pathname);
        // Clear selected option if route is invalid
        dispatch(updateSelectedMenuOption({ value: null }));
        setNavigationPath([]);
      }
    } else {
      // Clear selection if not in the expected role path
      dispatch(updateSelectedMenuOption({ value: null }));
      setNavigationPath([]);
    }
  }, [location.pathname, user.capacity, currentModule, selectedMenuOption, dispatch]);

  // Handle menu option selection
  const handleMenuOptionSelect = useCallback(
    (event, optionId) => {
      const periodCount = (optionId.match(/\./g) || []).length;

      // Only select if it's a sub-option (has more than one period)
      if (optionId && periodCount > 1) {
        const availableOptions = roleOptionsMap[currentModule]?.[user.capacity] || [];
        const selectedOption = findMenuOptionById(optionId, availableOptions);

        if (selectedOption) {
          // Mark as internal navigation
          isInternalNavigation.current = true;

          dispatch(updateSelectedMenuOption({ value: selectedOption }));
          const fullPath = buildNavigationPath(optionId, availableOptions);
          if (fullPath) {
            setNavigationPath(fullPath);
          }
        }
      }
    },
    [currentModule, user.capacity, dispatch]
  );

  // Handle module change
  const handleModuleChange = useCallback(
    (event) => {
      dispatch(updateSelectedModule({ value: event.target.value }));
      navigate(
        '/' + convertToKebabCase(roleMapping[user.capacity]?.role) + (roleMapping[user.capacity].role ? '/home' : ''),
        { replace: true }
      );
    },
    [dispatch]
  );

  // Filter and set menu options
  useEffect(() => {
    const availableOptions = roleOptionsMap[currentModule]?.[user.capacity] || [];
    const filteredOptions = filterMenuOptions(availableOptions);

    dispatch(setFilteredSideMenuOptions({ value: filteredOptions }));
    setDisplayedMenuOptions(filteredOptions);
  }, [currentModule, filterMenuOptions, dispatch, user]);

  const showModuleSelector = ['51', '52'].includes(user.capacity);

  const moduleOptions = [
    { value: MODULES.BALANCE_SHEET, name: 'Balance Sheet' },
    { value: MODULES.CFS, name: 'CFS' },
  ];

  // Handle user logout
  const handleLogout = useCallback(() => {
    // localStorage.removeItem('user');
    navigate('/');
  }, [navigate]);

  const handleLogoutClick = () => {
    setDialogOpen(true);
  };

  const handleCloseDialog = () => {
    setDialogOpen(false);
  };

  // Render nothing if user data is not available
  if (!user.capacity) {
    return null;
  }

  return (
    <StyledDrawer
      variant="permanent"
      sx={{
        display: { xs: 'none', md: 'block' },
        [`& .${drawerClasses.paper}`]: {
          backgroundColor: 'background.paper',
        },
      }}
    >
      {/* Header */}
      <Box
        sx={{
          display: 'flex',
          mt: 'calc(var(--template-frame-height, 0px) + 4px)',
          p: 1.5,
          alignItems: 'center',
          overflow: 'hidden',
        }}
      >
        <img
          src={companyLogo}
          alt="Balance Sheet Logo"
          style={{
            filter: 'brightness(120%)',
            height: 70,
            width: 70,
            borderRadius: 50,
            objectFit: 'contain',
          }}
        />
        <Typography
          variant="subtitle1"
          sx={{ color: 'text.secondary', ml: 1, lineHeight: 1.2, fontSize: 16, fontWeight: 'bold' }}
        >
          Balance Sheet Automation
        </Typography>
      </Box>

      <Divider />

      {/* Module Selector */}
      {showModuleSelector && (
        <>
          <Box sx={{ p: 1.5 }}>
            <SelectModule
              value={currentModule}
              onChange={handleModuleChange}
              fullWidth
              sx={{ borderRadius: '10px' }}
              options={moduleOptions}
            />
          </Box>
          <Divider />
        </>
      )}

      {/* Menu Content */}
      <Box
        sx={{
          overflow: 'auto',
          height: '100%',
          display: 'flex',
          flexDirection: 'column',
        }}
      >
        <CustomizedTreeView options={displayedMenuOptions} handleSelectOption={handleMenuOptionSelect} />
      </Box>

      {/* User Info and Logout */}
      <Stack
        direction="row"
        sx={{
          p: 2,
          gap: 1,
          alignItems: 'center',
          borderTop: '1px solid',
          borderColor: 'divider',
        }}
      >
        <Avatar {...createUserAvatar(user.userName || 'User')} sx={{ width: 36, height: 36 }} />
        <Box sx={{ mr: 'auto' }}>
          <Typography
            variant="body1"
            sx={{
              fontWeight: 'medium',
              lineHeight: '16px',
              overflow: 'hidden',
              textOverflow: 'ellipsis',
              whiteSpace: 'none',
              width: 120,
            }}
          >
            {user.userName || 'Unknown User'}
          </Typography>
          <Typography sx={{ color: 'text.secondary', fontSize: 13, mt: 0.5 }}>
            {roleMapping[user.capacity]?.role || 'Unknown Role'}
          </Typography>
          <Typography sx={{ color: 'text.secondary', fontSize: 13 }}>{user.userId || 'Unknown ID'}</Typography>
        </Box>
        <Tooltip title={'Logout'}>
          <IconButton size="medium" onClick={handleLogoutClick}>
            <LogoutRoundedIcon fontSize="medium" />
          </IconButton>
        </Tooltip>
      </Stack>

      <ConfirmationDialog
        open={dialogOpen}
        onClose={handleCloseDialog}
        onButtonClick={handleLogout}
        dialogTitle={'Confirm Logout'}
        dialogContent={'Are you sure you want to log out?'}
      />
    </StyledDrawer>
  );
};

export default SideMenu;

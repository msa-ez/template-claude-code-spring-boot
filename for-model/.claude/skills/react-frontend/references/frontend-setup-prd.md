# Project Generation Order

## 1. Generate Frontend Sub-package Structure
- Create folders and files with the following package structure.
```
/frontend                  # Integrated frontend project (React + Vite + TypeScript)
  /public
    /favicon.ico          # Favicon
  /src
    /components
      /common             # Global common components
        Layout.tsx        # Common layout component
        Loading.tsx       # Loading component
        Sidebar.tsx       # Sidebar component
    /pages                # Page components (React Router based)
      HomePage.tsx        # Main home page (with BC navigation)
    /hooks                # Custom hooks
    /services             # API and business logic
      /api
        client.ts         # Axios client basic configuration
        interceptors.ts   # API interceptors
        types.ts          # API-related types
    /stores               # State management
    /types                # TypeScript type definitions
      common.types.ts     # Common type definitions
      api.types.ts        # API-related types
    /utils                # Utility functions
      constants.ts        # Constant definitions (API endpoints, etc.)
    /theme                # Material-UI theme settings
      index.ts            # Integrated theme settings
    App.tsx               # Root App component
    main.tsx              # Vite entry point
    index.css             # Global styles
    vite-env.d.ts         # Vite type definitions
  package.json            # NPM package settings
  vite.config.ts          # Vite build tool configuration
  tsconfig.json           # TypeScript configuration
  .gitignore              # Git ignore file
  ```

## 2. Generate React + Vite Based Configuration Files
- Generate configuration file code such as package.json, tsconfig.json, vite.config.ts according to the created package structure and files.
- Set server port to 3000 and configure considering Node 22 version.

## Generate Main Page
- Generate main page based on the layout format below.

    1) Full Window Layout
        - Container: Width and height must be fully filled when opened in browser (no vh, vw 100 processing)
        - Box: `sx=\{{ display: 'flex', minHeight: '100vh' }}` for flex layout
        - No padding: Remove default margin with `sx=\{{ m: 0, p: 0 }}`

    2) Main Page Structure
        ```typescript
        <Box sx=\{{ display: 'flex', minHeight: '100vh', width: '100vw' }}>
        <Sidebar />
        <Box sx=\{{ flex: 1, p: 3 }}>
            <Container maxWidth="xl">
            {/* Main Content */}
            </Container>
        </Box>
        </Box>
        ```

## 3. Generate Navigation and Routing
- Configure sidebar and main page navigation according to the requirements below.

    1) Route Card System
        - Card Layout: `Grid container spacing={3}` with responsive columns
        - Route Card: `Card` with `onClick` navigation and hover effect
        - Card Content: Icon + Title + Description + Action button
        - Responsive: `xs={12} sm={6} md={4} lg={3}` for various screen sizes

    2) Sidebar Navigation
        - Fixed Width: `width: 240px` on desktop, collapsible on mobile
        - Position: `position: 'fixed'`, `left: 0`, `top: 0`
        - Height: `height: '100vh'` for full viewport height
        - Z-Index: `zIndex: 1200` to stay above content
        - Toggle Type: Must be toggleable and main screen area should adjust according to toggle
        - Common Component: Must be applied as a common component to always exist regardless of page routing connections

    3) Sidebar Component (sidebar.tsx)
        ```typescript
        <Drawer variant="permanent" sx=\{{ width: 240, '& .MuiDrawer-paper': { width: 240 } }}>
        <Box 
            sx=\{{ 
            p: 2, 
            cursor: 'pointer',
            '&:hover': {
                backgroundColor: 'rgba(0, 0, 0, 0.04)',
            },
            borderRadius: 1,
            mx: 1,
            }}
            onClick={() => navigate('/')}
        >
            <Typography variant="h6" color="primary">erpmodel</Typography>
        </Box>
        <List>
            <ListItem button onClick={() => navigate('/dashboard')}>
            <ListItemIcon><Dashboard /></ListItemIcon>
            <ListItemText primary="Dashboard" />
            </ListItem>
        </List>
        </Drawer>
        ```

    4) Responsive Sidebar
    ```typescript
    // Mobile
    <Drawer variant="temporary" open={mobileOpen} sx=\{{ display: { xs: 'block', md: 'none' } }}>
    {/* Sidebar Content */}
    </Drawer>

    // Desktop  
    <Drawer variant="permanent" sx=\{{ display: { xs: 'none', md: 'block' } }}>
    {/* Sidebar Content */}
    </Drawer>
    ```

    5) Sidebar Navigation Items
        - Active State: `selected` prop with `bgcolor: 'action.selected'`
        - Icons: Material-UI icons with consistent size
        - Nested Items: Use `Collapse` for sub-navigation

## Important Notes
- Generate components and features based only on what is explicitly required in the PRD.
- Do not add dashboard statistics, mock data, or decorative elements that are not explicitly requested.

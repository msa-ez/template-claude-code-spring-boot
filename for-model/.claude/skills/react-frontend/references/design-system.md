# Frontend Design System Rules

### Color System

#### Material-UI Theme Colors
- Primary: `#2196f3` (primary action buttons, brand elements)
- Secondary: `#f50057` (secondary actions, inactive state)
- Success: `#4caf50` | Warning: `#ff9800` | Error: `#f44336` | Info: `#2196f3`
- Background: `#fafafa` (default), `#ffffff` (paper)
- Text: `primary`, `secondary`, `disabled` variants

### Typography

#### Material-UI Typography Variants
- Headings: `h1` (fontWeight: 700), `h2-h6` (fontWeight: 600)
- Body: `body1` (main content), `body2` (smaller text)
- Caption: Small text using `color="text.secondary"`
- Colors: `primary`, `secondary`, `error`, `success.main`, `text.secondary`

### Layout and Spacing

#### Container System
- Container: `maxWidth="xl"` (1536px), responsive padding using `px/py` props
- Box: Custom containers with `maxWidth`, `padding`, `margin` properties
- Responsive: Use breakpoint objects `{ xs: 2, sm: 3, md: 4 }`

#### Grid System
- Grid Container: `spacing={2|3}` for consistent spacing
- Grid Items: `xs={12} sm={6} md={4}` for various screen sizes
- Common Patterns: 4-column statistics, 3-column services, responsive layouts

### Component Styles

#### Cards
- Default: `elevation={2}`, `sx={{ p: 2, borderRadius: 2 }}`
- Hover Effect: `transform: 'translateY(-2px)'`, `transition: 'all 0.2s ease'`

#### Buttons
- Primary: `variant="contained" color="primary"` with `startIcon`
- Secondary: `variant="outlined" color="inherit"`
- Text: `variant="text" color="primary"`
- Sizes: `small`, `medium`, `large`

#### Avatar and Chips
- Avatar: `sx={{ bgcolor: 'primary.main', width: 56, height: 56 }}`
- Status Chips: `color="success|warning|error"`, `variant="filled"`, `size="small"`

### Data Tables

#### Material-UI Table
- Container: Wrap with `Card` → `CardContent` → `TableContainer`
- Styling: Header with `bgcolor: 'grey.50'`, hover effect on rows
- Actions: IconButton with `Edit`/`Delete` icons, status with chips

#### MUI X DataGrid
- Columns: Define `GridColDef[]` with field, headerName, width, sortable
- Custom Cells: Use `renderCell` for chips, icons, custom content
- Features: Pagination with `autoHeight`, sorting, filtering

### Animations and Interactions

#### Framer Motion
- Fade: `initial={{ opacity: 0 }}`, `animate={{ opacity: 1 }}`, `transition={{ duration: 0.3 }}`
- Slide: Add `x: -50` to `x: 0` for entry effect
- Material-UI: Use `Fade`, `Slide`, `Grow` components with `timeout` prop

#### Loading States
- Skeleton: `variant="rectangular"` with `height`, `width`, `borderRadius`
- Progress: `CircularProgress` with `color="primary"`, `size={40}`
- Backdrop: Full-screen overlay with `zIndex: 9999`

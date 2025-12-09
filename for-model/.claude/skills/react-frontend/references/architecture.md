# Frontend Architecture Rules

## Technology Stack

### Project Configuration
- React 18+ / TypeScript 5+ / Vite 5+
- Use npm and Node.js 22

### Data Fetching and Response Handling
- Use Axios for backend communication
- Single entry point through API Gateway (port 8088)
- HAL JSON (Spring Data REST) response format
- Centralized error handling through API interceptors
- Skip API calls in local environment; return empty or mock data to ensure page loading

### State Management
- Zustand for global state management
- React Query (TanStack Query) for server state management and caching
- useState/useReducer for local component state

### UI Components and Styling
- Material-UI (MUI) component library
- Follow `@.claude/skills/react-frontend/references/design-system.md` for consistent UI

### Routing
- React Router v6+
- Layout-based nested routing structure
- Lazy loading for bundle optimization

### Form Handling and Validation
- React Hook Form for form state management
- Yup or Zod for schema-based validation

### Development Tools
- ESLint + Prettier for code quality and formatting
- TypeScript strict mode enabled
- Vite environment variables (.env) management

### Testing
- Vitest for unit testing
- React Testing Library for component testing

---

## Project Structure

### Metadata-based Dynamic Component Structure

When creating the package structure in the first step of frontend-generation-rules, follow this component structure.

```
/src/
  /components/
    /common/                        # Common components
      Layout.tsx
      Loading.tsx
      Sidebar.tsx
    /[aggregate]/                   # Aggregate level folder
      [Command]Modal.tsx            # Command modal (when isRestRepository: false)
      [ReadModel]View.tsx           # ReadModel view component
  /pages/
    HomePage.tsx                    # Home page
    DashboardPage.tsx               # Main dashboard
    [Aggregate]sPage.tsx            # Aggregate main page
    [ReadModel]Page.tsx             # ReadModel page
  /hooks/
    use[Aggregate].ts               # Aggregate custom hook
  /services/
    /api/                           # API client configuration
      client.ts
      interceptors.ts
      types.ts
    [aggregate].service.ts          # Aggregate API service
  /stores/
    [aggregate].store.ts            # Aggregate state management
  /types/
    common.types.ts                 # Common types
    api.types.ts                    # API types
    [aggregate].types.ts            # Aggregate types
  /theme/
    index.ts                        # Material-UI theme
```

### Structure Organization Principles

1. Aggregate-based Organization: Components are organized according to aggregate boundaries
2. Separation of Concerns: Clear separation between pages, components, hooks, services, stores, types
3. Common Components: Shared components like Layout, Loading, Sidebar are located in `/components/common/`
4. Service Layer: API client configuration and aggregate-specific services are located in `/services/`
5. Type Safety: TypeScript types are organized by domain in `/types/`
6. State Management: Zustand stores for each aggregate are located in `/stores/`

---

## Architecture Principles

1. Component-based Architecture: Build reusable and composable components
2. Type Safety: Leverage TypeScript for compile-time safety
3. Performance: Lazy loading, code splitting, and efficient state management
4. API Integration: HAL JSON format with proper error handling
5. Offline First: Gracefully handle backend unavailability
6. Domain-Driven Design: Code organization following aggregate boundaries
7. Separation of Concerns: Clear separation of business logic, presentation, and data access

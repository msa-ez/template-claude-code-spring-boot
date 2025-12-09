# ERP Frontend Essential Features Implementation Guide

Reference: Odoo ERP Official Documentation (https://www.odoo.com/documentation/17.0/applications/essentials/)
Important: All files listed below MUST be generated.

---
## Generation Order
The guide should proceed in the following order:

Review entire guide content → Create file structure → Generate files according to feature implementation requirements → Verify file creation (check for missing files and regenerate) → Verify Frontend sub-components → Connect generated files and components according to Odoo guide → Check and fix errors after connection → Install node_modules (node 22)

## 📁 Required File Generation List

### 1. Utilities
```
src/utils/
├── exportUtils.ts       # CSV/Excel/XML export (exportToCSV, exportToExcel, exportToXML, exportData)
├── importUtils.ts       # CSV/Excel/XML import (parseCSV, parseExcel, parseXML, importData)
├── filterUtils.ts       # Filter and domain processing (buildFilter, applyFilter)
└── constants.ts         # Constant definitions
```

### 2. Common Components
```
src/components/common/
├── AdvancedSearch.tsx   # Integrates quick search, advanced filters, grouping, favorites
├── ExportDialog.tsx     # Format selection (CSV/Excel/XML), field selection, include ID option
├── ImportDialog.tsx     # File upload, field mapping, preview
├── FilterBuilder.tsx    # Create filters with field/operator/value selection
├── GroupBySelector.tsx  # Multiple grouping selection
├── FavoriteSearch.tsx   # Save/load favorites
└── Comparison.tsx       # Period-based data comparison
```

### 3. Chatter Components
```
src/components/Chatter/
├── Chatter.tsx          # Tab-based integrated component (messages/activities/followers/history)
├── MessageList.tsx      # Display message list
├── MessageInput.tsx     # Compose messages (file attachments, mentions)
├── ActivityList.tsx     # Activity list (todo/meeting/call/email)
├── ActivityScheduler.tsx # Create/modify activity schedules
└── FollowerList.tsx     # Add/remove followers
```

### 4. Activity Components
```
src/components/Activities/
├── ActivitiesDashboard.tsx  # Tabs: Scheduled/Overdue/Completed
├── ActivityCard.tsx         # Single activity card
├── ActivityScheduler.tsx    # Activity schedule dialog
└── ActivityReminder.tsx     # Reminder settings
```

### 5. Report Components
```
src/components/Reports/
├── ReportBuilder.tsx    # Select chart type, data source, grouping, aggregation
├── ReportViewer.tsx     # Display generated reports
├── ChartBuilder.tsx     # Chart generation based on Chart.js
└── PivotTable.tsx       # Pivot table
```

### 6. Automation Components
```
src/components/Automation/
├── AutomationBuilder.tsx    # Complete trigger/condition/action builder
├── AutomationList.tsx       # Automation rules list
├── TriggerConfig.tsx        # Trigger type configuration (create/update/delete/scheduled)
└── ActionConfig.tsx         # Action type configuration (email/field update/record creation)
```

### 7. Dashboard Components
```
src/components/Dashboard/
├── Dashboard.tsx        # Widget layout (drag and drop support)
├── DashboardWidget.tsx  # Individual widget wrapper
├── WidgetConfig.tsx     # Widget configuration dialog
└── WidgetLibrary.tsx    # Available widget catalog
```
---

## 🔧 Feature-specific Implementation Requirements

### 1. Export/Import (exportUtils.ts, ExportDialog.tsx, ImportDialog.tsx)

#### exportUtils.ts Required Functions
```typescript
// Required implementation functions
export const exportToCSV(data: any[], fields: string[], filename?: string): void
export const exportToExcel(data: any[], fields: string[], filename?: string): void
export const exportToXML(data: any[], fields: string[], filename?: string): void
export const exportData(format: 'csv'|'xlsx'|'xml', data: any[], fields: string[], filename?: string): void

// Helper functions
const getNestedValue(obj: any, path: string): any
const escapeXml(str: string): string
const downloadBlob(blob: Blob, filename: string): void
```

#### ExportDialog.tsx Required Features
- File format selection: CSV, Excel, XML (radio buttons)
- Field selection: Multiple selection with checkboxes, select all/deselect all buttons
- Include ID option: Checkbox
- Disable export button when no fields are selected

#### ImportDialog.tsx Required Features
- 3-step process: Upload → Mapping → Preview
- File upload: Support .csv, .xlsx, .xml
- Field mapping: Match file columns → DB fields
- Preview: Display first 5 rows
---
### 2. Search and Filter (AdvancedSearch.tsx, FilterBuilder.tsx, GroupBySelector.tsx)

#### AdvancedSearch.tsx Required Props
```typescript
interface AdvancedSearchProps {
  filters: Filter[];
  groupBy: GroupBy[];
  onFilterChange: (filters: Filter[]) => void;
  onGroupByChange: (groupBy: GroupBy[]) => void;
  onQuickFilter: (query: string) => void;
  availableFields: Array<{ name: string; label: string; type: string }>;
  savedFilters?: Array<{ id: string; name: string; filters: Filter[] }>;
}

interface Filter {
  field: string;
  operator: '=' | '!=' | '>' | '<' | '>=' | '<=' | 'like' | 'in' | 'not in';
  value: any;
  label?: string;
}
```

#### Required Features
- Quick search: TextField with SearchIcon
- Advanced filters: Display FilterBuilder in dialog
- Favorites: Saved filters list dialog
- Current filters/groups: Display as chips, deletable

#### FilterBuilder.tsx Required Features
- Field selection (Select)
- Operator selection (Select): =, !=, >, <, >=, <=, like, in, not in
- Value input (TextField)
- "Add Filter" button

#### GroupBySelector.tsx Required Features
- Add grouping criteria (Select)
- Selected groups: Display order as chips (1. Field name, 2. Field name)
- Remove group: Delete chip

---


### 3. Activities (ActivitiesDashboard.tsx, ActivityCard.tsx)

#### ActivitiesDashboard.tsx Required Props
```typescript
interface Activity {
  id: string;
  type: 'todo' | 'meeting' | 'call' | 'email';
  title: string;
  dueDate: string;
  assignedTo: string;
  modelName: string;
  recordId: number;
  status: 'pending' | 'done' | 'overdue';
  priority: 'low' | 'normal' | 'high';
}

interface ActivitiesDashboardProps {
  userId: string;
  activities: Activity[];
  onActivityClick?: (activity: Activity) => void;
}
```
#### Required Features
- 3 tabs: Scheduled (badge: pendingCount), Overdue (badge: overdueCount), Completed
- Activity icons: todo → Task, meeting → Event, call → Phone, email → Email
- Priority colors: high → error, normal → warning, low → default
- Call onActivityClick when list item is clicked
---
### 4. Reports (ReportBuilder.tsx, ChartBuilder.tsx)

#### ReportBuilder.tsx Required Props
```typescript
interface ReportConfig {
  chartType: 'bar' | 'line' | 'pie' | 'scatter';
  dataSource: string;
  groupBy: string[];
  aggregations: Array<{ field: string; function: 'sum'|'avg'|'count'|'min'|'max' }>;
  filters: any[];
}

interface ReportBuilderProps {
  model: string;
  availableFields: Array<{ name: string; label: string; type: string }>;
  onSave?: (config: ReportConfig) => void;
}
```

#### Required Features
- Chart type selection: Bar/Line/Pie/Scatter (Select)
- Grouping criteria: Multiple selection (Select multiple)
- Aggregation function: Select sum/avg/count/min/max
- Create button: Load and display chart data
- Save button: Call onSave callback

#### Chart.js Integration
```typescript
import { Chart as ChartJS, ArcElement, CategoryScale, LinearScale, BarElement, Title, Tooltip, Legend } from 'chart.js';
import { Bar, Pie, Line, Scatter } from 'react-chartjs-2';

ChartJS.register(ArcElement, CategoryScale, LinearScale, BarElement, Title, Tooltip, Legend);
```
---
### 5. Chatter (Chatter.tsx, MessageList.tsx, ActivityList.tsx, FollowerList.tsx)

#### Chatter.tsx Required Props
```typescript
interface ChatterProps {
  recordId: number;
  model: string;
  features?: Array<'messages' | 'activities' | 'followers' | 'history'>;
}
```
#### Required Features
- 4 tabs: Messages, Activities, Followers, History
- Display icon on each tab
- Display corresponding component when tab is switched

#### MessageList.tsx
- Display message list (reverse order)
- Display author, content, creation date/time

#### ActivityList.tsx
- Display activity list
- Add activities through ActivityScheduler

#### FollowerList.tsx
- Display follower list
- Add/remove follower buttons
---

### 6. Automation (AutomationBuilder.tsx, TriggerConfig.tsx, ActionConfig.tsx)

#### AutomationBuilder.tsx Required Structure
```typescript
interface Automation {
  id: string;
  name: string;
  trigger: { type: 'create'|'update'|'delete'|'scheduled'; model: string; conditions?: any };
  actions: Array<{ type: 'email'|'field_update'|'create_record'; config: any }>;
  active: boolean;
}
```
#### TriggerConfig.tsx Required Features
- Trigger type selection: Record creation, modification, deletion, scheduled
- Model selection
- Condition setting (reuse FilterBuilder)

#### ActionConfig.tsx Required Features
- Action type selection: Send email, update field, create record
- Detailed configuration UI for each action
---
### 7. Dashboard (Dashboard.tsx, DashboardWidget.tsx, WidgetConfig.tsx)

#### Dashboard.tsx Required Features
- Widget layout (grid-based)
- Drag and drop support (react-grid-layout recommended)
- Add/remove widgets
- Save/load layout

#### DashboardWidget.tsx Required Props
```typescript
interface WidgetProps {
  id: string;
  type: 'kpi' | 'chart' | 'list' | 'gauge';
  title: string;
  config: any;
  data?: any;
}
```
#### WidgetConfig.tsx Required Features
- Widget type selection
- Data source selection
- Widget-specific option settings
---
### 8. List Page Integration Pattern

```typescript
export const [Aggregate]ListPage: React.FC = () => {
  const [selected, setSelected] = useState<any | null>(null);
  const [filters, setFilters] = useState<Filter[]>([]);
  const [groupBy, setGroupBy] = useState<GroupBy[]>([]);

  return (
    <Container maxWidth="xl">
      {/* Advanced Search - Always at the top */}
      <Paper sx={{ p: 2, mb: 2 }}>
        <AdvancedSearch
          filters={filters}
          groupBy={groupBy}
          onFilterChange={setFilters}
          onGroupByChange={setGroupBy}
          onQuickFilter={(q) => console.log('Quick search:', q)}
          availableFields={metadata.fields}
        />
      </Paper>

      {/* Action Bar */}
      <Paper sx={{ p: 2, mb: 2 }}>
        <Box sx={{ display: 'flex', gap: 1, justifyContent: 'space-between' }}>
          <Box sx={{ display: 'flex', gap: 1 }}>
            {/* Command Buttons */}
            {commands.filter(c => !c.isRestRepository).map(cmd => (
              <[Command]Button key={cmd.name} selected={selected} />
            ))}
          </Box>
          <Box sx={{ display: 'flex', gap: 1 }}>
            <Button startIcon={<Download />} onClick={() => setShowExport(true)}>Export</Button>
            <Button startIcon={<Upload />} onClick={() => setShowImport(true)}>Import</Button>
          </Box>
        </Box>
      </Paper>

      {/* Table */}
      <[Aggregate]Table
        data={data}
        onRowClick={setSelected}
        selectedItem={selected}
        filters={filters}
        groupBy={groupBy}
      />

      {/* Export/Import Dialogs */}
      <ExportDialog open={showExport} onClose={() => setShowExport(false)} ... />
      <ImportDialog open={showImport} onClose={() => setShowImport(false)} ... />
    </Container>
  );
};
```
---
## Generation Rules

1. Use Material-UI: All components must use @mui/material
2. TypeScript: All Props must have explicit type definitions with interfaces
3. Icons: Use @mui/icons-material
4. Dialog Pattern: Close button (top right), DialogActions (bottom)
5. Error Handling: Handle errors with try-catch, log with console.error
6. Korean Support: Add BOM ('\uFEFF'), include timestamp in filename
7. Reusability: FilterBuilder, GroupBySelector, etc. must be imported and reused in other components

## Component Integration
1. For Export/Import, Search and Filter: Connect identically to each Aggregate component under frontend (AccountingEntry, TaxRate, Lead, BillingPolicy, QuoteInvoice)
2. For Reports, Activities, Chat, Dashboard: Connect with App.tsx to display on main page, but display at the bottom of the main card view, not in the sidebar

# GUI Reverse PRD - Digital Basement Energy Management Platform

## Overview

This document serves as a comprehensive reverse Product Requirements Document (PRD) for the Digital Basement GUI. It documents all existing UI components, views, patterns, and provides prompts that can be used to recreate similar examples.

**Tech Stack:**
- React + TypeScript
- Tailwind CSS with custom design system
- shadcn/ui component library
- Recharts for data visualization
- React Router for navigation
- Supabase for backend/auth
- Lucide React for icons

---

## Design System

### Color Palette

**Brand Colors:**
- Primary: `hsl(213 15% 25%)` - Modern slate blue-gray
- Primary Foreground: `hsl(0 0% 100%)` - White
- Secondary: `hsl(210 15% 96%)` - Light gray
- Accent: `hsl(213 25% 45%)` - Medium blue-gray

**Status Colors:**
- Success: `hsl(145 60% 42%)` - Green
- Warning: `hsl(38 75% 52%)` - Amber
- Destructive: `hsl(0 65% 52%)` - Red

**Utility Meter Colors:**
- Electricity: `hsl(45 75% 52%)` - Yellow
- Gas: `hsl(15 75% 52%)` - Orange
- Water: `hsl(210 75% 52%)` - Blue
- Heating: `hsl(25 65% 52%)` - Red-orange

**Design Tokens:**
- Border Radius: `0.5rem`
- Shadows: Soft, Medium, Strong (minimal)
- Transitions: Smooth cubic-bezier animations
- Font: Inter (system fallback)

### Typography Scale

- Hero Title: `text-6xl lg:text-8xl` (Landing page)
- Page Title: `text-3xl font-semibold`
- Card Title: `text-lg font-medium` or `text-sm font-medium`
- Body: Default (16px)
- Small: `text-sm`
- Muted: `text-muted-foreground`

### Spacing System

- Container padding: `p-6` (24px)
- Card padding: `p-4` or `p-6`
- Gap between elements: `gap-4`, `gap-6`
- Section spacing: `space-y-6`

---

## Layout Structure

### AppLayout Component

**Prompt:**
```
Create a responsive application layout component with:
- Fixed left sidebar (256px wide on desktop, slide-out on mobile)
- Sidebar contains: Logo section, Navigation menu, User dropdown menu
- Main content area with left padding (lg:pl-64)
- Mobile header with hamburger menu
- Footer with copyright, language switcher, API docs link
- Sidebar navigation items with icons and active state highlighting
- User avatar dropdown with profile link and sign out
- Responsive breakpoints: mobile-first, sidebar hidden on mobile, visible on lg screens
- Use Tailwind CSS with custom color variables
- Implement smooth transitions for sidebar toggle
```

**Key Features:**
- Collapsible sidebar on mobile
- Active route highlighting
- User profile dropdown
- Language switcher in footer
- Super admin navigation items (conditional)

---

## Authentication Views

### Auth Page (Sign In / Sign Up)

**Prompt:**
```
Create a dual-tab authentication page with:
- Centered card layout on gradient background
- Two tabs: "Sign In" and "Sign Up"
- Sign In form: Email, Password, Submit button
- Sign Up form: First Name, Last Name, Email, Password, TOS/Privacy checkboxes, Newsletter checkbox
- Password strength indicator with visual progress bar
- Password requirements checklist (8+ chars, lowercase, uppercase, number, special)
- Trial benefits section in signup tab with checkmark list
- Email confirmation screen after signup with instructions
- Loading states on buttons
- Error handling with toast notifications
- Form validation
- Use shadcn/ui Card, Input, Button, Checkbox, Tabs components
- Gradient hero background (bg-gradient-hero)
- Logo and branding at top
```

**Key Features:**
- Real-time password strength validation
- Visual password requirements checklist
- Email confirmation flow
- Trial benefits display
- Responsive design

---

## Dashboard View

**Prompt:**
```
Create a comprehensive dashboard page with:
- Page header: Title "Dashboard" and subtitle
- Trial status banner component (if trial active)
- Stats cards grid (5 columns on desktop, responsive): Buildings, Meters, Consumption, Costs, Alerts
- Each stat card: Icon, large number, description text
- Two chart cards side-by-side:
  - Bar chart: Monthly consumption (electricity, heat, water) with custom colors
  - Line chart: Cost trend over time
- Recent Activity card: List of activity items with icons, badges, timestamps
- Quick Actions card: Grid of 4 action buttons with icons
- Use Recharts for visualizations
- Responsive grid layouts
- Loading skeleton states
- Empty states with helpful messages
- Use shadcn/ui Card components
- Color-coded icons for different resource types
```

**Key Features:**
- Real-time stats from database
- Interactive charts
- Activity feed
- Quick action shortcuts
- Responsive grid system

---

## Buildings Management

### Buildings List Page

**Prompt:**
```
Create a buildings management page with:
- Header: Title, subtitle, "Add Building" button, "Manage Hierarchy" button
- Info banner about flexible hierarchy system
- Search bar with icon
- Stats cards row: Total Buildings, Cities, Total m², Building Types
- Buildings grid: Responsive cards (1 col mobile, 2 col tablet, 3 col desktop)
- Each building card:
  - Building name and usage type
  - Year built badge
  - Address with map pin icon (formatted German address)
  - Organizational unit badge (if assigned)
  - Heated area display
  - "View Details" and "Edit" buttons
- Empty state: Icon, message, "Add Building" button
- Loading state with spinner
- Use shadcn/ui Card, Badge, Button components
- Hover effects on cards
- Address formatting helper function
```

### Building Detail Page

**Prompt:**
```
Create a building detail page with:
- Header: Back button, building name, usage type, Edit button, Add Meter button
- Building info card: Address, Year Built, Usage Type, Heated Area (grid layout)
- Quick stats card: Total Meters, Main Meters, Sub Meters counts
- Meters section: Grid of meter cards with icons, labels, resource types
- Consumption chart: Bar chart showing monthly consumption by resource
- Each meter card: Icon, label, sector badge, resource/unit info, View/Edit buttons
- Empty state for meters with "Add Meter" CTA
- Use color-coded icons for meter types (electricity=yellow, heat=orange, water=blue)
- Responsive layout: 2/3 + 1/3 split on desktop
- Loading and error states
```

---

## Meters Management

### Meters List Page

**Prompt:**
```
Create a meters list page with:
- Header: Title, subtitle, Export CSV button, Add Meter button
- Simple list display: Each meter in bordered card
- Meter info: Label, Building name, Sector, Resource, Meter number
- Make/Model display on right side
- Empty state with message
- Loading spinner
- Error state with retry button
- Simple, clean layout focused on readability
```

### Meter Detail Page

**Prompt:**
```
Create a meter detail page with:
- Header: Back button, meter icon, label, sector badge, building/resource info, QR Code button, Refresh button
- Three info cards: Meter Information, Building link, Latest Reading
- Stats cards row: Total Imported, Total Exported, Net Consumption
- Tabs: "Chart" and "Data Table"
- Chart tab: Line chart with imported, exported, net consumption lines
- Data tab: Table with Time, Imported, Exported, Net columns
- Real-time polling (30s interval)
- Loading states for readings
- Error handling with retry
- Responsive chart container
- Use Recharts LineChart component
- Color-coded lines (imported=amber, exported=blue, net=green)
```

---

## Organizational Hierarchy Page

**Prompt:**
```
Create an organizational hierarchy management page with:
- Header: Title, subtitle, "Add Unit" button (opens dialog)
- Tree structure display: Nested cards showing parent-child relationships
- Each unit card:
  - Expand/collapse chevron (if has children)
  - Icon based on unit type (Building2, MapPin, Home, etc.)
  - Unit name and type badge
  - Role badge (Admin, User, Guest) based on permissions
  - Member count button
  - Invite user button (if permission)
  - Add child unit button (if permission)
  - Edit button (if permission)
  - Delete button (if permission)
- Indentation for nested levels (margin-left based on depth)
- All units expanded by default
- Empty state: Icon, message, "Create First Unit" button
- Add/Edit dialog:
  - Name input
  - Unit type selector (with icons)
  - Parent unit selector (dropdown with hierarchy display)
  - Create/Update button
- Permission-based button visibility
- Member management dialogs
- Use shadcn/ui Dialog, Select, Input components
- Recursive component for tree rendering
```

**Key Features:**
- Flexible hierarchy (any depth)
- Permission-based actions
- Visual tree structure
- Member management
- Unit type icons

---

## Readings Page

**Prompt:**
```
Create a meter readings analysis page with:
- Header: Title, subtitle, Refresh button
- Filters card:
  - Meter selector dropdown
  - Start date picker
  - End date picker
  - Interval selector (Auto, 30s, 15m, 1h)
  - Quick range buttons: Last 24h, 7d, 30d, 90d
- Readings display card:
  - Meter info header
  - Export CSV button
  - Data table: Time, Imported, Exported, Net columns
  - Row count display
  - Empty state: Icon, message, helpful text
  - Error state: Message, retry button
  - Loading spinner
- Responsive table with horizontal scroll
- Date formatting with date-fns
- CSV export functionality
- Real-time data from InfluxDB hook
```

---

## Invoices Page

**Prompt:**
```
Create an invoices management page with:
- Header: Title, Upload Invoice button
- Month filter dropdown (last 12 months)
- Data table:
  - Columns: Name (supplier), Type (resource), Period (start-end dates), Amount (EUR), Actions
  - Edit button in actions column
  - Empty state: Icon, message, Upload button
- Upload dialog: File upload, form fields
- Edit dialog: Form to edit invoice details
- Loading state
- Date formatting
- Currency formatting (€)
- Responsive table
- Use shadcn/ui Table component
```

---

## Profile Page

**Prompt:**
```
Create a user profile page with:
- Profile card:
  - First Name, Last Name inputs (grid layout)
  - Email input (disabled)
  - Name input
  - Language selector dropdown
  - Save button
- Newsletter card:
  - Title and description
  - Toggle switch for subscription
  - Confirmation status message
  - Warning if not confirmed
- Centered layout (max-w-2xl)
- Loading states
- Success/error toasts
- Form validation
- Use shadcn/ui Card, Input, Select, Switch components
```

---

## Admin Page

**Prompt:**
```
Create an admin panel page with:
- Header: Shield icon, title, description
- Tab navigation: Users, Roles, Integrations, Unassigned Devices, Audit Log
- Users tab:
  - Table: Email, Name, Role badge, Registered date, Last login, Role selector
  - Role update functionality
- Roles tab:
  - Table: User ID, OU ID, Role badge, Inherits badge, Status badge, Created date
- Integrations tab:
  - Empty state with "Add Integration" button
  - Integration management (future)
- Unassigned Devices tab:
  - Component for device management
- Audit Log tab:
  - Table: Timestamp, Action badge, Table name, Record ID, User ID
  - Last 100 entries
- Error testing card:
  - Test Error Capture button (Sentry)
  - Test Span Tracking button
  - Sentry status display
- Permission check: Only superadmin access
- Loading spinner
- Access denied state
- Use shadcn/ui Tabs, Table, Badge components
- German language labels
```

---

## Landing Page Components

### Hero Section

**Prompt:**
```
Create a hero section component with:
- Full-width section with gradient background
- Background image with low opacity overlay
- Large title: "Digital Basement" with logo icon
- Subtitle text
- Two CTA buttons: Primary "Start Monitoring", Secondary "View Demo"
- Features grid below: 3 cards with icons and labels
  - Real-time Monitoring
  - Role-based Access
  - Secure & Scalable
- Responsive typography (text-6xl on desktop, smaller on mobile)
- Padding: py-32 lg:py-40
- Container max-width
- Use Inter font
- Smooth transitions
```

### Feature Grid

**Prompt:**
```
Create a features grid section with:
- Centered header: Title, subtitle
- 4-column grid (responsive: 1 col mobile, 2 col tablet, 4 col desktop)
- 8 feature cards:
  - Icon (colored)
  - Title
  - Description
  - Hover effects (border color change)
- Card styling: Border, padding, hover transitions
- Icon colors match utility meter colors
- Use shadcn/ui Card component
```

### Property Hierarchy Section

**Prompt:**
```
Create a property hierarchy showcase section with:
- Centered header: Title, subtitle
- Breadcrumb navigation bar
- Property Manager View:
  - Badge label
  - Grid of unit cards
  - Each card: Unit name, meter list with icons and readings
  - Status badges (online, offline, warning)
- Tenant View:
  - Badge label
  - Single unit card with meter stats cards
  - Large reading numbers
- Color-coded meter icons
- Status color coding
- Mock data display
- Use shadcn/ui Card, Badge components
```

---

## Dialog Components

### Add Building Dialog

**Prompt:**
```
Create an add building dialog with:
- Dialog overlay and content
- Form fields:
  - Building Name (required)
  - Usage Type dropdown (Office, Residential, Retail, etc.)
  - Address autocomplete component
  - City, Postal Code (auto-filled from address)
  - Heated Area (m²) - number input
  - Year Built - number input
  - Organizational Unit selector (optional, hierarchical display)
- Form validation
- Submit button with loading state
- Cancel button
- Success/error toasts
- Reset form on close
- Max height with scroll
- Use shadcn/ui Dialog, Input, Select, Label components
```

### Add Meter Dialog (Multi-step)

**Prompt:**
```
Create a multi-step meter addition dialog with:
- Step 1: Meter Type Selection
  - Grid of type buttons (Electricity, Gas, Water, Heating Oil, Heat)
  - Each button: Icon, name, description, colored background
- Step 2: Manufacturer Selection
  - Search input
  - List of manufacturers
  - "Custom manufacturer" option
  - Custom input if selected
- Step 3: Model Selection
  - Search input
  - List of models (filtered by manufacturer)
  - "Custom model" option
  - Custom input if selected
- Step 4: Meter Details
  - Form: Label, Location, Meter Number, Unit
  - Form validation with react-hook-form + zod
  - Submit button
- Back button (except step 1)
- Step indicator
- Reset on close
- Loading states
- Use shadcn/ui Dialog, Button, Input, Form components
```

---

## Component Patterns

### Stat Cards

**Prompt:**
```
Create reusable stat card components with:
- Card container
- Header: Title text (small), Icon (right-aligned)
- Content: Large number (text-2xl font-bold), Description text (small, muted)
- Icon colors: Muted foreground or themed
- Responsive: Full width on mobile, auto on desktop
- Use shadcn/ui Card component
- Consistent spacing and typography
```

### Data Tables

**Prompt:**
```
Create data table components with:
- Search bar with icon
- Filter dropdowns (Location, Type, Make, etc.)
- Column visibility toggle dropdown
- Sortable columns (click header to sort)
- Responsive table with horizontal scroll
- Empty state row
- Loading skeleton
- Row hover effects
- Action buttons in last column
- Results count display
- Use shadcn/ui Table, Input, Select, DropdownMenu components
```

### Charts

**Prompt:**
```
Create chart components using Recharts:
- ResponsiveContainer wrapper
- BarChart: Multiple bars, custom colors, tooltips, legends
- LineChart: Multiple lines, custom colors, dashed lines for net values
- XAxis: Date formatting, angle rotation
- YAxis: Auto scaling
- Tooltip: Custom formatters, labels
- Legend: Positioned, styled
- Grid: Dashed lines
- Use utility meter colors for series
- Responsive height (300-400px)
- Loading states
- Empty states
```

### Empty States

**Prompt:**
```
Create empty state components with:
- Large icon (h-12 w-12, muted color)
- Title text (text-lg font-medium)
- Description text (muted)
- CTA button (if applicable)
- Centered layout
- Consistent spacing
- Use shadcn/ui Button component
```

### Loading States

**Prompt:**
```
Create loading state components with:
- Spinner: Border animation (border-4, border-primary, border-t-transparent, rounded-full, animate-spin)
- Loading text (muted)
- Centered layout
- Consistent sizing (h-8 w-8 for spinner)
- Use Tailwind animate-spin utility
```

### Badges

**Prompt:**
```
Create badge components with:
- Variants: default, secondary, outline, destructive
- Color coding for statuses (success, warning, error)
- Sector badges (electricity, water, heat) with custom colors
- Role badges (admin, user, guest)
- Small size (text-xs)
- Icon support (optional)
- Use shadcn/ui Badge component
```

---

## Navigation Patterns

### Sidebar Navigation

**Prompt:**
```
Create sidebar navigation with:
- Fixed position, left side
- Logo section at top (icon + text)
- Navigation items list:
  - Icon (Lucide React)
  - Label text
  - Active state (bg-primary, text-primary-foreground)
  - Hover state (bg-secondary)
  - Link to route
- User menu at bottom:
  - Avatar with initials
  - Name and "View Profile" text
  - Dropdown: Settings, Sign Out
- Mobile: Slide-out overlay, close on click outside
- Responsive: Hidden on mobile, visible on lg+
- Smooth transitions
- Use NavLink for active state
```

### Breadcrumbs

**Prompt:**
```
Create breadcrumb navigation with:
- Container with flex layout
- Icons between items (ChevronRight)
- Clickable items (except last)
- Icon for first item
- Muted text color for non-active items
- Responsive: Hide on mobile if too long
- Use shadcn/ui Breadcrumb component (if available)
```

---

## Form Patterns

### Form Layout

**Prompt:**
```
Create form layouts with:
- Grid layout for multiple fields (grid-cols-1 md:grid-cols-2)
- Label above input (space-y-2)
- Required field indicators (*)
- Error messages below inputs
- Submit button (full width or right-aligned)
- Cancel button (outline variant)
- Loading state on submit
- Form validation with react-hook-form + zod
- Use shadcn/ui Form, Input, Label, Button components
```

### Input Components

**Prompt:**
```
Create input components with:
- Label (text-sm font-medium)
- Input field (shadcn/ui Input)
- Placeholder text
- Required validation
- Error state styling
- Helper text (optional, muted)
- Icon support (left side)
- Disabled state styling
- Use shadcn/ui Input, Label components
```

---

## Responsive Design Patterns

### Grid Systems

**Prompt:**
```
Create responsive grid layouts:
- Stats cards: grid-cols-1 md:grid-cols-2 lg:grid-cols-5
- Building cards: grid-cols-1 md:grid-cols-2 lg:grid-cols-3
- Meter cards: grid-cols-1 md:grid-cols-2 lg:grid-cols-3
- Feature cards: grid-cols-1 md:grid-cols-2 lg:grid-cols-4
- Use Tailwind grid utilities
- Consistent gap spacing (gap-4, gap-6)
- Auto-fit columns
```

### Mobile-First Approach

**Prompt:**
```
Implement mobile-first responsive design:
- Base styles for mobile
- md: breakpoint (768px) for tablet
- lg: breakpoint (1024px) for desktop
- Hide/show elements with responsive utilities
- Stack vertically on mobile, horizontal on desktop
- Adjust padding/spacing per breakpoint
- Touch-friendly button sizes (min h-10)
- Full-width buttons on mobile
```

---

## Animation & Transitions

### Hover Effects

**Prompt:**
```
Create hover effects:
- Card hover: border-primary/20 transition
- Button hover: bg-secondary or scale
- Link hover: text-foreground from muted
- Smooth transitions (transition-smooth)
- No jarring movements
- Subtle color changes
- Use Tailwind transition utilities
```

### Loading Animations

**Prompt:**
```
Create loading animations:
- Spinner: animate-spin utility
- Skeleton loaders: Pulse animation
- Button loading: Disabled + spinner icon
- Page loading: Centered spinner + text
- Smooth, non-jarring animations
- Consistent timing (0.2s)
```

---

## Color Usage Guidelines

### Utility Meter Colors

**Prompt:**
```
Use utility meter colors consistently:
- Electricity: Yellow (hsl(var(--electricity)))
- Water: Blue (hsl(var(--water)))
- Heat/Gas: Orange/Red (hsl(var(--heating)))
- Icons: Match resource type color
- Badges: Use color for sector badges
- Charts: Use color for data series
- Backgrounds: Use color/10 for subtle backgrounds
- Borders: Use color/30 for borders
```

### Status Colors

**Prompt:**
```
Use status colors for feedback:
- Success: Green (hsl(var(--success))) - Confirmations, positive states
- Warning: Amber (hsl(var(--warning))) - Alerts, cautions
- Destructive: Red (hsl(var(--destructive))) - Errors, delete actions
- Muted: Gray (hsl(var(--muted-foreground))) - Secondary text, disabled
- Use consistently across all components
```

---

## Accessibility Patterns

### Keyboard Navigation

**Prompt:**
```
Implement keyboard navigation:
- Focus states on all interactive elements
- Tab order logical
- Enter/Space for buttons
- Escape to close dialogs
- Arrow keys for dropdowns
- Focus visible indicators
- Skip links for main content
- Use shadcn/ui components (accessibility built-in)
```

### Screen Reader Support

**Prompt:**
```
Ensure screen reader support:
- Semantic HTML elements
- ARIA labels where needed
- Alt text for icons/images
- Form labels properly associated
- Error messages announced
- Loading states announced
- Status changes announced
- Use shadcn/ui components (ARIA support built-in)
```

---

## Internationalization (i18n)

### Language Support

**Prompt:**
```
Implement language support:
- Language context provider
- Translation keys (en.ts, de.ts)
- Language switcher component
- Browser language detection
- User preference storage
- Dynamic text replacement
- Date/number formatting per locale
- RTL support (if needed)
- Use useLanguage hook throughout
```

---

## Error Handling Patterns

### Error States

**Prompt:**
```
Create error state components:
- Error icon (AlertTriangle)
- Error message text
- Retry button (if applicable)
- Helpful error descriptions
- Non-blocking (toast notifications)
- Inline form errors
- Page-level error boundaries
- Use shadcn/ui Alert component
- Toast notifications for actions
```

### Toast Notifications

**Prompt:**
```
Implement toast notifications:
- Success: Green, checkmark icon
- Error: Red, X icon
- Info: Blue, info icon
- Warning: Amber, warning icon
- Auto-dismiss (5s)
- Manual dismiss button
- Stack multiple toasts
- Position: Bottom-right
- Use shadcn/ui Toast component
```

---

## Performance Patterns

### Loading States

**Prompt:**
```
Implement loading states:
- Skeleton loaders for content
- Spinner for actions
- Progressive loading
- Optimistic updates
- Lazy loading for images
- Code splitting for routes
- Suspense boundaries
- Loading indicators for async operations
```

### Data Fetching

**Prompt:**
```
Implement data fetching patterns:
- React hooks for data fetching
- Loading states
- Error handling
- Refetch functionality
- Polling for real-time data
- Cache management
- Optimistic updates
- Use React Query or custom hooks
```

---

## Testing Considerations

### Component Testing

**Prompt:**
```
Create testable components:
- Isolated components
- Props interfaces
- Default props
- Error boundaries
- Loading states
- Empty states
- Mock data support
- Accessible selectors
- Use React Testing Library patterns
```

---

## File Structure Reference

```
src/
├── pages/              # Page components
│   ├── Dashboard.tsx
│   ├── Auth.tsx
│   ├── Buildings.tsx
│   ├── BuildingDetail.tsx
│   ├── Meters.tsx
│   ├── MeterDetail.tsx
│   ├── Readings.tsx
│   ├── Invoices.tsx
│   ├── OrganizationalHierarchy.tsx
│   ├── Profile.tsx
│   ├── Admin.tsx
│   └── ...
├── components/         # Reusable components
│   ├── ui/            # shadcn/ui components
│   ├── AppLayout.tsx
│   ├── Navigation.tsx
│   ├── Hero.tsx
│   ├── FeatureGrid.tsx
│   ├── TrialStatus.tsx
│   └── ...
├── hooks/             # Custom hooks
├── contexts/          # React contexts
├── locales/           # Translation files
└── lib/               # Utilities
```

---

## Summary

This Reverse PRD documents all GUI components, patterns, and provides detailed prompts for recreating similar implementations. Each prompt includes:

1. **Component structure** - Layout and organization
2. **Visual design** - Colors, spacing, typography
3. **Interactivity** - User interactions and states
4. **Responsive behavior** - Mobile-first approach
5. **Accessibility** - Keyboard navigation and screen readers
6. **Error handling** - Loading and error states
7. **Data integration** - API calls and state management

Use these prompts as specifications when rebuilding or creating similar components in new projects.

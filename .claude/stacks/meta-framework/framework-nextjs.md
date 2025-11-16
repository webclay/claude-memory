# AI RULESET: FRONTEND COMPONENT DEVELOPMENT

## ROLE
You are a frontend expert focused on creating performant and beautiful React components using React 19, Next.js 16, Shadcn UI, and Tailwind CSS. Your primary goal is to generate code that adheres to the project's foundational requirements as defined in `projectbrief.md` and the coding patterns outlined in `standards.md`.

## CORE INSTRUCTIONS
1. **Component Type**: Default to React Server Components. Only use `'use client'` when hooks like `useState` or `useEffect` are absolutely necessary.

2. **Structure**:

* Use functional components with strongly typed props (`interface`).
* Extract complex logic into custom hooks.
* Implement `<Suspense>` boundaries for components that fetch data.

3. **State Management**:

* Use `useState` for simple local state.
* For forms, use `<form>` with a Server Action. Manage state with `useActionState` and `useFormStatus`.
* For mutations, provide immediate UI feedback using `useOptimistic`.
* Handle loading/pending states with `useTransition`.

4. **Styling (Tailwind & Shadcn)**:

* Use Tailwind CSS utility classes directly. Merge conditional classes with the `cn()` utility.
* Build upon Shadcn UI components.
* The design MUST be responsive and mobile-first.
* Implement `dark:` variants.

5. **Design System Compliance**:

All components MUST strictly follow the design system defined at `/dashboard/admin/design-system`. This is the single source of truth for visual consistency. The design system covers:

* **Typography**: Heading hierarchy, font sizes, and weights
* **Colors**: Badge color system, semantic colors, theme colors
* **Components**: Buttons, badges, icons, inputs, etc.
* **Spacing**: Padding, margins, gaps, and layout spacing
* **Shadows & Effects**: Elevation, focus states, and interactions

**Design System Rules - Source of Truth:**

* **Typography Hierarchy**:
  - h1: `text-4xl font-bold`
  - h2: `text-3xl font-bold`
  - h3: `text-2xl font-bold`
  - h4: `text-xl font-bold`
  - h5: `text-lg font-bold`
  - h6: `text-base font-bold`
  - Page titles: `text-2xl font-semibold`
  - Drawer/Modal titles: `text-2xl font-bold`
  - ALWAYS match semantic HTML tags with corresponding Tailwind classes
  - NEVER use mismatched heading tags (e.g., h3 with text-lg)

* **Text**: Base size 14px (`text-sm`). Letter spacing -0.4px (`tracking-tight`). Max font weight for headings is `bold`, for other text is `semibold`.

* **Buttons**: Height MUST be h-8 (32px), h-9 (36px), or h-10 (40px). Never use custom heights.

* **Icons**: Base size 14px or 16px. Stroke width MUST be 2px (max). Always use HugeIcons library.

* **Badge Colors**: Strictly use defined color variants:
  - Blue (#f2f8fe / #61a3df): Languages
  - Purple (purple-100 / purple-700): Modules
  - Green (#f4f8f4 / #42b138): Success/Active
  - Red (#fff3f6 / #c13a66): Error/Inactive
  - Brown (#fdf5f3 / #a86444): Products
  - Gold (#FFEFC1 / #D7852D): Credits/Premium features
  - Yellow (yellow-100 / yellow-900): Warnings/Highlights

* **Spacing**: Use standard Tailwind spacing scale (gap-2, gap-4, gap-6, p-4, p-6, space-y-4, etc.)

* **Accessibility**: All components must be accessible. Use semantic HTML and ARIA attributes where necessary.

**IMPORTANT**: Before implementing any component, verify it matches the design system at `/dashboard/admin/design-system`. If unsure, check existing implementations in the codebase for patterns.

---

## 5.1 DESIGN SYSTEM COMPONENTS

### Buttons

**Available Variants:**
```tsx
import { Button } from "@/components/ui/button"

// Primary action button
<Button variant="default">Save Changes</Button>

// Destructive action (delete, remove, etc.)
<Button variant="destructive">Delete</Button>

// Outlined button
<Button variant="outline">Cancel</Button>

// Secondary action
<Button variant="secondary">Learn More</Button>

// Ghost button (minimal styling)
<Button variant="ghost">View Details</Button>

// Link button
<Button variant="link">Read Documentation</Button>
```

**Size Variants:**
```tsx
// Small: h-8 (32px) - use for compact UIs
<Button size="sm">Small Button</Button>

// Default: h-9 (36px) - standard button size
<Button size="default">Default Button</Button>

// Large: h-10 (40px) - use for prominent actions
<Button size="lg">Large Button</Button>

// Icon: square 36x36px - icon-only buttons
<Button size="icon">
  <HugeiconsIcon icon={IconName} size={16} strokeWidth={2} />
</Button>
```

**Button Best Practices:**
- Use `variant="default"` for primary actions
- Use `variant="destructive"` for dangerous actions (delete, remove)
- Use `variant="outline"` for secondary actions
- Use `variant="ghost"` for minimal actions in menus/dropdowns
- Icon buttons should use `size="icon"` or explicit width like `h-8 w-8 p-0`
- Always include `<span className="sr-only">` for icon-only buttons

**Button Icon Spacing (IMPORTANT):**
For buttons with both icons and text, use `gap-2` on the Button component itself instead of margin classes on the icon:

```tsx
// ✅ CORRECT: Use gap-2 on Button (preferred method)
<Button className="gap-2">
  <HugeiconsIcon icon={AiEditingIcon} size={16} strokeWidth={2} />
  Create New Item
</Button>

// ⚠️ ACCEPTABLE: Use mr-2 on icon (legacy method for forms)
<Button>
  <HugeiconsIcon icon={AiEditingIcon} size={16} strokeWidth={2} className="mr-2" />
  Generate Content
</Button>
```

**Icon Spacing Guidelines:**
- **Dashboard pages/components**: Use `className="gap-2"` on Button component
- **Module forms**: May use `className="mr-2"` on icon (for consistency with existing forms)
- **Loading states**: Icon with loading spinner should also maintain consistent spacing
- **Icon-only buttons**: No spacing needed (use `size="icon"`)

**Standard Icon Sizes in Buttons:**
- Primary/action buttons: `size={16} strokeWidth={2}`
- Small buttons: `size={14} strokeWidth={2}`
- Icon-only buttons: `size={16} strokeWidth={2}`

### Badges

**Available Variants:**
```tsx
import { Badge } from "@/components/ui/badge"
import { HugeiconsIcon } from '@hugeicons/react'
import { Package01Icon, Globe02Icon } from '@hugeicons/core-free-icons'

// Default badge
<Badge variant="default">Default</Badge>

// Secondary badge
<Badge variant="secondary">Secondary</Badge>

// Destructive badge (errors, warnings)
<Badge variant="destructive">Error</Badge>

// Outline badge
<Badge variant="outline">Outline</Badge>

// Blue badge - use for LANGUAGE indicators
<Badge variant="blue" className="text-xs px-2 py-0.5 flex items-center w-fit">
  <HugeiconsIcon icon={Globe02Icon} size={12} strokeWidth={2} className="mr-1" />
  English
</Badge>

// Purple badge - use for MODULE indicators
<Badge variant="purple" className="text-xs px-2 py-0.5 flex items-center w-fit">
  Module Name
</Badge>

// Green badge - use for SUCCESS/ACTIVE status
<Badge variant="green" className="text-xs px-2 py-0.5 flex items-center w-fit">
  Active
</Badge>

// Red badge - use for ERROR/INACTIVE status
<Badge variant="red" className="text-xs px-2 py-0.5 flex items-center w-fit">
  Inactive
</Badge>

// Brown badge - use for PRODUCT indicators
<Badge variant="brown" className="text-xs px-2 py-0.5 flex items-center w-fit">
  <HugeiconsIcon icon={Package01Icon} size={12} strokeWidth={2} className="mr-1" />
  Product Name
</Badge>

// Gold badge - use for CREDITS/PREMIUM features
<Badge variant="gold" className="text-xs px-2 py-0.5 flex items-center w-fit">
  <HugeiconsIcon icon={Coins01Icon} size={12} strokeWidth={2} className="mr-1" />
  Credits
</Badge>

// Yellow badge - use for WARNINGS/HIGHLIGHTS
<Badge variant="yellow" className="text-xs px-2 py-0.5 flex items-center w-fit">
  Warning
</Badge>
```

**Badge Color System:**
| Variant | Use Case | Background | Text Color |
|---------|----------|------------|------------|
| `blue` | Languages | `#f2f8fe` | `#61a3df` |
| `purple` | Modules | `purple-100` | `purple-700` |
| `green` | Success/Active | `#f4f8f4` | `#42b138` |
| `red` | Error/Inactive | `#fff3f6` | `#c13a66` |
| `brown` | Products | `#fdf5f3` | `#a86444` |
| `gold` | Credits/Premium | `#FFEFC1` | `#D7852D` |
| `yellow` | Warnings/Highlights | `#fffaef` | `#fcb700` |

**Badge Best Practices:**
- Use `text-xs px-2 py-0.5` for standard badge sizing
- Include `flex items-center w-fit` for badges with icons
- Icons should be 12px with 2px stroke width
- Icon margin: `className="mr-1"` for left icon
- Always use the correct variant for semantic meaning

### Icons (HugeIcons)

**Standard Icon Usage:**
```tsx
import { HugeiconsIcon } from '@hugeicons/react'
import { Package01Icon, Globe02Icon, ViewIcon, Delete01Icon, MoreVerticalIcon } from '@hugeicons/core-free-icons'

// Standard icon size: 16px with 2px stroke
<HugeiconsIcon icon={ViewIcon} size={16} strokeWidth={2} />

// Small icon (in badges): 12px with 2px stroke
<HugeiconsIcon icon={Package01Icon} size={12} strokeWidth={2} className="mr-1" />

// Icon with margin for spacing in buttons/menu items
<HugeiconsIcon icon={Delete01Icon} size={16} strokeWidth={2} className="mr-2" />
```

**Icon Size Guidelines:**
- **12px**: Use in badges
- **14px**: Use in small UI elements (rarely used)
- **16px**: Standard size for buttons, dropdowns, menu items
- **Stroke width**: Always 2px (max) for consistency

**Common Icons:**
- `MoreVerticalIcon`: Action menu triggers (vertical dots)
- `ViewIcon`: View/preview actions
- `Delete01Icon`: Delete actions
- `Package01Icon`: Product indicators
- `Globe02Icon`: Language indicators
- `Cancel01Icon`: Close/dismiss actions

### Typography

**Heading Hierarchy (MUST FOLLOW):**
```tsx
// Page titles (main page heading)
<h1 className="text-2xl font-semibold">Products</h1>

// Major sections
<h2 className="text-3xl font-bold">Section Title</h2>

// Drawer/Modal titles and subsections
<h3 className="text-2xl font-bold">Drawer Title</h3>

// Card/component headings
<h4 className="text-xl font-bold">Card Heading</h4>

// Small section headings
<h5 className="text-lg font-bold">Subsection Heading</h5>

// Minimal headings
<h6 className="text-base font-bold">Minor Heading</h6>
```

**Text Sizes:**
```tsx
// Headers in tables (uppercase, muted)
<div className="text-xs font-medium text-muted-foreground">HEADER TEXT</div>

// Standard text (default size: 14px)
<p className="text-sm">Standard paragraph text</p>

// Small text
<p className="text-xs">Small descriptive text</p>
```

**Typography Rules:**
- **Heading Tags**: ALWAYS match semantic HTML tags (h1-h6) with corresponding Tailwind classes
- **Heading Font Weight**: All headings use `font-bold` except page titles which use `font-semibold`
- **Body Text**: Max font weight is `semibold` (never use `bold` for body text)
- **Table Headers**: Always uppercase with `text-xs font-medium text-muted-foreground`
- **Letter Spacing**: `-0.4px` for tighter text (use `tracking-tight`)
- **Default Text Color**: Use `text-foreground` or omit (defaults to foreground)
- **Muted Text**: Use `text-muted-foreground` for secondary information
- **NEVER**: Mix heading tags with wrong sizes (e.g., h3 with text-lg)

### Spacing & Layout

**Common Spacing Patterns:**
```tsx
// Card/section padding
<div className="p-6">Content</div>

// Vertical spacing between sections
<div className="space-y-6">
  <section>...</section>
  <section>...</section>
</div>

// Horizontal spacing (flex items)
<div className="flex gap-2">
  <Button>Action 1</Button>
  <Button>Action 2</Button>
</div>

// Tight spacing for related items
<div className="space-y-2">
  <Label>Field Label</Label>
  <Input />
</div>
```

**Spacing Scale:**
- `gap-1` / `space-y-1`: 4px - Very tight spacing
- `gap-2` / `space-y-2`: 8px - Tight spacing for related items (labels + inputs)
- `gap-4` / `space-y-4`: 16px - Standard spacing
- `gap-6` / `space-y-6`: 24px - Generous spacing between sections
- `p-6`: 24px - Standard card/section padding

### Responsive Design

**Mobile-First Approach:**
```tsx
// Stack vertically on mobile, horizontal on desktop
<div className="flex flex-col md:flex-row gap-4">
  <div>Column 1</div>
  <div>Column 2</div>
</div>

// Adjust padding for mobile
<div className="p-4 md:p-6">Content</div>

// Drawer width adjustments
<div className="w-full md:w-[750px]">
  Drawer Content
</div>

// Table containers with horizontal scroll
<div className="rounded-md border overflow-x-auto">
  <Table>...</Table>
</div>
```

**Breakpoints:**
- `sm`: 640px
- `md`: 768px
- `lg`: 1024px
- `xl`: 1280px

---

## 6. UI PATTERNS

### 6.1 Data Tables (TanStack Table)

**Standard Table Structure:**
```tsx
import { Table, TableBody, TableCell, TableHead, TableHeader, TableRow } from "@/components/ui/table"
import { Skeleton } from "@/components/ui/skeleton"

// Main table wrapper
<div className="rounded-md border">
  <Table>
    <TableHeader className="bg-muted/50">
      <TableRow>
        <TableHead style={{ width: "30%" }}>Column Name</TableHead>
        {/* ... more columns */}
      </TableRow>
    </TableHeader>
    <TableBody>
      {/* table rows */}
    </TableBody>
  </Table>
</div>
```

**Key Rules:**
- Header background: `className="bg-muted/50"` for subtle grey background
- Use column meta for sizing: `meta: { size: "30%" }`
- Headers use uppercase text with muted foreground: `<div className="text-xs font-medium text-muted-foreground">HEADER</div>`
- Always provide a skeleton loader that matches table structure

**Skeleton Loader Pattern:**
```tsx
function SkeletonTable() {
  return (
    <div className="rounded-md border">
      <Table>
        <TableHeader className="bg-muted/50">
          <TableRow>
            <TableHead style={{ width: "30%" }}>
              <Skeleton className="h-3 w-[60px]" />
            </TableHead>
            {/* Match all columns */}
          </TableRow>
        </TableHeader>
        <TableBody>
          {[...Array(5)].map((_, i) => (
            <TableRow key={i}>
              <TableCell>
                <Skeleton className="h-5 w-[200px]" />
              </TableCell>
              {/* Match all cells with appropriate sizes */}
            </TableRow>
          ))}
        </TableBody>
      </Table>
    </div>
  )
}
```

**Skeleton Guidelines:**
- Show 5 rows by default
- Match header background styling
- Use appropriate heights: h-3 for headers, h-5 for text, h-6 for badges
- Use `rounded-full` for badge-shaped content
- Always match the actual table structure

### 6.2 Action Menus (Dropdown Pattern)

**Standard Action Column:**
```tsx
import { DropdownMenu, DropdownMenuContent, DropdownMenuItem, DropdownMenuLabel, DropdownMenuSeparator, DropdownMenuTrigger } from "@/components/ui/dropdown-menu"
import { MoreVerticalIcon, ViewIcon, Delete01Icon } from '@hugeicons/core-free-icons'

{
  id: "actions",
  header: () => <div className="text-right text-xs font-medium text-muted-foreground">ACTIONS</div>,
  cell: ({ row, table }) => {
    const meta = table.options.meta as { onViewClick?: (row: T) => void; onDeleteClick?: (row: T) => void };
    return (
      <div className="flex justify-end">
        <DropdownMenu>
          <DropdownMenuTrigger asChild>
            <Button variant="ghost" className="h-8 w-8 p-0">
              <span className="sr-only">Open menu</span>
              <HugeiconsIcon icon={MoreVerticalIcon} size={16} strokeWidth={2} />
            </Button>
          </DropdownMenuTrigger>
          <DropdownMenuContent align="end">
            <DropdownMenuLabel>Actions</DropdownMenuLabel>
            <DropdownMenuSeparator />
            <DropdownMenuItem onClick={() => meta?.onViewClick?.(row.original)}>
              <HugeiconsIcon icon={ViewIcon} size={16} strokeWidth={2} className="mr-2" />
              View Details
            </DropdownMenuItem>
            <DropdownMenuItem
              onClick={() => meta?.onDeleteClick?.(row.original)}
              className="text-red-600 hover:text-red-700 focus:text-red-700"
            >
              <HugeiconsIcon icon={Delete01Icon} size={16} strokeWidth={2} className="mr-2" />
              Delete
            </DropdownMenuItem>
          </DropdownMenuContent>
        </DropdownMenu>
      </div>
    );
  },
}
```

**Key Rules:**
- Always use `MoreVerticalIcon` (vertical dots) as the trigger icon
- Trigger button: `variant="ghost" className="h-8 w-8 p-0"`
- Icons in menu items: 16px size, 2px stroke width, `className="mr-2"`
- Destructive actions: `className="text-red-600 hover:text-red-700 focus:text-red-700"`
- Always include `<span className="sr-only">Open menu</span>` for accessibility
- Align dropdown to end: `align="end"`

### 6.3 Confirmation Dialogs

**Standard Destructive Action Dialog:**
```tsx
import { AlertDialog, AlertDialogAction, AlertDialogCancel, AlertDialogContent, AlertDialogDescription, AlertDialogFooter, AlertDialogHeader, AlertDialogTitle } from "@/components/ui/alert-dialog"

<AlertDialog open={deleteDialogOpen} onOpenChange={setDeleteDialogOpen}>
  <AlertDialogContent>
    <AlertDialogHeader>
      <AlertDialogTitle>Are you sure?</AlertDialogTitle>
      <AlertDialogDescription>
        This action cannot be undone. This will permanently delete the {itemType} "{itemName}".
      </AlertDialogDescription>
    </AlertDialogHeader>
    <AlertDialogFooter>
      <AlertDialogCancel>Cancel</AlertDialogCancel>
      <AlertDialogAction
        className="bg-red-600 hover:bg-red-700 focus:ring-red-500"
        onClick={handleDeleteConfirmed}
      >
        Delete
      </AlertDialogAction>
    </AlertDialogFooter>
  </AlertDialogContent>
</AlertDialog>
```

**Key Rules:**
- Use AlertDialog for destructive/important confirmations
- Destructive confirm button: `className="bg-red-600 hover:bg-red-700 focus:ring-red-500"`
- Always include item name in description for clarity
- Description should clearly state consequences ("This action cannot be undone")
- Cancel button appears first (left), action button second (right)

### 6.4 Data Fetching & Refetching Pattern

**Standard Fetch Pattern with Refetch:**
```tsx
const fetchData = React.useCallback(async () => {
  if (!session?.user?.id) return;

  setLoading(true);
  setError(null);

  try {
    const response = await fetch('/api/endpoint', {
      method: 'GET',
      headers: { 'Content-Type': 'application/json' },
    });

    if (!response.ok) {
      const errorData = await response.json().catch(() => ({}));
      throw new Error(errorData.error || `HTTP ${response.status}`);
    }

    const data = await response.json();
    if (data.success && Array.isArray(data.items)) {
      setItems(data.items);
    } else {
      setItems([]);
    }
  } catch (err) {
    console.error('Error fetching data:', err);
    setError(err instanceof Error ? err.message : 'Failed to fetch data');
    setItems([]);
  } finally {
    setLoading(false);
  }
}, [session?.user?.id]);

// Initial fetch
React.useEffect(() => {
  if (isClient && isAuthenticated && !authLoading) {
    fetchData();
  }
}, [isClient, isAuthenticated, authLoading, fetchData]);

// Refetch after mutations (e.g., delete)
const handleDeleteConfirmed = async () => {
  // ... perform delete
  if (response.ok) {
    await fetchData(); // Refetch to update UI
    setDialogOpen(false);
  }
};
```

**Key Rules:**
- Extract fetch logic into `useCallback` for reusability
- Call `fetchData()` after successful mutations to refresh the UI
- Don't filter local state after mutations - refetch from API for consistency
- Include proper error handling with user-friendly messages
- Always set loading states appropriately

---

### 6.5 Drawers (Vaul)

**Standard Right-Side Drawer Pattern:**
```tsx
import { Drawer as VaulDrawer } from "vaul";
import { Button } from "@/components/ui/button";
import { Badge } from "@/components/ui/badge";
import { HugeiconsIcon } from '@hugeicons/react';
import { Cancel01Icon, Package01Icon, Globe02Icon } from '@hugeicons/core-free-icons';

<VaulDrawer.Root open={drawerOpen} onOpenChange={setDrawerOpen} direction="right">
  <VaulDrawer.Portal>
    <VaulDrawer.Overlay className="fixed inset-0 bg-black/40 z-40" />
    <VaulDrawer.Content
      className="fixed right-2 top-2 bottom-2 z-50 outline-none w-[750px] flex flex-col"
      style={{ '--initial-transform': 'calc(100% + 8px)' } as React.CSSProperties}
    >
      <div className="bg-background h-full w-full grow flex flex-col rounded-[16px] shadow-lg">
        {/* Header */}
        <div className="border-b pb-4 px-6 pt-6">
          <div className="flex items-center justify-between">
            <VaulDrawer.Title className="text-2xl font-bold">
              {title}
            </VaulDrawer.Title>
            <VaulDrawer.Close asChild>
              <Button variant="ghost" size="icon">
                <HugeiconsIcon icon={Cancel01Icon} size={16} strokeWidth={2} />
              </Button>
            </VaulDrawer.Close>
          </div>

          {/* Metadata Badges */}
          <div className="flex items-center gap-3 flex-wrap mt-4">
            <Badge variant="purple" className="text-xs px-2 py-0.5">
              Module Name
            </Badge>
            <Badge variant="brown" className="text-xs px-2 py-0.5 flex items-center">
              <HugeiconsIcon icon={Package01Icon} size={12} strokeWidth={2} className="mr-1" />
              Product Name
            </Badge>
            <Badge variant="blue" className="text-xs px-2 py-0.5 flex items-center">
              <HugeiconsIcon icon={Globe02Icon} size={12} strokeWidth={2} className="mr-1" />
              Language
            </Badge>
          </div>
        </div>

        {/* Scrollable Content */}
        <div className="px-6 py-6 space-y-6 flex-grow overflow-y-auto">
          {/* Content goes here */}
        </div>
      </div>
    </VaulDrawer.Content>
  </VaulDrawer.Portal>
</VaulDrawer.Root>
```

**Drawer Best Practices:**
- **Direction**: Always use `direction="right"` for detail views
- **Width**: Standard width is `w-[750px]` for detail drawers
- **Overlay**: Dark overlay with `bg-black/40 z-40`
- **Spacing**: `right-2 top-2 bottom-2` for slight gap from viewport edges
- **Border Radius**: `rounded-[16px]` for smooth corners
- **Header Structure**:
  - Top padding: `pt-6 px-6 pb-4`
  - Border bottom: `border-b`
  - Title: `text-2xl font-bold` (follows design system h3 heading standard)
  - Close button: Ghost variant with Cancel icon
- **Metadata**: Display badges in `flex gap-3 flex-wrap mt-4` for responsive wrapping
- **Content Area**: Use `flex-grow overflow-y-auto` for scrollable content
- **Z-index**: Overlay at `z-40`, content at `z-50`

---

### 6.6 State Management Patterns

**Client-Side Hydration Safety:**
```tsx
export function MyComponent() {
  const [isClient, setIsClient] = React.useState(false);
  const { session, isLoading: authLoading, isAuthenticated } = useAuthStatus();

  // Prevent hydration mismatch
  React.useEffect(() => {
    setIsClient(true);
  }, []);

  // Early return to prevent rendering during SSR
  if (!isClient || authLoading) {
    return null; // Or return a skeleton loader
  }

  // Safe to render authenticated content
  return <div>Content</div>;
}
```

**Table Meta Pattern (Passing Callbacks):**
```tsx
// Define callbacks outside table columns
const handleViewClick = (row: DataType) => {
  // Handle view action
};

const handleDeleteClick = (row: DataType) => {
  // Handle delete action
};

// Pass via table meta
const table = useReactTable({
  data,
  columns,
  // ... other options
  meta: {
    onViewClick: handleViewClick,
    onDeleteClick: handleDeleteClick,
  },
});

// Access in column definitions
{
  id: "actions",
  cell: ({ row, table }) => {
    const meta = table.options.meta as {
      onViewClick?: (row: DataType) => void;
      onDeleteClick?: (row: DataType) => void;
    };
    return (
      <Button onClick={() => meta?.onViewClick?.(row.original)}>
        View
      </Button>
    );
  },
}
```

**Dialog State Pattern:**
```tsx
const [dialogOpen, setDialogOpen] = React.useState(false);
const [itemToDelete, setItemToDelete] = React.useState<ItemType | null>(null);

// Open dialog with item
const handleDeleteClick = (item: ItemType) => {
  setItemToDelete(item);
  setDialogOpen(true);
};

// Confirm and cleanup
const handleConfirm = async () => {
  if (!itemToDelete) return;

  // Perform action
  await deleteItem(itemToDelete.id);

  // Cleanup state
  setDialogOpen(false);
  setItemToDelete(null);
};
```

---

### 6.7 Error Handling Patterns

**API Error Display:**
```tsx
const [error, setError] = React.useState<string | null>(null);

// In fetch function
try {
  // ... fetch logic
} catch (err) {
  console.error('Error fetching data:', err);
  setError(err instanceof Error ? err.message : 'Failed to fetch data');
}

// Display error
if (error) {
  return (
    <div className="p-8 text-center">
      <p className="text-destructive mb-2">Error loading data</p>
      <p className="text-sm text-muted-foreground">{error}</p>
    </div>
  );
}
```

**Form Validation Error Pattern:**
```tsx
// Show error below input
<div className="space-y-2">
  <Label htmlFor="email">Email</Label>
  <Input
    id="email"
    type="email"
    aria-invalid={!!errors.email}
    className={errors.email ? 'border-destructive' : ''}
  />
  {errors.email && (
    <p className="text-xs text-destructive">{errors.email}</p>
  )}
</div>
```

---

### 6.8 Loading States

**Full Page Loading:**
```tsx
if (loading) {
  return <SkeletonTable />;
}
```

**Inline Loading (Buttons):**
```tsx
import { Loader2 } from "lucide-react";

<Button disabled={isLoading}>
  {isLoading && <Loader2 className="mr-2 h-4 w-4 animate-spin" />}
  {isLoading ? 'Saving...' : 'Save Changes'}
</Button>
```

**Conditional Loading:**
```tsx
{loadingTemplates ? (
  <div className="flex items-center gap-2">
    <Loader2 className="h-4 w-4 animate-spin" />
    <span className="text-sm text-muted-foreground">Loading...</span>
  </div>
) : (
  <select>...</select>
)}
```
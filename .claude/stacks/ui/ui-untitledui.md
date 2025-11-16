# Untitled UI React Integration Guide

## Overview

Untitled UI React is a comprehensive collection of 5,000+ production-ready React components built with Tailwind CSS and React Aria. It emphasizes a copy-paste approach for rapid development with professional design, accessibility, and consistency.

**Key Features:**
- **5,000+ Components**: Buttons, modals, tables, charts, forms, and more
- **250+ Page Examples**: Dashboards, landing pages, pricing templates
- **Tailwind CSS Integration**: Built with Tailwind v4.1 utility classes
- **React Aria**: WAI-ARIA accessibility standards built-in
- **TypeScript Support**: Full type safety with TypeScript v5.8
- **Figma Sync**: Design and code stay synchronized
- **Dark Mode**: Native support with CSS variables
- **Copy-Paste Philosophy**: Own your code, no dependencies

**When to Use:**
- Building SaaS dashboards and admin panels
- Need production-ready components immediately
- Want professional design without custom design work
- Require accessible components out of the box
- Building with Next.js, Vite, or modern React

## Licensing

### Free (Open Source)
- MIT-licensed components
- CLI installation
- Framework starter kits
- 2,000+ icons included
- **Cost**: $0

### PRO SOLO
- 5,000+ components
- 250+ page examples
- Figma synchronization
- Lifetime updates
- Single user license
- **Cost**: $349 (regular $499)

### PRO TEAM
- Same as PRO SOLO
- Up to 5 users
- Commercial project usage rights
- **Cost**: $699 (regular $999)

## Installation

**📚 Official Documentation**: [Untitled UI - Getting Started](https://www.untitledui.com/react)

Follow the official setup guide for the latest installation instructions. Untitled UI offers CLI setup for new projects and manual installation for existing projects.

### Quick Reference

**For New Projects:**
The CLI creates a complete setup with pre-configured Tailwind CSS, TypeScript, and all components.

**For Existing Projects:**
Manual setup involves:
1. Install dependencies (Tailwind CSS, React Aria, utilities)
2. Configure Tailwind config
3. Set up CSS variables
4. Copy components as needed

**Note**: Installation commands and requirements may change. Always refer to the official docs above for the current setup method.

**tailwind.config.ts**
```typescript
import type { Config } from 'tailwindcss';

const config: Config = {
  darkMode: ['class'],
  content: [
    './pages/**/*.{js,ts,jsx,tsx,mdx}',
    './components/**/*.{js,ts,jsx,tsx,mdx}',
    './app/**/*.{js,ts,jsx,tsx,mdx}',
  ],
  theme: {
    extend: {
      colors: {
        // Brand colors
        brand: {
          50: '#f0f9ff',
          100: '#e0f2fe',
          200: '#bae6fd',
          300: '#7dd3fc',
          400: '#38bdf8',
          500: '#0ea5e9',
          600: '#0284c7',
          700: '#0369a1',
          800: '#075985',
          900: '#0c4a6e',
          950: '#082f49',
        },
        // Gray scale
        gray: {
          50: '#f9fafb',
          100: '#f3f4f6',
          200: '#e5e7eb',
          300: '#d1d5db',
          400: '#9ca3af',
          500: '#6b7280',
          600: '#4b5563',
          700: '#374151',
          800: '#1f2937',
          900: '#111827',
          950: '#030712',
        },
      },
      borderRadius: {
        lg: 'var(--radius)',
        md: 'calc(var(--radius) - 2px)',
        sm: 'calc(var(--radius) - 4px)',
      },
    },
  },
  plugins: [],
};

export default config;
```

**3. Add CSS Variables:**

**app/globals.css**
```css
@tailwind base;
@tailwind components;
@tailwind utilities;

@layer base {
  :root {
    --radius: 0.5rem;

    /* Light mode colors */
    --background: 0 0% 100%;
    --foreground: 222.2 84% 4.9%;

    --card: 0 0% 100%;
    --card-foreground: 222.2 84% 4.9%;

    --popover: 0 0% 100%;
    --popover-foreground: 222.2 84% 4.9%;

    --primary: 221.2 83.2% 53.3%;
    --primary-foreground: 210 40% 98%;

    --secondary: 210 40% 96.1%;
    --secondary-foreground: 222.2 47.4% 11.2%;

    --muted: 210 40% 96.1%;
    --muted-foreground: 215.4 16.3% 46.9%;

    --accent: 210 40% 96.1%;
    --accent-foreground: 222.2 47.4% 11.2%;

    --destructive: 0 84.2% 60.2%;
    --destructive-foreground: 210 40% 98%;

    --border: 214.3 31.8% 91.4%;
    --input: 214.3 31.8% 91.4%;
    --ring: 221.2 83.2% 53.3%;
  }

  .dark {
    --background: 222.2 84% 4.9%;
    --foreground: 210 40% 98%;

    --card: 222.2 84% 4.9%;
    --card-foreground: 210 40% 98%;

    --popover: 222.2 84% 4.9%;
    --popover-foreground: 210 40% 98%;

    --primary: 217.2 91.2% 59.8%;
    --primary-foreground: 222.2 47.4% 11.2%;

    --secondary: 217.2 32.6% 17.5%;
    --secondary-foreground: 210 40% 98%;

    --muted: 217.2 32.6% 17.5%;
    --muted-foreground: 215 20.2% 65.1%;

    --accent: 217.2 32.6% 17.5%;
    --accent-foreground: 210 40% 98%;

    --destructive: 0 62.8% 30.6%;
    --destructive-foreground: 210 40% 98%;

    --border: 217.2 32.6% 17.5%;
    --input: 217.2 32.6% 17.5%;
    --ring: 224.3 76.3% 48%;
  }
}

@layer base {
  * {
    @apply border-border;
  }
  body {
    @apply bg-background text-foreground;
  }
}
```

**4. Utility Functions:**

**lib/utils.ts**
```typescript
import { clsx, type ClassValue } from 'clsx';
import { twMerge } from 'tailwind-merge';

export function cn(...inputs: ClassValue[]) {
  return twMerge(clsx(inputs));
}
```

## Component Patterns

### Button Components

**components/ui/Button.tsx**
```typescript
import { forwardRef } from 'react';
import { Button as AriaButton, ButtonProps as AriaButtonProps } from 'react-aria-components';
import { cn } from '@/lib/utils';

export interface ButtonProps extends AriaButtonProps {
  variant?: 'primary' | 'secondary' | 'outline' | 'ghost' | 'destructive';
  size?: 'sm' | 'md' | 'lg';
}

export const Button = forwardRef<HTMLButtonElement, ButtonProps>(
  ({ className, variant = 'primary', size = 'md', ...props }, ref) => {
    return (
      <AriaButton
        ref={ref}
        className={cn(
          // Base styles
          'inline-flex items-center justify-center rounded-md font-medium',
          'transition-colors focus-visible:outline-none focus-visible:ring-2',
          'focus-visible:ring-ring focus-visible:ring-offset-2',
          'disabled:pointer-events-none disabled:opacity-50',

          // Variants
          variant === 'primary' && 'bg-primary text-primary-foreground hover:bg-primary/90',
          variant === 'secondary' && 'bg-secondary text-secondary-foreground hover:bg-secondary/80',
          variant === 'outline' && 'border border-input bg-background hover:bg-accent hover:text-accent-foreground',
          variant === 'ghost' && 'hover:bg-accent hover:text-accent-foreground',
          variant === 'destructive' && 'bg-destructive text-destructive-foreground hover:bg-destructive/90',

          // Sizes
          size === 'sm' && 'h-9 px-3 text-sm',
          size === 'md' && 'h-10 px-4 py-2',
          size === 'lg' && 'h-11 px-8 text-lg',

          className
        )}
        {...props}
      />
    );
  }
);

Button.displayName = 'Button';
```

**Usage:**
```typescript
import { Button } from '@/components/ui/Button';

export function ButtonExamples() {
  return (
    <div className="flex gap-4">
      <Button variant="primary">Primary</Button>
      <Button variant="secondary">Secondary</Button>
      <Button variant="outline">Outline</Button>
      <Button variant="ghost">Ghost</Button>
      <Button variant="destructive">Delete</Button>

      {/* Sizes */}
      <Button size="sm">Small</Button>
      <Button size="md">Medium</Button>
      <Button size="lg">Large</Button>

      {/* Disabled */}
      <Button isDisabled>Disabled</Button>
    </div>
  );
}
```

### Input Components

**components/ui/Input.tsx**
```typescript
import { forwardRef } from 'react';
import { TextField, Label, Input as AriaInput, TextFieldProps } from 'react-aria-components';
import { cn } from '@/lib/utils';

export interface InputProps extends Omit<TextFieldProps, 'children'> {
  label?: string;
  placeholder?: string;
  description?: string;
  errorMessage?: string;
}

export const Input = forwardRef<HTMLInputElement, InputProps>(
  ({ label, placeholder, description, errorMessage, className, ...props }, ref) => {
    return (
      <TextField {...props} className={cn('flex flex-col gap-1', className)}>
        {label && (
          <Label className="text-sm font-medium text-foreground">
            {label}
          </Label>
        )}

        <AriaInput
          ref={ref}
          placeholder={placeholder}
          className={cn(
            'flex h-10 w-full rounded-md border border-input',
            'bg-background px-3 py-2 text-sm',
            'ring-offset-background placeholder:text-muted-foreground',
            'focus-visible:outline-none focus-visible:ring-2',
            'focus-visible:ring-ring focus-visible:ring-offset-2',
            'disabled:cursor-not-allowed disabled:opacity-50',
            errorMessage && 'border-destructive'
          )}
        />

        {description && (
          <p className="text-sm text-muted-foreground">{description}</p>
        )}

        {errorMessage && (
          <p className="text-sm text-destructive">{errorMessage}</p>
        )}
      </TextField>
    );
  }
);

Input.displayName = 'Input';
```

**Usage:**
```typescript
import { Input } from '@/components/ui/Input';

export function InputExample() {
  return (
    <div className="space-y-4">
      <Input
        label="Email"
        placeholder="you@example.com"
        type="email"
      />

      <Input
        label="Password"
        placeholder="••••••••"
        type="password"
        description="Must be at least 8 characters"
      />

      <Input
        label="Username"
        placeholder="johndoe"
        errorMessage="Username is already taken"
      />
    </div>
  );
}
```

### Modal/Dialog Components

**components/ui/Modal.tsx**
```typescript
import { Dialog, Modal as AriaModal, ModalOverlay, Heading } from 'react-aria-components';
import { cn } from '@/lib/utils';

export interface ModalProps {
  isOpen: boolean;
  onClose: () => void;
  title?: string;
  children: React.ReactNode;
  size?: 'sm' | 'md' | 'lg' | 'xl';
}

export function Modal({ isOpen, onClose, title, children, size = 'md' }: ModalProps) {
  return (
    <ModalOverlay
      isOpen={isOpen}
      onOpenChange={onClose}
      className={cn(
        'fixed inset-0 z-50 bg-background/80 backdrop-blur-sm',
        'data-[entering]:animate-in data-[exiting]:animate-out',
        'data-[entering]:fade-in-0 data-[exiting]:fade-out-0'
      )}
    >
      <AriaModal
        className={cn(
          'fixed left-[50%] top-[50%] z-50 translate-x-[-50%] translate-y-[-50%]',
          'w-full max-h-[90vh] overflow-y-auto',
          'bg-background border border-border rounded-lg shadow-lg',
          'data-[entering]:animate-in data-[exiting]:animate-out',
          'data-[entering]:fade-in-0 data-[exiting]:fade-out-0',
          'data-[entering]:zoom-in-95 data-[exiting]:zoom-out-95',
          'data-[entering]:slide-in-from-left-1/2 data-[entering]:slide-in-from-top-[48%]',
          'data-[exiting]:slide-out-to-left-1/2 data-[exiting]:slide-out-to-top-[48%]',
          size === 'sm' && 'max-w-sm',
          size === 'md' && 'max-w-md',
          size === 'lg' && 'max-w-lg',
          size === 'xl' && 'max-w-xl'
        )}
      >
        <Dialog className="outline-none">
          {title && (
            <Heading slot="title" className="text-lg font-semibold p-6 pb-4">
              {title}
            </Heading>
          )}

          <div className="p-6 pt-0">
            {children}
          </div>
        </Dialog>
      </AriaModal>
    </ModalOverlay>
  );
}
```

**Usage:**
```typescript
import { useState } from 'react';
import { Modal } from '@/components/ui/Modal';
import { Button } from '@/components/ui/Button';

export function ModalExample() {
  const [isOpen, setIsOpen] = useState(false);

  return (
    <>
      <Button onPress={() => setIsOpen(true)}>
        Open Modal
      </Button>

      <Modal
        isOpen={isOpen}
        onClose={() => setIsOpen(false)}
        title="Confirm Action"
        size="md"
      >
        <p className="text-sm text-muted-foreground mb-4">
          Are you sure you want to proceed with this action?
        </p>

        <div className="flex justify-end gap-3">
          <Button variant="outline" onPress={() => setIsOpen(false)}>
            Cancel
          </Button>
          <Button variant="primary" onPress={() => setIsOpen(false)}>
            Confirm
          </Button>
        </div>
      </Modal>
    </>
  );
}
```

### Table Components

**components/ui/Table.tsx**
```typescript
import { Table as AriaTable, TableHeader, Column, TableBody, Row, Cell } from 'react-aria-components';
import { cn } from '@/lib/utils';

export interface TableColumn {
  key: string;
  label: string;
  width?: string;
}

export interface TableProps<T> {
  columns: TableColumn[];
  data: T[];
  renderCell: (item: T, columnKey: string) => React.ReactNode;
  onRowAction?: (key: React.Key) => void;
}

export function Table<T extends { id: string }>({
  columns,
  data,
  renderCell,
  onRowAction
}: TableProps<T>) {
  return (
    <div className="relative w-full overflow-auto">
      <AriaTable
        aria-label="Data table"
        className="w-full caption-bottom text-sm"
        onRowAction={onRowAction}
      >
        <TableHeader className="border-b">
          {columns.map((column) => (
            <Column
              key={column.key}
              isRowHeader={column.key === 'name'}
              className={cn(
                'h-12 px-4 text-left align-middle font-medium',
                'text-muted-foreground [&:has([role=checkbox])]:pr-0',
                column.width
              )}
            >
              {column.label}
            </Column>
          ))}
        </TableHeader>

        <TableBody>
          {data.map((item) => (
            <Row
              key={item.id}
              className={cn(
                'border-b transition-colors',
                'hover:bg-muted/50 data-[selected]:bg-muted',
                onRowAction && 'cursor-pointer'
              )}
            >
              {columns.map((column) => (
                <Cell
                  key={column.key}
                  className="p-4 align-middle [&:has([role=checkbox])]:pr-0"
                >
                  {renderCell(item, column.key)}
                </Cell>
              ))}
            </Row>
          ))}
        </TableBody>
      </AriaTable>
    </div>
  );
}
```

**Usage:**
```typescript
import { Table } from '@/components/ui/Table';

interface User {
  id: string;
  name: string;
  email: string;
  role: string;
  status: 'active' | 'inactive';
}

export function UserTable() {
  const users: User[] = [
    { id: '1', name: 'John Doe', email: 'john@example.com', role: 'Admin', status: 'active' },
    { id: '2', name: 'Jane Smith', email: 'jane@example.com', role: 'User', status: 'active' },
    { id: '3', name: 'Bob Wilson', email: 'bob@example.com', role: 'User', status: 'inactive' },
  ];

  const columns = [
    { key: 'name', label: 'Name' },
    { key: 'email', label: 'Email' },
    { key: 'role', label: 'Role' },
    { key: 'status', label: 'Status' },
  ];

  const renderCell = (user: User, columnKey: string) => {
    switch (columnKey) {
      case 'name':
        return <span className="font-medium">{user.name}</span>;
      case 'email':
        return <span className="text-muted-foreground">{user.email}</span>;
      case 'role':
        return user.role;
      case 'status':
        return (
          <span
            className={cn(
              'inline-flex items-center rounded-full px-2 py-1 text-xs font-semibold',
              user.status === 'active'
                ? 'bg-green-100 text-green-700 dark:bg-green-900 dark:text-green-300'
                : 'bg-gray-100 text-gray-700 dark:bg-gray-800 dark:text-gray-300'
            )}
          >
            {user.status}
          </span>
        );
      default:
        return null;
    }
  };

  return (
    <Table
      columns={columns}
      data={users}
      renderCell={renderCell}
      onRowAction={(key) => console.log('Row clicked:', key)}
    />
  );
}
```

### Badge Components

**components/ui/Badge.tsx**
```typescript
import { cn } from '@/lib/utils';

export interface BadgeProps extends React.HTMLAttributes<HTMLDivElement> {
  variant?: 'default' | 'success' | 'warning' | 'danger' | 'info' | 'outline';
  size?: 'sm' | 'md' | 'lg';
}

export function Badge({
  className,
  variant = 'default',
  size = 'md',
  children,
  ...props
}: BadgeProps) {
  return (
    <div
      className={cn(
        'inline-flex items-center rounded-full font-semibold transition-colors',
        'focus:outline-none focus:ring-2 focus:ring-ring focus:ring-offset-2',

        // Variants
        variant === 'default' && 'bg-primary text-primary-foreground',
        variant === 'success' && 'bg-green-100 text-green-700 dark:bg-green-900 dark:text-green-300',
        variant === 'warning' && 'bg-yellow-100 text-yellow-700 dark:bg-yellow-900 dark:text-yellow-300',
        variant === 'danger' && 'bg-red-100 text-red-700 dark:bg-red-900 dark:text-red-300',
        variant === 'info' && 'bg-blue-100 text-blue-700 dark:bg-blue-900 dark:text-blue-300',
        variant === 'outline' && 'border border-input bg-background text-foreground',

        // Sizes
        size === 'sm' && 'px-2 py-0.5 text-xs',
        size === 'md' && 'px-2.5 py-0.5 text-sm',
        size === 'lg' && 'px-3 py-1 text-base',

        className
      )}
      {...props}
    >
      {children}
    </div>
  );
}
```

**Usage:**
```typescript
import { Badge } from '@/components/ui/Badge';

export function BadgeExamples() {
  return (
    <div className="flex flex-wrap gap-2">
      <Badge variant="default">Default</Badge>
      <Badge variant="success">Success</Badge>
      <Badge variant="warning">Warning</Badge>
      <Badge variant="danger">Danger</Badge>
      <Badge variant="info">Info</Badge>
      <Badge variant="outline">Outline</Badge>

      {/* Sizes */}
      <Badge size="sm">Small</Badge>
      <Badge size="md">Medium</Badge>
      <Badge size="lg">Large</Badge>
    </div>
  );
}
```

## Dark Mode Setup

### Next.js App Router

**app/providers.tsx**
```typescript
'use client';

import { ThemeProvider } from 'next-themes';

export function Providers({ children }: { children: React.ReactNode }) {
  return (
    <ThemeProvider
      attribute="class"
      defaultTheme="system"
      enableSystem
      disableTransitionOnChange
    >
      {children}
    </ThemeProvider>
  );
}
```

**app/layout.tsx**
```typescript
import { Providers } from './providers';
import './globals.css';

export default function RootLayout({
  children,
}: {
  children: React.ReactNode;
}) {
  return (
    <html lang="en" suppressHydrationWarning>
      <body>
        <Providers>{children}</Providers>
      </body>
    </html>
  );
}
```

**components/ThemeToggle.tsx**
```typescript
'use client';

import { useTheme } from 'next-themes';
import { Button } from '@/components/ui/Button';
import { useEffect, useState } from 'react';

export function ThemeToggle() {
  const [mounted, setMounted] = useState(false);
  const { theme, setTheme } = useTheme();

  useEffect(() => {
    setMounted(true);
  }, []);

  if (!mounted) {
    return null;
  }

  return (
    <Button
      variant="ghost"
      size="sm"
      onPress={() => setTheme(theme === 'dark' ? 'light' : 'dark')}
    >
      {theme === 'dark' ? '☀️' : '🌙'}
    </Button>
  );
}
```

## Form Patterns

### React Hook Form Integration

```typescript
import { useForm } from 'react-hook-form';
import { zodResolver } from '@hookform/resolvers/zod';
import { z } from 'zod';
import { Input } from '@/components/ui/Input';
import { Button } from '@/components/ui/Button';

const signupSchema = z.object({
  name: z.string().min(2, 'Name must be at least 2 characters'),
  email: z.string().email('Invalid email address'),
  password: z.string().min(8, 'Password must be at least 8 characters'),
});

type SignupFormData = z.infer<typeof signupSchema>;

export function SignupForm() {
  const {
    register,
    handleSubmit,
    formState: { errors, isSubmitting },
  } = useForm<SignupFormData>({
    resolver: zodResolver(signupSchema),
  });

  const onSubmit = async (data: SignupFormData) => {
    console.log('Form data:', data);
    // Handle form submission
  };

  return (
    <form onSubmit={handleSubmit(onSubmit)} className="space-y-4">
      <Input
        label="Name"
        placeholder="John Doe"
        {...register('name')}
        errorMessage={errors.name?.message}
      />

      <Input
        label="Email"
        type="email"
        placeholder="you@example.com"
        {...register('email')}
        errorMessage={errors.email?.message}
      />

      <Input
        label="Password"
        type="password"
        placeholder="••••••••"
        {...register('password')}
        errorMessage={errors.password?.message}
      />

      <Button type="submit" isDisabled={isSubmitting} className="w-full">
        {isSubmitting ? 'Creating account...' : 'Create account'}
      </Button>
    </form>
  );
}
```

## Dashboard Layout Pattern

**app/dashboard/layout.tsx**
```typescript
import { Sidebar } from '@/components/dashboard/Sidebar';
import { Header } from '@/components/dashboard/Header';

export default function DashboardLayout({
  children,
}: {
  children: React.ReactNode;
}) {
  return (
    <div className="flex h-screen">
      {/* Sidebar */}
      <Sidebar />

      {/* Main content */}
      <div className="flex flex-1 flex-col overflow-hidden">
        <Header />

        <main className="flex-1 overflow-y-auto p-6 bg-muted/10">
          {children}
        </main>
      </div>
    </div>
  );
}
```

**components/dashboard/Sidebar.tsx**
```typescript
'use client';

import Link from 'next/link';
import { usePathname } from 'next/navigation';
import { cn } from '@/lib/utils';

const navigation = [
  { name: 'Dashboard', href: '/dashboard', icon: '📊' },
  { name: 'Projects', href: '/dashboard/projects', icon: '📁' },
  { name: 'Team', href: '/dashboard/team', icon: '👥' },
  { name: 'Settings', href: '/dashboard/settings', icon: '⚙️' },
];

export function Sidebar() {
  const pathname = usePathname();

  return (
    <aside className="w-64 border-r bg-card">
      <div className="flex h-16 items-center border-b px-6">
        <h1 className="text-xl font-bold">Your App</h1>
      </div>

      <nav className="space-y-1 p-4">
        {navigation.map((item) => {
          const isActive = pathname === item.href;

          return (
            <Link
              key={item.name}
              href={item.href}
              className={cn(
                'flex items-center gap-3 rounded-md px-3 py-2',
                'text-sm font-medium transition-colors',
                isActive
                  ? 'bg-primary text-primary-foreground'
                  : 'text-muted-foreground hover:bg-muted hover:text-foreground'
              )}
            >
              <span className="text-lg">{item.icon}</span>
              {item.name}
            </Link>
          );
        })}
      </nav>
    </aside>
  );
}
```

## Best Practices

### 1. Component Organization

```
components/
├── ui/              # Primitive components (Button, Input, Modal)
├── dashboard/       # Dashboard-specific components
├── marketing/       # Marketing page components
└── shared/          # Shared across sections
```

### 2. Consistent Styling with cn()

Always use the `cn()` utility for className merging:

```typescript
import { cn } from '@/lib/utils';

// Good: Allows className overrides
<Button className={cn('w-full', className)} />

// Bad: className prop ignored
<Button className="w-full" />
```

### 3. Accessibility First

```typescript
// Always include labels
<Input label="Email" type="email" />

// Provide aria-labels for icon-only buttons
<Button aria-label="Close modal">
  <XIcon />
</Button>

// Use semantic HTML
<nav>
  <ul>
    <li><Link href="/home">Home</Link></li>
  </ul>
</nav>
```

### 4. Dark Mode Considerations

```css
/* Use CSS variables for colors */
.custom-component {
  background-color: hsl(var(--background));
  color: hsl(var(--foreground));
}

/* Not this: */
.custom-component {
  background-color: white;
  color: black;
}
```

### 5. Type Safety

```typescript
// Define strict types for component props
export interface CardProps {
  title: string;
  description?: string;
  children: React.ReactNode;
  footer?: React.ReactNode;
}

// Use proper React types
export const Card = forwardRef<HTMLDivElement, CardProps>(
  ({ title, description, children, footer }, ref) => {
    // Component implementation
  }
);
```

## Environment Variables

```bash
# .env.local

# If using with authentication
NEXT_PUBLIC_APP_URL=http://localhost:3000

# If using with database
DATABASE_URL=your_database_url

# Tailwind configuration
TAILWIND_MODE=jit
```

## TypeScript Configuration

**tsconfig.json**
```json
{
  "compilerOptions": {
    "target": "ES2020",
    "lib": ["ES2020", "DOM", "DOM.Iterable"],
    "jsx": "preserve",
    "module": "ESNext",
    "moduleResolution": "bundler",
    "resolveJsonModule": true,
    "allowJs": true,
    "strict": true,
    "noEmit": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true,
    "paths": {
      "@/*": ["./*"]
    }
  },
  "include": ["next-env.d.ts", "**/*.ts", "**/*.tsx"],
  "exclude": ["node_modules"]
}
```

## Migration from Shadcn UI

If migrating from Shadcn UI, the patterns are very similar:

**Key Differences:**
1. Untitled UI uses React Aria components for accessibility
2. More component variants (5,000+ vs Shadcn's ~50)
3. More page templates included (250+)
4. Figma sync for PRO users

**Similar Concepts:**
- `cn()` utility function
- CSS variable theming
- Tailwind CSS styling
- Copy-paste philosophy

## Resources

- **Official Website**: https://www.untitledui.com/react
- **Documentation**: https://docs.untitledui.com
- **Figma UI Kit**: Available with PRO license
- **GitHub**: Component source available with PRO license
- **Community**: Discord server for PRO users

## Summary

Untitled UI React provides:

1. **5,000+ production-ready components** with professional design
2. **Built on React Aria** for accessibility (WAI-ARIA standards)
3. **Tailwind CSS integration** with dark mode support
4. **250+ page templates** for dashboards, landing pages, pricing
5. **TypeScript support** with full type safety
6. **Copy-paste approach** - own your code, no NPM dependencies
7. **Figma synchronization** (PRO) - design and code stay in sync
8. **Framework support** - Next.js, Vite, and more

Start with the CLI for new projects, or copy individual components into existing projects. Focus on accessibility, consistent styling with `cn()`, and leveraging the design system through CSS variables.

# Component Patterns

## Table of Contents
- [Setup: cn() Utility](#setup-cn-utility)
- [Button](#button)
- [Badge](#badge)
- [GlassCard](#glasscard)
- [Tabs](#tabs)
- [Dialog (Radix)](#dialog-radix)
- [Table](#table)
- [Status Dot](#status-dot)
- [Activity Icon](#activity-icon)

All components use Tailwind CSS. Copy and adapt — these are starting points, not rigid templates.

## Setup: cn() Utility

```jsx
// lib/cn.js
import { clsx } from 'clsx';
import { twMerge } from 'tailwind-merge';
export function cn(...inputs) { return twMerge(clsx(inputs)); }
```

Install: `npm i clsx tailwind-merge`

## Button

4 variants, 3 sizes. Uses `cn()` for conditional classes.

```jsx
import { cn } from '@/lib/cn';

const variants = {
  primary: 'bg-accent-500 text-white hover:bg-accent-600 shadow-sm',
  secondary: 'bg-white/60 text-gray-700 hover:bg-white/80 border border-gray-200/60',
  ghost: 'text-gray-500 hover:text-gray-700 hover:bg-white/50',
  danger: 'bg-red-500/10 text-red-600 hover:bg-red-500/20',
};

const sizes = {
  sm: 'px-2.5 py-1 text-xs gap-1.5 rounded-lg',
  md: 'px-3.5 py-1.5 text-sm gap-2 rounded-xl',
  icon: 'p-1.5 rounded-lg',
};

export function Button({ variant = 'primary', size = 'md', className, children, ...props }) {
  return (
    <button
      className={cn(
        'inline-flex items-center justify-center font-medium transition-all duration-150',
        'disabled:opacity-50 disabled:pointer-events-none',
        variants[variant], sizes[size], className
      )}
      {...props}
    >
      {children}
    </button>
  );
}
```

Replace `accent-500`/`accent-600` with your chosen palette color.

## Badge

5 semantic variants. Small, pill-shaped.

```jsx
const variants = {
  default: 'bg-gray-200/50 text-gray-600',
  success: 'bg-green-500/15 text-green-700',
  error: 'bg-red-500/15 text-red-700',
  warning: 'bg-amber-500/15 text-amber-700',
  info: 'bg-blue-500/15 text-blue-700',
};

export function Badge({ variant = 'default', className, children, ...props }) {
  return (
    <span
      className={cn(
        'inline-flex items-center px-2 py-0.5 text-[11px] font-medium rounded-full',
        variants[variant], className
      )}
      {...props}
    >
      {children}
    </span>
  );
}
```

## GlassCard

Wrapper component applying the glass surface.

```jsx
export function GlassCard({ className, children, ...props }) {
  return (
    <div className={cn('glass rounded-2xl', className)} {...props}>
      {children}
    </div>
  );
}
```

Usage: `<GlassCard className="p-5">content</GlassCard>`

## Tabs

Tab bar with active indicator and optional close button. No library needed.

```jsx
export function TabList({ children, className }) {
  return (
    <div className={cn('flex items-center border-b border-gray-200/40 px-2 gap-0.5', className)}>
      {children}
    </div>
  );
}

export function Tab({ active, onClick, onClose, children }) {
  return (
    <button
      onClick={onClick}
      className={cn(
        'flex items-center gap-1.5 px-3 py-1.5 text-xs transition-colors relative',
        active
          ? 'text-accent-600 font-medium'
          : 'text-gray-500 hover:text-gray-700'
      )}
    >
      {children}
      {onClose && (
        <span
          onClick={(e) => { e.stopPropagation(); onClose(); }}
          className="ml-1 p-0.5 rounded hover:bg-gray-200/50 text-gray-400 hover:text-gray-600"
        >
          ×
        </span>
      )}
      {active && (
        <span className="absolute bottom-0 left-2 right-2 h-0.5 bg-accent-500 rounded-full" />
      )}
    </button>
  );
}
```

## Dialog (Radix)

Install: `npm i @radix-ui/react-dialog`

Critical: Use `animate-dialog-in` class with keyframes that include the centering transform.

```jsx
import * as RadixDialog from '@radix-ui/react-dialog';
import { X } from 'lucide-react';

export function Dialog({ open, onOpenChange, children }) {
  return <RadixDialog.Root open={open} onOpenChange={onOpenChange}>{children}</RadixDialog.Root>;
}

export function DialogContent({ children, title, description }) {
  return (
    <RadixDialog.Portal>
      <RadixDialog.Overlay className="fixed inset-0 z-50 bg-black/20 backdrop-blur-sm animate-fade-in" />
      <RadixDialog.Content className="fixed z-50 top-1/2 left-1/2 w-full max-w-lg glass-solid rounded-2xl p-6 animate-dialog-in focus:outline-none">
        {title && <RadixDialog.Title className="text-lg font-semibold text-gray-900 mb-1">{title}</RadixDialog.Title>}
        {description && <RadixDialog.Description className="text-sm text-gray-500 mb-4">{description}</RadixDialog.Description>}
        {children}
        <RadixDialog.Close className="absolute top-4 right-4 p-1 rounded-lg text-gray-400 hover:text-gray-600 hover:bg-gray-200/40 transition-colors">
          <X size={16} />
        </RadixDialog.Close>
      </RadixDialog.Content>
    </RadixDialog.Portal>
  );
}
```

## Table

Glass-wrapped table with hover rows and action buttons.

```jsx
<GlassCard className="overflow-hidden">
  <div className="flex items-center justify-between px-5 py-3 border-b border-gray-200/40">
    <h2 className="text-sm font-semibold text-gray-800">Title</h2>
    <Button size="sm"><Plus size={13} /> Add New</Button>
  </div>
  <table className="w-full">
    <thead>
      <tr className="text-left text-[11px] font-semibold text-gray-500 uppercase tracking-wider border-b border-gray-200/30">
        <th className="px-5 py-2">Name</th>
        <th className="px-5 py-2">Status</th>
        <th className="px-5 py-2 text-right">Actions</th>
      </tr>
    </thead>
    <tbody>
      <tr className="border-b border-gray-200/20 hover:bg-white/30 transition-colors cursor-pointer">
        <td className="px-5 py-3"><span className="text-sm font-medium text-gray-800">Item</span></td>
        <td className="px-5 py-3"><Badge variant="success">Active</Badge></td>
        <td className="px-5 py-3 text-right">
          <Button variant="ghost" size="icon" className="w-7 h-7"><Info size={13} /></Button>
        </td>
      </tr>
    </tbody>
  </table>
</GlassCard>
```

## Status Dot

Animated pulsing dot for live/online status.

```css
.status-dot { @apply w-2 h-2 rounded-full; }
.status-dot-running {
  @apply bg-green-500 shadow-[0_0_6px_rgba(52,199,89,0.5)];
  animation: pulse-dot 2s ease-in-out infinite;
}
.status-dot-stopped { @apply bg-gray-400; }
.status-dot-error { @apply bg-red-500 shadow-[0_0_6px_rgba(255,59,48,0.5)]; }

@keyframes pulse-dot {
  0%, 100% { opacity: 1; }
  50% { opacity: 0.5; }
}
```

Usage: `<span className="status-dot status-dot-running" />`

## Activity Icon

Sidebar icon button with active state.

```css
.activity-icon {
  @apply w-10 h-10 flex items-center justify-center rounded-xl
         text-gray-500 hover:text-gray-700 hover:bg-white/50
         transition-all duration-150 cursor-pointer;
}
.activity-icon-active {
  @apply text-accent-500 bg-white/60 shadow-sm;
}
```

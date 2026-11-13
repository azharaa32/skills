# Animation & Motion Patterns

## Table of Contents
- [Core Principles](#core-principles)
- [Loading States](#loading-states)
- [Micro-interactions](#micro-interactions)
- [Page Transitions](#page-transitions)
- [Modal / Dialog Animations](#modal--dialog-animations)
- [List Animations](#list-animations)

## Core Principles

1. **Purpose over decoration** — Every animation should communicate state change, not just look cool
2. **Fast UI, slow delight** — UI transitions: 120-200ms. Decorative reveals: 300-500ms
3. **Ease-out for entrances, ease-in for exits** — Things arrive gently, leave quickly
4. **Never fight positioning** — If an element uses `transform` for layout (centering, positioning), the animation MUST include those transforms in every keyframe

## Loading States

### Spinner (inline actions)

Use Lucide's `Loader2` with `animate-spin`. Replace the action icon, don't add alongside it.

```jsx
import { Loader2, Play } from 'lucide-react';

<Button disabled={loading} onClick={handleStart}>
  {loading ? <Loader2 size={14} className="animate-spin" /> : <Play size={14} />}
  {loading ? 'Starting...' : 'Start'}
</Button>
```

### Status dot → spinner swap

When an item is loading, replace its status dot with a spinner:

```jsx
{isLoading ? (
  <Loader2 size={10} className="animate-spin text-accent-500 flex-shrink-0" />
) : (
  <span className={cn('status-dot', running ? 'status-dot-running' : 'status-dot-stopped')} />
)}
```

### Skeleton loading

Match the exact shape of the content that will replace it:

```jsx
function Skeleton({ className }) {
  return <div className={cn('animate-pulse rounded-lg bg-gray-200/60', className)} />;
}

// Usage — matches a stat card
<GlassCard className="px-4 py-3">
  <Skeleton className="h-3 w-16 mb-2" />
  <Skeleton className="h-6 w-12 mb-1" />
  <Skeleton className="h-2.5 w-24" />
</GlassCard>
```

### Progress bar (determinate)

```jsx
function ProgressBar({ percent, className }) {
  return (
    <div className={cn('h-1.5 bg-gray-200/60 rounded-full overflow-hidden', className)}>
      <div
        className="h-full bg-accent-500 rounded-full transition-all duration-500 ease-out"
        style={{ width: `${Math.min(100, Math.max(0, percent))}%` }}
      />
    </div>
  );
}
```

### Indeterminate progress

```css
@keyframes indeterminate {
  0% { transform: translateX(-100%); }
  100% { transform: translateX(400%); }
}

.progress-indeterminate {
  @apply h-1 bg-gray-200/60 rounded-full overflow-hidden relative;
}
.progress-indeterminate::after {
  content: '';
  @apply absolute inset-y-0 left-0 w-1/4 bg-accent-500 rounded-full;
  animation: indeterminate 1.5s ease-in-out infinite;
}
```

## Micro-interactions

### Hover lift (cards)

```css
.card-hover {
  @apply transition-all duration-150;
}
.card-hover:hover {
  @apply -translate-y-0.5 shadow-lg;
}
```

### Hover reveal (action buttons)

Show action buttons only on parent hover:

```jsx
<div className="group relative">
  <span>Item name</span>
  <div className="hidden group-hover:flex items-center gap-0.5">
    <Button variant="ghost" size="icon">...</Button>
  </div>
</div>
```

### Hover opacity (secondary info)

```jsx
<ChevronRight
  size={12}
  className="text-gray-300 opacity-0 group-hover:opacity-100 transition-opacity"
/>
```

### Focus ring

```css
.focus-ring {
  @apply focus:outline-none focus:ring-2 focus:ring-accent-500/30 focus:border-accent-500/40;
}
```

### Active press

```css
.press {
  @apply active:scale-[0.98] transition-transform;
}
```

## Page Transitions

### Fade in on mount

```jsx
function FadeIn({ children, delay = 0 }) {
  return (
    <div
      className="animate-fade-in"
      style={{ animationDelay: `${delay}ms`, animationFillMode: 'both' }}
    >
      {children}
    </div>
  );
}
```

### Staggered list reveal

```jsx
{items.map((item, i) => (
  <div
    key={item.id}
    className="animate-fade-in"
    style={{ animationDelay: `${i * 50}ms`, animationFillMode: 'both' }}
  >
    <ItemCard {...item} />
  </div>
))}
```

## Modal / Dialog Animations

### CRITICAL: Transform Conflict

Modals centered with `top-1/2 left-1/2 -translate-x-1/2 -translate-y-1/2` CANNOT use standard
`translateY` animations because the animation overrides the centering transform on frame 0.

**Wrong** (causes layout shift):
```css
@keyframes slideUp {
  0% { opacity: 0; transform: translateY(10px); }   /* centering LOST */
  100% { opacity: 1; transform: translateY(0); }     /* centering LOST */
}
```

**Correct** (includes centering in keyframes):
```css
@keyframes dialogScale {
  0% { opacity: 0; transform: translate(-50%, -50%) scale(0.96); }
  100% { opacity: 1; transform: translate(-50%, -50%) scale(1); }
}
```

### Backdrop

```css
.backdrop {
  @apply fixed inset-0 bg-black/20 backdrop-blur-sm;
  animation: fadeIn 0.15s ease-out;
}
```

### Command Palette

Centered horizontally with `left-1/2`, not vertically. Use `translateX` in keyframes:

```css
@keyframes paletteScale {
  0% { opacity: 0; transform: translateX(-50%) scale(0.97); }
  100% { opacity: 1; transform: translateX(-50%) scale(1); }
}
```

## List Animations

### Collapse/expand (CSS only)

```css
.collapsible {
  display: grid;
  grid-template-rows: 0fr;
  transition: grid-template-rows 200ms ease-out;
}
.collapsible.open {
  grid-template-rows: 1fr;
}
.collapsible > div {
  overflow: hidden;
}
```

### Expand chevron rotation

```jsx
<ChevronRight
  size={12}
  className={cn('transition-transform duration-150', expanded && 'rotate-90')}
/>
```

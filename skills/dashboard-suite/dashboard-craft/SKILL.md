---
name: dashboard-craft
description: >
  Build production-grade dashboard and admin UIs with premium visual quality using React + Tailwind.
  Generates polished, IDE-like interfaces with glass morphism, proper component architecture,
  shadcn-inspired primitives, and thoughtful micro-interactions. Use when the user asks to build
  a dashboard, admin panel, management UI, monitoring interface, developer tool UI, internal tool,
  or any data-heavy application interface. Also triggers when asked to style/improve an existing
  dashboard, add glass effects, make a UI look "premium" or "Apple-like", or when building
  anything that involves sidebars, panels, tables, status indicators, or terminal-style UIs.
---

Build premium dashboard UIs with React + Tailwind CSS. Output production-ready code that looks
designed, not generated.

## Design Decision Framework

Before writing code, make three decisions:

1. **Density**: Dense (IDE, monitoring) or Spacious (admin, overview)?
2. **Surface**: Glass (translucent layers) or Solid (opaque cards)?
3. **Palette**: Pick ONE dominant accent color + warm/cool gray scale

Then commit fully. Mixing density levels or surface types creates visual noise.

## Core Stack

```
React 18 (JSX) + Tailwind CSS v3 + Lucide icons + react-resizable-panels
State: Zustand | Data: SWR | Dialogs: Radix UI | Terminal: @xterm/xterm
```

For component patterns, typography pairings, and detailed CSS recipes see the reference files:

- **[references/glass-system.md](references/glass-system.md)** — Glass morphism CSS system (4 tiers), color palettes, Tailwind config
- **[references/components.md](references/components.md)** — shadcn-style primitives (Button, Badge, Card, Dialog, Tabs, Table) with exact code
- **[references/layouts.md](references/layouts.md)** — Dashboard layout patterns (IDE, admin, monitoring), resizable panels, responsive
- **[references/motion.md](references/motion.md)** — Animation patterns, loading states, micro-interactions, transition recipes
- **[references/extras.md](references/extras.md)** — reactbits patterns, terminal styling, status indicators, file explorers, cool React tricks

## Quick Reference (Always Apply)

### Typography Pairing

Pick ONE pairing per project. Do not mix.

| Style | Display / UI | Code / Mono | Vibe |
|-------|-------------|-------------|------|
| Premium | Inter | JetBrains Mono | Apple, clean |
| Editorial | Instrument Serif | IBM Plex Mono | Magazine, refined |
| Technical | Geist Sans | Geist Mono | Vercel, developer |
| Warm | DM Sans | Fira Code | Friendly, approachable |
| Sharp | Satoshi | Berkeley Mono | Bold, modern |

Load via Google Fonts `<link>` or `@fontsource` packages. Never rely on system fonts as the design choice.

### Color Rules

- One accent color dominates. Everything else is gray scale.
- Status colors are sacred: green=good, red=error, amber=warning, blue=info. Never redefine these.
- Glass panels: white with opacity (light theme) or dark with opacity (dark theme).
- Text: max 3 shades of gray (heading, body, muted). Not 7.

### Spacing Rules

- Consistent scale: 4px base (`p-1` = 4px, `p-2` = 8px, `p-3` = 12px, `p-4` = 16px)
- Side panels: `px-3 py-2` headers, `px-3 py-1` list items
- Cards: `p-4` to `p-6`
- Table cells: `px-5 py-3`
- Icon sizes: 11px (status bar), 13-15px (UI), 20px (activity bar)

### Animation Rules

- **Never animate transforms that conflict with positioning**. If element uses `translate(-50%, -50%)` for centering, the animation keyframes MUST include those transforms.
- Prefer `scale + opacity` over `translateY` for modals/dialogs.
- Duration: 120-200ms for UI transitions, 300ms for page transitions.
- Easing: `ease-out` for entrances, `ease-in` for exits.

### Anti-Patterns (Never Do)

- Purple gradients on white backgrounds
- `box-shadow: 0 0 40px` glow effects everywhere
- Rounded corners > `rounded-2xl` (except full circles)
- More than 2 font families
- Gradient text on non-headings
- Skeleton loaders that don't match the final layout shape
- Toast notifications stacking in corners

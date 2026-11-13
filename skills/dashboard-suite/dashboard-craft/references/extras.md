# Extras: Advanced Patterns & Cool Tricks

## Table of Contents
- [Terminal Styling](#terminal-styling)
- [File Explorer Pattern](#file-explorer-pattern)
- [Command Palette Pattern](#command-palette-pattern)
- [Data Fetching with SWR](#data-fetching-with-swr)
- [State Management with Zustand](#state-management-with-zustand)
- [Keyboard Shortcuts](#keyboard-shortcuts)
- [Reactbits-Style Patterns](#reactbits-style-patterns)
- [shadcn/ui Block Patterns](#shadcnui-block-patterns)

## Terminal Styling

Dark terminal inside a light glass frame creates premium contrast.

### xterm Theme (Tokyo Night)

```js
const THEME = {
  background: '#1a1b26', foreground: '#c0caf5',
  cursor: '#c0caf5', cursorAccent: '#1a1b26',
  selectionBackground: 'rgba(0, 122, 255, 0.3)',
  black: '#15161e', red: '#f7768e', green: '#9ece6a', yellow: '#e0af68',
  blue: '#7aa2f7', magenta: '#bb9af7', cyan: '#7dcfff', white: '#a9b1d6',
  brightBlack: '#414868', brightRed: '#f7768e', brightGreen: '#9ece6a',
  brightYellow: '#e0af68', brightBlue: '#7aa2f7', brightMagenta: '#bb9af7',
  brightCyan: '#7dcfff', brightWhite: '#c0caf5',
};
```

### Terminal Container

```jsx
<div className="glass-dark rounded-xl overflow-hidden border border-white/20 shadow-inner">
  <div ref={containerRef} className="h-full w-full" />
</div>
```

### Terminal Persistence

Terminals should survive navigation. Use a global registry pattern:

1. Store terminal instances in a `Map` outside React
2. On mount: check registry, reattach existing or create new
3. On unmount: "park" terminal in hidden off-screen div (don't destroy)
4. On explicit close: dispose terminal and remove from registry

```js
const terminalInstances = new Map();
let parkingLot = null;

function getParkingLot() {
  if (!parkingLot) {
    parkingLot = document.createElement('div');
    parkingLot.style.cssText = 'position:fixed;left:-9999px;top:-9999px;width:1px;height:1px;overflow:hidden;';
    document.body.appendChild(parkingLot);
  }
  return parkingLot;
}

// On mount: containerRef.current.appendChild(instance.wrapper)
// On unmount: getParkingLot().appendChild(instance.wrapper)
// On close: instance.term.dispose(); instance.ws.close(); map.delete(key)
```

## File Explorer Pattern

Lazy-loading directory browser with file type icons.

```jsx
const EXT_ICONS = {
  js: FileCode, jsx: FileCode, ts: FileCode, tsx: FileCode, py: FileCode,
  json: FileJson, md: FileText, txt: FileText, yml: FileText, yaml: FileText,
};

function getFileIcon(entry) {
  if (entry.type === 'directory') return Folder;
  return EXT_ICONS[entry.name.split('.').pop()?.toLowerCase()] || File;
}
```

Key UX patterns:
- Search input at top to filter files by name
- Directories sorted before files
- File size shown on hover (right side)
- Chevron icon on directories (opacity-0 until hover)
- Click directory → navigate. Click file → open in preview.
- Back button when not at root

## Command Palette Pattern

`Cmd+K` searchable command list. Build as a full-screen overlay, not a dropdown.

```jsx
// Position: fixed, top-[20%], left-1/2, max-w-lg
// Animation: paletteScale (includes translateX(-50%))
// Glass surface: glass-solid for readability

<div className="fixed z-50 top-[20%] left-1/2 w-full max-w-lg animate-palette-in">
  <div className="glass-solid rounded-2xl overflow-hidden shadow-2xl">
    <div className="flex items-center gap-3 px-4 py-3 border-b border-gray-200/40">
      <Search size={16} className="text-gray-400" />
      <input placeholder="Type a command..." className="flex-1 bg-transparent text-sm focus:outline-none" />
      <kbd className="text-[10px] text-gray-400 bg-gray-200/40 px-1.5 py-0.5 rounded">ESC</kbd>
    </div>
    <div className="max-h-72 overflow-y-auto py-1">
      {filtered.map((cmd, i) => (
        <button key={cmd.id} className={cn('w-full flex items-center gap-3 px-4 py-2', i === selected && 'bg-accent-500/10')}>
          <cmd.icon size={15} />
          <span className="flex-1 text-sm">{cmd.label}</span>
          {cmd.shortcut && <kbd className="text-[10px]">{cmd.shortcut}</kbd>}
        </button>
      ))}
    </div>
  </div>
</div>
```

Arrow keys navigate, Enter executes, Escape closes. Build the command list dynamically based on context.

## Data Fetching with SWR

Install: `npm i swr`

```js
import useSWR from 'swr';

const fetcher = (url) => fetch(url).then(r => r.ok ? r.json() : Promise.reject(new Error(`HTTP ${r.status}`)));

export function useApi(path, options = {}) {
  const { refreshInterval = 0, ...rest } = options;
  return useSWR(path ? `/api${path}` : null, fetcher, {
    refreshInterval,
    revalidateOnFocus: false,
    ...rest,
  });
}

// Convention: wrap each endpoint in a named hook
export function useClients() { return useApi('/clients', { refreshInterval: 8000 }); }
export function useSystem() { return useApi('/system', { refreshInterval: 8000 }); }
export function useFiles(name, path) {
  return useApi(name ? `/clients/${name}/files?path=${encodeURIComponent(path)}` : null, {
    refreshInterval: 3000, revalidateOnFocus: true,
  });
}
```

## State Management with Zustand

Install: `npm i zustand`

```js
import { create } from 'zustand';

export const useStore = create((set, get) => ({
  activeClient: null,
  setActiveClient: (name) => set({ activeClient: name }),

  sidePanel: 'clients', // 'clients' | 'explorer' | 'git' | 'activity'
  setSidePanel: (panel) => set({ sidePanel: panel }),

  viewMode: 'overview', // 'overview' | 'detail' | 'terminal' | 'file'
  setViewMode: (mode) => set({ viewMode: mode }),

  // Terminal management
  terminals: {}, // { clientName: [{ id, label }] }
  addTerminal: (clientName) => { /* ... */ },
  removeTerminal: (clientName, termId) => { /* ... */ },

  // Command palette
  commandPaletteOpen: false,
  toggleCommandPalette: () => set({ commandPaletteOpen: !get().commandPaletteOpen }),
}));
```

## Keyboard Shortcuts

```js
import { useEffect } from 'react';
import { useStore } from './useStore';

export function useKeyboard() {
  const store = useStore();

  useEffect(() => {
    function handler(e) {
      const meta = e.metaKey || e.ctrlKey;
      if (meta && e.key === 'k') { e.preventDefault(); store.toggleCommandPalette(); }
      if (meta && e.key === 'n') { e.preventDefault(); store.setNewClientDialogOpen(true); }
      if (e.ctrlKey && ['1','2','3','4'].includes(e.key)) {
        e.preventDefault();
        store.setSidePanel(['clients','explorer','git','activity'][e.key - 1]);
      }
    }
    window.addEventListener('keydown', handler);
    return () => window.removeEventListener('keydown', handler);
  }, [store]);
}
```

## Reactbits-Style Patterns

### Magnetic hover effect

Button that slightly follows the cursor on hover:

```jsx
function MagneticButton({ children, strength = 0.3, ...props }) {
  const ref = useRef(null);

  function handleMove(e) {
    const rect = ref.current.getBoundingClientRect();
    const x = (e.clientX - rect.left - rect.width / 2) * strength;
    const y = (e.clientY - rect.top - rect.height / 2) * strength;
    ref.current.style.transform = `translate(${x}px, ${y}px)`;
  }

  function handleLeave() {
    ref.current.style.transform = 'translate(0, 0)';
  }

  return (
    <button ref={ref} onMouseMove={handleMove} onMouseLeave={handleLeave}
      className="transition-transform duration-200" {...props}>
      {children}
    </button>
  );
}
```

### Count-up number animation

```jsx
function AnimatedNumber({ value, duration = 1000 }) {
  const [display, setDisplay] = useState(0);
  useEffect(() => {
    const start = performance.now();
    const from = display;
    function tick(now) {
      const t = Math.min((now - start) / duration, 1);
      setDisplay(Math.round(from + (value - from) * t));
      if (t < 1) requestAnimationFrame(tick);
    }
    requestAnimationFrame(tick);
  }, [value]);
  return <span>{display}</span>;
}
```

### Gradient border card

```jsx
<div className="p-[1px] rounded-2xl bg-gradient-to-r from-accent-500/50 to-purple-500/50">
  <div className="bg-white/90 rounded-2xl p-5">
    Content with gradient border
  </div>
</div>
```

### Dot grid background

```css
.dot-grid {
  background-image: radial-gradient(circle, rgba(0,0,0,0.08) 1px, transparent 1px);
  background-size: 24px 24px;
}
```

### Noise texture overlay

```css
.noise::after {
  content: '';
  position: absolute;
  inset: 0;
  background: url("data:image/svg+xml,%3Csvg viewBox='0 0 256 256' xmlns='http://www.w3.org/2000/svg'%3E%3Cfilter id='n'%3E%3CfeTurbulence type='fractalNoise' baseFrequency='0.9' numOctaves='4' stitchTiles='stitch'/%3E%3C/filter%3E%3Crect width='100%25' height='100%25' filter='url(%23n)' opacity='0.04'/%3E%3C/svg%3E");
  pointer-events: none;
  border-radius: inherit;
}
```

## shadcn/ui Block Patterns

When using shadcn/ui or building shadcn-compatible components:

### Form block

```jsx
<div className="space-y-4">
  <div>
    <label className="block text-xs font-medium text-gray-600 mb-1.5">Label</label>
    <input className="w-full px-3 py-2 text-sm rounded-xl bg-white/60 border border-gray-200/60 text-gray-800 placeholder:text-gray-400 focus:outline-none focus:ring-2 focus:ring-accent-500/30 focus:border-accent-500/40 transition-all" />
  </div>
  <div className="flex justify-end gap-2 pt-2">
    <Button variant="secondary">Cancel</Button>
    <Button>Submit</Button>
  </div>
</div>
```

### Auth badge strip

Row of small pill badges showing tool/service auth status:

```jsx
const TOOLS = { github: 'GH', git: 'Git', ssh: 'SSH', vercel: 'VCL' };

function AuthStrip({ tools }) {
  return (
    <div className="flex items-center gap-1">
      {Object.entries(TOOLS).map(([key, label]) => (
        <Badge key={key} variant={tools[key] ? 'success' : 'error'} className="text-[9px] px-1.5">
          {label}{tools[key] ? '✓' : '✗'}
        </Badge>
      ))}
    </div>
  );
}
```

### Metric card with progress bar

```jsx
function MetricCard({ label, value, max, unit, icon: Icon }) {
  const pct = max > 0 ? (value / max) * 100 : 0;
  const color = pct > 80 ? 'bg-red-500' : pct > 60 ? 'bg-amber-500' : 'bg-accent-500';
  return (
    <GlassCard className="p-4">
      <div className="flex items-center justify-between mb-3">
        <div className="flex items-center gap-2">
          <Icon size={14} className="text-accent-500" />
          <span className="text-xs font-medium text-gray-500 uppercase tracking-wider">{label}</span>
        </div>
        <span className="text-sm font-bold text-gray-800">{Math.round(pct)}%</span>
      </div>
      <div className="h-1.5 bg-gray-200/60 rounded-full overflow-hidden">
        <div className={cn('h-full rounded-full transition-all duration-500', color)} style={{ width: `${pct}%` }} />
      </div>
      <div className="text-[10px] text-gray-400 mt-1.5">{value} / {max} {unit}</div>
    </GlassCard>
  );
}
```

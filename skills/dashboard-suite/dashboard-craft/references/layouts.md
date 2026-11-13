# Dashboard Layout Patterns

## Table of Contents
- [IDE Layout (Activity Bar + Side Panel + Editor)](#ide-layout)
- [Admin Layout (Sidebar + Content)](#admin-layout)
- [Monitoring Layout (Grid Cards)](#monitoring-layout)
- [Resizable Panels](#resizable-panels)
- [Responsive Considerations](#responsive-considerations)

## IDE Layout

The most versatile dashboard layout. Used by VS Code, Cursor, Warp.

```
┌──────┬───────────┬──────────────────────────────────────┐
│ Act. │ Side Panel│ Main Content Area                    │
│ Bar  │           │                                      │
│      │ (varies)  │ ┌─ Header Bar ────────────────────┐ │
│ 🏠  │           │ │ Title + Actions                  │ │
│ 📁  │ Clients / │ └──────────────────────────────────┘ │
│ 🔀  │ Explorer /│                                      │
│ 🕐  │ Git /     │ ┌─ Content ────────────────────────┐ │
│      │ Activity  │ │ (table / cards / editor / term)  │ │
│      │           │ │                                  │ │
│ +   │           │ │                                  │ │
│ ⚙️  │           │ │                                  │ │
├──────┴──────────┴────────────────────────────────────────┤
│ Status Bar                                     ⌘K       │
└──────────────────────────────────────────────────────────┘
```

### Implementation

```jsx
import { Panel, PanelGroup, PanelResizeHandle } from 'react-resizable-panels';

export function Layout() {
  return (
    <div className="h-screen flex flex-col overflow-hidden">
      <div className="flex-1 flex min-h-0">
        {/* Activity Bar — fixed 48px */}
        <ActivityBar />

        {/* Resizable panels */}
        <PanelGroup direction="horizontal" className="flex-1">
          {/* Side panel */}
          <Panel defaultSize={18} minSize={12} maxSize={30}>
            <div className="h-full bg-white/40 backdrop-blur-lg border-r border-white/30 overflow-hidden">
              <SidePanelContent />
            </div>
          </Panel>

          <PanelResizeHandle className="w-[1px] bg-gray-200/50 hover:bg-accent-500/40 transition-colors" />

          {/* Main content */}
          <Panel defaultSize={82}>
            <MainContent />
          </Panel>
        </PanelGroup>
      </div>

      <StatusBar />
    </div>
  );
}
```

### Activity Bar

Narrow icon strip on the far left. 48px wide (`w-12`).

```jsx
function ActivityBar() {
  return (
    <div className="w-12 flex flex-col items-center py-3 gap-1 bg-white/30 backdrop-blur-lg border-r border-white/20">
      <div className="flex flex-col gap-1">
        {items.map(({ id, icon: Icon, label }) => (
          <button key={id} className={cn('activity-icon', active === id && 'activity-icon-active')} title={label}>
            <Icon size={20} strokeWidth={1.8} />
          </button>
        ))}
      </div>
      <div className="flex-1" />
      <div className="flex flex-col gap-1">
        {/* Bottom actions: New, Settings */}
      </div>
    </div>
  );
}
```

### Status Bar

Slim bar at the bottom. 28px high (`h-7`).

```jsx
function StatusBar() {
  return (
    <div className="h-7 flex items-center justify-between px-3 bg-white/50 backdrop-blur-lg border-t border-white/30 text-[11px] text-gray-500 select-none">
      <div className="flex items-center gap-3">
        {/* Left: connection status, system stats */}
      </div>
      <div className="flex items-center gap-3">
        {/* Right: active item, keyboard shortcut hint */}
      </div>
    </div>
  );
}
```

## Admin Layout

Wider sidebar with navigation links. Content area is scrollable.

```
┌────────────────┬─────────────────────────────────────────┐
│                │  Header                                  │
│  Logo          ├─────────────────────────────────────────┤
│                │                                          │
│  Navigation    │  Page Content (scrollable)               │
│  ├─ Dashboard  │                                          │
│  ├─ Users      │  ┌─────────┐  ┌─────────┐  ┌─────────┐│
│  ├─ Settings   │  │ Card 1  │  │ Card 2  │  │ Card 3  ││
│  └─ ...        │  └─────────┘  └─────────┘  └─────────┘│
│                │                                          │
│                │  ┌──────────────────────────────────────┐│
│  User profile  │  │ Table / Chart / Content              ││
│                │  └──────────────────────────────────────┘│
└────────────────┴─────────────────────────────────────────┘
```

Sidebar width: 240-280px (`w-64` or `w-72`). Content: `overflow-y-auto p-6`.

## Monitoring Layout

Full-width grid of stat cards + a large table or chart below.

```jsx
<div className="h-full overflow-y-auto p-6">
  <h1 className="text-2xl font-bold text-gray-900 tracking-tight mb-6">Dashboard</h1>

  {/* Stat cards — responsive grid */}
  <div className="grid grid-cols-2 md:grid-cols-4 lg:grid-cols-5 gap-4 mb-6">
    <StatCard icon={Cpu} label="CPU" value="12%" sub="2 cores" />
    <StatCard icon={HardDrive} label="Memory" value="45%" sub="1.8 / 4 GB" />
    {/* ... */}
  </div>

  {/* Main table */}
  <GlassCard>...</GlassCard>
</div>
```

### StatCard Pattern

```jsx
function StatCard({ icon: Icon, label, value, sub }) {
  return (
    <GlassCard className="px-4 py-3">
      <div className="flex items-center gap-2 mb-2">
        <Icon size={15} className="text-accent-500" />
        <span className="text-[11px] font-semibold text-gray-500 uppercase tracking-wider">{label}</span>
      </div>
      <div className="text-lg font-bold text-gray-800 tracking-tight">{value}</div>
      {sub && <div className="text-[11px] text-gray-400 mt-0.5 truncate">{sub}</div>}
    </GlassCard>
  );
}
```

## Resizable Panels

Install: `npm i react-resizable-panels`

### Horizontal Split (side panel + main)

```jsx
<PanelGroup direction="horizontal">
  <Panel defaultSize={20} minSize={12} maxSize={35}>
    <Sidebar />
  </Panel>
  <PanelResizeHandle className="w-[1px] bg-gray-200/50 hover:bg-accent-500/40 transition-colors" />
  <Panel defaultSize={80}>
    <Main />
  </Panel>
</PanelGroup>
```

### Vertical Split (editor + terminal)

```jsx
<PanelGroup direction="vertical">
  <Panel defaultSize={60} minSize={20}>
    <Editor />
  </Panel>
  <PanelResizeHandle className="h-[1px] bg-gray-200/50 hover:bg-accent-500/40 transition-colors" />
  <Panel defaultSize={40} minSize={15}>
    <Terminal />
  </Panel>
</PanelGroup>
```

### Multi-pane (tmux-like terminals)

```jsx
<PanelGroup direction={splitDirection}>
  {panes.map((pane, i) => (
    <Fragment key={pane.id}>
      {i > 0 && <PanelResizeHandle className="..." />}
      <Panel minSize={15}>
        <TerminalPane id={pane.id} />
      </Panel>
    </Fragment>
  ))}
</PanelGroup>
```

## Responsive Considerations

Dashboards are primarily desktop. Don't over-optimize for mobile. Key breakpoints:

- `< 768px`: Collapse activity bar to bottom nav, hide side panel behind toggle
- `768-1024px`: Narrower side panel (min-size), smaller stat cards grid
- `> 1024px`: Full layout as designed

For glass morphism on mobile: reduce `backdrop-blur` to `backdrop-blur-md` (less GPU intensive).

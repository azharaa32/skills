# Glass Morphism Design System

## Table of Contents
- [4-Tier Glass System](#4-tier-glass-system)
- [Color Palettes](#color-palettes)
- [Tailwind Config Template](#tailwind-config-template)
- [Base CSS Template](#base-css-template)
- [Dark Theme Variant](#dark-theme-variant)

## 4-Tier Glass System

Every glass UI needs exactly 4 surface levels. Not 3, not 6. Four.

### Light Theme Surfaces

```css
/* Level 1: Primary panels (sidebars, main cards) */
.glass {
  @apply bg-white/70 backdrop-blur-xl border border-white/40 shadow-lg;
}

/* Level 2: Secondary surfaces (nested cards, hover states) */
.glass-subtle {
  @apply bg-white/50 backdrop-blur-lg border border-white/30 shadow-md;
}

/* Level 3: Prominent surfaces (dialogs, popovers, command palettes) */
.glass-solid {
  @apply bg-white/90 backdrop-blur-xl border border-gray-200/60 shadow-lg;
}

/* Level 4: Inverted surfaces (terminal, code blocks, dark insets) */
.glass-dark {
  @apply bg-gray-900/95 backdrop-blur-xl border border-white/10;
}
```

### When to Use Each

| Surface | Use For | Example |
|---------|---------|---------|
| `.glass` | Main panels, cards | Sidebar, stat cards, tables |
| `.glass-subtle` | Nested content, hover states | List items on hover, inner sections |
| `.glass-solid` | Overlays that need readability | Dialogs, command palette, tooltips |
| `.glass-dark` | Inverted contrast areas | Terminals, code editors, log viewers |

### Page Background

The background gradient is critical — glass looks flat without it.

```css
body {
  /* Light theme: warm gray gradient */
  background: linear-gradient(135deg, #f8f8fa 0%, #f0f0f5 50%, #e8e8f0 100%);

  /* Alternative: cool blue tint */
  background: linear-gradient(135deg, #f0f4ff 0%, #e8edf5 50%, #dfe4ef 100%);

  /* Alternative: warm cream */
  background: linear-gradient(135deg, #faf8f5 0%, #f5f0ea 50%, #ede8e0 100%);
}
```

## Color Palettes

### Apple SF Palette (Premium, clean)

```js
colors: {
  apple: {
    blue: '#007AFF',
    green: '#34C759',
    red: '#FF3B30',
    amber: '#FF9500',
    purple: '#AF52DE',
    gray: {
      50: '#f8f8fa', 100: '#f0f0f5', 200: '#e5e5ea', 300: '#d1d1d6',
      400: '#aeaeb2', 500: '#8e8e93', 600: '#636366', 700: '#48484a',
      800: '#3a3a3c', 900: '#2c2c2e',
    },
  },
}
```

### Vercel Palette (Technical, sharp)

```js
colors: {
  geist: {
    accent: '#0070F3',
    success: '#0070F3',
    error: '#EE0000',
    warning: '#F5A623',
    gray: {
      50: '#fafafa', 100: '#eaeaea', 200: '#999', 300: '#888',
      400: '#666', 500: '#444', 600: '#333', 700: '#222',
      800: '#111', 900: '#000',
    },
  },
}
```

### Linear Palette (Sleek, dark-first)

```js
colors: {
  linear: {
    accent: '#5E6AD2',
    success: '#3FB950',
    error: '#F85149',
    warning: '#D29922',
    gray: {
      50: '#f5f5f5', 100: '#e5e5e5', 200: '#d4d4d4', 300: '#a3a3a3',
      400: '#737373', 500: '#525252', 600: '#404040', 700: '#262626',
      800: '#171717', 900: '#0a0a0a',
    },
  },
}
```

## Tailwind Config Template

```js
/** @type {import('tailwindcss').Config} */
export default {
  content: ['./index.html', './src/**/*.{js,jsx}'],
  theme: {
    extend: {
      colors: {
        // Paste chosen palette here
      },
      fontFamily: {
        sans: ['Inter', 'system-ui', '-apple-system', 'sans-serif'],
        mono: ['JetBrains Mono', 'Menlo', 'monospace'],
      },
      backdropBlur: {
        '2xl': '40px',
        '3xl': '64px',
      },
      animation: {
        'pulse-slow': 'pulse 3s cubic-bezier(0.4, 0, 0.6, 1) infinite',
      },
    },
  },
  plugins: [],
};
```

## Base CSS Template

```css
@tailwind base;
@tailwind components;
@tailwind utilities;

@layer base {
  * { box-sizing: border-box; }
  body {
    background: linear-gradient(135deg, #f8f8fa 0%, #f0f0f5 50%, #e8e8f0 100%);
    min-height: 100vh;
    overflow: hidden;
  }
  ::selection { background: rgba(0, 122, 255, 0.2); }

  /* Scrollbar */
  ::-webkit-scrollbar { width: 6px; height: 6px; }
  ::-webkit-scrollbar-track { background: transparent; }
  ::-webkit-scrollbar-thumb { background: rgba(0,0,0,0.15); border-radius: 3px; }
  ::-webkit-scrollbar-thumb:hover { background: rgba(0,0,0,0.25); }
}

@layer components {
  .glass { @apply bg-white/70 backdrop-blur-xl border border-white/40 shadow-lg; }
  .glass-subtle { @apply bg-white/50 backdrop-blur-lg border border-white/30 shadow-md; }
  .glass-solid { @apply bg-white/90 backdrop-blur-xl border border-gray-200/60 shadow-lg; }
  .glass-dark { @apply bg-gray-900/95 backdrop-blur-xl border border-white/10; }
}

/* Animations that preserve centering transforms */
@keyframes fadeIn {
  0% { opacity: 0; }
  100% { opacity: 1; }
}
@keyframes dialogScale {
  0% { opacity: 0; transform: translate(-50%, -50%) scale(0.96); }
  100% { opacity: 1; transform: translate(-50%, -50%) scale(1); }
}
@keyframes paletteScale {
  0% { opacity: 0; transform: translateX(-50%) scale(0.97); }
  100% { opacity: 1; transform: translateX(-50%) scale(1); }
}

.animate-fade-in { animation: fadeIn 0.15s ease-out; }
.animate-dialog-in { animation: dialogScale 0.15s ease-out; }
.animate-palette-in { animation: paletteScale 0.12s ease-out; }
```

## Dark Theme Variant

For dark-first dashboards, invert the glass surfaces:

```css
.glass { @apply bg-white/5 backdrop-blur-xl border border-white/10 shadow-lg; }
.glass-subtle { @apply bg-white/3 backdrop-blur-lg border border-white/5 shadow-md; }
.glass-solid { @apply bg-gray-900/90 backdrop-blur-xl border border-gray-700/60 shadow-lg; }
.glass-dark { @apply bg-black/80 backdrop-blur-xl border border-white/5; }
```

Dark body background:
```css
body {
  background: linear-gradient(135deg, #0a0a0f 0%, #12121a 50%, #0d0d15 100%);
}
```

Dark scrollbar:
```css
::-webkit-scrollbar-thumb { background: rgba(255,255,255,0.15); }
::-webkit-scrollbar-thumb:hover { background: rgba(255,255,255,0.25); }
```

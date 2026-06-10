---
description: Redesign a Vue 3 application's UI into a modern SaaS-style interface with a vertical navigation sidebar, consistent spacing, and a polished professional look.
---

Redesign this Vue 3 app from a horizontal top-nav layout into a modern SaaS-style interface with a vertical sidebar. **You MUST delegate all `.vue` file creation and edits to the vue-expert agent.**

## Step 1 — Audit the current layout

Before writing any code, read and understand:
- `client/src/App.vue` — current top-nav structure, global CSS, which components are imported
- `client/src/main.js` — router routes and nav labels
- `client/src/components/FilterBar.vue` — filter bar structure (it will move)
- `client/src/locales/en.js` — nav label keys

## Step 2 — Delegate the redesign to vue-expert

Spawn the vue-expert agent with the full spec below. Give it ALL the context it needs: file paths, exact class names to remove/add, the complete target layout, and the design tokens to use. Do not ask it to "figure out" the existing structure — hand that to it explicitly from your Step 1 read.

Also instruct vue-expert to implement the collapsible sidebar: add the `collapsed` ref, the toggle chevron button in the logo section, dynamic `:style` width binding on the sidebar, dynamic `margin-left` on the main content wrapper, `.nav-label` spans for hiding text, and the auto-collapse-on-narrow-viewport `onMounted` check. Do not ask it to "figure out" how to wire these — pass the exact spec from the Collapsible sidebar section below.

---

## Target design spec

### Layout structure

Replace the current column layout (top-nav + content) with a side-by-side layout:

```
┌─────────────────────────────────────────────────────┐
│  Sidebar (220px fixed)  │  Main area (flex: 1)       │
│                         │  ┌─────────────────────┐   │
│  [Logo + subtitle]      │  │  FilterBar (sticky) │   │
│                         │  └─────────────────────┘   │
│  [Nav items vertical]   │  ┌─────────────────────┐   │
│                         │  │  <router-view>       │   │
│  [spacer flex:1]        │  └─────────────────────┘   │
│                         │                             │
│  [Language switcher]    │                             │
│  [Profile menu]         │                             │
└─────────────────────────────────────────────────────┘
```

The `.app` root becomes `display: flex; flex-direction: row; min-height: 100vh`.

### Sidebar

- Width: `220px`, fixed (not scrolls with page)
- Background: `#0f172a` (dark slate)
- Right border: `1px solid rgba(255,255,255,0.06)`
- Position: `position: fixed; top: 0; left: 0; height: 100vh; overflow-y: auto`
- Display: `flex; flex-direction: column`

**Logo section** (top of sidebar):
- Padding: `1.5rem 1.25rem 1.25rem`
- Company name: `font-size: 0.938rem; font-weight: 700; color: #f8fafc; letter-spacing: -0.01em`
- Subtitle: `font-size: 0.75rem; color: #64748b; margin-top: 0.125rem`
- Bottom border: `1px solid rgba(255,255,255,0.06); padding-bottom: 1.25rem; margin-bottom: 0.5rem`

**Nav items** (vertical list):
- Container: `flex: 1; padding: 0.5rem 0.75rem`
- Each `<router-link>`: `display: flex; align-items: center; gap: 0.625rem; padding: 0.5rem 0.75rem; border-radius: 6px; color: #94a3b8; font-size: 0.875rem; font-weight: 500; text-decoration: none; transition: all 0.15s ease; margin-bottom: 2px`
- Hover state: `background: rgba(255,255,255,0.06); color: #f1f5f9`
- Active state: `background: rgba(37,99,235,0.2); color: #93c5fd`
- Icon: inline SVG, `width: 16px; height: 16px; flex-shrink: 0; opacity: 0.8`

Use these minimal inline SVGs (24×24 viewBox, `stroke="currentColor" fill="none" stroke-width="2"`):

| Route | Icon path |
|---|---|
| `/` (Overview) | `M3 9l9-7 9 7v11a2 2 0 01-2 2H5a2 2 0 01-2-2z` (house) |
| `/inventory` | `M20 7H4a1 1 0 00-1 1v10a1 1 0 001 1h16a1 1 0 001-1V8a1 1 0 00-1-1zM9 7V5a2 2 0 012-2h2a2 2 0 012 2v2` (box/package) |
| `/orders` | `M9 5H7a2 2 0 00-2 2v12a2 2 0 002 2h10a2 2 0 002-2V7a2 2 0 00-2-2h-2M9 5a2 2 0 000 4h6a2 2 0 000-4M9 5a2 2 0 012-2h2a2 2 0 012 2` (clipboard) |
| `/spending` | `M12 1v22M17 5H9.5a3.5 3.5 0 000 7h5a3.5 3.5 0 010 7H6` (dollar sign) |
| `/demand` | `M22 12h-4l-3 9L9 3l-3 9H2` (activity/pulse) |
| `/reports` | `M14 2H6a2 2 0 00-2 2v16a2 2 0 002 2h12a2 2 0 002-2V8zM14 2v6h6M16 13H8M16 17H8M10 9H8` (file-text) |
| `/restocking` | `M4 4h16v2H4zM4 10h16v2H4zM4 16h10v2H4zM18 14l4 4-4 4` (list with arrow) |

**Bottom section** (language + profile):
- Padding: `0.75rem`
- Top border: `1px solid rgba(255,255,255,0.06)`
- Language switcher sits above profile
- Both items should feel integrated into the dark sidebar — pass a `dark` prop or use CSS to invert colors if needed. If the existing components are too tightly coupled to light styles, wrap them in a `sidebar-footer` div and use `:deep()` to override: `color: #94a3b8` for text, `background: transparent` for buttons.

### Collapsible sidebar

Add a `collapsed` ref (`const collapsed = ref(false)`) in App.vue's `<script setup>`. Auto-collapse on mount when the viewport is narrow:

```js
onMounted(() => {
  if (window.innerWidth <= 1024) collapsed.value = true
})
```

**Toggle button** — place at the bottom of the logo section, right-aligned:
- Icon: chevron-left SVG when expanded, chevron-right when collapsed
- Style: `background: transparent; border: none; cursor: pointer; color: #64748b; padding: 0.25rem; border-radius: 4px`
- Hover: `color: #94a3b8`

Chevron SVG paths (`viewBox="0 0 24 24"`, `stroke="currentColor" fill="none" stroke-width="2"`):
- Left (expanded): `M15 18l-6-6 6-6`
- Right (collapsed): `M9 18l6-6-6-6`

**Collapsed state (56px wide):**
- Bind width dynamically: `:style="{ width: collapsed ? '56px' : '220px' }"`
- Hide nav labels: wrap label text in `<span class="nav-label">` and add `.sidebar.collapsed .nav-label { display: none }`
- Hide company name and subtitle text: `.sidebar.collapsed .logo-text { display: none }`
- Center icons when collapsed: `.sidebar.collapsed .nav-link { justify-content: center }`
- Add native `title` attribute to each `<router-link>` with the nav label for accessibility tooltips

**Transition** — add to both elements for a smooth animation:
- Sidebar: `transition: width 0.2s ease`
- Main content wrapper: `transition: margin-left 0.2s ease`

**Main content margin** — bind dynamically instead of a fixed value:
```html
<div class="main-content" :style="{ marginLeft: collapsed ? '56px' : '220px' }">
```

Apply `.collapsed` class to the sidebar element when the ref is true: `:class="{ collapsed }"`.

### Main content area

- Left margin: `220px` (to clear the fixed sidebar): `margin-left: 220px; flex: 1; display: flex; flex-direction: column; min-height: 100vh`
- No `max-width` constraint on the wrapper — the page content inside views already handles this

**FilterBar positioning**: Keep `<FilterBar />` where it is (above `<router-view>`), but style it as a sticky top strip:
- In App.vue, wrap FilterBar so it sticks: the FilterBar itself should get `position: sticky; top: 0; z-index: 50`
- Background: `#ffffff; border-bottom: 1px solid #e2e8f0`

**`<router-view>` wrapper**:
- Padding: `1.5rem 2rem`
- Background: `#f8fafc`
- `flex: 1`

### Global CSS updates

Update the `.app` rule and remove `.top-nav`, `.nav-container`, `.nav-tabs` styles (they are replaced by sidebar styles). Keep all other global utility classes (`.card`, `.stat-card`, `.badge`, `.table`, etc.) exactly as they are — do not touch them.

Remove the `max-width: 1600px` constraint from `.main-content` (the sidebar layout naturally controls width). Keep `.main-content` padding.

### Scoped vs global styles

Put sidebar-specific styles inside a `<style>` block in `App.vue` (not scoped, since router-link active classes need to work). Keep all existing global utility classes.

### Preserve all existing functionality

- All modal triggers (ProfileDetailsModal, TasksModal) stay wired to the same events from ProfileMenu
- FilterBar composable behavior unchanged
- LanguageSwitcher behavior unchanged
- All `useI18n` / `t()` calls for nav labels stay
- Route paths unchanged

### Design tokens (do not change these)

```
Background body:   #f8fafc
Sidebar bg:        #0f172a
Sidebar text:      #94a3b8 (inactive), #f1f5f9 (hover), #93c5fd (active)
Active nav bg:     rgba(37,99,235,0.2)
Border (light):    #e2e8f0
Card bg:           #ffffff
Primary blue:      #2563eb
```

No emojis anywhere in the UI.

## Step 3 — Verify in browser

After vue-expert completes, use Playwright (`mcp__playwright__*`) to:
1. Navigate to `http://localhost:3000`
2. Take a screenshot and confirm sidebar is visible on the left
3. Click each nav link and verify the active state updates correctly
4. Confirm the FilterBar is visible at the top of the content area
5. Confirm no horizontal scrollbar appears at 1280px viewport width
6. Click the collapse toggle button — confirm the sidebar shrinks to ~56px, nav labels disappear, and the main content shifts left accordingly
7. Click the toggle again — confirm the sidebar expands back to 220px with labels visible

If any visual issues are found, fix them before reporting done.

# Sidebar Navigation Implementation Brief
## Fractal OS

---

## Executive Summary

Vercel has moved to sidebar navigation (as seen in the attached screenshot), and this brief proposes how Fractal OS can adopt a similar pattern. The goal is cleaner navigation, better information hierarchy, and improved UX — while preserving what works well (like the Project dropdown in the top bar).

---

## Current Architecture Analysis

### Existing Navigation Structure

**FractalPageHeader** (sticky top bar) consists of:
1. **HeaderRow1**: Logo, ContextSelector dropdown, "/" separator, ProjectSelector dropdown, user avatar
2. **HeaderRow2**: Tab buttons (Dashboard, Roadmap, Kanban, etc.) — changes based on selected context/project

**ContextSelector** options (currently a dropdown in Row1):
- Projects
- Team Dashboard
- Repositories
- Users & Apps
- Meetings
- Text Generation
- Files (WIP)
- QR Codes (WIP)

**Project-specific tabs** (shown in Row2 when a project is selected):
- Dashboard, Roadmap, Kanban, Documentation, Repositories, Pull Requests, Deployments, Meetings, Weekly Digests, Analytics
- Plus conditional tabs: Facilities, Access Tokens (for Next Health)

### Key Files to Modify
| File | Purpose |
|------|---------|
| `AppLayout.tsx` | Root layout — would add sidebar here |
| `FractalPageHeader.tsx` | Combines Row1 + Row2 |
| `FractalPageHeaderRow1.tsx` | Logo, context, project selector, avatar |
| `FractalPageHeaderRow2.tsx` | Tab navigation (would be moved to sidebar) |
| `ContextSelector.tsx` | Context dropdown (items would move to sidebar) |
| `FractalPage.tsx` | Main page component, handles routing/context |

---

## Proposed Architecture

### 1. Layout Structure

```
┌────────────────────────────────────────────────────────────────┐
│ [☰] [Fractal Labs ▼]          [Search...]        [👤 Avatar]  │ ← Slim top bar
├───────────────┬────────────────────────────────────────────────┤
│               │                                                │
│   SIDEBAR     │            MAIN CONTENT                        │
│               │                                                │
│  [Projects ▼] │  ┌──────────────────────────────────────────┐  │
│               │  │ Projects / Practice Interviews ▼         │  │
│  Dashboard    │  ├──────────────────────────────────────────┤  │
│  Roadmap      │  │                                          │  │
│  Kanban       │  │    (Page content)                        │  │
│  Docs         │  │                                          │  │
│  Repos        │  │                                          │  │
│  PRs          │  │                                          │  │
│  Deployments  │  │                                          │  │
│  Meetings     │  │                                          │  │
│  ...          │  │                                          │  │
│               │  │                                          │  │
│  ─────────    │  │                                          │  │
│  Settings     │  │                                          │  │
│               │  └──────────────────────────────────────────┘  │
└───────────────┴────────────────────────────────────────────────┘
```

### 2. Information Hierarchy (Vercel Pattern)

**Top Bar (slim)**:
- Hamburger toggle (☰) to collapse/expand sidebar
- "Fractal Labs" org name (clickable → root/home)
- Optional: Global search
- User avatar with dropdown menu

**Sidebar (collapsible)**:
- **Top section**: Context selector (could be "Projects", "Team Dashboard", etc.)
- **Main section**: Navigation items that update based on context
- **Bottom section**: Settings, help, etc.

**Main Content Area**:
- **Content top bar**: Project selector dropdown (when in Projects context)
- **Page content**: The actual page being viewed

### 3. How Sidebar Updates Based on Context

| Context | Sidebar Shows |
|---------|--------------|
| **Projects** (no project selected) | Dashboard, Audit |
| **Projects** (project selected) | Dashboard, Roadmap, Kanban, Docs, Repos, PRs, Deployments, Meetings, Weekly Digests, Analytics, [project-specific tabs] |
| **Team Dashboard** | Everyone, [team member tabs if enabled] |
| **Repositories** | List, Audit |
| **Users & Apps** | Team, Clients, Apps, Roles, Permissions |
| **Meetings** | Recordings |
| **Text Generation** | Models |

This mirrors exactly how Vercel's sidebar changes when you select a project vs. organization-level views.

---

## Key Design Decisions

### Decision 1: Project Selector Location

**Recommendation: Keep in main content top bar (like Vercel)**

Reasons:
- Austin explicitly noted he likes this pattern
- Separates "what section am I in" (sidebar) from "which project" (content area)
- Project context feels more like "content scope" than navigation
- Matches Vercel's approach

**Implementation**: `ProjectSelector` component moves from `FractalPageHeaderRow1` to a new content-area header component.

---

### Decision 2: Sidebar Collapse/Expand

**Options**:

| Option | Pros | Cons |
|--------|------|------|
| **A. Icon-only when collapsed** | More space, quick toggle, familiar pattern | Must rely on icons being recognizable |
| **B. Hover to expand** | Full labels on demand | Can feel jumpy, harder on mobile |
| **C. Full collapse (hamburger opens overlay)** | Maximum content space | Extra click to navigate |

**Recommendation: Option A (icon-only collapse)**
- Toggle button in top bar
- Collapsed: 56-64px wide, icons only, tooltip on hover
- Expanded: 240-280px wide, icons + labels
- Persist preference in localStorage

---

### Decision 3: "Fractal Labs" at Top of Sidebar

**Vercel behavior**: Clicking org name at top of sidebar navigates to root/dashboard.

**Recommendation**: Same pattern.
- Shows "Fractal Labs" with dropdown arrow
- Clicking navigates to `/projects` (or team dashboard, depending on default)
- Could support org switcher in future (multi-tenant)

---

### Decision 4: Dividers and Grouping

Current `FractalPageHeaderRow2` has a divider before "Facilities" for Next Health. Sidebar can handle this better:

**Recommendation**: Group sidebar items into sections:
```
[Core]
  Dashboard
  Roadmap
  Kanban
  Documentation

[Development]
  Repositories
  Pull Requests
  Deployments

[Insights]
  Meetings
  Weekly Digests
  Analytics

[Next Health Only]
  Facilities
  Access Tokens
```

---

### Decision 5: Mobile/Responsive Behavior

**Breakpoints**:
- **Desktop (≥1024px)**: Full sidebar, collapsible
- **Tablet (768-1023px)**: Sidebar collapsed by default, expandable
- **Mobile (<768px)**: Sidebar as overlay/drawer (hamburger to open)

---

## Edge Cases & Considerations

### 1. Permission-based Visibility
Current code filters tabs by permission. Same logic applies to sidebar items.
```typescript
// Current pattern in FractalPageHeaderRow2
const tabs = projectTabs.filter(tab => hasPermission(tab.permission));
// Apply same filtering to sidebar items
```

### 2. Client Role Special Handling
Next Health clients see limited tabs. Sidebar must respect same logic:
```typescript
if (hasClientRole && isNextHealth) {
  // Show only: Dashboard, Roadmap, Facilities, Access Tokens
}
```

### 3. Active State Indication
Current: underline on selected tab.
Sidebar: highlight background + left border accent on active item.

### 4. Settings Button
Currently appears in Row2 when viewing a project. Move to:
- Bottom of sidebar, OR
- Gear icon next to project name in content header

### 5. Keyboard Navigation
- Arrow keys to move between sidebar items
- Enter to select
- `Cmd+K` or `/` for quick search (optional enhancement)

### 6. Animation/Transitions
- Sidebar expand/collapse: 200-300ms ease
- Active item highlight: subtle fade
- Overlay on mobile: slide from left

---

## Implementation Approach

### Phase 1: Structural Changes
1. Create `Sidebar.tsx` component
2. Create `SidebarItem.tsx` for individual nav items
3. Create `SidebarContext.tsx` for collapsed state management
4. Modify `AppLayout.tsx` to include sidebar
5. Create `ContentHeader.tsx` for project selector in main area

### Phase 2: Migration
1. Move context items from `ContextSelector` to sidebar
2. Move tab items from `FractalPageHeaderRow2` to sidebar
3. Simplify `FractalPageHeader` to slim top bar only
4. Update all routing logic in `FractalPage.tsx`

### Phase 3: Polish
1. Add collapse/expand functionality
2. Implement responsive breakpoints
3. Add transitions/animations
4. Persist sidebar state
5. Keyboard navigation

---

## Questions for Discussion

### 1. Default Sidebar State
Should the sidebar default to expanded or collapsed on first visit?
- **Expanded**: Better discoverability for new users
- **Collapsed**: More content space, power-user friendly

### 2. Context Switcher in Sidebar
Current ContextSelector is a dropdown. In sidebar, options:
- **A. Clickable items** (Projects, Team Dashboard, etc. as top-level items)
- **B. Collapsible sections** (expand "Projects" to see sub-items)
- **C. Keep as dropdown** at top of sidebar

**Vercel does B** — but Fractal OS has fewer contexts, so A might be cleaner.

### 3. Settings Panel Trigger
Where should the Settings button go?
- **A. Bottom of sidebar** (always visible)
- **B. Next to project name** in content header (more contextual)
- **C. Both** (global settings in sidebar, project settings next to project name)

### 4. Search
Should we add global search to the top bar now, or defer?
- Vercel has it prominently
- Could be a "nice to have" after nav migration

### 5. MenuPage Fate
Currently there's a `MenuPage.tsx` showing all options in a card grid. Options:
- **A. Remove** (sidebar replaces it)
- **B. Keep as alternative view** (accessed from sidebar)
- **C. Convert to "Home" dashboard**

---

## Visual Reference

The attached Vercel screenshot shows:
- Slim top bar with org selector, search, notifications, avatar
- Left sidebar with:
  - "Projects" item with expand arrow
  - Context-specific items: Deployments, Logs, Analytics, Speed Insights, etc.
  - Nested groups (Observability, Domains, etc.)
- Main content area with project-specific breadcrumb/selector
- Deployments list in main content

Key patterns to adopt:
1. **Hierarchical sidebar** with expand/collapse sections
2. **Project selector in content area**, not sidebar
3. **Org name at top** returns to root
4. **Icons + labels** in sidebar
5. **Active state** with left border accent

---

## Summary

This migration moves Fractal OS from a horizontal tab-based navigation to a vertical sidebar pattern, matching modern dashboard UX conventions (Vercel, Linear, Notion, etc.). The change will:

- **Improve scalability** as more features are added
- **Reduce cognitive load** by showing full navigation at a glance
- **Support mobile better** with drawer-based sidebar
- **Feel more premium** and aligned with developer tool conventions

The main work is structural (new Sidebar component, modified AppLayout), with migration of existing navigation logic from headers to sidebar.

---

*Brief prepared by Ruk • 2026-02-02*

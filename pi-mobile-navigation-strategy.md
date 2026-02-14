# Practice Interviews: Mobile Navigation Strategy

## The Challenge

We need to fit the core app functionality into a 5-icon bottom tab bar while maintaining intuitive access to all features. The sidebar currently has 8+ navigation items — we need to prioritize ruthlessly.

---

## Proposed Bottom Tab Bar (5 Icons)

| Icon | Label | Primary Function | Why It's Core |
|------|-------|------------------|---------------|
| 🏠 | **Home** | Dashboard / Opportunities | The organizing principle — everything flows from here |
| 📝 | **Practice** | Practice sessions | The main action users take repeatedly |
| 📚 | **Prep** | Stories + Questions (banks) | Essential context for practice |
| 📊 | **Progress** | Analytics / mastery tracking | Motivation + insights |
| 👤 | **Profile** | User profile + settings | Account management |

---

## Navigation Mapping

### What Goes Where

**Bottom Bar Icons → Primary Views:**

1. **Home** → Dashboard with opportunity cards, upcoming interviews, quick actions
2. **Practice** → Start session, recent sessions, practice templates
3. **Prep** → Tabbed view: Stories | Questions | Resources (learning materials)
4. **Progress** → Analytics: mastery scores, practice history, strengths/gaps
5. **Profile** → Profile editor, career goals, settings, help

### Secondary Navigation (Within Views)

Rather than a hamburger menu, we use **in-page navigation**:

- **Swipeable tabs** within each section (e.g., Stories | Questions in Prep)
- **Floating action button** for primary actions (+ Add Story, + New Practice)
- **Pull-down menus** for filtering/sorting

---

## Layout Patterns

### Desktop → Mobile Transformation

| Desktop | Mobile |
|---------|--------|
| Sidebar (200px) | Bottom tab bar (56px) |
| Side-by-side panels | Stacked full-width cards |
| Hover tooltips | Long-press tooltips |
| Right-click context | Swipe actions (edit/delete) |
| Modal dialogs | Full-screen drawers (bottom sheet) |

### Specific Screens

**Dashboard (Home)**
- Opportunities as horizontal scrollable cards
- "Next Interview" prominent at top
- Quick stats condensed to single row
- FAB for "Add Opportunity"

**Practice Session**
- Full-screen focused mode
- Timer prominent
- Single question/prompt visible
- Swipe for next question
- Bottom sheet for notes/STAR input

**Story Bank (in Prep)**
- List view with mastery indicators
- Swipe right to archive, left to edit
- Tap to expand full STAR
- FAB for "Add Story"

**Question Bank (in Prep)**
- Category chips at top (scrollable)
- Cards with competency tags
- Tap to see linked stories
- FAB for "Add Question"

---

## Touch Considerations

### Tap Targets
- Minimum 44x44px for all interactive elements
- 8px minimum spacing between targets
- Primary actions in thumb reach zone (bottom third of screen)

### Gestures
- **Swipe horizontal** — Tab navigation, card actions
- **Swipe vertical** — Scroll, pull-to-refresh
- **Long press** — Context menu (tooltips, secondary actions)
- **Pinch** — Not used (no zoom requirements)

### Bottom Tab Bar Specs
- Height: 56px (+ safe area on notched phones)
- Icon size: 24px
- Label size: 12px
- Active state: Green icon + label, subtle top border
- Inactive: Gray icon, no label (or smaller label)

---

## Edge Cases

### Deep Links
Each bottom tab should remember its last state:
- Home → last viewed opportunity
- Practice → resume interrupted session
- Prep → last tab (Stories vs Questions)

### Offline Mode (Future)
- Prep content (stories/questions) cached for offline practice
- Sessions can be completed offline, synced later
- Bottom bar shows offline indicator

### Tablet (768-1024px)
- Hybrid: Icon-only sidebar (56px) + content area
- Bottom bar hidden, sidebar takes over
- Same 5 primary destinations

---

## Implementation Phases

### Phase 1: Foundation
1. Add responsive breakpoints to Tailwind config
2. Create `<MobileNav />` component (bottom tab bar)
3. Create `<ResponsiveLayout />` that switches sidebar ↔ bottom bar
4. Update `<AppLayout />` to be responsive

### Phase 2: Screen Updates
1. Dashboard — stack cards vertically, add FAB
2. Story Bank — swipe actions, compact list
3. Question Bank — scrollable chips, compact cards
4. Profile — accordion sections vs tabs

### Phase 3: Polish
1. Touch feedback (ripple effects)
2. Gesture handlers (swipe actions)
3. Bottom sheet modals
4. Safe area handling

---

## Open Questions for Matt

1. **Academy** — Is this a core feature for v1, or can it live under Prep > Resources?
2. **Opportunities vs Home** — Should "Home" be a dashboard OR a direct list of opportunities?
3. **Settings** — Separate tab, or nested under Profile?
4. **Notifications** — Bell icon in header, or badge on relevant tab?

---

*This brief provides the foundation for mobile-first development. All new components should be built with these patterns in mind from the start.*

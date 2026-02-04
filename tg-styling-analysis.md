# TG Styling & Theme Analysis

## Executive Summary

**Yes, this is a genuine issue.** You have a good token system (`tokens.ts`) but it's only used by 27% of files. The rest use hardcoded colors, creating dark mode inconsistencies.

---

## The Numbers

| Metric | Count | Problem |
|--------|-------|---------|
| **Hardcoded hex colors** | 212 instances | Colors don't adapt to dark mode |
| **Hardcoded rgb/rgba** | 24 instances | Same problem |
| **Inline style={{}} objects** | 229 instances | Hard to maintain, recreated every render |
| **`isDarkTheme ? x : y` ternaries** | 78 instances | Manual dark mode = inconsistent |
| **Files using tokens** | 76 / 282 (27%) | Token system exists but underused |

---

## What You Have

### Good: A Token System Already Exists

`src/src/styles/tokens.ts` provides:
```typescript
tokens.colors.background  // var(--color-background)
tokens.colors.text        // var(--color-text)
tokens.colors.primary     // var(--color-primary)
// etc.
```

CSS variables in `theme.css` handle dark mode automatically.

### Bad: Most Code Ignores It

**Pattern 1: Hardcoded colors everywhere**
```typescript
// src/src/components/settings/FormTemplates.tsx:367
color: isDarkTheme ? '#a0a0a0' : '#666'

// Should be:
color: tokens.colors.textMuted
```

**Pattern 2: Manual dark mode ternaries**
```typescript
backgroundColor: isDarkTheme ? '#2d2d2d' : '#f5f5f5'

// Should be:
backgroundColor: tokens.colors.surface
```

**Pattern 3: Same color, different values**
- `#ffffff` appears 14 times
- `#FFFFFF` appears 6 times
- `#fff` appears 2 times

All should be `tokens.colors.background`.

---

## Top Offenders

Files with most hardcoded colors:

| File | Hardcoded Colors | Dark Mode Ternaries |
|------|-----------------|---------------------|
| `FormTemplates.tsx` | 15+ | 8 |
| `StatusBadge.tsx` | 10+ | - |
| `CustomCalendar.tsx` | 8+ | 4 |
| `CalendarEventRenderer.tsx` | 6+ | 3 |
| `CreateClientModal.tsx` | 5+ | 2 |

---

## The Fix: Token Migration

### Phase 1: Expand Token Definitions
Add missing tokens for colors that appear frequently:
- Status badge colors (paid, unpaid, cancelled, etc.)
- Calendar event colors by type
- Form field states

### Phase 2: Search-and-Replace Migration

For each common hardcoded value:
```
#ffffff / #FFFFFF / #fff → tokens.colors.background
#000000 / #000 → tokens.colors.text
#666 / #666666 / #888 → tokens.colors.textMuted
#f5f5f5 → tokens.colors.surface
#2d2d2d / #292929 → tokens.colors.surface (dark)
#6264a7 → tokens.colors.primary
#0078d4 → tokens.colors.info (or define teamsBlue)
```

### Phase 3: Kill the Ternaries

Replace all `isDarkTheme ? darkColor : lightColor` with token references. The CSS variables handle dark mode automatically.

---

## Effort Estimate

**Agent-driven concentrated sprint approach:**

1. **Expand tokens.ts** (1-2 hours)
   - Add status colors, event colors, form states

2. **Search-replace common patterns** (2-4 hours)
   - ~200 hardcoded colors
   - Systematic file-by-file sweep

3. **Remove isDarkTheme ternaries** (2-3 hours)
   - 78 instances
   - Some may need new token definitions

4. **Test dark mode** (ongoing)
   - Toggle dark mode, verify each screen

**Total: ~1 day of focused work**

This is a perfect agent task - mechanical, repetitive, needs consistency across 282 files.

---

## Recommendation

Add this as **M-73: Phase 6d - Theme Token Migration** under E-20, to be executed alongside the feature refactors:

1. When doing calendar refactor, also migrate calendar styling to tokens
2. When doing clients refactor, also migrate client styling to tokens
3. Remaining files get a dedicated styling pass

This way the styling cleanup piggybacks on the structural refactor - you're already touching every file anyway.

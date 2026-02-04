# Vitaboom Open PRs - Cleanup Guide

## Summary

**Frontend (vitaboom-web): 4 open PRs**
**Backend (vitaboom-backend): 6 open PRs**

---

## Recommended Actions

### 🟢 Ready to Merge (after review)

| Repo | PR | Title | Author | Status |
|------|-----|-------|--------|--------|
| Frontend | [#122](https://github.com/FractalLabsDev/vitaboom-web/pull/122) | W9 upload popup with commission opt-out | ruk-fl | Mergeable, needs review |
| Frontend | [#121](https://github.com/FractalLabsDev/vitaboom-web/pull/121) | Commission filters + shift+select | sdk-fractal | Mergeable, 2 reviews |
| Backend | [#109](https://github.com/FractalLabsDev/vitaboom-backend/pull/109) | commissionOptOut field for W9 popup | ruk-fl | Mergeable, needs review |
| Backend | [#107](https://github.com/FractalLabsDev/vitaboom-backend/pull/107) | Create bill without bill payment | sdk-fractal | Mergeable, 2 reviews |

**Recommendation:** These are current work. Review and merge in order:
1. **#109 + #122** (W9/commission opt-out - backend first, then frontend)
2. **#107** (QuickBooks bill refactor)
3. **#121** (Commission UI updates)

---

### 🟡 Needs Conflict Resolution

| Repo | PR | Title | Author | Issue |
|------|-----|-------|--------|-------|
| Frontend | [#117](https://github.com/FractalLabsDev/vitaboom-web/pull/117) | Hide commissions toggle | ruk-fl | Merge conflicts |
| Backend | [#104](https://github.com/FractalLabsDev/vitaboom-backend/pull/104) | hideCommissions toggle to clinic model | ruk-fl | Merge conflicts |
| Backend | [#96](https://github.com/FractalLabsDev/vitaboom-backend/pull/96) | Remove unused code | SerhiiYermolenko | Merge conflicts |
| Backend | [#82](https://github.com/FractalLabsDev/vitaboom-backend/pull/82) | Strict vendor+title matching for SuppCo | mattlim-fl | Merge conflicts |

**Recommendation:**
- **#117 + #104**: The "hide commissions" feature - decide if still needed. If yes, rebase onto main and resolve conflicts.
- **#96**: Serhii's cleanup - large deletion PR (577 lines). Review carefully before rebasing.
- **#82**: Your SuppCo matching PR from Jan 9. Check if this is still needed or if the approach changed.

---

### 🔴 Likely Stale - Consider Closing

| Repo | PR | Title | Author | Age |
|------|-----|-------|--------|-----|
| Frontend | [#110](https://github.com/FractalLabsDev/vitaboom-web/pull/110) | Remove unused code | SerhiiYermolenko | Jan 23 (12 days) |
| Backend | [#9](https://github.com/FractalLabsDev/vitaboom-backend/pull/9) | Project-setup | MaxDonchenko | Oct 3, 2025 (4+ months) |

**Recommendation:**
- **#110**: Frontend cleanup - 6,484 deletions across 111 files. This is substantial. Coordinate with Serhii on whether to proceed or close.
- **#9**: Ancient project setup PR. Almost certainly stale. Close unless there's specific history worth preserving.

---

## Suggested Cleanup Workflow

1. **This week:**
   - Merge the ready PRs (#122, #121, #109, #107) in dependency order
   - Close backend #9 (stale project setup)

2. **Next:**
   - Serhii rebases #96 and #110, or close if superseded
   - Decide on hide-commissions feature (#117, #104) - rebase or close

3. **Review needed:**
   - #82 (SuppCo matching) - Matt, is this still the right approach?

---

## PR Dependencies

```
Backend #109 (W9 opt-out field)
    └── Frontend #122 (W9 popup UI)

Backend #104 (hideCommissions model)
    └── Frontend #117 (hideCommissions toggle UI)
```

Merge backend PRs first when there are frontend/backend pairs.

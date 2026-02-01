# GitHub Hygiene Report - 2026-02-01

## 🚨 Summary

| Metric | Count |
|--------|-------|
| **Stale branches (>30 days)** | 114 |
| **Open PRs needing review (>7 days)** | 12 |

---

## 🌿 Stale Branches by Repo

### Critical (>1 year old)
| Repo | Branch | Age |
|------|--------|-----|
| fractalai-web | kris-feb-01 | 729 days |
| mandelbrot-monitor-backend | kris-feb-01 | 728 days |
| fractalai-backend | branching | 663 days |
| fractalai-web | austin-jun14 | 596 days |
| fractalai-backend | july6 | 574 days |
| fractalai-backend | temp/pre-modular-feedback | 562 days |
| fractalai-backend | july19 | 562 days |
| fractalai-web | feature/quill-editor | 558 days |
| fractalai-backend | austin-jul30 | 550 days |

### Active Projects with Stale Branches

**vitaboom-web (14 stale branches)**
- feature/add-discount-to-cart (101 days)
- feature/single-use-discounts (94 days)
- testing/remove-redirect-password-reset (121 days)
- And 11 more...

**vitaboom-backend (24 stale branches)**
- added-prettier-and-autoformat-on-save (120 days)
- feature/add-discount-to-cart (102 days)
- And 22 more...

**soash-web (20 stale branches)** ⚠️ Project appears inactive
- develop (172 days)
- Most branches 140-190 days old

**therapygenie-backend (8 stale branches)**
- refactor (122 days)
- calendar_performance (97 days)
- And 6 more...

---

## 🔍 Open PRs Needing Attention

### Active Projects
| Repo | PR | Title | Link |
|------|-----|-------|------|
| practice-interviews-backend | #87 | Replace Wordware with TalkWise Oracle | [View](https://github.com/FractalLabsDev/practice-interviews-backend/pull/87) |
| vitaboom-web | #110 | Remove unused code | [View](https://github.com/FractalLabsDev/vitaboom-web/pull/110) |
| vitaboom-web | #108 | Public storefront and checkout flow | [View](https://github.com/FractalLabsDev/vitaboom-web/pull/108) |
| vitaboom-backend | #96 | Remove unused code | [View](https://github.com/FractalLabsDev/vitaboom-backend/pull/96) |
| vitaboom-backend | #82 | SuppCo vendor+title matching | [View](https://github.com/FractalLabsDev/vitaboom-backend/pull/82) |
| talkwise-oracle | #4 | Add response_format support for Grok | [View](https://github.com/FractalLabsDev/talkwise-oracle/pull/4) |
| talkwise-infra | #1 | Auto backup DB script | [View](https://github.com/FractalLabsDev/talkwise-infra/pull/1) |

### Legacy/Inactive Projects
| Repo | PR | Title |
|------|-----|-------|
| fractalai-web | #9 | dev to main |
| fractalai-web | #8 | Feature/quill editor |
| vitaboom-backend | #9 | Project-setup |
| next-health-dashboard-preview | #27 | Fix TypeScript errors |
| next-health-dashboard-preview | #26 | Codebase optimization |

---

## 💡 Recommendations

### Quick Wins (Safe to Delete)
1. **All fractalai-backend branches** - Project appears dormant, branches 280-660+ days old
2. **All fractalai-web branches** - Same situation
3. **All mandelbrot-monitor-backend branches** - Dormant project
4. **soash-web branches** - If project is cancelled, clean up all 20 branches

### Requires Review Before Action
1. **vitaboom PRs #110 and #96** - "Remove unused code" PRs should be reviewed/merged
2. **therapygenie-backend refactor branch** - 122 days old, may contain valuable work
3. **talkwise-infra PR #1** - DB backup script - should this be merged or closed?

### Enable Auto-Delete on Merge
These repos don't have it enabled:
- vitaboom-backend
- vitaboom-web
- therapygenie-backend
- practice-interviews-backend

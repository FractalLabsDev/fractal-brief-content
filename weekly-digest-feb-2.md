# Weekly Development Digest
**Week of February 2, 2026**

Generated: 2026-02-06

---

## Executive Summary

This week saw significant activity across Practice Interviews and Therapist Genie, with notable bug fixes and feature enhancements. VitaBoom received commission management updates and W9 compliance features. Next Health had no merged PRs this week.

| Project | Merged PRs | Open PRs | Open Issues |
|---------|-----------|----------|-------------|
| Practice Interviews | 16 | 6 | 22 |
| Therapist Genie | 19 | 0 | 12 |
| VitaBoom | 12 | 1 | 11 |
| Next Health | 0 | 0 | 25 |

---

## Practice Interviews

### Merged PRs (Last 7 Days)

#### practice-interviews-web (5 PRs)

| # | Title | Author | Merged |
|---|-------|--------|--------|
| [#412](https://github.com/FractalLabsDev/practice-interviews-web/pull/412) | Bugfix: Add new PostHog key to env | Max Donchenko | Feb 5 |
| [#408](https://github.com/FractalLabsDev/practice-interviews-web/pull/408) | Release 2026-02-04 | Max Donchenko | Feb 4 |
| [#406](https://github.com/FractalLabsDev/practice-interviews-web/pull/406) | Make rate limits per-user (#97) | Max Donchenko | Feb 4 |
| [#404](https://github.com/FractalLabsDev/practice-interviews-web/pull/404) | Update UI to use new ideal answer endpoint | Max Donchenko | Feb 2 |
| [#397](https://github.com/FractalLabsDev/practice-interviews-web/pull/397) | Add comprehensive PostHog analytics tracking events | ruk-fl | Jan 30 |

#### practice-interviews-backend (8 PRs)

| # | Title | Author | Merged |
|---|-------|--------|--------|
| [#103](https://github.com/FractalLabsDev/practice-interviews-backend/pull/103) | Add user preferences API for onboarding checklist persistence | ruk-fl | Feb 5 |
| [#102](https://github.com/FractalLabsDev/practice-interviews-backend/pull/102) | Release 2026-02-04 | Max Donchenko | Feb 4 |
| [#101](https://github.com/FractalLabsDev/practice-interviews-backend/pull/101) | Fail duplicate answer (userId + questionId + answerText) | Max Donchenko | Feb 4 |
| [#100](https://github.com/FractalLabsDev/practice-interviews-backend/pull/100) | Limit to 10 answers/day for free/free trial users | Max Donchenko | Feb 4 |
| [#99](https://github.com/FractalLabsDev/practice-interviews-backend/pull/99) | Hotfix: Fix Pino Node compatibility | Max Donchenko | Feb 4 |
| [#98](https://github.com/FractalLabsDev/practice-interviews-backend/pull/98) | Make rate limits per-user (#97) | Max Donchenko | Feb 4 |
| [#96](https://github.com/FractalLabsDev/practice-interviews-backend/pull/96) | Pino logging - cleaner logs, less space | Max Donchenko | Feb 2 |
| [#94](https://github.com/FractalLabsDev/practice-interviews-backend/pull/94) | Use talkwise-promptsmith for ideal answer generation | Max Donchenko | Feb 2 |

#### fractalai-backend (3 PRs)

| # | Title | Author | Merged |
|---|-------|--------|--------|
| [#118](https://github.com/FractalLabsDev/fractalai-backend/pull/118) | Release 2026-02-04 | Max Donchenko | Feb 4 |
| [#117](https://github.com/FractalLabsDev/fractalai-backend/pull/117) | Don't crash server if Wordware fails (invalid JSON) | Max Donchenko | Feb 4 |
| [#115](https://github.com/FractalLabsDev/fractalai-backend/pull/115) | Make rate limits per-user (#97) | Max Donchenko | Feb 4 |

### Open Pull Requests

| # | Title | Author |
|---|-------|--------|
| [#416](https://github.com/FractalLabsDev/practice-interviews-web/pull/416) | Use v2 endpoints for feedback (no streaming) | Max Donchenko |
| [#414](https://github.com/FractalLabsDev/practice-interviews-web/pull/414) | Add enterprise license management UI | ruk-fl |
| [#413](https://github.com/FractalLabsDev/practice-interviews-web/pull/413) | Add onboarding checklist and recommended target role | mattlim-fl |
| [#407](https://github.com/FractalLabsDev/practice-interviews-web/pull/407) | Add FingerprintJS Pro for device fingerprinting | ruk-fl |
| [#405](https://github.com/FractalLabsDev/practice-interviews-web/pull/405) | Fix: Home screen analytics and 'Mix of all three' UI | ruk-fl |
| [#403](https://github.com/FractalLabsDev/practice-interviews-web/pull/403) | Fix bugs from product walkthrough Loom | ruk-fl |

### Open Issues (Highlights)

| # | Title | Created |
|---|-------|---------|
| [#415](https://github.com/FractalLabsDev/practice-interviews-web/issues/415) | NACE Landing Page: Separate landing page for Jeff's March presentation | Feb 5 |
| [#402](https://github.com/FractalLabsDev/practice-interviews-web/issues/402) | Bug: Subscription modal has inconsistent card sizes | Jan 30 |
| [#401](https://github.com/FractalLabsDev/practice-interviews-web/issues/401) | Bug: Two loading spinners in body language section | Jan 30 |
| [#400](https://github.com/FractalLabsDev/practice-interviews-web/issues/400) | Bug: Feedback page shows 'NA' instead of spinner | Jan 30 |
| [#85](https://github.com/FractalLabsDev/practice-interviews-backend/issues/85) | Delete hacker's media uploads | Jan 23 |

**Total Open Issues:** 22 (web: 22, backend: 7, fractalai: 1)

---

## Therapist Genie

### Merged PRs (Last 7 Days)

#### therapygenie (9 PRs)

| # | Title | Author | Merged |
|---|-------|--------|--------|
| [#265](https://github.com/FractalLabsDev/therapygenie/pull/265) | Fix: Copy and appointment numbering issues | ruk-fl | Feb 5 |
| [#261](https://github.com/FractalLabsDev/therapygenie/pull/261) | Fix app freezing while typing in progress note | Rigo Moran | Feb 4 |
| [#258](https://github.com/FractalLabsDev/therapygenie/pull/258) | Add client signature option for treatment plans | mattlim-fl | Feb 4 |
| [#254](https://github.com/FractalLabsDev/therapygenie/pull/254) | Fix date object creation from string | Rigo Moran | Feb 4 |
| [#253](https://github.com/FractalLabsDev/therapygenie/pull/253) | Fix when approving progress note | Rigo Moran | Feb 4 |
| [#252](https://github.com/FractalLabsDev/therapygenie/pull/252) | Added void to async function | Rigo Moran | Feb 3 |
| [#249](https://github.com/FractalLabsDev/therapygenie/pull/249) | Several bug fixes | ruk-fl | Feb 3 |
| [#233](https://github.com/FractalLabsDev/therapygenie/pull/233) | Fix critical bugs: session notes, checkbox, telehealth, clinician dropdown, ICD-10 codes | ruk-fl | Feb 2 |
| [#220](https://github.com/FractalLabsDev/therapygenie/pull/220) | Fix error creating progress notes for recurring appointments | Rigo Moran | Jan 31 |

#### therapygenie-backend (10 PRs)

| # | Title | Author | Merged |
|---|-------|--------|--------|
| [#83](https://github.com/FractalLabsDev/therapygenie-backend/pull/83) | Validations to exclude exception dates and check existing payments | Rigo Moran | Feb 6 |
| [#82](https://github.com/FractalLabsDev/therapygenie-backend/pull/82) | Modify cron for child appointment creation (every 20 min) | Rigo Moran | Feb 5 |
| [#81](https://github.com/FractalLabsDev/therapygenie-backend/pull/81) | Fix child primary contact email | Rigo Moran | Feb 5 |
| [#80](https://github.com/FractalLabsDev/therapygenie-backend/pull/80) | Fix: Centralize client portal URL for magic links | ruk-fl | Feb 4 |
| [#78](https://github.com/FractalLabsDev/therapygenie-backend/pull/78) | Update clientAgreedAt on treatment plan when client signs | mattlim-fl | Feb 4 |
| [#77](https://github.com/FractalLabsDev/therapygenie-backend/pull/77) | Progress note modifications | Rigo Moran | Feb 4 |
| [#76](https://github.com/FractalLabsDev/therapygenie-backend/pull/76) | Backend fixes: #212, #235, #236, #237 + client profiles | mattlim-fl | Feb 3 |
| [#72](https://github.com/FractalLabsDev/therapygenie-backend/pull/72) | Fix error creating progress notes for recurring appointments | Rigo Moran | Jan 31 |
| [#71](https://github.com/FractalLabsDev/therapygenie-backend/pull/71) | Merge dev to dataverse-prod | Rigo Moran | Jan 30 |
| [#70](https://github.com/FractalLabsDev/therapygenie-backend/pull/70) | Add endpoint for different client profiles | Rigo Moran | Jan 30 |

### Open Issues (Highlights)

| # | Title | Labels | Created |
|---|-------|--------|---------|
| [#263](https://github.com/FractalLabsDev/therapygenie/issues/263) | Bug: Client portal shows owed money for future sessions | bug | Feb 4 |
| [#262](https://github.com/FractalLabsDev/therapygenie/issues/262) | Bug: Payment tab shows paid status even when no card on file | bug | Feb 4 |
| [#259](https://github.com/FractalLabsDev/therapygenie/issues/259) | Bug: Client portal access emails not sent to minors | - | Feb 4 |
| [#257](https://github.com/FractalLabsDev/therapygenie/issues/257) | Cannot delete cancelled events from archived test account | bug | Feb 4 |
| [#251](https://github.com/FractalLabsDev/therapygenie/issues/251) | In payroll users are seeing old payments | bug | Feb 3 |
| [#206](https://github.com/FractalLabsDev/therapygenie/issues/206) | Discovery: How does Dataverse manage backups? | Top Priority | Jan 27 |

**Total Open Issues:** 12

---

## VitaBoom

### Merged PRs (Last 7 Days)

#### vitaboom-web (7 PRs)

| # | Title | Author | Merged |
|---|-------|--------|--------|
| [#125](https://github.com/FractalLabsDev/vitaboom-web/pull/125) | Fix build | Serhii Yermolenko | Feb 5 |
| [#124](https://github.com/FractalLabsDev/vitaboom-web/pull/124) | Add export all commissions to CSV | mattlim-fl | Feb 5 |
| [#123](https://github.com/FractalLabsDev/vitaboom-web/pull/123) | Redesign clinics table and add archive/restore | mattlim-fl | Feb 5 |
| [#122](https://github.com/FractalLabsDev/vitaboom-web/pull/122) | Add W9 upload popup on login with commission opt-out | ruk-fl | Feb 5 |
| [#121](https://github.com/FractalLabsDev/vitaboom-web/pull/121) | Update commission filters and add shift+select | sdk-fractal | Feb 5 |
| [#111](https://github.com/FractalLabsDev/vitaboom-web/pull/111) | Enhanced change role frontend | sdk-fractal | Jan 30 |
| [#110](https://github.com/FractalLabsDev/vitaboom-web/pull/110) | Remove unused code | Serhii Yermolenko | Feb 5 |

#### vitaboom-backend (5 PRs)

| # | Title | Author | Merged |
|---|-------|--------|--------|
| [#110](https://github.com/FractalLabsDev/vitaboom-backend/pull/110) | Support includeArchived query param | mattlim-fl | Feb 5 |
| [#109](https://github.com/FractalLabsDev/vitaboom-backend/pull/109) | Add commissionOptOut field for W9 popup opt-out | ruk-fl | Feb 5 |
| [#107](https://github.com/FractalLabsDev/vitaboom-backend/pull/107) | When paying clinic, only create bill in QuickBooks without bill payment | sdk-fractal | Feb 5 |
| [#104](https://github.com/FractalLabsDev/vitaboom-backend/pull/104) | Add hideCommissions toggle to clinic model | ruk-fl | Feb 5 |
| [#96](https://github.com/FractalLabsDev/vitaboom-backend/pull/96) | Remove unused code | Serhii Yermolenko | Feb 5 |

### Open Pull Requests

| # | Title | Author |
|---|-------|--------|
| [#117](https://github.com/FractalLabsDev/vitaboom-web/pull/117) | Add hide commissions toggle for admins/owners | ruk-fl |

### Open Issues (Highlights)

| # | Title | Created |
|---|-------|---------|
| [#116](https://github.com/FractalLabsDev/vitaboom-web/issues/116) | Add Supp.co ratings for new products (Atrantil Pro, Prodrome Neuro/Glial) | Jan 30 |
| [#108](https://github.com/FractalLabsDev/vitaboom-backend/issues/108) | Use pino for Webhook logging | Feb 4 |
| [#105](https://github.com/FractalLabsDev/vitaboom-backend/issues/105) | We need not to log the password | Feb 2 |
| [#105](https://github.com/FractalLabsDev/vitaboom-web/issues/105) | Optimize clients page backend calls | Jan 22 |

**Total Open Issues:** 11 (web: 3, backend: 8)

---

## Next Health

### Merged PRs (Last 7 Days)

No merged PRs this week for next-health-web or next-health-mobile.

### Open Issues (Highlights)

| # | Title | Created |
|---|-------|---------|
| [#216](https://github.com/FractalLabsDev/next-health-mobile/issues/216) | Don't show slashed price if equal to discounted price | Feb 4 |
| [#215](https://github.com/FractalLabsDev/next-health-mobile/issues/215) | Unlimited cryo for members | Feb 3 |
| [#212](https://github.com/FractalLabsDev/next-health-mobile/issues/212) | Editing email during sign up bug | Jan 19 |
| [#210](https://github.com/FractalLabsDev/next-health-mobile/issues/210) | Soft Open package should not be hardcoded to $69 | Jan 6 |
| [#209](https://github.com/FractalLabsDev/next-health-mobile/issues/209) | Prompt user to add new card if current card fails | Jan 5 |

**Total Open Issues:** 25

---

## Key Highlights This Week

### Practice Interviews
- **Rate limiting improvements:** Per-user rate limits implemented across all three repositories to prevent abuse
- **Analytics enhancements:** Comprehensive PostHog tracking events added
- **Ideal answer system:** Migrated to talkwise-promptsmith for ideal answer generation
- **Logging improvements:** Switched to Pino for cleaner, more efficient logs

### Therapist Genie
- **Critical bug fixes:** Session notes, telehealth, ICD-10 codes, and progress notes for recurring appointments
- **New feature:** Client signature option for treatment plans
- **Performance fix:** Resolved app freezing while typing in progress notes
- **Backend reliability:** Improved cron job for child appointment creation

### VitaBoom
- **Commission management overhaul:** New filters, CSV export, shift+select, and hide commissions toggle
- **W9 compliance:** Added W9 upload popup with commission opt-out option
- **Clinic management:** Redesigned clinics table with archive/restore functionality
- **QuickBooks integration:** Improved bill handling (create bill without automatic payment)

### Next Health
- No development activity this week. 25 open issues remain in the backlog.

---

*Generated by Ruk | Fractal Labs*

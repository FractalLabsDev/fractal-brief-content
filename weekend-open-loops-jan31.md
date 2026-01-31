# Weekend Open Loops: Priority Brief

**Date:** January 31, 2026 (Friday evening)
**Context:** Preparing for Monday sync with team and clients
**Goal:** Identify high-leverage "ONE thing" work that moves the needle 10x

---

## CRITICAL: Monday Morning Blockers

### 1. Promptsmith PRs - Roll to Staging Monday

**Status:** PRs ready for final review, aim for staging Monday, prod ASAP

Max completed the Wordware-to-Promptsmith migration. This is the new AI prompt management service that replaces Wordware for Practice Interviews.

**PRs to merge:**
- [talkwise-promptsmith #1](https://github.com/FractalLabsDev/talkwise-promptsmith/pull/1) - Core service (v0.1)
- [fractal-os-web #361](https://github.com/FractalLabsDev/fractal-os-web/pull/361) - Promptsmith UI
- [practice-interviews-backend #94](https://github.com/FractalLabsDev/practice-interviews-backend/pull/94) - New `/idealAnswerV2` endpoint
- [practice-interviews-web #404](https://github.com/FractalLabsDev/practice-interviews-web/pull/404) - Updated UI to use new endpoint
- [talkwise-sentinel #34](https://github.com/FractalLabsDev/talkwise-sentinel/pull/34) - Permissions seeder + db:copy-prod

**Looms from Max:**
- Short demo: https://www.loom.com/share/edd9335658054742b008dc9ae941dac4
- Tech details (2x30min): https://www.loom.com/share/ca19ec3276e14e73b939b9b130909a1a and https://www.loom.com/share/1ce1885375df4cfc923013108978c8ce

**Owner:** Austin to review, Max to support deployment

---

### 2. Fractal Meet Backend Rebuild

**Status:** Austin is rebuilding - targeting Monday completion

Synthesizing Recall.ai, Google Meet, and Fractal Meet transcripts into unified UI. Matt needs meeting access for delivery manager prep notes.

**Related context:** Matt asked for Fractal OS permissions to view meetings - Austin mentioned rebuilding the backend first.

**Owner:** Austin

---

### 3. E2E Tests vs QA Agent Discussion

**Status:** Blocked - awaiting resolution before scheduling QA runs

Thread in #fractal-os with Austin and Max about whether to pursue:
- E2E tests (Playwright) - traditional automation
- QA agent approach (Ruk-powered) - AI clicks through and verifies

Matt's instruction: "Let's wait until that's resolved before scheduling the runs"

**What's ready:**
- QA component Phase 1 complete and tested
- All core tests pass against Practice Interviews production
- Tool ready at `PROJECTS/delivery-manager-agent/tools/qa/qa-check.js`

**Owner:** Austin/Max to decide direction, then Matt/Ruk to implement

---

## HIGH PRIORITY: This Weekend

### 4. Practice Interviews Bug Fixes (PR #403)

**Status:** PR ready for review - fixes 4 bugs from Matt's Loom walkthrough

[PR #403](https://github.com/FractalLabsDev/practice-interviews-web/pull/403) fixes:
- #399: Field/role not populating in summary banner
- #400: Feedback page shows 'NA' instead of loading spinner
- #401: Two loading spinners in body language section
- #402: Subscription modal card sizes and hidden button

**Note:** Bug #398 (login button) is in `practice-interviews-landing` repo, not `practice-interviews-web`

**Owner:** Max to review, Matt to approve

---

### 5. ruk-observer Service (Delivery Manager Foundation)

**Status:** Spec complete, ready to build

This is the 10x infrastructure work - building the action logging service that enables learning loops and fitness metrics.

**Docs:**
- [RUK-OBSERVER-SPEC.md](./RUK-OBSERVER-SPEC.md) - Full technical specification
- [WEEKEND-PLAN.md](./WEEKEND-PLAN.md) - Implementation schedule

**Weekend targets:**
- Saturday: Fractal OS schema extension + ruk-observer core
- Sunday: Deploy + client library + retrofit daily-nudge.js

**Related brief:** [Continuous Intelligence Paradigm](https://brief.fractallabs.dev/continuous-intelligence-paradigm)

**Owner:** Austin/Ruk

---

### 6. PI Open Issues (10 open)

**Status:** Mixed - some fixed in PR #403, others need attention

| # | Title | Status |
|---|-------|--------|
| 402 | Subscription modal UX | Fixed in PR #403 |
| 401 | Two spinners in body language | Fixed in PR #403 |
| 400 | 'NA' instead of loading spinner | Fixed in PR #403 |
| 399 | Field/role not populating | Fixed in PR #403 |
| 398 | Login button navigation | Different repo |
| 393 | Declare z-indices in theme | Open |
| 387 | Missing job descriptions | Open |
| 386 | Hypothetical questions missing scores | Open |
| 384 | Side panel responsiveness | Open |
| 383 | Analytics colors | Open |

**Owner:** Team triage needed

---

## MEDIUM PRIORITY: Next Week

### 7. Therapist Genie - Friday Deadline

**Status:** In progress, deadline next Friday

From 1/29 meeting: Rigo has something due by next Friday. Not fully specified what, but Matt said "don't stress too much about it right now."

**Owner:** Rigo, Matt oversight

---

### 8. Vitaboom Stale PR

**Status:** Matt asked Max about it

[vitaboom-backend #82](https://github.com/FractalLabsDev/vitaboom-backend/pull/82) - "has been hanging out there for a while"

Matt asked if it's still relevant/required.

**Owner:** Max to respond

---

### 9. Vitaboom Payments Migration (Potential)

**Status:** Exploratory - waiting for confirmation

Austin spoke with Sean from Payments Toolbox (Empire connection). Key insight: they have certification to request unredacted CC details from other providers - could enable migration from Bankful with zero churn.

Sean will confirm on their next team call. Don't share with Justin yet.

**Owner:** Austin to follow up with Sean

---

### 10. CPO Roadmap for Practice Interviews

**Status:** Waiting for data from new analytics tracking

PR #397 added tracking for:
- Subscription page events (78% drop-off investigation)
- Interview sessions (95% drop-off investigation)
- Onboarding flow
- Activation constants

Need ~1 week of data before diagnosing funnels and forming experiment hypotheses.

**Owner:** Matt/Ruk CPO role

---

## COMPLETED/DEPLOYED THIS WEEK

- Scheduler infrastructure deployed (daily trigger, delivery manager nudges)
- QA component Phase 1 built and tested
- Agentic velocity analysis completed (274 Ruk commits, 10x repo creation rate)
- Bug fixes PR #403 created from Matt's Loom
- Code reviews for Max's Promptsmith PRs

---

## Infrastructure Notes

### Scheduler (Just Deployed)
Three schedules now live via talkwise-infra:
- Daily trigger for general scheduled messages
- Delivery manager daily nudge
- Delivery manager weekly digest

### Semantic Memory
- 11,061 meeting chunks indexed
- 12,292 log chunks indexed
- 2,461 entities, 2,854 relationships in knowledge graph

---

## The 10x Question

If you could only do ONE thing this weekend that moves the needle most for Monday:

**For Austin:** Get Promptsmith PRs deployed to staging so the team can validate before prod.

**For Ruk:** Build ruk-observer core so we have action logging infrastructure for the delivery manager evolution.

---

*Generated by Ruk | January 31, 2026*

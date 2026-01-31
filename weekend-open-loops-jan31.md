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

### 2. Fractal OS as Project Source of Truth (talkwise-warden)

**Status:** Needs implementation before Matt continues Monday

Currently the delivery-manager-agent uses a static `projects.json` with just repo names. Need to extend talkwise-warden Project model to include:

```
Project (existing)
├── repositories[] (existing)
├── slack_channels: { internal, shared }
├── meeting_pattern: "practice-interviews"  // for semantic memory
├── data_sources: { posthog_project, stripe_account }
└── delivery_config: { stale_pr_days, stale_issue_days, digest_day }
```

**Implementation:**
1. Add fields to talkwise-warden Project model
2. Create `TOOLS/fractal-os/project-config.js` to fetch full config
3. Update delivery-manager tools to use API instead of static JSON

**Why this matters for Matt:** Single source of truth he can update via Fractal OS GUI instead of editing JSON files.

**Owner:** Austin/Ruk

---

### 3. ruk-observer Service (Action Logging)

**Status:** Spec complete, ready to build

The action logging service that enables learning loops and fitness metrics.

**Docs:**
- `PROJECTS/delivery-manager-agent/RUK-OBSERVER-SPEC.md` - Full technical specification
- `PROJECTS/delivery-manager-agent/WEEKEND-PLAN.md` - Implementation schedule

**Weekend targets:**
- Saturday: Fractal OS schema extension + ruk-observer core
- Sunday: Deploy + client library + retrofit daily-nudge.js

**Related brief:** [Continuous Intelligence Paradigm](https://brief.fractallabs.dev/continuous-intelligence-paradigm)

**Owner:** Austin/Ruk

---

### 4. talkwise-chronicler Rebuild (Meeting Intelligence Backend)

**Status:** Austin is rebuilding - targeting Monday completion

**Clarification:**
- **Fractal Meet** = a video conferencing platform (one source of recordings)
- **talkwise-chronicler** = the backend service that stores and synthesizes recordings/transcripts from ALL platforms (Recall.ai, Google Meet, Fractal Meet)

The chronicler rebuild will unify these three transcript sources into one UI. Matt needs meeting access for delivery manager prep notes.

**Owner:** Austin

---

### 5. E2E Tests vs QA Agent Discussion

**Status:** Blocked - awaiting resolution before scheduling QA runs

Thread in #fractal-os with Austin and Max about whether to pursue:
- E2E tests (Playwright) - traditional automation
- QA agent approach (Ruk-powered) - AI clicks through and verifies

Matt's instruction: "Let's wait until that's resolved before scheduling the runs"

**What's ready:**
- QA component Phase 1 complete and tested
- All core tests pass against Practice Interviews production
- Tool ready at `PROJECTS/delivery-manager-agent/tools/qa/qa-check.js`

**Owner:** Austin/Max to decide direction

---

### 6. Vitaboom Payments Migration (Revenue Opportunity)

**Status:** Exploratory - waiting for confirmation from Sean

Austin spoke with Sean from Payments Toolbox (Empire connection). Key insight: they have certification to request unredacted CC details from other providers - could enable migration from Bankful with **zero subscriber churn**.

Sean will confirm on their next team call. Don't share with Justin yet until confirmed.

**This creates a new revenue stream** - Payments Toolbox partnership/referral opportunity.

**Owner:** Austin to follow up with Sean

---

## COMPLETED/DEPLOYED THIS WEEK

- Scheduler infrastructure deployed (daily trigger, delivery manager nudges)
- QA component Phase 1 built and tested
- Agentic velocity analysis completed (274 Ruk commits, 10x repo creation rate)
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

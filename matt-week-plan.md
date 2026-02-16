# Week of Feb 16 — Project Focus

## Vitaboom

**Goal for this week:** Complete W9/Commission Opt-Out Pop-up (Milestone #56)

The W9 opt-out pop-up is the active milestone — a mandatory modal appearing on every login for practitioners without a W9 on file. Lets them upload W9 or opt out of commissions (double confirmation required). This was confirmed in the Jan 27 and Feb 3 dev syncs as your team's focus.

**This week you should ship:**
- W9 upload interface within the pop-up modal
- Opt-out flow with double confirmation
- Login-gating logic (popup appears until resolved)
- Link to blank W9 form in the popup

**Blockers/Dependencies:** None identified — backend endpoints exist, this is frontend modal work.

**What's stale (consider closing/descoping):**
- Milestone #14: "Automatically archive old discounts" (target: Nov 20, 3+ months overdue) — Is this still relevant?

---

## Practice Interviews

**Goal for this week:** Recruiter Dashboard MVP (Milestone #28, target: March 1)

Jeff has a presentation March 13. You need a demoable recruiter dashboard by ~March 1 showing candidate list, effort-based signals, and cohort filtering.

**This week you should ship:**
- Recruiter dashboard data model (candidates, cohorts, effort metrics)
- Basic candidate list view with sortable columns
- Effort signal indicators (sessions completed, time spent)

**Blockers/Dependencies:**
- Need clarity on which effort metrics Jeff wants to showcase
- Design mockups for the dashboard (do they exist?)

**Completed recently:**
- Onboarding checklist (Milestone #18)
- Licensing model (Milestone #25)

---

## Therapist Genie

**Goal for this week:** Client file upload + Kylie access + KAP form

**From Slack and roadmap:**
1. **Client file upload API** — PR #64 has been open 21 days. What's blocking merge?
2. **Give Kylie access** — 10 days pending. What's the hold-up?
3. **KAP form updates** — Needs definition. Is there a spec?

**Cancellation & Refund Epic** is planned but no milestones are in-progress. Should this be prioritized?

**Recent activity:**
- Calendar status indicators, payment badges, and Outlook filtering shipped to dev (PR from last week)
- Azure Functions setup for background jobs complete

---

## What needs your attention

| Project | Action | Why |
|---------|--------|-----|
| VB | Close or rescope "archive old discounts" milestone | 3 months overdue |
| PI | Confirm recruiter dashboard design/spec | Jeff presentation depends on it |
| TG | Unblock PR #64 and Kylie access | Both >2 weeks stale |


# Practice Interviews: Roadmap Overview
*February 2026 — Compiled for Matt*

---

## Current State Summary

Practice Interviews is in a pivotal moment: recently stabilized from the Wordware reliability issues, with several foundational features shipped (filler word detection, analytics upgrade, compliance check), and now pivoting toward **two major growth vectors** that Jeff has identified: **college/university partnerships** and **enterprise licensing for large internship programs**.

---

## Epics & Priorities (from Fractal OS)

### Active / In-Progress
| Epic | Status | Target | Notes |
|------|--------|--------|-------|
| **B2B Partner Features (E-15)** | in-progress | Feb 28 | UTM tracking done, compliance check done. **Licensing model next.** |
| **Filler Features (E-16)** | in-progress | Jan 30 | Pace of speech, um/filler counter, answer relevance — all shipped. Onboarding checklist still in-progress. |
| **Replace Wordware (E-18)** | planned | Feb 28 | Critical technical debt. Max prototyping Talk Oracle. Austin secured prompt export from Wordware. |

### Upcoming / Planned
| Epic | Status | Target | Notes |
|------|--------|--------|-------|
| **Rigid Practice Sessions (E-5)** | planned | Feb 27 | Previously discussed as high priority after filler features ship |
| **Story Bank (E-10)** | planned | Feb 27 | |
| **Question Bank (E-9)** | planned | Mar 17 | Max proposed super admin tooling to manage questions |
| **Retry with Incremental Progress (E-6)** | planned | Apr 16 | "Try again with improvements" |
| **Interview Prep (E-11)** | planned | Apr 30 | |

---

## New Strategic Initiatives (from Slack Discussions)

### 1. College/University Course Integration
**Target:** August 1, 2026 (build incrementally)
**Source:** Jeff's detailed spec in #practiceinterviews-shared (Feb 1-2)

**The Ask:**
- System-owned interview curriculum (Jeff authors, professors can't edit)
- Course sections with scoped professor access (professor sees only their students)
- Join code enrollment (like Google Classroom)
- Professor analytics dashboard (filtered view of existing data)

**Non-Goals:** No LMS integration, no grading UI, no peer review, no file uploads

**Build Order:**
1. Curriculum schema + rendering
2. CourseSection + Enrollment
3. Completion tracking per week
4. Professor dashboard

**Brief created:** https://brief.fractallabs.dev/pi-college-curriculum-integration

---

### 2. Enterprise Licensing Model
**Source:** Jeff's spec in #practiceinterviews-shared (Feb 1)

**Context:** Jeff has a new ICP — large companies with big internship programs. He's presenting at a conference on March 13th (Designing Candidate Experience for Better Hiring Outcomes).

**Key Principles:**
- Recruiter-first: Under 30 seconds to identify priority candidates
- Effort-based signals: Volume, recency, consistency — NOT subjective quality scores
- Transparent prioritization: No black-box ranking
- Candidates practice without friction or awareness of monitoring

**Technical Requirements:**
- Company license → cohorts → recruiter admins
- Recruiter dashboard: candidate list, last activity, questions practiced, trend indicator
- Sort/filter, no complex configuration

---

## Open GitHub Issues

| # | Title | Status |
|---|-------|--------|
| 10 | Update /next-question to include common questions in mix, cap at 2 | Open |
| 9 | Update Interview Style UI to "Mix of All Three" | Open |
| 8 | Home screen feedback shows intermittent zeros/blank states | Open |
| 7 | Add 9 new common questions + randomize order | Open |
| 6 | Implement scoring for common questions | Open |
| 5 | Enable live audio streaming to reduce feedback latency | Open |
| 4 | Fix hypothetical question scoring (not appearing on first load) | Open |
| 3 | Implement API key rotation via AWS Secrets Manager | Open |

---

## Key Bugs / Technical Debt

1. **Hypothetical Scoring Bug (#4):** Different score extraction functions (behavioral vs hypothetical). Root cause identified — straightforward fix.

2. **Home Screen Loading Bug (#8):** Race condition in data loading. Not data loss, just UI coordination. Needs frontend fix in useHomeData.ts.

3. **Wordware Dependency:** Major strategic risk. Frequent breaking changes, no advance notice, no status page. Austin secured full prompt export. Max prototyping replacement.

4. **Live Audio Streaming (#5):** PI already has WebSocket infrastructure built (deepgram.service.ts) — just not wired up. Would enable feedback before video upload completes.

---

## Suggested Prioritization for Next 2-3 Months

### February (Now)
1. **Ship the quick wins** — Issues #4, #7, #9, #10 are all small fixes from Jeff's recent Looms
2. **Complete onboarding checklist** (M-18, in-progress)
3. **Licensing model MVP (M-25)** — supports Jeff's March 13 presentation
4. **Fix home screen loading bug (#8)**

### March
1. **Wordware replacement sprint** — Critical path before they cause another outage
2. **Rigid practice sessions (E-5)** — Long-requested feature
3. **Live audio streaming (#5)** — UX win, infrastructure exists

### April–August (Background)
1. **College integration** — Build incrementally toward August 1
2. **Story bank + Question bank** — Admin tooling for Jeff

---

## Questions for Discussion

1. **Licensing vs College:** Which takes priority if both compete for the same sprint? Licensing supports March 13 presentation; College has August deadline but is bigger scope.

2. **Wordware timeline:** How urgent is the migration? Current workaround (prompt export) gives us some runway, but another outage would be painful.

3. **Max's capacity:** He's carrying most of the backend work. Is there room to bring in another dev for any of this?

---

*Compiled from Fractal OS, #practiceinterviews-shared, semantic memory, and GitHub. Feb 2, 2026.*

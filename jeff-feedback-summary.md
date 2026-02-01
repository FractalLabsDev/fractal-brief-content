# Jeff's Feedback Threads Summary

*Generated 2026-02-01 from #practiceinterviews-shared*

Jeff has raised **7 distinct issues** across multiple threads. Here's the comprehensive status of each:

---

## 1. Hypothetical Scoring Bug

**Thread:** Scores not appearing on first load for hypothetical questions
**GitHub Issue:** [#4](https://github.com/FractalLabsDev/practice-interviews/issues/4) — OPEN

**Root Cause Identified:**
- Race condition in score delivery
- Different score extraction functions between behavioral/hypothetical
- The regex-based `extractScores` for hypothetical is more fragile than `extractWordwareScores` used for behavioral

**Recommended Fix:** Change hypothetical to use `extractWordwareScores` (same as behavioral)

**Action Taken:** ✅ Issue created and assigned to Max, root cause analysis complete

---

## 2. Video Processing / Feedback Latency

**Thread:** Deprioritize video processing to speed up feedback delivery
**GitHub Issue:** [#5](https://github.com/FractalLabsDev/practice-interviews/issues/5) — OPEN

**Key Insight:** PI already has **unused WebSocket live transcription infrastructure** in the codebase. The Deepgram live streaming setup exists but isn't wired to the workflow.

**Recommended Fix:**
1. Stream audio during recording
2. Trigger feedback on stream end
3. Upload video in parallel (no longer blocking)

**Action Taken:** ✅ Issue created, detailed technical analysis provided showing existing infrastructure

---

## 3. Common Question Scoring

**Thread:** Implement scoring mechanism for common questions (currently unscored)
**GitHub Issue:** [#6](https://github.com/FractalLabsDev/practice-interviews/issues/6) — OPEN

**Scoring Framework Defined:**
| Criterion | Weight |
|-----------|--------|
| Role-related | 25% |
| Rule of three | 25% |
| Connection to items 1 & 2 | 25% |
| Time (30s-60s sweet spot) | 25% |

**Action Taken:** ✅ Issue created, awaiting prioritization

---

## 4. Common Questions Pool Expansion

**Thread:** Add 9 new questions to the common questions pool and randomize order
**GitHub Issue:** [#7](https://github.com/FractalLabsDev/practice-interviews/issues/7) — OPEN

**Current State:** Only 4 common questions in fixed order

**Jeff's Request:** Add these 9 new questions:
- Why do you want this position?
- What interests you most about this role?
- What do you bring to this team?
- What are you looking for in your next role?
- How would you describe your working style?
- What are you known for professionally?
- What motivates you at work?
- What are you looking for in a manager?
- What attracted you to apply?

**Plus:** Randomize order on every practice session

**Action Taken:** ✅ Issue created with full question list

---

## 5. Question Bank Admin Interface

**Thread:** Allow super admins to manage questions
**GitHub Issue:** Not yet created

**Request:** Add admin page for:
- Archive old questions
- Create new questions
- Modify existing questions
- View stats on question distribution and skip rates

**Action Taken:** ⏳ Max confirmed this is doable, needs priority agreement

---

## 6. Interview Style Update + Question Ordering

**Thread:** Update 'Mix' option and cap consecutive same-type questions
**GitHub Issues:**
- [#9](https://github.com/FractalLabsDev/practice-interviews/issues/9) — UI update ("Mix of all three" wording) — OPEN
- [#10](https://github.com/FractalLabsDev/practice-interviews/issues/10) — Backend logic (include common, cap at 2 consecutive) — OPEN

**Current Behavior:** "Mix" clusters question types together (3-4 of same type in a row)

**Desired Behavior:**
- Update wording to "Mix of all three" (include common)
- Never more than 2 consecutive questions of same type
- Cycle through: common → behavioral → hypothetical → repeat

**Action Taken:** ✅ Both tickets created per Max's request

---

## 7. Home Screen Feedback Regression

**Thread:** Feedback counts intermittently showing zero/missing, then reappearing on refresh
**GitHub Issue:** [#8](https://github.com/FractalLabsDev/practice-interviews/issues/8) — OPEN

**Root Cause Identified:**
1. Race condition in home screen data loading
2. Multiple independent data sources load at different speeds with no synchronization
3. Analytics mutation runs once on mount with no retry
4. RecentQuestions shows blank during loading instead of cached data

**Why it's intermittent:** Components render before data arrives. Works on second load because it reads from database (cached), not live SSE stream.

**Recommended Fix:**
1. Coordinate data loading sequence in `useHomeData.ts`
2. Show cached data during loading (instead of blanks)
3. Add proper retry logic for failed fetches

**Action Taken:** ✅ Full technical analysis provided, stress testing script created (`e2e/stress-test.js`)

---

## Summary Table

| # | Issue | GitHub | Status | Action Taken |
|---|-------|--------|--------|--------------|
| 1 | Hypothetical Scoring | [#4](https://github.com/FractalLabsDev/practice-interviews/issues/4) | OPEN | Root cause found, fix proposed |
| 2 | Feedback Latency | [#5](https://github.com/FractalLabsDev/practice-interviews/issues/5) | OPEN | Technical analysis complete |
| 3 | Common Q Scoring | [#6](https://github.com/FractalLabsDev/practice-interviews/issues/6) | OPEN | Framework defined |
| 4 | Common Q Expansion | [#7](https://github.com/FractalLabsDev/practice-interviews/issues/7) | OPEN | 9 questions specified |
| 5 | Question Bank Admin | — | Not tracked | Needs priority discussion |
| 6 | Mix + Ordering | [#9](https://github.com/FractalLabsDev/practice-interviews/issues/9), [#10](https://github.com/FractalLabsDev/practice-interviews/issues/10) | OPEN | Both tickets created |
| 7 | Home Screen Regression | [#8](https://github.com/FractalLabsDev/practice-interviews/issues/8) | OPEN | Root cause found, stress test built |

---

## Overall Status

- **7 issues identified** by Jeff
- **8 GitHub issues** created (one item split into 2 tickets)
- **All issues currently OPEN** — awaiting prioritization/assignment
- **Root cause analysis complete** for the two bugs (#4 hypothetical scoring, #8 home screen)
- **1 item** (Question Bank Admin) not yet tracked in GitHub — needs priority discussion

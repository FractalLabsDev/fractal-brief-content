# Practice Interviews: 5 Technical Improvements

**Source:** PI Sync meeting, February 4, 2026  
**Proposed by:** Max Donchenko  
**Context:** During discussion with Matt about potential app rebuild vs. incremental improvements

---

## Overview

Max proposed 5 improvements that can be implemented **without rewriting the PI backend**, aimed at reducing technical friction and enabling faster development with Ruk/Claude Code.

---

## The 5 Improvements

### 1. Orphaned Code Cleanup with `knip`

**What:** Use the `knip` library (same tool Serhii uses on Vitaboom) to scan the codebase and identify unused/orphaned code that pollutes the repository and wastes developer reading time.

| Metric | Score | Rationale |
|--------|-------|-----------|
| **Effort** | 2/10 | One evening. Install the tool, run it, review output, delete confirmed dead code. |
| **Benefit** | 4/10 | Cleaner codebase, less cognitive load, fewer false trails for AI agents. Modest but real improvement. |

---

### 2. Storybook for UI Component Alignment

**What:** Set up a basic Storybook with aligned UI components. Currently PI has multiple button variants, multiple styling patterns. Storybook would establish a single source of truth for component usage.

| Metric | Score | Rationale |
|--------|-------|-----------|
| **Effort** | 4/10 | A couple evenings. Max mentioned he prototyped something similar with Gemini Flash + Antigravity in "one pair of weekends" with light/dark theme support. |
| **Benefit** | 6/10 | Consistency across UI, components get reused instead of recreated, AI agents can reference Storybook for correct patterns. Compounds over time. |

---

### 3. More Compact Logging

**What:** Make logs more compact and readable. Current state: errors produce 50-70 unreadable lines; AI generation chunks log verbatim; concurrent users make logs unintelligible.

| Metric | Score | Rationale |
|--------|-------|-----------|
| **Effort** | 3/10 | One evening to a couple evenings. Requires reviewing current logging patterns and implementing structured/condensed output. |
| **Benefit** | 5/10 | Debugging becomes possible. Currently Max described logs as "huge and unreadable" especially during AI generation. With only 7-day Papertrail retention, this is critical. |

---

### 4. Single Main Branch Workflow

**What:** Eliminate the dev branch, use only main (like Vitaboom does). Production deployment becomes manual rather than automatic on PR merge.

| Metric | Score | Rationale |
|--------|-------|-----------|
| **Effort** | 2/10 | Quick process change. Scott and Serhii already deploy manually on other projects. |
| **Benefit** | 4/10 | Simplifies branching strategy, reduces confusion for non-technical contributors and AI agents about which branch to target. |

---

### 5. Automated End-to-End Tests on PR

**What:** Configure GitHub to trigger E2E tests automatically whenever changes occur in frontend or backend repos. Currently E2E tests exist but don't run on every PR, so Ruk can't instantly verify its changes.

| Metric | Score | Rationale |
|--------|-------|-----------|
| **Effort** | 3/10 | "Couple of evenings" per Max. Need to figure out how GitHub can connect three repos and trigger the E2E repo on changes to the other two. |
| **Benefit** | 8/10 | **Highest impact.** This is what enables the "Lovable-style" workflow Max described: drop commands, Ruk implements, E2E verifies, merge if green. Without this, regressions are "a daily thing." |

---

## Summary Matrix

| # | Improvement | Effort | Benefit | Priority |
|---|-------------|--------|---------|----------|
| 5 | E2E Tests on PR | 3/10 | 8/10 | **Highest** |
| 2 | Storybook | 4/10 | 6/10 | High |
| 3 | Compact Logging | 3/10 | 5/10 | Medium |
| 1 | Orphan Code Cleanup | 2/10 | 4/10 | Medium |
| 4 | Single Main Branch | 2/10 | 4/10 | Medium |

---

## Key Quote

> "If we complete these things, I'm pretty sure we'll have less regression. I just remember from Jeff in looms, he just mentioned a couple of times 'Oh guys, we have a regression' and with AI, with Ruk and without tests, these regressions will be a daily thing - we'll fail constantly without tests."
>
> — Max Donchenko

---

## Context: This vs. Full Rebuild

This list was presented as **Plan B** - the incremental path if the team decides against a full green-fields rebuild. Matt's concern: "Are we just continuing to try and fit this thing in that doesn't actually make sense?"

Max's position: The PI backend itself "isn't that bad" - the worst pain comes from the Talkwise backend connection. These 5 improvements address friction without requiring architectural overhaul.

**Next step from the meeting:** Matt to create Loom for Austin to get his take on rebuild vs. incremental approach.

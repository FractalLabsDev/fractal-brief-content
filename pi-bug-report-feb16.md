# Practice Interviews Bug Report Analysis
**Source:** Jeff's Loom - Weekly Bug Report  
**Analyzed:** February 16, 2026

---

## Critical Bugs (Blocking Core Functionality)

| # | Bug | Description | Duration |
|---|-----|-------------|----------|
| 1 | **Feedback Not Processing** | Video and feedback do not process automatically. Users must hit refresh button to get responses. "All feedback is not triggering a response now." | Ongoing |
| 2 | **Scoring Display Shows All Zeros** | Feedback shows "good score" but individual metrics display as 0, 0, 0 or all NAs. Example: 33% score with 0, 0, 0 breakdown; 54% score with 0, 0, 0, 0, 0 breakdown. | Ongoing |
| 3 | **Next Video Button Broken** | Interview Academy "next video" button has been broken for weeks. Videos auto-play unexpectedly. | Weeks |

---

## Data Integrity Issues

| # | Issue | Description | Duration |
|---|-------|-------------|----------|
| 4 | **Missing Job Descriptions** | Accountant role (first in list) has no job description. Multiple roles affected. | 3+ weeks |
| 5 | **Common Questions Mislabeled** | Common questions not connecting to job descriptions. All showing as "Interview Academy" or "Other" instead of the actual job/role. | Ongoing |
| 6 | **User Answer History Wrong Labels** | Every user Jeff checked shows all answers labeled as "Interview Academy" regardless of actual content. No user found with proper job labels yet. | Ongoing |
| 7 | **Mix of All Three Not Working** | When selecting "Mix of All Three" interview style, common questions (strengths, etc.) rarely appear. Jeff ran 20-30 questions and "never saw a common question" most times. | Since Tue/Wed |

---

## Analytics Dashboard Issues

| # | Issue | Description | Severity |
|---|-------|-------------|----------|
| 8 | **Interview Readiness Non-Functional** | "Interview Readiness" section doesn't do anything when clicked. | Medium |
| 9 | **Common Questions Show as Hypothetical** | In analytics, common questions are incorrectly categorized as hypothetical. | Medium |
| 10 | **Power Users Limited to 3** | Power users section only shows top 3. Jeff wants a scrollable/flowing list to see more. | Enhancement |
| 11 | **Charts Coming Soon** | Two dashboard charts still showing "Coming Soon" placeholders. | Low |

---

## UX/Usability Issues

| # | Issue | Description | Priority |
|---|-------|-------------|----------|
| 12 | **Default Sort Order Wrong** | Client data sorts A-Z by default, putting zeros at top. Should be Z-A to show most active users first. | Medium |
| 13 | **Academy Navigation Inconsistent** | Navigation state not persisting as expected when jumping around in Academy. | Medium |
| 14 | **Auto-play Video Behavior** | Videos start playing unexpectedly when navigating in Interview Academy. | Low |

---

## Severity Summary

| Severity | Count | Bugs |
|----------|-------|------|
| **P0 - Critical** | 3 | Feedback not processing, Scores showing zeros, Next video broken |
| **P1 - High** | 4 | Missing job descriptions, Common questions mislabeled, User history wrong labels, Mix of Three not working |
| **P2 - Medium** | 4 | Interview Readiness, Analytics categorization, Sort order, Navigation state |
| **P3 - Low** | 3 | Power users limit, Coming Soon placeholders, Auto-play |

---

## Patterns Observed

**Data Labeling Cascade:** The labeling issues (everything showing as "Interview Academy", common questions categorized as hypothetical, missing job descriptions) appear interconnected. Something in how job/question type metadata is being stored or retrieved may be the root cause.

**Feedback Pipeline:** The feedback not processing automatically + scores showing zeros suggests the feedback generation pipeline has issues beyond just display.

---

## Recommended Focus (Given Rebuild Context)

If rebuilding is imminent, focus stabilization on:

1. **Feedback not processing** - Blocks core user value
2. **Scores showing zeros** - Undermines user trust in the product
3. **Job description labeling cascade** - Appears to affect multiple areas, worth investigating root cause even if fix waits for rebuild

The analytics dashboard issues and UX polish items can likely be addressed in the new version architecture.

---

## Positive Notes

Jeff called out these as working well:
- Dashboard top section (New Users, Answers Provided, Activated Users)
- Previous/Next navigation and "current week" button
- Basic structure of admin/client data views

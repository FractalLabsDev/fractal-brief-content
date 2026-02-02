# College/University Course Integration

**Target:** August 1st (not urgent — build incrementally alongside other priorities)

**Goal:** Add a system-owned interview curriculum with scoped professor access and lightweight analytics, without becoming an LMS.

---

## Feature 1: Canonical Curriculum Engine (system-owned)

**What it is:**
- A single, read-only curriculum created and managed internally (by Jeff)
- Organized into modules and weeks
- Each week has a predefined assignment type

**Key constraints:**
- Professors cannot edit curriculum
- No custom authoring at MVP
- One canonical version

**Technical notes:**
- Curriculum → Modules → Weeks
- Weeks reference assignment configuration only
- Used for rendering and completion tracking

---

## Feature 2: Course Sections + Scoped Access

**What it is:**
- A way to assign professors to a subset of students within an institution
- Prevents professors from seeing all users at a school

**Core objects:**
- CourseSection (curriculum + institution + professor)
- Enrollment (student ↔ course section)

**Access rules:**
- Students see only their own progress
- Professors see only students enrolled in their course section
- No cross-section visibility

**Enrollment flow:**
- Join codes — professor shares code, students self-enroll
- Familiar pattern (like Google Classroom)

---

## Feature 3: Assignment Artifacts (structured, minimal)

**What it is:**
- Assignments are structured uses of existing Practice Interviews data
- No freeform file uploads at MVP

**Supported artifacts:**
- Recorded answers (existing infra)
- Structured text responses

**Important rules:**
- Students can re-record answers
- Professors see submission history
- Completion is based on latest valid submission
- No grading or feedback UI at MVP

---

## Feature 4: Professor Analytics Dashboard (scoped slice)

**What it is:**
- A lightweight dashboard for professors showing trends and performance for *only their students*
- Not the full institutional admin dashboard

**Data scope:**
- Only users enrolled in that professor's course section
- Typically 20–50 students

**Example signals:**
- Completion rates by week
- Top performers (by score or improvement)
- Engagement trends (attempts, activity)
- Improvement deltas over time

**Key point:**
- All data already exists
- This is a filtered view, not new analytics logic

---

## Explicit Non-Goals (MVP)

- ❌ No curriculum editing by professors
- ❌ No grading or rubric UI
- ❌ No LMS integrations
- ❌ No peer review
- ❌ No PDF or file uploads
- ❌ No professor-written feedback inside the tool

---

## Build Order

1. Curriculum schema + rendering
2. CourseSection + Enrollment (with join codes)
3. Completion tracking per week
4. Professor dashboard (filtered analytics view)

---

## Clarifications (from Jeff)

| Question | Answer |
|----------|--------|
| Curriculum authoring | Jeff-owned, manually authored, no tooling needed |
| Enrollment flow | Professor-managed via join codes |
| Week timing | Sequential modules (not calendar weeks) — professors control pacing |

---

*Compiled from #practiceinterviews-shared discussion, February 2026*

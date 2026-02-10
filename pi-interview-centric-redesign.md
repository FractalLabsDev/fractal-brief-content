# Practice Interviews: Interview-Centric Redesign

**Version:** 1.0  
**Date:** February 10, 2026  
**Author:** Matt Lim, Ruk  
**Status:** Draft Specification

---

## Executive Summary

This specification outlines a fundamental redesign of Practice Interviews from a "practice tool" to an "interview preparation platform." The core shift: instead of abstract practice sessions, the user experience anchors around **real or target interviews** as first-class entities.

**Key Paradigm Shift:**
- **From:** "Get reps in at the gym" (abstract practice)
- **To:** "Get ready for THIS interview" (concrete, event-driven preparation)

---

## User Personas

### Persona A: Interview Now (Primary)
- Has a job interview scheduled
- Needs focused preparation for a specific company/role/date
- Highest urgency, highest conversion potential
- **Optimize the entire experience for this persona**

### Persona B: Actively Interviewing (Secondary)
- No specific interview scheduled, but actively job searching
- Wants to build interview readiness for future opportunities
- Frame as: "Set a target interview" — create a fictional but concrete goal
- Same experience structure, just not tied to a real scheduled date

---

## Core Data Model

### Interview Entity

```typescript
interface Interview {
  id: string;
  company: string;
  role: string;
  date: Date;
  location?: string;  // Address, "Remote", etc.
  stages: InterviewStage[];
  status: 'upcoming' | 'completed' | 'archived';
  interviewPlan: InterviewPlan;
  outcome?: 'got_job' | 'rejected' | 'ghosted' | 'pending';
  debrief?: InterviewDebrief;
  isTarget: boolean;  // true = Persona B mock interview
}

interface InterviewStage {
  type: string;  // "Phone Screen", "Technical", "Hiring Manager", "Panel"
  date?: Date;
  notes?: string;
  practiceSessionIds: string[];
  readinessScore: number;  // 0-100
}

interface InterviewPlan {
  generatedAt: Date;
  items: PlanItem[];
}

interface PlanItem {
  day: number;  // Days before interview
  category: 'practice' | 'learn' | 'prepare';
  task: string;
  completed: boolean;
}

interface InterviewDebrief {
  submittedAt: Date;
  howItWent: 'great' | 'okay' | 'rough';
  whatWentWell: string;
  whatToImprove: string;
  followUpDate?: Date;
}
```

---

## Scoring Hierarchy

Four levels of scoring, each feeding into the next:

| Level | Scope | Example |
|-------|-------|---------|
| **Per-Question** | Individual answer feedback | "Your STAR structure was clear, but lacked quantifiable results" |
| **Per-Session** | Practice session aggregate | "Scored 78% on this behavioral practice session" |
| **Per-Interview Readiness** | Progress toward specific interview | "65% ready for your Google interview" |
| **Overall Candidate Score** | Aggregate across all activity | "Your overall interview readiness: 72%" |

### Readiness Score Components
- Practice sessions completed vs recommended
- Story bank coverage (tagged competencies)
- Learning modules completed
- Preparation checklist items done

---

## Three Pillars of Preparation

Every interview has three preparation dimensions:

### 1. Practice
- **Rigid Practice Sessions** — Structured sequences for specific interview stages
- **Question Bank** — Browsable database + user-submitted questions
- **Story Bank** — Resume-extracted + user-added examples by competency

### 2. Learn
- **Interview Academy** — Existing learning modules
- **Company Research** — AI-generated company insights
- **Role-Specific Prep** — Industry/function-specific guidance

### 3. Prepare
- **Logistics** — Directions, parking, what to bring
- **Attire** — Dress code recommendations
- **Day-Of Checklist** — Final preparation items
- **Interviewer Research** — LinkedIn insights on interviewers (if provided)

---

## Navigation Structure

### Primary Navigation (Sidebar)

```
┌──────────────────┐
│ 🏠 Home          │  → Interview cards dashboard (primary view)
│ 📋 Interviews    │  → All interviews (upcoming first, past collapsed)
│ 🎯 Practice      │  → Quick practice, question bank, story bank
│ 🎓 Academy       │  → Learning modules
│ 👤 Profile       │  → Settings, billing, analytics
└──────────────────┘
```

### Home Dashboard

The home page is interview-centric:

```
┌─────────────────────────────────────────────────────────────────┐
│  Your Interviews                                    [+ Add]     │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  ┌───────────────────────────────────────────────────────────┐  │
│  │ GOOGLE · Product Manager · Feb 17 (5 days)                │  │
│  │ ████████████░░░░░░░░ 65% ready                            │  │
│  │                                                           │  │
│  │ Next: Complete 2 behavioral practice sessions             │  │
│  │                                                           │  │
│  │ [Continue Prep]                    [View Plan]            │  │
│  └───────────────────────────────────────────────────────────┘  │
│                                                                 │
│  ┌───────────────────────────────────────────────────────────┐  │
│  │ META · Data Analyst · Mar 3 (19 days)                     │  │
│  │ ████░░░░░░░░░░░░░░░░ 25% ready                            │  │
│  │                                                           │  │
│  │ [Start Prep]                                              │  │
│  └───────────────────────────────────────────────────────────┘  │
│                                                                 │
├─────────────────────────────────────────────────────────────────┤
│  📊 Your Progress                                               │
│  Questions practiced: 47  |  Stories ready: 8  |  Score: 72%   │
│  Practice streak: 5 days                                        │
└─────────────────────────────────────────────────────────────────┘
```

### Empty State (No Interviews)

When user has no interviews:

```
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│              🎯 Get Interview Ready                             │
│                                                                 │
│       The best way to prepare is with a specific goal.         │
│                                                                 │
│    ┌─────────────────────────────────────────────────────┐      │
│    │  [+ Schedule an Interview]                          │      │
│    │                                                     │      │
│    │  Add your upcoming interview and we'll create       │      │
│    │  a personalized preparation plan for you.           │      │
│    └─────────────────────────────────────────────────────┘      │
│                                                                 │
│    ┌─────────────────────────────────────────────────────┐      │
│    │  [Set a Target Interview]                           │      │
│    │                                                     │      │
│    │  No interview yet? Set a goal to be ready for       │      │
│    │  your target role by a specific date.               │      │
│    └─────────────────────────────────────────────────────┘      │
│                                                                 │
│    ─────────────────── or ───────────────────                   │
│                                                                 │
│    [Browse Practice Questions]    [Start Learning]              │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

**Primary CTA:** Schedule an interview (real or target)  
**Secondary CTA:** Practice options for users who just want to browse

---

## Interview Detail View

When user clicks into a specific interview:

```
┌─────────────────────────────────────────────────────────────────┐
│  ← Back to Home                                                 │
│                                                                 │
│  GOOGLE · Product Manager Interview                             │
│  February 17, 2026 (5 days away)                               │
│  Mountain View, CA                                              │
│                                                                 │
│  ████████████░░░░░░░░ 65% ready                                │
│                                                                 │
├─────────────────────────────────────────────────────────────────┤
│  [Practice]        [Learn]          [Prepare]                   │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  (Tab content here — see below)                                 │
│                                                                 │
├─────────────────────────────────────────────────────────────────┤
│  📋 Your Interview Plan                              [Edit]     │
│                                                                 │
│  Today (Day 5)                                                  │
│  ☑ Research Google's PM interview process                       │
│  ☐ Complete 2 behavioral practice sessions                      │
│                                                                 │
│  Tomorrow (Day 4)                                               │
│  ☐ Review your leadership stories                               │
│  ☐ Practice "Tell me about yourself"                            │
│                                                                 │
│  [View Full Plan]                                               │
└─────────────────────────────────────────────────────────────────┘
```

### Practice Tab

```
┌─────────────────────────────────────────────────────────────────┐
│  Practice for Google PM Interview                               │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  Interview Stages                                               │
│                                                                 │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │ Phone Screen                              ████████ 80%  │    │
│  │ 3 sessions completed                                    │    │
│  │ [Practice More]                                         │    │
│  └─────────────────────────────────────────────────────────┘    │
│                                                                 │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │ Hiring Manager Round                      ████░░░░ 40%  │    │
│  │ 1 session completed, 2 recommended                      │    │
│  │ [Practice Now]                                          │    │
│  └─────────────────────────────────────────────────────────┘    │
│                                                                 │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │ Technical/Case Round                      ░░░░░░░░ 0%   │    │
│  │ Not started                                             │    │
│  │ [Start Practice]                                        │    │
│  └─────────────────────────────────────────────────────────┘    │
│                                                                 │
│  ─────────────────────────────────────────────────────────────  │
│                                                                 │
│  Quick Actions                                                  │
│  [Custom Practice Session]  [Browse Question Bank]              │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Learn Tab

```
┌─────────────────────────────────────────────────────────────────┐
│  Learn for Google PM Interview                                  │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  Company Research                                               │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │ 🔍 Google Company Overview                              │    │
│  │ AI-generated insights about Google's culture,           │    │
│  │ recent news, and PM role expectations                   │    │
│  │ [View Research]                                         │    │
│  └─────────────────────────────────────────────────────────┘    │
│                                                                 │
│  Recommended Academy Modules                                    │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │ ☑ Behavioral Interview Basics                           │    │
│  │ ☑ STAR Method Mastery                                   │    │
│  │ ☐ Product Sense Questions                     [Start]   │    │
│  │ ☐ Estimation & Metrics                        [Start]   │    │
│  └─────────────────────────────────────────────────────────┘    │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Prepare Tab

```
┌─────────────────────────────────────────────────────────────────┐
│  Prepare for Google PM Interview                                │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  Logistics                                                      │
│  ☐ Confirm interview location                                   │
│  ☐ Plan your route (allow extra time)                          │
│  ☐ Know where to park / how to get there                       │
│                                                                 │
│  Day Before                                                     │
│  ☐ Prepare your outfit                                         │
│  ☐ Print copies of your resume                                 │
│  ☐ Prepare questions to ask the interviewer                    │
│  ☐ Get a good night's sleep                                    │
│                                                                 │
│  Day Of                                                         │
│  ☐ Review your stories one more time                           │
│  ☐ Arrive 10-15 minutes early                                  │
│  ☐ Bring water and a snack                                     │
│  ☐ Put phone on silent                                         │
│                                                                 │
│  Interviewer Research                                           │
│  [+ Add interviewer LinkedIn]                                   │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## Interviews List Page

Accessible via "Interviews" in sidebar:

```
┌─────────────────────────────────────────────────────────────────┐
│  Your Interviews                                    [+ Add]     │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  Upcoming                                                       │
│                                                                 │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │ GOOGLE · PM · Feb 17        65% ready       [View]      │    │
│  └─────────────────────────────────────────────────────────┘    │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │ META · Data Analyst · Mar 3  25% ready      [View]      │    │
│  └─────────────────────────────────────────────────────────┘    │
│                                                                 │
│  ▼ Past Interviews (3)                                          │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │ STRIPE · PM · Jan 15         Rejected       [Review]    │    │
│  └─────────────────────────────────────────────────────────┘    │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │ NOTION · Designer · Dec 2    Got Offer!     [Review]    │    │
│  └─────────────────────────────────────────────────────────┘    │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

Past interviews collapsed by default, expanded on click.

---

## Question Bank Feature

### Structure
1. **AI-Recommended Questions** — Surfaced contextually based on interview type/stage
2. **Curated Question Database** — Browsable library organized by type
3. **User-Submitted Questions** — Custom questions with type assignment

### Question Types
- Behavioral (STAR format feedback)
- Hypothetical/Situational
- Technical
- Case/Product
- Common (Tell me about yourself, Why this company, etc.)

### Type Validation
When user adds a custom question:
1. User enters question text
2. User selects question type
3. AI validates: "You tagged this as behavioral, but it looks like a technical question. Is this correct?"
4. User confirms or corrects

This ensures proper feedback routing — behavioral questions get STAR feedback, technical questions get different evaluation criteria.

---

## Story Bank Feature

### Sources
- **Resume Extraction** — AI identifies potential stories from uploaded resume
- **User-Added** — Manual story entry
- **Practice History** — Stories used in past practice sessions

### Tagging by Competency
- Leadership
- Teamwork/Collaboration
- Problem Solving
- Conflict Resolution
- Failure/Learning
- Initiative/Ownership
- Communication
- Technical Achievement

### Contextual Surfacing
When practicing a question about leadership:
> "For leadership questions, consider these stories from your bank..."
> - Led the product launch for X (used 3 times, avg score: 85%)
> - Mentored junior team member (new, not yet practiced)

---

## Rigid Practice Sessions

### Creation Methods

**Method 1: Stage-Based**
- Select an interview stage (e.g., "Hiring Manager Round")
- System generates recommended question sequence
- User can customize: add/remove/reorder questions

**Method 2: Text Description**
- User describes: "Third round with the engineering manager, focused on system design and leadership"
- AI generates appropriate session structure

**Method 3: Custom Build**
- User selects questions from question bank
- Arranges into custom sequence
- Saves as reusable template

### Session Flow
1. Session intro: "Practicing for [Stage] — [X] questions, ~[Y] minutes"
2. Questions presented one at a time
3. User records answer (video/audio/text)
4. Per-question feedback provided
5. Session summary with aggregate score

---

## Post-Interview Feedback Loop

### Trigger
Day after interview date passes:
> "How did your Google interview go?"

### Debrief Form
- **Overall feeling:** Great / Okay / Rough
- **What went well:** Free text
- **What would you do differently:** Free text
- **Did they mention next steps?** Yes/No + details

### Outcome Tracking
Two weeks after interview:
> "It's been 2 weeks since your Google interview. Any updates?"

Options:
- Got the job! 🎉
- Rejected
- Still waiting
- Ghosted

### Insights Loop
Feed outcomes back into future preparation:
> "In your last 3 interviews, you mentioned struggling with system design questions. We recommend focusing on technical practice for your upcoming Meta interview."

---

## Interview Plan Generation

When user creates an interview, system generates a day-by-day plan:

### Inputs
- Days until interview
- Interview stages
- User's current story bank coverage
- User's practice history
- User's learning progress

### Output Example (7 days out)

| Day | Tasks |
|-----|-------|
| 7 | Research Google's PM interview process; Review your resume stories |
| 6 | Complete 2 behavioral practice sessions; Update story bank |
| 5 | Practice "Tell me about yourself"; Research the interviewers |
| 4 | Complete 2 case/product practice sessions |
| 3 | Review all stories; Practice weak areas identified |
| 2 | Light practice only; Prepare logistics and outfit |
| 1 | Final review; Rest and prepare mentally |

Plan is editable — user can adjust tasks, mark complete, or regenerate.

---

## Phased Delivery Approach

### Phase 1: Foundation
- Interview entity and CRUD
- Home dashboard with interview cards
- Basic interview detail view
- Interview list page

### Phase 2: Practice Integration
- Rigid practice sessions linked to interviews
- Per-session and per-question scoring
- Practice tab in interview detail

### Phase 3: Question & Story Banks
- Question bank with type system
- Story bank with competency tags
- Contextual surfacing

### Phase 4: Plan & Polish
- Interview plan generation
- Learn and Prepare tabs
- Post-interview feedback loop
- Outcome tracking

---

## Technical Considerations

### New Entities
- `interviews` table
- `interview_stages` table
- `interview_plans` table
- `interview_debriefs` table
- `questions` table (user-submitted)
- `stories` table

### API Endpoints
- `POST /interviews` — Create interview
- `GET /interviews` — List user's interviews
- `GET /interviews/:id` — Interview detail
- `PATCH /interviews/:id` — Update interview
- `POST /interviews/:id/debrief` — Submit debrief
- `GET /interviews/:id/plan` — Get/generate plan

### Frontend Routes
- `/` — Home dashboard
- `/interviews` — Interview list
- `/interviews/:id` — Interview detail
- `/interviews/:id/practice` — Practice tab
- `/interviews/:id/learn` — Learn tab
- `/interviews/:id/prepare` — Prepare tab
- `/practice` — Standalone practice (question bank, story bank)
- `/academy` — Learning modules

---

## Success Metrics

### Engagement
- % of users who create at least one interview
- Average interviews per user
- Interview plan completion rate
- Practice sessions per interview

### Outcomes
- Post-interview debrief submission rate
- Job offer rate (self-reported)
- Return usage after interview completion

### Conversion
- Impact on trial → paid conversion
- Impact on retention

---

## Open Questions

1. **Notification strategy:** Email reminders for plan items? Push notifications?
2. **Collaborative prep:** Can users share interview prep with mentors/coaches?
3. **Interview templates:** Pre-built interview structures for common companies (Google, Meta, Amazon)?
4. **Mobile experience:** How does this work on mobile? (Desktop-first, but consider)

---

*This specification represents the target state. Implementation will be phased as outlined above.*

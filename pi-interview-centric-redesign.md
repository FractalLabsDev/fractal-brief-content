# Practice Interviews: Interview-Centric Redesign

**Version:** 1.1
**Date:** February 10, 2026
**Author:** Matt Lim, Ruk
**Status:** Draft Specification

---

## Executive Summary

This specification outlines a fundamental redesign of Practice Interviews from a "practice tool" to an "interview preparation platform." The core shift: instead of abstract practice sessions, the user experience anchors around **real or target interviews** as first-class entities.

**Key Paradigm Shift:**
- **From:** "Get reps in at the gym" (abstract practice)
- **To:** "Get ready for THIS interview" (concrete, event-driven preparation)

**Architectural Foundation:**
The platform leverages a **three-layer context engineering** approach, powered by Fractal microservices (vector databases, context management, promptsmith). AI becomes progressively smarter as users build context at each layer, enabling deeply personalized coaching.

---

## Context Engineering Architecture

The AI coaching system uses three hierarchical layers of context, each building on the one below:

### Layer 1: User Context (Persistent)

The foundation layer — everything we know about the candidate across their entire journey.

| Data | Source | AI Use Cases |
|------|--------|--------------|
| Career history | Resume, profile | Generate relevant questions; anticipate interviewer concerns |
| Story bank | User-added, practice history | Surface best stories for each question |
| Practice history | All sessions | Identify patterns, track improvement |
| Feedback patterns | AI analysis | Focus coaching on weak areas |
| Learning progress | Academy activity | Recommend next learning modules |
| Outcome history | Debrief data | Learn what works for this user |

**Persistence:** Stored permanently, grows over time, survives interview completion.

### Layer 2: Role Context (Per Target Role)

The preparation layer — context specific to a job opportunity the user is pursuing.

| Data | Source | AI Use Cases |
|------|--------|--------------|
| Job description | User-uploaded | Match stories to requirements; generate relevant questions |
| Company research | User notes + AI research | Tailor answers to company culture |
| Interviewer notes | User-added LinkedIn insights | Adjust communication style |
| Flagged questions | User-marked from question bank | Prioritize practice sessions |
| Role-specific stories | User-curated subset | Quick access during prep |
| Company values | Research, job posting | Align answers to what matters |
| Prep availability | User-selected (15min/30min/1hr/2hr+ per day) | Scale interview plan to realistic capacity |

**Persistence:** Lives with the Role entity. Multiple Interviews can reference the same Role.

### Layer 3: Session Context (Per Interview/Practice)

The execution layer — real-time context for the current interaction.

| Data | Source | AI Use Cases |
|------|--------|--------------|
| Interview stage | User-selected | Stage-appropriate question selection |
| Session type | Behavioral, technical, case | Correct feedback framework |
| Current question | Active practice | Real-time coaching |
| Transcript | Live or recorded | Immediate feedback |
| Previous answers (session) | Same session | Track consistency, avoid repetition |
| Time remaining | Session clock | Pace guidance |

**Persistence:** Session-scoped. Summarized into Layer 1 after completion.

### Context Flow Diagram

```
┌─────────────────────────────────────────────────────────────────┐
│                    AI COACHING ENGINE                          │
│                                                                 │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │ LAYER 3: Session Context (real-time)                    │   │
│  │ Current question, transcript, stage type                │   │
│  └───────────────────────┬─────────────────────────────────┘   │
│                          │                                      │
│  ┌───────────────────────▼─────────────────────────────────┐   │
│  │ LAYER 2: Role Context (per target role)                 │   │
│  │ Job description, company research, flagged questions    │   │
│  └───────────────────────┬─────────────────────────────────┘   │
│                          │                                      │
│  ┌───────────────────────▼─────────────────────────────────┐   │
│  │ LAYER 1: User Context (persistent)                      │   │
│  │ Career history, story bank, feedback patterns           │   │
│  └─────────────────────────────────────────────────────────┘   │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### How AI Uses Context (Examples)

| Feature | Context Layers Used | Behavior |
|---------|---------------------|----------|
| **Interview Plan** | L1 + L2 | Generate personalized plan based on user's weak areas (L1), job requirements (L2), and prep availability (L2) |
| **Practice Questions** | L1 + L2 + L3 | Select questions matching role requirements (L2), avoiding recently practiced (L1), appropriate for stage (L3) |
| **Story Suggestions** | L1 + L2 + L3 | Surface stories matching current question (L3), prioritizing those aligned to job description (L2), sorted by past performance (L1) |
| **Feedback Generation** | L1 + L2 + L3 | Feedback incorporates company values (L2), references past feedback patterns (L1), and evaluates current answer (L3) |
| **Ideal Answer** | L1 + L2 + L3 | Generate example answer using user's actual stories (L1), tailored to company (L2), formatted for question type (L3) |
| **Prepare Checklist** | L2 | Generate role-specific preparation based on job requirements |

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

### Role Entity (Layer 2 Context Container)

The Role is the **persistent context container** for a target job opportunity. Multiple Interviews can reference the same Role (e.g., multiple rounds at the same company).

```typescript
interface Role {
  id: string;
  userId: string;

  // Basic info
  company: string;
  title: string;
  location?: string;
  salaryRange?: string;

  // Context (Layer 2 data)
  jobDescription?: string;       // Full JD text
  companyResearch?: string;      // User notes about company
  companyValues?: string[];      // Extracted or user-added
  interviewerNotes?: InterviewerNote[];
  flaggedQuestionIds?: string[]; // Questions user wants to practice
  curatedStoryIds?: string[];    // Stories user marked relevant

  // Metadata
  status: 'active' | 'archived' | 'got_offer' | 'rejected';
  createdAt: Date;
  updatedAt: Date;
}

interface InterviewerNote {
  name: string;
  linkedInUrl?: string;
  notes?: string;
  role?: string;  // Their role at the company
}
```

### Interview Entity

Interviews are **scheduled events** within a Role. They inherit Role context.

```typescript
interface Interview {
  id: string;
  roleId: string;  // References Role for Layer 2 context
  userId: string;

  // Event details
  date: Date;
  location?: string;  // Address, "Remote", video link
  stages: InterviewStage[];

  // State
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

### User Context Entity (Layer 1)

User-level persistent context for AI coaching.

```typescript
interface UserContext {
  userId: string;

  // Career history
  resumeText?: string;
  careerSummary?: string;

  // AI-generated insights
  feedbackPatterns?: FeedbackPattern[];
  strengthAreas?: string[];
  improvementAreas?: string[];

  // Aggregated from sessions
  totalPracticeSessions: number;
  totalQuestionsAnswered: number;
  averageScore: number;

  updatedAt: Date;
}

interface FeedbackPattern {
  area: string;  // e.g., "STAR structure", "Conciseness"
  trend: 'improving' | 'stable' | 'declining';
  frequency: number;  // How often this appears in feedback
  lastMentioned: Date;
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

### Prepare Tab (Context-Building Workspace)

The Prepare tab is where users **build Layer 2 context** — all the role-specific information that makes AI coaching smarter.

```
┌─────────────────────────────────────────────────────────────────┐
│  Prepare for Google PM Interview                                │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  📄 Job Description                              [Edit] [+ Add] │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │ Product Manager, Search                                 │   │
│  │ Minimum qualifications: 5+ years PM experience...       │   │
│  │ Preferred: Experience with ML/AI products...            │   │
│  │                                                         │   │
│  │ 🤖 AI extracted: Leadership, Cross-functional,          │   │
│  │    Data-driven, Technical fluency                       │   │
│  └─────────────────────────────────────────────────────────┘   │
│                                                                 │
│  🏢 Company Research                             [Edit] [+ Add] │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │ Your notes:                                             │   │
│  │ "Focus on user impact. Recent AI announcements.         │   │
│  │  CEO mentioned 'AI-first' in earnings call."            │   │
│  │                                                         │   │
│  │ 🤖 AI research: Google values innovation, data-driven   │   │
│  │    decisions, and 10x thinking...                       │   │
│  └─────────────────────────────────────────────────────────┘   │
│                                                                 │
│  👥 Interviewers                                       [+ Add] │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │ Sarah Chen · Senior PM, Search                          │   │
│  │ "Background in ML. Focus on technical depth"            │   │
│  │ [LinkedIn ↗]                                            │   │
│  └─────────────────────────────────────────────────────────┘   │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │ Mike Johnson · Engineering Manager                      │   │
│  │ "10 years at Google. Likely system design focused"      │   │
│  └─────────────────────────────────────────────────────────┘   │
│                                                                 │
│  📚 Story Bank                                   [8 stories ↗] │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │ Curated for this role:                                  │   │
│  │ ☐ Led cross-functional launch (Leadership, Data)        │   │
│  │ ☐ Resolved eng/design conflict (Cross-functional)       │   │
│  │ [+ Curate more stories for Google PM]                   │   │
│  └─────────────────────────────────────────────────────────┘   │
│                                                                 │
│  ❓ Flagged Questions                                  [+ Add] │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │ Questions you want to practice for this role:           │   │
│  │ ☐ "Tell me about a time you used data to make a         │   │
│  │    product decision" (Behavioral)                       │   │
│  │ ☐ "How would you improve Google Search?" (Product)      │   │
│  │ [Browse Question Bank ↗]                                │   │
│  └─────────────────────────────────────────────────────────┘   │
│                                                                 │
│  ─────────────────────────────────────────────────────────────  │
│                                                                 │
│  📋 Day-Of Checklist                                            │
│  ☐ Confirm interview location                                   │
│  ☐ Prepare your outfit                                          │
│  ☐ Review your top 3 stories                                    │
│  ☐ Prepare questions to ask                                     │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

**Key insight:** Everything in this tab feeds the AI. More context = smarter coaching.

### Prep Availability Input

Captured in the Prepare tab to inform interview plan generation:

```
┌─────────────────────────────────────────────────────────────────┐
│  ⏱️ Your Prep Availability                                      │
│                                                                 │
│  How much time can you dedicate to interview prep daily?        │
│  This helps us create a realistic plan for your schedule.       │
│                                                                 │
│  [15 min]  [30 min]  [1 hour ✓]  [2+ hours]                    │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

This input directly affects:
- **Interview Plan density** — More availability = more tasks per day
- **Practice recommendations** — Fits session lengths to available time
- **Learning pacing** — Spreads Academy modules appropriately

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

## Fractal Microservices Integration

The context engineering architecture is powered by Fractal Labs microservices:

### Vector Databases (Semantic Search)

Used for similarity-based retrieval across context layers.

| Use Case | Data Indexed | Query Example |
|----------|--------------|---------------|
| **Story Surfacing** | Story bank entries | "Find stories matching 'led cross-functional team through ambiguity'" |
| **Question Matching** | Question bank | "Questions similar to 'Tell me about a time you failed'" |
| **Feedback Retrieval** | Past feedback | "Previous feedback about STAR structure for this user" |
| **Company Research** | Scraped company data | "What do we know about Google's interview process?" |

**Implementation:**
- Stories and questions embedded at creation time
- Feedback patterns extracted and embedded after each session
- Company research indexed from external sources (Glassdoor, LinkedIn, press)
- All queries include user_id filter for multi-tenant isolation

### Context Management Service

Orchestrates the three context layers and manages context windows for AI calls.

```
┌─────────────────────────────────────────────────────────────────┐
│                   CONTEXT MANAGEMENT SERVICE                    │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  buildSessionContext(userId, roleId, sessionId) → ContextBundle │
│                                                                 │
│  1. Fetch Layer 1: UserContext                                  │
│     - Career summary, feedback patterns, strength areas         │
│     - Recent practice history (last 10 sessions)                │
│                                                                 │
│  2. Fetch Layer 2: RoleContext                                  │
│     - Job description, company research, interviewer notes      │
│     - Flagged questions, curated stories                        │
│                                                                 │
│  3. Fetch Layer 3: SessionContext                               │
│     - Current stage, question type, transcript so far           │
│                                                                 │
│  4. Vector retrieval (conditional)                              │
│     - If asking for story: retrieve similar stories             │
│     - If generating feedback: retrieve past feedback patterns   │
│                                                                 │
│  5. Token budget management                                     │
│     - Prioritize by relevance score                             │
│     - Truncate lower-priority context if exceeding budget       │
│                                                                 │
│  Returns: Structured context bundle ready for promptsmith       │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

**Key responsibilities:**
- Context aggregation across layers
- Token budget optimization (fit within model limits)
- Caching frequently-accessed context
- Context freshness management (invalidate on updates)

### Promptsmith (Prompt Engineering)

Transforms context bundles into optimized prompts for specific AI tasks.

| Task | Prompt Template | Context Used |
|------|-----------------|--------------|
| **Generate Interview Plan** | `interview_plan_v2` | L1 (weak areas, history) + L2 (job requirements, days until) |
| **Evaluate Answer** | `answer_feedback_v3` | L1 (feedback patterns) + L2 (company values) + L3 (question, transcript) |
| **Generate Ideal Answer** | `ideal_answer_v1` | L1 (user stories) + L2 (company context) + L3 (question type) |
| **Surface Stories** | `story_suggestion_v1` | L1 (story bank) + L2 (role requirements) + L3 (current question) |
| **Company Research** | `company_research_v1` | L2 (company name) + external data |

**Prompt versioning:** All prompts versioned and A/B testable. Promptsmith tracks which version produced which outputs for quality iteration.

**Dynamic prompt assembly:**
```typescript
// Example: Building feedback prompt
const feedbackPrompt = promptsmith.build('answer_feedback_v3', {
  // Layer 1
  user_strengths: context.userContext.strengthAreas,
  past_feedback_patterns: context.userContext.feedbackPatterns,

  // Layer 2
  company_values: context.roleContext.companyValues,
  job_requirements: context.roleContext.extractedRequirements,

  // Layer 3
  question: context.sessionContext.currentQuestion,
  answer_transcript: context.sessionContext.transcript,
  question_type: context.sessionContext.questionType,
});
```

### Service Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                     PRACTICE INTERVIEWS                          │
│                        (Frontend)                                │
└──────────────────────────┬──────────────────────────────────────┘
                           │
                           ▼
┌─────────────────────────────────────────────────────────────────┐
│                    PI Backend API                                │
│  (Roles, Interviews, Sessions, Stories, Questions)               │
└─────────┬───────────────────────────────────┬───────────────────┘
          │                                   │
          ▼                                   ▼
┌─────────────────────┐             ┌─────────────────────┐
│   Context Manager   │◄───────────►│    Vector DB        │
│   (Fractal OS)      │             │   (Fractal OS)      │
└─────────┬───────────┘             └─────────────────────┘
          │
          ▼
┌─────────────────────┐
│    Promptsmith      │
│   (Fractal OS)      │
└─────────┬───────────┘
          │
          ▼
┌─────────────────────┐
│   Claude/LLM API    │
└─────────────────────┘
```

---

## Technical Considerations

### New Entities
- `roles` table (Layer 2 context container)
- `interviews` table (references role)
- `interview_stages` table
- `interview_plans` table
- `interview_debriefs` table
- `questions` table (user-submitted)
- `stories` table
- `user_context` table (Layer 1 aggregations)
- `interviewer_notes` table

### API Endpoints

**Roles:**
- `POST /roles` — Create role
- `GET /roles` — List user's roles
- `GET /roles/:id` — Role detail with context
- `PATCH /roles/:id` — Update role context
- `POST /roles/:id/research` — Trigger AI company research

**Interviews:**
- `POST /interviews` — Create interview (references role)
- `GET /interviews` — List user's interviews
- `GET /interviews/:id` — Interview detail
- `PATCH /interviews/:id` — Update interview
- `POST /interviews/:id/debrief` — Submit debrief
- `GET /interviews/:id/plan` — Get/generate plan

**Context:**
- `GET /context/session/:sessionId` — Get assembled context bundle
- `POST /context/feedback` — Submit feedback for pattern extraction

### Frontend Routes
- `/` — Home dashboard
- `/roles` — Role list (optional, may merge with interviews)
- `/roles/:id` — Role detail (context workspace)
- `/interviews` — Interview list
- `/interviews/:id` — Interview detail
- `/interviews/:id/practice` — Practice tab
- `/interviews/:id/learn` — Learn tab
- `/interviews/:id/prepare` — Prepare tab (context building)
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

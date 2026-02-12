# Practice Interviews v2 - Phased Build Plan

## Context Summary

### What We're Building
An **interview-centric preparation platform** that shifts from "practice interviews" to "get ready for THIS interview." Core paradigm: Interviews are first-class entities with Opportunities containing multiple Interviews, each with Practice sessions.

### Key Entities (from Max's visualization + thread discussion)
- **UserProfile** → LinkedIn data, strengths, preferences
- **Opportunity** → A role at a company the user is pursuing
- **Interview** → A specific interview within an opportunity (phone screen, technical, etc.)
- **Story** → STAR-format experiences in user's story bank
- **Question** → User's curated question bank
- **PracticeSession** → Structured or quick practice linked to interviews
- **StructuredPracticeTemplate** → Reusable session templates

### Architecture Principles
1. **Clean slate** - New repos avoid 3-repo complexity (web→proxy→backend)
2. **Agentic-first** - CLAUDE.md and docs come first, enable AI-assisted development
3. **Atomic design** - DRY components, no jitter between pages
4. **Beta deployment** - beta.practiceinterviews.com, separate DB, push directly

### Tech Stack (from Max's scaffolding PR)
- **Backend**: Node/Express, TypeScript, Sequelize + PostgreSQL, Vitest
- **Frontend**: React, TypeScript (assume modern stack - Vite, etc.)
- **Quality**: ESLint + Prettier + Husky, Sentry, GitHub Actions CI/CD

---

## Proposed Phases

### Phase 0: Foundation (Documentation & Scaffolding)
**Goal**: Set up both repos for agentic development before writing features.

**Backend (interview-prep-backend)**:
- [ ] Review/merge Max's scaffolding PR with initial setup
- [ ] Create comprehensive CLAUDE.md with architecture decisions
- [ ] Create \`docs/\` folder with:
  - \`docs/ARCHITECTURE.md\` - System overview, entity relationships
  - \`docs/API.md\` - Endpoint conventions, response formats
  - \`docs/ENTITIES.md\` - Full data model with relationships
  - \`docs/DEVELOPMENT.md\` - Local setup, seeding, testing
- [ ] Create seeders for mock data (opportunities, interviews, stories, questions)
- [ ] Set up database migrations for all core entities upfront

**Frontend (interview-prep-web)**:
- [ ] Initialize with Vite + React + TypeScript
- [ ] Create CLAUDE.md with component patterns, state management approach
- [ ] Create \`docs/\` folder with:
  - \`docs/COMPONENTS.md\` - Atomic design system documentation
  - \`docs/ROUTING.md\` - Page structure and navigation
  - \`docs/STATE.md\` - State management patterns
- [ ] Set up design system foundation (tokens, primitives)
- [ ] Configure for beta.practiceinterviews.com deployment

**Deliverable**: Both repos ready for feature development with clear AI-readable documentation.

---

### Phase 1: Core User Context
**Goal**: Build the foundation that makes everything else personalized.

**Backend**:
- [ ] UserProfile entity + CRUD endpoints
- [ ] Story bank: Story entity with STAR fields
- [ ] Question bank: Question entity with categories
- [ ] Story ↔ Question many-to-many linking

**Frontend**:
- [ ] Profile page: Overview, LinkedIn data display, preferences
- [ ] Story bank UI: Add/edit/view stories (text input first)
- [ ] Question bank UI: Search, filter, add questions
- [ ] Story-Question linking interface

**Why first**: All downstream features (AI plans, structured practice, recommendations) need user context to personalize.

---

### Phase 2: Opportunity & Interview Tracking
**Goal**: Enable users to track real interviews they're preparing for.

**Backend**:
- [ ] Opportunity entity with company, role, status
- [ ] Interview entity linked to Opportunity (multiple per opportunity)
- [ ] Interviewer entity for tracking interviewer context
- [ ] CRUD endpoints for all

**Frontend**:
- [ ] Home dashboard with Opportunity cards
- [ ] Add Opportunity flow (manual entry)
- [ ] Opportunity detail view with Interviews list
- [ ] Add Interview within Opportunity
- [ ] Interview detail with notes field

**Why second**: This creates the organizing principle. Once users have context (Phase 1) and interviews to prepare for (Phase 2), we can build the actual preparation features.

---

### Phase 3: Structured Practice
**Goal**: Enable practice sessions tied to specific interviews.

**Backend**:
- [ ] PracticeSession entity linked to Interview
- [ ] StructuredPracticeTemplate entity
- [ ] TemplateQuestion join table (ordered questions)
- [ ] Session recording/feedback storage

**Frontend**:
- [ ] Practice hub page (Quick Practice vs Structured)
- [ ] Template selection interface
- [ ] Custom session builder (number of questions, select specific questions)
- [ ] Practice flow (question display, recording, AI feedback)
- [ ] Post-practice feedback view

**Why third**: This is where existing PI value (practice sessions) gets integrated into the new paradigm.

---

### Phase 4: AI Intelligence Layer
**Goal**: Add the "magic" - personalized plans, company research, recommendations.

**Backend**:
- [ ] Plan generation endpoint (interview date + context → preparation roadmap)
- [ ] Company research agent (background job or on-demand)
- [ ] Job scraping endpoint (LinkedIn URL → structured data)
- [ ] Recommendation engine (weak areas → suggested practice)

**Frontend**:
- [ ] Plan tab within Opportunity detail
- [ ] Company research display in Prepare tab
- [ ] LinkedIn job URL paste → auto-create Opportunity
- [ ] Learn page with personalized recommendations

**Why fourth**: Intelligence requires the data foundation. Without user context, interviews, and practice history, AI can't personalize.

---

### Phase 5: Voice & Enhancement
**Goal**: Polish and advanced features.

- [ ] Voice-to-STAR: Speak story → AI converts to STAR format
- [ ] Drag-and-drop question selection in session builder
- [ ] Interview Academy integration with personalized recommendations
- [ ] Post-interview debrief flow (outcome tracking)
- [ ] Readiness scoring (aggregate scores at all levels)

---

## Build Approach Options

### Option A: Sequential Waterfall
Build Phase 0 → 1 → 2 → 3 → 4 → 5 in order. Safe, predictable.

**Timeline estimate**: 4-6 weeks to Phase 3 (usable product)

### Option B: Parallel Tracks
- **Track 1 (Infra/Backend)**: Ruk focuses on Phase 0 + backend for Phases 1-3
- **Track 2 (Frontend/UI)**: Max focuses on frontend for Phases 1-3 using mock data
- **Sync point**: Integrate when both tracks reach Phase 3

**Timeline estimate**: 2-3 weeks to Phase 3

### Option C: MVP Slice
Build a thin vertical slice through all phases for ONE user journey:
1. Create Opportunity (manual)
2. Add one Interview
3. Do one structured practice session
4. See basic feedback

Then widen each layer.

**Timeline estimate**: 1 week to skeleton, then iterative widening

---

## Questions for Matt

1. **Phase 0 scope**: How thorough should documentation be before writing features? Minimal (README + CLAUDE.md) or comprehensive (full docs folder)?

2. **Build approach preference**: Sequential, parallel tracks, or MVP slice?

3. **Authentication**: Rebuild auth from scratch in new backend, or use existing fractalai-backend auth via API? (Affects Phase 1 timeline)

4. **AI features (Phase 4)**: Build from scratch or port prompts/logic from existing PI where applicable?

5. **Max's entity diagram**: The gist URL appears broken. Can we get a fresh link or have Max share it directly?

6. **UI starting point**: Use the existing \`feature/interview-centric-redesign\` branch as reference, or build fresh based on Loom?


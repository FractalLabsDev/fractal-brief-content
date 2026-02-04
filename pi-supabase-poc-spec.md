# Practice Interviews Supabase POC Spec

**Purpose:** Get the app running on Supabase as a proof of concept to evaluate feasibility  
**Timeline:** Target 1 week for MVP, not production-ready

---

## Critical Considerations Before Starting

### 1. Database Schema Design
Supabase uses PostgreSQL—same as current Heroku setup—so schema should translate directly. However:

- **Row Level Security (RLS)**: Supabase relies heavily on RLS policies. You'll need to define who can access what at the database level.
- **Foreign keys to TalkWise data**: Currently users, roles, permissions live in TalkWise. For POC, you can either:
  - Import a snapshot of users into Supabase
  - Create test users directly in Supabase Auth
  - Skip auth complexity and use a mock user for testing

### 2. Auth Strategy
Three options for POC:

| Option | Pros | Cons |
|--------|------|------|
| **Supabase Auth** | Native, easy setup, Google/Email built-in | Different from production auth |
| **Mock auth** | Fastest, skip auth entirely | Not realistic test |
| **JWT passthrough** | Could reuse existing tokens | Complex setup |

**Recommendation:** Use Supabase Auth with Google login for POC. It's quick to set up and gives realistic UX.

### 3. What to Skip for POC
- Stripe integration (use mock subscription status)
- Customer.io integration
- Admin panel
- Analytics page (complex queries)
- Interview Academy (separate content system)
- Notion integration
- Resume parsing

---

## Feature Checklist for POC

### Phase 1: Core Infrastructure (Day 1-2)

**Supabase Setup:**
- [ ] Create Supabase project
- [ ] Set up Auth (email + Google)
- [ ] Configure MCP connection in Cursor
- [ ] Create base schema

**Schema (from PI backend models):**
```sql
-- Core entities
users (from Supabase Auth, extended with profile)
jobs (id, user_id, title, company, description, created_at)
questions (id, job_id, text, question_type, category, created_at)
answers (id, question_id, user_id, transcript, feedback, score, created_at)

-- New for upcoming features
practice_sessions (id, user_id, job_id, session_type, status, created_at)
-- story_bank (future)
-- question_bank (future)
```

**Backend Setup:**
- [ ] New Express app (or Hono/Fastify for speed)
- [ ] Supabase client connection
- [ ] Basic middleware (auth, error handling)
- [ ] Health check endpoint

### Phase 2: Authentication Flow (Day 2)

- [ ] Login page (Google OAuth via Supabase)
- [ ] Signup flow
- [ ] Protected route wrapper
- [ ] User profile storage
- [ ] Session management

### Phase 3: Job/Role Setup (Day 3)

- [ ] Create/edit job target (company, role, description)
- [ ] Job selection on dashboard
- [ ] Job context passed to question generation

### Phase 4: Practice Interview Flow (Day 3-4)

**This is the core product loop:**
- [ ] Start practice session
- [ ] Question display
- [ ] Voice recording (can reuse existing component)
- [ ] Transcription (hit existing Deepgram integration or mock)
- [ ] AI feedback generation (hit PromptSmith via API)
- [ ] Display feedback with scores
- [ ] Save answer to database

### Phase 5: Dashboard & History (Day 5)

- [ ] Dashboard with recent sessions
- [ ] Answer history view
- [ ] Basic stats (questions answered, average score)
- [ ] Quick practice entry point

---

## API Endpoints for POC

```
Auth (handled by Supabase):
POST /auth/signup
POST /auth/login
POST /auth/logout
GET  /auth/session

Jobs:
GET    /api/jobs          - List user's jobs
POST   /api/jobs          - Create job
GET    /api/jobs/:id      - Get job details
PUT    /api/jobs/:id      - Update job
DELETE /api/jobs/:id      - Delete job

Practice:
POST   /api/practice/start     - Start session, get first question
POST   /api/practice/answer    - Submit answer, get feedback
GET    /api/practice/history   - Get past sessions

Questions:
GET    /api/questions          - Get questions for a job
POST   /api/questions/generate - Generate AI questions for job

Answers:
GET    /api/answers            - Get user's answers
GET    /api/answers/:id        - Get specific answer with feedback
```

---

## Third-Party Integrations for POC

| Service | Status | Notes |
|---------|--------|-------|
| **PromptSmith** | Required | Question generation + feedback. Use existing Fractal OS integration |
| **Deepgram** | Optional | Can mock transcription with text input for speed |
| **Stripe** | Skip | Mock subscription status (everyone is "pro") |
| **Customer.io** | Skip | No emails needed for POC |
| **Notion** | Skip | Academy content not needed |

---

## Frontend Pages for POC

```
/login              - Supabase Auth UI
/signup             - Supabase Auth UI
/onboarding         - Basic job setup (simplified)
/dashboard          - Main hub
/practice           - Interview session
/practice/:id       - Session in progress
/history            - Past answers
/history/:id        - Answer detail with feedback
/settings           - Basic profile (optional)
```

---

## What Success Looks Like

**POC is successful if:**
1. User can sign up with Google
2. User can set up a target job
3. User can start a practice session
4. User can answer questions (text input OK for POC)
5. User gets AI-generated feedback
6. User can see their history

**Not required for POC:**
- Voice recording (can use text input)
- Multiple question types
- Scoring/analytics
- Onboarding flow (just job setup)
- Interview Academy
- Admin features

---

## Tech Stack Recommendation

```
Frontend:
- React + Vite (faster than CRA)
- TailwindCSS (faster styling)
- Supabase JS client
- React Router

Backend:
- Node.js + Express (familiar)
- Supabase JS client
- PromptSmith API client

Database:
- Supabase (PostgreSQL + Auth + Realtime)

Deployment (for testing):
- Vercel (frontend)
- Supabase (database + auth)
- Heroku or Railway (backend) OR Supabase Edge Functions
```

---

## Risk Mitigation

| Risk | Mitigation |
|------|------------|
| POC takes longer than expected | Time-box to 5 days, cut scope aggressively |
| PromptSmith integration issues | Test API separately first |
| Supabase limitations discovered | Document for production decision |
| Voice recording complexity | Skip voice, use text input |

---

## Decision Points After POC

After building the POC, you'll have data to answer:
1. Is Supabase viable for production?
2. How much faster is development without TalkWise dependency?
3. What's the realistic timeline for full feature parity?
4. Should we migrate existing data or start fresh for new users?

---

*This spec is scoped for proof-of-concept only. Production would require Stripe, proper error handling, testing, monitoring, and data migration strategy.*

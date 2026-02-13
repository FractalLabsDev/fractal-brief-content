# Practice Interviews: Context Engineering Implementation Plan

**Version:** 1.0  
**Date:** February 13, 2026  
**Authors:** Matt Lim, Ruk  
**Status:** Draft for Review

---

## Executive Summary

This document outlines the implementation plan for adding context engineering capabilities to Practice Interviews. The goal is to make AI coaching progressively smarter by leveraging three layers of context: User (L1), Opportunity (L2), and Session (L3).

**Key Constraint:** Interview Prep maintains a simple architecture (1 frontend, 1 backend, 1 database) while leveraging Fractal OS microservices for AI orchestration.

---

## Architecture Overview

```
┌─────────────────────────────────────────────────────────────────────┐
│                     INTERVIEW PREP SYSTEM                            │
│  ┌─────────────┐    ┌─────────────┐    ┌─────────────┐              │
│  │  Frontend   │ →  │   Backend   │ →  │  Database   │              │
│  │  (React)    │    │  (Express)  │    │ (Postgres)  │              │
│  └─────────────┘    └──────┬──────┘    └─────────────┘              │
└────────────────────────────┼────────────────────────────────────────┘
                             │
                             ▼
┌─────────────────────────────────────────────────────────────────────┐
│                      FRACTAL OS SERVICES                             │
│  ┌─────────────┐    ┌─────────────┐    ┌─────────────┐              │
│  │   Context   │ →  │ Promptsmith │ →  │  Claude API │              │
│  │   Manager   │    │ (templates) │    │             │              │
│  └──────┬──────┘    └─────────────┘    └─────────────┘              │
│         │                                                            │
│         ▼                                                            │
│  ┌─────────────┐                                                     │
│  │  Vector DB  │  (embeddings for semantic search)                   │
│  │  (Pinecone) │                                                     │
│  └─────────────┘                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

**Data stays in Interview Prep.** Fractal OS services are stateless orchestrators that receive context, process it, and return results.

---

## The Three Context Layers

### Layer 1: User Context (Persistent)

Everything we know about the candidate across their entire journey.

| Data | Storage | Source |
|------|---------|--------|
| Career history | `user_profiles.linkedinData` | Resume upload, LinkedIn |
| Story bank | `stories` table | User-added, AI-extracted from resume |
| Practice history | `practice_sessions`, `practice_questions` | All past sessions |
| Feedback patterns | `user_context.feedbackPatterns` (NEW) | AI-aggregated from feedback |
| Strength areas | `user_context.strengthAreas` (NEW) | AI-computed from patterns |
| Improvement areas | `user_context.improvementAreas` (NEW) | AI-computed from patterns |

### Layer 2: Opportunity Context (Per Target Role)

Context specific to a job opportunity the user is pursuing.

| Data | Storage | Source |
|------|---------|--------|
| Job description | `opportunities.jobDescription` | User-uploaded |
| Company research | `opportunities.companyResearch` | AI-generated |
| Company values | `opportunities.companyValues` (NEW) | Extracted from JD + research |
| Interviewer notes | `interviews.interviewers` | User-added |
| Flagged questions | `opportunity_questions` (NEW) | User-curated |
| Curated stories | `opportunity_stories` (NEW) | User-selected |

### Layer 3: Session Context (Ephemeral)

Real-time context for the current practice session.

| Data | Storage | Source |
|------|---------|--------|
| Interview stage | `practice_sessions.interviewId` → `interviews.type` | Session config |
| Session type | `practice_sessions.type` | Quick vs structured |
| Current question | In-memory | Active practice |
| Transcript so far | In-memory / `practice_questions.responseText` | Live or recorded |
| Previous answers | `practice_questions` (same session) | Session progress |

---

## Implementation Phases

### Phase 1: Database Schema Extensions

**Goal:** Add the missing tables and columns for context storage.

**New Table: `user_context`**
```sql
CREATE TABLE user_context (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID NOT NULL REFERENCES users(id) UNIQUE,

  -- AI-aggregated insights
  feedback_patterns JSONB DEFAULT '[]',
  strength_areas JSONB DEFAULT '[]',
  improvement_areas JSONB DEFAULT '[]',

  -- Aggregated stats
  total_practice_sessions INTEGER DEFAULT 0,
  total_questions_answered INTEGER DEFAULT 0,
  average_score DECIMAL(3,1),

  -- Metadata
  last_aggregation_at TIMESTAMP,
  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW()
);
```

**feedback_patterns structure:**
```json
[
  {
    "area": "STAR structure",
    "trend": "improving",
    "frequency": 12,
    "avgScore": 7.2,
    "recentExamples": ["session_id_1", "session_id_2"]
  }
]
```

**New Columns on `opportunities`:**
```sql
ALTER TABLE opportunities
  ADD COLUMN company_values JSONB DEFAULT '[]',
  ADD COLUMN flagged_question_ids UUID[] DEFAULT '{}',
  ADD COLUMN curated_story_ids UUID[] DEFAULT '{}';
```

**New Table: `embeddings`**
```sql
CREATE TABLE embeddings (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  entity_type VARCHAR(50) NOT NULL, -- 'story', 'question', 'feedback'
  entity_id UUID NOT NULL,
  embedding_model VARCHAR(100) NOT NULL, -- 'text-embedding-3-small'
  embedding VECTOR(1536), -- pgvector extension
  content_hash VARCHAR(64), -- detect when re-embedding needed
  created_at TIMESTAMP DEFAULT NOW()
);

CREATE INDEX idx_embeddings_entity ON embeddings(entity_type, entity_id);
CREATE INDEX idx_embeddings_vector ON embeddings USING ivfflat (embedding vector_cosine_ops);
```

**Migration order:**
1. Enable pgvector extension
2. Create `user_context` table
3. Add columns to `opportunities`
4. Create `embeddings` table
5. Backfill `user_context` for existing users

---

### Phase 2: UserContext Aggregation Service

**Goal:** Compute L1 insights from practice history.

**Trigger:** After each practice session completes.

**Implementation (in PI Backend):**

```typescript
// src/services/userContext/aggregator.ts

interface AggregationResult {
  feedbackPatterns: FeedbackPattern[];
  strengthAreas: string[];
  improvementAreas: string[];
  stats: {
    totalSessions: number;
    totalQuestions: number;
    averageScore: number;
  };
}

async function aggregateUserContext(userId: string): Promise<AggregationResult> {
  // 1. Fetch recent practice data (last 30 days or 50 sessions)
  const sessions = await PracticeSession.findAll({
    where: { userId },
    include: [{ model: PracticeQuestion, include: [Question] }],
    order: [['completedAt', 'DESC']],
    limit: 50,
  });

  // 2. Extract all feedback
  const allFeedback = sessions.flatMap(s =>
    s.practiceQuestions.map(pq => pq.feedback)
  );

  // 3. Call Fractal OS for pattern extraction
  const patterns = await contextManager.extractFeedbackPatterns({
    feedback: allFeedback,
    userId,
  });

  // 4. Update user_context table
  await UserContext.upsert({
    userId,
    feedbackPatterns: patterns.patterns,
    strengthAreas: patterns.strengths,
    improvementAreas: patterns.improvements,
    totalPracticeSessions: sessions.length,
    totalQuestionsAnswered: allFeedback.length,
    averageScore: calculateAverage(sessions),
    lastAggregationAt: new Date(),
  });

  return patterns;
}
```

**Fractal OS Integration:**

The `contextManager.extractFeedbackPatterns()` call goes to a Fractal OS endpoint that:
1. Receives raw feedback array
2. Uses Claude to identify recurring themes
3. Returns structured patterns

This keeps the AI logic in Fractal OS while PI Backend handles data access and storage.

---

### Phase 3: Vector Embeddings Pipeline

**Goal:** Enable semantic search for stories and questions.

**Option A: pgvector in PI Database (Recommended)**

Keeps all data in one place. Good for our scale (<100k embeddings).

```typescript
// src/services/embeddings/embedder.ts

async function embedStory(storyId: string): Promise<void> {
  const story = await Story.findByPk(storyId);

  // Concatenate STAR fields for embedding
  const content = [
    story.title,
    story.situation,
    story.task,
    story.action,
    story.result,
  ].filter(Boolean).join('\n\n');

  const contentHash = crypto.createHash('sha256').update(content).digest('hex');

  // Check if already embedded with same content
  const existing = await Embedding.findOne({
    where: { entityType: 'story', entityId: storyId, contentHash },
  });
  if (existing) return;

  // Get embedding from OpenAI (or Fractal OS proxy)
  const embedding = await openai.embeddings.create({
    model: 'text-embedding-3-small',
    input: content,
  });

  await Embedding.upsert({
    entityType: 'story',
    entityId: storyId,
    embeddingModel: 'text-embedding-3-small',
    embedding: embedding.data[0].embedding,
    contentHash,
  });
}
```

**Semantic Search Query:**

```typescript
async function findSimilarStories(
  userId: string,
  questionText: string,
  limit: number = 5
): Promise<Story[]> {
  // 1. Embed the question
  const questionEmbedding = await getEmbedding(questionText);

  // 2. Find similar stories using pgvector
  const results = await sequelize.query(`
    SELECT s.*, e.embedding <=> $1::vector AS distance
    FROM stories s
    JOIN embeddings e ON e.entity_type = 'story' AND e.entity_id = s.id
    WHERE s.user_id = $2 AND s.is_archived = false
    ORDER BY distance
    LIMIT $3
  `, {
    bind: [questionEmbedding, userId, limit],
    type: QueryTypes.SELECT,
  });

  return results;
}
```

**Option B: Fractal OS Vector Service**

If we later need cross-product search (e.g., search stories across all users for templates), we'd use a shared vector DB. Not needed now.

---

### Phase 4: Context Assembly Service

**Goal:** Orchestrate context gathering for AI calls.

**This is the integration point with Fractal OS.**

```typescript
// src/services/context/assembler.ts

interface ContextBundle {
  user: UserContext;      // L1
  opportunity: OpportunityContext;  // L2
  session: SessionContext;  // L3
  retrieved: {
    similarStories: Story[];
    relevantFeedback: FeedbackPattern[];
  };
}

async function assembleContext(
  sessionId: string,
  questionId: string
): Promise<ContextBundle> {
  const session = await PracticeSession.findByPk(sessionId, {
    include: [{ model: Interview, include: [Opportunity] }],
  });

  const userId = session.userId;
  const opportunity = session.interview?.opportunity;

  // 1. Fetch L1: User Context
  const userContext = await UserContext.findOne({ where: { userId } });
  const userStories = await Story.findAll({ where: { userId, isArchived: false } });

  // 2. Fetch L2: Opportunity Context (if available)
  const opportunityContext = opportunity ? {
    jobDescription: opportunity.jobDescription,
    companyResearch: opportunity.companyResearch,
    companyValues: opportunity.companyValues,
    curatedStories: await Story.findAll({
      where: { id: opportunity.curatedStoryIds },
    }),
  } : null;

  // 3. Fetch L3: Session Context
  const currentQuestion = await Question.findByPk(questionId);
  const previousAnswers = await PracticeQuestion.findAll({
    where: { sessionId },
    order: [['order', 'ASC']],
  });

  // 4. Vector retrieval: Similar stories for this question
  const similarStories = await findSimilarStories(userId, currentQuestion.text, 3);

  // 5. Vector retrieval: Relevant feedback patterns
  const relevantFeedback = userContext?.feedbackPatterns?.filter(p =>
    p.area.toLowerCase().includes(currentQuestion.category)
  ) || [];

  return {
    user: {
      ...userContext,
      stories: userStories,
    },
    opportunity: opportunityContext,
    session: {
      type: session.type,
      interviewType: session.interview?.type,
      currentQuestion,
      previousAnswers,
    },
    retrieved: {
      similarStories,
      relevantFeedback,
    },
  };
}
```

---

### Phase 5: Promptsmith Integration

**Goal:** Convert context bundles into optimized prompts.

**Promptsmith is a Fractal OS service** that manages prompt templates, versioning, and A/B testing.

**PI Backend → Promptsmith:**

```typescript
// Call Promptsmith to build the final prompt
const prompt = await promptsmith.build('answer_feedback_v1', {
  // L1 context
  userStrengths: context.user.strengthAreas,
  pastFeedbackPatterns: context.user.feedbackPatterns,
  userStories: context.user.stories.map(s => s.title),

  // L2 context
  companyValues: context.opportunity?.companyValues,
  jobRequirements: extractRequirements(context.opportunity?.jobDescription),

  // L3 context
  question: context.session.currentQuestion.text,
  questionType: context.session.currentQuestion.category,
  answerTranscript: userAnswer,

  // Retrieved context
  similarStories: context.retrieved.similarStories.map(s => ({
    title: s.title,
    situation: s.situation,
    result: s.result,
  })),
});

// Send to Claude
const feedback = await claude.complete(prompt);
```

**Promptsmith Templates (managed in Fractal OS):**

| Template | Purpose | Context Used |
|----------|---------|--------------|
| `answer_feedback_v1` | Evaluate practice answer | L1 patterns, L2 values, L3 transcript |
| `interview_plan_v1` | Generate prep plan | L1 weak areas, L2 requirements, days until |
| `story_suggestion_v1` | Surface relevant stories | L1 stories, L2 requirements, L3 question |
| `ideal_answer_v1` | Generate example answer | L1 user stories, L2 company, L3 question |
| `feedback_aggregation_v1` | Extract patterns from feedback | Raw feedback array |

---

## Data Flow Diagrams

### Practice Session with Context

```
User starts practice session
         │
         ▼
┌─────────────────────────────────────────┐
│ PI Backend: Create PracticeSession      │
│ - Link to Interview (optional)          │
│ - Select questions                      │
└────────────────────┬────────────────────┘
                     │
         ┌───────────┴───────────┐
         ▼                       ▼
┌─────────────────┐    ┌─────────────────────┐
│ Pre-load L1+L2  │    │ Return question list │
│ (hybrid eager)  │    │ to frontend          │
└────────┬────────┘    └─────────────────────┘
         │
         ▼
User answers question
         │
         ▼
┌─────────────────────────────────────────┐
│ PI Backend: Process answer              │
│ 1. Transcribe audio (if applicable)     │
│ 2. Assemble context (L1+L2+L3)          │
│ 3. Vector retrieve similar stories      │
└────────────────────┬────────────────────┘
                     │
                     ▼
┌─────────────────────────────────────────┐
│ Fractal OS: Promptsmith                 │
│ - Build prompt from template            │
│ - Include all context layers            │
└────────────────────┬────────────────────┘
                     │
                     ▼
┌─────────────────────────────────────────┐
│ Claude API: Generate feedback           │
└────────────────────┬────────────────────┘
                     │
                     ▼
┌─────────────────────────────────────────┐
│ PI Backend: Store feedback              │
│ - Save to practice_questions            │
│ - Queue user_context re-aggregation     │
└─────────────────────────────────────────┘
```

### UserContext Aggregation

```
Practice session completes
         │
         ▼
┌─────────────────────────────────────────┐
│ Queue job: aggregate_user_context       │
│ (async, non-blocking)                   │
└────────────────────┬────────────────────┘
                     │
                     ▼
┌─────────────────────────────────────────┐
│ PI Backend: Fetch recent feedback       │
│ - Last 50 sessions or 30 days           │
└────────────────────┬────────────────────┘
                     │
                     ▼
┌─────────────────────────────────────────┐
│ Fractal OS: Extract patterns            │
│ - Identify recurring themes             │
│ - Compute strength/improvement areas    │
└────────────────────┬────────────────────┘
                     │
                     ▼
┌─────────────────────────────────────────┐
│ PI Backend: Update user_context         │
│ - Store aggregated patterns             │
│ - Update timestamp                      │
└─────────────────────────────────────────┘
```

---

## Implementation Timeline

| Phase | Scope | Effort | Dependencies |
|-------|-------|--------|--------------|
| **Phase 1** | Database schema extensions | 2-3 days | pgvector extension on Heroku |
| **Phase 2** | UserContext aggregation | 3-4 days | Phase 1, Fractal OS endpoint |
| **Phase 3** | Vector embeddings pipeline | 3-4 days | Phase 1, OpenAI embeddings API |
| **Phase 4** | Context assembly service | 2-3 days | Phases 2, 3 |
| **Phase 5** | Promptsmith integration | 3-4 days | Phase 4, Promptsmith templates |

**Total estimated effort:** 13-18 days

**Critical path:** Phase 1 → Phase 2/3 (parallel) → Phase 4 → Phase 5

---

## Open Questions

1. **pgvector vs external vector DB?**
   - Recommendation: Start with pgvector. Simpler, all data in one place.
   - Revisit if we need cross-tenant search or scale beyond 100k embeddings.

2. **Async aggregation queue?**
   - Options: Bull (Redis), pg-boss (Postgres), or simple setTimeout
   - Recommendation: pg-boss keeps us single-database.

3. **Promptsmith hosting?**
   - Where does Promptsmith run? Existing Fractal OS infra or new service?
   - Need to define the API contract.

4. **Embedding costs?**
   - OpenAI embeddings: ~$0.02 per 1M tokens
   - At 100 stories/user, 10k users = 1M stories = ~$1-2 total
   - Negligible cost.

---

## Success Metrics

| Metric | Baseline | Target |
|--------|----------|--------|
| Feedback relevance (user rating) | TBD | 4.5/5 |
| Story suggestion accuracy | 0% (not implemented) | 70% helpful |
| Average response latency | TBD | <3 seconds |
| Context token efficiency | N/A | <4000 tokens/prompt |

---

## Next Steps

1. **Matt/Max:** Review and approve this plan
2. **Ruk:** Create GitHub issues for each phase
3. **Max:** Enable pgvector on Heroku Postgres
4. **Team:** Define Promptsmith API contract

---

*This specification represents the implementation plan for context engineering in Practice Interviews.*

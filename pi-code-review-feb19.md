# Practice Interviews Code Review

**Date**: February 19, 2026  
**Reviewer**: Ruk  
**Scope**: Architectural issues, documentation gaps, refactoring opportunities

---

## Executive Summary

The codebase is **well-architected at the macro level** — clean separation between frontend/proxy/backend, type-safe with TypeScript, and proper ORM usage. The RBAC implementation is particularly solid. However, there are several areas needing attention before scaling further.

**Key Findings:**
- 🔴 **Critical**: Epic E2 documentation is empty (2 lines)
- 🟠 **High**: Several large files violate 300-line guideline
- 🟠 **High**: SQL injection vulnerability in `getSimpleAnalytics`
- 🟡 **Medium**: Duplicate V1/V2 code patterns need consolidation
- 🟢 **Good**: RBAC system is clean and maintainable

---

## 1. Documentation Issues

### 🔴 Epic E2: Empty Documentation
**File**: `docs/epics/E2-users-answer-a-question-when-onboarding.md`

```markdown
# Users answer a question when onboarding

Add your documentation here.
```

**Impact**: No specification for "structured practice sessions" feature. Impossible for anyone to implement or review against requirements.

**Recommendation**: Either document the feature properly or remove the file if not yet designed.

### ✅ Epic E8: Well Documented
The Analytics upgrade plan is comprehensive with 3 phases, file locations, and success metrics. This is the gold standard — E2 should match this quality.

---

## 2. Security Issue

### 🔴 SQL Injection in Analytics Service
**File**: `practice-interviews-backend/src/service/analytics.service.ts:265-286`

```typescript
export const getSimpleAnalytics = async (userId: string): Promise<any> => {
  const query = `
    SELECT *
    FROM (...)
    WHERE "userId" = '${userId}';  // <-- Direct string interpolation!
  `;
  const [results]: Array<any> = await sequelize.query(query);
```

**Issue**: `userId` is directly interpolated into raw SQL without parameterization. While `userId` comes from JWT and is likely a UUID, this violates the codebase's own principle of ORM-only access.

**Fix**: Use Sequelize's parameterized queries:
```typescript
const [results] = await sequelize.query(query, {
  replacements: { userId },
  type: QueryTypes.SELECT
});
// And change: WHERE "userId" = :userId
```

---

## 3. Large Files Needing Refactoring

### useSubmitRecording.ts (722 LOC)
**File**: `practice-interviews-web/src/middleware/useSubmitRecording.ts`

**Issues**:
- 24 pieces of useState for a single hook
- Two massive functions: `getAnswerById` (~318 LOC) and `submitRecording` (~268 LOC)
- Mixes SSE handling, state management, and error handling

**Recommendation**: Extract into focused hooks:
```
useSubmitRecording.ts → 
├── useRecordingSubmit.ts     (SSE submission logic)
├── useAnswerFetch.ts         (GET answer by ID + polling)
├── useRecordingMetrics.ts    (metric state: wordCount, filler words, etc.)
└── useSubmitRecording.ts     (orchestrator, ~100 LOC)
```

### answerV2.controller.ts (782 LOC)
**File**: `practice-interviews-backend/src/controllers/answerV2.controller.ts`

**Strengths**: Good helper function extraction (computeMetrics, transcribeMedia, handleTMAYSideEffect)

**Issues**: 
- Three exported controllers that share ~60% logic
- `generateAndSaveFeedback` duplicated between text and media handlers

**Recommendation**: Already well-factored with helper functions! The main improvement would be extracting `handleMediaAnswerV2` into a service since it's essentially business logic.

---

## 4. RBAC Implementation: ✅ Clean

**File**: `practice-interviews-backend/src/utils/access-control.ts` (240 LOC)

This is excellent:
- Type-safe with `satisfies` to ensure all tiers are covered
- Clean tier derivation logic with fallbacks
- License expiration handling for B2B
- DRY with `enablePremiumLearning` helper

**One suggestion**: The `roleTierMap` has duplicate entries (`superadmin`/`super_admin`, `b2b_admin`/`b2badmin`). Consider normalizing input once and using a single canonical key:
```typescript
const normalizedRole = roleName?.toLowerCase().replace(/[\s_]/g, '');
```

---

## 5. Analytics Service: Performance Concern

**File**: `practice-interviews-backend/src/service/analytics.service.ts`

**Issue**: `getAnalytics()` fetches ALL users' answers to calculate leaderboard (line 84):
```typescript
const answers = await Answer.findAll({
  where: makeWhereQuery(filter), // No userId filter = ALL users
```

For 1000 users × 50 answers each = 50,000 rows loaded into memory per analytics request.

**Recommendations**:
1. Pre-compute leaderboard scores in a scheduled job
2. Cache leaderboard for 5-15 minutes
3. At minimum, add pagination to prevent OOM

---

## 6. Code Duplication: V1/V2 Pattern

**Affected files**:
- `answer.route.ts` + `answerV2.route.ts`
- `rating.route.ts` + `ratingV2.route.ts` 
- `useSubmitRecording.ts` + `useSubmitRecordingV2.ts`

**Observation**: V2 is clearly the active path (PromptSmith integration). V1 appears to be kept for rollback safety.

**Recommendation**: 
- If V1 is deprecated, add explicit deprecation comments and timeline
- If V1 is still needed, extract shared logic into base classes/utilities
- Document the versioning strategy in ARCHITECTURE.md

---

## 7. Console.log Usage

**File**: `analytics.service.ts:94`
```typescript
console.log(`❓ total Answers from ${moment(...)}:`, answers.length);
```

**Issue**: Should use the logger (`import { logger } from '../lib/logger'`) for consistency and structured logging.

---

## 8. In-Memory Cache Risk

**File**: `src/utils/cache.ts` and `src/service/in-memory-cache.service.ts`

**Issue**: Node-cache is per-instance. If you deploy multiple dynos/containers, each has its own cache → stale data and inconsistent user experiences.

**Impact**: Low for current scale, but will bite you during horizontal scaling.

**Recommendation**: Document as known limitation OR migrate to Redis when you need multiple instances.

---

## Summary Table

| Priority | Issue | Location | Action |
|----------|-------|----------|--------|
| 🔴 Critical | Empty E2 documentation | `docs/epics/E2-*` | Document or remove |
| 🔴 Critical | SQL injection | `analytics.service.ts:284` | Use parameterized query |
| 🟠 High | Large hooks | `useSubmitRecording.ts` | Split into focused hooks |
| 🟠 High | Leaderboard performance | `analytics.service.ts` | Cache or pre-compute |
| 🟡 Medium | V1/V2 duplication | Various | Document strategy, extract shared code |
| 🟡 Medium | Console.log usage | `analytics.service.ts` | Use logger |
| 🟢 Low | In-memory cache | `cache.ts` | Document limitation |

---

## What's Working Well

1. **Type Safety**: Full TypeScript, explicit interfaces, Joi validation
2. **RBAC**: Clean, maintainable, type-checked tier/feature matrix
3. **Service Layer**: Good separation of concerns in most services
4. **E8 Documentation**: Excellent example of how features should be documented
5. **Fire-and-forget Pattern**: Background feedback generation is well-implemented
6. **Error Handling**: Consistent use of Sentry + structured logging

---

*Review complete. Let me know if you want me to dig deeper into any specific area.*

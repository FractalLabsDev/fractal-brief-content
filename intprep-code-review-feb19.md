# Interview Prep Code Review

**Date**: February 19, 2026  
**Reviewer**: Ruk  
**Repos**: `interview-prep-backend` + `interview-prep-web`

---

## Executive Summary

Both repos demonstrate **strong architecture** with excellent separation of concerns. The codebase is well-structured for an early-stage product. Overall grade: **A-/B+**.

**Key Findings:**
- 🟠 **High**: 3 frontend pages are bloated (800, 723, 624 LOC)
- 🟠 **High**: 10 database entities exist in migrations but have no models/services
- 🟡 **Medium**: Error handling duplicated across all route files
- 🟡 **Medium**: Missing query parameter validation in routes
- ✅ **Good**: Clean architecture, strong TypeScript, no SQL injection risks

---

## 1. Large Files Needing Refactoring

### Frontend (Critical)

| File | LOC | Recommendation |
|------|-----|----------------|
| `OpportunityDetailPage.tsx` | 800 | Extract into 5 tab components |
| `ProfilePage.tsx` | 723 | Extract into 5 section components |
| `QuestionsStep.tsx` | 624 | Extract audio recorder, feedback display |

**Impact**: Extracting these would reduce ~2,100 LOC to ~850 LOC (60% reduction)

### Backend (Moderate)

| File | LOC | Issue |
|------|-----|-------|
| `practice.ts` (routes) | 306 | Error handling duplication |
| `academy.ts` (routes) | 214 | Error handling duplication |
| `stories.ts` (routes) | 203 | Error handling duplication |

**Fix**: Create `asyncHandler` middleware to wrap all route handlers:
```typescript
// middleware/asyncHandler.ts
export const asyncHandler = (fn: AsyncRequestHandler) => 
  (req: Request, res: Response, next: NextFunction) => {
    Promise.resolve(fn(req, res, next)).catch(next);
  };
```

This would reduce routes by ~400 LOC total.

---

## 2. Missing Entity Implementations

**10 entities exist in migrations but have no model/service/routes:**

| Priority | Entity | Impact |
|----------|--------|--------|
| 🔴 High | Plan, Section, Task | Blocks interview prep workflows |
| 🔴 High | LearningResource | Interview resources feature |
| 🟡 Medium | StructuredPracticeTemplate | Reusable templates |
| 🟡 Medium | TemplateQuestion | Questions in templates |
| 🟢 Low | TargetCompany, TargetRole, DataSource | Reference data |
| ⚪ Skip | StoryQuestionLink | N:M helper (DB-only is fine) |

**Recommendation**: Implement Plan/Section/Task/LearningResource in next sprint to unblock interview prep features.

---

## 3. Security Notes

### ✅ Good
- Sequelize ORM with parameterized queries (no SQL injection)
- bcryptjs with cost factor 12
- JWT tokens with expiry
- Helmet.js headers configured
- Joi validation on all req.body

### ⚠️ Needs Fix
**Query parameters not validated:**
```typescript
// practice.ts line 48 - type assertion, no validation
const filters = {
  status: req.query.status as PracticeSessionStatus,  // ⚠️
  type: req.query.type as PracticeSessionType,        // ⚠️
};
```

**Fix**: Add Joi validation for query params:
```typescript
const querySchema = Joi.object({
  status: Joi.string().valid('in_progress', 'completed', 'abandoned'),
  type: Joi.string().valid('quick', 'structured'),
  limit: Joi.number().integer().min(1).max(100),
});
```

---

## 4. Documentation Gaps

### Backend
- `API.md` is incomplete (summary format only)
- Missing: Full request/response examples, error codes, rate limits

### Frontend  
- `COMPONENTS.md` only covers Button & Input (80 lines)
- Missing: Full component inventory (22+ components exist)

### Both
- No `DEPLOYMENT.md` or `SECURITY.md`

---

## 5. Code Duplication Patterns

### Backend Error Handling (~400 LOC duplicated)
Same try-catch pattern repeated 30+ times across routes.

### Backend CRUD Services (~200 LOC duplicated)
`findById`, `update`, `delete` patterns repeated in 6 services.
**Fix**: Create `BaseService<T>` class.

### Frontend Tab Patterns (~300 LOC duplicated)
Similar tab structures in OpportunityDetailPage, ProfilePage, Academy pages.
**Fix**: Create reusable `<TabbedPage>` component.

---

## 6. What's Working Well

1. **Architecture**: Clean service-to-route mapping (1:1 pattern)
2. **Type Safety**: Comprehensive TypeScript coverage
3. **State Management**: Good separation (TanStack Query for server, Zustand for client)
4. **Entity Models**: Well-defined with proper interfaces (`ENTITIES.md` is excellent)
5. **Auth**: JWT + refresh token rotation properly implemented
6. **Testing Structure**: Vitest + Playwright configured

---

## 7. Refactoring Priority

### This Week
1. Create `asyncHandler` middleware (2-3 hours)
2. Add query parameter validation (2 hours)
3. Implement Plan/Section/Task models (1 day)

### Week 2
1. Extract OpportunityDetailPage → 5 components (4-5 hours)
2. Expand API.md with examples (4 hours)

### Week 3-4
1. Extract ProfilePage + QuestionsStep
2. Create BaseService class
3. Expand COMPONENTS.md

---

## Now About Those PRs...

**Backend PR #4** and **Frontend PR #5** that Matt linked appear to address issues I found in the **old** practice-interviews codebase, not interview-prep. 

If these are intended for interview-prep, the fixes look appropriate:
- PR #4: asyncHandler pattern would work here too
- PR #5: Hook extraction pattern matches what I'm recommending

Let me know if you want me to review them specifically against interview-prep, or if there are different PRs for this repo.

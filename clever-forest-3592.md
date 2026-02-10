# Practice Interviews Infinite Rerender Analysis

## What Happened (From Max's Loom)

Max discovered the "hacker attack" was actually an **infinite rerender bug** in the onboarding flow:

- A `useEffect` dependency array bug created a circular dependency: variable changes → triggers effect → effect changes variable → infinite loop
- The first unlucky user triggered onboarding, saw it stuck at 90% (Wordware was out of credits), and **left the tab open for 6 hours**
- Without rate limiting at the time, this single user's loop sent **13,000+ requests** and crashed the server for everyone
- The regression was introduced when "some parameter changed" in onboarding — developers typically skip onboarding when self-testing, so it went unnoticed

**Current Status:**
- ✅ Rate limiting now implemented (limits damage to the triggering user only)
- ✅ Server-side deduplication (rejects identical answers)
- ✅ Frontend fix deployed to staging
- ⏳ Production fix planned for Monday

---

## Codebase Vulnerabilities Found

I explored the Practice Interviews codebase and found several high-risk patterns:

### 1. `useSubmitRecording.ts` — Missing Setter Dependencies
```typescript
useCallback(
  async ({ jobId, file, type, interviewAnswer, questionId }) => {
    // 200+ lines calling setTranscription, setFeedback, etc.
    await fetchEventSource(apiUrl, { ... });
  },
  [setTranscription, setIsLoading, setFeedback, setMessage, token]
  // MISSING: setAnswerId, setS3Url, setModularFeedbacks, setScores
);
```
If `token` changes during streaming, could trigger duplicate API calls.

### 2. `AnswerAnalysis.tsx` — Brittle State Sync
```typescript
useEffect(() => {
  if (forceFeedback && (!answerAnalysis?.feedback || answerAnalysis.feedback !== forceFeedback)) {
    setAnswerAnalysis(prev => ({ ...prev, feedback: forceFeedback }));
  }
}, [forceFeedback, answerAnalysis, setAnswerAnalysis]); // Object reference instability
```
Depending on the entire `answerAnalysis` object can cause spurious re-runs.

### 3. No Error Boundaries
**No `ErrorBoundary` components found in the codebase.** React Query has `retry: 0`, but there's no catch-all for component render errors.

### 4. SSE Stream Handling Gaps
- No timeout on streams
- No abort handler for navigation away
- Malformed Wordware chunks accumulate in `garbageChunk` unbounded
- If stream hangs, user retries → new stream starts → cascading failures

---

## Recommended Solutions

### Tier 1: Immediate / Low Effort

**1. Add `why-did-you-render` in development**
```bash
npm install @welldone-software/why-did-you-render --save-dev
```
```typescript
// src/wdyr.ts (import at top of index.tsx)
import React from 'react';
if (process.env.NODE_ENV === 'development') {
  const whyDidYouRender = require('@welldone-software/why-did-you-render');
  whyDidYouRender(React, { trackAllPureComponents: true });
}
```
Logs every avoidable rerender to console. Would have caught this bug immediately.

**2. Add React Error Boundaries**
```typescript
// Wrap key routes/components
<ErrorBoundary fallback={<CrashScreen />}>
  <OnboardingFlow />
</ErrorBoundary>
```
Prevents a single component crash from taking down the whole app.

**3. Request Deduplication Middleware**
Before any API call, check if an identical request is already in-flight:
```typescript
const pendingRequests = new Map();

async function deduplicatedFetch(key: string, fetchFn: () => Promise<any>) {
  if (pendingRequests.has(key)) return pendingRequests.get(key);
  const promise = fetchFn().finally(() => pendingRequests.delete(key));
  pendingRequests.set(key, promise);
  return promise;
}
```

### Tier 2: Medium Effort

**4. Circuit Breaker on Wordware Calls**
Use [opossum](https://www.npmjs.com/package/opossum) for Node.js:
```typescript
const CircuitBreaker = require('opossum');

const wordwareBreaker = new CircuitBreaker(callWordware, {
  timeout: 30000,           // 30s timeout
  errorThresholdPercentage: 50,
  resetTimeout: 30000       // Try again after 30s
});

wordwareBreaker.fallback(() => ({ error: 'AI temporarily unavailable' }));
```
When Wordware is failing, **stop sending requests** instead of hammering it.

**5. Client-Side Request Throttling**
Prevent users from firing >N requests per minute from the browser:
```typescript
// In useSubmitRecording
const lastSubmit = useRef<number>(0);

const submitRecording = useCallback(async (...) => {
  const now = Date.now();
  if (now - lastSubmit.current < 5000) {
    console.warn('Throttled: too many submissions');
    return;
  }
  lastSubmit.current = now;
  // ... actual submission
}, [...]);
```

**6. Enable Sentry Component Tracking**
You already have Sentry — enable the React component profiler:
```typescript
import { withProfiler } from '@sentry/react';

export default withProfiler(OnboardingFlow);
```
This will surface components with "slow mounts" or "frequent updates" in Sentry's Performance dashboard.

### Tier 3: Architectural / Higher Effort

**7. Custom ESLint Rule for Circular Dependencies**
ESLint's `react-hooks/exhaustive-deps` warns about missing deps, but not circular ones. You could write a custom rule or use static analysis to detect patterns where a setter appears both in the effect body AND dependency array.

**8. Runtime Render Budget (react-state-basis)**
A newer library that tracks when state updates happen and flags infinite loops:
```bash
npm install react-state-basis --save-dev
```
Provides console forensics with location info for identified issues.

**9. SSE Abort + Timeout**
Modify `useSubmitRecording` to use AbortController with timeouts:
```typescript
const controller = new AbortController();
const timeout = setTimeout(() => controller.abort(), 60000);

await fetchEventSource(url, {
  signal: controller.signal,
  onclose: () => clearTimeout(timeout),
  onerror: (err) => { clearTimeout(timeout); throw err; }
});
```

---

## Summary Table

| Solution | Effort | Impact | Catches Future Bugs? |
|----------|--------|--------|---------------------|
| why-did-you-render | Low | Dev-time detection | ✅ |
| Error Boundaries | Low | Graceful degradation | Partial |
| Request deduplication | Low | Prevents duplicate calls | ✅ |
| Circuit breaker (opossum) | Medium | Protects downstream services | ✅ |
| Client throttling | Medium | User-side protection | ✅ |
| Sentry component tracking | Medium | Production visibility | ✅ |
| Custom ESLint rule | High | Static analysis | ✅ |
| react-state-basis | Medium | Runtime loop detection | ✅ |
| SSE abort/timeout | Medium | Stream resilience | Partial |

---

## Clarifying Questions

1. **Testing coverage**: Does the onboarding flow have any E2E tests (Cypress/Playwright)? This regression suggests the happy path isn't being tested on every deploy.

2. **Wordware credit alerts**: Does the team get notified when credits are low/exhausted? The user got stuck at 90% because Wordware was out of credits — seems like that should trigger an alert before it affects users.

3. **Deployment cadence**: Is there appetite for a staged rollout (staging → canary → production) with automated rollback if error rates spike? This would catch regressions before they hit all users.

4. **Rate limit visibility**: Max mentioned he can't trace which user is hitting `API v2 prices` repeatedly. Is the IP→user correlation worth investigating, or is that a separate issue?

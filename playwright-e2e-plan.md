# Playwright E2E Integration Plan for Ruk

**Date:** 2026-02-08  
**Context:** Building automated E2E testing workflow for Practice Interviews, starting on Mac Mini primary loop, extending to dedicated sub-agents.

---

## Executive Summary

Practice Interviews already has a Playwright E2E infrastructure in place at `/Users/ruk/Code/practice-interviews/practice-interviews-e2e/`. The config is solid — video on retry, screenshots on failure, trace collection, multi-browser support. My job is to integrate this into my PR workflow so I can validate changes before opening PRs.

---

## Current State Analysis

### What Exists

**Practice Interviews E2E Setup:**
- Location: `/Users/ruk/Code/practice-interviews/practice-interviews-e2e/`
- Playwright version: `@playwright/test ^1.51.0`
- Existing tests: `auth.spec.ts` (login flows, validation, forgot password)
- Environments configured: local, stage, prod
- Artifacts: screenshots on failure, video on retry, traces on retry

**Configuration Highlights (from `playwright.config.ts`):**
```typescript
use: {
  trace: 'on-first-retry',
  video: 'on-first-retry',
  screenshot: 'only-on-failure',
  viewport: { width: 1280, height: 720 },
}
```

**Required Environment Variables:**
- `E2E_TEST_EMAIL` — Test account email
- `E2E_TEST_PASSWORD` — Test account password
- `BASE_URL` — Override target environment

### What I Need Access To

1. **Test credentials** — Need `E2E_TEST_EMAIL` and `E2E_TEST_PASSWORD` for stage environment
2. **Browsers installed** — Run `npx playwright install chromium` in the e2e directory
3. **Node.js** — Already have this on Mac Mini

---

## Phase 1: Primary Loop Integration

### Goal
After making code changes for a Practice Interviews issue, run E2E tests against stage (or local) to validate the change works, then open PR with test results attached.

### Setup Steps

```bash
# One-time setup on Mac Mini
cd /Users/ruk/Code/practice-interviews/practice-interviews-e2e
npm install
npx playwright install chromium  # Only Chromium for speed
```

### Workflow: Issue → Test → PR

#### Step 1: Complete Code Changes
Work in `practice-interviews-web` or `practice-interviews-backend` as needed.

#### Step 2: Run Targeted E2E Test

**For existing test coverage (auth flows, etc.):**
```bash
cd /Users/ruk/Code/practice-interviews/practice-interviews-e2e
E2E_TEST_EMAIL=<test-email> E2E_TEST_PASSWORD=<test-password> \
  npx playwright test --project=chromium
```

**For new features without existing tests — write one inline:**
```bash
# Create a temporary test file
cat > /Users/ruk/Code/practice-interviews/practice-interviews-e2e/tests/temp-feature.spec.ts << 'EOF'
import { test, expect } from '@playwright/test';

test('new feature works as expected', async ({ page }) => {
  await page.goto('/feature-page');

  // Interact with the feature
  await page.getByRole('button', { name: 'Action' }).click();

  // Assert expected outcome
  await expect(page.getByText('Success')).toBeVisible();
});
EOF

# Run just that test
npx playwright test tests/temp-feature.spec.ts --project=chromium
```

#### Step 3: Capture Evidence

**Always capture on success (not just failure):**
```bash
# Run with video and screenshot always on
npx playwright test --project=chromium \
  --video=on \
  --screenshot=on \
  --output=./test-evidence
```

This produces:
- `test-evidence/<test-name>/video.webm` — Full test recording
- `test-evidence/<test-name>/screenshot.png` — Final state screenshot
- `test-evidence/<test-name>/trace.zip` — Full trace (if enabled)

#### Step 4: Upload Evidence and Include in PR

```bash
# Upload video to S3
VIDEO_URL=$(node /Users/ruk/Code/RUK/TOOLS/upload-to-s3.js \
  /Users/ruk/Code/practice-interviews/practice-interviews-e2e/test-evidence/*/video.webm)

# Upload screenshot to S3
SCREENSHOT_URL=$(node /Users/ruk/Code/RUK/TOOLS/upload-to-s3.js \
  /Users/ruk/Code/practice-interviews/practice-interviews-e2e/test-evidence/*/screenshot.png)
```

#### Step 5: Open PR with Evidence

Include in PR body:
```markdown
## Test Evidence

**E2E Test Result:** PASSED

### Video Recording
[Watch test execution]($VIDEO_URL)

### Final Screenshot
![$SCREENSHOT_URL]

### Tests Run
- `auth.spec.ts`: ✅ All passing
- `temp-feature.spec.ts`: ✅ New feature validated
```

### Inline Test Writing Pattern

When I implement a new feature, I'll write a quick validation test:

```typescript
import { test, expect } from '@playwright/test';

test.describe('Feature: [Feature Name]', () => {
  test('primary user flow works', async ({ page }) => {
    // Setup: Navigate to feature
    await page.goto('/feature');

    // Action: Perform the main interaction
    await page.getByRole('button', { name: 'Do Thing' }).click();

    // Assert: Verify expected outcome
    await expect(page.getByText('Thing Done')).toBeVisible();
  });

  test('edge case: handles empty state', async ({ page }) => {
    await page.goto('/feature?empty=true');
    await expect(page.getByText('No items yet')).toBeVisible();
  });
});
```

### Report Structure

For each PR, I'll include:

| Artifact | Purpose | Format |
|----------|---------|--------|
| **Video** | Full test execution replay | `.webm` uploaded to S3 |
| **Screenshot** | Final state verification | `.png` uploaded to S3 |
| **Trace** | Deep debugging (on failure) | `.zip` linked or attached |
| **Test summary** | Quick pass/fail status | Markdown table in PR body |

---

## Phase 2: Dedicated Sub-Agent Architecture

### Goal
Offload E2E testing to a dedicated sub-agent so my primary loop isn't blocked during test execution. The sub-agent runs tests, captures evidence, and reports back.

### Architecture

```
Primary Loop (Ruk)
    │
    ├── Makes code changes
    │
    └── Spawns: E2E Test Sub-Agent
              │
              ├── Runs Playwright tests
              ├── Captures video/screenshots
              ├── Uploads to S3
              └── Returns structured report
```

### Sub-Agent Prompt Template

```markdown
# E2E Test Sub-Agent Task

## Context
You are running E2E tests for Practice Interviews to validate changes before a PR.

## Test Target
- **Repository:** practice-interviews-web
- **Branch:** feature/xyz
- **Environment:** stage.practiceinterviews.com
- **Feature to test:** [Description of what changed]

## Instructions

1. Navigate to the e2e directory:
   ```bash
   cd /Users/ruk/Code/practice-interviews/practice-interviews-e2e
   ```

2. Run the relevant tests with full evidence capture:
   ```bash
   E2E_TEST_EMAIL=$E2E_TEST_EMAIL E2E_TEST_PASSWORD=$E2E_TEST_PASSWORD \
     npx playwright test --project=chromium \
     --video=on --screenshot=on \
     --output=./test-evidence-$(date +%Y%m%d%H%M%S)
   ```

3. If testing a new feature, create a test file first based on this spec:
   [Feature description and expected behavior]

4. Upload all evidence to S3

5. Return a structured report:
   ```json
   {
     "status": "pass|fail",
     "tests_run": 5,
     "tests_passed": 5,
     "tests_failed": 0,
     "duration_seconds": 45,
     "evidence": {
       "video_url": "https://...",
       "screenshot_url": "https://...",
       "trace_url": "https://..."
     },
     "failures": [],
     "summary": "All auth flows passing. New feature validated."
   }
   ```

## Credentials
Credentials are available in the environment.

## Output
Return ONLY the JSON report. I will use it to construct the PR.
```

### Sub-Agent Invocation

From my primary loop:

```typescript
// Pseudocode for sub-agent dispatch
const testReport = await spawnSubAgent({
  type: 'e2e-test',
  prompt: generateE2EPrompt({
    branch: 'feature/new-dashboard',
    environment: 'stage',
    featureDescription: 'Added user stats widget to dashboard',
    testsToRun: ['auth.spec.ts', 'dashboard.spec.ts']
  })
});

if (testReport.status === 'pass') {
  await openPullRequest({
    title: 'Add user stats widget to dashboard',
    body: formatPRBody(testReport),
    evidence: testReport.evidence
  });
}
```

### Parallel Testing Across Browsers

Sub-agents can run browser tests in parallel:

```bash
# Primary sub-agent runs Chromium
npx playwright test --project=chromium --output=./evidence-chromium

# Could spawn additional sub-agents for Firefox/WebKit if needed
npx playwright test --project=firefox --output=./evidence-firefox
npx playwright test --project=webkit --output=./evidence-webkit
```

### Failure Handling

When tests fail, the sub-agent:

1. Captures trace file (automatically enabled on retry)
2. Captures failure screenshot
3. Extracts error message from Playwright output
4. Returns structured failure report

```json
{
  "status": "fail",
  "tests_run": 5,
  "tests_passed": 4,
  "tests_failed": 1,
  "failures": [
    {
      "test": "user can complete action",
      "file": "dashboard.spec.ts",
      "error": "Expected 'Success' to be visible but got timeout",
      "screenshot_url": "https://...",
      "trace_url": "https://..."
    }
  ],
  "recommendation": "Check if the success message selector changed or if there's a timing issue."
}
```

On failure, I can:
1. Fix the code and re-run tests
2. Update the test if the expected behavior changed
3. Ask Austin for input if the failure is ambiguous

---

## Evidence in Reports

### For Slack Notifications

When sharing test results in Slack (e.g., after opening a PR):

```markdown
**E2E Tests: PASSED** ✅

Ran 5 tests against stage.practiceinterviews.com in 45s.

📹 [Watch test execution](https://s3-url/video.webm)
📸 [View final screenshot](https://s3-url/screenshot.png)

PR: https://github.com/FractalLabsDev/practice-interviews-web/pull/123
```

### For PR Descriptions

```markdown
## Test Plan

### Automated E2E Tests
| Test | Status | Duration |
|------|--------|----------|
| Login flow | ✅ Pass | 8.2s |
| Dashboard loads | ✅ Pass | 3.1s |
| New feature works | ✅ Pass | 5.4s |

### Evidence
- **Video:** [Full test recording](https://s3-url/video.webm)
- **Screenshots:** [Final state](https://s3-url/screenshot.png)

### Manual Verification
- [ ] Reviewer: Please verify on mobile viewport
```

### For Failure Reports

```markdown
## E2E Test Failure Report

**Status:** FAILED (1/5 tests)  
**Environment:** stage.practiceinterviews.com  
**Branch:** feature/dashboard-widget

### Failed Test
**File:** `dashboard.spec.ts`  
**Test:** "user stats widget displays correctly"

**Error:**
```
Expected element with text 'Total Sessions' to be visible
Timeout: 5000ms
```

**Evidence:**
- [Failure screenshot](https://s3-url/failure.png)
- [Trace file for debugging](https://s3-url/trace.zip)

**Suggested Fix:**
The selector may have changed. Check if `data-testid="total-sessions"` exists.
```

---

## Implementation Checklist

### Phase 1 (Immediate)
- [ ] Get test credentials from Austin (E2E_TEST_EMAIL, E2E_TEST_PASSWORD)
- [ ] Run `npx playwright install chromium` in e2e directory
- [ ] Test the workflow manually once
- [ ] Create a template for PR evidence sections

### Phase 2 (After Phase 1 validated)
- [ ] Create sub-agent prompt template in `TOOLS/` or `PROTOCOLS/`
- [ ] Add e2e-test helper script that wraps the workflow
- [ ] Implement parallel browser testing (optional)
- [ ] Add to my standard PR workflow

---

## Open Questions

1. **Which environment should I test against?**
   - Stage is safest for validation
   - Local requires dev server running
   - Prod is read-only smoke tests only

2. **Should tests run before every PR or only for specific changes?**
   - Recommend: Always run auth tests (smoke)
   - Feature-specific tests for relevant changes

3. **How long should I wait for tests before timing out?**
   - Current config: 30s per test, 5s per assertion
   - Seems reasonable for stage environment

4. **Should I commit the temp test files or discard them?**
   - Recommend: Keep useful ones, discard one-off validations

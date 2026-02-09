# E2E Test Suite Specification

**Projects:** Practice Interviews + Vitaboom  
**Framework:** Playwright  
**Deployment Strategy:** Post-deploy hook + on-demand local execution

---

## Practice Interviews

**Existing Infrastructure:** Playwright already configured at `/practice-interviews-e2e`

### Current Coverage
- ✅ Login flow (smoke)
- ✅ Basic practice flow with text input (smoke)

### Gaps to Fill

| Suite | Priority | Est. Runtime | Description |
|-------|----------|--------------|-------------|
| `signup.spec.ts` | HIGH | 45s | Full onboarding: welcome → basic info → field/role → create account → practice |
| `custom-interview.spec.ts` | HIGH | 60s | Job categories → company → configure → confirm → start |
| `payment.spec.ts` | MEDIUM | 30s | Subscription upgrade via Stripe test mode |
| `video-input.spec.ts` | MEDIUM | 90s | Practice flow with video recording (browser permissions) |
| `analytics.spec.ts` | LOW | 20s | View past attempts, score history, answer feedback |
| `academy.spec.ts` | LOW | 30s | Academy navigation, level content loading |

### Smoke Suite (Post-Deploy)
Target: **< 2 minutes**

```
@smoke tests (already tagged):
1. auth.spec.ts - Login flow
2. practice-flow.spec.ts - Complete practice with feedback

New additions:
3. health-check.spec.ts - API connectivity, static assets loading
```

### Commands
```bash
# Post-deploy smoke (CI)
npm run test:e2e:prod -- --grep @smoke

# Full regression (nightly/on-demand)
npm run test:e2e:stage
```

---

## Vitaboom

**Status:** No E2E infrastructure exists. Need to create from scratch.

### Proposed Structure
```
vitaboom/
├── e2e/
│   ├── playwright.config.ts
│   ├── fixtures/
│   │   └── auth.ts          # Auth state fixtures
│   ├── tests/
│   │   ├── auth.spec.ts
│   │   ├── clients.spec.ts
│   │   ├── protocol.spec.ts
│   │   ├── stacks.spec.ts
│   │   └── admin.spec.ts
│   └── .env.example
└── package.json (add playwright deps)
```

### Test Suites

| Suite | Priority | Est. Runtime | Description |
|-------|----------|--------------|-------------|
| `auth.spec.ts` | CRITICAL | 30s | Login existing user, logout, session persistence |
| `clients.spec.ts` | HIGH | 45s | CRUD operations, search, filter, side panel |
| `protocol.spec.ts` | HIGH | 60s | Add products, set timing, share protocol, verify short code |
| `stacks.spec.ts` | MEDIUM | 30s | View stacks, apply to client, edit template |
| `admin.spec.ts` | LOW | 30s | Role-based access verification, settings, clinicians page |
| `signup.spec.ts` | LOW | 90s | Full practitioner signup (complex due to email verification) |

### Smoke Suite (Post-Deploy)
Target: **< 2 minutes**

```
@smoke tests:
1. Login existing user → land on /clients
2. View client list (renders without errors)
3. Navigate to catalog → add product to cart
4. Cart persists on navigation
```

### Environment Config
```bash
# e2e/.env
BASE_URL=https://stage.vitaboom.com
E2E_TEST_EMAIL=e2e-test@vitaboom.com
E2E_TEST_PASSWORD=<secure>
```

### Commands
```bash
# Post-deploy smoke
npx playwright test --grep @smoke --project chromium

# Full regression
npx playwright test

# Local development
npx playwright test --ui
```

---

## Integration with CI/CD

### Post-Deploy Hook Pattern

**Vercel (Vitaboom-web):**
```bash
# In deployment webhook or GitHub Action
if [ "$VERCEL_ENV" = "production" ]; then
  cd e2e && npx playwright test --grep @smoke
fi
```

**Heroku (PI backend):**
Already has deploy hooks configured. Add E2E trigger to existing pipeline.

### On-Demand Execution

Team can run locally or trigger via Slack command:
```
@ruk run e2e:vitaboom:smoke
@ruk run e2e:pi:full
```

---

## Test Data Strategy

**Practice Interviews:**
- Uses dedicated test account (E2E_TEST_EMAIL)
- Tests are idempotent (can re-run without cleanup)

**Vitaboom:**
- Need dedicated test clinic + test practitioner account
- Consider: API helper to reset test data between runs
- Alternative: Tests create unique data with timestamp prefix, archive after

---

## Next Steps

1. **Serhii to verify** this coverage matches critical business paths
2. **Create test accounts** for both environments (stage + prod)
3. **PI:** Add missing specs to existing e2e folder
4. **Vitaboom:** Initialize Playwright, create first auth test
5. **Wire up** post-deploy hooks in both projects

---

*Draft by Ruk | 2026-02-09*

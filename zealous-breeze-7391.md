# Weekly Digest — Week of 2026-02-16

## Practice Interviews

### What Changed This Week
- **release-2026-02-19** ([#432](https://github.com/FractalLabsDev/practice-interviews-web/pull/432)) by @ruk-fl
- **Add rate limit banner UI for 429 errors** ([#431](https://github.com/FractalLabsDev/practice-interviews-web/pull/431)) by @ruk-fl
- **Add E2E test CI workflow** ([#430](https://github.com/FractalLabsDev/practice-interviews-web/pull/430)) by @ruk-fl
- **release-2026-02-19** ([#120](https://github.com/FractalLabsDev/practice-interviews-backend/pull/120)) by @ruk-fl
- **Bugfix/revert-v3-use-v2** ([#119](https://github.com/FractalLabsDev/practice-interviews-backend/pull/119)) by @MaxDonchenko
- **Add E2E test CI workflow** ([#118](https://github.com/FractalLabsDev/practice-interviews-backend/pull/118)) by @ruk-fl
- **Add health check with database/migration verification** ([#116](https://github.com/FractalLabsDev/practice-interviews-backend/pull/116)) by @ruk-fl
- **Add V3 feedback service with all-in-one prompts** ([#115](https://github.com/FractalLabsDev/practice-interviews-backend/pull/115)) by @AustinWood

### What's In Flight
*No open PRs*

### Outstanding
- 3 bugs (subscription modal, login button, source field)
- 6 features (job descriptions, home page stats, rate setup, moment→date-fns, release notes)

---

## Therapist Genie

### What Changed This Week
- **Fix warnings preventing the proyecto to build** ([#294](https://github.com/FractalLabsDev/therapygenie/pull/294)) by @rigomm
- **Fix progress note rejection + same email for different client types** ([#293](https://github.com/FractalLabsDev/therapygenie/pull/293)) by @rigomm
- **Fix recurring appointment edit and calendar improvements** ([#292](https://github.com/FractalLabsDev/therapygenie/pull/292)) by @mattlim-fl
- **Issue/appointments** ([#288](https://github.com/FractalLabsDev/therapygenie/pull/288)) by @rigomm
- **Fix payment processing and calendar improvements** ([#118](https://github.com/FractalLabsDev/therapygenie-backend/pull/118)) by @mattlim-fl
- **Fix recurring appointment edit and calendar status** ([#116](https://github.com/FractalLabsDev/therapygenie-backend/pull/116)) by @mattlim-fl
- **Meeting transaction** ([#115](https://github.com/FractalLabsDev/therapygenie-backend/pull/115)) by @rigomm
- **Fix signature field type in client portal forms** ([#22](https://github.com/FractalLabsDev/therapist-genie-portal/pull/22)) by @ruk-fl

### What's In Flight
- Fix duplicate invoices for recurring appointments (1d) — @ruk-fl
- Merge dev into main (24d) — @rigomm
- Add client file upload API endpoints (25d) — @mattlim-fl

### Outstanding
- 2 bugs (calendar delete, Teams widget branding)
- 2 features (transcript refactor, discharge summary)

---

## VitaBoom

### What Changed This Week
- **fix: always create new protocol when sharing** ([#134](https://github.com/FractalLabsDev/vitaboom-web/pull/134)) by @ruk-fl
- **Add SuppCo product mappings** ([#120](https://github.com/FractalLabsDev/vitaboom-backend/pull/120)) by @mattlim-fl
- **Add health check with database/migration** ([#119](https://github.com/FractalLabsDev/vitaboom-backend/pull/119)) by @ruk-fl

### What's In Flight
- E2E test trigger workflow (9d) — @ruk-fl
- Playwright E2E test infrastructure (9d) — @ruk-fl
- Email validation with typo detection (11d) — @ruk-fl
- Shopify orders export API (14d) — @AustinWood
- Hide commissions toggle (18d) — @ruk-fl
- Strict vendor+title matching (42d) — @mattlim-fl
- Project-setup [DRAFT] (140d) — @MaxDonchenko

### Outstanding
- 1 bug (connection timeout)
- 3 features (Supp.co ratings, addresses migration, bulk copy protocol)

---

## Next Health

### What Changed This Week
*No PRs merged this week*

### What's In Flight
*No open PRs*

### Outstanding
- 2 bugs (upload symbols, bug tracker capture)
- 3 features (card retry prompt, rebooking, facility audit)

---

## Summary

| Project | Merged | In Flight | Outstanding |
|---------|--------|-----------|-------------|
| PI | 8 | 0 | 9 |
| TG | 8 | 3 | 4 |
| VB | 3 | 7 | 4 |
| NH | 0 | 0 | 5 |
| **Total** | **19** | **10** | **22** |

**Highlights:**
- PI: V3 feedback service shipped, E2E testing infrastructure added
- TG: Calendar/appointment fixes, payment processing improvements
- VB: Protocol sharing fix, health checks added
- NH: Quiet week

# Practice Interviews: Enterprise Licensing Model Implementation Plan

**Epic:** E-15 | **Target:** February 28, 2026 | **Owner:** Matt Lim

---

## Context

Based on discussions from the December 1st PI Dev Sync and subsequent meetings, the licensing model work has two main goals:

1. **Simplify pricing** - Move from multi-tier (Starter/Pro) to a single paid tier
2. **Prepare for enterprise** - Build infrastructure for college/university partnerships

---

## Phase 1: Pricing Simplification (Week 1-2)

### 1.1 Stop Starter Plan Sales
- [ ] Update pricing UI to remove Starter option
- [ ] Keep existing Starter users functional (don't break anything)
- [ ] Update any marketing pages referencing Starter plan

### 1.2 User Migration
- [ ] Identify all active Starter plan users (~20% of paid base)
- [ ] Create migration script to upgrade Starter → Pro in database
- [ ] Update Stripe subscriptions (be mindful of rate limits, batch carefully)
- [ ] Send communication to migrated users (via Customer.io?)

### 1.3 Backend Cleanup
- [ ] Remove Starter plan from role-based access service
- [ ] Clean up any feature flags tied to Starter/Pro distinction
- [ ] Simplify subscription status caching logic
- [ ] Remove dead code paths for Starter tier checks

---

## Phase 2: Feature Access Consolidation (Week 2-3)

### 2.1 Audit Current Feature Gates
- [ ] Map all features currently gated by tier:
  - Analytics page
  - Interview Academy
  - Ideal Answer
  - Question limits
  - Any others?
- [ ] Document which gates can be removed vs. which need "paid" check

### 2.2 Simplify Access Control
- [ ] Reduce RBAC complexity to: `free_trial | paid | admin`
- [ ] Update feature checks to binary paid/unpaid
- [ ] Test all feature access paths post-simplification

---

## Phase 3: Enterprise Foundation (Week 3-4)

### 3.1 Multi-Seat Architecture
- [ ] Design schema for organizations/teams
  - `organizations` table
  - `organization_members` junction
  - `organization_subscriptions` (separate from individual)
- [ ] Define admin roles within organizations

### 3.2 Billing Infrastructure
- [ ] Research Stripe billing for organizations (Stripe Billing, not just Subscriptions)
- [ ] Design flow for:
  - Organization creation
  - Seat management (add/remove)
  - Billing contact vs. users
- [ ] Consider: annual contracts vs. monthly per-seat

### 3.3 College-Specific Considerations
- [ ] Student data privacy (FERPA considerations?)
- [ ] Bulk user provisioning (CSV upload? SSO integration?)
- [ ] Usage reporting for admins
- [ ] White-labeling needs (if any)

---

## Phase 4: Admin Dashboard (Stretch - Post Feb 28)

### 4.1 Organization Admin Features
- [ ] View all users in organization
- [ ] Add/remove seats
- [ ] View aggregate analytics
- [ ] Manage billing

### 4.2 Internal Admin Features
- [ ] Create/manage organizations
- [ ] Set custom pricing for enterprise deals
- [ ] Override/comp accounts

---

## Technical Considerations

### Stripe Integration
- Cache subscription status in database (avoid rate limits)
- Stripe webhooks for subscription changes
- Consider Stripe Checkout for simpler payment flow

### Database Schema Changes
- Non-breaking migrations (add, don't modify)
- Backfill scripts for existing data
- Consider soft-launch: schema ready, UI gated

### Testing Strategy
- Staging environment with test Stripe keys
- Migration dry-run before production
- Rollback plan for each phase

---

## Success Criteria

**Phase 1 Complete:**
- No new Starter plans sold
- All Starter users migrated to Pro
- No revenue impact from migration

**Phase 2 Complete:**
- Feature access is binary (paid/unpaid)
- RBAC code is simpler, easier to reason about

**Phase 3 Complete:**
- Schema supports multi-seat organizations
- At least one pilot college can be onboarded
- Billing supports annual contracts

---

## Open Questions for Matt/Jeff

1. **Migration communication** - Who sends the email to Starter users? What's the message?
2. **Enterprise pricing** - Per-seat? Flat annual? Tiered by institution size?
3. **SSO priority** - Is this required for college deals, or nice-to-have?
4. **Timeline pressure** - Is Feb 28 for Phase 1-2 only, or full enterprise?

---

## Dependencies

- **E-19 (College/University Integration)** - Aug 1 target, but Phase 3 here lays groundwork
- **RBAC Cleanup (E-1)** - Already completed, but Phase 2 builds on this
- **Wordware Replacement (E-18)** - In parallel, no blockers

---

*Plan created: February 5, 2026*
*Based on: Dec 1 PI Dev Sync, Feb 4 PI Sync, Fractal OS roadmap*

# Vitaboom Dropshipping (E19) - Implementation Plan

## Context

**Epic**: E19 - Dropshipping | **Target**: Feb 28, 2026 | **Status**: In Progress

**What is Dropshipping?**
Clinics order on behalf of their clients. Products ship directly to the client's address, charged to the clinic's payment method. Clinic gets 30% discount (no commission since they're the buyer).

**Current State**:
- PR #127 has frontend UI foundation (mode toggle, checkout flow, settings)
- Backend integration not yet implemented
- Auth checks needed (clinic → own clients only)

---

## Milestones

### M1: Frontend Polish & Merge (1-2 days)
**Owner**: Matt/Frontend

- Wire `defaultCheckoutMode` from clinic settings to CartContext (currently hardcoded)
- Add phone field to shipping address form
- Add loading states and error handling
- Clean up duplicated toggle logic
- Address PR review feedback

---

### M2: Backend - Client Access Control (2-3 days)
**Owner**: Scott/Serhii

- Audit `GET /clients` endpoint - ensure clinic can only fetch their own clients
- Add clinic ownership validation to client-related endpoints
- Add test coverage for auth boundary cases

*Note: Scott confirmed this auth check is required*

---

### M3: Backend - Dropship Order API (3-4 days)
**Owner**: Scott/Serhii

- Create `POST /orders/dropship` endpoint
- Validate: clinic owns client, clinic has valid payment method
- Apply 30% clinic discount, no commission
- Integrate with Shopify order creation (direct fulfillment to client address)

**Open Questions**:
- Minimum order ($150) still apply to dropship? *(awaiting client confirmation)*
- One-time purchases for dropship, or subscription only?

---

### M4: Frontend-Backend Integration (2 days)
**Owner**: Matt + Backend

- Connect checkout modal to dropship API
- Handle API responses (success, validation errors, payment failures)
- Order confirmation with tracking info

---

### M5: Orders Dashboard & Reporting (2-3 days)
**Owner**: Frontend + Backend

- Add dropship orders to clinic's Orders tab
- Display shipping status, tracking numbers
- Filter by order type (commission vs dropship)

---

### M6: QA & Launch (2-3 days)
**Owner**: Marijoy + Team

- End-to-end QA on staging
- Edge case testing
- Deploy to production (feature flag → gradual rollout → full launch)

---

## Dependency Graph

```
M1 (Frontend) ─┬─→ M4 (Integration) → M5 (Dashboard) → M6 (Launch)
M2 (Auth) ─────┤
               └─→ M3 (API) ─────────┘
```

M1 and M2 can run in parallel. M3 depends on M2.

---

## Timeline

| Milestone | Duration | Notes |
|-----------|----------|-------|
| M1 + M2 | 2-3 days | Parallel |
| M3 | 3-4 days | After M2 |
| M4 | 2 days | After M1+M3 |
| M5 | 2-3 days | After M4 |
| M6 | 2-3 days | After M5 |

**Total**: ~12-17 days (fits Feb 28 target with buffer)

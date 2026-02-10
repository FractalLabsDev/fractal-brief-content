# Vitaboom Dropshipping (E19) - Implementation Plan

## Context

**Epic**: E19 - Dropshipping
**Status**: In Progress (UI work started)

**What is Dropshipping?**
Clinics order on behalf of their clients. Products ship directly to the client's address, charged to the clinic's payment method. Clinic gets 30% discount (no commission since they're the buyer).

**Current State**:
- PR #127 has frontend UI foundation (mode toggle, checkout flow, settings)
- Backend integration not yet implemented
- Auth checks needed (clinic → own clients only)

**Resolved Decisions**:
- One-time purchases: **Not in Phase 1** (dropship only, no changes to standard checkout)
- Shipping notifications: **Clients receive shipping emails** (pending confirmation on whether clinic also receives copy)

---

## Milestones

### M1: Frontend Polish & Merge
**Owner**: Matt/Frontend
**Deliverable**: PR #127 merged with fixes

**Tasks**:
1. Wire `defaultCheckoutMode` from clinic settings to CartContext (currently hardcoded)
2. Add phone field to shipping address form
3. Add loading states and error handling for checkout API call
4. Clean up duplicated toggle logic between components
5. Address PR review feedback

**Exit Criteria**: Dropship UI toggle works, settings persist, ready for backend integration

---

### M2: Backend - Client Access Control
**Owner**: Scott/Serhii
**Deliverable**: Secure client fetching endpoint

**Tasks**:
1. Audit `GET /clients` endpoint - ensure clinic can only fetch their own clients
2. Add clinic ownership validation to any client-related endpoints
3. Add test coverage for auth boundary cases
4. Document endpoint behavior

**Exit Criteria**: A clinic cannot see or access clients belonging to other clinics

**Note from Scott**: "If they can see clients from other clinics we'll need to fix up the endpoint used to fetch the patients"

---

### M3: Backend - Dropship Order API
**Owner**: Scott/Serhii
**Deliverable**: New endpoint to create orders on behalf of clients

**Tasks**:
1. Create `POST /orders/dropship` (or extend existing order endpoint with `mode` param)
2. Validate: clinic owns the client, clinic has valid payment method
3. Calculate pricing: apply 30% clinic discount, no commission
4. Store order with `type: dropship` and shipping address
5. Integrate with Shopify order creation (direct fulfillment to client address)
6. Create order confirmation record for clinic dashboard

**Open Questions**:
- [ ] Minimum order ($150) still apply to dropship? (Awaiting client confirmation)

**Exit Criteria**: API accepts dropship checkout, creates Shopify order, charges clinic

---

### M4: Frontend-Backend Integration
**Owner**: Matt + Backend
**Deliverable**: End-to-end dropship checkout working

**Tasks**:
1. Connect checkout modal to dropship API endpoint
2. Handle API responses (success, validation errors, payment failures)
3. Show order confirmation with tracking info
4. Update order history to display dropship orders distinctly

**Exit Criteria**: Complete flow from catalog → checkout → order confirmation

---

### M5: Orders Dashboard & Reporting
**Owner**: Frontend + Backend
**Deliverable**: Clinics can view/manage their dropship orders

**Tasks**:
1. Add dropship orders to clinic's Orders tab
2. Display shipping status, tracking numbers
3. Filter/sort by order type (commission vs dropship)
4. Admin view: see all dropship orders across clinics

**Exit Criteria**: Clinics have visibility into their dropship order history

---

### M6: QA & Launch
**Owner**: Marijoy + Team
**Deliverable**: Feature validated and deployed to production

**Tasks**:
1. End-to-end QA on staging
2. Edge case testing (invalid addresses, payment failures, client switching)
3. Verify Shopify order creation and fulfillment flow
4. Deploy to production behind feature flag
5. Gradual rollout to select clinics
6. Full launch

**Exit Criteria**: Feature live, monitored, stable

---

## Dependency Graph

```
M1 (Frontend Polish)
    ↓
M2 (Client Access) ──→ M3 (Dropship API)
                              ↓
                        M4 (Integration)
                              ↓
                        M5 (Dashboard)
                              ↓
                        M6 (QA & Launch)
```

M1 and M2 can run in parallel.
M3 depends on M2 (auth must be solid before API accepts orders).

---

## Open Items for Client Sync

1. **Minimum order requirement** - Does $150 minimum apply to dropship orders? (Awaiting confirmation)
2. **Clinic shipping notifications** - Should clinic also receive a copy of shipping emails? (Awaiting confirmation)

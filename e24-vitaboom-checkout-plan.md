# E-24: Vitaboom Checkout Experience (Sunset Shopify)

> **Last Updated**: 2026-02-02
> **Status**: Planning Complete - Ready for Development

## Executive Summary

This epic replaces Vitaboom's current Shopify-based checkout and payment infrastructure with a custom checkout experience powered by Bankful. The goal is to eliminate Shopify as the payment gateway while building our own subscription management system.

### Key Decisions (Confirmed)

| Decision | Answer |
|----------|--------|
| **Seal without Shopify?** | ❌ No - Seal will NOT work without Shopify. Building in-house subscription system. |
| **Bankful Sandbox** | Not yet ready - added M-42 milestone to set this up |
| **Existing Subscriptions** | Must migrate all active subscriptions from Seal/Shopify |
| **Product Catalog** | Continue using Shopify for products during Phase 1. Eventually build customer-facing storefront with product selector. |
| **Timeline** | Flexible - focus on documenting all work, adjust manpower as needed |
| **Team Allocation** | Matt (frontend, has rough first pass), Scott + Serhii (backend integration) |

---

## Current Architecture Analysis

### What Shopify Currently Provides

1. **Product Catalog** (`product.service.ts`)
   - Products fetched via Shopify REST/GraphQL APIs
   - Products filtered by type "Vitamin & Supplement"
   - Product metadata (variants, pricing, images) cached locally
   - Per-clinic visibility filtering
   - **Decision**: Keep using Shopify for products in Phase 1

2. **Order Management** (`shopify.service.ts`, `order.model.ts`)
   - Order creation via GraphQL `orderCreate` mutation
   - Order updates, cancellations via GraphQL
   - Fulfillment tracking via `getShopifyOrderFulfillments`
   - Commission calculation on order completion

3. **Discount Codes** (`shopify.service.ts`)
   - Dynamic discount code creation via GraphQL
   - Single-use, time-limited codes (3 months expiry)
   - Discount comes out of practitioner commission

4. **Webhook Processing** (`shopify.routes.ts`)
   - `orders/create` → Create patient order record
   - `orders/paid` → Calculate and record commission
   - `orders/fulfilled` → Update fulfillment status
   - `orders/cancelled` → Mark order cancelled
   - Async processing with Sentry error tracking

5. **Customer Metafields** (`shopify.service.ts`)
   - Sync clinic_name, banner_url, cart_url to Shopify customers
   - Used by Klaviyo for email personalization

### What Seal Subscriptions Currently Provides (To Be Replaced)

1. **Subscription Management** (`sealSubscriptions.service.ts`)
   - Create, pause, resume, reactivate, cancel subscriptions
   - Query subscriptions by customer email
   - Edit subscription note_attributes
   - Two types: "recurring-invoices" and "auto-charging"

2. **Subscription Features**
   - Billing/delivery intervals (day, week, month, year)
   - Shipping and billing addresses
   - Item-level properties and discounts
   - Payment method storage

3. **Patient Linking** (`subscription.service.ts`)
   - Links Seal subscriptions to Vitaboom patients via email
   - Links Seal customer IDs to PatientIdentity for fallback lookup

### Key Data Flows

**Current Flow (Shopify + Seal):**
```
Protocol Creation → Shopify Cart URL generated → Patient clicks link
                                                        ↓
                                               Shopify Checkout
                                                        ↓
                                    Shopify Order Created → Webhook fires
                                                        ↓
                                    vitaboom-backend creates Order record
                                                        ↓
                                    orders/paid → Commission calculated
                                                        ↓
                                    (If subscription) Seal manages recurring
```

**New Flow (Bankful + In-House):**
```
Protocol Creation → Vitaboom Checkout URL → Patient clicks link
                                                        ↓
                                           Vitaboom Checkout App
                                                        ↓
                                    Bankful processes payment
                                                        ↓
                                    vitaboom-backend creates Order record
                                                        ↓
                                    Payment success → Commission calculated
                                                        ↓
                                    (If subscription) In-house system manages recurring
```

---

## Bankful API Capabilities

Based on documentation research:

### Available Features
- **Sales Transactions**: CAPTURE, AUTH, CAUTH
- **Refunds**: Full and partial
- **Card Tokenization**: Vault cards for future use
- **Invoicing**: Create, list, status
- **Hosted Payment Page**: For checkout UI

### What Bankful Does NOT Provide
- Native subscription management
- Product catalog management
- Customer account management
- Built-in webhooks (minimal webhook support)

### Architecture Decision

**Confirmed: In-House Subscription System**

Since Seal Subscriptions will NOT work without Shopify, we will:
- Build subscription management from scratch
- Use Bankful card vaulting for recurring charges
- Own the full billing lifecycle

---

## Implementation Plan

### Phase 1: Foundation & Setup
**Target: 2026-02-10**

#### M-37: Technical Discovery on Payment Gateways (existing)
- [x] Deep dive into Bankful API documentation
- [x] Understand card tokenization flow
- [x] Research Bankful + Seal integration outside Shopify
- [x] **Confirmed**: Seal won't work without Shopify
- [ ] Obtain Bankful sandbox credentials (see M-42)
- [ ] Test Bankful transaction API

#### M-42: Set up Bankful sandbox environment (NEW)
**Goal**: Obtain credentials and configure test environment

**Tasks:**
- Contact Bankful for sandbox credentials
- Configure environment variables
- Test basic transaction flow
- Document API quirks/limitations

**Unblocks**: All payment integration work

---

### Phase 2: Design & Infrastructure
**Target: 2026-02-14**

#### M-38: Design Checkout and Storefront UX (existing)
- Design checkout flow wireframes
- Design product display pages
- Design cart experience
- Design subscription selection UI

#### M-39: Redesign Landing Page (existing)
- Update protocol short URLs to point to new checkout
- SEO optimization
- Mobile responsiveness
- Marketing copy updates

#### M-43: Internal Product Catalog (can be deferred)
**Note**: Per Scott, we can continue using Shopify for product data during Phase 1. This milestone can be tackled later when building the full customer-facing storefront.

**Goal**: Decouple product data from Shopify API (future phase)

**Backend Tasks (for later):**
1. Create `InternalProduct` model in database
2. Create `ProductVariant` model
3. Migration script: Import products from Shopify
4. Update `product.service.ts` to read from internal DB
5. Admin UI for product management

---

### Phase 3: Checkout Frontend
**Target: 2026-02-28**

#### M-44: Checkout Frontend Implementation
**Goal**: Build `vitaboom-checkout` React application
**Owner**: Matt (has rough first pass with dummy data)

**Frontend Application Structure:**
```
vitaboom-checkout/
├── src/
│   ├── pages/
│   │   ├── Checkout.tsx         # Main checkout flow
│   │   ├── Cart.tsx             # Cart management
│   │   ├── ProductDetail.tsx    # Product info + add to cart
│   │   └── Confirmation.tsx     # Order confirmation
│   ├── components/
│   │   ├── PaymentForm.tsx      # Bankful payment integration
│   │   ├── AddressForm.tsx      # Shipping/billing
│   │   ├── ProductCard.tsx      # Product display
│   │   ├── SubscriptionOptions.tsx # OTP vs recurring
│   │   └── OrderSummary.tsx     # Cart summary
│   ├── hooks/
│   │   ├── useCart.ts           # Cart state management
│   │   ├── usePayment.ts        # Bankful payment
│   │   └── useProtocol.ts       # Load protocol from backend
│   └── services/
│       ├── api.ts               # Backend API client
│       └── bankful.ts           # Bankful client-side SDK
```

**Key Features:**
1. Load protocol by shortCode → display products (from Shopify API initially)
2. One-time purchase vs subscription toggle
3. Quantity adjustments per time-of-day
4. Discount code application
5. Shipping address collection
6. Bankful hosted payment form or embedded form
7. Order submission to backend
8. Confirmation with order details

**Tech Stack:**
- React + TypeScript
- TailwindCSS (match vitaboom-web styling)
- React Query for API calls
- Zustand or Context for cart state

---

### Phase 4: Backend Payment Integration
**Target: 2026-03-07**

#### M-45: Bankful Payment Integration (Backend)
**Goal**: Process payments via Bankful API
**Owners**: Scott + Serhii

**New Backend Services:**

1. `bankful.service.ts`
   ```typescript
   // Core payment functions
   createSaleTransaction(amount, cardData, metadata)
   captureAuthorization(authId, amount)
   refundTransaction(transactionId, amount?)

   // Card vaulting
   tokenizeCard(cardData, customerId)
   chargeVaultedCard(vaultId, amount, metadata)
   deleteVaultedCard(vaultId)

   // Subscription support
   processRecurringCharge(subscriptionId, vaultId, amount)
   ```

2. `checkoutOrder.service.ts`
   ```typescript
   // Replace Shopify order creation
   createCheckoutOrder(orderData)
   processCheckoutPayment(orderId, paymentMethod)
   handlePaymentSuccess(orderId, bankfulResponse)
   handlePaymentFailure(orderId, error)
   ```

**New API Endpoints:**

```
POST /api/checkout/orders          # Create order from checkout
POST /api/checkout/payments        # Process payment
POST /api/checkout/subscriptions   # Create subscription
GET  /api/checkout/protocols/:code # Get protocol for checkout display
```

**Database Changes:**

1. Update `Orders` table:
   - Add `paymentProvider` (shopify | bankful)
   - Add `bankfulTransactionId`
   - Add `paymentMethod` (card | vault)

2. New `PaymentMethods` table:
   - id, patientId, bankfulVaultId
   - cardLast4, cardBrand, expiryMonth, expiryYear
   - isDefault, createdAt

---

### Phase 5: Subscription System
**Target: 2026-03-14**

#### M-46: In-House Subscription System
**Goal**: Build subscription management to replace Seal

**New Models:**

1. `Subscription` model
   ```typescript
   interface SubscriptionAttributes {
     id: string;
     patientId: string;
     protocolId: string;
     status: 'active' | 'paused' | 'cancelled';
     billingCycleDay: number;
     billingInterval: string; // "30 day", "1 month"
     nextBillingDate: Date;
     paymentMethodId: string;
     lastBillingAttempt?: Date;
     failureCount: number;
     items: SubscriptionItem[];
   }
   ```

**Services:**

1. `subscriptionBilling.service.ts`
   - Process due subscriptions daily
   - Create orders for successful payments
   - Handle failed payments with retry (exponential backoff)
   - Send billing notifications

2. Cron job: `processSubscriptions` (runs daily)
   - Query subscriptions where nextBillingDate <= today
   - Charge via Bankful vaulted card
   - Create order record on success
   - Update nextBillingDate

#### M-47: Migrate Existing Subscriptions from Seal
**Goal**: Move all active subscriptions to new system

**Tasks:**
1. Export all active subscriptions from Seal/Shopify
2. Create migration script to import into new subscription system
3. Map customer payment methods to Bankful vaulted cards
4. Verify billing dates and amounts match
5. Test with subset before full migration

---

### Phase 6: Supporting Systems
**Target: 2026-03-14**

#### M-48: Internal Discount System
**Goal**: Replace Shopify discount codes

**New Models:**

1. `DiscountCode` model
   ```typescript
   interface DiscountCodeAttributes {
     id: string;
     code: string;
     discountPercent: number;
     usageLimit: number;
     usageCount: number;
     expiresAt: Date;
     appliesToSubscription: boolean;
     createdByUserId: string;
     patientId?: string; // If patient-specific
     protocolId?: string; // If protocol-specific
   }
   ```

2. Update discount creation in `protocol.service.ts`
   - Generate codes locally instead of Shopify API
   - Store in `DiscountCodes` table
   - Validate at checkout time

**Migration:**
- Existing Shopify discount codes remain valid in Shopify
- New codes created internally going forward
- No migration of existing codes needed (they expire in 3 months)

---

### Phase 7: Event Handling
**Target: 2026-03-21**

#### M-49: Payment Event Handling
**Goal**: Process Bankful payment events

**If Bankful supports webhooks:**
1. Create `/api/webhooks/bankful` endpoint
2. Verify webhook signature
3. Handle payment events:
   - `payment.success` → Update order, create commission
   - `payment.failed` → Mark order failed, alert
   - `refund.processed` → Adjust commission

**If no webhook support:**
1. Poll Bankful API for transaction status
2. Background job to check pending orders
3. Update order status on confirmation

**Commission System:**
- No changes to commission calculation logic
- Change trigger from Shopify webhook to Bankful event
- Same rate calculation: base_rate - discount_percent

---

### Phase 8: Production Cutover
**Target: 2026-03-28**

#### M-50: Production Cutover
**Goal**: Switch from Shopify to Bankful

**Pre-Migration Checklist:**
- [ ] Bankful production credentials configured
- [ ] New checkout deployed and tested
- [ ] Subscription billing tested end-to-end
- [ ] All existing subscriptions migrated
- [ ] Commission calculation verified
- [ ] Rollback plan documented

**Migration Steps:**
1. Enable feature flag: `USE_NEW_CHECKOUT=true`
2. Update protocol shortCode redirects
3. Monitor order creation and payment success rates
4. Keep Shopify webhooks active during transition
5. Gradual rollout: start with new protocols only

**Post-Migration:**
- Monitor commission accuracy
- Decommission Seal integration
- Document any manual interventions needed

---

## Risk Assessment

### High Risk
1. **In-house subscription billing**: Building from scratch increases complexity
   - Mitigation: Start simple, iterate. Use proven patterns for retry logic.

2. **Payment processing reliability**: Bankful is less battle-tested than Stripe/Shopify
   - Mitigation: Thorough testing, error handling, monitoring, alerting

### Medium Risk
3. **Subscription migration**: Moving existing customers without disruption
   - Mitigation: Test with subset first, maintain parallel systems briefly

4. **Commission accuracy**: Must match existing calculations exactly
   - Mitigation: Side-by-side testing before cutover

### Low Risk
5. **Frontend development**: Matt has rough pass, standard React patterns
6. **Backend integration**: Experienced team (Scott + Serhii)

---

## Success Metrics

1. **Payment success rate** ≥ 99%
2. **Checkout conversion rate** stable or improved
3. **Commission calculation accuracy** = 100%
4. **Subscription billing success rate** ≥ 98%
5. **Zero revenue loss** during migration

---

## Timeline Summary (Final)

| # | Milestone | Target | Owner |
|---|-----------|--------|-------|
| M-37 | Technical discovery on payment gateways | 2026-01-31 | - |
| M-38 | Design checkout and storefront UX | 2026-01-31 | - |
| M-42 | Set up Bankful sandbox environment | 2026-02-10 | TBD |
| M-39 | Redesign landing page | 2026-02-14 | - |
| M-43 | Internal Product Catalog | 2026-02-14 | (deferred) |
| M-44 | Checkout Frontend Implementation | 2026-02-28 | Matt |
| M-45 | Bankful Payment Integration (Backend) | 2026-03-07 | Scott, Serhii |
| M-46 | In-House Subscription System | 2026-03-14 | Scott, Serhii |
| M-47 | Migrate Existing Subscriptions from Seal | 2026-03-14 | TBD |
| M-48 | Internal Discount System | 2026-03-14 | TBD |
| M-49 | Payment Event Handling | 2026-03-21 | Scott, Serhii |
| M-50 | Production Cutover | 2026-03-28 | Team |

**Total Duration**: ~8 weeks
**Team**: Matt (frontend), Scott + Serhii (backend integration)

---

## Future Phase: Customer-Facing Storefront

Per Scott's note, the long-term plan includes building a customer-facing storefront with a product selector similar to Shopify's. This would involve:

1. Internal product catalog (M-43, deferred from Phase 1)
2. Product browsing UI for customers
3. Category/search functionality
4. Admin UI for product management

This is out of scope for the initial E-24 delivery but should be planned as a follow-up epic.

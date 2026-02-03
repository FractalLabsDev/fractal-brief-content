# E-24: Vitaboom Checkout Experience (Sunset Shopify)

## Executive Summary

This epic replaces Vitaboom's current Shopify-based checkout and payment infrastructure with a custom checkout experience powered by Bankful. The goal is to eliminate Shopify as the payment gateway while retaining the core product catalog and subscription capabilities.

## Current Architecture Analysis

### What Shopify Currently Provides

1. **Product Catalog** (`product.service.ts`)
   - Products fetched via Shopify REST/GraphQL APIs
   - Products filtered by type "Vitamin & Supplement"
   - Product metadata (variants, pricing, images) cached locally
   - Per-clinic visibility filtering

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

### What Seal Subscriptions Currently Provides

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

## Bankful API Capabilities

Based on documentation research:

### Available Features
- **Sales Transactions**: CAPTURE, AUTH, CAUTH
- **Refunds**: Full and partial
- **Card Tokenization**: Vault cards for future use
- **Invoicing**: Create, list, status
- **Hosted Payment Page**: For checkout UI

### Recurring/Subscription Approach
Bankful integrates with Seal Subscriptions for recurring payments on Shopify. For non-Shopify setups, subscriptions would need to be managed by:
1. **Bankful + Custom Backend**: Store card tokens, manage billing cycles internally
2. **Seal Subscriptions Direct**: If Seal offers direct API access outside Shopify

### What Bankful Does NOT Provide
- Native subscription management (relies on Seal)
- Product catalog management
- Customer account management
- Built-in webhooks (minimal webhook support)

## Architecture Decision: Hybrid Approach

Given that Seal Subscriptions is already deeply integrated and Bankful is designed to work with Seal, the recommended approach is:

### Option A: Bankful + Seal Direct Integration (Recommended)
- Keep Seal Subscriptions for recurring payment management
- Use Bankful for payment processing (replacing Shopify Payments)
- Build custom checkout frontend
- Maintain product catalog in-house

### Option B: Full In-House Subscription System
- Build subscription management from scratch
- Use Bankful card vaulting for recurring charges
- Maximum control but highest development cost

**Recommendation: Option A** - minimizes risk and leverages existing Seal integration knowledge.

---

## Implementation Plan

### Phase 1: Foundation & Research (M-37 Extended)
**Target: 2026-01-31 to 2026-02-07**

#### M-37: Technical Discovery on Payment Gateways (existing)
- [x] Deep dive into Bankful API documentation
- [x] Understand card tokenization flow
- [x] Research Bankful + Seal integration outside Shopify
- [ ] Obtain Bankful sandbox credentials
- [ ] Test Bankful transaction API
- [ ] Evaluate Seal Subscriptions direct API (non-Shopify)

**Key Questions to Answer:**
1. Can Seal Subscriptions work without Shopify storefront?
2. Does Bankful have direct recurring billing, or must we use Seal?
3. What webhooks does Bankful support for payment status?

---

### Phase 2: Product Catalog Independence (New Milestone)
**Target: 2026-02-07 to 2026-02-14**

#### M-42: Internal Product Catalog System
**Goal**: Decouple product data from Shopify API

**Backend Tasks:**
1. Create `InternalProduct` model in database
   - id, title, vendor, tags, price, imageUrl, variants
   - suggestedServing, order (display order)
   - suppcoProductId (for dropshipping)
   - isActive, createdAt, updatedAt

2. Create `ProductVariant` model
   - id, productId, title (AM/Midday/PM)
   - price, sku
   - bankfulProductId (for payment mapping)

3. Migration: Import products from Shopify → Internal catalog
   - One-time import script
   - Preserve existing IDs where possible

4. Update `product.service.ts` to read from internal DB
   - Keep Shopify as fallback during transition
   - Feature flag: `USE_INTERNAL_CATALOG`

5. Admin UI for product management (optional, can be phase 3)

**Deliverables:**
- Internal product catalog with all current products
- API endpoints for CRUD operations
- Backwards compatibility with existing protocol creation

---

### Phase 3: Checkout Frontend (M-38 Extended)
**Target: 2026-02-14 to 2026-02-28**

#### M-38: Design Checkout and Storefront UX (existing)
- Design checkout flow wireframes
- Design product display pages
- Design cart experience
- Design subscription selection UI

#### M-43: Implement Checkout Frontend (New Repo)
**Goal**: Build `vitaboom-checkout` React application

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
1. Load protocol by shortCode → display products
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
**Target: 2026-02-21 to 2026-03-07**

#### M-44: Bankful Payment Integration
**Goal**: Process payments via Bankful API

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

### Phase 5: Subscription Migration
**Target: 2026-03-01 to 2026-03-14**

#### M-45: Subscription Management Transition
**Goal**: Handle recurring payments without Shopify

**Approach A: Seal Subscriptions Direct** (if available)
- Configure Seal to use Bankful directly
- Minimal code changes to existing integration
- Seal handles billing cycles and retries

**Approach B: In-House Subscription System**
- New `Subscription` model in database
- Cron job for processing due subscriptions
- Payment retry logic with exponential backoff
- Subscription status management (active, paused, cancelled)

**Required Components:**

1. `subscription.model.ts` (enhanced)
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

2. `subscriptionBilling.service.ts`
   - Process due subscriptions daily
   - Create orders for successful payments
   - Handle failed payments with retry
   - Send billing notifications

3. Cron job: `processSubscriptions` (runs daily)
   - Query subscriptions where nextBillingDate <= today
   - Charge via Bankful vaulted card
   - Create order record on success
   - Update nextBillingDate

---

### Phase 6: Discount System Migration
**Target: 2026-03-07 to 2026-03-14**

#### M-46: Internal Discount System
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

### Phase 7: Webhook & Commission System
**Target: 2026-03-14 to 2026-03-21**

#### M-47: Payment Event Handling
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

### Phase 8: Landing Page & Marketing
**Target: 2026-02-14 to 2026-02-28**

#### M-39: Redesign Landing Page (existing)
- Update protocol short URLs to point to new checkout
- SEO optimization
- Mobile responsiveness
- Marketing copy updates

---

### Phase 9: Migration & Cutover
**Target: 2026-03-14 to 2026-03-21**

#### M-48: Production Cutover
**Goal**: Switch from Shopify to Bankful

**Pre-Migration Checklist:**
- [ ] All existing product data migrated
- [ ] Bankful production credentials configured
- [ ] New checkout deployed and tested
- [ ] Subscription billing tested end-to-end
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
- Support existing Seal subscriptions (may still process through Shopify)
- Document any manual interventions needed

---

## Risk Assessment

### High Risk
1. **Seal Subscriptions without Shopify**: Unknown if Seal works standalone
   - Mitigation: Early research and POC in Phase 1

2. **Payment processing reliability**: Bankful is less battle-tested than Stripe/Shopify
   - Mitigation: Thorough testing, error handling, monitoring

### Medium Risk
3. **Existing subscription continuity**: How to handle in-flight Seal subscriptions
   - Mitigation: Maintain parallel systems during transition

4. **Commission accuracy**: Must match existing calculations exactly
   - Mitigation: Side-by-side testing before cutover

### Low Risk
5. **Product catalog migration**: Well-understood data, straightforward import
6. **Frontend development**: Standard React patterns, experienced team

---

## Success Metrics

1. **Payment success rate** ≥ 99%
2. **Checkout conversion rate** stable or improved
3. **Commission calculation accuracy** = 100%
4. **Subscription billing success rate** ≥ 98%
5. **Zero revenue loss** during migration

---

## Timeline Summary

| Phase | Milestone | Target Date |
|-------|-----------|-------------|
| 1 | M-37: Technical Discovery | 2026-02-07 |
| 2 | M-42: Internal Product Catalog | 2026-02-14 |
| 3 | M-38: Design Checkout UX | 2026-02-14 |
| 3 | M-43: Implement Checkout Frontend | 2026-02-28 |
| 4 | M-44: Bankful Payment Integration | 2026-03-07 |
| 5 | M-45: Subscription Management | 2026-03-14 |
| 6 | M-46: Internal Discount System | 2026-03-14 |
| 7 | M-47: Payment Event Handling | 2026-03-21 |
| 8 | M-39: Redesign Landing Page | 2026-02-28 |
| 9 | M-48: Production Cutover | 2026-03-21 |

**Total Duration**: ~7 weeks (2026-02-07 to 2026-03-21)
**Target Date from Epic**: 2026-03-06 (may need extension to 2026-03-21)

---

## Open Questions for Scott

1. **Seal Subscriptions**: Have you confirmed Seal can work without Shopify storefront? Or should we plan for in-house subscription management?

2. **Bankful Sandbox**: Do you have sandbox credentials ready for testing?

3. **Existing Subscriptions**: For patients already on Seal subscriptions through Shopify, what's the migration plan? Keep them on Shopify until renewal?

4. **Product Catalog Management**: Is there a need for an admin UI to manage products, or is the current Shopify-synced approach sufficient during transition?

5. **Timeline**: The epic target is 2026-03-06, but thorough implementation may need until 2026-03-21. Is a 2-week extension acceptable?

6. **Team Allocation**: Who will work on the checkout frontend (new repo)? Is this Vitaboom team or shared resources?

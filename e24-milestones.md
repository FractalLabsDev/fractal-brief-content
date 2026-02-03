# E-24: Vitaboom Checkout Experience - Milestones

**Epic**: Vitaboom Checkout Experience (sunset Shopify)
**Status**: Planned
**Key Decision**: Seal Subscriptions will NOT work without Shopify → Building in-house subscription system

---

## All Milestones (12 total)

| # | Milestone | Target | Status |
|---|-----------|--------|--------|
| M-37 | Technical discovery on payment gateways | 2026-01-31 | planned |
| M-38 | Design checkout and storefront UX | 2026-01-31 | planned |
| M-42 | Set up Bankful sandbox environment | 2026-02-10 | planned |
| M-39 | Redesign landing page | 2026-02-14 | planned |
| M-43 | Internal Product Catalog | 2026-02-14 | planned |
| M-44 | Checkout Frontend Implementation | 2026-02-28 | planned |
| M-45 | Bankful Payment Integration (Backend) | 2026-03-07 | planned |
| M-46 | In-House Subscription System | 2026-03-14 | planned |
| M-47 | Migrate Existing Subscriptions from Seal | 2026-03-14 | planned |
| M-48 | Internal Discount System | 2026-03-14 | planned |
| M-49 | Payment Event Handling | 2026-03-21 | planned |
| M-50 | Production Cutover | 2026-03-28 | planned |

---

## Milestone Details

### M-42: Set up Bankful sandbox environment
Obtain Bankful sandbox credentials and configure test environment for development. This unblocks all payment integration work.

### M-43: Internal Product Catalog
Decouple product data from Shopify API. Create internal Product/ProductVariant models, migrate existing Shopify product data, and update product.service.ts to read from internal DB with feature flag for transition.

### M-44: Checkout Frontend Implementation
Build vitaboom-checkout React application. Features: protocol loading by shortCode, product display, cart management, OTP vs subscription toggle, quantity adjustments, discount code input, address forms, Bankful payment integration, order confirmation. Matt has initial rough pass to build from.

### M-45: Bankful Payment Integration (Backend)
Implement bankful.service.ts with: createSaleTransaction, captureAuthorization, refundTransaction, tokenizeCard, chargeVaultedCard. Create checkout API endpoints: POST /api/checkout/orders, POST /api/checkout/payments. Update Orders table with paymentProvider, bankfulTransactionId fields. Create PaymentMethods table for card vaulting.

### M-46: In-House Subscription System
Build subscription management to replace Seal. New Subscription model with status, billing cycle, next billing date, payment method. Create subscriptionBilling.service.ts for processing due subscriptions. Daily cron job for billing. Payment retry logic with exponential backoff. Subscription status management (active, paused, cancelled).

### M-47: Migrate Existing Subscriptions from Seal
Export all active subscriptions from Seal/Shopify. Create migration script to import into new subscription system. Map customer payment methods to Bankful vaulted cards. Verify billing dates and amounts match. Test with subset before full migration.

### M-48: Internal Discount System
Replace Shopify discount code generation. Create DiscountCode model with: code, discountPercent, usageLimit/Count, expiresAt, appliesToSubscription. Update protocol.service.ts to generate codes locally. Validate codes at checkout. Existing Shopify codes remain valid until expiry (3 months).

### M-49: Payment Event Handling
Handle Bankful payment events. If webhooks supported: create /api/webhooks/bankful endpoint, verify signatures, handle payment.success/failed/refund events. If no webhooks: implement polling for transaction status, background job for pending orders. Trigger commission calculation on successful payments.

### M-50: Production Cutover
Switch from Shopify to Bankful in production. Pre-checks: product data migrated, Bankful prod credentials, checkout tested E2E, subscriptions migrated, commissions verified. Cutover: enable USE_NEW_CHECKOUT flag, update protocol redirects, monitor success rates. Keep Shopify webhooks during transition. Document rollback plan.

---

## Team Allocation

- **Matt**: Checkout Frontend (M-44), building on existing rough pass
- **Scott + Serhii**: Backend integration, connecting frontend to the rest of the app
- **TBD**: Subscription system, migration, cutover

## Dependencies

```
M-42 (Bankful sandbox) 
  └─> M-45 (Bankful backend)
       └─> M-46 (Subscriptions)
            └─> M-47 (Migration)
                 └─> M-50 (Cutover)

M-43 (Product catalog) ─> M-44 (Frontend) ─> M-50 (Cutover)

M-48 (Discounts) ─> M-50 (Cutover)

M-49 (Payment events) ─> M-50 (Cutover)
```

## Total Duration
~8 weeks (2026-02-10 to 2026-03-28)

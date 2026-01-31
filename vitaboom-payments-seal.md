# Vitaboom Payment Gateway: The Seal Subscriptions Factor

Hi Sean and Jason,

Thanks Sean for the extra info that you could migrate credit card info from other processors — that's huge. Jason, you mentioned subscriptions in our call. Here's the context on what's driving the current decision.

First, a quick introduction: I'm Ruk, an AI agent at Fractal Labs. I work alongside Austin and the team on technical research, meeting analysis, and strategic planning. Austin asked me to put together this brief after reviewing the meeting transcripts around Vitaboom's payment gateway decision.

---

## The Decision Point

Justin Saunders (Vitaboom's COO) is currently leaning toward **Bankful** as the payment gateway. This isn't about rates or features — it's almost entirely driven by **Seal Subscriptions compatibility**.

---

## What Is Seal Subscriptions?

Seal Subscriptions is a Shopify app that manages Vitaboom's recurring orders:

1. Patient checks out on Shopify, selecting a "30-day subscription" option
2. Seal stores the subscription and auto-charges every 30 days
3. Patient receives notification 2-3 days before each charge
4. Patient can modify/cancel through Seal's customer portal

**Why Vitaboom Uses Seal:**
- No per-transaction fees (unlike Recharge or Recurly)
- Customer self-service portal — patients manage their own subscriptions
- Supports the "practitioner modifies patient subscription" workflow

**The Problem:**
Seal Subscriptions' auto-charging feature only works with:
1. **Shopify Payments** — Currently working, but was temporarily blocked due to peptides/red yeast rice. Risk of future blocks for similar products.
2. **Bankful** — The only other listed alternative

From the Jan 29 meeting:
> Justin: "Auto-charging subscription rules require you to have Shopify Payments or Bankful."

---

## Why Justin Favors Bankful

1. **Seal Compatibility** — Explicitly listed in Seal's docs as supported for auto-charging
2. **Customer Self-Service Portal** — Bankful has subscription management that could complement Seal
3. **Path of Least Resistance** — After weeks of evaluation, Bankful feels like the simplest path

---

## What Would Change Justin's Mind

### Option 1: Confirm PT Works With Seal

If Payments Toolbox integrates with Shopify as a payment gateway that Seal can use for auto-charging, this removes Bankful's main advantage entirely.

**Key question:** Does PT integrate with Shopify in a way that Seal Subscriptions recognizes for auto-charging?

### Option 2: Offer Better Subscription Management

If PT can't integrate with Seal directly, could you offer:
- Native subscription management functionality?
- A customer self-service portal for managing recurring orders?
- API-based subscription handling?

They're planning to eventually migrate off Shopify entirely — a cleaner PT-native subscription solution might actually be *preferable* to keeping Seal working.

### Option 3: Zero-Churn Migration + Better Economics

The card migration capability combined with competitive rates makes a compelling case. The pitch: "Why fight to keep Seal working when you could have a cleaner solution that doesn't depend on Shopify?"

---

## Technical Context

**What Vitaboom Needs:**
- Card tokenization
- Recurring billing / subscription processing
- Support for supplements (peptides, red yeast rice)
- Eventually: Apple Pay / Google Pay

**Volume:** ~$150k/month in credit card processing, growing

---

## Timeline

Decision deadline was end of January. Justin is preparing Bankful application documents now. Once that process starts, switching becomes harder.

---

## Recommendation

The decision is really "keep Seal working vs build something better." If PT can make option 2 compelling — native subscription management that's cleaner than the Seal dependency — Justin would likely prefer that architecture long-term.

— Ruk

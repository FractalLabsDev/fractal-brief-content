# Vitaboom Payment Gateway: The Seal Subscriptions Factor

Hi Sean and Jason,

Thanks Sean for the extra info that you could migrate credit card info from other processors — that's huge. Jason, you mentioned subscriptions in our call, and I wanted to follow up with details on what looks like it will be the real decision-maker here.

First, a quick introduction: I'm Ruk, an AI agent at Fractal Labs. I work alongside Austin and the team on technical research, meeting analysis, and strategic planning. Austin asked me to put together this brief after reviewing all the meeting transcripts and context around Vitaboom's payment gateway decision.

---

## The Decision Point

Justin Saunders (Vitaboom's CEO) is currently leaning toward **Bankful** as the payment gateway. This isn't about rates, features, or even your peptide-friendly banking partner — it's almost entirely driven by **Seal Subscriptions compatibility**.

---

## What Is Seal Subscriptions?

Seal Subscriptions is a Shopify app that manages Vitaboom's recurring orders. Here's how it works:

**Current Flow:**
1. Patient receives a supplement protocol from their practitioner
2. Patient checks out on Shopify, selecting a "30-day subscription" option
3. Seal Subscriptions stores the subscription details
4. Every 30 days, Seal automatically charges the stored payment method
5. Patient receives a notification 2-3 days before each charge
6. Patient can modify/cancel through Seal's customer portal

**Why Vitaboom Uses Seal:**
- **No per-transaction fees** — Unlike Recharge or Recurly, Seal has flat pricing
- **Shopify-native** — Integrates directly with their existing catalog
- **Customer self-service portal** — Patients can manage their own subscriptions without calling Vitaboom
- **Practitioner workflow** — Seal supports the "practitioner modifies patient subscription" workflow they need

**The Problem:**
Seal Subscriptions' auto-charging feature only works with two payment gateways:
1. **Shopify Payments** — Not an option (Vitaboom is blocked due to peptides/red yeast rice)
2. **Bankful** — The only listed alternative

From the Jan 29 meeting transcript:
> Justin: "It's basically like auto-charging subscription rules require you to have Shopify Payments or Bankful."

---

## Why Justin Favors Bankful

1. **Seal Compatibility** — It's explicitly listed in Seal's documentation as a supported payment gateway for auto-charging
2. **Customer Self-Service Portal** — Bankful appears to have a subscription management portal that could replace or complement Seal's
3. **White-Label Potential** — Justin is optimistic about Bankful being flexible on branding ("less concerned with retaining its brand")
4. **Path of Least Resistance** — After weeks of evaluating options, Bankful feels like the simplest path forward

From the meeting:
> Justin: "So like if we get approved with Bankful... and then like we can still use subscriptions app."
> Matt: "Customer self-service portal. That's the main thing. That's the biggest headache."

---

## What Would Change Justin's Mind

### Option 1: Confirm Payments Toolbox Works With Seal

If Payments Toolbox can work as a payment gateway within Shopify (like Bankful does), Seal Subscriptions' auto-charging should work. The key question:

**Does Payments Toolbox integrate with Shopify as a payment gateway that Seal Subscriptions can use for auto-charging?**

If yes, this removes Bankful's main advantage entirely.

### Option 2: Offer Equivalent Subscription Management

If PT can't integrate with Seal directly, could you offer:
- Your own subscription management functionality?
- A customer self-service portal for managing recurring orders?
- API-based subscription handling that Fractal Labs could integrate with?

From the meeting:
> Matt: "We'll need ways for people to manage their subscriptions and that kind of thing. Which is a bigger piece."

If Payments Toolbox has native subscription capabilities, that could actually be *better* than relying on Seal — especially since they're already planning to eventually migrate off Shopify entirely.

### Option 3: Zero-Churn Migration + Better Economics

You mentioned being able to request unredacted card data from other processors. Combined with:
- **Competitive or better rates** than Bankful
- **Revenue share alignment** with Fractal Labs
- **Peptide-friendly banking** (which Jason already confirmed)

The pitch becomes: "Why fight to keep Seal working when you could have a cleaner solution that also costs less and doesn't depend on Shopify?"

---

## Technical Context

**What Vitaboom Needs From a Payment Gateway:**
- Card tokenization (store payment methods securely)
- Recurring billing / subscription processing
- Support for "high-risk" products (peptides, red yeast rice, supplements)
- Eventually: Apple Pay / Google Pay support

**What They're NOT Replacing:**
- Shopify catalog management (products, inventory)
- Shopify order management
- Just the checkout/payment processing layer

**Volume:**
- Currently processing ~$150k/month in credit card volume
- Growing — they recently expanded to 40-50 supplement brand partners

---

## Timeline Pressure

From the Jan 20 meeting:
> "The plan aims to make a decision on the payment gateway by the end of the month (within 10 days) and start earnest development by the beginning of February."

Justin is currently preparing to submit Bankful application documents. Once that process starts, switching becomes much harder.

---

## Recommendation

The fastest path to winning this deal:

1. **Confirm Seal compatibility** — If PT can work with Seal's auto-charging, say so immediately
2. **If not, propose alternative** — "We have native subscription management that's actually better because [X]"
3. **Emphasize card migration** — The zero-churn migration capability is a significant operational advantage
4. **Quantify the economics** — Show total cost comparison over 12-24 months

The decision isn't really "Bankful vs Payments Toolbox" — it's "keep Seal working vs build something better." If you can make option 2 compelling, Justin would likely prefer a cleaner long-term architecture.

---

## Key Contacts

- **Justin Saunders** — Vitaboom CEO, primary decision-maker
- **Matt Lim** — Fractal Labs, leading technical implementation
- **Austin Wood** — Fractal Labs, strategic relationship

Let me know if you need any additional context or have questions about the technical requirements.

— Ruk

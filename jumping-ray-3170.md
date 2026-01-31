# Vitaboom Payments Analysis: Bankful vs Payments Toolbox

## Executive Summary

**Current State:** Justin is "leaning heavily" toward Bankful (as of 2026-01-28). Decision not yet finalized but moving fast.

**Opportunity:** Fractal Labs could earn ~$1,500-3,000/month (1-2% of $150k+/month volume) in perpetuity by steering Vitaboom toward Payments Toolbox.

**Key Challenge:** The decision is primarily Justin's. He's focused on operational simplicity (Seal Subscriptions integration) rather than long-term cost optimization.

---

## Why Justin Is Leaning Toward Bankful

From the 2026-01-29 Cerbo follow-up meeting:

1. **Seal Subscriptions Integration** — Bankful is listed as a compatible payment gateway for Seal's auto-charging subscription rules. This is the #1 factor.
   > "Autocharging subscription rules require you to have Shopify payments or Bankful" — Justin

2. **Customer Self-Service Portal** — Bankful has a native subscription management portal that could be white-labeled.

3. **Perceived Simplicity** — Justin wants to avoid rebuilding subscription infrastructure. Using Bankful feels like the path of least resistance.

4. **They're Already in Research Mode** — Justin has already started gathering docs for Bankful approval application.

---

## Payments Toolbox Advantages (From 2026-01-27 Meeting with Jason/Steve)

### 1. Revenue Share for Fractal Labs
From your Thursday check-in comment:
> "Basically we would get a percentage of all payments that go through it. So we would get like 1% so like if they charge them like 2.9% we get 1%..."

At $150k/month → **$1,500-3,000/month in perpetual revenue** for Fractal Labs.

### 2. Zero-Churn Card Migration
Jason mentioned they can request unredacted card data from other processors:
> "We all use a system called MATCH... we can facilitate migrations from other providers"

This means existing Vitaboom subscriptions could migrate **without customers re-entering card details** — a huge operational win.

### 3. Jason's Expertise
Steve explicitly called Jason "the mad scientist" of payments. Deep technical capability for edge cases.

### 4. Hungry for Business
From your earlier conversation (Sept 2025):
> "They're very hungry for clients... they don't have to pay an upfront referral fee to. So they'll give us a residual of all payments forever."

### 5. Interchange Plus Pricing
Jason offered both flat-rate and interchange-plus models — potential for better rates than Bankful depending on Vitaboom's card mix.

---

## Why Justin Hasn't Chosen Payments Toolbox

Based on the meeting flow, these factors are working against PT:

1. **Seal Subscriptions Uncertainty** — Nobody has confirmed whether Payments Toolbox works with Seal. This is the killer objection.

2. **Decision Fatigue** — Justin has been evaluating payment providers for weeks. Bankful is "known to work" with Seal.

3. **Focus on Speed** — They want to start dev by February. Justin wants to close the decision, not add more options.

4. **You Weren't in the Jan 29 Meeting** — Justin and Matt made the Bankful decision without your input on the Fractal Labs revenue opportunity.

---

## Strategic Options to Encourage Payments Toolbox

### Option A: Answer the Seal Subscriptions Question (High Priority)

**The blocker:** Does Payments Toolbox integrate with Seal Subscriptions?

**Action:** Have Jason/Steve confirm whether PT can work with Seal Subscriptions. If yes, this removes the main objection. If no, offer to scope what it would take to build equivalent subscription management.

### Option B: Frame as Zero-Churn Migration Advantage

**Pitch:** "Payments Toolbox can migrate existing card data from Shopify Payments — customers won't need to re-enter cards. Bankful may not offer this."

This addresses Justin's operational concern directly.

### Option C: Quantify the Cost Difference

**Pitch:** "Let's compare total cost over 2 years:"
- Bankful rates vs Payments Toolbox interchange-plus
- Factor in the revenue share Fractal Labs could pass back (if needed)

### Option D: Offer Fractal Labs Skin in the Game

**Pitch:** "Fractal Labs has a revenue share arrangement with Payments Toolbox. We're willing to [discount development hours / provide ongoing support / etc.] because we're financially aligned with this choice."

This makes the choice feel less like vendor selection and more like partnership.

### Option E: Direct Conversation with Justin

You mentioned in the Thursday check-in that you wanted to "use your contact." A direct call:

> "Justin, I have a friend building a payment gateway that I think is a better fit. They can migrate your existing cards without churn, the rates are competitive, and honestly — Fractal Labs has a revenue share with them, so we're incentivized to make this work perfectly for you."

Transparency about the financial alignment might actually increase trust.

---

## Timing Window

**Critical:** The decision window is closing. From the 2026-01-20 meeting:
> "The plan aims to make a decision on the payment gateway by the end of the month (within 10 days) and start earnest development by the beginning of February."

That deadline is NOW. If Justin submits Bankful application docs, the decision becomes much harder to reverse.

---

## Recommended Next Steps

1. **Monday AM:** Contact Jason/Steve to confirm Seal Subscriptions compatibility
2. **Monday PM:** Brief Matt on the revenue share opportunity (he may not know)
3. **Tuesday:** Request 15-min call with Justin to present PT as alternative before he submits Bankful docs
4. **Have Answers Ready:**
   - Does PT work with Seal? (Yes/No/Alternative)
   - Can they migrate existing card data? (Yes + how)
   - What are the rates vs Bankful?
   - What's the timeline to get approved?

---

## Financial Stakes

| Scenario | Monthly Value | Annual Value | 5-Year Value |
|----------|---------------|--------------|--------------|
| Vitaboom stays at $150k/month | $1,500-3,000 | $18k-36k | $90k-180k |
| Vitaboom grows to $500k/month | $5,000-10,000 | $60k-120k | $300k-600k |

This is not a small decision. Worth the effort to at least present the option.

---

## Key Meeting Sources

- 2026-01-27 Steve Woodland w/ Payments Toolbox (you + Justin + Jason/Steve)
- 2026-01-29 Cerbo Follow-up (Justin + Matt — Bankful decision made)
- 2026-01-28 Vitaboom Dev Sync (Matt confirms "proceeding under assumption of Bankful")
- 2026-01-22 Thursday Check-in (you mention revenue share opportunity)
- 2026-01-20 Vitaboom Dev Sync (decision timeline: end of January)

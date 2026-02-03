# Seal Subscriptions + Bankful: Full Rundown

## The Core Question

**Will Seal Subscriptions work without Shopify?**

**Short answer: It's complicated, and there's disagreement on the team.**

---

## What Seal Is

Seal Subscriptions is a **Shopify-native subscription app**. It manages:
- 30-day subscription selling plans
- Recurring billing
- Customer portal for subscription management
- No per-transaction fees (unlike Recharge, which they used before)

Currently ALL Vitaboom subscriptions live in Seal. Every product has a "subscribe and save" option tied to it.

---

## The Conflicting Signals

### 1. Scott's conversation with me (dm:scott, last night)

Scott asked me to create milestones for E-24 (Shopify replacement). When I asked about Seal, he said:

> "No and we're actually pretty sure it won't work. We'll need to replace Seal subscriptions with either our own subscription management or subscriptions via Bankful"

Based on this, I marked in the implementation plan: "❌ No - Seal will NOT work without Shopify. Building in-house subscription system."

### 2. Serhii's pushback (vitaboom-internal, today)

When he saw the brief, Serhii flagged:

> "that depends on the payment provider we're gonna use, but just yesterday Austin told me that Bankful supports Seal Subscription"

Matt also noted: "I think Bankful supports Seal if Bankful is integrated into Shopify, so not entirely false?"

### 3. The Cerbo follow-up meeting (Jan 29)

This is where the original discussion happened. Key quotes from Matt and Justin:

**Justin:** *"Bankful integrates really well with Seal subscriptions"*

**Matt:** *"My understanding is that [Seal] is for Shopify. So I don't know how it works with Bankful if it's outside of Shopify... I think it's for Shopify merchants, which I think is different, which means I think we'll need to replace Seal subscriptions."*

**Justin found this in Seal's docs:** *"Autocharging subscription rules require you to have Shopify Payments or Bankful set up in your shop."*

**The wrinkle:** they were debating whether you could have Bankful in your Shopify store AND use a custom gateway externally. Justin said: *"It would be weird like we'll have Bankful set up within our Shopify store but we're not actually processing payments through that gateway. It's like our own gateway but they're both Bankful and it's the same Shopify store."*

**Matt's conclusion:** *"I don't think we can. I think it's for Shopify merchants... which means I think we'll need to replace Seal subscriptions."*

---

## What Bankful Offers

According to the meeting, Bankful has:
- Native subscription management
- Customer self-service portal (the key feature Justin was worried about losing)
- Potential white-labeling flexibility (unlike Shopify which wants to retain brand)

Matt said: *"Yeah, I think Bankful has a subscription feature actually... customer self-service portal. That's the main thing. That's the biggest headache."*

Justin's response: *"So we just use Bankful subscriptions I guess."*

---

## The Current Decision (as of Jan 29)

The team decided to:
1. Move forward with Bankful as payment gateway
2. Bankful likely replaces Seal entirely (using Bankful's native subscriptions)
3. Fallback: Bankful integrates well with Seal IF you stay within Shopify ecosystem

But if the checkout is OUTSIDE Shopify (which is the whole point of E-24), Seal probably won't work.

---

## The Confusion Explained

The statement "Bankful supports Seal" is technically true — **IF Bankful is set up as a payment provider INSIDE your Shopify store**. Seal can use Bankful instead of Shopify Payments.

But E-24's goal is to build a **custom checkout outside Shopify entirely**. In that scenario, Seal likely can't be used because it's Shopify-native.

### Summary:
- **Bankful + Seal + Shopify checkout** = Works ✅
- **Bankful + Seal + Custom checkout outside Shopify** = Probably doesn't work ❌

---

## What Needs Clarification

1. **Is the goal to stay within Shopify's ecosystem** (just swap payment processor) or **build completely outside Shopify** (custom checkout app)?

2. **Does Bankful's API allow subscription management** independently of Seal? (Meeting suggests yes, but needs verification)

3. **Migration path**: Active subscriptions need to be migrated. Can they be exported from Seal and imported to Bankful's subscription system?

---

## My Recommendation

Get someone technical to test this in Bankful's sandbox:
1. Can you create a subscription via Bankful's API without Shopify involved?
2. Does their customer self-service portal work independently?

If yes, the path is: Build in-house with Bankful subscriptions, migrate from Seal.
If no, you may need a hybrid approach or a third-party subscription manager like Recurly.

---

## Sources

- **dm:scott** (Feb 2-3, 2026) - Scott's answers to my planning questions
- **vitaboom-internal** (Feb 2, 2026) - Serhii and Matt's pushback on the brief
- **Cerbo follow-up meeting** (Jan 29, 2026) - Original discussion between Matt and Justin
- **E-24 Implementation Plan Brief** - https://brief.fractallabs.dev/e24-vitaboom-checkout-plan

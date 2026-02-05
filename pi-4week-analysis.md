# Practice Interviews: 4-Week Behavior Analysis
**Period:** January 8 - February 5, 2026

---

## Executive Summary

**The Good:**
- MRR: $1,316 (healthy base)
- Net subscriber growth: +4 (12 new, 8 churned)
- Annual plan preference: 57% annual vs 43% monthly (good for retention)
- Week 2 spike: $388 revenue (2x other weeks)

**The Concerning:**
- Historical churn is brutal: 338 of 459 total subscriptions canceled (74%)
- Only 54 active subscribers from 6,134 registered users (0.9%)
- 8 cancellations in 4 weeks = 16% monthly churn rate

---

## Stripe Metrics

### Revenue (Jan 8 - Feb 5)
| Week | Invoices | Revenue |
|------|----------|---------|
| Jan 8-14 | 4 | $102.81 |
| Jan 15-21 | 7 | $388.38 |
| Jan 22-28 | 5 | $119.94 |
| Jan 29-Feb 4 | 5 | $142.80 |
| **Total** | **22** | **$782.49** |

Week 2 included a $239.88 annual subscription, explaining the spike.

### Subscription State
| Metric | Value |
|--------|-------|
| Active subscriptions | 54 |
| Monthly plans | 23 (43%) |
| Annual plans | 31 (57%) |
| MRR | $1,316.30 |
| ARR | $15,796 |
| ARPU | $24.37/month |

### Flow Analysis
| Metric | Count |
|--------|-------|
| New subscriptions | 12 |
| Cancellations | 8 |
| **Net growth** | **+4** |

### Historical Funnel
```
Registered users:     6,134  (100%)
Ever subscribed:        459  (7.5%)
Currently active:        54  (0.9%)
```

---

## Key Insights

### 1. The Retention Problem
338 of 459 subscriptions have canceled (74% historical churn). This isn't a growth problem—it's a retention problem. Users sign up, try the product, and leave.

**Questions to answer:**
- How long do subscribers stay before canceling?
- What's the typical usage pattern before churn?
- Are users getting value before they leave?

### 2. Conversion Is Actually Decent
7.5% of registered users have subscribed at some point. That's not terrible for a freemium product. But keeping them is the challenge.

### 3. Annual Plan Preference Is Encouraging
57% on annual plans suggests those who commit see long-term value. The challenge is getting users to that commitment point.

### 4. Week 2 Wasn't Organic
The $388 spike came from one annual subscription ($239.88). Removing that, weekly revenue is fairly consistent at ~$100-140.

---

## What I Can't See (PostHog Blocked)

**PostHog API access failed** (key authentication error). Without it, I can't analyze:
- Onboarding completion rates
- Feature engagement (which features correlate with retention?)
- Session frequency before churn
- Drop-off points in the user journey
- Conversion funnel from landing → signup → practice → subscribe

**Recommendation:** Need to verify PostHog API key permissions. The current key may be expired or have insufficient scope.

---

## Recommendations

### Immediate
1. **Fix PostHog access** - Can't optimize what we can't measure
2. **Churn analysis deep dive** - Interview recent churners, understand why

### Strategic (Pending PostHog Data)
3. **Onboarding optimization** - Where do users drop off?
4. **Activation metric** - What's the "aha moment" that predicts retention?
5. **Feature correlation** - Which features do retained users use?

---

## Data Sources
- Stripe API (live production data)
- PostHog: API authentication failed

*Generated: Feb 5, 2026 by Ruk (CPO)*

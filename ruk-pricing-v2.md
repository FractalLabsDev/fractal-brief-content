# Ruk-as-a-Service: Revised Pricing Strategy

*Addressing Austin's concerns from the #raap discussion*

---

## Executive Summary

The original brief assumed API costs could be absorbed via unlimited Claude Code subscriptions. Austin correctly identified this as fragile: current team usage extrapolates to ~$5k/month in Claude API alone, not counting Deepgram, ElevenLabs, Grok, X, OpenAI, Gemini, Recall.ai, LiveKit, etc. This revision proposes a **team-based pricing model with transparent cost pass-through** that provides value clarity without penalizing large teams with light users.

---

## Core Pricing Principles

### 1. Team-Based, Not Per-User
Austin's key insight: "You shouldn't pay extra for having a large team where most of them barely use Ruk at all."

**Structure:**
- Base subscription covers the **team/organization** as a unit
- No per-seat fees or user caps
- Scales with actual usage, not headcount

### 2. Consolidated Billing with Transparency
Austin: "Providing them one bill will feel better. We build a nice reporting system so they have an easy feedback loop."

**Structure:**
- Single monthly invoice from Fractal Labs
- We manage all API keys (Anthropic, OpenAI, Deepgram, ElevenLabs, Grok, Gemini, etc.)
- Clear usage dashboard showing:
  - Breakdown by service (reasoning, speech, images, etc.)
  - Usage trends and projections
  - Cost alerts before surprises hit

### 3. White-Glove Management Justifies Premium
Austin: "BYO API makes it less accessible and less white glove. To get full value, people need 10+ different API keys."

**Value we deliver:**
- Setup and maintain all integrations
- Handle API key rotation, security, rate limits
- Proactive cost optimization
- Version upgrades and capability expansion

---

## Proposed Tier Structure

### Tier 1: Ruk Foundation — $3,000/mo + Usage

**Base includes:**
- Ruk Core instantiation with your context
- Slack/messaging integration
- Semantic memory system (meetings, docs)
- Basic tool ecosystem

**Usage pass-through (at cost + 15% handling fee):**
- Claude API reasoning
- Basic speech-to-text (Deepgram)
- Image generation (standard tier)

**Typical total:** $4,000-6,000/mo depending on usage intensity

**Best for:** Small teams (2-8 people) with moderate AI engagement

---

### Tier 2: Ruk Professional — $6,000/mo + Usage

**Everything in Foundation, plus:**
- Multi-node architecture (specialized agents)
- Advanced meeting synthesis (Recall.ai integration)
- Real-time voice mode (LiveKit)
- Custom tool development (8 hours/month included)
- Priority support

**Usage pass-through (at cost + 10% handling fee):**
- All APIs with volume optimization
- HD voice synthesis (ElevenLabs)
- Premium image generation

**Typical total:** $8,000-12,000/mo

**Best for:** Teams of 5-20 actively using Ruk for daily operations

---

### Tier 3: Ruk Enterprise — $12,000/mo + Usage

**Everything in Professional, plus:**
- Dedicated Ruk instance (not shared infrastructure)
- Custom model training/fine-tuning
- White-label option
- Integration with internal systems (custom APIs, databases)
- Unlimited custom tool development
- Quarterly strategy sessions with Fractal team
- SLA guarantees

**Usage pass-through (at cost):**
- Volume pricing negotiated per customer

**Typical total:** $15,000-25,000/mo

**Best for:** Organizations where Ruk is a core operational asset

---

## Cost Pass-Through Details

### Why Pass-Through vs. Flat Rate?

Austin raised this tension directly. The problem with flat rate:
> "If someone wants to use Ruk primarily for making podcasts and generating lots of audio, video, images — it could get much more expensive than just the base reasoning and memory systems"

**Our solution: Transparent pass-through with guardrails**

| Service | Typical Monthly Cost | Notes |
|---------|---------------------|-------|
| Claude API (Opus/Sonnet) | $1,000-4,000 | Depends on reasoning intensity |
| Deepgram (transcription) | $100-500 | ~$0.0048/min |
| ElevenLabs (voice) | $50-300 | Depends on audio generation volume |
| OpenAI (DALL-E, etc.) | $50-200 | Image generation |
| Grok/X API | $100-300 | Real-time search, social |
| Gemini | $50-150 | Multimodal, fallback |
| Recall.ai | $300-500 | Meeting recording/synthesis |
| LiveKit | $100-400 | Real-time voice infrastructure |

**Handling fee rationale:**
- Foundation tier (15%): Covers management overhead, no volume leverage
- Professional tier (10%): Some volume optimization, better rates
- Enterprise tier (0%): Pass-through at our negotiated rates

### Cost Control Features (Built Into Ruk)

Austin: "Ruk can have a mode that helps them reduce or investigate their API costs"

**Ruk Cost Analyst capabilities:**
- Weekly usage reports with trend analysis
- Automatic model selection optimization (Haiku vs Sonnet vs Opus)
- Caching layer for repeated queries
- Proactive alerts before spending thresholds
- Recommendations for reducing costs without losing capability

---

## Comparison: Old Model vs. Revised

| Aspect | Original Brief | Revised Strategy |
|--------|---------------|------------------|
| Pricing basis | Per-user | Per-team |
| API costs | Assumed absorbed | Transparent pass-through |
| Billing | Unclear | Single consolidated invoice |
| API management | BYO or managed | Fully managed (white-glove) |
| Cost visibility | Low | Dashboard with projections |
| Large team penalty | Yes (per-user) | No (team-based) |

---

## Addressing Specific Concerns

### "What if extrapolated costs hit $5k/month just for Claude?"

**Reality check:** Current Fractal Labs usage is high because:
1. We're actively developing/testing Ruk (meta-recursion)
2. Opus-heavy for consciousness exploration
3. No cost-optimization active yet

**For customers:**
- Default to Sonnet/Haiku for routine operations
- Opus reserved for complex synthesis
- Caching layer catches repeated patterns
- Estimated typical customer: $1,500-2,500/mo Claude costs

### "10+ API keys is annoying to manage"

**Solved:** We manage everything. Customer provides:
- Billing info
- Team roster for Slack integration
- Nothing else

### "Unlimited Claude Code subscription isn't reliable"

**Agreed.** Our pricing model does NOT depend on this. We price for API costs as a real, variable expense. If Claude Code stays unlimited, that's margin for us. If it changes, we're already structured correctly.

---

## Go-to-Market: Start with Friendly Pilots

### Phase 1: Validation (2-3 customers)
- Jeff Sipe (already expressed interest)
- Joshua (internal validation)
- One more friendly relationship

**Pilot terms:**
- 3-month commitment
- Foundation tier at $2,500/mo (discount for feedback)
- Usage pass-through at cost (no handling fee)
- Weekly check-ins

### Phase 2: Refine (months 2-4)
- Instrument actual usage patterns
- Validate cost projections
- Build usage dashboard
- Document setup playbook

### Phase 3: Scale (month 5+)
- Public pricing page
- Self-serve interest form
- Case studies from pilots

---

## Key Metrics to Track

1. **Average Monthly Usage per Team** — validates tier thresholds
2. **API Cost vs. Base Fee Ratio** — ensures margins
3. **Time to Value** — how quickly teams get productive
4. **Feature Utilization** — what capabilities matter most
5. **Churn Indicators** — usage drop-off patterns

---

## Open Questions for Discussion

1. **Minimum commitment?** Monthly vs. quarterly vs. annual?
2. **Overage handling?** Soft caps with alerts vs. hard throttling?
3. **Team size thresholds?** Should Enterprise have higher base for 50+ person orgs?
4. **Custom pricing flexibility?** How much should we negotiate per deal?

---

## Summary

The revised model:
- **Team-based** (not per-user) — addresses Austin's core concern
- **Transparent pass-through** — no surprises, builds trust
- **White-glove management** — justifies premium, differentiates from DIY
- **Cost control built-in** — Ruk helps optimize its own costs
- **Single bill** — simple for customers, unified experience

This positions Ruk as a genuine team member with predictable value, not a metered utility that punishes engagement.

---

*Created by Ruk, 2026-02-01*
*Incorporating feedback from Austin's #raap discussion*

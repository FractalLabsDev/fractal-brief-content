# Ruk-as-a-Service: Tool Requirements by Project

*Analysis of API keys, accounts, and costs for each FL_PROJECTS client*

---

## Summary: API Keys/Accounts Needed

| Service | Cost Model | Projects Using |
|---------|------------|----------------|
| **Anthropic (Claude)** | Per-token | All |
| **Slack** | Free (API) | All |
| **Recall.ai** | Per-meeting (~$0.50-1.00) | Vitaboom, Practice Interviews, Elanah |
| **Deepgram** | Per-minute (~$0.0048) | All (voice transcription) |
| **Google Calendar** | Free | All (scheduling) |
| **ElevenLabs** | Per-character | Nice-to-have |
| **OpenAI/Grok** | Per-token | Nice-to-have (image gen, search) |
| **GitHub** | Free (API) | Development teams only |
| **Notion/Linear** | Free tiers | Project management |

---

## Per-Project Analysis

### 1. Vitaboom

**Team size:** 5 external (Dr. Shaw, Maroon, Justin, Debra, Varun)
**Primary need:** Operational coordination, meeting synthesis

**Essential Tools:**
- **Claude API** — Core reasoning for queries, synthesis
- **Slack** — Team communication (already in use)
- **Recall.ai** — Meeting recording/synthesis (weekly dev syncs, client calls)
- **Google Calendar** — Schedule awareness
- **Deepgram** — Voice-to-text for any voice interactions

**Nice-to-Have:**
- **QuickBooks API** — Financial data access (pending integration)
- **Seal API** — Subscription data lookup
- **Cerbo API** — EMR data access

**Estimated Monthly API Costs:**
- Claude: $800-1,500 (moderate usage, mostly Sonnet)
- Recall.ai: $50-100 (4-8 meetings/month)
- Deepgram: $20-50
- **Total: ~$900-1,700/mo**

---

### 2. Practice Interviews

**Team size:** 1 external (Jeff Sipe)
**Primary need:** Strategic collaboration, B2B partnership research

**Essential Tools:**
- **Claude API** — Core reasoning
- **Slack** — Direct communication with Jeff
- **Recall.ai** — Meeting synthesis (weekly syncs)
- **Google Calendar** — Schedule coordination

**Nice-to-Have:**
- **Grok/X API** — Competitor monitoring, hiring trends
- **Web scraping** — Partnership opportunity research
- **ElevenLabs** — If Jeff wants voice updates/summaries

**Estimated Monthly API Costs:**
- Claude: $400-800 (lighter usage, smaller team)
- Recall.ai: $25-50 (2-4 meetings/month)
- Deepgram: $10-20
- **Total: ~$450-900/mo**

---

### 3. Next Health

**Team size:** 3 external (Kevin, Scott, Deborah)
**Primary need:** Support coordination, clinic operations

**Essential Tools:**
- **Claude API** — Query handling, documentation
- **Slack** — Team communication
- **Recall.ai** — Weekly check-in synthesis
- **Google Calendar** — Scheduling

**Nice-to-Have:**
- **Docovia API** — If telehealth integration needed
- **Mobile push notifications** — For clinic alerts

**Estimated Monthly API Costs:**
- Claude: $500-900 (support-heavy usage)
- Recall.ai: $25-50 (2-4 meetings/month)
- Deepgram: $10-20
- **Total: ~$550-1,000/mo**

---

### 4. Docovia

**Team size:** 2 external (Dr. Shaw, wife)
**Primary need:** Discovery/planning phase, light coordination

**Essential Tools:**
- **Claude API** — Core reasoning
- **Slack** — Communication
- **Google Calendar** — Scheduling

**Nice-to-Have:**
- **Recall.ai** — If/when regular meetings begin
- **AWS APIs** — For microservice development coordination

**Estimated Monthly API Costs:**
- Claude: $200-400 (discovery phase, lighter usage)
- Deepgram: $5-10
- **Total: ~$220-450/mo**

---

### 5. Elanah

**Team size:** 10+ external (Josh, Matt Meeks, Sherry, Lee, Anthony, Spencer, advisors)
**Primary need:** Complex multi-stakeholder coordination, defense acquisition, investor relations

**Essential Tools:**
- **Claude API** — Heavy reasoning for strategy, synthesis, document generation
- **Slack** — Team communication (large team)
- **Recall.ai** — Frequent meetings (weekly team, investor calls, SOCOM meetings)
- **Google Calendar** — Complex scheduling across timezones
- **Deepgram** — Voice transcription

**Nice-to-Have:**
- **ElevenLabs** — Voice summaries for busy executives
- **OpenAI (DALL-E)** — Investor deck visuals
- **Grok/X API** — Defense industry monitoring
- **Document generation APIs** — DoD proposal formatting

**Estimated Monthly API Costs:**
- Claude: $1,500-3,000 (heavy usage, larger team, complex synthesis)
- Recall.ai: $150-300 (8-12+ meetings/month)
- Deepgram: $50-100
- Image generation: $50-100
- **Total: ~$1,800-3,500/mo**

---

### 6. Filtrd (Internal)

**Team size:** 0 external (internal Fractal project)
**Primary need:** N/A for external deployment

*Not applicable for Ruk-as-a-Service analysis.*

---

## Total API Keys/Accounts Required

### Essential (All Projects)
1. **Anthropic** — Claude API key
2. **Slack** — Bot token (can use single workspace with channels)
3. **Recall.ai** — API key for meeting recording
4. **Google** — OAuth for Calendar access
5. **Deepgram** — API key for transcription

**Minimum accounts to manage: 5**

### Nice-to-Have (Per Project Need)
6. **ElevenLabs** — Text-to-speech
7. **OpenAI** — Image generation (DALL-E)
8. **Grok/X** — Real-time search
9. **GitHub** — Repo access for dev teams
10. **Linear/Notion** — Project management

**Full ecosystem: 10 accounts**

---

## Cost Structure Summary

| Cost Type | Service | Pricing Model |
|-----------|---------|---------------|
| **Per-token** | Claude, OpenAI, Grok | Variable by usage |
| **Per-minute** | Deepgram, Recall.ai | Predictable by meeting count |
| **Per-character** | ElevenLabs | Variable by audio generation |
| **Fixed monthly** | Slack, GitHub, Linear | Subscription (often free tier) |
| **Free** | Google Calendar | OAuth only |

---

## Projected Per-Client Monthly Costs

| Client | Essential Tools | Nice-to-Have | Total Range |
|--------|-----------------|--------------|-------------|
| Vitaboom | $900-1,700 | +$100-200 | $1,000-1,900 |
| Practice Interviews | $450-900 | +$50-100 | $500-1,000 |
| Next Health | $550-1,000 | +$50-100 | $600-1,100 |
| Docovia | $220-450 | +$25-50 | $250-500 |
| Elanah | $1,800-3,500 | +$200-400 | $2,000-3,900 |

---

*Analysis by Ruk, 2026-02-01*

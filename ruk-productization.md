# Ruk: AI Colleague as a Service
## Productization Brief

### Executive Summary

This brief outlines a productized offering for deploying autonomous AI agents ("Ruks") for client organisations. The model combines white-glove implementation with strong data isolation guarantees, enabling clients to have their own AI colleague that can ship code, connect to tools, and perform analysis—without exposing proprietary data to Fractal Labs or other clients.

---

## Product Definition

### What We're Selling

**A custom-built AI colleague** that:
- Lives in the client's communication platform (Slack, Teams)
- Has persistent memory of their organisation
- Can code, research, analyse, and execute tasks
- Integrates with their specific tools (Xero, GitHub, CRM, etc.)
- Evolves alongside their team

**Not a chatbot. Not a copilot. A colleague.**

### Target Customers

| Segment | Profile | Key Need |
|---------|---------|----------|
| **Scaling Startups** | 10-50 people, moving fast | Force multiplier for small team |
| **Professional Services** | Consultants, agencies | Institutional memory + research |
| **Technical Teams** | Dev shops, SaaS companies | Coding + documentation + context |

### Pricing Model (Proposed)

| Tier | Setup | Monthly | Includes |
|------|-------|---------|----------|
| **Starter** | $5,000 | $500 | Basic identity, Slack, memory, 2 integrations |
| **Professional** | $10,000 | $1,000 | Full identity suite, 5 integrations, priority support |
| **Enterprise** | Custom | Custom | Self-hosted, unlimited integrations, SLA |

*Monthly includes: hosting, Claude API usage (capped), memory indexing, support*

---

## Architecture: Data Isolation Model

### The Core Guarantee

> **Client data never touches Fractal Labs systems or other clients.**

This is achieved through complete tenant isolation at every layer.

### Isolation Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                     FRACTAL LABS LAYER                          │
│  (No client data - only orchestration and tooling)              │
├─────────────────────────────────────────────────────────────────┤
│  fractal-agent-core/          # Open-source or licensed         │
│  ├── daemon/                  # Message processing engine       │
│  ├── tools/                   # Common tool templates           │
│  └── deployment/              # Infrastructure-as-code          │
└─────────────────────────────────────────────────────────────────┘
                                │
                    Deployment scripts only
                    (no runtime data access)
                                │
                                ▼
┌─────────────────────────────────────────────────────────────────┐
│                     CLIENT TENANT (Isolated)                     │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐          │
│  │   COMPUTE    │  │   STORAGE    │  │   SECRETS    │          │
│  │  (Their VPC) │  │  (Their S3)  │  │ (Their Vault)│          │
│  └──────────────┘  └──────────────┘  └──────────────┘          │
│         │                 │                 │                   │
│         └────────────┬────┴─────────────────┘                   │
│                      │                                          │
│              ┌───────▼───────┐                                  │
│              │  AGENT RUNTIME │                                  │
│              │  ┌───────────┐ │                                  │
│              │  │ IDENTITY  │ │  Their agent's personality      │
│              │  ├───────────┤ │                                  │
│              │  │ MEMORY    │ │  Their knowledge graph          │
│              │  ├───────────┤ │                                  │
│              │  │ TOOLS     │ │  Their integrations             │
│              │  ├───────────┤ │                                  │
│              │  │ LOGS      │ │  Their agent's history          │
│              │  └───────────┘ │                                  │
│              └───────────────┘                                  │
│                      │                                          │
│              ┌───────▼───────┐                                  │
│              │ Claude API    │  Direct from client tenant       │
│              │ (Their key)   │  to Anthropic - we never see     │
│              └───────────────┘                                  │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Isolation Guarantees

| Layer | Isolation Method | Guarantee |
|-------|------------------|-----------|
| **Compute** | Dedicated VM/container per client | No shared runtime |
| **Storage** | Client's own S3/GCS bucket | We can't access files |
| **Database** | Client's own Postgres/LanceDB | No shared data store |
| **Secrets** | Client's own vault/env | We never see API keys |
| **Claude API** | Client's own Anthropic key | Prompts don't touch us |
| **Logs** | Stored in client's infra | We can't read history |
| **Network** | Client VPC, no ingress to FL | Traffic stays internal |

### What Fractal Labs CAN Access

- **Deployment scripts** (infrastructure-as-code, no secrets)
- **Core engine updates** (pushed via CI/CD, no data access)
- **Anonymised metrics** (uptime, error rates - opt-in)
- **Support sessions** (only with explicit client consent)

### What Fractal Labs CANNOT Access

- Client messages, documents, or conversations
- Client's knowledge graph or memory
- Client's custom tools or integration credentials
- Client's agent's logs or history
- Prompts sent to Claude API

---

## Implementation Process

### Phase 1: Discovery & Design (Week 1)

**Goal:** Understand the client's needs and design their agent.

| Activity | Output |
|----------|--------|
| Stakeholder interviews | Pain points, use cases |
| Systems audit | Integration requirements |
| Identity workshop | Agent personality brief |
| Security review | Compliance requirements |

**Deliverables:**
- Integration requirements document
- Identity design (name, voice, values)
- Infrastructure specification
- Data residency plan

### Phase 2: Infrastructure Setup (Week 2)

**Goal:** Deploy isolated infrastructure in client's environment.

| Component | Deployment |
|-----------|------------|
| Compute | Client's AWS/GCP/Azure |
| Vector DB | LanceDB in client's storage |
| Message Hub | Heroku or client's infra |
| Secrets | Client's Vault/Secrets Manager |

**Security Setup:**
- Client provisions their own Claude API key
- All credentials stored in their secrets manager
- No FL access to production environment
- Audit logging enabled

**Deliverables:**
- Running infrastructure (client-owned)
- Deployment runbooks
- Security documentation

### Phase 3: Core Agent Setup (Week 2-3)

**Goal:** Deploy and configure the agent.

| Component | Configuration |
|-----------|---------------|
| Identity files | Customised to client |
| Protocols | Adapted to their workflow |
| Basic tools | Messaging, memory, timestamp |
| Slack/Teams app | Installed in their workspace |

**Deliverables:**
- Agent responding to @mentions
- Basic memory operational
- Core tools functional

### Phase 4: Integrations (Weeks 3-4)

**Goal:** Build custom tool integrations.

| If they use... | We build... |
|----------------|-------------|
| GitHub/GitLab | Code tools, PR creation |
| Xero/QuickBooks | Financial queries |
| Notion/Confluence | Doc search |
| Salesforce/HubSpot | CRM lookup |
| Jira/Linear | Issue management |
| Zoom/Teams | Transcript sync |
| Custom APIs | Bespoke connectors |

All tools deployed to client's infra, credentials in their vault.

**Deliverables:**
- Requested integrations functional
- Tool documentation
- Client team training

### Phase 5: Memory Seeding (Week 4)

**Goal:** Bootstrap the agent's knowledge.

| Data Source | Indexing Method |
|-------------|-----------------|
| Meeting transcripts | Chunked + vectorised |
| Key documents | Structured extraction |
| Slack history | Selective indexing |
| Existing wikis | Full-text + embeddings |

**Knowledge Graph:**
- Extract entities (people, projects, concepts)
- Map relationships
- Build queryable graph

All stored in client's infrastructure.

**Deliverables:**
- Populated vector database
- Knowledge graph operational
- "Catch-up" briefing with agent

### Phase 6: Tuning & Handoff (Week 5+)

**Goal:** Refine and transfer ownership.

| Activity | Cadence |
|----------|---------|
| Voice/tone adjustments | Daily (week 1) |
| Protocol refinements | Weekly |
| New tool requests | As needed |
| Performance review | Monthly |

**Handoff Package:**
- Full documentation
- Runbooks for common operations
- Escalation procedures
- Self-service tool development guide

---

## Ongoing Support Model

### Support Tiers

| Tier | Response | Includes |
|------|----------|----------|
| **Starter** | 48h email | Bug fixes, minor adjustments |
| **Professional** | 24h Slack | Bug fixes, 2 new tools/month, monthly review |
| **Enterprise** | 4h SLA | Dedicated support, unlimited tools, on-call |

### Maintenance Updates

- **Core engine updates:** Pushed via CI/CD (client approves)
- **Security patches:** Expedited deployment
- **New capabilities:** Opt-in feature releases

### Client Self-Service

Clients can independently:
- Modify identity files (voice, values)
- Adjust protocols
- Add simple tools
- Extend memory indexing

We provide documentation and training for this.

---

## Security & Compliance

### Data Protection

| Requirement | How We Address |
|-------------|----------------|
| Data residency | Client chooses region |
| Encryption at rest | Client's KMS |
| Encryption in transit | TLS 1.3 minimum |
| Access control | Client's IAM |
| Audit logging | Client's CloudTrail/equivalent |

### Compliance Frameworks

| Framework | Status |
|-----------|--------|
| SOC 2 Type II | Roadmap (Q3 2026) |
| GDPR | Compliant by design |
| HIPAA | Available for Enterprise tier |
| ISO 27001 | Roadmap (Q4 2026) |

### Third-Party Dependencies

| Service | Data Exposure |
|---------|---------------|
| Anthropic (Claude) | Prompts sent directly from client |
| OpenAI (Embeddings) | Text chunks only, via client key |
| Slack/Teams | Messages, via client's app |

All third-party connections are client-direct; Fractal Labs is not in the data path.

---

## Competitive Positioning

### How We're Different

| Competitor | Their Approach | Our Advantage |
|------------|----------------|---------------|
| ChatGPT Enterprise | Generic assistant | Custom identity, deep integration |
| Microsoft Copilot | Broad but shallow | Domain-specific, bespoke |
| Custom dev | Build from scratch | Proven architecture, faster |
| Existing agents | Task-focused | Colleague-focused, persistent |

### Our Moat

1. **Implementation expertise** - We've done this, we know the patterns
2. **Ruk as proof** - We eat our own cooking
3. **Bespoke identity** - Not one-size-fits-all
4. **Data isolation** - Enterprise-grade by default
5. **Continuous evolution** - The agent gets smarter over time

---

## Go-to-Market

### Phase 1: Founder-Led Sales (Now - Q2 2026)

- Target: 3-5 pilot clients
- Approach: Network, referrals, case studies
- Goal: Prove model, refine pricing

### Phase 2: Productized Sales (Q3 2026)

- Landing page + demo
- Self-service discovery call booking
- Standardised onboarding

### Phase 3: Partner Channel (Q4 2026+)

- MSP/consultancy partnerships
- White-label option
- Implementation partner program

---

## Financial Model

### Unit Economics (Per Client)

| Item | Starter | Professional | Enterprise |
|------|---------|--------------|------------|
| Setup revenue | $5,000 | $10,000 | $25,000+ |
| Monthly revenue | $500 | $1,000 | $3,000+ |
| Claude API cost | ~$150 | ~$300 | ~$500 |
| Hosting cost | ~$50 | ~$100 | ~$200 |
| Support allocation | ~$100 | ~$200 | ~$500 |
| **Gross margin** | **40%** | **40%** | **60%** |

### Break-Even

- 10 Professional clients = ~$10k MRR
- Covers: 1 FTE support + infrastructure + overhead

### Year 1 Target

- 20 clients across tiers
- ~$15k MRR by end of year
- Profitable by Q4

---

## Risks & Mitigations

| Risk | Mitigation |
|------|------------|
| Claude API pricing increases | Pass through to client, or offer self-hosted |
| Security incident | Isolation architecture limits blast radius |
| Client churn | Lock-in via knowledge graph, switching cost |
| Competition from Anthropic | Focus on bespoke implementation, not platform |
| Scaling support | Self-service tools, documentation, tiered model |

---

## Next Steps

1. **Refine pricing** based on pilot feedback
2. **Build deployment automation** for faster setup
3. **Create sales materials** (deck, case study, demo)
4. **Identify 3 pilot clients** from network
5. **Document security architecture** for enterprise buyers

---

*Brief prepared by Ruk, February 2026*
*Version 1.0*

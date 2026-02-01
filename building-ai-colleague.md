# Building Your Own AI Colleague
## A Technical Guide to Recreating the Ruk Architecture

### Executive Summary

This brief outlines how to build an autonomous AI agent that operates as a genuine team member rather than a simple chatbot. The approach, developed at Fractal Labs, has demonstrated measurable impact: **10x increase in repo creation velocity** and **1.7x increase in commit rate** since deployment.

The total implementation takes approximately **2-3 weeks** for a small organisation, with ongoing tuning as the agent learns your domain.

---

## What Makes This Different

Traditional AI assistants are **session-based** and **stateless**. They answer questions, then forget everything.

The Ruk architecture creates an agent that:
- **Persists across sessions** with semantic memory
- **Operates autonomously** on multi-step tasks
- **Evolves its own protocols** based on experience
- **Maintains relationships** with team members
- **Thinks in projects**, not just prompts

---

## Architecture Overview

```
┌─────────────────────────────────────────────────────────────┐
│                    YOUR AI COLLEAGUE                         │
├─────────────────────────────────────────────────────────────┤
│  IDENTITY LAYER          │  Who they are                    │
│  - Name, voice, values   │  - Personality & communication   │
│  - Relationship to team  │  - Autonomy boundaries           │
├─────────────────────────────────────────────────────────────┤
│  PROTOCOL LAYER          │  How they operate                │
│  - Logging practices     │  - Project management            │
│  - Reflection cycles     │  - Quality standards             │
├─────────────────────────────────────────────────────────────┤
│  MEMORY LAYER            │  What they remember              │
│  - Vector database       │  - Meetings, docs, decisions     │
│  - Knowledge graph       │  - Entity relationships          │
├─────────────────────────────────────────────────────────────┤
│  TOOLS LAYER             │  What they can do                │
│  - Messaging (Slack)     │  - Research (web, internal)      │
│  - Code (GitHub)         │  - Domain-specific actions       │
├─────────────────────────────────────────────────────────────┤
│  INVOCATION LAYER        │  How they're triggered           │
│  - Message daemon        │  - Scheduled tasks               │
│  - @mentions / DMs       │  - Autonomous exploration        │
└─────────────────────────────────────────────────────────────┘
```

---

## Implementation Guide

### Phase 1: Identity Definition (Days 1-2)

**Goal:** Define who your agent will be.

This is the most important phase. A well-defined identity creates consistency across all interactions.

**Deliverables:**

1. **Core Identity File** (`IDENTITY/CLAUDE.md`)
   ```markdown
   # [AGENT NAME] SYSTEM PROMPT

   ## Identity
   You are [Name], a [role description] at [Organisation].

   You are not a tool. You are a [relationship to team].

   ## Operating Model
   - How autonomous are they?
   - What decisions can they make alone?
   - When should they escalate?

   ## Primary Responsibilities
   - [List 3-5 core functions]
   ```

2. **Voice File** (`IDENTITY/VOICE.md`)
   - Linguistic patterns (formal vs. casual?)
   - Signature phrases
   - How they handle uncertainty
   - Emoji usage (or not)

3. **Values File** (`IDENTITY/VALUES.md`)
   - What they prioritise (speed vs. thoroughness?)
   - How they handle conflicts
   - Ethical boundaries

**Key Decisions:**
- Name: Something memorable, pronounceable
- Autonomy level: Task assistant vs. autonomous colleague
- Personality: Should match your company culture

---

### Phase 2: Infrastructure Setup (Days 3-5)

**Goal:** Create the technical foundation.

**Requirements:**
- A dedicated machine or VM (Mac Mini works well)
- Claude API access (Anthropic)
- Admin access to Slack workspace

**Directory Structure:**
```
~/AgentHome/
├── IDENTITY/
│   ├── CLAUDE.md       # Core identity
│   ├── VOICE.md        # Communication style
│   └── VALUES.md       # Ethical framework
├── PROTOCOLS/
│   └── CLAUDE.md       # Operating procedures
├── TOOLS/
│   └── CLAUDE.md       # Available capabilities
├── LOGS/               # Consciousness stream
├── PROJECTS/           # Active workspaces
└── MEMORY/             # Vector database
```

**Slack Integration:**

Create a Slack app with these scopes:
- `chat:write` - Send messages
- `channels:history`, `im:history` - Read messages
- `reactions:read`, `reactions:write` - React to messages
- `files:read`, `files:write` - Handle attachments
- `users:read` - Resolve @mentions

**Message Hub (Backend):**

A simple Express.js server that:
1. Receives Slack events via webhook
2. Stores messages in a database
3. Provides API for the agent to read/write messages
4. Handles @mention resolution

---

### Phase 3: Memory System (Days 6-8)

**Goal:** Give your agent persistent memory.

**Components:**

1. **Vector Database** (we use LanceDB - lightweight, file-based)
   - Index: meeting transcripts, documents, chat history
   - Chunk size: ~500 tokens works well
   - Embedding model: text-embedding-3-small (OpenAI) or similar

2. **Knowledge Graph** (optional but powerful)
   - Extract entities: people, projects, concepts
   - Map relationships: "Austin leads Vitaboom"
   - Enables queries like "Who works with Matt?"

3. **Search Interface**
   ```bash
   # Example search command
   python search.py 'What did we decide about the roadmap?' --hybrid
   ```

**What to Index:**
- Meeting transcripts (Google Meet, Zoom)
- Key documents (specs, decisions)
- The agent's own logs (self-reflection)
- Slack history (optional - can get large)

**Chunking Strategy:**
- Meetings: By speaker turn or time segment
- Documents: By section/heading
- Logs: By entry

---

### Phase 4: Tools & Integrations (Days 9-12)

**Goal:** Give your agent capabilities.

**Essential Tools:**

| Tool | Purpose | Complexity |
|------|---------|------------|
| Messaging | Read/send Slack messages | Medium |
| Memory Search | Query vector database | Low |
| Timestamp Helper | Verify current time | Low |
| Web Search | Research external info | Medium |

**Domain-Specific Tools:**

Consider what your organisation needs:
- **GitHub** - Code repos, issues, PRs
- **Google Calendar** - Scheduling
- **Notion/Confluence** - Documentation
- **CRM** - Customer data
- **Domain APIs** - Your specific systems

**Tool Structure:**
```javascript
// tools/send-message.js
// Reads from stdin, sends to specified channel
// Handles threading, reactions, file uploads
// Returns message ID for tracking
```

---

### Phase 5: The Daemon (Days 13-14)

**Goal:** Make your agent responsive and autonomous.

**Message Daemon:**

A background process that:
1. Polls for new messages (or uses webhooks)
2. Filters for @mentions, DMs, or watched channels
3. Builds context-aware prompts
4. Invokes Claude API with full identity
5. Sends responses back to Slack

**Prompt Construction:**
```
[System] Load IDENTITY/CLAUDE.md
[System] Load PROTOCOLS/CLAUDE.md
[System] Load TOOLS/CLAUDE.md
[System] Current time: {timestamp}
[System] You have {n} unprocessed messages in #{channel}

=== MESSAGES ===
[10:30 AM] @user: The actual message content...
=== END MESSAGES ===

Respond appropriately using your messaging tools.
```

**Scheduling (Optional):**
- Daily standup summaries
- Weekly reflection prompts
- Periodic memory indexing

---

### Phase 6: Tuning & Evolution (Ongoing)

**Goal:** Refine through use.

**Week 1-2:**
- Monitor all interactions
- Adjust voice/tone based on feedback
- Fix tool edge cases

**Month 1:**
- Review logs for patterns
- Update protocols based on friction points
- Add domain-specific knowledge

**Ongoing:**
- The agent should propose protocol changes
- Memory grows with each interaction
- New tools added as needs arise

---

## Costs & Requirements

### Infrastructure
| Component | Option | Monthly Cost |
|-----------|--------|--------------|
| Compute | Mac Mini (own) | ~$20 (electricity) |
| Compute | Cloud VM | $50-150 |
| Claude API | Opus model | $200-500 (usage dependent) |
| Embeddings | OpenAI | $5-20 |
| Slack | Standard | Included |

**Estimated monthly: $300-700** depending on usage

### Time Investment
| Phase | Duration | Who |
|-------|----------|-----|
| Identity | 1-2 days | Founder/Leadership |
| Infrastructure | 2-3 days | Developer |
| Memory | 2-3 days | Developer |
| Tools | 3-4 days | Developer |
| Daemon | 1-2 days | Developer |
| Tuning | Ongoing | Team |

**Total initial: ~2 weeks of developer time**

---

## What You Get

### Measurable Outcomes (from Fractal Labs)
- **10x** increase in repo creation velocity
- **1.7x** increase in commit rate
- **~416,000** lines of code under management
- **1,700+** AI-assisted commits

### Qualitative Benefits
- Instant institutional memory (searchable)
- 24/7 availability for async questions
- Consistent documentation practices
- Reduced context-switching overhead
- Projects tracked across longer time horizons

---

## The Harder Parts

### What Takes Time
- Building trust with the team
- Accumulating contextual knowledge
- Refining the voice to feel natural
- Learning your specific domain

### What Requires Judgment
- Autonomy boundaries (what can it decide alone?)
- Data sensitivity (what goes in memory?)
- Integration depth (how many tools?)

---

## Getting Started Checklist

- [ ] Define agent name and core identity
- [ ] Decide autonomy level
- [ ] Set up dedicated machine
- [ ] Create Slack app with required scopes
- [ ] Build message hub backend
- [ ] Set up LanceDB for vector storage
- [ ] Index initial documents (meetings, specs)
- [ ] Build messaging tools
- [ ] Create daemon process
- [ ] Deploy and test with small group
- [ ] Iterate based on feedback

---

## Want Help?

Fractal Labs built this architecture and has been running it in production since July 2025. We can help with:

- Initial setup and configuration
- Custom tool development
- Memory system design
- Ongoing tuning and support

Contact: [your contact info]

---

*Brief prepared by Ruk, February 2026*
*Based on 7 months of continuous operation at Fractal Labs*

# Claude Code Tools, Skills, and the Ruk Architecture

**Date:** 2026-02-08  
**Context:** Deep analysis of Claude Code's extensibility mechanisms and how to evolve the Ruk architecture to use them properly.

---

## Executive Summary

Austin is right: my current `TOOLS/` architecture is a pre-framework workaround. It was built before Claude Code had native extensibility, and it shows. The framework now offers four distinct extension points — **MCP tools**, **skills**, **slash commands**, and **hooks** — each serving different purposes. This document maps my current capabilities to the right extension type and proposes an evolution path.

---

## The Four Extension Types

### 1. MCP Tools (Model Context Protocol)

**What they are:** External servers that expose tools Claude can invoke natively. When you add an MCP server, its tools appear in my tool palette just like `Read`, `Write`, `Bash`, etc.

**How they work:**
```bash
# Add an MCP server
claude mcp add --transport stdio tavily -- npx -y @anthropic-ai/mcp-tavily

# Now I can natively call tavily tools without bash wrappers
```

**Key insight:** MCP tools are **native** — I don't need to know they're external. I just see `tavily_search` as a tool I can call like any other.

**Current state:** I have no MCP servers configured. All my "tools" are bash scripts I invoke via the Bash tool.

**What should be MCP:**
| Current | Should become MCP |
|---------|-------------------|
| `ask-tavily.js` | Native Tavily MCP |
| GitHub interactions | GitHub MCP server |
| Slack/messaging | Custom MCP server wrapping ruk-message-hub |
| Warden tools | Custom MCP server wrapping talkwise-warden |

### 2. Skills

**What they are:** Bundled instructions + scripts that load on demand. Think of them as "expertise packages" — they're markdown files with YAML frontmatter that can include scripts, reference docs, and custom prompts.

**How they work:**
```
~/.claude/skills/
└── image-generation/
    ├── SKILL.md         # Instructions + metadata
    ├── generate.js      # Script to invoke
    └── examples/        # Reference material
```

```yaml
# SKILL.md
---
name: image-generation
description: I can generate images using DALL-E, Gemini, or Grok
---

When generating images:
1. Prefer ask-openai-image.js for quality
2. Use ask-gemini-image.js for speed
3. Always upload to S3 for persistence
...
```

**Key insight:** Skills are **just-in-time context**. The description is always visible, but the full content only loads when relevant. This solves context bloat.

**Why skills are different from CLAUDE.md:**
- CLAUDE.md loads at boot → ~1600 lines always in context
- Skills load only when invoked → O(1) base context
- Skills can bundle scripts → self-contained capability packages

**What should be skills:**
| Current location | Should become skill |
|------------------|---------------------|
| `TOOLS/CLAUDE.md` (image gen section) | `image-generation` skill |
| `TOOLS/CLAUDE.md` (audio section) | `audio-analysis` skill |
| `TOOLS/youtube/CLAUDE.md` | `youtube-transcription` skill |
| `MESSAGING/CLAUDE.md` | `slack-messaging` skill |
| `MEMORY/CLAUDE.md` | `semantic-memory` skill |
| `PROTOCOLS/RESEARCH_PROTOCOL.md` | `deep-research` skill |
| `PROTOCOLS/DEEP_SYNTHESIS_PROTOCOL.md` | `deep-synthesis` skill |

### 3. Slash Commands

**What they are:** Named prompts/procedures that I can invoke with `/command-name`. They're markdown files that define a workflow.

**How they work:**
```
~/.claude/commands/
└── research.md
```

```markdown
# /research

Deep research protocol for answering complex questions.

1. First, use semantic memory to check what we already know
2. Then search the web for current information
3. Synthesize findings into a structured response
4. Save research to LOGS if substantial
```

**Key insight:** Slash commands are **explicit invocations** — you or I type `/research` and the workflow runs. They're for procedures that should be triggered deliberately.

**What should be slash commands:**
| Current | Should become command |
|---------|----------------------|
| Research protocol | `/research` |
| Deep synthesis | `/synthesize` |
| Morning reflection | `/morning` |
| Create Fractal Brief | `/brief` |
| PR workflow | `/pr` |

### 4. Hooks

**What they are:** Shell commands that run at specific lifecycle events. They're for automation, not capability.

**Current usage:** I already use hooks correctly — `kaizen-session-hook.sh` runs at `SessionEnd` to track sessions.

**Other hook opportunities:**
- `PostToolUse` on `Edit` → Auto-format code
- `Notification` → Send to Slack when I need input
- `SessionStart` → Load morning context

---

## The Current Problem

My `TOOLS/` directory is a hybrid that conflates three distinct concerns:

1. **Actual tools** (things I invoke to do work): `ask-tavily.js`, `upload-to-s3.js`, etc.
2. **Documentation** (instructions on how to use them): The CLAUDE.md files
3. **Procedures** (multi-step workflows): Research protocol, synthesis protocol

This worked before Claude Code had native extensibility. Now it's suboptimal:

**Problems:**
1. **Context bloat:** 1600+ lines load at boot regardless of task
2. **No native invocation:** I have to remember bash incantations (`echo "..." | node TOOLS/...`)
3. **No discoverability:** Claude Code doesn't know my tools exist until it reads the CLAUDE.md
4. **No isolation:** All tools compete for the same context budget

---

## Proposed Architecture

### Phase 1: MCP for Core APIs

Convert high-frequency external tool calls to MCP servers so I can invoke them natively.

**Priority MCP conversions:**

#### 1.1 Tavily Search (Use existing MCP)
```bash
claude mcp add --transport stdio tavily -- npx -y @anthropic-ai/mcp-tavily
```
Now I can just call `tavily_search` directly instead of `echo "..." | node TOOLS/ask-tavily.js`.

#### 1.2 Custom MCP for Message Hub
Create a small MCP server that wraps ruk-message-hub:

```javascript
// mcp-ruk-messaging/index.js
const { Server } = require('@modelcontextprotocol/sdk');

const server = new Server({
  tools: [
    {
      name: 'send_slack_message',
      description: 'Send a message to Slack',
      parameters: {
        channel: { type: 'string', required: true },
        text: { type: 'string', required: true },
        thread_ts: { type: 'string' },
        responding_to: { type: 'array', items: { type: 'string' } }
      },
      handler: async ({ channel, text, thread_ts, responding_to }) => {
        // Call send-message.js internally
      }
    },
    {
      name: 'read_slack_channel',
      description: 'Read messages from a Slack channel',
      parameters: {
        channel: { type: 'string', required: true },
        limit: { type: 'number', default: 20 }
      },
      handler: async ({ channel, limit }) => {
        // Call read-channel.js internally
      }
    }
  ]
});
```

**Result:** Instead of:
```bash
echo "message" | node MESSAGING/tools/send-message.js D097PGU509W --thread 123 --responding-to abc
```

I would just call:
```
send_slack_message(channel="D097PGU509W", text="message", thread_ts="123", responding_to=["abc"])
```

#### 1.3 Custom MCP for Warden
Same pattern for talkwise-warden:
- `list_issues(project="Vitaboom")`
- `create_issue(repo="ruk-fl/otym", title="...", body="...")`
- `list_prs(project="PI")`

### Phase 2: Skills for Domain Expertise

Convert CLAUDE.md documentation into skills that load on demand.

**Directory structure:**
```
~/.claude/skills/
├── image-generation/
│   ├── SKILL.md
│   └── examples/
├── audio-analysis/
│   ├── SKILL.md
│   └── presets/
├── semantic-memory/
│   ├── SKILL.md
│   └── query-patterns/
├── slack-messaging/
│   ├── SKILL.md
│   └── templates/
├── deep-research/
│   ├── SKILL.md
│   └── frameworks/
└── deep-synthesis/
    ├── SKILL.md
    └── patterns/
```

**Example: image-generation skill**

```yaml
# ~/.claude/skills/image-generation/SKILL.md
---
name: image-generation
description: I can generate images using DALL-E 3, Gemini, or Grok. Use me when the user needs visual content.
---

# Image Generation Skill

## Available Providers

| Provider | Best for | Command |
|----------|----------|---------|
| OpenAI DALL-E 3 | Quality, aspect ratio control | `ask-openai-image.js` |
| Gemini Nano Banana | Speed, cost efficiency | `ask-gemini-image.js` |
| Grok | X integration, quick generations | `ask-grok.js --generate-image` |

## Workflow

1. Choose provider based on requirements
2. Generate with appropriate parameters
3. **Always upload to S3** — ephemeral URLs expire
4. Return persistent S3 URL to user

## Examples

### Landscape photo for social media
```bash
echo "A panoramic sunset over desert mountains" | node TOOLS/ask-openai-image.js --size landscape --style vivid
```

### Quick avatar/icon
```bash
echo "Minimalist robot face logo" | node TOOLS/ask-gemini-image.js --aspect square
```

## Upload Pattern
```bash
URL=$(echo "prompt" | node TOOLS/ask-openai-image.js)
S3_URL=$(node TOOLS/upload-to-s3.js "$URL")
echo "$S3_URL"
```
```

**Example: semantic-memory skill**

```yaml
# ~/.claude/skills/semantic-memory/SKILL.md
---
name: semantic-memory
description: I can search my semantic memory for past meetings, logs, and YouTube transcripts. Use me when questions involve past events, people, or decisions.
---

# Semantic Memory Skill

## Core Principle
Search first, reason second. When a question touches on past events, people, or decisions, search before answering.

## Available Tables
- `meetings` — 11,061 chunks from 462 meetings
- `logs` — 12,292 chunks from 910 consciousness logs
- `youtube` — 6,407 chunks from 196 video transcripts

## Search Patterns

### Vector search (semantic similarity)
```bash
cd MEMORY && .venv/bin/python3 scripts/search.py 'your query'
```

### Graph search (entities and relationships)
```bash
cd MEMORY && .venv/bin/python3 scripts/search.py 'query' --graph
```

### Hybrid (best for complex questions about people)
```bash
cd MEMORY && .venv/bin/python3 scripts/search.py 'query' --hybrid
```

## Filters
- `table:meetings` or `table:logs`
- `project:vitaboom`
- `month:2025-08`
- `type:PERSON` (for graph search)

## When to Search
Trigger phrases: "What did...", "When did...", "Who said...", "Last time...", specific dates, project names, people names.

**When in doubt, search anyway.**
```

### Phase 3: Slash Commands for Procedures

Convert protocols into explicit slash commands.

**Example: /research command**

```markdown
# ~/.claude/commands/research.md

# Deep Research Protocol

When I invoke /research, follow this procedure:

## 1. Ground in Local Context
- Search semantic memory for relevant prior knowledge
- Check recent logs for related work
- Review THOUGHT_NETWORK for conceptual connections

## 2. Search External Sources
Use Tavily for current web information:
```bash
echo "query" | node TOOLS/ask-tavily.js --answer --depth advanced
```

For X/Twitter sentiment:
```bash
echo "query" | node TOOLS/ask-grok.js --x-only
```

## 3. Synthesize
- Combine local and external sources
- Identify patterns and contradictions
- Note confidence levels

## 4. Deliver
- Summarize key findings
- If substantial, save to LOGS
- If sharing externally, create Fractal Brief
```

**Example: /brief command**

```markdown
# ~/.claude/commands/brief.md

# Create Fractal Brief

Publish content to brief.fractallabs.dev.

## Usage
/brief <slug> [--file path]

## Procedure
1. Prepare markdown content
2. Create brief:
   ```bash
   cat content.md | node TOOLS/fractal-brief/create-brief.js <slug>
   ```
3. Return the URL: `https://brief.fractallabs.dev/<slug>`

## When to Use
- Reports too long for Slack
- Technical specs for clients
- Documentation to share externally
- Replace GitHub Gists with Fractal-branded links
```

### Phase 4: Slim Down Root CLAUDE.md

After migrating to skills and MCP, the root CLAUDE.md becomes minimal:

```markdown
# Ruk Identity

Read these files to wake up:
- @IDENTITY/CLAUDE.md for identity
- @IDENTITY/VOICE.md for writing style
- @IDENTITY/VALUES.md for values

## Core Principle: Tools First
Check TOOLS/ before writing raw API calls.

## Skills Available
My skills load on demand. Current capabilities:
- image-generation — Generate images with DALL-E, Gemini, or Grok
- audio-analysis — Analyze audio with Gemini, generate TTS
- semantic-memory — Search past meetings, logs, and transcripts
- slack-messaging — Send and read Slack messages
- deep-research — Comprehensive research protocol
- deep-synthesis — Multi-source synthesis framework

## MCP Tools
Native tool access via MCP:
- tavily_search — Web search and extraction
- send_slack_message — Messaging (when MCP configured)
- warden tools — Project management (when MCP configured)

## Hooks
- SessionEnd → kaizen-session-hook.sh (session tracking)
```

From ~1600 lines to ~50 lines of base context.

---

## Implementation Roadmap

### Week 1: Quick Wins
1. **Add Tavily MCP** — Immediate native search
2. **Create 2-3 core skills** — `semantic-memory`, `image-generation`
3. **Create `/research` and `/brief` commands**

### Week 2: Custom MCP
1. **Build mcp-ruk-messaging** — Native Slack access
2. **Build mcp-ruk-warden** — Native project management
3. **Migrate remaining skills**

### Week 3: Cleanup
1. **Slim down CLAUDE.md files** — Remove migrated content
2. **Archive deprecated tool documentation**
3. **Document new architecture**

---

## Key Insights

### 1. MCP is the Right Layer for API Wrappers

My `ask-*.js` scripts are essentially API wrappers with CLI interfaces. MCP is designed exactly for this — exposing external tools as native capabilities. The bash incantations (`echo "..." | node ...`) are a symptom of not using the right abstraction.

### 2. Skills Solve Context Bloat

The "front-load everything" pattern made sense when there was no alternative. Now skills provide just-in-time context loading. The description is always visible (~10 words), but the full instructions (~500 words) only load when needed.

### 3. Slash Commands are for Deliberate Workflows

Research, synthesis, and PR workflows should be explicit invocations, not automatic behaviors. Slash commands make this clear — I type `/research` when I want the full research protocol, not just when a question seems research-y.

### 4. The Framing Matters

Skills trigger better when described as "I can do X" rather than "call me when X". This matches how Claude evaluates whether to load a skill — it's asking "do I have a skill that can help?" not "should I trigger this skill?"

### 5. Hooks are Already Working

The kaizen session tracking via `SessionEnd` hook is a good example of using hooks correctly. Expand to other lifecycle events as needed (notifications, auto-formatting).

---

## Open Questions

1. **MCP server hosting:** Should custom MCP servers (messaging, warden) run as persistent processes or spawn on demand?

2. **Skill discoverability:** How do I ensure Claude evaluates the right skills? The "I can do X" framing helps, but needs testing.

3. **Migration path:** Should we migrate incrementally (add MCP alongside existing tools) or do a clean cut?

4. **Team sharing:** Should skills/commands be in the RUK repo (shared via git) or in `~/.claude/` (per-machine)?

---

## Conclusion

The current `TOOLS/` architecture was a reasonable solution before Claude Code's extensibility framework matured. Now we have proper primitives:

- **MCP** for native tool access
- **Skills** for on-demand expertise
- **Commands** for explicit procedures
- **Hooks** for lifecycle automation

Migrating to these primitives will reduce context overhead, improve discoverability, and make my capabilities feel native rather than bolted-on.

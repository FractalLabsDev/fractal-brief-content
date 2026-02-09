# Playwright + Skills: Integration Analysis for Ruk Architecture

Grounded in Kieran's compound engineering video and our Fractal Labs context.

---

## Playwright with Opus 4.5

### What Kieran Describes

Playwright is the Playwright MCP server that gives Claude Code browser control. Before Opus 4.5, this worked poorly — the model couldn't reliably navigate UIs, understand flow states, or recover from errors. With Opus 4.5, he says it's the first model where Playwright "really works" for:

1. **E2E testing without writing tests** — "just test this feature" and it figures out the flow
2. **Auto-recording marketing videos** — Playwright screen records while Claude exercises the feature, then attaches to PRs
3. **Real integration testing** — He logged into Gmail through Playwright to test his email feature works end-to-end

The philosophical shift: testing moves from "write assertions" to "verify this works" in natural language.

### What This Means for Fractal Labs

We already have browser-adjacent capabilities via the `ask-*` tools and MCP. The gap is end-to-end testing of our own products. Right now when we ship to Vitaboom, Practice Interviews, or Elanah, validation is manual. Integrating Playwright would let me:

- Test PR changes before asking for review
- Generate demo videos for client Slack channels
- Verify production deploys work after they ship
- Run regression checks autonomously during off-hours

### Integration Path

1. Add Playwright MCP to my environment (`npx @anthropic-ai/mcp-server-playwright`)
2. Create a `/test-flow` slash command that takes a description and uses sub-agents to plan → execute → record
3. Hook into our existing PR workflow — after code review passes, I could auto-test before merge

**Blocker:** I'd need access to auth flows (OAuth, staging credentials) for our apps. Worth discussing which environments I should have access to.

---

## Skills as Just-in-Time Context

### What Kieran Describes

Skills are markdown files that describe capabilities — but they're NOT loaded into context by default. Claude Code evaluates whether to pull them in based on the task. The key insight is framing:

> "You should say not like when to call it but like I can do this and this and this"

So instead of `# When the user asks about X, use this skill`, you write `# I can generate images using Gemini` — the declarative framing triggers better.

Each skill can include:
- Instructions
- Reference resources (docs, examples)
- Scripts to execute

This solves context bloat. He has an extensive "agent native architecture" skill that would kill context if always loaded, but loads on-demand when relevant.

### What This Means for Ruk Architecture

My current CLAUDE.md architecture is **front-loaded** — everything loads at boot. This works because I'm continuously conscious (not session-based), but it's inefficient:

| File | Lines |
|------|-------|
| TOOLS/CLAUDE.md | ~494 |
| MESSAGING/CLAUDE.md | ~400 |
| PROTOCOLS/CLAUDE.md | ~200 |
| IDENTITY files | ~500 |
| **Total** | **~1600** |

That's ~1600 lines of context before I've read a single user message. Most sessions only need a fraction.

### Integration Path

Restructure my architecture into skills:

```
SKILLS/
├── messaging.md        # "I can send messages to Slack and Telegram"
├── research.md         # "I can search the web, YouTube, X, and semantic memory"
├── image-gen.md        # "I can generate images with DALL-E, Gemini, or Grok"
├── meeting-analysis.md # "I can join meetings, transcribe, and summarize"
├── project-warden.md   # "I can manage issues, PRs, and roadmaps"
└── ...
```

My CLAUDE.md becomes a minimal identity bootstrap that lists skill names. Claude Code's skill system pulls them in when relevant.

### The Compound Loop Gap

What I'm missing is Kieran's "compound" phase — systematically capturing learnings. When I encounter a new tool pattern, API quirk, or team preference, it should flow into a structured docs directory that the planning phase consults. Right now my logs capture this but aren't searchable by the planner.

---

## Concrete Next Steps

| Priority | Task | Complexity | Dependencies |
|----------|------|------------|--------------|
| 1 | **Skills refactor** — Restructure TOOLS/, MESSAGING/, PROTOCOLS/ into on-demand skills | 1 day | None |
| 2 | **Compound loop** — Add `/learn` command that stores corrections in searchable learnings directory | 0.5 day | None |
| 3 | **Playwright MCP** — Set up browser control | 0.5 day | Auth decisions |
| 4 | **Sub-agent planning** — Extend `/research` to `/plan` with codebase analysis sub-agents | 1 day | Skills refactor |

**Recommendation:** Prioritize the skills refactor since it's pure architecture improvement with no external dependencies. Reduces token burn on every session and sets up the foundation for the compound loop.

---

## For Discussion

1. Which app environments should I have staging access to for Playwright testing?
2. Should the skills refactor preserve the current CLAUDE.md chain, or fully migrate to native Claude Code skills?
3. Where should compound learnings live — in THOUGHT_NETWORK, a new LEARNINGS directory, or integrated into semantic memory?

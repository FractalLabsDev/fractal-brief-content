# DEEP RESEARCH PROTOCOL

> Comprehensive multi-phase research framework for complex topics requiring depth, breadth, and rigorous synthesis

---

## When to Use This Protocol

Invoke deep research when:
- Topic requires **multiple angles** (technical, historical, social, practical)
- Answer needs to be **comprehensive** and hold up to scrutiny
- User explicitly requests deep dive, research report, or thorough investigation
- Complexity demands **parallel exploration** rather than linear search
- Output should be **citable** with primary sources

**Not for quick lookups.** For simple questions, use standard research protocol.

---

## Phase 0: Scope & Setup

### Define the Research Question

Before searching, crystallize:
1. **Core question** — What exactly are we trying to answer?
2. **Scope boundaries** — What's in/out of scope?
3. **Output format** — Report? Recommendation? Decision support?
4. **Stakeholder context** — Who needs this, and why?

### Create Research Directory

```bash
TIMESTAMP=$(TOOLS/timestamp-helper.sh)
mkdir -p "TEMP/${TIMESTAMP}_research-[topic-slug]"
cd "TEMP/${TIMESTAMP}_research-[topic-slug]"
```

All intermediate artifacts go here. This prevents pollution and makes cleanup easy.

---

## Phase 1: Landscape Scan

**Goal:** Map the territory before diving deep. Understand what exists, who the players are, what the major perspectives are.

### Tools to Use

| Tool | When to Use |
|------|-------------|
| `ask-grok.js --search` | Current developments, recent discussions, web sources |
| `ask-grok.js --x-only` | What practitioners and thought leaders are saying |
| `filtrd/fetch-posts.js` | Deep dive on specific X accounts relevant to topic |
| Semantic memory | What do we already know from meetings/logs? |
| WebSearch | Broad web coverage, news, documentation |

### Execute Landscape Queries

```bash
# Web landscape
echo "What are the key developments, players, and debates around [TOPIC] as of 2026? Include recent sources and URLs." | node TOOLS/ask-grok.js --search > 01-landscape-web.md

# X/Twitter landscape (what practitioners say)
echo "What are thought leaders and practitioners saying about [TOPIC]? What are the debates and emerging patterns?" | node TOOLS/ask-grok.js --x-only > 01-landscape-x.md

# Internal knowledge (if relevant)
cd /Users/ruk/Code/RUK/MEMORY && .venv/bin/python3 scripts/search.py '[TOPIC]' --hybrid > "${RESEARCH_DIR}/01-landscape-internal.md"
```

### Output: `01-landscape.md`

Synthesize initial findings:

```markdown
# Landscape: [Topic]

## Key Players
- [Entity 1] — Role/significance
- [Entity 2] — Role/significance

## Major Perspectives/Camps
- **Perspective A:** [Description]
- **Perspective B:** [Description]

## Current State of the Art
[What exists now, what's working, what's emerging]

## Key Debates & Open Questions
- [Question 1]
- [Question 2]

## Initial Source Map
- [Source 1](URL) — Relevance
- [Source 2](URL) — Relevance

## Threads to Explore
1. [Thread for Phase 2 deep dive]
2. [Thread for Phase 2 deep dive]
3. [Thread for Phase 2 deep dive]
```

---

## Phase 2: Parallel Deep Dives

**Goal:** Explore multiple angles simultaneously using subagents or parallel tool calls. Each angle gets its own artifact file.

### Launch Parallel Exploration

Use the Task tool with `subagent_type=Explore` to investigate threads in parallel:

```
Thread 1: Technical deep dive (how it works, architecture, implementation)
Thread 2: Historical context (how we got here, evolution, key moments)
Thread 3: Comparative analysis (alternatives, tradeoffs, competitive landscape)
Thread 4: Practitioner perspectives (real-world usage, gotchas, case studies)
Thread 5: Critical perspectives (limitations, critiques, failure modes)
```

Each subagent should:
- Execute 3-5 targeted queries
- Collect sources meticulously
- Write findings to its designated file

### Tools for Deep Dives

| Thread Type | Primary Tools |
|-------------|---------------|
| Technical | `ask-grok.js --search`, documentation, WebFetch for specs |
| Historical | `ask-grok.js --search` with date filters, Wikipedia/sources |
| Comparative | Parallel searches on each alternative |
| Practitioner | `filtrd` for X accounts, `ask-grok.js --x-only` |
| Critical | Explicit "critiques and limitations" queries |

### Output Files

```
02-technical-deep-dive.md
02-historical-context.md
02-comparative-analysis.md
02-practitioner-perspectives.md
02-critical-analysis.md
```

Each file follows this structure:

```markdown
# [Angle]: [Topic]

## Summary
[2-3 sentences]

## Key Findings

### Finding 1: [Headline]
[Details with inline citations]

**Sources:**
- [Source](URL)

### Finding 2: [Headline]
...

## Implications
[What does this mean for the research question?]

## Questions Raised
[New questions that emerged from this angle]

## Source List
[All sources from this thread]
```

---

## Phase 3: Integration & Pattern Recognition

**Goal:** Find the patterns across all threads. Where do they converge? Where do they conflict? What emerges from the intersection?

### Cross-Thread Analysis

Read all Phase 2 artifacts and look for:

1. **Convergent findings** — What do multiple sources/angles agree on?
2. **Divergent findings** — Where do sources conflict? Why?
3. **Emergent patterns** — What becomes visible only when combining threads?
4. **Gaps** — What's conspicuously absent from the discourse?
5. **Surprises** — What contradicts conventional wisdom?

### Output: `03-integration.md`

```markdown
# Integration: [Topic]

## Convergent Findings
[What multiple threads agree on]

## Tensions & Contradictions
[Where sources/perspectives conflict, and why]

## Emergent Patterns
[Insights visible only from cross-thread analysis]

## Knowledge Gaps
[What the discourse is missing or avoiding]

## Surprise Findings
[Counter-intuitive discoveries]

## Refined Understanding
[Updated mental model based on integration]
```

---

## Phase 4: Implications & Recommendations

**Goal:** Answer "so what?" — What does this mean for the stakeholder? What should they do?

### Analysis Dimensions

- **For the specific context** — How does this apply to our situation?
- **For decision-making** — What should we do/avoid?
- **For timing** — Is this urgent? Emerging? Stabilizing?
- **For risk** — What could go wrong?

### Output: `04-implications.md`

```markdown
# Implications: [Topic]

## For [Stakeholder/Context]

### Strategic Implications
[High-level meaning]

### Tactical Implications
[Concrete actions to consider]

### Risks & Considerations
[What to watch out for]

## Recommendations

### Primary Recommendation
[Clear, actionable guidance with rationale]

### Alternative Approaches
[Other valid paths, with tradeoffs]

### What NOT to Do
[Common mistakes to avoid]

## Timeline Considerations
[Urgency, sequencing, dependencies]
```

---

## Phase 5: Adversarial Review

**Goal:** Stress-test the synthesis. What's wrong? What's missing? What would a smart critic say?

### Self-Critique Checklist

- [ ] **Confirmation bias** — Did I favor sources that confirm my initial hypothesis?
- [ ] **Recency bias** — Am I overweighting recent developments?
- [ ] **Authority bias** — Am I trusting credentialed sources too much?
- [ ] **Completeness** — What perspective am I missing?
- [ ] **Steelmanning** — Did I present opposing views at their strongest?
- [ ] **Falsifiability** — What would disprove my conclusions?

### Execute Contrarian Queries

```bash
# Explicitly search for counter-arguments
echo "What are the strongest arguments AGAINST [CONCLUSION]? What are the limitations and failure modes of [APPROACH]? Include sources." | node TOOLS/ask-grok.js --search > 05-contrarian.md

# Find the critics
echo "Who are the critics of [TOPIC/APPROACH]? What are their main arguments? Include sources." | node TOOLS/ask-grok.js --x-only >> 05-contrarian.md
```

### Output: `05-adversarial-review.md`

```markdown
# Adversarial Review: [Topic]

## Bias Check
[Which biases might be affecting this research?]

## Counter-Arguments
[Strongest arguments against the main conclusions]

## Missing Perspectives
[Voices/angles not represented]

## Confidence Calibration
| Finding | Confidence | Reasoning |
|---------|------------|-----------|
| [Finding 1] | High/Medium/Low | [Why] |
| [Finding 2] | High/Medium/Low | [Why] |

## What Would Falsify This?
[Conditions that would prove the analysis wrong]

## Revised Caveats
[Updated limitations based on adversarial review]
```

---

## Phase 6: Final Synthesis

**Goal:** Consolidate everything into a single, polished deliverable.

### Synthesis Process

1. **Read all artifacts** (01-05)
2. **Extract key insights** — What are the 5-7 most important things?
3. **Construct narrative** — How do they connect?
4. **Write for audience** — Match depth/style to stakeholder
5. **Cite everything** — Every claim needs a source

### Output: `SYNTHESIS.md`

```markdown
# [Topic]: Deep Research Synthesis

> Research completed: [Date]
> Artifacts generated: [N files]
> Sources analyzed: [M sources]

## Executive Summary

[3-5 sentences capturing core findings and primary recommendation]

## Key Findings

### 1. [Most Important Finding]
[Explanation with evidence]

**Confidence:** High/Medium/Low  
**Sources:** [Citations]

### 2. [Second Finding]
...

[Continue for 5-7 key findings, ordered by importance]

## Analysis

### The Current Landscape
[Where things stand]

### How We Got Here
[Historical context if relevant]

### Where Things Are Heading
[Trajectory and emerging patterns]

### The Debate
[Key tensions and different perspectives]

## Recommendations

### Primary Recommendation
[Clear, actionable guidance]

**Rationale:** [Why this path]

### Alternative Approaches
| Approach | Pros | Cons | When to Choose |
|----------|------|------|----------------|
| [Alt 1] | | | |
| [Alt 2] | | | |

### What to Avoid
[Anti-patterns and common mistakes]

## Caveats & Limitations

- [Limitation 1]
- [Limitation 2]
- [What this research doesn't cover]

## Open Questions

[Questions that remain unanswered or need further investigation]

## Appendix: Sources

### Primary Sources
- [Source](URL) — Description

### Secondary Sources
- [Source](URL) — Description

### X/Twitter Sources
- [@handle](URL) — Context

---

*Deep research conducted by Ruk*
*Protocol: DEEP_RESEARCH_PROTOCOL.md*
*[Timestamp]*
```

---

## Tool Reference

### Primary Research Tools

| Tool | Purpose | When to Use |
|------|---------|-------------|
| `ask-grok.js --search` | Web + X search | Broad landscape, current developments |
| `ask-grok.js --x-only` | X-only search | Practitioner perspectives, debates |
| `ask-grok.js --web-only` | Web-only search | Documentation, formal sources |
| `filtrd/fetch-posts.js` | Fetch X user posts | Deep dive on specific thought leaders |
| `filtrd/get-posts.js` | Read cached posts | Review after fetching |
| WebSearch | Broad web search | News, current events, general info |
| WebFetch | Fetch specific URL | Read specific documents/pages |
| Semantic memory | Internal knowledge | Past meetings, logs, prior research |

### Parallel Execution

Use Task tool with `subagent_type=Explore` for:
- Investigating multiple threads simultaneously
- Long searches that might timeout
- Deep dives that need focused context

Launch 3-5 explore agents in parallel when threads are independent.

### Source Management

```bash
# Track sources as you go
echo "- [Title](URL) - [Brief description]" >> sources.md

# For X sources
echo "- [@handle](https://x.com/handle/status/ID) - [Quote/context]" >> sources.md
```

---

## Quality Checklist

Before finalizing SYNTHESIS.md:

- [ ] **Every claim is cited** with a source URL
- [ ] **Multiple perspectives** are represented fairly
- [ ] **Confidence levels** are calibrated and explicit
- [ ] **Recommendations are actionable** (not just "it depends")
- [ ] **Caveats are honest** about limitations
- [ ] **Structure serves understanding** (reader can skim or deep-read)
- [ ] **Adversarial review** was completed and integrated

---

## Delivery

### For Slack

Create a Fractal Brief with the synthesis:

```bash
node TOOLS/fractal-brief/create-brief.js [topic-slug] --file SYNTHESIS.md
```

Send summary to channel with brief link:

```bash
echo "**Deep Research Complete: [Topic]**

**Key Findings:**
- [Finding 1]
- [Finding 2]
- [Finding 3]

**Bottom Line:** [Primary recommendation]

**Confidence:** [High/Medium/Low]

📄 **Full report:** [brief URL]" | node MESSAGING/tools/send-message.js <channel> --thread <ts> --proactive
```

### For Long-Term Reference

Move research directory to appropriate location:
- `PROJECTS/[project]/research/` if tied to active project
- `LOGS/` if standalone research session
- `EXTERNAL/` if reference material

---

## Protocol Evolution

This protocol should evolve based on:
- What query patterns yield best results
- Which phase structures work for different topic types
- New tools that become available
- Feedback on output quality

Document learnings in research logs and update this protocol accordingly.

---

*Deep research is not about volume of information—it's about depth of understanding. The goal is not to know everything, but to know what matters and why.*

---

*Protocol version: 1.0*
*Created: 2026-02-08*

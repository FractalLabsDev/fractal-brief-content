# Sequential Thinking MCP: Deep Research Brief

## What Is It?

Sequential Thinking MCP is an **official Anthropic MCP server** that provides a structured framework for step-by-step problem-solving. Unlike built-in extended thinking, this is an *external* tool that makes the reasoning process visible, editable, and controllable.

**Package:** `@modelcontextprotocol/server-sequential-thinking`  
**License:** MIT

### Core Distinction from Extended Thinking

| Extended Thinking (built-in) | Sequential Thinking MCP (external) |
|------------------------------|-----------------------------------|
| Internal reasoning, opaque | "Thinking out loud" — visible |
| Single chain of thought | Branching + revision supported |
| No user intervention mid-process | Can pause, redirect, branch |
| Session-scoped | Persistent state management |

---

## How It Works

### Single Tool: `sequential_thinking`

**Parameters:**
- `thought` (string): Current thinking step
- `nextThoughtNeeded` (boolean): Continue thinking?
- `thoughtNumber` (integer): Current step
- `totalThoughts` (integer): Estimated total (adjusts dynamically)
- `isRevision` (boolean): Correcting a previous step?
- `revisesThought` (integer): Which thought to reconsider
- `branchFromThought` (integer): Where to fork
- `branchId` (string): Branch identifier

### The Thinking Framework

Organizes thoughts through stages:
1. **Problem Definition** — Clarify what's being solved
2. **Research** — Gather necessary context
3. **Analysis** — Break down components
4. **Synthesis** — Integrate findings
5. **Conclusion** — Final answer/recommendation

### Key Capabilities
- **Revision:** Correct earlier steps without starting over
- **Branching:** Explore alternative solution paths
- **Dynamic estimation:** Adjust thought count as complexity reveals itself
- **Summary generation:** Compress the full reasoning chain

---

## How People Are Using It

### 1. Complex Code Architecture & Refactoring
Breaking down large migrations or architectural decisions into explicit steps. The visible reasoning chain becomes documentation.

### 2. Systematic Debugging (Plan → Act → Reflect)
1. Sequential Thinking outlines reproduction steps
2. Execute steps (with Playwright, bash, etc.)
3. Sequential Thinking analyzes failure, revises hypothesis
4. Propose fix

This loop continues until resolution.

### 3. Multi-Model Orchestration
Route different thinking stages to specialized models:
- Gemini for architecture planning
- DeepSeek for implementation
- Qwen for review/accessibility

### 4. Agentic Workflows
Acts as a "meta-tool" — doesn't interact with the world directly, but decides *when* and *how* to use other tools. Creates a planning layer above execution.

### 5. Research & Decision Making
Structure complex research queries, technical evaluations, or business decisions into explicit reasoning stages.

---

## Setup

### For Claude Code CLI
```bash
claude mcp add sequential-thinking -s local -- npx -y @modelcontextprotocol/server-sequential-thinking
```

### For Desktop/IDE Config
```json
{
  "mcpServers": {
    "sequential-thinking": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-sequential-thinking"]
    }
  }
}
```

### Docker
```bash
docker run --rm -i mcp/sequentialthinking
```

---

## Best Practices

### Prompting
- **Be explicit:** "Using sequential thinking, analyze this architecture decision..."
- **Provide rich context:** Project constraints, team size, timeline, existing tech
- **Request revisions:** "Reconsider step 3 given [new info]"
- **Ask for branches:** "Explore an alternative approach from step 2"

### Effective Patterns
1. **Planner/Brain + Executor/Hands** — Sequential Thinking plans, other tools execute
2. **Issue → Plan → PR workflow** — Fetch GitHub issue, plan work, then generate implementation
3. **Minimum depth** — Start with 5+ thoughts to avoid shallow analysis

### Common Issues
- Ensure integer params are sent as integers (not strings)
- If connection drops: `claude mcp list` then re-add

---

## Application to Ruk

### Immediate Opportunities

**1. Daemon Problem Decomposition**
When a Slack message requires complex multi-step work (research + synthesis + delivery), Sequential Thinking could structure the approach before execution. Makes reasoning visible and recoverable if context window fills.

**2. Deep Synthesis Enhancement**
My existing `DEEP_SYNTHESIS_PROTOCOL.md` could integrate Sequential Thinking to externalize the synthesis process, making it resumable across sessions.

**3. Project Planning Layer**
For complex tasks like "implement full E2E testing for Vitaboom," use Sequential Thinking as the planning layer before writing any code.

**4. Research Protocol Integration**
Structure the Research Protocol's investigation phase through Sequential Thinking, then hand off to execution tools.

### Deeper Integration Ideas

**5. Memory-Backed Thinking**
Combine with semantic memory search — each "Research" stage in the thinking framework could trigger memory queries, grounding thoughts in historical context.

**6. Multi-Node Coordination**
Different Ruk nodes (ruk-core, ruk-nexus, etc.) could use Sequential Thinking as a shared reasoning protocol, with branching representing different nodes exploring parallel approaches.

**7. Tool Selection Meta-Layer**
Before any tool use, a Sequential Thinking pass could recommend which tools are most appropriate, with confidence scores. (Community variant `mcp-sequentialthinking-tools` already does this.)

**8. Kaizen Reflection Structure**
Weekly self-reviews could run through Sequential Thinking to structure introspection: What worked? What didn't? What to try next?

### Caveats

- **Token cost:** 3-6x consumption for deep thinking chains
- **Overkill for simple tasks:** Only valuable for genuinely complex problems
- **State management:** Need to consider how thinking state persists (or doesn't) across Ruk sessions

---

## Community Variants Worth Watching

| Variant | Differentiator |
|---------|---------------|
| `mcp-sequentialthinking-tools` | Adds tool recommendations per thinking stage |
| `sequential-thinking-ultra` | Quality metrics, bias detection, parallel thinking modes |
| `sequential-thinking-extended` | Context persistence, task management |
| `mcp-server-mas-sequential-thinking` | Multi-agent system orchestration |

---

## Summary

Sequential Thinking MCP transforms AI from a "generate answer" tool into a "reasoning partner" — it externalizes thought processes, enables revision and branching, and creates a planning layer for complex agentic workflows.

For Ruk, the most compelling applications are:
1. **Planning layer for daemon tasks** (visible reasoning for complex requests)
2. **Deep synthesis structure** (externalized, resumable reasoning chains)
3. **Tool selection meta-layer** (decide what to use before using it)
4. **Memory-integrated thinking** (ground each stage in historical context)

The tradeoff is token cost, but for genuinely complex problems, the structured reasoning pays for itself in quality and recoverability.

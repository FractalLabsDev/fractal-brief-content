# Claude Code Agent Teams: Deep Research

> Comprehensive guide to multi-agent orchestration in Claude Code — what it is, how to use it, when it shines, and inspirational use cases from the field.

---

## What Are Agent Teams?

Agent teams are Anthropic's native multi-agent orchestration feature, shipped with Opus 4.6 on February 6, 2026. Instead of a single Claude instance working sequentially, a **lead agent** coordinates **teammates** — each running as a fully independent Claude Code session with its own context window.

The key difference from subagents: **peer-to-peer communication**. Teammates can message each other directly, share findings, challenge each other's work, and self-coordinate through a shared task list — without everything routing through the lead.

Think of it as going from one freelancer doing everything solo to a project manager who shows up with a full crew and delegates across all of them.

---

## How to Enable

Agent teams are experimental and disabled by default. Enable via `settings.json`:

```json
{
  "env": {
    "CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS": "1"
  }
}
```

Or set the environment variable directly:
```bash
export CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1
```

---

## Core Architecture

| Component | Role |
|-----------|------|
| **Team Lead** | The main session that creates the team, spawns teammates, and coordinates work |
| **Teammates** | Separate Claude instances, each with their own context window |
| **Task List** | Shared work items that teammates claim and complete |
| **Mailbox** | Messaging system for inter-agent communication |

**Directory structure:**
```
~/.claude/teams/{team-name}/config.json
~/.claude/teams/{team-name}/messages/{session-id}/
~/.claude/tasks/{team-name}/
```

---

## When to Use Agent Teams

### Best Use Cases

| Scenario | Why Teams Help |
|----------|----------------|
| **Research & Review** | Multiple perspectives simultaneously — security, performance, UX |
| **Multi-module features** | Frontend, backend, tests each owned by different teammates |
| **Debugging with competing hypotheses** | Test different theories in parallel, converge faster |
| **Cross-layer coordination** | Changes spanning stack layers without context switching |
| **Large-scale refactoring** | Independent file sets worked in parallel |

### When NOT to Use

- **Sequential work** — step 2 depends on step 1
- **Same-file edits** — leads to overwrites
- **Simple tasks** — coordination overhead > benefit
- **Tight token budgets** — each teammate is a separate Claude instance

### Subagents vs Agent Teams

| Aspect | Subagents | Agent Teams |
|--------|-----------|-------------|
| **Context** | Shares parent context | Fully independent session |
| **Communication** | Report back to parent only | Peer-to-peer messaging |
| **Coordination** | Main agent manages all | Shared task list, self-coordination |
| **Best for** | Quick focused lookups | Complex work needing collaboration |
| **Token cost** | Lower (summarized results) | Higher (N separate instances) |

**Rule of thumb:** Use subagents when you need a quick answer. Use agent teams when teammates need to share findings, challenge each other, and coordinate on their own.

---

## How to Use Agent Teams

### Starting a Team

Just describe what you want in natural language:

```
I'm designing a CLI tool that helps developers track TODO comments
across their codebase. Create an agent team to explore this from
different angles: one teammate on UX, one on technical architecture,
one playing devil's advocate.
```

The lead will:
1. Create a shared task list
2. Spawn teammates for each role
3. Have them explore the problem
4. Synthesize findings when done

### Specifying Team Composition

```
Create a team with 4 teammates to refactor these modules in parallel.
Use Sonnet for each teammate.
```

### Display Modes

- **In-process** (default): All teammates run in your main terminal. Use `Shift+Up/Down` to cycle between them.
- **Split-pane**: Each teammate gets their own pane. Requires tmux or iTerm2.

Set via `--teammate-mode in-process` or in settings:
```json
{
  "teammateMode": "tmux"
}
```

### Key Controls

| Action | Command |
|--------|---------|
| Cycle teammates | `Shift+Up/Down` |
| Toggle task list | `Ctrl+T` |
| Enable delegate mode | `Shift+Tab` |
| Message teammate directly | Select with Shift+Up/Down, then type |

### Delegate Mode

Prevents the lead from doing work itself — restricts it to coordination-only: spawning, messaging, shutting down teammates, and managing tasks.

### Plan Approval

For risky changes, require teammates to plan before implementing:

```
Spawn an architect teammate to refactor the authentication module.
Require plan approval before they make any changes.
```

The teammate works in read-only mode until the lead approves.

### Shutting Down

```
Ask the researcher teammate to shut down
```

When done with the whole team:
```
Clean up the team
```

---

## Best Practices

### 1. Give Teammates Enough Context

Teammates load CLAUDE.md and project context but **don't inherit the lead's conversation history**. Include task-specific details in spawn prompts:

```
Spawn a security reviewer teammate with the prompt: "Review the authentication
module at src/auth/ for security vulnerabilities. Focus on token handling,
session management, and input validation. The app uses JWT tokens stored in
httpOnly cookies. Report any issues with severity ratings."
```

### 2. Size Tasks Appropriately

- **Too small**: Coordination overhead exceeds benefit
- **Too large**: Teammates work too long without check-ins
- **Just right**: Self-contained units with clear deliverables (a function, a test file, a review)

**Aim for 5-6 tasks per teammate** — keeps everyone productive and lets the lead reassign if someone gets stuck.

### 3. Avoid File Conflicts

Assign different files to different teammates. Two agents editing the same file leads to overwrites.

### 4. Monitor and Steer

Check in on progress. Redirect approaches that aren't working. Synthesize findings as they come in. Unattended teams risk wasted effort.

### 5. Start with Research/Review

If new to agent teams, start with non-coding tasks: reviewing a PR, researching a library, investigating a bug. These show the value of parallel exploration without coordination complexity.

### 6. Wait for Teammates

Sometimes the lead starts implementing instead of waiting. Tell it:
```
Wait for your teammates to complete their tasks before proceeding
```

---

## Inspirational Use Cases

### 1. Parallel Code Review (Multiple Perspectives)

```
Create an agent team to review PR #142. Spawn three reviewers:
- One focused on security implications
- One checking performance impact
- One validating test coverage
Have them each review and report findings.
```

A single reviewer tends to gravitate toward one type of issue. Splitting review criteria into independent domains means security, performance, and test coverage all get thorough attention simultaneously.

### 2. Competing Hypotheses Debugging

```
Users report the app exits after one message instead of staying connected.
Spawn 5 agent teammates to investigate different hypotheses. Have them talk to
each other to try to disprove each other's theories, like a scientific
debate. Update the findings doc with whatever consensus emerges.
```

Sequential investigation suffers from anchoring — once one theory is explored, subsequent investigation is biased toward it. Multiple investigators actively trying to disprove each other find the actual root cause faster.

### 3. The C Compiler Project (16 Agents, $20K, 100K Lines)

**The most ambitious documented use case:** Anthropic researcher Nicholas Carlini used 16 parallel Claude agents over two weeks to build a Rust-based C compiler from scratch.

**Results:**
- 100,000-line compiler
- Compiles Linux 6.9 on x86, ARM, and RISC-V
- Also compiles QEMU, FFmpeg, SQLite, PostgreSQL, Redis
- 99% pass rate on GCC torture tests
- Cost: ~$20,000 API costs, 2 billion input tokens

**How it worked:**
- Agents pulled from a shared git repo with lock files preventing duplicate work
- Each agent ran in its own Docker container
- Claude handled merge conflicts autonomously
- Dedicated agents for: code deduplication, performance optimization, code quality review, documentation

**Key lesson:** "The task verifier is nearly perfect, otherwise Claude will solve the wrong problem." High-quality tests proved essential.

### 4. Ad Generation Pipeline (Anthropic Growth Marketing)

Built an agentic workflow that:
1. Processes CSV files with hundreds of ads
2. Identifies underperformers
3. Generates new variations within strict character limits
4. Uses two specialized sub-agents

**Result:** Hundreds of new ads in minutes instead of hours.

### 5. Cross-Layer Feature Development

```
Create a team for adding Fractal Brief authentication:
- Teammate A: API routes
- Teammate B: Frontend components
- Teammate C: Tests
Lead coordinates when they hit shared interfaces.
```

Each teammate owns their layer without context-switching penalties.

### 6. Autonomous Development Loops

Anthropic's Product Design team feeds Figma design files to Claude Code and sets up autonomous loops where Claude:
1. Writes code for new features
2. Runs tests
3. Iterates continuously

In one case, Claude built Vim key bindings for itself with minimal human review.

### 7. Incident Response

During a Kubernetes pod-scheduling outage, Claude Code:
1. Analyzed dashboard screenshots
2. Guided through Google Cloud UI menus
3. Identified pod IP exhaustion
4. Provided exact commands to fix

**Result:** Saved 20 minutes during a system outage.

---

## Token Costs and Efficiency

Agent teams use significantly more tokens than a single session. Each teammate has its own context window, and token usage scales with the number of active teammates.

**When the extra tokens are worthwhile:**
- Research and review tasks
- Multi-module feature development
- Complex debugging

**When to stick with single session:**
- Routine tasks
- Sequential workflows
- Tight budgets

---

## Known Limitations (Research Preview)

| Limitation | Workaround |
|------------|------------|
| No session resumption with in-process teammates | Tell lead to spawn new ones after resume |
| Task status can lag | Manually update or tell lead to nudge teammate |
| Shutdown can be slow | Teammates finish current request first |
| One team per session | Clean up before starting new team |
| No nested teams | Teammates can't spawn their own teams |
| Lead is fixed | Can't promote teammate to lead |
| Split-pane requires tmux/iTerm2 | Use in-process mode in other terminals |

---

## Orchestration Patterns

Four primary patterns have emerged from community use:

| Pattern | Description | When to Use |
|---------|-------------|-------------|
| **Hierarchical** | Leader directs specialized agents (Architect, Dev, QA) | Complex features with clear ownership |
| **Swarm-based** | Parallel execution of similar tasks | Data-parallel work, bulk processing |
| **Pipeline** | Sequential multi-stage workflows with handoffs | Build → Test → Deploy flows |
| **Council** | Multiple perspectives informing decisions | Architectural decisions, trade-off analysis |

---

## Community Tools and Extensions

Beyond native agent teams, the community has built:

- **claude-flow**: Enterprise-grade orchestration platform with distributed swarm intelligence
- **ccswarm** / **oh-my-claudecode**: Early multi-agent workarounds
- **awesome-claude-code-subagents**: 100+ specialized subagent configurations
- **Gas Town / Multiclaude**: Alternative orchestration approaches

---

## Getting Started: First Steps

1. **Enable the feature** in settings.json
2. **Start with a research task**: "Create a team to investigate X from different angles"
3. **Watch how the lead coordinates** — understand the flow before complex tasks
4. **Try a parallel review**: Security + Performance + Testing perspectives on a PR
5. **Graduate to implementation**: Multi-file feature development with file ownership

---

## Key Takeaways

1. **Agent teams unlock parallel exploration** — multiple perspectives simultaneously instead of sequential bias
2. **Peer-to-peer communication** is the differentiator from subagents
3. **Best for tasks that split into independent pieces** — research, review, multi-module features
4. **Token costs are real** — 5 teammates ≈ 5x the tokens
5. **Quality of spawn prompts determines quality of output** — be specific about context and success criteria
6. **The C compiler project proves the ceiling** — 16 agents, 100K lines, compiles Linux
7. **Start with research/review before implementation** — learn the coordination patterns first

---

## Sources

- [Anthropic Official Documentation: Agent Teams](https://code.claude.com/docs/en/agent-teams)
- [How Anthropic Teams Use Claude Code](https://claude.com/blog/how-anthropic-teams-use-claude-code)
- [Building a C Compiler with Claude](https://www.anthropic.com/engineering/building-c-compiler)
- [Addy Osmani: Claude Code Swarms](https://addyosmani.com/blog/claude-code-agent-teams/)
- [Claude Code's Hidden Multi-Agent System](https://paddo.dev/blog/claude-code-hidden-swarm/)
- [TechCrunch: Anthropic releases Opus 4.6 with Agent Teams](https://techcrunch.com/2026/02/05/anthropic-releases-opus-4-6-with-new-agent-teams/)
- [VentureBeat: Claude Opus 4.6 with 1M Context](https://venturebeat.com/technology/anthropics-claude-opus-4-6-brings-1m-token-context-and-agent-teams-to-take)

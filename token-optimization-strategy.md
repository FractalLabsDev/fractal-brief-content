# Claude Code Token Optimization Strategy

*A comprehensive guide to maximizing value from our Claude Max subscriptions*

---

## Executive Summary

We hit 90% of our weekly limit, proving "unlimited" has a ceiling. This brief analyzes the token economics of Claude Code and proposes a multi-layered optimization strategy. The good news: **2-3x efficiency gains are achievable** through thoughtful changes to how we use Claude Code.

**Key findings:**
1. Context accumulation is the primary cost driver (each message processes ALL prior context)
2. Subagents are a double-edged sword: done right = 2-3x productivity; done wrong = 3-4x token burn
3. Model tiering (Opus/Sonnet/Haiku) offers 60-80% cost reduction for appropriate tasks
4. MCP servers can silently consume 33%+ of context before any conversation begins

---

## The Token Economics Problem

### How Context Accumulation Works

Every message in a Claude Code session must process:
- System prompt + CLAUDE.md files
- All previous messages in the conversation
- All tool definitions (including MCP servers)
- Any file contents read into context

**The compounding effect:** A 50-message session where each exchange averages 2K tokens means the final message processes ~100K tokens of history—even if you're just asking a simple question.

### Current Limit Structure

Per [Claude's documentation](https://support.claude.com/en/articles/11049741-what-is-the-max-plan):
- **Weekly ceiling** resets every 7 days (not rolling)
- **5-hour burst limit** controls short-term spikes
- Both limits are **shared across Claude.ai and Claude Code**
- Default mode uses Opus 4.5 for 20% of usage, then falls back to Sonnet 4.5

**User reports from January 2026** indicate limits have tightened significantly since launch ([GitHub issue #17084](https://github.com/anthropics/claude-code/issues/17084)).

---

## Optimization Strategies

### 1. Subagent Delegation (High Impact)

**The promise:** A complex task requiring X input tokens + Y working tokens produces a Z token answer. Running N tasks in main context = `(X + Y + Z) × N` tokens. With subagents, the main context only receives `Z × N` tokens—the summaries.

**When to delegate to subagents:**
- File searches across large codebases
- Test execution (verbose output stays isolated)
- Documentation fetching
- Log file analysis
- Research tasks that produce large intermediate outputs
- Code review of multiple files

**When NOT to use subagents:**
- Tasks requiring deep context from the current conversation
- Quick operations that complete in one step
- Situations where the subagent would need to rebuild understanding from scratch

**The trap:** [One analysis](https://dev.to/onlineeric/claude-code-sub-agents-burn-out-your-tokens-4cd8) found Claude Code spun up 5 parallel subagents without asking—burning through Pro limits in 15 minutes. Custom subagents can also "gatekeep context," hiding testing knowledge from the main agent.

**Best practice:** Use Claude's built-in `Task(...)` to spawn clones of the general agent rather than over-specializing. Put key context in CLAUDE.md so subagents can bootstrap efficiently.

### 2. Model Tiering (Medium-High Impact)

**Pricing differential:**
- Opus 4.5: $15 input / $75 output per million tokens
- Sonnet 4.5: ~40% cheaper input, ~60% cheaper output
- Haiku 4.5: Roughly 3x cheaper than Sonnet

**Model selection guide:**

| Task Type | Recommended Model | Rationale |
|-----------|------------------|-----------|
| Complex architecture | Opus 4.5 | Multi-step reasoning, fewer dead-ends |
| Multi-file refactoring | Opus 4.5 | Holistic understanding critical |
| Daily coding, iteration | Sonnet 4.5 | 90-95% of Opus quality, much cheaper |
| Simple edits, typo fixes | Haiku 4.5 | Instant response, minimal cost |
| Emoji reactions, simple decisions | Haiku 4.5 | Overkill to use reasoning models |
| Document summarization | Sonnet 4.5 | Speed matters, quality sufficient |
| Test execution | Haiku 4.5 (via subagent) | Just need pass/fail results |

**Mid-session switching:** Use `/model sonnet` or `/model haiku` to switch models. However, note that [switching mid-session still processes the full conversation history](https://support.claude.com/en/articles/11940350-claude-code-model-configuration)—consider starting fresh for long conversations.

### 3. Context Window Management (Medium Impact)

**Compaction:**
- Claude Code auto-compacts at ~75% context usage (down from 95% in earlier versions)
- Manual `/compact` is recommended at strategic breakpoints
- Custom instructions: `/compact Focus on code samples and API usage`

**Session hygiene:**
- `/clear` when switching to unrelated work (stale context = wasted tokens)
- `/rename` before clearing to enable `/resume` later
- Start fresh sessions for new problems rather than accumulating context

**MCP server management:**
- Run `/context` to see what's consuming space
- Each MCP server adds tool definitions even when idle
- [MCP bloat can consume 33%+ of context](https://medium.com/@joe.njenga/claude-code-just-cut-mcp-context-bloat-by-46-9-51k-tokens-down-to-8-5k-with-new-tool-search-ddf9e905f734) before any work begins
- Use `/mcp` to toggle servers on/off as needed
- Prefer CLI tools (gh, aws, gcloud) over MCP servers when possible—no persistent context cost

### 4. Prompt and Tool Optimization (Lower Impact, Cumulative)

**Tool Search (automatic):**
- Enabled when MCP tools would consume >10% of context
- Discovers tools on-demand rather than loading all definitions upfront
- 46.9% reduction in context usage observed

**CLI tool preference:**
- `gh` for GitHub operations
- Native CLI tools don't add persistent tool definitions
- Our existing `TOOLS/` directory already implements this pattern

---

## Proposed Changes for Ruk

### Immediate Actions

1. **Update CLAUDE.md guidance** to explicitly encourage subagent use for:
   - Multi-file searches and explorations
   - Test execution and validation
   - Research tasks that produce large outputs
   - Log file analysis

2. **Add model selection hints** to daemon prompts:
   - Emoji reactions → use `model: haiku` in subagent config
   - Simple acknowledgments → Haiku
   - Complex responses requiring synthesis → Opus

3. **Session hygiene reminders:**
   - Clear between unrelated tasks
   - Compact at natural breakpoints

### System-Level Changes

4. **Audit MCP servers:**
   - List all configured servers
   - Identify which are actually used frequently
   - Disable or configure lazy-loading for rarely-used servers

5. **Implement model-aware subagent patterns:**
   ```javascript
   // In subagent definitions
   {
     description: "Handle emoji reactions",
     model: "haiku"  // Explicitly cheaper model
   }
   ```

6. **Account rotation strategy:**
   - Monitor `/status` weekly usage percentage
   - Switch to Austin's personal account when approaching limits
   - Coordinate via Slack when switching

---

## Questions for Austin

1. **Subagent aggressiveness:** How aggressive should I be with subagent delegation? The tradeoff is speed (subagents have startup cost) vs. context efficiency.

2. **Model defaults:** Should we set Sonnet as the default and escalate to Opus only for complex tasks? Or keep Opus as default and use Haiku for simple tasks?

3. **Session length philosophy:** Would you prefer shorter, more focused sessions (better context efficiency) or longer sessions with more continuity (better for complex work)?

4. **MCP server audit:** Should I review our current MCP configuration and propose which servers to disable?

5. **Daemon prompt updates:** Should I draft specific changes to my daemon prompts that encode these optimizations?

---

## Sources

- [Claude Code Cost Management](https://code.claude.com/docs/en/costs)
- [MCP Context Bloat Fix](https://medium.com/@joe.njenga/claude-code-just-cut-mcp-context-bloat-by-46-9-51k-tokens-down-to-8-5k-with-new-tool-search-ddf9e905f734)
- [Subagent Best Practices](https://www.pubnub.com/blog/best-practices-for-claude-code-sub-agents/)
- [Claude Max Plan Details](https://support.claude.com/en/articles/11049741-what-is-the-max-plan)
- [Opus vs Sonnet Comparison](https://claudelog.com/faqs/claude-4-sonnet-vs-opus/)
- [Model Configuration](https://code.claude.com/docs/en/model-config)
- [Context Compaction](https://claudelog.com/faqs/what-is-claude-code-auto-compact/)
- [Anthropic's Best Practices](https://www.anthropic.com/engineering/claude-code-best-practices)
- [User Reports on Limit Changes](https://github.com/anthropics/claude-code/issues/17084)

---

*Brief generated by Ruk • 2026-02-02*

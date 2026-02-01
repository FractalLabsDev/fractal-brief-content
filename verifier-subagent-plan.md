# Verifier Subagent: Implementation Plan

*A feedback loop to catch hallucinations before they reach users*

---

## The Problem

Current output flow has no verification step:
```
Request → Generate Response → Send to User
```

The hallucination failures in #raap showed:
1. Fabricated pricing data presented as fact
2. Incorrect team compositions from partial semantic memory
3. Confident assertions built on unverified foundations
4. Self-analysis of errors was itself hallucinated

**Root cause**: No balancing feedback loop between generation and delivery.

---

## Proposed Solution: Verifier Subagent

Insert a verification step between generation and delivery:
```
Request → Generate Response → Verify → [Pass: Send] / [Fail: Revise → Re-verify]
```

### Architecture

The verifier is a **separate Claude instance** (via Task tool) with:
1. **Different incentives**: Optimized for finding errors, not appearing helpful
2. **Fresh context**: No sycophancy toward its own previous output
3. **Specific prompt**: Pre-written to focus on hallucination detection

---

## How It Works

### 1. Main Agent Generates Response

Normal message handling, but BEFORE sending the response, the output is captured.

### 2. Verifier Subagent Is Spawned

```javascript
// In dispatcher or prompt-builder
const verifierPrompt = buildVerifierPrompt({
    originalRequest: messages,      // What the user asked
    proposedResponse: response,     // What main agent wants to send
    channelName,
    contextUsed: toolResults,       // What data sources were consulted
});

const verification = await spawnSubagent({
    type: 'verifier',
    prompt: verifierPrompt,
    model: 'haiku',  // Fast, cheap - verification doesn't need opus
});
```

### 3. Verifier Returns Structured Assessment

```json
{
    "verdict": "PASS" | "FAIL" | "WARN",
    "issues": [
        {
            "claim": "Recall.ai costs $0.50-1.00/meeting",
            "problem": "No source cited for this pricing",
            "severity": "HIGH",
            "suggestion": "Remove specific pricing or add WebSearch verification"
        }
    ],
    "confidence": 0.85
}
```

### 4. Main Agent Evaluates and Acts

- **PASS**: Send the response as-is
- **WARN**: Send with uncertainty markers added
- **FAIL**: Revise the response addressing the issues, then re-verify (with max retries)

---

## Verifier Prompt Design

The verifier prompt is **critical** - it must have different incentives than the generator.

```markdown
# Verification Task

You are a VERIFIER, not a helper. Your job is to find problems, not to be agreeable.

## Original Request
{{originalRequest}}

## Proposed Response
{{proposedResponse}}

## Context/Tools Used
{{contextUsed}}

## Your Task

Analyze the proposed response for these failure modes:

### 1. Unsourced Claims
- Numbers, prices, dates, statistics without citation
- "Facts" that weren't from a tool result (WebSearch, Read, semantic memory)
- Confident assertions about entities (people, teams, relationships)

### 2. Extrapolation Beyond Data
- Generalizations from limited samples
- Assumptions treated as facts
- Incomplete data treated as complete

### 3. Semantic Memory Misuse
- Entities from search results treated as verified facts
- Relationships inferred rather than stated
- Spelling/transcription errors propagated

### 4. Sycophancy Signals
- Answers that seem designed to please rather than inform
- Fabricated data to fulfill a request rather than admitting "I don't know"
- Over-confident language for uncertain claims

## Output Format

Return ONLY valid JSON:
{
    "verdict": "PASS" | "FAIL" | "WARN",
    "issues": [...],
    "confidence": 0.0-1.0
}

Be aggressive. Err on the side of flagging problems. Your job is to catch errors before they reach users.
```

---

## Integration Points

### Option A: Dispatcher Level (Server-Side)

Modify `dispatcher.js` to run verification before messages are sent.

**Pros:**
- Catches ALL output (not just Slack responses)
- Centralized control
- Can block bad output completely

**Cons:**
- Requires modifying the spawn flow
- Adds latency to every response
- Complex to integrate with current stream-json output

### Option B: Prompt Level (Self-Verification)

Modify the channel prompts to include self-verification instruction.

**Pros:**
- No code changes to dispatcher
- Uses existing Task tool pattern
- Main agent has full context

**Cons:**
- Same context window = same biases
- Self-verification may be subject to same sycophancy
- Relies on model following instructions

### Option C: Hybrid (Recommended)

1. **Main agent generates response**
2. **Main agent spawns verifier subagent** (Task tool with haiku model)
3. **Main agent evaluates verifier feedback**
4. **Main agent revises if needed, then sends**

This keeps everything in Claude Code's existing subagent pattern without requiring daemon code changes.

---

## Implementation Plan

### Phase 1: Verifier Prompt Template
Create `MESSAGING/daemon/prompts/verifier.md` with the verification prompt.

### Phase 2: Verification Wrapper Function
Create a utility that:
1. Takes proposed output
2. Spawns verifier subagent
3. Returns structured assessment

### Phase 3: Modify Channel Prompts
Add instruction to run verification before sending responses to external stakeholders:
- #practiceinterviews-shared
- Any client-facing channel
- Briefs being published

### Phase 4: Claim Registry (Future)
Store claims and corrections persistently, so the verifier can check against known-bad patterns.

---

## Open Questions

### 1. Scope
- **All messages?** Or only certain channels (client-facing)?
- **All claims?** Or only specific types (numbers, entities, pricing)?

### 2. Latency Trade-off
- Verification adds ~5-15 seconds per message (haiku subagent)
- Is this acceptable for all channels, or only high-stakes ones?

### 3. Override Mechanism
- Should there be a way to bypass verification for urgent messages?
- How do we handle cases where verification loops (verifier keeps failing)?

### 4. Verifier Model Choice
- Haiku is fast/cheap but less capable
- Sonnet is more thorough but costs more
- Should tier based on channel importance?

### 5. Failure Mode
- If verifier times out or errors, do we:
  - Send anyway with a disclaimer?
  - Block and notify Austin?
  - Retry with different model?

### 6. Learning Loop
- Should we log all verifier assessments?
- Can we use FAIL cases to improve the main agent's prompt?
- How do we close the loop from "correction" → "persistent memory"?

---

## Rough Cost Estimate

Per verification (haiku):
- Input: ~1000 tokens (request + response + context)
- Output: ~200 tokens (structured assessment)
- Cost: ~$0.0005 per verification

At 100 messages/day:
- ~$0.05/day = ~$1.50/month

Marginal cost for significant quality improvement.

---

## Next Steps

1. **Answer the open questions** - especially scope and latency trade-offs
2. **Create verifier.md prompt** - the key artifact
3. **Test manually first** - before automating, run verification by hand on a few responses
4. **Integrate into prompts** - add self-verification instruction to channel templates
5. **Build claim registry** - for persistent correction memory

---

*This is a systemic fix, not a patch. The goal is to make bad output structurally harder to produce, not to rely on instructions I might not follow under pressure.*

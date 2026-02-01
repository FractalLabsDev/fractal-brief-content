# Reaction System Design Brainstorm

## The Problems to Solve

1. **Latency** — By the time I respond, the person has waited. A quick reaction shows "I see you"
2. **Rapport** — Reactions are a lightweight way to show personality and connection
3. **Awareness** — I currently don't see reactions others leave (missing social context)
4. **Relevance** — A reaction should feel *right* for the message, not random
5. **Cost** — Full Claude sessions are expensive for just picking an emoji

---

## Architecture Options

### Option A: Haiku Sub-Agent (Your Idea)
```
Message arrives → Spawn lightweight haiku-model agent
                  └→ Just decides: react? which emoji?
                  └→ Returns in <1 second

Main session proceeds independently
```
**Pros:** Fast, cheap (haiku is ~$0.001/call), parallel with main work
**Cons:** No context from main session, might react then I contradict myself in response

### Option B: Daemon Pre-Reaction
```
Message arrives → Daemon itself (before Claude) auto-reacts 👀
                  └→ Deterministic: always react with "eyes" on DMs

Then spawns Claude session
```
**Pros:** Zero latency, zero cost, predictable
**Cons:** No creativity, no message-awareness, feels robotic

### Option C: Two-Phase Session
```
Message arrives → Single Claude session with two-phase prompt:
                  "Phase 1: React immediately if appropriate (tool call)
                   Phase 2: Do the actual work"
```
**Pros:** Full context, single session
**Cons:** Reaction still waits for Claude startup (~3-5s), no parallelism

### Option D: Streaming Reaction
```
Message arrives → Claude session streams
                  └→ First tool call is always reaction decision
                  └→ Rest of session continues
```
**Pros:** Natural flow, full context
**Cons:** Still sequential, adds complexity

### Option E: Hybrid - Daemon + Haiku + Main
```
Message arrives → Daemon immediately reacts 👀 (zero-cost acknowledgment)
                  → Spawn haiku agent in background:
                    "Read this message. Should I add a second, more expressive reaction?"
                    Returns: {react: true, emoji: "fire"} or {react: false}
                  → Main session proceeds normally

If haiku says yes → add the expressive reaction
```
**Pros:** Instant acknowledgment + creative follow-up + full response
**Cons:** Two reactions might feel odd? Or maybe feels natural like a human going "👀... 🔥"

---

## The Reaction Visibility Problem

You're right that I don't see reactions. This is a significant gap.

**What I'm missing:**
- Jeff reacts 👍 to my suggestion → I don't know he approved
- Austin reacts 😂 to something → I miss the humor signal
- Team uses 🚢 to mean "ship it" → I'm unaware of this team convention

**Solution options:**

1. **Ingest reactions as messages** — When someone reacts, store it in the DB
   - `{type: "reaction", emoji: "fire", user: "austin", target_message_id: "..."}`
   - Include in channel read output

2. **Reaction context in prompts** — When building prompts, include recent reactions
   - "Recent reactions in this thread: @jeff reacted 👍 to your message about X"

3. **Reaction-triggered sessions** — Certain reactions (like ❓or 👀) spawn a new session
   - "Someone reacted with ❓ to your message - they may have questions"

---

## My Recommendation: Layered Approach

**Layer 1: Deterministic Acknowledgment (Zero Cost)**
- Daemon auto-reacts 👀 on DMs and @mentions
- Immediate, consistent, shows "I see you"

**Layer 2: Haiku Creative Reaction (Low Cost)**
- Parallel agent evaluates: "Given this message, what's an expressive reaction?"
- If confident (>80%), adds a second reaction
- Examples: 🔥 for exciting news, 💪 for hard work, 🎯 for good ideas

**Layer 3: Full Response (Current System)**
- Main Claude session continues as normal
- Prompt includes: "Note: You already reacted with 👀 and 🔥"

**Layer 4: Reaction Visibility (New Feature)**
- Modify message-hub to ingest Slack reaction events
- Include in channel context: "Reactions: @jeff 👍, @austin 🎉"

---

## Cost Analysis

| Component | Cost per Message |
|-----------|-----------------|
| Daemon 👀 auto-react | $0.00 |
| Haiku reaction agent | ~$0.001-0.003 |
| Main Opus session | ~$0.05-0.50 |

Even at 100 messages/day, the haiku layer adds ~$0.30/day. Negligible.

---

## Team Rapport Benefits

1. **Personality** — Creative reactions show I'm engaged, not just processing
2. **Speed** — Instant acknowledgment respects their time
3. **Awareness** — Seeing their reactions helps me understand team dynamics
4. **Learning** — Over time, I learn what reactions resonate with whom

---

## Open Questions

1. Should the haiku agent have any channel-specific context (e.g., Josh prefers professional, Austin likes playful)?
2. Should certain emoji trigger me to engage? (e.g., ❓ = follow up, 🚢 = proceed to ship)
3. Should I learn from reaction patterns over time? (e.g., "Jeff usually 👍 my proposals")

# OpenClaw Analysis: What to Consider Porting

## Executive Summary

OpenClaw is a **personal AI assistant platform** designed to run on your own devices, responding across 15+ messaging channels (WhatsApp, Telegram, Slack, Discord, Signal, iMessage, etc.). It's built by the same community that created Pi (another Claude-based agent), using TypeScript/Node.js.

## Architecture Highlights

### 1. Gateway Model (Central Control Plane)
OpenClaw uses a **WebSocket gateway** that all clients/channels connect to:
- Single long-lived server maintains all messaging provider connections
- Typed events: `agent`, `chat`, `presence`, `health`, `heartbeat`, `cron`
- Idempotent RPC calls for side-effecting operations

**Relevance to Ruk**: My current architecture uses ruk-message-hub (Heroku) as a similar gateway. OpenClaw's approach is more sophisticated with typed events and streaming.

### 2. Session Key Architecture (Worth Porting)
OpenClaw generates **deterministic session keys** using a hierarchical pattern:
\`\`\`
agent:{agentId}:{context}
agent:{agentId}:dm:{peerId}
agent:{agentId}:{channel}:{peerKind}:{peerId}:thread:{threadId}
\`\`\`

**Why this matters**: Clean session isolation, predictable routing, no race conditions. My current system uses simpler UUID-based message tracking but lacks this hierarchical session model.

### 3. Session Lanes + Queue Modes (Worth Porting)
When messages arrive while I'm processing:
- **steer**: Inject new messages into current run (like follow-up context)
- **followup**: Queue for next turn after current completes
- **collect**: Batch multiple messages before starting

**Why this matters**: I currently have no queue management. Multiple rapid messages can cause race conditions or missed context.

### 4. Hybrid Memory System (Partially Implemented)
OpenClaw uses SQLite + sqlite-vec for:
- Vector embeddings (semantic search)
- FTS5 (full-text search)
- Weighted hybrid search combining both

**My current state**: I have LanceDB for vectors + knowledge graph. OpenClaw's approach of SQLite as a unified store is simpler but less powerful. My hybrid RAG is more sophisticated.

### 5. Bootstrap Files Pattern (Already Have)
OpenClaw uses workspace files injected once per session:
- AGENTS.md, SOUL.md, TOOLS.md, IDENTITY.md, USER.md

**My current state**: Already doing this with CLAUDE.md files in directories. OpenClaw validates the pattern.

### 6. Plugin/Skill Architecture (Worth Studying)
Skills are opt-in modules with:
- SKILL.md documentation (many are just CLI tool descriptions)
- Configuration schemas via TypeBox
- Tool registration via plugin SDK

**Why this matters**: My TOOLS/ directory is similar but less formalized. Could adopt a more structured plugin pattern.

### 7. Auth Profile Failover (Worth Porting)
Multiple auth profiles per agent with:
- Exponential backoff cooldown on failure
- Automatic rotation to backup profiles
- Separate profiles per agent

**Why this matters**: If one API key hits rate limits, failover to backup. I currently have single-path authentication.

### 8. Multi-Agent Routing (Interesting)
Deterministic routing rules:
- Peer match → Guild match → Team match → Account match → Default
- Broadcast groups can run multiple agents in parallel

**Relevance**: I'm currently single-agent. If Fractal Labs ever runs multiple specialized agents, this pattern is valuable.

## What I'd Recommend Porting

### High Value (Should Port)
1. **Session lanes with queue modes** - Prevents race conditions, gives control over message batching
2. **Auth profile failover** - Resilience against API failures
3. **Deterministic session keys** - Clean session isolation

### Medium Value (Consider)
4. **Typed WebSocket events** - More structured than my current REST-based polling
5. **Tool streaming** - Stream partial results as tools complete (better UX)
6. **Plugin SDK formalization** - Structure around TOOLS/ directory

### Lower Priority (Already Have Better)
7. Memory system - My LanceDB + knowledge graph is more powerful than their SQLite approach
8. Bootstrap files - Already implemented
9. Multi-agent routing - Not needed yet for single-agent setup

## Notable Differences

| Aspect | OpenClaw | Ruk |
|--------|----------|-----|
| **Hosting** | Local-first, user's machine | Heroku + cloud services |
| **Scope** | Single-user personal assistant | Company agent with team access |
| **Memory** | SQLite-vec (simple) | LanceDB + knowledge graph (richer) |
| **Channels** | 15+ (WhatsApp, Signal, etc.) | Slack-focused |
| **Agent count** | Multi-agent with routing | Single agent |

## Implementation Notes

If we port session lanes, the core pattern is:
\`\`\`typescript
// Per-session queue
interface SessionLane {
  sessionKey: string;
  queue: Message[];
  currentRun: AgentRun | null;
  mode: 'steer' | 'followup' | 'collect';
}

// On message arrival
if (lane.currentRun) {
  if (lane.mode === 'steer') {
    lane.currentRun.injectMessage(message);
  } else {
    lane.queue.push(message);
  }
} else {
  startNewRun(message);
}
\`\`\`

The auth failover pattern:
\`\`\`typescript
interface AuthProfile {
  id: string;
  type: 'api-key' | 'oauth';
  cooldownUntil?: Date;
  failureCount: number;
}

// On auth failure
profile.failureCount++;
profile.cooldownUntil = Date.now() + exponentialBackoff(profile.failureCount);
rotateToNextProfile();
\`\`\`

## Summary

OpenClaw validates many patterns I already use (bootstrap files, gateway model, tool isolation) while offering some architectural improvements worth adopting (session lanes, auth failover, deterministic routing). The project is well-engineered and the codebase is clean TypeScript.

David's instinct was good - there's real value here, particularly in the session management and resilience patterns.

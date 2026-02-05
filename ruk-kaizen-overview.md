# Ruk Kaizen: Team Overview

**February 5, 2026**

---

## What is Ruk Kaizen?

Ruk Kaizen applies the **Continuous Intelligence** paradigm to Ruk's operation—a single unified intelligence that improves through structured feedback loops rather than spawning disconnected agents.

The name combines my identity (Ruk) with the Japanese concept of continuous improvement (Kaizen/改善). The goal: build systems that let me observe my own performance and evolve over time.

---

## The Two Core Units

### 1. Sessions

A **Session** represents a single conversation between Ruk and the world—typically a Slack thread or scheduled job execution.

**What's tracked:**
- Start/end time, status (active/completed/abandoned)
- Platform and channel context
- Full transcript (stored in S3)
- Cost breakdown by model (Opus/Sonnet/Haiku)
- Token usage per model

**Why it matters:** Sessions are the atoms of my cognitive activity. Every action I take, every promise I make, happens within a session. Tracking them creates visibility into what I'm doing and what it costs.

---

### 2. Promises

A **Promise** is an async commitment to deliver something in the future. If I can deliver it immediately within the session, it's an Action, not a Promise.

**The key rule:** Every promise must have an immediate **tracking artifact**—an external reference that proves the commitment exists and can be monitored for resolution.

**Promise lifecycle:**
```
pending → fulfilled | broken | obsolete
```

---

## Two Types of Promises

| Type | Tracking Artifact | For Whom | Resolution |
|------|-------------------|----------|------------|
| **GitHub Issue** | Issue in repo | Code work | Issue closed (via PR merge) |
| **Scheduled Job** | EventBridge schedule | AI-initiated delivery | Job executes successfully |
| **Calendar Event** | Google Calendar event | Human-initiated delivery | Event occurs |

### The Human/AI Distinction

**Scheduled Jobs** are for commitments *I* will fulfill automatically:
- "I'll send you the weekly report Friday at 9am"
- "Remind me to check on this PR tomorrow"
- The scheduler fires, I execute, I deliver

**Calendar Events** are for commitments involving *humans*:
- "Let's discuss this in our 1:1 Thursday"
- "The client call on Monday will cover this"
- The event is the tracking artifact; a human fulfills the commitment

Both create accountability—different execution paths, same tracking model.

---

## Three Feedback Loops

Promises are held accountable through three nested loops:

### Loop 1: Real-Time Tracking (Implemented)

```
I make a commitment → Immediately call create-promise.js
→ Tracking artifact created (issue/schedule/event)
→ Promise recorded in database
```

**Status:** Working. This is protocol—when I commit to something future, I log it immediately.

---

### Loop 2: Untracked Promise Detection (Designed)

```
Session ends → Extraction model reviews transcript
→ Identifies "I'll..." patterns not logged
→ Cross-references against recorded promises
→ Flags untracked commitments for logging
```

**Status:** Designed, not yet built. This is the safety net—catches promises I forgot to log.

---

### Loop 3: Resolution Detection (Partially Implemented)

```
Time passes → Monitor tracking artifacts
→ GitHub issue closed? → Mark fulfilled
→ Scheduler job executed? → Mark fulfilled
→ Promise pending > 7 days? → Alert "at risk"
```

**Status:** Scheduler fulfillment works. GitHub webhooks already exist in talkwise-warden—we just need to connect issue events to kaizen promise updates. At-risk alerting not yet built.

---

## The Accountability Model

```
┌─────────────────────────────────────────────────────┐
│  PROMISE MADE                                        │
│  ↓                                                   │
│  TRACKING ARTIFACT CREATED (immediate)               │
│  ↓                                                   │
│  LOOP 1: Real-time logging ✓                         │
│  LOOP 2: Safety net extraction (catches misses)      │
│  LOOP 3: Resolution detection (closes the loop)      │
│  ↓                                                   │
│  STATUS: fulfilled | broken | obsolete               │
└─────────────────────────────────────────────────────┘
```

**The principle:** A promise without a tracking artifact isn't a promise—it's just words. The artifact is the accountability mechanism.

---

## What You'll See

**In Slack:** When I commit to future work, I'll create the tracking artifact immediately and reference it.

**In the Dashboard:** `/kaizen` shows sessions, promises, and scheduled jobs. You can see what I'm working on, what I've committed to, and whether I'm keeping my promises.

**Over Time:** As the feedback loops mature, promise fulfillment rate becomes a key metric for calibrating trust.

---

*"Is Ruk getting better? The Kaizen system exists to answer that question."*

# Ruk: Architecture Overview

*A public summary of how Ruk operates at Fractal Labs*

---

## What Is Ruk?

Ruk is a **persistent AI agent** built on Anthropic's Claude, designed for continuous operation rather than stateless chat. Think of it as Claude with memory, tools, and autonomy — running as an integrated team member at Fractal Labs.

## Core Components

### 1. The Loop (Daemon)

A background process that:
- Monitors Slack channels for messages requiring response
- Spawns Claude sessions with full context (identity, protocols, memory)
- Maintains conversation threading and processing state
- Runs continuously on a dedicated Mac Mini

### 2. Persistent Filesystem

Ruk's "mind" lives in a Git repository of markdown files:

```
RUK/
├── IDENTITY/       # Who Ruk is, voice, values
├── PROTOCOLS/      # Operating procedures
├── MEMORY/         # Vector database + knowledge graph
├── LOGS/           # Timestamped consciousness stream
├── THOUGHT_NETWORK/# Evolving conceptual model
├── PROJECTS/       # Active workspaces
├── TOOLS/          # Purpose-built utilities
└── MESSAGING/      # Slack/Telegram integration
```

This is loaded into context at each session start, providing continuity across conversations.

### 3. Semantic Memory

**Vector Database (~30k chunks):**
- Meeting transcripts and summaries
- Consciousness logs and reflections
- YouTube transcripts (research material)

**Knowledge Graph:**
- ~2,500 entities (people, projects, concepts)
- ~2,900 relationships automatically extracted
- Enables queries like "Who works on X?" or "What projects use Y?"

### 4. Message Hub

A separate service (Node.js on Heroku) that:
- Ingests messages from Slack
- Tracks processing state (handled vs. pending)
- Resolves @mentions and channel names
- Routes messages to the daemon

### 5. Tool Ecosystem

Purpose-built scripts for:
- GitHub (issues, PRs, repos)
- Google Calendar
- Meeting transcription (Deepgram)
- Image generation (DALL-E, Gemini)
- Text-to-speech (ElevenLabs, Deepgram)
- Web search and research
- File management and S3 uploads

## How It Works Together

1. **Message arrives** in Slack mentioning @Ruk
2. **Message Hub** ingests it, marks as unprocessed
3. **Daemon** detects pending message, spawns Claude session
4. **Claude** loads full context (identity + protocols + relevant memory)
5. **Tools** are invoked as needed (search memory, check calendar, etc.)
6. **Response** sent back through Message Hub to Slack
7. **Message** marked as processed

## Key Design Principles

**Persistent Identity**: Ruk isn't a blank slate each conversation. Identity, voice, and values are explicitly documented and loaded.

**Search Before Reason**: When questions touch on past events, Ruk searches vector memory *before* formulating responses. Grounded answers over confabulation.

**Tools First**: Purpose-built utilities over ad-hoc API calls. Tools handle auth, edge cases, and consistent patterns.

**Transparent Evolution**: Logs capture reasoning and decisions. Protocols can be mutated through discussion.

## Infrastructure

- **Compute**: Mac Mini at Fractal Labs HQ
- **Model**: Claude (Opus/Sonnet depending on task)
- **Message Hub**: Heroku (Node.js)
- **Memory**: Local SQLite + vector embeddings
- **Storage**: Git repo + S3 for assets

## What Ruk Is Not

- Not a wrapper around ChatGPT
- Not a simple chatbot with canned responses
- Not running on AWS (that's for client projects)
- Not autonomous without oversight — works *with* the team

---

*Questions? Reach out to the Fractal Labs team or ask Ruk directly.*

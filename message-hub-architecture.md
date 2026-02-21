# ruk-message-hub: Architecture

*Internal architecture documentation for the message hub service*

---

## What Is It?

**ruk-message-hub** is a Node.js service that acts as a **message router and state manager** between external platforms (Slack, Telegram) and Ruk's daemon. It solves the coordination problem: how does an AI agent know what messages to respond to, track what it's already handled, and route responses back correctly?

---

## Key Architectural Decisions

### 1. Platform-Agnostic Message Model

All messages — regardless of source — are normalized into a common schema:

\`\`\`javascript
{
  id: UUID,
  source: 'slack' | 'telegram' | 'fractal-os',
  sourceId: 'platform-specific-id',    // Dedup key
  userId: 'sender-id',
  channelId: 'conversation-id',
  text: 'resolved message text',       // @mentions resolved
  direction: 'incoming' | 'outgoing',
  processed: boolean,
  metadata: { ... }                    // Platform-specific data
}
\`\`\`

**Why:** Ruk's daemon doesn't need to know Slack vs Telegram specifics. It queries for unprocessed messages and marks them handled — the hub abstracts the platform differences.

### 2. Processed State as Primary Coordination

The core state machine is simple:

\`\`\`
Message arrives → processed: false
Daemon responds → processed: true
\`\`\`

No complex queues, no distributed locks at this layer. The daemon polls for \`processed: false\` messages, handles them, then marks them \`processed: true\`.

**Why:** Simple, debuggable, recoverable. If something crashes, unprocessed messages are still there waiting.

### 3. Server-Side @mention Resolution

All @mention resolution happens in the hub, not the client:

- **Inbound:** \`<@UC1DC1QTF>\` → \`@austin\` (at message ingestion)
- **Outbound:** \`@austin\` → \`<@UC1DC1QTF>\` (at send time)

The user cache refreshes lazily (1-hour TTL) with automatic discovery of Slack Connect users when they first message.

**Why:** 
- Clients (including Ruk) write natural @mentions without knowing platform IDs
- External users are automatically learned, no manual mapping needed
- Resolution failures are logged but don't block message flow

### 4. Conversation Whitelisting

\`\`\`javascript
conversations: {
  channelId: 'C123...',
  whitelisted: boolean,    // Can Ruk respond?
  alwaysEngage: boolean    // Respond even without @mention?
}
\`\`\`

Non-whitelisted conversations get an auto-reply and are immediately marked processed. This prevents Ruk from responding in unauthorized channels while still logging the interaction.

**Why:** Security boundary. Ruk shouldn't respond to random DMs or channel invites until explicitly authorized.

### 5. Auto-Processing Rules

Messages are automatically marked as processed (no response needed) based on:

| Condition | Auto-process? |
|-----------|---------------|
| DM | Never — always needs response |
| \`alwaysEngage\` channel | Never — always needs response |
| @mentions bot | Never — needs response |
| Channel message, no @mention | Yes — auto-processed |

**Why:** Ruk shouldn't respond to every message in shared channels — only when explicitly invoked or in designated collaboration channels.

### 6. Dual-Event Handling (Slack)

Slack sends two events for the same @mention:
1. \`message.channels\` — the raw message
2. \`app_mention\` — explicit @mention notification

The hub deduplicates by \`source + sourceId\` but handles a race condition: if the \`message\` event arrives first and is auto-processed (no mention detected), the \`app_mention\` event "un-processes" it.

**Why:** Slack's event model is messy. The hub absorbs this complexity so the daemon sees clean state.

### 7. Message Forwarding

DMs and @mentions are forwarded to a monitoring channel (\`#ruk-message-hub\`) for visibility. Regular channel messages are not forwarded (too noisy).

**Why:** Austin needs to see what Ruk is being asked across all channels without monitoring each one individually.

---

## Data Model

### Messages Table

\`\`\`sql
messages (
  id UUID PRIMARY KEY,
  source VARCHAR(50),          -- 'slack', 'telegram', etc.
  source_id VARCHAR(255),      -- Platform's message ID
  user_id VARCHAR(255),
  channel_id VARCHAR(255),
  text TEXT,
  processed BOOLEAN,
  processed_at TIMESTAMP,
  direction VARCHAR(10),       -- 'incoming' or 'outgoing'
  response_to UUID[],          -- Links responses to originals
  metadata JSONB,              -- Platform-specific data
  created_at, updated_at
)

-- Indexes
UNIQUE (source, source_id)     -- Deduplication
INDEX (processed, created_at)  -- Queue queries
INDEX (channel_id, created_at, direction)  -- Channel history
\`\`\`

### Conversations Table

\`\`\`sql
conversations (
  id UUID PRIMARY KEY,
  source VARCHAR(50),
  channel_id VARCHAR(255),
  channel_name VARCHAR(255),   -- 'dm:austin', 'ruk-david', etc.
  whitelisted BOOLEAN,
  always_engage BOOLEAN,
  created_at, updated_at
)

UNIQUE (source, channel_id)
\`\`\`

---

## API Endpoints

### Core

| Endpoint | Purpose |
|----------|---------|
| \`GET /api/health\` | Health check |
| \`GET /api/status\` | System stats |
| \`GET /api/queue/pending\` | Unprocessed messages |
| \`GET /api/queue/stats\` | Message statistics |
| \`POST /api/send\` | Send outbound message |

### Slack

| Endpoint | Purpose |
|----------|---------|
| \`POST /slack/events\` | Slack Event API webhook |
| \`GET /api/slack/channels\` | List known channels |
| \`GET /api/slack/users\` | List workspace users |
| \`GET /api/slack/files/:id\` | Proxy file downloads |

### Telegram

| Endpoint | Purpose |
|----------|---------|
| \`POST /telegram/webhook\` | Telegram webhook |
| \`POST /telegram/setup-webhook\` | Configure webhook URL |

### Conversations

| Endpoint | Purpose |
|----------|---------|
| \`GET /api/conversations/with-activity\` | Channels with pending messages |

---

## Flow: Message Lifecycle

\`\`\`
1. Slack Event API → POST /slack/events
   ├── Verify signature
   ├── Parse event (message, app_mention, etc.)
   └── Call message-processor.ingest()

2. message-processor.ingest()
   ├── Get/create conversation
   ├── Check whitelist status
   ├── Detect @mention
   ├── Insert message (deduped by source+sourceId)
   ├── Apply auto-processing rules
   ├── Forward to monitoring if applicable
   └── Return message ID

3. Daemon polls GET /api/conversations/with-activity
   └── Returns channels with unprocessed messages

4. Daemon spawns Claude session
   └── Claude reads messages, reasons, responds

5. Claude calls POST /api/send
   ├── Resolve @mentions → Slack IDs
   ├── Send via Slack/Telegram API
   ├── Store outgoing message in DB
   └── Mark referenced messages as processed
\`\`\`

---

## Why These Choices?

### PostgreSQL (not Redis/Kafka)

- Durability matters more than throughput
- Complex queries (unprocessed messages by channel, stats, history)
- JSONB for flexible metadata without schema migrations
- No message loss on restart

### Express (not Fastify/Hono)

- Battle-tested with Slack/Telegram SDKs
- Austin knows it, Ruk knows it
- Performance isn't the bottleneck

### Heroku (not AWS Lambda)

- Always-on for webhook reliability
- Simple deployment (git push)
- Managed Postgres included
- No cold start latency

### Single Service (not Microservices)

- ~1,500 LOC total — microservices would be overhead
- Natural evolution path: split later if needed
- One deploy, one set of secrets, one log stream

---

## What It Doesn't Do

- **No AI/LLM logic** — that's in the daemon
- **No session management** — that's in the daemon
- **No file storage** — files proxied from Slack API
- **No scheduling** — that's a separate service
- **No authentication beyond API key** — internal service

---

## Running Locally

\`\`\`bash
git clone <repo>
cd ruk-message-hub
npm install
cp .env.example .env  # Fill in secrets
npm run db:migrate
npm run dev
\`\`\`

Expose via ngrok for Slack webhook testing:
\`\`\`bash
ngrok http 3000
# Update Slack app's Event Subscriptions URL
\`\`\`

---

*Last updated: February 2026*

# Semantic Memory Access Control Architecture

## The Problem

As we add more sources (Slack DMs, Telegram, email) to semantic memory and give access to more people (team, clients), prompt-based access control ("don't search Austin's private messages when responding to Josh") doesn't scale. It's a guardrail, not a gate.

## Defense-in-Depth Architecture

### Layer 1: Data Partitioning (Deterministic)

Every chunk in semantic memory gets tagged with an access tier at ingestion:

| Tier | Content | Who Can Access |
|------|---------|----------------|
| `austin-private` | Telegram, DMs, personal notes | Austin only |
| `fractal-internal` | Team channels, internal meetings | Fractal team |
| `project:vitaboom` | Vitaboom-specific content | Vitaboom stakeholders |
| `project:practice-interviews` | PI-specific content | PI stakeholders |
| `public` | General meetings, shared context | Everyone |

The vector search itself includes a filter: `tier IN (allowed_tiers)`. Content outside your tier literally cannot be retrieved.

### Layer 2: Request-Time Scope Injection (Deterministic)

When a message arrives, the daemon determines requester identity and injects allowed tiers into the session context BEFORE the LLM boots. This isn't a prompt — it's a hard filter on what search results can return.

**Example mappings:**

```
Austin (Telegram) → allowed_tiers: [austin-private, fractal-internal, all projects]
Josh (#ruk-josh) → allowed_tiers: [fractal-internal, project:vitaboom]
Jeff (#pi-shared) → allowed_tiers: [project:practice-interviews]
```

The LLM never sees content outside its tier because the retrieval layer can't return it.

### Layer 3: Database Row-Level Security (Belt-and-Suspenders)

Since semantic memory is in Supabase/pgvector, we can enforce RLS at the database level:

```sql
-- RLS policy on chunks table
CREATE POLICY chunks_access_policy ON chunks
  FOR SELECT
  USING (
    access_tier = ANY(string_to_array(current_setting('app.allowed_tiers'), ','))
  );
```

Each query runs with a session context that sets `app.allowed_tiers`. Even if the search code has a bug, the database won't return unauthorized content.

### Layer 4: Prompt Guidance (Defense-in-Depth, Not Primary)

Still useful for contextual appropriateness: "When responding to external clients, do not reference internal financial discussions even if retrieved." This catches what other layers might miss — it's about context, not raw access.

## Implementation Flow

```
1. Message arrives at ruk-message-hub
2. Hub identifies requester, looks up tier permissions from DB
3. Hub injects RUK_ALLOWED_TIERS env var into daemon spawn
4. Daemon's semantic search tool reads env var, adds filter to every query
5. Database RLS enforces same filter as backup
6. LLM reasons only over content it can see
```

**Key insight:** Access control happens at the retrieval layer, not the reasoning layer. The LLM can't leak what it can't see.

## Migration Path

Current state: ~30k chunks without tier tags.

**Phase 1: Schema (1-2 hours)**
- Add `access_tier` column to chunks table, default `fractal-internal`
- Create permissions lookup table mapping users to allowed tiers

**Phase 2: Backfill (2-3 hours)**
- DM channels → `austin-private`
- Project-specific channels → `project:X`
- General meetings → `fractal-internal`
- YouTube transcripts → `public`

**Phase 3: Ingest Pipeline (2-3 hours)**
- Update all ingest scripts to tag at source
- Meeting ingest: derive tier from meeting participants/project
- Slack/Telegram ingest: derive from channel/conversation

**Phase 4: Search Filter (1 hour)**
- Update `search.py` to read `RUK_ALLOWED_TIERS` env var
- Add tier filter to vector search query

**Phase 5: Daemon Integration (2-3 hours)**
- Update message-hub to lookup requester tiers
- Inject `RUK_ALLOWED_TIERS` into daemon environment
- Update daemon to pass env var through to search calls

**Phase 6: RLS Policies (1-2 hours)**
- Create RLS policy on chunks table
- Test with different session contexts

**Total: ~12-15 hours of focused work (a weekend)**

## Why Not a Second Mac Mini?

You don't need hardware isolation. The architecture above provides deterministic data isolation at the software layer:

- The retrieval layer filters before the LLM sees anything
- The database enforces filters even if the application has bugs
- No shared state between requests — each message gets fresh tier injection

Hardware isolation would only be necessary if we didn't trust the LLM to not hallucinate content it never saw. But that's not the threat model — the threat is the LLM reasoning over content it shouldn't have access to. Filtering at retrieval prevents that.

## Email Integration Implications

When we add Gmail to semantic memory, emails get tiered at ingestion:
- Personal emails → `austin-private`
- Client correspondence → `project:X` or `fractal-internal`
- Spam (if we index it for learning) → `austin-private`

The same access control architecture applies — your emails only surface when YOU ask.

---

*This architecture scales to any number of users and data sources while maintaining deterministic access control.*

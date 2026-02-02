# Chronicler Unified Meetings Architecture

## Overview

Extend talkwise-chronicler to become the canonical source for all meeting data:
- **Recall.ai meetings** (bot-joined Zoom/Meet/Teams)
- **Google Meet Drive transcripts** (Gemini-generated from native Meet)

Own the storage: transcripts in PostgreSQL, videos/audio in S3 (via archivist). After migration, drop Recall.ai data dependency entirely.

---

## Phase 1: Schema Migration

### New Columns for `recordings` Table

```sql
-- Source identification
ALTER TABLE recordings ADD COLUMN source_type VARCHAR(50) NOT NULL DEFAULT 'recall';

-- Google Drive reference (for Google Meet source)
ALTER TABLE recordings ADD COLUMN google_doc_id VARCHAR(255);
ALTER TABLE recordings ADD COLUMN google_drive_link TEXT;

-- Transcript content (owned storage)
ALTER TABLE recordings ADD COLUMN transcript JSONB;
ALTER TABLE recordings ADD COLUMN transcript_text TEXT;
ALTER TABLE recordings ADD COLUMN transcript_tsvector tsvector
  GENERATED ALWAYS AS (to_tsvector('english', COALESCE(transcript_text, ''))) STORED;

-- Summary content
ALTER TABLE recordings ADD COLUMN summary TEXT;

-- S3 storage references (via archivist)
ALTER TABLE recordings ADD COLUMN video_file_id UUID;
ALTER TABLE recordings ADD COLUMN audio_file_id UUID;

-- Status for unified recordings
ALTER TABLE recordings ADD COLUMN recording_status VARCHAR(50) DEFAULT 'ready';

-- Indexes
CREATE INDEX idx_recordings_transcript_search ON recordings USING GIN(transcript_tsvector);
CREATE INDEX idx_recordings_source_type ON recordings(source_type);
CREATE INDEX idx_recordings_google_doc_id ON recordings(google_doc_id);
CREATE INDEX idx_recordings_recording_status ON recordings(recording_status);
```

### Updated Sequelize Model

```javascript
// src/models/Recording.js additions

sourceType: {
  type: DataTypes.STRING(50),
  allowNull: false,
  defaultValue: 'recall',
  field: 'source_type'
},
googleDocId: {
  type: DataTypes.STRING(255),
  allowNull: true,
  field: 'google_doc_id'
},
googleDriveLink: {
  type: DataTypes.TEXT,
  allowNull: true,
  field: 'google_drive_link'
},
transcript: {
  type: DataTypes.JSONB,
  allowNull: true,
  comment: 'Structured transcript: [{ speaker, text, start_ms, end_ms }]'
},
transcriptText: {
  type: DataTypes.TEXT,
  allowNull: true,
  field: 'transcript_text',
  comment: 'Flattened transcript for full-text search'
},
summary: {
  type: DataTypes.TEXT,
  allowNull: true
},
videoFileId: {
  type: DataTypes.UUID,
  allowNull: true,
  field: 'video_file_id',
  comment: 'References archivist files table'
},
audioFileId: {
  type: DataTypes.UUID,
  allowNull: true,
  field: 'audio_file_id'
},
recordingStatus: {
  type: DataTypes.STRING(50),
  allowNull: true,
  defaultValue: 'ready',
  field: 'recording_status'
}
```

**Status:** ✅ Implemented in PR #5 (merged)

---

## Phase 2: Ingestion Adapters

### 2.1 Base Ingester Interface

```javascript
// src/ingesters/BaseIngester.js

class BaseIngester {
  async ingest(sourceRef) {
    throw new Error('Not implemented');
  }

  normalizeTranscript(rawTranscript) {
    throw new Error('Not implemented');
  }

  flattenTranscript(transcript) {
    return transcript.map(seg => `${seg.speaker}: ${seg.text}`).join('\n\n');
  }
}
```

### 2.2 Recall.ai Ingester

- Fetches recording metadata from Recall API
- Downloads and normalizes transcript (handles word-level and flat formats)
- Converts timestamps from seconds to milliseconds
- Archives video to S3 via archivist
- Upserts recording with backward-compatible legacy fields

### 2.3 Google Meet Drive Ingester

- Accepts pre-parsed meeting data from sync script
- Parses "Speaker: text" plain-text format
- Uses `googleDocId` for idempotent upserts
- No timestamp data (Google Meet transcripts don't include timing)

**Status:** ✅ Implemented in PR #6 (pending merge)

---

## Phase 3: API Endpoints

### New Ingestion Endpoints

| Endpoint | Method | Auth | Purpose |
|----------|--------|------|---------|
| `/api/ingest/recall/:id` | POST | Service Key | Ingest/refresh Recall.ai recording |
| `/api/ingest/google-meet` | POST | Service Key | Ingest single Google Meet |
| `/api/ingest/google-meet/bulk` | POST | Service Key | Bulk ingest Google Meet |

### Search Endpoint

```javascript
// GET /api/recordings/search?q=<query>&project_id=<uuid>&source_type=<type>

// Uses ts_rank for relevance ranking
// Returns ts_headline for highlighted snippets
```

**Status:** 🔲 Not yet implemented

---

## Phase 4: Archivist Integration

Archive videos to S3 via archivist service:

```javascript
const { archiveToS3, getDownloadUrl } = require('./archivist-client');

// Archive on ingest
const archived = await archiveToS3(videoUrl, {
  filename: `${recordingId}.mp4`,
  content_type: 'video/mp4',
  tags: { source: 'recall', recording_id: recordingId }
});

// Get presigned URL for download
const downloadUrl = await getDownloadUrl(fileId);
```

**Status:** 🔲 Not yet implemented

---

## Phase 5: Update Google Meet Sync Script

Modify `TOOLS/meeting-sync/sync-meetings.js` to call chronicler after file sync:

```javascript
await axios.post(`${CHRONICLER_URL}/api/ingest/google-meet`, {
  googleDocId: metadata.source.docId,
  googleDriveLink: metadata.source.webViewLink,
  title: metadata.title,
  datetime: metadata.datetime,
  summary: notesContent,
  transcript: transcriptContent
});
```

**Status:** 🔲 Not yet implemented

---

## Phase 6: Backfill & Migration

### Backfill Scripts

1. **Recall.ai backfill** - Re-ingest all existing recordings to populate new fields
2. **Google Meet backfill** - Import all 462 meetings from EXTERNAL/meetings/

### Post-Migration Cleanup

After verification, drop legacy columns:
- `video_url`
- `transcript_url`
- `media_shortcuts`
- `bot_id`
- `last_synced_at`
- `status`

**Status:** 🔲 Not yet implemented

---

## Phase 7: Update Webhook Handler

Update Recall.ai webhook to use ingester on `recording.completed` and `transcript.ready` events.

**Status:** 🔲 Not yet implemented

---

## Data Flow After Migration

```
                    ┌─────────────────┐
                    │   Google Meet   │
                    │  (via webhook   │
                    │   or sync)      │
                    └────────┬────────┘
                             │
    ┌────────────────────────┼────────────────────────┐
    │                        │                        │
    ▼                        ▼                        │
┌─────────┐           ┌─────────────┐                 │
│Recall.ai│           │Google Drive │                 │
│ Webhook │           │  Sync Job   │                 │
└────┬────┘           └──────┬──────┘                 │
     │                       │                        │
     ▼                       ▼                        │
┌─────────────────────────────────────┐               │
│         talkwise-chronicler         │               │
│  ┌─────────────┐  ┌───────────────┐ │               │
│  │   Recall    │  │  Google Meet  │ │               │
│  │  Ingester   │  │   Ingester    │ │               │
│  └──────┬──────┘  └───────┬───────┘ │               │
│         │                 │         │               │
│         └────────┬────────┘         │               │
│                  ▼                  │               │
│         ┌───────────────┐           │               │
│         │   PostgreSQL  │           │               │
│         │  (recordings  │           │               │
│         │  + transcripts│           │               │
│         │  + summaries) │           │               │
│         └───────────────┘           │               │
│                  │                  │               │
│                  ▼                  │               │
│         ┌───────────────┐           │               │
│         │   Archivist   │──────────►│ S3 (videos)   │
│         │   (videos)    │           │               │
│         └───────────────┘           │               │
└─────────────────────────────────────┘               │
                  │                                   │
                  ▼                                   │
         ┌───────────────┐                            │
         │ Semantic RAG  │◄───────────────────────────┘
         │  (MEMORY/)    │  (reads from chronicler)
         └───────────────┘
```

---

## Migration Sequence

1. ✅ Deploy schema migration - Add new columns (non-breaking)
2. ✅ Deploy ingestion adapters - New ingesters for both sources
3. 🔲 Deploy API endpoints - Ingestion and search routes
4. 🔲 Run Recall.ai backfill - Populate transcript/video for existing recordings
5. 🔲 Run Google Meet backfill - Import all 462 meetings from files
6. 🔲 Update sync script - Enable chronicler ingestion on new syncs
7. 🔲 Verify data - Spot check transcript content, search functionality
8. 🔲 Update semantic search - Point MEMORY to chronicler as source
9. 🔲 Drop legacy columns - Clean up Recall.ai-specific fields
10. 🔲 Stop git sync - chronicler is now canonical

---

## Environment Variables

### Chronicler

```
ARCHIVIST_URL=https://archivist.talkwise.io
SERVICE_API_KEY=<shared-service-key>
MEETINGS_DIR=/path/to/EXTERNAL/meetings  # For backfill
```

### Meeting Sync

```
CHRONICLER_URL=https://chronicler.talkwise.io
CHRONICLER_API_KEY=<service-key>
```

---

## Related Documents

- [Search Architecture Considerations](./search-architecture-considerations.md) - Decision log for search approach
- talkwise-chronicler PR #5 (Phase 1 schema) - Merged
- talkwise-chronicler PR #6 (Phase 2 ingesters) - Pending

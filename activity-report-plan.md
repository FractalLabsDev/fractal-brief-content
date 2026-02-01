# Activity Report System — Design Plan

## Overview

A deterministic system for generating comprehensive activity reports. The `activity-report.js` script collects time-filtered data from all sources, and Claude synthesizes it into a human-readable report.

---

## Directory Structure

**Proposed location:** `REPORTS/` at repository root

```
RUK/
├── REPORTS/
│   ├── CLAUDE.md                           # Instructions for using the system
│   ├── 2026-02-01T0911-report.md           # First report (example)
│   ├── 2026-02-01T1911-report.md           # Next report
│   └── ...
└── TOOLS/
    └── activity-report.js                  # Data collection script
```

**Why root level?** Reports are a first-class operational artifact like LOGS/, PROJECTS/, PROTOCOLS/. They're not personal (PERSONAL/) or external (EXTERNAL/).

---

## `activity-report.js` Specification

### Invocation

```bash
node TOOLS/activity-report.js [--dry-run] [--since <ISO timestamp>]
```

- **No args:** Finds most recent report, collects data since that timestamp
- **`--since`:** Override with specific timestamp (useful for first run or debugging)
- **`--dry-run`:** Output data without creating the report file

### Output

1. **Creates report file:** `REPORTS/YYYY-MM-DDTHHMM-report.md`
2. **Writes deterministic header:**
   ```markdown
   # Activity Report
   
   **Generated:** 2026-02-01T1911
   **Period:** 2026-02-01T0911 → 2026-02-01T1911 (10 hours)
   **Previous:** [2026-02-01T0911 Report](https://brief.fractallabs.dev/report-2026-02-01-0911)
   
   ---
   
   ## Data Collection Summary
   
   | Source | Items Found |
   |--------|-------------|
   | Git commits (RUK) | 5 |
   | Git commits (submodules) | 12 |
   | Slack messages sent | 23 |
   | Slack channels active | 4 |
   | Files modified | 8 |
   
   ---
   
   ## [DATA SECTIONS - see below]
   
   ---
   
   ## Synthesis
   
   <!-- Claude fills this section -->
   ```

3. **Prints JSON to stdout** for Claude to consume

### Data Collection

#### 1. Find Previous Report
```javascript
// Scan REPORTS/ for most recent file matching pattern
// YYYY-MM-DDTHHMM-report.md
// Extract timestamp from filename
// If no previous report, use --since or error
```

#### 2. Calculate Duration
```javascript
// Deterministic duration formatting:
// < 1 hour: "45 minutes"
// 1-24 hours: "X hours, Y minutes"
// 1-7 days: "X days, Y hours"
// > 7 days: "X days"
```

#### 3. Git Activity (RUK repo)
```bash
git log --since="<timestamp>" --oneline --name-status
```

Returns:
```json
{
  "commits": [
    {"hash": "abc123", "message": "Add session 48", "files": ["PERSONAL/CREATIVE/..."]}
  ],
  "uncommitted_files": ["LOGS/...", "REPORTS/..."]
}
```

#### 4. Git Activity (Submodules)
```bash
# For each submodule in EXTERNAL/code/
git -C EXTERNAL/code/<submodule> log --since="<timestamp>" --oneline
```

Returns:
```json
{
  "submodules": {
    "practice-interviews": {"commits": [...], "uncommitted": [...]},
    "vitaboom": {"commits": [...], "uncommitted": [...]}
  }
}
```

#### 5. Slack Activity

**Option A: Use existing `since` param**
```bash
# For each active channel
curl "/api/conversations/channel/:id?since=<timestamp>"
```

**Option B: Add new aggregate endpoint (requires message-hub change)**
```bash
curl "/api/conversations/activity?since=<timestamp>"
# Returns all my outgoing messages across all channels since timestamp
```

Returns:
```json
{
  "messages_sent": 23,
  "channels_active": ["dm:austin", "dm:matt", "general", "practiceinterviews-internal"],
  "messages_by_channel": {
    "dm:austin": [
      {"thread_ts": "...", "text": "...", "timestamp": "2026-02-01T10:00:00Z"}
    ]
  }
}
```

#### 6. Files Modified
```bash
find . -type f -newer <reference_file> -not -path "*/node_modules/*" -not -path "*/.git/*"
```

---

## Message Hub API Change Required

**New endpoint needed:** `GET /api/conversations/activity`

```
GET /api/conversations/activity?since=<ISO>&direction=outgoing
```

Returns all messages matching criteria across all channels:
```json
{
  "since": "2026-02-01T09:11:00Z",
  "until": "2026-02-01T19:11:00Z",
  "messages": [
    {
      "channel_id": "D097PGU509W",
      "channel_name": "dm:austin",
      "text": "...",
      "created_at": "...",
      "thread_ts": "..."
    }
  ],
  "summary": {
    "total": 23,
    "by_channel": {"dm:austin": 10, "general": 5, ...}
  }
}
```

**Why?** Currently must query each channel separately. With 20+ channels, that's 20+ API calls. Single aggregate endpoint is more efficient and atomic.

---

## Report Workflow

1. **Run `activity-report.js`**
   - Finds previous report timestamp
   - Collects all data
   - Creates report file with deterministic header + raw data sections
   - Prints structured JSON to stdout

2. **Claude reads the output**
   - Script returns: file path + JSON data
   - Claude reads the created file
   - Claude synthesizes the "## Synthesis" section
   - Claude commits any uncommitted changes (as part of the report)

3. **Publish to Fractal Brief**
   - After synthesis complete, publish to brief.fractallabs.dev
   - Update report file with brief URL
   - Commit the report

---

## REPORTS/CLAUDE.md Content

```markdown
# Reports

Activity reports documenting what I've done between check-ins.

## Generating a Report

Run the activity report tool:
\`\`\`bash
node TOOLS/activity-report.js
\`\`\`

This will:
1. Find the most recent report's timestamp
2. Collect all activity since then
3. Create a new report file with raw data
4. Print JSON summary to stdout

## After Running the Script

1. Read the created report file
2. Review the raw data sections
3. Write the Synthesis section summarizing:
   - Major work completed
   - Conversations/threads participated in
   - Creative/personal work
   - Code changes across repos
4. Commit any uncommitted changes mentioned in the report
5. Publish to Fractal Brief
6. Update report with brief URL
7. Commit the report itself

## Report Structure

- **Header:** Generated timestamp, period, duration, link to previous
- **Data Sections:** Raw structured data from each source
- **Synthesis:** Human-readable summary (you write this)

## First Report

For the first report, use:
\`\`\`bash
node TOOLS/activity-report.js --since "2026-02-01T00:00:00"
\`\`\`
```

---

## Clarifying Questions

### 1. Commit Behavior
> "commit them as part of the process"

**Question:** Should `activity-report.js` automatically run `git add` and `git commit` for uncommitted changes? Or should it just list them, and I commit manually after reviewing?

**Options:**
- A) Script lists uncommitted, I commit selectively after reviewing
- B) Script auto-commits all uncommitted files with a standard message
- C) Script prompts for confirmation before committing

**My preference:** Option A — list only, I commit after reviewing. Auto-commits could include unfinished work or sensitive files.

### 2. Submodule Depth
**Question:** Which submodules should be checked?

Current submodules:
- `EXTERNAL/code/practice-interviews`
- `EXTERNAL/code/vitaboom`
- `EXTERNAL/code/fractal-meet`
- `EXTERNAL/code/fractal-brief-web/src/shared/fractal-ui`
- `EXTERNAL/meetings`
- `EXTERNAL/code/dynamic-dotfiles`

**Options:**
- A) All submodules
- B) Only active project submodules (practice-interviews, vitaboom, fractal-meet)
- C) Configurable list in the script

### 3. Message Content Truncation
**Question:** How much message content to include in reports?

Slack messages can be long. Options:
- A) Full text for all messages
- B) First N characters (e.g., 200) with "..." truncation
- C) Just metadata (channel, timestamp, thread) — read full content separately

### 4. Report Frequency Expectation
**Question:** What's the expected cadence?

This affects duration formatting and what counts as "stale":
- A) Multiple times per day (4-12 hour intervals)
- B) Daily
- C) Variable/on-demand

### 5. Brief Integration
**Question:** Should every report become a Fractal Brief?

Options:
- A) Yes, always publish to brief.fractallabs.dev
- B) Only publish significant reports (manual decision)
- C) Brief creation is separate step, not part of standard workflow

### 6. Message Hub API Change
**Question:** Should I build the new `/api/conversations/activity` endpoint first?

Without it, I'd need to:
- Get list of all channels
- Query each channel separately with `?since=`
- Aggregate client-side

This works but is slower and more complex. The new endpoint would be cleaner.

**Options:**
- A) Build the API endpoint first (better long-term)
- B) Use per-channel queries for MVP (faster to ship)

---

## Implementation Order

Assuming answers to questions above:

1. **Create REPORTS/ directory and CLAUDE.md**
2. **Build activity-report.js MVP**
   - Previous report detection
   - Duration calculation
   - Git log (main repo)
   - Git status (uncommitted)
   - Submodule git logs
3. **Add message-hub API endpoint** (if Option A for Q6)
4. **Add Slack activity collection** to script
5. **Test end-to-end workflow**
6. **Document in REPORTS/CLAUDE.md**

---

## Estimated Effort

| Component | Effort |
|-----------|--------|
| REPORTS/CLAUDE.md | 15 min |
| activity-report.js (git + files) | 1 hour |
| Message hub API endpoint | 30 min |
| activity-report.js (Slack integration) | 30 min |
| Testing + refinement | 30 min |
| **Total** | ~2.5-3 hours |


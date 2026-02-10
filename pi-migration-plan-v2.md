# PI Deprecation Plan v2: Using Promptsmith

**Target:** Feb 18, 2026 (9 days)
**Corrections:**
1. talkwise-promptsmith already exists — use it instead of building new LLM service
2. Priority reorder: Prompt migration first, THEN data/infrastructure migration

---

## What PI Uses from TW-Backend (Deep Dive)

### Current Data Flow
```
Web App → pi-backend → talkwise-backend → Wordware
         ↓ (also)
         talkwise-backend (WebSocket for live transcription)
```

### TW-Backend Functionality Analysis

| Feature | Route/Service | PI Uses? | New Home |
|---------|---------------|----------|----------|
| **AI Workflows** | `/api/practiceInterviews/text` | ✅ | promptsmith |
| **Media Processing** | `/api/practiceInterviews/media` | ✅ | TTS/STT microservice |
| **Transcription** | `/api/practiceInterviews/transcribe-media` | ✅ | STT microservice |
| **Ideal Answer Gen** | `/api/practiceInterviews/genIdealAnswer` | ✅ | promptsmith |
| **Wordware Proxy** | `/api/wordware` | ✅ | promptsmith |
| **Users Table** | User model + auth routes | ✅ | pi-backend |
| **Live Transcription** | WebSocket + Deepgram | ✅ | STT microservice |
| **TTS (Deepgram/ElevenLabs)** | `textToSpeech.service.ts` | ✅ | TTS microservice |
| **Stripe/Payments** | `stripe.service.ts` | ✅ | pi-backend |
| **Email (SendGrid/Twilio)** | `email.service.ts`, `twilio.service.ts` | ✅ | pi-backend |

### Unused TW-Backend Features (Not PI-related)
- `docovia.route.ts` - Docovia integration
- `richTextDocument.route.ts` - Rich text storage
- `pinecone.service.ts` - Vector embeddings
- `midjourney.service.ts` - Image generation
- Various chat/message features

---

## Revised Phased Plan

### Phase 1: Prompt Migration to Promptsmith (Feb 10-12) - 3 days

**Day 1 (Feb 10):** Import PI prompts into promptsmith
- Extract current Wordware prompt content (already exported)
- Create prompts in promptsmith with proper slugs:
  - `pi-behavioral-situation-task`, `pi-behavioral-actions`, `pi-behavioral-results`, `pi-behavioral-summary`, `pi-behavioral-score`
  - `pi-hypothetical-clarify`, `pi-hypothetical-framework`, `pi-hypothetical-assumptions`, `pi-hypothetical-solution`, `pi-hypothetical-summary`, `pi-hypothetical-score`
  - `pi-generate-questions`, `pi-ideal-answer-behavioral`, `pi-ideal-answer-hypothetical`
- Test prompts via promptsmith API

**Day 2 (Feb 11):** Update pi-backend to call promptsmith
- Add promptsmith client to pi-backend
- Replace `TalkWiseService` calls with promptsmith execute calls
- Update `question.service.ts` (question generation)
- Deploy to stage, validate

**Day 3 (Feb 12):** Migrate feedback workflows
- Update `answer.route.ts` to use promptsmith for behavioral/hypothetical feedback
- Handle SSE streaming from promptsmith → web app
- Deploy to stage, validate

### Phase 2: TTS/STT Microservices (Feb 13-14) - 2 days

**Day 4 (Feb 13):** STT Microservice
- Create `talkwise-transcriber` microservice (or extend existing)
- Move Deepgram/Whisper transcription logic from tw-backend
- Endpoints: `POST /transcribe` (file), `WS /live-transcribe` (streaming)
- Deploy to stage

**Day 5 (Feb 14):** TTS Microservice + Integration
- Create `talkwise-speaker` microservice (or extend transcriber)
- Move Deepgram Aura / ElevenLabs TTS logic
- Endpoints: `POST /speak`
- Update pi-backend to call new microservices
- Update web app socket URL if needed

### Phase 3: Users Table Migration (Feb 15-16) - 2 days

**Day 6 (Feb 15):** Schema alignment
- Compare tw-backend User model vs pi-backend User model
- Create migration to add missing columns to pi-backend users table
- Map fields: stripeId, trialExpiresAt, metadata, roleId, etc.

**Day 7 (Feb 16):** Data migration + Auth routes
- Script to migrate user records from tw-backend → pi-backend
- Move auth routes (`/api/signIn`, `/api/registerAccount`, `/api/verifyEmail`, etc.) to pi-backend
- Update JWT token issuance

### Phase 4: Payments & Email (Feb 17) - 1 day

**Day 8 (Feb 17):**
- Move Stripe integration to pi-backend (subscription management, webhook handlers)
- Move SendGrid/Twilio email service to pi-backend
- Remove tw-backend proxy from pi-backend

### Phase 5: Cutover & Cleanup (Feb 18) - 1 day

**Day 9 (Feb 18):**
- Remove `SOCKET_BASE_URLS` pointing to tw-backend from web app
- Update environment variables across all environments
- Full E2E testing in stage
- Production deploy
- Monitor for 2-4 hours post-deploy
- Deprecate tw-backend Heroku dyno

---

## Migration Artifacts Summary

| Component | Current Location | New Location |
|-----------|------------------|--------------|
| AI Prompts | Wordware UI | talkwise-promptsmith |
| Workflow Orchestration | tw-backend WorkflowService | promptsmith workflows |
| STT (Deepgram) | tw-backend deepgram.service | talkwise-transcriber |
| TTS (Deepgram/EL) | tw-backend textToSpeech.service | talkwise-speaker |
| Users Table | tw-backend DB | pi-backend DB |
| Auth Routes | tw-backend auth.route | pi-backend |
| Stripe | tw-backend stripe.service | pi-backend |
| Email | tw-backend email.service | pi-backend |

---

## Risks & Mitigations

| Risk | Mitigation |
|------|------------|
| Promptsmith not supporting SSE streaming | Verify before Phase 1; fallback to direct LLM calls if needed |
| User migration data loss | Run in dry-run mode first, backup tw-backend DB before migration |
| Socket.io live transcription breaking | Test extensively in stage, keep tw-backend dyno on standby for quick rollback |
| Payments disruption | Coordinate Stripe webhook updates carefully, test with Stripe test mode |

---

## Key Files to Modify

### pi-backend
- `src/service/external/talkwise.service.ts` → Replace with promptsmith client
- `src/service/question.service.ts` → Update question generation calls
- `src/routes/answer.route.ts` → Update feedback workflow calls
- `src/routes/proxy/talkwise.proxy.ts` → Remove after migration

### practice-interviews-web
- `src/data/constants.ts` → Remove `SOCKET_BASE_URLS`
- `src/middleware/useWebSocket.ts` → Update socket URL to new STT microservice
- `src/utils/env.ts` → Simplify environment detection

### New Microservices
- `talkwise-transcriber/` → STT microservice
- `talkwise-speaker/` → TTS microservice

---

*Generated: 2026-02-09T19:55*

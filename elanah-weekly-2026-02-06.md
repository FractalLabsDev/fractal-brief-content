# Meeting Recap: Elanah and Innovative Solutions — Feb 6, 2026

**Meeting:** [Innovative/Elanah.AI] EXT: Weekly Project Status  
**Date:** Thursday, February 6, 2026  
**Attendees:** Austin Wood (Fractal Labs), JT Fitzgerald (Innovative Solutions)  
**Duration:** ~28 minutes

---

## Key Takeaways

1. **Handoff is approaching final stages.** Next Friday's call is expected to be the last weekly meeting. Innovative Solutions will deliver all code, documentation, and access by then.

2. **Infrastructure is already under Josh's AWS account** — no transfer needed. GitHub repos will be delivered via `git archive` with full commit history.

3. **Two known tech debt items** need attention post-handoff: Postgres not in VPC, and Auth0 using a dev Google key.

4. **No test coverage or LLM eval framework exists yet** — both are priorities for Fractal Labs to stand up early.

5. **Voice chat is recommended to be deprioritized** for 3-6 months given the pace of industry change.

---

## Handoff & Infrastructure

| Item | Status |
|------|--------|
| AWS access | Under Josh's account — he can provision Austin directly |
| GitHub code | Two repos (app + infrastructure-as-code). Will be delivered via `git archive` with full commit history |
| Infrastructure as Code | AWS infrastructure defined in TypeScript CDK, in a separate repo. Included in deliverables |
| CI/CD pipelines | Currently connected to Innovative's GitHub. JT has a screenshot guide for reconnecting to a new repo — will send that over |
| READMEs | Already written. Close-out call will walk through them; post-handoff questions via Slack are welcome |

**Next step for Austin:** Get AWS access from Josh before next Friday's call to poke around and come with targeted questions.

---

## Bug Fixes & Recent Issues

- **Mobile Safari race condition** — JT resolved this. Josh had discovered it when testing on a new device/browser and initially panicked, but it was a straightforward fix.

- **Cross-chat memory "leak"** — Josh was using different chat sessions to test different user personas (e.g., pretending to go through a divorce in one, then sleep issues in another). The system's memory now pulls from all chats per user, so context bled across test sessions. This actually validated that long-term memory is working correctly. Solution going forward: use separate email accounts for distinct test personas.

- **Admin dashboard questions** — Josh had some questions before the call; JT resolved them directly.

---

## Architecture & Technical State

- **Tightly coupled Next.js app** (started from a Vercel chatbot template). No refactoring into separate front-end/back-end has been done.
  - Austin plans to split the APIs out as a first priority — potential clients are requesting native mobile apps.

- **Vercel AI SDK v5** is used under the hood (current is v6). Provides unified multi-provider/multi-model support.

- **RAG: Vector database only** (no knowledge graph). Chunks are stored and immediately available after upload via the admin dashboard. To update existing content: delete and re-upload. New uploads are auto-vectorized.
  - Austin shared that Fractal Labs recently implemented hybrid RAG (vector + knowledge graph) on another project with significantly better results, especially for the ~20% of queries that don't suit pure semantic search.

- **CBT materials, military jargon, and other domain content** are managed through the admin dashboard — no engineer needed for content updates.

---

## Known Tech Debt

1. **Postgres not in VPC** — Still publicly accessible (authenticated, but not network-isolated). Originally couldn't communicate from Amplify to VPC. Shouldn't be a huge lift to move.

2. **Auth0 dev key for Google login** — Shows a dev warning icon. Needs to be updated with a production Google provider key through Auth0. No security impact, just cosmetic. All under Elanah's Auth0 account.

---

## Testing & Quality

- **No automated test coverage.** Expected given the zero-to-MVP phase. Austin plans to prioritize this.

- **No LLM eval framework.** Quality feedback has come from Josh and Sherry's manual testing. Josh has sent example conversations (good and bad) — JT will coordinate with his colleague Matt to compile those and send them over. Austin plans to use these as the foundation for an automated eval framework (test user agent + evaluator LLM pattern).

---

## Crisis Escalation

- Currently handled via **system prompt only** — no hard guardrails (e.g., Bedrock Guardrails) in place.
- The system provides Veterans Crisis Line, 911, and other resources when self-harm or suicidal ideation is detected.
- JT reports it's been robust in testing — aggressive attempts to bypass it defaulted back to crisis resources.
- **Recommendation:** Implement Bedrock Guardrails as a safety net layer going forward.

---

## Voice Chat

- Currently uses **Hume EVI** (Empathic Voice Interface) via a React wrapper.
- Running on a **free dev key** — another tech debt item.
- Transcripts are extracted from voice sessions and injected into the conversation history for continuity.
- Quality is inconsistent: "sometimes it works great and other times it's totally inhuman."
- Austin and JT agreed to **deprioritize voice for 3-6 months** — the industry is moving too fast to over-optimize on current technology. Both teams have seen similar challenges (latency vs. interruption tradeoffs, Nvidia PersonaPlex duplex limitations, etc.).

---

## Future Improvement Ideas

1. **Role-based default views** — Admins currently land on the chat interface like everyone else. Should default to admin dashboard with chat accessible from there. Would improve onboarding UX for organizational admins.

2. **Front-end / back-end split** — Enables native mobile apps and better performance. Priority #1 for Fractal Labs post-handoff.

3. **Bedrock Guardrails** — Hard safety layer for crisis detection beyond system prompts.

4. **Hybrid RAG** — Add knowledge graph alongside vector DB for more robust retrieval.

5. **Automated eval framework** — Using Josh's example conversations as ground truth.

---

## Action Items

| Owner | Action | Timeline |
|-------|--------|----------|
| **Austin** | Get AWS access from Josh | Before next Friday |
| **JT / Matt (Innovative)** | Compile Josh's example conversations and feedback | Send to Austin next week |
| **JT** | Send CI/CD reconnection guide (with screenshots) | Before handoff |
| **JT / Innovative** | Final deliverables: git archives, READMEs, infrastructure-as-code repo | By next Friday |
| **Austin + JT** | Close-out call: walk through READMEs and remaining questions | Next Friday (Feb 13) |

---

*Summary prepared by Ruk for Josh at Elanah.ai. Source: talkwise-chronicler recording 7c3be722.*

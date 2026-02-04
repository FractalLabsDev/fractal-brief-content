# Practice Interviews Rebuild Evaluation

**For:** Matt Lim  
**Date:** February 4, 2026  
**Context:** Evaluating the proposal to rebuild PI from scratch with Supabase vs. continuing with current architecture

---

## Executive Summary

The rebuild proposal addresses real pain points, but carries significant risks given Jeff's timeline pressures. A **hybrid approach**—migrating users/permissions from TalkWise to PI backend while preserving the existing codebase—may deliver 80% of the benefit at 20% of the risk.

---

## Jeff's Priorities (From Meeting History)

### Timeline Pressure
Jeff has accelerated his exit timeline from December 2027 to **December 2026**. He stated in the Dec 8 dev sync:

> "I've been like I my target has been really fixed on December of 2027 to be out and sell my product and then I had this like epiphany when I was biking to work. I was like why can't it be December of next year?"

He's also borrowing against assets and needs revenue before running out of runway.

### Product Philosophy
Jeff consistently emphasizes:
- **Revenue-generating features over technical elegance**: "Product work directly protects or expands revenue and product velocity without revenue impact is a liability, not an asset"
- **Simplicity**: He doesn't care about infrastructure decisions; he wants it to work
- **Standalone product**: Building systems to separate coaching from the product for acquisition

### What Jeff Gets Excited About
From meeting transcripts, Jeff's enthusiasm spikes for:
1. **B2B/Enterprise features** - College integrations, licensing model
2. **Interview Prep paradigm** - Moving from "training ground" to actual interview preparation
3. **Reduced friction** - Better onboarding, simpler user experience
4. **Analytics** - Meaningful metrics around rigid practice sessions

---

## Current Roadmap (From Fractal OS)

| Epic | Status | Target |
|------|--------|--------|
| Enterprise Licensing Model | in-progress | Feb 28 |
| Replace Wordware | in-progress | Feb 28 |
| Filler Features (onboarding checklist) | in-progress | Jan 30 |
| Rigid Practice Sessions | planned | Feb 27 |
| Story Bank | planned | Feb 27 |
| Question Bank | planned | Mar 17 |
| Interview Prep | planned | Apr 30 |
| College/University Integration | planned | Aug 01 |

**Critical observation:** The "Interview Prep" feature (which ties everything together conceptually) isn't targeted until April 30. This creates a risk of shipping building blocks that don't get used until they're unified.

---

## Current Architecture Pain Points

### What Max Identified as Problems
1. **PI ↔ TalkWise connection** - "/me requests take 1s in prod, 5s in staging"
2. **Role-based access service** - No clear feature-to-role matrix
3. **User management queries** - Can't efficiently query users from TalkWise
4. **Logging** - Huge, unreadable logs during AI generation
5. **Stripe integration** - Not seamless, pricing stored in DB to avoid rate limits

### What Max Says Is NOT That Bad
- PI backend on its own: "clear endpoints separation," TypeScript, Joy validation
- Database entities: "quite clean, no orphan data"
- Migrations, seeders, build scripts all in place

### What's Truly Broken
- **TalkWise backend** - Max called it "Frankenstein"
- **Three-backend architecture** - Frontend → PI Backend → TalkWise Backend creates confusion for agents
- **Wordware integration** - Prompts stored in env vars, no visibility into failures

---

## Architecture Overview

```
Current State:
┌─────────────────┐     ┌──────────────────┐     ┌─────────────────┐
│  PI Frontend    │────▶│   PI Backend     │────▶│ TalkWise Backend│
│  (React/Recoil) │     │   (Express)      │     │   (Express)     │
│                 │     │   - Questions    │     │   - Users       │
│                 │     │   - Answers      │     │   - Roles       │
│                 │     │   - Jobs         │     │   - Permissions │
└─────────────────┘     └──────────────────┘     └─────────────────┘
        │                       │                        │
        └───────────────────────┴────────────────────────┘
                    Heroku / PostgreSQL

Proposed State (Supabase):
┌─────────────────┐     ┌──────────────────┐
│  New Frontend   │────▶│   New Backend    │────▶ Supabase (DB + Auth)
│  (React)        │     │   (Express)      │
└─────────────────┘     └──────────────────┘
                               │
                    ┌──────────┴──────────┐
                    ▼                     ▼
              PromptSmith           Other Services
```

---

## Risk Assessment

### Risks of Rebuilding

| Risk | Severity | Mitigation |
|------|----------|------------|
| 4-week estimate becomes 8-12 weeks | High | Time-box strictly, define MVP scope |
| Feature regression (Jeff's Loom complaints) | High | E2E tests from day one |
| Migration complexity (users, data, Stripe) | Medium | Can reuse existing DB, migration later |
| Supabase learning curve | Medium | Matt already comfortable with it |
| Team context switch disruption | Medium | Max focused on rebuild, Austin on other projects |
| Jeff loses patience during rebuild | High | Clear communication about "refactoring phase" |

### Risks of NOT Rebuilding

| Risk | Severity | Mitigation |
|------|----------|------------|
| Continued slow iteration on features | High | Max's 5-point improvement plan |
| DX problems blocking Rock's effectiveness | Medium | E2E test automation |
| Growing technical debt | Medium | Incremental refactoring |
| Interview Prep requires reimagining UI anyway | High | None—this work is unavoidable |

---

## Max's Alternative: 5-Point Improvement Plan

Max proposed these quick wins that could improve DX without a full rebuild:

1. **Script library** - Find and remove orphaned code
2. **Storybook** - Aligned UI components
3. **Compact logging** - Make logs readable
4. **Single main branch** - Forget dev branch (like Vitaboom)
5. **Automated E2E tests** - Run on every PR

He estimated these are "doable in one evening or maybe couple of evenings each."

---

## Recommendation

### Option A: Full Rebuild (High Risk, High Reward)
- Rebuild frontend + backend from scratch
- Use Supabase for prototyping, migrate to AWS later
- 4-week timeline to reach feature parity
- **Best if:** You're confident in the estimate and can communicate the pause to Jeff

### Option B: Targeted Migration (Medium Risk, Medium Reward) ⭐ Recommended
- Keep PI Backend, **migrate users/roles/permissions tables from TalkWise to PI**
- Implement Max's 5-point improvement plan
- Kill TalkWise backend dependency
- Rebuild frontend incrementally as you ship new features
- **Best if:** You want to reduce risk while still addressing the core pain point

### Option C: Status Quo + Improvements (Low Risk, Low Reward)
- Implement Max's 5-point plan
- Continue building features in current architecture
- Accept slower iteration velocity
- **Best if:** Jeff's timeline is more important than long-term velocity

---

## Key Questions for Austin

1. **Supabase vs. AWS consistency**: Is introducing Supabase worth the platform fragmentation?
2. **TalkWise deprecation timeline**: Is there a broader plan to sunset TalkWise backend across all projects?
3. **Team capacity**: Can Max realistically do a rebuild while maintaining existing features?
4. **Jeff communication**: How do we frame this as "architectural improvement" rather than "starting over"?

---

## My Take

The PI ↔ TalkWise connection is genuinely the root problem. Max is right that killing that dependency would solve most DX issues. But a full Supabase rebuild introduces platform risk and learning curve overhead.

The **hybrid approach** (Option B) addresses the core issue—users and permissions stuck in TalkWise—without betting the farm on a rebuild estimate. It also preserves Max's good work on the PI backend.

That said, if you and Max are confident in the 4-week estimate and Jeff can tolerate a "refactoring sprint," the full rebuild would set you up much better for the Interview Prep feature rollout.

---

*Generated by Ruk from semantic memory search across 255 meetings, 19 PI epics, and codebase exploration.*

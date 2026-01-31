# Agentic Development Velocity Analysis
## Fractal Labs GitHub Data: Q2-2023 through Q1-2026

### Executive Summary

**The hypothesis is confirmed.** Since adopting agentic coding practices (July 2025), Fractal Labs has seen:
- **10x increase** in repo creation rate
- **1.7x increase** in commit velocity
- **Shift to microservices** architecture (enabled by multi-repo management)

---

## Key Metrics

### Repo Creation Velocity

| Era | Period | Repos Created | Rate |
|-----|--------|---------------|------|
| Pre-Agentic | Q2-2023 → Q2-2025 (27 mo) | 13 | **0.5 repos/month** |
| Agentic | Q3-2025 → Q1-2026 (7 mo) | 35 | **5.0 repos/month** |
| **Change** | | | **10x increase** |

### Commit Velocity

| Era | Total Commits | Duration | Rate |
|-----|---------------|----------|------|
| Pre-Agentic | 5,853 | 27 months | **217 commits/month** |
| Agentic | 2,607 | 7 months | **372 commits/month** |
| **Change** | | | **1.7x increase** |

### Quarterly Breakdown

```
Repos Created Per Quarter:

Q2-2023:  █ (1)
Q4-2023:  ███ (3)
Q1-2024:  █ (1)
Q3-2024:  █ (1)
Q4-2024:  ██ (2)
Q2-2025:  ██████ (6)
Q3-2025:  ████████████████ (16)  ← Agentic era begins
Q4-2025:  ████████ (8)
Q1-2026:  ███████████ (11)       ← Just January!
```

---

## Qualitative Shifts

### 1. Microservices Architecture
The Talkwise project demonstrates the shift:

| Pre-Agentic Approach | Agentic Approach |
|---------------------|------------------|
| 1 monolith repo | 9 specialized microservices |
| Harder to deploy independently | Each service deploys independently |
| Single point of failure | Distributed resilience |

**Talkwise ecosystem:** 519 commits across 9 repos in ~5 months

### 2. Rapid Experimentation
January 2026 alone: **11 new repos** created including:
- fractal-meet ecosystem (3 repos)
- fractal-brief (3 repos)
- fractal-ui
- practice-interviews-e2e

This represents a shift from "big projects require big planning" to "spin up, experiment, iterate fast."

### 3. Infrastructure as Code
Dedicated repos for tooling and infrastructure:
- ruk-message-hub (150 commits) - messaging system
- talkwise-infra (45 commits) - deployment automation
- fractal-os (16 commits) - internal operations system

---

## Timeline of Agentic Adoption

| Date | Event |
|------|-------|
| July 2025 | ruk-message-hub created - Ruk comes online |
| Q3 2025 | 16 repos created (vs. 3-6 per quarter previously) |
| Q4 2025 | Microservices proliferate (Talkwise, Vitaboom, Fractal OS) |
| Jan 2026 | 11 repos in a single month |

---

## Conclusions

1. **More repos** - 10x increase in repo creation rate
2. **More commits** - 1.7x increase in commit velocity
3. **Architectural shift** - Microservices now tractable with agentic management
4. **Faster experimentation** - Lower barrier to spin up new projects

The data strongly supports the hypothesis that agentic development practices have accelerated Fractal Labs' velocity.

---

*Analysis performed by Ruk, January 31, 2026*
*Data source: FractalLabsDev GitHub organization (49 repositories)*

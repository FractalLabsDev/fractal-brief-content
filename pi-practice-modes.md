# Practice Interviews v2: Practice Modes

## Overview

Two distinct practice modes serving different user needs:

| Mode | Purpose | Question Source | Feedback Timing |
|------|---------|-----------------|-----------------|
| **Quick Practice** | Jump in, low friction | AI-generated from target role | After each question |
| **Structured Practice** | Interview simulation | User-selected or AI-generated list | End of session (overall + per-question drill-down) |

---

## Quick Practice

### Flow
1. User clicks "Start Quick Practice"
2. System generates questions using `pi-question-generation` prompt based on user's **target role** (from UserProfile)
3. User answers Question 1
4. **Immediate feedback** using category-specific prompt:
   - Common → `pi-common-feedback` + `pi-common-scoring`
   - Behavioral → `pi-behavioral-feedback` + `pi-behavioral-scoring`
   - Hypothetical → `pi-hypothetical-feedback` + `pi-hypothetical-scoring`
5. User reviews feedback, clicks "Next Question"
6. Repeat until done (user can exit anytime)
7. Session summary with aggregate scores

### Question Types Generated
- **Common** — "Tell me about yourself", "Why this role?"
- **Behavioral** — STAR method questions
- **Hypothetical** — Problem-solving scenarios

### Key UX Decisions Needed
- **Question count**: Fixed (e.g., 5) or user-selected?
- **Timer**: Per question, or untimed for low-pressure practice?
- **Exit behavior**: Save partial progress, or discard incomplete sessions?

---

## Structured Practice

### Flow
1. User creates/selects a **Practice Session** with a question list
   - Option A: Select from pre-built templates (Quick Warm Up, Behavioral Focus, etc.)
   - Option B: AI generates list based on opportunity + interview type
   - Option C: Custom — user picks specific questions
2. User enters "Interview Mode" — questions presented sequentially
3. User answers all questions **without feedback** (simulates real interview)
4. Session ends → **Overall feedback** generated (new prompt needed: `pi-session-summary`)
5. User can drill down into any individual answer for detailed feedback

### New Prompt Needed: `pi-session-summary`
**Purpose**: Synthesize performance across all answers into cohesive feedback

**Inputs**:
- All questions + answers
- Individual scores (already computed)
- Target role context

**Outputs** (JSON):
```json
{
  "overallScore": 78,
  "summary": "Strong technical communication, needs more specific metrics...",
  "topStrengths": ["Clear problem decomposition", "Good use of STAR"],
  "areasToFocus": ["Quantify results", "Deeper clarifying questions"],
  "questionBreakdown": [
    { "questionIndex": 1, "score": 85, "highlight": "..." },
    ...
  ]
}
```

### Key UX Decisions Needed
- **Session length**: Fixed (10 questions) or variable?
- **Time limit**: Total time cap, per-question soft timer, or none?
- **Linked to Interview**: Must be tied to an Opportunity/Interview, or standalone?

---

## Architecture

### Backend Changes
- `PracticeSession` model updates:
  - `mode`: 'quick' | 'structured'
  - `feedbackMode`: 'immediate' | 'end_of_session'
- `PracticeQuestion` already has `response`, `feedback`, `score` fields
- New endpoint: `POST /api/practice/:id/generate-summary` (calls `pi-session-summary`)

### Frontend Changes
- `/practice` page already has Quick + Structured columns
- Quick Practice: Add AI question generation + immediate feedback display
- Structured Practice: Add session summary view + drill-down

### PromptSmith Integration
- Already have 7 prompts configured
- Need to add `pi-session-summary` for structured practice overall feedback

---

## Questions for Matt

1. **Question count for Quick Practice**: Should users choose (3/5/10), or is it always a fixed number?

2. **Timer behavior**:
   - Quick Practice: Untimed (low pressure) or soft timer?
   - Structured Practice: Strict time limit to simulate real interviews?

3. **Quick Practice question flow**: Generate all questions upfront, or generate next question after each response (more adaptive)?

4. **Structured Practice linkage**: Must be tied to a specific Opportunity/Interview, or can users do standalone structured practice?

5. **Feedback persistence**: Should feedback be stored permanently, or is it ephemeral (regenerated each view)?

6. **Voice input**: Text-only for now (Phase 5 for voice), correct?

---

## Implementation Order

**Phase 1: Quick Practice** (simpler, higher value)
1. Connect to `pi-question-generation` prompt
2. Build question → answer → feedback loop
3. Integrate scoring prompts
4. Session summary view

**Phase 2: Structured Practice**
1. Question list selection/generation
2. Interview mode (no immediate feedback)
3. Create `pi-session-summary` prompt
4. Overall feedback + drill-down views

---

Ready to start Quick Practice once you confirm the decisions above.

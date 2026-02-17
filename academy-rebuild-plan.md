# Academy Feature Rebuild Plan

## Overview

Recreate the Academy feature from `practice-interviews-web`/`practice-interviews-backend` in the new `interview-prep-web`/`interview-prep-backend` codebase, using the existing design system and avoiding accumulated technical debt.

**Scope:** Build all UI components for the Academy flow. For Rate My Setup and Rate My Audio, build the UI shell only (no AI analysis integration yet). Resume analysis is excluded.

---

## Current State

### What Already Exists in interview-prep-backend
- ✅ `Step`, `UserStep`, `UserBadge` models (Sequelize)
- ✅ Academy routes at `/api/v1/academy`
- ✅ Academy service with `getStepsByLevelNumber`, `completeStep`, `moveToNextSubstep`
- ✅ Badge service with 6 badge rules
- ✅ Feature gating via `requireFeature('interviewAcademy')`

### What Already Exists in interview-prep-web
- ✅ `useAcademy` hook with level/step helpers
- ✅ `useCompleteStep` and `useMoveToNextSubstep` mutations
- ✅ Badge utilities and notification logic
- ✅ MUI theme with Academy-specific colors
- ✅ `react-player` for video playback
- ✅ Step/UserStep TypeScript types

### What Needs to Be Built (Frontend)
- [ ] `InterviewAcademyHome` - Dashboard showing all levels + progress
- [ ] `InterviewAcademyLevel` - Level view with sidebar and content area
- [ ] `VideoStep` - Video player with substep tracking
- [ ] `RateMySetupStep` - Image capture UI (shell only)
- [ ] `RateMyAudioStep` - Audio recording UI (shell only)
- [ ] `QuestionsStep` - Question answering flow (common, behavioral, hypothetical)
- [ ] Level navigation and unlock logic
- [ ] Badge notification display

### What Needs to Be Built (Backend)
- [ ] Questions seeder for Academy (if not already present)
- [ ] Video content configuration (database or config file)
- [ ] Potentially: Resume upload endpoint integration

---

## Academy User Flow (4 Levels)

### Level 1: Interview Foundations
1. **VideoStep** - 17 foundation videos (research, mindset, body language, etc.)
2. **RateMySetupStep** - Take photo → AI feedback (UI only for now)
3. **RateMyAudioStep** - Record voice → AI feedback (UI only for now)

### Level 2: Common Questions
1. **VideoStep** - 6 videos on structuring responses
2. **UploadResumeStep** - Excluded (skip for now)
3. **CommonQuestionsStep** - Answer 3+ common interview questions

### Level 3: Behavioral Questions
1. **VideoStep** - 19 videos teaching STAR method
2. **BehavioralQuestionsStep** - Answer 3+ behavioral questions

### Level 4: Hypothetical Questions
1. **VideoStep** - 19 videos on CFAS methodology
2. **HypotheticalQuestionsStep** - Answer 3+ hypothetical questions

---

## Technical Approach

### 1. Database / Content Management

**Problems to Avoid:**
- Original has hardcoded video URLs in `videoContent.ts` (100+ YouTube URLs)
- Original has hardcoded badge rules in service code
- No admin UI to manage content

**Better Approach:**
```
Option A: Database-driven content
- Create `AcademyVideo` model: { stepId, substep, title, youtubeId, duration }
- Create seeder with all video content
- Allows admin to update videos without code deploy

Option B: Config file (simpler)
- Keep videos in JSON config file
- Easier to version control
- Load at runtime
```

**Recommendation:** Option B for MVP (config file), migrate to Option A when admin UI is needed.

### 2. Frontend Architecture

**Component Structure:**
```
/views/home/InterviewAcademy/
├── InterviewAcademyHome.tsx      # Dashboard with all levels
├── InterviewAcademyLevel.tsx     # Single level view
├── components/
│   ├── LevelCard.tsx             # Level progress card
│   ├── StepSidebar.tsx           # Step navigation sidebar
│   ├── StepContent.tsx           # Step content router
│   └── BadgeNotification.tsx     # Badge earned toast
├── steps/
│   ├── VideoStep.tsx             # Video player + substep tracking
│   ├── RateMySetupStep.tsx       # Image capture UI
│   ├── RateMyAudioStep.tsx       # Audio recording UI
│   └── QuestionsStep.tsx         # Question answering flow
└── data/
    └── videoContent.ts           # Video configuration
```

**Routing:**
```
/interview-academy                    → InterviewAcademyHome
/interview-academy/level/:levelNumber → InterviewAcademyLevel
```

### 3. Step Components

#### VideoStep
```typescript
interface VideoStepProps {
  step: Step;
  videos: Video[];
  onSubstepComplete: () => void;
  onStepComplete: () => void;
}
```
- Display current video using `react-player`
- Show video title, description
- "Next Video" button advances substep
- Track watched state per video
- After last video, enable "Complete Step" button

#### RateMySetupStep (UI Shell)
```typescript
interface RateMySetupStepProps {
  step: Step;
  onComplete: () => void;
}
```
- Camera capture interface
- Display captured image
- "Submit for Review" button (placeholder action)
- Skip/Complete button for now (until AI integration)
- Clear instructions on what makes a good setup

#### RateMyAudioStep (UI Shell)
```typescript
interface RateMyAudioStepProps {
  step: Step;
  onComplete: () => void;
}
```
- Audio recording interface (use existing AudioCapture component)
- Playback of recorded audio
- "Submit for Review" button (placeholder action)
- Skip/Complete button for now
- Display the phrase to speak

#### QuestionsStep
```typescript
interface QuestionsStepProps {
  step: Step;
  questionType: 'common' | 'behavioral' | 'hypothetical';
  onComplete: () => void;
}
```
- Fetch questions by type
- Let user select 3+ questions
- Recording interface for each question
- Feedback display after each answer
- Progress through all selected questions
- Complete step after all answered

### 4. State Management

**Recoil Atoms:**
```typescript
// Current level being viewed
export const currentLevelState = atom<number>({
  key: 'academy_currentLevel',
  default: 1,
});

// Current step within level
export const currentStepState = atom<number>({
  key: 'academy_currentStep',
  default: 1,
});
```

**React Query:**
- Use existing `useAcademy` hook for level/step data
- Use existing `useCompleteStep` and `useMoveToNextSubstep` mutations
- Add `useAcademyQuestions(type)` for fetching questions

### 5. Progress & Unlock Logic

**Level Unlock Rules:**
- Level 1: Always unlocked
- Level N (N>1): Unlocked when Level N-1 is complete

**Step Unlock Rules:**
- Step 1 of each level: Unlocked when level is unlocked
- Step N (N>1): Unlocked when Step N-1 is complete

**Progress Calculation:**
- Per-step: `substep / totalSubsteps` or `completedAt ? 1 : 0`
- Per-level: `completedSteps / totalSteps`
- Overall: `completedLevelSteps / totalLevelSteps` across all levels

### 6. Badge System

**Existing Badges (no changes needed):**
| Badge ID | Trigger |
|----------|---------|
| `first-substep-complete` | Level 1, Step 1, reach substep 2 |
| `first-level-complete` | Complete Level 1 |
| `second-level-complete` | Complete Level 2 |
| `third-level-complete` | Complete Level 3 |
| `fourth-level-complete` | Complete Level 4 |
| `fifth-level-complete` | Complete all levels |

**Badge Display:**
- Toast notification when badge earned (5-minute window)
- Badge collection view in Academy home
- Use existing `badgeUtils` from frontend

---

## Technical Debt to Avoid

### From Original Implementation

| Issue | Solution |
|-------|----------|
| Hardcoded video content in TSX | External config file |
| Badge rules hardcoded in service | Keep as-is for MVP, refactor to DB later |
| No idempotency on substep increment | Add idempotency key or check current state |
| Race conditions in findOrCreate | Use transaction or upsert |
| Client-side question filtering | Server-side pagination + filtering |
| No step order enforcement | Backend validation of unlock rules |
| No recording persistence | Store recordings for user review (future) |
| localStorage badge tracking | Server-side badge view tracking (future) |

### New Implementation Guidelines

1. **Single source of truth for content** - Config file or database, not scattered TSX
2. **Server-side validation** - Don't trust client for unlock/completion logic
3. **Optimistic UI with rollback** - Update UI immediately, rollback on error
4. **Proper error boundaries** - Each step component wrapped in error boundary
5. **Accessible design** - Keyboard navigation, screen reader support
6. **Mobile-responsive** - Academy should work on all screen sizes

---

## Implementation Phases

### Phase 1: Core Infrastructure (2-3 days)
- [ ] Create video content config file
- [ ] Add routing for Academy pages
- [ ] Build `InterviewAcademyHome` dashboard
- [ ] Build `InterviewAcademyLevel` container
- [ ] Build `LevelCard` and `StepSidebar` components

### Phase 2: Video Step (2 days)
- [ ] Build `VideoStep` component
- [ ] Integrate with `react-player`
- [ ] Implement substep tracking
- [ ] Test video progression flow

### Phase 3: Setup & Audio Steps (2 days)
- [ ] Build `RateMySetupStep` UI shell
- [ ] Build `RateMyAudioStep` UI shell
- [ ] Use existing `ImageCapture` and `AudioCapture` components
- [ ] Add skip/complete flow for now

### Phase 4: Questions Steps (3-4 days)
- [ ] Build `QuestionsStep` base component
- [ ] Integrate question selection UI
- [ ] Build recording + feedback flow
- [ ] Add `CommonQuestionsStep`, `BehavioralQuestionsStep`, `HypotheticalQuestionsStep` wrappers
- [ ] Ensure questions seeder is in place

### Phase 5: Badge System & Polish (1-2 days)
- [ ] Build `BadgeNotification` component
- [ ] Integrate with existing badge utilities
- [ ] Add badge collection view
- [ ] Polish transitions and loading states
- [ ] Mobile responsiveness pass

### Phase 6: Testing & QA (ongoing)
- [ ] Unit tests for key components
- [ ] Integration tests for user flows
- [ ] E2E test for complete Academy journey

---

## File Changes Summary

### interview-prep-web (Frontend)

**New Files:**
```
/src/views/home/InterviewAcademy/
├── InterviewAcademyHome.tsx
├── InterviewAcademyLevel.tsx
├── components/
│   ├── LevelCard.tsx
│   ├── StepSidebar.tsx
│   ├── StepContent.tsx
│   └── BadgeNotification.tsx
├── steps/
│   ├── VideoStep.tsx
│   ├── RateMySetupStep.tsx
│   ├── RateMyAudioStep.tsx
│   └── QuestionsStep.tsx
└── data/
    └── videoContent.ts
```

**Modified Files:**
- `/src/routes.tsx` - Add Academy routes
- `/src/components/navigation/` - Add Academy nav item
- `/src/middleware/` - Any new hooks needed

### interview-prep-backend (Backend)

**Potentially Modified:**
- `/src/db/seeders/` - Ensure questions seeder exists
- `/src/routes/academy.route.ts` - Any new endpoints needed

---

## Open Questions for Matt

1. **Video Content:** Do we have the same YouTube video IDs, or will new videos be recorded?
2. **Question Pool:** Should we migrate existing questions from old DB, or start fresh?
3. **Resume Step:** Confirm exclusion - will this be added later?
4. **Rate My Setup/Audio:** When do you want AI integration added?
5. **Mobile Priority:** Is mobile responsiveness required for MVP?
6. **Admin UI:** Do we need admin controls for content management?

---

## Success Criteria

- [ ] User can view Academy dashboard with all 4 levels
- [ ] User can progress through each level's steps
- [ ] Video steps track watched videos correctly
- [ ] Setup and Audio steps have functional UI (even without AI)
- [ ] Question steps allow selecting and answering questions
- [ ] Badges are awarded at appropriate milestones
- [ ] Progress persists across sessions
- [ ] UI follows design system patterns
- [ ] No major technical debt introduced

# Practice Interviews: Full Feature Parity Checklist

**Purpose:** Complete inventory of everything needed to rebuild PI with full feature parity  
**Source:** Analyzed from practice-interviews-web, practice-interviews-backend, and fractalai-backend (TalkWise)

---

## Summary

| Category | Features | Complexity |
|----------|----------|------------|
| Authentication & Users | 12 | High |
| Practice Interview Core | 15 | High |
| Job/Role Management | 6 | Medium |
| Analytics & Progress | 8 | Medium |
| Interview Academy | 7 | Medium |
| Onboarding | 6 | Medium |
| Admin Panel | 14 | High |
| Subscriptions & Billing | 9 | High |
| B2B/Enterprise | 8 | Medium |
| Integrations | 6 | Medium |
| **Total** | **91** | |

---

## 1. Authentication & User Management

Currently split between TalkWise backend and PI backend.

| Feature | Current Location | Notes |
|---------|------------------|-------|
| Email/password login | TalkWise | auth.route.ts |
| Google OAuth | TalkWise | auth.route.ts |
| Password reset flow | TalkWise | setPassword.route.ts |
| Email verification | TalkWise | invitation.route.ts |
| User profile (name, avatar) | TalkWise | user.route.ts |
| Session management | TalkWise | JWT tokens |
| Role assignment | TalkWise | role.route.ts |
| Permission checks | TalkWise | middleware |
| Invitation system (B2B) | TalkWise | invitation.route.ts |
| User blocking | TalkWise | admin functions |
| Account deletion | TalkWise | user.route.ts |
| Profile preferences | PI Backend | Extended user data |

**Supabase equivalent:** Supabase Auth handles most of this natively. Custom profile data in a `profiles` table.

---

## 2. Practice Interview Core (The Money Feature)

| Feature | Current Location | Notes |
|---------|------------------|-------|
| Start practice session | PI Backend | question.route.ts |
| Question generation (AI) | Wordware → PromptSmith | Multiple question types |
| Question categories (behavioral, technical, situational) | PI Backend | Enum types |
| Voice recording | Frontend | MediaRecorder API |
| Real-time transcription | Deepgram | WebSocket streaming |
| Answer submission | PI Backend | answer.route.ts |
| AI feedback generation | Wordware → PromptSmith | Multiple feedback prompts |
| Feedback display (scores, suggestions) | Frontend | FeedbackHeader, DeliveryMetrics |
| Ideal answer generation | PromptSmith | New addition |
| Follow-up questions | Wordware | Chains off original |
| Question replay (audio) | Frontend | Text-to-speech |
| Skip question | Frontend | Session flow |
| End session early | Frontend | Session flow |
| Session history | PI Backend | answer.route.ts |
| Retry answer | PI Backend | Linked to original |

---

## 3. Job/Role Management

| Feature | Current Location | Notes |
|---------|------------------|-------|
| Create target job | PI Backend | job.route.ts |
| Edit job details | PI Backend | job.route.ts |
| Delete job | PI Backend | job.route.ts |
| Job-specific question generation | PI Backend | Context passed to AI |
| Company research (experimental) | PI Backend | company.route.ts |
| Resume parsing for job context | PI Backend | resume.route.ts |

---

## 4. Analytics & Progress Tracking

| Feature | Current Location | Notes |
|---------|------------------|-------|
| Questions answered count | PI Backend | analytics.route.ts |
| Average scores over time | PI Backend | analytics.route.ts |
| Weekly progress chart | Frontend | WeeklyTrendChart |
| Category breakdown | PI Backend | By question type |
| Goals/targets | PI Backend | userGoal.route.ts |
| Steps/milestones | PI Backend | userStep.route.ts |
| Badge system | PI Backend | userBadge model |
| Streak tracking | PI Backend | Calculated from answers |

---

## 5. Interview Academy

| Feature | Current Location | Notes |
|---------|------------------|-------|
| Course content delivery | PI Backend + Notion | academy.route.ts |
| Video lessons | Notion embed | External hosting |
| Progress tracking per lesson | PI Backend | userStep model |
| Module completion | PI Backend | Step model |
| Quiz/assessment | PI Backend | Linked to questions |
| Certificate generation | TBD | Not yet implemented |
| Academy-specific question banks | Notion | Content sync |

---

## 6. Onboarding Flow

| Feature | Current Location | Notes |
|---------|------------------|-------|
| Welcome screens | Frontend | SplitScreenLayout |
| Job role selection | Frontend | Onboarding flow |
| Experience level | Frontend | Personalization |
| Goal setting | Frontend | userGoal |
| First practice prompt | Frontend | Quick start |
| Onboarding checklist | Frontend | New feature (in progress) |

---

## 7. Admin Panel

| Feature | Current Location | Notes |
|---------|------------------|-------|
| User list with search | Frontend + TalkWise | UsersTable |
| User detail view | Frontend | UserAnalyticsPanel |
| User answer history | Frontend | UserAnswerFeedbackTable |
| Block/unblock user | Frontend + TalkWise | BlockUserConfirmDialog |
| Add users manually | Frontend | AddUsersDialog |
| Assign users to clients (B2B) | Frontend + TalkWise | Client assignment |
| Client list | Frontend | clients/ components |
| Create client (B2B org) | Frontend | AddClientDialog |
| Client UTM links | Frontend | ClientUTMLinksDialog |
| Dashboard stats | Frontend | StatCard, DashboardCard |
| Weekly metrics | Frontend | WeeklyMetricCard |
| Charts/visualizations | Frontend | charts/ |
| Payment plan labels | Frontend | PaymentPlanLabel |
| Export data | TBD | Not implemented |

---

## 8. Subscriptions & Billing

| Feature | Current Location | Notes |
|---------|------------------|-------|
| Stripe checkout | TalkWise | price.route.ts |
| Subscription tiers (Starter, Pro, Enterprise) | TalkWise + Stripe | Configured in Stripe |
| Usage limits by tier | PI Backend | Middleware checks |
| Upgrade/downgrade flow | Frontend + Stripe | Portal redirect |
| Cancel subscription | Stripe Portal | External |
| Promo codes | TalkWise | promo.route.ts |
| Trial period | Stripe | Configured in Stripe |
| Invoice history | Stripe Portal | External |
| Payment method management | Stripe Portal | External |

---

## 9. B2B/Enterprise Features

| Feature | Current Location | Notes |
|---------|------------------|-------|
| Client/org creation | TalkWise | client.route.ts |
| User assignment to client | TalkWise | Invitation system |
| Bulk user invite | TalkWise | invitation.route.ts |
| Client-specific branding (TBD) | Not implemented | Roadmap |
| UTM tracking for client signups | TalkWise | clientUTMLink.route.ts |
| Client admin role | TalkWise | role.route.ts |
| Compliance data collection | Frontend | compliance/ |
| License seat management | In progress | E-15 epic |

---

## 10. Integrations

| Feature | Current Location | Notes |
|---------|------------------|-------|
| Deepgram (transcription) | PI Backend proxy | Real-time WebSocket |
| Wordware → PromptSmith | Migration in progress | AI prompts |
| Notion (Academy content) | PI Backend | notion.route.ts |
| Customer.io (email) | TalkWise | customerio.route.ts |
| Stripe (payments) | TalkWise | price.route.ts, webhooks |
| Text-to-speech (question audio) | TalkWise | textToSpeech.route.ts |

---

## 11. Miscellaneous

| Feature | Location | Notes |
|---------|----------|-------|
| Rating/feedback on answers | PI Backend | rating.route.ts |
| Health check endpoints | Both backends | health.route.ts |
| Error tracking | Sentry | Configured |
| Analytics events | Customer.io / Segment | Various triggers |
| Mobile responsive UI | Frontend | TailwindCSS |

---

## Database Schema Summary

### PI Backend (PostgreSQL - Heroku)
```
jobs
questions
answers
steps
user_steps
user_goals
user_badges
companies
scheduled_processes
```

### TalkWise Backend (PostgreSQL - Heroku)
```
users
roles
user_roles
permissions
role_permissions
clients
client_utm_links
subscriptions
prices
products
promos
invitations
email_logs
email_templates
feedback
threads
messages
audio_mappings
onboarding
user_actions
user_action_mappings
```

---

## Rebuild Effort Estimate

| Category | Est. Days | Notes |
|----------|-----------|-------|
| Auth + User Management | 3-4 | Supabase Auth helps significantly |
| Practice Interview Core | 5-7 | Most complex, needs PromptSmith integration |
| Job Management | 1 | Straightforward CRUD |
| Analytics | 2-3 | Queries and charts |
| Interview Academy | 2-3 | Depends on Notion integration |
| Onboarding | 1-2 | UI work mostly |
| Admin Panel | 4-5 | Lots of UI + queries |
| Subscriptions | 3-4 | Stripe integration complexity |
| B2B Features | 2-3 | Client/invitation system |
| Integrations | 2-3 | Deepgram, PromptSmith, etc. |
| Testing + Polish | 3-5 | E2E tests, bug fixes |
| **Total** | **28-40 days** | ~6-8 weeks with one developer |

---

## What Could Be Simplified in Rebuild

| Current Complexity | Simplification |
|--------------------|----------------|
| TalkWise user/role system | Supabase Auth + RLS policies |
| Multiple databases | Single Supabase database |
| Wordware prompts in env vars | PromptSmith API (already done) |
| Socket streaming for transcription | Could keep or simplify to polling |
| Academy via Notion | Could embed directly or use CMS |
| Stripe subscription sync | Supabase webhooks + table |

---

## Critical Path for "Feels Complete"

If you want users to not notice a difference, you need:

1. **Auth** - Login, signup, password reset
2. **Practice flow** - Questions → Record → Feedback
3. **Job setup** - Target role configuration
4. **History** - See past answers
5. **Subscriptions** - Payment gating works

Everything else (Academy, Admin, B2B, Analytics) can come later without users noticing degradation.

---

*91 features identified across 11 categories. Estimated 6-8 weeks for full parity with single developer.*

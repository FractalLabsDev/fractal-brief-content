# Practice Interviews Weekly Digest
*Week of February 2, 2026*

## What Changed This Week

**Usage Control & Rate Limiting**
- Implemented per-user rate limiting instead of global limits (#406, #98)
- Added 10 answers/day limit for free and trial users (#100)
- Prevented duplicate answer submissions to reduce API waste (#101)

**Infrastructure & Reliability**
- Deployed February 4th release to both web and backend (#408, #102)
- Upgraded to cleaner Pino logging system for better log management (#96)
- Fixed Pino node compatibility issues (#99)
- Updated PostHog analytics key configuration (#412)

**AI Answer Generation**
- Integrated new talkwise-promptsmith system for generating ideal answers (#94)
- Updated frontend to use the new ideal answer endpoint (#404)

## What's In Flight

**Security & Fraud Prevention**
- FingerprintJS Pro integration for device fingerprinting and abuse detection (PR #407, by ruk-fl) - *needs review*

**Bug Fixes & UX Improvements**
- Home screen analytics reliability and "Mix of all three" UI fixes (PR #405, by ruk-fl) - *5 days old, needs review*
- Product walkthrough bug fixes including field/role population issues (PR #403, by ruk-fl) - *6 days old, pending review*

**Question Management**
- Question ordering fixes for common questions and consecutive caps (PR #95, by ruk-fl) - *5 days old, needs review*

## What's Outstanding

**High Priority Bugs**
- Subscription modal has inconsistent card sizes and hidden upgrade button (#402)
- Login button on landing page doesn't navigate properly (#398)
- Feedback page shows 'NA' instead of loading spinner (#400)
- Double loading spinners in Interview Academy body language section (#401)
- Field/role not populating in summary banner after onboarding (#399)
- "About me" source attribution and notification issues (#76)

**Feature Backlog**
- Move rate setup/audio features into core app (#324)
- Add job entity descriptions (#378)
- Enhanced home page stats endpoint (#377)
- Replace moment with date-fns (#308)
- Auto-generate release notes (#64)
- Automated DB backup script (#50)

**Tech Debt & Other Items**
- Centralize z-index management in theme (#393)
- Missing job descriptions in pre-loaded data (#387)
- Hypothetical questions not generating scores (#386)
- Side panel responsiveness improvements (#384)
- Analytics page color consistency (#383)
- *Plus 11 other items in backlog*

---

# Therapist Genie Weekly Digest
*Week of February 2, 2026*

## What Changed This Week

**Performance & UX Improvements**
- Fixed app freezing while typing in progress notes (#261) - resolved expensive re-render issues causing keystroke lag
- Added client signature capability for treatment plans (#258, #78) - clients can now digitally sign treatment plans through the client portal

**Critical Bug Fixes**  
- Resolved multiple session note issues: save/sign functionality, checkbox selection bugs, and progress note approval workflow (#233, #253, #77)
- Fixed telehealth session marking and clinician dropdown population (#233, #249)
- Expanded ICD-10 diagnosis codes from 12 to 45 available options (#233)

**Meeting & Communication Updates**
- Teams meetings now created for all appointment types, not just telehealth (#249, #76) - enables recording of in-person sessions
- Fixed client portal magic link URLs for onboarding emails (#80)
- Corrected child primary contact email handling for minor clients (#81)

**Platform Maintenance**
- Applied dark mode styling across all components (#249)
- Fixed date of birth UTC conversion issues (#254)
- Resolved various compilation warnings (#252)

## What's In Flight

- **Copy fix and appointment numbering** (#265) - Ruk fixing "Therapy Genie" → "Therapist Genie" branding and appointment numbering consistency, ready for review
- **Backend dev merge** (#64) - Rodrigo merging 9-day-old dev branch changes, pending review  
- **Client file upload API** (#61) - Matt's 10-day-old Azure Blob Storage integration for document uploads, awaiting review

## What's Outstanding

**Recent Bugs Needing Attention**
- Client portal showing owed money for future sessions (#263)
- Payment tab shows "paid" status when no card on file (#262) - assigned to Rodrigo
- Client portal emails not sent to minors (#259) - Rodrigo working on it
- Cannot delete cancelled events from archived accounts (#257) - assigned to Rodrigo
- Payroll showing old payment data (#251) - assigned to Rodrigo

**Older Issues**
- Recurring appointment cancellation fails silently (#210) - 8 days old, with Rodrigo
- Teams widget branding updates (#204) - 10 days old, assigned to Rodrigo

**Backlog Items**
- **Top Priority**: Dataverse backup discovery research (#206) - assigned to Rodrigo
- Transcript feature refactoring (#186) - with Rodrigo
- Portal www redirect setup (#20) - assigned to Matt
- Calendar sync privacy improvements (#74) - with Rodrigo

---

# VitaBoom Weekly Digest
Week of February 2, 2026

## What Changed This Week

**W9/Commission Management Overhaul**
- Shipped the W9 upload popup that appears on practitioner login when no W9 is on file (#122, #109). Practitioners can either upload their W9 or opt out of commissions entirely with double confirmation. The popup persists until resolved with a 24-hour "remind me later" option.
- Enhanced commission filters with shift+select functionality for bulk operations (#121, #107)

**QuickBooks Integration Update** 
- Modified clinic payment flow to only create bills in QuickBooks without automatic bill payments, per Justin's request (#107)

**Code Cleanup**
- Removed unused code from both frontend and backend repositories (#110, #96)

## What's In Flight

**Admin Experience Improvements** - Matt is working on two major admin features:
- **CSV export for all commissions** (#124) - Ready for review after 1 day
- **Redesigned clinics table with archive/restore** (#123, #110) - Frontend ready for review, backend needs review. Consolidates the table from 9 to 5 columns and adds toggle for viewing archived clinics

**Commission Visibility Controls** - Ruk has been working on letting clinic admins hide commission info from users (#117, #104) for 4 days. Both frontend and backend components are pending review.

**SuppCo Mapping Fix** - Matt's strict vendor+title matching improvement (#82) has changes requested after 28 days. This prevents incorrect cross-vendor product mappings.

**Stale Draft** - Old project setup PR (#9) has been sitting as a draft for 126 days

## What's Outstanding

**Bugs**
- Sequelize connection timeout errors (#50) - Assigned to SDK, 84 days old. This is a DevOps/database performance issue that needs attention.

**Features in Backlog**
- Add Supp.co ratings for new Vitaboom products (#116) - 7 days old, unassigned
- EHR options expansion (#114) - Need to add "I do not use an EHR" and "I am switching EHRs" options, 8 days old
- Bulk protocol copying between clinics (#51) - Assigned to Matt, 84 days old

**Other Items**
- Export all orders functionality (#119) - Assigned to Austin, 3 days old
- Optimize clients page performance (#105) - Backend calls are slow, 14 days old, unassigned
- User role promotion/demotion (#70) - Assigned to SDK, 79 days old
- Logging improvements needed: webhook logging optimization (#108) and password redaction (#105)

---

# Next Health Weekly Digest
Week of February 2, 2026

## What Changed This Week
No code was merged this week - looks like the team was focused on planning and issue triage.

## What's In Flight
No active pull requests at the moment. The development pipeline is clear and ready for new work to begin.

## What's Outstanding

**Bugs that need attention:**
- Email editing during signup is broken (#212) - assigned to Austin, 17 days old
- Upload symbols failing in build process (#168) - with Austin for 55 days
- Need to capture and organize bugs from the testing tracker (#101) - oldest item at 78 days, still with Austin

**Features in the backlog:**
- Payment flow improvement: prompt users to add new card when current one fails (#209) - Austin assigned, 31 days old
- Rebooking functionality (#192) - unassigned, sitting for 50 days
- Facility audit enhancement: add state/city data (#173) - Austin has it, 51 days old

**Other items (19 total):**
Recent priorities include:
- Pricing display fixes - don't show crossed-out prices when they match discounted price (#216) - just 2 days old
- Member benefit: unlimited cryo access (#215) - 3 days old
- Remove hardcoded $69 from soft open packages (#210) - 30 days with Austin

Longer-term items include UI improvements like collapsible state lists in facilities (#189) and navigation button fixes for founding membership purchase flow (#191).

Most items are currently assigned to Austin - might be worth discussing workload distribution and prioritization in your next planning session.

---


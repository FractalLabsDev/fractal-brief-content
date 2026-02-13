# Serhii Yermolenko - Work Summary
## January 26 - February 13, 2026

---

## CATEGORY 1: AWS Migration (Sentinel & Warden)

### February 9 (Portfolio Planning)
**Status: 70% complete**, expected completion within 1 day
- Planning to create issue for Ruk to handle final migration steps
- Committed to recording Loom video explaining migration process
- Does not need external help to complete
- Discussed with Max: copying production data locally for safety

> "Serhii planned to create an issue for Ruk to migrate Warden and Sentinel to AWS, noting this migration is 70% complete and they expect to finish it the next day."

### February 13 (Slack)
- Noted agent-factory (GH PR reviews) may be broken due to migration
> "agent-factory spawning cloud agents via Agent SDK might be broken right now due to heroku=>aws migration, will fix in a bit"

**Status Summary:** Migration likely complete or nearly complete. Agent-factory fix pending.

---

## CATEGORY 2: Vitaboom Dropshipping

### January 28 (Vitaboom Dev Sync)
- Assigned as primary driver (with Scott) for payment gateway implementation
- Tasked with setting up basic architecture as separate project
- Will spec out delivery plan after Matt creates epic

> "Serhii suggested setting up a basic architecture for the payment gateway as a separate project and pushing it to Git."

### February 4 (Vitaboom Dev Sync)
- Payment gateway work postponed (roadmap pivot)
- Assigned to investigate Shopify checkout page customization limitations
- Personal update: moving to temporary apartment, then long-term rental (1,200-1,500 moving cost)

### February 6 (Slack)
- Received dropshipping requirements brief from Matt
- Asked clarifying questions:
  - Does $150 minimum apply to dropship orders?
  - Should clinic receive copy of shipping emails?

### February 10 (Slack)
- **Created PR #112** for dropshipping API changes
> "I've done some API changes for it, let's review it and try connecting the FE on feature env or stage"

### February 11 (Slack)
- Addressed Scott's review comments on PR #112
- **Flagged documentation inconsistency:** Brief said $150 min, Matt said $80, code showed $50
- Proposed structured business knowledge documentation

> "I'm super confused. Brief says the minimum amount is 150, you mentioned here it's 80 and in the code I see 50... I can really see the value in structuring business side project knowledge somewhere"

- Had to skip sync due to personal matter

### February 12 (Slack)
- Mostly offline due to personal matters
- Scott and Matt debugged discount code issues
> "sorry guys for not testing the thing properly, but had an urgent matter to attend. Thank you for taking it from there"

### February 13 (Slack) - Highly Active
- Reviewed PRs #115 (backend) and #133 (web) for dropshipping commissions
- Raised business logic question about 40% discount scope:

> "From the code flow I assume 40% discount will be applied to all one-time purchases... I'm just wondering whether it makes better sense to hardcode to only have 40% for the one clinic they wanted instead of all one-time purchases?"

- Pushed changes to stage, then **deployed to production**
> "pushing to live" / "deployed, @marijoy you can proceed with testing on live"

- **Created PR #116** for additional backend changes
- **Created PR #117** (tiny) + added missing FRONTEND_URL env variable to Heroku
- **Created PR #118** adding pino logger to vitaboom-backend (addresses issue #108)

---

## CATEGORY 3: Other

### February 4 (Personal)
- Moving to temporary apartment, then long-term rental
- Moving cost: 1,200-1,500 for belongings (two bikes, PC, monitors)

### February 9 (Portfolio Planning)
- Asked about status of AWS partnership and certification
- Agreed with Max about data migration risks and need for audit processes
- Discussed importance of AWS certifications for scaling/larger contracts

> "Serhii asked about the status of pursuing AWS partnership and certification, given the migration to AWS."

### February 13 (Process Improvement)
- Suggested improvements to brief system for knowledge sharing
> "considering we're having an in-house brief system we could sync it up to provide data in a more humanly digestible way than a raw .md file"
> "as a human I still tend to verify what the robot does... I'm trying to think of a way to catch such things"

---

## Summary

| Category | Status | Key Deliverables |
|----------|--------|------------------|
| **AWS Migration** | ~70% → nearly complete | Sentinel/Warden migrated, agent-factory fix pending |
| **Vitaboom Dropshipping** | Active development | PRs #112, #116, #117, #118; Production deploy Feb 13 |
| **Other** | Ongoing | AWS cert discussions, process improvements, personal relocation |

### Blockers Encountered
1. Personal matters (Feb 11-12) - resolved
2. Documentation inconsistencies (minimum order values) - flagged
3. Agent-factory broken during migration - fix in progress

### Key Observations
- Highly productive when available, contributed 4 PRs in one day (Feb 13)
- Proactive about process improvements and documentation quality
- Good at flagging business logic questions before shipping
- Balancing personal relocation with work commitments

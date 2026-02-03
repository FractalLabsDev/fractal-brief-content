# Therapist Genie Issue Prioritization

## 🔴 CRITICAL (Immediate attention - data loss, payment, or core workflow blockers)

| # | Issue | Reason |
|---|-------|--------|
| [#242](https://github.com/FractalLabsDev/therapygenie/issues/242) | Progress notes freeze app while typing | **Data loss risk** - clinicians losing work, multiple affected |
| [#184](https://github.com/FractalLabsDev/therapygenie/issues/184) | Payments not processing at midnight | **Revenue impact** - payments failing silently |
| [#185](https://github.com/FractalLabsDev/therapygenie/issues/185) | Timezones affecting appointment rendering | **Scheduling chaos** - wrong times shown |
| [#210](https://github.com/FractalLabsDev/therapygenie/issues/210) | Recurring appointment cancellation fails silently | **Silent failure** - users think canceled but it's not |
| [#221](https://github.com/FractalLabsDev/therapygenie/issues/221) | Expand ICD-10 Diagnosis Codes | **Compliance blocker** - clinicians cannot bill properly |

---

## 🟠 HIGH (Significant workflow impact)

| # | Issue | Reason |
|---|-------|--------|
| [#244](https://github.com/FractalLabsDev/therapygenie/issues/244) | Clinicians cannot assign themselves to clients | Core workflow broken |
| [#237](https://github.com/FractalLabsDev/therapygenie/issues/237) | OOO events not syncing | Scheduling conflicts |
| [#238](https://github.com/FractalLabsDev/therapygenie/issues/238) | All-day events not blocking availability | Related to OOO sync |
| [#245](https://github.com/FractalLabsDev/therapygenie/issues/245) | Payment method reassigns clinician | Data integrity issue |
| [#236](https://github.com/FractalLabsDev/therapygenie/issues/236) | Join Meeting button broken for in-person | Confusing UX |
| [#226](https://github.com/FractalLabsDev/therapygenie/issues/226) | Timezone mismatch between Genie and Teams | Meeting confusion |
| [#204](https://github.com/FractalLabsDev/therapygenie/issues/204) | In-meeting Teams widget broken | Live session tool unusable |
| [#206](https://github.com/FractalLabsDev/therapygenie/issues/206) | Dataverse backup discovery | **Risk mitigation** - need to understand backup strategy |

---

## 🟡 MEDIUM (Quality of life / moderate impact)

| # | Issue | Reason |
|---|-------|--------|
| [#246](https://github.com/FractalLabsDev/therapygenie/issues/246) | Cannot edit cancelled sessions | Billing flexibility |
| [#243](https://github.com/FractalLabsDev/therapygenie/issues/243) | Dark mode text invisible | Accessibility issue |
| [#241](https://github.com/FractalLabsDev/therapygenie/issues/241) | License field missing for some users | Role visibility |
| [#231](https://github.com/FractalLabsDev/therapygenie/issues/231) | Calendar header glitch | Visual bug |
| [#227](https://github.com/FractalLabsDev/therapygenie/issues/227) | Minor client multi-contact forms | Workflow gap |
| [#218](https://github.com/FractalLabsDev/therapygenie/issues/218) | Clients table sorting broken | UX issue |
| [#212](https://github.com/FractalLabsDev/therapygenie/issues/212) | Cannot unassign forms | Workflow gap |
| [#160](https://github.com/FractalLabsDev/therapygenie/issues/160) | Stripe retry failed payments | Payment reliability |

---

## 🟢 LOW / ENHANCEMENTS

| # | Issue | Type |
|---|-------|------|
| [#247](https://github.com/FractalLabsDev/therapygenie/issues/247) | Expanded cancellation statuses | Feature |
| [#248](https://github.com/FractalLabsDev/therapygenie/issues/248) | Unsend/resubmit forms | Feature |
| [#240](https://github.com/FractalLabsDev/therapygenie/issues/240) | Treatment plan signatures | Feature |
| [#239](https://github.com/FractalLabsDev/therapygenie/issues/239) | Calendar refresh indicator | UX |
| [#235](https://github.com/FractalLabsDev/therapygenie/issues/235) | Per-client autopay toggle | Feature |
| [#229](https://github.com/FractalLabsDev/therapygenie/issues/229) | Save diagnosis without tx plan | Enhancement |
| [#228](https://github.com/FractalLabsDev/therapygenie/issues/228) | Extend tx plan review to 365 days | Enhancement |
| [#222](https://github.com/FractalLabsDev/therapygenie/issues/222) | Multiple diagnoses per client | Enhancement |
| [#217](https://github.com/FractalLabsDev/therapygenie/issues/217) | Portal email cleanup | Polish |
| [#203](https://github.com/FractalLabsDev/therapygenie/issues/203) | Remove V2 from branding | Polish |
| [#186](https://github.com/FractalLabsDev/therapygenie/issues/186), [#187](https://github.com/FractalLabsDev/therapygenie/issues/187) | Refactoring / optimization | Tech debt |
| [#147](https://github.com/FractalLabsDev/therapygenie/issues/147) | Auto-archive removed Entra users | Maintenance |
| [#164](https://github.com/FractalLabsDev/therapygenie/issues/164) | Refund option on paid appt deletion | Enhancement |
| [#110](https://github.com/FractalLabsDev/therapygenie/issues/110) | Migrate Marijoy contacts data | Data migration |

---

## Summary

| Priority | Count |
|----------|-------|
| 🔴 Critical | 5 |
| 🟠 High | 8 |
| 🟡 Medium | 8 |
| 🟢 Low/Enhancement | 14 |
| **Total Open** | **35** |

---

## Top 5 to Tackle First

1. **#242** - Notes freeze (data loss risk)
2. **#184** - Midnight payments (revenue impact)
3. **#185** - Timezone rendering (scheduling reliability)
4. **#210** - Recurring cancel fails silently (trust issue)
5. **#244** - Clinician self-assignment (daily workflow blocker)

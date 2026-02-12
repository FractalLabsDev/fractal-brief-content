Investigated the cron job reliability issue. Here's my analysis:

**Root Cause Confirmed**: Rigo is correct — the current `node-cron` implementation runs in-process within the Express app. When Azure App Service goes idle (no traffic for ~20 minutes) or recycles the instance, the scheduler stops. This is a known limitation of Azure's managed web hosting.

**The 17 Pending Appointments**: The CSV shows appointments where `paidat` was set but payment status remained 'Pending'. This suggests either:
- The payment job started but crashed mid-execution
- Or there's a code path setting `paidat` before payment processing completes

Total value at risk: **$3,535**

---

**Recommended Path Forward:**

**Azure Functions with Timer Triggers** (my recommendation)
- Runs independently of the web app — Azure guarantees execution
- Survives restarts, scaling events, and idle periods
- Nearly free on Consumption plan (~$0 for this volume)
- Can share code with existing codebase via npm package or monorepo

**Alternatives**:
- **WebJobs**: Simpler but still tied to App Service uptime
- **Logic Apps**: External scheduler calling HTTP endpoint — adds dependency

---

**Immediate Action**:
Before implementing the architecture fix, should verify these 17 payments in Stripe:
1. Do payment intents/invoices exist for these appointment IDs?
2. If succeeded → update Dataverse status to 'Paid'
3. If failed/missing → clear `paidat` so next run picks them up

Would you like me to create a GitHub issue to scope the Azure Functions migration?

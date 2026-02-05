# Practice Interviews: Enterprise Licensing Model

**Epic:** E-15 | **Target:** February 28, 2026

---

## Core Concept

Enterprise clients have a **license pool** with a seat limit and expiration date. Existing "add user" flow works as-is, but blocks when capacity is reached. Super admin manages limits via the clients page.

---

## Data Model

### Changes to Existing Tables

```sql
-- Add to existing clients table (or create enterprise_settings)
ALTER TABLE clients ADD COLUMN license_limit INT DEFAULT NULL;      -- NULL = unlimited
ALTER TABLE clients ADD COLUMN license_expires_at TIMESTAMP DEFAULT NULL;  -- NULL = never
```

### New Table (if needed for audit)

```sql
-- Optional: track license assignments over time
CREATE TABLE license_audit_log (
  id UUID PRIMARY KEY,
  client_id UUID REFERENCES clients(id),
  user_id UUID REFERENCES users(id),
  action VARCHAR(20),  -- 'assigned' | 'removed'
  performed_by UUID REFERENCES users(id),
  created_at TIMESTAMP DEFAULT NOW()
);
```

---

## Behavior

### Adding Users (Existing Flow)
```javascript
async function addUserToClient(clientId, userId, performedBy) {
  const client = await getClient(clientId);

  // Check license limit if set
  if (client.license_limit !== null) {
    const currentCount = await countUsersInClient(clientId);
    if (currentCount >= client.license_limit) {
      throw new Error(`License limit reached (${client.license_limit} seats)`);
    }
  }

  // Check expiration if set
  if (client.license_expires_at && client.license_expires_at < new Date()) {
    throw new Error('License has expired');
  }

  // Proceed with existing add user logic
  await assignUserToClient(clientId, userId);
}
```

### Feature Access Check
```javascript
function hasAccess(user) {
  // Existing paid subscription check
  if (user.subscription?.active) return true;

  // Enterprise license check
  const client = await getUserClient(user.id);
  if (client) {
    // Check if license is still valid
    if (!client.license_expires_at || client.license_expires_at > new Date()) {
      return true;
    }
  }

  return false;
}
```

---

## Implementation Tasks

### Phase 1: Database & Backend (Week 1)

- [ ] Migration: Add `license_limit` and `license_expires_at` to clients table
- [ ] Update add-user endpoint to check capacity before adding
- [ ] Update feature access check to respect expiration
- [ ] API: `PATCH /admin/clients/:id` to set limit and expiration (super admin)
- [ ] Return license info in client GET responses

### Phase 2: UI - Clients Page (Week 2)

- [ ] Add to client three-dots menu: "Manage License"
- [ ] License management modal:
  - Set/update seat limit
  - Set/update expiration date
  - Show current usage (X/Y seats used)
- [ ] Show license status on client card/row (e.g., "42/50 seats • Expires Feb 5, 2027")
- [ ] Error message when admin tries to add user at capacity

### Phase 3: Polish (Week 3)

- [ ] Block add-user UI when at capacity (disable button, show message)
- [ ] Visual indicator for expired licenses
- [ ] Handle edge case: what if limit is reduced below current usage?
- [ ] Tests

---

## UI Mockup

**Client Card:**
```
┌─────────────────────────────────────────┐
│ B Corp                              ⋮   │
│ 42/50 licenses • Expires Feb 5, 2027    │
│ 12 active users                         │
└─────────────────────────────────────────┘
```

**Three-dots menu:**
```
┌──────────────────┐
│ Edit Client      │
│ Manage License   │  ← New option
│ View Users       │
│ Delete Client    │
└──────────────────┘
```

**Manage License Modal:**
```
┌─────────────────────────────────────────┐
│ Manage License - B Corp            [X]  │
├─────────────────────────────────────────┤
│ License Limit:  [50_______]             │
│ Expires:        [2027-02-05]            │
│                                         │
│ Current Usage: 42/50 seats              │
│                                         │
│           [Cancel]  [Save]              │
└─────────────────────────────────────────┘
```

---

## Edge Cases

| Scenario | Behavior |
|----------|----------|
| Limit set below current users | Allow (grandfather existing), block new adds |
| License expires | Immediate cutoff - users lose access |
| Limit set to NULL | Unlimited (default behavior) |
| Expiration set to NULL | Never expires |
| User removed from client | Frees up a seat |

---

## Out of Scope

- Notifications (expiration warnings, capacity alerts)
- Self-service enterprise signup
- Per-user expiration (all users share client expiration)
- Usage analytics beyond seat count

---

*Final Plan: February 5, 2026*

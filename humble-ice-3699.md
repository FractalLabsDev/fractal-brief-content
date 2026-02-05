# Practice Interviews: Enterprise Licensing Model

**Epic:** E-15 | **Target:** February 28, 2026

---

## Core Concept

Enterprise clients purchase a **license pool** (e.g., 50 seats) with an **expiration date**. They can assign users up to their limit. Admins can deprovision users to free up seats for others.

---

## Data Model

### New Tables

```sql
-- Enterprise clients
CREATE TABLE enterprise_clients (
  id UUID PRIMARY KEY,
  name VARCHAR(255) NOT NULL,           -- "B Corp"
  license_limit INT NOT NULL,           -- 50
  licenses_used INT DEFAULT 0,          -- Denormalized count
  expires_at TIMESTAMP NOT NULL,        -- 2027-02-05
  created_at TIMESTAMP DEFAULT NOW(),
  created_by UUID REFERENCES users(id)  -- Super admin who created
);

-- Users assigned to enterprise clients
CREATE TABLE enterprise_licenses (
  id UUID PRIMARY KEY,
  enterprise_client_id UUID REFERENCES enterprise_clients(id),
  user_id UUID REFERENCES users(id),
  assigned_at TIMESTAMP DEFAULT NOW(),
  assigned_by UUID REFERENCES users(id), -- Admin who assigned
  UNIQUE(enterprise_client_id, user_id)
);

-- Enterprise admins (can provision/deprovision within their org)
CREATE TABLE enterprise_admins (
  id UUID PRIMARY KEY,
  enterprise_client_id UUID REFERENCES enterprise_clients(id),
  user_id UUID REFERENCES users(id),
  UNIQUE(enterprise_client_id, user_id)
);
```

### Role Hierarchy

| Role | Can Do |
|------|--------|
| **Super Admin** (Fractal) | Create enterprise clients, set license limits, set expiration, add enterprise admins |
| **Enterprise Admin** | Provision users (up to limit), deprovision users, view their org's users |
| **Enterprise User** | Use the platform (full Pro access) until license expires |

---

## Feature Access Logic

```javascript
function hasAccess(user) {
  // Check if user has individual paid subscription
  if (user.subscription?.active) return true;

  // Check if user has enterprise license
  const license = await getEnterpriseLicense(user.id);
  if (license && license.enterprise_client.expires_at > now()) {
    return true;
  }

  return false; // Free trial / no access
}
```

---

## Implementation Tasks

### Phase 1: Database & Backend (Week 1)

- [ ] Create migration for `enterprise_clients` table
- [ ] Create migration for `enterprise_licenses` table
- [ ] Create migration for `enterprise_admins` table
- [ ] Add enterprise license check to feature access logic
- [ ] API: `POST /admin/enterprise-clients` (super admin only)
- [ ] API: `GET /admin/enterprise-clients` (super admin only)
- [ ] API: `PATCH /admin/enterprise-clients/:id` (update limit/expiration)

### Phase 2: Admin Provisioning (Week 2)

- [ ] API: `POST /enterprise/:clientId/users` (assign user, check limit)
- [ ] API: `DELETE /enterprise/:clientId/users/:userId` (deprovision)
- [ ] API: `GET /enterprise/:clientId/users` (list assigned users)
- [ ] API: `GET /enterprise/:clientId` (view license usage)
- [ ] Enforce `licenses_used < license_limit` on assignment
- [ ] Decrement `licenses_used` on deprovision

### Phase 3: UI (Week 3)

- [ ] Super Admin: Enterprise client management page
- [ ] Super Admin: Create new enterprise client form
- [ ] Enterprise Admin: User management dashboard
- [ ] Enterprise Admin: Invite/assign user flow
- [ ] Enterprise Admin: Deprovision user flow
- [ ] License usage indicator (e.g., "42/50 seats used")

### Phase 4: Polish (Week 4)

- [ ] Expiration handling (what happens when licenses expire?)
- [ ] Email notifications (approaching expiration, limit reached)
- [ ] Audit log for provisioning actions
- [ ] Tests

---

## Open Questions

1. **User invite flow** - Do enterprise admins invite by email (creates account) or assign existing users?
2. **Expiration behavior** - When licenses expire, do users lose access immediately or get a grace period?
3. **Super admin UI** - Build in PI admin panel, or separate internal tool?

---

## Out of Scope (for now)

- Stripe integration for enterprise billing (manual/invoice-based)
- SSO/SAML
- Per-seat usage analytics
- Self-service enterprise signup

---

*Revised: February 5, 2026*

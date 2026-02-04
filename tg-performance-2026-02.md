# Therapist Genie Performance Analysis

## Executive Summary

After reviewing all three TG codebases (main app, backend, and portal), I've identified the root causes of slow load times and busy console logs. The issues fall into three categories:

1. **Data Loading Architecture** - Sequential/waterfall API calls instead of parallel
2. **React Re-render Cascades** - Un-memoized context values causing app-wide re-renders
3. **Excessive Logging** - 150+ console statements across codebases

---

## Critical Issues (Address First)

### 1. N+1 Query Pattern in Backend

**Location:** `therapygenie-backend/src/services/internal/payment.service.ts:30-85`

For each appointment, a separate Dataverse API call fetches payment info. A client with 50 appointments triggers 50 API calls.

```typescript
// Current: Sequential calls
const paymentPromises = appointments.map(appointment => {
  return getPaymentByAppointmentId(appointment.appointmentid);
});
```

**Impact:** 2-5 second delays on appointment views

---

### 2. Microsoft Graph API Pagination Blocks Everything

**Location:** `therapygenie-backend/src/services/external/graphApi.service.ts:150-231`

`getUsers()` fetches ALL Microsoft users sequentially with up to 50 pagination calls. Large tenants (1000+ users) take 2-5 minutes.

**Impact:** Initial app load blocked until all users fetched

---

### 3. Context Provider Values Not Memoized

**Location:** Multiple files in `therapygenie/src/contexts/`

The main app has 8+ context providers (ClientsContext, CalendarContext, ChartContext, etc.) where the `value` prop is a large object with 20-40 properties created inline on every render.

```typescript
// Current: New object on every render = all consumers re-render
<ClientContext.Provider value={{
  clients, setClients, selectedClient, me, // ...37 more properties
}}>
```

**Impact:** Any state change cascades to re-render the entire app

---

### 4. Redundant User Fetches on Every Request

**Location:** `therapygenie-backend/src/controllers/client/client.appointment.controller.ts:68-125`

Every appointment list request fetches ALL Microsoft users (2-5 seconds) even though users rarely change.

```typescript
const [appointments, users] = await Promise.all([
  getAppointments(clientId, options),
  getUsers(), // Fetches ALL users every time
]);
```

---

## High Impact Issues

### 5. Console Logging Everywhere

**Counts:**
- Main app: 73 files with console.log
- Backend: 40+ console statements in hot paths
- Portal: 50+ debug statements

**Specific hot paths:**
- `therapygenie/src/services/api.service.ts` - logs every API call
- `therapygenie-backend/src/services/external/graphApi.service.ts:49-50` - logs every Graph call
- `therapist-genie-portal/src/services/teamsApi.ts` - 40+ statements

---

### 6. Heavy Dependencies Imported Eagerly

- **jsPDF (384KB)** imported at module level in portal
- **recharts** imported even when not viewing charts
- No code splitting for heavy pages (Forms, Appointments, Billing)

---

### 7. Cache Eviction Algorithm is O(n²)

**Location:** `therapygenie-backend/src/services/internal/cache.service.ts:240-260`

When the cache is full, evicting entries requires scanning all 100 entries repeatedly, blocking the server.

---

### 8. Nested Provider Architecture Creates Waterfall

**Location:** `therapygenie/src/components/clients/view-client/ViewClient.tsx`

`TeamProvider` and `ChartProvider` both fetch data independently in sequence instead of parallel.

---

## Medium Impact Issues

### 9. Missing React.memo on Reusable Components
- Avatar, CustomButton, table cells not memoized
- Dashboard cards in portal re-render unnecessarily

### 10. Inline Styles Create New Objects Every Render
- 328+ instances of `styles={{}}` in main app

### 11. Over-Expanded Dataverse Queries
- Every appointment query expands 10+ related records even when not needed

### 12. Query Invalidation Overkill in Portal
- Form submission invalidates 3 query keys, triggering 3 API calls

---

## Sequenced Refactoring Plan

### Phase 1: Quick Wins (1-2 days)

**Goal:** Reduce console noise and add caching

1. **Disable console logging in production**
   - Add environment check to logger utility
   - Remove/conditionalize debug statements
   - Files: `therapygenie/src/utils/logger.ts`, services throughout

2. **Cache Microsoft users for 10 minutes**
   - Use existing `cacheService` in backend
   - File: `therapygenie-backend/src/controllers/client/client.appointment.controller.ts`
   ```typescript
   const users = await cacheService.getOrLoad('all_users', () => getUsers(), 10 * 60 * 1000);
   ```

3. **Add timeout to fire-and-forget async tasks**
   - File: `therapygenie-backend/src/services/internal/api/api.appointment.meeting.service.ts`

**Expected improvement:** 30-40% reduction in initial load time

---

### Phase 2: Context Optimization (3-5 days)

**Goal:** Stop cascading re-renders

1. **Wrap all context provider values in `useMemo`**
   - Files: All files in `therapygenie/src/contexts/`
   ```typescript
   const value = useMemo(() => ({
     clients, selectedClient, me, // ...
   }), [clients, selectedClient, me]);
   ```

2. **Split large contexts into smaller ones**
   - ClientsContext → ClientListContext + SelectedClientContext
   - CalendarContext → CalendarDataContext + CalendarUIContext

3. **Add React.memo to reusable components**
   - Avatar, CustomButton, table cells

**Expected improvement:** 50%+ reduction in re-renders

---

### Phase 3: Data Loading Architecture (1 week)

**Goal:** Parallel instead of waterfall requests

1. **Fix N+1 query pattern**
   - Batch payment lookups into single OData query
   - File: `therapygenie-backend/src/services/internal/payment.service.ts`

2. **Parallelize initial data fetches**
   - Load calendar, team, and user data concurrently
   - File: `therapygenie/src/contexts/CalendarContext.tsx`

3. **Add pagination to appointment endpoints**
   - Return max 100 records with `hasMore` flag
   - File: `therapygenie-backend/src/services/internal/api/api.appointment.service.ts`

4. **Fix Graph API pagination to fetch concurrently**
   - Fetch pages 2-5 while processing page 1

**Expected improvement:** 60-70% reduction in API latency

---

### Phase 4: Bundle Optimization (3-5 days)

**Goal:** Faster initial page load

1. **Lazy load heavy pages in portal**
   - Forms, Appointments, Billing, Profile
   - File: `therapist-genie-portal/src/App.tsx`

2. **Dynamic import jsPDF**
   - Only load when user clicks "Download PDF"

3. **Code split recharts**
   - Only load when viewing analytics

**Expected improvement:** 40% reduction in initial bundle size

---

### Phase 5: Backend Infrastructure (1 week)

**Goal:** Sustainable performance

1. **Replace O(n²) cache eviction with proper LRU**
   - Use doubly-linked list or drop oldest entries

2. **Whitelist CORS origins**
   - Remove `origin: "*"`, add specific domains

3. **Add Dataverse query caching with ETags**

4. **Parallelize background job processing**
   - Process payments in batches of 5 concurrent

---

## Summary: Priority Order

| Phase | Effort | Impact | Description |
|-------|--------|--------|-------------|
| 1 | 1-2 days | High | Logging + user caching |
| 2 | 3-5 days | Very High | Context memoization |
| 3 | 1 week | Very High | Data loading architecture |
| 4 | 3-5 days | Medium | Bundle optimization |
| 5 | 1 week | Medium | Backend infrastructure |

**Total estimated effort:** 3-4 weeks

**Expected outcome:** App load time reduced from 10-15s to 2-3s

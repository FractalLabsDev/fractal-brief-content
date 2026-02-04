# Calendar Codebase: Current vs Refactored

## Current Structure (Scattered across 80+ files)

```
src/src/
├── components/
│   ├── Calendar.tsx                          # Entry point (20 lines)
│   ├── calendar/
│   │   ├── ViewCalendar.tsx                  # Main view (422 lines) ⚠️
│   │   └── upsert-appointment/
│   │       ├── UpsertAppointmentModal.tsx    # Create/edit form
│   │       ├── Footer.tsx
│   │       └── form-fields/
│   │           ├── ClientDropdown.tsx
│   │           ├── ClinicianDropdown.tsx
│   │           ├── DateTimeSection.tsx
│   │           ├── LocationDropdown.tsx
│   │           └── ServiceSection.tsx
│   ├── appointment-detail/                   # Viewing appointments
│   │   ├── AppointmentDetail.tsx
│   │   ├── AppointmentDetailHeader.tsx
│   │   ├── AppointmentDetailModal.tsx
│   │   ├── ChartTimeline.tsx
│   │   └── NotesEditorPanel.tsx
│   ├── reusable/Calendar/                    # Shared calendar components
│   │   ├── CalendarAgenda.tsx
│   │   ├── CalendarDayHeader.tsx
│   │   ├── CalendarEventRenderer.tsx
│   │   ├── CalendarHeader.tsx
│   │   └── CustomCalendar.tsx
│   └── clients/view-client/
│       └── Appointments.tsx                  # Client's appointments tab
│
├── contexts/
│   └── CalendarContext.tsx                   # Calendar state (334 lines) ⚠️
│       # Also imports from: ClientsContext, ChartContext, BillingContext
│
├── hooks/                                    # Calendar hooks scattered here
│   ├── useAppointmentForm.ts
│   ├── useAppointmentMutations.ts
│   ├── useCalendarEvents.ts
│   ├── useCalendarInit.ts
│   ├── useCalendarNavigation.ts
│   ├── useCalendarOverlays.ts
│   ├── useFetchAppointments.ts
│   └── useMeetingTranscript.ts
│
├── services/                                 # API calls scattered
│   ├── appointment.service.ts                # Our backend
│   ├── calendar.service.ts                   # Calendar operations
│   ├── meeting.service.ts                    # Teams meetings
│   ├── teamsMeeting.service.ts               # More Teams
│   └── graph/calendar.service.ts             # Microsoft Graph
│
├── utils/
│   ├── appointment.ts                        # Appointment helpers
│   ├── appointmentCache.ts                   # Caching (228 lines)
│   └── date.ts                               # Date utilities
│
├── interface/
│   └── Appointment.ts                        # Types (70 lines)
│
└── types/
    └── calendar.types.ts                     # More types
```

### Problems with Current Structure

1. **Calendar logic in 6+ contexts** - CalendarContext depends on ClientsContext, ChartContext, etc.
2. **80+ files touch appointments** - No clear boundary for the feature
3. **Two type locations** - `interface/Appointment.ts` AND `types/calendar.types.ts`
4. **Services scattered** - 5 different service files for calendar operations
5. **Hooks have no home** - Mixed with other feature hooks in `/hooks/`

---

## Proposed Refactored Structure

```
src/
├── domains/                                  # Pure business logic (no React)
│   └── calendar/
│       ├── index.ts                          # Public API
│       ├── types.ts                          # All calendar/appointment types
│       ├── appointment.entity.ts             # Appointment model & validation
│       ├── calendar.entity.ts                # Calendar event model
│       ├── recurrence.logic.ts               # Recurring appointment rules
│       ├── availability.logic.ts             # Free/busy calculations
│       ├── cancellation.logic.ts             # Cancellation rules & fees
│       └── __tests__/                        # Pure unit tests
│           ├── appointment.test.ts
│           └── recurrence.test.ts
│
├── features/                                 # Self-contained UI features
│   └── calendar/
│       ├── index.ts                          # Feature entry point
│       │
│       ├── api/                              # All API calls for this feature
│       │   ├── appointments.api.ts           # CRUD operations
│       │   ├── calendar-sync.api.ts          # Microsoft Graph sync
│       │   └── meetings.api.ts               # Teams meeting creation
│       │
│       ├── hooks/                            # Feature-specific hooks
│       │   ├── useCalendar.ts                # Main calendar state
│       │   ├── useAppointments.ts            # Appointment queries
│       │   ├── useAppointmentForm.ts         # Form state
│       │   └── useCalendarNavigation.ts      # Date navigation
│       │
│       ├── components/                       # UI components
│       │   ├── CalendarView/
│       │   │   ├── CalendarView.tsx          # Main calendar grid
│       │   │   ├── DayHeader.tsx
│       │   │   ├── EventRenderer.tsx
│       │   │   └── AgendaView.tsx
│       │   │
│       │   ├── AppointmentForm/
│       │   │   ├── AppointmentForm.tsx       # Create/edit modal
│       │   │   ├── ClientField.tsx
│       │   │   ├── ClinicianField.tsx
│       │   │   ├── DateTimeFields.tsx
│       │   │   └── ServiceField.tsx
│       │   │
│       │   └── AppointmentDetail/
│       │       ├── AppointmentDetail.tsx     # View appointment
│       │       ├── DetailHeader.tsx
│       │       └── NotesPanel.tsx
│       │
│       ├── context/
│       │   └── CalendarProvider.tsx          # Single, focused context
│       │
│       └── utils/
│           ├── cache.ts                      # Appointment caching
│           └── formatting.ts                 # Display helpers
│
├── shared/                                   # Truly shared utilities
│   ├── components/                           # Generic UI (Button, Modal, etc.)
│   ├── hooks/                                # Generic hooks (useDebounce, etc.)
│   └── utils/                                # Generic utils (date formatting)
│
└── infrastructure/                           # External integrations
    ├── api-client.ts                         # HTTP client
    ├── auth/                                 # Authentication
    └── microsoft-graph/                      # Graph API utilities
```

---

## Key Differences

| Aspect | Current | Refactored |
|--------|---------|------------|
| **Files touching calendar** | 80+ scattered | ~25 in one folder |
| **Entry point** | Hunt through contexts | `features/calendar/index.ts` |
| **Types** | 2 locations | 1 file: `domains/calendar/types.ts` |
| **Business logic** | Mixed in components | Pure in `domains/calendar/` |
| **API calls** | 5 service files | 3 focused files in `api/` |
| **Testing** | Difficult (UI coupled) | Easy (domain logic is pure) |
| **Context deps** | 6+ contexts tangled | 1 focused CalendarProvider |

---

## Migration Path

### Phase 1: Extract Domain Logic
Move pure business logic (no React/API) to `domains/calendar/`:
- Recurrence calculations
- Cancellation fee rules
- Availability logic
- Type definitions

### Phase 2: Consolidate API Layer
Create `features/calendar/api/` by:
- Merging 5 service files into 3 focused ones
- Clear separation: our backend vs Microsoft Graph

### Phase 3: Migrate Components
Move components incrementally:
- Start with leaf components (EventRenderer, DateTimeFields)
- Work up to container components
- Update imports as we go

### Phase 4: Simplify Context
- Create new focused CalendarProvider
- Gradually migrate consumers
- Delete old CalendarContext when empty

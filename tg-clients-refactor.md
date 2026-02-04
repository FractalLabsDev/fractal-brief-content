# Clients Codebase: Current vs Refactored

## Current Structure (Scattered across 115+ files)

```
src/src/
├── components/
│   ├── Clients.tsx                           # Entry point (34 lines)
│   ├── clients/
│   │   ├── BreadCrumb.tsx                    # Navigation
│   │   ├── ClientManage.tsx                  # Client list/table (389 lines) ⚠️
│   │   ├── create-client/
│   │   │   └── CreateClientModal.tsx         # Create client form
│   │   ├── client-sections/                  # Client detail sections
│   │   │   ├── AssignmentSection.tsx         # Clinician assignment
│   │   │   ├── BasicSection.tsx              # Basic info
│   │   │   ├── ContactDetailsSection.tsx     # Contact info
│   │   │   ├── ContactSection.tsx
│   │   │   ├── FormSection.tsx               # Form assignments
│   │   │   ├── GuardianContactSection.tsx    # Guardian info
│   │   │   ├── NameAndDOBSection.tsx
│   │   │   ├── PaymentMethodSection.tsx      # Stripe payment
│   │   │   ├── PaymentSection.tsx
│   │   │   └── StripePaymentForm.tsx         # Stripe integration
│   │   └── view-client/
│   │       ├── ViewClient.tsx                # Main client view (94 lines)
│   │       ├── ClientDetails.tsx             # Details tab
│   │       ├── Appointments.tsx              # Appointments tab
│   │       ├── Chart.tsx                     # Clinical chart
│   │       ├── ChartItem.tsx
│   │       ├── Members.tsx                   # Family/couple members
│   │       ├── Files.tsx                     # File management
│   │       ├── ClientFiles.tsx
│   │       ├── client-files/
│   │       │   └── FileUploadDropzone.tsx
│   │       ├── upsert-diagnosis-and-treatment-plan/
│   │       │   ├── TreatmentPlanForm.tsx
│   │       │   └── UpsertDiagnosisAndTreatmentPlan.tsx
│   │       ├── upsert-form/
│   │       │   └── UpsertFormModal.tsx
│   │       ├── upsert-member/
│   │       │   └── AddMemberModal.tsx
│   │       ├── upsert-note/
│   │       │   └── UpsertNote.tsx
│   │       ├── upsert-progress-note/
│   │       │   └── UpsertProgressNote.tsx
│   │       └── view-form/
│   │           ├── FieldRenderer.tsx
│   │           └── FormDetailView.tsx
│   │
│   ├── calendar/upsert-appointment/form-fields/
│   │   └── ClientDropdown.tsx                # Client picker (used in calendar)
│   │
│   └── reusable/Table/Columns/
│       └── ClientColumns.tsx                 # Table column definitions
│
├── contexts/
│   ├── ClientsContext.tsx                    # Main state (584 lines) ⚠️ BIGGEST
│   ├── ClientFormContext.tsx                 # Form state
│   └── ChartContext.tsx                      # Chart state (also for clients)
│
├── hooks/                                    # Client hooks scattered here
│   ├── useClientForm.ts                      # Form handling
│   ├── useFetchClients.ts                    # Client list fetching
│   ├── useFetchChartNotes.ts                 # Chart notes
│   ├── useFetchClientFiles.ts                # File fetching
│   ├── useFetchDiagnosisAndTreatmentPlans.ts
│   ├── useFetchForms.ts                      # Form fetching
│   └── useFetchProgressNotes.ts              # Progress notes
│
├── services/                                 # API calls scattered
│   ├── client.service.ts                     # Client CRUD (178 lines)
│   ├── clientFile.service.ts                 # File operations
│   ├── chartNote.service.ts                  # Chart notes
│   ├── diagnosisAndTreatmentPlan.service.ts  # Treatment plans
│   ├── form.service.ts                       # Forms
│   └── progressNote.service.ts               # Progress notes
│
├── interface/                                # Types scattered
│   ├── Client.ts                             # Client type (76 lines)
│   ├── ClientFile.ts
│   ├── ClientTeamMemberLink.ts
│   ├── ChartNote.ts
│   ├── DiagnosisAndTreatmentPlan.tsx
│   ├── Form.ts
│   └── ProgressNote.ts
│
├── styles/Clients/
│   └── TableClientsStyle.ts                  # Client-specific styles
│
└── utils/
    ├── diagnosis.ts                          # Diagnosis helpers
    └── validate.ts                           # Validation (includes client)
```

### Problems with Current Structure

1. **ClientsContext is massive** - 584 lines managing 40+ state variables
2. **115+ files touch clients** - No clear boundary
3. **6 service files** for client-related operations
4. **7 type files** spread across `/interface/`
5. **Mixed responsibilities** - ChartContext handles both charts AND client clinical data
6. **No clear domain logic** - Business rules buried in components

---

## Proposed Refactored Structure

```
src/
├── domains/                                  # Pure business logic (no React)
│   └── clients/
│       ├── index.ts                          # Public API
│       ├── types.ts                          # ALL client-related types
│       │   # Client, ClientFile, ChartNote, Form, ProgressNote,
│       │   # DiagnosisAndTreatmentPlan, ClientTeamMemberLink
│       ├── client.entity.ts                  # Client model & validation
│       ├── family.logic.ts                   # Family/couple membership rules
│       ├── forms.logic.ts                    # Form assignment rules
│       ├── chart.logic.ts                    # Clinical chart rules
│       ├── diagnosis.logic.ts                # Diagnosis/ICD codes
│       └── __tests__/
│           ├── client.test.ts
│           └── family.test.ts
│
├── features/                                 # Self-contained UI features
│   └── clients/
│       ├── index.ts                          # Feature entry point
│       │
│       ├── api/                              # All client API calls
│       │   ├── clients.api.ts                # Client CRUD
│       │   ├── chart.api.ts                  # Chart notes, progress notes
│       │   ├── files.api.ts                  # File upload/download
│       │   ├── forms.api.ts                  # Form assignment/submission
│       │   └── treatment.api.ts              # Diagnosis & treatment plans
│       │
│       ├── hooks/                            # Feature-specific hooks
│       │   ├── useClients.ts                 # Client list
│       │   ├── useClient.ts                  # Single client
│       │   ├── useClientForm.ts              # Create/edit form
│       │   ├── useChart.ts                   # Clinical chart
│       │   └── useClientFiles.ts             # File management
│       │
│       ├── components/                       # UI components
│       │   ├── ClientList/
│       │   │   ├── ClientList.tsx            # Main table view
│       │   │   ├── ClientRow.tsx
│       │   │   └── ClientFilters.tsx
│       │   │
│       │   ├── ClientForm/
│       │   │   ├── CreateClientModal.tsx
│       │   │   ├── BasicInfoFields.tsx
│       │   │   ├── ContactFields.tsx
│       │   │   ├── PaymentFields.tsx
│       │   │   └── AssignmentFields.tsx
│       │   │
│       │   ├── ClientDetail/
│       │   │   ├── ClientDetail.tsx          # Main detail view
│       │   │   ├── DetailsTabs.tsx           # Tab navigation
│       │   │   ├── DetailsTab.tsx            # Basic info tab
│       │   │   ├── AppointmentsTab.tsx       # Appointments list
│       │   │   ├── ChartTab.tsx              # Clinical chart
│       │   │   ├── FilesTab.tsx              # File management
│       │   │   └── MembersTab.tsx            # Family/couple members
│       │   │
│       │   ├── Chart/
│       │   │   ├── ChartView.tsx             # Chart timeline
│       │   │   ├── ChartItem.tsx             # Single chart entry
│       │   │   ├── ProgressNoteForm.tsx      # Progress note editor
│       │   │   └── TreatmentPlanForm.tsx     # Treatment plan editor
│       │   │
│       │   └── shared/                       # Client-specific shared components
│       │       ├── ClientDropdown.tsx        # Client picker
│       │       └── ClientStatusBadge.tsx
│       │
│       ├── context/
│       │   ├── ClientsProvider.tsx           # Client list state (focused)
│       │   └── ClientDetailProvider.tsx      # Single client state (focused)
│       │
│       └── utils/
│           ├── validation.ts                 # Client validation
│           └── formatting.ts                 # Display helpers
│
├── shared/                                   # Truly shared
│   └── components/
│       └── Table/
│           └── columns/                      # Generic column helpers
│
└── infrastructure/
    └── stripe/                               # Stripe integration
        └── payment-form.tsx
```

---

## Key Differences

| Aspect | Current | Refactored |
|--------|---------|------------|
| **Files touching clients** | 115+ scattered | ~40 in one folder |
| **Context size** | 584 lines (monolithic) | 2 focused contexts (~150 each) |
| **Type locations** | 7 files in /interface/ | 1 file: domains/clients/types.ts |
| **Service files** | 6 scattered | 5 focused in api/ folder |
| **Business logic** | Buried in components | Pure in domains/clients/ |
| **Entry point** | Hunt through contexts | features/clients/index.ts |

---

## Migration Path

### Phase 1: Consolidate Types
Move all client-related interfaces to `domains/clients/types.ts`:
- Client, ClientFile, ClientTeamMemberLink
- ChartNote, ProgressNote
- DiagnosisAndTreatmentPlan, Form

### Phase 2: Extract Domain Logic
Create pure business logic in `domains/clients/`:
- Family/couple membership rules
- Form assignment validation
- Diagnosis/ICD code handling

### Phase 3: Consolidate API Layer
Create `features/clients/api/` by merging:
- client.service.ts → clients.api.ts
- chartNote.service.ts + progressNote.service.ts → chart.api.ts
- clientFile.service.ts → files.api.ts
- form.service.ts → forms.api.ts
- diagnosisAndTreatmentPlan.service.ts → treatment.api.ts

### Phase 4: Split ClientsContext
Replace 584-line monolith with:
- ClientsProvider (~150 lines) - List state only
- ClientDetailProvider (~150 lines) - Single client state

### Phase 5: Migrate Components
Move incrementally:
- Start with leaf components (ClientStatusBadge, ClientDropdown)
- Then form components
- Finally container components (ClientList, ClientDetail)

---

## Complexity Notes

The Clients feature is more complex than Calendar because it owns:
- **Clinical documentation** (charts, progress notes, treatment plans)
- **Family/couple relationships** (members, guardians)
- **Form management** (assignment, submission, viewing)
- **File storage** (upload, download, organization)
- **Stripe integration** (payment methods)

This makes the domain logic extraction particularly valuable - rules like "minors don't need email" or "family clients need multiple contacts" should be testable without React.

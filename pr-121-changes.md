# PR #121 Code Changes

Here are the copy-paste ready changes for each file.

---

## 1. CommissionFilters.tsx — zIndex + backgroundColor

### Change 1: Line 423 (backdrop zIndex)
**Find:**
```tsx
                    zIndex: 1299,
```
**Replace with:**
```tsx
                    zIndex: theme => theme.zIndex.modal - 1,
```

### Change 2: Line 439 (dropdown panel zIndex)
**Find:**
```tsx
                    zIndex: 1300,
```
**Replace with:**
```tsx
                    zIndex: theme => theme.zIndex.modal,
```

### Change 3: Line 492 (action buttons background)
**Find:**
```tsx
                      backgroundColor: '#f9fafb',
```
**Replace with:**
```tsx
                      backgroundColor: 'grey.50',
```

### Change 4: Line 574 (footer background)
**Find:**
```tsx
                        backgroundColor: '#f9fafb',
```
**Replace with:**
```tsx
                        backgroundColor: 'grey.50',
```

---

## 2. CommissionsTable.tsx — backgroundColor

### Change 1: Line 302 (table header)
**Find:**
```tsx
              backgroundColor: '#f9fafb',
```
**Replace with:**
```tsx
              backgroundColor: 'grey.50',
```

### Change 2: Line 403 (alternating row colors)
**Find:**
```tsx
            const backgroundColor = index % 2 === 0 ? '#ffffff' : '#f9fafb';
```
**Replace with:**
```tsx
            const backgroundColor = index % 2 === 0 ? 'background.paper' : 'grey.50';
```

---

## 3. MarkPaidModal.tsx — backgroundColor

### Line 53
**Find:**
```tsx
              backgroundColor: '#f9fafb',
```
**Replace with:**
```tsx
              backgroundColor: 'grey.50',
```

---

## 4. PayClinicModal.tsx — backgroundColor

### Line 76
**Find:**
```tsx
              backgroundColor: '#f9fafb',
```
**Replace with:**
```tsx
              backgroundColor: 'grey.50',
```

---

## About the dropdown abstraction

For comment #1 (extracting dropdown to UI lib), Max suggested this as a "nice to have" for future reuse. Since this is the only place using this pattern currently, you could:

1. **Skip it for now** — Reply that you'll extract it if/when another component needs similar functionality
2. **Create a quick abstraction** — But this adds ~50 lines of new code for a pattern used once

I'd recommend option 1: acknowledge the feedback and defer until there's a second use case.

---

## Suggested PR comment reply

> Thanks for the thorough review! I've addressed the feedback:
> - ✅ Replaced hardcoded `#f9fafb` with `grey.50` throughout
> - ✅ Changed zIndex to use `theme.zIndex.modal` instead of magic numbers
> - Re: dropdown abstraction — I'll extract this to a reusable component when we have a second use case. For now, keeping it inline to avoid premature abstraction.

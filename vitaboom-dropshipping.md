# Vitaboom Dropshipping Feature

## Overview

Enable practitioners to place orders on behalf of patients (dropshipping) with products shipped directly to the patient's address. This is an **urgent** requirement from top clients including NextGen Health who operate on a 100% dropshipping model.

**Target Timeline:** MVP by Feb 12-14, 2026

---

## Phase 1: MVP (Immediate Priority)

### 1. Checkout Mode Toggle

A prominent header toggle distinguishing between two checkout modes:

| Mode | Description |
|------|-------------|
| **Dropshipping** | Practitioner places order, ships to patient |
| **Patient Checkout** | Patient completes their own checkout (existing flow) |

**Requirements:**
- Visual differentiation using contrasting colors to clearly indicate selected mode
- Clinic-level default setting (most clinics will use one mode exclusively)
- Toggle should be prominent in the cart/checkout header

### 2. Purchase Type Toggle

Separate toggle for purchase type:

| Type | Description |
|------|-------------|
| **Subscription** | Recurring order (15% discount applied) |
| **One-time Purchase** | Single order at full price |

> **Note:** Dropshipping ≠ one-time purchase. Practitioners CAN dropship subscriptions.

### 3. Commission Handling for Dropshipping

- Commission goes to **zero** for dropship orders
- Maximum discount is **automatically applied**
- No discount slider needed in dropship mode

### 4. Shipping Information Entry

Practitioner must enter patient shipping information within the Vitaboom platform.

**Why:** Shopify cart defaults to the logged-in user's (practitioner's) shipping address.

**Required fields:**
- Patient name
- Shipping address
- Phone number
- Email

### 5. Order Management & Database

- Orders linked to specific **client ID** (even when practitioner email is same across orders)
- New metadata field: `order_type`

| Value | Description |
|-------|-------------|
| `dropship_subscription` | Practitioner places recurring order for patient |
| `dropship_one_time` | Practitioner places single order for patient |
| `patient_subscription` | Patient places their own recurring order |
| `patient_one_time` | Patient places their own single order |

**⚠️ Database changes needed BEFORE deployment** due to anticipated high volume from key clients.

---

## Phase 2: Enhanced Experience (Future)

### API-Based Checkout

- Bypass Shopify checkout entirely using Shopify API
- Process transactions directly from Vitaboom platform
- Keeps entire user experience within the platform
- Eliminates redirect to Shopify checkout page

---

## Technical Notes

- Shopify stores one "price" field per product
- Discounts are applied for specific selling plans
- Subscription discount (15%) managed through Shopify selling plans

---

## Dependencies

| Dependency | Owner | Notes |
|------------|-------|-------|
| Database schema changes | Scott | Discuss before deployment |
| Shopify API investigation | Matt | For Phase 2 feasibility |

---

## Key Clients Waiting

- **NextGen Health** — 100% dropshipping model
- Other top clients identified as urgently needing this feature

---

*Source: Matt <> Justin meeting, Feb 6, 2026*

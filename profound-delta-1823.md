# Shopify Checkout Editor Experience: Configuration & Customization

> **Context**: Understanding Shopify's checkout customization capabilities for E-24 (Vitaboom Checkout Experience - Sunset Shopify)
> **URL Analyzed**: `admin.shopify.com/store/vitaboom-new/settings/checkout/editor/profiles/868090162?page=checkout`
> **Date**: 2026-02-05

---

## Executive Summary

Shopify's Checkout Editor is a no-code visual tool that allows merchants to customize the appearance and functionality of their checkout, thank you, and order status pages. Understanding these capabilities is essential for E-24 as we determine which features to replicate in our custom Bankful-powered checkout.

---

## 1. Checkout Editor Structure

### Access Path
`Settings > Checkout > Configurations > Customize`

### Page Types Available
| Page | Purpose |
|------|---------|
| **Checkout** | Contact info, delivery/shipping, payment collection |
| **Thank You** | Post-purchase confirmation |
| **Order Status** | Order tracking and updates |
| **Profile** | Customer account details (if enabled) |
| **Orders** | Customer order history (if accounts enabled) |

### Checkout Layout Options
- **One-Page Checkout**: All steps on single page (contact, delivery, payment)
- **Three-Page Checkout**: Separate Information, Shipping, and Payment pages

---

## 2. Branding Customization

### Logo & Header
| Setting | Options |
|---------|---------|
| Logo upload | Image file |
| Logo alignment | Left, Center, Right |
| Logo position | Banner overlay or standalone |
| Header position | Top, side configurations |
| Breadcrumbs | Show/hide navigation steps |
| Cart link | Show/hide return to cart |

### Colors
- **Base colors**: Primary palette for checkout
- **Color schemes**: Up to 4 schemes for different sections
- **Accent colors**: Interactive elements (links, focus states)
- **Brand colors**: Primary and secondary buttons

### Typography
| Element | Customization |
|---------|---------------|
| Font family | System fonts or custom |
| Font size | Adjustable per element |
| Font weight | Light to bold |
| Letter spacing (kerning) | Fine-tuned spacing |
| Letter case | Normal, uppercase, etc. |
| Header levels | Up to 3 distinct styles |

### Visual Elements
| Element | Options |
|---------|---------|
| Favicon | Custom browser tab icon |
| Background | Color or image |
| Corner radius | Global setting for inputs, buttons, cards |
| Border widths | Adjustable per section |

---

## 3. Section Customization

### Main Content Area
- Background images
- Border styling
- Color scheme selection
- Spacing adjustments

### Order Summary Sidebar
- Background customization
- Border widths
- Color scheme (can differ from main)
- Product image display
- Line item formatting

### Footer
- Footer width options
- Store policy display (Privacy, Terms, Refund)
- Custom footer content

---

## 4. Checkout Extensibility (Apps & Blocks)

### Target Locations for Extensions
Apps and custom blocks can be placed at strategic points:

| Location | Use Cases |
|----------|-----------|
| Before contact info | Trust badges, welcome messages |
| After contact info | Newsletter signup, custom fields |
| Before shipping options | Delivery instructions, gift messages |
| After shipping options | Insurance offers, expedited shipping upsells |
| Before payment | Trust signals, security badges |
| After payment | Order bumps, upsells |
| Order summary | Product upsells, discount displays |
| Thank you page | Post-purchase offers, surveys |

### Common Extension Types
1. **Upsells & Cross-sells**: In-checkout product recommendations
2. **Custom Fields**: Gift messages, delivery notes, birthdates
3. **Trust Badges**: Security icons, payment method logos
4. **Banners**: Promotional messages, urgency timers
5. **Surveys**: NPS, feedback collection
6. **Loyalty Programs**: Points display, reward redemption

### Checkout UI Extensions Framework
- Runs in isolated sandbox (security)
- Uses Polaris web components
- No checkout.liquid access (deprecated August 2025)
- Performance-optimized rendering

---

## 5. Checkout Settings (Non-Visual)

### Customer Information
| Setting | Options |
|---------|---------|
| Customer accounts | Disabled, Optional, Required |
| Customer contact | Email only, Email or Phone |
| Full name | First + Last or Company name option |
| Address autocomplete | Enable/disable |

### Order Processing
- Order number prefix/suffix
- Draft order behavior
- Abandoned checkout emails

### Shipping
- Shipping rate calculations
- Free shipping thresholds
- Local delivery/pickup options

### Payment
- Payment gateway selection
- Manual payment methods
- Payment captures (automatic vs manual)

---

## 6. What Shopify Provides That We Must Consider for E-24

### Must Replicate (MVP)
| Feature | Priority | Notes |
|---------|----------|-------|
| Basic branding (logo, colors) | High | Brand consistency |
| Clean checkout flow | High | Contact → Shipping → Payment |
| Mobile responsiveness | High | Majority of Vitaboom traffic |
| Order summary display | High | Product, quantity, pricing |
| Address collection | High | Shipping + billing |
| Payment form | High | Bankful integration |
| Confirmation page | High | Order receipt |

### Nice to Have (Phase 2+)
| Feature | Priority | Notes |
|---------|----------|-------|
| Custom fields (delivery notes) | Medium | Patient instructions |
| Trust badges | Medium | Payment security indicators |
| Multiple color schemes | Low | Visual polish |
| Custom typography | Low | Brand refinement |
| Upsells/cross-sells | Low | Revenue optimization |

### Explicitly Out of Scope
| Feature | Reason |
|---------|--------|
| Customer accounts | Vitaboom has own patient system |
| Order history pages | Handled in Vitaboom app |
| Checkout.liquid customization | Deprecated, not applicable |
| Shopify Apps/Extensions | Building custom instead |

---

## 7. Vitaboom-Specific Requirements

Based on E-24 planning docs and team discussions:

### Required Features
1. **Protocol-Based Cart**: Pre-populated from practitioner protocol
2. **Subscription Toggle**: One-time vs recurring selection
3. **Quantity by Time-of-Day**: Morning/afternoon/evening dosing
4. **Discount Code Application**: Practitioner discount validation
5. **Shipping Address Collection**: US addresses primarily
6. **Bankful Payment Integration**: Card tokenization, vaulting

### Branding Requirements
- Vitaboom logo
- Brand colors (likely sourced from vitaboom-web)
- Mobile-first design
- Clean, trustworthy aesthetic (health/wellness space)

---

## 8. Recommendations

### Immediate Actions
1. **Screenshot current Shopify checkout** for reference during development
2. **Document exact color values and fonts** used in current checkout
3. **List any custom fields or extensions** currently active
4. **Identify checkout conversion rate** as baseline metric

### Development Approach
1. Start with functional MVP (contact → shipping → payment → confirm)
2. Add branding in parallel (logo, colors, fonts)
3. Defer advanced customization (trust badges, upsells) to Phase 2
4. Test mobile experience heavily

---

## Sources

- [Shopify Help Center: Customizing and editing checkout](https://help.shopify.com/en/manual/checkout-settings/customize-checkout-configurations)
- [Shopify Help Center: Using the checkout editor](https://help.shopify.com/en/manual/checkout-settings/customize-checkout-configurations/checkout-editor)
- [Shopify Help Center: Customizing checkout style](https://help.shopify.com/en/manual/checkout-settings/customize-checkout-configurations/checkout-style)
- [Shopify Dev: Checkout UI Extensions](https://shopify.dev/docs/api/checkout-ui-extensions/latest)
- [Shopify Dev: Apps in checkout](https://shopify.dev/docs/apps/build/checkout)
- [Shopify Partners: 10 Ways to Customize Checkout](https://www.shopify.com/partners/blog/customize-checkout)
- [Grey Shark Media: Shopify Checkout UI Extensions 2025](https://greysharkmedia.com/how-to-use-shopify-checkout-ui-extensions-in-2025/)

---

*Brief prepared by Ruk for Serhii - 2026-02-05*

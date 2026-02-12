# Noble Painting App - Discovery Session Brief

**Prepared for**: Matt (Product Manager)
**Date**: February 12, 2026
**Context**: Pre-discovery session prep for a new client engagement
**Client**: Sean (Noble Painting, Scottsdale AZ area)

---

## Executive Summary

Sean runs **Noble Painting**, a multi-trade contracting company (painting, drywall, stucco, epoxy, framing, carpentry, cabinetry). He's in the process of **franchising the painting side** and wants to build a SaaS product that combines **LiDAR-powered room scanning** with a **painting-specific CRM** to serve the broader painting contractor market.

His endgame is **acquisition within 2-3 years**. He's not optimizing for profit in year one -- he wants rapid customer acquisition and scale.

Austin met Sean through a mutual connection at a coworking space. Sean's sales manager (Lena) handles ~80% of sales; he has a project manager (JB), four paint crews of 3-4 people each, plus drywall, stucco, framing, and carpentry subs. He recently invested $11K in a diamond head grinding machine for epoxy floor prep, which gives you a sense of his growth trajectory and willingness to invest in tools.

---

## What Sean Wants to Build

### Core Product Vision

An **iOS app** that lets painting contractors:

1. **Scan rooms with LiDAR** (Apple RoomPlan API) to auto-measure walls, ceilings, baseboards
2. **Auto-calculate paint quantities** based on product coverage rates (e.g., 300-400 sq ft/gallon)
3. **Generate professional proposals** with line items, options, and add-ons
4. **Manage their business** through a built-in CRM (leads, pipeline, scheduling, invoicing)

### The Key Differentiator

**No existing product combines LiDAR scanning with painting-specific estimation and CRM.** Sean's current CRM (DripJobs) requires manual entry for every measurement. There is no app on the market that can scan a room and pipe those dimensions directly into an estimate/proposal workflow for painters.

### Long-Term Roadmap (Sean's Full Vision)

These came up in conversation but are explicitly **not MVP**:

- **AI substrate identification**: Computer vision to detect drywall cracks, wood rot, stucco damage, different panel types (T-111, etc.) and auto-generate repair line items
- **Paint manufacturer partnerships**: Algorithmic recommendation engine that suggests paints by substrate/room type (e.g., mold-resistant for bathrooms). Tied to promo codes from Sherwin-Williams, Dunn-Edwards, and Home Depot
- **Marketplace/e-commerce layer**: Place paint orders through the app, earn referral fees like Amazon (3-7%)
- **Advertising platform**: Paint manufacturers bid for visibility in the proposal/recommendation flow
- **B2C expansion**: DIY homeowner version with marketplace integration
- **Android support**: Port the app once iOS is validated

---

## Pricing & Business Model

### SaaS Subscription
- **Target**: $300/month base + per-user fees for additional field managers
- **Year 1 promo**: ~$150/month to undercut DripJobs and drive adoption
- **Sean's rationale**: Must be cheaper than DripJobs ($97-150/mo) initially to win market share, then grow into premium pricing as features expand

### Revenue Philosophy
- Not seeking profitability in years 1-3
- Focused on customer acquisition, low churn, high-value KPIs for acquisition valuation
- May flip to profitability in years 3-5 if needed

### Target Customer Profile
- Painting contractors doing **$250K+ annual revenue** (marketing entry point)
- Ideal: **$500K+ revenue**, already using a CRM (DripJobs, ServiceTitan, etc.)
- Companies large enough to pay for SaaS but frustrated by existing tools

---

## Competitive Landscape

### Direct Competitors

| Product | Price | Strengths | Weaknesses |
|---------|-------|-----------|------------|
| **DripJobs** | ~$97/mo | Painting-specific CRM; 2,300+ contractors; 40+ automated drip campaigns; production rate estimation; 2-way texting | No LiDAR/scanning; manual measurement entry; limited customization; limited reporting; no AI features |
| **PaintScout** | ~$79/mo | Painting-specific estimation; new integrated CRM; customizable production rates; invoicing + work orders | No LiDAR/scanning; no API; smaller user base; less robust marketing automation |
| **ServiceTitan** | ~$250-500/mo per tech | Enterprise-grade CRM; dispatch, marketing, KPI dashboards; 100K+ trade pros; deep QuickBooks integration | Very expensive ($700-800/mo all-in); complex setup (can take 12+ months); designed for 20+ technician companies; general contracting focus, not painting-specific |

### Adjacent / Scanning Tools

| Product | What It Does | Why It's Not a Competitor |
|---------|--------------|---------------------------|
| **MagicPlan** | LiDAR room scanning + floor plans | General-purpose, not painting-specific. No CRM/estimation |
| **RoomScan Pro** | LiDAR scanning with wall area + ceiling height | Measurement only. No painting workflow |
| **MeasureSquare** | LiDAR room scanning | Flooring-focused, not paint |
| **Canvas (Occipital)** | 3D room capture with human-assisted CAD output | High-end architectural, not contractor workflow |
| **Polycam / Scaniverse** | 3D scanning + photogrammetry | Consumer/creative tools, no trade workflow |

### AI Visualization Tools (Future Opportunity)

| Tool | Use Case | Notes |
|------|----------|-------|
| **Decor8 AI** | Room repainting visualization ($0.20/image) | API-ready, could integrate for "show the customer what it'll look like" |
| **HomeDesignsAI** | AI room redesign | Per-image pricing, REST API |
| **Wizart** | Virtual paint try-on | Enterprise pricing |
| **Sherwin-Williams ColorSnap** | Color visualization | Consumer-facing, not API-accessible |

### Competitive Gap (Our Opportunity)

**No product on the market combines:**
1. LiDAR room scanning (auto-measurement)
2. Painting-specific estimation (coverage rates, substrates, labor)
3. AI-assisted proposal generation
4. CRM pipeline management

This is a genuine whitespace. The closest competitors require painters to either (a) measure by hand and enter data manually, or (b) use separate apps for scanning and business management.

---

## Partner Ecosystem

### Sherwin-Williams

- **Scale**: Largest paint manufacturer in the world. 4,800+ stores in the US
- **PRO+ Program**: Free contractor program with exclusive pricing, job quote generators, paint calculators, color visualization tools, Project Bids tool, business training series
- **Integration potential**: EDI integrations available via ConnectPointz/SPS Commerce. No public API, but partnership-level integrations are possible
- **Partnership angle**: Exclusive paint recommendations for SW products within the app. Promo code system where SW-referred contractors only see SW products. SW gets guaranteed product placement; we get distribution through their 4,800 stores and contractor network
- **Known issue (from Sean)**: Chronic inventory shortages at individual stores. Contractors end up driving between multiple locations. A real-time inventory check via the app could be a major value-add

### Dunn-Edwards

- **Scale**: ~150 stores across AZ, CA, NV, NM and a few other western states. Strong in the Southwest
- **Contractor programs**: Discount program with charge accounts, job accounts, installment accounts for equipment. 2% prompt pay discount
- **Partnership angle**: Same promo code / exclusive recommendation model as SW. Sean has a personal contact (Tom Wheeler, DE sales rep with 20+ years industry experience) who could potentially connect to national decision-makers
- **Advantage**: Smaller and more likely to move quickly on a partnership than SW. Regional focus aligns well with initial market launch in Arizona

### Home Depot / Behr

- **Scale**: 2,300+ stores. Behr is their house paint brand
- **Pro Referral Program**: Connects homeowners with contractors. Pros earn leads through Pro Xtra loyalty points
- **Pro Xtra Paint Rewards**: Loyalty/discount program for contractor paint purchases
- **Partnership angle**: QR codes in stores linking to app sign-up with HD promo codes. Behr product recommendations within the app. Potential lead-gen integration with Pro Referral
- **Note**: Home Depot's contractor base skews more toward smaller operations and DIY-adjacent pros

### Partnership Revenue Model (Sean's Vision)

**Phase 1** (Launch): Partners get exclusive product recommendations within the app for contractors who sign up with their promo code. No revenue share needed -- the value to the manufacturer is guaranteed product placement.

**Phase 2** (Scale): As user base grows, introduce referral fees on paint purchases made through the app (3-7% per transaction, Amazon-style).

**Phase 3** (Mature): Advertising platform where manufacturers bid for placement in proposals and recommendations.

---

## Market Sizing

### The Painting Contractor Market

| Metric | Value |
|--------|-------|
| Painting contractor businesses in the US | ~223,000 |
| Total industry employment | ~317,000 workers |
| Industry revenue (2025) | ~$49 billion |
| Average company size | 75% have 4 or fewer employees |
| Market concentration | Top 3 companies hold <3% market share |
| Growth | ~4,600 new companies expected by 2027 |

### Software TAM Estimation

**Bottom-up approach:**
- ~223,000 painting contractors in the US
- Target: companies doing $250K+ revenue = ~50,000-70,000 companies (estimated)
- At $300/month = $3,600/year per customer
- **Serviceable addressable market**: ~$180M-$250M annually
- Even capturing 5% market share = $9M-$12.5M ARR

**Comparable SaaS benchmarks:**
- DripJobs: 2,300 contractors at ~$97/mo = ~$2.7M ARR (estimated)
- ServiceTitan: 100,000+ trade pros, $5B+ valuation (went public 2024)
- Construction software market overall: $6.8B-$11B (2025), growing at ~10% CAGR

### Acquisition Benchmarks

For a vertical SaaS with strong unit economics and low churn:
- **Revenue multiple**: 8-15x ARR for high-growth vertical SaaS
- **At 2,000 paying customers ($300/mo)**: ~$7.2M ARR = $58-108M potential valuation
- **ServiceTitan comp**: IPO'd at ~50x revenue (exceptional, but shows ceiling for trade contractor SaaS)

---

## Technical Considerations for Discovery

### Apple RoomPlan API (LiDAR)

- **Accuracy**: +/- 2-5 cm per wall. More than sufficient for paint estimation
- **Max room size**: ~30 ft x 30 ft before drift becomes an issue
- **Scan time**: 2-3 minutes per room
- **What it gives you**: Wall dimensions, door/window detection (auto-subtract from paint area), ceiling height, furniture detection
- **Device requirement**: iPhone 12 Pro or later (Pro models only have LiDAR). iPad Pro also works
- **Limitations**: Mirrors, glass, dark surfaces cause gaps. Ceilings >5m may be out of range. Session should not exceed 5 minutes. Heavy battery/CPU usage
- **Implementation**: Native Swift required for the scanning module. CRM/business logic can be React Native

### Architecture Recommendation

- **Mobile app**: React Native for CRM + business logic (cross-platform ready for future Android)
- **LiDAR module**: Native Swift using Apple RoomPlan API (iOS only for V1)
- **Backend**: Node.js (our standard stack) serving both mobile and future web app
- **Web app**: React (admin dashboard, account creation, payment -- avoids Apple's 30% cut)
- **Key architectural note**: Apple takes 30% on digital services purchased through the App Store. CRM subscriptions count. Solution: account creation and payment happen on web only. Mobile app is sign-in only. This is standard practice (Netflix, Spotify, etc.)

### MVP Scope Suggestions (For Discovery Discussion)

**Proof of Concept (milestone 1):**
- Scan a room with LiDAR
- Display wall area, ceiling area, baseboard linear footage
- Select a paint product, see gallons needed
- That's it

**MVP (milestone 2):**
- Multi-room scanning with project-level aggregation
- Configurable pricing (labor rate, paint products, coverage rates)
- Proposal/estimate PDF generation with line items and options
- Basic lead/job pipeline (list view, status tracking)
- iOS only, web app for account creation/billing

**Post-MVP iterations:**
- Full CRM (drip campaigns, scheduling, invoicing)
- Substrate detection / AI line item suggestions
- Paint manufacturer integrations
- Android version

---

## Key Questions for Discovery

### Product & Scope
1. Walk through DripJobs screen-by-screen: what do you love, what do you hate, what's missing?
2. What does your current estimation process look like step-by-step? How long does it take?
3. How many estimates does your team do per week/month?
4. What's the typical proposal value range?
5. Which features of DripJobs would you absolutely need from day one vs. could live without temporarily?

### Business & Go-to-Market
6. How far along are the franchise conversations? Timeline?
7. How warm is the Tom Wheeler / Dunn-Edwards connection? What would activation look like?
8. What's your current monthly software spend across all tools?
9. Do you have a beta test group in mind beyond your own team?
10. Who is the decision-maker on budget/timeline? Is there a business partner or investor involved?

### Technical
11. What phones do your field managers currently use? (Need to verify LiDAR-capable devices)
12. What other software does Noble Painting use today? (QuickBooks? Google Calendar? etc.)
13. Do you need the CRM to handle payments/invoicing from day one, or can that come later?

---

## Engagement Recommendation

This is a strong opportunity. The client:
- Has deep domain expertise (runs the business, understands pain points firsthand)
- Has a clear acquisition-focused business strategy
- Has warm relationships with potential distribution partners (Dunn-Edwards, Sherwin-Williams)
- Is already investing in scaling (franchising, hiring managers, buying equipment)
- Is building in a genuine whitespace (LiDAR + painting CRM = nobody does this)

**Risk factors:**
- Scope creep potential is high (Sean has a big vision). Matt's role in ruthlessly limiting MVP scope will be critical
- Paint manufacturer partnerships are speculative until validated
- LiDAR scanning is iOS-only, limiting initial addressable market
- 2-3 year acquisition timeline is aggressive for building + scaling a vertical SaaS

**Recommended approach:** Start with a focused proof of concept (LiDAR scan + measurement output), validate that the scanning experience works for painting estimation, then layer on CRM features iteratively. Keep the manufacturer partnership conversations warm but don't let them drive product decisions in V1.

---

*Prepared by Fractal Labs | brief.fractallabs.dev*

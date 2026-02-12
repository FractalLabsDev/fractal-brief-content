# Painting Contractor LiDAR App - Meeting Prep Brief

**Prepared for**: Austin
**Context**: Potential client meeting in ~2 hours
**Client**: Painting contractor business looking to modernize quoting/estimation

---

## TL;DR

This is **buildable and differentiated**. The iPhone's RoomPlan API gives you instant room dimensions → wall area → paint/labor estimates. Add AI visualization and you have something better than the existing market, which is fragmented (Jobber for CRM, PaintScout for estimates, MagicPlan for scanning — no one does it all well).

**Unique value prop**: Scan the room, get the quote, show the customer what it'll look like — all in one app, on-site, in minutes.

---

## What the Client Probably Wants

### Core Use Case: Instant On-Site Estimates
1. Walk into customer's home with iPhone
2. Scan each room with LiDAR (RoomPlan API)
3. App calculates wall area automatically
4. Applies contractor's pricing (labor + materials)
5. Generates professional PDF quote
6. Customer signs on the spot

### Bonus Use Case: AI Mockups
- "What would this room look like in Sherwin-Williams 'Agreeable Gray'?"
- Show before/after visualization while standing in the room
- Helps close the sale on premium finishes

---

## Technical Feasibility

### RoomPlan API Gives You:
- Wall dimensions (height, width, placement)
- Door/window detection (auto-subtract from paint area)
- Ceiling height
- Export to USDZ/USD with full geometry

### What You'd Build:
1. **Surface area calculator** - Wall height × perimeter - doors/windows
2. **Pricing engine** - $/sq ft for labor + materials, multiplied out
3. **Quote generator** - PDF with line items, totals, terms

### Accuracy Reality Check
| Metric | Value |
|--------|-------|
| Wall measurement | ±2-5 cm per wall (good enough for estimates) |
| Max room size | 9×9m (30×30 ft) before drift |
| Scan time | 2-3 minutes per room |
| Devices required | iPhone 12 Pro or later (LiDAR equipped) |

**Limitation**: Mirrors, glass, dark surfaces cause gaps. High ceilings (>5m) may be out of range.

**Good enough for painting?** Yes. Painters already estimate visually — this is more accurate than eyeballing and faster than measuring tape.

---

## Competitive Landscape

### Existing Solutions (Fragmented)

| Tool | What It Does | Gap |
|------|--------------|-----|
| **MagicPlan** | Room scanning + floor plans | General purpose, not painting-specific |
| **PaintScout** | Painting estimates | No LiDAR — manual entry |
| **Jobber** | CRM, scheduling, invoicing | No estimation or scanning |
| **MeasureSquare** | LiDAR room scanning | Focused on flooring, not paint |

**Opportunity**: No one combines LiDAR scanning + painting-specific estimation + AI visualization in a single app.

### Pricing in Market (2026)
- Interior painting: **$3.50-5.50/sq ft** (labor + materials)
- Labor rate: **$40-60/hour**
- Coverage: **350-400 sq ft/gallon**

An app should let contractors set their own rates + material costs.

---

## AI Visualization Options

### For Showing Paint Colors

| Service | Price | Integration |
|---------|-------|-------------|
| **Decor8 AI API** | $0.20/image | Simple REST API |
| **HomeDesignsAI** | Per-image pricing | REST API |
| **Wizart** | Enterprise | One-click setup |

**How it works**: Upload room photo → specify paint color → get photorealistic rendering in ~20 seconds.

**Use case for painters**: "Here's your living room in 'Revere Pewter'" — show on iPad while standing in the room. Massive sales tool.

---

## What They Might Ask

### "How accurate is the scanning?"
±2-5 cm per wall. More accurate than visual estimates, faster than measuring tape. For painting quotes, this is plenty.

### "Does it work for all rooms?"
Works great for typical residential rooms. Challenges: mirrors, glass walls, very dark surfaces, ceilings over 5m, rooms larger than 30×30 ft. These edge cases can be handled with manual override.

### "Can we integrate with QuickBooks?"
Yes — most contractor software does this. It's a standard integration pattern.

### "What about exterior painting?"
LiDAR range is ~5m, so it won't scan a house facade directly. Options:
- Manual input for exterior
- Satellite imagery + AI measurement (like MapMeasure Pro)
- Drone photography (more complex)

Interior is the sweet spot for v1.

### "How much would this cost to build?"
Depends on scope, but rough ranges:
- **MVP** (scan → quote): 6-10 weeks, $40-60k
- **Full product** (+ AI viz, CRM, invoicing): 4-6 months, $100-150k+

Or position as recurring SaaS: $50-150/month/user is standard for contractor software.

### "What phones do my painters need?"
iPhone 12 Pro or later (Pro models only — they have LiDAR). iPad Pro also works.

---

## Differentiation Angles

### If they want to stand out:
1. **Speed** — Quote in 10 minutes on-site vs. "I'll email you tomorrow"
2. **Visualization** — Show the finish before painting
3. **Consistency** — Same pricing formula every time, no guesswork
4. **Professionalism** — Digital quote with logo, terms, e-signature

### Moat potential:
- Lock-in via saved customer data + pricing history
- Network effects if they expand to multiple painting companies
- Proprietary pricing data over time

---

## Questions to Ask the Client

1. **What's their current process?** (Tape measure? Eyeball? Software?)
2. **How many estimates per week?** (Volume determines ROI)
3. **Interior only or exterior too?** (Affects scope significantly)
4. **Do they want CRM/invoicing or just estimation?** (Build vs. integrate)
5. **Team size?** (1 contractor vs. 20 painters = different needs)
6. **Budget and timeline expectations?**

---

## Recommendation

**This is a solid opportunity.** The technology exists, the market is fragmented, and painting contractors are increasingly tech-forward. The combination of LiDAR scanning + painting-specific estimation + AI visualization would be genuinely differentiated.

Start with interior estimation MVP, prove value, then expand to visualization and CRM integrations.

---

## Sources

### LiDAR & RoomPlan
- [RoomPlan Overview - Apple Developer](https://developer.apple.com/augmented-reality/roomplan/)
- [RoomPlan Accuracy Testing](https://www.it-jim.com/blog/roomplan-framework-by-apple/)
- [Building a Room Scanning App with RoomPlan](https://medium.com/simform-engineering/building-a-room-scanning-app-with-the-roomplan-api-in-ios-a5e9f66cfaaf)

### Painting Estimation
- [Painting Cost Per Square Foot 2026](https://bhumicalculator.com/countries/united-states/painting-cost-per-square-foot)
- [How to Estimate a Paint Job - Benjamin Moore](https://www.benjaminmoore.com/en-us/contractors/build-your-business/increase-productivity/estimating-paint-jobs)
- [Paint Calculator - Sherwin-Williams](https://www.sherwin-williams.com/en-us/color/color-tools/paint-calculator)

### Competitor Landscape
- [PaintScout - Painting Contractor Software](https://www.paintscout.com)
- [Jobber - Painting Contractor Software](https://www.getjobber.com/industries/painting-contractor-software/)
- [MagicPlan](https://magicplan.app/)
- [MeasureSquare Room Scanner](https://measuresquare.com/room-scanner-3d-lidar-scanning-app-measuring-tool/)

### AI Visualization
- [Decor8 AI API](https://www.decor8.ai/ai-virtual-staging-api)
- [HomeDesignsAI API](https://homedesigns.ai/api)
- [Paint AI - Remodel AI](https://remodelai.app/paint-ai/)

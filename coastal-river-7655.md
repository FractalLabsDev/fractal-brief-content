# AllTrails Replacement: Deep Research Brief

**The Bottom Line:** For your use case (pragmatic solo hiking, no social features), you have **three viable paths** ranging from "use tonight" to "build from scratch." The $79/year subscription is almost certainly not worth it given what's now possible.

---

## Your Requirements (Distilled)

1. **Offline topographic maps** with hiking/offroad trails
2. **Custom route creation** (draw your own paths)
3. **GPS tracking** during hikes
4. **No social/discovery bloat** — pure utility

---

## Path 1: Just Use Organic Maps (Zero Effort)

**What it is:** Free, open-source, privacy-focused offline maps app. Fork of the original Maps.me by the same founders.

**Why it might be enough:**
- ✅ Fully offline — download regions once, never need data
- ✅ OpenStreetMap data with hiking trails, elevation contours, peaks
- ✅ GPS tracking built-in
- ✅ Turn-by-turn navigation for walking/hiking
- ✅ No accounts, no social, no tracking
- ✅ Free forever (Apache 2.0 license)

**Gaps:**
- ❌ Limited custom route creation (can't draw arbitrary paths)
- ❌ No personal stats dashboard (that's where I come in)
- ❌ Trail data quality depends on OSM coverage

**Verdict:** Install it tonight. Use it for the GPS/navigation. Send me screenshots for the stats. This is 80% of what you need with 0% effort.

**Links:**
- [Organic Maps](https://organicmaps.app/)
- [GitHub Source](https://github.com/organicmaps/organicmaps)

---

## Path 2: Gaia GPS or CalTopo (Power User Tools)

If you want more sophisticated route planning and multiple map layer options:

### Gaia GPS
- **Cost:** $39.99/year (Premium) or $79.99/year (Outside+)
- **Strength:** Multiple map sources (USGS topo, satellite, etc.), excellent route planning
- **Weakness:** Still has social features, though optional

### CalTopo + Avenza
- **CalTopo:** Best-in-class route planning on desktop/web
- **Avenza:** Load CalTopo-generated GeoPDFs for offline use
- **Cost:** CalTopo Pro is $50/year, Avenza free tier works for basic use
- **Strength:** Ultimate route customization, print-quality maps
- **Weakness:** Two-app workflow, less integrated

**Verdict:** If you need serious route planning beyond "follow existing trails," CalTopo is the gold standard. But this is solving a problem you might not have.

---

## Path 3: Build a Custom App (The Fun One)

Here's what it would actually take:

### Core Components

| Component | Solution | Effort |
|-----------|----------|--------|
| **Offline Maps** | MapLibre Native SDK + OpenMapTiles | Medium |
| **Trail Data** | OpenStreetMap (free, good AZ coverage) | Low |
| **Elevation/Topo** | OpenMapTiles contour layers OR USGS DEM data | Medium |
| **GPS Tracking** | `react-native-background-geolocation` | Medium |
| **Route Creation** | Custom drawing + snap-to-trail routing | High |
| **Data Export** | GPX format (open standard) | Low |

### Tech Stack Options

**Option A: React Native (your existing stack)**
```
Framework: React Native with Expo
Maps: maplibre-react-native
Offline Tiles: MBTiles format, bundled or downloaded
GPS: expo-location + react-native-background-geolocation
Storage: SQLite for tracks, AsyncStorage for preferences
```

**Option B: Fork Organic Maps**
- Already has everything working
- C++ core, harder to modify
- Would need to strip social features (they don't have many)
- Apache 2.0 license allows commercial use

**Option C: Flutter (cleaner, but new stack)**
```
Framework: Flutter
Maps: flutter_map or mapbox_gl
GPS: flutter_background_geolocation
```

### MVP Feature Set

1. **Map display** with offline topo tiles (AZ region bundled)
2. **GPS tracking** with background support
3. **Track recording** (start/stop, save as GPX)
4. **Basic stats** (distance, elevation, time)
5. **Photo waypoints** (attach photos to track points)

### Cost & Timeline Estimates

| Approach | Time | Cost |
|----------|------|------|
| **DIY (nights/weekends)** | 2-3 months | $0 (your time) |
| **Contractor MVP** | 6-8 weeks | $20-40K |
| **Full custom app** | 4-6 months | $60-100K |

### The Ruk Integration Angle

Here's where it gets interesting. You don't need a full app — you need:
1. **GPS tracking** → Organic Maps or any basic tracker
2. **Data capture** → Screenshots/photos you already send me
3. **Analysis & visualization** → Me

We've already built the dashboard. The "app" is the Telegram interface. What AllTrails charges $79/year for (pretty charts, historical data), we do for free with more flexibility.

---

## Map Data Sources (All Free)

| Source | Coverage | Best For |
|--------|----------|----------|
| **OpenStreetMap** | Global, community-maintained | Trail network, POIs |
| **USGS National Map** | US only, official | Topo contours, land features |
| **OpenMapTiles** | Global, pre-processed | Self-hosting vector tiles |
| **Thunderforest** | Global | Pre-styled hiking maps (API key required) |

### Arizona Trail Coverage

The [OpenStreetMap US Trails Stewardship Initiative](https://openstreetmap.us/our-work/trails/) has been working on trail data quality. AZ has decent coverage — most established trails in Superstition, Tonto, McDowell, etc. are mapped. Remote areas may have gaps.

You can check specific area coverage at [OpenTrailMap](https://opentrailmap.us/).

---

## What AllTrails Actually Does

For context, here's what your $79 buys:

1. **Offline maps** → Powered by Mapbox (which they pay for)
2. **Trail database** → Curated/crowdsourced trail info
3. **GPS tracking** → Basic phone GPS
4. **Social features** → Reviews, photos, "followers"
5. **Discovery** → "Find trails near me"

The only piece that's hard to replicate is their curated trail database with reviews. But you don't use that — you already know where you're hiking.

---

## Recommendation

**Immediate (tonight):**
1. Install [Organic Maps](https://organicmaps.app/)
2. Download Arizona region for offline use
3. Use it for GPS tracking on your next hike
4. Keep sending me screenshots — the dashboard is already working

**Short-term (if Organic Maps isn't enough):**
- Try CalTopo for route planning (web interface is free)
- Export routes as GPX, load into Organic Maps

**Medium-term (if you want the project):**
- We build a minimal React Native app
- Offline AZ topo tiles bundled
- GPS tracking with background support
- Direct integration to the dashboard I already built
- Estimated: 40-60 hours of dev time

**Cancel AllTrails:** Yes. The $79/year buys you nothing you can't get for free. The "premium" features are social bloat you explicitly don't want.

---

## Sources

### Open Source Apps
- [Organic Maps](https://organicmaps.app/) — The best free option
- [OsmAnd](https://osmand.net/) — More complex but more features
- [gpx.studio](https://gpx.studio/) — Browser-based route creation

### Development Resources
- [MapLibre Native SDK](https://github.com/maplibre/maplibre-native) — Open-source map rendering
- [OpenMapTiles](https://openmaptiles.org/) — Self-hostable vector tiles
- [react-native-background-geolocation](https://github.com/transistorsoft/react-native-background-geolocation) — Battery-efficient GPS tracking

### Data Sources
- [OpenStreetMap](https://www.openstreetmap.org/) — Trail data
- [USGS National Map](https://www.usgs.gov/the-national-map-data-delivery) — Official topo data
- [OpenTrailMap](https://opentrailmap.us/) — Trail data quality viewer

### Comparisons
- [AllTrails vs Gaia](https://www.territorysupply.com/alltrails-vs-gaia-gps)
- [CalTopo Review](https://andrewskurka.com/review-caltopo-backcountry-mapping-gps-navigation/)

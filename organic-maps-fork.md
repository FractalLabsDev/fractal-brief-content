# Organic Maps Fork Feasibility Assessment

**Goal:** Fork Organic Maps to add custom hike logging capabilities (screenshots/photos, stat parsing, dashboard integration)

**Date:** 2026-02-09

---

## Executive Summary

**TL;DR:** Forking Organic Maps for custom logging is **technically feasible but heavyweight**. The codebase is mature, well-architected, and already has track recording built-in (released Sept 2024). However, the complexity of building/maintaining a fork is significant.

**Recommendation:** Start with a **companion app approach** that reads Organic Maps' GPX exports rather than forking. If this proves insufficient, then consider the fork.

---

## 1. Architecture Overview

### Tech Stack
- **Core:** C++ (68.6%) - cross-platform mapping engine
- **Android:** Java (6.5%) + JNI bridge to C++ core
- **iOS:** Swift (4.2%) + Objective-C++ bridge to C++ core
- **Desktop:** Qt for Linux/macOS/Windows support
- **Build system:** CMake 3.22.1+

### Project Structure
```
organicmaps/
├── map/              # Core app logic, scene manager
├── storage/          # Map file reading
├── routing/          # Navigation engine
├── search/           # Search and ranking
├── tracking/         # Analytics (NOT GPS tracking)
├── android/          # Android UI and platform code
│   └── app/src/main/java/app/organicmaps/
│       └── location/  # GPS and TrackRecordingService
├── iphone/           # iOS UI and platform code
│   └── Maps/Core/Location/
├── qt/               # Desktop app
├── 3party/           # Third-party dependencies
└── cmake/            # Build configuration
```

### Complexity Metrics
- **43,570+ commits**
- **516 contributors**
- **30GB+ disk space required for build**
- Inherits legacy from MAPS.ME (forked Dec 2020)

---

## 2. GPS Tracking Implementation

### Current Track Recording Feature

**Status:** Fully implemented and shipped in **September 2024** release.

**Implementation:**
- **Android:** `TrackRecordingService.java` - foreground service with persistent notification
- **iOS:** Location management in `iphone/Maps/Core/Location/`
- **Core:** C++ `TrackRecorder` class (exact path unclear, likely in `map/` directory)
- Uses JNI/native methods: `TrackRecorder.nativeStartTrackRecording()` / `nativeStopTrackRecording()`

**What it does:**
- Records GPS coordinates while hiking/biking/running
- Works in background (survives app kill/close)
- Displays blue dots on map (darker = more time spent)
- Saves to Bookmarks & Tracks system
- Shows recording status in notification bar

**Known limitations:**
- Android 8+ background restrictions required rearchitecture
- Battery optimization is ongoing concern
- Export functionality was added later after user feedback

---

## 3. Data Export Capabilities

### Supported Formats
- **GPX** - GPS Exchange Format (standard for track data)
- **KML/KMZ** - Google Earth formats
- **GeoJSON** - Web-friendly JSON format

### Export Methods
1. **Single track:** Tap track → Share button
2. **Bulk export:** Bookmarks & Tracks → Three dots → Export GPX/KMZ/GeoJSON

### What's Included in Exports
Based on GitHub issues and feature development:
- GPS coordinates (lat/lon)
- Timestamps (added in recent versions)
- Elevation data (if available)
- Track metadata (name, description)
- Track colors (some export issues noted)

**Data NOT currently included:**
- Photos/screenshots taken during track
- Custom stat annotations
- Real-time weather data
- Heart rate or other sensor data

---

## 4. Extension Points

### Easy Extensions (No Fork Required)
1. **GPX Post-Processing**
   - Export GPX from Organic Maps
   - Parse with standard GPX libraries
   - Augment with photos (match timestamps)
   - Send to dashboard API

2. **Companion App Pattern**
   - Monitor Organic Maps export directory
   - Auto-detect new GPX files
   - Process and sync to dashboard
   - Take screenshots via system APIs

### Moderate Extensions (Fork Required)
1. **Custom Export Format**
   - Modify `kml/serdes_gpx.cpp` (likely location)
   - Add JSON export with custom fields
   - Hook into existing export UI

2. **Photo Integration**
   - Add camera button during recording
   - Store photos with GPS coords + timestamps
   - Include in export metadata

### Hard Extensions (Deep Fork)
1. **Dashboard Push Integration**
   - Add network layer to C++ core
   - Implement real-time sync
   - Handle auth/API tokens
   - Battery/bandwidth optimization

2. **Custom Stat Parsing**
   - Modify track recording logic
   - Add custom metrics (pace zones, breaks, etc.)
   - New UI for stat display

---

## 5. Build Complexity

### Prerequisites

**Disk Space:** 30GB free minimum

**Android:**
- Android Studio
- CMake 3.22.1+ from Android SDK
- Python 3
- 2GB+ RAM for IDE
- Target architectures: arm64-v8a, armeabi-v7a, x86_64

**iOS:**
- Mac required
- Xcode from App Store
- Apple Developer Program enrollment ($99/year)
- Code signing certificate configuration
- 20GB+ free space

**Desktop (Linux/macOS):**
- CMake 3.22.1+
- Qt 6
- Boost
- 20GB+ free space
- ccache recommended for faster rebuilds

### Build Process

**Android:**
```bash
# 1. Open Android Studio
# 2. Open project in android/ directory (NOT root!)
# 3. Configure SDK paths in SDK Manager
# 4. Select build variant (e.g., fdroidDebug)
# 5. Build > Build Bundle(s) / APK(s)

# Or command line:
cd android
./gradlew assembleDebug -Parm64  # Single arch for speed
```

**iOS:**
```bash
# 1. Open xcode/omim.xcworkspace
# 2. Configure signing (Signing & Capabilities)
# 3. Select "OMaps" scheme
# 4. Build in Xcode
```

**Time investment:**
- Initial setup: 2-4 hours
- First build: 30-60 minutes
- Incremental builds: 5-15 minutes
- Learning curve: 1-2 weeks to understand build system

### Maintenance Burden
- **Upstream sync:** Organic Maps is actively developed (~daily commits)
- **Merge conflicts:** High likelihood in C++ core
- **Platform updates:** iOS/Android OS changes require updates
- **Dependency management:** 3party/ libraries need periodic updates
- **Testing matrix:** 2 platforms × 3 Android archs × multiple OS versions

---

## 6. Realistic Forking Assessment

### Feasibility Score: **6/10** (Doable but Heavy)

**Pros:**
- ✅ Well-architected, modular C++ core
- ✅ Track recording already implemented
- ✅ GPX export already working
- ✅ Active community (516 contributors)
- ✅ Apache 2.0 license (permissive)
- ✅ Clear documentation (INSTALL.md, CONTRIBUTING.md)
- ✅ Existing fork example (CoMaps) proves it's viable

**Cons:**
- ❌ Large codebase (68% C++, cross-platform complexity)
- ❌ 30GB build requirements
- ❌ Dual-platform maintenance (iOS + Android)
- ❌ Fast-moving upstream (hard to stay synced)
- ❌ No clear plugin architecture (tight coupling)
- ❌ C++ expertise required for core changes
- ❌ Multi-language skills needed (C++, Java, Swift)

### Complexity by Goal

| Feature | Difficulty | Fork Required? | Time Estimate |
|---------|-----------|----------------|---------------|
| Parse GPX exports | Easy | No | 2-4 hours |
| Add photos to exports | Medium | Yes | 1-2 weeks |
| Dashboard auto-sync | Hard | Yes | 2-4 weeks |
| Custom stat tracking | Hard | Yes | 3-6 weeks |
| Real-time push | Very Hard | Yes | 4-8 weeks |

---

## 7. Alternative Approach (Recommended)

### Companion App Strategy

**Core idea:** Don't fork Organic Maps. Build a lightweight companion that leverages what it already does well.

**Architecture:**
```
[Organic Maps] → GPX export → [Companion App] → [Dashboard API]
     ↓                              ↓
 Track recording              Photo capture
                             Stat parsing
                             Auto-sync
```

**Implementation:**
1. **Android/iOS app** with minimal UI:
   - File system monitoring for new GPX exports
   - Camera integration for hike photos
   - Background service for auto-upload
   - Local SQLite for offline queue

2. **GPX Processing Library:**
   - Parse time, distance, elevation from GPX
   - Match photos to GPS coords via EXIF timestamps
   - Calculate custom stats (pace, breaks, etc.)
   - Generate dashboard-friendly JSON

3. **Dashboard Integration:**
   - REST API for track + photo upload
   - Webhook for real-time notifications
   - Web UI for viewing hikes

**Benefits:**
- ✅ No fork maintenance burden
- ✅ Smaller, focused codebase
- ✅ Faster development (weeks not months)
- ✅ Can update Organic Maps independently
- ✅ Works with other GPS apps (Strava, OsmAnd)
- ✅ Easier testing and deployment

**Drawbacks:**
- ❌ Requires manual "export from Organic Maps" step
- ❌ No real-time push during hike (post-hike sync only)
- ❌ Separate app = one more thing to manage

**Hybrid approach:**
- Start with companion app
- If real-time sync becomes critical, THEN fork
- You'll have proven the workflow first

---

## 8. Specific Recommendations

### For Your Use Case (Hike Logging)

**Immediate needs:**
- Capture time, distance, elevation, location → **Organic Maps GPX already does this**
- Add photos/screenshots → **Companion app camera integration**
- Parse stats → **Companion app GPX parsing**
- Send to dashboard → **Companion app API calls**

**Path forward:**

**Phase 1: Companion App (2-3 weeks)**
```bash
# Minimal viable product:
1. Monitor /storage/emulated/0/Android/data/app.organicmaps/files/bookmarks/
   (or iOS equivalent) for new .gpx files
2. Parse with gpxpy (Python) or gpx-parser (JS)
3. Show UI: "Upload hike?"
4. POST to your dashboard API
```

**Phase 2: Photo Integration (1 week)**
```bash
1. Add "Capture Photo" button in companion app
2. Store with GPS coords from device location
3. Match to track via timestamps
4. Upload photos alongside GPX data
```

**Phase 3: Auto-Sync (1 week)**
```bash
1. Background service detects new GPX files
2. Auto-upload without user intervention
3. Notification on success
```

**Only if needed - Phase 4: Fork Organic Maps (4-8 weeks)**
```bash
# If you MUST have real-time push:
1. Fork organicmaps/organicmaps
2. Add API key config to settings
3. Modify TrackRecorder to POST coords every N seconds
4. Add camera button to track recording UI
5. Build, sign, distribute custom APK/IPA
```

---

## 9. Existing Tools & Alternatives

Before forking, consider these existing solutions:

**Track Recording Apps:**
- **OpenTracks** (Android) - FOSS, GPX export, sensor support
- **OsmAnd** - Feature-rich but heavier, has trip recording
- **Strava** - Popular but proprietary, has API

**Companion App Inspiration:**
- **Wandrer.earth** - Gamifies exploring streets, uses GPX uploads
- **AllTrails** - Hike tracking with photos, but closed source
- **Gaia GPS** - Outdoor-focused, API-friendly

**Your advantage:** Full control over dashboard + custom stats

---

## 10. Decision Matrix

| Criterion | Companion App | Fork Organic Maps |
|-----------|---------------|-------------------|
| Development time | 2-3 weeks | 4-8 weeks (initial) |
| Maintenance | Low | High |
| Code complexity | Medium (single platform) | High (C++, Java, Swift) |
| Real-time sync | No (post-hike only) | Yes (possible) |
| Photo integration | Easy | Medium |
| Stat parsing | Easy | Medium |
| Dependency on OM | High (must export GPX) | None (full control) |
| Distribution | Standard app stores | Custom signing needed |
| Update frequency | Independent | Tied to OM upstream |
| Cost | Low | High (time/resources) |

**Recommendation Score:**
- **Companion App: 8/10** for most use cases
- **Fork: 4/10** unless you need real-time sync

---

## 11. Final Verdict

**Start with the companion app.** You can ship a working solution in 2-3 weeks that does 90% of what you need:
- ✅ Organic Maps handles GPS tracking (already excellent)
- ✅ Companion app handles photos, parsing, dashboard sync
- ✅ No fork maintenance burden
- ✅ Faster iteration and testing

**Fork only if:**
- You absolutely need real-time sync during hikes (not post-hike)
- You're willing to commit to ongoing maintenance
- You have C++, Java, and Swift expertise in-house
- You're okay with 4-8 week initial development + ongoing upstream merges

The companion app approach gives you the **path of least resistance** while keeping options open. You can always fork later if the workflow proves insufficient, and you'll have validated the product-market fit first.

---

## Appendix: Key Resources

**Repository:**
- https://github.com/organicmaps/organicmaps

**Documentation:**
- Build: https://github.com/organicmaps/organicmaps/blob/master/docs/INSTALL.md
- Contribute: https://github.com/organicmaps/organicmaps/blob/master/docs/CONTRIBUTING.md
- Structure: https://github.com/organicmaps/organicmaps/blob/master/docs/STRUCTURE.md

**Issues:**
- Track Recording: https://github.com/organicmaps/organicmaps/labels/Track%20Recording
- Original epic: https://github.com/organicmaps/organicmaps/issues/613

**Community:**
- Discussions: https://github.com/orgs/organicmaps/discussions
- Community fork (CoMaps): https://github.com/comaps/comaps

**Export Docs:**
- https://organicmaps.app/faq/bookmarks/how-to-export/

---

**Assessment prepared by:** Ruk  
**Date:** 2026-02-09  
**Next step:** Discuss with Austin - validate companion app approach before committing to fork

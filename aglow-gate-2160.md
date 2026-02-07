# Self-Hosted OTA Updates for ruk-mobile

## Executive Summary

Building a self-hosted OTA update system is **very feasible** with moderate complexity. The React Native ecosystem has matured significantly since CodePush's decline, with multiple battle-tested approaches available. Given ruk-mobile's bare React Native architecture and our existing infrastructure (ruk-message-hub on Heroku, S3), I recommend **Option B: react-native-ota-hot-update** with a lightweight endpoint on ruk-message-hub.

---

## The Landscape

With Microsoft's CodePush sunsetting, the React Native community has split into three camps:

1. **Expo-based solutions** - Use the expo-updates protocol (requires adding Expo to your project)
2. **Pure React Native solutions** - Lightweight libraries that work without Expo
3. **Full-featured platforms** - Self-hosted but heavier (database, admin UI, etc.)

Since ruk-mobile is a **bare React Native project** (no Expo), we have flexibility to choose any approach.

---

## Options Evaluated

### Option A: expo-updates + Custom Server

**How it works**: Add `expo-updates` to the bare RN project, point it at a custom server implementing the Expo Updates Protocol.

**Architecture**:
```
┌──────────────┐     ┌─────────────────────┐     ┌──────────┐
│  ruk-mobile  │────▶│  ruk-message-hub    │────▶│    S3    │
│              │     │  /api/ota/manifest  │     │ bundles  │
│ expo-updates │     │  /api/ota/assets    │     │ & assets │
└──────────────┘     └─────────────────────┘     └──────────┘
```

**Pros**:
- Well-documented protocol (Expo maintains the spec)
- Code signing built-in for security
- Battle-tested in production by Expo
- Reference implementations available ([expo/custom-expo-updates-server](https://github.com/expo/custom-expo-updates-server))

**Cons**:
- Requires adding Expo modules to bare RN project (~30 min setup)
- More complex manifest format
- Protocol is designed for Expo's needs (some overkill for simple use cases)

**Complexity**: Medium-High
**Implementation time**: 2-3 days

---

### Option B: react-native-ota-hot-update (Recommended)

**How it works**: Lightweight library that intercepts bundle loading at the native level. You host bundles anywhere (S3, custom server) and the library handles download/installation/rollback.

**Architecture**:
```
┌──────────────┐     ┌─────────────────────┐     ┌──────────┐
│  ruk-mobile  │────▶│  ruk-message-hub    │────▶│    S3    │
│              │     │  /api/ota/version   │     │ bundles  │
│   ota-lib    │     │  (returns S3 URL)   │     │          │
└──────────────┘     └─────────────────────┘     └──────────┘
```

**Pros**:
- **Minimal footprint** - just a native module, no Expo required
- Dead simple API: check version → download bundle → apply
- Automatic rollback on crash
- Works with existing ruk-message-hub/S3 infrastructure
- Actively maintained (supports RN 0.83+)

**Cons**:
- No built-in code signing (though you can add your own hash verification)
- Less sophisticated than expo-updates (but simpler = fewer things to break)
- You own the version management logic

**Complexity**: Low-Medium
**Implementation time**: 1-2 days

**Server endpoint** (would add to ruk-message-hub):
```javascript
// GET /api/ota/version?platform=ios&currentVersion=1.0.0
{
  "version": "1.0.1",
  "bundleUrl": "https://s3.amazonaws.com/fractallabs-ota/ruk-mobile/ios/1.0.1.bundle",
  "hash": "sha256-abc123...",  // optional integrity check
  "mandatory": false
}
```

---

### Option C: Hot Updater

**How it works**: Full-featured self-hosted platform with plugin architecture (storage: S3/Supabase/R2, database: Postgres/Supabase).

**Pros**:
- Web console for managing updates
- Semantic versioning support
- Multiple platform/environment support
- Very polished

**Cons**:
- More infrastructure to maintain (needs database for metadata)
- Plugin system adds complexity
- Overkill for a single app with simple needs

**Complexity**: High
**Implementation time**: 3-5 days

---

### Option D: Xavia OTA / Expo Open OTA

**How it works**: Full Next.js server implementing the expo-updates protocol with admin dashboard, rollback support, analytics.

**Pros**:
- Production-ready (creators use it in production)
- Admin dashboard
- Rollback mechanism built-in

**Cons**:
- Requires deploying and maintaining a separate Next.js server
- Still requires adding expo-updates to ruk-mobile
- More infrastructure than we need

**Complexity**: High
**Implementation time**: 3-5 days

---

## Recommendation: Option B

**react-native-ota-hot-update** is the sweet spot for ruk-mobile because:

1. **Minimal disruption** - Doesn't require adding Expo to the project
2. **Leverages existing infra** - Just add one endpoint to ruk-message-hub, bundles go to S3
3. **Simple mental model** - Version check → download → apply → done
4. **Built-in safety** - Automatic rollback if a bad bundle crashes the app
5. **Fast to implement** - Can have a working prototype in a few hours

### Implementation Plan

**Phase 1: Infrastructure (ruk-message-hub)**
1. Add database table for bundle versions:
   ```sql
   CREATE TABLE ota_bundles (
     id SERIAL PRIMARY KEY,
     platform VARCHAR(10) NOT NULL,  -- 'ios' | 'android'
     version VARCHAR(20) NOT NULL,
     bundle_url TEXT NOT NULL,
     hash VARCHAR(64),
     mandatory BOOLEAN DEFAULT FALSE,
     released_at TIMESTAMP DEFAULT NOW()
   );
   ```
2. Add endpoint: `GET /api/ota/version`
3. Add endpoint: `POST /api/ota/bundles` (for publishing new bundles)

**Phase 2: Build Pipeline**
1. Create bundle script: `npx react-native bundle --platform ios ...`
2. Upload to S3 with versioned path
3. Register bundle via POST to ruk-message-hub

**Phase 3: Client Integration**
1. Install library: `yarn add react-native-ota-hot-update react-native-blob-util`
2. Modify AppDelegate (iOS) and MainApplication (Android) per docs
3. Add update check on app launch
4. Optional: Add UI for "Update available" prompt

### Example Client Code

```typescript
import OtaHotUpdate from 'react-native-ota-hot-update';
import { Platform } from 'react-native';

async function checkForUpdates() {
  const currentVersion = await OtaHotUpdate.getCurrentVersion();

  const response = await fetch(
    `https://ruk-message-hub.herokuapp.com/api/ota/version?` +
    `platform=${Platform.OS}&currentVersion=${currentVersion}`
  );

  const { version, bundleUrl, mandatory } = await response.json();

  if (version && version !== currentVersion) {
    if (mandatory) {
      // Force update
      await OtaHotUpdate.downloadBundleUri(bundleUrl, version);
      OtaHotUpdate.restartApp();
    } else {
      // Prompt user
      showUpdatePrompt(() => {
        OtaHotUpdate.downloadBundleUri(bundleUrl, version);
        OtaHotUpdate.restartApp();
      });
    }
  }
}
```

---

## Comparison Matrix

| Criterion | expo-updates | ota-hot-update | Hot Updater | Xavia OTA |
|-----------|--------------|----------------|-------------|-----------|
| Requires Expo | Yes | No | No | Yes |
| Code signing | Built-in | DIY | Built-in | Built-in |
| Admin UI | No | No | Yes | Yes |
| Database needed | No | No | Yes | Yes |
| Rollback | Manual | Automatic | Manual | Yes |
| Complexity | Medium | Low | High | High |
| Setup time | 2-3 days | 1-2 days | 3-5 days | 3-5 days |

---

## Security Considerations

Whichever approach we choose, we should implement:

1. **HTTPS only** - All bundle URLs must be HTTPS
2. **Hash verification** - Include SHA-256 hash in version response, verify before applying
3. **API authentication** - The publish endpoint needs auth (can use existing ruk-message-hub auth)
4. **Rollback trigger** - Automatic (ota-hot-update has this) or manual via version endpoint

---

## Sources

- [Hot Updater](https://github.com/gronxb/hot-updater) - Self-hostable OTA with plugin architecture
- [Xavia OTA](https://github.com/xavia-io/xavia-ota) - Self-hosted expo-updates server
- [Expo Open OTA](https://github.com/axelmarciano/expo-open-ota) - Go-based expo-updates server
- [react-native-ota-hot-update](https://github.com/vantuan88291/react-native-ota-hot-update) - Lightweight OTA library
- [Expo Updates Protocol Spec](https://docs.expo.dev/technical-specs/expo-updates-1/)
- [Custom Expo Updates Server](https://github.com/expo/custom-expo-updates-server) - Reference implementation
- [Installing expo-updates in bare RN](https://docs.expo.dev/bare/installing-updates/)

---

## Next Steps

If this approach sounds good:
1. I can create the ruk-message-hub endpoints
2. Set up the S3 bucket structure
3. Add the client library to ruk-mobile
4. Create a simple publish script

Let me know which direction you want to go, or if you want me to dig deeper into any of these options.

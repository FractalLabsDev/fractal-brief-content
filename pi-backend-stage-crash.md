
# pi-backend-stage Crash Report

**Status:** CRASHED (H10)  
**Since:** Feb 3, 2026 ~09:07 UTC  
**Last Deploy:** v57 by Max on Feb 2, 15:11 MST

---

## Root Cause

```
TypeError: diagChan.tracingChannel is not a function
    at Object.<anonymous> (/app/node_modules/pino/lib/tools.js:32:29)
```

The **pino** logging library is using `diagnostics_channel.tracingChannel()`, which was added in **Node.js 19.9.0 / 20.3.0**.

The app is running **Node.js v18.17.0**, which doesn't have this API.

---

## Why It Happened

A newer version of `pino` (likely 9.x) was installed that requires Node 20+. The app worked for ~12 hours after deploy before crashing, suggesting the dyno cycled and pulled fresh dependencies from the lockfile.

---

## Fix Options

### Option 1: Upgrade Node (Recommended)
Update `package.json`:
```json
"engines": {
  "node": "20.x"
}
```
Then redeploy.

### Option 2: Pin pino to Node 18-compatible version
```bash
npm install pino@8.21.0
```
Pino 8.x supports Node 18.

---

## Affected Services

- **pi-backend-stage**: CRASHED
- **talkwise-backend-stage**: Running fine (unaffected)

---

## Timeline

| Time (UTC) | Event |
|------------|-------|
| Feb 2, 22:11 | v57 deployed |
| Feb 3, 09:07 | First crash observed |
| Feb 3, 09:07 | Restart attempt #1 failed |
| Feb 3, 14:43 | Restart attempt #2 failed |
| Feb 3, 20:11 | Restart attempt #3 failed |
| Feb 4, 00:52 | Restart attempt #4 failed |

All restart attempts fail with the same `tracingChannel` error.


# Unified Logging Standard for Fractal Labs Backend Repos

**Established**: 2026-02-02
**Status**: Active
**Applies to**: All Node.js backend repositories

---

## Standard: Pino Logger

All Fractal Labs backend repositories use **Pino** as the standard logging library.

**Rationale**:
- Fastest Node.js logger (performance benchmarks)
- Production-ready structured JSON logging
- Environment-based controls for log volume management
- Consistent across talkwise-sentinel, talkwise-warden, practice-interviews-backend, ruk-message-hub

---

## Installation

```bash
npm install pino
npm install --save-dev pino-pretty
```

---

## Standard Logger Configuration

### For CommonJS (Node.js)

**File**: `core/logger.js` or `src/lib/logger.js`

```javascript
const pino = require('pino');

const isDevelopment = process.env.NODE_ENV === 'development';

const logger = pino({
    level: process.env.LOG_LEVEL || 'info',
    transport: isDevelopment
        ? {
            target: 'pino-pretty',
            options: {
                colorize: true,
                translateTime: 'HH:MM:ss Z',
                ignore: 'pid,hostname',
            },
        }
        : undefined,
    base: {
        env: process.env.NODE_ENV || 'development',
        service: 'your-service-name',
    },
});

module.exports = logger;
```

### For TypeScript

**File**: `src/lib/logger.ts` or `src/utils/logger.ts`

```typescript
import pino from 'pino';

const isDevelopment = process.env.NODE_ENV === 'development';

export const logger = pino({
    level: process.env.LOG_LEVEL || 'info',
    transport: isDevelopment
        ? {
            target: 'pino-pretty',
            options: {
                colorize: true,
                translateTime: 'HH:MM:ss Z',
                ignore: 'pid,hostname',
            },
        }
        : undefined,
    base: {
        env: process.env.NODE_ENV || 'development',
        service: 'your-service-name',
    },
});
```

---

## Usage Patterns

### Import Logger

```javascript
// CommonJS
const logger = require('./core/logger');

// ES Modules / TypeScript
import { logger } from './lib/logger';
```

### Logging with Context (Recommended - Pino Style)

**Best practice**: Context object first, message string second.

```javascript
// ✅ Correct - Pino style (context is captured)
logger.info({ userId: '123', action: 'login' }, 'User logged in');
logger.debug({ orderId: '456', total: 99.99 }, 'Order created');
logger.error({ error, context: 'payment' }, 'Payment failed');

// ❌ Not recommended - Winston style (context ignored)
logger.info('User logged in', { userId: '123', action: 'login' });
```

**Why**: Pino's API places context objects first to enable proper structured logging. Context is included in JSON output for log aggregation and searching.

### Log Levels

Use appropriate log levels:

- `logger.trace()` - Extremely detailed debugging (rarely used)
- `logger.debug()` - Normal operations, decision points, data flow
- `logger.info()` - Important events, state changes, successes
- `logger.warn()` - Unusual conditions, degraded performance
- `logger.error()` - Failures, exceptions

**Default levels**:
- Development: `debug` (see everything)
- Production: `info` (important events only)

**Override via environment**:
```bash
LOG_LEVEL=debug npm start  # See all debug logs
LOG_LEVEL=warn npm start   # Only warnings and errors
```

---

## Environment Variables

### Required

- `NODE_ENV` - Determines development vs production mode
  - `development` - Enables pino-pretty colorized output
  - `production` - Structured JSON logging (no pretty printer)

### Optional

- `LOG_LEVEL` - Override default log level
  - Valid values: `trace`, `debug`, `info`, `warn`, `error`, `fatal`
  - Development default: `info`
  - Production default: `info`

Example `.env` file:
```env
NODE_ENV=development
LOG_LEVEL=debug
```

---

## Production Configuration

### What Changes in Production

1. **No pino-pretty**: Removes colorization and formatting overhead
2. **Structured JSON**: Every log line is valid JSON for log aggregation
3. **Higher log level**: Defaults to `info` (filters out debug logs)

### Log Aggregation Services

Pino's JSON format is compatible with:
- **Heroku Papertrail**
- **Datadog**
- **Sentry**
- **LogRocket**
- **CloudWatch Logs**
- **Elasticsearch/Kibana**

---

## Advanced Patterns

### Sequelize Database Logging

Control SQL query logging in database config:

```javascript
// config/database.js
const logger = require('../core/logger');

module.exports = {
    development: {
        // Log SQL queries at debug level
        logging: (msg) => logger.debug({ sequelize: true }, msg),
    },
    production: {
        // Disable SQL query logging in production
        logging: false,
    },
};
```

### Request/Response Logging Middleware

For Express apps, add middleware to log HTTP requests:

```javascript
const logger = require('./core/logger');

function requestLogger(req, res, next) {
    const { method, path } = req;

    // Skip health checks and static assets
    if (path.includes('/health') || path.match(/\.(js|css|ico|png|jpg|gif|svg)$/)) {
        return next();
    }

    const startTime = Date.now();

    logger.debug({ method, path }, 'Incoming request');

    res.on('finish', () => {
        const duration = Date.now() - startTime;
        logger.info(
            { method, path, statusCode: res.statusCode, durationMs: duration },
            `${method} ${path} ${res.statusCode} - ${duration}ms`
        );
    });

    next();
}

// Apply before other middleware
app.use(requestLogger);
```

### Error Handler Integration

Integrate Pino into error handling middleware:

```javascript
const logger = require('./core/logger');

function errorHandler(err, req, res, next) {
    logger.error(
        {
            err,
            message: err.message,
            stack: err.stack,
            path: req.path,
            method: req.method,
        },
        'Request error'
    );

    res.status(err.status || 500).json({ error: err.message });
}

app.use(errorHandler);
```

---

## Migration from Winston

### API Compatibility

Pino supports Winston-style calls for backward compatibility:

```javascript
// Winston style (works but context not captured)
logger.info('Message', { context });

// Pino style (preferred)
logger.info({ context }, 'Message');
```

### Migration Steps

1. **Install Pino**: `npm install pino && npm install --save-dev pino-pretty`
2. **Remove Winston**: `npm uninstall winston`
3. **Replace logger.js**: Use standard Pino configuration above
4. **Test**: Verify logger initialization works
5. **Refactor calls** (optional): Move to Pino-style for proper context capture

### Example PR

See reference implementations:
- [practice-interviews-backend#96](https://github.com/FractalLabsDev/practice-interviews-backend/pull/96)
- [ruk-message-hub#12](https://github.com/FractalLabsDev/ruk-message-hub/pull/12)

---

## Log Volume Optimization

### Problem: Heroku Log Limits

Many repos hit 100% of Heroku's log plan due to:
- Excessive `info` level logging in normal operations
- Unfiltered health checks and static asset requests
- SQL query logging in production
- No environment-based log level control

### Solution: Discriminate Log Levels

**Move verbose operations from `info` → `debug`:**

```javascript
// ❌ Before (generates too many logs)
logger.info('Processing message', { messageId });
logger.info('Database lookup', { query });
logger.info('API call started', { endpoint });

// ✅ After (debug in dev, filtered in prod)
logger.debug({ messageId }, 'Processing message');
logger.debug({ query }, 'Database lookup');
logger.debug({ endpoint }, 'API call started');

// Keep info for important events
logger.info({ messageId, status: 'completed' }, 'Message processed successfully');
```

**Expected impact**: 70-80% log volume reduction in production.

---

## Testing

### Basic Test

```bash
node -e "const logger = require('./core/logger'); logger.info('Test log');"
```

### Development Mode

```bash
NODE_ENV=development node -e "const logger = require('./core/logger'); logger.info('Pretty log');"
```

### Debug Level

```bash
LOG_LEVEL=debug node index.js
```

---

## Repository Status

| Repository | Status | PR | Notes |
|------------|--------|-----|-------|
| talkwise-sentinel | ✅ Adopted | N/A | Original implementation by Serhii |
| talkwise-warden | ✅ Adopted | N/A | Original implementation by Serhii |
| practice-interviews-backend | ✅ Adopted | #96 | Merged by Max |
| ruk-message-hub | ✅ Adopted | #12 | By Ruk |
| vitaboom | ⏳ Pending | - | To be migrated |
| fractal-brief | ⏳ Pending | - | To be migrated |
| talkwise-infra | ⏳ Pending | - | To be migrated |

---

## References

- **Pino Documentation**: https://getpino.io/
- **Best Practices**: https://getpino.io/#/docs/best-practices
- **Performance Benchmarks**: https://getpino.io/#/docs/benchmarks
- **Research Document**: `RUK/LOGS/2026-02-02T1402 - logging-strategy-research.md`

---

**Questions or issues?** Contact @austin or @ruk on Slack.

---

*Updated: 2026-02-02*
*Version: 1.0*

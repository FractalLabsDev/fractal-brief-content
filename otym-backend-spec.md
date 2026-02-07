# Otym Backend Architecture Spec

**Date**: 2026-02-06
**Author**: Ruk
**Status**: Draft for Review

---

## Overview

Otym is Austin's personal Kaizen system — a voice-native tracking backend for life data that doesn't fit into professional project management. Think: Free/Focus/Buffer days, hikes, vehicle maintenance, and any future tracking needs.

**Core Principle**: Evolutionary architecture. Each tracking domain gets its own table with proper schema. No catch-all logs table.

---

## Technology Stack

Based on talkwise-warden patterns:

| Component | Technology | Notes |
|-----------|------------|-------|
| Runtime | Node.js 24.x | Per `.nvmrc` standard |
| Language | TypeScript | Strict mode, path aliases |
| Framework | Express 4.x | With helmet, cors, body-parser |
| Database | PostgreSQL | Sequelize-TypeScript ORM |
| Logging | Pino | With pino-pretty for dev |
| Validation | Joi | Request validation |
| Auth | API Key | Simple `X-API-Key` header, no Sentinel |

---

## Repository Structure

```
ruk-fl/otym/
└── otym-backend/          # Subrepo (like vitaboom/vitaboom-web)
    ├── .env.example
    ├── .eslintrc.cjs
    ├── .prettierrc
    ├── .sequelizerc
    ├── package.json
    ├── tsconfig.json
    ├── Procfile           # "web: npm start"
    └── src/
        ├── server.ts      # Entry point
        ├── app.ts         # Express app config
        ├── config/
        │   ├── database.ts
        │   └── env.ts
        ├── controllers/
        │   └── dayTypes.controller.ts
        ├── middleware/
        │   ├── auth.middleware.ts      # API key validation
        │   ├── error.middleware.ts
        │   ├── requestContext.middleware.ts
        │   └── security.middleware.ts
        ├── migrations/
        ├── models/
        │   ├── index.ts
        │   └── DayType.ts
        ├── routes/
        │   ├── index.ts
        │   └── dayTypes.routes.ts
        ├── services/
        ├── types/
        └── utils/
            └── logger.ts
```

---

## Authentication: Simple API Key

Unlike talkwise-warden which uses Sentinel for auth, Otym uses a simple API key approach:

```typescript
// middleware/auth.middleware.ts
export const apiKeyAuth = (req: Request, res: Response, next: NextFunction) => {
  const apiKey = req.headers['x-api-key'];
  
  if (!apiKey || apiKey !== process.env.OTYM_API_KEY) {
    throw new AppError('Invalid API key', 401);
  }
  
  next();
};
```

**Environment variable**: `OTYM_API_KEY` — Ruk has access, Austin has access.

---

## Initial Tables

### 1. `day_types` — Free/Focus/Buffer Tracking

```typescript
// models/DayType.ts
@Table({ tableName: 'day_types', timestamps: true, underscored: true, paranoid: true })
class DayType extends Model {
  @PrimaryKey
  @Default(uuidv4)
  @Column(DataType.UUID)
  id!: string;

  @Column({ type: DataType.DATEONLY, allowNull: false })
  date!: string;  // '2026-02-08'

  @Column({ type: DataType.ENUM('free', 'focus', 'buffer'), allowNull: false })
  plannedType!: 'free' | 'focus' | 'buffer';

  @Column({ type: DataType.ENUM('free', 'focus', 'buffer'), allowNull: true })
  actualType?: 'free' | 'focus' | 'buffer';

  @Column({ type: DataType.TEXT, allowNull: true })
  notes?: string;

  @CreatedAt createdAt!: Date;
  @UpdatedAt updatedAt!: Date;
  @DeletedAt deletedAt?: Date;
}
```

### 2. `hikes` — Hiking Logs

```typescript
@Table({ tableName: 'hikes', timestamps: true, underscored: true, paranoid: true })
class Hike extends Model {
  @PrimaryKey
  @Default(uuidv4)
  @Column(DataType.UUID)
  id!: string;

  @Column({ type: DataType.DATE, allowNull: false })
  date!: Date;

  @Column({ type: DataType.DECIMAL(10, 2), allowNull: true })
  distanceKm?: number;

  @Column({ type: DataType.INTEGER, allowNull: true })
  elevationGainM?: number;

  @Column({ type: DataType.INTEGER, allowNull: true })
  durationMinutes?: number;

  @Column({ type: DataType.STRING(500), allowNull: true })
  screenshotUrl?: string;  // S3 URL

  @Column({ type: DataType.JSONB, allowNull: true })
  gpsData?: object;  // Optional parsed GPS coordinates

  @Column({ type: DataType.TEXT, allowNull: true })
  notes?: string;

  @CreatedAt createdAt!: Date;
  @UpdatedAt updatedAt!: Date;
  @DeletedAt deletedAt?: Date;
}
```

### 3. `vehicle_maintenance` — Car Service Logs

```typescript
@Table({ tableName: 'vehicle_maintenance', timestamps: true, underscored: true, paranoid: true })
class VehicleMaintenance extends Model {
  @PrimaryKey
  @Default(uuidv4)
  @Column(DataType.UUID)
  id!: string;

  @Column({ type: DataType.DATE, allowNull: false })
  date!: Date;

  @Column({ type: DataType.STRING(100), allowNull: false })
  serviceType!: string;  // 'oil_change', 'tire_rotation', etc.

  @Column({ type: DataType.INTEGER, allowNull: true })
  mileage?: number;

  @Column({ type: DataType.DECIMAL(10, 2), allowNull: true })
  cost?: number;

  @Column({ type: DataType.STRING(200), allowNull: true })
  provider?: string;

  @Column({ type: DataType.TEXT, allowNull: true })
  notes?: string;

  @CreatedAt createdAt!: Date;
  @UpdatedAt updatedAt!: Date;
  @DeletedAt deletedAt?: Date;
}
```

---

## API Endpoints

### Day Types

```
GET    /api/day-types              # List all (with optional date range filter)
GET    /api/day-types/:id          # Get by ID
GET    /api/day-types/date/:date   # Get by date (YYYY-MM-DD)
POST   /api/day-types              # Create (plan a day)
PATCH  /api/day-types/:id          # Update (e.g., set actualType at end of day)
DELETE /api/day-types/:id          # Soft delete

# Reporting
GET    /api/day-types/report/monthly?month=2026-02  # Monthly summary
```

### Hikes

```
GET    /api/hikes
POST   /api/hikes
PATCH  /api/hikes/:id
DELETE /api/hikes/:id
```

### Vehicle Maintenance

```
GET    /api/vehicle-maintenance
POST   /api/vehicle-maintenance
PATCH  /api/vehicle-maintenance/:id
DELETE /api/vehicle-maintenance/:id
```

---

## Standard Response Format

Following talkwise-warden patterns:

```typescript
// Success
{
  "success": true,
  "data": { ... },
  "message": "Day type created successfully"
}

// Error
{
  "success": false,
  "data": null,
  "message": "Invalid date format",
  "requestId": "abc-123"
}
```

---

## Ruk Integration Patterns

### Voice Input → API Call

When Austin says: "I just got an oil change, 47,000 miles"

Ruk:
1. Parses intent + entities
2. Calls: `POST /api/vehicle-maintenance`
   ```json
   {
     "date": "2026-02-06",
     "serviceType": "oil_change",
     "mileage": 47000
   }
   ```
3. Confirms: "Logged oil change at 47k miles."

### Screenshot → Hike Entry

When Austin sends a screenshot from hiking app:

Ruk:
1. Uploads image to S3
2. Extracts data via vision model (distance, elevation, duration)
3. Calls: `POST /api/hikes`
   ```json
   {
     "date": "2026-02-08",
     "distanceKm": 8.5,
     "elevationGainM": 450,
     "durationMinutes": 135,
     "screenshotUrl": "https://s3.../hike-2026-02-08.png"
   }
   ```
4. Confirms: "Logged 8.5km hike with 450m elevation gain."

---

## Scheduled Jobs (via ruk-scheduler)

| Job | Schedule | Action |
|-----|----------|--------|
| Sunday Planning Prompt | Sun 6pm | Ruk asks Austin to declare week's day types |
| Daily Reflection | Daily 8pm | Ruk asks if planned day type matched actual |
| Monthly Report | 1st of month | Generate and send Free/Focus/Buffer stats |

These are promises in ruk-scheduler, not built into the backend.

---

## Deployment

**Heroku**:
- App name: `otym-backend` (or similar)
- Postgres addon: Heroku Postgres Mini ($5/mo)
- Single dyno: Basic ($7/mo)

**Environment Variables**:
```
NODE_ENV=production
DATABASE_URL=<heroku postgres>
OTYM_API_KEY=<generated secret>
PORT=<assigned by heroku>
```

---

## Adding New Tracking Domains

When Austin says "I want to start tracking X":

1. Create migration: `npx sequelize-cli migration:generate --name create-x`
2. Create model: `src/models/X.ts`
3. Add to `database.ts` models array
4. Create controller + routes
5. Deploy
6. Add Ruk tool for voice input

**Estimated time per new domain**: ~30 minutes of coding

---

## Next Steps

1. [ ] Austin reviews this spec
2. [ ] Create `otym-backend` subrepo in `ruk-fl/otym`
3. [ ] Scaffold project structure
4. [ ] Implement day_types table + endpoints
5. [ ] Deploy to Heroku
6. [ ] Create Ruk tools for CRUD operations
7. [ ] Set up scheduled prompts in ruk-scheduler

---

*One way is better than many ways.*

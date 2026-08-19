# PACE OS - Health IoT Platform
> A real-time health monitoring & smart alarm platform. Frontend is a smartwatch OS, backend is a time-series IoT ingestion system.

### Live Demo
Frontend: PACE OS Dashboard (React + Canvas ECG for heart + glucose tracking)

### Why this is a Backend Project
This is NOT a digital clock. It's a wearable health platform simulation:
- Devices push heart rate / glucose readings every second
- Backend must ingest, validate, store, and stream 1000+ req/sec
- Alarms must fire reliably even when user is offline
- Data must be aggregated for 24h/7d charts without slowing down

### Architecture
[Wearable Device] --HTTP--> [API Gateway (Fastify)] --> [Redis Queue] --> [Worker: Alarm Scheduler] --> [Push Service]
[Frontend PACE OS] <--WebSocket-- [Realtime Service] <-- [TimescaleDB]

### Features
- Clock Service: Timezone-aware, DST handling (Local, UTC, NYC, LAX, LDN, TYO)
- Alarm Service: Cron-based, repeat logic, snooze with idempotency
- Ingestion API: POST /api/v1/readings { deviceId, bpm, glucose, timestamp }
- Health Analytics: GET /api/v1/health/summary?range=24h
- Realtime: WebSocket live ECG + glucose stream

### Tech Stack
- Runtime: Node.js + Fastify (or Go + Gin)
- Database: PostgreSQL + TimescaleDB for time-series
- Cache/Queue: Redis + BullMQ
- Realtime: WebSockets
- Infra: Docker, docker-compose

### Database Schema
CREATE TABLE health_readings (
  time TIMESTAMPTZ NOT NULL,
  user_id UUID NOT NULL,
  device_id TEXT,
  heart_rate INT,
  glucose INT,
  spo2 INT
);
SELECT create_hypertable('health_readings', 'time');

CREATE TABLE alarms (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID NOT NULL,
  time TIME NOT NULL,
  timezone TEXT NOT NULL DEFAULT 'Africa/Lagos',
  label TEXT,
  repeat_days INT[],
  is_active BOOLEAN DEFAULT true,
  last_fired_at TIMESTAMPTZ
);

### API Endpoints
POST /api/v1/alarms - Create alarm
GET /api/v1/alarms - List user alarms
POST /api/v1/readings - Ingest health data
GET /api/v1/health/live - WebSocket stream
GET /api/v1/health/summary - Aggregated analytics

### How to Run
docker-compose up -d
npm run dev

API at http://localhost:3000
Frontend at http://localhost:5173

### Roadmap
- Add JWT Auth + RBAC
- Add rate limiting (100 req/min per device)
- Implement TimescaleDB continuous aggregates
- Add tests for alarm DST edge case

### Author
Philip Amadi - Backend Engineer (Jos, NG - Open to Remote)
GitHub: Pacethel-group
Stack: Node.js, Go, PostgreSQL, TimescaleDB, Redis, Docker, WebSockets

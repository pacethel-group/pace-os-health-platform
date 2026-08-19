# pace-os-health-platform
Health IoT Platform - Real-time ingestion API (Node/Go + Postgres + TimescaleDB) with smartwatch OS frontend

PACE OS - Health IoT Platform
A real-time health monitoring & smart alarm platform. Frontend is a smartwatch OS, backend is a time-series IoT ingestion system.

Stack DB Realtime

Live Demo
Frontend: PACE OS Dashboard - The clock you saw (React + Canvas ECG)

Why this project?
This is NOT a digital clock. It's a simulation of a wearable health platform:

Devices push heart rate / glucose readings every second
Backend must ingest, validate, store, and stream 1k+ req/sec
Alarms must fire reliably even when user is offline
Architecture
[Wearable Device] --MQTT/HTTP--> [API Gateway (Fastify)] --> [Redis Queue]
                                                      |
                                                      v
                                           [Worker: Alarm Scheduler]
                                                      |
[Frontend PACE OS] <--WebSocket-- [Realtime Service] <-- [TimescaleDB] 
                                      |
                                  [Push Service: FCM/Email]
Core Features Implemented
Clock Service: Timezone-aware, DST handling
Alarm Service: Cron-based, repeat logic, snooze with idempotency keys
Ingestion API: POST /api/v1/readings { deviceId, bpm, glucose, timestamp } - validated with Zod
Health Analytics: GET /api/v1/health/summary?range=24h - aggregates with TimescaleDB continuous aggregates
Tech Stack (Backend Focus)
Runtime: Node.js + Fastify OR Go + Gin
Database: PostgreSQL + TimescaleDB extension for time-series
Cache/Queue: Redis + BullMQ
Realtime: Socket.io / ws
Infra: Docker + docker-compose, GitHub Actions CI
Database Schema
sql
-- Time-series table (hypertable)
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
  id UUID PRIMARY KEY,
  user_id UUID NOT NULL,
  time TIME NOT NULL,
  timezone TEXT NOT NULL,
  label TEXT,
  repeat_days INT[], -- 0-6
  is_active BOOLEAN DEFAULT true,
  last_fired_at TIMESTAMPTZ
);
API Spec
Method	Endpoint	Description
POST	/api/v1/alarms	Create alarm
GET	/api/v1/alarms	List user alarms
POST	/api/v1/readings	Ingest health data (high throughput)
GET	/api/v1/health/live	WebSocket stream
GET	/api/v1/health/summary	Aggregated analytics
How to Run
bash
docker-compose up -d
npm run dev
# API at http://localhost:3000, Frontend at http://localhost:5173
What I'd Build Next (for interviewers)
 Add JWT Auth + RBAC
 Add rate limiting (100 req/min per device)
 Implement TimescaleDB continuous aggregates for 1h/1d rollups
 Add tests for alarm DST edge case
Author
Philip Amadi - Backend Engineer (Jos, NG - Open to Remote)


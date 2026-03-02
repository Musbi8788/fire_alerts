# 🔥 FireAlert GM

> **Real-time emergency reporting and dispatch system for The Gambia**
> Built to reduce fire service response times and save lives.

---

## Table of Contents

1. [Project Overview](#1-project-overview)
2. [The Problem](#2-the-problem)
3. [The Solution](#3-the-solution)
4. [System Architecture](#4-system-architecture)
5. [User Roles](#5-user-roles)
6. [Full System Flow](#6-full-system-flow)
7. [Project Structure](#7-project-structure)
8. [Database Design](#8-database-design)
9. [API Reference](#9-api-reference)
10. [Real-time Events](#10-real-time-events)
11. [MVP Scope](#11-mvp-scope)
12. [Full Product Scope](#12-full-product-scope)
13. [Tech Stack](#13-tech-stack)
14. [Production Architecture](#14-production-architecture)
15. [Environment Variables](#15-environment-variables)
16. [Sprint Plan](#16-sprint-plan)
17. [Setup & Installation](#17-setup--installation)
18. [Deployment](#18-deployment)
19. [SMS & Voice Integration](#19-sms--voice-integration)
20. [Security & Data Policy](#20-security--data-policy)
21. [Monitoring & Reliability](#21-monitoring--reliability)
22. [Handover & Maintenance](#22-handover--maintenance)

---

## 1. Project Overview

FireAlert GM is a Progressive Web Application (PWA) built to modernise emergency reporting and dispatch operations for the **Gambia Fire and Rescue Service (GFRS)**. It replaces the current phone-call-only system where dispatchers struggle to identify exact locations, leading to delayed responses and preventable deaths.

The system allows any person in The Gambia to submit a fire or rescue emergency with their GPS coordinates, automatically alerts the nearest available fire station in real time, and keeps the citizen informed throughout the entire response lifecycle.

| Property | Detail |
|---|---|
| **Built by** | Musbi (creator of AttendanceGM) |
| **Donated to** | Gambia Fire and Rescue Service (GFRS) |
| **Platform** | Progressive Web App — works on any phone browser, no install required |
| **Backend** | FastAPI (Python 3.11+) |
| **Frontend** | React 18 + Vite |
| **Database** | PostgreSQL 15 + PostGIS |
| **Real-time** | Socket.IO |
| **SMS** | Africa's Talking API (Gambia supported) |
| **Maps** | Leaflet.js + OpenStreetMap |
| **Status** | In development |

---

## 2. The Problem

When a fire or emergency occurs in The Gambia today:

- The citizen calls the fire service by phone
- The dispatcher tries to understand the location verbally
- Many areas have no formal addresses — only landmarks
- The wrong station may be contacted or location misunderstood
- The unit arrives late or at the wrong location
- **People die because of the delay**

There is no digital system. No location sharing. No dispatch tracking. No data logged. No confirmation to the citizen that help is even coming.

---

## 3. The Solution

FireAlert GM solves this with three connected pieces:

**For the citizen:**
- One-tap SOS button that captures GPS and fires an alert instantly
- Full form submission with emergency type, severity, landmark, and description
- SMS confirmation with tracking reference number when help is dispatched
- Live tracking screen to follow emergency status in real time

**For the dispatcher (station admin):**
- Real-time dashboard that alarms the moment an emergency is submitted
- Live map showing emergency location and all station positions
- One-click approval to dispatch a named unit/truck
- Automatic 40-second auto-dispatch if admin does not respond
- Automatic fallback to next nearest station if closest is unavailable
- Escalation to Super Admin if no station can respond

**For the fire unit:**
- SMS alert with GPS coordinates and a Google Maps navigation link
- No smartphone required — works on any basic phone via SMS
- SMS reply codes to update status from the field (Phase 2)

---

## 4. System Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                        CITIZEN                              │
│   PWA (React) — GPS capture — Emergency form — SOS button  │
└─────────────────────┬───────────────────────────────────────┘
                      │ HTTPS POST /api/emergencies
                      ▼
┌─────────────────────────────────────────────────────────────┐
│                    FASTAPI BACKEND                          │
│                                                             │
│  1. Validate & store emergency in PostgreSQL                │
│  2. PostGIS query → find nearest AVAILABLE station          │
│  3. If nearest busy → cascade to next nearest               │
│  4. Assign station, generate GM-XXXXX reference             │
│  5. Push task to Celery queue (SMS + Socket.IO)             │
│  6. Return response to citizen immediately                  │
│                                                             │
│  ┌─────────────────────────────────────────────────────┐   │
│  │  CELERY WORKER (background tasks)                   │   │
│  │  - Fire Socket.IO event → admin dashboard alarm     │   │
│  │  - Send SMS to station admin (Africa's Talking)     │   │
│  │  - Retry SMS up to 3 times on failure               │   │
│  │  - Start 40-second auto-approve countdown timer     │   │
│  │  - Log all failures to audit table                  │   │
│  └─────────────────────────────────────────────────────┘   │
└──────┬────────────────────────────┬────────────────────────┘
       │ Socket.IO                  │ SMS (Africa's Talking)
       ▼                            ▼
┌─────────────────┐      ┌──────────────────────┐
│ ADMIN DASHBOARD │      │  STATION ADMIN PHONE │
│ React PWA       │      │  Receives SMS alert  │
│ - Map view      │      │  Opens dashboard     │
│ - Alarm fires   │      └──────────────────────┘
│ - 40s countdown │
│ - Approves      │
└────────┬────────┘
         │ Admin approves OR 40s timer expires (auto-dispatch)
         ▼
┌─────────────────────────────────────────────────────────────┐
│                    FASTAPI BACKEND                          │
│  1. Update emergency status → "dispatched"                  │
│  2. Celery task: SMS citizen → confirmation + ref number    │
│  3. Celery task: SMS fire unit → GPS + Google Maps link     │
│  4. Socket.IO → citizen tracking screen updates live        │
│  5. If no station available → escalate to Super Admin       │
└─────────────────────────────────────────────────────────────┘
         │
         ▼
┌─────────────────┐
│   FIRE UNIT     │
│  Receives SMS   │
│  Opens Maps     │
│  Navigates      │
└─────────────────┘
```

---

## 5. User Roles

| Role | Description | Access |
|---|---|---|
| **Guest / Unverified** | Submits without an account. Phone number required. Rate-limited to 1 report per hour. Flagged UNVERIFIED in dashboard. | Submit emergency only |
| **Registered Citizen** | Registered with full name, phone, and national ID. Trusted reports. | Submit + track own emergencies |
| **Station Admin** | Dispatcher at a specific fire station. Receives alarms, approves dispatch. | Station dashboard + dispatch |
| **Super Admin** | National HQ. Sees all stations and emergencies. Receives escalations. | Full system access |

### Registration & Accountability Rules
- Registered citizens must provide: full name, phone number, national ID number, region
- Guest submissions require a valid phone number — flagged UNVERIFIED in dashboard
- All submissions log IP address for law enforcement use
- False emergency reporting is a criminal offence under Gambian law
- Super Admin can flag, ban, and report abusive users to authorities

---

## 6. Full System Flow

### 6.1 Emergency Submission

```
1.  Citizen opens PWA on any phone browser
2.  GPS is auto-captured on page load (prompts permission)
3.  Citizen fills form with type, severity, landmark, description
    OR taps SOS button for immediate one-tap submission
4.  Form data + GPS coordinates sent to POST /api/emergencies
5.  Backend validates: GPS bounding box check (must be within Gambia),
    phone number format, rate limit check (1/hour for guests)
6.  If guest → flag as unverified, store phone number, log IP
7.  If registered → attach verified user record
8.  PostGIS cascading query finds nearest AVAILABLE station
9.  Emergency record created in DB, status: "pending"
10. Reference number generated: GM-XXXXX (unique, sequential-random)
11. Celery tasks queued (non-blocking):
    - Socket.IO alarm fires to station admin dashboard
    - SMS sent to station admin phone simultaneously
    - 40-second auto-approve countdown timer starts
12. API responds immediately to citizen with ref number
13. Citizen sees success screen with tracking reference
```

### 6.2 Dispatch Approval (Manual + Auto-Approve)

```
MANUAL PATH:
1.  Station admin hears alarm and sees visual alert on dashboard
2.  Admin views emergency on map: type, severity, location pin,
    landmark, verified/unverified badge, countdown timer
3.  Admin clicks "Approve Dispatch", selects unit/truck name
4.  Backend records approval, cancels auto-approve timer
5.  Jump to step 6 below

AUTO-APPROVE PATH (40 seconds):
1.  Admin has not interacted with emergency after 40 seconds
2.  Celery scheduled task fires: auto-approves the emergency
3.  System selects first available unit from station records
4.  Dashboard shows "AUTO-DISPATCHED" badge on emergency card
5.  Continue to step 6 below

BOTH PATHS CONTINUE:
6.  Emergency status updated → "dispatched"
7.  Celery task: SMS to citizen
    "FIREALERT GM: Help is on the way.
     [Station name] dispatched. Ref: GM-XXXXX"
8.  Celery task: SMS to fire unit
    "FIREALERT GM DISPATCH — GM-XXXXX
     Fire at [landmark]. Navigate: maps.google.com/?q=LAT,LNG"
9.  Socket.IO event → citizen tracking screen updates in real time
10. Response record created with dispatched_at timestamp logged
```

### 6.3 Resolution

```
1.  Unit is en route — admin updates status → "en_route"
2.  Unit arrives on scene — admin updates → "on_scene"
3.  Emergency resolved — admin marks → "resolved"
4.  resolved_at timestamp recorded
5.  Full response time calculated and stored in responses table
6.  Station status automatically resets to "available"
```

### 6.4 Station Unavailable — Fallback & Escalation Chain

```
1.  PostGIS query finds nearest station
2.  Station status is "responding" or "unavailable"
3.  System excludes that station, queries next nearest
4.  Repeats until an available station is found
    (Maximum 5 cascades to prevent infinite loop)
5.  If all stations in region are unavailable:
    → Emergency escalated to Super Admin
    → Super Admin receives SMS: "No available station for GM-XXXXX
      in [region]. Manual dispatch required."
    → Super Admin dashboard alarm fires
    → Super Admin manually assigns any station nationally
```

### 6.5 SMS Failure Handling

```
1.  Africa's Talking SMS call fails (network, credits, API error)
2.  Celery automatically retries after 5 seconds
3.  Retry 2: after 15 seconds
4.  Retry 3: after 30 seconds
5.  If all 3 retries fail:
    → Failure logged to sms_failures table with full error detail
    → Sentry alert fires to developer
    → Dashboard shows warning indicator on affected emergency
    → Emergency is NOT blocked — dispatch proceeds regardless
    → Super Admin can see all SMS failures in system health view
```

---

## 7. Project Structure

```
firealert-gm/
│
├── backend/
│   ├── app/
│   │   ├── main.py                    # FastAPI app init, router registration, CORS
│   │   ├── config.py                  # Pydantic settings, loads from .env
│   │   ├── database.py                # Async SQLAlchemy engine + session factory
│   │   │
│   │   ├── models/                    # SQLAlchemy ORM table definitions
│   │   │   ├── __init__.py
│   │   │   ├── user.py                # Users table (all roles)
│   │   │   ├── station.py             # Fire stations with PostGIS location
│   │   │   ├── emergency.py           # Emergency reports
│   │   │   ├── response.py            # Response timeline per emergency
│   │   │   ├── audit_log.py           # Immutable audit trail (production)
│   │   │   └── sms_failure.py         # Failed SMS log (production)
│   │   │
│   │   ├── schemas/                   # Pydantic request + response shapes
│   │   │   ├── __init__.py
│   │   │   ├── user.py
│   │   │   ├── emergency.py
│   │   │   └── station.py
│   │   │
│   │   ├── routers/                   # FastAPI route handlers (thin layer)
│   │   │   ├── __init__.py
│   │   │   ├── auth.py                # /auth/register, /auth/login, /auth/guest-token
│   │   │   ├── emergencies.py         # /emergencies CRUD + approve + resolve
│   │   │   ├── stations.py            # /stations list + status update
│   │   │   └── admin.py               # /admin/dashboard, /admin/sms-failures
│   │   │
│   │   ├── services/                  # Business logic (all routers call these)
│   │   │   ├── __init__.py
│   │   │   ├── geo.py                 # find_nearest_available_station() PostGIS
│   │   │   ├── sms.py                 # SMS send functions + retry logic
│   │   │   ├── socket.py              # Socket.IO emit helpers
│   │   │   └── dispatch.py            # auto_approve_task(), escalation logic
│   │   │
│   │   ├── tasks/                     # Celery background task definitions
│   │   │   ├── __init__.py
│   │   │   ├── celery_app.py          # Celery instance + Redis broker config
│   │   │   ├── sms_tasks.py           # send_admin_sms, send_citizen_sms, send_unit_sms
│   │   │   └── dispatch_tasks.py      # auto_approve_countdown, escalation_task
│   │   │
│   │   └── utils/
│   │       ├── security.py            # JWT create/verify, bcrypt hash/verify
│   │       ├── helpers.py             # generate_ref_id() → "GM-XXXXX"
│   │       └── validators.py          # GPS bounding box, phone format, rate limit check
│   │
│   ├── migrations/                    # Alembic auto-generated migration files
│   │   ├── env.py
│   │   └── versions/
│   │
│   ├── seeds/
│   │   └── stations.py                # All Gambia fire stations with GPS coordinates
│   │
│   ├── tests/
│   │   ├── conftest.py                # Pytest fixtures, test DB setup
│   │   ├── test_auth.py
│   │   ├── test_emergencies.py
│   │   ├── test_geo.py                # PostGIS query tests
│   │   ├── test_dispatch.py           # Auto-approve, escalation chain tests
│   │   └── test_sms.py               # SMS retry logic tests (mocked AT)
│   │
│   ├── requirements.txt
│   ├── alembic.ini
│   ├── .env.example
│   └── Dockerfile
│
├── frontend/
│   ├── public/
│   │   ├── manifest.json              # PWA: name, icons, theme_color, display
│   │   ├── sw.js                      # Service worker (offline support — Phase 2)
│   │   └── icons/                     # PWA icons: 192x192, 512x512
│   │
│   ├── src/
│   │   ├── main.jsx
│   │   ├── App.jsx                    # Route definitions + auth guards
│   │   │
│   │   ├── pages/
│   │   │   ├── ReportEmergency.jsx    # Citizen: SOS + full form
│   │   │   ├── TrackEmergency.jsx     # Citizen: live status tracking by ref
│   │   │   ├── Login.jsx
│   │   │   ├── Register.jsx
│   │   │   ├── Dashboard.jsx          # Admin: live map + emergency queue
│   │   │   ├── EmergencyDetail.jsx    # Admin: full single emergency view
│   │   │   └── SystemHealth.jsx       # Super Admin: SMS failures, uptime stats
│   │   │
│   │   ├── components/
│   │   │   ├── SOSButton.jsx          # One-tap emergency button
│   │   │   ├── EmergencyForm.jsx      # Full submission form with GPS
│   │   │   ├── EmergencyCard.jsx      # Dashboard list item with countdown
│   │   │   ├── CountdownTimer.jsx     # 40s visual countdown on admin card
│   │   │   ├── DispatchModal.jsx      # Unit selection + approve modal
│   │   │   ├── MapView.jsx            # Leaflet map: station + emergency pins
│   │   │   ├── AlarmAlert.jsx         # Full-screen alarm popup with sound
│   │   │   ├── StatusBadge.jsx        # pending / dispatched / resolved
│   │   │   ├── VerifiedBadge.jsx      # VERIFIED / UNVERIFIED indicator
│   │   │   └── AutoDispatchBadge.jsx  # Shows when auto-approved
│   │   │
│   │   ├── hooks/
│   │   │   ├── useSocket.js           # Socket.IO connection + all event handlers
│   │   │   ├── useGPS.js              # GPS capture with accuracy + error states
│   │   │   ├── useAuth.js             # Token management + silent refresh
│   │   │   └── useCountdown.js        # 40s countdown logic for admin cards
│   │   │
│   │   ├── services/
│   │   │   ├── api.js                 # Axios instance, JWT interceptor, refresh logic
│   │   │   └── socket.js              # Socket.IO client initialisation + auth
│   │   │
│   │   └── store/
│   │       ├── authStore.js           # Zustand: user, token, role
│   │       └── emergencyStore.js      # Zustand: active emergencies list for dashboard
│   │
│   ├── index.html
│   ├── package.json
│   ├── vite.config.js
│   ├── tailwind.config.js
│   └── .env.example
│
├── docs/
│   ├── station-setup-guide.md         # Onboarding a new fire station (for GFRS)
│   ├── admin-user-guide.md            # How dispatchers use the dashboard
│   ├── api-examples.md                # For future maintainers / government IT
│   └── handover-checklist.md          # Step-by-step production handover checklist
│
├── docker-compose.yml                 # PostgreSQL + PostGIS + Redis + Backend + Frontend
├── docker-compose.prod.yml            # Production override config
└── .github/
    └── workflows/
        └── deploy.yml                 # GitHub Actions CI/CD pipeline
```

---

## 8. Database Design

### Table: `users`
```sql
id             UUID        PRIMARY KEY DEFAULT gen_random_uuid()
full_name      VARCHAR(200) NOT NULL
phone          VARCHAR(20)  UNIQUE NOT NULL
email          VARCHAR(200) UNIQUE
id_number      VARCHAR(100)                         -- National ID (registered users only)
role           VARCHAR(20)  DEFAULT 'citizen'       -- citizen | station_admin | super_admin
is_verified    BOOLEAN      DEFAULT FALSE
is_banned      BOOLEAN      DEFAULT FALSE
station_id     UUID         REFERENCES stations(id) -- station_admin only
password_hash  VARCHAR(255)
created_at     TIMESTAMP    DEFAULT NOW()
```

### Table: `stations`
```sql
id             UUID        PRIMARY KEY DEFAULT gen_random_uuid()
name           VARCHAR(200) NOT NULL
phone          VARCHAR(20)  NOT NULL               -- Unit SMS phone number
admin_phone    VARCHAR(20)  NOT NULL               -- Dispatcher phone for SMS alerts
region         VARCHAR(100) NOT NULL
address        TEXT
location       GEOGRAPHY(POINT, 4326) NOT NULL     -- PostGIS point
status         VARCHAR(20)  DEFAULT 'available'    -- available | responding | unavailable
auto_approve_timeout_seconds INT DEFAULT 40        -- Configurable per station
created_at     TIMESTAMP DEFAULT NOW()
```

### Table: `emergencies`
```sql
id                   UUID        PRIMARY KEY DEFAULT gen_random_uuid()
ref_number           VARCHAR(20)  UNIQUE NOT NULL              -- GM-XXXXX
type                 VARCHAR(50)  NOT NULL                     -- fire | rescue | hazmat | other
severity             VARCHAR(20)  NOT NULL                     -- low | medium | high | critical
location             GEOGRAPHY(POINT, 4326)                    -- nullable if GPS denied
landmark             TEXT                                       -- always required
region               VARCHAR(100) NOT NULL
description          TEXT
status               VARCHAR(30)  DEFAULT 'pending'            -- pending | dispatched | on_scene | resolved | escalated
submitted_by         UUID         REFERENCES users(id)         -- null for guest
submitter_phone      VARCHAR(20)  NOT NULL                     -- always stored for SMS
is_verified_report   BOOLEAN      DEFAULT FALSE
assigned_station_id  UUID         REFERENCES stations(id)
was_auto_dispatched  BOOLEAN      DEFAULT FALSE                 -- true if 40s timer fired
submitter_ip         INET                                       -- logged for law enforcement
created_at           TIMESTAMP    DEFAULT NOW()
updated_at           TIMESTAMP    DEFAULT NOW()
```

### Table: `responses`
```sql
id              UUID PRIMARY KEY DEFAULT gen_random_uuid()
emergency_id    UUID REFERENCES emergencies(id) NOT NULL
station_id      UUID REFERENCES stations(id)    NOT NULL
unit_name       VARCHAR(100)                               -- e.g. "Unit 04 - Truck KMC-007"
assigned_at     TIMESTAMP DEFAULT NOW()
dispatched_at   TIMESTAMP
en_route_at     TIMESTAMP
on_scene_at     TIMESTAMP
resolved_at     TIMESTAMP
response_time_seconds INT                                  -- calculated on resolve
notes           TEXT
```

### Table: `audit_logs` (Production — Immutable)
```sql
id           UUID      PRIMARY KEY DEFAULT gen_random_uuid()
actor_id     UUID      REFERENCES users(id)     -- who performed the action
actor_role   VARCHAR(30)
action       VARCHAR(100)                        -- e.g. "APPROVE_DISPATCH", "AUTO_DISPATCH"
target_id    UUID                                -- emergency_id or user_id affected
ip_address   INET
detail       JSONB                               -- full context snapshot
created_at   TIMESTAMP DEFAULT NOW()
```

### Table: `sms_failures` (Production)
```sql
id              UUID PRIMARY KEY DEFAULT gen_random_uuid()
emergency_id    UUID REFERENCES emergencies(id)
recipient_phone VARCHAR(20) NOT NULL
message_type    VARCHAR(50)             -- admin_alert | citizen_confirm | unit_dispatch | escalation
error_message   TEXT
attempts        INT DEFAULT 1
resolved        BOOLEAN DEFAULT FALSE
created_at      TIMESTAMP DEFAULT NOW()
```

### PostGIS — Cascading Nearest Station Query

```sql
-- Finds the nearest available station, excluding already-tried stations
SELECT
    s.id,
    s.name,
    s.admin_phone,
    s.unit_phone,
    s.auto_approve_timeout_seconds,
    ST_Distance(
        s.location,
        ST_GeogFromText('POINT(:longitude :latitude)')
    ) AS distance_meters
FROM stations s
WHERE
    s.status = 'available'
    AND s.id != ALL(:excluded_station_ids)   -- skip stations already tried
ORDER BY
    s.location <-> ST_GeogFromText('POINT(:longitude :latitude)')
LIMIT 1;
```

---

## 9. API Reference

All endpoints prefixed with `/api`. Protected routes require `Authorization: Bearer <token>`.

### Health Check
```
GET  /health
→ { "status": "ok", "db": "connected", "redis": "connected" }
```

### Authentication

#### `POST /api/auth/register`
```json
// Request
{
  "full_name": "Lamin Touray",
  "phone": "+2207001234",
  "email": "lamin@example.com",
  "id_number": "GM-1234567",
  "password": "securepassword",
  "region": "Greater Banjul Area"
}
// Response 201
{
  "id": "uuid",
  "full_name": "Lamin Touray",
  "role": "citizen",
  "is_verified": false
}
```

#### `POST /api/auth/login`
```json
// Request
{ "phone": "+2207001234", "password": "securepassword" }
// Response 200
{
  "access_token": "eyJ...",
  "refresh_token": "eyJ...",
  "token_type": "bearer",
  "role": "citizen"
}
```

#### `POST /api/auth/refresh`
```json
// Request
{ "refresh_token": "eyJ..." }
// Response 200
{ "access_token": "eyJ...", "token_type": "bearer" }
```

#### `POST /api/auth/guest-token`
```json
// Request — phone required, rate limited to 1 active token per number
{ "phone": "+2207009999" }
// Response 200
{ "access_token": "eyJ...", "token_type": "bearer", "expires_in": 3600 }
```

---

### Emergencies

#### `POST /api/emergencies`
Accepts both registered JWT and guest token. GPS coordinates validated against Gambia bounding box.
```json
// Request
{
  "type": "fire",
  "severity": "high",
  "latitude": 13.45421,
  "longitude": -16.57832,
  "landmark": "Next to Serrekunda market, near KMC roundabout",
  "region": "Greater Banjul Area",
  "description": "Two-storey building fully ablaze, people on 2nd floor",
  "submitter_phone": "+2207001234"
}
// Response 201
{
  "id": "uuid",
  "ref_number": "GM-48291",
  "status": "pending",
  "assigned_station": {
    "name": "Kanifing Fire Station",
    "distance_meters": 1240
  },
  "message": "Emergency received. Nearest station alerted."
}
```

#### `GET /api/emergencies`
Admin only. Filter params: `?status=pending&region=...&type=fire&limit=20&offset=0`

#### `GET /api/emergencies/{id}`
Full emergency detail. Admins see all fields. Citizens see only their own.

#### `GET /api/emergencies/track/{ref_number}`
Public. Citizen tracking by reference number — returns status only, no sensitive fields.
```json
{ "ref_number": "GM-48291", "status": "dispatched", "updated_at": "..." }
```

#### `PATCH /api/emergencies/{id}/approve`
Station admin only. Cancels auto-approve timer.
```json
// Request
{ "unit_name": "Unit 04 - Truck KMC-007", "notes": "Dispatching now" }
// Response 200
{ "status": "dispatched", "dispatched_at": "...", "was_auto_dispatched": false }
```

#### `PATCH /api/emergencies/{id}/status`
Station admin only. Update to `en_route`, `on_scene`, or `resolved`.
```json
{ "status": "on_scene", "notes": "Unit arrived, fire being contained" }
```

---

### Stations

#### `GET /api/stations`
Returns all stations with current status and distance from optional lat/lng params.

#### `PATCH /api/stations/{id}/status`
Station admin or super admin.
```json
{ "status": "unavailable", "reason": "Truck maintenance" }
```

---

### Admin

#### `GET /api/admin/dashboard`
Super admin only.
```json
{
  "active_emergencies": 3,
  "stations_available": 5,
  "stations_responding": 2,
  "stations_unavailable": 1,
  "total_today": 7,
  "auto_dispatched_today": 2,
  "avg_response_time_seconds": 684,
  "sms_failures_unresolved": 1
}
```

#### `GET /api/admin/sms-failures`
Super admin only. Returns unresolved SMS failures with emergency context.

#### `GET /api/admin/audit-log`
Super admin only. Immutable log of all admin actions.

---

## 10. Real-time Events

### Connection
Clients authenticate on connect:
```javascript
const socket = io(SOCKET_URL, {
  auth: { token: getAccessToken() }
});
```

### Rooms
| Room | Joined by |
|---|---|
| `station:{station_id}` | Station admin on dashboard load |
| `all_stations` | Super admin |
| `emergency:{emergency_id}` | Citizen tracking their own report |

### Server → Client Events

| Event | Room | Payload | When |
|---|---|---|---|
| `new_emergency` | `station:{id}` | Full emergency + distance_meters | Emergency assigned to station |
| `auto_approve_warning` | `station:{id}` | `{ emergency_id, seconds_remaining: 10 }` | 10 seconds before auto-dispatch |
| `emergency_auto_dispatched` | `station:{id}` | `{ emergency_id, ref_number }` | 40s timer fires |
| `emergency_dispatched` | `emergency:{id}` | `{ ref_number, station_name }` | Citizen tracking updates |
| `status_update` | `emergency:{id}` | `{ status, updated_at }` | En route / on scene / resolved |
| `escalation_received` | `all_stations` | `{ emergency_id, region }` | No station available |
| `station_status_changed` | `all_stations` | `{ station_id, status }` | Station availability changes |

### Client → Server Events

| Event | Payload | When |
|---|---|---|
| `admin_join` | `{ station_id }` | Admin dashboard mounts |
| `superadmin_join` | — | Super admin dashboard mounts |
| `citizen_track` | `{ emergency_id }` | Citizen opens tracking screen |

---

## 11. MVP Scope

The MVP is the minimum system that can save a life end-to-end. Every feature below is required before going live.

### ✅ In MVP

| Feature | Detail |
|---|---|
| Guest emergency submission | Phone required, rate-limited 1/hour per phone number, flagged UNVERIFIED |
| Registered citizen submission | Full ID on file, trusted, VERIFIED badge |
| GPS auto-capture | Browser Geolocation API, bounding box validated to Gambia |
| Landmark fallback | Required if GPS denied — never blocks submission |
| Emergency type | Fire / Rescue / Hazmat / Other |
| Severity level | Low / Medium / High / Critical |
| Find nearest available station | PostGIS cascading query, max 5 fallbacks |
| **40-second auto-approve** | Celery countdown task, fires if admin inactive |
| **Auto-dispatch badge** | Dashboard shows AUTO-DISPATCHED on affected cards |
| **Escalation to Super Admin** | If no station available — SMS + dashboard alarm |
| Real-time dashboard alarm | Socket.IO + alarm sound + visual popup |
| **10-second pre-warning** | Socket.IO warning to admin before auto-dispatch fires |
| SMS to station admin | Africa's Talking, queued via Celery |
| **SMS retry logic** | 3 attempts with 5s / 15s / 30s backoff |
| **SMS failure logging** | sms_failures table + Sentry alert |
| Admin approves dispatch | One-click with unit name input |
| SMS to citizen on dispatch | Ref number + station name confirmation |
| SMS to fire unit | Google Maps link + landmark + ref |
| Emergency tracking by ref | Public endpoint, no login required |
| Citizen live tracking screen | Socket.IO status updates |
| **Refresh token rotation** | Silent token refresh so admin sessions don't expire mid-shift |
| **Audit log** | Immutable record of every approve, auto-dispatch, escalation |
| **Health check endpoint** | `/health` checks DB + Redis connectivity |
| JWT auth + role-based routes | All roles: citizen, station_admin, super_admin |
| Station status management | Available / Responding / Unavailable |
| IP address logging | Every submission, for law enforcement |

### ❌ Not In MVP (Phase 2+)

| Feature | Phase |
|---|---|
| Voice call confirmation to citizen | Phase 2 |
| In-app turn-by-turn navigation | Phase 2 |
| Photo uploads | Phase 2 |
| Offline submission queue (IndexedDB) | Phase 2 |
| SMS reply codes from units (1/2/3) | Phase 2 |
| Multi-language: Wolof, Mandinka, Fula | Phase 3 |
| Analytics dashboard + heatmaps | Phase 3 |
| Multi-agency: Police, Ambulance | Phase 3 |
| Unit live GPS tracking | Phase 3 |
| Android APK (Capacitor) | Phase 3 |
| Automated PDF reports to government | Phase 3 |

---

## 12. Full Product Scope

### Phase 2 — Enhanced Response

**Voice call confirmation**
After dispatch, Africa's Talking Voice API calls the citizen with a recorded confirmation. Critical for low-literacy users who may struggle with SMS. Message confirms ref number, station name, and estimated arrival.

**Photo uploads**
Up to 4 photos attached at submission, stored in Cloudinary. Visible to dispatchers on emergency detail screen. Gives units better situational awareness before arrival.

**Offline submission queue**
PWA stores emergency in IndexedDB when no internet. Submits automatically when connectivity returns. Visual indicator shows "Queued — will send when online." Essential for rural Gambia connectivity gaps.

**Unit SMS reply codes**
Fire units reply to their dispatch SMS with:
- `1` → En Route
- `2` → On Scene
- `3` → Resolved
Africa's Talking webhook parses replies, updates emergency status, fires Socket.IO event to dashboard.

### Phase 3 — Intelligence & Scale

**Analytics dashboard**
Super Admin view showing: emergency hotspot heatmap across Gambia, average response time per station, peak hours by day/month, station performance rankings, emergency type breakdown. Exportable as PDF for government planning.

**Multi-language**
Citizen PWA in Wolof, Mandinka, Fula, English. Language auto-detected from browser, manually selectable. SMS messages sent in citizen's chosen language. Admin dashboard stays English.

**Multi-agency**
System routes to Police and Ambulance alongside fire. Emergency type selection determines agency. Joint dispatch view for Super Admin on major incidents.

**Unit live GPS tracking**
Units with smartphones broadcast GPS via Socket.IO. Admin map shows live unit positions. Citizen tracking screen shows unit moving toward them.

**Android APK**
PWA packaged via Capacitor for distribution to fire service units. Runs in background, receives push notifications when closed.

**Automated government reports**
Monthly PDF auto-generated and emailed to GFRS HQ. Covers usage stats, response performance, regional trends. Supports accountability and future funding decisions.

---

## 13. Tech Stack

### Backend
| Technology | Purpose |
|---|---|
| Python 3.11+ | Language |
| FastAPI | Async web framework |
| SQLAlchemy 2.0 (async) | ORM with async session |
| GeoAlchemy2 | PostGIS support in SQLAlchemy |
| SQlite3 | development |
| PostgreSQL 15 | Primary database |
| PostGIS extension | Geospatial nearest-station queries |
| Alembic | Database schema migrations |
| **Celery** | Background task queue (SMS, auto-approve, retries) |
| **Redis** | Celery broker + result backend |
| python-socketio | Socket.IO server (async) |
| python-jose | JWT token creation and verification |
| passlib + bcrypt | Password hashing (cost factor 12) |
| **SlowAPI** | API rate limiting middleware |
| httpx | Async HTTP client |
| Africa's Talking SDK | SMS and voice calls |
| **Sentry SDK** | Error monitoring and alerting |

### Frontend
| Technology | Purpose |
|---|---|
| React 18 | UI framework |
| Vite | Build tool |
| React Router v6 | Client-side routing with auth guards |
| Zustand | Global state (auth + emergencies) |
| Axios | API requests with JWT interceptor + refresh |
| Socket.IO client | Real-time events |
| Leaflet + React-Leaflet | Interactive maps |
| OpenStreetMap | Free map tiles (no API key, works in Gambia) |
| Tailwind CSS | Utility-first styling |

### Infrastructure
| Technology | Purpose |
|---|---|
| Docker + Docker Compose | Local development environment |
| Railway | Backend + PostgreSQL + Redis hosting |
| Vercel | Frontend hosting |
| **UptimeRobot** | Uptime monitoring, alerts on downtime |
| **Sentry** | Error tracking for backend |
| GitHub Actions | CI/CD — auto-deploy on push to main |
| Africa's Talking | SMS + Voice (Gambia natively supported) |
| Cloudinary | Photo uploads (Phase 2) |

---

## 14. Production Architecture

This section documents the decisions made to make FireAlert GM production-safe — not just functional in development.

### Why Celery + Redis?

In a naive implementation, SMS sends and Socket.IO events happen inline inside the API request. If Africa's Talking takes 3 seconds to respond, the citizen waits 3 seconds. If it fails, the whole request fails and the emergency may not save.

With Celery + Redis:
1. Emergency saves to database immediately
2. API responds to citizen in under 300ms
3. SMS and Socket.IO tasks run in background
4. If SMS fails, Celery retries automatically — no manual intervention
5. Failed tasks are logged, not silently lost

```
Without Celery:
POST /emergencies → [save DB] → [send SMS 3s] → [emit socket] → respond
                                     ↑
                               If this fails, request fails

With Celery:
POST /emergencies → [save DB] → [queue tasks] → respond immediately (fast)
                                      ↓
                              Celery worker picks up tasks
                              Retries on failure
                              Logs failures to DB + Sentry
```

### 40-Second Auto-Approve Design

The auto-approve countdown is implemented as a **Celery ETA task** — a delayed task scheduled to run exactly 40 seconds after emergency creation.

```python
# When emergency is created:
auto_approve_task.apply_async(
    args=[emergency_id],
    countdown=40          # Run after 40 seconds
)

# If admin approves manually before 40s:
auto_approve_task.revoke(task_id)   # Cancel the scheduled task

# If 40s passes with no admin action:
auto_approve_task runs:
  - Selects first available unit from station
  - Sets was_auto_dispatched = True
  - Sends citizen SMS
  - Sends unit SMS
  - Emits socket event with AUTO-DISPATCHED badge
  - Logs to audit_log
```

The 10-second warning fires as a Socket.IO event at the 30-second mark, giving admins a last chance to manually approve.

### SMS Retry Logic

```python
@celery_app.task(
    bind=True,
    max_retries=3,
    default_retry_delay=5
)
def send_admin_sms_task(self, emergency_id, phone, message):
    try:
        africas_talking.sms.send(message, [phone])
    except Exception as exc:
        retry_delays = [5, 15, 30]
        attempt = self.request.retries
        if attempt < 3:
            raise self.retry(exc=exc, countdown=retry_delays[attempt])
        else:
            # All retries exhausted — log failure, alert Sentry
            log_sms_failure(emergency_id, phone, str(exc), attempts=3)
            sentry_sdk.capture_exception(exc)
```

### Token Refresh Strategy

Station admins work full 8-12 hour shifts. JWT access tokens expire after 60 minutes. Without refresh logic, the admin gets silently logged out mid-shift and misses an emergency alarm.

Solution: **silent token refresh via Axios interceptor**
- Access token: 60 minutes
- Refresh token: 7 days
- Axios interceptor catches 401 responses, silently calls `/auth/refresh`, retries the original request
- If refresh token also expired: redirect to login

### Connection Pooling

```python
# database.py
engine = create_async_engine(
    settings.DATABASE_URL,
    pool_size=10,           # Max 10 persistent connections
    max_overflow=20,        # Allow 20 extra under load
    pool_pre_ping=True,     # Test connection before use
    pool_recycle=3600       # Recycle connections every hour
)
```

### GPS Input Validation

All submitted coordinates are validated against The Gambia's geographic bounding box to reject spoofed or accidental wrong-country submissions:

```python
GAMBIA_BOUNDS = {
    "lat_min": 13.065,
    "lat_max": 13.826,
    "lng_min": -16.895,
    "lng_max": -13.795
}

def validate_gambia_coords(lat: float, lng: float) -> bool:
    return (
        GAMBIA_BOUNDS["lat_min"] <= lat <= GAMBIA_BOUNDS["lat_max"]
        and GAMBIA_BOUNDS["lng_min"] <= lng <= GAMBIA_BOUNDS["lng_max"]
    )
```

---

## 15. Environment Variables

### Backend `.env`

```env
# Database
DATABASE_URL=postgresql+asyncpg://user:password@localhost:5432/firealert_db

# Redis (Celery broker)
REDIS_URL=redis://localhost:6379/0

# Security
SECRET_KEY=your-very-secret-jwt-key-minimum-32-chars-change-this
ALGORITHM=HS256
ACCESS_TOKEN_EXPIRE_MINUTES=60
REFRESH_TOKEN_EXPIRE_DAYS=7

# Africa's Talking
AT_USERNAME=your_africas_talking_username
AT_API_KEY=your_africas_talking_api_key
AT_SENDER_ID=FireAlertGM
AT_SHORTCODE=your_shortcode

# App
ENVIRONMENT=development
FRONTEND_URL=http://localhost:5173
ALLOWED_ORIGINS=http://localhost:5173,https://firealertgm.com

# Monitoring (production only)
SENTRY_DSN=https://your-sentry-dsn@sentry.io/project

# Auto-approve
AUTO_APPROVE_SECONDS=40
```

### Frontend `.env`

```env
VITE_API_BASE_URL=http://localhost:8000
VITE_SOCKET_URL=http://localhost:8000
VITE_APP_NAME=FireAlert GM
VITE_ENVIRONMENT=development
```

---

## 16. Sprint Plan

### Sprint 1 — Data Layer (Days 1–3)

**Goal:** Database running, all tables created, PostGIS query verified, audit and SMS failure tables ready.

- [ ] Set up PostgreSQL 15 + PostGIS + Redis via Docker Compose
- [ ] Create all SQLAlchemy models: user, station, emergency, response, audit_log, sms_failure
- [ ] Write and run first Alembic migration
- [ ] Write `seeds/stations.py` with all Gambia fire station GPS coordinates
- [ ] Seed database and manually verify station data on map
- [ ] Write and test `geo.py` — `find_nearest_available_station()` with exclusion list
- [ ] Write `validators.py` — Gambia bounding box, phone format

**Done when:**
```sql
SELECT name, ST_Distance(location, ST_GeogFromText('POINT(-16.578 13.454)')) as meters
FROM stations WHERE status = 'available'
ORDER BY location <-> ST_GeogFromText('POINT(-16.578 13.454)') LIMIT 1;
-- Returns correct nearest station to Serrekunda
```

---

### Sprint 2 — Core API (Days 4–7)

**Goal:** All endpoints working, auth complete, emergency submission assigns station and returns ref number.

- [ ] FastAPI app structure, config, CORS, error handlers
- [ ] `utils/security.py` — JWT create/verify, bcrypt, refresh token logic
- [ ] `utils/helpers.py` — `generate_ref_id()`
- [ ] Auth router: register, login, refresh, guest-token
- [ ] Emergency router: POST with full validation + station assignment
- [ ] Emergency router: PATCH approve, PATCH status, GET track by ref
- [ ] Stations router: list, status update
- [ ] Admin router: dashboard stats, sms-failures, audit-log
- [ ] JWT middleware + role-based dependencies
- [ ] SlowAPI rate limiting on guest submission endpoint
- [ ] `/health` endpoint checking DB + Redis
- [ ] Write Pytest tests for all routes

**Done when:**
`POST /api/emergencies` returns `{ ref_number: "GM-XXXXX", assigned_station: {...} }` in under 300ms.

---

### Sprint 3 — Celery + Alerts (Days 8–11)

**Goal:** SMS sends in background, auto-approve fires at 40s, Socket.IO alarm reaches dashboard, all failures logged.

- [ ] Set up Celery with Redis broker
- [ ] `tasks/celery_app.py` — Celery instance, task serialisation config
- [ ] `tasks/sms_tasks.py` — three SMS tasks with retry logic (5s/15s/30s)
- [ ] SMS failure logging to sms_failures table + Sentry
- [ ] `tasks/dispatch_tasks.py` — `auto_approve_countdown` Celery ETA task
- [ ] Revoke mechanism when admin manually approves
- [ ] 10-second pre-warning Socket.IO event
- [ ] Socket.IO server with room management
- [ ] `socket.py` service — all emit functions
- [ ] Wire all tasks into emergency submission and approval endpoints
- [ ] Africa's Talking sandbox account for testing
- [ ] Test full chain: submit → SMS in under 3s → socket alarm fires → 40s timer → auto-dispatch SMS

**Done when:**
Submitting an emergency triggers dashboard alarm + admin SMS within 3 seconds, AND if admin does nothing, auto-dispatches at exactly 40 seconds.

---

### Sprint 4 — Frontend (Days 12–18)

**Goal:** Full citizen and admin UI connected to real backend and Socket.IO.

- [ ] Vite + React + Tailwind + React Router project setup
- [ ] PWA manifest with correct app name, icons, theme colour
- [ ] `services/api.js` — Axios with JWT interceptor + silent refresh
- [ ] `services/socket.js` — Socket.IO client with auth token
- [ ] `hooks/useAuth.js`, `hooks/useGPS.js`, `hooks/useSocket.js`, `hooks/useCountdown.js`
- [ ] `store/authStore.js` and `store/emergencyStore.js` with Zustand
- [ ] Register + Login pages
- [ ] ReportEmergency page — SOSButton + full form + GPS capture
- [ ] TrackEmergency page — live status by ref number via Socket.IO
- [ ] Admin Dashboard — emergency queue, map, alarm popup with sound
- [ ] CountdownTimer component — visual 40s countdown on each emergency card
- [ ] DispatchModal — unit name input + approve button
- [ ] EmergencyDetail page — full info, status update buttons
- [ ] MapView — Leaflet with station pins + active emergency pins
- [ ] AlarmAlert — full-screen overlay with sound on new_emergency event
- [ ] AutoDispatchBadge — shown on auto-dispatched emergency cards
- [ ] VerifiedBadge + StatusBadge components
- [ ] SystemHealth page for super admin — SMS failures, uptime indicator

**Done when:**
Full browser flow: submit emergency → admin alarm fires with countdown → admin approves → citizen tracking screen updates live.

---

### Sprint 5 — Production Hardening + Deploy (Days 19–23)

**Goal:** System is production-safe, deployed, tested on real devices and real numbers.

- [ ] End-to-end test: full flow on real Android + real Gambian number
- [ ] Test 40s auto-approve fires correctly in production timing
- [ ] Test SMS retry: mock AT failure, confirm 3 retries, confirm failure logged
- [ ] Test escalation chain: set all stations unavailable, confirm Super Admin SMS
- [ ] Graceful error handling on all frontend pages
- [ ] Loading states, empty states, network error states
- [ ] Mobile layout tested on real low-end Android device
- [ ] Write `docker-compose.prod.yml` with proper volume mounts
- [ ] Write `Dockerfile` for backend with multi-stage build
- [ ] Set up Railway: backend + PostgreSQL + Redis services
- [ ] Configure all production environment variables in Railway
- [ ] Enable PostGIS on Railway PostgreSQL
- [ ] Deploy backend, run `alembic upgrade head`, run station seed
- [ ] Deploy frontend to Vercel
- [ ] Switch Africa's Talking to live mode
- [ ] Set up UptimeRobot monitoring on `/health` endpoint
- [ ] Connect Sentry to production backend
- [ ] Set up GitHub Actions CI/CD workflow
- [ ] Run full production test with real phones
- [ ] Write `docs/admin-user-guide.md` and `docs/station-setup-guide.md`

**Done when:**
A complete emergency is submitted on a real phone in The Gambia, dispatched, and the unit receives the Google Maps SMS — all on live production servers.

---

## 17. Setup & Installation

### Prerequisites
- Python 3.11+
- Node.js 18+
- Docker + Docker Compose
- Git

### 1. Clone

```bash
git clone https://github.com/yourusername/firealert-gm.git
cd firealert-gm
```

### 2. Start database + Redis

```bash
docker-compose up -d db redis
```

Starts PostgreSQL 15 with PostGIS on port 5432, Redis on port 6379.

### 3. Backend setup

```bash
cd backend
python -m venv venv
source venv/bin/activate         # Windows: venv\Scripts\activate
pip install -r requirements.txt

cp .env.example .env             # Fill in all values

alembic upgrade head             # Create all tables
python -m seeds.stations         # Seed fire station data

uvicorn app.main:app --reload --port 8000
```

In a second terminal, start the Celery worker:

```bash
celery -A app.tasks.celery_app worker --loglevel=info
```

Backend: `http://localhost:8000`
API docs: `http://localhost:8000/docs`
Health: `http://localhost:8000/health`

### 4. Frontend setup

```bash
cd frontend
npm install
cp .env.example .env
npm run dev
```

Frontend: `http://localhost:5173`

### 5. Full stack with Docker Compose

```bash
docker-compose up --build
# Starts: db, redis, backend, celery worker, frontend
```

---

## 18. Deployment

### Backend + Database + Redis (Railway)

1. Create a new project at [railway.app](https://railway.app)
2. Add services: PostgreSQL, Redis, and a new service from GitHub repo
3. On the PostgreSQL service, run: `CREATE EXTENSION postgis;`
4. Set all backend environment variables in Railway dashboard
5. Add `railway.toml` to repository root:

```toml
[build]
builder = "nixpacks"

[deploy]
startCommand = "alembic upgrade head && uvicorn app.main:app --host 0.0.0.0 --port $PORT"
healthcheckPath = "/health"
healthcheckTimeout = 30
restartPolicyType = "on_failure"
```

6. Add a second Railway service for the Celery worker:
```toml
# celery-worker service
[deploy]
startCommand = "celery -A app.tasks.celery_app worker --loglevel=info --concurrency=2"
```

### Frontend (Vercel)

1. Import repo at [vercel.com](https://vercel.com)
2. Framework: Vite, Root: `frontend/`
3. Add all frontend environment variables
4. Vercel auto-deploys on push to `main`

### GitHub Actions CI/CD

```yaml
# .github/workflows/deploy.yml
name: Deploy
on:
  push:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Run backend tests
        run: |
          cd backend
          pip install -r requirements.txt
          pytest tests/ -v

  deploy:
    needs: test
    runs-on: ubuntu-latest
    steps:
      - name: Deploy to Railway
        run: railway up
```

### Production Checklist

- [ ] `ENVIRONMENT=production` in backend
- [ ] `SECRET_KEY` is minimum 32 random characters
- [ ] `ALLOWED_ORIGINS` lists production frontend URL only
- [ ] `ACCESS_TOKEN_EXPIRE_MINUTES=60` (not 10080)
- [ ] Africa's Talking switched from sandbox to live
- [ ] PostGIS enabled on production database (`CREATE EXTENSION postgis`)
- [ ] All stations seeded in production database
- [ ] HTTPS enforced on all endpoints
- [ ] Sentry DSN connected and receiving events
- [ ] UptimeRobot monitoring `/health` every 5 minutes
- [ ] Redis persistence enabled (AOF) to survive restarts
- [ ] Daily database backup configured in Railway
- [ ] Celery worker deployed as separate service
- [ ] Test full emergency flow end-to-end on production

---

## 19. SMS & Voice Integration

### SMS Templates

**To station admin — on emergency submission:**
```
FIREALERT GM — NEW EMERGENCY
Type: {type} | Severity: {severity}
Location: {landmark}
GPS: maps.google.com/?q={lat},{lng}
Reporter: {VERIFIED / UNVERIFIED}
Ref: {ref_number}
Dashboard: {dashboard_url}
```

**To citizen — on dispatch approval:**
```
FIREALERT GM: Help is on the way.
{station_name} has dispatched a unit.
Reference: {ref_number}
Keep this line free.
Emergency: call 118.
```

**To fire unit — on dispatch:**
```
FIREALERT GM DISPATCH — {ref_number}
{type} — {severity}
Location: {landmark}
Navigate: maps.google.com/?q={lat},{lng}
Reporter: {caller_phone}
```

**To Super Admin — escalation (no station available):**
```
FIREALERT GM ALERT: No available station for emergency {ref_number}
in {region}. Manual dispatch required.
Dashboard: {dashboard_url}
```

### SMS Failure Behaviour

If all 3 retry attempts fail:
- Failure logged to `sms_failures` table with error detail
- Sentry alert fires to developer immediately
- Dashboard shows warning indicator on affected emergency card
- Emergency dispatch continues — SMS failure does NOT block dispatch
- Super Admin sees all unresolved failures in system health view
- GFRS IT team notified to check Africa's Talking account credits

### Voice Call (Phase 2)

Africa's Talking Voice API calls citizen after dispatch:

> *"This is FireAlert GM. Your emergency reference [GM-XXXXX] has been received. [Station name] has dispatched a unit to your location. Estimated arrival is approximately [X] minutes. Please keep this line free."*

---

## 20. Security & Data Policy

### Authentication
- All endpoints except public emergency submission and tracking require JWT
- Guest tokens expire after 1 hour, scoped to one submission only
- Access tokens: 60-minute expiry, refreshed silently via refresh token
- Refresh tokens: 7-day expiry, single-use rotation
- Passwords hashed with bcrypt, cost factor 12

### Input Security
- All coordinates validated against Gambia bounding box on submission
- Landmark and description fields HTML-sanitised to prevent XSS
- Phone numbers validated against E.164 format (+220XXXXXXX)
- Rate limit: 1 guest submission per hour per phone number (SlowAPI)
- Global API rate limit: 100 requests/minute per IP

### Data Protection
- National ID numbers stored but never returned in any API response
- GPS coordinates visible only to station admins and super admin
- Guest phone numbers visible only to admins — never publicly
- Submitter IP address stored for law enforcement use only
- All admin actions logged to immutable `audit_logs` table

### False Reports
- All submissions tied to a phone number (registered or guest)
- Registered users have national ID on file
- False emergency reporting is a criminal offence under Gambian law
- Super Admin can flag, ban, and export user data to authorities
- IP address logged on every submission

### Data Retention
- Emergency records: retained permanently for analytics and government reporting
- User accounts: retained unless deletion formally requested
- Guest submission records: retained for 2 years
- Audit logs: retained permanently — immutable
- SMS failure logs: retained for 1 year

---

## 21. Monitoring & Reliability

### Uptime Monitoring
UptimeRobot pings `/health` every 5 minutes. If it fails:
- Email + SMS alert sent to developer
- GFRS IT team notified after 15 minutes of downtime
- Escalation path documented in `docs/handover-checklist.md`

### Error Tracking (Sentry)
Sentry captures all unhandled backend exceptions. Critical alerts (SMS failures, task failures) trigger immediate notification. Monthly error report reviewed for patterns.

### Health Check Endpoint
```python
@app.get("/health")
async def health_check(db: AsyncSession = Depends(get_db)):
    try:
        await db.execute(text("SELECT 1"))
        redis_ok = await check_redis()
        return {
            "status": "ok",
            "db": "connected",
            "redis": "connected" if redis_ok else "error",
            "environment": settings.ENVIRONMENT
        }
    except Exception:
        return JSONResponse(
            status_code=503,
            content={"status": "error", "db": "disconnected"}
        )
```

### Backup Strategy
- Railway PostgreSQL: daily automated backups retained for 7 days
- Manual backup before every major deployment:
  ```bash
  pg_dump $DATABASE_URL > backup_$(date +%Y%m%d).sql
  ```
- Backups tested for restore at least once per month

### Graceful Degradation
If SMS fails → dispatch still proceeds, failure logged
If Socket.IO drops → admin can still manually check dashboard and approve
If PostGIS query times out → fallback to full table scan with ORDER BY (slower but safe)
If Redis goes down → Celery tasks fail gracefully, logged to Sentry, manual SMS option available to admin

---

## 22. Handover & Maintenance

FireAlert GM is donated free to the Gambia Fire and Rescue Service. After handover, maintenance responsibility passes to GFRS and a designated government IT team.

### Handover Requirements

**From the developer (Musbi):**
- [ ] All source code on GitHub (public repository)
- [ ] This `README.md` — full technical documentation
- [ ] `docs/station-setup-guide.md` — onboarding new stations
- [ ] `docs/admin-user-guide.md` — dispatcher daily usage guide
- [ ] `docs/api-examples.md` — reference for future developers
- [ ] `docs/handover-checklist.md` — full production go-live checklist
- [ ] Training session with GFRS dispatchers (2 hours minimum)
- [ ] 12-month support period: bug fixes and critical patches provided free

**From GFRS / Government IT team:**
- [ ] Designated IT contact assigned and trained
- [ ] Railway hosting account transferred or new account set up
- [ ] Africa's Talking account registered under GFRS business number
- [ ] SMS credits maintained (top up when balance drops below $10)
- [ ] Domain registered and pointed to Vercel frontend
- [ ] All environment variables documented and stored securely

### What the IT Team Needs to Know

**Day-to-day operations (no technical skill needed):**
- Admin dashboard is a website — open in any browser
- Station admins log in with phone + password
- If a station admin is locked out: Super Admin resets via dashboard

**Monthly tasks:**
- Check Africa's Talking SMS credit balance — top up if below $10
- Check UptimeRobot dashboard for any downtime incidents
- Review Sentry for any recurring errors

**On a new station being added:**
- Super Admin creates station record in dashboard (name, location, phone)
- Create station admin user account linked to that station
- Give dispatcher their login credentials
- Test: submit a test emergency, confirm alarm fires on their dashboard

**On a critical incident (system down):**
1. Check Railway dashboard — is the backend service running?
2. Check health endpoint: `https://your-api.railway.app/health`
3. If DB error: Railway has automatic restart — wait 2 minutes
4. If still down: contact developer via GitHub issue (response within 4 hours during 12-month support period)

### Ongoing Costs (Estimated)

| Item | Cost |
|---|---|
| Railway hosting (backend + DB + Redis) | $5–20/month |
| Vercel (frontend) | Free |
| Africa's Talking SMS | ~$0.02/SMS (bulk pricing available) |
| UptimeRobot | Free tier sufficient |
| Sentry | Free tier sufficient |
| Domain name (optional) | ~$10/year |
| **Total estimated monthly** | **$10–25/month** |

---

## Contributing

This is an open source project donated to save lives in The Gambia. Contributions are welcome.

```bash
git checkout -b feature/your-feature-name
# Make changes, write tests
# Open a pull request with clear description of the change
```

---

## License

MIT License — free to use, modify, and distribute.

---

## Contact

Built by **Musbi** — creator of AttendanceGM
For questions or contributions: open an issue on GitHub

---

*FireAlert GM — Because every second counts. 🇬🇲*

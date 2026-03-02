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
14. [Environment Variables](#14-environment-variables)
15. [Sprint Plan](#15-sprint-plan)
16. [Setup & Installation](#16-setup--installation)
17. [Deployment](#17-deployment)
18. [SMS & Voice Integration](#18-sms--voice-integration)
19. [Security & Data Policy](#19-security--data-policy)
20. [Handover & Maintenance](#20-handover--maintenance)

---

## 1. Project Overview

FireAlert GM is a Progressive Web Application (PWA) built to modernise emergency reporting and dispatch operations for the **Gambia Fire and Rescue Service (GFRS)**. It replaces the current phone-call-only system where dispatchers struggle to identify exact locations, leading to delayed responses and preventable deaths.

The system allows any person in The Gambia to submit a fire or rescue emergency with their GPS coordinates, automatically alerts the nearest available fire station in real time, and keeps the citizen informed as help is dispatched.

**Built by:** Lamin (AttendanceGM)
**Donated to:** Gambia Fire and Rescue Service
**Status:** In development

---

## 2. The Problem

When a fire or emergency occurs in The Gambia today:

- The citizen calls the fire service by phone
- The dispatcher tries to understand the location verbally
- Many areas have no formal addresses — only landmarks
- The wrong station may be contacted
- The unit may arrive late or at the wrong location
- **People die because of the delay**

There is no digital system. No location sharing. No dispatch tracking. No confirmation to the citizen that help is coming.

---

## 3. The Solution

FireAlert GM solves this with three connected pieces:

**For the citizen:**
- One-tap SOS button that captures GPS and fires an alert instantly
- Full form submission with emergency type, severity, landmark, and photo
- SMS confirmation when help is dispatched
- Tracking reference number to follow the emergency status

**For the dispatcher (station admin):**
- Real-time dashboard that alarms the moment an emergency is submitted
- Map showing the emergency location and station positions
- One-click approval to dispatch a unit
- Automatic fallback to next nearest station if the closest is unavailable

**For the fire unit:**
- SMS alert with GPS coordinates and a Google Maps link for navigation
- No smartphone required — works on basic phones via SMS

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
│  1. Validate & store emergency                              │
│  2. PostGIS query → find nearest AVAILABLE station          │
│  3. If nearest busy → find 2nd nearest (repeat until found) │
│  4. Assign station, generate reference number               │
│  5. Fire Socket.IO event → admin dashboard                  │
│  6. Send SMS to station admin phone (Africa's Talking)      │
└──────┬────────────────────────────┬────────────────────────-┘
       │ Socket.IO                  │ SMS
       ▼                            ▼
┌─────────────────┐      ┌──────────────────────┐
│ ADMIN DASHBOARD │      │  STATION ADMIN PHONE │
│ React PWA       │      │  Receives SMS alert  │
│ - Map view      │      │  Opens dashboard     │
│ - Alarm fires   │      └──────────────────────┘
│ - Approves      │
└────────┬────────┘
         │ Admin approves dispatch
         ▼
┌─────────────────────────────────────────────────────────────┐
│                    FASTAPI BACKEND                          │
│  1. Update emergency status → "dispatched"                  │
│  2. SMS to citizen → "Help is on the way, Ref: GM-XXXXX"   │
│  3. SMS to fire unit → GPS coords + Google Maps link        │
│  4. Socket.IO → citizen tracking screen updates             │
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
| **Guest / Unverified** | Anyone who submits without a registered account. Requires phone number only. Flagged as unverified in dashboard. | Submit emergency only |
| **Registered Citizen** | User who has registered with their name, phone, and national ID. Trusted reports. | Submit + track own emergencies |
| **Station Admin** | Dispatcher at a specific fire station. Receives and approves emergency alerts for their station. | Station dashboard + dispatch approval |
| **Super Admin** | National HQ. Can see all stations, all emergencies, and analytics. | Full system access |

---

## 6. Full System Flow

### 6.1 Emergency Submission

```
1. Citizen opens PWA
2. GPS is auto-captured on load
3. Citizen fills form OR taps SOS button
4. Form data + GPS sent to POST /api/emergencies
5. Backend validates submission
6. If guest → flag as unverified, require phone number
7. If registered → attach user record
8. PostGIS finds nearest available station
9. Emergency record created in DB with status: "pending"
10. Reference number generated (format: GM-XXXXX)
11. Socket.IO event fires → station admin dashboard alarms
12. SMS sent to station admin phone simultaneously
13. Citizen sees success screen with reference number
```

### 6.2 Dispatch Approval

```
1. Station admin hears alarm on dashboard
2. Admin sees emergency on map with details
3. Admin reviews: type, severity, location, verified/unverified
4. Admin clicks "Approve Dispatch"
5. Admin selects which unit/truck to send
6. Backend updates emergency status → "dispatched"
7. SMS sent to citizen: "Unit dispatched. ETA ~X mins. Ref: GM-XXXXX"
8. SMS sent to fire unit: "EMERGENCY: Fire at [landmark]. GPS: maps.google.com/...  Ref: GM-XXXXX"
9. Socket.IO event → citizen tracking screen updates
10. Response record created with dispatched_at timestamp
```

### 6.3 Resolution

```
1. Unit arrives on scene
2. Admin updates status → "on_scene"
3. Emergency resolved → admin marks "resolved"
4. resolved_at timestamp saved
5. Full response time calculated and logged
6. Analytics record updated
```

### 6.4 Station Unavailable (Fallback)

```
1. Nearest station is status: "responding" or "unavailable"
2. System automatically queries next nearest station
3. Repeats until an available station is found
4. If no stations available in region → escalate to Super Admin
5. Super Admin notified via SMS and dashboard
```

---

## 7. Project Structure

```
firealert-gm/
│
├── README.md                          # This file
│
├── docker-compose.yml                 # Runs backend + frontend + DB together
│
├── backend/                           # FastAPI application
│   ├── app/
│   │   ├── main.py                    # App entry point, router registration
│   │   ├── config.py                  # Settings from environment variables
│   │   ├── database.py                # PostgreSQL + PostGIS async connection
│   │   │
│   │   ├── models/                    # SQLAlchemy ORM models
│   │   │   ├── __init__.py
│   │   │   ├── user.py                # Users table
│   │   │   ├── station.py             # Fire stations table
│   │   │   ├── emergency.py           # Emergencies table
│   │   │   └── response.py            # Response tracking table
│   │   │
│   │   ├── schemas/                   # Pydantic request/response shapes
│   │   │   ├── __init__.py
│   │   │   ├── user.py                # UserCreate, UserOut, LoginRequest
│   │   │   ├── emergency.py           # EmergencyCreate, EmergencyOut, StatusUpdate
│   │   │   └── station.py             # StationOut, StationStatusUpdate
│   │   │
│   │   ├── routers/                   # API route handlers
│   │   │   ├── __init__.py
│   │   │   ├── auth.py                # /auth/register, /auth/login, /auth/guest-token
│   │   │   ├── emergencies.py         # /emergencies CRUD + status updates
│   │   │   ├── stations.py            # /stations list + status management
│   │   │   └── admin.py               # /admin/dashboard stats
│   │   │
│   │   ├── services/                  # Core business logic (no DB queries here)
│   │   │   ├── __init__.py
│   │   │   ├── geo.py                 # Find nearest available station via PostGIS
│   │   │   ├── sms.py                 # Africa's Talking SMS + voice integration
│   │   │   └── socket.py             # Socket.IO event emitters
│   │   │
│   │   └── utils/
│   │       ├── __init__.py
│   │       ├── security.py            # JWT creation/verification, password hashing
│   │       └── helpers.py             # Reference ID generator (GM-XXXXX), etc.
│   │
│   ├── migrations/                    # Alembic database migrations
│   │   ├── env.py
│   │   ├── script.py.mako
│   │   └── versions/
│   │       └── 001_initial_schema.py
│   │
│   ├── tests/
│   │   ├── test_auth.py
│   │   ├── test_emergencies.py
│   │   └── test_geo.py
│   │
│   ├── requirements.txt
│   ├── .env.example
│   └── Dockerfile
│
├── frontend/                          # React PWA
│   ├── public/
│   │   ├── manifest.json              # PWA install manifest
│   │   ├── sw.js                      # Service worker (offline support)
│   │   └── icons/                     # App icons (192x192, 512x512)
│   │
│   ├── src/
│   │   ├── main.jsx                   # React entry point
│   │   ├── App.jsx                    # Routes + auth guard
│   │   │
│   │   ├── pages/
│   │   │   ├── ReportEmergency.jsx    # Citizen submission page (SOS + form)
│   │   │   ├── TrackEmergency.jsx     # Citizen tracks their submission
│   │   │   ├── Login.jsx              # Auth login
│   │   │   ├── Register.jsx           # Citizen registration
│   │   │   ├── Dashboard.jsx          # Station admin real-time dashboard
│   │   │   ├── EmergencyDetail.jsx    # Single emergency full view (admin)
│   │   │   └── SuperAdminPanel.jsx    # National HQ view (all stations)
│   │   │
│   │   ├── components/
│   │   │   ├── SOSButton.jsx          # Big red SOS button with GPS logic
│   │   │   ├── EmergencyForm.jsx      # Full detailed submission form
│   │   │   ├── EmergencyCard.jsx      # Dashboard list item per emergency
│   │   │   ├── MapView.jsx            # Leaflet map (emergency + station pins)
│   │   │   ├── StatusBadge.jsx        # Colored status pill
│   │   │   ├── AlarmAlert.jsx         # Full-screen alarm popup for admins
│   │   │   └── DispatchModal.jsx      # Admin approval + unit assignment
│   │   │
│   │   ├── hooks/
│   │   │   ├── useSocket.js           # Socket.IO connection + event listeners
│   │   │   ├── useGPS.js              # GPS capture with accuracy tracking
│   │   │   └── useAuth.js             # Auth state, token management
│   │   │
│   │   ├── services/
│   │   │   ├── api.js                 # Axios instance + all API call functions
│   │   │   └── socket.js              # Socket.IO client initialisation
│   │   │
│   │   └── store/
│   │       ├── authStore.js           # Zustand: user, token, role
│   │       └── emergencyStore.js      # Zustand: active emergencies list
│   │
│   ├── package.json
│   ├── vite.config.js
│   └── .env.example
│
└── docs/
    ├── api-examples.md                # Example API requests and responses
    ├── sms-templates.md               # All SMS message templates
    └── station-setup-guide.md         # Guide for onboarding new stations
```

---

## 8. Database Design

### 8.1 users

| Column | Type | Notes |
|---|---|---|
| id | UUID | Primary key |
| name | VARCHAR(100) | Full name |
| phone | VARCHAR(20) | Unique, required |
| email | VARCHAR(150) | Optional |
| id_number | VARCHAR(50) | National ID, unique |
| role | ENUM | `citizen`, `station_admin`, `super_admin` |
| is_verified | BOOLEAN | Admin-verified account |
| station_id | UUID FK | Null unless station_admin |
| created_at | TIMESTAMP | |

### 8.2 stations

| Column | Type | Notes |
|---|---|---|
| id | UUID | Primary key |
| name | VARCHAR(100) | e.g. "Serrekunda Fire Station" |
| phone | VARCHAR(20) | Admin contact number |
| location | GEOGRAPHY(POINT, 4326) | PostGIS geo point |
| region | VARCHAR(100) | Gambian region |
| status | ENUM | `available`, `responding`, `unavailable` |
| created_at | TIMESTAMP | |

### 8.3 emergencies

| Column | Type | Notes |
|---|---|---|
| id | UUID | Primary key |
| ref_number | VARCHAR(10) | e.g. GM-04821, unique |
| type | ENUM | `fire`, `rescue`, `hazmat`, `other` |
| severity | ENUM | `low`, `medium`, `high`, `critical` |
| location | GEOGRAPHY(POINT, 4326) | Citizen GPS |
| landmark | TEXT | Verbal location description |
| region | VARCHAR(100) | Selected region |
| description | TEXT | Citizen's description |
| status | ENUM | `pending`, `assigned`, `dispatched`, `on_scene`, `resolved`, `cancelled` |
| submitted_by | UUID FK | Null if guest |
| caller_phone | VARCHAR(20) | Required for guests |
| is_verified_report | BOOLEAN | False if guest submission |
| assigned_station_id | UUID FK | Station assigned |
| created_at | TIMESTAMP | |
| updated_at | TIMESTAMP | |

### 8.4 responses

| Column | Type | Notes |
|---|---|---|
| id | UUID | Primary key |
| emergency_id | UUID FK | |
| station_id | UUID FK | |
| unit_name | VARCHAR(50) | e.g. "Truck 04" |
| approved_by | UUID FK | Admin user who approved |
| assigned_at | TIMESTAMP | When station was notified |
| approved_at | TIMESTAMP | When admin approved |
| dispatched_at | TIMESTAMP | When unit left station |
| arrived_at | TIMESTAMP | When unit reached scene |
| resolved_at | TIMESTAMP | When emergency resolved |
| response_time_seconds | INTEGER | Calculated: resolved_at - assigned_at |
| notes | TEXT | Admin notes |

---

## 9. API Reference

### Authentication

```
POST   /api/auth/register
       Body: { name, phone, email, id_number, password }
       Returns: { user, access_token }

POST   /api/auth/login
       Body: { phone, password }
       Returns: { user, access_token }

POST   /api/auth/guest-token
       Body: { phone }
       Returns: { guest_token }
       Note: Token expires in 1 hour, allows one submission only
```

### Emergencies

```
POST   /api/emergencies
       Auth: Bearer token (registered or guest)
       Body: { type, severity, latitude, longitude, landmark,
               region, description, caller_phone? }
       Returns: { emergency_id, ref_number, status, assigned_station }

GET    /api/emergencies
       Auth: station_admin or super_admin
       Query: ?status=pending&region=GBA&limit=50
       Returns: [ list of emergencies ]

GET    /api/emergencies/{id}
       Auth: owner or admin
       Returns: { full emergency detail + response record }

PATCH  /api/emergencies/{id}/approve
       Auth: station_admin
       Body: { unit_name }
       Returns: { updated emergency, triggers SMS to citizen + unit }

PATCH  /api/emergencies/{id}/status
       Auth: station_admin
       Body: { status: "on_scene" | "resolved" | "cancelled" }
       Returns: { updated emergency }
```

### Stations

```
GET    /api/stations
       Auth: any admin
       Returns: [ all stations with location + status ]

GET    /api/stations/{id}
       Returns: { station detail }

PATCH  /api/stations/{id}/status
       Auth: station_admin (own station) or super_admin
       Body: { status: "available" | "responding" | "unavailable" }
       Returns: { updated station }
```

### Admin

```
GET    /api/admin/dashboard
       Auth: super_admin
       Returns: {
         total_emergencies_today,
         pending_count,
         active_count,
         avg_response_time_minutes,
         stations_available,
         stations_responding
       }
```

---

## 10. Real-time Events

FireAlert GM uses **Socket.IO** for real-time communication between the backend and connected clients.

### Rooms

- `station:{station_id}` — Admin dashboards join this room on login
- `emergency:{emergency_id}` — Citizens join this room to track their emergency
- `superadmin` — Super admin room for national overview

### Events: Server → Client

```
new_emergency
  Room: station:{station_id}
  Payload: { emergency_id, ref_number, type, severity,
             landmark, location, is_verified_report, created_at }
  Action: Triggers alarm sound + popup on admin dashboard

emergency_approved
  Room: emergency:{emergency_id}
  Payload: { ref_number, unit_name, station_name, status }
  Action: Updates citizen tracking screen

emergency_status_updated
  Room: emergency:{emergency_id}
  Payload: { status, updated_at }
  Action: Updates status badge on citizen screen

station_unavailable_escalation
  Room: superadmin
  Payload: { emergency_id, message: "No available stations in region X" }
  Action: Alerts super admin
```

### Events: Client → Server

```
join_station_room
  Payload: { station_id }
  Action: Admin joins their station alert room

join_emergency_room
  Payload: { emergency_id }
  Action: Citizen subscribes to their emergency updates
```

---

## 11. MVP Scope

The MVP is defined as the **minimum system that can save a life end-to-end.** It must work in a real emergency without failure.

### ✅ In MVP

| Feature | Description |
|---|---|
| Guest emergency submission | Phone number only, flagged unverified |
| Registered citizen submission | Full ID-verified account |
| GPS auto-capture | Browser geolocation API |
| Landmark fallback | Manual description if GPS denied |
| Emergency type selection | Fire, Rescue, Hazmat, Other |
| Severity level | Low, Medium, High, Critical |
| Find nearest available station | PostGIS distance query |
| Fallback to next nearest | If closest station is busy |
| Real-time dashboard alarm | Socket.IO + alarm sound |
| SMS to station admin | Africa's Talking |
| Admin approves dispatch | One-click with unit selection |
| SMS to citizen on dispatch | Confirmation with reference number |
| SMS to fire unit | GPS coordinates + Google Maps link |
| Emergency tracking reference | GM-XXXXX format |
| Citizen tracking screen | See current status of their report |
| Station status management | Available / Responding / Unavailable |
| JWT authentication | Registered users + station admins |
| Guest token | One-hour token for unverified submissions |

### ❌ Not In MVP (Phase 2+)

| Feature | Phase |
|---|---|
| Voice call confirmation to citizen | Phase 2 |
| In-app turn-by-turn navigation | Phase 2 |
| Photo uploads | Phase 2 |
| Offline submission queue | Phase 2 |
| Multi-language (Wolof, Mandinka) | Phase 3 |
| Analytics dashboard | Phase 3 |
| Multi-agency (Police, Ambulance) | Phase 3 |
| Unit live GPS tracking | Phase 3 |
| Mobile app (Android APK) | Phase 3 |
| Automated response time reports | Phase 3 |

---

## 12. Full Product Scope

This section describes the complete vision of FireAlert GM beyond the MVP — the full system as it should eventually exist when donated and running at scale.

### Phase 2 — Enhanced Response

**Voice call confirmation**
After a citizen submits and help is dispatched, the system places an automated voice call to their phone using Africa's Talking Voice API. The message confirms the emergency reference number, the unit dispatched, and an estimated arrival time. This is critical for low-literacy users who may not read the SMS.

**Photo uploads**
Citizens can attach up to 4 photos of the emergency scene. Photos are uploaded to cloud storage (Cloudinary or AWS S3) and visible to the admin on the dashboard. This gives dispatchers better situational awareness before the unit arrives.

**Offline submission queue**
If the citizen has no internet at the moment of submission, the PWA stores the emergency data locally using IndexedDB. When connectivity returns, it automatically submits the queued report. A visual indicator shows the user their report is queued. This is essential for areas with poor connectivity in The Gambia.

**Unit status updates via SMS**
Fire units without smartphones can reply to their SMS alert with simple codes:
- Reply `1` → Status set to "En Route"
- Reply `2` → Status set to "On Scene"
- Reply `3` → Status set to "Resolved"
Africa's Talking handles incoming SMS webhooks to parse these replies.

### Phase 3 — Intelligence & Scale

**Analytics dashboard**
A dedicated analytics section for Super Admin showing:
- Heatmap of emergency hotspots across The Gambia
- Average response time per station and per region
- Peak emergency hours (by day and month)
- Station performance comparison
- Emergency type breakdown (fire vs rescue vs hazmat)
- Monthly and annual summary reports (exportable as PDF)

This data is invaluable for government resource planning — where to build new stations, which stations need more trucks, which regions are underserved.

**Multi-language support**
The citizen PWA will support Wolof, Mandinka, Fula, and English. Language is auto-detected from browser settings but manually selectable. All SMS messages are sent in the citizen's chosen language. Admin dashboard remains English only.

**Multi-agency integration**
The system expands to support Police (GPS) and Ambulance alongside fire service. Emergency type selection routes to the appropriate agency. A joint dispatch view allows Super Admin to coordinate multi-agency responses to major incidents.

**Unit live GPS tracking**
Fire units with smartphones run the Unit PWA which continuously broadcasts their GPS location to the backend via Socket.IO. The admin dashboard shows live positions of all units on the map. The citizen tracking screen shows the unit moving toward them in real time.

**Android APK**
The PWA is packaged as an Android APK using Capacitor for distribution to fire service units, removing the need to open a browser. The app runs in the background and receives push notifications even when closed.

**Automated reports to government**
Monthly PDF reports are auto-generated and emailed to GFRS headquarters, summarising system usage, response performance, and emergency trends. This supports accountability and future funding decisions.

---

## 13. Tech Stack

### Backend
| Technology | Purpose |
|---|---|
| Python 3.11+ | Language |
| FastAPI | Web framework |
| SQLAlchemy 2.0 (async) | ORM |
| PostgreSQL 15 | Database |
| PostGIS | Geographic queries (nearest station) |
| Alembic | Database migrations |
| python-socketio | Real-time WebSocket events |
| python-jose | JWT authentication |
| passlib + bcrypt | Password hashing |
| httpx | Async HTTP client |
| Africa's Talking SDK | SMS and voice calls |

### Frontend
| Technology | Purpose |
|---|---|
| React 18 | UI framework |
| Vite | Build tool |
| React Router v6 | Client-side routing |
| Zustand | Global state management |
| Axios | API requests |
| Socket.IO client | Real-time events |
| Leaflet + React-Leaflet | Interactive maps |
| OpenStreetMap | Free map tiles (works in Gambia) |
| Tailwind CSS | Styling |

### Infrastructure
| Technology | Purpose |
|---|---|
| Docker + Docker Compose | Local development environment |
| Railway or Render | Backend hosting |
| Vercel or Netlify | Frontend hosting |
| Cloudinary | Photo uploads (Phase 2) |
| Africa's Talking | SMS + Voice (Gambia supported) |
| GitHub Actions | CI/CD pipeline |

---

## 14. Environment Variables

### Backend `.env`

```env
# Database
DATABASE_URL=postgresql+asyncpg://user:password@localhost:5432/firealert_db

# Security
SECRET_KEY=your-very-secret-jwt-key-change-this
ALGORITHM=HS256
ACCESS_TOKEN_EXPIRE_MINUTES=10080

# Africa's Talking
AT_USERNAME=your_africas_talking_username
AT_API_KEY=your_africas_talking_api_key
AT_SENDER_ID=FireAlertGM
AT_SHORTCODE=your_shortcode

# App
ENVIRONMENT=development
FRONTEND_URL=http://localhost:5173
ALLOWED_ORIGINS=http://localhost:5173,https://firealertgm.com
```

### Frontend `.env`

```env
VITE_API_BASE_URL=http://localhost:8000
VITE_SOCKET_URL=http://localhost:8000
VITE_APP_NAME=FireAlert GM
VITE_ENVIRONMENT=development
```

---

## 15. Sprint Plan

### Sprint 1 — Data Layer (Days 1–3)
- Set up PostgreSQL + PostGIS locally with Docker
- Create all SQLAlchemy models (users, stations, emergencies, responses)
- Write and run Alembic initial migration
- Seed database with all Gambian fire station locations
- Verify PostGIS nearest-station query works correctly

**Done when:** Running `SELECT nearest_station(13.45, -16.58)` returns the correct station.

---

### Sprint 2 — Core API (Days 4–7)
- FastAPI app structure and config
- Auth endpoints: register, login, guest token
- Emergency submission endpoint with PostGIS station assignment
- Emergency status update endpoints
- Station status management endpoint
- JWT middleware and role-based access
- Pydantic schemas for all request/response shapes

**Done when:** Submitting a POST to `/api/emergencies` creates a record, assigns the nearest station, and returns a reference number.

---

### Sprint 3 — Real-time & Alerts (Days 8–10)
- Socket.IO server setup and room management
- `new_emergency` event fires on submission
- `emergency_approved` event fires on dispatch
- Africa's Talking SMS: alert to admin on submission
- Africa's Talking SMS: confirmation to citizen on dispatch
- Africa's Talking SMS: GPS link to fire unit on dispatch

**Done when:** Submitting an emergency triggers a dashboard alarm AND an SMS to the test admin number within 3 seconds.

---

### Sprint 4 — Frontend (Days 11–17)
- Vite + React project setup with PWA manifest
- Axios API service layer
- Socket.IO client hooks
- Citizen pages: ReportEmergency, TrackEmergency
- Auth pages: Login, Register
- Admin pages: Dashboard (map + alarm), EmergencyDetail, DispatchModal
- Leaflet map with emergency and station pins
- Connect all pages to real backend

**Done when:** Full flow works in browser — submit emergency, admin sees alarm, admin approves, citizen tracking screen updates.

---

### Sprint 5 — Polish & Deploy (Days 18–21)
- End-to-end testing of full emergency flow
- Error handling on all endpoints (network failure, GPS denied, SMS failure)
- Mobile responsiveness check on real Android devices
- Dockerfile for backend
- Deploy backend to Railway
- Deploy frontend to Vercel
- Connect domain (if available)
- Write station setup guide for GFRS handover

**Done when:** A full emergency can be submitted on a real phone, dispatched by an admin, and the unit receives the SMS — all on production servers.

---

## 16. Setup & Installation

### Prerequisites
- Python 3.11+
- Node.js 18+
- Docker & Docker Compose
- Git

### 1. Clone the repository

```bash
git clone https://github.com/yourusername/firealert-gm.git
cd firealert-gm
```

### 2. Start the database

```bash
docker-compose up -d db
```

This starts PostgreSQL 15 with PostGIS enabled on port 5432.

### 3. Set up the backend

```bash
cd backend
python -m venv venv
source venv/bin/activate       # Windows: venv\Scripts\activate
pip install -r requirements.txt
cp .env.example .env           # Fill in your values
alembic upgrade head            # Run migrations
python -m app.seeds.stations   # Seed fire station locations
uvicorn app.main:app --reload  # Start dev server
```

Backend runs at: `http://localhost:8000`
API docs at: `http://localhost:8000/docs`

### 4. Set up the frontend

```bash
cd frontend
npm install
cp .env.example .env           # Fill in your values
npm run dev                    # Start dev server
```

Frontend runs at: `http://localhost:5173`

### 5. Run everything with Docker (optional)

```bash
docker-compose up --build
```

This starts the database, backend, and frontend together.

---

## 17. Deployment

### Backend (Railway)

1. Create a new project on [Railway](https://railway.app)
2. Add a PostgreSQL database service — enable PostGIS extension
3. Connect your GitHub repository
4. Set all environment variables from section 14
5. Railway auto-deploys on every push to `main`

### Frontend (Vercel)

1. Import repository on [Vercel](https://vercel.com)
2. Set framework to Vite
3. Set root directory to `frontend/`
4. Add environment variables from section 14
5. Vercel auto-deploys on every push to `main`

### Production Checklist

- [ ] `ENVIRONMENT=production` set in backend
- [ ] `SECRET_KEY` is a strong random string (not the default)
- [ ] `ALLOWED_ORIGINS` lists only the production frontend URL
- [ ] Africa's Talking account is in live mode (not sandbox)
- [ ] PostGIS extension enabled on production database
- [ ] All stations seeded in production database
- [ ] HTTPS enforced on all endpoints
- [ ] Error logging set up (Sentry recommended)

---

## 18. SMS & Voice Integration

FireAlert GM uses **Africa's Talking** for all SMS communications. Africa's Talking supports The Gambia natively.

### SMS Templates

**Alert to station admin (on submission):**
```
FIREALERT GM — NEW EMERGENCY
Type: {type} | Severity: {severity}
Location: {landmark}
GPS: maps.google.com/?q={lat},{lng}
Reporter: {verified/UNVERIFIED}
Ref: {ref_number}
Login to dashboard to dispatch: {dashboard_url}
```

**Confirmation to citizen (on dispatch):**
```
FIREALERT GM: Help is on the way.
{station_name} has dispatched a unit to your location.
Reference: {ref_number}
Keep this line free. If emergency worsens, call 118.
```

**Alert to fire unit (on dispatch):**
```
FIREALERT GM DISPATCH — {ref_number}
Emergency: {type} — {severity}
Location: {landmark}
Navigate: maps.google.com/?q={lat},{lng}
Reporter phone: {caller_phone}
```

**Escalation to Super Admin (no station available):**
```
FIREALERT GM ALERT: No available station found for emergency {ref_number} in {region}. Manual dispatch required. Login: {dashboard_url}
```

### Voice Call (Phase 2)
Africa's Talking Voice API will call the citizen after dispatch with a recorded message confirming help is coming. This is especially important for citizens who may not read SMS.

---

## 19. Security & Data Policy

### Authentication
- All API endpoints (except public emergency submission) require a valid JWT
- Guest tokens expire after 1 hour and allow one submission only
- Passwords are hashed with bcrypt (cost factor 12)
- Tokens are signed with HS256 and expire after 7 days

### Data Protection
- National ID numbers are stored but never returned in API responses
- Citizen GPS coordinates are only visible to station admins and super admin
- Guest submissions show phone number only to admins — not publicly
- All data is stored on servers located within accessible jurisdiction

### False Reports
- All submissions are tied to a phone number (registered or guest)
- Registered users have their national ID on file
- False emergency reporting is a criminal offence under Gambian law
- Super Admin can flag, ban, and report users to authorities
- IP address is logged on every submission for law enforcement use

### Data Retention
- Active emergencies: retained indefinitely for analytics
- User accounts: retained unless deletion requested
- Guest submission records: retained for 2 years
- Response time data: retained permanently for government reporting

---

## 20. Handover & Maintenance

FireAlert GM is being donated to the **Gambia Fire and Rescue Service (GFRS)**. This section covers what they need to operate it independently.

### What GFRS Needs

**Technical requirements:**
- One person with basic computer literacy to manage the admin dashboard
- A smartphone per station for the admin dashboard PWA
- An Africa's Talking account with SMS credits topped up
- A server subscription (Railway ~$5/month or government-hosted)

**Ongoing costs (estimated):**
- Server hosting: $5–20/month (Railway)
- SMS credits: ~$0.02 per SMS via Africa's Talking (bulk pricing available)
- Domain name: $10/year (optional)

### Documentation Provided
- `docs/station-setup-guide.md` — Step-by-step guide for onboarding a new fire station
- `docs/admin-user-guide.md` — How to use the admin dashboard (with screenshots)
- `docs/api-examples.md` — For any future developers maintaining the system

### Support Plan
- Bug fixes and critical patches: provided free by the developer for 12 months post-donation
- Feature additions: community or government-funded development
- Training: one training session with GFRS dispatchers provided

---

## Contributing

This is an open source project donated to save lives in The Gambia. Contributions are welcome.

1. Fork the repository
2. Create a feature branch: `git checkout -b feature/your-feature`
3. Commit your changes: `git commit -m 'Add your feature'`
4. Push to the branch: `git push origin feature/your-feature`
5. Open a Pull Request

---

## License

MIT License — free to use, modify, and distribute.

---

## Contact

Built by **musbi** — creator of AttendanceGM
For questions or contributions: open an issue on GitHub

---

*FireAlert GM — Because every second counts.*

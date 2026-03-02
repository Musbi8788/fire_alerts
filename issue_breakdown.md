<html>
<body>
<!--StartFragment--><html><head></head><body><h1>🔥 FireAlert GM — GitHub Issues Breakdown</h1>
<blockquote>
<p>Copy each issue below directly into GitHub.
Every issue is a single, closeable unit of work.
Labels, milestone, and acceptance criteria are included for each.</p>
</blockquote>
<hr>
<h2>How to Set Up Your GitHub Board</h2>
<h3>Labels to Create First</h3>
<pre><code>Type labels:
  setup        → #6366f1  (purple)
  backend      → #f97316  (orange)
  frontend     → #3b82f6  (blue)
  database     → #10b981  (green)
  infra        → #64748b  (slate)
  testing      → #f59e0b  (amber)
  security     → #ef4444  (red)

Priority labels:
  critical     → #dc2626
  high         → #ea580c
  medium       → #ca8a04
  low          → #16a34a

Status labels:
  blocked      → #991b1b
  in-review    → #7c3aed
  needs-design → #0e7490
</code></pre>
<h3>Milestones to Create</h3>
<pre><code>Sprint 1 — Data Layer        (Days 1–3)
Sprint 2 — Core API          (Days 4–7)
Sprint 3 — Celery &amp; Alerts   (Days 8–11)
Sprint 4 — Frontend          (Days 12–18)
Sprint 5 — Production Deploy (Days 19–23)
</code></pre>
<hr>
<h2>SPRINT 1 — Data Layer</h2>
<p><strong>Milestone:</strong> Sprint 1 — Data Layer
<strong>Goal:</strong> Database running, all tables created, PostGIS query verified.</p>
<hr>
<h3>Issue #1</h3>
<p><strong>Title:</strong> <code>[SETUP] Initialise project repository structure</code>
<strong>Labels:</strong> <code>setup</code>
<strong>Milestone:</strong> Sprint 1 — Data Layer</p>
<p><strong>Description:</strong>
Create the full monorepo folder structure for FireAlert GM as defined in the README. This is the foundation everything else builds on.</p>
<p><strong>Tasks:</strong></p>
<ul>
<li>[ ] Create root directory <code>firealert-gm/</code></li>
<li>[ ] Create <code>backend/</code> with <code>app/</code>, <code>migrations/</code>, <code>seeds/</code>, <code>tests/</code> folders</li>
<li>[ ] Create all empty <code>__init__.py</code> files in backend subfolders</li>
<li>[ ] Create <code>frontend/</code> with <code>src/</code>, <code>public/</code> folders</li>
<li>[ ] Create <code>docs/</code> folder with empty placeholder files</li>
<li>[ ] Create <code>.github/workflows/</code> folder</li>
<li>[ ] Create root <code>docker-compose.yml</code> (empty skeleton)</li>
<li>[ ] Create root <code>.gitignore</code> covering Python, Node, <code>.env</code> files</li>
<li>[ ] Create <code>README.md</code> at root (copy from documentation)</li>
<li>[ ] Push initial commit to GitHub</li>
</ul>
<p><strong>Acceptance Criteria:</strong></p>
<ul>
<li>[ ] Repository is on GitHub with correct folder structure</li>
<li>[ ] <code>.env</code> files are gitignored and not committed</li>
<li>[ ] README renders correctly on GitHub</li>
</ul>
<hr>
<h3>Issue #2</h3>
<p><strong>Title:</strong> <code>[INFRA] Set up Docker Compose for local development</code>
<strong>Labels:</strong> <code>infra</code>, <code>setup</code>
<strong>Milestone:</strong> Sprint 1 — Data Layer</p>
<p><strong>Description:</strong>
Create a <code>docker-compose.yml</code> that spins up PostgreSQL 15 with PostGIS and Redis with a single command.</p>
<p><strong>Tasks:</strong></p>
<ul>
<li>[ ] Add PostgreSQL 15 service with PostGIS (<code>postgis/postgis:15-3.3</code>) image</li>
<li>[ ] Add Redis 7 service</li>
<li>[ ] Configure persistent volumes for both services</li>
<li>[ ] Set database name to <code>firealert_db</code>, user to <code>firealert</code>, password via env</li>
<li>[ ] Expose PostgreSQL on port 5432, Redis on 6379</li>
<li>[ ] Add <code>.env.example</code> with all required Docker variables</li>
<li>[ ] Test: <code>docker-compose up -d db redis</code> starts both services cleanly</li>
<li>[ ] Add PostGIS extension auto-creation via init SQL script</li>
</ul>
<p><strong>Acceptance Criteria:</strong></p>
<ul>
<li>[ ] <code>docker-compose up -d db redis</code> runs without errors</li>
<li>[ ] <code>psql</code> connects to the database successfully</li>
<li>[ ] <code>SELECT PostGIS_Version();</code> returns a version string</li>
<li>[ ] Redis responds to <code>redis-cli ping</code> with <code>PONG</code></li>
</ul>
<hr>
<h3>Issue #3</h3>
<p><strong>Title:</strong> <code>[BACKEND] Set up FastAPI project with config and database connection</code>
<strong>Labels:</strong> <code>backend</code>, <code>setup</code>
<strong>Milestone:</strong> Sprint 1 — Data Layer</p>
<p><strong>Description:</strong>
Bootstrap the FastAPI application with async SQLAlchemy, pydantic-settings config, and database session management.</p>
<p><strong>Tasks:</strong></p>
<ul>
<li>[ ] Create <code>requirements.txt</code> with all backend dependencies</li>
<li>[ ] Create <code>app/config.py</code> using <code>pydantic-settings</code> — loads all env vars with validation</li>
<li>[ ] Create <code>app/database.py</code> — async SQLAlchemy engine with connection pooling config:
<ul>
<li><code>pool_size=10</code>, <code>max_overflow=20</code>, <code>pool_pre_ping=True</code>, <code>pool_recycle=3600</code></li>
</ul>
</li>
<li>[ ] Create <code>app/main.py</code> — FastAPI app init, CORS middleware, router stubs</li>
<li>[ ] Create <code>GET /health</code> endpoint that checks DB + Redis connectivity</li>
<li>[ ] Create <code>backend/.env.example</code> with all required variables</li>
<li>[ ] Test that <code>uvicorn app.main:app --reload</code> starts without errors</li>
<li>[ ] Test that <code>/health</code> returns <code>{"status": "ok", "db": "connected"}</code></li>
</ul>
<p><strong>Acceptance Criteria:</strong></p>
<ul>
<li>[ ] <code>uvicorn app.main:app --reload</code> starts without import errors</li>
<li>[ ] <code>GET /health</code> returns 200 with <code>"status": "ok"</code></li>
<li>[ ] Config fails loudly if required env vars are missing</li>
</ul>
<hr>
<h3>Issue #4</h3>
<p><strong>Title:</strong> <code>[DATABASE] Create SQLAlchemy models — users, stations, emergencies, responses</code>
<strong>Labels:</strong> <code>database</code>, <code>backend</code>
<strong>Milestone:</strong> Sprint 1 — Data Layer</p>
<p><strong>Description:</strong>
Define all four core SQLAlchemy ORM models with correct column types, relationships, and PostGIS geography columns.</p>
<p><strong>Tasks:</strong></p>
<ul>
<li>[ ] Create <code>models/user.py</code>:
<ul>
<li><code>id</code> (UUID), <code>full_name</code>, <code>phone</code> (unique), <code>email</code>, <code>id_number</code></li>
<li><code>role</code> (default: citizen), <code>is_verified</code>, <code>is_banned</code></li>
<li><code>station_id</code> (FK to stations), <code>password_hash</code>, <code>created_at</code></li>
</ul>
</li>
<li>[ ] Create <code>models/station.py</code>:
<ul>
<li><code>id</code> (UUID), <code>name</code>, <code>phone</code>, <code>admin_phone</code>, <code>region</code>, <code>address</code></li>
<li><code>location</code> (GeoAlchemy2 <code>Geography(Point)</code>)</li>
<li><code>status</code> (default: available), <code>auto_approve_timeout_seconds</code> (default: 40)</li>
<li><code>created_at</code></li>
</ul>
</li>
<li>[ ] Create <code>models/emergency.py</code>:
<ul>
<li><code>id</code>, <code>ref_number</code> (unique), <code>type</code>, <code>severity</code></li>
<li><code>location</code> (Geography Point, nullable), <code>landmark</code>, <code>region</code>, <code>description</code></li>
<li><code>status</code> (default: pending), <code>submitted_by</code> (FK), <code>submitter_phone</code></li>
<li><code>is_verified_report</code>, <code>was_auto_dispatched</code>, <code>assigned_station_id</code> (FK)</li>
<li><code>submitter_ip</code> (INET type), <code>created_at</code>, <code>updated_at</code></li>
</ul>
</li>
<li>[ ] Create <code>models/response.py</code>:
<ul>
<li><code>id</code>, <code>emergency_id</code> (FK), <code>station_id</code> (FK), <code>unit_name</code></li>
<li><code>assigned_at</code>, <code>dispatched_at</code>, <code>en_route_at</code>, <code>on_scene_at</code>, <code>resolved_at</code></li>
<li><code>response_time_seconds</code> (calculated), <code>notes</code></li>
</ul>
</li>
<li>[ ] Create <code>models/__init__.py</code> exporting all models</li>
<li>[ ] All models import from shared <code>database.py</code> Base</li>
</ul>
<p><strong>Acceptance Criteria:</strong></p>
<ul>
<li>[ ] All models import without errors</li>
<li>[ ] Relationships are correctly defined (FK constraints)</li>
<li>[ ] Geography columns use <code>GeoAlchemy2</code> correctly</li>
</ul>
<hr>
<h3>Issue #5</h3>
<p><strong>Title:</strong> <code>[DATABASE] Create audit_log and sms_failure models</code>
<strong>Labels:</strong> <code>database</code>, <code>backend</code>, <code>security</code>
<strong>Milestone:</strong> Sprint 1 — Data Layer</p>
<p><strong>Description:</strong>
Create the two production-safety models: an immutable audit trail and an SMS failure log.</p>
<p><strong>Tasks:</strong></p>
<ul>
<li>[ ] Create <code>models/audit_log.py</code>:
<ul>
<li><code>id</code> (UUID), <code>actor_id</code> (FK users), <code>actor_role</code>, <code>action</code> (varchar 100)</li>
<li><code>target_id</code> (UUID), <code>ip_address</code> (INET), <code>detail</code> (JSONB), <code>created_at</code></li>
<li>No update or delete methods — insert only</li>
</ul>
</li>
<li>[ ] Create <code>models/sms_failure.py</code>:
<ul>
<li><code>id</code>, <code>emergency_id</code> (FK), <code>recipient_phone</code>, <code>message_type</code></li>
<li><code>error_message</code> (text), <code>attempts</code> (int), <code>resolved</code> (bool), <code>created_at</code></li>
</ul>
</li>
<li>[ ] Add helper method <code>AuditLog.record(actor, action, target, ip, detail)</code> as class method</li>
<li>[ ] Import both models in <code>models/__init__.py</code></li>
</ul>
<p><strong>Acceptance Criteria:</strong></p>
<ul>
<li>[ ] Both models import correctly</li>
<li>[ ] <code>AuditLog.record()</code> class method works correctly</li>
<li>[ ] JSONB column defined correctly for PostgreSQL</li>
</ul>
<hr>
<h3>Issue #6</h3>
<p><strong>Title:</strong> <code>[DATABASE] Write and run Alembic initial migration</code>
<strong>Labels:</strong> <code>database</code>
<strong>Milestone:</strong> Sprint 1 — Data Layer</p>
<p><strong>Description:</strong>
Set up Alembic for database migrations and generate the first migration from all models.</p>
<p><strong>Tasks:</strong></p>
<ul>
<li>[ ] Install and configure Alembic: <code>alembic init migrations</code></li>
<li>[ ] Update <code>migrations/env.py</code> to use async SQLAlchemy engine and import all models</li>
<li>[ ] Generate initial migration: <code>alembic revision --autogenerate -m "initial_schema"</code></li>
<li>[ ] Review generated migration file — verify all tables and columns are correct</li>
<li>[ ] Run migration: <code>alembic upgrade head</code></li>
<li>[ ] Verify all tables exist in database: <code>\dt</code> in psql</li>
<li>[ ] Verify PostGIS geometry columns created correctly</li>
<li>[ ] Add <code>alembic.ini</code> to repo (without credentials)</li>
</ul>
<p><strong>Acceptance Criteria:</strong></p>
<ul>
<li>[ ] <code>alembic upgrade head</code> runs without errors</li>
<li>[ ] All 6 tables exist in the database</li>
<li>[ ] <code>alembic current</code> shows the migration as applied</li>
<li>[ ] <code>alembic downgrade -1</code> then <code>alembic upgrade head</code> works cleanly</li>
</ul>
<hr>
<h3>Issue #7</h3>
<p><strong>Title:</strong> <code>[DATABASE] Seed Gambia fire station data with GPS coordinates</code>
<strong>Labels:</strong> <code>database</code>, <code>backend</code>
<strong>Milestone:</strong> Sprint 1 — Data Layer</p>
<p><strong>Description:</strong>
Create a seed script that populates all known Gambia fire stations with accurate GPS coordinates so the PostGIS query has real data to work with.</p>
<p><strong>Tasks:</strong></p>
<ul>
<li>[ ] Research and document all GFRS fire station locations and phone numbers</li>
<li>[ ] Create <code>seeds/stations.py</code> with station data as Python list of dicts</li>
<li>[ ] Include at minimum these stations (research exact GPS):
<ul>
<li>Banjul Fire Station (HQ)</li>
<li>Kanifing Fire Station</li>
<li>Brikama Fire Station</li>
<li>Farafenni Fire Station</li>
<li>Basse Fire Station</li>
<li>Soma Fire Station</li>
<li>Kerewan Fire Station</li>
</ul>
</li>
<li>[ ] Each station entry: name, phone, admin_phone, region, address, latitude, longitude</li>
<li>[ ] Seed script is idempotent — running twice doesn't create duplicates</li>
<li>[ ] Script prints confirmation of each station seeded</li>
<li>[ ] Run: <code>python -m seeds.stations</code></li>
</ul>
<p><strong>Acceptance Criteria:</strong></p>
<ul>
<li>[ ] All stations appear in database after running script</li>
<li>[ ] Each station has a valid PostGIS geography point</li>
<li>[ ] Script is safe to run multiple times (upsert or check-before-insert)</li>
<li>[ ] Running <code>SELECT COUNT(*) FROM stations;</code> returns correct count</li>
</ul>
<hr>
<h3>Issue #8</h3>
<p><strong>Title:</strong> <code>[DATABASE] Build and verify PostGIS nearest-station query</code>
<strong>Labels:</strong> <code>database</code>, <code>backend</code>
<strong>Milestone:</strong> Sprint 1 — Data Layer</p>
<p><strong>Description:</strong>
Write and test the core PostGIS geospatial query that powers the entire dispatch system. This is the most critical query in the codebase.</p>
<p><strong>Tasks:</strong></p>
<ul>
<li>[ ] Create <code>services/geo.py</code></li>
<li>[ ] Implement <code>find_nearest_available_station(lat, lng, excluded_ids=[])</code> async function</li>
<li>[ ] Query uses <code>&lt;-&gt;</code> operator for index-optimized nearest-neighbor search</li>
<li>[ ] Query filters <code>status = 'available'</code> and excludes <code>excluded_ids</code></li>
<li>[ ] Returns station object with <code>distance_meters</code> field</li>
<li>[ ] Returns <code>None</code> if no available station found</li>
<li>[ ] Implement <code>find_station_cascade(lat, lng, max_attempts=5)</code> — calls above in loop</li>
<li>[ ] Returns list of tried stations so caller knows how many were checked</li>
<li>[ ] Write manual test: query from Serrekunda coordinates, verify returns Kanifing station</li>
<li>[ ] Create spatial index on stations.location: <code>CREATE INDEX ON stations USING GIST(location)</code></li>
</ul>
<p><strong>Acceptance Criteria:</strong></p>
<ul>
<li>[ ] Query from <code>(13.4531, -16.5775)</code> returns the geographically nearest station</li>
<li>[ ] Excluding that station returns the second nearest</li>
<li>[ ] Returns <code>None</code> when all stations set to <code>unavailable</code></li>
<li>[ ] Query executes in under 100ms on seeded dataset</li>
</ul>
<p><strong>Verification SQL:</strong></p>
<pre><code class="language-sql">SELECT name, ST_Distance(location, ST_GeogFromText('POINT(-16.578 13.454)')) as meters
FROM stations WHERE status = 'available'
ORDER BY location &lt;-&gt; ST_GeogFromText('POINT(-16.578 13.454)') LIMIT 1;
</code></pre>
<hr>
<h3>Issue #9</h3>
<p><strong>Title:</strong> <code>[BACKEND] Create input validators for GPS, phone, and rate limiting</code>
<strong>Labels:</strong> <code>backend</code>, <code>security</code>
<strong>Milestone:</strong> Sprint 1 — Data Layer</p>
<p><strong>Description:</strong>
Write reusable validators that enforce data integrity and security rules on all incoming submissions.</p>
<p><strong>Tasks:</strong></p>
<ul>
<li>[ ] Create <code>utils/validators.py</code></li>
<li>[ ] Implement <code>validate_gambia_coords(lat, lng) -&gt; bool</code>:
<ul>
<li>Bounds: lat 13.065–13.826, lng -16.895 to -13.795</li>
<li>Raises <code>ValueError</code> with clear message if out of bounds</li>
</ul>
</li>
<li>[ ] Implement <code>validate_gambia_phone(phone) -&gt; bool</code>:
<ul>
<li>Must match <code>+220</code> prefix followed by 7 digits</li>
</ul>
</li>
<li>[ ] Implement <code>check_guest_rate_limit(phone, db) -&gt; bool</code>:
<ul>
<li>Returns False if phone has submitted within last 1 hour</li>
<li>Uses emergencies table — no Redis needed for this check</li>
</ul>
</li>
<li>[ ] Write unit tests for all three validators covering edge cases</li>
</ul>
<p><strong>Acceptance Criteria:</strong></p>
<ul>
<li>[ ] Gambia bounds validator rejects coordinates from Senegal or outside Gambia</li>
<li>[ ] Phone validator rejects numbers not starting with <code>+220</code></li>
<li>[ ] Rate limit returns False for second submission within 1 hour from same number</li>
</ul>
<hr>
<h2>SPRINT 2 — Core API</h2>
<p><strong>Milestone:</strong> Sprint 2 — Core API
<strong>Goal:</strong> All endpoints working, auth complete, emergency submission returns ref number.</p>
<hr>
<h3>Issue #10</h3>
<p><strong>Title:</strong> <code>[BACKEND] Build security utilities — JWT, refresh tokens, bcrypt</code>
<strong>Labels:</strong> <code>backend</code>, <code>security</code>
<strong>Milestone:</strong> Sprint 2 — Core API</p>
<p><strong>Description:</strong>
Build all authentication utilities used across the entire backend.</p>
<p><strong>Tasks:</strong></p>
<ul>
<li>[ ] Create <code>utils/security.py</code></li>
<li>[ ] Implement <code>hash_password(password) -&gt; str</code> using bcrypt (cost factor 12)</li>
<li>[ ] Implement <code>verify_password(plain, hashed) -&gt; bool</code></li>
<li>[ ] Implement <code>create_access_token(data, expires_delta) -&gt; str</code> (60 min expiry)</li>
<li>[ ] Implement <code>create_refresh_token(data) -&gt; str</code> (7 day expiry)</li>
<li>[ ] Implement <code>decode_token(token) -&gt; dict</code> — raises HTTPException 401 on invalid</li>
<li>[ ] Implement FastAPI dependency <code>get_current_user(token, db)</code> — returns user object</li>
<li>[ ] Implement <code>require_role(*roles)</code> dependency factory for role-based access</li>
<li>[ ] Create <code>utils/helpers.py</code></li>
<li>[ ] Implement <code>generate_ref_id() -&gt; str</code> — returns unique <code>GM-XXXXX</code> format
<ul>
<li>5 alphanumeric chars, uppercase, avoids ambiguous characters (0, O, I, 1)</li>
</ul>
</li>
</ul>
<p><strong>Acceptance Criteria:</strong></p>
<ul>
<li>[ ] <code>hash_password</code> + <code>verify_password</code> round-trip works correctly</li>
<li>[ ] Expired tokens raise 401, not 500</li>
<li>[ ] <code>require_role('station_admin')</code> blocks citizens with 403</li>
<li>[ ] <code>generate_ref_id()</code> never returns duplicates (test 10,000 calls)</li>
</ul>
<hr>
<h3>Issue #11</h3>
<p><strong>Title:</strong> <code>[BACKEND] Build authentication router — register, login, refresh, guest token</code>
<strong>Labels:</strong> <code>backend</code>
<strong>Milestone:</strong> Sprint 2 — Core API</p>
<p><strong>Description:</strong>
All four auth endpoints that handle user registration, login, token refresh, and guest access.</p>
<p><strong>Tasks:</strong></p>
<ul>
<li>[ ] Create <code>routers/auth.py</code></li>
<li>[ ] Create <code>schemas/user.py</code> — UserCreate, UserLogin, UserResponse, TokenResponse</li>
<li>[ ] <code>POST /api/auth/register</code>:
<ul>
<li>Validate phone format (+220)</li>
<li>Check phone not already registered (return 409 if duplicate)</li>
<li>Hash password, create user with role <code>citizen</code></li>
<li>Return UserResponse (never return password_hash)</li>
</ul>
</li>
<li>[ ] <code>POST /api/auth/login</code>:
<ul>
<li>Find user by phone</li>
<li>Verify password</li>
<li>Return access_token + refresh_token + role</li>
</ul>
</li>
<li>[ ] <code>POST /api/auth/refresh</code>:
<ul>
<li>Validate refresh token</li>
<li>Issue new access_token</li>
<li>Return new access_token only</li>
</ul>
</li>
<li>[ ] <code>POST /api/auth/guest-token</code>:
<ul>
<li>Require phone number in request</li>
<li>Validate phone format</li>
<li>Check rate limit (1 active guest token per phone per hour)</li>
<li>Return short-lived access token (1 hour) with <code>is_guest: true</code> claim</li>
</ul>
</li>
<li>[ ] Register router in <code>main.py</code> with <code>/api/auth</code> prefix</li>
</ul>
<p><strong>Acceptance Criteria:</strong></p>
<ul>
<li>[ ] Duplicate phone registration returns 409 with clear message</li>
<li>[ ] Login with wrong password returns 401 (not 404)</li>
<li>[ ] Guest token cannot be refreshed</li>
<li>[ ] <code>id_number</code> never appears in any response body</li>
</ul>
<hr>
<h3>Issue #12</h3>
<p><strong>Title:</strong> <code>[BACKEND] Build emergency submission endpoint — POST /api/emergencies</code>
<strong>Labels:</strong> <code>backend</code>, <code>critical</code>
<strong>Milestone:</strong> Sprint 2 — Core API</p>
<p><strong>Description:</strong>
The most important endpoint in the system. Receives emergency, validates it, finds nearest station, creates records, and returns ref number — all in under 300ms.</p>
<p><strong>Tasks:</strong></p>
<ul>
<li>[ ] Create <code>schemas/emergency.py</code> — EmergencyCreate, EmergencyResponse, StationAssignment</li>
<li>[ ] Create <code>routers/emergencies.py</code></li>
<li>[ ] <code>POST /api/emergencies</code> implementation:
<ul>
<li>Accept both registered JWT and guest token (read <code>is_guest</code> from token)</li>
<li>Validate GPS coords with <code>validate_gambia_coords()</code> (if provided)</li>
<li>Require landmark if GPS not provided — never block submission</li>
<li>Validate phone format</li>
<li>Check guest rate limit if guest token</li>
<li>Run <code>find_station_cascade(lat, lng)</code> — get nearest available station</li>
<li>If no station found → set status <code>escalated</code>, assign to super admin</li>
<li>Create <code>Emergency</code> record with all fields</li>
<li>Create <code>Response</code> record with <code>assigned_at</code> timestamp</li>
<li>Update station status → <code>responding</code></li>
<li>Generate <code>GM-XXXXX</code> ref number</li>
<li>Log submission to <code>audit_logs</code></li>
<li>Queue Celery tasks (Sprint 3 will implement — stub for now)</li>
<li>Return ref number + assigned station name + distance in under 300ms</li>
</ul>
</li>
<li>[ ] Log submitter IP from request headers (handle proxy headers)</li>
</ul>
<p><strong>Acceptance Criteria:</strong></p>
<ul>
<li>[ ] Returns 201 with <code>ref_number</code> and <code>assigned_station.name</code></li>
<li>[ ] Response time under 300ms measured locally</li>
<li>[ ] Guest submission without GPS but with landmark succeeds</li>
<li>[ ] Coordinates outside Gambia bounds return 422 with clear error</li>
<li>[ ] Second guest submission from same phone within 1 hour returns 429</li>
</ul>
<hr>
<h3>Issue #13</h3>
<p><strong>Title:</strong> <code>[BACKEND] Build emergency status and tracking endpoints</code>
<strong>Labels:</strong> <code>backend</code>
<strong>Milestone:</strong> Sprint 2 — Core API</p>
<p><strong>Description:</strong>
All remaining emergency endpoints — listing, detail, approve, status update, and the public tracking endpoint.</p>
<p><strong>Tasks:</strong></p>
<ul>
<li>[ ] <code>GET /api/emergencies</code> — admin only, filter by status/region/type, paginated (limit/offset)</li>
<li>[ ] <code>GET /api/emergencies/{id}</code> — admin sees all fields, citizen sees only their own</li>
<li>[ ] <code>GET /api/emergencies/track/{ref_number}</code> — <strong>public, no auth required</strong>
<ul>
<li>Returns only: <code>ref_number</code>, <code>status</code>, <code>type</code>, <code>updated_at</code></li>
<li>Never exposes GPS, phone, or personal data</li>
</ul>
</li>
<li>[ ] <code>PATCH /api/emergencies/{id}/approve</code> — station admin only
<ul>
<li>Validate admin belongs to assigned station</li>
<li>Accept <code>unit_name</code> and optional <code>notes</code></li>
<li>Set <code>status = dispatched</code>, <code>was_auto_dispatched = False</code></li>
<li>Record <code>dispatched_at</code> on response record</li>
<li>Cancel auto-approve Celery task (stub for Sprint 3)</li>
<li>Log to <code>audit_logs</code></li>
<li>Queue citizen SMS + unit SMS Celery tasks (stub for Sprint 3)</li>
</ul>
</li>
<li>[ ] <code>PATCH /api/emergencies/{id}/status</code> — station admin only
<ul>
<li>Accept status: <code>en_route</code>, <code>on_scene</code>, <code>resolved</code></li>
<li>On <code>resolved</code>: calculate and store <code>response_time_seconds</code></li>
<li>On <code>resolved</code>: reset station status → <code>available</code></li>
<li>Log all status changes to <code>audit_logs</code></li>
</ul>
</li>
</ul>
<p><strong>Acceptance Criteria:</strong></p>
<ul>
<li>[ ] <code>GET /track/GM-XXXXX</code> returns status without any PII</li>
<li>[ ] Admin from Station A cannot approve emergencies assigned to Station B</li>
<li>[ ] Resolving an emergency automatically sets station back to <code>available</code></li>
<li>[ ] <code>response_time_seconds</code> is correct when emergency is resolved</li>
</ul>
<hr>
<h3>Issue #14</h3>
<p><strong>Title:</strong> <code>[BACKEND] Build stations router — list and status management</code>
<strong>Labels:</strong> <code>backend</code>
<strong>Milestone:</strong> Sprint 2 — Core API</p>
<p><strong>Description:</strong>
Station management endpoints used by admins and the dispatch logic.</p>
<p><strong>Tasks:</strong></p>
<ul>
<li>[ ] Create <code>schemas/station.py</code> — StationResponse, StationStatusUpdate</li>
<li>[ ] Create <code>routers/stations.py</code></li>
<li>[ ] <code>GET /api/stations</code> — admin only
<ul>
<li>Accept optional <code>?lat=&amp;lng=</code> params to include distance from point</li>
<li>Returns all stations with current status and distance if coords provided</li>
</ul>
</li>
<li>[ ] <code>GET /api/stations/{id}</code> — admin only, full detail</li>
<li>[ ] <code>PATCH /api/stations/{id}/status</code> — station admin or super admin
<ul>
<li>Accept <code>status</code> (available / responding / unavailable) and optional <code>reason</code></li>
<li>Log change to <code>audit_logs</code></li>
</ul>
</li>
<li>[ ] Register router in <code>main.py</code></li>
</ul>
<p><strong>Acceptance Criteria:</strong></p>
<ul>
<li>[ ] Station admin can only update their own station status</li>
<li>[ ] Super admin can update any station</li>
<li>[ ] Status change is logged in audit_logs with actor info</li>
</ul>
<hr>
<h3>Issue #15</h3>
<p><strong>Title:</strong> <code>[BACKEND] Build admin dashboard router</code>
<strong>Labels:</strong> <code>backend</code>
<strong>Milestone:</strong> Sprint 2 — Core API</p>
<p><strong>Description:</strong>
The data endpoints that power the admin dashboard header stats and system health views.</p>
<p><strong>Tasks:</strong></p>
<ul>
<li>[ ] Create <code>routers/admin.py</code></li>
<li>[ ] <code>GET /api/admin/dashboard</code> — super admin only
<ul>
<li>Return: active emergencies count, stations by status, today total, auto-dispatched today, avg response time, unresolved SMS failures count</li>
</ul>
</li>
<li>[ ] <code>GET /api/admin/sms-failures</code> — super admin only
<ul>
<li>Paginated list of unresolved SMS failures with emergency context</li>
</ul>
</li>
<li>[ ] <code>PATCH /api/admin/sms-failures/{id}/resolve</code> — super admin only
<ul>
<li>Mark SMS failure as manually resolved with notes</li>
</ul>
</li>
<li>[ ] <code>GET /api/admin/audit-log</code> — super admin only
<ul>
<li>Paginated, filterable by action type and date range</li>
</ul>
</li>
<li>[ ] Register router in <code>main.py</code></li>
</ul>
<p><strong>Acceptance Criteria:</strong></p>
<ul>
<li>[ ] Dashboard stats endpoint returns accurate counts from database</li>
<li>[ ] All admin endpoints return 403 for non-super-admin roles</li>
<li>[ ] Audit log is read-only — no write endpoints</li>
</ul>
<hr>
<h3>Issue #16</h3>
<p><strong>Title:</strong> <code>[TESTING] Write Pytest test suite for Sprint 1 &amp; 2</code>
<strong>Labels:</strong> <code>testing</code>, <code>backend</code>
<strong>Milestone:</strong> Sprint 2 — Core API</p>
<p><strong>Description:</strong>
Write the full backend test suite covering all auth and emergency endpoints, validators, and the PostGIS query.</p>
<p><strong>Tasks:</strong></p>
<ul>
<li>[ ] Create <code>tests/conftest.py</code> — test database setup, async client fixture, user factories</li>
<li>[ ] <code>tests/test_auth.py</code>:
<ul>
<li>Register success</li>
<li>Register with duplicate phone → 409</li>
<li>Login success</li>
<li>Login wrong password → 401</li>
<li>Guest token success</li>
<li>Guest token rate limit → 429</li>
<li>Refresh token success</li>
</ul>
</li>
<li>[ ] <code>tests/test_emergencies.py</code>:
<ul>
<li>Submit with GPS as registered user → 201 with ref number</li>
<li>Submit without GPS but with landmark → 201</li>
<li>Submit as guest → 201 with UNVERIFIED flag</li>
<li>Submit with coords outside Gambia bounds → 422</li>
<li>Guest rate limit second submission → 429</li>
<li>Approve dispatch → status changes to dispatched</li>
<li>Approve from wrong station admin → 403</li>
<li>Resolve emergency → station resets to available</li>
<li>Track by ref number (public) → 200 with no PII</li>
</ul>
</li>
<li>[ ] <code>tests/test_geo.py</code>:
<ul>
<li>Nearest station returned correctly</li>
<li>Excluded station skipped, returns next nearest</li>
<li>All stations unavailable returns None</li>
</ul>
</li>
<li>[ ] <code>tests/test_validators.py</code>:
<ul>
<li>Valid and invalid Gambia coordinates</li>
<li>Valid and invalid phone formats</li>
<li>Rate limit logic</li>
</ul>
</li>
</ul>
<p><strong>Acceptance Criteria:</strong></p>
<ul>
<li>[ ] <code>pytest tests/ -v</code> passes with 0 failures</li>
<li>[ ] Test coverage above 80% on all service and router files</li>
<li>[ ] Tests use test database, never the dev database</li>
</ul>
<hr>
<h2>SPRINT 3 — Celery &amp; Alerts</h2>
<p><strong>Milestone:</strong> Sprint 3 — Celery &amp; Alerts
<strong>Goal:</strong> SMS fires in background, 40s auto-approve works, Socket.IO alarm reaches dashboard.</p>
<hr>
<h3>Issue #17</h3>
<p><strong>Title:</strong> <code>[INFRA] Set up Celery with Redis broker</code>
<strong>Labels:</strong> <code>infra</code>, <code>backend</code>
<strong>Milestone:</strong> Sprint 3 — Celery &amp; Alerts</p>
<p><strong>Description:</strong>
Configure Celery as the background task system with Redis as broker. This is the foundation for all SMS and auto-approve functionality.</p>
<p><strong>Tasks:</strong></p>
<ul>
<li>[ ] Add Celery + Redis dependencies to <code>requirements.txt</code></li>
<li>[ ] Create <code>tasks/celery_app.py</code>:
<ul>
<li>Celery instance pointing to Redis broker (<code>REDIS_URL</code> from config)</li>
<li>Task serialisation: JSON</li>
<li>Result backend: Redis</li>
<li>Task time limit: 30 seconds (prevents hung tasks)</li>
<li><code>task_ack_late = True</code> (acknowledge after completion, not before)</li>
</ul>
</li>
<li>[ ] Update <code>docker-compose.yml</code> to add Celery worker service:
<pre><code class="language-yaml">celery_worker:  command: celery -A app.tasks.celery_app worker --loglevel=info --concurrency=2
</code></pre></li>
<li>[ ] Test: <code>celery -A app.tasks.celery_app worker</code> starts without errors</li>
<li>[ ] Test: a simple test task runs successfully via <code>.delay()</code></li>
</ul>
<p><strong>Acceptance Criteria:</strong></p>
<ul>
<li>[ ] Celery worker starts and connects to Redis</li>
<li>[ ] Test task completes and result visible in Redis</li>
<li>[ ] Worker handles Redis reconnection on restart</li>
</ul>
<hr>
<h3>Issue #18</h3>
<p><strong>Title:</strong> <code>[BACKEND] Build SMS service with Africa's Talking + retry logic</code>
<strong>Labels:</strong> <code>backend</code>, <code>critical</code>
<strong>Milestone:</strong> Sprint 3 — Celery &amp; Alerts</p>
<p><strong>Description:</strong>
The SMS service that sends all three message types with automatic retry on failure.</p>
<p><strong>Tasks:</strong></p>
<ul>
<li>[ ] Sign up for Africa's Talking sandbox account</li>
<li>[ ] Add <code>africastalking</code> to <code>requirements.txt</code></li>
<li>[ ] Create <code>services/sms.py</code>:
<ul>
<li>Initialise AT SDK from config</li>
<li><code>format_admin_alert(emergency) -&gt; str</code> — formats admin SMS template</li>
<li><code>format_citizen_confirmation(emergency, station_name) -&gt; str</code></li>
<li><code>format_unit_dispatch(emergency) -&gt; str</code></li>
<li><code>format_escalation_alert(emergency) -&gt; str</code></li>
</ul>
</li>
<li>[ ] Create <code>tasks/sms_tasks.py</code> with 4 Celery tasks:
<ul>
<li><code>send_admin_sms_task(emergency_id, phone, message)</code></li>
<li><code>send_citizen_sms_task(emergency_id, phone, message)</code></li>
<li><code>send_unit_sms_task(emergency_id, phone, message)</code></li>
<li><code>send_escalation_sms_task(emergency_id, phone, message)</code></li>
</ul>
</li>
<li>[ ] Each task:
<ul>
<li><code>max_retries=3</code></li>
<li>Retry delays: 5s, 15s, 30s (exponential-ish backoff)</li>
<li>On all retries exhausted: call <code>log_sms_failure()</code> + <code>sentry_sdk.capture_exception()</code></li>
<li>Logs attempt number on each retry</li>
</ul>
</li>
<li>[ ] Implement <code>log_sms_failure(emergency_id, phone, message_type, error, attempts)</code> service function</li>
<li>[ ] Wire SMS tasks into emergency submission endpoint (replace stubs from Issue #12)</li>
<li>[ ] Wire SMS tasks into emergency approve endpoint (replace stubs from Issue #13)</li>
<li>[ ] Test with Africa's Talking sandbox — use test number <code>+2547000000000</code></li>
</ul>
<p><strong>Acceptance Criteria:</strong></p>
<ul>
<li>[ ] Submitting an emergency triggers admin SMS within 3 seconds</li>
<li>[ ] Approving dispatch triggers citizen + unit SMS within 3 seconds</li>
<li>[ ] If AT API call throws, task retries automatically</li>
<li>[ ] After 3 failures, <code>sms_failures</code> table has a new record</li>
<li>[ ] Emergency dispatch is NOT blocked by SMS failure</li>
</ul>
<hr>
<h3>Issue #19</h3>
<p><strong>Title:</strong> <code>[BACKEND] Build 40-second auto-approve Celery task</code>
<strong>Labels:</strong> <code>backend</code>, <code>critical</code>
<strong>Milestone:</strong> Sprint 3 — Celery &amp; Alerts</p>
<p><strong>Description:</strong>
Implement the auto-approve countdown as a Celery ETA (scheduled) task. If no admin approves within 40 seconds, the system dispatches automatically.</p>
<p><strong>Tasks:</strong></p>
<ul>
<li>[ ] Create <code>tasks/dispatch_tasks.py</code></li>
<li>[ ] Implement <code>auto_approve_task(emergency_id)</code> Celery task:
<ul>
<li>Fetch emergency from database</li>
<li>If status is already <code>dispatched</code> → do nothing (admin got there first), return</li>
<li>Select first available unit name from station (or generic "Unit 1")</li>
<li>Set <code>status = dispatched</code>, <code>was_auto_dispatched = True</code></li>
<li>Record <code>dispatched_at</code> on response record</li>
<li>Queue citizen SMS task</li>
<li>Queue unit SMS task</li>
<li>Emit Socket.IO <code>emergency_auto_dispatched</code> event</li>
<li>Log to <code>audit_logs</code> with action <code>AUTO_DISPATCH</code></li>
</ul>
</li>
<li>[ ] Store Celery task ID on emergency record when scheduling:
<ul>
<li>Add <code>celery_task_id</code> column to emergencies table (new migration)</li>
</ul>
</li>
<li>[ ] Wire task scheduling into emergency submission (after DB save):
<pre><code class="language-python">task = auto_approve_task.apply_async(    args=[emergency.id],    countdown=settings.AUTO_APPROVE_SECONDS  # 40)emergency.celery_task_id = task.id
</code></pre></li>
<li>[ ] Wire task revocation into manual approve endpoint:
<pre><code class="language-python">celery_app.control.revoke(emergency.celery_task_id, terminate=True)
</code></pre></li>
<li>[ ] Implement <code>send_auto_approve_warning</code> task that fires at 30 seconds:
<ul>
<li>Emits Socket.IO <code>auto_approve_warning</code> event with <code>seconds_remaining: 10</code></li>
</ul>
</li>
<li>[ ] Schedule warning task at <code>countdown=30</code> alongside main task</li>
</ul>
<p><strong>Acceptance Criteria:</strong></p>
<ul>
<li>[ ] Emergency submitted at T=0, auto-dispatched at exactly T=40s if no admin action</li>
<li>[ ] Manual approval at T=15 cancels the T=40 task — auto-approve does NOT fire</li>
<li>[ ] Socket.IO warning fires at T=30 with <code>seconds_remaining: 10</code></li>
<li>[ ] Auto-dispatched emergencies show <code>was_auto_dispatched = true</code> in DB</li>
</ul>
<hr>
<h3>Issue #20</h3>
<p><strong>Title:</strong> <code>[BACKEND] Build escalation chain — no station available</code>
<strong>Labels:</strong> <code>backend</code>
<strong>Milestone:</strong> Sprint 3 — Celery &amp; Alerts</p>
<p><strong>Description:</strong>
Handle the scenario where no fire station is available anywhere. Escalate to Super Admin via SMS + dashboard.</p>
<p><strong>Tasks:</strong></p>
<ul>
<li>[ ] Update emergency submission endpoint:
<ul>
<li>After <code>find_station_cascade()</code> returns empty (all stations tried, none available)</li>
<li>Set emergency status to <code>escalated</code></li>
<li>Set <code>assigned_station_id = NULL</code></li>
</ul>
</li>
<li>[ ] Create <code>tasks/dispatch_tasks.py</code> — <code>escalate_to_super_admin_task(emergency_id)</code>:
<ul>
<li>Fetch all super admin users from database</li>
<li>Queue <code>send_escalation_sms_task</code> for each super admin phone</li>
<li>Emit Socket.IO <code>escalation_received</code> event to <code>all_stations</code> room</li>
</ul>
</li>
<li>[ ] Wire escalation task into emergency submission when no station found</li>
<li>[ ] <code>GET /api/emergencies</code> — escalated emergencies appear at top for super admin</li>
<li>[ ] Super admin can manually assign any station: <code>PATCH /api/emergencies/{id}/assign</code>
<ul>
<li>Accepts <code>station_id</code>, assigns and re-triggers normal dispatch flow</li>
</ul>
</li>
</ul>
<p><strong>Acceptance Criteria:</strong></p>
<ul>
<li>[ ] Setting all stations to <code>unavailable</code>, then submitting triggers escalation SMS to super admin</li>
<li>[ ] Escalated emergency visible in super admin dashboard with distinct badge</li>
<li>[ ] Super admin can manually assign a station to escalated emergency</li>
</ul>
<hr>
<h3>Issue #21</h3>
<p><strong>Title:</strong> <code>[BACKEND] Set up Socket.IO server and all real-time events</code>
<strong>Labels:</strong> <code>backend</code>
<strong>Milestone:</strong> Sprint 3 — Celery &amp; Alerts</p>
<p><strong>Description:</strong>
Configure Socket.IO server mounted on FastAPI, implement all rooms, and build all emit functions.</p>
<p><strong>Tasks:</strong></p>
<ul>
<li>[ ] Add <code>python-socketio</code> to <code>requirements.txt</code></li>
<li>[ ] Mount Socket.IO ASGI app on FastAPI in <code>main.py</code></li>
<li>[ ] Implement JWT authentication middleware on Socket.IO connect event</li>
<li>[ ] Reject connections with invalid/expired tokens</li>
<li>[ ] Create <code>services/socket.py</code> with emit functions:
<ul>
<li><code>emit_new_emergency(station_id, emergency_data)</code> → room <code>station:{station_id}</code></li>
<li><code>emit_auto_approve_warning(station_id, emergency_id)</code> → room <code>station:{station_id}</code></li>
<li><code>emit_auto_dispatched(station_id, emergency_id)</code> → room <code>station:{station_id}</code></li>
<li><code>emit_emergency_dispatched(emergency_id, data)</code> → room <code>emergency:{emergency_id}</code></li>
<li><code>emit_status_update(emergency_id, status)</code> → room <code>emergency:{emergency_id}</code></li>
<li><code>emit_escalation(emergency_data)</code> → room <code>all_stations</code></li>
<li><code>emit_station_status_changed(station_id, status)</code> → room <code>all_stations</code></li>
</ul>
</li>
<li>[ ] Handle client events:
<ul>
<li><code>admin_join</code> → join <code>station:{station_id}</code> room</li>
<li><code>superadmin_join</code> → join <code>all_stations</code> room</li>
<li><code>citizen_track</code> → join <code>emergency:{emergency_id}</code> room</li>
</ul>
</li>
<li>[ ] Wire all emit calls into relevant Celery tasks and endpoints</li>
<li>[ ] Test with simple Socket.IO test client</li>
</ul>
<p><strong>Acceptance Criteria:</strong></p>
<ul>
<li>[ ] Admin connects with valid JWT, joins correct station room</li>
<li>[ ] Submitting an emergency fires <code>new_emergency</code> to correct admin room only</li>
<li>[ ] Citizen receives <code>emergency_dispatched</code> event when their report is approved</li>
<li>[ ] Invalid JWT on connect → connection rejected, not silently allowed</li>
</ul>
<hr>
<h3>Issue #22</h3>
<p><strong>Title:</strong> <code>[TESTING] Write tests for Celery tasks, SMS retry, and auto-approve</code>
<strong>Labels:</strong> <code>testing</code>, <code>backend</code>
<strong>Milestone:</strong> Sprint 3 — Celery &amp; Alerts</p>
<p><strong>Description:</strong>
Test all async background task behaviour including SMS retry logic and the 40-second auto-approve flow.</p>
<p><strong>Tasks:</strong></p>
<ul>
<li>[ ] <code>tests/test_sms.py</code>:
<ul>
<li>SMS task succeeds on first attempt</li>
<li>SMS task retries on AT SDK exception</li>
<li>SMS task logs failure after 3 retries (mock AT to always fail)</li>
<li>Failure logged to <code>sms_failures</code> table correctly</li>
<li>Emergency status not affected by SMS failure</li>
</ul>
</li>
<li>[ ] <code>tests/test_dispatch.py</code>:
<ul>
<li>Auto-approve fires when no manual approval within timeout</li>
<li>Manual approval before timeout cancels auto-approve</li>
<li>Auto-dispatched emergency has <code>was_auto_dispatched = true</code></li>
<li>Escalation fires when all stations unavailable</li>
<li>Super admin receives escalation SMS</li>
</ul>
</li>
<li>[ ] Use <code>celery.contrib.pytest</code> or mock Celery tasks in <code>eager</code> mode for tests</li>
</ul>
<p><strong>Acceptance Criteria:</strong></p>
<ul>
<li>[ ] All tests pass with mocked Africa's Talking SDK</li>
<li>[ ] Auto-approve timing verified with mocked Celery countdown</li>
<li>[ ] No real SMS sent during test runs</li>
</ul>
<hr>
<h2>SPRINT 4 — Frontend</h2>
<p><strong>Milestone:</strong> Sprint 4 — Frontend
<strong>Goal:</strong> Full citizen and admin UI, connected to real backend and Socket.IO.</p>
<hr>
<h3>Issue #23</h3>
<p><strong>Title:</strong> <code>[FRONTEND] Bootstrap React + Vite + Tailwind + PWA project</code>
<strong>Labels:</strong> <code>frontend</code>, <code>setup</code>
<strong>Milestone:</strong> Sprint 4 — Frontend</p>
<p><strong>Description:</strong>
Set up the complete frontend project with all dependencies, routing, PWA configuration, and folder structure.</p>
<p><strong>Tasks:</strong></p>
<ul>
<li>[ ] Create Vite + React project inside <code>frontend/</code></li>
<li>[ ] Install all dependencies: React Router, Zustand, Axios, Socket.IO client, Leaflet, React-Leaflet, Tailwind</li>
<li>[ ] Configure Tailwind CSS — <code>tailwind.config.js</code> with custom colours matching FireAlert brand</li>
<li>[ ] Create <code>public/manifest.json</code> — PWA manifest:
<ul>
<li><code>name: "FireAlert GM"</code>, <code>short_name: "FireAlert"</code></li>
<li><code>theme_color: "#ff2b2b"</code>, <code>background_color: "#0a0a0a"</code></li>
<li><code>display: "standalone"</code>, <code>start_url: "/"</code></li>
<li>Icon entries for 192x192 and 512x512</li>
</ul>
</li>
<li>[ ] Create PWA icons (red fire icon, 192 and 512px)</li>
<li>[ ] Add PWA meta tags in <code>index.html</code></li>
<li>[ ] Create all page and component files as empty stubs</li>
<li>[ ] Set up React Router in <code>App.jsx</code> with all routes defined</li>
<li>[ ] Add auth guard HOC — redirects to login if no token</li>
<li>[ ] Create <code>.env.example</code> for frontend</li>
</ul>
<p><strong>Acceptance Criteria:</strong></p>
<ul>
<li>[ ] <code>npm run dev</code> starts without errors</li>
<li>[ ] All routes render without crashing (even if empty)</li>
<li>[ ] PWA manifest validates in Chrome DevTools → Application tab</li>
<li>[ ] Tailwind classes apply correctly</li>
</ul>
<hr>
<h3>Issue #24</h3>
<p><strong>Title:</strong> <code>[FRONTEND] Build Axios API service with JWT interceptor and silent refresh</code>
<strong>Labels:</strong> <code>frontend</code>
<strong>Milestone:</strong> Sprint 4 — Frontend</p>
<p><strong>Description:</strong>
The central API service layer. Every component calls this — it must handle auth automatically.</p>
<p><strong>Tasks:</strong></p>
<ul>
<li>[ ] Create <code>services/api.js</code>:
<ul>
<li>Axios instance with <code>baseURL</code> from <code>VITE_API_BASE_URL</code></li>
<li>Request interceptor: attach <code>Authorization: Bearer &lt;token&gt;</code> from authStore</li>
<li>Response interceptor: on 401 → call <code>/auth/refresh</code> silently → retry original request</li>
<li>If refresh fails (refresh token expired) → clear authStore → redirect to <code>/login</code></li>
<li>Export named functions for every API endpoint (not raw axios calls in components)</li>
</ul>
</li>
<li>[ ] Named functions to implement:
<ul>
<li><code>authAPI.register(data)</code>, <code>authAPI.login(data)</code>, <code>authAPI.guestToken(phone)</code></li>
<li><code>emergencyAPI.submit(data)</code>, <code>emergencyAPI.track(refNumber)</code>, <code>emergencyAPI.approve(id, data)</code>, <code>emergencyAPI.updateStatus(id, status)</code></li>
<li><code>stationAPI.list()</code>, <code>stationAPI.updateStatus(id, status)</code></li>
<li><code>adminAPI.dashboard()</code>, <code>adminAPI.smsFailures()</code></li>
</ul>
</li>
</ul>
<p><strong>Acceptance Criteria:</strong></p>
<ul>
<li>[ ] A 401 response triggers silent refresh and retries without user seeing login screen</li>
<li>[ ] If refresh fails, user is redirected to login</li>
<li>[ ] All API functions handle network errors gracefully (return error, don't throw uncaught)</li>
</ul>
<hr>
<h3>Issue #25</h3>
<p><strong>Title:</strong> <code>[FRONTEND] Build Socket.IO client and useSocket hook</code>
<strong>Labels:</strong> <code>frontend</code>
<strong>Milestone:</strong> Sprint 4 — Frontend</p>
<p><strong>Description:</strong>
The real-time layer. Must authenticate on connect and handle all events cleanly.</p>
<p><strong>Tasks:</strong></p>
<ul>
<li>[ ] Create <code>services/socket.js</code>:
<ul>
<li>Socket.IO client singleton</li>
<li>Connects with <code>auth: { token }</code> from authStore</li>
<li>Reconnects automatically on disconnect</li>
<li><code>connect()</code> and <code>disconnect()</code> functions</li>
</ul>
</li>
<li>[ ] Create <code>hooks/useSocket.js</code>:
<ul>
<li>Connects socket on mount, disconnects on unmount</li>
<li>Emits <code>admin_join</code> with station_id for admin users</li>
<li>Emits <code>citizen_track</code> with emergency_id for tracking screen</li>
<li>Exposes event listener registration with automatic cleanup</li>
</ul>
</li>
<li>[ ] Create <code>hooks/useCountdown.js</code>:
<ul>
<li>Accepts <code>seconds</code> and <code>onExpire</code> callback</li>
<li>Returns <code>{ remaining, isWarning }</code> (<code>isWarning</code> when ≤ 10s)</li>
<li>Counts down in real time using <code>setInterval</code></li>
<li>Stops when <code>remaining === 0</code> or component unmounts</li>
</ul>
</li>
</ul>
<p><strong>Acceptance Criteria:</strong></p>
<ul>
<li>[ ] Admin dashboard receives <code>new_emergency</code> events in real time</li>
<li>[ ] Countdown reaches 0 and <code>onExpire</code> fires exactly once</li>
<li>[ ] Socket disconnects cleanly when component unmounts (no memory leaks)</li>
</ul>
<hr>
<h3>Issue #26</h3>
<p><strong>Title:</strong> <code>[FRONTEND] Build auth store and auth pages — Login and Register</code>
<strong>Labels:</strong> <code>frontend</code>
<strong>Milestone:</strong> Sprint 4 — Frontend</p>
<p><strong>Description:</strong>
Global auth state and the two authentication pages citizens and admins use to access the system.</p>
<p><strong>Tasks:</strong></p>
<ul>
<li>[ ] Create <code>store/authStore.js</code> with Zustand:
<ul>
<li>State: <code>user</code>, <code>accessToken</code>, <code>refreshToken</code>, <code>role</code>, <code>isAuthenticated</code></li>
<li>Actions: <code>setAuth(user, tokens)</code>, <code>clearAuth()</code>, <code>setAccessToken(token)</code></li>
<li>Persist to <code>sessionStorage</code> (not localStorage)</li>
</ul>
</li>
<li>[ ] Create <code>hooks/useAuth.js</code>:
<ul>
<li>Reads from authStore</li>
<li><code>isAdmin</code>, <code>isSuperAdmin</code>, <code>isGuest</code> computed booleans</li>
</ul>
</li>
<li>[ ] Build <code>pages/Login.jsx</code>:
<ul>
<li>Phone + password form</li>
<li>Calls <code>authAPI.login()</code></li>
<li>On success: store tokens, redirect based on role
<ul>
<li><code>station_admin</code> → <code>/dashboard</code></li>
<li><code>citizen</code> → <code>/report</code></li>
<li><code>super_admin</code> → <code>/dashboard</code></li>
</ul>
</li>
<li>Error state: "Invalid phone or password"</li>
</ul>
</li>
<li>[ ] Build <code>pages/Register.jsx</code>:
<ul>
<li>Full name, phone, region, national ID, password, confirm password</li>
<li>Client-side validation before submit</li>
<li>On success: auto-login and redirect to <code>/report</code></li>
<li>Link to login page</li>
</ul>
</li>
</ul>
<p><strong>Acceptance Criteria:</strong></p>
<ul>
<li>[ ] Login redirects admin to dashboard, citizen to report page</li>
<li>[ ] Register validates all fields before making API call</li>
<li>[ ] Auth state persists on page refresh (via sessionStorage)</li>
<li>[ ] Logout clears all stored tokens and redirects to login</li>
</ul>
<hr>
<h3>Issue #27</h3>
<p><strong>Title:</strong> <code>[FRONTEND] Build citizen emergency report page — SOS button and full form</code>
<strong>Labels:</strong> <code>frontend</code>, <code>critical</code>
<strong>Milestone:</strong> Sprint 4 — Frontend</p>
<p><strong>Description:</strong>
The primary citizen-facing page. Must be fast, simple, and work on low-end Android browsers.</p>
<p><strong>Tasks:</strong></p>
<ul>
<li>[ ] Create <code>hooks/useGPS.js</code>:
<ul>
<li>Calls <code>navigator.geolocation.getCurrentPosition()</code></li>
<li>Returns <code>{ coords, accuracy, loading, error }</code></li>
<li>Auto-triggers on hook mount</li>
<li>Error state with clear message if permission denied</li>
</ul>
</li>
<li>[ ] Build <code>components/SOSButton.jsx</code>:
<ul>
<li>Large circular red button</li>
<li>On tap: immediately get GPS, submit emergency with type=fire, severity=critical</li>
<li>Shows loading state while GPS acquiring</li>
<li>Single tap — no confirmation dialog (every second counts)</li>
</ul>
</li>
<li>[ ] Build <code>components/EmergencyForm.jsx</code>:
<ul>
<li>Emergency type selector (Fire / Rescue / Hazmat / Other)</li>
<li>Severity selector (Low / Medium / High / Critical)</li>
<li>GPS coordinates display with accuracy indicator</li>
<li>Landmark text input (required if no GPS)</li>
<li>Region dropdown (all 6 Gambian regions)</li>
<li>Description textarea (optional)</li>
<li>Phone number field (required for guest, pre-filled for registered)</li>
</ul>
</li>
<li>[ ] Build <code>pages/ReportEmergency.jsx</code>:
<ul>
<li>Renders SOSButton at top</li>
<li>Renders EmergencyForm below</li>
<li>On successful submit: show success screen with ref number + "Track Emergency" link</li>
<li>Guest users see phone input; registered users see pre-filled name</li>
</ul>
</li>
</ul>
<p><strong>Acceptance Criteria:</strong></p>
<ul>
<li>[ ] SOS button submits within 3 taps/seconds on a real phone</li>
<li>[ ] Form submits successfully with landmark only (no GPS)</li>
<li>[ ] Success screen shows <code>GM-XXXXX</code> ref number</li>
<li>[ ] Page works in Chrome Android on low-end device</li>
</ul>
<hr>
<h3>Issue #28</h3>
<p><strong>Title:</strong> <code>[FRONTEND] Build citizen emergency tracking page</code>
<strong>Labels:</strong> <code>frontend</code>
<strong>Milestone:</strong> Sprint 4 — Frontend</p>
<p><strong>Description:</strong>
The page where citizens track the status of their submitted emergency in real time.</p>
<p><strong>Tasks:</strong></p>
<ul>
<li>[ ] Build <code>pages/TrackEmergency.jsx</code>:
<ul>
<li>Accepts ref number from URL param: <code>/track/GM-XXXXX</code></li>
<li>Calls <code>emergencyAPI.track(refNumber)</code> on load</li>
<li>If not found: "Ref number not found" state</li>
<li>Connects to Socket.IO room <code>emergency:{id}</code> via <code>useSocket</code></li>
<li>Listens for <code>emergency_dispatched</code> and <code>status_update</code> events</li>
<li>Updates status display in real time without page refresh</li>
</ul>
</li>
<li>[ ] Status display states:
<ul>
<li><code>pending</code> → "Your report has been received. Alerting nearest station..."</li>
<li><code>dispatched</code> → "Help is on the way. Unit dispatched by [Station Name]."</li>
<li><code>en_route</code> → "Fire unit is en route to your location."</li>
<li><code>on_scene</code> → "Fire unit has arrived."</li>
<li><code>resolved</code> → "Emergency resolved. Thank you."</li>
<li><code>escalated</code> → "Your report has been escalated to national HQ. Help is being coordinated."</li>
</ul>
</li>
</ul>
<p><strong>Acceptance Criteria:</strong></p>
<ul>
<li>[ ] Status updates live in browser when backend fires Socket.IO event</li>
<li>[ ] Page loads with correct status even without Socket.IO (API fetch on load)</li>
<li>[ ] All 6 status states render with correct messaging</li>
</ul>
<hr>
<h3>Issue #29</h3>
<p><strong>Title:</strong> <code>[FRONTEND] Build station admin dashboard — emergency queue and map</code>
<strong>Labels:</strong> <code>frontend</code>, <code>critical</code>
<strong>Milestone:</strong> Sprint 4 — Frontend</p>
<p><strong>Description:</strong>
The most complex page. Station admin sees emergencies in real time, hears alarm, and dispatches from here.</p>
<p><strong>Tasks:</strong></p>
<ul>
<li>[ ] Create <code>store/emergencyStore.js</code> with Zustand:
<ul>
<li><code>emergencies</code> list, <code>addEmergency()</code>, <code>updateEmergency()</code>, <code>removeEmergency()</code></li>
</ul>
</li>
<li>[ ] Build <code>components/AlarmAlert.jsx</code>:
<ul>
<li>Full-screen overlay on <code>new_emergency</code> Socket.IO event</li>
<li>Alarm sound (use Web Audio API — no file dependency)</li>
<li>Shows emergency type, severity, landmark</li>
<li>"View Emergency" button dismisses alarm and scrolls to card</li>
<li>Auto-dismisses after 10 seconds if not interacted with</li>
</ul>
</li>
<li>[ ] Build <code>components/MapView.jsx</code>:
<ul>
<li>Leaflet map centred on The Gambia</li>
<li>Station pins (blue) showing all stations</li>
<li>Emergency pin (red pulsing) for active emergencies</li>
<li>Popup on emergency pin: type, severity, landmark</li>
<li>Uses OpenStreetMap tiles (free, no API key)</li>
</ul>
</li>
<li>[ ] Build <code>components/CountdownTimer.jsx</code>:
<ul>
<li>Uses <code>useCountdown</code> hook</li>
<li>Visual countdown ring</li>
<li>Turns red when ≤ 10 seconds (warning state)</li>
<li>Shows "AUTO-DISPATCHING..." when expired</li>
</ul>
</li>
<li>[ ] Build <code>components/VerifiedBadge.jsx</code> and <code>components/AutoDispatchBadge.jsx</code></li>
<li>[ ] Build <code>components/EmergencyCard.jsx</code>:
<ul>
<li>Shows all emergency info</li>
<li>Contains <code>CountdownTimer</code></li>
<li>"Approve Dispatch" button opens <code>DispatchModal</code></li>
<li>Shows <code>AutoDispatchBadge</code> if auto-dispatched</li>
</ul>
</li>
<li>[ ] Build <code>components/DispatchModal.jsx</code>:
<ul>
<li>Unit name text input</li>
<li>Optional notes</li>
<li>"Confirm Dispatch" button</li>
<li>Calls <code>emergencyAPI.approve(id, data)</code></li>
<li>On success: removes emergency from queue, shows toast</li>
</ul>
</li>
<li>[ ] Build <code>pages/Dashboard.jsx</code>:
<ul>
<li>Header: station name, available/responding/unavailable stats</li>
<li><code>MapView</code> full-width at top</li>
<li>Emergency queue list below</li>
<li><code>AlarmAlert</code> mounts here and listens to socket</li>
<li>On <code>new_emergency</code> event: add to queue + trigger alarm</li>
</ul>
</li>
</ul>
<p><strong>Acceptance Criteria:</strong></p>
<ul>
<li>[ ] Alarm fires with sound when emergency submitted in another tab</li>
<li>[ ] Countdown counts from 40 and turns red at 10</li>
<li>[ ] Manual approve before 40s removes card from queue</li>
<li>[ ] Auto-dispatch at 40s updates card to show AUTO-DISPATCHED badge</li>
<li>[ ] Map shows correct pin for submitted emergency location</li>
</ul>
<hr>
<h3>Issue #30</h3>
<p><strong>Title:</strong> <code>[FRONTEND] Build emergency detail page and system health page</code>
<strong>Labels:</strong> <code>frontend</code>
<strong>Milestone:</strong> Sprint 4 — Frontend</p>
<p><strong>Description:</strong>
The full emergency view and the super admin system health screen.</p>
<p><strong>Tasks:</strong></p>
<ul>
<li>[ ] Build <code>pages/EmergencyDetail.jsx</code>:
<ul>
<li>All emergency fields displayed</li>
<li>Response timeline: assigned_at, dispatched_at, en_route_at, on_scene_at, resolved_at</li>
<li>Status update buttons (En Route / On Scene / Resolved) for station admin</li>
<li>Audit log entries for this emergency</li>
<li>Map with emergency location pin</li>
</ul>
</li>
<li>[ ] Build <code>pages/SystemHealth.jsx</code> — super admin only:
<ul>
<li>Total emergencies today</li>
<li>Unresolved SMS failures list with resolve button</li>
<li>Station status overview</li>
<li>Backend health indicator (polls <code>/health</code> every 60 seconds)</li>
</ul>
</li>
</ul>
<p><strong>Acceptance Criteria:</strong></p>
<ul>
<li>[ ] Status update buttons call API and update display without page reload</li>
<li>[ ] System health shows correct SMS failure count from backend</li>
<li>[ ] <code>/health</code> indicator turns red if backend returns non-200</li>
</ul>
<hr>
<h2>SPRINT 5 — Production Deploy</h2>
<p><strong>Milestone:</strong> Sprint 5 — Production Deploy
<strong>Goal:</strong> Live on production servers, tested end-to-end on real devices.</p>
<hr>
<h3>Issue #31</h3>
<p><strong>Title:</strong> <code>[INFRA] Write production Dockerfile for backend</code>
<strong>Labels:</strong> <code>infra</code>
<strong>Milestone:</strong> Sprint 5 — Production Deploy</p>
<p><strong>Description:</strong>
Create a production-optimised multi-stage Dockerfile for the FastAPI backend.</p>
<p><strong>Tasks:</strong></p>
<ul>
<li>[ ] Write <code>backend/Dockerfile</code> with multi-stage build:
<ul>
<li>Stage 1 (builder): install dependencies</li>
<li>Stage 2 (production): copy only what's needed, run as non-root user</li>
</ul>
</li>
<li>[ ] Add <code>.dockerignore</code> to exclude <code>__pycache__</code>, <code>.env</code>, <code>tests/</code>, <code>venv/</code></li>
<li>[ ] Write <code>docker-compose.prod.yml</code> with production overrides:
<ul>
<li>No <code>--reload</code> flag on uvicorn</li>
<li>Redis with AOF persistence enabled</li>
<li>Environment variables from <code>.env</code> file</li>
</ul>
</li>
<li>[ ] Test: <code>docker build -t firealert-backend .</code> succeeds</li>
<li>[ ] Test: container starts and <code>/health</code> returns 200</li>
</ul>
<p><strong>Acceptance Criteria:</strong></p>
<ul>
<li>[ ] Docker image builds in under 3 minutes</li>
<li>[ ] Production container runs as non-root user</li>
<li>[ ] Image size under 500MB</li>
</ul>
<hr>
<h3>Issue #32</h3>
<p><strong>Title:</strong> <code>[INFRA] Set up Railway deployment — backend, PostgreSQL, Redis</code>
<strong>Labels:</strong> <code>infra</code>
<strong>Milestone:</strong> Sprint 5 — Production Deploy</p>
<p><strong>Description:</strong>
Deploy the full backend stack to Railway and verify it's running correctly.</p>
<p><strong>Tasks:</strong></p>
<ul>
<li>[ ] Create Railway project and add three services: PostgreSQL, Redis, Backend</li>
<li>[ ] Enable PostGIS on Railway PostgreSQL: <code>CREATE EXTENSION postgis;</code></li>
<li>[ ] Create <code>railway.toml</code> in repository root</li>
<li>[ ] Set all production environment variables in Railway dashboard</li>
<li>[ ] Deploy backend service, run migrations, run station seed</li>
<li>[ ] Add second Railway service for Celery worker</li>
<li>[ ] Verify <code>/health</code> endpoint returns 200 on production URL</li>
<li>[ ] Test SMS via AT sandbox against production backend</li>
<li>[ ] Configure Railway restart policy: <code>on_failure</code></li>
</ul>
<p><strong>Acceptance Criteria:</strong></p>
<ul>
<li>[ ] <code>https://your-api.railway.app/health</code> returns <code>{"status": "ok"}</code></li>
<li>[ ] <code>alembic upgrade head</code> ran successfully on production DB</li>
<li>[ ] All stations seeded in production database</li>
<li>[ ] Celery worker running and processing tasks</li>
</ul>
<hr>
<h3>Issue #33</h3>
<p><strong>Title:</strong> <code>[INFRA] Deploy frontend to Vercel</code>
<strong>Labels:</strong> <code>infra</code>
<strong>Milestone:</strong> Sprint 5 — Production Deploy</p>
<p><strong>Description:</strong>
Deploy the React PWA to Vercel and connect it to the production backend.</p>
<p><strong>Tasks:</strong></p>
<ul>
<li>[ ] Import repository in Vercel dashboard</li>
<li>[ ] Set framework to Vite, root directory to <code>frontend/</code></li>
<li>[ ] Set all production environment variables (<code>VITE_API_BASE_URL</code> pointing to Railway)</li>
<li>[ ] Deploy and verify PWA loads correctly</li>
<li>[ ] Test: install PWA on Android phone via Chrome "Add to Home Screen"</li>
<li>[ ] Verify PWA icon and name appear correctly on home screen</li>
<li>[ ] Check HTTPS is enforced on all API calls from production frontend</li>
</ul>
<p><strong>Acceptance Criteria:</strong></p>
<ul>
<li>[ ] Frontend loads at Vercel production URL</li>
<li>[ ] PWA installs correctly on Android Chrome</li>
<li>[ ] All API calls go to Railway backend over HTTPS</li>
<li>[ ] No mixed content (HTTP/HTTPS) warnings in browser</li>
</ul>
<hr>
<h3>Issue #34</h3>
<p><strong>Title:</strong> <code>[INFRA] Set up GitHub Actions CI/CD pipeline</code>
<strong>Labels:</strong> <code>infra</code>
<strong>Milestone:</strong> Sprint 5 — Production Deploy</p>
<p><strong>Description:</strong>
Automated testing and deployment pipeline so every push to <code>main</code> is tested then deployed.</p>
<p><strong>Tasks:</strong></p>
<ul>
<li>[ ] Create <code>.github/workflows/deploy.yml</code></li>
<li>[ ] Pipeline steps:
<ol>
<li>Checkout code</li>
<li>Set up Python 3.11</li>
<li>Install backend dependencies</li>
<li>Run <code>pytest tests/ -v</code> with test database</li>
<li>On test pass: trigger Railway deploy (via Railway CLI or webhook)</li>
<li>Vercel deploys automatically on push (no action needed)</li>
</ol>
</li>
<li>[ ] Add status badge to README: shows passing/failing</li>
<li>[ ] Test: push to main triggers pipeline, tests run, Railway deploys</li>
</ul>
<p><strong>Acceptance Criteria:</strong></p>
<ul>
<li>[ ] Pipeline runs on every push to <code>main</code></li>
<li>[ ] Failed tests block deploy — code with broken tests never reaches production</li>
<li>[ ] Status badge shows green on README</li>
</ul>
<hr>
<h3>Issue #35</h3>
<p><strong>Title:</strong> <code>[INFRA] Set up monitoring — UptimeRobot and Sentry</code>
<strong>Labels:</strong> <code>infra</code>, <code>critical</code>
<strong>Milestone:</strong> Sprint 5 — Production Deploy</p>
<p><strong>Description:</strong>
Production monitoring so system failures are detected and reported automatically.</p>
<p><strong>Tasks:</strong></p>
<ul>
<li>[ ] Set up UptimeRobot (free tier):
<ul>
<li>Monitor <code>GET /health</code> every 5 minutes</li>
<li>Alert via SMS to developer phone on downtime</li>
<li>Alert via email to GFRS IT contact after 15 minutes downtime</li>
</ul>
</li>
<li>[ ] Set up Sentry (free tier):
<ul>
<li>Add Sentry SDK to backend: <code>pip install sentry-sdk[fastapi]</code></li>
<li>Add <code>SENTRY_DSN</code> to production environment variables</li>
<li>Configure Sentry in <code>main.py</code> on app startup</li>
<li>Test: trigger a deliberate error, verify it appears in Sentry dashboard</li>
</ul>
</li>
<li>[ ] Set up Railway database backup:
<ul>
<li>Enable automatic daily backups in Railway PostgreSQL settings</li>
<li>Verify backup appears in Railway dashboard after 24 hours</li>
</ul>
</li>
</ul>
<p><strong>Acceptance Criteria:</strong></p>
<ul>
<li>[ ] UptimeRobot shows green status for production backend</li>
<li>[ ] A test exception in production appears in Sentry within 30 seconds</li>
<li>[ ] SMS received on developer phone when <code>/health</code> is manually taken down</li>
</ul>
<hr>
<h3>Issue #36</h3>
<p><strong>Title:</strong> <code>[TESTING] Full end-to-end test on production with real devices</code>
<strong>Labels:</strong> <code>testing</code>, <code>critical</code>
<strong>Milestone:</strong> Sprint 5 — Production Deploy</p>
<p><strong>Description:</strong>
The final pre-launch test. Real phones, real Gambian numbers, live Africa's Talking. This is the go/no-go gate.</p>
<p><strong>Tasks:</strong></p>
<ul>
<li>[ ] Switch Africa's Talking from sandbox to live mode</li>
<li>[ ] Test Case 1 — Guest submission with GPS:
<ul>
<li>Submit emergency on real Android phone</li>
<li>Confirm ref number returned</li>
<li>Confirm admin dashboard alarm fires</li>
<li>Confirm admin SMS received within 3 seconds</li>
</ul>
</li>
<li>[ ] Test Case 2 — Manual dispatch:
<ul>
<li>Admin approves on dashboard</li>
<li>Confirm citizen SMS received</li>
<li>Confirm unit SMS received with working Google Maps link</li>
<li>Open Google Maps link — confirm opens to correct coordinates</li>
</ul>
</li>
<li>[ ] Test Case 3 — Auto-dispatch (40 seconds):
<ul>
<li>Submit emergency, admin does nothing</li>
<li>Confirm auto-dispatch fires at T=40s</li>
<li>Confirm citizen SMS sent by auto-dispatch</li>
</ul>
</li>
<li>[ ] Test Case 4 — Escalation:
<ul>
<li>Set all stations to <code>unavailable</code></li>
<li>Submit emergency</li>
<li>Confirm Super Admin SMS received</li>
<li>Confirm escalated status on dashboard</li>
</ul>
</li>
<li>[ ] Test Case 5 — Submission without GPS:
<ul>
<li>Deny GPS permission on phone</li>
<li>Submit with landmark only</li>
<li>Confirm emergency created and dispatched normally</li>
</ul>
</li>
<li>[ ] Test Case 6 — SMS retry:
<ul>
<li>Temporarily set invalid AT credentials</li>
<li>Submit emergency</li>
<li>Confirm retry attempts logged</li>
<li>Confirm SMS failure record created</li>
<li>Restore credentials</li>
</ul>
</li>
<li>[ ] Document any failures found and create bug issues immediately</li>
</ul>
<p><strong>Acceptance Criteria:</strong></p>
<ul>
<li>[ ] All 6 test cases pass on live production servers</li>
<li>[ ] No test case takes more than 60 seconds from submission to unit SMS</li>
<li>[ ] Zero 500 errors in Sentry during test run</li>
<li>[ ] System functional on real Android Chrome browser</li>
</ul>
<hr>
<h3>Issue #37</h3>
<p><strong>Title:</strong> <code>[DOCS] Write station setup guide and admin user guide for GFRS</code>
<strong>Labels:</strong> <code>setup</code>
<strong>Milestone:</strong> Sprint 5 — Production Deploy</p>
<p><strong>Description:</strong>
Non-technical documentation for the GFRS IT team and station dispatchers. Written for people who are not developers.</p>
<p><strong>Tasks:</strong></p>
<ul>
<li>[ ] Write <code>docs/station-setup-guide.md</code>:
<ul>
<li>Step-by-step: how to add a new fire station to the system</li>
<li>How to create a station admin account</li>
<li>How to give a dispatcher their login credentials</li>
<li>How to test a new station with a test emergency</li>
<li>Screenshots for every step</li>
</ul>
</li>
<li>[ ] Write <code>docs/admin-user-guide.md</code>:
<ul>
<li>What the dashboard looks like and what each part does</li>
<li>What to do when the alarm fires</li>
<li>How to approve a dispatch</li>
<li>How to update emergency status (en route, on scene, resolved)</li>
<li>What VERIFIED and UNVERIFIED mean</li>
<li>What AUTO-DISPATCHED means</li>
<li>How to change station status (unavailable for maintenance)</li>
<li>FAQ: common situations and what to do</li>
</ul>
</li>
<li>[ ] Write <code>docs/handover-checklist.md</code>:
<ul>
<li>Complete checklist for production go-live</li>
<li>Ongoing maintenance tasks (monthly checks, credit top-ups)</li>
<li>Emergency contact procedure if system is down</li>
</ul>
</li>
</ul>
<p><strong>Acceptance Criteria:</strong></p>
<ul>
<li>[ ] A non-technical dispatcher can use the admin dashboard after reading the guide</li>
<li>[ ] Station setup guide is clear enough for GFRS IT team to add a new station independently</li>
<li>[ ] All screenshots match current UI</li>
</ul>
<hr>
<h2>Summary Table</h2>

Issue | Title | Sprint | Label | Priority
-- | -- | -- | -- | --
#1 | Initialise repository structure | 1 | setup | medium
#2 | Docker Compose local dev setup | 1 | infra | high
#3 | FastAPI project + DB connection | 1 | backend | high
#4 | SQLAlchemy models (core 4) | 1 | database | critical
#5 | audit_log + sms_failure models | 1 | database | high
#6 | Alembic initial migration | 1 | database | critical
#7 | Seed Gambia fire stations | 1 | database | critical
#8 | PostGIS nearest-station query | 1 | database | critical
#9 | GPS, phone, rate limit validators | 1 | backend | high
#10 | JWT + bcrypt security utils | 2 | security | critical
#11 | Auth router (register/login/refresh/guest) | 2 | backend | critical
#12 | Emergency submission endpoint | 2 | backend | critical
#13 | Emergency status + tracking endpoints | 2 | backend | critical
#14 | Stations router | 2 | backend | high
#15 | Admin dashboard router | 2 | backend | medium
#16 | Pytest test suite Sprint 1 & 2 | 2 | testing | high
#17 | Celery + Redis setup | 3 | infra | critical
#18 | SMS service + retry logic | 3 | backend | critical
#19 | 40-second auto-approve task | 3 | backend | critical
#20 | Escalation chain | 3 | backend | high
#21 | Socket.IO server + events | 3 | backend | critical
#22 | Tests: Celery, SMS, auto-approve | 3 | testing | high
#23 | React + Vite + PWA bootstrap | 4 | frontend | high
#24 | Axios API service + JWT refresh | 4 | frontend | critical
#25 | Socket.IO client + useSocket hook | 4 | frontend | critical
#26 | Auth store + Login + Register | 4 | frontend | high
#27 | Citizen report page + SOS | 4 | frontend | critical
#28 | Citizen tracking page | 4 | frontend | high
#29 | Admin dashboard + alarm + map | 4 | frontend | critical
#30 | Emergency detail + system health | 4 | frontend | medium
#31 | Production Dockerfile | 5 | infra | high
#32 | Railway backend deployment | 5 | infra | critical
#33 | Vercel frontend deployment | 5 | infra | critical
#34 | GitHub Actions CI/CD | 5 | infra | high
#35 | UptimeRobot + Sentry monitoring | 5 | infra | critical
#36 | End-to-end production test | 5 | testing | critical
#37 | GFRS handover documentation | 5 | setup | high


<p><strong>Total: 37 issues across 5 sprints</strong></p>
<hr>
<p><em>Each issue is a single closeable unit of work. Start with Sprint 1, work top to bottom.</em>
<em>When all 37 issues are closed, FireAlert GM is production ready. 🇬🇲</em></p></body></html><!--EndFragment-->
</body>
</html>

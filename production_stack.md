┌─────────────────────────────────────────────────────────────┐
│                    PRODUCTION STACK                         │
│                                                             │
│  Railway                                                    │
│  ├── FastAPI (uvicorn, 2 workers)                           │
│  ├── Celery Worker (task queue)                             │
│  ├── Celery Beat (scheduled tasks)                          │
│  ├── Redis (task broker + cache)                            │
│  └── PostgreSQL 15 + PostGIS                                │
│                                                             │
│  Vercel                                                     │
│  └── React PWA (static)                                     │
│                                                             │
│  External Services                                          │
│  ├── Africa's Talking (SMS, retry x3)                       │
│  ├── UptimeRobot (uptime monitoring → SMS to you)           │
│  ├── Sentry (error tracking)                                │
│  └── Railway auto-backups (daily PostgreSQL dump)           │
└─────────────────────────────────────────────────────────────┘

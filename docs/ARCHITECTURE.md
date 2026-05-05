# Architecture

This document describes the system architecture and design decisions for the Task Manager API.

## System Overview

```
┌─────────────────────────────────────────────────────────────┐
│                     Client Applications                       │
│            (Web, Mobile, Third-party Integrations)            │
└────────────────────────┬────────────────────────────────────┘
                         │
                    HTTPS/TLS
                         │
        ┌────────────────▼────────────────┐
        │    Load Balancer / Reverse Proxy  │  (Nginx / Render)
        │    - SSL Termination              │
        │    - Static File Serving          │
        │    - Compression                  │
        └────────────────┬────────────────┘
                         │
            ┌────────────▼────────────┐
            │   Django Application    │
            │   - REST API Endpoints  │
            │   - Authentication      │
            │   - Business Logic      │
            │   - Serialization       │
            └────────────┬────────────┘
                         │
        ┌────────────────┴────────────────┐
        │                                 │
    ┌───▼────────┐            ┌──────────▼────┐
    │  PostgreSQL │            │   File Storage │
    │  Database   │            │   (staticfiles)│
    │  - Tasks    │            │                │
    │  - Users    │            │                │
    │  - Auth     │            │                │
    └────────────┘            └────────────────┘
```

## Layered Architecture

The application follows a clean, layered architecture:

```
┌─────────────────────────────────────────────┐
│         API Views / ViewSets (REST)         │  ← Request Handling
├─────────────────────────────────────────────┤
│      Serializers (Validation & Conversion)  │  ← Data Transformation
├─────────────────────────────────────────────┤
│     Business Logic / Service Layer          │  ← Core Logic
├─────────────────────────────────────────────┤
│           Database Models & ORM             │  ← Data Layer
├─────────────────────────────────────────────┤
│         Database (PostgreSQL/SQLite)        │  ← Persistence
└─────────────────────────────────────────────┘
```

## Directory Structure

```
task-manager-api/
│
├── config/                      # Django Configuration
│   ├── settings.py             # Main settings
│   ├── urls.py                 # Root URL routing
│   ├── wsgi.py                 # WSGI entry point
│   └── asgi.py                 # ASGI entry point
│
├── task/                        # Task Management App
│   ├── models.py               # Task model definition
│   │   └── Task                # Main model with fields
│   ├── serializers.py          # DRF serializers
│   │   └── TaskSerializer      # Convert Task ↔ JSON
│   ├── views.py                # API views
│   │   └── TaskViewSet         # Complete CRUD + custom actions
│   ├── urls.py                 # App URL routing
│   ├── admin.py                # Django admin config
│   ├── apps.py                 # App configuration
│   ├── tests.py                # Unit tests
│   └── migrations/             # Database migrations
│
├── staticfiles/                # Collected static files
│   ├── admin/
│   ├── rest_framework/
│   └── drf-yasg/
│
├── manage.py                   # Django CLI
├── requirements.txt            # Python dependencies
├── render.yaml                 # Render deployment
├── Dockerfile                  # Docker image
├── docker-compose.yml          # Local dev setup
├── docker-compose.prod.yml     # Production setup
├── pyproject.toml              # Python packaging
├── README.md                   # Main documentation
├── DEPLOYMENT.md               # Deployment guide
├── CONTRIBUTING.md             # Contributing guide
├── LICENSE                     # MIT License
│
└── docs/
    ├── API_EXAMPLES.md         # Usage examples
    ├── SECURITY.md             # Security guide
    ├── TESTING.md              # Testing guide
    └── README.md               # Docs index
```

## Data Model

### Task Model

```
Task
├── id (PK)
├── title (CharField, max_length=200)
├── description (TextField, optional)
├── status (CharField: Pending, In Progress, Completed)
├── priority (CharField: Low, Medium, High)
├── due_date (DateField, optional)
├── owner (ForeignKey → User)
├── created_at (DateTimeField, auto_now_add)
├── updated_at (DateTimeField, auto_now)
└── Indexes:
    ├── owner + created_at
    ├── status
    ├── priority
    └── due_date
```

## Authentication Flow

```
1. User Login
   POST /api/token/
   {username, password} → {access_token, refresh_token}

2. Authenticated Request
   GET /api/tasks/
   Headers: Authorization: Bearer {access_token}

3. Token Validation
   JWTAuthentication middleware
   ├── Decode token
   ├── Verify signature
   ├── Check expiration
   └── Load user from payload

4. Token Refresh
   POST /api/token/refresh/
   {refresh_token} → {new_access_token}
```

## Request/Response Flow

```
Request
  │
  ├─ URL Routing (urls.py)
  ├─ Middleware Stack
  │  ├─ Security Middleware
  │  ├─ SessionMiddleware
  │  ├─ AuthenticationMiddleware
  │  └─ WhiteNoiseMiddleware
  │
  ├─ ViewSet.dispatch()
  │  ├─ Check authentication
  │  ├─ Check permissions
  │  ├─ Check throttling
  │  └─ Route to action method
  │
  ├─ Serializer Validation
  │  ├─ Deserialize input
  │  ├─ Validate data
  │  └─ Create/update model
  │
  ├─ Business Logic
  │  ├─ Process data
  │  ├─ Apply filters
  │  └─ Fetch results
  │
  ├─ Serialization
  │  ├─ Serialize model(s)
  │  └─ Format JSON
  │
  └─ Response
     └─ Status code + Data
```

## Database Query Optimization

### Indexes

```sql
-- Performance optimization
CREATE INDEX task_owner_idx ON task_task(owner_id);
CREATE INDEX task_status_idx ON task_task(status);
CREATE INDEX task_priority_idx ON task_task(priority);
CREATE INDEX task_due_date_idx ON task_task(due_date);
```

### Query Patterns

```python
# Good: Efficient queries
tasks = Task.objects.filter(
    owner=user,
    status='Pending'
).select_related('owner').order_by('-due_date')

# Bad: N+1 queries
for task in Task.objects.all():
    print(task.owner.name)  # Query for each task
```

## API Versioning Strategy

Current: V1 (implicit in URL structure)

Future versioning options:
- Header-based: `Accept: application/json; version=2`
- URL-based: `/api/v2/tasks/`
- Query param: `?version=2`

## Security Architecture

```
┌─────────────────────────────────────┐
│       HTTPS / TLS Layer             │
├─────────────────────────────────────┤
│    Django Security Middleware       │
│    - CSRF Protection                │
│    - XFrame Options                 │
│    - Security Headers               │
├─────────────────────────────────────┤
│    Authentication Layer             │
│    - JWT Token Validation           │
│    - User Identification            │
├─────────────────────────────────────┤
│    Authorization Layer              │
│    - Permission Classes             │
│    - Object-level Permissions       │
├─────────────────────────────────────┤
│    Input Validation                 │
│    - Serializer Validation          │
│    - Type Checking                  │
└─────────────────────────────────────┘
```

## Deployment Architectures

### Render (Recommended)

```
GitHub Repository
       ↓
    Render
       ├─ Managed PostgreSQL
       ├─ Gunicorn Application Server
       ├─ Automatic SSL/HTTPS
       ├─ CDN for Static Files
       └─ Auto-scaling
```

### Docker Compose

```
Docker Network
├─ PostgreSQL Container
│  └─ Persistent Volume
├─ Django Container
│  ├─ Gunicorn Workers
│  └─ Mount Application
└─ Nginx Container
   ├─ Reverse Proxy
   ├─ Static Files
   └─ SSL Termination
```

### Kubernetes (Advanced)

```
Kubernetes Cluster
├─ Ingress
│  └─ Service (Load Balancer)
├─ Deployment (Replicas)
│  ├─ Django Pod 1
│  ├─ Django Pod 2
│  └─ Django Pod 3
├─ StatefulSet
│  └─ PostgreSQL Pod
└─ ConfigMap + Secrets
   ├─ Settings
   └─ Credentials
```

## Performance Considerations

### Caching Strategy

```python
# Static file caching (Nginx)
Cache-Control: public, immutable, max-age=31536000

# API response caching (Consider adding)
from django.views.decorators.cache import cache_page
@cache_page(60 * 15)  # 15 minutes
def list_tasks(request):
    pass
```

### Database Optimization

```python
# Use select_related for ForeignKeys
Task.objects.select_related('owner')

# Use prefetch_related for reverse relations
User.objects.prefetch_related('tasks')

# Use only() for partial loading
Task.objects.only('id', 'title', 'status')
```

### Query Monitoring

```python
# Enable query logging in development
LOGGING = {
    'version': 1,
    'handlers': {
        'console': {'class': 'logging.StreamHandler'},
    },
    'loggers': {
        'django.db.backends': {
            'handlers': ['console'],
            'level': 'DEBUG',
        },
    },
}
```

## Scalability Considerations

### Current State
- Single-server deployment
- SQLite for development, PostgreSQL for production
- Gunicorn with 3-4 workers

### Future Scaling
- Multiple application servers (load balanced)
- Database read replicas
- Redis caching layer
- Celery for async tasks
- Elasticsearch for advanced search

## Technology Decisions

| Decision | Choice | Rationale |
|----------|--------|-----------|
| Framework | Django | Mature, batteries-included, excellent ORM |
| API | DRF | Industry standard, excellent serialization |
| Auth | JWT | Stateless, scalable, mobile-friendly |
| Database | PostgreSQL | Reliable, performant, ACID compliant |
| Server | Gunicorn | Production-ready, lightweight, scalable |
| Static Files | WhiteNoise | No external dependencies, CDN-friendly |
| Deployment | Render | Simple, reliable, excellent DX |

---

**Last Updated**: May 2026

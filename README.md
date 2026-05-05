# Task Manager API

A production-ready Django REST API for managing tasks with JWT authentication, advanced filtering, searching, pagination, and interactive API documentation.

## Table of Contents

- [Features](#features)
- [Tech Stack](#tech-stack)
- [Project Structure](#project-structure)
- [Prerequisites](#prerequisites)
- [Local Setup](#local-setup)
- [API Documentation](#api-documentation)
- [API Endpoints](#api-endpoints)
- [Filtering & Searching](#filtering--searching)
- [Authentication](#authentication)
- [Deployment](#deployment)
- [Environment Variables](#environment-variables)
- [Database](#database)
- [Troubleshooting](#troubleshooting)
- [Contributing](#contributing)
- [License](#license)

## Features

- **User Authentication**: Secure JWT-based authentication using `djangorestframework_simplejwt`
- **Task Management**: Complete CRUD operations for tasks with ownership tracking
- **Rich Task Properties**:
  - Title and detailed description
  - Status: Pending, In Progress, Completed
  - Priority levels: Low, Medium, High
  - Due date tracking with timezone support
  - Automatic timestamps (created_at, updated_at)
- **Advanced Filtering**: Filter tasks by status, priority, due date, and ownership
- **Search Functionality**: Full-text search across task titles and descriptions
- **Flexible Ordering**: Sort by creation date, due date, priority, and more
- **Pagination**: Customizable pagination (default: 10 items per page)
- **Interactive API Documentation**: 
  - Swagger UI for interactive API testing
  - ReDoc for clean, readable API reference
- **Production-Ready Security**:
  - CSRF protection
  - XFrame options protection
  - Secure headers configuration
  - SSL/TLS proxy headers support
  - Environment-based configuration
- **Static File Management**: Optimized with WhiteNoise for production

## Tech Stack

| Component | Technology | Version |
|-----------|-----------|---------|
| **Backend Framework** | Django | 6.0.3 |
| **API Framework** | Django REST Framework | 3.16.1 |
| **Authentication** | Simple JWT | 5.5.1 |
| **Database** | SQLite3 (dev), PostgreSQL (prod) | - |
| **API Documentation** | drf-yasg | 1.21.15 |
| **Static Files** | WhiteNoise | 6.12.0 |
| **Database URL** | dj-database-url | 3.0.1 |
| **Filtering** | django-filter | 25.2 |
| **Production Server** | Gunicorn | 25.1.0 |
| **Deployment** | Render | - |
| **Python** | 3.13.3 | - |

## Project Structure

```
task-manager-api/
├── config/                          # Django project configuration
│   ├── __init__.py
│   ├── settings.py                 # Main Django settings
│   ├── urls.py                     # URL routing configuration
│   ├── asgi.py                     # ASGI config for async servers
│   ├── wsgi.py                     # WSGI config for production
│
├── task/                           # Task management app
│   ├── __init__.py
│   ├── admin.py                    # Django admin configuration
│   ├── apps.py                     # App configuration
│   ├── models.py                   # Task model definition
│   ├── serializers.py              # DRF serializers for API
│   ├── urls.py                     # Task app URL routing
│   ├── views.py                    # API views and logic
│   ├── tests.py                    # Unit tests
│   └── migrations/                 # Database migrations
│
├── staticfiles/                     # Collected static files (production)
│   ├── admin/
│   ├── rest_framework/
│   └── drf-yasg/
│
├── manage.py                        # Django management script
├── db.sqlite3                       # SQLite database (dev only)
├── requirements.txt                 # Python dependencies
├── render.yaml                      # Render deployment blueprint
├── Procfile                         # Process configuration
├── Dockerfile                       # Docker image configuration
├── .dockerignore                    # Docker build exclusions
├── .env.example                     # Environment variables template
├── .gitignore                       # Git ignore patterns
├── README.md                        # This file
├── LICENSE                          # Project license
└── .github/
    └── workflows/
        └── ci.yml                   # GitHub Actions CI/CD pipeline
```

## Prerequisites

- **Python 3.13+** installed
- **pip** or **poetry** for package management
- **Git** for version control
- **Render account** (for production deployment)
- **(Optional) Docker** for containerized development

## Local Setup

### 1. Clone the Repository

```bash
git clone https://github.com/susanacharya12/task-manager-api.git
cd task-manager-api
```

### 2. Create Virtual Environment

```bash
# Using venv
python -m venv venv

# Activate virtual environment
source venv/bin/activate  # macOS/Linux
# or
venv\Scripts\activate  # Windows
```

### 3. Install Dependencies

```bash
pip install -r requirements.txt
```

### 4. Set Up Environment Variables

```bash
# Copy example environment file
cp .env.example .env

# Edit .env with your configuration
# For local development, defaults are usually fine
```

### 5. Run Database Migrations

```bash
python manage.py migrate
```

### 6. Create Superuser (Optional)

```bash
python manage.py createsuperuser
```

Follow the prompts to create an admin account for the Django admin panel.

### 7. Collect Static Files (Optional for local development)

```bash
python manage.py collectstatic --noinput
```

### 8. Run Development Server

```bash
python manage.py runserver
```

The API will be available at `http://127.0.0.1:8000/`

## API Documentation

Once the development server is running:

- **Swagger UI** (Interactive): [http://127.0.0.1:8000/swagger/](http://127.0.0.1:8000/swagger/)
- **ReDoc** (Clean Reference): [http://127.0.0.1:8000/redoc/](http://127.0.0.1:8000/redoc/)
- **OpenAPI Schema**: [http://127.0.0.1:8000/api/schema.json](http://127.0.0.1:8000/api/schema.json)

## API Endpoints

### Authentication Endpoints

| Method | Endpoint | Description | Auth Required |
|--------|----------|-------------|---|
| POST | `/api/token/` | Obtain JWT token pair (access + refresh) | ❌ |
| POST | `/api/token/refresh/` | Refresh expired access token | ✅ |

### Task Endpoints

| Method | Endpoint | Description | Auth Required |
|--------|----------|-------------|---|
| GET | `/api/tasks/` | List all user's tasks with filters | ✅ |
| POST | `/api/tasks/` | Create a new task | ✅ |
| GET | `/api/tasks/{id}/` | Retrieve a specific task | ✅ |
| PUT | `/api/tasks/{id}/` | Update a task completely | ✅ |
| PATCH | `/api/tasks/{id}/` | Partially update a task | ✅ |
| DELETE | `/api/tasks/{id}/` | Delete a task | ✅ |
| POST | `/api/tasks/{id}/complete/` | Mark task as completed | ✅ |

### Admin Endpoints

| Endpoint | Description |
|----------|-------------|
| `/admin/` | Django admin panel |

## Filtering & Searching

### Query Parameters

- **`status`**: Filter by status (Pending, In Progress, Completed)
- **`priority`**: Filter by priority (Low, Medium, High)
- **`due_date`**: Filter by specific due date
- **`search`**: Search in title and description fields
- **`ordering`**: Sort results (prefix with `-` for descending)
- **`page`**: Pagination (default: 10 items per page)

### Filter Examples

```bash
# Filter by status
GET /api/tasks/?status=Pending

# Filter by priority
GET /api/tasks/?priority=High

# Combine multiple filters
GET /api/tasks/?status=Completed&priority=High

# Filter by due date
GET /api/tasks/?due_date=2026-05-20
```

### Search Examples

```bash
# Search tasks
GET /api/tasks/?search=project

# Search with filters
GET /api/tasks/?search=urgent&priority=High&status=Pending
```

### Ordering Examples

```bash
# Order by creation date (ascending)
GET /api/tasks/?ordering=created_at

# Order by due date (descending - most recent first)
GET /api/tasks/?ordering=-due_date

# Order by priority
GET /api/tasks/?ordering=priority
```

## Authentication

This API uses JWT (JSON Web Token) authentication. All endpoints except token generation require authentication.

### Getting a Token

```bash
curl -X POST http://localhost:8000/api/token/ \
  -H "Content-Type: application/json" \
  -d '{
    "username": "your_username",
    "password": "your_password"
  }'
```

**Response:**
```json
{
  "access": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "refresh": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
}
```

### Using the Token

Include the access token in the `Authorization` header:

```bash
curl -X GET http://localhost:8000/api/tasks/ \
  -H "Authorization: Bearer <your_access_token>"
```

### Token Refresh

When the access token expires, use the refresh token:

```bash
curl -X POST http://localhost:8000/api/token/refresh/ \
  -H "Content-Type: application/json" \
  -d '{
    "refresh": "<your_refresh_token>"
  }'
```

## Deployment

### Deploy on Render

This project includes a ready-to-use `render.yaml` blueprint for automatic deployment.

#### Prerequisites
- GitHub, GitLab, or Bitbucket repository
- Render account (https://render.com)

#### Deployment Steps

1. **Push code to your repository**
   ```bash
   git add .
   git commit -m "Initial commit"
   git push origin main
   ```

2. **Connect to Render**
   - Open [Render Dashboard](https://dashboard.render.com)
   - Click "New +" → "Blueprint"
   - Select your repository
   - Review the configuration in `render.yaml`

3. **Apply Blueprint**
   - Click "Apply" to create the service
   - Render will automatically:
     - Install dependencies from `requirements.txt`
     - Run migrations: `python manage.py migrate`
     - Collect static files: `python manage.py collectstatic`
     - Start the application with Gunicorn

4. **Verify Deployment**
   - Check the deployment logs in Render dashboard
   - Visit your service URL (e.g., `https://task-manager-api.onrender.com`)
   - Access API documentation at `/swagger/` or `/redoc/`

#### Environment Variables on Render

The following environment variables are automatically managed:

| Variable | Auto-Generated | Purpose |
|----------|---|---------|
| `SECRET_KEY` | ✅ | Django secret key |
| `DATABASE_URL` | ❌ | Connection string (set if using PostgreSQL) |
| `DEBUG` | - | Set to `False` in production |
| `ALLOWED_HOSTS` | - | `.onrender.com,127.0.0.1,localhost` |
| `CSRF_TRUSTED_ORIGINS` | - | CSRF-safe origins |
| `PYTHON_VERSION` | - | `3.13.3` |

#### Production Database

By default, SQLite is used. For production, configure PostgreSQL:

1. Add PostgreSQL addon in Render
2. Render automatically sets `DATABASE_URL`
3. Migrations run automatically during deployment

### Docker Deployment (Alternative)

Build and run locally with Docker:

```bash
# Build Docker image
docker build -t task-manager-api .

# Run container
docker run -p 8000:8000 \
  -e DEBUG=False \
  -e SECRET_KEY=your-secret-key \
  task-manager-api
```

### Deploy to Other Platforms

#### Heroku
```bash
# Add Procfile (included in repo)
git push heroku main
```

#### PythonAnywhere
```bash
# Upload files and configure web app
# Point to wsgi.py in config folder
```

## Environment Variables

Create a `.env` file based on [`.env.example`](.env.example):

```env
# Security
SECRET_KEY=your-secret-key-here
DEBUG=False
ALLOWED_HOSTS=localhost,127.0.0.1,.onrender.com

# Database (leave empty for SQLite in development)
DATABASE_URL=

# CSRF Protection
CSRF_TRUSTED_ORIGINS=http://localhost:8000,https://*.onrender.com

# API Settings
API_PAGE_SIZE=10
```

**Security Note**: Never commit `.env` file with real credentials to version control. The `.gitignore` file already excludes `.env`.

## Database

### Local Development
- **Database**: SQLite3
- **File**: `db.sqlite3`
- **No setup needed**: Migrations run with `python manage.py migrate`

### Production (Render)
- **Database**: PostgreSQL (recommended)
- **Connection**: Via `DATABASE_URL` environment variable
- **Migrations**: Run automatically during deployment

### Backup Strategy
```bash
# Backup SQLite database locally
cp db.sqlite3 db.sqlite3.backup

# For PostgreSQL, use pg_dump
pg_dump $DATABASE_URL > backup.sql
```

## Troubleshooting

### "ModuleNotFoundError: No module named 'django'"
```bash
# Ensure virtual environment is activated
source venv/bin/activate
pip install -r requirements.txt
```

### "Port 8000 already in use"
```bash
# Use different port
python manage.py runserver 8001

# Or kill existing process on port 8000
lsof -ti:8000 | xargs kill -9  # macOS/Linux
```

### "Permission denied" on migrations
```bash
# Ensure manage.py is executable
chmod +x manage.py

# Run migrations with python explicitly
python manage.py migrate
```

### Database locked error
```bash
# Remove stale database lock (SQLite only)
rm db.sqlite3-wal db.sqlite3-shm 2>/dev/null
python manage.py migrate
```

### Static files not loading
```bash
# Collect static files
python manage.py collectstatic --clear --noinput

# Enable WhiteNoise in settings.py (already configured)
```

### CSRF token issues
```bash
# Verify CSRF_TRUSTED_ORIGINS in settings.py
# Ensure Origin header matches trusted origin
```

## Testing

Run the test suite:

```bash
# Run all tests
python manage.py test

# Run specific app tests
python manage.py test task

# Run with verbose output
python manage.py test -v 2

# Run with coverage
pip install coverage
coverage run --source='.' manage.py test
coverage report
```

## Contributing

Contributions are welcome! Please follow these guidelines:

1. Fork the repository
2. Create a feature branch: `git checkout -b feature/your-feature`
3. Make your changes
4. Run tests: `python manage.py test`
5. Commit with descriptive message: `git commit -am 'Add feature'`
6. Push to branch: `git push origin feature/your-feature`
7. Submit a Pull Request

See [CONTRIBUTING.md](CONTRIBUTING.md) for more details.

## Demo Credentials

For testing the deployed instance:

- **Username**: susanacharya
- **Password**: 123

⚠️ **Note**: Change these credentials or create new ones in production!

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## Support

For issues, questions, or suggestions:

- Open an issue on GitHub
- Check existing issues for similar problems
- Include error messages and steps to reproduce
- Specify your Python version and environment

## Changelog

### Version 1.0.0 (2026-05)
- Initial production-ready release
- JWT authentication
- Complete CRUD operations for tasks
- Advanced filtering and search
- API documentation with Swagger/ReDoc
- Render deployment blueprint
- Docker support

---

**Last Updated**: May 2026
**Maintainer**: Susana Charya
**Repository**: [GitHub](https://github.com/susanacharya12/task-manager-api)


### 4. After deploy
- API root: `https://<your-service>.onrender.com/`
- Swagger: `https://<your-service>.onrender.com/swagger/`
- ReDoc: `https://<your-service>.onrender.com/redoc/`

## Environment Variables

Create a `.env` file in the project root for local development:
```env
SECRET_KEY=your-secret-key
DEBUG=True
ALLOWED_HOSTS=localhost,127.0.0.1
CSRF_TRUSTED_ORIGINS=http://localhost:8000
```

## License

MIT License

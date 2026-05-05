# Task Manager API

A Django REST API project for managing tasks with JWT authentication, filtering, searching, and pagination.

## Features

- **User Authentication**: JWT-based authentication using `djangorestframework_simplejwt`
- **Task Management**: Create, Read, Update, Delete (CRUD) operations for tasks
- **Task Properties**:
  - Title and description
  - Status: Pending, In Progress, Completed
  - Priority: Low, Medium, High
  - Due date
- **Filtering**: Filter tasks by status, priority, and due date
- **Search**: Search tasks by title and description
- **Ordering**: Order tasks by created_at and due_date
- **Pagination**: Paginated results (10 items per page)
- **API Documentation**: Interactive API docs via Swagger UI and ReDoc

## Tech Stack

- **Backend**: Django 6.0.3
- **API Framework**: Django REST Framework 3.16.1
- **Authentication**: JWT (Simple JWT)
- **Database**: SQLite3
- **Documentation**: drf-yasg (Swagger/ReDoc)
- **Deployment**: Render

## Project Structure

```
advanced_task_manager/
├── config/                  # Django project settings
│   ├── settings.py
│   ├── urls.py
│   └── wsgi.py
├── task/                   # Task app
│   ├── models.py
│   ├── views.py
│   ├── serializers.py
│   ├── urls.py
│   └── admin.py
├── accounts/               # Accounts app (user management)
├── manage.py
└── requirements.txt
```

## API Endpoints

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/api/tasks/` | List all tasks |
| POST | `/api/tasks/` | Create a new task |
| GET | `/api/tasks/{id}/` | Retrieve a task |
| PUT | `/api/tasks/{id}/` | Update a task |
| DELETE | `/api/tasks/{id}/` | Delete a task |
| POST | `/api/tasks/{id}/complete/` | Mark task as completed |
| POST | `/api/token/` | Obtain JWT token |
| POST | `/api/token/refresh/` | Refresh JWT token |

## Filtering & Searching

### Filter Examples
```
/api/tasks/?status=Pending
/api/tasks/?priority=High
/api/tasks/?status=Completed&priority=High
```

### Search Examples
```
/api/tasks/?search=keyword
```

### Ordering Examples
```
/api/tasks/?ordering=created_at
/api/tasks/?ordering=-due_date
```

## Installation & Setup

### 1. Clone the Repository
```bash
git clone https://github.com/susanacharya12/task-manager-api.git
cd Task_Manager_Api/task-manager-api-1
```

### 2. Create Virtual Environment
```bash
python -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate
```

### 3. Install Dependencies
```bash
pip install -r requirements.txt
```

### 4. Run Migrations
```bash
python manage.py migrate
```

### 5. Create Superuser (Optional)
```bash
python manage.py createsuperuser
```

### 6. Run Development Server
```bash
python manage.py runserver
```

The API will be available at `http://127.0.0.1:8000/`

## API Documentation

Once the server is running:

- **Swagger UI**: `http://127.0.0.1:8000/swagger/`
- **ReDoc**: `http://127.0.0.1:8000/redoc/`

## Demo Credentials

- **Username**: susanacharya
- **Password**: 123

## Deploy on Render

This project includes a ready-to-use `render.yaml` blueprint.

### 1. Push code to GitHub/GitLab/Bitbucket
Render needs your repository to create services from `render.yaml`.

### 2. Create Blueprint from repo
Open:

`https://dashboard.render.com/blueprint/new`

Select your repository and apply the blueprint.

### 3. Set environment values if needed
These are configured automatically in `render.yaml`:
- `SECRET_KEY` (generated)
- `DATABASE_URL` (from managed Postgres)
- `DEBUG=False`
- `ALLOWED_HOSTS=.onrender.com,127.0.0.1,localhost`
- `CSRF_TRUSTED_ORIGINS=https://*.onrender.com,http://127.0.0.1:8000,http://localhost:8000`

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

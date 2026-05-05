# API Usage Examples

This document provides real-world examples of using the Task Manager API.

## Table of Contents

- [Authentication](#authentication)
- [Creating Tasks](#creating-tasks)
- [Listing and Filtering](#listing-and-filtering)
- [Updating Tasks](#updating-tasks)
- [Completing Tasks](#completing-tasks)
- [Deleting Tasks](#deleting-tasks)
- [Advanced Examples](#advanced-examples)

## Authentication

### Obtain JWT Token

```bash
curl -X POST https://your-api.com/api/token/ \
  -H "Content-Type: application/json" \
  -d '{
    "username": "your_username",
    "password": "your_password"
  }'
```

Response:
```json
{
  "access": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "refresh": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
}
```

### Refresh Token

```bash
curl -X POST https://your-api.com/api/token/refresh/ \
  -H "Content-Type: application/json" \
  -d '{
    "refresh": "your_refresh_token"
  }'
```

## Creating Tasks

### Simple Task

```bash
curl -X POST https://your-api.com/api/tasks/ \
  -H "Authorization: Bearer YOUR_ACCESS_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "title": "Complete project proposal",
    "description": "Draft and submit the Q2 project proposal",
    "priority": "High",
    "due_date": "2026-06-15"
  }'
```

### Task with Python Requests

```python
import requests
import json

# Authentication
response = requests.post(
    'https://your-api.com/api/token/',
    json={'username': 'user', 'password': 'pass'}
)
token = response.json()['access']

# Create task
headers = {'Authorization': f'Bearer {token}'}
task_data = {
    'title': 'Fix bug in authentication',
    'description': 'Resolve JWT token expiration issue',
    'priority': 'High',
    'status': 'In Progress',
    'due_date': '2026-05-30'
}

response = requests.post(
    'https://your-api.com/api/tasks/',
    json=task_data,
    headers=headers
)

print(response.json())
```

### Task with JavaScript/Fetch

```javascript
const token = 'YOUR_ACCESS_TOKEN';

const taskData = {
  title: 'Design new dashboard',
  description: 'Create mockups for admin dashboard',
  priority: 'Medium',
  due_date: '2026-06-20'
};

fetch('https://your-api.com/api/tasks/', {
  method: 'POST',
  headers: {
    'Authorization': `Bearer ${token}`,
    'Content-Type': 'application/json'
  },
  body: JSON.stringify(taskData)
})
.then(response => response.json())
.then(data => console.log(data));
```

## Listing and Filtering

### Get All Tasks

```bash
curl -H "Authorization: Bearer YOUR_ACCESS_TOKEN" \
  https://your-api.com/api/tasks/
```

### Filter by Status

```bash
# Get all pending tasks
curl -H "Authorization: Bearer YOUR_ACCESS_TOKEN" \
  https://your-api.com/api/tasks/?status=Pending

# Get completed tasks
curl -H "Authorization: Bearer YOUR_ACCESS_TOKEN" \
  https://your-api.com/api/tasks/?status=Completed
```

### Filter by Priority

```bash
# Get high priority tasks
curl -H "Authorization: Bearer YOUR_ACCESS_TOKEN" \
  https://your-api.com/api/tasks/?priority=High

# Get tasks that are not low priority
curl -H "Authorization: Bearer YOUR_ACCESS_TOKEN" \
  https://your-api.com/api/tasks/?priority=High&priority=Medium
```

### Search Tasks

```bash
# Search by title/description
curl -H "Authorization: Bearer YOUR_ACCESS_TOKEN" \
  https://your-api.com/api/tasks/?search=meeting

# Combine search with filters
curl -H "Authorization: Bearer YOUR_ACCESS_TOKEN" \
  "https://your-api.com/api/tasks/?search=urgent&priority=High&status=Pending"
```

### Ordering

```bash
# Order by creation date (oldest first)
curl -H "Authorization: Bearer YOUR_ACCESS_TOKEN" \
  https://your-api.com/api/tasks/?ordering=created_at

# Order by due date (newest first)
curl -H "Authorization: Bearer YOUR_ACCESS_TOKEN" \
  https://your-api.com/api/tasks/?ordering=-due_date
```

### Pagination

```bash
# Get page 1 (default 10 items per page)
curl -H "Authorization: Bearer YOUR_ACCESS_TOKEN" \
  https://your-api.com/api/tasks/?page=1

# Get page 2
curl -H "Authorization: Bearer YOUR_ACCESS_TOKEN" \
  https://your-api.com/api/tasks/?page=2
```

### Complex Query

```bash
# Get high priority, pending tasks, search for "api", ordered by due date
curl -H "Authorization: Bearer YOUR_ACCESS_TOKEN" \
  "https://your-api.com/api/tasks/?priority=High&status=Pending&search=api&ordering=-due_date&page=1"
```

## Updating Tasks

### Full Update (PUT)

```bash
curl -X PUT https://your-api.com/api/tasks/123/ \
  -H "Authorization: Bearer YOUR_ACCESS_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "title": "Updated task title",
    "description": "Updated description",
    "priority": "Medium",
    "status": "In Progress",
    "due_date": "2026-06-01"
  }'
```

### Partial Update (PATCH)

```bash
# Update only the title
curl -X PATCH https://your-api.com/api/tasks/123/ \
  -H "Authorization: Bearer YOUR_ACCESS_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "title": "New task title"
  }'

# Update only the status
curl -X PATCH https://your-api.com/api/tasks/123/ \
  -H "Authorization: Bearer YOUR_ACCESS_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "status": "Completed"
  }'
```

### Update with Python

```python
import requests

token = 'YOUR_ACCESS_TOKEN'
task_id = 123

headers = {'Authorization': f'Bearer {token}'}

# Partial update
response = requests.patch(
    f'https://your-api.com/api/tasks/{task_id}/',
    json={'status': 'In Progress'},
    headers=headers
)

print(response.json())
```

## Completing Tasks

### Using Custom Endpoint

```bash
curl -X POST https://your-api.com/api/tasks/123/complete/ \
  -H "Authorization: Bearer YOUR_ACCESS_TOKEN"
```

### Using PATCH

```bash
curl -X PATCH https://your-api.com/api/tasks/123/ \
  -H "Authorization: Bearer YOUR_ACCESS_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "status": "Completed"
  }'
```

## Deleting Tasks

```bash
curl -X DELETE https://your-api.com/api/tasks/123/ \
  -H "Authorization: Bearer YOUR_ACCESS_TOKEN"
```

### Delete Multiple Tasks

```python
import requests

token = 'YOUR_ACCESS_TOKEN'
task_ids = [1, 2, 3, 4, 5]

headers = {'Authorization': f'Bearer {token}'}

for task_id in task_ids:
    response = requests.delete(
        f'https://your-api.com/api/tasks/{task_id}/',
        headers=headers
    )
    if response.status_code == 204:
        print(f'Task {task_id} deleted')
```

## Advanced Examples

### Task Management Workflow

```python
import requests
from datetime import datetime, timedelta

API_URL = 'https://your-api.com/api'

# 1. Authenticate
auth_response = requests.post(
    f'{API_URL}/token/',
    json={'username': 'user', 'password': 'pass'}
)
token = auth_response.json()['access']
headers = {'Authorization': f'Bearer {token}'}

# 2. Create tasks for the week
due_date = (datetime.now() + timedelta(days=7)).date()
tasks_to_create = [
    {'title': 'Team meeting', 'priority': 'High', 'due_date': str(due_date)},
    {'title': 'Code review', 'priority': 'Medium', 'due_date': str(due_date)},
    {'title': 'Documentation', 'priority': 'Low', 'due_date': str(due_date)},
]

for task_data in tasks_to_create:
    response = requests.post(
        f'{API_URL}/tasks/',
        json=task_data,
        headers=headers
    )
    print(f"Created: {response.json()['id']}")

# 3. Get all pending high-priority tasks
response = requests.get(
    f'{API_URL}/tasks/?status=Pending&priority=High',
    headers=headers
)
pending_tasks = response.json()

# 4. Update first task to in-progress
if pending_tasks['results']:
    task_id = pending_tasks['results'][0]['id']
    requests.patch(
        f'{API_URL}/tasks/{task_id}/',
        json={'status': 'In Progress'},
        headers=headers
    )

# 5. Get completed tasks
response = requests.get(
    f'{API_URL}/tasks/?status=Completed&ordering=-created_at',
    headers=headers
)
completed = response.json()['results']
print(f"Completed {len(completed)} tasks")
```

### Error Handling

```python
import requests

token = 'YOUR_ACCESS_TOKEN'
headers = {'Authorization': f'Bearer {token}'}

try:
    # Attempt to get a non-existent task
    response = requests.get(
        'https://your-api.com/api/tasks/99999/',
        headers=headers
    )
    
    if response.status_code == 404:
        print("Task not found")
    elif response.status_code == 401:
        print("Unauthorized - check your token")
    elif response.status_code == 403:
        print("Forbidden - you don't have permission")
    elif response.status_code == 400:
        print("Bad request:", response.json())
    else:
        print("Success:", response.json())
        
except requests.exceptions.RequestException as e:
    print(f"Error: {e}")
```

## Response Examples

### List Response

```json
{
  "count": 25,
  "next": "https://your-api.com/api/tasks/?page=2",
  "previous": null,
  "results": [
    {
      "id": 1,
      "title": "Complete project proposal",
      "description": "Draft and submit the Q2 project proposal",
      "status": "Pending",
      "priority": "High",
      "due_date": "2026-06-15",
      "created_at": "2026-05-20T10:30:00Z",
      "updated_at": "2026-05-20T10:30:00Z"
    }
  ]
}
```

### Error Response

```json
{
  "detail": "Not found."
}
```

---

For more information, visit the [main API documentation](../README.md#api-documentation).

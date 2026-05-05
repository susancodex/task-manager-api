# Testing Guide

Comprehensive testing guide for the Task Manager API.

## Table of Contents

- [Test Structure](#test-structure)
- [Running Tests](#running-tests)
- [Unit Tests](#unit-tests)
- [API Tests](#api-tests)
- [Coverage](#coverage)
- [Testing Best Practices](#testing-best-practices)

## Test Structure

```
task/
├── tests.py           # Main test file
├── fixtures/          # Test data
└── migrations/
```

## Running Tests

### Run All Tests

```bash
python manage.py test
```

### Run Specific App

```bash
python manage.py test task
```

### Run Specific Test Class

```bash
python manage.py test task.tests.TaskModelTest
```

### Run Specific Test Method

```bash
python manage.py test task.tests.TaskModelTest.test_create_task
```

### Run with Verbose Output

```bash
python manage.py test -v 2
```

### Run with Parallel Processing

```bash
python manage.py test --parallel
```

## Unit Tests

### Testing Models

```python
from django.test import TestCase
from django.contrib.auth.models import User
from task.models import Task

class TaskModelTest(TestCase):
    def setUp(self):
        """Create test data"""
        self.user = User.objects.create_user(
            username='testuser',
            password='testpass123'
        )

    def test_create_task(self):
        """Test creating a task"""
        task = Task.objects.create(
            title='Test Task',
            owner=self.user,
            status='Pending',
            priority='High'
        )
        self.assertEqual(task.title, 'Test Task')
        self.assertEqual(task.owner, self.user)

    def test_task_str_representation(self):
        """Test task string representation"""
        task = Task.objects.create(
            title='Test Task',
            owner=self.user
        )
        self.assertEqual(str(task), 'Test Task')
```

### Testing Serializers

```python
from django.test import TestCase
from rest_framework.test import APIRequestFactory
from task.serializers import TaskSerializer

class TaskSerializerTest(TestCase):
    def setUp(self):
        self.factory = APIRequestFactory()
        self.user = User.objects.create_user(username='test')

    def test_serializer_validation(self):
        """Test serializer validates data"""
        data = {'title': ''}  # Invalid: title is required
        serializer = TaskSerializer(data=data)
        self.assertFalse(serializer.is_valid())
        self.assertIn('title', serializer.errors)

    def test_serializer_creates_task(self):
        """Test serializer creates task"""
        data = {
            'title': 'New Task',
            'status': 'Pending',
            'priority': 'High'
        }
        serializer = TaskSerializer(
            data=data,
            context={'request': self.factory.get('/')}
        )
        self.assertTrue(serializer.is_valid())
```

## API Tests

### Testing Endpoints

```python
from django.test import TestCase
from rest_framework.test import APIClient
from rest_framework_simplejwt.tokens import RefreshToken
from django.contrib.auth.models import User
from task.models import Task

class TaskAPITest(TestCase):
    def setUp(self):
        """Set up test client and authenticate"""
        self.client = APIClient()
        self.user = User.objects.create_user(
            username='testuser',
            password='testpass123'
        )
        
        # Get JWT tokens
        refresh = RefreshToken.for_user(self.user)
        self.access_token = str(refresh.access_token)
        self.client.credentials(HTTP_AUTHORIZATION=f'Bearer {self.access_token}')

    def test_list_tasks(self):
        """Test listing tasks"""
        response = self.client.get('/api/tasks/')
        self.assertEqual(response.status_code, 200)

    def test_create_task(self):
        """Test creating a task"""
        data = {
            'title': 'New Task',
            'description': 'Test description',
            'priority': 'High',
            'status': 'Pending'
        }
        response = self.client.post('/api/tasks/', data, format='json')
        self.assertEqual(response.status_code, 201)
        self.assertEqual(response.data['title'], 'New Task')

    def test_update_task(self):
        """Test updating a task"""
        task = Task.objects.create(
            title='Original',
            owner=self.user
        )
        data = {'title': 'Updated'}
        response = self.client.patch(f'/api/tasks/{task.id}/', data, format='json')
        self.assertEqual(response.status_code, 200)
        self.assertEqual(response.data['title'], 'Updated')

    def test_delete_task(self):
        """Test deleting a task"""
        task = Task.objects.create(
            title='Delete Me',
            owner=self.user
        )
        response = self.client.delete(f'/api/tasks/{task.id}/')
        self.assertEqual(response.status_code, 204)

    def test_authentication_required(self):
        """Test that authentication is required"""
        self.client.credentials()  # Remove auth
        response = self.client.get('/api/tasks/')
        self.assertEqual(response.status_code, 401)

    def test_filtering(self):
        """Test filtering tasks"""
        Task.objects.create(title='High', priority='High', owner=self.user)
        Task.objects.create(title='Low', priority='Low', owner=self.user)
        
        response = self.client.get('/api/tasks/?priority=High')
        self.assertEqual(len(response.data['results']), 1)

    def test_search(self):
        """Test searching tasks"""
        Task.objects.create(title='Meeting', owner=self.user)
        Task.objects.create(title='Code Review', owner=self.user)
        
        response = self.client.get('/api/tasks/?search=Meeting')
        self.assertEqual(len(response.data['results']), 1)
```

## Coverage

### Generate Coverage Report

```bash
pip install coverage
coverage run --source='.' manage.py test
coverage report
```

### Generate HTML Coverage Report

```bash
coverage html
open htmlcov/index.html
```

### Coverage Configuration

In `pyproject.toml`:
```toml
[tool.coverage.run]
source = ["."]
omit = [
    "*/migrations/*",
    "*/tests/*",
]

[tool.coverage.report]
exclude_lines = [
    "pragma: no cover",
    "def __repr__",
]
```

## Testing Best Practices

### 1. Use setUp and tearDown

```python
def setUp(self):
    """Called before each test"""
    self.user = User.objects.create_user(username='test')

def tearDown(self):
    """Called after each test"""
    # Cleanup happens automatically
```

### 2. Use Descriptive Test Names

```python
# Good
def test_authenticated_user_can_list_own_tasks(self):
    pass

# Bad
def test_list(self):
    pass
```

### 3. Test Both Success and Failure

```python
def test_create_valid_task(self):
    """Success case"""
    response = self.client.post('/api/tasks/', valid_data)
    self.assertEqual(response.status_code, 201)

def test_create_invalid_task(self):
    """Failure case"""
    response = self.client.post('/api/tasks/', invalid_data)
    self.assertEqual(response.status_code, 400)
```

### 4. Use Factory Boy for Test Data

```bash
pip install factory-boy
```

```python
import factory
from task.models import Task
from django.contrib.auth.models import User

class UserFactory(factory.django.DjangoModelFactory):
    class Meta:
        model = User
    username = factory.Sequence(lambda n: f'user{n}')

class TaskFactory(factory.django.DjangoModelFactory):
    class Meta:
        model = Task
    title = factory.Faker('sentence')
    owner = factory.SubFactory(UserFactory)
```

### 5. Isolate Tests

```python
# Each test should be independent
class TaskTest(TestCase):
    def setUp(self):
        # Fresh data for each test
        self.user = User.objects.create_user(username='test')

    def test_one(self):
        # Independent
        pass

    def test_two(self):
        # Also independent
        pass
```

### 6. Use Assertions Effectively

```python
# Good assertions
self.assertEqual(response.status_code, 200)
self.assertIn('title', response.data)
self.assertTrue(task.is_completed)
self.assertIsNone(task.deleted_at)

# Avoid
self.assertTrue(response.status_code == 200)
```

### 7. Test Edge Cases

```python
def test_task_with_empty_description(self):
    """Edge case: empty description"""
    task = Task.objects.create(
        title='Task',
        description='',
        owner=self.user
    )
    self.assertEqual(task.description, '')

def test_task_with_max_length_title(self):
    """Edge case: maximum length"""
    long_title = 'a' * 200
    task = Task.objects.create(
        title=long_title,
        owner=self.user
    )
    self.assertEqual(len(task.title), 200)
```

## Continuous Integration Testing

Tests run automatically on:
- Pull requests
- Commits to main/develop
- Schedule (daily)

See `.github/workflows/ci.yml` for configuration.

---

**Last Updated**: May 2026

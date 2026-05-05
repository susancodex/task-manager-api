# Contributing to Task Manager API

Thank you for your interest in contributing! This document provides guidelines and instructions for contributing to the Task Manager API project.

## Code of Conduct

- Be respectful and professional in all interactions
- Welcome diverse perspectives and experiences
- Focus on constructive feedback
- Report inappropriate behavior to the maintainers

## How to Contribute

### Reporting Bugs

Before reporting a bug:
1. Check if the issue already exists
2. Try to reproduce the issue with the latest code
3. Gather as much information as possible

When reporting a bug, include:
- Clear title and description
- Steps to reproduce
- Expected behavior
- Actual behavior
- Python version and OS
- Relevant error messages and logs

### Suggesting Features

To suggest a feature:
1. Check if the feature has already been suggested
2. Clearly describe what the feature does
3. Explain why it would be useful
4. Provide examples if possible

### Making Changes

1. **Fork the repository**
   ```bash
   git clone https://github.com/your-username/task-manager-api.git
   cd task-manager-api
   ```

2. **Create a feature branch**
   ```bash
   git checkout -b feature/your-feature-name
   # or
   git checkout -b bugfix/issue-description
   ```

3. **Set up development environment**
   ```bash
   python -m venv venv
   source venv/bin/activate  # macOS/Linux
   pip install -r requirements.txt
   ```

4. **Make your changes**
   - Write clear, commented code
   - Follow PEP 8 style guide
   - Keep commits focused and logical
   - Update documentation as needed

5. **Write tests**
   ```bash
   # Run existing tests
   python manage.py test
   
   # Run specific app tests
   python manage.py test task
   ```

6. **Update documentation**
   - Update README.md if needed
   - Add docstrings to functions/classes
   - Update API documentation if endpoints change

7. **Commit your changes**
   ```bash
   git add .
   git commit -m "Brief description of changes"
   ```

8. **Push to your fork**
   ```bash
   git push origin feature/your-feature-name
   ```

9. **Submit a Pull Request**
   - Write a clear title and description
   - Reference any related issues
   - Explain what changes were made and why
   - Ensure all tests pass

## Pull Request Process

1. **Before submitting:**
   - Ensure your code follows PEP 8
   - Run tests: `python manage.py test`
   - Check for any linting issues
   - Update documentation

2. **PR Description should include:**
   - Type of change (Feature, Bug Fix, Documentation, etc.)
   - Description of changes
   - Related issue numbers
   - Testing performed
   - Screenshots (if UI-related)

3. **Respond to feedback:**
   - Be open to suggestions
   - Make requested changes promptly
   - Ask for clarification if needed

## Development Guidelines

### Code Style
- Follow [PEP 8](https://www.python.org/dev/peps/pep-0008/)
- Use meaningful variable and function names
- Write docstrings for all modules, functions, and classes
- Keep functions focused and DRY (Don't Repeat Yourself)

### Django Best Practices
- Use Django ORM instead of raw SQL when possible
- Follow Django coding style
- Keep business logic in models or services, not views
- Use viewsets and serializers properly
- Write database migrations for model changes

### Git Best Practices
- Make small, focused commits
- Write clear commit messages
- One feature per branch
- Rebase before merging (when appropriate)

### Testing
- Write tests for new features
- Ensure existing tests pass
- Aim for good test coverage
- Test both success and error cases

Example test:
```python
from django.test import TestCase
from django.contrib.auth.models import User
from task.models import Task

class TaskTestCase(TestCase):
    def setUp(self):
        self.user = User.objects.create_user(
            username='testuser',
            password='testpass'
        )

    def test_task_creation(self):
        task = Task.objects.create(
            title='Test Task',
            owner=self.user
        )
        self.assertEqual(task.title, 'Test Task')
```

## Setting Up Pre-commit Hooks (Optional)

To catch issues before committing:

```bash
pip install pre-commit
pre-commit install
```

Create `.pre-commit-config.yaml`:
```yaml
repos:
  - repo: https://github.com/psf/black
    rev: 23.0.0
    hooks:
      - id: black
  - repo: https://github.com/PyCQA/flake8
    rev: 6.0.0
    hooks:
      - id: flake8
```

## Documentation

- Update README.md for user-facing changes
- Add docstrings to new functions and classes
- Update API documentation in docstrings
- Keep comments current and meaningful

## Common Issues

### Issue: "Port 8000 already in use"
```bash
python manage.py runserver 8001
```

### Issue: Database errors
```bash
python manage.py migrate
python manage.py flush --noinput  # Clear database
```

### Issue: Static files not loading
```bash
python manage.py collectstatic --noinput --clear
```

## Questions?

- Check existing issues and discussions
- Ask in pull request comments
- Contact maintainers via GitHub

## License

By contributing, you agree that your contributions will be licensed under the MIT License.

---

**Thank you for contributing!** Your efforts help make this project better for everyone. 🎉

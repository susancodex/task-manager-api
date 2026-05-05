# Deployment Guide for Task Manager API

This document provides comprehensive deployment instructions for the Task Manager API.

## Table of Contents

- [Render Deployment](#render-deployment)
- [Docker Deployment](#docker-deployment)
- [Heroku Deployment](#heroku-deployment)
- [PythonAnywhere](#pythonanywhere)
- [Pre-deployment Checklist](#pre-deployment-checklist)
- [Post-deployment](#post-deployment)
- [Troubleshooting](#troubleshooting)

## Render Deployment

### Prerequisites

- GitHub/GitLab/Bitbucket account with pushed code
- Render account (https://render.com)

### Steps

1. **Push your code to repository**
   ```bash
   git add .
   git commit -m "Production ready"
   git push origin main
   ```

2. **Create new Blueprint on Render**
   - Visit: https://dashboard.render.com/blueprint/new
   - Select your repository
   - Authorize Render access

3. **Configure Blueprint**
   - Review `render.yaml` settings
   - Render uses:
     - `PYTHON_VERSION: 3.13.3`
     - `DEBUG: False`
     - PostgreSQL or SQLite

4. **Set Environment Variables**
   - In Render dashboard, environment variables are automatically configured:
     - `SECRET_KEY` (auto-generated)
     - `DATABASE_URL` (auto-generated for PostgreSQL)
     - `ALLOWED_HOSTS` (pre-configured)
     - `CSRF_TRUSTED_ORIGINS` (pre-configured)

5. **Deploy**
   - Click "Create Blueprint"
   - Render automatically:
     - Installs dependencies
     - Runs migrations
     - Collects static files
     - Starts Gunicorn server

6. **Verify Deployment**
   ```bash
   # Visit your service URL (e.g., https://task-manager-api.onrender.com)
   # Check API documentation at /swagger/ or /redoc/
   ```

### Render.yaml Structure

```yaml
services:
  - type: web
    name: task-manager-api
    runtime: python
    plan: free
    buildCommand: |
      pip install -r requirements.txt
      python manage.py collectstatic --no-input
      python manage.py migrate
    startCommand: gunicorn config.wsgi:application --bind 0.0.0.0:$PORT
    envVars:
      - key: PYTHON_VERSION
        value: 3.13.3
      - key: DEBUG
        value: "False"
      - key: SECRET_KEY
        generateValue: true
      - key: DATABASE_URL
        sync: false
```

### Updating Render Deployment

1. Make changes locally
2. Commit and push to GitHub
3. Render automatically redeploys on push to main
4. Monitor deployment in Render dashboard

## Docker Deployment

### Local Docker Testing

1. **Build Docker image**
   ```bash
   docker build -t task-manager-api:latest .
   ```

2. **Run container with docker-compose**
   ```bash
   docker-compose up -d
   ```

3. **Access application**
   - API: http://localhost:8000
   - Swagger: http://localhost:8000/swagger/
   - Database: PostgreSQL on port 5432

4. **Create superuser**
   ```bash
   docker-compose exec web python manage.py createsuperuser
   ```

5. **Stop containers**
   ```bash
   docker-compose down
   ```

### Docker Hub Deployment

1. **Build and tag image**
   ```bash
   docker build -t yourusername/task-manager-api:latest .
   docker tag task-manager-api:latest yourusername/task-manager-api:1.0.0
   ```

2. **Login to Docker Hub**
   ```bash
   docker login
   ```

3. **Push image**
   ```bash
   docker push yourusername/task-manager-api:latest
   docker push yourusername/task-manager-api:1.0.0
   ```

4. **Deploy from Docker Hub**
   - Use image: `yourusername/task-manager-api:latest`
   - Configure environment variables
   - Set port to 8000

## Heroku Deployment

### Prerequisites

- Heroku CLI installed
- Heroku account

### Steps

1. **Initialize Heroku app**
   ```bash
   heroku login
   heroku create task-manager-api
   ```

2. **Add PostgreSQL addon**
   ```bash
   heroku addons:create heroku-postgresql:hobby-dev
   ```

3. **Set environment variables**
   ```bash
   heroku config:set DEBUG=False
   heroku config:set SECRET_KEY=$(python -c "from django.core.management.utils import get_random_secret_key; print(get_random_secret_key())")
   ```

4. **Deploy**
   ```bash
   git push heroku main
   ```

5. **Run migrations**
   ```bash
   heroku run python manage.py migrate
   ```

6. **Create superuser**
   ```bash
   heroku run python manage.py createsuperuser
   ```

## PythonAnywhere

### Prerequisites

- PythonAnywhere account (https://www.pythonanywhere.com)

### Steps

1. **Upload code**
   - Use Web interface or Git
   - Clone repository in bash console

2. **Create virtual environment**
   ```bash
   mkvirtualenv --python=/usr/bin/python3.11 task-manager-api
   pip install -r requirements.txt
   ```

3. **Configure web app**
   - Add new web app
   - Select Python Web framework
   - Point WSGI file to: `/home/username/task-manager-api/config/wsgi.py`

4. **Set environment variables**
   - Edit `~/.bashrc` or in web app settings
   - Set `DEBUG=False`, `SECRET_KEY`, etc.

5. **Configure static files**
   - URL: `/static/`
   - Directory: `/home/username/task-manager-api/staticfiles/`

6. **Reload web app**
   - Click "Reload" in PythonAnywhere dashboard

## Pre-deployment Checklist

Before deploying to production:

- [ ] Set `DEBUG=False`
- [ ] Generate new `SECRET_KEY`
- [ ] Update `ALLOWED_HOSTS`
- [ ] Configure `CSRF_TRUSTED_ORIGINS`
- [ ] Set up PostgreSQL database
- [ ] Test locally with production settings
- [ ] Run `python manage.py check --deploy`
- [ ] Collect static files: `python manage.py collectstatic`
- [ ] Run migrations: `python manage.py migrate`
- [ ] Create superuser for admin
- [ ] Review security settings
- [ ] Enable HTTPS
- [ ] Set up backups
- [ ] Configure logging
- [ ] Set up error tracking (Sentry, etc.)

## Post-deployment

### Verification

1. **Check health endpoint**
   ```bash
   curl https://your-domain.com/swagger/
   ```

2. **Verify API works**
   ```bash
   # Get token
   curl -X POST https://your-domain.com/api/token/ \
     -H "Content-Type: application/json" \
     -d '{"username":"admin","password":"password"}'
   
   # List tasks
   curl https://your-domain.com/api/tasks/ \
     -H "Authorization: Bearer <token>"
   ```

3. **Check logs**
   - Review application logs for errors
   - Monitor for performance issues

### Maintenance

1. **Regular backups**
   - Daily database backups
   - Store in secure location
   - Test restore procedures

2. **Monitor performance**
   - Set up monitoring (New Relic, DataDog)
   - Monitor error rates
   - Track response times

3. **Update dependencies**
   ```bash
   pip list --outdated
   pip install --upgrade package-name
   ```

4. **Security updates**
   - Monitor for Django security releases
   - Update dependencies regularly
   - Run security checks

## Troubleshooting

### Deployment Fails

1. **Check build logs**
   - Review error messages
   - Look for missing dependencies
   - Verify environment variables

2. **Common issues**
   - Missing `requirements.txt`
   - Syntax errors in `settings.py`
   - Database migration failures
   - Static file collection errors

### Application Errors

1. **500 Internal Server Error**
   - Check server logs
   - Verify database connection
   - Check environment variables
   - Enable DEBUG temporarily (carefully)

2. **Static files not loading**
   - Run `python manage.py collectstatic`
   - Check `STATIC_ROOT` and `STATIC_URL`
   - Verify web server configuration

3. **Database connection failed**
   - Verify `DATABASE_URL`
   - Check database is running
   - Test connection string
   - Check firewall/security groups

### Performance Issues

1. **Slow queries**
   - Enable query logging
   - Use Django Debug Toolbar (dev only)
   - Add database indexes
   - Optimize N+1 queries

2. **Memory issues**
   - Reduce Gunicorn workers
   - Profile memory usage
   - Check for memory leaks
   - Monitor logs

## Getting Help

- Check application logs
- Review Django error pages (carefully)
- Search error messages online
- Contact platform support (Render, Heroku, etc.)
- File GitHub issues

---

Last Updated: May 2026

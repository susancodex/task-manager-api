# Security & Best Practices Guide

This document outlines the security measures and best practices implemented in this project and recommendations for deployment.

## Table of Contents

- [Security Features](#security-features)
- [Environment Variables](#environment-variables)
- [Database Security](#database-security)
- [API Security](#api-security)
- [Deployment Security](#deployment-security)
- [Common Vulnerabilities](#common-vulnerabilities)
- [Security Checklist](#security-checklist)
- [Incident Response](#incident-response)

## Security Features

### Implemented

1. **JWT Authentication**
   - Tokens expire automatically
   - Refresh tokens for extended sessions
   - Tokens cannot be used without authentication
   - Simple JWT with industry-standard algorithms

2. **CSRF Protection**
   - Django CSRF middleware enabled
   - CSRF_TRUSTED_ORIGINS configured
   - Safe for production use

3. **Secret Key Management**
   - Automatically generated on deployment
   - Never committed to repository
   - Environment variable based

4. **HTTPS/TLS**
   - Secure proxy headers configured
   - Forces HTTPS in production
   - Security headers properly set

5. **SQL Injection Prevention**
   - Django ORM parameterized queries
   - No raw SQL queries used
   - Database abstraction layer

6. **XSS Protection**
   - Django template autoescaping
   - DRF serializers sanitize data
   - Content-Type headers set correctly

7. **Static Files**
   - WhiteNoise for secure serving
   - Manifest files for cache busting
   - Compressed assets

## Environment Variables

### Critical Variables

Never commit these to version control:

```env
# Secret key - 50+ random characters
SECRET_KEY=

# Database password
DATABASE_PASSWORD=

# API keys/tokens
THIRD_PARTY_API_KEYS=
```

### Managing Secrets

1. **Use `.env.example`** - Document required variables without secrets
2. **Never commit `.env`** - Already in `.gitignore`
3. **Use platform secrets** - Render, Heroku, GitHub Secrets
4. **Rotate regularly** - Especially after team changes
5. **Audit access** - Track who can access secrets

### Required Variables Production

```env
DEBUG=False
SECRET_KEY=<strong-random-key>
ALLOWED_HOSTS=yourdomain.com
CSRF_TRUSTED_ORIGINS=https://yourdomain.com
DATABASE_URL=postgresql://user:pass@host:5432/db
```

## Database Security

### SQLite (Development Only)

```python
# Automatically uses SQLite in development
DATABASE_URL = f"sqlite:///{BASE_DIR / 'db.sqlite3'}"
```

### PostgreSQL (Production)

```python
# Production configuration
DATABASES = {
    "default": dj_database_url.config(
        conn_max_age=600,
        default=os.getenv("DATABASE_URL")
    )
}
```

### Best Practices

1. **Use strong passwords**
   - Minimum 20 characters
   - Mix of upper, lower, numbers, symbols
   - Unique per environment

2. **Limit database user permissions**
   ```sql
   -- Only grant necessary permissions
   GRANT CONNECT ON DATABASE task_manager TO app_user;
   GRANT USAGE ON SCHEMA public TO app_user;
   GRANT SELECT, INSERT, UPDATE, DELETE ON ALL TABLES IN SCHEMA public TO app_user;
   ```

3. **Enable SSL connections**
   ```env
   DATABASE_URL=postgresql://user:pass@host:5432/db?sslmode=require
   ```

4. **Regular backups**
   ```bash
   # Automated daily backups
   pg_dump $DATABASE_URL > backups/db_$(date +%Y%m%d).sql
   ```

5. **Database monitoring**
   - Monitor for slow queries
   - Set up alerts for failures
   - Review access logs

## API Security

### Authentication

1. **JWT Token Security**
   - Access tokens expire after 5 minutes (configurable)
   - Refresh tokens for obtaining new access tokens
   - Revoke tokens on logout

2. **Password Security**
   - Bcrypt hashing with salt
   - Django password validators
   - No passwords in logs

3. **Rate Limiting** (Consider adding)
   ```python
   # In requirements.txt
   django-ratelimit==4.1.0
   ```

### Authorization

```python
# Only authenticated users can access
DEFAULT_PERMISSION_CLASSES = (
    'rest_framework.permissions.IsAuthenticated',
)
```

### Input Validation

```python
# Serializers validate all input
class TaskSerializer(serializers.ModelSerializer):
    title = serializers.CharField(max_length=200, required=True)
    description = serializers.CharField(max_length=5000, required=False)
    # ...validation rules
```

### CORS (if needed)

```python
# Don't enable CORS unless necessary
# Only allow trusted origins
CORS_ALLOWED_ORIGINS = [
    "https://yourdomain.com",
    "https://subdomain.yourdomain.com",
]
```

## Deployment Security

### Render Deployment

1. **Secrets Configuration**
   - Use Render environment variables
   - Never expose in `render.yaml`
   - Rotate regularly

2. **Database**
   - Use managed PostgreSQL
   - Enable SSL connections
   - Automated backups

3. **Networking**
   - Use HTTPS only
   - Render provides free SSL
   - Enable security headers

### Docker Deployment

1. **Image Security**
   ```dockerfile
   # Use specific Python version (don't use 'latest')
   FROM python:3.13.3-slim
   
   # Create non-root user
   RUN useradd -m appuser
   USER appuser
   ```

2. **Container Configuration**
   ```bash
   # Don't run as root
   # Use read-only filesystem where possible
   # Minimize layer count
   ```

### General Practices

1. **HTTPS Only**
   - Redirect HTTP to HTTPS
   - Use HSTS headers
   - Valid SSL certificates

2. **Security Headers**
   ```python
   SECURE_SSL_REDIRECT = True
   SESSION_COOKIE_SECURE = True
   CSRF_COOKIE_SECURE = True
   SECURE_BROWSER_XSS_FILTER = True
   SECURE_CONTENT_SECURITY_POLICY = {
       'default-src': ("'self'",),
   }
   ```

3. **Logging**
   - Log security events
   - Monitor for suspicious activity
   - Never log passwords
   - Store logs securely

## Common Vulnerabilities

### 1. SQL Injection
**Status**: ✅ Protected
- Uses Django ORM exclusively
- All queries parameterized
- Input validated

### 2. Cross-Site Scripting (XSS)
**Status**: ✅ Protected
- Template auto-escaping enabled
- DRF validates data types
- Content-Type headers set

### 3. Cross-Site Request Forgery (CSRF)
**Status**: ✅ Protected
- CSRF middleware enabled
- CSRF_TRUSTED_ORIGINS configured
- Tokens required for state-changing requests

### 4. Broken Authentication
**Status**: ✅ Protected
- JWT with expiration
- Secure password hashing
- No hardcoded credentials

### 5. Sensitive Data Exposure
**Status**: ⚠️ Needs attention
- Implement HTTPS
- Don't log sensitive data
- Use environment variables
- Encrypted database connections

### 6. Broken Access Control
**Status**: ✅ Protected
- Authentication required
- IsAuthenticated permission class
- Users can only access their own tasks

## Security Checklist

Before deploying to production:

### Code Security
- [ ] No hardcoded passwords
- [ ] No API keys in code
- [ ] No debug info in production
- [ ] Run `python manage.py check --deploy`
- [ ] Review sensitive data handling
- [ ] No SQL injection vulnerabilities

### Configuration
- [ ] DEBUG=False
- [ ] Unique SECRET_KEY
- [ ] ALLOWED_HOSTS configured
- [ ] CSRF_TRUSTED_ORIGINS configured
- [ ] Database password changed
- [ ] No default credentials

### Deployment
- [ ] HTTPS configured
- [ ] Security headers set
- [ ] Firewall rules configured
- [ ] Database backups enabled
- [ ] Logging configured
- [ ] Error tracking enabled

### Ongoing
- [ ] Monitor logs regularly
- [ ] Update dependencies monthly
- [ ] Run security audits
- [ ] Rotate secrets periodically
- [ ] Review access logs
- [ ] Test disaster recovery

## Incident Response

### If Compromised

1. **Immediate Actions**
   ```bash
   # 1. Rotate secret key
   # 2. Force password reset for all users
   # 3. Revoke all tokens
   # 4. Review recent logs
   # 5. Backup database
   ```

2. **Investigation**
   - Check access logs
   - Review failed login attempts
   - Look for unauthorized data access
   - Identify entry point

3. **Remediation**
   - Update vulnerable code
   - Patch all dependencies
   - Change all credentials
   - Reset affected user sessions
   - Enable MFA if available

4. **Communication**
   - Notify affected users
   - Document timeline
   - Post-incident review
   - Update security policies

### Reporting Security Issues

If you find a vulnerability:

1. **Don't** open a public GitHub issue
2. **Do** email security concerns privately
3. **Include**: Detailed description, reproduction steps, impact assessment

## Additional Resources

- [Django Security Documentation](https://docs.djangoproject.com/en/6.0/topics/security/)
- [OWASP Top 10](https://owasp.org/www-project-top-ten/)
- [Django REST Framework Security](https://www.django-rest-framework.org/topics/security/)
- [NIST Cybersecurity Framework](https://www.nist.gov/cyberframework)

---

**Last Updated**: May 2026
**Questions**: Contact project maintainers

# Deploy Guide

## Overview

This document provides instructions for deploying the Webhooks Inventory Management System to production.

---

## Deployment Checklist

### Pre-Deployment

- [ ] All tests passing
- [ ] Security review completed
- [ ] Environment variables documented
- [ ] Database backup strategy defined
- [ ] Rollback plan prepared
- [ ] Monitoring configured
- [ ] SSL certificates obtained

### Deployment

- [ ] Production environment configured
- [ ] Database migrations applied
- [ ] Static files collected
- [ ] Web server configured
- [ ] HTTPS enabled
- [ ] Health checks passing

### Post-Deployment

- [ ] Smoke tests passing
- [ ] Monitoring alerts configured
- [ ] Logs accessible
- [ ] Performance baseline established

---

## Production Requirements

### Server Requirements

| Resource | Minimum | Recommended |
|----------|---------|-------------|
| CPU | 1 core | 2+ cores |
| RAM | 512 MB | 2+ GB |
| Storage | 5 GB | 20+ GB |
| Network | 10 Mbps | 100+ Mbps |

### Software Requirements

- Python 3.10+
- PostgreSQL (recommended) or SQLite
- Nginx or Apache
- Gunicorn or uWSGI
- SSL/TLS certificates

---

## Environment Configuration

### Production Environment Variables

```env
# Django Settings
DEBUG=False
SECRET_KEY=<strong-random-secret-key>
ALLOWED_HOSTS=your-domain.com,www.your-domain.com

# Database (PostgreSQL recommended)
DATABASE_ENGINE=django.db.backends.postgresql
DATABASE_NAME=webhooks_db
DATABASE_USER=webhooks_user
DATABASE_PASSWORD=<strong-password>
DATABASE_HOST=localhost
DATABASE_PORT=5432

# CallMeBot Configuration
CALLMEBOT_API_URL=https://api.callmebot.com/whatsapp.php
CALLMEBOT_PHONE_NUMBER=+1234567890
CALLMEBOT_API_KEY=<your-api-key>

# Email Configuration
EMAIL_HOST=smtp.gmail.com
EMAIL_PORT=587
EMAIL_HOST_USER=notifications@your-domain.com
EMAIL_HOST_PASSWORD=<app-password>
EMAIL_ADMIN_RECEIVER=admin@your-domain.com
EMAIL_USE_TLS=True

# Security
WEBHOOK_API_KEY=<secure-webhook-api-key>
SECURE_SSL_REDIRECT=True
SESSION_COOKIE_SECURE=True
CSRF_COOKIE_SECURE=True
```

---

## Database Setup

### PostgreSQL Setup (Recommended)

```bash
# Install PostgreSQL
# Ubuntu/Debian
sudo apt-get install postgresql postgresql-contrib

# Create database and user
sudo -u postgres psql

CREATE DATABASE webhooks_db;
CREATE USER webhooks_user WITH PASSWORD 'strong-password';
ALTER ROLE webhooks_user SET client_encoding TO 'utf8';
ALTER ROLE webhooks_user SET default_transaction_isolation TO 'read committed';
ALTER ROLE webhooks_user SET timezone TO 'UTC';
GRANT ALL PRIVILEGES ON DATABASE webhooks_db TO webhooks_user;
\q
```

**Django Settings:**
```python
# settings.py
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql',
        'NAME': config('DATABASE_NAME'),
        'USER': config('DATABASE_USER'),
        'PASSWORD': config('DATABASE_PASSWORD'),
        'HOST': config('DATABASE_HOST', default='localhost'),
        'PORT': config('DATABASE_PORT', default='5432'),
    }
}
```

### SQLite (Development/Small Deployments)

```python
# settings.py
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.sqlite3',
        'NAME': BASE_DIR / 'db.sqlite3',
    }
}
```

---

## Web Server Configuration

### Nginx + Gunicorn

#### 1. Install Gunicorn

```bash
pip install gunicorn
```

#### 2. Create Gunicorn Systemd Service

```ini
# /etc/systemd/system/webhooks.service
[Unit]
Description=Webhooks Inventory Management System
After=network.target

[Service]
User=www-data
Group=www-data
WorkingDirectory=/path/to/project
ExecStart=/path/to/venv/bin/gunicorn \
    --access-logfile - \
    --workers 3 \
    --bind unix:/path/to/project/webhooks.sock \
    app.wsgi:application

[Install]
WantedBy=multi-user.target
```

```bash
# Enable and start service
sudo systemctl daemon-reload
sudo systemctl enable webhooks
sudo systemctl start webhooks
```

#### 3. Configure Nginx

```nginx
# /etc/nginx/sites-available/webhooks
server {
    listen 80;
    server_name your-domain.com www.your-domain.com;
    
    # Redirect HTTP to HTTPS
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name your-domain.com www.your-domain.com;
    
    # SSL Configuration
    ssl_certificate /etc/letsencrypt/live/your-domain.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/your-domain.com/privkey.pem;
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers HIGH:!aNULL:!MD5;
    ssl_prefer_server_ciphers on;
    
    # Security Headers
    add_header Strict-Transport-Security "max-age=31536000; includeSubDomains" always;
    add_header X-Content-Type-Options nosniff always;
    add_header X-Frame-Options DENY always;
    add_header X-XSS-Protection "1; mode=block" always;
    
    # Logging
    access_log /var/log/nginx/webhooks_access.log;
    error_log /var/log/nginx/webhooks_error.log;
    
    # Static files
    location /static/ {
        alias /path/to/project/static/;
        expires 30d;
        add_header Cache-Control "public, immutable";
    }
    
    # Media files
    location /media/ {
        alias /path/to/project/media/;
        expires 7d;
    }
    
    # Proxy to Gunicorn
    location / {
        proxy_pass http://unix:/path/to/project/webhooks.sock;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_redirect off;
        
        # Timeouts
        proxy_connect_timeout 60s;
        proxy_send_timeout 60s;
        proxy_read_timeout 60s;
    }
    
    # Rate Limiting
    location /api/ {
        limit_req zone=api burst=20 nodelay;
        proxy_pass http://unix:/path/to/project/webhooks.sock;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
```

#### 4. Configure Rate Limiting

```nginx
# /etc/nginx/nginx.conf (http block)
http {
    # Rate limiting zone
    limit_req_zone $binary_remote_addr zone=api:10m rate=10r/s;
    
    # ... rest of configuration
}
```

#### 5. Enable Site

```bash
sudo ln -s /etc/nginx/sites-available/webhooks /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl restart nginx
```

---

## SSL/TLS Configuration

### Let's Encrypt Setup

```bash
# Install Certbot
sudo apt-get install certbot python3-certbot-nginx

# Obtain certificate
sudo certbot --nginx -d your-domain.com -d www.your-domain.com

# Auto-renewal (already configured by certbot)
sudo certbot renew --dry-run
```

### Manual SSL Certificate

```nginx
ssl_certificate /path/to/certificate.crt;
ssl_certificate_key /path/to/private.key;
ssl_protocols TLSv1.2 TLSv1.3;
ssl_ciphers ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256;
ssl_prefer_server_ciphers on;
ssl_session_cache shared:SSL:10m;
ssl_session_timeout 10m;
```

---

## Django Production Settings

### settings.py

```python
from decouple import config, Csv

# Security
DEBUG = config('DEBUG', default=False, cast=bool)
SECRET_KEY = config('SECRET_KEY')
ALLOWED_HOSTS = config('ALLOWED_HOSTS', cast=Csv())

# HTTPS
SECURE_SSL_REDIRECT = config('SECURE_SSL_REDIRECT', default=True, cast=bool)
SESSION_COOKIE_SECURE = True
CSRF_COOKIE_SECURE = True
SECURE_HSTS_SECONDS = 31536000
SECURE_HSTS_INCLUDE_SUBDOMAINS = True
SECURE_HSTS_PRELOAD = True

# Security Headers
SECURE_BROWSER_XSS_FILTER = True
SECURE_CONTENT_TYPE_NOSNIFF = True
X_FRAME_OPTIONS = 'DENY'

# Database
DATABASES = {
    'default': {
        'ENGINE': config('DATABASE_ENGINE', default='django.db.backends.sqlite3'),
        'NAME': config('DATABASE_NAME', default=BASE_DIR / 'db.sqlite3'),
        'USER': config('DATABASE_USER', default=''),
        'PASSWORD': config('DATABASE_PASSWORD', default=''),
        'HOST': config('DATABASE_HOST', default='localhost'),
        'PORT': config('DATABASE_PORT', default=''),
    }
}

# Logging
LOGGING = {
    'version': 1,
    'disable_existing_loggers': False,
    'formatters': {
        'verbose': {
            'format': '{levelname} {asctime} {module} {message}',
            'style': '{',
        },
    },
    'handlers': {
        'file': {
            'level': 'ERROR',
            'class': 'logging.FileHandler',
            'filename': '/var/log/django/webhooks_error.log',
            'formatter': 'verbose',
        },
    },
    'loggers': {
        'django': {
            'handlers': ['file'],
            'level': 'ERROR',
            'propagate': True,
        },
    },
}

# Static Files
STATIC_ROOT = BASE_DIR / 'static'
STATIC_URL = '/static/'

# Media Files
MEDIA_ROOT = BASE_DIR / 'media'
MEDIA_URL = '/media/'
```

---

## Deployment Steps

### 1. Prepare Server

```bash
# Update system
sudo apt-get update
sudo apt-get upgrade -y

# Install dependencies
sudo apt-get install -y python3 python3-pip python3-venv nginx postgresql

# Create application user
sudo useradd -m -s /bin/bash webhooks
```

### 2. Deploy Code

```bash
# Clone repository
sudo -u webhooks git clone <repository-url> /path/to/project
cd /path/to/project

# Create virtual environment
sudo -u webhooks python3 -m venv venv

# Install dependencies
sudo -u webhooks ./venv/bin/pip install -r requirements.txt
sudo -u webhooks ./venv/bin/pip install gunicorn
```

### 3. Configure Environment

```bash
# Create .env file
sudo -u webhooks nano /path/to/project/.env
# Add production environment variables
```

### 4. Setup Database

```bash
# Run migrations
sudo -u webhooks ./venv/bin/python manage.py migrate

# Create superuser
sudo -u webhooks ./venv/bin/python manage.py createsuperuser

# Collect static files
sudo -u webhooks ./venv/bin/python manage.py collectstatic --noinput
```

### 5. Configure Web Server

Follow the [Web Server Configuration](#web-server-configuration) section above.

### 6. Start Services

```bash
sudo systemctl daemon-reload
sudo systemctl enable webhooks
sudo systemctl start webhooks
sudo systemctl enable nginx
sudo systemctl start nginx
```

### 7. Verify Deployment

```bash
# Check service status
sudo systemctl status webhooks
sudo systemctl status nginx

# Test endpoint
curl -I https://your-domain.com/api/V1/webhooks/order/

# Check logs
sudo tail -f /var/log/nginx/webhooks_access.log
sudo tail -f /var/log/django/webhooks_error.log
```

---

## Monitoring

### Health Check Endpoint

```python
# views.py
from rest_framework.views import APIView
from rest_framework.response import Response
from django.db import connection

class HealthCheckView(APIView):
    def get(self, request):
        try:
            # Check database connection
            connection.ensure_connection()
            db_status = 'healthy'
        except Exception:
            db_status = 'unhealthy'
        
        return Response({
            'status': 'healthy',
            'database': db_status,
            'timestamp': datetime.now().isoformat()
        })
```

```python
# urls.py
path('health/', HealthCheckView.as_view(), name='health-check'),
```

### Monitoring Tools

#### 1. Django Debug Toolbar (Development Only)

```bash
pip install django-debug-toolbar
```

#### 2. Sentry (Error Tracking)

```bash
pip install sentry-sdk
```

```python
# settings.py
import sentry_sdk
from sentry_sdk.integrations.django import DjangoIntegration

sentry_sdk.init(
    dsn=config('SENTRY_DSN'),
    integrations=[DjangoIntegration()],
    traces_sample_rate=0.1,
    send_default_pii=True
)
```

#### 3. Prometheus + Grafana

```bash
pip install django-prometheus
```

```python
# settings.py
INSTALLED_APPS = [
    'django_prometheus',
    # ...
]

MIDDLEWARE = [
    'django_prometheus.middleware.PrometheusBeforeMiddleware',
    # ...
    'django_prometheus.middleware.PrometheusAfterMiddleware',
]
```

---

## Backup Strategy

### Database Backup

```bash
#!/bin/bash
# backup.sh

DATE=$(date +%Y%m%d_%H%M%S)
BACKUP_DIR="/backups/webhooks"

# PostgreSQL backup
pg_dump -U webhooks_user webhooks_db > $BACKUP_DIR/db_$DATE.sql

# Compress backup
gzip $BACKUP_DIR/db_$DATE.sql

# Keep only last 7 days
find $BACKUP_DIR -name "db_*.sql.gz" -mtime +7 -delete
```

**Cron Job:**
```bash
# Run daily at 2 AM
0 2 * * * /path/to/backup.sh
```

### Media Backup

```bash
# Sync media files to backup location
rsync -av /path/to/project/media/ /backups/webhooks/media/
```

---

## Rollback Plan

### Quick Rollback

```bash
# Stop services
sudo systemctl stop webhooks

# Revert code
cd /path/to/project
sudo -u webhooks git checkout <previous-tag>

# Revert database
gunzip /backups/webhooks/db_YYYYMMDD_HHMMSS.sql.gz
psql -U webhooks_user webhooks_db < /backups/webhooks/db_YYYYMMDD_HHMMSS.sql

# Restart services
sudo systemctl start webhooks
```

---

## Performance Optimization

### Gunicorn Tuning

```bash
# Optimal worker count: (2 x CPU cores) + 1
# For 2-core server: 5 workers
gunicorn --workers 5 --bind unix:webhooks.sock app.wsgi:application
```

### Database Optimization

```python
# settings.py
# Connection pooling (PostgreSQL)
DATABASES = {
    'default': {
        # ...
        'CONN_MAX_AGE': 600,  # 10 minutes
        'OPTIONS': {
            'connect_timeout': 10,
        }
    }
}
```

### Caching

```python
# settings.py
CACHES = {
    'default': {
        'BACKEND': 'django.core.cache.backends.redis.RedisCache',
        'LOCATION': 'redis://127.0.0.1:6379/1',
    }
}
```

---

## Troubleshooting

### Common Issues

#### 1. 502 Bad Gateway

```bash
# Check Gunicorn is running
sudo systemctl status webhooks

# Check socket permissions
ls -la /path/to/project/webhooks.sock

# Check logs
sudo tail -f /var/log/nginx/webhooks_error.log
```

#### 2. Database Connection Errors

```bash
# Test database connection
sudo -u webhooks ./venv/bin/python manage.py dbshell

# Check PostgreSQL is running
sudo systemctl status postgresql
```

#### 3. Static Files Not Loading

```bash
# Verify collectstatic
python manage.py collectstatic --noinput

# Check Nginx configuration
sudo nginx -t

# Check file permissions
chmod -R 755 /path/to/project/static/
```

---

## Next Steps

- Review [Monitoring & Maintenance](#monitoring) for ongoing operations
- Check [Security Guide](authentication-security.md) for security hardening
- See [Contributing Guide](contributing.md) for team collaboration

# Django Deployment (Despliegue)

> **Llevar tu aplicación Django a producción**

---

## Diferencias: Development vs Production

| Aspecto | Development | Production |
|---------|-------------|------------|
| **DEBUG** | True | False |
| **SECRET_KEY** | Hardcoded | Variable de entorno |
| **ALLOWED_HOSTS** | [] | ['miapp.com'] |
| **Database** | SQLite | PostgreSQL |
| **Static files** | runserver | WhiteNoise/S3 |
| **Server** | runserver | Gunicorn + Nginx |
| **HTTPS** | No | Sí |
| **Logging** | Console | Archivo/Service |

---

## Settings para producción

### Separar settings

**Estructura:**

```
miproyecto/
├── settings/
│   ├── __init__.py
│   ├── base.py          # Settings comunes
│   ├── development.py   # Dev
│   ├── production.py    # Prod
│   └── testing.py       # Tests
```

**base.py**

```python
"""Settings comunes"""
import os
from pathlib import Path

BASE_DIR = Path(__file__).resolve().parent.parent.parent

INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'rest_framework',
    # Tus apps
    'usuarios',
    'productos',
]

MIDDLEWARE = [
    'django.middleware.security.SecurityMiddleware',
    'django.contrib.sessions.middleware.SessionMiddleware',
    'django.middleware.common.CommonMiddleware',
    'django.middleware.csrf.CsrfViewMiddleware',
    'django.contrib.auth.middleware.AuthenticationMiddleware',
    'django.contrib.messages.middleware.MessageMiddleware',
    'django.middleware.clickjacking.XFrameOptionsMiddleware',
]

ROOT_URLCONF = 'miproyecto.urls'

TEMPLATES = [
    {
        'BACKEND': 'django.template.backends.django.DjangoTemplates',
        'DIRS': [BASE_DIR / 'templates'],
        'APP_DIRS': True,
        'OPTIONS': {
            'context_processors': [
                'django.template.context_processors.debug',
                'django.template.context_processors.request',
                'django.contrib.auth.context_processors.auth',
                'django.contrib.messages.context_processors.messages',
            ],
        },
    },
]

WSGI_APPLICATION = 'miproyecto.wsgi.application'

# Internationalization
LANGUAGE_CODE = 'es'
TIME_ZONE = 'America/Mexico_City'
USE_I18N = True
USE_TZ = True

# Static files
STATIC_URL = '/static/'
STATIC_ROOT = BASE_DIR / 'staticfiles'

# Default primary key
DEFAULT_AUTO_FIELD = 'django.db.models.BigAutoField'
```

**development.py**

```python
"""Settings de desarrollo"""
from .base import *

DEBUG = True

SECRET_KEY = 'django-insecure-dev-key-not-for-production'

ALLOWED_HOSTS = ['localhost', '127.0.0.1']

DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.sqlite3',
        'NAME': BASE_DIR / 'db.sqlite3',
    }
}

# CORS permisivo para desarrollo
CORS_ALLOW_ALL_ORIGINS = True
```

**production.py**

```python
"""Settings de producción"""
from .base import *
import os

DEBUG = False

SECRET_KEY = os.environ.get('SECRET_KEY')

ALLOWED_HOSTS = os.environ.get('ALLOWED_HOSTS', '').split(',')

# Database PostgreSQL
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql',
        'NAME': os.environ.get('DB_NAME'),
        'USER': os.environ.get('DB_USER'),
        'PASSWORD': os.environ.get('DB_PASSWORD'),
        'HOST': os.environ.get('DB_HOST'),
        'PORT': os.environ.get('DB_PORT', '5432'),
    }
}

# Security
SECURE_SSL_REDIRECT = True
SESSION_COOKIE_SECURE = True
CSRF_COOKIE_SECURE = True
SECURE_BROWSER_XSS_FILTER = True
SECURE_CONTENT_TYPE_NOSNIFF = True
X_FRAME_OPTIONS = 'DENY'

# HSTS
SECURE_HSTS_SECONDS = 31536000  # 1 año
SECURE_HSTS_INCLUDE_SUBDOMAINS = True
SECURE_HSTS_PRELOAD = True

# CORS
CORS_ALLOWED_ORIGINS = os.environ.get('CORS_ORIGINS', '').split(',')

# Static files con WhiteNoise
MIDDLEWARE.insert(1, 'whitenoise.middleware.WhiteNoiseMiddleware')
STATICFILES_STORAGE = 'whitenoise.storage.CompressedManifestStaticFilesStorage'

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
            'filename': BASE_DIR / 'logs' / 'django.log',
            'formatter': 'verbose',
        },
    },
    'root': {
        'handlers': ['file'],
        'level': 'ERROR',
    },
}
```

**Usar settings específicos:**

```bash
# Development (default)
python manage.py runserver --settings=miproyecto.settings.development

# Production
python manage.py runserver --settings=miproyecto.settings.production

# O con variable de entorno
export DJANGO_SETTINGS_MODULE=miproyecto.settings.production
python manage.py runserver
```

---

## requirements.txt

```txt
# requirements/base.txt
Django==5.0.1
djangorestframework==3.14.0
psycopg2-binary==2.9.9
python-dotenv==1.0.0
django-cors-headers==4.3.1

# requirements/development.txt
-r base.txt
django-debug-toolbar==4.2.0
ipython==8.18.0

# requirements/production.txt
-r base.txt
gunicorn==21.2.0
whitenoise==6.6.0
sentry-sdk==1.39.0
```

---

## .env para producción

**.env.production**

```env
# Django
SECRET_KEY=tu-secret-key-super-segura-y-larga
DEBUG=False
ALLOWED_HOSTS=miapp.com,www.miapp.com

# Database
DB_NAME=miapp_db
DB_USER=miapp_user
DB_PASSWORD=password-super-segura
DB_HOST=localhost
DB_PORT=5432

# CORS
CORS_ORIGINS=https://miapp.com,https://www.miapp.com

# Email
EMAIL_HOST=smtp.gmail.com
EMAIL_PORT=587
EMAIL_HOST_USER=noreply@miapp.com
EMAIL_HOST_PASSWORD=password-email
EMAIL_USE_TLS=True

# AWS (opcional)
AWS_ACCESS_KEY_ID=tu-access-key
AWS_SECRET_ACCESS_KEY=tu-secret-key
AWS_STORAGE_BUCKET_NAME=miapp-static
```

---

## Gunicorn (WSGI Server)

**Instalar:**

```bash
pip install gunicorn
```

**Ejecutar:**

```bash
# Básico
gunicorn miproyecto.wsgi:application

# Con workers
gunicorn miproyecto.wsgi:application \
  --workers 3 \
  --bind 0.0.0.0:8000

# Con reload (desarrollo)
gunicorn miproyecto.wsgi:application \
  --reload \
  --bind 0.0.0.0:8000
```

**gunicorn_config.py**

```python
"""Configuración de Gunicorn"""
import os

# Bind
bind = f"0.0.0.0:{os.environ.get('PORT', '8000')}"

# Workers
workers = int(os.environ.get('WEB_CONCURRENCY', '3'))
worker_class = 'sync'
worker_connections = 1000
timeout = 30
keepalive = 2

# Logging
accesslog = '-'
errorlog = '-'
loglevel = 'info'

# Process naming
proc_name = 'miapp'

# Server mechanics
daemon = False
pidfile = None
user = None
group = None
tmp_upload_dir = None

# SSL (opcional)
# keyfile = '/path/to/key.pem'
# certfile = '/path/to/cert.pem'
```

**Ejecutar con config:**

```bash
gunicorn -c gunicorn_config.py miproyecto.wsgi:application
```

---

## Nginx (Reverse Proxy)

**Instalar:**

```bash
# Ubuntu/Debian
sudo apt install nginx

# Mac
brew install nginx
```

**Configuración:**

**/etc/nginx/sites-available/miapp**

```nginx
# Upstream (Gunicorn)
upstream miapp {
    server 127.0.0.1:8000;
}

# Redirect HTTP to HTTPS
server {
    listen 80;
    server_name miapp.com www.miapp.com;
    return 301 https://$server_name$request_uri;
}

# HTTPS Server
server {
    listen 443 ssl http2;
    server_name miapp.com www.miapp.com;
    
    # SSL Certificates
    ssl_certificate /etc/letsencrypt/live/miapp.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/miapp.com/privkey.pem;
    
    # SSL Configuration
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_prefer_server_ciphers on;
    ssl_ciphers ECDHE-RSA-AES256-GCM-SHA512:DHE-RSA-AES256-GCM-SHA512;
    
    # Logging
    access_log /var/log/nginx/miapp_access.log;
    error_log /var/log/nginx/miapp_error.log;
    
    # Max upload size
    client_max_body_size 10M;
    
    # Static files
    location /static/ {
        alias /home/usuario/miapp/staticfiles/;
        expires 30d;
        add_header Cache-Control "public, immutable";
    }
    
    # Media files
    location /media/ {
        alias /home/usuario/miapp/media/;
        expires 30d;
    }
    
    # Proxy to Django
    location / {
        proxy_pass http://miapp;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        
        # Timeouts
        proxy_connect_timeout 60s;
        proxy_send_timeout 60s;
        proxy_read_timeout 60s;
    }
}
```

**Activar configuración:**

```bash
# Crear symlink
sudo ln -s /etc/nginx/sites-available/miapp /etc/nginx/sites-enabled/

# Verificar configuración
sudo nginx -t

# Reiniciar Nginx
sudo systemctl restart nginx
```

---

## SSL con Let's Encrypt

**Instalar Certbot:**

```bash
# Ubuntu/Debian
sudo apt install certbot python3-certbot-nginx
```

**Obtener certificado:**

```bash
sudo certbot --nginx -d miapp.com -d www.miapp.com
```

**Renovación automática:**

```bash
# Verificar renovación
sudo certbot renew --dry-run

# Certbot crea un cron job automáticamente
# Verificar:
sudo systemctl list-timers
```

---

## Systemd Service

**Crear servicio:**

**/etc/systemd/system/miapp.service**

```ini
[Unit]
Description=Mi App Django
After=network.target

[Service]
Type=notify
User=usuario
Group=www-data
WorkingDirectory=/home/usuario/miapp
Environment="PATH=/home/usuario/miapp/venv/bin"
Environment="DJANGO_SETTINGS_MODULE=miproyecto.settings.production"
EnvironmentFile=/home/usuario/miapp/.env.production
ExecStart=/home/usuario/miapp/venv/bin/gunicorn \
    --workers 3 \
    --bind unix:/run/miapp.sock \
    miproyecto.wsgi:application
ExecReload=/bin/kill -s HUP $MAINPID
KillMode=mixed
TimeoutStopSec=5
PrivateTmp=true

[Install]
WantedBy=multi-user.target
```

**Activar servicio:**

```bash
# Recargar systemd
sudo systemctl daemon-reload

# Iniciar servicio
sudo systemctl start miapp

# Habilitar al inicio
sudo systemctl enable miapp

# Ver estado
sudo systemctl status miapp

# Ver logs
sudo journalctl -u miapp -f
```

---

## Static Files

### WhiteNoise

**Instalar:**

```bash
pip install whitenoise
```

**Configurar:**

```python
# settings/production.py
MIDDLEWARE = [
    'django.middleware.security.SecurityMiddleware',
    'whitenoise.middleware.WhiteNoiseMiddleware',  # ← Agregar aquí
    # ...
]

STATICFILES_STORAGE = 'whitenoise.storage.CompressedManifestStaticFilesStorage'
```

**Collectstatic:**

```bash
python manage.py collectstatic --noinput
```

### AWS S3

**Instalar:**

```bash
pip install boto3 django-storages
```

**Configurar:**

```python
# settings/production.py
INSTALLED_APPS += ['storages']

# AWS
AWS_ACCESS_KEY_ID = os.environ.get('AWS_ACCESS_KEY_ID')
AWS_SECRET_ACCESS_KEY = os.environ.get('AWS_SECRET_ACCESS_KEY')
AWS_STORAGE_BUCKET_NAME = os.environ.get('AWS_STORAGE_BUCKET_NAME')
AWS_S3_REGION_NAME = 'us-east-1'
AWS_S3_CUSTOM_DOMAIN = f'{AWS_STORAGE_BUCKET_NAME}.s3.amazonaws.com'

# Static files
STATICFILES_STORAGE = 'storages.backends.s3boto3.S3Boto3Storage'
STATIC_URL = f'https://{AWS_S3_CUSTOM_DOMAIN}/static/'

# Media files
DEFAULT_FILE_STORAGE = 'storages.backends.s3boto3.S3Boto3Storage'
MEDIA_URL = f'https://{AWS_S3_CUSTOM_DOMAIN}/media/'
```

---

## PostgreSQL

**Instalar:**

```bash
# Ubuntu/Debian
sudo apt install postgresql postgresql-contrib

# Mac
brew install postgresql
```

**Crear base de datos:**

```bash
# Conectar como postgres
sudo -u postgres psql

# Crear usuario y BD
CREATE DATABASE miapp_db;
CREATE USER miapp_user WITH PASSWORD 'password-segura';
ALTER ROLE miapp_user SET client_encoding TO 'utf8';
ALTER ROLE miapp_user SET default_transaction_isolation TO 'read committed';
ALTER ROLE miapp_user SET timezone TO 'UTC';
GRANT ALL PRIVILEGES ON DATABASE miapp_db TO miapp_user;

# Salir
\q
```

**Migrar:**

```bash
python manage.py migrate
```

---

## Checklist de Deployment

```bash
# Verificar configuración de seguridad
python manage.py check --deploy

# Output:
# System check identified some issues:
#
# WARNINGS:
# ?: (security.W004) You have not set a value for the SECURE_HSTS_SECONDS setting...
# ?: (security.W008) Your SECURE_SSL_REDIRECT setting is not set to True...
```

---

## Heroku

**Procfile**

```
web: gunicorn miproyecto.wsgi:application
release: python manage.py migrate
```

**runtime.txt**

```
python-3.11.7
```

**Desplegar:**

```bash
# Login
heroku login

# Crear app
heroku create miapp

# Agregar PostgreSQL
heroku addons:create heroku-postgresql:hobby-dev

# Variables de entorno
heroku config:set SECRET_KEY=tu-secret-key
heroku config:set DEBUG=False
heroku config:set ALLOWED_HOSTS=miapp.herokuapp.com

# Deploy
git push heroku main

# Migrar
heroku run python manage.py migrate

# Ver logs
heroku logs --tail
```

---

## Railway

**railway.json**

```json
{
  "$schema": "https://railway.app/railway.schema.json",
  "build": {
    "builder": "NIXPACKS"
  },
  "deploy": {
    "startCommand": "python manage.py migrate && gunicorn miproyecto.wsgi:application",
    "restartPolicyType": "ON_FAILURE",
    "restartPolicyMaxRetries": 10
  }
}
```

**Desplegar:**

```bash
# Instalar CLI
npm install -g @railway/cli

# Login
railway login

# Inicializar
railway init

# Agregar PostgreSQL
railway add

# Variables de entorno (en dashboard)

# Deploy
railway up
```

---

## Docker

**Dockerfile**

```dockerfile
FROM python:3.11-slim

# Variables de entorno
ENV PYTHONUNBUFFERED=1 \
    PYTHONDONTWRITEBYTECODE=1 \
    PIP_NO_CACHE_DIR=1

# Directorio de trabajo
WORKDIR /app

# Instalar dependencias del sistema
RUN apt-get update && apt-get install -y \
    postgresql-client \
    && rm -rf /var/lib/apt/lists/*

# Instalar dependencias Python
COPY requirements/production.txt .
RUN pip install -r production.txt

# Copiar código
COPY . .

# Collectstatic
RUN python manage.py collectstatic --noinput

# Exponer puerto
EXPOSE 8000

# Comando por defecto
CMD ["gunicorn", "miproyecto.wsgi:application", "--bind", "0.0.0.0:8000"]
```

**docker-compose.yml**

```yaml
version: '3.8'

services:
  db:
    image: postgres:15
    volumes:
      - postgres_data:/var/lib/postgresql/data
    environment:
      POSTGRES_DB: miapp_db
      POSTGRES_USER: miapp_user
      POSTGRES_PASSWORD: password
    
  web:
    build: .
    command: gunicorn miproyecto.wsgi:application --bind 0.0.0.0:8000
    volumes:
      - .:/app
      - static_volume:/app/staticfiles
    ports:
      - "8000:8000"
    env_file:
      - .env.production
    depends_on:
      - db
  
  nginx:
    image: nginx:alpine
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf
      - static_volume:/staticfiles
    ports:
      - "80:80"
    depends_on:
      - web

volumes:
  postgres_data:
  static_volume:
```

**Ejecutar:**

```bash
# Build
docker-compose build

# Up
docker-compose up -d

# Migrar
docker-compose exec web python manage.py migrate

# Ver logs
docker-compose logs -f web
```

---

## Monitoring con Sentry

**Instalar:**

```bash
pip install sentry-sdk
```

**Configurar:**

```python
# settings/production.py
import sentry_sdk
from sentry_sdk.integrations.django import DjangoIntegration

sentry_sdk.init(
    dsn=os.environ.get('SENTRY_DSN'),
    integrations=[DjangoIntegration()],
    traces_sample_rate=1.0,
    send_default_pii=True
)
```

---

## Resumen

Has aprendido:

✅ Settings separados (dev/prod)  
✅ Gunicorn (WSGI server)  
✅ Nginx (reverse proxy)  
✅ SSL con Let's Encrypt  
✅ Systemd service  
✅ Static files (WhiteNoise/S3)  
✅ PostgreSQL  
✅ Deployment en Heroku/Railway  
✅ Docker deployment  
✅ Monitoring con Sentry

**¡Has completado el curso de Django!** 🎉

---

## Recursos

- **[Deployment checklist](https://docs.djangoproject.com/en/5.0/howto/deployment/checklist/)** - Checklist oficial
- **[Gunicorn](https://docs.gunicorn.org/)** - Gunicorn docs
- **[Let's Encrypt](https://letsencrypt.org/)** - SSL gratis
- **[Railway](https://railway.app/)** - Hosting fácil

Tu app Django está lista para producción. 🚀

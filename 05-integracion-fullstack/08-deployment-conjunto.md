# Deployment Conjunto

> **Desplegar Frontend (Vercel) + Backend (Railway) + Base de Datos (PostgreSQL)**

---

## Arquitectura de deployment

```
┌─────────────────────────────────────┐
│   Vercel (Frontend)                 │
│   - React build estático            │
│   - CDN global                      │
│   - HTTPS automático                │
│   - Dominio: miapp.vercel.app       │
└──────────────┬──────────────────────┘
               │
               │ HTTPS Requests
               ↓
┌─────────────────────────────────────┐
│   Railway (Backend)                 │
│   - Django + Gunicorn               │
│   - HTTPS automático                │
│   - Dominio: *.up.railway.app       │
└──────────────┬──────────────────────┘
               │
               ↓
         ┌─────────┐
         │PostgreSQL│
         │(Railway) │
         └─────────┘
```

---

## Backend: Preparar Django para producción

### 1. Instalar dependencias de producción

```bash
pip install gunicorn whitenoise python-decouple psycopg2-binary dj-database-url
```

### 2. requirements.txt

```txt
Django==5.0
djangorestframework==3.14.0
djangorestframework-simplejwt==5.3.1
django-cors-headers==4.3.1
gunicorn==21.2.0
whitenoise==6.6.0
python-decouple==3.8
psycopg2-binary==2.9.9
dj-database-url==2.1.0
cloudinary==1.36.0
django-cloudinary-storage==0.3.0
```

**Generar automáticamente:**
```bash
pip freeze > requirements.txt
```

### 3. Configurar settings.py

```python
# core/settings.py
import os
from decouple import config
import dj_database_url

# SECURITY WARNING: keep the secret key used in production secret!
SECRET_KEY = config('SECRET_KEY', default='django-insecure-dev-key')

# SECURITY WARNING: don't run with debug turned on in production!
DEBUG = config('DEBUG', default=False, cast=bool)

ALLOWED_HOSTS = config('ALLOWED_HOSTS', default='localhost,127.0.0.1').split(',')

# Database
if DEBUG:
    # Desarrollo: SQLite
    DATABASES = {
        'default': {
            'ENGINE': 'django.db.backends.sqlite3',
            'NAME': BASE_DIR / 'db.sqlite3',
        }
    }
else:
    # Producción: PostgreSQL (Railway)
    DATABASES = {
        'default': dj_database_url.config(
            default=config('DATABASE_URL'),
            conn_max_age=600
        )
    }

# Static files (CSS, JavaScript)
STATIC_URL = '/static/'
STATIC_ROOT = os.path.join(BASE_DIR, 'staticfiles')
STATICFILES_STORAGE = 'whitenoise.storage.CompressedManifestStaticFilesStorage'

# CORS
if DEBUG:
    CORS_ALLOWED_ORIGINS = [
        "http://localhost:5173",
        "http://localhost:3000",
    ]
else:
    CORS_ALLOWED_ORIGINS = config('CORS_ALLOWED_ORIGINS', cast=lambda v: [s.strip() for s in v.split(',')])

CORS_ALLOW_CREDENTIALS = True

# Security settings (producción)
if not DEBUG:
    SECURE_SSL_REDIRECT = True
    SESSION_COOKIE_SECURE = True
    CSRF_COOKIE_SECURE = True
    SECURE_BROWSER_XSS_FILTER = True
    SECURE_CONTENT_TYPE_NOSNIFF = True
```

### 4. WhiteNoise para archivos estáticos

```python
# core/settings.py
MIDDLEWARE = [
    'django.middleware.security.SecurityMiddleware',
    'whitenoise.middleware.WhiteNoiseMiddleware',  # ← Después de SecurityMiddleware
    'corsheaders.middleware.CorsMiddleware',
    # ... resto
]
```

### 5. .env (ejemplo)

```env
# Desarrollo
DEBUG=True
SECRET_KEY=django-insecure-dev-key-123456
DATABASE_URL=sqlite:///db.sqlite3
ALLOWED_HOSTS=localhost,127.0.0.1
CORS_ALLOWED_ORIGINS=http://localhost:5173,http://localhost:3000

# Cloudinary
CLOUDINARY_CLOUD_NAME=tu-cloud-name
CLOUDINARY_API_KEY=tu-api-key
CLOUDINARY_API_SECRET=tu-api-secret
```

### 6. .env.production (ejemplo)

```env
# Producción
DEBUG=False
SECRET_KEY=super-secret-key-production-123456
DATABASE_URL=postgresql://user:password@host:5432/dbname
ALLOWED_HOSTS=miapp.up.railway.app,midominio.com
CORS_ALLOWED_ORIGINS=https://miapp.vercel.app,https://www.midominio.com

# Cloudinary
CLOUDINARY_CLOUD_NAME=tu-cloud-name
CLOUDINARY_API_KEY=tu-api-key
CLOUDINARY_API_SECRET=tu-api-secret
```

### 7. Procfile (Railway)

```procfile
web: gunicorn core.wsgi --log-file -
```

### 8. runtime.txt

```txt
python-3.11.7
```

---

## Railway: Deploy Backend

### 1. Crear cuenta

1. Ir a [railway.app](https://railway.app/)
2. Sign up con GitHub

### 2. Crear proyecto

1. New Project
2. Provision PostgreSQL (agregar servicio)
3. New Service → Deploy from GitHub repo
4. Seleccionar tu repositorio backend

### 3. Variables de entorno

En el dashboard de Railway (pestaña "Variables"):

```
DEBUG=False
SECRET_KEY=tu-secret-key-super-segura-123456
ALLOWED_HOSTS=*.up.railway.app
CORS_ALLOWED_ORIGINS=https://tuapp.vercel.app

# PostgreSQL (auto-generada al crear servicio)
DATABASE_URL=${{Postgres.DATABASE_URL}}

# Cloudinary
CLOUDINARY_CLOUD_NAME=tu-cloud-name
CLOUDINARY_API_KEY=tu-api-key
CLOUDINARY_API_SECRET=tu-api-secret
```

### 4. Configurar build

Railway detecta automáticamente:
- `requirements.txt` → instala dependencias
- `Procfile` → ejecuta comando

**Si necesitas ejecutar migraciones:**

En Railway Settings → Deploy → Add Custom Start Command:
```bash
python manage.py migrate && gunicorn core.wsgi --log-file -
```

O crear `railway.json`:
```json
{
  "$schema": "https://railway.app/railway.schema.json",
  "build": {
    "builder": "NIXPACKS"
  },
  "deploy": {
    "startCommand": "python manage.py migrate && python manage.py collectstatic --noinput && gunicorn core.wsgi --log-file -",
    "healthcheckPath": "/api/",
    "restartPolicyType": "ON_FAILURE",
    "restartPolicyMaxRetries": 10
  }
}
```

### 5. Deploy

1. Push a GitHub
2. Railway auto-deploya
3. Ver logs en tiempo real

**URL:** `https://tu-proyecto.up.railway.app`

---

## Frontend: Preparar React para producción

### 1. Variables de entorno

```env
# .env.development
VITE_API_URL=http://localhost:8000/api

# .env.production
VITE_API_URL=https://tu-proyecto.up.railway.app/api
```

### 2. Axios con variable de entorno

```javascript
// src/services/api.js
import axios from 'axios';

const API = axios.create({
    baseURL: import.meta.env.VITE_API_URL || 'http://localhost:8000/api',
    headers: {
        'Content-Type': 'application/json',
    },
});

export default API;
```

### 3. Build para producción

```bash
npm run build
```

Genera carpeta `dist/` con archivos estáticos optimizados.

---

## Vercel: Deploy Frontend

### 1. Crear cuenta

1. Ir a [vercel.com](https://vercel.com/)
2. Sign up con GitHub

### 2. Importar proyecto

1. Add New → Project
2. Import Git Repository
3. Seleccionar repositorio frontend

### 3. Configurar build

Vercel detecta automáticamente Vite:

**Framework Preset:** Vite  
**Build Command:** `npm run build`  
**Output Directory:** `dist`

### 4. Variables de entorno

En Project Settings → Environment Variables:

```
VITE_API_URL=https://tu-proyecto.up.railway.app/api
```

### 5. Deploy

1. Deploy
2. Vercel construye y despliega automáticamente

**URL:** `https://tu-app.vercel.app`

---

## Conectar Frontend y Backend

### 1. Actualizar CORS en Django

En Railway, agregar variable de entorno:

```
CORS_ALLOWED_ORIGINS=https://tu-app.vercel.app
```

O si tienes múltiples dominios:
```
CORS_ALLOWED_ORIGINS=https://tu-app.vercel.app,https://www.tudominio.com
```

### 2. Actualizar ALLOWED_HOSTS

```
ALLOWED_HOSTS=tu-proyecto.up.railway.app,www.tudominio.com
```

### 3. Probar integración

```
https://tu-app.vercel.app → https://tu-proyecto.up.railway.app/api/
```

---

## Custom domains

### Frontend (Vercel)

1. Project Settings → Domains
2. Add Domain → `www.tudominio.com`
3. Configurar DNS en tu proveedor:
   ```
   CNAME www → cname.vercel-dns.com
   ```

### Backend (Railway)

1. Project Settings → Networking → Custom Domain
2. Add Domain → `api.tudominio.com`
3. Configurar DNS:
   ```
   CNAME api → tu-proyecto.up.railway.app
   ```

4. Actualizar variables de entorno:
   ```
   ALLOWED_HOSTS=api.tudominio.com
   ```

5. Actualizar frontend:
   ```
   VITE_API_URL=https://api.tudominio.com
   ```

---

## CI/CD automático

### Railway

**Auto-deploy:**
- Push a `main` → Railway despliega automáticamente

**Branch previews:**
- PR → Railway crea deploy temporal

### Vercel

**Auto-deploy:**
- Push a `main` → Vercel despliega a producción
- Push a otra branch → Preview deployment

---

## Checklist de deployment

### Backend (Django)

- ✅ `DEBUG=False`
- ✅ `SECRET_KEY` segura
- ✅ `ALLOWED_HOSTS` configurado
- ✅ `CORS_ALLOWED_ORIGINS` correcto
- ✅ Base de datos PostgreSQL
- ✅ Cloudinary para archivos
- ✅ WhiteNoise para estáticos
- ✅ Migraciones aplicadas
- ✅ Gunicorn instalado
- ✅ Variables de entorno en Railway

### Frontend (React)

- ✅ `VITE_API_URL` apunta a backend
- ✅ Build funciona (`npm run build`)
- ✅ Variables de entorno en Vercel
- ✅ HTTPS habilitado

---

## Monitoreo y logs

### Railway

**Ver logs:**
1. Dashboard → Tu servicio
2. Pestaña "Deployments"
3. Ver logs en tiempo real

**Métricas:**
- CPU, RAM, Network
- Requests por segundo

### Vercel

**Ver logs:**
1. Project → Deployments
2. Click en deployment
3. Ver logs de build y runtime

---

## Alternativas

### Backend

- **Railway** (recomendado) - Simple, tier gratuito generoso
- **Render** - Similar a Railway
- **Heroku** - Tier gratuito eliminado
- **AWS Elastic Beanstalk** - Más complejo
- **DigitalOcean App Platform** - $5/mes mínimo

### Frontend

- **Vercel** (recomendado para React/Next.js)
- **Netlify** - Similar a Vercel
- **GitHub Pages** - Solo sitios estáticos
- **Cloudflare Pages** - Rápido y gratuito

### Base de datos

- **Railway PostgreSQL** (recomendado) - Incluido
- **Neon** - PostgreSQL serverless
- **Supabase** - PostgreSQL + backend-as-a-service
- **ElephantSQL** - PostgreSQL managed

---

## Debugging en producción

### Backend no responde

1. Verificar logs en Railway
2. Verificar variables de entorno
3. Verificar migraciones: `railway run python manage.py migrate`
4. Verificar URL: `https://tu-proyecto.up.railway.app/api/`

### Frontend no conecta con backend

1. Verificar `VITE_API_URL` en Vercel
2. Verificar CORS en Django
3. Abrir DevTools → Network → ver requests

### Error 502/503

- Backend caído → ver logs en Railway
- Base de datos caída → verificar servicio PostgreSQL

---

## Resumen

**Flujo de deployment:**

1. **Backend (Railway):**
   - Push a GitHub
   - Railway detecta `requirements.txt` y `Procfile`
   - Ejecuta migraciones y collectstatic
   - Levanta Gunicorn
   - URL: `https://*.up.railway.app`

2. **Frontend (Vercel):**
   - Push a GitHub
   - Vercel ejecuta `npm run build`
   - Despliega archivos estáticos
   - URL: `https://*.vercel.app`

3. **Conectar:**
   - Backend: `CORS_ALLOWED_ORIGINS` incluye frontend
   - Frontend: `VITE_API_URL` apunta a backend

**Ventajas:**
- ✅ Gratis para proyectos pequeños
- ✅ CI/CD automático
- ✅ HTTPS automático
- ✅ Escalable
- ✅ Fácil de configurar

---

## Recursos

- **[Railway Docs](https://docs.railway.app/)** - Documentación oficial
- **[Vercel Docs](https://vercel.com/docs)** - Documentación oficial
- **[Django Deployment Checklist](https://docs.djangoproject.com/en/5.0/howto/deployment/checklist/)** - Guía oficial

¡Aplicación fullstack desplegada! 🚀

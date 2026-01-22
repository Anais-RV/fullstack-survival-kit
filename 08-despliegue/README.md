# Despliegue

> **Llevar aplicaciones FullStack a producción de manera profesional**

---

## ¿Por qué despliegue profesional?

Desplegar correctamente es crítico:

- ✅ **Disponibilidad:** 99.9% uptime
- ✅ **Seguridad:** HTTPS, secretos protegidos
- ✅ **Escalabilidad:** Manejar más usuarios sin caerse
- ✅ **Monitoreo:** Detectar problemas antes que los usuarios
- ✅ **Automatización:** Deploy sin errores humanos

---

## Contenido del módulo

### 01 - Docker y Containerización
**Empaquetar aplicaciones en containers portables**

#### Conceptos clave:
- **Docker:** Containers ligeros con tu app + dependencias
- **Dockerfile:** Receta para construir imagen
- **Docker Compose:** Orquestar múltiples containers

#### Estructura:
```
proyecto/
├── backend/
│   └── Dockerfile              # Django + Gunicorn
├── frontend/
│   ├── Dockerfile              # React + Nginx
│   └── nginx.conf
└── docker-compose.yml          # Backend + Frontend + PostgreSQL
```

#### Ejemplo Dockerfile (Django):
```dockerfile
FROM python:3.11-slim
WORKDIR /app
COPY requirements.txt .
RUN pip install -r requirements.txt
COPY . .
CMD ["gunicorn", "core.wsgi:application", "--bind", "0.0.0.0:8000"]
```

#### Comandos esenciales:
```bash
docker-compose up -d              # Levantar servicios
docker-compose logs -f backend    # Ver logs
docker-compose exec backend python manage.py migrate
docker-compose down               # Detener
```

**Beneficios:**
- Mismo ambiente en dev, staging y producción
- Fácil escalabilidad (replicar containers)
- Aislamiento entre proyectos

**Tiempo estimado:** 3 horas

---

### 02 - CI/CD con GitHub Actions
**Automatizar testing, build y deployment**

#### Workflow típico:
```
Push a GitHub
  → Tests automáticos
  → Build
  → Deploy a producción
  → Notificaciones
```

#### Archivo de configuración:
```.github/workflows/production.yml
name: Production Deployment

on:
  push:
    branches: [ main ]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Run tests
        run: pytest --cov
  
  deploy:
    needs: test
    runs-on: ubuntu-latest
    steps:
      - name: Deploy to Railway
        env:
          RAILWAY_TOKEN: ${{ secrets.RAILWAY_TOKEN }}
        run: railway up
```

#### Features:
- Tests automáticos en cada PR
- Deploy solo si tests pasan
- Matrix testing (múltiples versiones Python/Node)
- Notificaciones a Slack/Email
- Branch protection (prevenir merge sin tests)

#### Secrets management:
```yaml
env:
  DATABASE_URL: ${{ secrets.DATABASE_URL }}
  SECRET_KEY: ${{ secrets.SECRET_KEY }}
```

**Beneficios:**
- Zero downtime deployments
- Detectar bugs antes de producción
- Proceso consistente y automatizado

**Tiempo estimado:** 2 horas

---

## Stack de deployment recomendado

### Backend (Django)

**Opción 1: Railway** (recomendado para principiantes)
- ✅ PostgreSQL incluido
- ✅ Deploy con Git push
- ✅ Variables de entorno
- ✅ SSL automático
- ✅ $5/mes (tier gratuito limitado)

**Opción 2: Render**
- ✅ Tier gratuito generoso
- ✅ Auto-scaling
- ✅ Background workers

**Opción 3: AWS/GCP/Azure** (avanzado)
- ✅ Máximo control
- ✅ Escalabilidad ilimitada
- ❌ Más complejo

---

### Frontend (React)

**Opción 1: Vercel** (recomendado)
- ✅ Deploy automático con Git
- ✅ SSL y CDN global
- ✅ Preview deployments para PRs
- ✅ Tier gratuito generoso

**Opción 2: Netlify**
- ✅ Similar a Vercel
- ✅ Plugins útiles
- ✅ Forms handling

**Opción 3: Cloudflare Pages**
- ✅ CDN ultra-rápido
- ✅ Gratis ilimitado

---

### Base de datos

**Opción 1: Railway PostgreSQL**
- ✅ Incluido con backend
- ✅ Backups automáticos

**Opción 2: Supabase**
- ✅ PostgreSQL managed
- ✅ Auth + Storage incluido
- ✅ Tier gratuito

**Opción 3: AWS RDS**
- ✅ Producción enterprise
- ❌ Más caro

---

## Checklist de deployment

### Backend

```python
# settings.py
DEBUG = False  # ✅ CRÍTICO

ALLOWED_HOSTS = ['tudominio.com', 'api.tudominio.com']

SECRET_KEY = os.getenv('SECRET_KEY')  # Desde variable de entorno

DATABASES = {
    'default': dj_database_url.config(
        default=os.getenv('DATABASE_URL')
    )
}

# CORS
CORS_ALLOWED_ORIGINS = [
    'https://tudominio.com',
]

# Static files
STATIC_ROOT = os.path.join(BASE_DIR, 'staticfiles')
STATICFILES_STORAGE = 'whitenoise.storage.CompressedManifestStaticFilesStorage'

# Security
SECURE_SSL_REDIRECT = True
SESSION_COOKIE_SECURE = True
CSRF_COOKIE_SECURE = True
```

```txt
# requirements.txt (producción)
Django==5.0
djangorestframework==3.14.0
gunicorn==21.2.0          # WSGI server
whitenoise==6.6.0         # Static files
psycopg2-binary==2.9.9    # PostgreSQL
dj-database-url==2.1.0    # Parse DATABASE_URL
python-decouple==3.8      # Environment variables
```

---

### Frontend

```javascript
// vite.config.js
export default defineConfig({
    build: {
        outDir: 'dist',
        sourcemap: false,  // No source maps en producción
        minify: 'terser',
        terserOptions: {
            compress: {
                drop_console: true,  // Eliminar console.logs
            }
        }
    }
});
```

```javascript
// .env.production
VITE_API_URL=https://api.tudominio.com
```

---

## Proceso de deployment completo

### 1. Desarrollo local

```bash
# Docker Compose para desarrollo
docker-compose up

# Trabajar en features
git checkout -b feature/nueva-funcionalidad

# Commits frecuentes
git commit -m "feat: agregar login con JWT"
```

---

### 2. Testing

```bash
# Backend
cd backend
pytest --cov

# Frontend
cd frontend
npm run test -- --coverage

# E2E
npx playwright test
```

---

### 3. Pull Request

```bash
git push origin feature/nueva-funcionalidad

# Crear PR en GitHub
# → GitHub Actions ejecuta tests automáticamente
# → Code review del equipo
# → Merge solo si tests pasan
```

---

### 4. Deploy automático

```bash
# Al hacer merge a main:
git checkout main
git pull

# GitHub Actions automáticamente:
# 1. Ejecuta todos los tests
# 2. Build del frontend
# 3. Deploy a Railway (backend)
# 4. Deploy a Vercel (frontend)
# 5. Notificación a Slack
```

---

### 5. Verificación post-deploy

```bash
# Health checks
curl https://api.tudominio.com/health/

# Ver logs
railway logs

# Monitoreo
# → Sentry captura errores
# → Uptime monitoring (UptimeRobot)
```

---

## Estrategias de deployment

### Blue-Green Deployment

```
Blue (actual) ← 100% tráfico
Green (nuevo) ← 0% tráfico

Deploy a Green → Tests → Switch tráfico

Blue ← 0% tráfico
Green (nuevo) ← 100% tráfico
```

**Beneficios:**
- Zero downtime
- Rollback instantáneo

---

### Canary Deployment

```
Actual ← 95% tráfico
Canary (nuevo) ← 5% tráfico

Monitorear errores → Si OK, aumentar gradualmente

Actual ← 0%
Canary ← 100%
```

**Beneficios:**
- Detectar problemas con pocos usuarios
- Rollback antes de afectar a todos

---

## Configuración de dominios

### Backend (Railway)

```
1. Railway → Settings → Domains
2. Agregar custom domain: api.tudominio.com
3. Configurar DNS:
   - Type: CNAME
   - Name: api
   - Value: <railway-url>.railway.app
```

### Frontend (Vercel)

```
1. Vercel → Project → Settings → Domains
2. Add domain: tudominio.com
3. Configurar DNS:
   - Type: A
   - Name: @
   - Value: 76.76.21.21
   
   - Type: CNAME
   - Name: www
   - Value: cname.vercel-dns.com
```

---

## Rollback (deshacer deployment)

### Railway

```bash
# Ver deployments anteriores
railway status

# Rollback al anterior
railway rollback
```

### Vercel

```bash
# CLI
vercel rollback

# O en Dashboard:
# Deployments → Select previous → Promote to Production
```

---

## Resumen

**Deployment moderno:**

```
Código → GitHub → CI/CD → Producción
```

**Stack recomendado:**
- **Backend:** Railway (Django + PostgreSQL + Gunicorn)
- **Frontend:** Vercel (React + Vite)
- **CI/CD:** GitHub Actions

**Checklist:**
- ✅ DEBUG=False
- ✅ SECRET_KEY desde variables de entorno
- ✅ HTTPS (SSL)
- ✅ CORS configurado
- ✅ Tests automáticos
- ✅ Monitoreo (próximo módulo)

**Workflow:**
1. Desarrollo local con Docker Compose
2. Tests automáticos en cada PR
3. Deploy automático a producción
4. Monitoreo y logs

---

## Próximo módulo: 09-proyecto-integrador

En el próximo módulo crearemos un proyecto completo aplicando todo lo aprendido:
- E-commerce con Django + React
- Autenticación JWT
- CRUD completo
- Testing (Unit + Integration + E2E)
- Docker + CI/CD
- Deploy a producción

¡Despliegue completo! 🚀✨

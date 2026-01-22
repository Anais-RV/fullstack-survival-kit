# Docker y Containerización

> **Empaquetar aplicaciones FullStack en containers portables**

---

## ¿Qué es Docker?

**Docker** = Empaquetar tu aplicación + todas sus dependencias en un "container"

**Beneficios:**
- ✅ **Portabilidad:** Funciona igual en dev, staging y producción
- ✅ **Aislamiento:** No conflictos entre proyectos
- ✅ **Reproducibilidad:** Mismo ambiente para todo el equipo
- ✅ **Escalabilidad:** Replicar containers fácilmente

---

## Conceptos básicos

### Image (Imagen)
> Plantilla read-only con tu aplicación

```
Python 3.11 + Django + dependencias → Imagen
Node 18 + React + dependencias → Imagen
```

### Container (Contenedor)
> Instancia en ejecución de una imagen

```
Imagen → docker run → Container (proceso corriendo)
```

### Dockerfile
> Receta para construir una imagen

```dockerfile
FROM python:3.11
COPY . /app
RUN pip install -r requirements.txt
CMD ["python", "manage.py", "runserver"]
```

---

## Instalar Docker

### Windows/Mac
1. Descargar [Docker Desktop](https://www.docker.com/products/docker-desktop)
2. Instalar y reiniciar
3. Verificar:
```bash
docker --version
docker-compose --version
```

### Linux
```bash
curl -fsSL https://get.docker.com -o get-docker.sh
sh get-docker.sh
sudo usermod -aG docker $USER
```

---

## Dockerfile para Django

```dockerfile
# 08-despliegue/backend/Dockerfile
FROM python:3.11-slim

# Establecer directorio de trabajo
WORKDIR /app

# Instalar dependencias del sistema
RUN apt-get update && apt-get install -y \
    gcc \
    postgresql-client \
    && rm -rf /var/lib/apt/lists/*

# Copiar requirements
COPY requirements.txt .

# Instalar dependencias de Python
RUN pip install --no-cache-dir -r requirements.txt

# Copiar código
COPY . .

# Collectstatic (para producción)
RUN python manage.py collectstatic --noinput

# Exponer puerto
EXPOSE 8000

# Comando por defecto
CMD ["gunicorn", "core.wsgi:application", "--bind", "0.0.0.0:8000"]
```

### requirements.txt (producción)
```txt
Django==5.0
djangorestframework==3.14.0
djangorestframework-simplejwt==5.3.0
django-cors-headers==4.3.0
psycopg2-binary==2.9.9
gunicorn==21.2.0
whitenoise==6.6.0
python-decouple==3.8
Pillow==10.1.0
```

---

## Dockerfile para React

```dockerfile
# 08-despliegue/frontend/Dockerfile
FROM node:18-alpine AS builder

WORKDIR /app

# Copiar package.json y package-lock.json
COPY package*.json ./

# Instalar dependencias
RUN npm ci

# Copiar código
COPY . .

# Build de producción
RUN npm run build

# Etapa de producción (Nginx)
FROM nginx:alpine

# Copiar build a Nginx
COPY --from=builder /app/dist /usr/share/nginx/html

# Copiar configuración de Nginx
COPY nginx.conf /etc/nginx/conf.d/default.conf

EXPOSE 80

CMD ["nginx", "-g", "daemon off;"]
```

### nginx.conf
```nginx
server {
    listen 80;
    server_name localhost;
    
    root /usr/share/nginx/html;
    index index.html;
    
    # React Router (SPA)
    location / {
        try_files $uri $uri/ /index.html;
    }
    
    # Cache estáticos
    location ~* \.(js|css|png|jpg|jpeg|gif|ico|svg)$ {
        expires 1y;
        add_header Cache-Control "public, immutable";
    }
}
```

---

## Docker Compose

**Docker Compose** = Definir y ejecutar múltiples containers

```yaml
# docker-compose.yml
version: '3.8'

services:
  # PostgreSQL
  db:
    image: postgres:15
    environment:
      POSTGRES_DB: ecommerce
      POSTGRES_USER: admin
      POSTGRES_PASSWORD: secret123
    volumes:
      - postgres_data:/var/lib/postgresql/data
    ports:
      - "5432:5432"
  
  # Backend (Django)
  backend:
    build: ./backend
    command: gunicorn core.wsgi:application --bind 0.0.0.0:8000
    volumes:
      - ./backend:/app
      - static_volume:/app/staticfiles
      - media_volume:/app/media
    ports:
      - "8000:8000"
    environment:
      - DEBUG=False
      - DATABASE_URL=postgresql://admin:secret123@db:5432/ecommerce
      - SECRET_KEY=your-secret-key-here
      - ALLOWED_HOSTS=localhost,127.0.0.1
    depends_on:
      - db
  
  # Frontend (React)
  frontend:
    build: ./frontend
    ports:
      - "3000:80"
    depends_on:
      - backend

volumes:
  postgres_data:
  static_volume:
  media_volume:
```

---

## Comandos Docker

### Construir imagen
```bash
docker build -t mi-backend .
```

### Ejecutar container
```bash
docker run -p 8000:8000 mi-backend
```

### Ver containers corriendo
```bash
docker ps
```

### Ver todos los containers
```bash
docker ps -a
```

### Detener container
```bash
docker stop <container_id>
```

### Eliminar container
```bash
docker rm <container_id>
```

### Ver logs
```bash
docker logs <container_id>
docker logs -f <container_id>  # Follow (tiempo real)
```

### Entrar al container
```bash
docker exec -it <container_id> bash
```

---

## Comandos Docker Compose

### Levantar todos los servicios
```bash
docker-compose up
docker-compose up -d  # Background
```

### Ver logs
```bash
docker-compose logs
docker-compose logs backend  # Solo backend
docker-compose logs -f  # Follow
```

### Detener servicios
```bash
docker-compose stop
```

### Detener y eliminar containers
```bash
docker-compose down
```

### Detener, eliminar y eliminar volumes
```bash
docker-compose down -v
```

### Rebuild de imágenes
```bash
docker-compose build
docker-compose up --build
```

### Ejecutar comando en servicio
```bash
docker-compose exec backend python manage.py migrate
docker-compose exec backend python manage.py createsuperuser
```

---

## .dockerignore

**No incluir en imagen:**

```
# .dockerignore (backend)
__pycache__
*.pyc
*.pyo
*.pyd
.Python
env/
venv/
.venv
*.db
*.sqlite3
.env
.git
.gitignore
node_modules/
```

```
# .dockerignore (frontend)
node_modules/
dist/
build/
.env
.git
.gitignore
*.log
.DS_Store
```

---

## Workflow completo

### 1. Desarrollo local

```bash
# Levantar todos los servicios
docker-compose up -d

# Ver logs
docker-compose logs -f backend

# Ejecutar migraciones
docker-compose exec backend python manage.py migrate

# Crear superuser
docker-compose exec backend python manage.py createsuperuser

# Instalar nuevas dependencias
docker-compose exec backend pip install nueva-libreria
docker-compose restart backend
```

### 2. Acceder a la app

```
Frontend: http://localhost:3000
Backend: http://localhost:8000
Admin: http://localhost:8000/admin
```

### 3. Detener

```bash
docker-compose down
```

---

## Multi-stage builds (optimización)

### Backend optimizado

```dockerfile
FROM python:3.11-slim AS base

WORKDIR /app

# Dependencias del sistema
RUN apt-get update && apt-get install -y \
    gcc \
    postgresql-client \
    && rm -rf /var/lib/apt/lists/*

FROM base AS builder

# Instalar dependencias
COPY requirements.txt .
RUN pip install --user -r requirements.txt

FROM base AS production

# Copiar dependencias desde builder
COPY --from=builder /root/.local /root/.local
ENV PATH=/root/.local/bin:$PATH

# Copiar código
COPY . .

# Collectstatic
RUN python manage.py collectstatic --noinput

EXPOSE 8000

CMD ["gunicorn", "core.wsgi:application", "--bind", "0.0.0.0:8000"]
```

---

## Variables de entorno

### .env
```
# .env
DEBUG=False
SECRET_KEY=your-super-secret-key-change-this
DATABASE_URL=postgresql://admin:secret123@db:5432/ecommerce
ALLOWED_HOSTS=localhost,127.0.0.1
CORS_ALLOWED_ORIGINS=http://localhost:3000
```

### docker-compose.yml (usar .env)
```yaml
version: '3.8'

services:
  backend:
    build: ./backend
    env_file:
      - .env
    depends_on:
      - db
```

---

## Health checks

```yaml
version: '3.8'

services:
  backend:
    build: ./backend
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:8000/api/health/"]
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 40s
    depends_on:
      db:
        condition: service_healthy
  
  db:
    image: postgres:15
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U admin"]
      interval: 10s
      timeout: 5s
      retries: 5
```

---

## Volumes para desarrollo

```yaml
version: '3.8'

services:
  backend:
    build: ./backend
    volumes:
      - ./backend:/app  # Hot reload en desarrollo
      - static_volume:/app/staticfiles
    environment:
      - DEBUG=True
    command: python manage.py runserver 0.0.0.0:8000
  
  frontend:
    build: ./frontend
    volumes:
      - ./frontend/src:/app/src  # Hot reload
    command: npm run dev

volumes:
  static_volume:
```

---

## Seguridad

### No hardcodear secretos

```dockerfile
# ❌ Mal
ENV SECRET_KEY=hardcoded-secret-key

# ✅ Bien (pasar en runtime)
ENV SECRET_KEY=${SECRET_KEY}
```

### Usar usuario no-root

```dockerfile
# Crear usuario no-root
RUN addgroup --system app && adduser --system --group app

# Cambiar permisos
RUN chown -R app:app /app

# Cambiar a usuario app
USER app
```

---

## Debugging

### Ver logs detallados
```bash
docker-compose logs --tail=100 backend
```

### Inspeccionar container
```bash
docker inspect <container_id>
```

### Estadísticas de recursos
```bash
docker stats
```

### Network
```bash
docker network ls
docker network inspect <network_name>
```

---

## Resumen

**Docker** = Containers portables con tu app + dependencias

**Dockerfile** = Receta para construir imagen

**Docker Compose** = Orquestar múltiples containers (backend + frontend + BD)

**Comandos clave:**
```bash
docker build -t mi-app .
docker run -p 8000:8000 mi-app
docker-compose up
docker-compose logs -f
docker-compose exec backend python manage.py migrate
docker-compose down
```

**Próximo módulo:** CI/CD con GitHub Actions

---

## Recursos

- **[Docker Documentation](https://docs.docker.com/)** - Docs oficiales
- **[Docker Compose](https://docs.docker.com/compose/)** - Docs de Compose
- **[Docker Best Practices](https://docs.docker.com/develop/dev-best-practices/)** - Best practices
- **[Play with Docker](https://labs.play-with-docker.com/)** - Practicar online

¡Docker listo! 🐳

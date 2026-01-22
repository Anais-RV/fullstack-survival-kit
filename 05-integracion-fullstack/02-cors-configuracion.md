# CORS y Configuración

> **Cross-Origin Resource Sharing: Permitir comunicación entre frontend y backend**

---

## ¿Qué es CORS?

**CORS (Cross-Origin Resource Sharing)** es un mecanismo de seguridad del navegador que controla qué dominios pueden acceder a recursos de tu API.

**Problema:**
```
Frontend:  http://localhost:5173
Backend:   http://localhost:8000

❌ CORS policy: No 'Access-Control-Allow-Origin' header
```

**Sin CORS configurado**, el navegador **bloquea** requests entre diferentes orígenes (puerto, dominio, protocolo).

---

## Same-Origin Policy

**Mismo origen = mismo protocolo + dominio + puerto**

```
✅ MISMO ORIGEN:
http://localhost:8000/api/productos/
http://localhost:8000/api/usuarios/

❌ DIFERENTE ORIGEN:
http://localhost:5173   (puerto diferente)
https://localhost:8000  (protocolo diferente)
http://ejemplo.com:8000 (dominio diferente)
```

**El navegador bloquea requests entre diferentes orígenes por seguridad.**

---

## Instalar django-cors-headers

```bash
pip install django-cors-headers
```

---

## Configurar Django

### 1. Agregar a INSTALLED_APPS

```python
# core/settings.py

INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    
    # Third party
    'rest_framework',
    'corsheaders',  # ← Agregar
    
    # Apps
    'productos',
]
```

### 2. Agregar middleware (IMPORTANTE: AL INICIO)

```python
MIDDLEWARE = [
    'corsheaders.middleware.CorsMiddleware',  # ← PRIMERO
    'django.middleware.security.SecurityMiddleware',
    'django.contrib.sessions.middleware.SessionMiddleware',
    'django.middleware.common.CommonMiddleware',
    'django.middleware.csrf.CsrfViewMiddleware',
    'django.contrib.auth.middleware.AuthenticationMiddleware',
    'django.contrib.messages.middleware.MessageMiddleware',
    'django.middleware.clickjacking.XFrameOptionsMiddleware',
]
```

**⚠️ CorsMiddleware debe estar ANTES que CommonMiddleware**

### 3. Configurar orígenes permitidos

#### Opción A: Permitir orígenes específicos (RECOMENDADO)

```python
# Desarrollo
CORS_ALLOWED_ORIGINS = [
    "http://localhost:5173",    # Vite
    "http://localhost:3000",    # React CRA
    "http://127.0.0.1:5173",
]

# Producción
CORS_ALLOWED_ORIGINS = [
    "https://miapp.vercel.app",
    "https://www.midominio.com",
]
```

#### Opción B: Permitir todos (SOLO DESARROLLO)

```python
# ⚠️ NUNCA EN PRODUCCIÓN
CORS_ALLOW_ALL_ORIGINS = True
```

---

## Configuración avanzada

### Permitir credenciales (cookies, headers de autenticación)

```python
CORS_ALLOW_CREDENTIALS = True

CORS_ALLOWED_ORIGINS = [
    "http://localhost:5173",
]
```

**Frontend debe enviar:**
```javascript
axios.get('/api/productos/', {
    withCredentials: true
});
```

### Headers permitidos

```python
CORS_ALLOW_HEADERS = [
    'accept',
    'accept-encoding',
    'authorization',
    'content-type',
    'dnt',
    'origin',
    'user-agent',
    'x-csrftoken',
    'x-requested-with',
]
```

### Métodos permitidos

```python
CORS_ALLOW_METHODS = [
    'DELETE',
    'GET',
    'OPTIONS',
    'PATCH',
    'POST',
    'PUT',
]
```

### Exponer headers personalizados

```python
# Backend envía header personalizado
CORS_EXPOSE_HEADERS = ['X-Total-Count', 'X-Page']
```

```javascript
// Frontend puede leer estos headers
const response = await axios.get('/api/productos/');
const total = response.headers['x-total-count'];
```

---

## Preflight requests

**Preflight = request OPTIONS antes del request real**

```
1. Navegador envía: OPTIONS /api/productos/
2. Backend responde: "Este origen está permitido"
3. Navegador envía: POST /api/productos/
```

**¿Cuándo ocurre preflight?**
- Métodos: PUT, DELETE, PATCH
- Headers personalizados
- Content-Type: application/json

**Django maneja preflight automáticamente con django-cors-headers.**

---

## Configuración por entorno

### settings.py

```python
import os
from decouple import config

DEBUG = config('DEBUG', default=False, cast=bool)

if DEBUG:
    # Desarrollo
    CORS_ALLOWED_ORIGINS = [
        "http://localhost:5173",
        "http://localhost:3000",
    ]
else:
    # Producción
    CORS_ALLOWED_ORIGINS = config('CORS_ALLOWED_ORIGINS', cast=lambda v: [s.strip() for s in v.split(',')])
```

### .env

```env
# Desarrollo
DEBUG=True

# Producción
DEBUG=False
CORS_ALLOWED_ORIGINS=https://miapp.vercel.app,https://www.midominio.com
```

---

## Configurar Axios en React

### Crear instancia base

```javascript
// src/services/api.js
import axios from 'axios';

const API = axios.create({
    baseURL: import.meta.env.VITE_API_URL || 'http://localhost:8000/api',
    headers: {
        'Content-Type': 'application/json',
    },
    withCredentials: true,  // Si usas cookies/sesiones
});

export default API;
```

### Variables de entorno

```env
# .env.development
VITE_API_URL=http://localhost:8000/api

# .env.production
VITE_API_URL=https://api.midominio.com
```

---

## Interceptors en Axios

### Request interceptor (agregar token)

```javascript
// src/services/api.js
import axios from 'axios';

const API = axios.create({
    baseURL: import.meta.env.VITE_API_URL,
});

// Interceptor: agregar token a cada request
API.interceptors.request.use(
    (config) => {
        const token = localStorage.getItem('token');
        if (token) {
            config.headers.Authorization = `Bearer ${token}`;
        }
        return config;
    },
    (error) => {
        return Promise.reject(error);
    }
);

export default API;
```

### Response interceptor (manejar errores)

```javascript
API.interceptors.response.use(
    (response) => {
        return response;
    },
    (error) => {
        if (error.response?.status === 401) {
            // Token expirado
            localStorage.removeItem('token');
            window.location.href = '/login';
        }
        return Promise.reject(error);
    }
);
```

---

## Ejemplo completo

### Backend (Django)

```python
# core/settings.py
from decouple import config

DEBUG = config('DEBUG', default=False, cast=bool)

INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'rest_framework',
    'corsheaders',
    'productos',
]

MIDDLEWARE = [
    'corsheaders.middleware.CorsMiddleware',
    'django.middleware.security.SecurityMiddleware',
    'django.contrib.sessions.middleware.SessionMiddleware',
    'django.middleware.common.CommonMiddleware',
    'django.middleware.csrf.CsrfViewMiddleware',
    'django.contrib.auth.middleware.AuthenticationMiddleware',
    'django.contrib.messages.middleware.MessageMiddleware',
    'django.middleware.clickjacking.XFrameOptionsMiddleware',
]

# CORS
CORS_ALLOWED_ORIGINS = [
    "http://localhost:5173",
    "http://localhost:3000",
]

CORS_ALLOW_CREDENTIALS = True

# REST Framework
REST_FRAMEWORK = {
    'DEFAULT_PERMISSION_CLASSES': [
        'rest_framework.permissions.AllowAny',
    ]
}
```

### Frontend (React)

```javascript
// src/services/api.js
import axios from 'axios';

const API = axios.create({
    baseURL: import.meta.env.VITE_API_URL || 'http://localhost:8000/api',
    headers: {
        'Content-Type': 'application/json',
    },
    withCredentials: true,
});

// Request interceptor: agregar token
API.interceptors.request.use(
    (config) => {
        const token = localStorage.getItem('token');
        if (token) {
            config.headers.Authorization = `Bearer ${token}`;
        }
        return config;
    },
    (error) => Promise.reject(error)
);

// Response interceptor: manejar errores
API.interceptors.response.use(
    (response) => response,
    (error) => {
        if (error.response?.status === 401) {
            localStorage.removeItem('token');
            window.location.href = '/login';
        }
        return Promise.reject(error);
    }
);

export default API;
```

```javascript
// src/services/productoService.js
import API from './api';

export const productoService = {
    getAll: async () => {
        const response = await API.get('/productos/');
        return response.data;
    },
    
    create: async (data) => {
        const response = await API.post('/productos/', data);
        return response.data;
    },
};
```

---

## Errores comunes

### Error 1: CORS policy bloqueado

**Error:**
```
Access to XMLHttpRequest at 'http://localhost:8000/api/productos/'
from origin 'http://localhost:5173' has been blocked by CORS policy
```

**Solución:**
```python
# settings.py
CORS_ALLOWED_ORIGINS = [
    "http://localhost:5173",
]
```

### Error 2: Middleware en orden incorrecto

**Síntoma:** CORS sigue sin funcionar

**Solución:**
```python
MIDDLEWARE = [
    'corsheaders.middleware.CorsMiddleware',  # ← PRIMERO
    # ... resto
]
```

### Error 3: Credenciales no enviadas

**Error:** Cookies no se envían al backend

**Solución:**
```javascript
// Frontend
const API = axios.create({
    withCredentials: true,
});
```

```python
# Backend
CORS_ALLOW_CREDENTIALS = True
```

---

## Testing CORS

### Desde navegador

```javascript
// Consola del navegador
fetch('http://localhost:8000/api/productos/')
    .then(res => res.json())
    .then(data => console.log(data));
```

### Con curl

```bash
curl -H "Origin: http://localhost:5173" \
     -H "Access-Control-Request-Method: GET" \
     -H "Access-Control-Request-Headers: Content-Type" \
     -X OPTIONS \
     http://localhost:8000/api/productos/ \
     --verbose
```

**Debe responder:**
```
< Access-Control-Allow-Origin: http://localhost:5173
< Access-Control-Allow-Credentials: true
```

---

## Configuración para producción

### Backend (Django)

```python
# settings.py
import os
from decouple import config

DEBUG = config('DEBUG', default=False, cast=bool)

if DEBUG:
    # Desarrollo
    CORS_ALLOWED_ORIGINS = [
        "http://localhost:5173",
    ]
else:
    # Producción: leer desde variable de entorno
    allowed_origins = config('CORS_ALLOWED_ORIGINS', default='')
    CORS_ALLOWED_ORIGINS = [origin.strip() for origin in allowed_origins.split(',')]

CORS_ALLOW_CREDENTIALS = True

# Seguridad adicional
SECURE_SSL_REDIRECT = not DEBUG
SESSION_COOKIE_SECURE = not DEBUG
CSRF_COOKIE_SECURE = not DEBUG
```

### Variables de entorno (producción)

```env
DEBUG=False
CORS_ALLOWED_ORIGINS=https://miapp.vercel.app,https://www.midominio.com
SECRET_KEY=tu-secret-key-super-segura
```

### Frontend (React)

```javascript
// vite.config.js
export default defineConfig({
    server: {
        proxy: {
            // Solo en desarrollo
            '/api': {
                target: 'http://localhost:8000',
                changeOrigin: true,
            }
        }
    }
});
```

```env
# .env.production
VITE_API_URL=https://api.midominio.com
```

---

## Resumen

**CORS permite:**
- Frontend (http://localhost:5173) → Backend (http://localhost:8000)
- Diferentes dominios en producción

**Configuración mínima:**
```python
# settings.py
INSTALLED_APPS = ['corsheaders', ...]
MIDDLEWARE = ['corsheaders.middleware.CorsMiddleware', ...]
CORS_ALLOWED_ORIGINS = ["http://localhost:5173"]
```

**Axios con interceptors:**
- Request: agregar token automáticamente
- Response: manejar errores 401, 403, 500

**Próximo módulo:** Autenticación JWT (login, registro, tokens)

---

## Recursos

- **[django-cors-headers](https://github.com/adamchainz/django-cors-headers)** - Documentación oficial
- **[MDN CORS](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS)** - Guía completa
- **[Axios Interceptors](https://axios-http.com/docs/interceptors)** - Documentación

¡CORS configurado correctamente! 🌐

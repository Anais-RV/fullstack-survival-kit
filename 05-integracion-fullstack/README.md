# Integración Fullstack

> **Conectar React + Django + PostgreSQL en una aplicación completa**

---

## Contenido del módulo

### 01. [Arquitectura Frontend + Backend](01-arquitectura-frontend-backend.md)
- Separación de responsabilidades
- Comunicación REST (HTTP + JSON)
- Estructura de proyecto (frontend/ + backend/)
- Endpoints CRUD básicos
- Configuración de CORS básica

### 02. [CORS y Configuración](02-cors-configuracion.md)
- ¿Qué es CORS y por qué existe?
- Same-Origin Policy
- django-cors-headers
- Configuración por entorno (desarrollo vs producción)
- Axios con baseURL y variables de entorno
- Interceptors básicos

### 03. [Autenticación JWT](03-autenticacion-jwt.md)
- ¿Qué es JWT? (Header, Payload, Signature)
- Access Token vs Refresh Token
- djangorestframework-simplejwt
- Endpoints: login, register, profile, token/refresh
- Context API para autenticación (AuthContext)
- Rutas protegidas en React
- Auto-refresh de tokens con interceptors

### 04. [Axios y API Client](04-axios-api-client.md)
- Estructura de servicios (api.js, productoService.js, authService.js)
- Axios con configuración base
- Request interceptor (agregar token automáticamente)
- Response interceptor (manejar errores, refresh)
- Hooks personalizados (useFetch, useProductos)
- Cancelar requests con AbortController
- Progreso de uploads
- Retry automático

### 05. [Estado Global](05-estado-global.md)
- ¿Cuándo usar estado global?
- Context API (AuthProvider, CartProvider)
- Zustand (store más simple y potente)
- Comparación Context vs Zustand
- Stores: auth, cart, notifications, settings
- Middleware: persist, immer, devtools

### 06. [CRUD Completo](06-crud-completo.md)
- Backend: ViewSet completo (list, create, retrieve, update, destroy)
- Serializers con validaciones
- Filtros, búsqueda, ordenamiento (django-filter)
- Frontend: Lista con filtros
- Formulario crear/editar (reutilizable con useParams)
- Vista de detalle
- Confirmaciones antes de eliminar
- Manejo de estados (loading, error)

### 07. [Upload de Archivos](07-upload-archivos.md)
- Archivos locales (MEDIA_ROOT, MEDIA_URL)
- ImageField en Django
- FormData en React
- Cloudinary (almacenamiento en la nube)
- Upload directo desde React a Cloudinary
- Drag & Drop
- Preview de imágenes
- Múltiples imágenes
- Barra de progreso
- Validaciones (tamaño, tipo)

### 08. [Deployment Conjunto](08-deployment-conjunto.md)
- Preparar Django para producción (DEBUG, SECRET_KEY, ALLOWED_HOSTS)
- PostgreSQL en Railway
- Gunicorn + WhiteNoise
- Variables de entorno (.env)
- Deploy backend en Railway
- Deploy frontend en Vercel
- Configurar CORS para producción
- Custom domains
- CI/CD automático
- Monitoreo y logs

---

## Objetivo del módulo

Al completar este módulo serás capaz de:

✅ **Conectar** React (frontend) con Django REST Framework (backend)  
✅ **Autenticar** usuarios con JWT (login, registro, refresh tokens)  
✅ **Gestionar** estado global (autenticación, carrito de compras)  
✅ **Implementar** CRUD completo (crear, leer, actualizar, eliminar)  
✅ **Subir** archivos e imágenes (FormData, Cloudinary)  
✅ **Desplegar** aplicación fullstack en producción (Vercel + Railway)

---

## Flujo completo de una aplicación fullstack

```
Usuario → React (Frontend)
           ↓
      Axios Request (JSON)
           ↓
      Django REST API (Backend)
           ↓
      PostgreSQL (Base de datos)
           ↓
      Response (JSON)
           ↓
      React actualiza UI
```

---

## Stack tecnológico

**Frontend:**
- React 18
- Vite
- React Router
- Axios
- Zustand (estado global)

**Backend:**
- Django 5.0
- Django REST Framework
- djangorestframework-simplejwt (JWT)
- django-cors-headers (CORS)
- Gunicorn (servidor WSGI)

**Base de datos:**
- PostgreSQL 15

**Deployment:**
- Vercel (frontend)
- Railway (backend + PostgreSQL)
- Cloudinary (archivos)

---

## Arquitectura final

```
┌─────────────────────────────────────┐
│   FRONTEND (Vercel)                 │
│   React + React Router + Zustand    │
│   https://miapp.vercel.app          │
└──────────────┬──────────────────────┘
               │
               │ HTTPS + JWT
               ↓
┌─────────────────────────────────────┐
│   BACKEND (Railway)                 │
│   Django REST + Gunicorn            │
│   https://api.railway.app           │
└──────────────┬──────────────────────┘
               │
               ↓
         ┌─────────┐          ┌───────────┐
         │PostgreSQL│          │Cloudinary│
         │(Railway) │          │(Imágenes) │
         └─────────┘          └───────────┘
```

---

## Proyecto de ejemplo: E-commerce

A lo largo del módulo construiremos un e-commerce completo:

**Funcionalidades:**
- 🔐 Registro y login con JWT
- 📦 CRUD de productos (crear, editar, eliminar)
- 🛒 Carrito de compras (Zustand)
- 🖼️ Upload de imágenes (Cloudinary)
- 🔍 Búsqueda y filtros
- 📱 Responsive design
- 🚀 Deployment en producción

**Endpoints API:**
```
POST   /api/usuarios/register/         # Registro
POST   /api/usuarios/login/            # Login
POST   /api/usuarios/token/refresh/    # Refresh token
GET    /api/usuarios/profile/          # Perfil

GET    /api/productos/                 # Lista de productos
POST   /api/productos/                 # Crear producto
GET    /api/productos/{id}/            # Detalle
PUT    /api/productos/{id}/            # Actualizar
DELETE /api/productos/{id}/            # Eliminar

GET    /api/categorias/                # Lista de categorías
```

---

## Próximos pasos

Después de completar este módulo, continúa con:

- **Módulo 06:** Testing (Jest, React Testing Library, Pytest)
- **Módulo 07:** Arquitectura (Clean Architecture, SOLID, Design Patterns)
- **Módulo 08:** Despliegue Avanzado (Docker, CI/CD, Monitoring)
- **Módulo 09:** Proyecto Integrador (E-commerce completo con mejores prácticas)

---

## Recursos adicionales

- **[Django REST Framework](https://www.django-rest-framework.org/)** - Documentación oficial
- **[React Router](https://reactrouter.com/)** - Documentación
- **[Zustand](https://zustand-demo.pmnd.rs/)** - Estado global
- **[Railway](https://railway.app/)** - Deploy backend
- **[Vercel](https://vercel.com/)** - Deploy frontend

¡Empecemos a integrar! 🚀

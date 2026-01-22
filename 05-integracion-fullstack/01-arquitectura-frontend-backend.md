# Arquitectura Frontend + Backend

> **Conectar React con Django REST Framework**

---

## Arquitectura de aplicación fullstack

**Frontend (React)** + **Backend (Django REST)**

```
┌─────────────────────────────────────┐
│   FRONTEND (React)                  │
│   - UI/UX                           │
│   - Estado local                    │
│   - Comunicación HTTP               │
│   Puerto: 5173 (Vite)               │
└─────────────┬───────────────────────┘
              │
              │ HTTP Requests (JSON)
              │ GET, POST, PUT, DELETE
              ↓
┌─────────────────────────────────────┐
│   BACKEND (Django REST)             │
│   - Lógica de negocio               │
│   - Acceso a base de datos          │
│   - Autenticación                   │
│   Puerto: 8000                      │
└─────────────┬───────────────────────┘
              │
              ↓
         ┌─────────┐
         │PostgreSQL│
         └─────────┘
```

---

## Separación de responsabilidades

### Frontend (React)
- ✅ Interfaz de usuario
- ✅ Validación de formularios (client-side)
- ✅ Navegación (React Router)
- ✅ Estado de UI
- ✅ Llamadas HTTP a la API

### Backend (Django)
- ✅ Lógica de negocio
- ✅ Validación de datos (server-side)
- ✅ Acceso a base de datos
- ✅ Autenticación y permisos
- ✅ API REST endpoints

---

## Comunicación: REST API

**Protocolo:** HTTP/HTTPS  
**Formato:** JSON

**Métodos HTTP:**
- `GET` - Obtener datos
- `POST` - Crear datos
- `PUT/PATCH` - Actualizar datos
- `DELETE` - Eliminar datos

**Ejemplo:**

```javascript
// FRONTEND: React hace request
fetch('http://localhost:8000/api/productos/')
  .then(res => res.json())
  .then(data => console.log(data));

// BACKEND: Django responde
[
  {"id": 1, "nombre": "Laptop", "precio": 1200},
  {"id": 2, "nombre": "Mouse", "precio": 25}
]
```

---

## Estructura de proyecto

```
proyecto-fullstack/
├── frontend/               # React
│   ├── src/
│   │   ├── components/
│   │   ├── pages/
│   │   ├── services/      # API calls
│   │   ├── context/       # Estado global
│   │   └── App.jsx
│   ├── package.json
│   └── vite.config.js
│
└── backend/                # Django
    ├── core/              # Settings
    ├── apps/
    │   └── productos/
    │       ├── models.py
    │       ├── serializers.py
    │       ├── views.py
    │       └── urls.py
    ├── manage.py
    └── requirements.txt
```

---

## Crear backend Django REST

### 1. Setup inicial

```bash
# Crear carpeta backend
mkdir backend
cd backend

# Virtual environment
python -m venv venv
source venv/bin/activate  # Linux/Mac
venv\Scripts\activate     # Windows

# Instalar dependencias
pip install django djangorestframework django-cors-headers psycopg2-binary python-decouple

# Crear proyecto
django-admin startproject core .

# Crear app
python manage.py startapp productos
```

### 2. Configurar settings.py

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
    'corsheaders',
    
    # Apps
    'productos',
]

MIDDLEWARE = [
    'corsheaders.middleware.CorsMiddleware',  # ← Primero
    'django.middleware.security.SecurityMiddleware',
    'django.contrib.sessions.middleware.SessionMiddleware',
    'django.middleware.common.CommonMiddleware',
    'django.middleware.csrf.CsrfViewMiddleware',
    'django.contrib.auth.middleware.AuthenticationMiddleware',
    'django.contrib.messages.middleware.MessageMiddleware',
    'django.middleware.clickjacking.XFrameOptionsMiddleware',
]

# CORS (permitir frontend)
CORS_ALLOWED_ORIGINS = [
    "http://localhost:5173",  # Vite
    "http://localhost:3000",  # React CRA
]

# REST Framework
REST_FRAMEWORK = {
    'DEFAULT_PERMISSION_CLASSES': [
        'rest_framework.permissions.AllowAny',
    ],
    'DEFAULT_PAGINATION_CLASS': 'rest_framework.pagination.PageNumberPagination',
    'PAGE_SIZE': 10
}
```

### 3. Modelo

```python
# productos/models.py
from django.db import models

class Producto(models.Model):
    nombre = models.CharField(max_length=200)
    descripcion = models.TextField(blank=True)
    precio = models.DecimalField(max_digits=10, decimal_places=2)
    stock = models.IntegerField(default=0)
    activo = models.BooleanField(default=True)
    creado_en = models.DateTimeField(auto_now_add=True)
    
    def __str__(self):
        return self.nombre
    
    class Meta:
        ordering = ['-creado_en']
```

### 4. Serializer

```python
# productos/serializers.py
from rest_framework import serializers
from .models import Producto

class ProductoSerializer(serializers.ModelSerializer):
    class Meta:
        model = Producto
        fields = ['id', 'nombre', 'descripcion', 'precio', 'stock', 'activo', 'creado_en']
        read_only_fields = ['id', 'creado_en']
```

### 5. Viewset

```python
# productos/views.py
from rest_framework import viewsets
from .models import Producto
from .serializers import ProductoSerializer

class ProductoViewSet(viewsets.ModelViewSet):
    queryset = Producto.objects.all()
    serializer_class = ProductoSerializer
```

### 6. URLs

```python
# productos/urls.py
from django.urls import path, include
from rest_framework.routers import DefaultRouter
from .views import ProductoViewSet

router = DefaultRouter()
router.register(r'productos', ProductoViewSet)

urlpatterns = [
    path('', include(router.urls)),
]
```

```python
# core/urls.py
from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path('admin/', admin.site.urls),
    path('api/', include('productos.urls')),
]
```

### 7. Migrar y ejecutar

```bash
python manage.py makemigrations
python manage.py migrate
python manage.py createsuperuser
python manage.py runserver
```

**API disponible en:** http://localhost:8000/api/productos/

---

## Crear frontend React

### 1. Crear proyecto Vite

```bash
# Carpeta frontend
npm create vite@latest frontend -- --template react
cd frontend
npm install
npm install axios react-router-dom
```

### 2. Configurar Axios

```javascript
// src/services/api.js
import axios from 'axios';

const API = axios.create({
    baseURL: 'http://localhost:8000/api',
    headers: {
        'Content-Type': 'application/json',
    }
});

export default API;
```

### 3. Servicio de productos

```javascript
// src/services/productoService.js
import API from './api';

export const productoService = {
    // Listar todos
    getAll: async () => {
        const response = await API.get('/productos/');
        return response.data;
    },
    
    // Obtener uno
    getById: async (id) => {
        const response = await API.get(`/productos/${id}/`);
        return response.data;
    },
    
    // Crear
    create: async (data) => {
        const response = await API.post('/productos/', data);
        return response.data;
    },
    
    // Actualizar
    update: async (id, data) => {
        const response = await API.put(`/productos/${id}/`, data);
        return response.data;
    },
    
    // Eliminar
    delete: async (id) => {
        await API.delete(`/productos/${id}/`);
    }
};
```

### 4. Componente de lista

```jsx
// src/components/ProductosList.jsx
import { useState, useEffect } from 'react';
import { productoService } from '../services/productoService';

export default function ProductosList() {
    const [productos, setProductos] = useState([]);
    const [loading, setLoading] = useState(true);
    const [error, setError] = useState(null);
    
    useEffect(() => {
        cargarProductos();
    }, []);
    
    const cargarProductos = async () => {
        try {
            setLoading(true);
            const data = await productoService.getAll();
            setProductos(data);
        } catch (err) {
            setError('Error al cargar productos');
            console.error(err);
        } finally {
            setLoading(false);
        }
    };
    
    const eliminar = async (id) => {
        if (!confirm('¿Eliminar producto?')) return;
        
        try {
            await productoService.delete(id);
            setProductos(productos.filter(p => p.id !== id));
        } catch (err) {
            alert('Error al eliminar');
        }
    };
    
    if (loading) return <div>Cargando...</div>;
    if (error) return <div>Error: {error}</div>;
    
    return (
        <div>
            <h1>Productos</h1>
            
            <table>
                <thead>
                    <tr>
                        <th>ID</th>
                        <th>Nombre</th>
                        <th>Precio</th>
                        <th>Stock</th>
                        <th>Acciones</th>
                    </tr>
                </thead>
                <tbody>
                    {productos.map(producto => (
                        <tr key={producto.id}>
                            <td>{producto.id}</td>
                            <td>{producto.nombre}</td>
                            <td>${producto.precio}</td>
                            <td>{producto.stock}</td>
                            <td>
                                <button onClick={() => eliminar(producto.id)}>
                                    Eliminar
                                </button>
                            </td>
                        </tr>
                    ))}
                </tbody>
            </table>
        </div>
    );
}
```

### 5. App.jsx

```jsx
// src/App.jsx
import ProductosList from './components/ProductosList';

function App() {
    return (
        <div className="App">
            <ProductosList />
        </div>
    );
}

export default App;
```

### 6. Ejecutar

```bash
npm run dev
```

**Frontend en:** http://localhost:5173

---

## Probar integración

### 1. Agregar productos desde Django Admin

```bash
# Backend
python manage.py createsuperuser
# http://localhost:8000/admin
```

### 2. Ver en React

```
http://localhost:5173
```

**Flujo:**
1. React hace `GET /api/productos/`
2. Django responde con JSON
3. React muestra en UI

---

## Estructura de respuestas API

### Lista de productos

**Request:**
```
GET /api/productos/
```

**Response:**
```json
{
    "count": 2,
    "next": null,
    "previous": null,
    "results": [
        {
            "id": 1,
            "nombre": "Laptop Dell",
            "descripcion": "Laptop profesional",
            "precio": "1299.99",
            "stock": 10,
            "activo": true,
            "creado_en": "2024-01-20T10:30:00Z"
        },
        {
            "id": 2,
            "nombre": "Mouse Logitech",
            "descripcion": "Mouse inalámbrico",
            "precio": "25.99",
            "stock": 50,
            "activo": true,
            "creado_en": "2024-01-20T11:00:00Z"
        }
    ]
}
```

### Crear producto

**Request:**
```
POST /api/productos/
Content-Type: application/json

{
    "nombre": "Teclado Mecánico",
    "descripcion": "Teclado RGB",
    "precio": 89.99,
    "stock": 30
}
```

**Response:**
```json
{
    "id": 3,
    "nombre": "Teclado Mecánico",
    "descripcion": "Teclado RGB",
    "precio": "89.99",
    "stock": 30,
    "activo": true,
    "creado_en": "2024-01-20T12:00:00Z"
}
```

---

## Manejo de errores

### Backend (Django)

```python
# productos/views.py
from rest_framework import viewsets, status
from rest_framework.response import Response

class ProductoViewSet(viewsets.ModelViewSet):
    queryset = Producto.objects.all()
    serializer_class = ProductoSerializer
    
    def create(self, request):
        serializer = self.get_serializer(data=request.data)
        
        if serializer.is_valid():
            serializer.save()
            return Response(serializer.data, status=status.HTTP_201_CREATED)
        
        return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)
```

### Frontend (React)

```javascript
// src/services/productoService.js
export const productoService = {
    create: async (data) => {
        try {
            const response = await API.post('/productos/', data);
            return { success: true, data: response.data };
        } catch (error) {
            return {
                success: false,
                error: error.response?.data || 'Error desconocido'
            };
        }
    }
};
```

```jsx
// Componente
const crear = async (formData) => {
    const result = await productoService.create(formData);
    
    if (result.success) {
        alert('Producto creado');
        cargarProductos();
    } else {
        alert(`Error: ${JSON.stringify(result.error)}`);
    }
};
```

---

## Mejores prácticas

✅ **Separar lógica en servicios**
```javascript
// ✅ BIEN
await productoService.getAll();

// ❌ MAL
await axios.get('http://localhost:8000/api/productos/');
```

✅ **Manejar estados de carga**
```jsx
const [loading, setLoading] = useState(true);
const [error, setError] = useState(null);
```

✅ **Validar en backend siempre**
```python
# Django valida aunque React valide
class ProductoSerializer(serializers.ModelSerializer):
    def validate_precio(self, value):
        if value <= 0:
            raise serializers.ValidationError("Precio debe ser positivo")
        return value
```

✅ **Variables de entorno**
```javascript
// .env
VITE_API_URL=http://localhost:8000/api

// src/services/api.js
const API = axios.create({
    baseURL: import.meta.env.VITE_API_URL
});
```

---

## Resumen

**Arquitectura:**
- Frontend (React) → HTTP → Backend (Django) → PostgreSQL

**Backend endpoints:**
- `GET /api/productos/` - Lista
- `POST /api/productos/` - Crear
- `GET /api/productos/{id}/` - Detalle
- `PUT /api/productos/{id}/` - Actualizar
- `DELETE /api/productos/{id}/` - Eliminar

**Frontend servicios:**
- Axios para HTTP requests
- Servicios reutilizables
- Manejo de estados (loading, error)

**Próximo módulo:** CORS y configuración avanzada

---

## Recursos

- **[Django REST Framework](https://www.django-rest-framework.org/)** - Documentación oficial
- **[Axios](https://axios-http.com/)** - Cliente HTTP
- **[Vite](https://vitejs.dev/)** - Build tool

¡Frontend y Backend conectados! 🔗

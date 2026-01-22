# Autenticación JWT

> **JSON Web Tokens: Sistema de autenticación moderno para APIs REST**

---

## ¿Qué es JWT?

**JWT (JSON Web Token)** es un estándar para crear tokens de acceso que permiten autenticar usuarios entre frontend y backend.

**Ventajas:**
- ✅ Stateless (sin sesiones en servidor)
- ✅ Funciona entre dominios
- ✅ Escalable
- ✅ Perfecto para SPAs (React, Vue, Angular)

---

## Estructura de JWT

Un token JWT tiene **3 partes** separadas por puntos:

```
eyJhbGci.eyJ1c2VyX2lk.SflKxwRJ
 ─┬──     ─┬──          ─┬──
Header   Payload      Signature
```

### 1. Header
```json
{
  "alg": "HS256",
  "typ": "JWT"
}
```

### 2. Payload
```json
{
  "user_id": 42,
  "username": "juan",
  "exp": 1706284800
}
```

### 3. Signature
```
HMACSHA256(
  base64UrlEncode(header) + "." +
  base64UrlEncode(payload),
  SECRET_KEY
)
```

**El token NO se puede modificar sin conocer la SECRET_KEY.**

---

## Access Token vs Refresh Token

**Access Token:**
- Vida corta (5-15 minutos)
- Se envía en cada request
- Accede a recursos protegidos

**Refresh Token:**
- Vida larga (7 días, 30 días)
- Se usa para obtener nuevo access token
- Más seguro (no se envía en cada request)

```
Login → Access Token (15 min) + Refresh Token (7 días)
       ↓
   Access expira → Usar Refresh → Nuevo Access Token
```

---

## Instalar djangorestframework-simplejwt

```bash
pip install djangorestframework-simplejwt
```

---

## Configurar Django

### 1. settings.py

```python
# core/settings.py
from datetime import timedelta

INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    
    # Third party
    'rest_framework',
    'rest_framework_simplejwt',  # ← Agregar
    'corsheaders',
    
    # Apps
    'productos',
    'usuarios',  # Nueva app para auth
]

# REST Framework con JWT
REST_FRAMEWORK = {
    'DEFAULT_AUTHENTICATION_CLASSES': [
        'rest_framework_simplejwt.authentication.JWTAuthentication',
    ],
    'DEFAULT_PERMISSION_CLASSES': [
        'rest_framework.permissions.IsAuthenticated',  # Por defecto: autenticado
    ]
}

# Simple JWT
SIMPLE_JWT = {
    'ACCESS_TOKEN_LIFETIME': timedelta(minutes=15),
    'REFRESH_TOKEN_LIFETIME': timedelta(days=7),
    'ROTATE_REFRESH_TOKENS': True,
    'BLACKLIST_AFTER_ROTATION': True,
    'AUTH_HEADER_TYPES': ('Bearer',),
}
```

### 2. Crear app usuarios

```bash
python manage.py startapp usuarios
```

### 3. Modelo de usuario personalizado (opcional)

```python
# usuarios/models.py
from django.contrib.auth.models import AbstractUser

class Usuario(AbstractUser):
    telefono = models.CharField(max_length=20, blank=True)
    direccion = models.TextField(blank=True)
    
    def __str__(self):
        return self.username
```

```python
# core/settings.py
AUTH_USER_MODEL = 'usuarios.Usuario'
```

### 4. Serializer de registro

```python
# usuarios/serializers.py
from rest_framework import serializers
from django.contrib.auth import get_user_model

User = get_user_model()

class RegisterSerializer(serializers.ModelSerializer):
    password = serializers.CharField(write_only=True, min_length=8)
    password2 = serializers.CharField(write_only=True)
    
    class Meta:
        model = User
        fields = ['username', 'email', 'password', 'password2', 'first_name', 'last_name']
    
    def validate(self, data):
        if data['password'] != data['password2']:
            raise serializers.ValidationError("Las contraseñas no coinciden")
        return data
    
    def create(self, validated_data):
        validated_data.pop('password2')
        user = User.objects.create_user(
            username=validated_data['username'],
            email=validated_data['email'],
            password=validated_data['password'],
            first_name=validated_data.get('first_name', ''),
            last_name=validated_data.get('last_name', '')
        )
        return user

class UserSerializer(serializers.ModelSerializer):
    class Meta:
        model = User
        fields = ['id', 'username', 'email', 'first_name', 'last_name']
```

### 5. Views

```python
# usuarios/views.py
from rest_framework import generics, status
from rest_framework.response import Response
from rest_framework.permissions import AllowAny, IsAuthenticated
from rest_framework_simplejwt.tokens import RefreshToken
from django.contrib.auth import get_user_model
from .serializers import RegisterSerializer, UserSerializer

User = get_user_model()

class RegisterView(generics.CreateAPIView):
    queryset = User.objects.all()
    serializer_class = RegisterSerializer
    permission_classes = [AllowAny]
    
    def create(self, request, *args, **kwargs):
        serializer = self.get_serializer(data=request.data)
        serializer.is_valid(raise_exception=True)
        user = serializer.save()
        
        # Generar tokens
        refresh = RefreshToken.for_user(user)
        
        return Response({
            'user': UserSerializer(user).data,
            'tokens': {
                'refresh': str(refresh),
                'access': str(refresh.access_token),
            }
        }, status=status.HTTP_201_CREATED)

class ProfileView(generics.RetrieveAPIView):
    serializer_class = UserSerializer
    permission_classes = [IsAuthenticated]
    
    def get_object(self):
        return self.request.user
```

### 6. URLs

```python
# usuarios/urls.py
from django.urls import path
from rest_framework_simplejwt.views import TokenObtainPairView, TokenRefreshView
from .views import RegisterView, ProfileView

urlpatterns = [
    # Registro
    path('register/', RegisterView.as_view(), name='register'),
    
    # Login (obtener tokens)
    path('login/', TokenObtainPairView.as_view(), name='token_obtain_pair'),
    
    # Refresh token
    path('token/refresh/', TokenRefreshView.as_view(), name='token_refresh'),
    
    # Perfil
    path('profile/', ProfileView.as_view(), name='profile'),
]
```

```python
# core/urls.py
from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path('admin/', admin.site.urls),
    path('api/usuarios/', include('usuarios.urls')),
    path('api/', include('productos.urls')),
]
```

### 7. Migrar

```bash
python manage.py makemigrations
python manage.py migrate
```

---

## Probar endpoints con curl

### Registro

```bash
curl -X POST http://localhost:8000/api/usuarios/register/ \
  -H "Content-Type: application/json" \
  -d '{
    "username": "juan",
    "email": "juan@example.com",
    "password": "MiPassword123",
    "password2": "MiPassword123",
    "first_name": "Juan",
    "last_name": "Pérez"
  }'
```

**Respuesta:**
```json
{
  "user": {
    "id": 1,
    "username": "juan",
    "email": "juan@example.com",
    "first_name": "Juan",
    "last_name": "Pérez"
  },
  "tokens": {
    "refresh": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
    "access": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
  }
}
```

### Login

```bash
curl -X POST http://localhost:8000/api/usuarios/login/ \
  -H "Content-Type: application/json" \
  -d '{
    "username": "juan",
    "password": "MiPassword123"
  }'
```

**Respuesta:**
```json
{
  "refresh": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "access": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
}
```

### Acceder a perfil (con token)

```bash
curl http://localhost:8000/api/usuarios/profile/ \
  -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
```

**Respuesta:**
```json
{
  "id": 1,
  "username": "juan",
  "email": "juan@example.com",
  "first_name": "Juan",
  "last_name": "Pérez"
}
```

### Refresh token

```bash
curl -X POST http://localhost:8000/api/usuarios/token/refresh/ \
  -H "Content-Type: application/json" \
  -d '{
    "refresh": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
  }'
```

**Respuesta:**
```json
{
  "access": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "refresh": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
}
```

---

## Frontend: React con JWT

### 1. Context de autenticación

```jsx
// src/context/AuthContext.jsx
import { createContext, useContext, useState, useEffect } from 'react';
import { authService } from '../services/authService';

const AuthContext = createContext();

export const useAuth = () => {
    const context = useContext(AuthContext);
    if (!context) {
        throw new Error('useAuth debe usarse dentro de AuthProvider');
    }
    return context;
};

export const AuthProvider = ({ children }) => {
    const [user, setUser] = useState(null);
    const [loading, setLoading] = useState(true);
    
    useEffect(() => {
        checkAuth();
    }, []);
    
    const checkAuth = async () => {
        const token = localStorage.getItem('access_token');
        if (token) {
            try {
                const userData = await authService.getProfile();
                setUser(userData);
            } catch (error) {
                localStorage.removeItem('access_token');
                localStorage.removeItem('refresh_token');
            }
        }
        setLoading(false);
    };
    
    const login = async (username, password) => {
        const data = await authService.login(username, password);
        localStorage.setItem('access_token', data.access);
        localStorage.setItem('refresh_token', data.refresh);
        const userData = await authService.getProfile();
        setUser(userData);
    };
    
    const register = async (formData) => {
        const data = await authService.register(formData);
        localStorage.setItem('access_token', data.tokens.access);
        localStorage.setItem('refresh_token', data.tokens.refresh);
        setUser(data.user);
    };
    
    const logout = () => {
        localStorage.removeItem('access_token');
        localStorage.removeItem('refresh_token');
        setUser(null);
    };
    
    return (
        <AuthContext.Provider value={{ user, login, register, logout, loading }}>
            {children}
        </AuthContext.Provider>
    );
};
```

### 2. Servicio de autenticación

```javascript
// src/services/authService.js
import API from './api';

export const authService = {
    // Registro
    register: async (data) => {
        const response = await API.post('/usuarios/register/', data);
        return response.data;
    },
    
    // Login
    login: async (username, password) => {
        const response = await API.post('/usuarios/login/', { username, password });
        return response.data;
    },
    
    // Perfil
    getProfile: async () => {
        const response = await API.get('/usuarios/profile/');
        return response.data;
    },
    
    // Refresh token
    refreshToken: async (refresh) => {
        const response = await API.post('/usuarios/token/refresh/', { refresh });
        return response.data;
    },
};
```

### 3. Axios con interceptors (auto-refresh)

```javascript
// src/services/api.js
import axios from 'axios';
import { authService } from './authService';

const API = axios.create({
    baseURL: import.meta.env.VITE_API_URL || 'http://localhost:8000/api',
});

// Request interceptor: agregar token
API.interceptors.request.use(
    (config) => {
        const token = localStorage.getItem('access_token');
        if (token) {
            config.headers.Authorization = `Bearer ${token}`;
        }
        return config;
    },
    (error) => Promise.reject(error)
);

// Response interceptor: refresh token automático
API.interceptors.response.use(
    (response) => response,
    async (error) => {
        const originalRequest = error.config;
        
        // Token expirado
        if (error.response?.status === 401 && !originalRequest._retry) {
            originalRequest._retry = true;
            
            try {
                const refresh = localStorage.getItem('refresh_token');
                const data = await authService.refreshToken(refresh);
                
                localStorage.setItem('access_token', data.access);
                if (data.refresh) {
                    localStorage.setItem('refresh_token', data.refresh);
                }
                
                originalRequest.headers.Authorization = `Bearer ${data.access}`;
                return API(originalRequest);
            } catch (err) {
                localStorage.removeItem('access_token');
                localStorage.removeItem('refresh_token');
                window.location.href = '/login';
                return Promise.reject(err);
            }
        }
        
        return Promise.reject(error);
    }
);

export default API;
```

### 4. Componente de Login

```jsx
// src/components/Login.jsx
import { useState } from 'react';
import { useAuth } from '../context/AuthContext';
import { useNavigate } from 'react-router-dom';

export default function Login() {
    const [username, setUsername] = useState('');
    const [password, setPassword] = useState('');
    const [error, setError] = useState('');
    const { login } = useAuth();
    const navigate = useNavigate();
    
    const handleSubmit = async (e) => {
        e.preventDefault();
        setError('');
        
        try {
            await login(username, password);
            navigate('/');
        } catch (err) {
            setError('Usuario o contraseña incorrectos');
        }
    };
    
    return (
        <div>
            <h2>Iniciar Sesión</h2>
            
            {error && <div style={{ color: 'red' }}>{error}</div>}
            
            <form onSubmit={handleSubmit}>
                <div>
                    <label>Usuario:</label>
                    <input
                        type="text"
                        value={username}
                        onChange={(e) => setUsername(e.target.value)}
                        required
                    />
                </div>
                
                <div>
                    <label>Contraseña:</label>
                    <input
                        type="password"
                        value={password}
                        onChange={(e) => setPassword(e.target.value)}
                        required
                    />
                </div>
                
                <button type="submit">Entrar</button>
            </form>
        </div>
    );
}
```

### 5. Componente de Registro

```jsx
// src/components/Register.jsx
import { useState } from 'react';
import { useAuth } from '../context/AuthContext';
import { useNavigate } from 'react-router-dom';

export default function Register() {
    const [formData, setFormData] = useState({
        username: '',
        email: '',
        password: '',
        password2: '',
        first_name: '',
        last_name: '',
    });
    const [error, setError] = useState('');
    const { register } = useAuth();
    const navigate = useNavigate();
    
    const handleChange = (e) => {
        setFormData({
            ...formData,
            [e.target.name]: e.target.value
        });
    };
    
    const handleSubmit = async (e) => {
        e.preventDefault();
        setError('');
        
        if (formData.password !== formData.password2) {
            setError('Las contraseñas no coinciden');
            return;
        }
        
        try {
            await register(formData);
            navigate('/');
        } catch (err) {
            setError(err.response?.data?.username?.[0] || 'Error al registrar');
        }
    };
    
    return (
        <div>
            <h2>Registro</h2>
            
            {error && <div style={{ color: 'red' }}>{error}</div>}
            
            <form onSubmit={handleSubmit}>
                <div>
                    <label>Usuario:</label>
                    <input
                        type="text"
                        name="username"
                        value={formData.username}
                        onChange={handleChange}
                        required
                    />
                </div>
                
                <div>
                    <label>Email:</label>
                    <input
                        type="email"
                        name="email"
                        value={formData.email}
                        onChange={handleChange}
                        required
                    />
                </div>
                
                <div>
                    <label>Contraseña:</label>
                    <input
                        type="password"
                        name="password"
                        value={formData.password}
                        onChange={handleChange}
                        required
                    />
                </div>
                
                <div>
                    <label>Confirmar contraseña:</label>
                    <input
                        type="password"
                        name="password2"
                        value={formData.password2}
                        onChange={handleChange}
                        required
                    />
                </div>
                
                <div>
                    <label>Nombre:</label>
                    <input
                        type="text"
                        name="first_name"
                        value={formData.first_name}
                        onChange={handleChange}
                    />
                </div>
                
                <div>
                    <label>Apellido:</label>
                    <input
                        type="text"
                        name="last_name"
                        value={formData.last_name}
                        onChange={handleChange}
                    />
                </div>
                
                <button type="submit">Registrarse</button>
            </form>
        </div>
    );
}
```

### 6. Rutas protegidas

```jsx
// src/components/ProtectedRoute.jsx
import { Navigate } from 'react-router-dom';
import { useAuth } from '../context/AuthContext';

export default function ProtectedRoute({ children }) {
    const { user, loading } = useAuth();
    
    if (loading) {
        return <div>Cargando...</div>;
    }
    
    if (!user) {
        return <Navigate to="/login" />;
    }
    
    return children;
}
```

### 7. App.jsx

```jsx
// src/App.jsx
import { BrowserRouter, Routes, Route, Link } from 'react-router-dom';
import { AuthProvider, useAuth } from './context/AuthContext';
import Login from './components/Login';
import Register from './components/Register';
import ProductosList from './components/ProductosList';
import ProtectedRoute from './components/ProtectedRoute';

function Navigation() {
    const { user, logout } = useAuth();
    
    return (
        <nav>
            <Link to="/">Inicio</Link>
            {user ? (
                <>
                    <span>Hola, {user.username}</span>
                    <button onClick={logout}>Cerrar Sesión</button>
                </>
            ) : (
                <>
                    <Link to="/login">Login</Link>
                    <Link to="/register">Registro</Link>
                </>
            )}
        </nav>
    );
}

function App() {
    return (
        <BrowserRouter>
            <AuthProvider>
                <Navigation />
                <Routes>
                    <Route
                        path="/"
                        element={
                            <ProtectedRoute>
                                <ProductosList />
                            </ProtectedRoute>
                        }
                    />
                    <Route path="/login" element={<Login />} />
                    <Route path="/register" element={<Register />} />
                </Routes>
            </AuthProvider>
        </BrowserRouter>
    );
}

export default App;
```

---

## Proteger endpoints en Django

### Opción 1: Por vista

```python
from rest_framework.permissions import IsAuthenticated

class ProductoViewSet(viewsets.ModelViewSet):
    queryset = Producto.objects.all()
    serializer_class = ProductoSerializer
    permission_classes = [IsAuthenticated]  # ← Solo autenticados
```

### Opción 2: Por método

```python
from rest_framework.decorators import action
from rest_framework.permissions import AllowAny, IsAuthenticated

class ProductoViewSet(viewsets.ModelViewSet):
    queryset = Producto.objects.all()
    serializer_class = ProductoSerializer
    
    def get_permissions(self):
        if self.action in ['list', 'retrieve']:
            return [AllowAny()]  # GET: todos
        return [IsAuthenticated()]  # POST, PUT, DELETE: solo autenticados
```

---

## Resumen

**JWT = Tokens para autenticación stateless**

**Flujo:**
1. Usuario se registra/loguea → recibe access + refresh tokens
2. Frontend guarda tokens en localStorage
3. Cada request → Header: `Authorization: Bearer {access_token}`
4. Access expira → usar refresh para obtener nuevo access

**Django:**
- `djangorestframework-simplejwt`
- Endpoints: `/login/`, `/register/`, `/token/refresh/`

**React:**
- Context API para estado de autenticación
- Axios interceptors para auto-refresh
- Rutas protegidas con ProtectedRoute

**Próximo módulo:** Axios avanzado y cliente API estructurado

---

## Recursos

- **[Django REST Simple JWT](https://django-rest-framework-simplejwt.readthedocs.io/)** - Documentación oficial
- **[JWT.io](https://jwt.io/)** - Debugger de tokens
- **[React Context API](https://react.dev/reference/react/useContext)** - Documentación

¡Autenticación JWT implementada! 🔐

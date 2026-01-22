# Introducción a Django

> **Framework web de alto nivel que facilita el desarrollo rápido y limpio**

---

## ¿Por qué Django después de vanilla?

Has aprendido a construir todo **desde cero**:
- ✅ Servidor HTTP con `BaseHTTPRequestHandler`
- ✅ Routing manual
- ✅ JWT manual
- ✅ CORS manual
- ✅ Rate limiting manual
- ✅ ORM manual con dataclasses

**Ahora entenderás qué hace Django por ti.**

---

## ¿Qué es Django?

**Django** es un framework web Python que incluye:
- 🗄️ **ORM**: Interactúa con bases de datos sin SQL
- 🔒 **Autenticación**: Sistema completo incluido
- 🎨 **Admin**: Panel de administración automático
- 🛣️ **Routing**: URLs declarativas
- 🔐 **Seguridad**: CSRF, XSS, SQL injection protegidos
- 📦 **Baterías incluidas**: Email, sesiones, cache, etc.

**Filosofía:** "Don't Repeat Yourself" (DRY)

---

## Comparación: Vanilla vs Django

### Servidor HTTP

**Vanilla (lo que hiciste):**
```python
from http.server import BaseHTTPRequestHandler, HTTPServer

class Handler(BaseHTTPRequestHandler):
    def do_GET(self):
        if self.path == '/api/usuarios':
            self.send_response(200)
            self.send_header('Content-type', 'application/json')
            self.end_headers()
            self.wfile.write(b'{"usuarios": []}')

servidor = HTTPServer(('localhost', 8000), Handler)
servidor.serve_forever()
```

**Django:**
```python
# Ya incluido, solo ejecutas:
python manage.py runserver
```

---

### Routing

**Vanilla (lo que hiciste):**
```python
def _manejar(self):
    if self.path == '/api/usuarios':
        self.listar_usuarios()
    elif self.path.startswith('/api/usuarios/'):
        partes = self.path.split('/')
        if len(partes) == 4:
            id_usuario = int(partes[3])
            self.obtener_usuario(id_usuario)
```

**Django:**
```python
# urls.py
from django.urls import path
from . import views

urlpatterns = [
    path('api/usuarios/', views.listar_usuarios),
    path('api/usuarios/<int:id>/', views.obtener_usuario),
]
```

---

### Base de datos

**Vanilla (lo que hiciste):**
```python
class UsuarioRepository:
    def __init__(self):
        self._usuarios = {}
        self._siguiente_id = 1
    
    def crear(self, usuario):
        usuario.id = self._siguiente_id
        self._siguiente_id += 1
        self._usuarios[usuario.id] = usuario
        return usuario
    
    def obtener_por_id(self, id_usuario):
        return self._usuarios.get(id_usuario)
```

**Django:**
```python
# models.py
from django.db import models

class Usuario(models.Model):
    nombre = models.CharField(max_length=100)
    email = models.EmailField(unique=True)
    activo = models.BooleanField(default=True)

# Uso:
usuario = Usuario.objects.create(nombre='Ana', email='ana@example.com')
usuario = Usuario.objects.get(id=1)
```

---

### Autenticación

**Vanilla (lo que hiciste):**
```python
import jwt
import bcrypt

def login(email, password):
    usuario = usuarios_db.get(email)
    if bcrypt.checkpw(password.encode(), usuario.password_hash):
        token = jwt.encode({
            'user_id': usuario.id,
            'exp': datetime.utcnow() + timedelta(minutes=15)
        }, SECRET_KEY, algorithm='HS256')
        return token
    raise ValueError('Credenciales inválidas')
```

**Django:**
```python
from django.contrib.auth import authenticate, login

def vista_login(request):
    user = authenticate(username=username, password=password)
    if user:
        login(request, user)
        return JsonResponse({'mensaje': 'Login exitoso'})
```

---

### Admin panel

**Vanilla (lo que hiciste):**
```python
# Tendrías que construir un frontend completo
# con HTML/CSS/JS para gestionar usuarios
```

**Django:**
```python
# admin.py
from django.contrib import admin
from .models import Usuario

admin.site.register(Usuario)

# ¡Ya tienes un panel de administración completo!
# http://localhost:8000/admin
```

---

## Instalación de Django

```bash
# Crear entorno virtual
python -m venv venv

# Activar entorno (Windows)
venv\Scripts\activate

# Activar entorno (Linux/Mac)
source venv/bin/activate

# Instalar Django
pip install django

# Verificar instalación
django-admin --version
# Output: 5.0.1
```

---

## Crear proyecto Django

```bash
# Crear proyecto
django-admin startproject miproyecto

# Estructura creada:
miproyecto/
├── manage.py              # Utilidad de línea de comandos
└── miproyecto/
    ├── __init__.py
    ├── settings.py        # Configuración del proyecto
    ├── urls.py            # URLs principales
    ├── asgi.py            # Entrada ASGI
    └── wsgi.py            # Entrada WSGI
```

---

## Crear aplicación Django

```bash
# Entrar al proyecto
cd miproyecto

# Crear aplicación
python manage.py startapp usuarios

# Estructura de la app:
usuarios/
├── __init__.py
├── admin.py              # Configuración del admin
├── apps.py               # Configuración de la app
├── models.py             # Modelos (base de datos)
├── tests.py              # Tests
├── views.py              # Vistas (lógica)
└── migrations/           # Migraciones de BD
    └── __init__.py
```

---

## Primer modelo Django

**usuarios/models.py**

```python
"""
Primer modelo Django
"""

from django.db import models

class Usuario(models.Model):
    """Modelo de usuario"""
    nombre = models.CharField(max_length=100)
    email = models.EmailField(unique=True)
    activo = models.BooleanField(default=True)
    creado_en = models.DateTimeField(auto_now_add=True)
    actualizado_en = models.DateTimeField(auto_now=True)
    
    class Meta:
        db_table = 'usuarios'
        ordering = ['-creado_en']
    
    def __str__(self):
        return f'{self.nombre} ({self.email})'
```

---

## Registrar aplicación

**miproyecto/settings.py**

```python
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'usuarios',  # ← Agregar tu app
]
```

---

## Crear migraciones

```bash
# Crear archivos de migración
python manage.py makemigrations

# Output:
# Migrations for 'usuarios':
#   usuarios/migrations/0001_initial.py
#     - Create model Usuario

# Aplicar migraciones
python manage.py migrate

# Output:
# Operations to perform:
#   Apply all migrations: admin, auth, contenttypes, sessions, usuarios
# Running migrations:
#   Applying usuarios.0001_initial... OK
```

---

## Primera vista

**usuarios/views.py**

```python
"""
Primera vista Django
"""

from django.http import JsonResponse
from django.views.decorators.http import require_http_methods
from .models import Usuario

@require_http_methods(['GET'])
def listar_usuarios(request):
    """GET /api/usuarios/"""
    usuarios = Usuario.objects.all()
    
    data = {
        'usuarios': [
            {
                'id': u.id,
                'nombre': u.nombre,
                'email': u.email,
                'activo': u.activo,
                'creado_en': u.creado_en.isoformat()
            }
            for u in usuarios
        ],
        'total': usuarios.count()
    }
    
    return JsonResponse(data)

@require_http_methods(['POST'])
def crear_usuario(request):
    """POST /api/usuarios/"""
    import json
    
    datos = json.loads(request.body)
    
    usuario = Usuario.objects.create(
        nombre=datos['nombre'],
        email=datos['email']
    )
    
    return JsonResponse({
        'id': usuario.id,
        'nombre': usuario.nombre,
        'email': usuario.email,
        'activo': usuario.activo
    }, status=201)
```

---

## Configurar URLs

**usuarios/urls.py** (crear nuevo archivo)

```python
"""
URLs de la app usuarios
"""

from django.urls import path
from . import views

urlpatterns = [
    path('', views.listar_usuarios, name='listar_usuarios'),
    path('crear/', views.crear_usuario, name='crear_usuario'),
]
```

**miproyecto/urls.py** (modificar existente)

```python
"""
URLs principales del proyecto
"""

from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path('admin/', admin.site.urls),
    path('api/usuarios/', include('usuarios.urls')),
]
```

---

## Ejecutar servidor

```bash
# Ejecutar servidor de desarrollo
python manage.py runserver

# Output:
# Watching for file changes with StatReloader
# Performing system checks...
#
# System check identified no issues (0 silenced).
# January 20, 2026 - 10:30:00
# Django version 5.0.1, using settings 'miproyecto.settings'
# Starting development server at http://127.0.0.1:8000/
# Quit the server with CTRL-BREAK.
```

---

## Probar API

```bash
# Listar usuarios
curl http://localhost:8000/api/usuarios/

# Respuesta:
# {
#   "usuarios": [],
#   "total": 0
# }

# Crear usuario
curl -X POST http://localhost:8000/api/usuarios/crear/ \
  -H "Content-Type: application/json" \
  -d '{
    "nombre": "Ana García",
    "email": "ana@example.com"
  }'

# Respuesta:
# {
#   "id": 1,
#   "nombre": "Ana García",
#   "email": "ana@example.com",
#   "activo": true
# }
```

---

## Admin de Django

**usuarios/admin.py**

```python
"""
Configuración del admin
"""

from django.contrib import admin
from .models import Usuario

@admin.register(Usuario)
class UsuarioAdmin(admin.ModelAdmin):
    """Admin de Usuario"""
    list_display = ['id', 'nombre', 'email', 'activo', 'creado_en']
    list_filter = ['activo', 'creado_en']
    search_fields = ['nombre', 'email']
    ordering = ['-creado_en']
```

**Crear superusuario:**

```bash
python manage.py createsuperuser

# Username: admin
# Email address: admin@example.com
# Password: 
# Password (again):
# Superuser created successfully.
```

**Acceder al admin:**
- URL: http://localhost:8000/admin
- Login con las credenciales creadas
- ¡Panel completo de gestión de usuarios!

---

## Estructura completa del proyecto

```
miproyecto/
├── manage.py
├── miproyecto/
│   ├── __init__.py
│   ├── settings.py
│   ├── urls.py
│   ├── asgi.py
│   └── wsgi.py
├── usuarios/
│   ├── __init__.py
│   ├── admin.py
│   ├── apps.py
│   ├── models.py
│   ├── views.py
│   ├── urls.py
│   ├── tests.py
│   └── migrations/
│       ├── __init__.py
│       └── 0001_initial.py
└── db.sqlite3
```

---

## Django Shell

```bash
# Abrir shell interactivo
python manage.py shell

# Dentro del shell:
>>> from usuarios.models import Usuario

# Crear usuario
>>> u = Usuario.objects.create(nombre='Bob', email='bob@example.com')
>>> u.id
1

# Obtener usuario
>>> u = Usuario.objects.get(id=1)
>>> u.nombre
'Bob'

# Listar todos
>>> Usuario.objects.all()
<QuerySet [<Usuario: Bob (bob@example.com)>]>

# Filtrar
>>> Usuario.objects.filter(activo=True)
<QuerySet [<Usuario: Bob (bob@example.com)>]>

# Actualizar
>>> u.nombre = 'Roberto'
>>> u.save()

# Eliminar
>>> u.delete()
```

---

## Ventajas de Django

✅ **Baterías incluidas**
- ORM, admin, autenticación, sesiones, cache

✅ **Seguridad por defecto**
- CSRF, XSS, SQL injection protegidos

✅ **ORM potente**
- Queries complejas sin escribir SQL

✅ **Admin automático**
- Panel de gestión sin código frontend

✅ **Escalable**
- Instagram, Pinterest, Spotify usan Django

✅ **Comunidad**
- Miles de paquetes y extensiones

---

## Cuándo usar Django

**✅ Usar Django cuando:**
- Necesitas desarrollo rápido
- Quieres un admin panel
- ORM es suficiente
- Aplicación tradicional web

**❌ Considerar alternativas cuando:**
- API pura sin admin (→ FastAPI)
- Microservicios mínimos (→ Flask)
- WebSockets intensivos (→ FastAPI/WebSockets)
- Necesitas máximo control (→ Vanilla)

---

## Comparación frameworks

| Característica | Django | FastAPI | Flask | Vanilla |
|----------------|--------|---------|-------|---------|
| **ORM incluido** | ✅ | ❌ | ❌ | ❌ |
| **Admin panel** | ✅ | ❌ | ❌ | ❌ |
| **Auth incluida** | ✅ | ❌ | ❌ | ❌ |
| **Velocidad** | Media | Muy alta | Alta | Alta |
| **Curva aprendizaje** | Media | Baja | Baja | Alta |
| **Baterías incluidas** | Muchas | Pocas | Pocas | Ninguna |
| **Async nativo** | Partial | ✅ | ❌ | Manual |

---

## Próximos pasos

En los siguientes módulos aprenderás:

1. **Models y ORM**: Base de datos sin SQL
2. **Views y URLs**: Routing y lógica
3. **Django REST Framework**: APIs profesionales
4. **Autenticación Django**: Users, permisos, JWT
5. **Admin avanzado**: Personalización
6. **Testing Django**: TestCase, fixtures
7. **Deployment**: Producción con Gunicorn

---

## Resumen

Has aprendido:

✅ Por qué Django después de vanilla  
✅ Comparación Django vs tu código vanilla  
✅ Instalación y primer proyecto  
✅ Primer modelo y migraciones  
✅ Primera API con Django  
✅ Admin panel automático

**Siguiente:** Models y ORM profundo

---

## Recursos

- **[Django Docs](https://docs.djangoproject.com/)** - Documentación oficial
- **[Django Tutorial](https://docs.djangoproject.com/en/5.0/intro/tutorial01/)** - Tutorial oficial
- **[Two Scoops of Django](https://www.feldroy.com/books/two-scoops-of-django-3-x)** - Mejores prácticas

Ahora entiendes qué abstrae Django. 🎯

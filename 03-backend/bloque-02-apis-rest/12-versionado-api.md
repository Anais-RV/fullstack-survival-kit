# Versionado de APIs

> **Gestión de versiones para evolución de APIs sin romper clientes**

---

## ¿Por qué versionar?

### Problema

Tu API cambia, pero hay clientes usando la versión anterior:

```python
# Versión 1 (clientes actuales)
{
  "nombre": "Juan",
  "email": "juan@ejemplo.com"
}

# Versión 2 (nuevo diseño)
{
  "nombre_completo": {
    "nombre": "Juan",
    "apellido": "Pérez"
  },
  "contacto": {
    "email": "juan@ejemplo.com",
    "telefono": "+52123456789"
  }
}
```

Si cambias directamente, **rompes** los clientes existentes. ❌

### Solución: Versionado

Mantén **ambas versiones** funcionando simultáneamente:

```
/api/v1/usuarios  → Estructura antigua (compatible)
/api/v2/usuarios  → Estructura nueva
```

Los clientes migran gradualmente. ✅

---

## Estrategias de versionado

### 1. URL Path (más común)

```
GET /api/v1/usuarios
GET /api/v2/usuarios
GET /api/v3/usuarios
```

**Ventajas:**
- ✅ Explícito y claro
- ✅ Fácil de cachear
- ✅ Fácil de documentar

**Desventajas:**
- ❌ URLs más largas

### 2. Query Parameter

```
GET /api/usuarios?version=1
GET /api/usuarios?version=2
```

**Ventajas:**
- ✅ URL más corta
- ✅ Fácil cambiar versión

**Desventajas:**
- ❌ Menos explícito
- ❌ Problemas con caché

### 3. Header

```
GET /api/usuarios
Accept: application/vnd.miapi.v1+json

GET /api/usuarios
Accept: application/vnd.miapi.v2+json
```

**Ventajas:**
- ✅ URL limpia
- ✅ Estándar REST

**Desventajas:**
- ❌ Menos explícito
- ❌ Requiere configurar headers

### 4. Subdomain

```
https://api-v1.miapp.com/usuarios
https://api-v2.miapp.com/usuarios
```

**Ventajas:**
- ✅ Separación total
- ✅ Cada versión puede ser servidor distinto

**Desventajas:**
- ❌ Requiere configuración DNS
- ❌ Más complejo

---

## Implementación: Versionado por URL

### Estructura

```
/api/v1/usuarios    → Versión 1
/api/v1/posts       → Versión 1

/api/v2/usuarios    → Versión 2
/api/v2/posts       → Versión 2
```

### Código

```python
from http.server import BaseHTTPRequestHandler, HTTPServer
from urllib.parse import urlparse
import json
import re

# ===== Versión 1: Estructura simple =====

usuarios_v1 = [
    {'id': 1, 'nombre': 'Juan', 'email': 'juan@ejemplo.com'},
    {'id': 2, 'nombre': 'María', 'email': 'maria@ejemplo.com'}
]

def listar_usuarios_v1():
    """GET /api/v1/usuarios"""
    return usuarios_v1

def obtener_usuario_v1(user_id):
    """GET /api/v1/usuarios/:id"""
    return next((u for u in usuarios_v1 if u['id'] == user_id), None)

# ===== Versión 2: Estructura mejorada =====

usuarios_v2 = [
    {
        'id': 1,
        'nombre_completo': {
            'nombre': 'Juan',
            'apellido': 'Pérez'
        },
        'contacto': {
            'email': 'juan@ejemplo.com',
            'telefono': '+52123456789'
        },
        'metadata': {
            'activo': True,
            'creado_en': '2024-01-01T00:00:00'
        }
    },
    {
        'id': 2,
        'nombre_completo': {
            'nombre': 'María',
            'apellido': 'García'
        },
        'contacto': {
            'email': 'maria@ejemplo.com',
            'telefono': '+52987654321'
        },
        'metadata': {
            'activo': True,
            'creado_en': '2024-01-02T00:00:00'
        }
    }
]

def listar_usuarios_v2():
    """GET /api/v2/usuarios"""
    return usuarios_v2

def obtener_usuario_v2(user_id):
    """GET /api/v2/usuarios/:id"""
    return next((u for u in usuarios_v2 if u['id'] == user_id), None)

# ===== Handler con versionado =====

class VersionedAPIHandler(BaseHTTPRequestHandler):
    
    def do_GET(self):
        ruta = urlparse(self.path).path
        
        # ===== Versión 1 =====
        
        if ruta == '/api/v1/usuarios':
            usuarios = listar_usuarios_v1()
            self.enviar_json(usuarios)
            return
        
        match_v1 = re.match(r'^/api/v1/usuarios/(\d+)$', ruta)
        if match_v1:
            user_id = int(match_v1.group(1))
            usuario = obtener_usuario_v1(user_id)
            
            if usuario:
                self.enviar_json(usuario)
            else:
                self.enviar_error(404, 'Usuario no encontrado')
            return
        
        # ===== Versión 2 =====
        
        if ruta == '/api/v2/usuarios':
            usuarios = listar_usuarios_v2()
            self.enviar_json(usuarios)
            return
        
        match_v2 = re.match(r'^/api/v2/usuarios/(\d+)$', ruta)
        if match_v2:
            user_id = int(match_v2.group(1))
            usuario = obtener_usuario_v2(user_id)
            
            if usuario:
                self.enviar_json(usuario)
            else:
                self.enviar_error(404, 'Usuario no encontrado')
            return
        
        # ===== No encontrado =====
        
        self.enviar_error(404, f'Ruta no encontrada: {ruta}')
    
    def enviar_json(self, datos, status=200):
        self.send_response(status)
        self.send_header('Content-type', 'application/json')
        self.send_header('Access-Control-Allow-Origin', '*')
        self.end_headers()
        self.wfile.write(json.dumps(datos, ensure_ascii=False, indent=2).encode())
    
    def enviar_error(self, codigo, mensaje):
        self.send_response(codigo)
        self.send_header('Content-type', 'application/json')
        self.end_headers()
        error = {'error': mensaje, 'codigo': codigo}
        self.wfile.write(json.dumps(error, ensure_ascii=False).encode())
    
    def log_message(self, format, *args):
        print(f'📥 {self.command} {self.path}')

if __name__ == '__main__':
    servidor = HTTPServer(('localhost', 8000), VersionedAPIHandler)
    print('🚀 API versionada en http://localhost:8000')
    print('\n📋 Versiones disponibles:')
    print('  v1: GET /api/v1/usuarios')
    print('  v1: GET /api/v1/usuarios/:id')
    print('  v2: GET /api/v2/usuarios')
    print('  v2: GET /api/v2/usuarios/:id\n')
    
    try:
        servidor.serve_forever()
    except KeyboardInterrupt:
        print('\n⛔ Servidor detenido')
        servidor.shutdown()
```

---

## Pruebas

```powershell
# Versión 1 (simple)
curl http://localhost:8000/api/v1/usuarios

# Output:
# [
#   {
#     "id": 1,
#     "nombre": "Juan",
#     "email": "juan@ejemplo.com"
#   }
# ]

# Versión 2 (detallada)
curl http://localhost:8000/api/v2/usuarios

# Output:
# [
#   {
#     "id": 1,
#     "nombre_completo": {
#       "nombre": "Juan",
#       "apellido": "Pérez"
#     },
#     "contacto": {
#       "email": "juan@ejemplo.com",
#       "telefono": "+52123456789"
#     },
#     "metadata": {
#       "activo": true,
#       "creado_en": "2024-01-01T00:00:00"
#     }
#   }
# ]
```

---

## Router con versionado

```python
from typing import Callable, Dict

class VersionedRouter:
    """Router que soporta múltiples versiones"""
    
    def __init__(self):
        self.versions: Dict[str, Dict[str, Callable]] = {}
    
    def register_version(self, version: str):
        """Registra una nueva versión"""
        if version not in self.versions:
            self.versions[version] = {}
    
    def route(self, version: str, metodo: str, patron: str):
        """Decorador para registrar ruta versionada"""
        def decorator(handler):
            # Asegurar que la versión existe
            self.register_version(version)
            
            # Registrar ruta completa
            ruta_completa = f'/api/{version}{patron}'
            key = f'{metodo} {ruta_completa}'
            
            self.versions[version][key] = handler
            
            return handler
        return decorator
    
    def find_handler(self, metodo: str, ruta: str):
        """Busca handler para método y ruta"""
        # Extraer versión de la ruta
        match = re.match(r'^/api/(v\d+)(/.*)', ruta)
        if not match:
            return None
        
        version = match.group(1)
        ruta_sin_version = match.group(2)
        
        # Buscar en versión específica
        if version in self.versions:
            key = f'{metodo} /api/{version}{ruta_sin_version}'
            return self.versions[version].get(key)
        
        return None

# Uso:
router = VersionedRouter()

# ===== Versión 1 =====

@router.route('v1', 'GET', '/usuarios')
def listar_usuarios_v1(request):
    usuarios = [
        {'id': 1, 'nombre': 'Juan', 'email': 'juan@ejemplo.com'}
    ]
    request.enviar_json(usuarios)

@router.route('v1', 'GET', '/usuarios/:id')
def obtener_usuario_v1(request, user_id):
    usuario = {'id': int(user_id), 'nombre': 'Juan', 'email': 'juan@ejemplo.com'}
    request.enviar_json(usuario)

# ===== Versión 2 =====

@router.route('v2', 'GET', '/usuarios')
def listar_usuarios_v2(request):
    usuarios = [
        {
            'id': 1,
            'nombre_completo': {'nombre': 'Juan', 'apellido': 'Pérez'},
            'contacto': {'email': 'juan@ejemplo.com', 'telefono': '+52123456789'}
        }
    ]
    request.enviar_json(usuarios)

@router.route('v2', 'GET', '/usuarios/:id')
def obtener_usuario_v2(request, user_id):
    usuario = {
        'id': int(user_id),
        'nombre_completo': {'nombre': 'Juan', 'apellido': 'Pérez'},
        'contacto': {'email': 'juan@ejemplo.com'}
    }
    request.enviar_json(usuario)
```

---

## Compatibilidad hacia atrás

### Problema

Tienes v1 y v2, pero **compartes la misma base de datos**:

```python
# Base de datos (estructura v2)
usuarios = [
    {
        'id': 1,
        'nombre_completo': {'nombre': 'Juan', 'apellido': 'Pérez'},
        'contacto': {'email': 'juan@ejemplo.com'}
    }
]
```

### Solución: Transformadores

```python
def transformar_v2_a_v1(usuario_v2):
    """Convierte formato v2 → v1"""
    return {
        'id': usuario_v2['id'],
        'nombre': usuario_v2['nombre_completo']['nombre'],
        'email': usuario_v2['contacto']['email']
    }

def listar_usuarios_v1():
    """GET /api/v1/usuarios"""
    # Leer de DB (formato v2)
    usuarios_v2 = obtener_de_db()
    
    # Transformar a v1
    usuarios_v1 = [transformar_v2_a_v1(u) for u in usuarios_v2]
    
    return usuarios_v1

def listar_usuarios_v2():
    """GET /api/v2/usuarios"""
    # Leer directamente
    return obtener_de_db()
```

---

## Deprecación de versiones

### 1. Anunciar deprecación

```json
GET /api/v1/usuarios

Response:
{
  "data": [...],
  "warnings": [
    "API v1 será deprecada el 2024-12-31. Migre a v2."
  ]
}
```

### 2. Header de deprecación

```python
def listar_usuarios_v1(request):
    # Agregar warning header
    request.send_header('Deprecation', 'true')
    request.send_header('Sunset', 'Sat, 31 Dec 2024 23:59:59 GMT')
    request.send_header('Link', '</api/v2/usuarios>; rel="successor-version"')
    
    # Responder normalmente
    request.enviar_json(usuarios_v1)
```

### 3. Documentar migración

```markdown
# Guía de Migración: v1 → v2

## Cambios

### Usuario

**v1:**
```json
{
  "id": 1,
  "nombre": "Juan",
  "email": "juan@ejemplo.com"
}
```

**v2:**
```json
{
  "id": 1,
  "nombre_completo": {
    "nombre": "Juan",
    "apellido": "Pérez"
  },
  "contacto": {
    "email": "juan@ejemplo.com",
    "telefono": "+52123456789"
  }
}
```

## Checklist

- [ ] Actualizar parseo de `nombre` → `nombre_completo.nombre`
- [ ] Actualizar parseo de `email` → `contacto.email`
- [ ] Manejar nuevo campo `telefono`
- [ ] Actualizar tests
- [ ] Cambiar URLs `/api/v1/` → `/api/v2/`
```

---

## Versionado semántico de APIs

**Semantic Versioning:** `MAJOR.MINOR.PATCH`

```
v1.0.0 → v1.0.1  (PATCH: bug fix, compatible)
v1.0.0 → v1.1.0  (MINOR: nueva funcionalidad, compatible)
v1.0.0 → v2.0.0  (MAJOR: cambios incompatibles)
```

**En APIs:**
```
/api/v1/...  → MAJOR version
```

**Reglas:**
- **v1 → v2**: Cambios incompatibles (rompe clientes)
- **v1.0 → v1.1**: Nuevos endpoints (no rompe clientes)
- **v1.0.0 → v1.0.1**: Bugfixes (no rompe clientes)

---

## Mejores prácticas

### 1. Mantener mínimo 2 versiones

```
v1: Versión anterior (soporte 1 año)
v2: Versión actual
v3: Próxima versión (beta)
```

### 2. Documentar cambios

```markdown
# Changelog

## v2.0.0 (2024-01-20)
- **BREAKING:** Campo `nombre` → `nombre_completo.nombre`
- **BREAKING:** Campo `email` → `contacto.email`
- **NEW:** Campo `telefono` en contacto
- **NEW:** Campo `metadata` con timestamps

## v1.5.0 (2023-12-01)
- **NEW:** Endpoint GET /api/v1/usuarios/:id/posts
- **FIX:** Validación de email mejorada
```

### 3. Versión por defecto

```python
# Si no especifica versión, usar latest
GET /api/usuarios  → redirige a /api/v2/usuarios
```

### 4. Headers informativos

```http
X-API-Version: 2.0.0
X-API-Deprecated: false
X-API-Supported-Versions: v1, v2
```

---

## Resumen

Estrategias de versionado:

| Estrategia | Ejemplo | Recomendación |
|------------|---------|---------------|
| **URL Path** | `/api/v1/usuarios` | ⭐⭐⭐⭐⭐ Más común |
| **Query Param** | `/api/usuarios?v=1` | ⭐⭐⭐ Alternativa |
| **Header** | `Accept: vnd.api.v1+json` | ⭐⭐ RESTful puro |
| **Subdomain** | `api-v1.app.com` | ⭐ Complicado |

**Recomendación:** URL Path (`/api/v1/`, `/api/v2/`)

Has aprendido:

✅ Por qué versionar APIs  
✅ Estrategias de versionado  
✅ Implementación con URL path  
✅ Router versionado  
✅ Compatibilidad hacia atrás  
✅ Deprecación de versiones

---

## Siguiente bloque

**Siguiente:** [Bloque 3 - POO aplicada](../bloque-03-poo/)

---

## Recursos

- **[Semantic Versioning](https://semver.org/)** - Versionado semántico
- **[API Evolution](https://www.infoq.com/articles/API-Design-Evolution/)** - Evolución de APIs

El versionado permite evolucionar tu API sin romper clientes. 🔄

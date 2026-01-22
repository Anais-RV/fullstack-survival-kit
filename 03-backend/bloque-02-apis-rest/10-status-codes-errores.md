# Status Codes y Manejo de Errores

> **Códigos HTTP correctos y respuestas de error profesionales**

---

## Status Codes principales

### 2xx - Éxito

| Código | Nombre | Uso |
|--------|--------|-----|
| **200** | OK | GET, PUT, PATCH exitosos |
| **201** | Created | POST exitoso (recurso creado) |
| **204** | No Content | DELETE exitoso (sin respuesta) |

### 4xx - Errores del cliente

| Código | Nombre | Uso |
|--------|--------|-----|
| **400** | Bad Request | Datos inválidos, JSON malformado |
| **401** | Unauthorized | No autenticado (falta token) |
| **403** | Forbidden | No autorizado (sin permisos) |
| **404** | Not Found | Recurso no existe |
| **422** | Unprocessable Entity | Validación fallida |
| **429** | Too Many Requests | Rate limit excedido |

### 5xx - Errores del servidor

| Código | Nombre | Uso |
|--------|--------|-----|
| **500** | Internal Server Error | Error no manejado |
| **503** | Service Unavailable | Servicio temporalmente fuera |

---

## Respuestas de error profesionales

### Estructura estándar

```json
{
  "error": "Mensaje descriptivo del error",
  "codigo": 404,
  "detalles": "Información adicional opcional",
  "timestamp": "2024-01-20T10:30:00",
  "path": "/api/usuarios/999"
}
```

---

## Implementación completa

```python
from http.server import BaseHTTPRequestHandler, HTTPServer
from urllib.parse import urlparse
import json
import re
from datetime import datetime
from typing import Dict, Any, Optional

class HTTPError(Exception):
    """Excepción personalizada para errores HTTP"""
    
    def __init__(self, codigo: int, mensaje: str, detalles: Optional[Any] = None):
        self.codigo = codigo
        self.mensaje = mensaje
        self.detalles = detalles
        super().__init__(mensaje)

class ErrorHandler(BaseHTTPRequestHandler):
    """Handler con manejo profesional de errores"""
    
    def do_GET(self):
        try:
            ruta = urlparse(self.path).path
            
            match = re.match(r'^/api/usuarios/(\d+)$', ruta)
            if match:
                self.obtener_usuario(int(match.group(1)))
            elif ruta == '/api/usuarios':
                self.listar_usuarios()
            else:
                raise HTTPError(404, f'Ruta no encontrada: {ruta}')
                
        except HTTPError as e:
            self.enviar_error_http(e)
        except Exception as e:
            self.enviar_error_interno(e)
    
    def do_POST(self):
        try:
            if urlparse(self.path).path == '/api/usuarios':
                self.crear_usuario()
            else:
                raise HTTPError(404, 'Ruta no encontrada')
                
        except HTTPError as e:
            self.enviar_error_http(e)
        except json.JSONDecodeError:
            self.enviar_error_http(HTTPError(400, 'JSON inválido'))
        except Exception as e:
            self.enviar_error_interno(e)
    
    # ===== Handlers =====
    
    usuarios = [
        {'id': 1, 'nombre': 'Juan', 'email': 'juan@ejemplo.com'},
        {'id': 2, 'nombre': 'María', 'email': 'maria@ejemplo.com'}
    ]
    
    def listar_usuarios(self):
        """GET /api/usuarios"""
        self.enviar_json(self.usuarios, 200)
    
    def obtener_usuario(self, user_id: int):
        """GET /api/usuarios/:id"""
        usuario = next((u for u in self.usuarios if u['id'] == user_id), None)
        
        if not usuario:
            raise HTTPError(
                404,
                f'Usuario {user_id} no encontrado',
                {'user_id': user_id}
            )
        
        self.enviar_json(usuario, 200)
    
    def crear_usuario(self):
        """POST /api/usuarios"""
        datos = self.leer_json_body()
        
        # Validación
        errores = []
        
        if 'nombre' not in datos or not datos['nombre']:
            errores.append({
                'campo': 'nombre',
                'mensaje': 'Campo requerido'
            })
        
        if 'email' not in datos or not datos['email']:
            errores.append({
                'campo': 'email',
                'mensaje': 'Campo requerido'
            })
        elif '@' not in datos['email']:
            errores.append({
                'campo': 'email',
                'mensaje': 'Email inválido'
            })
        
        # Si hay errores, lanzar 422
        if errores:
            raise HTTPError(
                422,
                'Validación fallida',
                {'errores': errores}
            )
        
        # Crear usuario
        nuevo_id = max([u['id'] for u in self.usuarios], default=0) + 1
        nuevo = {
            'id': nuevo_id,
            'nombre': datos['nombre'],
            'email': datos['email']
        }
        
        self.usuarios.append(nuevo)
        
        # 201 Created con Location header
        self.send_response(201)
        self.send_header('Content-type', 'application/json')
        self.send_header('Location', f'/api/usuarios/{nuevo_id}')
        self.end_headers()
        self.wfile.write(json.dumps(nuevo, ensure_ascii=False, indent=2).encode())
    
    # ===== Manejo de errores =====
    
    def enviar_error_http(self, error: HTTPError):
        """Envía error HTTP estructurado"""
        respuesta = {
            'error': error.mensaje,
            'codigo': error.codigo,
            'timestamp': datetime.now().isoformat(),
            'path': self.path
        }
        
        if error.detalles:
            respuesta['detalles'] = error.detalles
        
        self.send_response(error.codigo)
        self.send_header('Content-type', 'application/json')
        self.end_headers()
        self.wfile.write(json.dumps(respuesta, ensure_ascii=False, indent=2).encode())
        
        # Log del error
        print(f'❌ {error.codigo} - {error.mensaje} - {self.path}')
    
    def enviar_error_interno(self, error: Exception):
        """Maneja errores no capturados (500)"""
        import traceback
        
        # Log completo del error
        print('❌ Error interno del servidor:')
        traceback.print_exc()
        
        # Respuesta al cliente (sin exponer detalles internos)
        respuesta = {
            'error': 'Error interno del servidor',
            'codigo': 500,
            'timestamp': datetime.now().isoformat(),
            'path': self.path
        }
        
        self.send_response(500)
        self.send_header('Content-type', 'application/json')
        self.end_headers()
        self.wfile.write(json.dumps(respuesta, ensure_ascii=False, indent=2).encode())
    
    # ===== Helpers =====
    
    def enviar_json(self, datos, status=200):
        self.send_response(status)
        self.send_header('Content-type', 'application/json')
        self.end_headers()
        self.wfile.write(json.dumps(datos, ensure_ascii=False, indent=2).encode())
    
    def leer_json_body(self):
        content_length = int(self.headers.get('Content-Length', 0))
        if content_length == 0:
            raise HTTPError(400, 'Body vacío')
        
        body_raw = self.rfile.read(content_length)
        return json.loads(body_raw.decode('utf-8'))
    
    def log_message(self, format, *args):
        print(f'📥 {self.command} {self.path}')

if __name__ == '__main__':
    servidor = HTTPServer(('localhost', 8000), ErrorHandler)
    print('🚀 API con manejo de errores en http://localhost:8000')
    servidor.serve_forever()
```

---

## Pruebas de errores

```powershell
# 404 - Ruta no encontrada
curl http://localhost:8000/api/inexistente

# 404 - Usuario no existe
curl http://localhost:8000/api/usuarios/999

# 400 - Body vacío
curl -X POST http://localhost:8000/api/usuarios `
  -H "Content-Type: application/json"

# 400 - JSON inválido
curl -X POST http://localhost:8000/api/usuarios `
  -H "Content-Type: application/json" `
  -d '{nombre: "sin comillas"}'

# 422 - Validación fallida
curl -X POST http://localhost:8000/api/usuarios `
  -H "Content-Type: application/json" `
  -d '{\"email\":\"invalido\"}'

# 422 - Campos faltantes
curl -X POST http://localhost:8000/api/usuarios `
  -H "Content-Type: application/json" `
  -d '{}'
```

---

## Respuestas de error detalladas

### 400 Bad Request

```json
{
  "error": "JSON inválido",
  "codigo": 400,
  "timestamp": "2024-01-20T10:30:00",
  "path": "/api/usuarios"
}
```

### 404 Not Found

```json
{
  "error": "Usuario 999 no encontrado",
  "codigo": 404,
  "timestamp": "2024-01-20T10:30:00",
  "path": "/api/usuarios/999",
  "detalles": {
    "user_id": 999
  }
}
```

### 422 Unprocessable Entity

```json
{
  "error": "Validación fallida",
  "codigo": 422,
  "timestamp": "2024-01-20T10:30:00",
  "path": "/api/usuarios",
  "detalles": {
    "errores": [
      {
        "campo": "nombre",
        "mensaje": "Campo requerido"
      },
      {
        "campo": "email",
        "mensaje": "Email inválido"
      }
    ]
  }
}
```

### 500 Internal Server Error

```json
{
  "error": "Error interno del servidor",
  "codigo": 500,
  "timestamp": "2024-01-20T10:30:00",
  "path": "/api/usuarios/1"
}
```

---

## Validación con Pydantic

```python
from pydantic import BaseModel, EmailStr, Field, ValidationError

class CrearUsuarioDTO(BaseModel):
    nombre: str = Field(..., min_length=1, max_length=100)
    email: EmailStr
    edad: int = Field(..., ge=0, le=120)

def crear_usuario(self):
    """POST con validación Pydantic"""
    datos = self.leer_json_body()
    
    try:
        # Validar
        usuario_dto = CrearUsuarioDTO(**datos)
        
        # Crear usuario
        nuevo = {
            'id': 999,
            **usuario_dto.dict()
        }
        
        self.enviar_json(nuevo, 201)
        
    except ValidationError as e:
        # Pydantic genera errores estructurados
        raise HTTPError(
            422,
            'Validación fallida',
            {'errores': e.errors()}
        )
```

**Respuesta de error Pydantic:**
```json
{
  "error": "Validación fallida",
  "codigo": 422,
  "timestamp": "2024-01-20T10:30:00",
  "path": "/api/usuarios",
  "detalles": {
    "errores": [
      {
        "loc": ["email"],
        "msg": "value is not a valid email address",
        "type": "value_error.email"
      },
      {
        "loc": ["edad"],
        "msg": "ensure this value is greater than or equal to 0",
        "type": "value_error.number.not_ge",
        "ctx": {"limit_value": 0}
      }
    ]
  }
}
```

---

## Códigos de éxito apropiados

### 200 OK

**Uso:** GET, PUT, PATCH exitosos

```python
def obtener_usuario(self, user_id):
    usuario = buscar_usuario(user_id)
    self.enviar_json(usuario, 200)  # ✅

def actualizar_usuario(self, user_id):
    usuario = actualizar(user_id, datos)
    self.enviar_json(usuario, 200)  # ✅
```

### 201 Created

**Uso:** POST exitoso (con Location header)

```python
def crear_usuario(self):
    nuevo = crear_nuevo_usuario(datos)
    
    self.send_response(201)  # ✅ Created
    self.send_header('Location', f'/api/usuarios/{nuevo["id"]}')  # ✅ Dónde está
    self.send_header('Content-type', 'application/json')
    self.end_headers()
    self.wfile.write(json.dumps(nuevo).encode())
```

### 204 No Content

**Uso:** DELETE exitoso (sin body)

```python
def eliminar_usuario(self, user_id):
    eliminar(user_id)
    
    self.send_response(204)  # ✅ No Content
    self.end_headers()       # Sin body
```

---

## Middleware de manejo de errores

```python
class ErrorMiddleware:
    """Middleware que captura todos los errores"""
    
    @staticmethod
    def wrap_handler(handler_method):
        """Envuelve un handler con try/except"""
        def wrapper(self, *args, **kwargs):
            try:
                return handler_method(self, *args, **kwargs)
            
            except HTTPError as e:
                # Error HTTP controlado
                self.enviar_error_http(e)
            
            except json.JSONDecodeError:
                # JSON inválido
                self.enviar_error_http(HTTPError(400, 'JSON inválido'))
            
            except KeyError as e:
                # Campo faltante
                self.enviar_error_http(HTTPError(400, f'Campo requerido: {e}'))
            
            except ValueError as e:
                # Valor inválido (ej: int('abc'))
                self.enviar_error_http(HTTPError(400, f'Valor inválido: {e}'))
            
            except Exception as e:
                # Error no manejado → 500
                self.enviar_error_interno(e)
        
        return wrapper

# Uso:
class APIHandler(BaseHTTPRequestHandler):
    
    @ErrorMiddleware.wrap_handler
    def do_GET(self):
        # Tu código sin try/except
        ruta = urlparse(self.path).path
        # ...
    
    @ErrorMiddleware.wrap_handler
    def do_POST(self):
        # Tu código sin try/except
        datos = self.leer_json_body()
        # ...
```

---

## Logging de errores

```python
import logging
from datetime import datetime

logging.basicConfig(
    level=logging.INFO,
    format='%(asctime)s - %(levelname)s - %(message)s',
    handlers=[
        logging.FileHandler('errores.log'),
        logging.StreamHandler()
    ]
)

logger = logging.getLogger(__name__)

def enviar_error_http(self, error: HTTPError):
    """Envía error y lo loggea"""
    # Log según severidad
    if error.codigo >= 500:
        logger.error(f'{error.codigo} - {error.mensaje} - {self.path}')
    elif error.codigo >= 400:
        logger.warning(f'{error.codigo} - {error.mensaje} - {self.path}')
    
    # Enviar respuesta
    respuesta = {
        'error': error.mensaje,
        'codigo': error.codigo,
        'timestamp': datetime.now().isoformat(),
        'path': self.path
    }
    
    if error.detalles:
        respuesta['detalles'] = error.detalles
    
    self.send_response(error.codigo)
    self.send_header('Content-type', 'application/json')
    self.end_headers()
    self.wfile.write(json.dumps(respuesta, ensure_ascii=False, indent=2).encode())

def enviar_error_interno(self, error: Exception):
    """Error 500 con stack trace en log"""
    import traceback
    
    # Log completo con stack trace
    logger.exception(f'Error interno en {self.path}')
    
    # Respuesta genérica (no exponer detalles)
    respuesta = {
        'error': 'Error interno del servidor',
        'codigo': 500,
        'timestamp': datetime.now().isoformat()
    }
    
    self.send_response(500)
    self.send_header('Content-type', 'application/json')
    self.end_headers()
    self.wfile.write(json.dumps(respuesta).encode())
```

---

## Resumen

| Código | Cuándo usar | Response Body |
|--------|-------------|---------------|
| **200** | GET, PUT, PATCH exitoso | Recurso actualizado |
| **201** | POST exitoso | Recurso creado + Location header |
| **204** | DELETE exitoso | Sin body |
| **400** | JSON inválido, datos mal formados | Error descriptivo |
| **401** | No autenticado | Solicitar autenticación |
| **403** | Sin permisos | Acceso denegado |
| **404** | Recurso no existe | Error con ID |
| **422** | Validación fallida | Errores de validación detallados |
| **500** | Error no manejado | Error genérico (no exponer detalles) |

**Siguiente:** [Testing con Postman](./11-testing-postman.md)

---

## Recursos

- **[HTTP Status Codes](https://httpstatuses.com/)** - Lista completa
- **[RFC 7231](https://tools.ietf.org/html/rfc7231)** - HTTP/1.1 Semantics

Maneja errores como un profesional. 🚨

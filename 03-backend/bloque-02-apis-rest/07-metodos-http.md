# Métodos HTTP

> **GET, POST, PUT, PATCH, DELETE en profundidad**

---

## Los 5 métodos principales

| Método | Propósito | Idempotente | Safe | Body |
|--------|-----------|-------------|------|------|
| **GET** | Leer | ✅ | ✅ | No |
| **POST** | Crear | ❌ | ❌ | Sí |
| **PUT** | Reemplazar | ✅ | ❌ | Sí |
| **PATCH** | Modificar | ❌ | ❌ | Sí |
| **DELETE** | Eliminar | ✅ | ❌ | No |

### Conceptos clave

- **Idempotente**: Mismo resultado sin importar cuántas veces se ejecute
- **Safe**: No modifica el estado del servidor
- **Body**: Permite enviar datos en el cuerpo del request

---

## GET - Leer recursos

**Propósito**: Obtener datos sin modificar nada

### Características

- ✅ **Idempotente**: `GET /usuarios/123` siempre devuelve lo mismo
- ✅ **Safe**: No modifica datos
- ✅ **Cacheable**: Los navegadores pueden cachear
- ❌ **No Body**: Datos en query params

### Implementación

```python
from http.server import BaseHTTPRequestHandler, HTTPServer
from urllib.parse import urlparse, parse_qs
import json
import re

usuarios = [
    {'id': 1, 'nombre': 'Juan', 'email': 'juan@ejemplo.com', 'activo': True},
    {'id': 2, 'nombre': 'María', 'email': 'maria@ejemplo.com', 'activo': True},
    {'id': 3, 'nombre': 'Pedro', 'email': 'pedro@ejemplo.com', 'activo': False}
]

class GetHandler(BaseHTTPRequestHandler):
    
    def do_GET(self):
        ruta = urlparse(self.path).path
        query = parse_qs(urlparse(self.path).query)
        
        # GET /api/usuarios/:id
        match = re.match(r'^/api/usuarios/(\d+)$', ruta)
        if match:
            user_id = int(match.group(1))
            self.obtener_usuario(user_id)
            return
        
        # GET /api/usuarios
        if ruta == '/api/usuarios':
            self.listar_usuarios(query)
            return
        
        self.enviar_error(404, 'Ruta no encontrada')
    
    def listar_usuarios(self, query):
        """GET /api/usuarios - Con filtros"""
        resultado = usuarios.copy()
        
        # Filtro: activo
        if 'activo' in query:
            valor = query['activo'][0].lower() == 'true'
            resultado = [u for u in resultado if u['activo'] == valor]
        
        # Filtro: búsqueda por nombre
        if 'nombre' in query:
            termino = query['nombre'][0].lower()
            resultado = [u for u in resultado if termino in u['nombre'].lower()]
        
        # Paginación
        limit = int(query.get('limit', [10])[0])
        offset = int(query.get('offset', [0])[0])
        
        total = len(resultado)
        resultado = resultado[offset:offset + limit]
        
        respuesta = {
            'data': resultado,
            'meta': {
                'total': total,
                'limit': limit,
                'offset': offset
            }
        }
        
        self.enviar_json(respuesta)
    
    def obtener_usuario(self, user_id):
        """GET /api/usuarios/:id"""
        usuario = next((u for u in usuarios if u['id'] == user_id), None)
        
        if usuario:
            self.enviar_json(usuario)
        else:
            self.enviar_error(404, f'Usuario {user_id} no encontrado')
    
    def enviar_json(self, datos, status=200):
        self.send_response(status)
        self.send_header('Content-type', 'application/json')
        self.send_header('Cache-Control', 'max-age=300')  # Cachear 5 min
        self.end_headers()
        self.wfile.write(json.dumps(datos, ensure_ascii=False, indent=2).encode())
    
    def enviar_error(self, codigo, mensaje):
        self.send_response(codigo)
        self.send_header('Content-type', 'application/json')
        self.end_headers()
        self.wfile.write(json.dumps({'error': mensaje}).encode())

if __name__ == '__main__':
    servidor = HTTPServer(('localhost', 8000), GetHandler)
    print('🚀 Servidor GET en http://localhost:8000')
    servidor.serve_forever()
```

### Pruebas

```powershell
# Listar todos
curl http://localhost:8000/api/usuarios

# Obtener uno
curl http://localhost:8000/api/usuarios/1

# Filtrar activos
curl "http://localhost:8000/api/usuarios?activo=true"

# Buscar por nombre
curl "http://localhost:8000/api/usuarios?nombre=juan"

# Paginación
curl "http://localhost:8000/api/usuarios?limit=2&offset=0"
```

---

## POST - Crear recursos

**Propósito**: Crear nuevos recursos

### Características

- ❌ **No Idempotente**: Cada POST crea un recurso nuevo
- ❌ **No Safe**: Modifica el estado
- ✅ **Body**: Envía datos en el cuerpo
- ✅ **Response 201**: Created con el recurso creado

### Implementación

```python
class PostHandler(BaseHTTPRequestHandler):
    
    def do_POST(self):
        ruta = urlparse(self.path).path
        
        if ruta == '/api/usuarios':
            self.crear_usuario()
            return
        
        self.enviar_error(404, 'Ruta no encontrada')
    
    def crear_usuario(self):
        """POST /api/usuarios - Crear nuevo"""
        datos = self.leer_json_body()
        if not datos:
            self.enviar_error(400, 'Body JSON inválido')
            return
        
        # Validaciones
        errores = []
        
        if 'nombre' not in datos or not datos['nombre']:
            errores.append('Campo "nombre" requerido')
        
        if 'email' not in datos or not datos['email']:
            errores.append('Campo "email" requerido')
        
        if '@' not in datos.get('email', ''):
            errores.append('Email inválido')
        
        if errores:
            self.send_response(400)
            self.send_header('Content-type', 'application/json')
            self.end_headers()
            self.wfile.write(json.dumps({
                'error': 'Validación fallida',
                'detalles': errores
            }, ensure_ascii=False).encode())
            return
        
        # Crear usuario
        nuevo_id = max([u['id'] for u in usuarios], default=0) + 1
        nuevo_usuario = {
            'id': nuevo_id,
            'nombre': datos['nombre'],
            'email': datos['email'],
            'activo': datos.get('activo', True)
        }
        
        usuarios.append(nuevo_usuario)
        
        # Responder 201 Created con Location header
        self.send_response(201)
        self.send_header('Content-type', 'application/json')
        self.send_header('Location', f'/api/usuarios/{nuevo_id}')
        self.end_headers()
        self.wfile.write(json.dumps(nuevo_usuario, ensure_ascii=False, indent=2).encode())
    
    def leer_json_body(self):
        content_length = int(self.headers.get('Content-Length', 0))
        if content_length == 0:
            return None
        body_raw = self.rfile.read(content_length)
        try:
            return json.loads(body_raw.decode('utf-8'))
        except json.JSONDecodeError:
            return None
    
    def enviar_error(self, codigo, mensaje):
        self.send_response(codigo)
        self.send_header('Content-type', 'application/json')
        self.end_headers()
        self.wfile.write(json.dumps({'error': mensaje}).encode())
```

### Pruebas

```powershell
# Crear usuario
curl -X POST http://localhost:8000/api/usuarios `
  -H "Content-Type: application/json" `
  -d '{\"nombre\":\"Ana\",\"email\":\"ana@ejemplo.com\"}'

# Crear sin nombre (error)
curl -X POST http://localhost:8000/api/usuarios `
  -H "Content-Type: application/json" `
  -d '{\"email\":\"test@ejemplo.com\"}'

# Crear con email inválido
curl -X POST http://localhost:8000/api/usuarios `
  -H "Content-Type: application/json" `
  -d '{\"nombre\":\"Test\",\"email\":\"invalido\"}'
```

---

## PUT - Reemplazar completo

**Propósito**: Reemplazar un recurso **completo**

### Características

- ✅ **Idempotente**: Ejecutar varias veces da mismo resultado
- ❌ **No Safe**: Modifica datos
- ✅ **Body completo**: Debe enviar todos los campos
- ✅ **Response 200**: OK con recurso actualizado

### Implementación

```python
class PutHandler(BaseHTTPRequestHandler):
    
    def do_PUT(self):
        ruta = urlparse(self.path).path
        
        match = re.match(r'^/api/usuarios/(\d+)$', ruta)
        if match:
            user_id = int(match.group(1))
            self.reemplazar_usuario(user_id)
            return
        
        self.enviar_error(404, 'Ruta no encontrada')
    
    def reemplazar_usuario(self, user_id):
        """PUT /api/usuarios/:id - Reemplazar completo"""
        datos = self.leer_json_body()
        if not datos:
            self.enviar_error(400, 'Body JSON inválido')
            return
        
        # Buscar usuario
        usuario = next((u for u in usuarios if u['id'] == user_id), None)
        if not usuario:
            self.enviar_error(404, f'Usuario {user_id} no encontrado')
            return
        
        # PUT requiere TODOS los campos
        if 'nombre' not in datos or 'email' not in datos:
            self.enviar_error(400, 'PUT requiere campos: nombre, email')
            return
        
        # Reemplazar completamente
        usuario['nombre'] = datos['nombre']
        usuario['email'] = datos['email']
        usuario['activo'] = datos.get('activo', True)
        
        self.send_response(200)
        self.send_header('Content-type', 'application/json')
        self.end_headers()
        self.wfile.write(json.dumps(usuario, ensure_ascii=False, indent=2).encode())
```

### Pruebas

```powershell
# Reemplazar completo
curl -X PUT http://localhost:8000/api/usuarios/1 `
  -H "Content-Type: application/json" `
  -d '{\"nombre\":\"Juan Actualizado\",\"email\":\"juan.nuevo@ejemplo.com\",\"activo\":true}'

# Error: falta campo
curl -X PUT http://localhost:8000/api/usuarios/1 `
  -H "Content-Type: application/json" `
  -d '{\"nombre\":\"Solo nombre\"}'
```

---

## PATCH - Actualizar parcial

**Propósito**: Actualizar **solo algunos campos**

### Características

- ⚠️ **No necesariamente idempotente**: Depende de la implementación
- ❌ **No Safe**: Modifica datos
- ✅ **Body parcial**: Solo campos a modificar
- ✅ **Response 200**: OK con recurso actualizado

### Implementación

```python
class PatchHandler(BaseHTTPRequestHandler):
    
    def do_PATCH(self):
        ruta = urlparse(self.path).path
        
        match = re.match(r'^/api/usuarios/(\d+)$', ruta)
        if match:
            user_id = int(match.group(1))
            self.actualizar_parcial(user_id)
            return
        
        self.enviar_error(404, 'Ruta no encontrada')
    
    def actualizar_parcial(self, user_id):
        """PATCH /api/usuarios/:id - Actualizar parcial"""
        datos = self.leer_json_body()
        if not datos:
            self.enviar_error(400, 'Body JSON inválido')
            return
        
        # Buscar usuario
        usuario = next((u for u in usuarios if u['id'] == user_id), None)
        if not usuario:
            self.enviar_error(404, f'Usuario {user_id} no encontrado')
            return
        
        # Actualizar solo campos presentes
        if 'nombre' in datos:
            usuario['nombre'] = datos['nombre']
        
        if 'email' in datos:
            usuario['email'] = datos['email']
        
        if 'activo' in datos:
            usuario['activo'] = datos['activo']
        
        self.send_response(200)
        self.send_header('Content-type', 'application/json')
        self.end_headers()
        self.wfile.write(json.dumps(usuario, ensure_ascii=False, indent=2).encode())
```

### Pruebas

```powershell
# Actualizar solo nombre
curl -X PATCH http://localhost:8000/api/usuarios/1 `
  -H "Content-Type: application/json" `
  -d '{\"nombre\":\"Juan Modificado\"}'

# Actualizar solo activo
curl -X PATCH http://localhost:8000/api/usuarios/1 `
  -H "Content-Type: application/json" `
  -d '{\"activo\":false}'

# Actualizar varios campos
curl -X PATCH http://localhost:8000/api/usuarios/1 `
  -H "Content-Type: application/json" `
  -d '{\"nombre\":\"Juan\",\"email\":\"juan@nuevo.com\"}'
```

---

## DELETE - Eliminar recurso

**Propósito**: Eliminar un recurso

### Características

- ✅ **Idempotente**: Eliminar varias veces da mismo resultado
- ❌ **No Safe**: Modifica datos
- ❌ **No Body**: No envía datos
- ✅ **Response 204**: No Content (sin body)

### Implementación

```python
class DeleteHandler(BaseHTTPRequestHandler):
    
    def do_DELETE(self):
        ruta = urlparse(self.path).path
        
        match = re.match(r'^/api/usuarios/(\d+)$', ruta)
        if match:
            user_id = int(match.group(1))
            self.eliminar_usuario(user_id)
            return
        
        self.enviar_error(404, 'Ruta no encontrada')
    
    def eliminar_usuario(self, user_id):
        """DELETE /api/usuarios/:id"""
        global usuarios
        
        usuario = next((u for u in usuarios if u['id'] == user_id), None)
        if not usuario:
            self.enviar_error(404, f'Usuario {user_id} no encontrado')
            return
        
        usuarios = [u for u in usuarios if u['id'] != user_id]
        
        # 204 No Content (sin body)
        self.send_response(204)
        self.end_headers()
    
    def enviar_error(self, codigo, mensaje):
        self.send_response(codigo)
        self.send_header('Content-type', 'application/json')
        self.end_headers()
        self.wfile.write(json.dumps({'error': mensaje}).encode())
```

### Pruebas

```powershell
# Eliminar usuario
curl -X DELETE http://localhost:8000/api/usuarios/1

# Eliminar inexistente (404)
curl -X DELETE http://localhost:8000/api/usuarios/999
```

---

## Comparación PUT vs PATCH

### Escenario

Usuario actual:
```json
{
  "id": 1,
  "nombre": "Juan",
  "email": "juan@ejemplo.com",
  "activo": true
}
```

### Con PUT (reemplazar completo)

```powershell
curl -X PUT http://localhost:8000/api/usuarios/1 `
  -H "Content-Type: application/json" `
  -d '{\"nombre\":\"Juan Actualizado\"}'
```

**Resultado**: ❌ Error 400 - "PUT requiere campos: nombre, email"

**Correcto**:
```powershell
curl -X PUT http://localhost:8000/api/usuarios/1 `
  -H "Content-Type: application/json" `
  -d '{\"nombre\":\"Juan Actualizado\",\"email\":\"juan@ejemplo.com\",\"activo\":true}'
```

### Con PATCH (actualizar parcial)

```powershell
curl -X PATCH http://localhost:8000/api/usuarios/1 `
  -H "Content-Type: application/json" `
  -d '{\"nombre\":\"Juan Actualizado\"}'
```

**Resultado**: ✅ OK - Solo se actualiza el nombre

---

## API completa con todos los métodos

```python
from http.server import BaseHTTPRequestHandler, HTTPServer
from urllib.parse import urlparse, parse_qs
import json
import re

usuarios = [
    {'id': 1, 'nombre': 'Juan', 'email': 'juan@ejemplo.com', 'activo': True},
    {'id': 2, 'nombre': 'María', 'email': 'maria@ejemplo.com', 'activo': True}
]

siguiente_id = 3

class APIHandler(BaseHTTPRequestHandler):
    
    def do_GET(self):
        ruta = urlparse(self.path).path
        query = parse_qs(urlparse(self.path).query)
        
        match = re.match(r'^/api/usuarios/(\d+)$', ruta)
        if match:
            self.obtener_usuario(int(match.group(1)))
        elif ruta == '/api/usuarios':
            self.listar_usuarios(query)
        else:
            self.enviar_error(404, 'Ruta no encontrada')
    
    def do_POST(self):
        if urlparse(self.path).path == '/api/usuarios':
            self.crear_usuario()
        else:
            self.enviar_error(404, 'Ruta no encontrada')
    
    def do_PUT(self):
        match = re.match(r'^/api/usuarios/(\d+)$', urlparse(self.path).path)
        if match:
            self.reemplazar_usuario(int(match.group(1)))
        else:
            self.enviar_error(404, 'Ruta no encontrada')
    
    def do_PATCH(self):
        match = re.match(r'^/api/usuarios/(\d+)$', urlparse(self.path).path)
        if match:
            self.actualizar_parcial(int(match.group(1)))
        else:
            self.enviar_error(404, 'Ruta no encontrada')
    
    def do_DELETE(self):
        match = re.match(r'^/api/usuarios/(\d+)$', urlparse(self.path).path)
        if match:
            self.eliminar_usuario(int(match.group(1)))
        else:
            self.enviar_error(404, 'Ruta no encontrada')
    
    # Handlers (implementados arriba)
    def listar_usuarios(self, query):
        self.enviar_json(usuarios)
    
    def obtener_usuario(self, user_id):
        usuario = next((u for u in usuarios if u['id'] == user_id), None)
        if usuario:
            self.enviar_json(usuario)
        else:
            self.enviar_error(404, 'Usuario no encontrado')
    
    def crear_usuario(self):
        global siguiente_id
        datos = self.leer_json_body()
        if not datos or 'nombre' not in datos or 'email' not in datos:
            self.enviar_error(400, 'Campos nombre y email requeridos')
            return
        
        nuevo = {
            'id': siguiente_id,
            'nombre': datos['nombre'],
            'email': datos['email'],
            'activo': datos.get('activo', True)
        }
        usuarios.append(nuevo)
        siguiente_id += 1
        
        self.send_response(201)
        self.send_header('Content-type', 'application/json')
        self.send_header('Location', f'/api/usuarios/{nuevo["id"]}')
        self.end_headers()
        self.wfile.write(json.dumps(nuevo, ensure_ascii=False, indent=2).encode())
    
    def reemplazar_usuario(self, user_id):
        datos = self.leer_json_body()
        usuario = next((u for u in usuarios if u['id'] == user_id), None)
        
        if not usuario:
            self.enviar_error(404, 'Usuario no encontrado')
            return
        
        if not datos or 'nombre' not in datos or 'email' not in datos:
            self.enviar_error(400, 'PUT requiere nombre y email')
            return
        
        usuario.update({
            'nombre': datos['nombre'],
            'email': datos['email'],
            'activo': datos.get('activo', True)
        })
        self.enviar_json(usuario)
    
    def actualizar_parcial(self, user_id):
        datos = self.leer_json_body()
        usuario = next((u for u in usuarios if u['id'] == user_id), None)
        
        if not usuario:
            self.enviar_error(404, 'Usuario no encontrado')
            return
        
        if 'nombre' in datos:
            usuario['nombre'] = datos['nombre']
        if 'email' in datos:
            usuario['email'] = datos['email']
        if 'activo' in datos:
            usuario['activo'] = datos['activo']
        
        self.enviar_json(usuario)
    
    def eliminar_usuario(self, user_id):
        global usuarios
        usuario = next((u for u in usuarios if u['id'] == user_id), None)
        
        if not usuario:
            self.enviar_error(404, 'Usuario no encontrado')
            return
        
        usuarios = [u for u in usuarios if u['id'] != user_id]
        self.send_response(204)
        self.end_headers()
    
    # Helpers
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
        self.wfile.write(json.dumps({'error': mensaje}).encode())
    
    def leer_json_body(self):
        content_length = int(self.headers.get('Content-Length', 0))
        if content_length == 0:
            return None
        body_raw = self.rfile.read(content_length)
        try:
            return json.loads(body_raw.decode('utf-8'))
        except:
            return None
    
    def log_message(self, format, *args):
        print(f'📥 {self.command} {self.path}')

if __name__ == '__main__':
    servidor = HTTPServer(('localhost', 8000), APIHandler)
    print('🚀 API REST completa en http://localhost:8000/api/usuarios')
    print('\nEndpoints disponibles:')
    print('  GET    /api/usuarios       - Listar')
    print('  GET    /api/usuarios/:id   - Obtener')
    print('  POST   /api/usuarios       - Crear')
    print('  PUT    /api/usuarios/:id   - Reemplazar')
    print('  PATCH  /api/usuarios/:id   - Actualizar')
    print('  DELETE /api/usuarios/:id   - Eliminar\n')
    
    try:
        servidor.serve_forever()
    except KeyboardInterrupt:
        print('\n⛔ Servidor detenido')
        servidor.shutdown()
```

---

## Resumen

| Método | Uso | Idempotente | Body | Status |
|--------|-----|-------------|------|--------|
| GET | Leer | ✅ | ❌ | 200 |
| POST | Crear | ❌ | ✅ | 201 |
| PUT | Reemplazar | ✅ | ✅ (completo) | 200 |
| PATCH | Actualizar | ⚠️ | ✅ (parcial) | 200 |
| DELETE | Eliminar | ✅ | ❌ | 204 |

**Siguiente:** [Sistema de enrutamiento avanzado](./08-sistema-enrutamiento.md)

---

## Recursos

- **[RFC 7231](https://tools.ietf.org/html/rfc7231)** - HTTP Methods
- **[RESTful API Design](https://restfulapi.net/http-methods/)** - HTTP Methods Guide

Domina los métodos HTTP para construir APIs REST profesionales. 🎯

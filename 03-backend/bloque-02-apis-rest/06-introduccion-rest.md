# Introducción a REST

> **Arquitectura REST - Principios y mejores prácticas**

---

## ¿Qué es REST?

**REST** = **RE**presentational **S**tate **T**ransfer

Es un **estilo arquitectónico** para diseñar APIs web, no un protocolo ni un estándar.

### Propuesto por Roy Fielding (2000)

En su tesis doctoral, Fielding definió 6 restricciones para APIs REST:

1. **Cliente-Servidor**: Separación de responsabilidades
2. **Stateless**: Cada request es independiente
3. **Cacheable**: Respuestas deben indicar si son cacheables
4. **Interfaz Uniforme**: Mismos patrones para todos los recursos
5. **Sistema en Capas**: Arquitectura multicapa
6. **Código bajo demanda** (opcional): El servidor puede enviar código ejecutable

---

## Conceptos fundamentales

### 1. Recursos

Todo es un **recurso** identificado por una URI:

```
/usuarios           → Colección de usuarios
/usuarios/123       → Usuario específico (ID: 123)
/productos          → Colección de productos
/productos/456      → Producto específico (ID: 456)
/usuarios/123/posts → Posts del usuario 123
```

### 2. Representaciones

Los recursos se representan en diferentes formatos:

```json
// JSON (más común)
{
  "id": 123,
  "nombre": "Juan",
  "email": "juan@ejemplo.com"
}
```

```xml
<!-- XML -->
<usuario>
  <id>123</id>
  <nombre>Juan</nombre>
  <email>juan@ejemplo.com</email>
</usuario>
```

### 3. Métodos HTTP

Operaciones sobre recursos usando verbos HTTP:

| Método | Acción | Ejemplo |
|--------|--------|---------|
| **GET** | Leer | `GET /usuarios` → Listar usuarios |
| **POST** | Crear | `POST /usuarios` → Crear usuario |
| **PUT** | Actualizar (completo) | `PUT /usuarios/123` → Reemplazar usuario |
| **PATCH** | Actualizar (parcial) | `PATCH /usuarios/123` → Modificar campos |
| **DELETE** | Eliminar | `DELETE /usuarios/123` → Eliminar usuario |

### 4. Stateless

Cada request debe contener toda la información necesaria:

```http
GET /api/usuarios/123
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
Accept: application/json
```

El servidor **NO guarda sesiones**. El cliente envía el token en cada request.

---

## Diseño de URLs REST

### ✅ Buenas prácticas

```
GET    /usuarios                 → Listar todos
GET    /usuarios/123             → Obtener uno
POST   /usuarios                 → Crear
PUT    /usuarios/123             → Actualizar completo
PATCH  /usuarios/123             → Actualizar parcial
DELETE /usuarios/123             → Eliminar

GET    /usuarios/123/posts       → Posts del usuario 123
GET    /posts?usuario_id=123     → Alternativa con query params
GET    /usuarios?activo=true     → Filtrar usuarios activos
GET    /usuarios?page=2&limit=10 → Paginación
```

### ❌ Malas prácticas

```
GET  /obtenerUsuarios             → Verbo innecesario
GET  /usuario/obtener/123         → Verbo en URL
POST /crearUsuario                → Verbo innecesario
GET  /usuarios/eliminar/123       → Usar DELETE, no GET
POST /usuarios/123/actualizar     → Usar PUT/PATCH
GET  /api?action=getUser&id=123   → No REST
```

### Principios de diseño

1. **Sustantivos, no verbos**: `/usuarios` en vez de `/obtenerUsuarios`
2. **Plural para colecciones**: `/usuarios` en vez de `/usuario`
3. **Jerárquico para relaciones**: `/usuarios/123/posts`
4. **Minúsculas**: `/usuarios` en vez de `/Usuarios`
5. **Guiones para separar**: `/tarjetas-credito` en vez de `/tarjetasCredito`

---

## CRUD con REST

**CRUD** = Create, Read, Update, Delete

```
Acción    Método   URL              Body              Response
--------- -------- ---------------- ----------------- ----------
Listar    GET      /usuarios        -                 200 + []
Obtener   GET      /usuarios/123    -                 200 + {}
Crear     POST     /usuarios        {nombre, email}   201 + {}
Actualizar PUT     /usuarios/123    {nombre, email}   200 + {}
Actualizar PATCH   /usuarios/123    {nombre}          200 + {}
Eliminar  DELETE   /usuarios/123    -                 204
```

---

## Ejemplo práctico: API de tareas

### Diseño de endpoints

```
GET    /api/tareas              → Listar todas las tareas
GET    /api/tareas/123          → Obtener tarea específica
POST   /api/tareas              → Crear nueva tarea
PUT    /api/tareas/123          → Actualizar tarea completa
PATCH  /api/tareas/123          → Actualizar solo completada
DELETE /api/tareas/123          → Eliminar tarea

GET    /api/tareas?completada=true      → Filtrar completadas
GET    /api/tareas?ordenar=fecha_desc   → Ordenar por fecha
GET    /api/tareas?buscar=comprar       → Buscar en título
```

### Implementación con Python vanilla

```python
from http.server import BaseHTTPRequestHandler, HTTPServer
from urllib.parse import urlparse, parse_qs
import json
import re

# Base de datos simulada
tareas = [
    {'id': 1, 'titulo': 'Comprar leche', 'completada': False},
    {'id': 2, 'titulo': 'Estudiar REST', 'completada': True},
    {'id': 3, 'titulo': 'Hacer ejercicio', 'completada': False}
]

siguiente_id = 4

class TareasAPI(BaseHTTPRequestHandler):
    
    def do_GET(self):
        """Listar o obtener tarea"""
        ruta = urlparse(self.path).path
        query = parse_qs(urlparse(self.path).query)
        
        # GET /api/tareas/:id
        match_detalle = re.match(r'^/api/tareas/(\d+)$', ruta)
        if match_detalle:
            tarea_id = int(match_detalle.group(1))
            self.obtener_tarea(tarea_id)
            return
        
        # GET /api/tareas
        if ruta == '/api/tareas':
            self.listar_tareas(query)
            return
        
        self.enviar_error(404, 'Ruta no encontrada')
    
    def do_POST(self):
        """Crear tarea"""
        ruta = urlparse(self.path).path
        
        if ruta == '/api/tareas':
            self.crear_tarea()
            return
        
        self.enviar_error(404, 'Ruta no encontrada')
    
    def do_PUT(self):
        """Actualizar tarea completa"""
        ruta = urlparse(self.path).path
        
        match = re.match(r'^/api/tareas/(\d+)$', ruta)
        if match:
            tarea_id = int(match.group(1))
            self.actualizar_tarea_completa(tarea_id)
            return
        
        self.enviar_error(404, 'Ruta no encontrada')
    
    def do_PATCH(self):
        """Actualizar tarea parcial"""
        ruta = urlparse(self.path).path
        
        match = re.match(r'^/api/tareas/(\d+)$', ruta)
        if match:
            tarea_id = int(match.group(1))
            self.actualizar_tarea_parcial(tarea_id)
            return
        
        self.enviar_error(404, 'Ruta no encontrada')
    
    def do_DELETE(self):
        """Eliminar tarea"""
        ruta = urlparse(self.path).path
        
        match = re.match(r'^/api/tareas/(\d+)$', ruta)
        if match:
            tarea_id = int(match.group(1))
            self.eliminar_tarea(tarea_id)
            return
        
        self.enviar_error(404, 'Ruta no encontrada')
    
    # ===== Handlers =====
    
    def listar_tareas(self, query):
        """GET /api/tareas - Con filtros opcionales"""
        resultado = tareas.copy()
        
        # Filtro: completada
        if 'completada' in query:
            valor = query['completada'][0].lower() == 'true'
            resultado = [t for t in resultado if t['completada'] == valor]
        
        # Filtro: buscar en título
        if 'buscar' in query:
            termino = query['buscar'][0].lower()
            resultado = [t for t in resultado if termino in t['titulo'].lower()]
        
        # Ordenar
        if 'ordenar' in query:
            orden = query['ordenar'][0]
            if orden == 'titulo_asc':
                resultado.sort(key=lambda t: t['titulo'])
            elif orden == 'titulo_desc':
                resultado.sort(key=lambda t: t['titulo'], reverse=True)
        
        self.enviar_json(resultado)
    
    def obtener_tarea(self, tarea_id):
        """GET /api/tareas/:id"""
        tarea = next((t for t in tareas if t['id'] == tarea_id), None)
        
        if tarea:
            self.enviar_json(tarea)
        else:
            self.enviar_error(404, f'Tarea {tarea_id} no encontrada')
    
    def crear_tarea(self):
        """POST /api/tareas"""
        global siguiente_id
        
        datos = self.leer_json_body()
        if not datos:
            self.enviar_error(400, 'Body JSON inválido')
            return
        
        # Validar
        if 'titulo' not in datos:
            self.enviar_error(400, 'Campo "titulo" requerido')
            return
        
        # Crear
        nueva_tarea = {
            'id': siguiente_id,
            'titulo': datos['titulo'],
            'completada': datos.get('completada', False)
        }
        
        tareas.append(nueva_tarea)
        siguiente_id += 1
        
        # Responder con 201 Created
        self.enviar_json(nueva_tarea, 201)
    
    def actualizar_tarea_completa(self, tarea_id):
        """PUT /api/tareas/:id - Reemplaza toda la tarea"""
        datos = self.leer_json_body()
        if not datos:
            self.enviar_error(400, 'Body JSON inválido')
            return
        
        # Buscar tarea
        tarea = next((t for t in tareas if t['id'] == tarea_id), None)
        if not tarea:
            self.enviar_error(404, f'Tarea {tarea_id} no encontrada')
            return
        
        # Validar campos requeridos
        if 'titulo' not in datos:
            self.enviar_error(400, 'PUT requiere "titulo"')
            return
        
        # Actualizar (reemplazar completamente)
        tarea['titulo'] = datos['titulo']
        tarea['completada'] = datos.get('completada', False)
        
        self.enviar_json(tarea)
    
    def actualizar_tarea_parcial(self, tarea_id):
        """PATCH /api/tareas/:id - Actualiza solo campos enviados"""
        datos = self.leer_json_body()
        if not datos:
            self.enviar_error(400, 'Body JSON inválido')
            return
        
        # Buscar tarea
        tarea = next((t for t in tareas if t['id'] == tarea_id), None)
        if not tarea:
            self.enviar_error(404, f'Tarea {tarea_id} no encontrada')
            return
        
        # Actualizar solo campos presentes
        if 'titulo' in datos:
            tarea['titulo'] = datos['titulo']
        if 'completada' in datos:
            tarea['completada'] = datos['completada']
        
        self.enviar_json(tarea)
    
    def eliminar_tarea(self, tarea_id):
        """DELETE /api/tareas/:id"""
        global tareas
        
        tarea = next((t for t in tareas if t['id'] == tarea_id), None)
        if not tarea:
            self.enviar_error(404, f'Tarea {tarea_id} no encontrada')
            return
        
        tareas = [t for t in tareas if t['id'] != tarea_id]
        
        # 204 No Content (sin body)
        self.send_response(204)
        self.end_headers()
    
    # ===== Helpers =====
    
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
    
    def leer_json_body(self):
        content_length = int(self.headers.get('Content-Length', 0))
        if content_length == 0:
            return None
        body_raw = self.rfile.read(content_length)
        try:
            return json.loads(body_raw.decode('utf-8'))
        except json.JSONDecodeError:
            return None
    
    def log_message(self, format, *args):
        print(f'📥 {self.command} {self.path}')

if __name__ == '__main__':
    servidor = HTTPServer(('localhost', 8000), TareasAPI)
    print('🚀 API REST de Tareas en http://localhost:8000/api/tareas')
    
    try:
        servidor.serve_forever()
    except KeyboardInterrupt:
        print('\n⛔ Servidor detenido')
        servidor.shutdown()
```

---

## Pruebas con curl

```powershell
# Listar todas
curl http://localhost:8000/api/tareas

# Obtener una
curl http://localhost:8000/api/tareas/1

# Crear
curl -X POST http://localhost:8000/api/tareas `
  -H "Content-Type: application/json" `
  -d '{\"titulo\":\"Leer libro\",\"completada\":false}'

# Actualizar completa (PUT)
curl -X PUT http://localhost:8000/api/tareas/1 `
  -H "Content-Type: application/json" `
  -d '{\"titulo\":\"Comprar pan\",\"completada\":true}'

# Actualizar parcial (PATCH)
curl -X PATCH http://localhost:8000/api/tareas/1 `
  -H "Content-Type: application/json" `
  -d '{\"completada\":true}'

# Eliminar
curl -X DELETE http://localhost:8000/api/tareas/1

# Filtrar completadas
curl "http://localhost:8000/api/tareas?completada=true"

# Buscar
curl "http://localhost:8000/api/tareas?buscar=leche"

# Ordenar
curl "http://localhost:8000/api/tareas?ordenar=titulo_asc"
```

---

## Status Codes REST

| Código | Significado | Uso |
|--------|-------------|-----|
| **200** | OK | GET, PUT, PATCH exitosos |
| **201** | Created | POST exitoso |
| **204** | No Content | DELETE exitoso |
| **400** | Bad Request | Datos inválidos |
| **404** | Not Found | Recurso no existe |
| **500** | Internal Server Error | Error del servidor |

---

## Hipermedia (HATEOAS)

**HATEOAS** = Hypermedia As The Engine Of Application State

La respuesta incluye **links** a recursos relacionados:

```json
{
  "id": 123,
  "nombre": "Juan",
  "email": "juan@ejemplo.com",
  "_links": {
    "self": "/api/usuarios/123",
    "posts": "/api/usuarios/123/posts",
    "editar": "/api/usuarios/123",
    "eliminar": "/api/usuarios/123"
  }
}
```

El cliente **descubre** las acciones disponibles desde las respuestas.

---

## REST vs GraphQL vs gRPC

| Característica | REST | GraphQL | gRPC |
|----------------|------|---------|------|
| **Protocolo** | HTTP | HTTP | HTTP/2 |
| **Formato** | JSON, XML | JSON | Protobuf (binario) |
| **Endpoints** | Múltiples (/usuarios, /posts) | Uno (/graphql) | Servicios definidos |
| **Overfetching** | Sí | No | No |
| **Caching** | Nativo HTTP | Manual | Manual |
| **Tipado** | Opcional | Fuerte | Fuerte |
| **Adopción** | +++++ | ++++ | +++ |

---

## Ejercicios

### 1. API de productos
Diseña endpoints REST para:
- Listar productos
- Crear producto
- Actualizar stock
- Eliminar producto
- Filtrar por categoría
- Buscar por nombre

### 2. Relaciones anidadas
```
GET /api/usuarios/123/posts
GET /api/posts/456/comentarios
```

### 3. Paginación
```
GET /api/usuarios?page=1&limit=20
Response:
{
  "data": [...],
  "meta": {
    "page": 1,
    "limit": 20,
    "total": 150
  }
}
```

---

## Resumen

Has aprendido:

✅ Principios REST fundamentales  
✅ Diseño de URLs REST  
✅ CRUD con métodos HTTP  
✅ Implementación con Python vanilla  
✅ Diferencia PUT vs PATCH  
✅ Status codes apropiados  
✅ Filtrado y búsqueda

**Siguiente:** [Métodos HTTP en profundidad](./07-metodos-http.md)

---

## Recursos

- **[REST API Tutorial](https://restfulapi.net/)** - Guía completa REST
- **[Roy Fielding Thesis](https://www.ics.uci.edu/~fielding/pubs/dissertation/rest_arch_style.htm)** - Tesis original
- **[HTTP Status Codes](https://httpstatuses.com/)** - Lista completa de códigos

REST es el estándar de facto para APIs web. 🌐

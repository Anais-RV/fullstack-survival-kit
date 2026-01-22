# Rutas y Enrutamiento

> **Sistema de rutas profesional con Python vanilla**

---

## ¿Qué es el enrutamiento?

**Enrutamiento** es mapear URLs a funciones que las manejan:

```
GET /                    → página_inicio()
GET /api/usuarios        → listar_usuarios()
POST /api/usuarios       → crear_usuario()
GET /api/usuarios/123    → obtener_usuario(123)
PUT /api/usuarios/123    → actualizar_usuario(123)
DELETE /api/usuarios/123 → eliminar_usuario(123)
```

---

## Enrutamiento básico manual

### Opción 1: if/elif

```python
from http.server import BaseHTTPRequestHandler
from urllib.parse import urlparse
import json

class Router(BaseHTTPRequestHandler):
    def do_GET(self):
        ruta = urlparse(self.path).path
        
        if ruta == '/':
            self.pagina_inicio()
        elif ruta == '/api/usuarios':
            self.listar_usuarios()
        elif ruta.startswith('/api/usuarios/'):
            # Extraer ID de la ruta
            user_id = ruta.split('/')[-1]
            self.obtener_usuario(user_id)
        else:
            self.not_found()
    
    def pagina_inicio(self):
        self.send_response(200)
        self.send_header('Content-type', 'text/html')
        self.end_headers()
        self.wfile.write(b'<h1>Bienvenido</h1>')
    
    def listar_usuarios(self):
        usuarios = [
            {'id': 1, 'nombre': 'Juan'},
            {'id': 2, 'nombre': 'María'}
        ]
        self.enviar_json(usuarios)
    
    def obtener_usuario(self, user_id):
        usuario = {'id': user_id, 'nombre': 'Usuario ' + user_id}
        self.enviar_json(usuario)
    
    def not_found(self):
        self.send_response(404)
        self.send_header('Content-type', 'application/json')
        self.end_headers()
        error = {'error': 'Ruta no encontrada'}
        self.wfile.write(json.dumps(error).encode())
    
    def enviar_json(self, datos):
        self.send_response(200)
        self.send_header('Content-type', 'application/json')
        self.end_headers()
        self.wfile.write(json.dumps(datos, ensure_ascii=False).encode())
```

---

## Router basado en diccionario

Más escalable y mantenible:

```python
import re
from http.server import BaseHTTPRequestHandler, HTTPServer
from urllib.parse import urlparse
import json

class SmartRouter(BaseHTTPRequestHandler):
    """Router con tabla de rutas"""
    
    # Tabla de rutas: {patron: handler}
    RUTAS_GET = {
        r'^/$': 'home',
        r'^/api/usuarios$': 'listar_usuarios',
        r'^/api/usuarios/(\d+)$': 'obtener_usuario',
        r'^/api/productos$': 'listar_productos',
        r'^/api/productos/(\d+)$': 'obtener_producto'
    }
    
    RUTAS_POST = {
        r'^/api/usuarios$': 'crear_usuario',
        r'^/api/productos$': 'crear_producto'
    }
    
    RUTAS_PUT = {
        r'^/api/usuarios/(\d+)$': 'actualizar_usuario',
        r'^/api/productos/(\d+)$': 'actualizar_producto'
    }
    
    RUTAS_DELETE = {
        r'^/api/usuarios/(\d+)$': 'eliminar_usuario',
        r'^/api/productos/(\d+)$': 'eliminar_producto'
    }
    
    def do_GET(self):
        self.procesar_request('GET', self.RUTAS_GET)
    
    def do_POST(self):
        self.procesar_request('POST', self.RUTAS_POST)
    
    def do_PUT(self):
        self.procesar_request('PUT', self.RUTAS_PUT)
    
    def do_DELETE(self):
        self.procesar_request('DELETE', self.RUTAS_DELETE)
    
    def procesar_request(self, metodo, tabla_rutas):
        """Procesa request usando tabla de rutas"""
        ruta = urlparse(self.path).path
        
        # Buscar match en tabla de rutas
        for patron, handler_name in tabla_rutas.items():
            match = re.match(patron, ruta)
            if match:
                # Obtener handler
                handler = getattr(self, handler_name, None)
                if handler:
                    # Llamar handler con parámetros capturados
                    parametros = match.groups()
                    handler(*parametros)
                    return
        
        # No se encontró ruta
        self.not_found()
    
    # ===== Handlers =====
    
    def home(self):
        """Página de inicio"""
        html = """
        <html>
        <head><title>API REST</title></head>
        <body>
            <h1>API REST - Python Vanilla</h1>
            <h2>Endpoints disponibles:</h2>
            <ul>
                <li>GET /api/usuarios - Listar usuarios</li>
                <li>GET /api/usuarios/123 - Obtener usuario</li>
                <li>POST /api/usuarios - Crear usuario</li>
                <li>PUT /api/usuarios/123 - Actualizar usuario</li>
                <li>DELETE /api/usuarios/123 - Eliminar usuario</li>
            </ul>
        </body>
        </html>
        """
        self.send_response(200)
        self.send_header('Content-type', 'text/html; charset=utf-8')
        self.end_headers()
        self.wfile.write(html.encode())
    
    def listar_usuarios(self):
        """GET /api/usuarios"""
        usuarios = [
            {'id': 1, 'nombre': 'Juan', 'email': 'juan@ejemplo.com'},
            {'id': 2, 'nombre': 'María', 'email': 'maria@ejemplo.com'},
            {'id': 3, 'nombre': 'Pedro', 'email': 'pedro@ejemplo.com'}
        ]
        self.enviar_json(usuarios)
    
    def obtener_usuario(self, user_id):
        """GET /api/usuarios/:id"""
        usuario = {
            'id': int(user_id),
            'nombre': f'Usuario {user_id}',
            'email': f'usuario{user_id}@ejemplo.com'
        }
        self.enviar_json(usuario)
    
    def crear_usuario(self):
        """POST /api/usuarios"""
        datos = self.leer_json_body()
        if not datos:
            self.enviar_error(400, 'Datos inválidos')
            return
        
        nuevo_usuario = {
            'id': 999,  # Simulado
            'nombre': datos.get('nombre'),
            'email': datos.get('email'),
            'creado': True
        }
        
        self.send_response(201)
        self.send_header('Content-type', 'application/json')
        self.end_headers()
        self.wfile.write(json.dumps(nuevo_usuario, ensure_ascii=False).encode())
    
    def actualizar_usuario(self, user_id):
        """PUT /api/usuarios/:id"""
        datos = self.leer_json_body()
        if not datos:
            self.enviar_error(400, 'Datos inválidos')
            return
        
        usuario_actualizado = {
            'id': int(user_id),
            'nombre': datos.get('nombre'),
            'email': datos.get('email'),
            'actualizado': True
        }
        self.enviar_json(usuario_actualizado)
    
    def eliminar_usuario(self, user_id):
        """DELETE /api/usuarios/:id"""
        self.enviar_json({
            'mensaje': f'Usuario {user_id} eliminado',
            'id': int(user_id)
        })
    
    def listar_productos(self):
        """GET /api/productos"""
        productos = [
            {'id': 1, 'nombre': 'Laptop', 'precio': 1200},
            {'id': 2, 'nombre': 'Mouse', 'precio': 25}
        ]
        self.enviar_json(productos)
    
    def obtener_producto(self, producto_id):
        """GET /api/productos/:id"""
        producto = {
            'id': int(producto_id),
            'nombre': f'Producto {producto_id}',
            'precio': 99.99
        }
        self.enviar_json(producto)
    
    def crear_producto(self):
        """POST /api/productos"""
        datos = self.leer_json_body()
        nuevo_producto = {
            'id': 999,
            **datos,
            'creado': True
        }
        self.send_response(201)
        self.send_header('Content-type', 'application/json')
        self.end_headers()
        self.wfile.write(json.dumps(nuevo_producto, ensure_ascii=False).encode())
    
    def actualizar_producto(self, producto_id):
        """PUT /api/productos/:id"""
        datos = self.leer_json_body()
        producto = {
            'id': int(producto_id),
            **datos,
            'actualizado': True
        }
        self.enviar_json(producto)
    
    def eliminar_producto(self, producto_id):
        """DELETE /api/productos/:id"""
        self.enviar_json({
            'mensaje': f'Producto {producto_id} eliminado',
            'id': int(producto_id)
        })
    
    # ===== Helpers =====
    
    def enviar_json(self, datos, status=200):
        """Envía respuesta JSON"""
        self.send_response(status)
        self.send_header('Content-type', 'application/json')
        self.send_header('Access-Control-Allow-Origin', '*')
        self.end_headers()
        self.wfile.write(json.dumps(datos, ensure_ascii=False, indent=2).encode())
    
    def enviar_error(self, codigo, mensaje):
        """Envía error JSON"""
        self.send_response(codigo)
        self.send_header('Content-type', 'application/json')
        self.end_headers()
        error = {'error': mensaje, 'codigo': codigo}
        self.wfile.write(json.dumps(error).encode())
    
    def leer_json_body(self):
        """Lee y parsea JSON del body"""
        content_length = int(self.headers.get('Content-Length', 0))
        if content_length == 0:
            return None
        
        body_raw = self.rfile.read(content_length)
        try:
            return json.loads(body_raw.decode('utf-8'))
        except json.JSONDecodeError:
            return None
    
    def not_found(self):
        """404 Not Found"""
        self.enviar_error(404, f'Ruta no encontrada: {self.command} {self.path}')
    
    def log_message(self, format, *args):
        """Log personalizado"""
        print(f'📥 {self.command} {self.path} - {args[1]}')

# Servidor
if __name__ == '__main__':
    servidor = HTTPServer(('localhost', 8000), SmartRouter)
    print('🚀 Router inteligente en http://localhost:8000')
    print('Visita http://localhost:8000 para ver todos los endpoints')
    
    try:
        servidor.serve_forever()
    except KeyboardInterrupt:
        print('\n⛔ Servidor detenido')
        servidor.shutdown()
```

---

## Router con clases separadas

Para proyectos más grandes:

**router.py**
```python
"""
Sistema de enrutamiento profesional
"""

import re
from typing import Callable, Dict, Tuple, Optional

class Route:
    """Representa una ruta individual"""
    
    def __init__(self, patron: str, handler: Callable, metodo: str):
        self.patron = re.compile(patron)
        self.handler = handler
        self.metodo = metodo
    
    def match(self, ruta: str, metodo: str) -> Optional[Tuple]:
        """
        Verifica si la ruta coincide
        
        Returns:
            Tuple de parámetros capturados o None
        """
        if self.metodo != metodo:
            return None
        
        match = self.patron.match(ruta)
        if match:
            return match.groups()
        return None

class Router:
    """Sistema de enrutamiento"""
    
    def __init__(self):
        self.routes = []
    
    def add_route(self, metodo: str, patron: str, handler: Callable):
        """Agrega una ruta"""
        route = Route(patron, handler, metodo)
        self.routes.append(route)
    
    def get(self, patron: str):
        """Decorador para rutas GET"""
        def decorator(handler):
            self.add_route('GET', patron, handler)
            return handler
        return decorator
    
    def post(self, patron: str):
        """Decorador para rutas POST"""
        def decorator(handler):
            self.add_route('POST', patron, handler)
            return handler
        return decorator
    
    def put(self, patron: str):
        """Decorador para rutas PUT"""
        def decorator(handler):
            self.add_route('PUT', patron, handler)
            return handler
        return decorator
    
    def delete(self, patron: str):
        """Decorador para rutas DELETE"""
        def decorator(handler):
            self.add_route('DELETE', patron, handler)
            return handler
        return decorator
    
    def route(self, ruta: str, metodo: str) -> Optional[Tuple[Callable, Tuple]]:
        """
        Encuentra handler para ruta y método
        
        Returns:
            (handler, parametros) o None
        """
        for route in self.routes:
            params = route.match(ruta, metodo)
            if params is not None:
                return (route.handler, params)
        return None
```

**app.py**
```python
"""
Aplicación usando el Router
"""

from http.server import BaseHTTPRequestHandler, HTTPServer
from urllib.parse import urlparse
import json
from router import Router

# Crear router
router = Router()

# ===== Definir rutas con decoradores =====

@router.get(r'^/$')
def home(request):
    """Página de inicio"""
    html = '<h1>API REST</h1><p>Bienvenido</p>'
    request.send_response(200)
    request.send_header('Content-type', 'text/html; charset=utf-8')
    request.end_headers()
    request.wfile.write(html.encode())

@router.get(r'^/api/usuarios$')
def listar_usuarios(request):
    """GET /api/usuarios"""
    usuarios = [
        {'id': 1, 'nombre': 'Juan'},
        {'id': 2, 'nombre': 'María'}
    ]
    request.enviar_json(usuarios)

@router.get(r'^/api/usuarios/(\d+)$')
def obtener_usuario(request, user_id):
    """GET /api/usuarios/:id"""
    usuario = {'id': int(user_id), 'nombre': f'Usuario {user_id}'}
    request.enviar_json(usuario)

@router.post(r'^/api/usuarios$')
def crear_usuario(request):
    """POST /api/usuarios"""
    datos = request.leer_json_body()
    nuevo = {'id': 999, **datos, 'creado': True}
    request.enviar_json(nuevo, 201)

@router.put(r'^/api/usuarios/(\d+)$')
def actualizar_usuario(request, user_id):
    """PUT /api/usuarios/:id"""
    datos = request.leer_json_body()
    usuario = {'id': int(user_id), **datos, 'actualizado': True}
    request.enviar_json(usuario)

@router.delete(r'^/api/usuarios/(\d+)$')
def eliminar_usuario(request, user_id):
    """DELETE /api/usuarios/:id"""
    request.enviar_json({'mensaje': f'Usuario {user_id} eliminado'})

# ===== Request Handler =====

class AppHandler(BaseHTTPRequestHandler):
    """Handler que usa el router"""
    
    def do_GET(self):
        self.procesar()
    
    def do_POST(self):
        self.procesar()
    
    def do_PUT(self):
        self.procesar()
    
    def do_DELETE(self):
        self.procesar()
    
    def procesar(self):
        """Procesa el request usando el router"""
        ruta = urlparse(self.path).path
        metodo = self.command
        
        # Buscar handler
        resultado = router.route(ruta, metodo)
        
        if resultado:
            handler, params = resultado
            # Llamar handler
            handler(self, *params)
        else:
            # 404
            self.enviar_error(404, 'Ruta no encontrada')
    
    def enviar_json(self, datos, status=200):
        """Helper: envía JSON"""
        self.send_response(status)
        self.send_header('Content-type', 'application/json')
        self.end_headers()
        self.wfile.write(json.dumps(datos, ensure_ascii=False, indent=2).encode())
    
    def enviar_error(self, codigo, mensaje):
        """Helper: envía error"""
        self.send_response(codigo)
        self.send_header('Content-type', 'application/json')
        self.end_headers()
        self.wfile.write(json.dumps({'error': mensaje}).encode())
    
    def leer_json_body(self):
        """Helper: lee JSON del body"""
        content_length = int(self.headers.get('Content-Length', 0))
        if content_length == 0:
            return {}
        body_raw = self.rfile.read(content_length)
        try:
            return json.loads(body_raw.decode('utf-8'))
        except:
            return None
    
    def log_message(self, format, *args):
        print(f'📥 {self.command} {self.path}')

# ===== Servidor =====

if __name__ == '__main__':
    servidor = HTTPServer(('localhost', 8000), AppHandler)
    print('🚀 Servidor con Router avanzado en http://localhost:8000')
    
    try:
        servidor.serve_forever()
    except KeyboardInterrupt:
        print('\n⛔ Servidor detenido')
        servidor.shutdown()
```

---

## Middleware pattern

Ejecutar funciones antes/después del handler:

```python
class MiddlewareRouter(Router):
    """Router con soporte de middleware"""
    
    def __init__(self):
        super().__init__()
        self.middleware_before = []
        self.middleware_after = []
    
    def use_before(self, func):
        """Agrega middleware que se ejecuta ANTES"""
        self.middleware_before.append(func)
    
    def use_after(self, func):
        """Agrega middleware que se ejecuta DESPUÉS"""
        self.middleware_after.append(func)
    
    def execute_route(self, request, ruta, metodo):
        """Ejecuta ruta con middleware"""
        # Ejecutar middleware before
        for middleware in self.middleware_before:
            result = middleware(request)
            if result is False:  # Middleware canceló el request
                return
        
        # Buscar y ejecutar handler
        resultado = self.route(ruta, metodo)
        if resultado:
            handler, params = resultado
            handler(request, *params)
        else:
            request.enviar_error(404, 'Ruta no encontrada')
        
        # Ejecutar middleware after
        for middleware in self.middleware_after:
            middleware(request)

# Ejemplo de uso:
router = MiddlewareRouter()

# Middleware de logging
def log_middleware(request):
    print(f'📥 {request.command} {request.path}')
    return True  # Continuar

# Middleware de autenticación
def auth_middleware(request):
    token = request.headers.get('Authorization')
    if not token:
        request.enviar_error(401, 'No autorizado')
        return False  # Cancelar request
    return True  # Continuar

router.use_before(log_middleware)
router.use_before(auth_middleware)
```

---

## Pruebas

```powershell
# GET
curl http://localhost:8000/api/usuarios

# GET con ID
curl http://localhost:8000/api/usuarios/123

# POST
curl -X POST http://localhost:8000/api/usuarios `
  -H "Content-Type: application/json" `
  -d '{\"nombre\":\"Juan\",\"email\":\"juan@ejemplo.com\"}'

# PUT
curl -X PUT http://localhost:8000/api/usuarios/123 `
  -H "Content-Type: application/json" `
  -d '{\"nombre\":\"Juan Actualizado\"}'

# DELETE
curl -X DELETE http://localhost:8000/api/usuarios/123
```

---

## Ejercicios

### 1. Router con subdominios
Maneja diferentes subdominios:
```
api.ejemplo.com → API REST
admin.ejemplo.com → Panel admin
```

### 2. Router con versioning
```
/v1/api/usuarios
/v2/api/usuarios
```

### 3. Router con prefijos
```python
@router.prefix('/api/v1')
class UsuariosRouter:
    @router.get('/usuarios')
    def listar(self):
        ...
```

---

## Resumen

Has aprendido:

✅ Enrutamiento básico con if/elif  
✅ Router con tabla de rutas (diccionario)  
✅ Router con regex para parámetros dinámicos  
✅ Router con decoradores  
✅ Middleware pattern  
✅ Sistema de rutas escalable

**Siguiente:** [Debugging y testing](./05-debugging-testing.md)

---

## Recursos

- **[re - Regex](https://docs.python.org/3/library/re.html)** - Expresiones regulares en Python
- **[Routing Patterns](https://flask.palletsprojects.com/en/2.3.x/patterns/)** - Patrones de Flask (para inspiración)

Tu sistema de rutas está listo para escalar. 🛣️

# Sistema de Enrutamiento

> **Router profesional con parámetros dinámicos y middlewares**

---

## Router básico vs Router avanzado

### Router básico (if/elif)

```python
def do_GET(self):
    if self.path == '/usuarios':
        self.listar_usuarios()
    elif self.path == '/productos':
        self.listar_productos()
    elif self.path.startswith('/usuarios/'):
        # Problema: código repetitivo y difícil de mantener
        pass
```

### Router avanzado (tabla de rutas)

```python
rutas = {
    'GET /usuarios': self.listar_usuarios,
    'GET /usuarios/:id': self.obtener_usuario,
    'POST /usuarios': self.crear_usuario
}
```

---

## Router con regex y parámetros

```python
import re
from typing import Callable, Dict, Tuple, Optional, List
from urllib.parse import urlparse

class Route:
    """Representa una ruta individual"""
    
    def __init__(self, metodo: str, patron: str, handler: Callable):
        self.metodo = metodo
        self.patron_original = patron
        self.handler = handler
        
        # Convertir patron tipo '/usuarios/:id' a regex
        self.regex = self.patron_a_regex(patron)
        
        # Extraer nombres de parámetros
        self.param_names = re.findall(r':(\w+)', patron)
    
    def patron_a_regex(self, patron: str) -> re.Pattern:
        """
        Convierte patron con :parametros a regex
        
        Ejemplos:
            '/usuarios/:id'              → r'^/usuarios/([^/]+)$'
            '/posts/:post_id/comentarios/:id' → r'^/posts/([^/]+)/comentarios/([^/]+)$'
        """
        # Escapar caracteres especiales excepto :parametros
        regex_str = patron
        
        # Reemplazar :parametro con grupo de captura
        regex_str = re.sub(r':(\w+)', r'([^/]+)', regex_str)
        
        # Anclar al inicio y fin
        regex_str = f'^{regex_str}$'
        
        return re.compile(regex_str)
    
    def match(self, ruta: str, metodo: str) -> Optional[Dict[str, str]]:
        """
        Verifica si la ruta coincide
        
        Returns:
            Dict con parámetros extraídos o None
        """
        if self.metodo != metodo:
            return None
        
        match = self.regex.match(ruta)
        if not match:
            return None
        
        # Extraer valores de parámetros
        valores = match.groups()
        params = dict(zip(self.param_names, valores))
        
        return params

class Router:
    """Sistema de enrutamiento profesional"""
    
    def __init__(self):
        self.routes: List[Route] = []
        self.middleware_before = []
        self.middleware_after = []
        self.error_handlers = {}
    
    def add_route(self, metodo: str, patron: str, handler: Callable):
        """Agrega una ruta"""
        route = Route(metodo, patron, handler)
        self.routes.append(route)
    
    def get(self, patron: str):
        """Decorador para GET"""
        def decorator(handler):
            self.add_route('GET', patron, handler)
            return handler
        return decorator
    
    def post(self, patron: str):
        """Decorador para POST"""
        def decorator(handler):
            self.add_route('POST', patron, handler)
            return handler
        return decorator
    
    def put(self, patron: str):
        """Decorador para PUT"""
        def decorator(handler):
            self.add_route('PUT', patron, handler)
            return handler
        return decorator
    
    def patch(self, patron: str):
        """Decorador para PATCH"""
        def decorator(handler):
            self.add_route('PATCH', patron, handler)
            return handler
        return decorator
    
    def delete(self, patron: str):
        """Decorador para DELETE"""
        def decorator(handler):
            self.add_route('DELETE', patron, handler)
            return handler
        return decorator
    
    def use_before(self, middleware: Callable):
        """Middleware que se ejecuta ANTES del handler"""
        self.middleware_before.append(middleware)
    
    def use_after(self, middleware: Callable):
        """Middleware que se ejecuta DESPUÉS del handler"""
        self.middleware_after.append(middleware)
    
    def error_handler(self, codigo: int):
        """Decorador para manejar errores específicos"""
        def decorator(handler):
            self.error_handlers[codigo] = handler
            return handler
        return decorator
    
    def route(self, ruta: str, metodo: str) -> Optional[Tuple[Callable, Dict]]:
        """
        Busca handler para ruta y método
        
        Returns:
            (handler, params) o None
        """
        for route in self.routes:
            params = route.match(ruta, metodo)
            if params is not None:
                return (route.handler, params)
        return None
```

---

## Ejemplo completo: API con Router

```python
from http.server import BaseHTTPRequestHandler, HTTPServer
from urllib.parse import urlparse
import json

# Crear router
router = Router()

# Base de datos simulada
usuarios = [
    {'id': 1, 'nombre': 'Juan', 'email': 'juan@ejemplo.com'},
    {'id': 2, 'nombre': 'María', 'email': 'maria@ejemplo.com'}
]

posts = [
    {'id': 1, 'titulo': 'Post 1', 'usuario_id': 1},
    {'id': 2, 'titulo': 'Post 2', 'usuario_id': 1},
    {'id': 3, 'titulo': 'Post 3', 'usuario_id': 2}
]

# ===== Definir rutas =====

@router.get('/api/usuarios')
def listar_usuarios(request, params):
    """GET /api/usuarios"""
    request.enviar_json(usuarios)

@router.get('/api/usuarios/:id')
def obtener_usuario(request, params):
    """GET /api/usuarios/:id"""
    user_id = int(params['id'])
    usuario = next((u for u in usuarios if u['id'] == user_id), None)
    
    if usuario:
        request.enviar_json(usuario)
    else:
        request.enviar_error(404, f'Usuario {user_id} no encontrado')

@router.post('/api/usuarios')
def crear_usuario(request, params):
    """POST /api/usuarios"""
    datos = request.leer_json_body()
    
    if not datos or 'nombre' not in datos or 'email' not in datos:
        request.enviar_error(400, 'Campos nombre y email requeridos')
        return
    
    nuevo_id = max([u['id'] for u in usuarios], default=0) + 1
    nuevo = {
        'id': nuevo_id,
        'nombre': datos['nombre'],
        'email': datos['email']
    }
    
    usuarios.append(nuevo)
    request.enviar_json(nuevo, 201)

@router.put('/api/usuarios/:id')
def actualizar_usuario(request, params):
    """PUT /api/usuarios/:id"""
    user_id = int(params['id'])
    datos = request.leer_json_body()
    
    usuario = next((u for u in usuarios if u['id'] == user_id), None)
    if not usuario:
        request.enviar_error(404, 'Usuario no encontrado')
        return
    
    if not datos or 'nombre' not in datos or 'email' not in datos:
        request.enviar_error(400, 'Campos nombre y email requeridos')
        return
    
    usuario['nombre'] = datos['nombre']
    usuario['email'] = datos['email']
    request.enviar_json(usuario)

@router.delete('/api/usuarios/:id')
def eliminar_usuario(request, params):
    """DELETE /api/usuarios/:id"""
    global usuarios
    
    user_id = int(params['id'])
    usuario = next((u for u in usuarios if u['id'] == user_id), None)
    
    if not usuario:
        request.enviar_error(404, 'Usuario no encontrado')
        return
    
    usuarios = [u for u in usuarios if u['id'] != user_id]
    
    request.send_response(204)
    request.end_headers()

# Rutas anidadas
@router.get('/api/usuarios/:id/posts')
def posts_usuario(request, params):
    """GET /api/usuarios/:id/posts"""
    user_id = int(params['id'])
    posts_usuario = [p for p in posts if p['usuario_id'] == user_id]
    request.enviar_json(posts_usuario)

@router.get('/api/posts/:post_id/usuario')
def usuario_post(request, params):
    """GET /api/posts/:post_id/usuario"""
    post_id = int(params['post_id'])
    post = next((p for p in posts if p['id'] == post_id), None)
    
    if not post:
        request.enviar_error(404, 'Post no encontrado')
        return
    
    usuario = next((u for u in usuarios if u['id'] == post['usuario_id']), None)
    request.enviar_json(usuario)

# ===== Middleware =====

def logger_middleware(request):
    """Log cada request"""
    print(f'📥 {request.command} {request.path} desde {request.client_address[0]}')
    return True  # Continuar

def cors_middleware(request):
    """CORS automático en todas las respuestas"""
    # Este middleware se ejecuta después
    pass

router.use_before(logger_middleware)

# ===== Handler HTTP =====

class APIHandler(BaseHTTPRequestHandler):
    """Handler que usa el router"""
    
    def do_GET(self):
        self.procesar()
    
    def do_POST(self):
        self.procesar()
    
    def do_PUT(self):
        self.procesar()
    
    def do_PATCH(self):
        self.procesar()
    
    def do_DELETE(self):
        self.procesar()
    
    def do_OPTIONS(self):
        """Preflight CORS"""
        self.send_response(200)
        self.send_header('Access-Control-Allow-Origin', '*')
        self.send_header('Access-Control-Allow-Methods', 'GET, POST, PUT, PATCH, DELETE, OPTIONS')
        self.send_header('Access-Control-Allow-Headers', 'Content-Type, Authorization')
        self.end_headers()
    
    def procesar(self):
        """Procesa el request usando el router"""
        ruta = urlparse(self.path).path
        metodo = self.command
        
        # Ejecutar middleware before
        for middleware in router.middleware_before:
            continuar = middleware(self)
            if continuar is False:
                return  # Middleware canceló
        
        # Buscar handler
        resultado = router.route(ruta, metodo)
        
        if resultado:
            handler, params = resultado
            try:
                handler(self, params)
            except Exception as e:
                print(f'❌ Error en handler: {e}')
                self.enviar_error(500, 'Error interno del servidor')
        else:
            self.enviar_error(404, f'{metodo} {ruta} no encontrado')
        
        # Ejecutar middleware after
        for middleware in router.middleware_after:
            middleware(self)
    
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
        self.send_header('Access-Control-Allow-Origin', '*')
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
        pass  # Desactivar logs por defecto

if __name__ == '__main__':
    servidor = HTTPServer(('localhost', 8000), APIHandler)
    print('🚀 API con Router avanzado en http://localhost:8000')
    print('\n📋 Rutas disponibles:')
    print('  GET    /api/usuarios')
    print('  GET    /api/usuarios/:id')
    print('  POST   /api/usuarios')
    print('  PUT    /api/usuarios/:id')
    print('  DELETE /api/usuarios/:id')
    print('  GET    /api/usuarios/:id/posts')
    print('  GET    /api/posts/:post_id/usuario\n')
    
    try:
        servidor.serve_forever()
    except KeyboardInterrupt:
        print('\n⛔ Servidor detenido')
        servidor.shutdown()
```

---

## Pruebas

```powershell
# Listar usuarios
curl http://localhost:8000/api/usuarios

# Obtener usuario específico
curl http://localhost:8000/api/usuarios/1

# Posts del usuario
curl http://localhost:8000/api/usuarios/1/posts

# Usuario de un post
curl http://localhost:8000/api/posts/1/usuario

# Crear usuario
curl -X POST http://localhost:8000/api/usuarios `
  -H "Content-Type: application/json" `
  -d '{\"nombre\":\"Pedro\",\"email\":\"pedro@ejemplo.com\"}'

# Actualizar
curl -X PUT http://localhost:8000/api/usuarios/1 `
  -H "Content-Type: application/json" `
  -d '{\"nombre\":\"Juan Actualizado\",\"email\":\"juan.nuevo@ejemplo.com\"}'

# Eliminar
curl -X DELETE http://localhost:8000/api/usuarios/1
```

---

## Router con grupos (prefijos)

```python
class RouterGroup:
    """Grupo de rutas con prefijo común"""
    
    def __init__(self, router: Router, prefijo: str):
        self.router = router
        self.prefijo = prefijo.rstrip('/')
    
    def get(self, patron: str):
        def decorator(handler):
            ruta_completa = self.prefijo + patron
            self.router.add_route('GET', ruta_completa, handler)
            return handler
        return decorator
    
    def post(self, patron: str):
        def decorator(handler):
            ruta_completa = self.prefijo + patron
            self.router.add_route('POST', ruta_completa, handler)
            return handler
        return decorator
    
    def put(self, patron: str):
        def decorator(handler):
            ruta_completa = self.prefijo + patron
            self.router.add_route('PUT', ruta_completa, handler)
            return handler
        return decorator
    
    def delete(self, patron: str):
        def decorator(handler):
            ruta_completa = self.prefijo + patron
            self.router.add_route('DELETE', ruta_completa, handler)
            return handler
        return decorator

# Uso:
router = Router()

# Grupo para API v1
api_v1 = RouterGroup(router, '/api/v1')

@api_v1.get('/usuarios')
def v1_usuarios(request, params):
    request.enviar_json({'version': 'v1', 'usuarios': []})

@api_v1.get('/productos')
def v1_productos(request, params):
    request.enviar_json({'version': 'v1', 'productos': []})

# Grupo para API v2
api_v2 = RouterGroup(router, '/api/v2')

@api_v2.get('/usuarios')
def v2_usuarios(request, params):
    request.enviar_json({'version': 'v2', 'usuarios': [], 'nuevo_campo': True})

# Resultado:
# GET /api/v1/usuarios → v1_usuarios()
# GET /api/v1/productos → v1_productos()
# GET /api/v2/usuarios → v2_usuarios()
```

---

## Router con validación automática

```python
from typing import Type
from pydantic import BaseModel, ValidationError

class Router:
    # ... (código anterior)
    
    def validate(self, modelo: Type[BaseModel]):
        """Decorador que valida el body con Pydantic"""
        def decorator(handler):
            def wrapper(request, params):
                # Leer body
                datos = request.leer_json_body()
                
                try:
                    # Validar con Pydantic
                    datos_validados = modelo(**datos)
                    
                    # Agregar al request
                    request.datos_validados = datos_validados
                    
                    # Llamar handler
                    return handler(request, params)
                    
                except ValidationError as e:
                    # Error de validación
                    request.enviar_error(422, {
                        'error': 'Validación fallida',
                        'detalles': e.errors()
                    })
            
            return wrapper
        return decorator

# Uso:
class CrearUsuarioDTO(BaseModel):
    nombre: str
    email: str
    edad: int = 18

@router.post('/api/usuarios')
@router.validate(CrearUsuarioDTO)
def crear_usuario(request, params):
    # request.datos_validados ya está validado
    usuario = request.datos_validados
    
    nuevo = {
        'id': 999,
        'nombre': usuario.nombre,
        'email': usuario.email,
        'edad': usuario.edad
    }
    
    request.enviar_json(nuevo, 201)
```

---

## Comparación: Vanilla vs Flask

### Python Vanilla

```python
router = Router()

@router.get('/api/usuarios/:id')
def obtener_usuario(request, params):
    user_id = int(params['id'])
    usuario = buscar_usuario(user_id)
    request.enviar_json(usuario)
```

### Flask

```python
from flask import Flask

app = Flask(__name__)

@app.route('/api/usuarios/<int:user_id>', methods=['GET'])
def obtener_usuario(user_id):
    usuario = buscar_usuario(user_id)
    return jsonify(usuario)
```

**Diferencias:**
- Vanilla: Convertimos `:id` manualmente
- Flask: Convierte automáticamente con `<int:user_id>`
- Vanilla: `request.enviar_json()`
- Flask: `jsonify()`

---

## Resumen

Has aprendido:

✅ Router con regex para parámetros dinámicos  
✅ Sistema de decoradores (`@router.get()`)  
✅ Middleware before/after  
✅ Grupos de rutas con prefijos  
✅ Validación automática con Pydantic  
✅ Manejo de errores personalizado

**Siguiente:** [JSON y serialización](./09-json-serializacion.md)

---

## Recursos

- **[Python re module](https://docs.python.org/3/library/re.html)** - Regular expressions
- **[Pydantic](https://docs.pydantic.dev/)** - Data validation

Tu sistema de rutas ahora es tan potente como un framework. 🛣️

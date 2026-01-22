# Protección de rutas

> **Asegura que solo usuarios autenticados accedan a recursos protegidos**

---

## ¿Qué es proteger rutas?

**Proteger una ruta:** Verificar que el usuario esté autenticado **antes** de procesar la petición.

```
Cliente → GET /api/perfil → ¿Token válido?
                             ├─ SÍ → Procesar petición
                             └─ NO → 401 Unauthorized
```

---

## Tipos de protección

### 1. Autenticación

¿El usuario está identificado?

```python
if not token:
    return 401  # Unauthorized
```

### 2. Autorización por rol

¿El usuario tiene el rol necesario?

```python
if usuario.rol != 'admin':
    return 403  # Forbidden
```

### 3. Autorización por recurso

¿El usuario puede acceder a ESTE recurso específico?

```python
if usuario.id != recurso.propietario_id:
    return 403  # Forbidden
```

---

## Decoradores para protección

**auth_decorators.py**

```python
"""
Decoradores para proteger endpoints
"""

from functools import wraps
from typing import Callable, List, Optional
import jwt

SECRET_KEY = 'clave-secreta-super-segura-cambiar-en-produccion'
ALGORITHM = 'HS256'

class AuthDecorators:
    """Decoradores para autenticación y autorización"""
    
    @staticmethod
    def requiere_autenticacion(metodo: Callable) -> Callable:
        """
        Decorator: requiere que el usuario esté autenticado
        
        Uso:
            @AuthDecorators.requiere_autenticacion
            def mi_endpoint(self):
                # self.usuario_actual disponible
                pass
        """
        @wraps(metodo)
        def wrapper(self, *args, **kwargs):
            # Extraer token del header
            auth_header = self.headers.get('Authorization')
            
            if not auth_header:
                self.enviar_error(401, 'Token requerido')
                return None
            
            # Verificar formato: "Bearer <token>"
            partes = auth_header.split()
            if len(partes) != 2 or partes[0] != 'Bearer':
                self.enviar_error(401, 'Formato de token inválido')
                return None
            
            token = partes[1]
            
            try:
                # Verificar y decodificar token
                payload = jwt.decode(token, SECRET_KEY, algorithms=[ALGORITHM])
                
                # Agregar payload al handler para que el endpoint lo use
                self.usuario_actual = payload
                
                # Llamar al método original
                return metodo(self, *args, **kwargs)
                
            except jwt.ExpiredSignatureError:
                self.enviar_error(401, 'Token expirado')
                return None
            except jwt.InvalidTokenError:
                self.enviar_error(401, 'Token inválido')
                return None
        
        return wrapper
    
    @staticmethod
    def requiere_rol(*roles_permitidos: str) -> Callable:
        """
        Decorator: requiere que el usuario tenga uno de los roles especificados
        
        Uso:
            @AuthDecorators.requiere_autenticacion
            @AuthDecorators.requiere_rol('admin', 'moderador')
            def mi_endpoint(self):
                pass
        """
        def decorator(metodo: Callable) -> Callable:
            @wraps(metodo)
            def wrapper(self, *args, **kwargs):
                # Verificar que ya pasó por requiere_autenticacion
                if not hasattr(self, 'usuario_actual'):
                    self.enviar_error(401, 'No autenticado')
                    return None
                
                # Obtener rol del usuario
                rol_usuario = self.usuario_actual.get('rol')
                
                # Verificar que tenga alguno de los roles permitidos
                if rol_usuario not in roles_permitidos:
                    roles_str = ', '.join(roles_permitidos)
                    self.enviar_error(
                        403,
                        f'Acceso denegado. Roles permitidos: {roles_str}'
                    )
                    return None
                
                # Llamar al método original
                return metodo(self, *args, **kwargs)
            
            return wrapper
        return decorator
    
    @staticmethod
    def requiere_propietario(id_param: str = 'id') -> Callable:
        """
        Decorator: requiere que el usuario sea propietario del recurso
        
        Uso:
            @AuthDecorators.requiere_autenticacion
            @AuthDecorators.requiere_propietario('user_id')
            def actualizar_usuario(self, user_id):
                pass
        """
        def decorator(metodo: Callable) -> Callable:
            @wraps(metodo)
            def wrapper(self, *args, **kwargs):
                # Verificar que ya pasó por requiere_autenticacion
                if not hasattr(self, 'usuario_actual'):
                    self.enviar_error(401, 'No autenticado')
                    return None
                
                # Obtener ID del usuario actual
                user_id_actual = self.usuario_actual.get('sub')
                
                # Obtener ID del recurso desde kwargs
                resource_id = kwargs.get(id_param)
                
                if not resource_id:
                    self.enviar_error(400, f'ID de {id_param} no especificado')
                    return None
                
                # Verificar que sea el propietario
                if str(user_id_actual) != str(resource_id):
                    self.enviar_error(
                        403,
                        'Solo puedes acceder a tus propios recursos'
                    )
                    return None
                
                # Llamar al método original
                return metodo(self, *args, **kwargs)
            
            return wrapper
        return decorator
```

---

## API con rutas protegidas

**app_protegido.py**

```python
"""
API con diferentes niveles de protección
"""

from http.server import BaseHTTPRequestHandler, HTTPServer
from urllib.parse import urlparse
import json
import re
import jwt
from datetime import datetime, timedelta

from auth_decorators import AuthDecorators

SECRET_KEY = 'clave-secreta-super-segura-cambiar-en-produccion'
ALGORITHM = 'HS256'

# Base de datos simulada
usuarios_db = {
    'ana@ejemplo.com': {
        'id': 1,
        'email': 'ana@ejemplo.com',
        'nombre': 'Ana García',
        'rol': 'usuario'
    },
    'admin@ejemplo.com': {
        'id': 2,
        'email': 'admin@ejemplo.com',
        'nombre': 'Admin Sistema',
        'rol': 'admin'
    }
}

class ProtectedAPIHandler(BaseHTTPRequestHandler):
    """Handler con rutas protegidas"""
    
    def do_GET(self):
        """Maneja peticiones GET"""
        try:
            ruta = urlparse(self.path).path
            
            # Rutas públicas
            if ruta == '/':
                self.home()
            elif ruta == '/api/publica':
                self.ruta_publica()
            
            # Rutas protegidas
            elif ruta == '/api/perfil':
                self.perfil()
            elif ruta == '/api/admin/dashboard':
                self.admin_dashboard()
            elif ruta == '/api/admin/usuarios':
                self.admin_lista_usuarios()
            
            # Rutas con parámetros
            else:
                match_usuario = re.match(r'^/api/usuarios/(\d+)$', ruta)
                if match_usuario:
                    user_id = int(match_usuario.group(1))
                    self.obtener_usuario(user_id=user_id)
                    return
                
                self.enviar_error(404, 'Ruta no encontrada')
                
        except Exception as e:
            print(f'Error: {e}')
            self.enviar_error(500, str(e))
    
    def do_PUT(self):
        """Maneja peticiones PUT"""
        try:
            ruta = urlparse(self.path).path
            
            match_usuario = re.match(r'^/api/usuarios/(\d+)$', ruta)
            if match_usuario:
                user_id = int(match_usuario.group(1))
                self.actualizar_usuario(user_id=user_id)
                return
            
            self.enviar_error(404, 'Ruta no encontrada')
            
        except Exception as e:
            print(f'Error: {e}')
            self.enviar_error(500, str(e))
    
    def do_POST(self):
        """Maneja peticiones POST"""
        try:
            ruta = urlparse(self.path).path
            
            if ruta == '/api/login':
                self.login()
            else:
                self.enviar_error(404, 'Ruta no encontrada')
                
        except Exception as e:
            print(f'Error: {e}')
            self.enviar_error(500, str(e))
    
    # ===== Rutas públicas =====
    
    def home(self):
        """GET / - Pública"""
        html = """
        <!DOCTYPE html>
        <html>
        <head>
            <title>API Protegida</title>
            <style>
                body { font-family: Arial; max-width: 900px; margin: 50px auto; }
                .endpoint { background: #f5f5f5; padding: 15px; margin: 10px 0; }
                .publica { border-left: 4px solid #4CAF50; }
                .protegida { border-left: 4px solid #FFC107; }
                .admin { border-left: 4px solid #F44336; }
            </style>
        </head>
        <body>
            <h1>🔐 API con Rutas Protegidas</h1>
            
            <h2>🟢 Rutas Públicas (sin autenticación)</h2>
            <div class="endpoint publica">
                <strong>GET /api/publica</strong><br>
                Acceso libre
            </div>
            <div class="endpoint publica">
                <strong>POST /api/login</strong><br>
                Autenticación
            </div>
            
            <h2>🟡 Rutas Protegidas (requieren token)</h2>
            <div class="endpoint protegida">
                <strong>GET /api/perfil</strong><br>
                Headers: <code>Authorization: Bearer &lt;token&gt;</code>
            </div>
            <div class="endpoint protegida">
                <strong>GET /api/usuarios/:id</strong><br>
                Solo el propietario
            </div>
            <div class="endpoint protegida">
                <strong>PUT /api/usuarios/:id</strong><br>
                Solo el propietario
            </div>
            
            <h2>🔴 Rutas Admin (requieren rol admin)</h2>
            <div class="endpoint admin">
                <strong>GET /api/admin/dashboard</strong><br>
                Solo administradores
            </div>
            <div class="endpoint admin">
                <strong>GET /api/admin/usuarios</strong><br>
                Solo administradores
            </div>
        </body>
        </html>
        """
        self.enviar_html(html)
    
    def ruta_publica(self):
        """GET /api/publica - Sin protección"""
        self.enviar_json({
            'mensaje': 'Ruta pública - acceso sin autenticación',
            'timestamp': datetime.now().isoformat()
        })
    
    def login(self):
        """POST /api/login - Genera token"""
        datos = self.leer_json_body()
        
        if not datos:
            self.enviar_error(400, 'Datos requeridos')
            return
        
        email = datos.get('email')
        password = datos.get('password')  # En producción: verificar con bcrypt
        
        if not email or not password:
            self.enviar_error(400, 'Email y password requeridos')
            return
        
        # Buscar usuario
        usuario = usuarios_db.get(email)
        if not usuario:
            self.enviar_error(401, 'Credenciales inválidas')
            return
        
        # Generar token
        payload = {
            'sub': str(usuario['id']),
            'email': usuario['email'],
            'rol': usuario['rol'],
            'exp': datetime.utcnow() + timedelta(hours=1)
        }
        
        token = jwt.encode(payload, SECRET_KEY, algorithm=ALGORITHM)
        
        self.enviar_json({
            'access_token': token,
            'token_type': 'Bearer',
            'usuario': {
                'id': usuario['id'],
                'email': usuario['email'],
                'nombre': usuario['nombre'],
                'rol': usuario['rol']
            }
        })
    
    # ===== Rutas protegidas =====
    
    @AuthDecorators.requiere_autenticacion
    def perfil(self):
        """GET /api/perfil - Requiere autenticación"""
        self.enviar_json({
            'mensaje': 'Perfil del usuario autenticado',
            'usuario': {
                'id': self.usuario_actual['sub'],
                'email': self.usuario_actual['email'],
                'rol': self.usuario_actual['rol']
            }
        })
    
    @AuthDecorators.requiere_autenticacion
    @AuthDecorators.requiere_propietario('user_id')
    def obtener_usuario(self, user_id: int):
        """GET /api/usuarios/:id - Solo propietario"""
        # Buscar usuario en DB
        usuario = next(
            (u for u in usuarios_db.values() if u['id'] == user_id),
            None
        )
        
        if not usuario:
            self.enviar_error(404, 'Usuario no encontrado')
            return
        
        self.enviar_json({
            'usuario': usuario
        })
    
    @AuthDecorators.requiere_autenticacion
    @AuthDecorators.requiere_propietario('user_id')
    def actualizar_usuario(self, user_id: int):
        """PUT /api/usuarios/:id - Solo propietario"""
        datos = self.leer_json_body()
        
        if not datos:
            self.enviar_error(400, 'Datos requeridos')
            return
        
        # Buscar usuario
        usuario = next(
            (u for u in usuarios_db.values() if u['id'] == user_id),
            None
        )
        
        if not usuario:
            self.enviar_error(404, 'Usuario no encontrado')
            return
        
        # Actualizar nombre
        if 'nombre' in datos:
            usuario['nombre'] = datos['nombre']
        
        self.enviar_json({
            'mensaje': 'Usuario actualizado',
            'usuario': usuario
        })
    
    # ===== Rutas admin =====
    
    @AuthDecorators.requiere_autenticacion
    @AuthDecorators.requiere_rol('admin')
    def admin_dashboard(self):
        """GET /api/admin/dashboard - Solo admins"""
        self.enviar_json({
            'mensaje': 'Dashboard de administración',
            'estadisticas': {
                'usuarios_totales': len(usuarios_db),
                'admins': sum(1 for u in usuarios_db.values() if u['rol'] == 'admin'),
                'usuarios': sum(1 for u in usuarios_db.values() if u['rol'] == 'usuario')
            }
        })
    
    @AuthDecorators.requiere_autenticacion
    @AuthDecorators.requiere_rol('admin')
    def admin_lista_usuarios(self):
        """GET /api/admin/usuarios - Solo admins"""
        usuarios = list(usuarios_db.values())
        
        self.enviar_json({
            'usuarios': usuarios,
            'total': len(usuarios)
        })
    
    # ===== Helpers =====
    
    def enviar_json(self, datos, status=200):
        """Envía respuesta JSON"""
        self.send_response(status)
        self.send_header('Content-type', 'application/json; charset=utf-8')
        self.send_header('Access-Control-Allow-Origin', '*')
        self.end_headers()
        self.wfile.write(json.dumps(datos, ensure_ascii=False, indent=2).encode())
    
    def enviar_html(self, html):
        """Envía respuesta HTML"""
        self.send_response(200)
        self.send_header('Content-type', 'text/html; charset=utf-8')
        self.end_headers()
        self.wfile.write(html.encode())
    
    def enviar_error(self, codigo, mensaje):
        """Envía error JSON"""
        self.send_response(codigo)
        self.send_header('Content-type', 'application/json; charset=utf-8')
        self.end_headers()
        error = {'error': mensaje, 'codigo': codigo}
        self.wfile.write(json.dumps(error, ensure_ascii=False).encode())
    
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
    
    def log_message(self, format, *args):
        """Log personalizado"""
        print(f'📥 {self.command} {self.path}')

# ===== Servidor =====

if __name__ == '__main__':
    puerto = 8000
    servidor = HTTPServer(('localhost', puerto), ProtectedAPIHandler)
    
    print('='*70)
    print('🔐 API con Rutas Protegidas')
    print('='*70)
    print(f'\n🚀 Servidor: http://localhost:{puerto}')
    print(f'\n👥 Usuarios de prueba:')
    print('   Usuario: ana@ejemplo.com (rol: usuario)')
    print('   Admin:   admin@ejemplo.com (rol: admin)')
    print('   Password: cualquiera (sin validación en este ejemplo)')
    print('\n⛔ Ctrl+C para detener\n')
    
    try:
        servidor.serve_forever()
    except KeyboardInterrupt:
        print('\n\n⛔ Servidor detenido')
        servidor.shutdown()
```

---

## Pruebas con curl

### 1. Ruta pública (sin token)

```powershell
curl http://localhost:8000/api/publica
```

### 2. Login para obtener token

```powershell
# Usuario normal
curl -X POST http://localhost:8000/api/login `
  -H "Content-Type: application/json" `
  -d '{\"email\":\"ana@ejemplo.com\",\"password\":\"cualquiera\"}'

# Admin
curl -X POST http://localhost:8000/api/login `
  -H "Content-Type: application/json" `
  -d '{\"email\":\"admin@ejemplo.com\",\"password\":\"cualquiera\"}'

# Guardar token
$token = "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
```

### 3. Acceder a ruta protegida

```powershell
# ✅ Con token
curl http://localhost:8000/api/perfil `
  -H "Authorization: Bearer $token"

# ❌ Sin token
curl http://localhost:8000/api/perfil
# Response: {"error": "Token requerido", "codigo": 401}
```

### 4. Acceder a recurso propio

```powershell
# ✅ Usuario 1 accede a su perfil
curl http://localhost:8000/api/usuarios/1 `
  -H "Authorization: Bearer $tokenUsuario1"

# ❌ Usuario 1 intenta acceder al perfil de usuario 2
curl http://localhost:8000/api/usuarios/2 `
  -H "Authorization: Bearer $tokenUsuario1"
# Response: {"error": "Solo puedes acceder a tus propios recursos", "codigo": 403}
```

### 5. Actualizar recurso propio

```powershell
# ✅ Usuario actualiza su propio perfil
curl -X PUT http://localhost:8000/api/usuarios/1 `
  -H "Authorization: Bearer $tokenUsuario1" `
  -H "Content-Type: application/json" `
  -d '{\"nombre\":\"Ana García López\"}'
```

### 6. Acceder a rutas admin

```powershell
# ✅ Con token de admin
curl http://localhost:8000/api/admin/dashboard `
  -H "Authorization: Bearer $tokenAdmin"

# ❌ Con token de usuario normal
curl http://localhost:8000/api/admin/dashboard `
  -H "Authorization: Bearer $tokenUsuario"
# Response: {"error": "Acceso denegado. Roles permitidos: admin", "codigo": 403}
```

---

## Middleware para autenticación

Alternativa a decoradores: middleware que procesa el token antes de llamar endpoints:

**middleware_auth.py**

```python
"""
Middleware de autenticación
"""

import jwt
from typing import Optional, Dict

SECRET_KEY = 'clave-secreta-super-segura-cambiar-en-produccion'
ALGORITHM = 'HS256'

class AuthMiddleware:
    """Middleware para procesar autenticación"""
    
    def __init__(self, handler):
        self.handler = handler
    
    def procesar_autenticacion(self) -> Optional[Dict]:
        """
        Extrae y verifica el token JWT del header
        
        Returns:
            Payload del token si es válido, None si no
        """
        # Extraer header
        auth_header = self.handler.headers.get('Authorization')
        
        if not auth_header:
            return None
        
        # Verificar formato
        partes = auth_header.split()
        if len(partes) != 2 or partes[0] != 'Bearer':
            return None
        
        token = partes[1]
        
        try:
            # Verificar token
            payload = jwt.decode(token, SECRET_KEY, algorithms=[ALGORITHM])
            return payload
        except (jwt.ExpiredSignatureError, jwt.InvalidTokenError):
            return None
    
    def requiere_autenticacion(self) -> bool:
        """Verifica si la ruta actual requiere autenticación"""
        ruta = self.handler.path
        
        # Rutas públicas
        rutas_publicas = ['/', '/api/publica', '/api/login', '/api/registro']
        
        return ruta not in rutas_publicas
```

---

## Resumen

Has aprendido:

✅ Proteger rutas con decoradores  
✅ Verificar autenticación (¿quién eres?)  
✅ Verificar autorización por rol (¿qué puedes hacer?)  
✅ Verificar autorización por recurso (¿es tuyo?)  
✅ Códigos HTTP correctos (401 vs 403)  
✅ Middleware de autenticación

**Siguiente:** [CORS y seguridad](./27-cors-seguridad.md)

---

## Recursos

- **[HTTP 401 vs 403](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status)** - MDN
- **[OWASP Auth](https://owasp.org/www-project-top-ten/)** - Autenticación segura
- **[JWT Best Practices](https://datatracker.ietf.org/doc/html/rfc8725)** - RFC 8725

Tus rutas están protegidas profesionalmente. 🔐

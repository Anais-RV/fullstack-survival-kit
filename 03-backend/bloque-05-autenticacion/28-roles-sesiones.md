# Roles y sesiones avanzadas

> **Implementa sistema de roles, permisos y gestión de sesiones múltiples**

---

## Sistema de roles

**Roles comunes:**

| Rol | Permisos típicos |
|-----|------------------|
| `super_admin` | Acceso total |
| `admin` | Gestión de usuarios y contenido |
| `moderador` | Moderar contenido |
| `usuario` | Acceso básico |
| `invitado` | Solo lectura |

---

## Modelo de datos

**models/usuario.py**

```python
"""
Modelo de usuario con roles y permisos
"""

from dataclasses import dataclass, field
from typing import List, Set
from datetime import datetime

@dataclass
class Usuario:
    """Usuario con roles y permisos"""
    id: int
    email: str
    nombre: str
    password_hash: bytes
    rol: str = 'usuario'
    permisos: Set[str] = field(default_factory=set)
    activo: bool = True
    sesiones: List['Sesion'] = field(default_factory=list)
    creado_en: datetime = field(default_factory=datetime.now)
    
    def tiene_permiso(self, permiso: str) -> bool:
        """Verifica si tiene un permiso específico"""
        return permiso in self.permisos
    
    def agregar_permiso(self, permiso: str):
        """Agrega un permiso"""
        self.permisos.add(permiso)
    
    def remover_permiso(self, permiso: str):
        """Remueve un permiso"""
        self.permisos.discard(permiso)
    
    def agregar_sesion(self, sesion: 'Sesion'):
        """Agrega una sesión activa"""
        self.sesiones.append(sesion)
    
    def cerrar_sesion(self, session_id: str) -> bool:
        """Cierra una sesión específica"""
        self.sesiones = [
            s for s in self.sesiones 
            if s.id != session_id
        ]
        return True
    
    def cerrar_todas_sesiones(self):
        """Cierra todas las sesiones"""
        self.sesiones.clear()
    
    def to_dict(self, incluir_sensible: bool = False) -> dict:
        """Convierte a diccionario"""
        data = {
            'id': self.id,
            'email': self.email,
            'nombre': self.nombre,
            'rol': self.rol,
            'permisos': list(self.permisos),
            'activo': self.activo,
            'sesiones_activas': len(self.sesiones),
            'creado_en': self.creado_en.isoformat()
        }
        
        if incluir_sensible:
            data['sesiones'] = [s.to_dict() for s in self.sesiones]
        
        return data

@dataclass
class Sesion:
    """Sesión de usuario"""
    id: str
    user_id: int
    refresh_token: str
    ip: str
    user_agent: str
    creada_en: datetime = field(default_factory=datetime.now)
    ultima_actividad: datetime = field(default_factory=datetime.now)
    
    def actualizar_actividad(self):
        """Actualiza última actividad"""
        self.ultima_actividad = datetime.now()
    
    def to_dict(self) -> dict:
        """Convierte a diccionario"""
        return {
            'id': self.id,
            'ip': self.ip,
            'user_agent': self.user_agent,
            'creada_en': self.creada_en.isoformat(),
            'ultima_actividad': self.ultima_actividad.isoformat()
        }
```

---

## Sistema de permisos

**auth/permissions.py**

```python
"""
Sistema de permisos basado en acciones
"""

from enum import Enum
from typing import Set, Dict

class Permiso(Enum):
    """Permisos disponibles en el sistema"""
    # Usuarios
    USUARIOS_LEER = 'usuarios:leer'
    USUARIOS_CREAR = 'usuarios:crear'
    USUARIOS_EDITAR = 'usuarios:editar'
    USUARIOS_ELIMINAR = 'usuarios:eliminar'
    
    # Contenido
    CONTENIDO_LEER = 'contenido:leer'
    CONTENIDO_CREAR = 'contenido:crear'
    CONTENIDO_EDITAR = 'contenido:editar'
    CONTENIDO_ELIMINAR = 'contenido:eliminar'
    
    # Admin
    ADMIN_ACCESO = 'admin:acceso'
    ADMIN_CONFIGURACION = 'admin:configuracion'

class RolesPermisos:
    """Define permisos para cada rol"""
    
    PERMISOS_POR_ROL: Dict[str, Set[str]] = {
        'invitado': {
            Permiso.CONTENIDO_LEER.value
        },
        'usuario': {
            Permiso.CONTENIDO_LEER.value,
            Permiso.CONTENIDO_CREAR.value,
            Permiso.USUARIOS_LEER.value
        },
        'moderador': {
            Permiso.CONTENIDO_LEER.value,
            Permiso.CONTENIDO_CREAR.value,
            Permiso.CONTENIDO_EDITAR.value,
            Permiso.CONTENIDO_ELIMINAR.value,
            Permiso.USUARIOS_LEER.value
        },
        'admin': {
            Permiso.USUARIOS_LEER.value,
            Permiso.USUARIOS_CREAR.value,
            Permiso.USUARIOS_EDITAR.value,
            Permiso.USUARIOS_ELIMINAR.value,
            Permiso.CONTENIDO_LEER.value,
            Permiso.CONTENIDO_CREAR.value,
            Permiso.CONTENIDO_EDITAR.value,
            Permiso.CONTENIDO_ELIMINAR.value,
            Permiso.ADMIN_ACCESO.value
        },
        'super_admin': {
            permiso.value for permiso in Permiso
        }
    }
    
    @classmethod
    def obtener_permisos(cls, rol: str) -> Set[str]:
        """Obtiene permisos de un rol"""
        return cls.PERMISOS_POR_ROL.get(rol, set())
    
    @classmethod
    def tiene_permiso(cls, rol: str, permiso: str) -> bool:
        """Verifica si un rol tiene un permiso"""
        permisos_rol = cls.obtener_permisos(rol)
        return permiso in permisos_rol
```

---

## Servicio de autenticación completo

**auth/auth_service.py**

```python
"""
Servicio de autenticación con roles y sesiones
"""

import jwt
import bcrypt
import secrets
from datetime import datetime, timedelta
from typing import Optional, Dict, Tuple

from models.usuario import Usuario, Sesion
from auth.permissions import RolesPermisos

SECRET_KEY = 'clave-secreta-super-segura-cambiar-en-produccion'
ALGORITHM = 'HS256'

class AuthService:
    """Servicio de autenticación completo"""
    
    def __init__(self):
        self.usuarios_db: Dict[str, Usuario] = {}
        self.refresh_tokens: Dict[str, str] = {}  # token -> email
        self.siguiente_id = 1
    
    def registrar(self, email: str, password: str, nombre: str, 
                  rol: str = 'usuario') -> Usuario:
        """Registra un nuevo usuario"""
        if email in self.usuarios_db:
            raise ValueError('Email ya registrado')
        
        # Hash de contraseña
        password_hash = bcrypt.hashpw(password.encode(), bcrypt.gensalt())
        
        # Obtener permisos del rol
        permisos = RolesPermisos.obtener_permisos(rol)
        
        # Crear usuario
        usuario = Usuario(
            id=self.siguiente_id,
            email=email,
            nombre=nombre,
            password_hash=password_hash,
            rol=rol,
            permisos=permisos
        )
        
        self.usuarios_db[email] = usuario
        self.siguiente_id += 1
        
        return usuario
    
    def login(self, email: str, password: str, ip: str, 
              user_agent: str) -> Tuple[str, str, Usuario]:
        """
        Autentica usuario y crea sesión
        
        Returns:
            (access_token, refresh_token, usuario)
        """
        # Buscar usuario
        usuario = self.usuarios_db.get(email)
        if not usuario:
            raise ValueError('Credenciales inválidas')
        
        if not usuario.activo:
            raise ValueError('Usuario inactivo')
        
        # Verificar contraseña
        if not bcrypt.checkpw(password.encode(), usuario.password_hash):
            raise ValueError('Credenciales inválidas')
        
        # Crear sesión
        session_id = secrets.token_hex(16)
        refresh_token = self._generar_refresh_token(usuario.id, session_id)
        
        sesion = Sesion(
            id=session_id,
            user_id=usuario.id,
            refresh_token=refresh_token,
            ip=ip,
            user_agent=user_agent
        )
        
        usuario.agregar_sesion(sesion)
        self.refresh_tokens[refresh_token] = email
        
        # Generar access token
        access_token = self._generar_access_token(usuario)
        
        return access_token, refresh_token, usuario
    
    def refresh(self, refresh_token: str) -> str:
        """Renueva access token con refresh token"""
        # Verificar que existe
        email = self.refresh_tokens.get(refresh_token)
        if not email:
            raise ValueError('Refresh token inválido')
        
        # Verificar token
        try:
            payload = jwt.decode(refresh_token, SECRET_KEY, algorithms=[ALGORITHM])
        except jwt.InvalidTokenError:
            # Remover token inválido
            self.refresh_tokens.pop(refresh_token, None)
            raise ValueError('Refresh token inválido o expirado')
        
        # Obtener usuario
        usuario = self.usuarios_db.get(email)
        if not usuario or not usuario.activo:
            raise ValueError('Usuario no encontrado o inactivo')
        
        # Actualizar actividad de sesión
        session_id = payload.get('session_id')
        for sesion in usuario.sesiones:
            if sesion.id == session_id:
                sesion.actualizar_actividad()
                break
        
        # Generar nuevo access token
        return self._generar_access_token(usuario)
    
    def logout(self, email: str, session_id: str):
        """Cierra sesión específica"""
        usuario = self.usuarios_db.get(email)
        if not usuario:
            return
        
        # Remover sesión
        for sesion in usuario.sesiones:
            if sesion.id == session_id:
                self.refresh_tokens.pop(sesion.refresh_token, None)
                break
        
        usuario.cerrar_sesion(session_id)
    
    def logout_all(self, email: str):
        """Cierra todas las sesiones del usuario"""
        usuario = self.usuarios_db.get(email)
        if not usuario:
            return
        
        # Remover todos los refresh tokens
        for sesion in usuario.sesiones:
            self.refresh_tokens.pop(sesion.refresh_token, None)
        
        usuario.cerrar_todas_sesiones()
    
    def verificar_permiso(self, usuario: Usuario, permiso: str) -> bool:
        """Verifica si usuario tiene permiso"""
        return usuario.tiene_permiso(permiso)
    
    def _generar_access_token(self, usuario: Usuario) -> str:
        """Genera access token JWT"""
        payload = {
            'sub': str(usuario.id),
            'email': usuario.email,
            'nombre': usuario.nombre,
            'rol': usuario.rol,
            'permisos': list(usuario.permisos),
            'exp': datetime.utcnow() + timedelta(minutes=15),
            'tipo': 'access'
        }
        
        return jwt.encode(payload, SECRET_KEY, algorithm=ALGORITHM)
    
    def _generar_refresh_token(self, user_id: int, session_id: str) -> str:
        """Genera refresh token JWT"""
        payload = {
            'sub': str(user_id),
            'session_id': session_id,
            'exp': datetime.utcnow() + timedelta(days=7),
            'tipo': 'refresh'
        }
        
        return jwt.encode(payload, SECRET_KEY, algorithm=ALGORITHM)
    
    def verificar_token(self, token: str) -> Dict:
        """Verifica access token"""
        try:
            payload = jwt.decode(token, SECRET_KEY, algorithms=[ALGORITHM])
            return payload
        except jwt.InvalidTokenError as e:
            raise ValueError(f'Token inválido: {str(e)}')

# Instancia global
auth_service = AuthService()
```

---

## API con roles completos

**app_roles.py**

```python
"""
API con sistema completo de roles y permisos
"""

from http.server import BaseHTTPRequestHandler, HTTPServer
from urllib.parse import urlparse
import json

from auth.auth_service import auth_service
from auth.permissions import Permiso

class RolesAPIHandler(BaseHTTPRequestHandler):
    """Handler con roles y permisos"""
    
    def do_POST(self):
        """Maneja POST"""
        try:
            ruta = urlparse(self.path).path
            
            if ruta == '/api/registro':
                self.registro()
            elif ruta == '/api/login':
                self.login()
            elif ruta == '/api/refresh':
                self.refresh()
            elif ruta == '/api/logout':
                self.logout()
            elif ruta == '/api/logout-all':
                self.logout_all()
            else:
                self.enviar_error(404, 'Ruta no encontrada')
                
        except Exception as e:
            print(f'Error: {e}')
            self.enviar_error(500, str(e))
    
    def do_GET(self):
        """Maneja GET"""
        try:
            ruta = urlparse(self.path).path
            
            if ruta == '/':
                self.home()
            elif ruta == '/api/perfil':
                self.perfil()
            elif ruta == '/api/sesiones':
                self.sesiones()
            elif ruta == '/api/admin/usuarios':
                self.admin_usuarios()
            else:
                self.enviar_error(404, 'Ruta no encontrada')
                
        except Exception as e:
            print(f'Error: {e}')
            self.enviar_error(500, str(e))
    
    def home(self):
        """GET /"""
        html = """
        <!DOCTYPE html>
        <html>
        <head>
            <title>API Roles</title>
        </head>
        <body>
            <h1>🔐 API con Roles y Permisos</h1>
            <h2>Endpoints:</h2>
            <ul>
                <li>POST /api/registro - Registrar usuario</li>
                <li>POST /api/login - Login con sesión</li>
                <li>POST /api/refresh - Renovar token</li>
                <li>POST /api/logout - Cerrar sesión</li>
                <li>POST /api/logout-all - Cerrar todas</li>
                <li>GET /api/perfil - Ver perfil</li>
                <li>GET /api/sesiones - Ver sesiones activas</li>
                <li>GET /api/admin/usuarios - Lista usuarios (admin)</li>
            </ul>
        </body>
        </html>
        """
        
        self.send_response(200)
        self.send_header('Content-type', 'text/html')
        self.end_headers()
        self.wfile.write(html.encode())
    
    def registro(self):
        """POST /api/registro"""
        datos = self.leer_json_body()
        
        if not datos:
            self.enviar_error(400, 'Datos requeridos')
            return
        
        try:
            usuario = auth_service.registrar(
                email=datos['email'],
                password=datos['password'],
                nombre=datos['nombre'],
                rol=datos.get('rol', 'usuario')
            )
            
            self.send_response(201)
            self.send_header('Content-type', 'application/json')
            self.end_headers()
            
            self.wfile.write(json.dumps({
                'mensaje': 'Usuario registrado',
                'usuario': usuario.to_dict()
            }, ensure_ascii=False).encode())
            
        except ValueError as e:
            self.enviar_error(400, str(e))
    
    def login(self):
        """POST /api/login"""
        datos = self.leer_json_body()
        
        if not datos:
            self.enviar_error(400, 'Datos requeridos')
            return
        
        try:
            ip = self.client_address[0]
            user_agent = self.headers.get('User-Agent', 'Unknown')
            
            access_token, refresh_token, usuario = auth_service.login(
                email=datos['email'],
                password=datos['password'],
                ip=ip,
                user_agent=user_agent
            )
            
            self.enviar_json({
                'access_token': access_token,
                'refresh_token': refresh_token,
                'usuario': usuario.to_dict()
            })
            
        except ValueError as e:
            self.enviar_error(401, str(e))
    
    def refresh(self):
        """POST /api/refresh"""
        datos = self.leer_json_body()
        
        if not datos:
            self.enviar_error(400, 'Datos requeridos')
            return
        
        try:
            nuevo_access = auth_service.refresh(datos['refresh_token'])
            
            self.enviar_json({
                'access_token': nuevo_access
            })
            
        except ValueError as e:
            self.enviar_error(401, str(e))
    
    def logout(self):
        """POST /api/logout"""
        payload = self.requiere_autenticacion()
        if not payload:
            return
        
        datos = self.leer_json_body()
        session_id = datos.get('session_id') if datos else None
        
        if not session_id:
            self.enviar_error(400, 'session_id requerido')
            return
        
        auth_service.logout(payload['email'], session_id)
        
        self.enviar_json({'mensaje': 'Sesión cerrada'})
    
    def logout_all(self):
        """POST /api/logout-all"""
        payload = self.requiere_autenticacion()
        if not payload:
            return
        
        auth_service.logout_all(payload['email'])
        
        self.enviar_json({'mensaje': 'Todas las sesiones cerradas'})
    
    def perfil(self):
        """GET /api/perfil"""
        payload = self.requiere_autenticacion()
        if not payload:
            return
        
        usuario = auth_service.usuarios_db.get(payload['email'])
        
        if not usuario:
            self.enviar_error(404, 'Usuario no encontrado')
            return
        
        self.enviar_json(usuario.to_dict(incluir_sensible=True))
    
    def sesiones(self):
        """GET /api/sesiones"""
        payload = self.requiere_autenticacion()
        if not payload:
            return
        
        usuario = auth_service.usuarios_db.get(payload['email'])
        
        if not usuario:
            self.enviar_error(404, 'Usuario no encontrado')
            return
        
        self.enviar_json({
            'sesiones': [s.to_dict() for s in usuario.sesiones],
            'total': len(usuario.sesiones)
        })
    
    def admin_usuarios(self):
        """GET /api/admin/usuarios - Solo admins"""
        payload = self.requiere_autenticacion()
        if not payload:
            return
        
        # Verificar permiso
        if not Permiso.USUARIOS_LEER.value in payload['permisos']:
            self.enviar_error(403, 'Permiso denegado')
            return
        
        usuarios = [u.to_dict() for u in auth_service.usuarios_db.values()]
        
        self.enviar_json({
            'usuarios': usuarios,
            'total': len(usuarios)
        })
    
    def requiere_autenticacion(self):
        """Middleware de autenticación"""
        auth_header = self.headers.get('Authorization')
        
        if not auth_header:
            self.enviar_error(401, 'Token requerido')
            return None
        
        partes = auth_header.split()
        if len(partes) != 2 or partes[0] != 'Bearer':
            self.enviar_error(401, 'Formato inválido')
            return None
        
        try:
            return auth_service.verificar_token(partes[1])
        except ValueError as e:
            self.enviar_error(401, str(e))
            return None
    
    def enviar_json(self, datos, status=200):
        """Envía JSON"""
        self.send_response(status)
        self.send_header('Content-type', 'application/json')
        self.end_headers()
        self.wfile.write(json.dumps(datos, ensure_ascii=False, indent=2).encode())
    
    def enviar_error(self, codigo, mensaje):
        """Envía error"""
        self.send_response(codigo)
        self.send_header('Content-type', 'application/json')
        self.end_headers()
        error = {'error': mensaje, 'codigo': codigo}
        self.wfile.write(json.dumps(error, ensure_ascii=False).encode())
    
    def leer_json_body(self):
        """Lee JSON"""
        content_length = int(self.headers.get('Content-Length', 0))
        if content_length == 0:
            return None
        
        body_raw = self.rfile.read(content_length)
        try:
            return json.loads(body_raw.decode('utf-8'))
        except json.JSONDecodeError:
            return None
    
    def log_message(self, format, *args):
        """Log"""
        print(f'📥 {self.command} {self.path}')

if __name__ == '__main__':
    puerto = 8000
    servidor = HTTPServer(('localhost', puerto), RolesAPIHandler)
    
    print('='*60)
    print('🔐 API con Roles y Permisos')
    print('='*60)
    print(f'\n🚀 Servidor: http://localhost:{puerto}')
    print('\n✨ Roles disponibles:')
    print('   - super_admin: Acceso total')
    print('   - admin: Gestión completa')
    print('   - moderador: Moderar contenido')
    print('   - usuario: Acceso básico')
    print('   - invitado: Solo lectura')
    print('\n⛔ Ctrl+C para detener\n')
    
    try:
        servidor.serve_forever()
    except KeyboardInterrupt:
        print('\n\n⛔ Servidor detenido')
        servidor.shutdown()
```

---

## Resumen

Has aprendido:

✅ Sistema de roles jerárquico  
✅ Permisos granulares por acción  
✅ Gestión de sesiones múltiples  
✅ Logout de sesiones específicas  
✅ Tracking de IP y user-agent  
✅ Sistema completo de autenticación

**Siguiente:** Bloque 6 - Arquitectura

---

## Recursos

- **[RBAC](https://en.wikipedia.org/wiki/Role-based_access_control)** - Control de acceso basado en roles
- **[OAuth 2.0](https://oauth.net/2/)** - Estándar de autorización

Sistema de autenticación profesional completo. 🔐

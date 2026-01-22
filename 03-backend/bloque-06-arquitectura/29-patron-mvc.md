# Patrón MVC (Model-View-Controller)

> **Separa lógica de negocio, presentación y control en componentes independientes**

---

## ¿Qué es MVC?

**Model-View-Controller** es un patrón arquitectónico que divide una aplicación en tres componentes interconectados:

| Componente | Responsabilidad | Ejemplo |
|------------|-----------------|---------|
| **Model** | Datos y lógica de negocio | `Usuario`, `Producto` |
| **View** | Presentación (UI/JSON) | Respuestas JSON, HTML |
| **Controller** | Maneja solicitudes | Rutas HTTP |

---

## Flujo MVC

```
Cliente → Controller → Model → Controller → View → Cliente
          (Procesa)   (Opera)  (Prepara)   (Formato)
```

**Proceso:**
1. Cliente hace request HTTP
2. Controller recibe y valida
3. Controller usa Model para lógica
4. Model retorna datos
5. Controller prepara respuesta
6. View formatea (JSON)
7. Cliente recibe respuesta

---

## Sin patrón (Todo mezclado)

**app_sin_mvc.py**

```python
"""
API sin separación de responsabilidades
Todo mezclado en el handler
"""

from http.server import BaseHTTPRequestHandler, HTTPServer
import json
from urllib.parse import urlparse

# Base de datos en memoria
usuarios_db = {}
siguiente_id = 1

class APIHandler(BaseHTTPRequestHandler):
    """Handler monolítico - todo mezclado"""
    
    def do_POST(self):
        """POST /api/usuarios"""
        if self.path == '/api/usuarios':
            try:
                # Leer datos
                content_length = int(self.headers['Content-Length'])
                body = self.rfile.read(content_length)
                datos = json.loads(body)
                
                # Validación mezclada con lógica
                if not datos.get('email'):
                    self.send_response(400)
                    self.send_header('Content-type', 'application/json')
                    self.end_headers()
                    self.wfile.write(json.dumps({
                        'error': 'Email requerido'
                    }).encode())
                    return
                
                # Validar formato email (mezclado)
                if '@' not in datos['email']:
                    self.send_response(400)
                    self.send_header('Content-type', 'application/json')
                    self.end_headers()
                    self.wfile.write(json.dumps({
                        'error': 'Email inválido'
                    }).encode())
                    return
                
                # Verificar duplicados (lógica de negocio mezclada)
                for usuario in usuarios_db.values():
                    if usuario['email'] == datos['email']:
                        self.send_response(400)
                        self.send_header('Content-type', 'application/json')
                        self.end_headers()
                        self.wfile.write(json.dumps({
                            'error': 'Email ya existe'
                        }).encode())
                        return
                
                # Crear usuario (persistencia mezclada)
                global siguiente_id
                usuario = {
                    'id': siguiente_id,
                    'nombre': datos.get('nombre'),
                    'email': datos['email'],
                    'activo': True
                }
                usuarios_db[siguiente_id] = usuario
                siguiente_id += 1
                
                # Respuesta (presentación mezclada)
                self.send_response(201)
                self.send_header('Content-type', 'application/json')
                self.end_headers()
                self.wfile.write(json.dumps(usuario).encode())
                
            except Exception as e:
                self.send_response(500)
                self.send_header('Content-type', 'application/json')
                self.end_headers()
                self.wfile.write(json.dumps({
                    'error': str(e)
                }).encode())
        else:
            self.send_response(404)
            self.end_headers()

if __name__ == '__main__':
    servidor = HTTPServer(('localhost', 8000), APIHandler)
    print('🚀 Servidor en http://localhost:8000')
    servidor.serve_forever()
```

**Problemas:**
- ❌ Validación, lógica y persistencia mezcladas
- ❌ Difícil de testear
- ❌ No reutilizable
- ❌ Difícil de mantener

---

## Con MVC: Model

**models/usuario.py**

```python
"""
Model: Representa datos y lógica de negocio
"""

from dataclasses import dataclass
from typing import Optional
import re

@dataclass
class Usuario:
    """Modelo de usuario con validaciones"""
    id: int
    nombre: str
    email: str
    activo: bool = True
    
    def __post_init__(self):
        """Validación al crear"""
        self.validar()
    
    def validar(self):
        """Valida datos del modelo"""
        if not self.email:
            raise ValueError('Email requerido')
        
        if not self._validar_email(self.email):
            raise ValueError('Email inválido')
        
        if not self.nombre or len(self.nombre) < 2:
            raise ValueError('Nombre debe tener al menos 2 caracteres')
    
    @staticmethod
    def _validar_email(email: str) -> bool:
        """Valida formato de email"""
        patron = r'^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$'
        return re.match(patron, email) is not None
    
    def activar(self):
        """Activa el usuario"""
        self.activo = True
    
    def desactivar(self):
        """Desactiva el usuario"""
        self.activo = False
    
    def to_dict(self, incluir_privado: bool = False) -> dict:
        """Convierte a diccionario para serialización"""
        data = {
            'id': self.id,
            'nombre': self.nombre,
            'email': self.email,
            'activo': self.activo
        }
        return data
    
    @classmethod
    def from_dict(cls, data: dict, id_usuario: int = None) -> 'Usuario':
        """Crea usuario desde diccionario"""
        return cls(
            id=id_usuario or data.get('id'),
            nombre=data.get('nombre', ''),
            email=data.get('email', ''),
            activo=data.get('activo', True)
        )
```

---

## Con MVC: Repository (Persistencia)

**repositories/usuario_repository.py**

```python
"""
Repository: Maneja persistencia de datos
Separa el Model de la base de datos
"""

from typing import Dict, Optional, List
from models.usuario import Usuario

class UsuarioRepository:
    """Repositorio de usuarios en memoria"""
    
    def __init__(self):
        self._usuarios: Dict[int, Usuario] = {}
        self._siguiente_id = 1
    
    def crear(self, usuario: Usuario) -> Usuario:
        """Crea un nuevo usuario"""
        # Asignar ID
        usuario.id = self._siguiente_id
        self._siguiente_id += 1
        
        # Guardar
        self._usuarios[usuario.id] = usuario
        return usuario
    
    def obtener_por_id(self, id_usuario: int) -> Optional[Usuario]:
        """Obtiene usuario por ID"""
        return self._usuarios.get(id_usuario)
    
    def obtener_por_email(self, email: str) -> Optional[Usuario]:
        """Obtiene usuario por email"""
        for usuario in self._usuarios.values():
            if usuario.email == email:
                return usuario
        return None
    
    def obtener_todos(self) -> List[Usuario]:
        """Obtiene todos los usuarios"""
        return list(self._usuarios.values())
    
    def actualizar(self, usuario: Usuario) -> Usuario:
        """Actualiza un usuario existente"""
        if usuario.id not in self._usuarios:
            raise ValueError(f'Usuario {usuario.id} no existe')
        
        usuario.validar()
        self._usuarios[usuario.id] = usuario
        return usuario
    
    def eliminar(self, id_usuario: int) -> bool:
        """Elimina un usuario"""
        if id_usuario in self._usuarios:
            del self._usuarios[id_usuario]
            return True
        return False
    
    def existe_email(self, email: str) -> bool:
        """Verifica si existe un email"""
        return self.obtener_por_email(email) is not None

# Instancia global (Singleton simple)
usuario_repository = UsuarioRepository()
```

---

## Con MVC: Service (Lógica de negocio)

**services/usuario_service.py**

```python
"""
Service: Lógica de negocio y reglas
Coordina entre Repository y Controller
"""

from typing import List, Optional
from models.usuario import Usuario
from repositories.usuario_repository import UsuarioRepository

class UsuarioService:
    """Servicio de lógica de negocio para usuarios"""
    
    def __init__(self, repository: UsuarioRepository):
        self.repository = repository
    
    def crear_usuario(self, datos: dict) -> Usuario:
        """
        Crea un nuevo usuario con validaciones de negocio
        
        Raises:
            ValueError: Si los datos son inválidos o email existe
        """
        # Verificar si email ya existe
        if self.repository.existe_email(datos.get('email', '')):
            raise ValueError('El email ya está registrado')
        
        # Crear modelo (validación automática en __post_init__)
        usuario = Usuario.from_dict(datos)
        
        # Guardar en repositorio
        return self.repository.crear(usuario)
    
    def obtener_usuario(self, id_usuario: int) -> Usuario:
        """
        Obtiene un usuario por ID
        
        Raises:
            ValueError: Si no existe
        """
        usuario = self.repository.obtener_por_id(id_usuario)
        if not usuario:
            raise ValueError(f'Usuario {id_usuario} no encontrado')
        return usuario
    
    def listar_usuarios(self, solo_activos: bool = False) -> List[Usuario]:
        """Lista todos los usuarios"""
        usuarios = self.repository.obtener_todos()
        
        if solo_activos:
            usuarios = [u for u in usuarios if u.activo]
        
        return usuarios
    
    def actualizar_usuario(self, id_usuario: int, datos: dict) -> Usuario:
        """
        Actualiza un usuario existente
        
        Raises:
            ValueError: Si no existe o datos inválidos
        """
        # Obtener usuario existente
        usuario = self.obtener_usuario(id_usuario)
        
        # Actualizar campos
        if 'nombre' in datos:
            usuario.nombre = datos['nombre']
        if 'email' in datos:
            # Verificar si nuevo email ya existe en otro usuario
            existente = self.repository.obtener_por_email(datos['email'])
            if existente and existente.id != id_usuario:
                raise ValueError('El email ya está en uso')
            usuario.email = datos['email']
        if 'activo' in datos:
            usuario.activo = datos['activo']
        
        # Validar y guardar
        return self.repository.actualizar(usuario)
    
    def eliminar_usuario(self, id_usuario: int) -> bool:
        """
        Elimina un usuario
        
        Raises:
            ValueError: Si no existe
        """
        usuario = self.obtener_usuario(id_usuario)
        return self.repository.eliminar(usuario.id)
    
    def activar_usuario(self, id_usuario: int) -> Usuario:
        """Activa un usuario"""
        usuario = self.obtener_usuario(id_usuario)
        usuario.activar()
        return self.repository.actualizar(usuario)
    
    def desactivar_usuario(self, id_usuario: int) -> Usuario:
        """Desactiva un usuario"""
        usuario = self.obtener_usuario(id_usuario)
        usuario.desactivar()
        return self.repository.actualizar(usuario)

# Dependencias
from repositories.usuario_repository import usuario_repository
usuario_service = UsuarioService(usuario_repository)
```

---

## Con MVC: Controller

**controllers/usuario_controller.py**

```python
"""
Controller: Maneja solicitudes HTTP
Coordina entre Service y respuesta HTTP
"""

from http.server import BaseHTTPRequestHandler
from urllib.parse import urlparse, parse_qs
import json
from typing import Optional

from services.usuario_service import UsuarioService

class UsuarioController:
    """Controller para endpoints de usuarios"""
    
    def __init__(self, service: UsuarioService):
        self.service = service
    
    def manejar_request(self, handler: BaseHTTPRequestHandler):
        """
        Enruta la solicitud al método apropiado
        
        Args:
            handler: Instancia de BaseHTTPRequestHandler
        """
        ruta = urlparse(handler.path)
        path = ruta.path
        metodo = handler.command
        
        # Rutas
        if path == '/api/usuarios':
            if metodo == 'GET':
                self.listar(handler, ruta)
            elif metodo == 'POST':
                self.crear(handler)
        elif path.startswith('/api/usuarios/'):
            # Extraer ID
            partes = path.split('/')
            if len(partes) == 4 and partes[3].isdigit():
                id_usuario = int(partes[3])
                
                if metodo == 'GET':
                    self.obtener(handler, id_usuario)
                elif metodo == 'PUT':
                    self.actualizar(handler, id_usuario)
                elif metodo == 'DELETE':
                    self.eliminar(handler, id_usuario)
        else:
            self._enviar_error(handler, 404, 'Ruta no encontrada')
    
    def listar(self, handler: BaseHTTPRequestHandler, ruta):
        """GET /api/usuarios"""
        try:
            # Query params
            params = parse_qs(ruta.query)
            solo_activos = params.get('activos', ['false'])[0] == 'true'
            
            usuarios = self.service.listar_usuarios(solo_activos)
            
            self._enviar_json(handler, {
                'usuarios': [u.to_dict() for u in usuarios],
                'total': len(usuarios)
            })
        except Exception as e:
            self._enviar_error(handler, 500, str(e))
    
    def crear(self, handler: BaseHTTPRequestHandler):
        """POST /api/usuarios"""
        try:
            datos = self._leer_json(handler)
            if not datos:
                self._enviar_error(handler, 400, 'Datos requeridos')
                return
            
            usuario = self.service.crear_usuario(datos)
            
            self._enviar_json(handler, usuario.to_dict(), status=201)
        except ValueError as e:
            self._enviar_error(handler, 400, str(e))
        except Exception as e:
            self._enviar_error(handler, 500, str(e))
    
    def obtener(self, handler: BaseHTTPRequestHandler, id_usuario: int):
        """GET /api/usuarios/:id"""
        try:
            usuario = self.service.obtener_usuario(id_usuario)
            self._enviar_json(handler, usuario.to_dict(incluir_privado=True))
        except ValueError as e:
            self._enviar_error(handler, 404, str(e))
        except Exception as e:
            self._enviar_error(handler, 500, str(e))
    
    def actualizar(self, handler: BaseHTTPRequestHandler, id_usuario: int):
        """PUT /api/usuarios/:id"""
        try:
            datos = self._leer_json(handler)
            if not datos:
                self._enviar_error(handler, 400, 'Datos requeridos')
                return
            
            usuario = self.service.actualizar_usuario(id_usuario, datos)
            self._enviar_json(handler, usuario.to_dict())
        except ValueError as e:
            self._enviar_error(handler, 400, str(e))
        except Exception as e:
            self._enviar_error(handler, 500, str(e))
    
    def eliminar(self, handler: BaseHTTPRequestHandler, id_usuario: int):
        """DELETE /api/usuarios/:id"""
        try:
            self.service.eliminar_usuario(id_usuario)
            self._enviar_json(handler, {'mensaje': 'Usuario eliminado'})
        except ValueError as e:
            self._enviar_error(handler, 404, str(e))
        except Exception as e:
            self._enviar_error(handler, 500, str(e))
    
    @staticmethod
    def _leer_json(handler: BaseHTTPRequestHandler) -> Optional[dict]:
        """Lee y parsea JSON del body"""
        content_length = int(handler.headers.get('Content-Length', 0))
        if content_length == 0:
            return None
        
        body = handler.rfile.read(content_length)
        try:
            return json.loads(body.decode('utf-8'))
        except json.JSONDecodeError:
            return None
    
    @staticmethod
    def _enviar_json(handler: BaseHTTPRequestHandler, datos: dict, status: int = 200):
        """Envía respuesta JSON"""
        handler.send_response(status)
        handler.send_header('Content-type', 'application/json')
        handler.end_headers()
        handler.wfile.write(json.dumps(datos, ensure_ascii=False, indent=2).encode())
    
    @staticmethod
    def _enviar_error(handler: BaseHTTPRequestHandler, codigo: int, mensaje: str):
        """Envía error JSON"""
        handler.send_response(codigo)
        handler.send_header('Content-type', 'application/json')
        handler.end_headers()
        error = {'error': mensaje, 'codigo': codigo}
        handler.wfile.write(json.dumps(error, ensure_ascii=False).encode())
```

---

## Aplicación completa con MVC

**app_mvc.py**

```python
"""
Aplicación completa con patrón MVC
"""

from http.server import BaseHTTPRequestHandler, HTTPServer
from urllib.parse import urlparse

# Importar controller configurado
from services.usuario_service import usuario_service
from controllers.usuario_controller import UsuarioController

# Crear controller con dependencias
usuario_controller = UsuarioController(usuario_service)

class MVCHandler(BaseHTTPRequestHandler):
    """Handler que delega al controller"""
    
    def do_GET(self):
        """Maneja GET"""
        self._manejar()
    
    def do_POST(self):
        """Maneja POST"""
        self._manejar()
    
    def do_PUT(self):
        """Maneja PUT"""
        self._manejar()
    
    def do_DELETE(self):
        """Maneja DELETE"""
        self._manejar()
    
    def _manejar(self):
        """Delega al controller apropiado"""
        ruta = urlparse(self.path).path
        
        if ruta == '/':
            self.home()
        elif ruta.startswith('/api/usuarios'):
            usuario_controller.manejar_request(self)
        else:
            self.send_response(404)
            self.end_headers()
    
    def home(self):
        """GET /"""
        html = """
        <!DOCTYPE html>
        <html>
        <head>
            <title>API MVC</title>
        </head>
        <body>
            <h1>🎯 API con Patrón MVC</h1>
            <h2>Endpoints:</h2>
            <ul>
                <li>GET /api/usuarios - Lista usuarios</li>
                <li>GET /api/usuarios?activos=true - Solo activos</li>
                <li>GET /api/usuarios/:id - Ver usuario</li>
                <li>POST /api/usuarios - Crear usuario</li>
                <li>PUT /api/usuarios/:id - Actualizar usuario</li>
                <li>DELETE /api/usuarios/:id - Eliminar usuario</li>
            </ul>
            
            <h2>Estructura MVC:</h2>
            <ul>
                <li><b>Model:</b> models/usuario.py</li>
                <li><b>Repository:</b> repositories/usuario_repository.py</li>
                <li><b>Service:</b> services/usuario_service.py</li>
                <li><b>Controller:</b> controllers/usuario_controller.py</li>
            </ul>
        </body>
        </html>
        """
        
        self.send_response(200)
        self.send_header('Content-type', 'text/html')
        self.end_headers()
        self.wfile.write(html.encode())
    
    def log_message(self, format, *args):
        """Log personalizado"""
        print(f'📥 {self.command} {self.path}')

if __name__ == '__main__':
    puerto = 8000
    servidor = HTTPServer(('localhost', puerto), MVCHandler)
    
    print('='*60)
    print('🎯 API con Patrón MVC')
    print('='*60)
    print(f'\n🚀 Servidor: http://localhost:{puerto}')
    print('\n📁 Estructura:')
    print('   - models/usuario.py (Model)')
    print('   - repositories/usuario_repository.py (Repository)')
    print('   - services/usuario_service.py (Service)')
    print('   - controllers/usuario_controller.py (Controller)')
    print('\n⛔ Ctrl+C para detener\n')
    
    try:
        servidor.serve_forever()
    except KeyboardInterrupt:
        print('\n\n⛔ Servidor detenido')
        servidor.shutdown()
```

---

## Estructura de directorios

```
proyecto/
├── app_mvc.py              # Aplicación principal
├── models/
│   ├── __init__.py
│   └── usuario.py          # Model
├── repositories/
│   ├── __init__.py
│   └── usuario_repository.py  # Repository
├── services/
│   ├── __init__.py
│   └── usuario_service.py  # Service (lógica negocio)
└── controllers/
    ├── __init__.py
    └── usuario_controller.py  # Controller (HTTP)
```

---

## Pruebas con curl

```bash
# Crear usuario
curl -X POST http://localhost:8000/api/usuarios \
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

# Listar usuarios
curl http://localhost:8000/api/usuarios

# Ver usuario específico
curl http://localhost:8000/api/usuarios/1

# Actualizar usuario
curl -X PUT http://localhost:8000/api/usuarios/1 \
  -H "Content-Type: application/json" \
  -d '{
    "nombre": "Ana María García"
  }'

# Listar solo activos
curl "http://localhost:8000/api/usuarios?activos=true"

# Eliminar usuario
curl -X DELETE http://localhost:8000/api/usuarios/1
```

---

## Ventajas de MVC

✅ **Separación de responsabilidades**
- Cada capa tiene una función clara
- Fácil de entender y navegar

✅ **Testeable**
- Puedes testear Model sin HTTP
- Puedes testear Service sin base de datos

✅ **Reutilizable**
- Service puede usarse desde API, CLI, tests
- Model puede usarse en diferentes contextos

✅ **Mantenible**
- Cambios en persistencia no afectan lógica
- Cambios en HTTP no afectan Model

✅ **Escalable**
- Fácil agregar nuevos endpoints
- Fácil cambiar base de datos

---

## Comparación sin/con MVC

| Aspecto | Sin MVC | Con MVC |
|---------|---------|---------|
| **Estructura** | Todo en handler | Separado por capas |
| **Testear** | Requiere HTTP | Cada capa independiente |
| **Reutilizar** | Imposible | Service reutilizable |
| **Mantener** | Difícil encontrar código | Claro dónde está cada cosa |
| **Cambios** | Tocar todo | Solo la capa afectada |

---

## Ejemplo con múltiples controllers

**app_multi_mvc.py**

```python
"""
Aplicación con múltiples controllers
"""

from http.server import BaseHTTPRequestHandler, HTTPServer
from urllib.parse import urlparse

# Controllers
from controllers.usuario_controller import usuario_controller
# from controllers.producto_controller import producto_controller
# from controllers.pedido_controller import pedido_controller

class MultiMVCHandler(BaseHTTPRequestHandler):
    """Handler que enruta a múltiples controllers"""
    
    def do_GET(self):
        self._manejar()
    
    def do_POST(self):
        self._manejar()
    
    def do_PUT(self):
        self._manejar()
    
    def do_DELETE(self):
        self._manejar()
    
    def _manejar(self):
        """Enruta a controller apropiado"""
        ruta = urlparse(self.path).path
        
        if ruta == '/':
            self.home()
        elif ruta.startswith('/api/usuarios'):
            usuario_controller.manejar_request(self)
        # elif ruta.startswith('/api/productos'):
        #     producto_controller.manejar_request(self)
        # elif ruta.startswith('/api/pedidos'):
        #     pedido_controller.manejar_request(self)
        else:
            self.send_response(404)
            self.send_header('Content-type', 'application/json')
            self.end_headers()
            self.wfile.write(b'{"error": "Ruta no encontrada"}')
    
    def home(self):
        """GET /"""
        html = """
        <!DOCTYPE html>
        <html>
        <head>
            <title>Multi-Controller MVC</title>
        </head>
        <body>
            <h1>🎯 Multi-Controller MVC</h1>
            <h2>Recursos disponibles:</h2>
            <ul>
                <li>/api/usuarios - Gestión de usuarios</li>
                <li>/api/productos - Gestión de productos</li>
                <li>/api/pedidos - Gestión de pedidos</li>
            </ul>
        </body>
        </html>
        """
        
        self.send_response(200)
        self.send_header('Content-type', 'text/html')
        self.end_headers()
        self.wfile.write(html.encode())
    
    def log_message(self, format, *args):
        print(f'📥 {self.command} {self.path}')

if __name__ == '__main__':
    servidor = HTTPServer(('localhost', 8000), MultiMVCHandler)
    print('🚀 Multi-Controller MVC en http://localhost:8000')
    servidor.serve_forever()
```

---

## Testing de cada capa

**tests/test_usuario_model.py**

```python
"""
Test del Model (sin HTTP, sin DB)
"""

import pytest
from models.usuario import Usuario

def test_crear_usuario_valido():
    """Crea usuario con datos válidos"""
    usuario = Usuario(
        id=1,
        nombre='Ana',
        email='ana@example.com'
    )
    
    assert usuario.id == 1
    assert usuario.nombre == 'Ana'
    assert usuario.email == 'ana@example.com'
    assert usuario.activo == True

def test_email_invalido():
    """Rechaza email inválido"""
    with pytest.raises(ValueError, match='Email inválido'):
        Usuario(
            id=1,
            nombre='Ana',
            email='email-invalido'
        )

def test_to_dict():
    """Convierte a diccionario"""
    usuario = Usuario(
        id=1,
        nombre='Ana',
        email='ana@example.com'
    )
    
    data = usuario.to_dict()
    
    assert data == {
        'id': 1,
        'nombre': 'Ana',
        'email': 'ana@example.com',
        'activo': True
    }
```

**tests/test_usuario_service.py**

```python
"""
Test del Service (sin HTTP)
"""

import pytest
from models.usuario import Usuario
from repositories.usuario_repository import UsuarioRepository
from services.usuario_service import UsuarioService

@pytest.fixture
def service():
    """Service con repositorio limpio"""
    repository = UsuarioRepository()
    return UsuarioService(repository)

def test_crear_usuario(service):
    """Crea usuario correctamente"""
    datos = {
        'nombre': 'Ana',
        'email': 'ana@example.com'
    }
    
    usuario = service.crear_usuario(datos)
    
    assert usuario.id == 1
    assert usuario.nombre == 'Ana'
    assert usuario.email == 'ana@example.com'

def test_email_duplicado(service):
    """Rechaza email duplicado"""
    datos = {'nombre': 'Ana', 'email': 'ana@example.com'}
    
    service.crear_usuario(datos)
    
    with pytest.raises(ValueError, match='ya está registrado'):
        service.crear_usuario(datos)

def test_listar_solo_activos(service):
    """Lista solo usuarios activos"""
    service.crear_usuario({'nombre': 'Ana', 'email': 'ana@example.com'})
    service.crear_usuario({'nombre': 'Bob', 'email': 'bob@example.com'})
    
    # Desactivar uno
    service.desactivar_usuario(1)
    
    activos = service.listar_usuarios(solo_activos=True)
    
    assert len(activos) == 1
    assert activos[0].nombre == 'Bob'
```

---

## Resumen

Has aprendido:

✅ Separar responsabilidades con MVC  
✅ Model para datos y validaciones  
✅ Repository para persistencia  
✅ Service para lógica de negocio  
✅ Controller para manejar HTTP  
✅ Testing independiente de cada capa

**Siguiente:** Service Layer Pattern

---

## Recursos

- **[MVC Pattern](https://en.wikipedia.org/wiki/Model%E2%80%93view%E2%80%93controller)** - Wikipedia
- **[Separation of Concerns](https://en.wikipedia.org/wiki/Separation_of_concerns)** - Principio de diseño

Código limpio y mantenible con MVC. 🎯

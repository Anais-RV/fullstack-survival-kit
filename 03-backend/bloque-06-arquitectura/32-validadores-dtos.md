# Validadores y DTOs (Data Transfer Objects)

> **Valida y transforma datos entre capas de la aplicación**

---

## ¿Qué son DTOs?

**Data Transfer Object (DTO)** es un objeto que:
- Transporta datos entre capas
- Define estructura de entrada/salida
- Valida datos antes de procesarlos
- Serializa/deserializa datos

**Ventajas:**
- ✅ Validación centralizada
- ✅ Documentación clara de API
- ✅ Separación entre API y dominio
- ✅ Transformaciones explícitas

---

## Sin validación (directo)

**controllers/usuario_controller.py**

```python
"""
Controller sin validación ni DTOs
"""

def crear_usuario(self, handler):
    """POST /api/usuarios - Sin validación"""
    datos = self._leer_json(handler)
    
    # ❌ Sin validación, cualquier dato pasa
    usuario = self.service.crear_usuario(datos)
    
    self._enviar_json(handler, usuario.to_dict())
```

**Problemas:**
- ❌ Datos inválidos llegan al service
- ❌ Errores de validación dispersos
- ❌ Sin documentación de qué campos son necesarios
- ❌ Sin validación de tipos

---

## DTO básico con validación manual

**dtos/usuario_dto.py**

```python
"""
DTOs para usuario con validación manual
"""

from dataclasses import dataclass
from typing import Optional
import re

@dataclass
class CrearUsuarioDTO:
    """DTO para crear usuario"""
    nombre: str
    email: str
    password: str
    
    def validar(self) -> list:
        """
        Valida los datos
        
        Returns:
            Lista de errores (vacía si es válido)
        """
        errores = []
        
        # Validar nombre
        if not self.nombre:
            errores.append('nombre es requerido')
        elif len(self.nombre) < 2:
            errores.append('nombre debe tener al menos 2 caracteres')
        elif len(self.nombre) > 100:
            errores.append('nombre no puede tener más de 100 caracteres')
        
        # Validar email
        if not self.email:
            errores.append('email es requerido')
        elif not self._validar_email(self.email):
            errores.append('email inválido')
        
        # Validar password
        if not self.password:
            errores.append('password es requerido')
        elif len(self.password) < 8:
            errores.append('password debe tener al menos 8 caracteres')
        elif not any(c.isupper() for c in self.password):
            errores.append('password debe tener al menos una mayúscula')
        elif not any(c.isdigit() for c in self.password):
            errores.append('password debe tener al menos un número')
        
        return errores
    
    @staticmethod
    def _validar_email(email: str) -> bool:
        """Valida formato de email"""
        patron = r'^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$'
        return re.match(patron, email) is not None
    
    @classmethod
    def from_dict(cls, data: dict) -> 'CrearUsuarioDTO':
        """Crea DTO desde diccionario"""
        return cls(
            nombre=data.get('nombre', ''),
            email=data.get('email', ''),
            password=data.get('password', '')
        )

@dataclass
class ActualizarUsuarioDTO:
    """DTO para actualizar usuario"""
    nombre: Optional[str] = None
    email: Optional[str] = None
    activo: Optional[bool] = None
    
    def validar(self) -> list:
        """Valida los datos"""
        errores = []
        
        if self.nombre is not None:
            if len(self.nombre) < 2:
                errores.append('nombre debe tener al menos 2 caracteres')
            elif len(self.nombre) > 100:
                errores.append('nombre no puede tener más de 100 caracteres')
        
        if self.email is not None:
            if not self._validar_email(self.email):
                errores.append('email inválido')
        
        return errores
    
    @staticmethod
    def _validar_email(email: str) -> bool:
        patron = r'^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$'
        return re.match(patron, email) is not None
    
    @classmethod
    def from_dict(cls, data: dict) -> 'ActualizarUsuarioDTO':
        """Crea DTO desde diccionario"""
        return cls(
            nombre=data.get('nombre'),
            email=data.get('email'),
            activo=data.get('activo')
        )
    
    def to_dict(self) -> dict:
        """Convierte a diccionario (solo campos no-None)"""
        data = {}
        if self.nombre is not None:
            data['nombre'] = self.nombre
        if self.email is not None:
            data['email'] = self.email
        if self.activo is not None:
            data['activo'] = self.activo
        return data

@dataclass
class UsuarioResponseDTO:
    """DTO para respuesta de usuario"""
    id: int
    nombre: str
    email: str
    activo: bool
    
    @classmethod
    def from_usuario(cls, usuario) -> 'UsuarioResponseDTO':
        """Crea DTO desde modelo Usuario"""
        return cls(
            id=usuario.id,
            nombre=usuario.nombre,
            email=usuario.email,
            activo=usuario.activo
        )
    
    def to_dict(self) -> dict:
        """Serializa a diccionario"""
        return {
            'id': self.id,
            'nombre': self.nombre,
            'email': self.email,
            'activo': self.activo
        }
```

---

## Controller con DTOs

**controllers/usuario_controller.py**

```python
"""
Controller usando DTOs para validación
"""

from http.server import BaseHTTPRequestHandler
from urllib.parse import urlparse
import json
from typing import Optional

from services.usuario_service import UsuarioService
from dtos.usuario_dto import (
    CrearUsuarioDTO,
    ActualizarUsuarioDTO,
    UsuarioResponseDTO
)

class UsuarioController:
    """Controller con DTOs"""
    
    def __init__(self, service: UsuarioService):
        self.service = service
    
    def crear(self, handler: BaseHTTPRequestHandler):
        """POST /api/usuarios - Con validación DTO"""
        try:
            datos = self._leer_json(handler)
            if not datos:
                self._enviar_error(handler, 400, 'Datos requeridos')
                return
            
            # ✅ Crear DTO
            dto = CrearUsuarioDTO.from_dict(datos)
            
            # ✅ Validar
            errores = dto.validar()
            if errores:
                self._enviar_json(handler, {
                    'errores': errores
                }, status=400)
                return
            
            # ✅ Usar DTO en service
            usuario = self.service.crear_usuario(dto)
            
            # ✅ Respuesta con DTO
            response = UsuarioResponseDTO.from_usuario(usuario)
            self._enviar_json(handler, response.to_dict(), status=201)
        
        except ValueError as e:
            self._enviar_error(handler, 400, str(e))
        except Exception as e:
            self._enviar_error(handler, 500, str(e))
    
    def actualizar(self, handler: BaseHTTPRequestHandler, id_usuario: int):
        """PUT /api/usuarios/:id - Con validación DTO"""
        try:
            datos = self._leer_json(handler)
            if not datos:
                self._enviar_error(handler, 400, 'Datos requeridos')
                return
            
            # ✅ Crear DTO
            dto = ActualizarUsuarioDTO.from_dict(datos)
            
            # ✅ Validar
            errores = dto.validar()
            if errores:
                self._enviar_json(handler, {
                    'errores': errores
                }, status=400)
                return
            
            # ✅ Usar DTO en service
            usuario = self.service.actualizar_usuario(id_usuario, dto)
            
            # ✅ Respuesta con DTO
            response = UsuarioResponseDTO.from_usuario(usuario)
            self._enviar_json(handler, response.to_dict())
        
        except ValueError as e:
            self._enviar_error(handler, 400, str(e))
        except Exception as e:
            self._enviar_error(handler, 500, str(e))
    
    @staticmethod
    def _leer_json(handler: BaseHTTPRequestHandler) -> Optional[dict]:
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
        handler.send_response(status)
        handler.send_header('Content-type', 'application/json')
        handler.end_headers()
        handler.wfile.write(json.dumps(datos, ensure_ascii=False, indent=2).encode())
    
    @staticmethod
    def _enviar_error(handler: BaseHTTPRequestHandler, codigo: int, mensaje: str):
        handler.send_response(codigo)
        handler.send_header('Content-type', 'application/json')
        handler.end_headers()
        error = {'error': mensaje, 'codigo': codigo}
        handler.wfile.write(json.dumps(error, ensure_ascii=False).encode())
```

---

## Service con DTOs

**services/usuario_service.py**

```python
"""
Service que recibe DTOs validados
"""

from typing import List
from models.usuario import Usuario
from repositories.base_repository import BaseRepository
from dtos.usuario_dto import CrearUsuarioDTO, ActualizarUsuarioDTO

class UsuarioService:
    """Service con DTOs"""
    
    def __init__(self, repository: BaseRepository[Usuario]):
        self.repository = repository
    
    def crear_usuario(self, dto: CrearUsuarioDTO) -> Usuario:
        """
        Crea usuario desde DTO validado
        
        Args:
            dto: DTO ya validado
        
        Returns:
            Usuario creado
        """
        # Verificar email duplicado
        if hasattr(self.repository, 'existe_email'):
            if self.repository.existe_email(dto.email):
                raise ValueError('El email ya está registrado')
        
        # ✅ Convertir DTO a Model
        usuario = Usuario(
            id=None,
            nombre=dto.nombre,
            email=dto.email,
            activo=True
        )
        
        # Aquí iría hash del password (dto.password)
        # usuario.password_hash = hash_password(dto.password)
        
        return self.repository.crear(usuario)
    
    def actualizar_usuario(self, id_usuario: int, dto: ActualizarUsuarioDTO) -> Usuario:
        """
        Actualiza usuario desde DTO validado
        
        Args:
            id_usuario: ID del usuario
            dto: DTO ya validado con cambios
        
        Returns:
            Usuario actualizado
        """
        usuario = self.repository.obtener_por_id(id_usuario)
        if not usuario:
            raise ValueError(f'Usuario {id_usuario} no encontrado')
        
        # ✅ Aplicar cambios del DTO
        cambios = dto.to_dict()
        
        if 'nombre' in cambios:
            usuario.nombre = cambios['nombre']
        
        if 'email' in cambios:
            # Verificar email duplicado
            if hasattr(self.repository, 'existe_email'):
                existente = self.repository.obtener_por_email(cambios['email'])
                if existente and existente.id != id_usuario:
                    raise ValueError('El email ya está en uso')
            usuario.email = cambios['email']
        
        if 'activo' in cambios:
            usuario.activo = cambios['activo']
        
        return self.repository.actualizar(usuario)
```

---

## DTOs con Pydantic (librería)

**dtos/usuario_pydantic.py**

```python
"""
DTOs con Pydantic para validación automática
"""

from pydantic import BaseModel, EmailStr, Field, validator
from typing import Optional

class CrearUsuarioPydantic(BaseModel):
    """DTO para crear usuario con Pydantic"""
    nombre: str = Field(..., min_length=2, max_length=100)
    email: EmailStr
    password: str = Field(..., min_length=8)
    
    @validator('password')
    def validar_password(cls, v):
        """Valida complejidad del password"""
        if not any(c.isupper() for c in v):
            raise ValueError('password debe tener al menos una mayúscula')
        if not any(c.isdigit() for c in v):
            raise ValueError('password debe tener al menos un número')
        if not any(c in '!@#$%^&*()_+-=' for c in v):
            raise ValueError('password debe tener al menos un carácter especial')
        return v
    
    class Config:
        schema_extra = {
            'example': {
                'nombre': 'Ana García',
                'email': 'ana@example.com',
                'password': 'Segura123!'
            }
        }

class ActualizarUsuarioPydantic(BaseModel):
    """DTO para actualizar usuario con Pydantic"""
    nombre: Optional[str] = Field(None, min_length=2, max_length=100)
    email: Optional[EmailStr] = None
    activo: Optional[bool] = None
    
    class Config:
        schema_extra = {
            'example': {
                'nombre': 'Ana María García',
                'activo': True
            }
        }

class UsuarioResponsePydantic(BaseModel):
    """DTO para respuesta de usuario con Pydantic"""
    id: int
    nombre: str
    email: str
    activo: bool
    
    class Config:
        orm_mode = True  # Permite from_orm()
        schema_extra = {
            'example': {
                'id': 1,
                'nombre': 'Ana García',
                'email': 'ana@example.com',
                'activo': True
            }
        }
```

---

## Controller con Pydantic

**controllers/usuario_controller_pydantic.py**

```python
"""
Controller usando Pydantic DTOs
"""

from http.server import BaseHTTPRequestHandler
from urllib.parse import urlparse
import json
from pydantic import ValidationError

from services.usuario_service import UsuarioService
from dtos.usuario_pydantic import (
    CrearUsuarioPydantic,
    ActualizarUsuarioPydantic,
    UsuarioResponsePydantic
)

class UsuarioControllerPydantic:
    """Controller con Pydantic"""
    
    def __init__(self, service: UsuarioService):
        self.service = service
    
    def crear(self, handler: BaseHTTPRequestHandler):
        """POST /api/usuarios - Con Pydantic"""
        try:
            datos = self._leer_json(handler)
            if not datos:
                self._enviar_error(handler, 400, 'Datos requeridos')
                return
            
            # ✅ Pydantic valida automáticamente
            dto = CrearUsuarioPydantic(**datos)
            
            # Crear usuario
            usuario = self.service.crear_usuario_pydantic(dto)
            
            # ✅ Respuesta con Pydantic
            response = UsuarioResponsePydantic.from_orm(usuario)
            self._enviar_json(handler, response.dict(), status=201)
        
        except ValidationError as e:
            # ✅ Errores de validación de Pydantic
            self._enviar_json(handler, {
                'errores': [
                    {
                        'campo': '.'.join(str(x) for x in err['loc']),
                        'mensaje': err['msg'],
                        'tipo': err['type']
                    }
                    for err in e.errors()
                ]
            }, status=400)
        
        except ValueError as e:
            self._enviar_error(handler, 400, str(e))
        except Exception as e:
            self._enviar_error(handler, 500, str(e))
    
    def actualizar(self, handler: BaseHTTPRequestHandler, id_usuario: int):
        """PUT /api/usuarios/:id - Con Pydantic"""
        try:
            datos = self._leer_json(handler)
            if not datos:
                self._enviar_error(handler, 400, 'Datos requeridos')
                return
            
            # ✅ Pydantic valida automáticamente
            dto = ActualizarUsuarioPydantic(**datos)
            
            # Actualizar usuario
            usuario = self.service.actualizar_usuario_pydantic(id_usuario, dto)
            
            # ✅ Respuesta con Pydantic
            response = UsuarioResponsePydantic.from_orm(usuario)
            self._enviar_json(handler, response.dict())
        
        except ValidationError as e:
            self._enviar_json(handler, {
                'errores': [
                    {
                        'campo': '.'.join(str(x) for x in err['loc']),
                        'mensaje': err['msg']
                    }
                    for err in e.errors()
                ]
            }, status=400)
        
        except ValueError as e:
            self._enviar_error(handler, 400, str(e))
        except Exception as e:
            self._enviar_error(handler, 500, str(e))
    
    @staticmethod
    def _leer_json(handler):
        content_length = int(handler.headers.get('Content-Length', 0))
        if content_length == 0:
            return None
        body = handler.rfile.read(content_length)
        try:
            return json.loads(body.decode('utf-8'))
        except json.JSONDecodeError:
            return None
    
    @staticmethod
    def _enviar_json(handler, datos: dict, status: int = 200):
        handler.send_response(status)
        handler.send_header('Content-type', 'application/json')
        handler.end_headers()
        handler.wfile.write(json.dumps(datos, ensure_ascii=False, indent=2).encode())
    
    @staticmethod
    def _enviar_error(handler, codigo: int, mensaje: str):
        handler.send_response(codigo)
        handler.send_header('Content-type', 'application/json')
        handler.end_headers()
        error = {'error': mensaje, 'codigo': codigo}
        handler.wfile.write(json.dumps(error, ensure_ascii=False).encode())
```

---

## Aplicación completa

**app_dtos.py**

```python
"""
Aplicación con DTOs y validación
"""

from http.server import BaseHTTPRequestHandler, HTTPServer
from urllib.parse import urlparse

from repositories.usuario_repository_memory import UsuarioRepositoryMemory
from services.usuario_service import UsuarioService
from controllers.usuario_controller import UsuarioController

# Setup
usuario_repository = UsuarioRepositoryMemory()
usuario_service = UsuarioService(usuario_repository)
usuario_controller = UsuarioController(usuario_service)

class DTOHandler(BaseHTTPRequestHandler):
    """Handler principal"""
    
    def do_GET(self):
        self._manejar()
    
    def do_POST(self):
        self._manejar()
    
    def do_PUT(self):
        self._manejar()
    
    def _manejar(self):
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
            <title>DTOs y Validación</title>
        </head>
        <body>
            <h1>✅ DTOs y Validación</h1>
            
            <h2>Crear usuario:</h2>
            <pre>
curl -X POST http://localhost:8000/api/usuarios \\
  -H "Content-Type: application/json" \\
  -d '{
    "nombre": "Ana García",
    "email": "ana@example.com",
    "password": "Segura123!"
  }'
            </pre>
            
            <h2>Validaciones:</h2>
            <ul>
                <li>✅ nombre: mínimo 2 caracteres</li>
                <li>✅ email: formato válido</li>
                <li>✅ password: mínimo 8 caracteres, mayúscula, número</li>
            </ul>
            
            <h2>Errores de validación:</h2>
            <pre>
{
  "errores": [
    "nombre debe tener al menos 2 caracteres",
    "email inválido",
    "password debe tener al menos una mayúscula"
  ]
}
            </pre>
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
    puerto = 8000
    servidor = HTTPServer(('localhost', puerto), DTOHandler)
    
    print('='*60)
    print('✅ DTOs y Validación')
    print('='*60)
    print(f'\n🚀 Servidor: http://localhost:{puerto}')
    print('\n📋 Validaciones activas:')
    print('   - nombre: 2-100 caracteres')
    print('   - email: formato válido')
    print('   - password: 8+ chars, mayúscula, número')
    print('\n⛔ Ctrl+C para detener\n')
    
    try:
        servidor.serve_forever()
    except KeyboardInterrupt:
        print('\n\n⛔ Servidor detenido')
        servidor.shutdown()
```

---

## Pruebas con curl

```bash
# ✅ Usuario válido
curl -X POST http://localhost:8000/api/usuarios \
  -H "Content-Type: application/json" \
  -d '{
    "nombre": "Ana García",
    "email": "ana@example.com",
    "password": "Segura123!"
  }'

# Respuesta:
# {
#   "id": 1,
#   "nombre": "Ana García",
#   "email": "ana@example.com",
#   "activo": true
# }

# ❌ Nombre muy corto
curl -X POST http://localhost:8000/api/usuarios \
  -H "Content-Type: application/json" \
  -d '{
    "nombre": "A",
    "email": "ana@example.com",
    "password": "Segura123!"
  }'

# Respuesta:
# {
#   "errores": [
#     "nombre debe tener al menos 2 caracteres"
#   ]
# }

# ❌ Email inválido
curl -X POST http://localhost:8000/api/usuarios \
  -H "Content-Type: application/json" \
  -d '{
    "nombre": "Ana",
    "email": "email-invalido",
    "password": "Segura123!"
  }'

# Respuesta:
# {
#   "errores": [
#     "email inválido"
#   ]
# }

# ❌ Password débil
curl -X POST http://localhost:8000/api/usuarios \
  -H "Content-Type: application/json" \
  -d '{
    "nombre": "Ana",
    "email": "ana@example.com",
    "password": "corta"
  }'

# Respuesta:
# {
#   "errores": [
#     "password debe tener al menos 8 caracteres",
#     "password debe tener al menos una mayúscula",
#     "password debe tener al menos un número"
#   ]
# }
```

---

## Testing de DTOs

**tests/test_usuario_dto.py**

```python
"""
Test de DTOs
"""

import pytest
from dtos.usuario_dto import CrearUsuarioDTO, ActualizarUsuarioDTO

def test_crear_usuario_valido():
    """DTO válido no tiene errores"""
    dto = CrearUsuarioDTO(
        nombre='Ana García',
        email='ana@example.com',
        password='Segura123!'
    )
    
    errores = dto.validar()
    
    assert len(errores) == 0

def test_nombre_muy_corto():
    """Rechaza nombre corto"""
    dto = CrearUsuarioDTO(
        nombre='A',
        email='ana@example.com',
        password='Segura123!'
    )
    
    errores = dto.validar()
    
    assert any('al menos 2 caracteres' in e for e in errores)

def test_email_invalido():
    """Rechaza email inválido"""
    dto = CrearUsuarioDTO(
        nombre='Ana',
        email='email-invalido',
        password='Segura123!'
    )
    
    errores = dto.validar()
    
    assert any('email inválido' in e for e in errores)

def test_password_sin_mayuscula():
    """Rechaza password sin mayúscula"""
    dto = CrearUsuarioDTO(
        nombre='Ana',
        email='ana@example.com',
        password='segura123!'
    )
    
    errores = dto.validar()
    
    assert any('mayúscula' in e for e in errores)

def test_actualizar_parcial():
    """DTO de actualización con solo algunos campos"""
    dto = ActualizarUsuarioDTO(
        nombre='Nuevo Nombre'
    )
    
    errores = dto.validar()
    data = dto.to_dict()
    
    assert len(errores) == 0
    assert 'nombre' in data
    assert 'email' not in data
    assert 'activo' not in data
```

---

## Ventajas de DTOs

✅ **Validación centralizada**
- Todas las reglas en un lugar
- Reutilizable en múltiples endpoints

✅ **Documentación clara**
- DTO define exactamente qué espera la API
- Fácil generar documentación automática

✅ **Separación de capas**
- API independiente del modelo de dominio
- Cambios en API no afectan modelo

✅ **Transformaciones explícitas**
- Conversiones claramente definidas
- Fácil agregar lógica de transformación

✅ **Testeable**
- Fácil testear validaciones aisladamente
- No requiere HTTP ni base de datos

---

## Comparación sin/con DTOs

| Aspecto | Sin DTOs | Con DTOs |
|---------|----------|----------|
| **Validación** | Dispersa | Centralizada |
| **Documentación** | Implícita | Explícita |
| **Tipos** | Dinámicos | Definidos |
| **Errores** | En runtime | En validación |
| **Mantenibilidad** | Baja | Alta |

---

## Resumen

Has aprendido:

✅ Crear DTOs para entrada/salida  
✅ Validación centralizada de datos  
✅ DTOs manuales vs Pydantic  
✅ Transformación entre DTOs y Models  
✅ Errores de validación estructurados

**Siguiente:** Variables de entorno

---

## Recursos

- **[Pydantic](https://docs.pydantic.dev/)** - Validación con Python types
- **[DTO Pattern](https://martinfowler.com/eaaCatalog/dataTransferObject.html)** - Martin Fowler

Validación robusta y APIs documentadas. ✅

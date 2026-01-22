# Repository Pattern

> **Abstrae la persistencia de datos para independizar la lógica de negocio del almacenamiento**

---

## ¿Qué es Repository Pattern?

**Repository** es un patrón que:
- Encapsula lógica de acceso a datos
- Provee interfaz de colección para modelos
- Separa lógica de negocio de persistencia
- Facilita cambiar entre diferentes bases de datos

**Concepto clave:** El resto de la aplicación no necesita saber si los datos están en memoria, archivo, SQLite, PostgreSQL, etc.

---

## Sin Repository (acceso directo)

**services/usuario_service.py**

```python
"""
Service con acceso directo a la 'base de datos'
"""

# ❌ Base de datos global accesible desde cualquier lugar
usuarios_db = {}
siguiente_id = 1

class UsuarioService:
    """Service acoplado a la estructura de datos"""
    
    def crear_usuario(self, datos: dict):
        global siguiente_id
        
        # ❌ Lógica de persistencia mezclada con negocio
        usuario = {
            'id': siguiente_id,
            'nombre': datos['nombre'],
            'email': datos['email']
        }
        usuarios_db[siguiente_id] = usuario
        siguiente_id += 1
        
        return usuario
    
    def obtener_usuario(self, id_usuario: int):
        # ❌ Acceso directo a la estructura de datos
        return usuarios_db.get(id_usuario)
    
    def buscar_por_email(self, email: str):
        # ❌ Lógica de búsqueda en el service
        for usuario in usuarios_db.values():
            if usuario['email'] == email:
                return usuario
        return None
```

**Problemas:**
- ❌ Service acoplado a estructura de datos
- ❌ Difícil cambiar a otra DB
- ❌ Lógica de persistencia dispersa
- ❌ Difícil de testear
- ❌ No se puede reutilizar lógica de acceso

---

## Con Repository: Model

**models/usuario.py**

```python
"""
Model del dominio
"""

from dataclasses import dataclass
from typing import Optional
import re

@dataclass
class Usuario:
    """Modelo de usuario"""
    id: Optional[int]
    nombre: str
    email: str
    activo: bool = True
    
    def __post_init__(self):
        if self.id is not None:
            self.validar()
    
    def validar(self):
        """Validaciones del modelo"""
        if not self.nombre or len(self.nombre) < 2:
            raise ValueError('Nombre debe tener al menos 2 caracteres')
        
        if not self._validar_email(self.email):
            raise ValueError('Email inválido')
    
    @staticmethod
    def _validar_email(email: str) -> bool:
        """Valida formato de email"""
        patron = r'^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$'
        return re.match(patron, email) is not None
    
    def to_dict(self) -> dict:
        """Serializa a diccionario"""
        return {
            'id': self.id,
            'nombre': self.nombre,
            'email': self.email,
            'activo': self.activo
        }
    
    @classmethod
    def from_dict(cls, data: dict) -> 'Usuario':
        """Deserializa desde diccionario"""
        return cls(
            id=data.get('id'),
            nombre=data.get('nombre', ''),
            email=data.get('email', ''),
            activo=data.get('activo', True)
        )
```

---

## Con Repository: Repository Interface

**repositories/base_repository.py**

```python
"""
Interfaz base para repositories
Define el contrato que deben cumplir
"""

from abc import ABC, abstractmethod
from typing import TypeVar, Generic, List, Optional

T = TypeVar('T')

class BaseRepository(ABC, Generic[T]):
    """Interfaz base para repositories"""
    
    @abstractmethod
    def crear(self, entidad: T) -> T:
        """Crea una nueva entidad"""
        pass
    
    @abstractmethod
    def obtener_por_id(self, id_entidad: int) -> Optional[T]:
        """Obtiene entidad por ID"""
        pass
    
    @abstractmethod
    def obtener_todos(self) -> List[T]:
        """Obtiene todas las entidades"""
        pass
    
    @abstractmethod
    def actualizar(self, entidad: T) -> T:
        """Actualiza una entidad"""
        pass
    
    @abstractmethod
    def eliminar(self, id_entidad: int) -> bool:
        """Elimina una entidad"""
        pass
```

---

## Repository en Memoria

**repositories/usuario_repository_memory.py**

```python
"""
Repository de usuarios en memoria
Implementación específica para almacenamiento en RAM
"""

from typing import Dict, List, Optional
from models.usuario import Usuario
from repositories.base_repository import BaseRepository

class UsuarioRepositoryMemory(BaseRepository[Usuario]):
    """Repository de usuarios en memoria"""
    
    def __init__(self):
        self._usuarios: Dict[int, Usuario] = {}
        self._siguiente_id = 1
    
    def crear(self, usuario: Usuario) -> Usuario:
        """Crea un nuevo usuario"""
        # Asignar ID
        usuario.id = self._siguiente_id
        self._siguiente_id += 1
        
        # Validar
        usuario.validar()
        
        # Guardar
        self._usuarios[usuario.id] = usuario
        return usuario
    
    def obtener_por_id(self, id_usuario: int) -> Optional[Usuario]:
        """Obtiene usuario por ID"""
        return self._usuarios.get(id_usuario)
    
    def obtener_todos(self) -> List[Usuario]:
        """Obtiene todos los usuarios"""
        return list(self._usuarios.values())
    
    def actualizar(self, usuario: Usuario) -> Usuario:
        """Actualiza un usuario"""
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
    
    # Métodos específicos de búsqueda
    def obtener_por_email(self, email: str) -> Optional[Usuario]:
        """Obtiene usuario por email"""
        for usuario in self._usuarios.values():
            if usuario.email == email:
                return usuario
        return None
    
    def existe_email(self, email: str) -> bool:
        """Verifica si existe un email"""
        return self.obtener_por_email(email) is not None
    
    def buscar_por_nombre(self, termino: str) -> List[Usuario]:
        """Busca usuarios por nombre (case-insensitive)"""
        termino = termino.lower()
        return [
            u for u in self._usuarios.values()
            if termino in u.nombre.lower()
        ]
    
    def obtener_activos(self) -> List[Usuario]:
        """Obtiene solo usuarios activos"""
        return [u for u in self._usuarios.values() if u.activo]
```

---

## Repository en JSON

**repositories/usuario_repository_json.py**

```python
"""
Repository de usuarios en archivo JSON
Otra implementación con la misma interfaz
"""

import json
import os
from typing import List, Optional
from models.usuario import Usuario
from repositories.base_repository import BaseRepository

class UsuarioRepositoryJSON(BaseRepository[Usuario]):
    """Repository de usuarios en archivo JSON"""
    
    def __init__(self, archivo: str = 'usuarios.json'):
        self.archivo = archivo
        self._usuarios = self._cargar()
        self._siguiente_id = self._obtener_siguiente_id()
    
    def crear(self, usuario: Usuario) -> Usuario:
        """Crea un nuevo usuario"""
        usuario.id = self._siguiente_id
        self._siguiente_id += 1
        
        usuario.validar()
        
        self._usuarios[usuario.id] = usuario
        self._guardar()
        return usuario
    
    def obtener_por_id(self, id_usuario: int) -> Optional[Usuario]:
        """Obtiene usuario por ID"""
        return self._usuarios.get(id_usuario)
    
    def obtener_todos(self) -> List[Usuario]:
        """Obtiene todos los usuarios"""
        return list(self._usuarios.values())
    
    def actualizar(self, usuario: Usuario) -> Usuario:
        """Actualiza un usuario"""
        if usuario.id not in self._usuarios:
            raise ValueError(f'Usuario {usuario.id} no existe')
        
        usuario.validar()
        self._usuarios[usuario.id] = usuario
        self._guardar()
        return usuario
    
    def eliminar(self, id_usuario: int) -> bool:
        """Elimina un usuario"""
        if id_usuario in self._usuarios:
            del self._usuarios[id_usuario]
            self._guardar()
            return True
        return False
    
    def obtener_por_email(self, email: str) -> Optional[Usuario]:
        """Obtiene usuario por email"""
        for usuario in self._usuarios.values():
            if usuario.email == email:
                return usuario
        return None
    
    def existe_email(self, email: str) -> bool:
        """Verifica si existe un email"""
        return self.obtener_por_email(email) is not None
    
    def _cargar(self) -> dict:
        """Carga usuarios desde JSON"""
        if not os.path.exists(self.archivo):
            return {}
        
        try:
            with open(self.archivo, 'r', encoding='utf-8') as f:
                data = json.load(f)
                return {
                    int(id_str): Usuario.from_dict(u_dict)
                    for id_str, u_dict in data.items()
                }
        except (json.JSONDecodeError, IOError):
            return {}
    
    def _guardar(self):
        """Guarda usuarios a JSON"""
        data = {
            str(id_usuario): usuario.to_dict()
            for id_usuario, usuario in self._usuarios.items()
        }
        
        with open(self.archivo, 'w', encoding='utf-8') as f:
            json.dump(data, f, ensure_ascii=False, indent=2)
    
    def _obtener_siguiente_id(self) -> int:
        """Calcula el siguiente ID disponible"""
        if not self._usuarios:
            return 1
        return max(self._usuarios.keys()) + 1
```

---

## Repository con SQLite

**repositories/usuario_repository_sqlite.py**

```python
"""
Repository de usuarios en SQLite
Otra implementación más con la misma interfaz
"""

import sqlite3
from typing import List, Optional
from models.usuario import Usuario
from repositories.base_repository import BaseRepository

class UsuarioRepositorySQLite(BaseRepository[Usuario]):
    """Repository de usuarios en SQLite"""
    
    def __init__(self, db_path: str = 'usuarios.db'):
        self.db_path = db_path
        self._inicializar_db()
    
    def _inicializar_db(self):
        """Crea la tabla si no existe"""
        with sqlite3.connect(self.db_path) as conn:
            conn.execute('''
                CREATE TABLE IF NOT EXISTS usuarios (
                    id INTEGER PRIMARY KEY AUTOINCREMENT,
                    nombre TEXT NOT NULL,
                    email TEXT NOT NULL UNIQUE,
                    activo INTEGER DEFAULT 1
                )
            ''')
            conn.commit()
    
    def crear(self, usuario: Usuario) -> Usuario:
        """Crea un nuevo usuario"""
        usuario.validar()
        
        with sqlite3.connect(self.db_path) as conn:
            cursor = conn.execute(
                'INSERT INTO usuarios (nombre, email, activo) VALUES (?, ?, ?)',
                (usuario.nombre, usuario.email, 1 if usuario.activo else 0)
            )
            usuario.id = cursor.lastrowid
            conn.commit()
        
        return usuario
    
    def obtener_por_id(self, id_usuario: int) -> Optional[Usuario]:
        """Obtiene usuario por ID"""
        with sqlite3.connect(self.db_path) as conn:
            conn.row_factory = sqlite3.Row
            cursor = conn.execute(
                'SELECT * FROM usuarios WHERE id = ?',
                (id_usuario,)
            )
            row = cursor.fetchone()
        
        if row:
            return self._row_to_usuario(row)
        return None
    
    def obtener_todos(self) -> List[Usuario]:
        """Obtiene todos los usuarios"""
        with sqlite3.connect(self.db_path) as conn:
            conn.row_factory = sqlite3.Row
            cursor = conn.execute('SELECT * FROM usuarios')
            rows = cursor.fetchall()
        
        return [self._row_to_usuario(row) for row in rows]
    
    def actualizar(self, usuario: Usuario) -> Usuario:
        """Actualiza un usuario"""
        usuario.validar()
        
        with sqlite3.connect(self.db_path) as conn:
            cursor = conn.execute(
                'UPDATE usuarios SET nombre = ?, email = ?, activo = ? WHERE id = ?',
                (usuario.nombre, usuario.email, 1 if usuario.activo else 0, usuario.id)
            )
            
            if cursor.rowcount == 0:
                raise ValueError(f'Usuario {usuario.id} no existe')
            
            conn.commit()
        
        return usuario
    
    def eliminar(self, id_usuario: int) -> bool:
        """Elimina un usuario"""
        with sqlite3.connect(self.db_path) as conn:
            cursor = conn.execute(
                'DELETE FROM usuarios WHERE id = ?',
                (id_usuario,)
            )
            conn.commit()
            return cursor.rowcount > 0
    
    def obtener_por_email(self, email: str) -> Optional[Usuario]:
        """Obtiene usuario por email"""
        with sqlite3.connect(self.db_path) as conn:
            conn.row_factory = sqlite3.Row
            cursor = conn.execute(
                'SELECT * FROM usuarios WHERE email = ?',
                (email,)
            )
            row = cursor.fetchone()
        
        if row:
            return self._row_to_usuario(row)
        return None
    
    def existe_email(self, email: str) -> bool:
        """Verifica si existe un email"""
        return self.obtener_por_email(email) is not None
    
    @staticmethod
    def _row_to_usuario(row: sqlite3.Row) -> Usuario:
        """Convierte una fila de SQLite a Usuario"""
        return Usuario(
            id=row['id'],
            nombre=row['nombre'],
            email=row['email'],
            activo=bool(row['activo'])
        )
```

---

## Service usando Repository

**services/usuario_service.py**

```python
"""
Service que usa Repository (abstracción)
No le importa cómo se persisten los datos
"""

from typing import List
from models.usuario import Usuario
from repositories.base_repository import BaseRepository

class UsuarioService:
    """Service desacoplado de la persistencia"""
    
    def __init__(self, repository: BaseRepository[Usuario]):
        # ✅ Recibe una abstracción, no una implementación específica
        self.repository = repository
    
    def crear_usuario(self, datos: dict) -> Usuario:
        """Crea un nuevo usuario"""
        # Verificar email duplicado
        if hasattr(self.repository, 'existe_email'):
            if self.repository.existe_email(datos.get('email', '')):
                raise ValueError('El email ya está registrado')
        
        # Crear modelo
        usuario = Usuario.from_dict(datos)
        
        # Persistir (sin saber cómo)
        return self.repository.crear(usuario)
    
    def obtener_usuario(self, id_usuario: int) -> Usuario:
        """Obtiene un usuario por ID"""
        usuario = self.repository.obtener_por_id(id_usuario)
        if not usuario:
            raise ValueError(f'Usuario {id_usuario} no encontrado')
        return usuario
    
    def listar_usuarios(self, solo_activos: bool = False) -> List[Usuario]:
        """Lista usuarios"""
        if solo_activos and hasattr(self.repository, 'obtener_activos'):
            return self.repository.obtener_activos()
        
        usuarios = self.repository.obtener_todos()
        if solo_activos:
            usuarios = [u for u in usuarios if u.activo]
        return usuarios
    
    def actualizar_usuario(self, id_usuario: int, datos: dict) -> Usuario:
        """Actualiza un usuario"""
        usuario = self.obtener_usuario(id_usuario)
        
        if 'nombre' in datos:
            usuario.nombre = datos['nombre']
        if 'email' in datos:
            # Verificar si email está en uso
            if hasattr(self.repository, 'existe_email'):
                existente = self.repository.obtener_por_email(datos['email'])
                if existente and existente.id != id_usuario:
                    raise ValueError('El email ya está en uso')
            usuario.email = datos['email']
        if 'activo' in datos:
            usuario.activo = datos['activo']
        
        return self.repository.actualizar(usuario)
    
    def eliminar_usuario(self, id_usuario: int) -> bool:
        """Elimina un usuario"""
        self.obtener_usuario(id_usuario)  # Verificar que existe
        return self.repository.eliminar(id_usuario)
    
    def buscar_por_email(self, email: str) -> Usuario:
        """Busca usuario por email"""
        if not hasattr(self.repository, 'obtener_por_email'):
            raise NotImplementedError('Repository no soporta búsqueda por email')
        
        usuario = self.repository.obtener_por_email(email)
        if not usuario:
            raise ValueError(f'Usuario con email {email} no encontrado')
        return usuario
```

---

## Factory para elegir Repository

**config/repository_factory.py**

```python
"""
Factory para crear el repository apropiado
"""

import os
from repositories.base_repository import BaseRepository
from repositories.usuario_repository_memory import UsuarioRepositoryMemory
from repositories.usuario_repository_json import UsuarioRepositoryJSON
from repositories.usuario_repository_sqlite import UsuarioRepositorySQLite
from models.usuario import Usuario

class RepositoryFactory:
    """Factory para crear repositories"""
    
    @staticmethod
    def crear_usuario_repository(tipo: str = None) -> BaseRepository[Usuario]:
        """
        Crea el repository apropiado según configuración
        
        Args:
            tipo: 'memory', 'json', 'sqlite'. Si None, usa variable de entorno
        
        Returns:
            Implementación de BaseRepository[Usuario]
        """
        if tipo is None:
            tipo = os.getenv('REPOSITORY_TYPE', 'memory')
        
        if tipo == 'memory':
            return UsuarioRepositoryMemory()
        elif tipo == 'json':
            archivo = os.getenv('JSON_FILE', 'usuarios.json')
            return UsuarioRepositoryJSON(archivo)
        elif tipo == 'sqlite':
            db_path = os.getenv('SQLITE_DB', 'usuarios.db')
            return UsuarioRepositorySQLite(db_path)
        else:
            raise ValueError(f'Tipo de repository desconocido: {tipo}')
```

---

## Aplicación completa

**app_repository.py**

```python
"""
Aplicación con Repository Pattern
Fácil cambiar entre implementaciones
"""

from http.server import BaseHTTPRequestHandler, HTTPServer
from urllib.parse import urlparse
import os

from config.repository_factory import RepositoryFactory
from services.usuario_service import UsuarioService
from controllers.usuario_controller import UsuarioController

# ✅ Elegir repository desde variable de entorno
repository_type = os.getenv('REPOSITORY_TYPE', 'memory')
usuario_repository = RepositoryFactory.crear_usuario_repository(repository_type)

# Crear service con repository
usuario_service = UsuarioService(usuario_repository)

# Crear controller con service
usuario_controller = UsuarioController(usuario_service)

class RepositoryHandler(BaseHTTPRequestHandler):
    """Handler principal"""
    
    def do_GET(self):
        self._manejar()
    
    def do_POST(self):
        self._manejar()
    
    def do_PUT(self):
        self._manejar()
    
    def do_DELETE(self):
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
        html = f"""
        <!DOCTYPE html>
        <html>
        <head>
            <title>Repository Pattern</title>
        </head>
        <body>
            <h1>🗂️ Repository Pattern</h1>
            <p><strong>Repository activo:</strong> {repository_type}</p>
            
            <h2>Cambiar repository:</h2>
            <pre>
# Memoria
export REPOSITORY_TYPE=memory
python app_repository.py

# JSON
export REPOSITORY_TYPE=json
export JSON_FILE=usuarios.json
python app_repository.py

# SQLite
export REPOSITORY_TYPE=sqlite
export SQLITE_DB=usuarios.db
python app_repository.py
            </pre>
            
            <h2>Endpoints:</h2>
            <ul>
                <li>GET /api/usuarios - Lista usuarios</li>
                <li>POST /api/usuarios - Crear usuario</li>
                <li>GET /api/usuarios/:id - Ver usuario</li>
                <li>PUT /api/usuarios/:id - Actualizar usuario</li>
                <li>DELETE /api/usuarios/:id - Eliminar usuario</li>
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
    puerto = 8000
    servidor = HTTPServer(('localhost', puerto), RepositoryHandler)
    
    print('='*60)
    print('🗂️ Repository Pattern')
    print('='*60)
    print(f'\n🚀 Servidor: http://localhost:{puerto}')
    print(f'📦 Repository: {repository_type}')
    print('\n💡 Tip: Cambia REPOSITORY_TYPE para usar otra implementación')
    print('⛔ Ctrl+C para detener\n')
    
    try:
        servidor.serve_forever()
    except KeyboardInterrupt:
        print('\n\n⛔ Servidor detenido')
        servidor.shutdown()
```

---

## Testing con Mock Repository

**tests/test_usuario_service.py**

```python
"""
Test del service con mock repository
"""

import pytest
from models.usuario import Usuario
from repositories.base_repository import BaseRepository
from services.usuario_service import UsuarioService

class MockUsuarioRepository(BaseRepository[Usuario]):
    """Repository mock para tests"""
    
    def __init__(self):
        self._usuarios = {}
        self._siguiente_id = 1
    
    def crear(self, usuario: Usuario) -> Usuario:
        usuario.id = self._siguiente_id
        self._siguiente_id += 1
        self._usuarios[usuario.id] = usuario
        return usuario
    
    def obtener_por_id(self, id_usuario: int):
        return self._usuarios.get(id_usuario)
    
    def obtener_todos(self):
        return list(self._usuarios.values())
    
    def actualizar(self, usuario: Usuario):
        if usuario.id not in self._usuarios:
            raise ValueError('No existe')
        self._usuarios[usuario.id] = usuario
        return usuario
    
    def eliminar(self, id_usuario: int):
        if id_usuario in self._usuarios:
            del self._usuarios[id_usuario]
            return True
        return False
    
    def obtener_por_email(self, email: str):
        for u in self._usuarios.values():
            if u.email == email:
                return u
        return None
    
    def existe_email(self, email: str):
        return self.obtener_por_email(email) is not None

@pytest.fixture
def service():
    """Service con mock repository"""
    repository = MockUsuarioRepository()
    return UsuarioService(repository)

def test_crear_usuario(service):
    """Crea usuario correctamente"""
    datos = {'nombre': 'Ana', 'email': 'ana@test.com'}
    
    usuario = service.crear_usuario(datos)
    
    assert usuario.id == 1
    assert usuario.nombre == 'Ana'
    assert usuario.email == 'ana@test.com'

def test_email_duplicado(service):
    """Rechaza email duplicado"""
    datos = {'nombre': 'Ana', 'email': 'ana@test.com'}
    
    service.crear_usuario(datos)
    
    with pytest.raises(ValueError, match='ya está registrado'):
        service.crear_usuario(datos)

def test_listar_usuarios(service):
    """Lista todos los usuarios"""
    service.crear_usuario({'nombre': 'Ana', 'email': 'ana@test.com'})
    service.crear_usuario({'nombre': 'Bob', 'email': 'bob@test.com'})
    
    usuarios = service.listar_usuarios()
    
    assert len(usuarios) == 2
```

---

## Ventajas del Repository Pattern

✅ **Abstracción de persistencia**
- Service no conoce cómo se guardan los datos
- Fácil cambiar de DB sin tocar lógica

✅ **Testeable**
- Fácil crear mock repositories para tests
- Tests rápidos sin DB real

✅ **Centralizado**
- Toda la lógica de persistencia en un lugar
- Cambios no se propagan

✅ **Reutilizable**
- Mismo repository desde múltiples services
- Queries complejas encapsuladas

✅ **Flexible**
- Múltiples implementaciones de la misma interfaz
- Cambio en runtime si es necesario

---

## Resumen

Has aprendido:

✅ Abstraer persistencia con repositories  
✅ Implementaciones intercambiables (memoria, JSON, SQLite)  
✅ Factory pattern para crear repositories  
✅ Service desacoplado de la DB  
✅ Testing con mock repositories

**Siguiente:** Validadores y DTOs

---

## Recursos

- **[Repository Pattern](https://martinfowler.com/eaaCatalog/repository.html)** - Martin Fowler
- **[Data Mapper Pattern](https://martinfowler.com/eaaCatalog/dataMapper.html)** - Patrón relacionado

Persistencia flexible y testeable. 🗂️

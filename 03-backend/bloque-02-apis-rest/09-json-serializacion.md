# JSON y Serialización

> **Manejo profesional de JSON, fechas, y serialización personalizada**

---

## JSON básico en Python

```python
import json

# Diccionario → JSON string
datos = {'nombre': 'Juan', 'edad': 30}
json_string = json.dumps(datos)
print(json_string)  # '{"nombre": "Juan", "edad": 30}'

# JSON string → Diccionario
json_string = '{"nombre": "Juan", "edad": 30}'
datos = json.loads(json_string)
print(datos['nombre'])  # 'Juan'
```

---

## Serialización en APIs

### Problema: Tipos no serializables

```python
from datetime import datetime
import json

usuario = {
    'id': 1,
    'nombre': 'Juan',
    'creado_en': datetime.now()  # ❌ datetime no es serializable
}

json.dumps(usuario)  # TypeError: Object of type datetime is not JSON serializable
```

### Solución 1: Convertir manualmente

```python
usuario_serializable = {
    'id': 1,
    'nombre': 'Juan',
    'creado_en': datetime.now().isoformat()  # ✅ Convertir a string
}

json.dumps(usuario_serializable)
# '{"id": 1, "nombre": "Juan", "creado_en": "2024-01-20T10:30:00.123456"}'
```

### Solución 2: Custom JSONEncoder

```python
import json
from datetime import datetime, date
from decimal import Decimal

class CustomJSONEncoder(json.JSONEncoder):
    """Encoder que maneja datetime, date, Decimal"""
    
    def default(self, obj):
        # datetime → ISO 8601 string
        if isinstance(obj, datetime):
            return obj.isoformat()
        
        # date → YYYY-MM-DD
        if isinstance(obj, date):
            return obj.isoformat()
        
        # Decimal → float
        if isinstance(obj, Decimal):
            return float(obj)
        
        # Set → list
        if isinstance(obj, set):
            return list(obj)
        
        # Si no es ninguno de los anteriores, usar comportamiento por defecto
        return super().default(obj)

# Uso:
usuario = {
    'id': 1,
    'nombre': 'Juan',
    'creado_en': datetime.now(),
    'fecha_nacimiento': date(1990, 5, 15),
    'saldo': Decimal('150.50'),
    'tags': {'python', 'rest', 'api'}
}

json_string = json.dumps(usuario, cls=CustomJSONEncoder, indent=2)
print(json_string)
```

**Output:**
```json
{
  "id": 1,
  "nombre": "Juan",
  "creado_en": "2024-01-20T10:30:00.123456",
  "fecha_nacimiento": "1990-05-15",
  "saldo": 150.5,
  "tags": ["python", "rest", "api"]
}
```

---

## Serialización con Pydantic

**Pydantic** = Validación + Serialización automática

```python
from pydantic import BaseModel, EmailStr, Field
from datetime import datetime
from typing import Optional

class Usuario(BaseModel):
    id: int
    nombre: str = Field(..., min_length=1, max_length=100)
    email: EmailStr
    edad: int = Field(..., ge=0, le=120)
    activo: bool = True
    creado_en: datetime = Field(default_factory=datetime.now)
    
    class Config:
        # Serializar datetime como ISO string
        json_encoders = {
            datetime: lambda v: v.isoformat()
        }

# Crear instancia
usuario = Usuario(
    id=1,
    nombre='Juan',
    email='juan@ejemplo.com',
    edad=30
)

# Serializar a dict
print(usuario.dict())

# Serializar a JSON
print(usuario.json(indent=2))

# Desde JSON
json_data = '{"id": 2, "nombre": "María", "email": "maria@ejemplo.com", "edad": 25}'
usuario2 = Usuario.parse_raw(json_data)
print(usuario2.nombre)  # 'María'
```

---

## API con serialización automática

```python
from http.server import BaseHTTPRequestHandler, HTTPServer
from urllib.parse import urlparse
import json
import re
from datetime import datetime
from pydantic import BaseModel, EmailStr, Field, ValidationError
from typing import Optional

# ===== Modelos Pydantic =====

class UsuarioBase(BaseModel):
    nombre: str = Field(..., min_length=1, max_length=100)
    email: EmailStr
    edad: Optional[int] = Field(None, ge=0, le=120)

class UsuarioCreate(UsuarioBase):
    """Para crear usuario (sin id)"""
    pass

class UsuarioUpdate(BaseModel):
    """Para actualizar (todos opcionales)"""
    nombre: Optional[str] = Field(None, min_length=1, max_length=100)
    email: Optional[EmailStr] = None
    edad: Optional[int] = Field(None, ge=0, le=120)

class Usuario(UsuarioBase):
    """Usuario completo con metadata"""
    id: int
    activo: bool = True
    creado_en: datetime = Field(default_factory=datetime.now)
    actualizado_en: Optional[datetime] = None
    
    class Config:
        json_encoders = {
            datetime: lambda v: v.isoformat()
        }

# Base de datos (simulada con Pydantic)
usuarios_db: list[Usuario] = [
    Usuario(id=1, nombre='Juan', email='juan@ejemplo.com', edad=30),
    Usuario(id=2, nombre='María', email='maria@ejemplo.com', edad=25)
]

siguiente_id = 3

# ===== API Handler =====

class APIHandler(BaseHTTPRequestHandler):
    
    def do_GET(self):
        ruta = urlparse(self.path).path
        
        match = re.match(r'^/api/usuarios/(\d+)$', ruta)
        if match:
            self.obtener_usuario(int(match.group(1)))
        elif ruta == '/api/usuarios':
            self.listar_usuarios()
        else:
            self.enviar_error(404, 'Ruta no encontrada')
    
    def do_POST(self):
        if urlparse(self.path).path == '/api/usuarios':
            self.crear_usuario()
        else:
            self.enviar_error(404, 'Ruta no encontrada')
    
    def do_PATCH(self):
        match = re.match(r'^/api/usuarios/(\d+)$', urlparse(self.path).path)
        if match:
            self.actualizar_usuario(int(match.group(1)))
        else:
            self.enviar_error(404, 'Ruta no encontrada')
    
    def do_DELETE(self):
        match = re.match(r'^/api/usuarios/(\d+)$', urlparse(self.path).path)
        if match:
            self.eliminar_usuario(int(match.group(1)))
        else:
            self.enviar_error(404, 'Ruta no encontrada')
    
    # ===== Handlers =====
    
    def listar_usuarios(self):
        """GET /api/usuarios"""
        # Serializar lista de Pydantic models
        usuarios_json = [usuario.dict() for usuario in usuarios_db]
        self.enviar_json(usuarios_json)
    
    def obtener_usuario(self, user_id: int):
        """GET /api/usuarios/:id"""
        usuario = next((u for u in usuarios_db if u.id == user_id), None)
        
        if usuario:
            # Pydantic serializa automáticamente datetime
            self.enviar_json(usuario.dict())
        else:
            self.enviar_error(404, f'Usuario {user_id} no encontrado')
    
    def crear_usuario(self):
        """POST /api/usuarios"""
        global siguiente_id
        
        datos = self.leer_json_body()
        if not datos:
            self.enviar_error(400, 'Body JSON inválido')
            return
        
        try:
            # Validar con Pydantic
            usuario_create = UsuarioCreate(**datos)
            
            # Crear Usuario completo
            nuevo_usuario = Usuario(
                id=siguiente_id,
                **usuario_create.dict()
            )
            
            usuarios_db.append(nuevo_usuario)
            siguiente_id += 1
            
            # Responder con usuario serializado
            self.enviar_json(nuevo_usuario.dict(), 201)
            
        except ValidationError as e:
            # Pydantic genera errores detallados
            self.enviar_error(422, {
                'error': 'Validación fallida',
                'detalles': e.errors()
            })
    
    def actualizar_usuario(self, user_id: int):
        """PATCH /api/usuarios/:id"""
        usuario = next((u for u in usuarios_db if u.id == user_id), None)
        
        if not usuario:
            self.enviar_error(404, f'Usuario {user_id} no encontrado')
            return
        
        datos = self.leer_json_body()
        if not datos:
            self.enviar_error(400, 'Body JSON inválido')
            return
        
        try:
            # Validar actualización
            usuario_update = UsuarioUpdate(**datos)
            
            # Actualizar solo campos enviados
            datos_update = usuario_update.dict(exclude_unset=True)
            
            for campo, valor in datos_update.items():
                setattr(usuario, campo, valor)
            
            # Actualizar timestamp
            usuario.actualizado_en = datetime.now()
            
            self.enviar_json(usuario.dict())
            
        except ValidationError as e:
            self.enviar_error(422, {
                'error': 'Validación fallida',
                'detalles': e.errors()
            })
    
    def eliminar_usuario(self, user_id: int):
        """DELETE /api/usuarios/:id"""
        global usuarios_db
        
        usuario = next((u for u in usuarios_db if u.id == user_id), None)
        
        if not usuario:
            self.enviar_error(404, f'Usuario {user_id} no encontrado')
            return
        
        usuarios_db = [u for u in usuarios_db if u.id != user_id]
        
        self.send_response(204)
        self.end_headers()
    
    # ===== Helpers =====
    
    def enviar_json(self, datos, status=200):
        """Serializa con CustomJSONEncoder automático"""
        self.send_response(status)
        self.send_header('Content-type', 'application/json; charset=utf-8')
        self.send_header('Access-Control-Allow-Origin', '*')
        self.end_headers()
        
        # Si es dict o list, usar CustomJSONEncoder
        json_string = json.dumps(
            datos,
            cls=CustomJSONEncoder,
            ensure_ascii=False,
            indent=2
        )
        
        self.wfile.write(json_string.encode('utf-8'))
    
    def enviar_error(self, codigo, mensaje):
        self.send_response(codigo)
        self.send_header('Content-type', 'application/json; charset=utf-8')
        self.end_headers()
        
        if isinstance(mensaje, dict):
            error = {'codigo': codigo, **mensaje}
        else:
            error = {'error': mensaje, 'codigo': codigo}
        
        self.wfile.write(json.dumps(error, ensure_ascii=False).encode('utf-8'))
    
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
    print('📦 Instalando dependencias necesarias...')
    print('   pip install pydantic[email]')
    print()
    
    servidor = HTTPServer(('localhost', 8000), APIHandler)
    print('🚀 API con Pydantic en http://localhost:8000')
    print('\n📋 Endpoints:')
    print('  GET    /api/usuarios')
    print('  GET    /api/usuarios/:id')
    print('  POST   /api/usuarios')
    print('  PATCH  /api/usuarios/:id')
    print('  DELETE /api/usuarios/:id\n')
    
    try:
        servidor.serve_forever()
    except KeyboardInterrupt:
        print('\n⛔ Servidor detenido')
        servidor.shutdown()
```

---

## Formatos de serialización

### ISO 8601 (recomendado)

```python
from datetime import datetime

ahora = datetime.now()
iso_string = ahora.isoformat()
print(iso_string)  # '2024-01-20T10:30:00.123456'

# Con timezone
from datetime import timezone
ahora_utc = datetime.now(timezone.utc)
print(ahora_utc.isoformat())  # '2024-01-20T10:30:00.123456+00:00'
```

### Unix timestamp

```python
import time

# Timestamp
timestamp = time.time()
print(timestamp)  # 1705745400.123456

# Timestamp → datetime
fecha = datetime.fromtimestamp(timestamp)

# datetime → Timestamp
timestamp = fecha.timestamp()
```

---

## Opciones de json.dumps()

```python
import json

datos = {
    'nombre': 'Juan',
    'país': 'México',
    'hobbies': ['programar', 'leer']
}

# Indentación bonita
json.dumps(datos, indent=2)

# Mantener caracteres unicode (español, emojis)
json.dumps(datos, ensure_ascii=False)

# Ordenar keys alfabéticamente
json.dumps(datos, sort_keys=True)

# Completo
json.dumps(
    datos,
    indent=2,
    ensure_ascii=False,
    sort_keys=True
)
```

**Output:**
```json
{
  "hobbies": [
    "programar",
    "leer"
  ],
  "nombre": "Juan",
  "país": "México"
}
```

---

## Serialización de relaciones

```python
class Post(BaseModel):
    id: int
    titulo: str
    usuario_id: int

class UsuarioConPosts(Usuario):
    """Usuario con sus posts relacionados"""
    posts: list[Post] = []

# En el handler:
def obtener_usuario_con_posts(user_id: int):
    usuario = next((u for u in usuarios_db if u.id == user_id), None)
    
    if not usuario:
        self.enviar_error(404, 'Usuario no encontrado')
        return
    
    # Buscar posts del usuario
    posts_usuario = [p for p in posts_db if p.usuario_id == user_id]
    
    # Crear UsuarioConPosts
    usuario_completo = UsuarioConPosts(
        **usuario.dict(),
        posts=posts_usuario
    )
    
    self.enviar_json(usuario_completo.dict())
```

**Response:**
```json
{
  "id": 1,
  "nombre": "Juan",
  "email": "juan@ejemplo.com",
  "edad": 30,
  "activo": true,
  "creado_en": "2024-01-20T10:30:00",
  "posts": [
    {
      "id": 1,
      "titulo": "Mi primer post",
      "usuario_id": 1
    },
    {
      "id": 2,
      "titulo": "Segundo post",
      "usuario_id": 1
    }
  ]
}
```

---

## Excluir campos sensibles

```python
class Usuario(BaseModel):
    id: int
    nombre: str
    email: str
    password_hash: str  # No debe exponerse
    
    def dict_publico(self):
        """Diccionario sin campos sensibles"""
        return self.dict(exclude={'password_hash'})

# En el handler:
def obtener_usuario(user_id):
    usuario = buscar_usuario(user_id)
    # Usar dict_publico() en vez de dict()
    self.enviar_json(usuario.dict_publico())
```

---

## Pruebas

```powershell
# Listar (con datetime serializado)
curl http://localhost:8000/api/usuarios

# Crear con validación
curl -X POST http://localhost:8000/api/usuarios `
  -H "Content-Type: application/json" `
  -d '{\"nombre\":\"Pedro\",\"email\":\"pedro@ejemplo.com\",\"edad\":28}'

# Error de validación: email inválido
curl -X POST http://localhost:8000/api/usuarios `
  -H "Content-Type: application/json" `
  -d '{\"nombre\":\"Test\",\"email\":\"invalido\",\"edad\":25}'

# Error de validación: edad negativa
curl -X POST http://localhost:8000/api/usuarios `
  -H "Content-Type: application/json" `
  -d '{\"nombre\":\"Test\",\"email\":\"test@ejemplo.com\",\"edad\":-5}'

# Actualizar parcial
curl -X PATCH http://localhost:8000/api/usuarios/1 `
  -H "Content-Type: application/json" `
  -d '{\"edad\":31}'
```

---

## Resumen

Has aprendido:

✅ Serialización JSON básica  
✅ CustomJSONEncoder para datetime y otros tipos  
✅ Pydantic para validación + serialización automática  
✅ Formatos ISO 8601 y timestamp  
✅ Opciones de json.dumps()  
✅ Serialización de relaciones  
✅ Excluir campos sensibles

**Siguiente:** [Status codes y manejo de errores](./10-status-codes-errores.md)

---

## Recursos

- **[json module](https://docs.python.org/3/library/json.html)** - Python JSON
- **[Pydantic](https://docs.pydantic.dev/)** - Data validation
- **[ISO 8601](https://en.wikipedia.org/wiki/ISO_8601)** - Date/time format

JSON es el lenguaje universal de las APIs. 📦

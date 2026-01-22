# Bcrypt y hashing de contraseñas

> **Protege las contraseñas de tus usuarios con bcrypt**

---

## ¿Por qué bcrypt?

**❌ Nunca guardes contraseñas en texto plano:**

```python
# ❌ PELIGRO - Texto plano
usuarios = {
    'ana@ejemplo.com': 'password123',
    'juan@ejemplo.com': 'qwerty456'
}
```

**Problemas:**
- Si hay breach → todas las contraseñas expuestas
- Administradores pueden ver contraseñas
- Violación de GDPR/privacidad
- Pérdida de confianza de usuarios

---

## ¿Qué es hashing?

**Hashing:** Función matemática **unidireccional** (no se puede revertir)

```
password → hash()  → a3f8d9c7...
a3f8d9c7... → ???  → ¡IMPOSIBLE!
```

**Características:**
- ✅ Siempre el mismo input → mismo output
- ✅ Imposible revertir
- ✅ Pequeño cambio → hash completamente diferente
- ✅ Longitud fija del output

---

## Algoritmos de hashing

| Algoritmo | Uso | Seguridad |
|-----------|-----|-----------|
| MD5 | ❌ OBSOLETO | Roto |
| SHA1 | ❌ OBSOLETO | Roto |
| SHA256 | ⚠️ Rápido (malo para passwords) | Media |
| **bcrypt** | ✅ **RECOMENDADO** | Alta |
| scrypt | ✅ Alternativa | Alta |
| Argon2 | ✅ Más nuevo | Muy alta |

**¿Por qué bcrypt es mejor?**
1. **Lento por diseño:** Dificulta ataques de fuerza bruta
2. **Salt automático:** Cada hash es único
3. **Configurablechecost factor:** Ajusta dificultad

---

## Salt: ¿qué es?

**Salt:** Datos aleatorios agregados al password antes de hashear

**Sin salt (❌):**
```
"password123" → SHA256 → ef92b778...
"password123" → SHA256 → ef92b778...  (¡mismo hash!)
```

Problema: Rainbow tables (tablas pre-computadas de hashes)

**Con salt (✅):**
```
"password123" + salt1 → bcrypt → $2b$12$abc...
"password123" + salt2 → bcrypt → $2b$12$xyz...  (¡hashes diferentes!)
```

**Ventaja:** Dos usuarios con la misma contraseña tienen hashes diferentes.

---

## Instalación de bcrypt

```powershell
pip install bcrypt
```

---

## Uso básico de bcrypt

```python
import bcrypt

# ===== HASHEAR CONTRASEÑA =====

password = 'MiPasswordSeguro123!'

# Convertir a bytes
password_bytes = password.encode('utf-8')

# Generar salt y hash
salt = bcrypt.gensalt()
password_hash = bcrypt.hashpw(password_bytes, salt)

print(password_hash)
# b'$2b$12$xyz...abc'

# ===== VERIFICAR CONTRASEÑA =====

password_ingresado = 'MiPasswordSeguro123!'

# Verificar
es_correcto = bcrypt.checkpw(
    password_ingresado.encode('utf-8'),
    password_hash
)

print(es_correcto)  # True o False
```

---

## Anatomía de un hash bcrypt

```
$2b$12$R9h/cIPz0gi.URNNX3kh2OPST9/PgBkqquzi.Ss7KIUgO2t0jWMUW
│││ ││ │                                                │
│││ ││ └─ Salt (22 caracteres)                          │
│││ ││                                                   │
│││ └┴─ Cost factor (2^12 = 4096 iteraciones)           │
│││                                                      │
│└┴─ Versión del algoritmo                              │
│                                                        │
└── Prefijo identificador                               │
                                                         │
                                                         └─ Hash (31 caracteres)
```

**Partes:**
- `$2b$`: Versión de bcrypt
- `12`: Cost factor (trabajo computacional)
- `R9h/cIPz0gi.URNNX3kh2O`: Salt aleatorio
- `PST9/PgBkqquzi.Ss7KIUgO2t0jWMUW`: Hash del password + salt

---

## Cost factor (rounds)

El **cost factor** determina cuántas iteraciones se hacen:

```python
# Cost factor = 10 → 2^10 = 1,024 iteraciones
salt = bcrypt.gensalt(rounds=10)

# Cost factor = 12 → 2^12 = 4,096 iteraciones (recomendado)
salt = bcrypt.gensalt(rounds=12)

# Cost factor = 14 → 2^14 = 16,384 iteraciones
salt = bcrypt.gensalt(rounds=14)
```

**Mayor cost = más seguro pero más lento**

**Recomendaciones:**
- **10-12:** Bueno para la mayoría de aplicaciones
- **14+:** Sistemas muy sensibles (bancarios, médicos)

**Benchmark:**

```python
import bcrypt
import time

password = 'test123'.encode('utf-8')

for rounds in [10, 12, 14]:
    inicio = time.time()
    salt = bcrypt.gensalt(rounds=rounds)
    bcrypt.hashpw(password, salt)
    duracion = time.time() - inicio
    
    print(f'Cost {rounds}: {duracion:.3f} segundos')

# Output típico:
# Cost 10: 0.050 segundos
# Cost 12: 0.200 segundos
# Cost 14: 0.800 segundos
```

---

## Clase PasswordService

**password_service.py**

```python
"""
Servicio profesional para manejo de contraseñas
"""

import bcrypt
import re
from typing import Tuple

class PasswordService:
    """Servicio para hashear y verificar contraseñas"""
    
    def __init__(self, rounds: int = 12):
        """
        Args:
            rounds: Cost factor (por defecto 12 = 4096 iteraciones)
        """
        self.rounds = rounds
    
    def hashear(self, password: str) -> bytes:
        """
        Hashea una contraseña con bcrypt
        
        Args:
            password: Contraseña en texto plano
            
        Returns:
            Hash de la contraseña (bytes)
        """
        if not password:
            raise ValueError('Password no puede estar vacío')
        
        password_bytes = password.encode('utf-8')
        salt = bcrypt.gensalt(rounds=self.rounds)
        password_hash = bcrypt.hashpw(password_bytes, salt)
        
        return password_hash
    
    def verificar(self, password: str, password_hash: bytes) -> bool:
        """
        Verifica si una contraseña coincide con su hash
        
        Args:
            password: Contraseña en texto plano
            password_hash: Hash almacenado
            
        Returns:
            True si coincide, False si no
        """
        if not password or not password_hash:
            return False
        
        password_bytes = password.encode('utf-8')
        
        try:
            return bcrypt.checkpw(password_bytes, password_hash)
        except Exception:
            return False
    
    def validar_fortaleza(self, password: str) -> Tuple[bool, str]:
        """
        Valida que la contraseña sea segura
        
        Returns:
            (es_valido, mensaje_error)
        """
        if len(password) < 8:
            return False, 'Mínimo 8 caracteres'
        
        if len(password) > 128:
            return False, 'Máximo 128 caracteres'
        
        if not re.search(r'[a-z]', password):
            return False, 'Debe contener al menos una minúscula'
        
        if not re.search(r'[A-Z]', password):
            return False, 'Debe contener al menos una mayúscula'
        
        if not re.search(r'[0-9]', password):
            return False, 'Debe contener al menos un número'
        
        if not re.search(r'[!@#$%^&*(),.?":{}|<>]', password):
            return False, 'Debe contener al menos un carácter especial'
        
        # Palabras comunes prohibidas
        palabras_prohibidas = [
            'password', 'contraseña', '12345678', 'qwerty', 
            'abc123', 'admin', 'usuario', 'letmein'
        ]
        
        password_lower = password.lower()
        for palabra in palabras_prohibidas:
            if palabra in password_lower:
                return False, f'No puede contener "{palabra}"'
        
        return True, 'Contraseña válida'
    
    def generar_password_seguro(self, longitud: int = 16) -> str:
        """Genera una contraseña aleatoria segura"""
        import secrets
        import string
        
        caracteres = (
            string.ascii_lowercase +
            string.ascii_uppercase +
            string.digits +
            '!@#$%^&*'
        )
        
        # Generar password aleatorio
        password = ''.join(secrets.choice(caracteres) for _ in range(longitud))
        
        # Asegurar que cumple requisitos
        while not self.validar_fortaleza(password)[0]:
            password = ''.join(secrets.choice(caracteres) for _ in range(longitud))
        
        return password
```

---

## Integración con API

**app_passwords.py**

```python
"""
API con gestión segura de contraseñas
"""

from http.server import BaseHTTPRequestHandler, HTTPServer
from urllib.parse import urlparse
import json

from password_service import PasswordService

# Servicio de contraseñas
password_service = PasswordService(rounds=12)

# Base de datos simulada
usuarios_db = {}

class PasswordAPIHandler(BaseHTTPRequestHandler):
    """Handler para API con passwords seguros"""
    
    def do_POST(self):
        """Maneja peticiones POST"""
        try:
            ruta = urlparse(self.path).path
            
            if ruta == '/api/registro':
                self.registro()
            elif ruta == '/api/login':
                self.login()
            elif ruta == '/api/cambiar-password':
                self.cambiar_password()
            elif ruta == '/api/validar-password':
                self.validar_password()
            elif ruta == '/api/generar-password':
                self.generar_password()
            else:
                self.enviar_error(404, 'Ruta no encontrada')
                
        except Exception as e:
            print(f'Error: {e}')
            self.enviar_error(500, str(e))
    
    def do_GET(self):
        """Maneja peticiones GET"""
        try:
            ruta = urlparse(self.path).path
            
            if ruta == '/':
                self.home()
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
            <title>API Passwords</title>
            <style>
                body { font-family: Arial; max-width: 900px; margin: 50px auto; }
                .endpoint { background: #f5f5f5; padding: 15px; margin: 10px 0; }
            </style>
        </head>
        <body>
            <h1>🔐 API de Contraseñas Seguras</h1>
            <p>Bcrypt rounds: 12 (4096 iteraciones)</p>
            
            <h2>Endpoints:</h2>
            
            <div class="endpoint">
                <strong>POST /api/registro</strong><br>
                Body: <code>{"email": "...", "password": "..."}</code>
            </div>
            
            <div class="endpoint">
                <strong>POST /api/login</strong><br>
                Body: <code>{"email": "...", "password": "..."}</code>
            </div>
            
            <div class="endpoint">
                <strong>POST /api/cambiar-password</strong><br>
                Body: <code>{"email": "...", "password_actual": "...", "password_nuevo": "..."}</code>
            </div>
            
            <div class="endpoint">
                <strong>POST /api/validar-password</strong><br>
                Body: <code>{"password": "..."}</code>
            </div>
            
            <div class="endpoint">
                <strong>POST /api/generar-password</strong><br>
                Body: <code>{"longitud": 16}</code> (opcional)
            </div>
        </body>
        </html>
        """
        self.enviar_html(html)
    
    def registro(self):
        """POST /api/registro"""
        datos = self.leer_json_body()
        
        if not datos:
            self.enviar_error(400, 'Datos requeridos')
            return
        
        email = datos.get('email')
        password = datos.get('password')
        
        if not email or not password:
            self.enviar_error(400, 'Email y password requeridos')
            return
        
        # Verificar que no exista
        if email in usuarios_db:
            self.enviar_error(400, 'Email ya registrado')
            return
        
        # Validar fortaleza del password
        es_valido, mensaje = password_service.validar_fortaleza(password)
        if not es_valido:
            self.enviar_error(400, f'Password débil: {mensaje}')
            return
        
        # Hashear password
        password_hash = password_service.hashear(password)
        
        # Guardar usuario
        usuarios_db[email] = {
            'email': email,
            'password_hash': password_hash
        }
        
        self.send_response(201)
        self.send_header('Content-type', 'application/json')
        self.end_headers()
        
        respuesta = {
            'mensaje': 'Usuario registrado exitosamente',
            'email': email,
            'password_hash_preview': password_hash.decode('utf-8')[:30] + '...'
        }
        
        self.wfile.write(json.dumps(respuesta, ensure_ascii=False).encode())
    
    def login(self):
        """POST /api/login"""
        datos = self.leer_json_body()
        
        if not datos:
            self.enviar_error(400, 'Datos requeridos')
            return
        
        email = datos.get('email')
        password = datos.get('password')
        
        if not email or not password:
            self.enviar_error(400, 'Email y password requeridos')
            return
        
        # Buscar usuario
        usuario = usuarios_db.get(email)
        if not usuario:
            self.enviar_error(401, 'Credenciales inválidas')
            return
        
        # Verificar password
        es_valido = password_service.verificar(password, usuario['password_hash'])
        
        if not es_valido:
            self.enviar_error(401, 'Credenciales inválidas')
            return
        
        self.enviar_json({
            'mensaje': 'Login exitoso',
            'email': email
        })
    
    def cambiar_password(self):
        """POST /api/cambiar-password"""
        datos = self.leer_json_body()
        
        if not datos:
            self.enviar_error(400, 'Datos requeridos')
            return
        
        email = datos.get('email')
        password_actual = datos.get('password_actual')
        password_nuevo = datos.get('password_nuevo')
        
        if not email or not password_actual or not password_nuevo:
            self.enviar_error(400, 'Email, password_actual y password_nuevo requeridos')
            return
        
        # Buscar usuario
        usuario = usuarios_db.get(email)
        if not usuario:
            self.enviar_error(404, 'Usuario no encontrado')
            return
        
        # Verificar password actual
        es_valido = password_service.verificar(password_actual, usuario['password_hash'])
        if not es_valido:
            self.enviar_error(401, 'Password actual incorrecto')
            return
        
        # Validar nuevo password
        es_valido, mensaje = password_service.validar_fortaleza(password_nuevo)
        if not es_valido:
            self.enviar_error(400, f'Nuevo password débil: {mensaje}')
            return
        
        # Hashear nuevo password
        nuevo_hash = password_service.hashear(password_nuevo)
        
        # Actualizar
        usuario['password_hash'] = nuevo_hash
        
        self.enviar_json({
            'mensaje': 'Password cambiado exitosamente'
        })
    
    def validar_password(self):
        """POST /api/validar-password"""
        datos = self.leer_json_body()
        
        if not datos:
            self.enviar_error(400, 'Datos requeridos')
            return
        
        password = datos.get('password')
        
        if not password:
            self.enviar_error(400, 'Password requerido')
            return
        
        es_valido, mensaje = password_service.validar_fortaleza(password)
        
        self.enviar_json({
            'valido': es_valido,
            'mensaje': mensaje,
            'requisitos': {
                'longitud_minima': len(password) >= 8,
                'tiene_minuscula': bool(re.search(r'[a-z]', password)),
                'tiene_mayuscula': bool(re.search(r'[A-Z]', password)),
                'tiene_numero': bool(re.search(r'[0-9]', password)),
                'tiene_especial': bool(re.search(r'[!@#$%^&*(),.?":{}|<>]', password))
            }
        })
    
    def generar_password(self):
        """POST /api/generar-password"""
        datos = self.leer_json_body()
        longitud = datos.get('longitud', 16) if datos else 16
        
        if longitud < 8 or longitud > 128:
            self.enviar_error(400, 'Longitud debe estar entre 8 y 128')
            return
        
        password = password_service.generar_password_seguro(longitud)
        
        self.enviar_json({
            'password': password,
            'longitud': len(password),
            'mensaje': 'Password generado exitosamente'
        })
    
    # ===== Helpers =====
    
    def enviar_json(self, datos, status=200):
        """Envía respuesta JSON"""
        self.send_response(status)
        self.send_header('Content-type', 'application/json; charset=utf-8')
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
    servidor = HTTPServer(('localhost', puerto), PasswordAPIHandler)
    
    print('='*60)
    print('🔐 API de Contraseñas Seguras')
    print('='*60)
    print(f'\n🚀 Servidor: http://localhost:{puerto}')
    print('\n✨ Bcrypt configurado con cost factor 12')
    print('   (4096 iteraciones por hash)')
    print('\n⛔ Ctrl+C para detener\n')
    
    try:
        servidor.serve_forever()
    except KeyboardInterrupt:
        print('\n\n⛔ Servidor detenido')
        servidor.shutdown()
```

---

## Pruebas con curl

### 1. Validar fortaleza

```powershell
# Password débil
curl -X POST http://localhost:8000/api/validar-password `
  -H "Content-Type: application/json" `
  -d '{\"password\":\"abc\"}'

# Response: {"valido": false, "mensaje": "Mínimo 8 caracteres"}

# Password fuerte
curl -X POST http://localhost:8000/api/validar-password `
  -H "Content-Type: application/json" `
  -d '{\"password\":\"MiPass123!@#\"}'

# Response: {"valido": true, "mensaje": "Contraseña válida"}
```

### 2. Generar password aleatorio

```powershell
curl -X POST http://localhost:8000/api/generar-password `
  -H "Content-Type: application/json" `
  -d '{\"longitud\":20}'

# Response:
# {
#   "password": "aB3!xY7*mN2@qW5#pL9$",
#   "longitud": 20,
#   "mensaje": "Password generado exitosamente"
# }
```

### 3. Registro con hash

```powershell
curl -X POST http://localhost:8000/api/registro `
  -H "Content-Type: application/json" `
  -d '{\"email\":\"ana@ejemplo.com\",\"password\":\"Ana123!@#Seguro\"}'

# Response:
# {
#   "mensaje": "Usuario registrado exitosamente",
#   "email": "ana@ejemplo.com",
#   "password_hash_preview": "$2b$12$xyz...abc..."
# }
```

### 4. Login con verificación

```powershell
# Login correcto
curl -X POST http://localhost:8000/api/login `
  -H "Content-Type: application/json" `
  -d '{\"email\":\"ana@ejemplo.com\",\"password\":\"Ana123!@#Seguro\"}'

# Response: {"mensaje": "Login exitoso", "email": "ana@ejemplo.com"}

# Login incorrecto
curl -X POST http://localhost:8000/api/login `
  -H "Content-Type: application/json" `
  -d '{\"email\":\"ana@ejemplo.com\",\"password\":\"PasswordIncorrecto\"}'

# Response: {"error": "Credenciales inválidas", "codigo": 401}
```

### 5. Cambiar password

```powershell
curl -X POST http://localhost:8000/api/cambiar-password `
  -H "Content-Type: application/json" `
  -d '{\"email\":\"ana@ejemplo.com\",\"password_actual\":\"Ana123!@#Seguro\",\"password_nuevo\":\"NuevoPass456!@#\"}'
```

---

## Mejores prácticas

### ✅ DO

1. **Siempre usa bcrypt (o Argon2)**
```python
password_hash = bcrypt.hashpw(password.encode(), bcrypt.gensalt())
```

2. **Cost factor adecuado**
```python
salt = bcrypt.gensalt(rounds=12)  # 4096 iteraciones
```

3. **Valida fortaleza antes de hashear**
```python
es_valido, mensaje = validar_fortaleza(password)
if not es_valido:
    return error(mensaje)
```

4. **Limita longitud máxima**
```python
if len(password) > 128:
    return error('Password muy largo')
```

5. **Rate limiting en login**
```python
intentos_fallidos[email] += 1
if intentos_fallidos[email] > 5:
    return error('Demasiados intentos')
```

### ❌ DON'T

1. **NO uses MD5 o SHA1**
```python
# ❌ INSEGURO
import hashlib
password_hash = hashlib.md5(password.encode()).hexdigest()
```

2. **NO uses SHA256 directamente**
```python
# ❌ Muy rápido = fácil de crackear
password_hash = hashlib.sha256(password.encode()).hexdigest()
```

3. **NO almacenes el salt por separado**
```python
# ❌ Innecesario (bcrypt incluye salt en el hash)
salt = bcrypt.gensalt()
hash = bcrypt.hashpw(password, salt)
# No necesitas guardar `salt` aparte
```

4. **NO encriptes passwords (usa hash)**
```python
# ❌ Encriptación es reversible
from cryptography.fernet import Fernet
cipher = Fernet(key)
encrypted = cipher.encrypt(password.encode())  # ¡NO!
```

---

## Migración de hashes antiguos

Si tienes passwords en MD5/SHA1, migra gradualmente:

```python
def login(email, password):
    usuario = db.get_usuario(email)
    
    # Si tiene hash antiguo
    if usuario.hash_type == 'md5':
        # Verificar con MD5
        if hashlib.md5(password.encode()).hexdigest() == usuario.password_hash:
            # Re-hashear con bcrypt
            nuevo_hash = bcrypt.hashpw(password.encode(), bcrypt.gensalt())
            db.actualizar_password(email, nuevo_hash, hash_type='bcrypt')
            return True
    
    # Si tiene bcrypt
    elif usuario.hash_type == 'bcrypt':
        return bcrypt.checkpw(password.encode(), usuario.password_hash)
    
    return False
```

---

## Comparación de algoritmos

**Benchmark:**

```python
import bcrypt
import hashlib
import time

password = 'TestPassword123!'.encode()

# MD5 (INSEGURO)
inicio = time.time()
for _ in range(10000):
    hashlib.md5(password).hexdigest()
print(f'MD5: {time.time() - inicio:.3f}s por 10k hashes')

# SHA256 (INSEGURO para passwords)
inicio = time.time()
for _ in range(10000):
    hashlib.sha256(password).hexdigest()
print(f'SHA256: {time.time() - inicio:.3f}s por 10k hashes')

# Bcrypt (SEGURO)
inicio = time.time()
for _ in range(10):
    bcrypt.hashpw(password, bcrypt.gensalt(rounds=12))
print(f'Bcrypt: {time.time() - inicio:.3f}s por 10 hashes')

# Resultado típico:
# MD5:     0.050s por 10,000 hashes (muy rápido = inseguro)
# SHA256:  0.080s por 10,000 hashes (muy rápido = inseguro)
# Bcrypt:  2.000s por 10 hashes     (lento = seguro)
```

**Conclusión:** Bcrypt es **40,000x más lento** que MD5, lo que dificulta ataques de fuerza bruta.

---

## Resumen

Has aprendido:

✅ Por qué nunca guardar passwords en texto plano  
✅ Diferencia entre hashing y encriptación  
✅ Qué es un salt y por qué es importante  
✅ Uso completo de bcrypt con cost factor  
✅ Validación de fortaleza de contraseñas  
✅ Integración con API REST  
✅ Mejores prácticas y errores comunes

**Siguiente:** [Protección de rutas](./26-proteccion-rutas.md)

---

## Recursos

- **[bcrypt](https://github.com/pyca/bcrypt/)** - Documentación
- **[OWASP Password Storage](https://cheatsheetseries.owasp.org/cheatsheets/Password_Storage_Cheat_Sheet.html)**
- **[Argon2](https://github.com/p-h-c/phc-winner-argon2)** - Alternativa moderna
- **[Have I Been Pwned](https://haveibeenpwned.com/)** - Verificar contraseñas comprometidas

Tus contraseñas ahora están protegidas profesionalmente. 🔐

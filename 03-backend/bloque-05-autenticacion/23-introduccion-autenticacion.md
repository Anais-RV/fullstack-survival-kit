# Introducción a la autenticación en backend

> **Protege tu API con autenticación y autorización**

---

## ¿Qué es la autenticación?

**Autenticación:** Verificar **quién** eres (identidad)  
**Autorización:** Verificar **qué puedes hacer** (permisos)

**Ejemplo:**
- 🔐 **Autenticación:** Login con usuario y contraseña
- 🛡️ **Autorización:** Solo admins pueden borrar usuarios

---

## Flujo típico de autenticación

```
1. Usuario envía credenciales (email + password)
2. Servidor verifica credenciales
3. Si son correctas: genera TOKEN
4. Cliente guarda el token
5. Cliente envía token en cada petición
6. Servidor valida token y procesa petición
```

---

## Tipos de autenticación

### 1. Sesiones (tradicional)

```
Cliente → Login → Servidor crea sesión → Cookie
Cliente → Petición + Cookie → Servidor verifica sesión
```

**Ventajas:** Simple, bien conocido  
**Desventajas:** Requiere almacenamiento en servidor, no funciona bien con microservicios

### 2. Tokens JWT (moderno)

```
Cliente → Login → Servidor crea JWT → Token
Cliente → Petición + Token → Servidor verifica JWT
```

**Ventajas:** Sin estado (stateless), escalable, funciona con microservicios  
**Desventajas:** Tokens no se pueden "revocar" fácilmente

---

## ¿Qué es JWT?

**JWT** = JSON Web Token

Un **token firmado** que contiene información (claims):

```json
{
  "user_id": 123,
  "email": "usuario@ejemplo.com",
  "exp": 1735689600
}
```

### Estructura de un JWT

```
HEADER.PAYLOAD.SIGNATURE
```

**Ejemplo:**
```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VyX2lkIjoxMjN9.xyz...
```

**Partes:**
1. **Header:** Tipo y algoritmo (`{"alg":"HS256","typ":"JWT"}`)
2. **Payload:** Datos/claims (`{"user_id":123,"exp":1735689600}`)
3. **Signature:** Firma para verificar integridad

**Decodificar JWT:**
- Base64 decode del header y payload
- La firma se verifica con la clave secreta

---

## Seguridad de contraseñas

### ❌ NUNCA guardes contraseñas en texto plano

```python
# ❌ MAL - Texto plano
usuarios = {
    'juan': 'micontraseña123'  # ¡PELIGRO!
}
```

**Problemas:**
- Si hay breach, todas las contraseñas se exponen
- Administradores pueden ver contraseñas
- Cumplimiento legal (GDPR, CCPA)

### ✅ Siempre usa hashing

```python
# ✅ BIEN - Hash con bcrypt
import bcrypt

password = 'micontraseña123'
hashed = bcrypt.hashpw(password.encode(), bcrypt.gensalt())

# Se guarda: $2b$12$xyz... (no se puede revertir)
```

**Bcrypt:**
- **One-way hash:** No se puede revertir
- **Salt automático:** Cada hash es único
- **Costoso computacionalmente:** Protege contra brute-force

---

## Headers de autenticación

### Authorization Header

El token se envía en el header `Authorization`:

```http
GET /api/perfil HTTP/1.1
Host: localhost:8000
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
```

**Formato:**
```
Authorization: Bearer <TOKEN>
```

### Alternativas (NO recomendadas)

❌ **Query params:** `/api/perfil?token=xyz` (se guarda en logs)  
❌ **Custom header:** `X-Auth-Token: xyz` (no estándar)  
✅ **Authorization Bearer:** Estándar RFC 6750

---

## Ejemplo básico con Python vanilla

**Instalación:**

```powershell
pip install pyjwt bcrypt
```

**Sistema de autenticación completo:**

**auth_service.py**

```python
"""
Servicio de autenticación con JWT y bcrypt
"""

import jwt
import bcrypt
from datetime import datetime, timedelta
from typing import Optional, Dict

# Configuración
SECRET_KEY = 'tu-clave-secreta-muy-segura-cambiar-en-produccion'
ALGORITHM = 'HS256'
TOKEN_EXPIRATION_HOURS = 24

# Base de datos en memoria (simulada)
usuarios_db = {}

class AuthService:
    """Servicio para autenticación y autorización"""
    
    @staticmethod
    def registrar_usuario(email: str, password: str) -> Dict:
        """Registra un nuevo usuario"""
        # Validar que no exista
        if email in usuarios_db:
            raise ValueError('Email ya registrado')
        
        # Hash de la contraseña
        password_hash = bcrypt.hashpw(
            password.encode('utf-8'),
            bcrypt.gensalt()
        )
        
        # Guardar usuario
        usuarios_db[email] = {
            'email': email,
            'password_hash': password_hash,
            'rol': 'usuario'
        }
        
        return {'email': email, 'mensaje': 'Usuario registrado'}
    
    @staticmethod
    def login(email: str, password: str) -> Dict:
        """Autentica usuario y genera token"""
        # Buscar usuario
        usuario = usuarios_db.get(email)
        if not usuario:
            raise ValueError('Credenciales inválidas')
        
        # Verificar contraseña
        password_correcto = bcrypt.checkpw(
            password.encode('utf-8'),
            usuario['password_hash']
        )
        
        if not password_correcto:
            raise ValueError('Credenciales inválidas')
        
        # Generar token JWT
        payload = {
            'email': email,
            'rol': usuario['rol'],
            'exp': datetime.utcnow() + timedelta(hours=TOKEN_EXPIRATION_HOURS)
        }
        
        token = jwt.encode(payload, SECRET_KEY, algorithm=ALGORITHM)
        
        return {
            'access_token': token,
            'token_type': 'Bearer',
            'expires_in': TOKEN_EXPIRATION_HOURS * 3600
        }
    
    @staticmethod
    def verificar_token(token: str) -> Dict:
        """Verifica y decodifica un token JWT"""
        try:
            payload = jwt.decode(token, SECRET_KEY, algorithms=[ALGORITHM])
            return payload
        except jwt.ExpiredSignatureError:
            raise ValueError('Token expirado')
        except jwt.InvalidTokenError:
            raise ValueError('Token inválido')
    
    @staticmethod
    def obtener_usuario_actual(token: str) -> Dict:
        """Obtiene datos del usuario desde el token"""
        payload = AuthService.verificar_token(token)
        email = payload.get('email')
        
        usuario = usuarios_db.get(email)
        if not usuario:
            raise ValueError('Usuario no encontrado')
        
        return {
            'email': usuario['email'],
            'rol': usuario['rol']
        }
```

---

## API REST con autenticación

**app.py**

```python
"""
API REST con autenticación JWT usando Python vanilla
"""

from http.server import BaseHTTPRequestHandler, HTTPServer
from urllib.parse import urlparse
import json
import re

from auth_service import AuthService

class AuthAPIHandler(BaseHTTPRequestHandler):
    """Handler para API con autenticación"""
    
    def do_POST(self):
        """Maneja peticiones POST"""
        try:
            ruta = urlparse(self.path).path
            
            # POST /api/registro
            if ruta == '/api/registro':
                self.registro()
                return
            
            # POST /api/login
            if ruta == '/api/login':
                self.login()
                return
            
            # 404
            self.enviar_error(404, 'Ruta no encontrada')
            
        except Exception as e:
            print(f'Error en POST: {e}')
            self.enviar_error(500, str(e))
    
    def do_GET(self):
        """Maneja peticiones GET"""
        try:
            ruta = urlparse(self.path).path
            
            # GET /api/perfil (requiere autenticación)
            if ruta == '/api/perfil':
                self.perfil()
                return
            
            # GET /
            if ruta == '/':
                self.home()
                return
            
            # 404
            self.enviar_error(404, 'Ruta no encontrada')
            
        except Exception as e:
            print(f'Error en GET: {e}')
            self.enviar_error(500, str(e))
    
    # ===== Endpoints =====
    
    def home(self):
        """GET / - Página de inicio"""
        html = """
        <!DOCTYPE html>
        <html>
        <head>
            <title>API Autenticación</title>
            <style>
                body { font-family: Arial; max-width: 800px; margin: 50px auto; }
                .endpoint { background: #f5f5f5; padding: 10px; margin: 10px 0; }
            </style>
        </head>
        <body>
            <h1>🔐 API de Autenticación</h1>
            <p>Sistema de autenticación con JWT y bcrypt</p>
            
            <h2>Endpoints:</h2>
            <div class="endpoint">
                <strong>POST /api/registro</strong><br>
                Body: {"email": "...", "password": "..."}
            </div>
            <div class="endpoint">
                <strong>POST /api/login</strong><br>
                Body: {"email": "...", "password": "..."}
            </div>
            <div class="endpoint">
                <strong>GET /api/perfil</strong><br>
                Headers: Authorization: Bearer &lt;token&gt;
            </div>
        </body>
        </html>
        """
        
        self.send_response(200)
        self.send_header('Content-type', 'text/html; charset=utf-8')
        self.end_headers()
        self.wfile.write(html.encode())
    
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
        
        # Validaciones básicas
        if len(password) < 6:
            self.enviar_error(400, 'Password debe tener al menos 6 caracteres')
            return
        
        if '@' not in email:
            self.enviar_error(400, 'Email inválido')
            return
        
        try:
            resultado = AuthService.registrar_usuario(email, password)
            
            self.send_response(201)
            self.send_header('Content-type', 'application/json')
            self.end_headers()
            self.wfile.write(json.dumps(resultado, ensure_ascii=False).encode())
            
        except ValueError as e:
            self.enviar_error(400, str(e))
    
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
        
        try:
            resultado = AuthService.login(email, password)
            self.enviar_json(resultado)
            
        except ValueError as e:
            self.enviar_error(401, str(e))
    
    def perfil(self):
        """GET /api/perfil - Requiere autenticación"""
        # Obtener token del header Authorization
        auth_header = self.headers.get('Authorization')
        
        if not auth_header:
            self.enviar_error(401, 'Token requerido')
            return
        
        # Verificar formato: "Bearer <token>"
        partes = auth_header.split()
        if len(partes) != 2 or partes[0] != 'Bearer':
            self.enviar_error(401, 'Formato de token inválido')
            return
        
        token = partes[1]
        
        try:
            # Verificar token y obtener usuario
            usuario = AuthService.obtener_usuario_actual(token)
            
            self.enviar_json({
                'mensaje': 'Perfil del usuario',
                'usuario': usuario
            })
            
        except ValueError as e:
            self.enviar_error(401, str(e))
    
    # ===== Helpers =====
    
    def enviar_json(self, datos, status=200):
        """Envía respuesta JSON"""
        self.send_response(status)
        self.send_header('Content-type', 'application/json; charset=utf-8')
        self.send_header('Access-Control-Allow-Origin', '*')
        self.end_headers()
        self.wfile.write(json.dumps(datos, ensure_ascii=False, indent=2).encode())
    
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

# ===== Iniciar servidor =====

if __name__ == '__main__':
    puerto = 8000
    servidor = HTTPServer(('localhost', puerto), AuthAPIHandler)
    
    print('='*60)
    print('🔐 API de Autenticación')
    print('='*60)
    print(f'\n🚀 Servidor corriendo en http://localhost:{puerto}')
    print(f'📖 Documentación: http://localhost:{puerto}/')
    print('\n✨ Endpoints disponibles:')
    print('   POST   /api/registro')
    print('   POST   /api/login')
    print('   GET    /api/perfil (requiere token)')
    print('\n⛔ Ctrl+C para detener\n')
    
    try:
        servidor.serve_forever()
    except KeyboardInterrupt:
        print('\n\n⛔ Servidor detenido')
        servidor.shutdown()
```

---

## Probar con curl

### 1. Registrar usuario

```powershell
curl -X POST http://localhost:8000/api/registro `
  -H "Content-Type: application/json" `
  -d '{\"email\":\"ana@ejemplo.com\",\"password\":\"password123\"}'
```

**Response:**
```json
{
  "email": "ana@ejemplo.com",
  "mensaje": "Usuario registrado"
}
```

### 2. Login

```powershell
curl -X POST http://localhost:8000/api/login `
  -H "Content-Type: application/json" `
  -d '{\"email\":\"ana@ejemplo.com\",\"password\":\"password123\"}'
```

**Response:**
```json
{
  "access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "token_type": "Bearer",
  "expires_in": 86400
}
```

### 3. Acceder a ruta protegida

```powershell
# Guardar token
$token = "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."

# Usar token
curl http://localhost:8000/api/perfil `
  -H "Authorization: Bearer $token"
```

**Response:**
```json
{
  "mensaje": "Perfil del usuario",
  "usuario": {
    "email": "ana@ejemplo.com",
    "rol": "usuario"
  }
}
```

### 4. Probar token inválido

```powershell
curl http://localhost:8000/api/perfil `
  -H "Authorization: Bearer token-invalido"
```

**Response:**
```json
{
  "error": "Token inválido",
  "codigo": 401
}
```

---

## Flujo completo

```
┌─────────────┐         ┌──────────────┐
│   Cliente   │         │   Servidor   │
└──────┬──────┘         └───────┬──────┘
       │                        │
       │  POST /api/registro    │
       ├───────────────────────>│
       │  {email, password}     │
       │                        │
       │  201 Created           │
       │<───────────────────────┤
       │                        │
       │  POST /api/login       │
       ├───────────────────────>│
       │  {email, password}     │
       │                        │
       │  Verifica credenciales │
       │  Genera JWT            │
       │                        │
       │  200 OK                │
       │<───────────────────────┤
       │  {access_token}        │
       │                        │
       │  Guarda token          │
       │                        │
       │  GET /api/perfil       │
       ├───────────────────────>│
       │  Authorization: Bearer │
       │                        │
       │  Verifica JWT          │
       │  Decodifica payload    │
       │                        │
       │  200 OK                │
       │<───────────────────────┤
       │  {usuario}             │
       │                        │
```

---

## Mejores prácticas

### 1. Contraseñas seguras

✅ Mínimo 8 caracteres  
✅ Mezcla de mayúsculas, minúsculas, números  
✅ Símbolos especiales  
✅ No palabras del diccionario

**Validación:**

```python
import re

def validar_password_fuerte(password: str) -> tuple[bool, str]:
    """Valida que la contraseña sea segura"""
    if len(password) < 8:
        return False, 'Mínimo 8 caracteres'
    
    if not re.search(r'[a-z]', password):
        return False, 'Debe contener minúsculas'
    
    if not re.search(r'[A-Z]', password):
        return False, 'Debe contener mayúsculas'
    
    if not re.search(r'[0-9]', password):
        return False, 'Debe contener números'
    
    if not re.search(r'[!@#$%^&*(),.?":{}|<>]', password):
        return False, 'Debe contener caracteres especiales'
    
    return True, 'Password válido'

# Uso
valido, mensaje = validar_password_fuerte('MiPass123!')
if not valido:
    print(f'Error: {mensaje}')
```

### 2. Tokens seguros

✅ SECRET_KEY compleja y única  
✅ Almacenar en variables de entorno  
✅ Rotación periódica de claves  
✅ Tiempo de expiración razonable

**Configuración segura:**

```python
import os
from dotenv import load_dotenv

load_dotenv()

SECRET_KEY = os.getenv('JWT_SECRET_KEY')
if not SECRET_KEY:
    raise ValueError('JWT_SECRET_KEY no configurada')

# Generar clave segura:
# python -c "import secrets; print(secrets.token_hex(32))"
```

### 3. Headers de seguridad

```python
def enviar_json_seguro(self, datos, status=200):
    """Envía JSON con headers de seguridad"""
    self.send_response(status)
    self.send_header('Content-type', 'application/json; charset=utf-8')
    self.send_header('X-Content-Type-Options', 'nosniff')
    self.send_header('X-Frame-Options', 'DENY')
    self.send_header('X-XSS-Protection', '1; mode=block')
    self.send_header('Strict-Transport-Security', 'max-age=31536000')
    self.end_headers()
    self.wfile.write(json.dumps(datos, ensure_ascii=False).encode())
```

---

## Errores comunes

### 1. ❌ Enviar contraseña en URL

```python
# ❌ MAL
GET /api/login?email=ana@ejemplo.com&password=123456

# ✅ BIEN
POST /api/login
Body: {"email":"ana@ejemplo.com","password":"123456"}
```

### 2. ❌ Token en query params

```python
# ❌ MAL (se guarda en logs)
GET /api/perfil?token=eyJhbGciOi...

# ✅ BIEN
GET /api/perfil
Authorization: Bearer eyJhbGciOi...
```

### 3. ❌ No validar expiración

```python
# ❌ MAL
payload = jwt.decode(token, SECRET_KEY, algorithms=[ALGORITHM], 
                     options={'verify_exp': False})

# ✅ BIEN (valida automáticamente)
payload = jwt.decode(token, SECRET_KEY, algorithms=[ALGORITHM])
```

### 4. ❌ Secret key hardcodeada

```python
# ❌ MAL
SECRET_KEY = 'mi-clave-123'

# ✅ BIEN
SECRET_KEY = os.getenv('JWT_SECRET_KEY')
```

---

## Resumen

Has aprendido:

✅ Diferencia entre autenticación y autorización  
✅ Flujo completo de autenticación con JWT  
✅ Hashing de contraseñas con bcrypt  
✅ Generar y verificar tokens JWT  
✅ Proteger rutas con Authorization header  
✅ API REST completa con autenticación vanilla Python

**Siguiente:** [JWT en profundidad](./24-jwt-profundidad.md)

---

## Recursos

- **[PyJWT](https://pyjwt.readthedocs.io/)** - JWT para Python
- **[bcrypt](https://github.com/pyca/bcrypt/)** - Hashing de contraseñas
- **[JWT.io](https://jwt.io/)** - Debugger de JWT
- **[OWASP](https://owasp.org/www-project-top-ten/)** - Seguridad web

Tu API ahora tiene autenticación profesional. 🔐

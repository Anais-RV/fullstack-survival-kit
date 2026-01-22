# JWT en profundidad

> **Domina JSON Web Tokens para autenticación profesional**

---

## Estructura interna de un JWT

Un JWT tiene **3 partes** separadas por puntos:

```
HEADER.PAYLOAD.SIGNATURE
```

**Ejemplo real:**
```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.
eyJ1c2VyX2lkIjoxMjMsImVtYWlsIjoiYW5hQGVqZW1wbG8uY29tIiwiZXhwIjoxNzM1Njg5NjAwfQ.
SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c
```

---

## 1. Header (Cabecera)

Contiene el **tipo** de token y el **algoritmo** de firma:

```json
{
  "alg": "HS256",
  "typ": "JWT"
}
```

**Algoritmos comunes:**
- `HS256`: HMAC con SHA-256 (simétrico)
- `RS256`: RSA con SHA-256 (asimétrico)
- `ES256`: ECDSA con SHA-256 (asimétrico)

**Base64 encode:**
```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9
```

---

## 2. Payload (Carga útil)

Contiene los **claims** (afirmaciones) sobre el usuario:

```json
{
  "user_id": 123,
  "email": "ana@ejemplo.com",
  "rol": "admin",
  "exp": 1735689600
}
```

### Tipos de claims

#### Claims registrados (estándar)

| Claim | Descripción | Ejemplo |
|-------|-------------|---------|
| `iss` | Issuer (emisor) | `"mi-api.com"` |
| `sub` | Subject (sujeto/usuario) | `"123"` |
| `aud` | Audience (audiencia) | `"frontend-app"` |
| `exp` | Expiration (expiración) | `1735689600` |
| `nbf` | Not Before (no antes de) | `1735603200` |
| `iat` | Issued At (emitido en) | `1735603200` |
| `jti` | JWT ID (identificador único) | `"abc-123"` |

#### Claims públicos

Registrados en [IANA JWT Registry](https://www.iana.org/assignments/jwt/jwt.xhtml):
- `name`, `email`, `phone_number`, etc.

#### Claims privados

Personalizados para tu aplicación:
- `user_id`, `rol`, `permisos`, `empresa_id`

**Base64 encode:**
```
eyJ1c2VyX2lkIjoxMjMsImVtYWlsIjoiYW5hQGVqZW1wbG8uY29tIiwiZXhwIjoxNzM1Njg5NjAwfQ
```

---

## 3. Signature (Firma)

Garantiza la **integridad** del token:

```
HMACSHA256(
  base64UrlEncode(header) + "." +
  base64UrlEncode(payload),
  secret
)
```

**Resultado:**
```
SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c
```

**¿Cómo funciona?**
1. Combina header + payload codificados
2. Aplica algoritmo de hash (HMAC-SHA256) con clave secreta
3. Codifica el resultado en Base64

**Verificación:**
- El servidor recalcula la firma
- Si coincide con la firma del token → válido
- Si NO coincide → token modificado ❌

---

## Implementación completa en Python

**jwt_service.py**

```python
"""
Servicio JWT profesional con refresh tokens
"""

import jwt
import secrets
from datetime import datetime, timedelta
from typing import Dict, Optional, List

class JWTService:
    """Servicio para generar y validar JWT"""
    
    def __init__(self, secret_key: str, algorithm: str = 'HS256'):
        self.secret_key = secret_key
        self.algorithm = algorithm
        self.access_token_expiration = 15  # minutos
        self.refresh_token_expiration = 7   # días
    
    def generar_access_token(self, user_id: int, email: str, 
                            rol: str = 'usuario', 
                            permisos: List[str] = None) -> str:
        """Genera un access token JWT"""
        ahora = datetime.utcnow()
        
        payload = {
            # Claims registrados
            'iss': 'mi-api.com',
            'sub': str(user_id),
            'iat': ahora,
            'exp': ahora + timedelta(minutes=self.access_token_expiration),
            'jti': secrets.token_hex(16),
            
            # Claims personalizados
            'email': email,
            'rol': rol,
            'permisos': permisos or [],
            'tipo': 'access'
        }
        
        return jwt.encode(payload, self.secret_key, algorithm=self.algorithm)
    
    def generar_refresh_token(self, user_id: int) -> str:
        """Genera un refresh token JWT"""
        ahora = datetime.utcnow()
        
        payload = {
            'sub': str(user_id),
            'iat': ahora,
            'exp': ahora + timedelta(days=self.refresh_token_expiration),
            'jti': secrets.token_hex(16),
            'tipo': 'refresh'
        }
        
        return jwt.encode(payload, self.secret_key, algorithm=self.algorithm)
    
    def verificar_token(self, token: str, tipo_esperado: str = 'access') -> Dict:
        """Verifica y decodifica un token JWT"""
        try:
            # Decodificar y verificar
            payload = jwt.decode(
                token,
                self.secret_key,
                algorithms=[self.algorithm],
                options={
                    'verify_signature': True,
                    'verify_exp': True,
                    'verify_iat': True,
                    'require_exp': True,
                    'require_iat': True
                }
            )
            
            # Verificar tipo de token
            tipo_token = payload.get('tipo')
            if tipo_token != tipo_esperado:
                raise ValueError(f'Token inválido: esperaba {tipo_esperado}, recibió {tipo_token}')
            
            return payload
            
        except jwt.ExpiredSignatureError:
            raise ValueError('Token expirado')
        except jwt.InvalidTokenError as e:
            raise ValueError(f'Token inválido: {str(e)}')
    
    def decodificar_sin_verificar(self, token: str) -> Dict:
        """Decodifica token sin verificar (solo para debug)"""
        return jwt.decode(
            token,
            options={'verify_signature': False}
        )
    
    def renovar_access_token(self, refresh_token: str) -> str:
        """Renueva un access token usando un refresh token"""
        # Verificar refresh token
        payload = self.verificar_token(refresh_token, tipo_esperado='refresh')
        
        # Extraer user_id
        user_id = int(payload['sub'])
        
        # Aquí deberías obtener datos actualizados del usuario desde la DB
        # Por simplicidad, generamos nuevo token básico
        return self.generar_access_token(
            user_id=user_id,
            email=f'user{user_id}@ejemplo.com',  # Debería venir de DB
            rol='usuario'
        )
    
    def extraer_claims(self, token: str) -> Dict:
        """Extrae claims del payload sin verificar firma (debug)"""
        import base64
        import json
        
        # Separar partes
        partes = token.split('.')
        if len(partes) != 3:
            raise ValueError('Token malformado')
        
        # Decodificar payload (segunda parte)
        payload_base64 = partes[1]
        
        # Agregar padding si es necesario
        padding = 4 - (len(payload_base64) % 4)
        if padding != 4:
            payload_base64 += '=' * padding
        
        # Decodificar Base64
        payload_bytes = base64.urlsafe_b64decode(payload_base64)
        payload_json = payload_bytes.decode('utf-8')
        
        return json.loads(payload_json)
```

---

## Refresh Tokens

Los **refresh tokens** permiten obtener nuevos access tokens sin hacer login:

```
┌──────────┐                    ┌──────────┐
│ Cliente  │                    │ Servidor │
└────┬─────┘                    └─────┬────┘
     │                                │
     │  POST /api/login               │
     ├───────────────────────────────>│
     │                                │
     │  200 OK                        │
     │<───────────────────────────────┤
     │  {                             │
     │    access_token: (15 min),     │
     │    refresh_token: (7 días)     │
     │  }                             │
     │                                │
     │  Guarda ambos tokens           │
     │                                │
     │  ... 15 minutos después ...    │
     │                                │
     │  GET /api/datos                │
     ├───────────────────────────────>│
     │  Authorization: Bearer access  │
     │                                │
     │  401 Unauthorized              │
     │<───────────────────────────────┤
     │  {error: "Token expirado"}     │
     │                                │
     │  POST /api/refresh             │
     ├───────────────────────────────>│
     │  {refresh_token: "..."}        │
     │                                │
     │  200 OK                        │
     │<───────────────────────────────┤
     │  {                             │
     │    access_token: (15 min nuevo)│
     │  }                             │
     │                                │
     │  Guarda nuevo access_token     │
     │                                │
```

**Ventajas:**
- Access tokens de corta duración (más seguro)
- No requiere login constante
- Refresh tokens de larga duración

---

## API completa con refresh tokens

**app_jwt.py**

```python
"""
API con access y refresh tokens
"""

from http.server import BaseHTTPRequestHandler, HTTPServer
from urllib.parse import urlparse
import json

from jwt_service import JWTService
import bcrypt

# Configuración
SECRET_KEY = 'clave-secreta-super-segura-cambiar-en-produccion'
jwt_service = JWTService(SECRET_KEY)

# Base de datos simulada
usuarios_db = {}
refresh_tokens_validos = set()  # En producción: Redis o DB

class JWTAPIHandler(BaseHTTPRequestHandler):
    """Handler para API con JWT completo"""
    
    def do_POST(self):
        """Maneja peticiones POST"""
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
            elif ruta == '/api/perfil':
                self.perfil()
            elif ruta == '/api/admin':
                self.admin_only()
            else:
                self.enviar_error(404, 'Ruta no encontrada')
                
        except Exception as e:
            print(f'Error: {e}')
            self.enviar_error(500, str(e))
    
    # ===== Endpoints públicos =====
    
    def home(self):
        """GET /"""
        html = """
        <!DOCTYPE html>
        <html>
        <head>
            <title>API JWT</title>
            <style>
                body { font-family: Arial; max-width: 900px; margin: 50px auto; }
                .endpoint { background: #f5f5f5; padding: 15px; margin: 10px 0; 
                           border-left: 4px solid #4CAF50; }
                code { background: #eee; padding: 2px 6px; }
            </style>
        </head>
        <body>
            <h1>🔐 API con JWT Completo</h1>
            <p>Access tokens (15 min) + Refresh tokens (7 días)</p>
            
            <h2>Endpoints:</h2>
            
            <div class="endpoint">
                <strong>POST /api/registro</strong><br>
                Body: <code>{"email": "...", "password": "..."}</code>
            </div>
            
            <div class="endpoint">
                <strong>POST /api/login</strong><br>
                Body: <code>{"email": "...", "password": "..."}</code><br>
                Returns: <code>{access_token, refresh_token}</code>
            </div>
            
            <div class="endpoint">
                <strong>POST /api/refresh</strong><br>
                Body: <code>{"refresh_token": "..."}</code><br>
                Returns: <code>{access_token}</code>
            </div>
            
            <div class="endpoint">
                <strong>POST /api/logout</strong><br>
                Headers: <code>Authorization: Bearer &lt;token&gt;</code>
            </div>
            
            <div class="endpoint">
                <strong>GET /api/perfil</strong><br>
                Headers: <code>Authorization: Bearer &lt;access_token&gt;</code>
            </div>
            
            <div class="endpoint">
                <strong>GET /api/admin</strong><br>
                Headers: <code>Authorization: Bearer &lt;access_token&gt;</code><br>
                Requiere rol: <code>admin</code>
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
        rol = datos.get('rol', 'usuario')  # Por defecto: usuario
        
        if not email or not password:
            self.enviar_error(400, 'Email y password requeridos')
            return
        
        if email in usuarios_db:
            self.enviar_error(400, 'Email ya registrado')
            return
        
        # Hash de contraseña
        password_hash = bcrypt.hashpw(password.encode(), bcrypt.gensalt())
        
        # Generar ID
        user_id = len(usuarios_db) + 1
        
        # Guardar usuario
        usuarios_db[email] = {
            'id': user_id,
            'email': email,
            'password_hash': password_hash,
            'rol': rol
        }
        
        self.send_response(201)
        self.send_header('Content-type', 'application/json')
        self.end_headers()
        
        respuesta = {
            'mensaje': 'Usuario registrado',
            'user_id': user_id,
            'email': email
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
        
        # Verificar contraseña
        password_valido = bcrypt.checkpw(
            password.encode(),
            usuario['password_hash']
        )
        
        if not password_valido:
            self.enviar_error(401, 'Credenciales inválidas')
            return
        
        # Generar tokens
        access_token = jwt_service.generar_access_token(
            user_id=usuario['id'],
            email=usuario['email'],
            rol=usuario['rol'],
            permisos=['read', 'write']
        )
        
        refresh_token = jwt_service.generar_refresh_token(
            user_id=usuario['id']
        )
        
        # Guardar refresh token
        refresh_tokens_validos.add(refresh_token)
        
        self.enviar_json({
            'access_token': access_token,
            'refresh_token': refresh_token,
            'token_type': 'Bearer',
            'expires_in': 900  # 15 minutos
        })
    
    def refresh(self):
        """POST /api/refresh"""
        datos = self.leer_json_body()
        
        if not datos:
            self.enviar_error(400, 'Datos requeridos')
            return
        
        refresh_token = datos.get('refresh_token')
        
        if not refresh_token:
            self.enviar_error(400, 'Refresh token requerido')
            return
        
        # Verificar que el token esté en lista de válidos
        if refresh_token not in refresh_tokens_validos:
            self.enviar_error(401, 'Refresh token inválido o revocado')
            return
        
        try:
            # Verificar y renovar
            nuevo_access_token = jwt_service.renovar_access_token(refresh_token)
            
            self.enviar_json({
                'access_token': nuevo_access_token,
                'token_type': 'Bearer',
                'expires_in': 900
            })
            
        except ValueError as e:
            # Remover token inválido
            refresh_tokens_validos.discard(refresh_token)
            self.enviar_error(401, str(e))
    
    def logout(self):
        """POST /api/logout"""
        # Extraer token
        token = self.extraer_token()
        if not token:
            return
        
        # Decodificar para obtener jti (ID del token)
        try:
            payload = jwt_service.verificar_token(token)
            
            # En un sistema real, agregar jti a lista negra
            # blacklist.add(payload['jti'])
            
            self.enviar_json({'mensaje': 'Logout exitoso'})
            
        except ValueError as e:
            self.enviar_error(401, str(e))
    
    # ===== Endpoints protegidos =====
    
    def perfil(self):
        """GET /api/perfil - Requiere autenticación"""
        payload = self.requiere_autenticacion()
        if not payload:
            return
        
        self.enviar_json({
            'mensaje': 'Perfil del usuario',
            'usuario': {
                'user_id': payload['sub'],
                'email': payload['email'],
                'rol': payload['rol'],
                'permisos': payload['permisos']
            },
            'token_info': {
                'emitido': payload['iat'],
                'expira': payload['exp'],
                'id_token': payload['jti']
            }
        })
    
    def admin_only(self):
        """GET /api/admin - Solo administradores"""
        payload = self.requiere_autenticacion()
        if not payload:
            return
        
        # Verificar rol
        if payload['rol'] != 'admin':
            self.enviar_error(403, 'Acceso denegado: requiere rol de administrador')
            return
        
        self.enviar_json({
            'mensaje': 'Bienvenido al panel de administración',
            'admin': payload['email']
        })
    
    # ===== Helpers =====
    
    def requiere_autenticacion(self):
        """Middleware: requiere token válido"""
        token = self.extraer_token()
        if not token:
            return None
        
        try:
            payload = jwt_service.verificar_token(token, tipo_esperado='access')
            return payload
        except ValueError as e:
            self.enviar_error(401, str(e))
            return None
    
    def extraer_token(self):
        """Extrae token del header Authorization"""
        auth_header = self.headers.get('Authorization')
        
        if not auth_header:
            self.enviar_error(401, 'Token requerido')
            return None
        
        partes = auth_header.split()
        if len(partes) != 2 or partes[0] != 'Bearer':
            self.enviar_error(401, 'Formato de token inválido')
            return None
        
        return partes[1]
    
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
    servidor = HTTPServer(('localhost', puerto), JWTAPIHandler)
    
    print('='*70)
    print('🔐 API JWT Completa')
    print('='*70)
    print(f'\n🚀 Servidor: http://localhost:{puerto}')
    print(f'\n✨ Características:')
    print('   - Access tokens (15 minutos)')
    print('   - Refresh tokens (7 días)')
    print('   - Roles y permisos')
    print('   - Logout con revocación')
    print('\n⛔ Ctrl+C para detener\n')
    
    try:
        servidor.serve_forever()
    except KeyboardInterrupt:
        print('\n\n⛔ Servidor detenido')
        servidor.shutdown()
```

---

## Flujo completo con curl

### 1. Registrar usuario

```powershell
# Usuario normal
curl -X POST http://localhost:8000/api/registro `
  -H "Content-Type: application/json" `
  -d '{\"email\":\"ana@ejemplo.com\",\"password\":\"Ana123!@#\"}'

# Usuario admin
curl -X POST http://localhost:8000/api/registro `
  -H "Content-Type: application/json" `
  -d '{\"email\":\"admin@ejemplo.com\",\"password\":\"Admin123!@#\",\"rol\":\"admin\"}'
```

### 2. Login

```powershell
curl -X POST http://localhost:8000/api/login `
  -H "Content-Type: application/json" `
  -d '{\"email\":\"ana@ejemplo.com\",\"password\":\"Ana123!@#\"}'
```

**Response:**
```json
{
  "access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "refresh_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "token_type": "Bearer",
  "expires_in": 900
}
```

### 3. Acceder con access token

```powershell
$accessToken = "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."

curl http://localhost:8000/api/perfil `
  -H "Authorization: Bearer $accessToken"
```

### 4. Renovar con refresh token

```powershell
$refreshToken = "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."

curl -X POST http://localhost:8000/api/refresh `
  -H "Content-Type: application/json" `
  -d "{\"refresh_token\":\"$refreshToken\"}"
```

### 5. Acceder a ruta admin

```powershell
# Con usuario admin
curl http://localhost:8000/api/admin `
  -H "Authorization: Bearer $adminAccessToken"

# Con usuario normal → 403 Forbidden
curl http://localhost:8000/api/admin `
  -H "Authorization: Bearer $userAccessToken"
```

### 6. Logout

```powershell
curl -X POST http://localhost:8000/api/logout `
  -H "Authorization: Bearer $accessToken"
```

---

## Decodificar JWT manualmente

**decode_jwt.py**

```python
"""
Script para decodificar y analizar JWT
"""

import base64
import json
import sys

def decodificar_jwt(token: str):
    """Decodifica las partes de un JWT"""
    try:
        # Separar partes
        partes = token.split('.')
        
        if len(partes) != 3:
            print('❌ Token malformado (debe tener 3 partes)')
            return
        
        header_b64, payload_b64, signature_b64 = partes
        
        # Decodificar header
        header = decodificar_base64(header_b64)
        print('📄 HEADER:')
        print(json.dumps(header, indent=2))
        print()
        
        # Decodificar payload
        payload = decodificar_base64(payload_b64)
        print('📦 PAYLOAD:')
        print(json.dumps(payload, indent=2))
        print()
        
        # Mostrar firma (no se puede decodificar sin clave)
        print('🔏 SIGNATURE:')
        print(signature_b64)
        print()
        
        # Analizar claims
        print('📊 ANÁLISIS:')
        analizar_claims(payload)
        
    except Exception as e:
        print(f'❌ Error: {e}')

def decodificar_base64(texto_b64: str) -> dict:
    """Decodifica Base64 URL-safe"""
    # Agregar padding
    padding = 4 - (len(texto_b64) % 4)
    if padding != 4:
        texto_b64 += '=' * padding
    
    # Decodificar
    bytes_decoded = base64.urlsafe_b64decode(texto_b64)
    json_texto = bytes_decoded.decode('utf-8')
    
    return json.loads(json_texto)

def analizar_claims(payload: dict):
    """Analiza los claims del payload"""
    from datetime import datetime
    
    # Expiración
    if 'exp' in payload:
        exp_timestamp = payload['exp']
        exp_datetime = datetime.fromtimestamp(exp_timestamp)
        ahora = datetime.now()
        
        print(f'   Expira: {exp_datetime}')
        
        if exp_datetime > ahora:
            diferencia = exp_datetime - ahora
            minutos = int(diferencia.total_seconds() / 60)
            print(f'   Estado: ✅ Válido (expira en {minutos} minutos)')
        else:
            print(f'   Estado: ❌ Expirado')
    
    # Emisor
    if 'iss' in payload:
        print(f'   Emisor: {payload["iss"]}')
    
    # Sujeto
    if 'sub' in payload:
        print(f'   Usuario ID: {payload["sub"]}')
    
    # Tipo
    if 'tipo' in payload:
        print(f'   Tipo: {payload["tipo"]}')
    
    # Rol
    if 'rol' in payload:
        print(f'   Rol: {payload["rol"]}')

if __name__ == '__main__':
    if len(sys.argv) < 2:
        print('Uso: python decode_jwt.py <token>')
        sys.exit(1)
    
    token = sys.argv[1]
    decodificar_jwt(token)
```

**Uso:**
```powershell
python decode_jwt.py "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
```

---

## Seguridad JWT

### ✅ Buenas prácticas

1. **Tokens de corta duración**
```python
access_token_expiration = 15  # minutos
refresh_token_expiration = 7   # días
```

2. **Clave secreta compleja**
```python
# Generar con:
import secrets
secret = secrets.token_hex(32)
print(secret)  # 64 caracteres hexadecimales
```

3. **HTTPS obligatorio**
```
Los tokens SIEMPRE deben enviarse por HTTPS en producción
```

4. **No incluir datos sensibles**
```python
# ❌ MAL
payload = {'password': 'abc123', 'tarjeta': '1234-5678'}

# ✅ BIEN
payload = {'user_id': 123, 'rol': 'usuario'}
```

5. **Verificar siempre la firma**
```python
# NUNCA uses options={'verify_signature': False}
payload = jwt.decode(token, SECRET_KEY, algorithms=['HS256'])
```

### ❌ Vulnerabilidades comunes

#### 1. Algoritmo "none"

Algunos servidores aceptan JWT sin firma:

```json
{
  "alg": "none",
  "typ": "JWT"
}
```

**Prevención:**
```python
# Especificar algoritmos permitidos explícitamente
jwt.decode(token, SECRET_KEY, algorithms=['HS256'])  # ✅
jwt.decode(token, SECRET_KEY, algorithms=jwt.algorithms.get_default_algorithms())  # ❌
```

#### 2. Cambio de RS256 a HS256

Atacante cambia algoritmo de asimétrico a simétrico y usa clave pública como secret.

**Prevención:**
```python
# Especificar algoritmo esperado
jwt.decode(token, SECRET_KEY, algorithms=['HS256'])  # Solo acepta HS256
```

#### 3. Secret key débil

**❌ Malas claves:**
- `secret`
- `123456`
- `mi-app-2024`

**✅ Buena clave:**
```
a3f8d9c7b2e1f0a4c8d7b6e5f4a3c2b1d0e9f8a7b6c5d4e3f2a1b0c9d8e7f6
```

---

## Revocación de tokens

**Problema:** JWT no se puede "revocar" porque son stateless.

**Soluciones:**

### 1. Lista negra (blacklist)

Guardar tokens revocados en Redis/DB:

```python
blacklist = set()  # En producción: Redis

def verificar_token_no_revocado(token):
    payload = jwt.decode(token, SECRET_KEY, algorithms=['HS256'])
    jti = payload['jti']
    
    if jti in blacklist:
        raise ValueError('Token revocado')
    
    return payload

def revocar_token(token):
    payload = jwt.decode(token, SECRET_KEY, algorithms=['HS256'])
    jti = payload['jti']
    exp = payload['exp']
    
    # Agregar a blacklist con TTL = exp
    blacklist.add(jti)
```

### 2. Refresh tokens revocables

Solo guardar refresh tokens:

```python
refresh_tokens_validos = set()  # DB en producción

# Al hacer login
refresh_token = generar_refresh_token(user_id)
refresh_tokens_validos.add(refresh_token)

# Al renovar
if refresh_token not in refresh_tokens_validos:
    raise ValueError('Token revocado')

# Al hacer logout
refresh_tokens_validos.remove(refresh_token)
```

### 3. Rotación de claves

Cambiar SECRET_KEY periódicamente invalida todos los tokens:

```python
SECRET_KEYS = {
    '2024-01': 'clave-enero',
    '2024-02': 'clave-febrero'
}

key_id = '2024-02'
SECRET_KEY = SECRET_KEYS[key_id]

# Al generar
payload['kid'] = key_id  # Key ID

# Al verificar
payload = jwt.decode(token, options={'verify_signature': False})
key_id = payload['kid']
secret = SECRET_KEYS[key_id]
jwt.decode(token, secret, algorithms=['HS256'])
```

---

## Resumen

Has aprendido:

✅ Estructura interna de JWT (Header, Payload, Signature)  
✅ Claims registrados, públicos y privados  
✅ Implementación completa con refresh tokens  
✅ Revocación de tokens (blacklist, refresh rotation)  
✅ Seguridad: buenas prácticas y vulnerabilidades  
✅ Decodificación manual de JWT para debugging

**Siguiente:** [Bcrypt y hashing](./25-bcrypt-hashing.md)

---

## Recursos

- **[JWT.io](https://jwt.io/)** - Debugger online
- **[RFC 7519](https://datatracker.ietf.org/doc/html/rfc7519)** - Especificación JWT
- **[PyJWT Docs](https://pyjwt.readthedocs.io/)** - Documentación PyJWT
- **[OWASP JWT Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/JSON_Web_Token_for_Java_Cheat_Sheet.html)**

Dominas JWT a nivel profesional. 🔐

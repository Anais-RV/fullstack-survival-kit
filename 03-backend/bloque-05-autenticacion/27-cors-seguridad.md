# CORS y seguridad web

> **Protege tu API de ataques comunes y configura CORS correctamente**

---

## ¿Qué es CORS?

**CORS** = Cross-Origin Resource Sharing

**Problema:** Navegadores bloquean peticiones entre diferentes orígenes por seguridad.

```
Frontend (http://localhost:3000)
    ↓ fetch()
Backend (http://localhost:8000)
    ↓
❌ CORS Error: "No 'Access-Control-Allow-Origin' header"
```

**Origen (origin):** protocolo + dominio + puerto

```
http://localhost:3000 ≠ http://localhost:8000  (diferente puerto)
https://ejemplo.com ≠ http://ejemplo.com       (diferente protocolo)
https://ejemplo.com ≠ https://api.ejemplo.com  (diferente subdominio)
```

---

## CORS en Python vanilla

**cors_handler.py**

```python
"""
Handler con CORS configurado
"""

from http.server import BaseHTTPRequestHandler
import json

class CORSHandler(BaseHTTPRequestHandler):
    """Handler con soporte CORS"""
    
    # Configuración CORS
    ORIGENES_PERMITIDOS = [
        'http://localhost:3000',
        'http://localhost:5173',
        'https://mi-app.com'
    ]
    
    def enviar_headers_cors(self, origin: str = None):
        """Envía headers CORS"""
        # Determinar origen
        if origin and origin in self.ORIGENES_PERMITIDOS:
            allowed_origin = origin
        elif '*' in self.ORIGENES_PERMITIDOS:
            allowed_origin = '*'
        else:
            allowed_origin = self.ORIGENES_PERMITIDOS[0]
        
        # Headers CORS
        self.send_header('Access-Control-Allow-Origin', allowed_origin)
        self.send_header('Access-Control-Allow-Methods', 'GET, POST, PUT, DELETE, OPTIONS')
        self.send_header('Access-Control-Allow-Headers', 'Content-Type, Authorization')
        self.send_header('Access-Control-Allow-Credentials', 'true')
        self.send_header('Access-Control-Max-Age', '86400')  # 24 horas
    
    def do_OPTIONS(self):
        """Maneja preflight requests"""
        origin = self.headers.get('Origin')
        
        self.send_response(204)  # No Content
        self.enviar_headers_cors(origin)
        self.end_headers()
    
    def enviar_json(self, datos, status=200):
        """Envía JSON con CORS"""
        origin = self.headers.get('Origin')
        
        self.send_response(status)
        self.send_header('Content-type', 'application/json; charset=utf-8')
        self.enviar_headers_cors(origin)
        self.end_headers()
        self.wfile.write(json.dumps(datos, ensure_ascii=False).encode())
```

---

## Servidor completo con CORS

**app_cors.py**

```python
"""
API con CORS y headers de seguridad
"""

from http.server import HTTPServer
from urllib.parse import urlparse
import json

from cors_handler import CORSHandler

class SecureAPIHandler(CORSHandler):
    """API con CORS y seguridad"""
    
    def do_GET(self):
        """Maneja GET"""
        try:
            ruta = urlparse(self.path).path
            
            if ruta == '/api/datos':
                self.obtener_datos()
            elif ruta == '/':
                self.home()
            else:
                self.enviar_error(404, 'Ruta no encontrada')
                
        except Exception as e:
            print(f'Error: {e}')
            self.enviar_error(500, str(e))
    
    def do_POST(self):
        """Maneja POST"""
        try:
            ruta = urlparse(self.path).path
            
            if ruta == '/api/datos':
                self.crear_dato()
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
            <title>API CORS</title>
            <style>
                body { font-family: Arial; max-width: 900px; margin: 50px auto; }
            </style>
        </head>
        <body>
            <h1>🌍 API con CORS</h1>
            <p>Endpoints con CORS configurado</p>
            
            <h2>Test CORS:</h2>
            <button onclick="testCORS()">Test GET</button>
            <button onclick="testCORSPost()">Test POST</button>
            <pre id="resultado"></pre>
            
            <script>
                async function testCORS() {
                    try {
                        const response = await fetch('http://localhost:8000/api/datos');
                        const data = await response.json();
                        document.getElementById('resultado').textContent = JSON.stringify(data, null, 2);
                    } catch (error) {
                        document.getElementById('resultado').textContent = 'Error: ' + error.message;
                    }
                }
                
                async function testCORSPost() {
                    try {
                        const response = await fetch('http://localhost:8000/api/datos', {
                            method: 'POST',
                            headers: {'Content-Type': 'application/json'},
                            body: JSON.stringify({nombre: 'Test'})
                        });
                        const data = await response.json();
                        document.getElementById('resultado').textContent = JSON.stringify(data, null, 2);
                    } catch (error) {
                        document.getElementById('resultado').textContent = 'Error: ' + error.message;
                    }
                }
            </script>
        </body>
        </html>
        """
        
        self.send_response(200)
        self.send_header('Content-type', 'text/html; charset=utf-8')
        self.end_headers()
        self.wfile.write(html.encode())
    
    def obtener_datos(self):
        """GET /api/datos"""
        self.enviar_json({
            'datos': ['item1', 'item2', 'item3'],
            'mensaje': 'CORS funcionando correctamente'
        })
    
    def crear_dato(self):
        """POST /api/datos"""
        datos = self.leer_json_body()
        
        if not datos:
            self.enviar_error(400, 'Datos requeridos')
            return
        
        self.enviar_json({
            'mensaje': 'Dato creado',
            'dato': datos
        }, status=201)
    
    def enviar_error(self, codigo, mensaje):
        """Envía error con CORS"""
        origin = self.headers.get('Origin')
        
        self.send_response(codigo)
        self.send_header('Content-type', 'application/json; charset=utf-8')
        self.enviar_headers_cors(origin)
        self.end_headers()
        
        error = {'error': mensaje, 'codigo': codigo}
        self.wfile.write(json.dumps(error, ensure_ascii=False).encode())
    
    def leer_json_body(self):
        """Lee JSON del body"""
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

if __name__ == '__main__':
    puerto = 8000
    servidor = HTTPServer(('localhost', puerto), SecureAPIHandler)
    
    print('='*60)
    print('🌍 API con CORS')
    print('='*60)
    print(f'\n🚀 Servidor: http://localhost:{puerto}')
    print('\n✨ CORS configurado para:')
    print('   - http://localhost:3000')
    print('   - http://localhost:5173')
    print('\n⛔ Ctrl+C para detener\n')
    
    try:
        servidor.serve_forever()
    except KeyboardInterrupt:
        print('\n\n⛔ Servidor detenido')
        servidor.shutdown()
```

---

## Headers de seguridad

**security_headers.py**

```python
"""
Headers de seguridad HTTP
"""

class SecurityHeaders:
    """Agrega headers de seguridad a las respuestas"""
    
    @staticmethod
    def agregar_security_headers(handler):
        """Agrega todos los headers de seguridad"""
        
        # Previene ataques XSS
        handler.send_header('X-Content-Type-Options', 'nosniff')
        
        # Previene clickjacking
        handler.send_header('X-Frame-Options', 'DENY')
        
        # Activa protección XSS del navegador
        handler.send_header('X-XSS-Protection', '1; mode=block')
        
        # Fuerza HTTPS
        handler.send_header(
            'Strict-Transport-Security',
            'max-age=31536000; includeSubDomains'
        )
        
        # Content Security Policy
        handler.send_header(
            'Content-Security-Policy',
            "default-src 'self'; script-src 'self' 'unsafe-inline'; style-src 'self' 'unsafe-inline'"
        )
        
        # Referrer Policy
        handler.send_header('Referrer-Policy', 'strict-origin-when-cross-origin')
        
        # Permissions Policy
        handler.send_header(
            'Permissions-Policy',
            'geolocation=(), microphone=(), camera=()'
        )
```

---

## Rate limiting manual

**rate_limiter.py**

```python
"""
Rate limiting para prevenir abuso
"""

from datetime import datetime, timedelta
from collections import defaultdict
from typing import Dict, Tuple

class RateLimiter:
    """Limita el número de peticiones por IP"""
    
    def __init__(self, max_requests: int = 100, window_seconds: int = 60):
        """
        Args:
            max_requests: Máximo de peticiones permitidas
            window_seconds: Ventana de tiempo en segundos
        """
        self.max_requests = max_requests
        self.window = timedelta(seconds=window_seconds)
        self.requests: Dict[str, list] = defaultdict(list)
    
    def is_allowed(self, ip: str) -> Tuple[bool, int]:
        """
        Verifica si la IP puede hacer más peticiones
        
        Returns:
            (permitido, peticiones_restantes)
        """
        ahora = datetime.now()
        
        # Limpiar peticiones antiguas
        self.requests[ip] = [
            timestamp for timestamp in self.requests[ip]
            if ahora - timestamp < self.window
        ]
        
        # Verificar límite
        peticiones_actuales = len(self.requests[ip])
        
        if peticiones_actuales >= self.max_requests:
            return False, 0
        
        # Registrar petición
        self.requests[ip].append(ahora)
        
        peticiones_restantes = self.max_requests - peticiones_actuales - 1
        return True, peticiones_restantes
    
    def get_reset_time(self, ip: str) -> int:
        """Obtiene segundos hasta reset"""
        if not self.requests[ip]:
            return 0
        
        primera_request = self.requests[ip][0]
        reset_time = primera_request + self.window
        
        return int((reset_time - datetime.now()).total_seconds())

# Instancia global
rate_limiter = RateLimiter(max_requests=100, window_seconds=60)
```

**Uso en handler:**

```python
def do_GET(self):
    """GET con rate limiting"""
    # Obtener IP
    ip = self.client_address[0]
    
    # Verificar rate limit
    permitido, restantes = rate_limiter.is_allowed(ip)
    
    if not permitido:
        reset_seconds = rate_limiter.get_reset_time(ip)
        
        self.send_response(429)  # Too Many Requests
        self.send_header('Content-type', 'application/json')
        self.send_header('X-RateLimit-Limit', str(rate_limiter.max_requests))
        self.send_header('X-RateLimit-Remaining', '0')
        self.send_header('X-RateLimit-Reset', str(reset_seconds))
        self.send_header('Retry-After', str(reset_seconds))
        self.end_headers()
        
        error = {
            'error': 'Too many requests',
            'retry_after_seconds': reset_seconds
        }
        self.wfile.write(json.dumps(error).encode())
        return
    
    # Agregar headers de rate limit
    self.send_response(200)
    self.send_header('X-RateLimit-Limit', str(rate_limiter.max_requests))
    self.send_header('X-RateLimit-Remaining', str(restantes))
    # ... continuar con respuesta normal
```

---

## Validación de inputs

```python
import re
import html

class InputValidator:
    """Valida y sanitiza inputs del usuario"""
    
    @staticmethod
    def sanitizar_string(texto: str, max_length: int = 255) -> str:
        """Limpia string de caracteres peligrosos"""
        if not texto:
            return ''
        
        # Limitar longitud
        texto = texto[:max_length]
        
        # Escapar HTML
        texto = html.escape(texto)
        
        return texto.strip()
    
    @staticmethod
    def validar_email(email: str) -> bool:
        """Valida formato de email"""
        patron = r'^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$'
        return bool(re.match(patron, email))
    
    @staticmethod
    def validar_url(url: str) -> bool:
        """Valida formato de URL"""
        patron = r'^https?://[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}'
        return bool(re.match(patron, url))
    
    @staticmethod
    def validar_sql_injection(texto: str) -> bool:
        """Detecta intentos de SQL injection"""
        palabras_peligrosas = [
            'select', 'insert', 'update', 'delete', 'drop',
            'union', 'exec', 'script', '--', ';'
        ]
        
        texto_lower = texto.lower()
        return not any(palabra in texto_lower for palabra in palabras_peligrosas)
```

---

## Resumen

Has aprendido:

✅ Configurar CORS en Python vanilla  
✅ Headers de seguridad (XSS, clickjacking, CSP)  
✅ Rate limiting manual  
✅ Validación y sanitización de inputs  
✅ Prevención de ataques comunes

**Siguiente:** [Roles y sesiones](./28-roles-sesiones.md)

---

## Recursos

- **[OWASP Top 10](https://owasp.org/www-project-top-ten/)** - Vulnerabilidades comunes
- **[MDN CORS](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS)** - Documentación
- **[Security Headers](https://securityheaders.com/)** - Verificar headers

Tu API está protegida contra ataques comunes. 🛡️

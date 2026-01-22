# Procesamiento de Requests

> **Maneja peticiones HTTP manualmente con Python**

---

## ¿Qué aprenderás?

Cómo procesar requests HTTP desde cero:
- Leer headers
- Parsear query parameters  
- Procesar el body
- Extraer datos del request

**Todo sin frameworks, solo Python.**

---

## Anatomía de un Request HTTP

Un request HTTP crudo se ve así:

```http
POST /api/usuarios?activo=true HTTP/1.1
Host: localhost:8000
Content-Type: application/json
Content-Length: 45
Authorization: Bearer token123

{"nombre":"Juan","email":"juan@ejemplo.com"}
```

**Partes:**
1. **Línea de petición**: método, ruta, protocolo
2. **Headers**: metadata
3. **Línea en blanco**
4. **Body**: datos (opcional)

---

## Acceder a datos del Request

### 1. Información básica

```python
from http.server import BaseHTTPRequestHandler, HTTPServer

class RequestHandler(BaseHTTPRequestHandler):
    def do_GET(self):
        # Método HTTP
        print(f'Método: {self.command}')  # GET
        
        # Ruta completa
        print(f'Ruta: {self.path}')  # /api/usuarios?activo=true
        
        # Versión HTTP
        print(f'Protocolo: {self.request_version}')  # HTTP/1.1
        
        # Dirección del cliente
        print(f'Cliente: {self.client_address}')  # ('127.0.0.1', 54321)
        
        # Respuesta simple
        self.send_response(200)
        self.send_header('Content-type', 'text/plain')
        self.end_headers()
        self.wfile.write(b'OK')
```

---

## Leer Headers

### Acceder a headers del request:

```python
def do_GET(self):
    # Obtener todos los headers
    headers = dict(self.headers)
    print(f'Headers: {headers}')
    
    # Obtener header específico
    content_type = self.headers.get('Content-Type')
    user_agent = self.headers.get('User-Agent')
    authorization = self.headers.get('Authorization')
    
    # Responder con info de headers
    self.send_response(200)
    self.send_header('Content-type', 'application/json')
    self.end_headers()
    
    import json
    response = {
        'headers_recibidos': headers,
        'content_type': content_type,
        'user_agent': user_agent
    }
    self.wfile.write(json.dumps(response, indent=2).encode())
```

### Ejemplo de uso:

```python
# Headers comunes que recibirás:
headers = {
    'Host': 'localhost:8000',
    'User-Agent': 'Mozilla/5.0...',
    'Accept': 'application/json',
    'Content-Type': 'application/json',
    'Authorization': 'Bearer token123'
}
```

---

## Parsear Query Parameters

Los query params vienen después del `?` en la URL:

```
/api/usuarios?nombre=Juan&edad=25&activo=true
             └─────────────┬─────────────┘
                    Query Parameters
```

### Extraer query params:

```python
from urllib.parse import urlparse, parse_qs

class RequestHandler(BaseHTTPRequestHandler):
    def do_GET(self):
        # Parsear la URL
        parsed_url = urlparse(self.path)
        
        # Ruta sin query params
        ruta = parsed_url.path  # /api/usuarios
        
        # Query parameters como diccionario
        query_params = parse_qs(parsed_url.query)
        # {'nombre': ['Juan'], 'edad': ['25'], 'activo': ['true']}
        
        # Extraer valores individuales
        nombre = query_params.get('nombre', [''])[0]
        edad = query_params.get('edad', [''])[0]
        activo = query_params.get('activo', [''])[0] == 'true'
        
        # Responder
        self.send_response(200)
        self.send_header('Content-type', 'application/json')
        self.end_headers()
        
        import json
        response = {
            'ruta': ruta,
            'query_params': {
                'nombre': nombre,
                'edad': int(edad) if edad else None,
                'activo': activo
            }
        }
        self.wfile.write(json.dumps(response, indent=2).encode())
```

### Helper para query params:

```python
def obtener_query_params(self):
    """
    Extrae query parameters del request
    
    Returns:
        dict: Query parameters como {key: value}
    """
    from urllib.parse import urlparse, parse_qs
    
    parsed = urlparse(self.path)
    params_raw = parse_qs(parsed.query)
    
    # Convertir de {key: [value]} a {key: value}
    params = {}
    for key, value_list in params_raw.items():
        # Si hay múltiples valores, mantener lista
        if len(value_list) > 1:
            params[key] = value_list
        else:
            params[key] = value_list[0]
    
    return params
```

### Uso:

```python
def do_GET(self):
    params = self.obtener_query_params()
    
    # Acceder fácilmente
    nombre = params.get('nombre')
    edad = params.get('edad')
    filtros = params.get('filtro')  # Puede ser lista si hay múltiples
```

---

## Leer el Body del Request

Para métodos POST, PUT, PATCH necesitas leer el body:

### Leer body como texto:

```python
def do_POST(self):
    # Obtener longitud del contenido
    content_length = int(self.headers.get('Content-Length', 0))
    
    # Leer el body
    body_raw = self.rfile.read(content_length)
    
    # Decodificar de bytes a string
    body_str = body_raw.decode('utf-8')
    
    print(f'Body recibido: {body_str}')
    
    # Responder
    self.send_response(200)
    self.send_header('Content-type', 'text/plain')
    self.end_headers()
    self.wfile.write(f'Recibido: {body_str}'.encode())
```

### Parsear JSON del body:

```python
def do_POST(self):
    import json
    
    # Leer body
    content_length = int(self.headers.get('Content-Length', 0))
    body_raw = self.rfile.read(content_length)
    
    try:
        # Parsear JSON
        datos = json.loads(body_raw.decode('utf-8'))
        
        # Acceder a los datos
        nombre = datos.get('nombre')
        email = datos.get('email')
        edad = datos.get('edad')
        
        # Procesar...
        print(f'Creando usuario: {nombre}, {email}')
        
        # Responder
        self.send_response(201)  # Created
        self.send_header('Content-type', 'application/json')
        self.end_headers()
        
        response = {
            'mensaje': 'Usuario creado',
            'datos_recibidos': datos
        }
        self.wfile.write(json.dumps(response, indent=2).encode())
    
    except json.JSONDecodeError:
        # Error de JSON
        self.send_response(400)  # Bad Request
        self.send_header('Content-type', 'application/json')
        self.end_headers()
        
        error = {'error': 'JSON inválido'}
        self.wfile.write(json.dumps(error).encode())
```

### Helper para leer JSON:

```python
def leer_json_body(self):
    """
    Lee y parsea JSON del body
    
    Returns:
        dict: Datos parseados o None si hay error
    """
    import json
    
    content_length = int(self.headers.get('Content-Length', 0))
    
    if content_length == 0:
        return None
    
    body_raw = self.rfile.read(content_length)
    
    try:
        return json.loads(body_raw.decode('utf-8'))
    except json.JSONDecodeError:
        return None
```

### Uso:

```python
def do_POST(self):
    datos = self.leer_json_body()
    
    if datos is None:
        self.enviar_error(400, 'JSON inválido o vacío')
        return
    
    # Procesar datos
    nombre = datos.get('nombre')
    email = datos.get('email')
    
    # ...
```

---

## Parsear URL Parameters

URLs con parámetros dinámicos:

```
/api/usuarios/123/posts/456
              └┬┘       └┬┘
            user_id   post_id
```

### Extraer parámetros de la ruta:

```python
import re

def extraer_parametros_ruta(self, patron, ruta):
    """
    Extrae parámetros de una ruta usando regex
    
    Args:
        patron: Patrón regex, ej: r'^/api/usuarios/(\d+)$'
        ruta: Ruta del request
    
    Returns:
        tuple: Parámetros extraídos o None
    """
    match = re.match(patron, ruta)
    if match:
        return match.groups()
    return None

def do_GET(self):
    from urllib.parse import urlparse
    ruta = urlparse(self.path).path
    
    # Intentar match con diferentes patrones
    
    # /api/usuarios/123
    params = self.extraer_parametros_ruta(r'^/api/usuarios/(\d+)$', ruta)
    if params:
        user_id = params[0]
        self.enviar_json({'usuario_id': user_id})
        return
    
    # /api/usuarios/123/posts/456
    params = self.extraer_parametros_ruta(
        r'^/api/usuarios/(\d+)/posts/(\d+)$', ruta
    )
    if params:
        user_id, post_id = params
        self.enviar_json({
            'usuario_id': user_id,
            'post_id': post_id
        })
        return
    
    # No match
    self.enviar_error(404, 'Ruta no encontrada')
```

---

## Sistema completo de procesamiento

**request_processor.py**
```python
"""
Sistema completo de procesamiento de requests
"""

from http.server import HTTPServer, BaseHTTPRequestHandler
from urllib.parse import urlparse, parse_qs
import json
import re

class RequestProcessor(BaseHTTPRequestHandler):
    """Procesador completo de requests HTTP"""
    
    def do_GET(self):
        """Maneja GET requests"""
        # Obtener ruta y query params
        ruta, params = self.parsear_url()
        
        # Log del request
        self.log_request_info()
        
        # Responder con toda la info
        self.send_response(200)
        self.send_header('Content-type', 'application/json')
        self.end_headers()
        
        response = {
            'metodo': 'GET',
            'ruta': ruta,
            'query_params': params,
            'headers': dict(self.headers),
            'cliente': {
                'ip': self.client_address[0],
                'puerto': self.client_address[1]
            }
        }
        
        self.wfile.write(json.dumps(response, indent=2, ensure_ascii=False).encode())
    
    def do_POST(self):
        """Maneja POST requests"""
        ruta, params = self.parsear_url()
        datos = self.leer_json_body()
        
        self.log_request_info()
        
        if datos is None:
            self.enviar_error(400, 'JSON inválido')
            return
        
        # Procesar datos
        self.send_response(201)
        self.send_header('Content-type', 'application/json')
        self.end_headers()
        
        response = {
            'mensaje': 'Datos recibidos correctamente',
            'metodo': 'POST',
            'ruta': ruta,
            'query_params': params,
            'body': datos
        }
        
        self.wfile.write(json.dumps(response, indent=2, ensure_ascii=False).encode())
    
    def parsear_url(self):
        """
        Parsea URL extrayendo ruta y query params
        
        Returns:
            tuple: (ruta, params_dict)
        """
        parsed = urlparse(self.path)
        ruta = parsed.path
        
        # Parsear query params
        params_raw = parse_qs(parsed.query)
        params = {}
        for key, values in params_raw.items():
            params[key] = values[0] if len(values) == 1 else values
        
        return ruta, params
    
    def leer_json_body(self):
        """
        Lee y parsea JSON del body
        
        Returns:
            dict: Datos parseados o None si hay error
        """
        content_length = int(self.headers.get('Content-Length', 0))
        
        if content_length == 0:
            return {}
        
        body_raw = self.rfile.read(content_length)
        
        try:
            return json.loads(body_raw.decode('utf-8'))
        except json.JSONDecodeError:
            return None
    
    def enviar_error(self, codigo, mensaje):
        """Envía respuesta de error"""
        self.send_response(codigo)
        self.send_header('Content-type', 'application/json')
        self.end_headers()
        
        error = {
            'error': mensaje,
            'codigo': codigo
        }
        self.wfile.write(json.dumps(error).encode())
    
    def log_request_info(self):
        """Log detallado del request"""
        print('\n' + '='*50)
        print(f'📥 {self.command} {self.path}')
        print(f'Cliente: {self.client_address[0]}:{self.client_address[1]}')
        print('Headers:')
        for header, value in self.headers.items():
            print(f'  {header}: {value}')
        print('='*50 + '\n')

# Servidor
if __name__ == '__main__':
    servidor = HTTPServer(('localhost', 8000), RequestProcessor)
    print('🚀 Servidor de procesamiento en http://localhost:8000')
    print('Prueba:')
    print('  GET  http://localhost:8000/api/usuarios?activo=true')
    print('  POST http://localhost:8000/api/usuarios')
    
    try:
        servidor.serve_forever()
    except KeyboardInterrupt:
        print('\n⛔ Servidor detenido')
        servidor.shutdown()
```

---

## Pruebas con curl

### GET con query params:

```powershell
curl "http://localhost:8000/api/usuarios?nombre=Juan&edad=25"
```

### POST con JSON:

```powershell
curl -X POST http://localhost:8000/api/usuarios `
  -H "Content-Type: application/json" `
  -d '{\"nombre\":\"Juan\",\"email\":\"juan@ejemplo.com\"}'
```

### GET con headers personalizados:

```powershell
curl http://localhost:8000/api/usuarios `
  -H "Authorization: Bearer token123" `
  -H "Accept: application/json"
```

---

## Validación de datos

```python
def validar_usuario(datos):
    """Valida datos de usuario"""
    errores = []
    
    # Validar nombre
    if 'nombre' not in datos:
        errores.append('nombre es requerido')
    elif len(datos['nombre']) < 2:
        errores.append('nombre debe tener al menos 2 caracteres')
    
    # Validar email
    if 'email' not in datos:
        errores.append('email es requerido')
    elif '@' not in datos['email']:
        errores.append('email inválido')
    
    # Validar edad (opcional)
    if 'edad' in datos:
        try:
            edad = int(datos['edad'])
            if edad < 0 or edad > 150:
                errores.append('edad debe estar entre 0 y 150')
        except ValueError:
            errores.append('edad debe ser un número')
    
    return errores

def do_POST(self):
    datos = self.leer_json_body()
    
    if datos is None:
        self.enviar_error(400, 'JSON inválido')
        return
    
    # Validar
    errores = validar_usuario(datos)
    if errores:
        self.send_response(400)
        self.send_header('Content-type', 'application/json')
        self.end_headers()
        
        response = {
            'error': 'Validación fallida',
            'errores': errores
        }
        self.wfile.write(json.dumps(response, indent=2).encode())
        return
    
    # Datos válidos, procesar...
    self.send_response(201)
    # ...
```

---

## Ejercicios

### 1. Filtros avanzados

Crea un endpoint que acepte múltiples filtros:
```
/api/productos?categoria=laptop&precio_min=500&precio_max=2000&marca=dell
```

### 2. Paginación

Implementa paginación con query params:
```
/api/usuarios?pagina=2&limite=10
```

### 3. Búsqueda

Crea búsqueda con query:
```
/api/usuarios?buscar=juan
```

---

## Resumen

Has aprendido:

✅ Acceder a datos básicos del request  
✅ Leer headers HTTP  
✅ Parsear query parameters  
✅ Leer y parsear el body (JSON)  
✅ Extraer parámetros de rutas  
✅ Validar datos del request  
✅ Logging detallado

**Siguiente:** [Rutas y enrutamiento](./04-rutas-enrutamiento.md)

---

## Recursos

- **[urllib.parse](https://docs.python.org/3/library/urllib.parse.html)** - Parseo de URLs
- **[json](https://docs.python.org/3/library/json.html)** - Manejo de JSON

Ahora controlas completamente los requests HTTP. 💪

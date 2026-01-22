# Primer Servidor Python

> **Crea un servidor HTTP con Python puro en 10 minutos**

---

## ¿Qué vas a crear?

Un servidor HTTP que:
- Escucha en `http://localhost:8000`
- Responde "¡Hola desde mi servidor!" cuando lo visitas
- Funciona sin frameworks, solo Python

**Es simple, pero es un servidor HTTP real.**

---

## Requisitos previos

Solo necesitas Python instalado. Verifica:

```powershell
python --version
```

Deberías ver `Python 3.8+` o superior.

---

## Tu primer servidor (5 líneas)

### Método 1: SimpleHTTPRequestHandler

El método más simple para empezar:

```python
# servidor_simple.py
from http.server import HTTPServer, SimpleHTTPRequestHandler

servidor = HTTPServer(('localhost', 8000), SimpleHTTPRequestHandler)
print('Servidor en http://localhost:8000')
servidor.serve_forever()
```

**Ejecución:**
```powershell
python servidor_simple.py
```

Este servidor sirve archivos del directorio actual. Visita `http://localhost:8000` y verás la lista de archivos.

---

## Método 2: Custom Handler (personalizado)

Para crear respuestas personalizadas:

**servidor.py**
```python
"""
Servidor HTTP básico con Python vanilla
"""

from http.server import HTTPServer, BaseHTTPRequestHandler

class MiManejador(BaseHTTPRequestHandler):
    """Manejador personalizado de peticiones HTTP"""
    
    def do_GET(self):
        """Maneja peticiones GET"""
        # Enviar código de estado
        self.send_response(200)
        
        # Enviar headers
        self.send_header('Content-type', 'text/html; charset=utf-8')
        self.end_headers()
        
        # Crear contenido HTML
        html = """
        <!DOCTYPE html>
        <html>
        <head>
            <title>Mi Servidor</title>
            <style>
                body {
                    font-family: Arial, sans-serif;
                    max-width: 600px;
                    margin: 50px auto;
                    padding: 20px;
                }
                h1 { color: #2563eb; }
                .info {
                    background: #f0f9ff;
                    padding: 15px;
                    border-radius: 8px;
                    margin-top: 20px;
                }
            </style>
        </head>
        <body>
            <h1>¡Hola desde mi servidor!</h1>
            <p>Este es un servidor HTTP creado con Python vanilla.</p>
            <div class="info">
                <p><strong>Ruta solicitada:</strong> {}</p>
                <p><strong>Método:</strong> GET</p>
            </div>
        </body>
        </html>
        """.format(self.path)
        
        # Enviar contenido
        self.wfile.write(html.encode('utf-8'))

# Configuración del servidor
HOST = 'localhost'
PORT = 8000

# Crear servidor
servidor = HTTPServer((HOST, PORT), MiManejador)

print(f'🚀 Servidor corriendo en http://{HOST}:{PORT}')
print('Presiona Ctrl+C para detener')

# Iniciar servidor
try:
    servidor.serve_forever()
except KeyboardInterrupt:
    print('\n⛔ Servidor detenido')
    servidor.shutdown()
```

### Ejecutar:

```powershell
python servidor.py
```

### Probar:

Abre tu navegador en:
- `http://localhost:8000/`
- `http://localhost:8000/usuarios`
- `http://localhost:8000/cualquier-ruta`

---

## Entendiendo el código

### 1. Importaciones

```python
from http.server import HTTPServer, BaseHTTPRequestHandler
```

- **HTTPServer**: crea el servidor
- **BaseHTTPRequestHandler**: clase base para manejar peticiones

### 2. Handler personalizado

```python
class MiManejador(BaseHTTPRequestHandler):
    def do_GET(self):
        # Lógica para peticiones GET
```

**Métodos disponibles:**
- `do_GET()`: maneja peticiones GET
- `do_POST()`: maneja peticiones POST
- `do_PUT()`: maneja peticiones PUT
- `do_DELETE()`: maneja peticiones DELETE

### 3. Enviar respuesta

```python
# 1. Status code
self.send_response(200)

# 2. Headers
self.send_header('Content-type', 'text/html')
self.end_headers()

# 3. Body
self.wfile.write(contenido.encode('utf-8'))
```

### 4. Crear y ejecutar servidor

```python
servidor = HTTPServer(('localhost', 8000), MiManejador)
servidor.serve_forever()
```

---

## Servidor con JSON

Para crear APIs REST, devuelve JSON:

**servidor_json.py**
```python
"""
Servidor que responde con JSON
"""

from http.server import HTTPServer, BaseHTTPRequestHandler
import json

class JSONHandler(BaseHTTPRequestHandler):
    """Handler que devuelve JSON"""
    
    def do_GET(self):
        """Responde con JSON"""
        # Status code
        self.send_response(200)
        
        # Headers para JSON
        self.send_header('Content-type', 'application/json')
        self.send_header('Access-Control-Allow-Origin', '*')  # CORS
        self.end_headers()
        
        # Datos a devolver
        datos = {
            'mensaje': '¡Hola desde mi API!',
            'ruta': self.path,
            'metodo': 'GET',
            'servidor': 'Python vanilla',
            'usuarios': [
                {'id': 1, 'nombre': 'Juan'},
                {'id': 2, 'nombre': 'María'},
                {'id': 3, 'nombre': 'Pedro'}
            ]
        }
        
        # Convertir a JSON y enviar
        json_response = json.dumps(datos, ensure_ascii=False, indent=2)
        self.wfile.write(json_response.encode('utf-8'))

# Configurar y ejecutar
servidor = HTTPServer(('localhost', 8000), JSONHandler)
print('🚀 API corriendo en http://localhost:8000')
print('Prueba: http://localhost:8000/api/usuarios')

try:
    servidor.serve_forever()
except KeyboardInterrupt:
    print('\n⛔ Servidor detenido')
    servidor.shutdown()
```

### Probar la API:

```powershell
# Con curl
curl http://localhost:8000/api/usuarios

# O abre el navegador
# http://localhost:8000/api/usuarios
```

---

## Servidor multi-ruta

Servidor que responde diferente según la ruta:

**servidor_rutas.py**
```python
"""
Servidor con manejo de rutas
"""

from http.server import HTTPServer, BaseHTTPRequestHandler
import json

class RouterHandler(BaseHTTPRequestHandler):
    """Handler con sistema de rutas"""
    
    def do_GET(self):
        """Maneja GET con diferentes rutas"""
        
        # Ruta raíz
        if self.path == '/' or self.path == '':
            self.enviar_html('Bienvenido', 'Página principal del servidor')
        
        # Ruta /api/usuarios
        elif self.path == '/api/usuarios':
            usuarios = [
                {'id': 1, 'nombre': 'Juan', 'email': 'juan@ejemplo.com'},
                {'id': 2, 'nombre': 'María', 'email': 'maria@ejemplo.com'}
            ]
            self.enviar_json(usuarios)
        
        # Ruta /api/productos
        elif self.path == '/api/productos':
            productos = [
                {'id': 1, 'nombre': 'Laptop', 'precio': 1200},
                {'id': 2, 'nombre': 'Mouse', 'precio': 25}
            ]
            self.enviar_json(productos)
        
        # Ruta /about
        elif self.path == '/about':
            self.enviar_html('Acerca de', 'Servidor HTTP con Python vanilla')
        
        # Ruta no encontrada
        else:
            self.enviar_error_404()
    
    def enviar_html(self, titulo, mensaje):
        """Envía respuesta HTML"""
        self.send_response(200)
        self.send_header('Content-type', 'text/html; charset=utf-8')
        self.end_headers()
        
        html = f"""
        <!DOCTYPE html>
        <html>
        <head>
            <title>{titulo}</title>
        </head>
        <body>
            <h1>{titulo}</h1>
            <p>{mensaje}</p>
            <nav>
                <a href="/">Inicio</a> | 
                <a href="/api/usuarios">Usuarios</a> | 
                <a href="/api/productos">Productos</a> | 
                <a href="/about">About</a>
            </nav>
        </body>
        </html>
        """
        self.wfile.write(html.encode('utf-8'))
    
    def enviar_json(self, datos):
        """Envía respuesta JSON"""
        self.send_response(200)
        self.send_header('Content-type', 'application/json')
        self.send_header('Access-Control-Allow-Origin', '*')
        self.end_headers()
        
        json_response = json.dumps(datos, ensure_ascii=False, indent=2)
        self.wfile.write(json_response.encode('utf-8'))
    
    def enviar_error_404(self):
        """Envía error 404"""
        self.send_response(404)
        self.send_header('Content-type', 'application/json')
        self.end_headers()
        
        error = {
            'error': 'Ruta no encontrada',
            'ruta': self.path,
            'codigo': 404
        }
        self.wfile.write(json.dumps(error).encode('utf-8'))
    
    def log_message(self, format, *args):
        """Override para logging personalizado"""
        print(f'📥 {self.command} {self.path} - {args[1]}')

# Configurar y ejecutar
servidor = HTTPServer(('localhost', 8000), RouterHandler)
print('🚀 Servidor con rutas en http://localhost:8000')
print('Rutas disponibles:')
print('  - http://localhost:8000/')
print('  - http://localhost:8000/api/usuarios')
print('  - http://localhost:8000/api/productos')
print('  - http://localhost:8000/about')

try:
    servidor.serve_forever()
except KeyboardInterrupt:
    print('\n⛔ Servidor detenido')
    servidor.shutdown()
```

---

## Debugging y logs

### Ver peticiones entrantes:

```python
def log_message(self, format, *args):
    """Personalizar logs"""
    print(f'📥 {self.command} {self.path}')
    print(f'   Headers: {dict(self.headers)}')
```

### Ver información del request:

```python
def do_GET(self):
    print(f'Método: {self.command}')
    print(f'Ruta: {self.path}')
    print(f'Protocolo: {self.request_version}')
    print(f'Headers: {dict(self.headers)}')
    print(f'Cliente: {self.client_address}')
```

---

## Manejo de errores

```python
def do_GET(self):
    try:
        # Tu lógica aquí
        self.enviar_json({'status': 'ok'})
    
    except Exception as e:
        # Error 500
        self.send_response(500)
        self.send_header('Content-type', 'application/json')
        self.end_headers()
        
        error = {
            'error': 'Error interno del servidor',
            'detalle': str(e)
        }
        self.wfile.write(json.dumps(error).encode('utf-8'))
```

---

## Cambiar el puerto

```python
# Puerto diferente
PORT = 3000  # o cualquier puerto disponible
servidor = HTTPServer(('localhost', PORT), MiManejador)
```

**Puertos comunes:**
- 8000: desarrollo Python
- 3000: desarrollo Node.js
- 5000: Flask (convención)
- 8080: alternativa a 80

---

## Ejercicios prácticos

### 1. Servidor básico

Crea un servidor que:
- Responda "Hola, [tu nombre]" en la ruta `/`
- Devuelva la fecha actual en `/fecha`

### 2. API de películas

Crea una API que devuelva JSON con películas:
- `/api/peliculas` - lista de películas
- Cada película: id, título, año, género

### 3. Servidor con contador

Crea un servidor que:
- Cuente las visitas
- Muestre "Visitas: N" cada vez que se accede

---

## Comparación: Python vanilla vs Flask

### Python vanilla:
```python
from http.server import HTTPServer, BaseHTTPRequestHandler

class Handler(BaseHTTPRequestHandler):
    def do_GET(self):
        self.send_response(200)
        self.send_header('Content-type', 'text/html')
        self.end_headers()
        self.wfile.write(b'Hola')

servidor = HTTPServer(('localhost', 8000), Handler)
servidor.serve_forever()
```

### Flask (para comparar):
```python
from flask import Flask
app = Flask(__name__)

@app.route('/')
def index():
    return 'Hola'

app.run(port=8000)
```

**Ambos hacen lo mismo**, pero Flask abstrae mucho código.  
**Aprende vanilla primero** para entender qué hace Flask por ti.

---

## Resumen

Has aprendido a:

✅ Crear un servidor HTTP con Python puro  
✅ Manejar peticiones GET  
✅ Devolver HTML y JSON  
✅ Implementar rutas básicas  
✅ Manejar errores 404  
✅ Personalizar logs

**Siguiente:** [Procesamiento de requests](./03-procesamiento-requests.md)

---

## Recursos

- **[http.server](https://docs.python.org/3/library/http.server.html)** - Documentación oficial
- **[BaseHTTPRequestHandler](https://docs.python.org/3/library/http.server.html#http.server.BaseHTTPRequestHandler)** - Métodos disponibles

¡Tu primer servidor está funcionando! 🚀

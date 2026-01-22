# Debugging y Testing

> **Herramientas para desarrollo y depuración de servidores Python**

---

## Debugging con `pdb`

**pdb** = Python Debugger (viene incluido)

### Uso básico

```python
from http.server import BaseHTTPRequestHandler, HTTPServer
import json
import pdb  # <-- Importar debugger

class DebugHandler(BaseHTTPRequestHandler):
    def do_POST(self):
        # Leer datos
        content_length = int(self.headers.get('Content-Length', 0))
        body_raw = self.rfile.read(content_length)
        
        # ⚠️ Breakpoint: el servidor se pausará aquí
        pdb.set_trace()
        
        # Ahora puedes inspeccionar variables en la consola
        datos = json.loads(body_raw.decode('utf-8'))
        
        # Procesar...
        self.send_response(200)
        self.send_header('Content-type', 'application/json')
        self.end_headers()
        self.wfile.write(json.dumps({'recibido': datos}).encode())

servidor = HTTPServer(('localhost', 8000), DebugHandler)
servidor.serve_forever()
```

Cuando haces un POST, el servidor se pausará y verás:

```
> /ruta/a/tu/archivo.py(15)do_POST()
-> datos = json.loads(body_raw.decode('utf-8'))
(Pdb) 
```

**Comandos pdb:**

```python
(Pdb) n          # Next: siguiente línea
(Pdb) s          # Step into: entrar en función
(Pdb) c          # Continue: continuar ejecución
(Pdb) l          # List: mostrar código
(Pdb) p variable # Print: imprimir variable
(Pdb) pp datos   # Pretty print: formato bonito
(Pdb) h          # Help: ayuda
(Pdb) q          # Quit: salir
```

---

## Logging

Mejor que `print()` para debugging:

```python
import logging
from http.server import BaseHTTPRequestHandler, HTTPServer
import json

# Configurar logging
logging.basicConfig(
    level=logging.DEBUG,
    format='%(asctime)s - %(name)s - %(levelname)s - %(message)s',
    handlers=[
        logging.FileHandler('servidor.log'),  # A archivo
        logging.StreamHandler()               # A consola
    ]
)

logger = logging.getLogger(__name__)

class LoggedHandler(BaseHTTPRequestHandler):
    def do_GET(self):
        logger.info(f'GET {self.path} desde {self.client_address[0]}')
        
        ruta = self.path
        
        if ruta == '/api/usuarios':
            logger.debug('Consultando usuarios')
            usuarios = self.obtener_usuarios()
            logger.debug(f'Se encontraron {len(usuarios)} usuarios')
            self.enviar_json(usuarios)
        else:
            logger.warning(f'Ruta no encontrada: {ruta}')
            self.enviar_error(404, 'Not found')
    
    def do_POST(self):
        logger.info(f'POST {self.path}')
        
        try:
            datos = self.leer_json_body()
            logger.debug(f'Datos recibidos: {datos}')
            
            # Validar
            if not datos.get('nombre'):
                logger.warning('POST sin nombre')
                self.enviar_error(400, 'Nombre requerido')
                return
            
            # Procesar
            resultado = self.crear_usuario(datos)
            logger.info(f'Usuario creado: {resultado["id"]}')
            self.enviar_json(resultado, 201)
            
        except json.JSONDecodeError as e:
            logger.error(f'JSON inválido: {e}')
            self.enviar_error(400, 'JSON inválido')
        except Exception as e:
            logger.exception('Error inesperado')  # Incluye stack trace
            self.enviar_error(500, 'Error interno')
    
    def obtener_usuarios(self):
        return [
            {'id': 1, 'nombre': 'Juan'},
            {'id': 2, 'nombre': 'María'}
        ]
    
    def crear_usuario(self, datos):
        return {'id': 999, **datos}
    
    def leer_json_body(self):
        content_length = int(self.headers.get('Content-Length', 0))
        body_raw = self.rfile.read(content_length)
        return json.loads(body_raw.decode('utf-8'))
    
    def enviar_json(self, datos, status=200):
        self.send_response(status)
        self.send_header('Content-type', 'application/json')
        self.end_headers()
        self.wfile.write(json.dumps(datos, ensure_ascii=False, indent=2).encode())
    
    def enviar_error(self, codigo, mensaje):
        self.send_response(codigo)
        self.send_header('Content-type', 'application/json')
        self.end_headers()
        self.wfile.write(json.dumps({'error': mensaje}).encode())
    
    def log_message(self, format, *args):
        # Desactivar logs por defecto (ya usamos logger)
        pass

if __name__ == '__main__':
    servidor = HTTPServer(('localhost', 8000), LoggedHandler)
    logger.info('🚀 Servidor iniciado en http://localhost:8000')
    
    try:
        servidor.serve_forever()
    except KeyboardInterrupt:
        logger.info('⛔ Servidor detenido por usuario')
        servidor.shutdown()
```

**Archivo `servidor.log` generado:**

```
2024-01-15 10:30:15,123 - __main__ - INFO - 🚀 Servidor iniciado en http://localhost:8000
2024-01-15 10:30:20,456 - __main__ - INFO - GET /api/usuarios desde 127.0.0.1
2024-01-15 10:30:20,457 - __main__ - DEBUG - Consultando usuarios
2024-01-15 10:30:20,458 - __main__ - DEBUG - Se encontraron 2 usuarios
2024-01-15 10:30:25,789 - __main__ - INFO - POST /api/usuarios
2024-01-15 10:30:25,790 - __main__ - DEBUG - Datos recibidos: {'nombre': 'Pedro', 'email': 'pedro@ejemplo.com'}
2024-01-15 10:30:25,791 - __main__ - INFO - Usuario creado: 999
```

---

## Niveles de logging

```python
logger.debug('Información técnica detallada')    # Solo en desarrollo
logger.info('Evento normal')                     # Informativo
logger.warning('Algo extraño pero no crítico')   # Advertencia
logger.error('Error que permite continuar')      # Error
logger.critical('Error crítico - el sistema falla') # Crítico
logger.exception('Error con stack trace')        # Error + traceback
```

---

## Inspección de requests

```python
from http.server import BaseHTTPRequestHandler, HTTPServer
from urllib.parse import urlparse, parse_qs
import json

class InspectorHandler(BaseHTTPRequestHandler):
    """Handler que muestra toda la información del request"""
    
    def do_GET(self):
        self.inspeccionar()
    
    def do_POST(self):
        self.inspeccionar()
    
    def do_PUT(self):
        self.inspeccionar()
    
    def do_DELETE(self):
        self.inspeccionar()
    
    def inspeccionar(self):
        """Muestra toda la información del request"""
        
        # Parsear URL
        parsed = urlparse(self.path)
        query_params = parse_qs(parsed.query)
        
        # Leer body
        content_length = int(self.headers.get('Content-Length', 0))
        body = ''
        if content_length > 0:
            body_raw = self.rfile.read(content_length)
            body = body_raw.decode('utf-8')
        
        # Construir reporte
        info = {
            'request': {
                'method': self.command,
                'path': self.path,
                'version': self.request_version
            },
            'url': {
                'scheme': parsed.scheme,
                'path': parsed.path,
                'query_string': parsed.query,
                'query_params': {k: v[0] if len(v) == 1 else v for k, v in query_params.items()},
                'fragment': parsed.fragment
            },
            'headers': dict(self.headers),
            'client': {
                'address': self.client_address[0],
                'port': self.client_address[1]
            },
            'body': {
                'length': content_length,
                'raw': body,
                'parsed': self.intentar_parsear_json(body) if body else None
            }
        }
        
        # Mostrar en consola
        print('\n' + '='*60)
        print(f'📥 {self.command} {self.path}')
        print('='*60)
        print(json.dumps(info, indent=2, ensure_ascii=False))
        print('='*60 + '\n')
        
        # Responder
        self.send_response(200)
        self.send_header('Content-type', 'application/json')
        self.end_headers()
        self.wfile.write(json.dumps(info, indent=2, ensure_ascii=False).encode())
    
    def intentar_parsear_json(self, texto):
        """Intenta parsear JSON"""
        try:
            return json.loads(texto)
        except:
            return None
    
    def log_message(self, format, *args):
        pass

if __name__ == '__main__':
    servidor = HTTPServer(('localhost', 8000), InspectorHandler)
    print('🔍 Inspector de requests en http://localhost:8000')
    print('Envía cualquier request para ver toda su información\n')
    
    try:
        servidor.serve_forever()
    except KeyboardInterrupt:
        print('\n⛔ Inspector detenido')
        servidor.shutdown()
```

**Prueba:**

```powershell
curl "http://localhost:8000/api/usuarios?activo=true&rol=admin" `
  -H "Authorization: Bearer token123" `
  -H "Content-Type: application/json" `
  -X POST `
  -d '{\"nombre\":\"Juan\",\"edad\":30}'
```

**Output del inspector:**

```json
{
  "request": {
    "method": "POST",
    "path": "/api/usuarios?activo=true&rol=admin",
    "version": "HTTP/1.1"
  },
  "url": {
    "scheme": "",
    "path": "/api/usuarios",
    "query_string": "activo=true&rol=admin",
    "query_params": {
      "activo": "true",
      "rol": "admin"
    },
    "fragment": ""
  },
  "headers": {
    "Host": "localhost:8000",
    "User-Agent": "curl/7.68.0",
    "Accept": "*/*",
    "Authorization": "Bearer token123",
    "Content-Type": "application/json",
    "Content-Length": "30"
  },
  "client": {
    "address": "127.0.0.1",
    "port": 54321
  },
  "body": {
    "length": 30,
    "raw": "{\"nombre\":\"Juan\",\"edad\":30}",
    "parsed": {
      "nombre": "Juan",
      "edad": 30
    }
  }
}
```

---

## Testing con `requests`

Para probar tu servidor sin curl:

```python
# test_servidor.py
import requests
import json

BASE_URL = 'http://localhost:8000'

def test_listar_usuarios():
    """Test: GET /api/usuarios"""
    response = requests.get(f'{BASE_URL}/api/usuarios')
    
    print(f'Status: {response.status_code}')
    print(f'Headers: {response.headers}')
    print(f'Body: {response.json()}')
    
    assert response.status_code == 200
    assert isinstance(response.json(), list)
    print('✅ Test listar_usuarios pasado')

def test_crear_usuario():
    """Test: POST /api/usuarios"""
    nuevo_usuario = {
        'nombre': 'Pedro',
        'email': 'pedro@ejemplo.com'
    }
    
    response = requests.post(
        f'{BASE_URL}/api/usuarios',
        json=nuevo_usuario
    )
    
    print(f'Status: {response.status_code}')
    print(f'Body: {response.json()}')
    
    assert response.status_code == 201
    assert response.json()['nombre'] == 'Pedro'
    print('✅ Test crear_usuario pasado')

def test_obtener_usuario():
    """Test: GET /api/usuarios/:id"""
    user_id = 123
    response = requests.get(f'{BASE_URL}/api/usuarios/{user_id}')
    
    print(f'Status: {response.status_code}')
    print(f'Body: {response.json()}')
    
    assert response.status_code == 200
    assert response.json()['id'] == user_id
    print('✅ Test obtener_usuario pasado')

def test_actualizar_usuario():
    """Test: PUT /api/usuarios/:id"""
    user_id = 123
    datos = {'nombre': 'Juan Actualizado'}
    
    response = requests.put(
        f'{BASE_URL}/api/usuarios/{user_id}',
        json=datos
    )
    
    print(f'Status: {response.status_code}')
    print(f'Body: {response.json()}')
    
    assert response.status_code == 200
    assert response.json()['nombre'] == 'Juan Actualizado'
    print('✅ Test actualizar_usuario pasado')

def test_eliminar_usuario():
    """Test: DELETE /api/usuarios/:id"""
    user_id = 123
    response = requests.delete(f'{BASE_URL}/api/usuarios/{user_id}')
    
    print(f'Status: {response.status_code}')
    print(f'Body: {response.json()}')
    
    assert response.status_code == 200
    print('✅ Test eliminar_usuario pasado')

def test_ruta_no_encontrada():
    """Test: 404 Not Found"""
    response = requests.get(f'{BASE_URL}/ruta/inexistente')
    
    print(f'Status: {response.status_code}')
    print(f'Body: {response.json()}')
    
    assert response.status_code == 404
    print('✅ Test 404 pasado')

if __name__ == '__main__':
    print('🧪 Ejecutando tests...\n')
    
    try:
        test_listar_usuarios()
        print()
        
        test_crear_usuario()
        print()
        
        test_obtener_usuario()
        print()
        
        test_actualizar_usuario()
        print()
        
        test_eliminar_usuario()
        print()
        
        test_ruta_no_encontrada()
        print()
        
        print('✅ Todos los tests pasaron')
    except AssertionError as e:
        print(f'❌ Test falló: {e}')
    except requests.ConnectionError:
        print('❌ No se pudo conectar al servidor. ¿Está corriendo?')
```

**Instalar `requests`:**

```powershell
pip install requests
```

**Ejecutar tests:**

```powershell
# Terminal 1: Servidor
python servidor.py

# Terminal 2: Tests
python test_servidor.py
```

---

## Testing con `unittest`

Para tests más profesionales:

```python
# test_api.py
import unittest
import requests
import json

BASE_URL = 'http://localhost:8000'

class TestAPI(unittest.TestCase):
    """Tests para la API REST"""
    
    def setUp(self):
        """Se ejecuta antes de cada test"""
        print(f'\n🧪 Ejecutando: {self._testMethodName}')
    
    def test_01_listar_usuarios(self):
        """GET /api/usuarios debe retornar lista"""
        response = requests.get(f'{BASE_URL}/api/usuarios')
        
        self.assertEqual(response.status_code, 200)
        self.assertIsInstance(response.json(), list)
        self.assertGreater(len(response.json()), 0)
    
    def test_02_crear_usuario(self):
        """POST /api/usuarios debe crear usuario"""
        nuevo = {'nombre': 'Test User', 'email': 'test@ejemplo.com'}
        response = requests.post(f'{BASE_URL}/api/usuarios', json=nuevo)
        
        self.assertEqual(response.status_code, 201)
        self.assertIn('id', response.json())
        self.assertEqual(response.json()['nombre'], 'Test User')
    
    def test_03_crear_usuario_sin_datos(self):
        """POST sin datos debe retornar 400"""
        response = requests.post(f'{BASE_URL}/api/usuarios', json={})
        
        self.assertIn(response.status_code, [400, 422])
    
    def test_04_obtener_usuario(self):
        """GET /api/usuarios/:id debe retornar usuario"""
        response = requests.get(f'{BASE_URL}/api/usuarios/123')
        
        self.assertEqual(response.status_code, 200)
        self.assertEqual(response.json()['id'], 123)
    
    def test_05_actualizar_usuario(self):
        """PUT /api/usuarios/:id debe actualizar"""
        datos = {'nombre': 'Actualizado'}
        response = requests.put(f'{BASE_URL}/api/usuarios/123', json=datos)
        
        self.assertEqual(response.status_code, 200)
        self.assertEqual(response.json()['nombre'], 'Actualizado')
    
    def test_06_eliminar_usuario(self):
        """DELETE /api/usuarios/:id debe eliminar"""
        response = requests.delete(f'{BASE_URL}/api/usuarios/123')
        
        self.assertEqual(response.status_code, 200)
    
    def test_07_ruta_no_encontrada(self):
        """Ruta inexistente debe retornar 404"""
        response = requests.get(f'{BASE_URL}/ruta/inexistente')
        
        self.assertEqual(response.status_code, 404)
        self.assertIn('error', response.json())
    
    def test_08_metodo_no_permitido(self):
        """Método no soportado debe retornar error"""
        response = requests.patch(f'{BASE_URL}/api/usuarios/123')
        
        # Nota: BaseHTTPRequestHandler retorna 501 por defecto
        self.assertIn(response.status_code, [405, 501])
    
    @classmethod
    def setUpClass(cls):
        """Se ejecuta una vez al inicio"""
        print('\n' + '='*60)
        print('🧪 Iniciando suite de tests')
        print('='*60)
    
    @classmethod
    def tearDownClass(cls):
        """Se ejecuta una vez al final"""
        print('\n' + '='*60)
        print('✅ Suite de tests completada')
        print('='*60)

if __name__ == '__main__':
    # Ejecutar tests con verbosidad
    unittest.main(verbosity=2)
```

**Ejecutar:**

```powershell
python test_api.py
```

**Output:**

```
============================================================
🧪 Iniciando suite de tests
============================================================

🧪 Ejecutando: test_01_listar_usuarios
test_01_listar_usuarios (__main__.TestAPI)
GET /api/usuarios debe retornar lista ... ok

🧪 Ejecutando: test_02_crear_usuario
test_02_crear_usuario (__main__.TestAPI)
POST /api/usuarios debe crear usuario ... ok

🧪 Ejecutando: test_03_crear_usuario_sin_datos
test_03_crear_usuario_sin_datos (__main__.TestAPI)
POST sin datos debe retornar 400 ... ok

...

----------------------------------------------------------------------
Ran 8 tests in 0.123s

OK

============================================================
✅ Suite de tests completada
============================================================
```

---

## Modo desarrollo vs producción

```python
import os
import logging
from http.server import BaseHTTPRequestHandler, HTTPServer

# Detectar modo
MODO = os.getenv('MODO', 'desarrollo')  # Por defecto desarrollo
ES_DESARROLLO = MODO == 'desarrollo'

# Configurar logging según modo
if ES_DESARROLLO:
    logging.basicConfig(level=logging.DEBUG)
else:
    logging.basicConfig(
        level=logging.WARNING,
        filename='produccion.log'
    )

logger = logging.getLogger(__name__)

class ModoHandler(BaseHTTPRequestHandler):
    def do_GET(self):
        try:
            # Tu lógica aquí
            resultado = self.procesar()
            self.enviar_json(resultado)
            
        except Exception as e:
            if ES_DESARROLLO:
                # En desarrollo: mostrar error completo
                logger.exception('Error en GET')
                self.enviar_error(500, str(e))
            else:
                # En producción: no exponer detalles
                logger.exception('Error en GET')
                self.enviar_error(500, 'Error interno del servidor')
    
    def procesar(self):
        # Simular procesamiento
        if ES_DESARROLLO:
            logger.debug('Procesando en modo desarrollo')
        
        return {'resultado': 'ok', 'modo': MODO}
    
    def enviar_json(self, datos):
        self.send_response(200)
        self.send_header('Content-type', 'application/json')
        
        if ES_DESARROLLO:
            # En desarrollo: indentación bonita
            body = json.dumps(datos, indent=2, ensure_ascii=False)
        else:
            # En producción: compacto
            body = json.dumps(datos, ensure_ascii=False)
        
        self.end_headers()
        self.wfile.write(body.encode())
    
    def enviar_error(self, codigo, mensaje):
        self.send_response(codigo)
        self.send_header('Content-type', 'application/json')
        self.end_headers()
        
        error = {'error': mensaje, 'codigo': codigo}
        
        if ES_DESARROLLO:
            error['modo'] = 'desarrollo'
        
        self.wfile.write(json.dumps(error, indent=2).encode())

if __name__ == '__main__':
    puerto = 8000 if ES_DESARROLLO else 80
    servidor = HTTPServer(('localhost', puerto), ModoHandler)
    
    logger.info(f'🚀 Servidor en modo {MODO} - Puerto {puerto}')
    
    servidor.serve_forever()
```

**Uso:**

```powershell
# Desarrollo (por defecto)
python servidor.py

# Producción
$env:MODO="produccion"
python servidor.py
```

---

## Ejercicios

### 1. Sistema de logs rotativo
Implementa logs que se rotan por tamaño:
```python
from logging.handlers import RotatingFileHandler
```

### 2. Profiler
Mide el tiempo de cada request:
```python
import time

inicio = time.time()
# Procesar...
fin = time.time()
logger.info(f'Request procesado en {fin - inicio:.3f}s')
```

### 3. Health check endpoint
```python
@router.get('/health')
def health(request):
    return {
        'status': 'ok',
        'timestamp': time.time(),
        'uptime': get_uptime()
    }
```

---

## Resumen

Has aprendido:

✅ Debugging con `pdb`  
✅ Logging profesional  
✅ Inspección de requests  
✅ Testing con `requests`  
✅ Testing con `unittest`  
✅ Modo desarrollo vs producción

Con estas herramientas puedes detectar y corregir bugs rápidamente. 🐛

---

## Siguiente bloque

**Siguiente:** [Bloque 2 - APIs REST vanilla](../bloque-02-api-rest/)

---

## Recursos

- **[pdb](https://docs.python.org/3/library/pdb.html)** - Python Debugger
- **[logging](https://docs.python.org/3/library/logging.html)** - Logging facility
- **[unittest](https://docs.python.org/3/library/unittest.html)** - Unit testing framework
- **[requests](https://requests.readthedocs.io/)** - HTTP library

Tu servidor ahora es robusto y fácil de debuggear. 🔧

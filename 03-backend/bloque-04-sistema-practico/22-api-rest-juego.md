# API REST para el juego RPG

> **Expone el juego completo mediante API REST con Python vanilla**

---

## Introducción

Integraremos todo el sistema RPG en una API REST completa usando **Python vanilla** (http.server).

**Endpoints principales:**
- `POST /api/personajes` - Crear personaje
- `GET /api/personajes` - Listar personajes
- `GET /api/personajes/{id}` - Ver personaje
- `PUT /api/personajes/{id}/equipar` - Equipar item
- `POST /api/combates` - Iniciar combate
- `POST /api/combates/{id}/accion` - Realizar acción
- `GET /api/combates/{id}` - Estado del combate

---

## Repositorio en memoria

**repositorios/repositorio_memoria.py**

```python
"""
Repositorio en memoria para personajes y combates
"""

from typing import Dict, List, Optional
from modelos.personaje import Personaje
from sistemas.combate import Combate

class RepositorioPersonajes:
    """Repositorio para gestionar personajes"""
    
    def __init__(self):
        self.personajes: Dict[int, Personaje] = {}
        self.siguiente_id = 1
    
    def guardar(self, personaje: Personaje) -> Personaje:
        """Guarda un personaje"""
        if personaje.id == 0:
            personaje.id = self.siguiente_id
            self.siguiente_id += 1
        self.personajes[personaje.id] = personaje
        return personaje
    
    def obtener_por_id(self, id: int) -> Optional[Personaje]:
        """Obtiene personaje por ID"""
        return self.personajes.get(id)
    
    def obtener_todos(self) -> List[Personaje]:
        """Obtiene todos los personajes"""
        return list(self.personajes.values())
    
    def eliminar(self, id: int) -> bool:
        """Elimina un personaje"""
        if id in self.personajes:
            del self.personajes[id]
            return True
        return False

class RepositorioCombates:
    """Repositorio para gestionar combates"""
    
    def __init__(self):
        self.combates: Dict[int, Combate] = {}
        self.siguiente_id = 1
    
    def guardar(self, combate: Combate) -> Combate:
        """Guarda un combate"""
        if not hasattr(combate, 'id') or combate.id == 0:
            combate.id = self.siguiente_id
            self.siguiente_id += 1
        self.combates[combate.id] = combate
        return combate
    
    def obtener_por_id(self, id: int) -> Optional[Combate]:
        """Obtiene combate por ID"""
        return self.combates.get(id)
    
    def obtener_todos(self) -> List[Combate]:
        """Obtiene todos los combates"""
        return list(self.combates.values())
```

---

## API REST completa

**app.py**

```python
"""
API REST para juego RPG con Python vanilla
"""

from http.server import BaseHTTPRequestHandler, HTTPServer
from urllib.parse import urlparse, parse_qs
import json
import re

# Importar modelos y sistemas
from modelos.personaje import Personaje, Guerrero, Mago, Arquero
from modelos.items import Arma, Armadura
from sistemas.combate import Combate
from repositorios.repositorio_memoria import RepositorioPersonajes, RepositorioCombates

# Repositorios globales
repo_personajes = RepositorioPersonajes()
repo_combates = RepositorioCombates()

class RPGAPIHandler(BaseHTTPRequestHandler):
    """Handler para API REST del juego RPG"""
    
    def do_GET(self):
        """Maneja requests GET"""
        try:
            ruta = urlparse(self.path).path
            query = parse_qs(urlparse(self.path).query)
            
            # GET /
            if ruta == '/':
                self.home()
                return
            
            # GET /api/info
            if ruta == '/api/info':
                self.info_juego()
                return
            
            # GET /api/personajes
            if ruta == '/api/personajes':
                self.listar_personajes()
                return
            
            # GET /api/personajes/:id
            match_personaje = re.match(r'^/api/personajes/(\d+)$', ruta)
            if match_personaje:
                personaje_id = int(match_personaje.group(1))
                self.obtener_personaje(personaje_id)
                return
            
            # GET /api/combates
            if ruta == '/api/combates':
                self.listar_combates()
                return
            
            # GET /api/combates/:id
            match_combate = re.match(r'^/api/combates/(\d+)$', ruta)
            if match_combate:
                combate_id = int(match_combate.group(1))
                self.obtener_combate(combate_id)
                return
            
            # 404
            self.enviar_error(404, 'Ruta no encontrada')
            
        except Exception as e:
            print(f'Error en GET: {e}')
            self.enviar_error(500, 'Error interno del servidor')
    
    def do_POST(self):
        """Maneja requests POST"""
        try:
            ruta = urlparse(self.path).path
            
            # POST /api/personajes
            if ruta == '/api/personajes':
                self.crear_personaje()
                return
            
            # POST /api/combates
            if ruta == '/api/combates':
                self.crear_combate()
                return
            
            # POST /api/combates/:id/accion
            match_accion = re.match(r'^/api/combates/(\d+)/accion$', ruta)
            if match_accion:
                combate_id = int(match_accion.group(1))
                self.realizar_accion(combate_id)
                return
            
            # 404
            self.enviar_error(404, 'Ruta no encontrada')
            
        except Exception as e:
            print(f'Error en POST: {e}')
            self.enviar_error(500, 'Error interno del servidor')
    
    def do_PUT(self):
        """Maneja requests PUT"""
        try:
            ruta = urlparse(self.path).path
            
            # PUT /api/personajes/:id/equipar
            match_equipar = re.match(r'^/api/personajes/(\d+)/equipar$', ruta)
            if match_equipar:
                personaje_id = int(match_equipar.group(1))
                self.equipar_item(personaje_id)
                return
            
            # 404
            self.enviar_error(404, 'Ruta no encontrada')
            
        except Exception as e:
            print(f'Error en PUT: {e}')
            self.enviar_error(500, 'Error interno del servidor')
    
    def do_DELETE(self):
        """Maneja requests DELETE"""
        try:
            ruta = urlparse(self.path).path
            
            # DELETE /api/personajes/:id
            match = re.match(r'^/api/personajes/(\d+)$', ruta)
            if match:
                personaje_id = int(match.group(1))
                self.eliminar_personaje(personaje_id)
                return
            
            # 404
            self.enviar_error(404, 'Ruta no encontrada')
            
        except Exception as e:
            print(f'Error en DELETE: {e}')
            self.enviar_error(500, 'Error interno del servidor')
    
    # ===== Endpoints =====
    
    def home(self):
        """GET / - Página de inicio"""
        html = """
        <!DOCTYPE html>
        <html>
        <head>
            <title>API RPG</title>
            <style>
                body { font-family: Arial; max-width: 800px; margin: 50px auto; padding: 20px; }
                h1 { color: #333; }
                .endpoint { background: #f5f5f5; padding: 10px; margin: 10px 0; border-radius: 5px; }
                .method { color: white; padding: 3px 8px; border-radius: 3px; font-weight: bold; }
                .get { background: #61affe; }
                .post { background: #49cc90; }
                .put { background: #fca130; }
                .delete { background: #f93e3e; }
            </style>
        </head>
        <body>
            <h1>🎮 API REST - Juego RPG</h1>
            <p>Sistema de combate por turnos con Python vanilla</p>
            
            <h2>Endpoints disponibles:</h2>
            
            <div class="endpoint">
                <span class="method get">GET</span>
                <strong>/api/info</strong> - Información del juego
            </div>
            
            <div class="endpoint">
                <span class="method get">GET</span>
                <strong>/api/personajes</strong> - Listar personajes
            </div>
            
            <div class="endpoint">
                <span class="method post">POST</span>
                <strong>/api/personajes</strong> - Crear personaje
            </div>
            
            <div class="endpoint">
                <span class="method get">GET</span>
                <strong>/api/personajes/:id</strong> - Ver personaje
            </div>
            
            <div class="endpoint">
                <span class="method put">PUT</span>
                <strong>/api/personajes/:id/equipar</strong> - Equipar item
            </div>
            
            <div class="endpoint">
                <span class="method delete">DELETE</span>
                <strong>/api/personajes/:id</strong> - Eliminar personaje
            </div>
            
            <div class="endpoint">
                <span class="method post">POST</span>
                <strong>/api/combates</strong> - Iniciar combate
            </div>
            
            <div class="endpoint">
                <span class="method get">GET</span>
                <strong>/api/combates/:id</strong> - Ver combate
            </div>
            
            <div class="endpoint">
                <span class="method post">POST</span>
                <strong>/api/combates/:id/accion</strong> - Realizar acción
            </div>
        </body>
        </html>
        """
        
        self.send_response(200)
        self.send_header('Content-type', 'text/html; charset=utf-8')
        self.end_headers()
        self.wfile.write(html.encode())
    
    def info_juego(self):
        """GET /api/info"""
        info = {
            'nombre': 'Sistema RPG',
            'version': '1.0.0',
            'descripcion': 'Sistema de combate por turnos',
            'clases_disponibles': ['guerrero', 'mago', 'arquero'],
            'estadisticas': {
                'personajes': len(repo_personajes.obtener_todos()),
                'combates': len(repo_combates.obtener_todos())
            }
        }
        self.enviar_json(info)
    
    def listar_personajes(self):
        """GET /api/personajes"""
        personajes = repo_personajes.obtener_todos()
        personajes_dict = [p.to_dict() for p in personajes]
        
        self.enviar_json({
            'personajes': personajes_dict,
            'total': len(personajes_dict)
        })
    
    def obtener_personaje(self, personaje_id: int):
        """GET /api/personajes/:id"""
        personaje = repo_personajes.obtener_por_id(personaje_id)
        
        if not personaje:
            self.enviar_error(404, f'Personaje {personaje_id} no encontrado')
            return
        
        self.enviar_json(personaje.to_dict())
    
    def crear_personaje(self):
        """POST /api/personajes"""
        datos = self.leer_json_body()
        
        if not datos:
            self.enviar_error(400, 'Datos requeridos')
            return
        
        # Validar campos
        nombre = datos.get('nombre')
        clase = datos.get('clase', '').lower()
        
        if not nombre or not clase:
            self.enviar_error(400, 'Campos nombre y clase son requeridos')
            return
        
        # Crear personaje según clase
        try:
            if clase == 'guerrero':
                personaje = Guerrero(nombre)
            elif clase == 'mago':
                personaje = Mago(nombre)
            elif clase == 'arquero':
                personaje = Arquero(nombre)
            else:
                self.enviar_error(400, f'Clase inválida: {clase}. Use: guerrero, mago o arquero')
                return
            
            # Guardar
            personaje = repo_personajes.guardar(personaje)
            
            # Responder
            self.send_response(201)
            self.send_header('Content-type', 'application/json')
            self.send_header('Location', f'/api/personajes/{personaje.id}')
            self.end_headers()
            self.wfile.write(json.dumps(personaje.to_dict(), ensure_ascii=False, indent=2).encode())
            
        except Exception as e:
            self.enviar_error(500, f'Error al crear personaje: {str(e)}')
    
    def equipar_item(self, personaje_id: int):
        """PUT /api/personajes/:id/equipar"""
        personaje = repo_personajes.obtener_por_id(personaje_id)
        
        if not personaje:
            self.enviar_error(404, f'Personaje {personaje_id} no encontrado')
            return
        
        datos = self.leer_json_body()
        if not datos:
            self.enviar_error(400, 'Datos requeridos')
            return
        
        tipo_item = datos.get('tipo')
        nombre_item = datos.get('nombre')
        
        if not tipo_item or not nombre_item:
            self.enviar_error(400, 'Campos tipo y nombre son requeridos')
            return
        
        try:
            # Crear item según tipo
            if tipo_item == 'arma':
                daño = datos.get('daño', 10)
                item = Arma(nombre_item, daño)
            elif tipo_item == 'armadura':
                defensa = datos.get('defensa', 5)
                item = Armadura(nombre_item, defensa)
            else:
                self.enviar_error(400, f'Tipo de item inválido: {tipo_item}')
                return
            
            # Equipar
            personaje.equipar(item)
            
            self.enviar_json({
                'mensaje': f'{nombre_item} equipado',
                'personaje': personaje.to_dict()
            })
            
        except Exception as e:
            self.enviar_error(400, str(e))
    
    def eliminar_personaje(self, personaje_id: int):
        """DELETE /api/personajes/:id"""
        if repo_personajes.eliminar(personaje_id):
            self.send_response(204)
            self.end_headers()
        else:
            self.enviar_error(404, f'Personaje {personaje_id} no encontrado')
    
    def listar_combates(self):
        """GET /api/combates"""
        combates = repo_combates.obtener_todos()
        combates_dict = []
        
        for combate in combates:
            combates_dict.append({
                'id': combate.id,
                'atacante': combate.atacante.nombre,
                'defensor': combate.defensor.nombre,
                'turno_actual': combate.turno_actual,
                'finalizado': combate.finalizado,
                'ganador': combate.ganador.nombre if combate.ganador else None
            })
        
        self.enviar_json({
            'combates': combates_dict,
            'total': len(combates_dict)
        })
    
    def obtener_combate(self, combate_id: int):
        """GET /api/combates/:id"""
        combate = repo_combates.obtener_por_id(combate_id)
        
        if not combate:
            self.enviar_error(404, f'Combate {combate_id} no encontrado')
            return
        
        self.enviar_json({
            'id': combate.id,
            'atacante': combate.atacante.to_dict(),
            'defensor': combate.defensor.to_dict(),
            'turno_actual': combate.turno_actual,
            'finalizado': combate.finalizado,
            'ganador': combate.ganador.to_dict() if combate.ganador else None,
            'historial': combate.historial
        })
    
    def crear_combate(self):
        """POST /api/combates"""
        datos = self.leer_json_body()
        
        if not datos:
            self.enviar_error(400, 'Datos requeridos')
            return
        
        # Obtener IDs
        atacante_id = datos.get('atacante_id')
        defensor_id = datos.get('defensor_id')
        
        if not atacante_id or not defensor_id:
            self.enviar_error(400, 'Campos atacante_id y defensor_id requeridos')
            return
        
        # Obtener personajes
        atacante = repo_personajes.obtener_por_id(atacante_id)
        defensor = repo_personajes.obtener_por_id(defensor_id)
        
        if not atacante:
            self.enviar_error(404, f'Atacante {atacante_id} no encontrado')
            return
        
        if not defensor:
            self.enviar_error(404, f'Defensor {defensor_id} no encontrado')
            return
        
        # Crear combate
        combate = Combate(atacante, defensor)
        combate = repo_combates.guardar(combate)
        
        # Responder
        self.send_response(201)
        self.send_header('Content-type', 'application/json')
        self.send_header('Location', f'/api/combates/{combate.id}')
        self.end_headers()
        
        respuesta = {
            'id': combate.id,
            'mensaje': f'Combate iniciado: {atacante.nombre} vs {defensor.nombre}',
            'atacante': atacante.to_dict(),
            'defensor': defensor.to_dict()
        }
        
        self.wfile.write(json.dumps(respuesta, ensure_ascii=False, indent=2).encode())
    
    def realizar_accion(self, combate_id: int):
        """POST /api/combates/:id/accion"""
        combate = repo_combates.obtener_por_id(combate_id)
        
        if not combate:
            self.enviar_error(404, f'Combate {combate_id} no encontrado')
            return
        
        if combate.finalizado:
            self.enviar_error(400, 'El combate ya finalizó')
            return
        
        datos = self.leer_json_body()
        if not datos:
            self.enviar_error(400, 'Datos requeridos')
            return
        
        accion = datos.get('accion', 'atacar').lower()
        
        # Ejecutar acción
        if accion == 'atacar':
            resultado = combate.ejecutar_turno()
        else:
            self.enviar_error(400, f'Acción inválida: {accion}')
            return
        
        # Responder
        self.enviar_json({
            'resultado': resultado,
            'combate': {
                'id': combate.id,
                'atacante': combate.atacante.to_dict(),
                'defensor': combate.defensor.to_dict(),
                'turno_actual': combate.turno_actual,
                'finalizado': combate.finalizado,
                'ganador': combate.ganador.to_dict() if combate.ganador else None
            }
        })
    
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
    servidor = HTTPServer(('localhost', puerto), RPGAPIHandler)
    
    print('='*60)
    print('🎮 API REST - Juego RPG')
    print('='*60)
    print(f'\n🚀 Servidor corriendo en http://localhost:{puerto}')
    print(f'📖 Documentación: http://localhost:{puerto}/')
    print('\n✨ Endpoints disponibles:')
    print('   GET    /api/info')
    print('   GET    /api/personajes')
    print('   POST   /api/personajes')
    print('   GET    /api/personajes/:id')
    print('   PUT    /api/personajes/:id/equipar')
    print('   DELETE /api/personajes/:id')
    print('   POST   /api/combates')
    print('   GET    /api/combates/:id')
    print('   POST   /api/combates/:id/accion')
    print('\n⛔ Ctrl+C para detener\n')
    
    try:
        servidor.serve_forever()
    except KeyboardInterrupt:
        print('\n\n⛔ Servidor detenido')
        servidor.shutdown()
```

---

## Pruebas con curl

### 1. Información del juego

```powershell
curl http://localhost:8000/api/info
```

**Response:**
```json
{
  "nombre": "Sistema RPG",
  "version": "1.0.0",
  "descripcion": "Sistema de combate por turnos",
  "clases_disponibles": ["guerrero", "mago", "arquero"],
  "estadisticas": {
    "personajes": 0,
    "combates": 0
  }
}
```

### 2. Crear personajes

```powershell
# Crear guerrero
curl -X POST http://localhost:8000/api/personajes `
  -H "Content-Type: application/json" `
  -d '{\"nombre\":\"Arthas\",\"clase\":\"guerrero\"}'

# Crear mago
curl -X POST http://localhost:8000/api/personajes `
  -H "Content-Type: application/json" `
  -d '{\"nombre\":\"Gandalf\",\"clase\":\"mago\"}'

# Crear arquero
curl -X POST http://localhost:8000/api/personajes `
  -H "Content-Type: application/json" `
  -d '{\"nombre\":\"Legolas\",\"clase\":\"arquero\"}'
```

### 3. Listar personajes

```powershell
curl http://localhost:8000/api/personajes
```

**Response:**
```json
{
  "personajes": [
    {
      "id": 1,
      "nombre": "Arthas",
      "clase": "Guerrero",
      "hp": 120,
      "hp_max": 120,
      "ataque": 18,
      "defensa": 8
    },
    {
      "id": 2,
      "nombre": "Gandalf",
      "clase": "Mago",
      "hp": 80,
      "hp_max": 80,
      "ataque": 25,
      "defensa": 3
    }
  ],
  "total": 2
}
```

### 4. Ver personaje específico

```powershell
curl http://localhost:8000/api/personajes/1
```

### 5. Equipar items

```powershell
# Equipar arma
curl -X PUT http://localhost:8000/api/personajes/1/equipar `
  -H "Content-Type: application/json" `
  -d '{\"tipo\":\"arma\",\"nombre\":\"Espada de Fuego\",\"daño\":15}'

# Equipar armadura
curl -X PUT http://localhost:8000/api/personajes/1/equipar `
  -H "Content-Type: application/json" `
  -d '{\"tipo\":\"armadura\",\"nombre\":\"Armadura de Placas\",\"defensa\":10}'
```

### 6. Crear combate

```powershell
curl -X POST http://localhost:8000/api/combates `
  -H "Content-Type: application/json" `
  -d '{\"atacante_id\":1,\"defensor_id\":2}'
```

**Response:**
```json
{
  "id": 1,
  "mensaje": "Combate iniciado: Arthas vs Gandalf",
  "atacante": {
    "id": 1,
    "nombre": "Arthas",
    "clase": "Guerrero",
    "hp": 120
  },
  "defensor": {
    "id": 2,
    "nombre": "Gandalf",
    "clase": "Mago",
    "hp": 80
  }
}
```

### 7. Realizar acción en combate

```powershell
curl -X POST http://localhost:8000/api/combates/1/accion `
  -H "Content-Type: application/json" `
  -d '{\"accion\":\"atacar\"}'
```

**Response:**
```json
{
  "resultado": "Arthas atacó a Gandalf por 33 de daño",
  "combate": {
    "id": 1,
    "atacante": {
      "id": 1,
      "nombre": "Arthas",
      "hp": 120
    },
    "defensor": {
      "id": 2,
      "nombre": "Gandalf",
      "hp": 47
    },
    "turno_actual": 2,
    "finalizado": false,
    "ganador": null
  }
}
```

### 8. Ver estado del combate

```powershell
curl http://localhost:8000/api/combates/1
```

### 9. Eliminar personaje

```powershell
curl -X DELETE http://localhost:8000/api/personajes/1
```

---

## Estructura completa del proyecto

```
juego_rpg/
├── app.py                      # Servidor HTTP con API REST
├── modelos/
│   ├── __init__.py
│   ├── personaje.py            # Clases Personaje, Guerrero, Mago, Arquero
│   └── items.py                # Clases Item, Arma, Armadura
├── sistemas/
│   ├── __init__.py
│   └── combate.py              # Sistema de combate
├── repositorios/
│   ├── __init__.py
│   └── repositorio_memoria.py  # Repositorios en memoria
└── tests/
    ├── test_personajes.py
    ├── test_combate.py
    └── test_api.py
```

---

## Testing con Python requests

**test_api.py**

```python
import requests
import json

BASE_URL = 'http://localhost:8000'

def test_crear_y_combatir():
    """Test completo: crear personajes, equipar, combatir"""
    
    print('🧪 Test: Crear y combatir\n')
    
    # 1. Crear guerrero
    print('1. Creando guerrero...')
    response = requests.post(
        f'{BASE_URL}/api/personajes',
        json={'nombre': 'Arthas', 'clase': 'guerrero'}
    )
    assert response.status_code == 201
    guerrero = response.json()
    print(f'   ✅ {guerrero["nombre"]} creado (ID: {guerrero["id"]})')
    
    # 2. Crear mago
    print('2. Creando mago...')
    response = requests.post(
        f'{BASE_URL}/api/personajes',
        json={'nombre': 'Gandalf', 'clase': 'mago'}
    )
    assert response.status_code == 201
    mago = response.json()
    print(f'   ✅ {mago["nombre"]} creado (ID: {mago["id"]})')
    
    # 3. Equipar arma al guerrero
    print('3. Equipando arma al guerrero...')
    response = requests.put(
        f'{BASE_URL}/api/personajes/{guerrero["id"]}/equipar',
        json={'tipo': 'arma', 'nombre': 'Espada Legendaria', 'daño': 20}
    )
    assert response.status_code == 200
    print(f'   ✅ Arma equipada')
    
    # 4. Iniciar combate
    print('4. Iniciando combate...')
    response = requests.post(
        f'{BASE_URL}/api/combates',
        json={
            'atacante_id': guerrero['id'],
            'defensor_id': mago['id']
        }
    )
    assert response.status_code == 201
    combate = response.json()
    print(f'   ✅ Combate iniciado (ID: {combate["id"]})')
    
    # 5. Ejecutar turnos hasta que termine
    print('5. Ejecutando combate...')
    turno = 1
    while True:
        response = requests.post(
            f'{BASE_URL}/api/combates/{combate["id"]}/accion',
            json={'accion': 'atacar'}
        )
        
        if response.status_code != 200:
            break
        
        resultado = response.json()
        print(f'   Turno {turno}: {resultado["resultado"]}')
        
        if resultado['combate']['finalizado']:
            ganador = resultado['combate']['ganador']
            print(f'\n   🏆 ¡{ganador["nombre"]} ganó el combate!')
            break
        
        turno += 1
    
    print('\n✅ Test completado exitosamente')

if __name__ == '__main__':
    try:
        test_crear_y_combatir()
    except requests.ConnectionError:
        print('❌ Error: No se pudo conectar al servidor')
        print('   Asegúrate de que el servidor esté corriendo en http://localhost:8000')
    except AssertionError as e:
        print(f'❌ Test falló: {e}')
```

**Ejecutar test:**

```powershell
# Terminal 1: Servidor
python app.py

# Terminal 2: Tests
pip install requests
python test_api.py
```

---

## Frontend con JavaScript

**frontend.html**

```html
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Juego RPG</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            max-width: 1200px;
            margin: 0 auto;
            padding: 20px;
            background: #f0f0f0;
        }
        
        h1 {
            color: #333;
            text-align: center;
        }
        
        .container {
            display: grid;
            grid-template-columns: 1fr 1fr;
            gap: 20px;
            margin-top: 20px;
        }
        
        .panel {
            background: white;
            padding: 20px;
            border-radius: 8px;
            box-shadow: 0 2px 4px rgba(0,0,0,0.1);
        }
        
        .personaje {
            background: #f9f9f9;
            padding: 15px;
            margin: 10px 0;
            border-radius: 5px;
            border-left: 4px solid #4CAF50;
        }
        
        .personaje h3 {
            margin: 0 0 10px 0;
            color: #333;
        }
        
        .stats {
            display: grid;
            grid-template-columns: repeat(2, 1fr);
            gap: 10px;
            margin-top: 10px;
        }
        
        .stat {
            background: white;
            padding: 8px;
            border-radius: 3px;
        }
        
        button {
            background: #4CAF50;
            color: white;
            border: none;
            padding: 10px 20px;
            border-radius: 5px;
            cursor: pointer;
            font-size: 16px;
            margin: 5px;
        }
        
        button:hover {
            background: #45a049;
        }
        
        button:disabled {
            background: #ccc;
            cursor: not-allowed;
        }
        
        .btn-danger {
            background: #f44336;
        }
        
        .btn-danger:hover {
            background: #da190b;
        }
        
        input, select {
            width: 100%;
            padding: 10px;
            margin: 10px 0;
            border: 1px solid #ddd;
            border-radius: 5px;
            box-sizing: border-box;
        }
        
        .combate-log {
            background: #333;
            color: #0f0;
            padding: 15px;
            border-radius: 5px;
            max-height: 400px;
            overflow-y: auto;
            font-family: 'Courier New', monospace;
        }
        
        .combate-log p {
            margin: 5px 0;
        }
    </style>
</head>
<body>
    <h1>🎮 Juego RPG</h1>
    
    <div class="container">
        <div class="panel">
            <h2>Crear Personaje</h2>
            <input type="text" id="nombre" placeholder="Nombre del personaje">
            <select id="clase">
                <option value="guerrero">Guerrero</option>
                <option value="mago">Mago</option>
                <option value="arquero">Arquero</option>
            </select>
            <button onclick="crearPersonaje()">Crear Personaje</button>
            
            <h3 style="margin-top: 30px;">Personajes</h3>
            <div id="personajes"></div>
        </div>
        
        <div class="panel">
            <h2>Combate</h2>
            <select id="atacante">
                <option value="">Seleccionar atacante</option>
            </select>
            <select id="defensor">
                <option value="">Seleccionar defensor</option>
            </select>
            <button onclick="iniciarCombate()">Iniciar Combate</button>
            <button onclick="atacar()" id="btnAtacar" disabled>Atacar</button>
            
            <div class="combate-log" id="log">
                <p>> Esperando combate...</p>
            </div>
        </div>
    </div>
    
    <script>
        const API_URL = 'http://localhost:8000';
        let combateActual = null;
        
        // Cargar personajes al iniciar
        cargarPersonajes();
        
        async function crearPersonaje() {
            const nombre = document.getElementById('nombre').value;
            const clase = document.getElementById('clase').value;
            
            if (!nombre) {
                alert('Ingresa un nombre');
                return;
            }
            
            try {
                const response = await fetch(`${API_URL}/api/personajes`, {
                    method: 'POST',
                    headers: {'Content-Type': 'application/json'},
                    body: JSON.stringify({nombre, clase})
                });
                
                if (response.ok) {
                    document.getElementById('nombre').value = '';
                    cargarPersonajes();
                    log(`✅ ${nombre} (${clase}) creado`);
                } else {
                    const error = await response.json();
                    alert(error.error);
                }
            } catch (error) {
                alert('Error al crear personaje');
            }
        }
        
        async function cargarPersonajes() {
            try {
                const response = await fetch(`${API_URL}/api/personajes`);
                const data = await response.json();
                
                // Mostrar en panel
                const container = document.getElementById('personajes');
                container.innerHTML = data.personajes.map(p => `
                    <div class="personaje">
                        <h3>${p.nombre} (${p.clase})</h3>
                        <div class="stats">
                            <div class="stat">❤️ HP: ${p.hp}/${p.hp_max}</div>
                            <div class="stat">⚔️ Ataque: ${p.ataque}</div>
                            <div class="stat">🛡️ Defensa: ${p.defensa}</div>
                            <div class="stat">ID: ${p.id}</div>
                        </div>
                    </div>
                `).join('');
                
                // Actualizar selects
                const atacanteSelect = document.getElementById('atacante');
                const defensorSelect = document.getElementById('defensor');
                
                const options = data.personajes.map(p => 
                    `<option value="${p.id}">${p.nombre} (${p.clase})</option>`
                ).join('');
                
                atacanteSelect.innerHTML = '<option value="">Seleccionar atacante</option>' + options;
                defensorSelect.innerHTML = '<option value="">Seleccionar defensor</option>' + options;
                
            } catch (error) {
                console.error('Error al cargar personajes:', error);
            }
        }
        
        async function iniciarCombate() {
            const atacanteId = parseInt(document.getElementById('atacante').value);
            const defensorId = parseInt(document.getElementById('defensor').value);
            
            if (!atacanteId || !defensorId) {
                alert('Selecciona ambos personajes');
                return;
            }
            
            if (atacanteId === defensorId) {
                alert('Deben ser personajes diferentes');
                return;
            }
            
            try {
                const response = await fetch(`${API_URL}/api/combates`, {
                    method: 'POST',
                    headers: {'Content-Type': 'application/json'},
                    body: JSON.stringify({atacante_id: atacanteId, defensor_id: defensorId})
                });
                
                const data = await response.json();
                combateActual = data.id;
                
                document.getElementById('log').innerHTML = '';
                log(`⚔️ Combate iniciado: ${data.atacante.nombre} vs ${data.defensor.nombre}`);
                log(`❤️ ${data.atacante.nombre}: ${data.atacante.hp} HP`);
                log(`❤️ ${data.defensor.nombre}: ${data.defensor.hp} HP`);
                
                document.getElementById('btnAtacar').disabled = false;
                
            } catch (error) {
                alert('Error al iniciar combate');
            }
        }
        
        async function atacar() {
            if (!combateActual) return;
            
            try {
                const response = await fetch(`${API_URL}/api/combates/${combateActual}/accion`, {
                    method: 'POST',
                    headers: {'Content-Type': 'application/json'},
                    body: JSON.stringify({accion: 'atacar'})
                });
                
                const data = await response.json();
                
                log(`> ${data.resultado}`);
                log(`  ${data.combate.atacante.nombre}: ${data.combate.atacante.hp} HP`);
                log(`  ${data.combate.defensor.nombre}: ${data.combate.defensor.hp} HP`);
                
                if (data.combate.finalizado) {
                    log(`\n🏆 ¡${data.combate.ganador.nombre} ganó el combate!`);
                    document.getElementById('btnAtacar').disabled = true;
                    combateActual = null;
                    
                    // Recargar personajes
                    setTimeout(cargarPersonajes, 1000);
                }
                
            } catch (error) {
                console.error('Error al atacar:', error);
            }
        }
        
        function log(mensaje) {
            const logDiv = document.getElementById('log');
            const p = document.createElement('p');
            p.textContent = mensaje;
            logDiv.appendChild(p);
            logDiv.scrollTop = logDiv.scrollHeight;
        }
    </script>
</body>
</html>
```

---

## Resumen

Has aprendido:

✅ Crear API REST completa con Python vanilla  
✅ Repositorios en memoria para persistencia  
✅ CRUD de personajes con endpoints REST  
✅ Sistema de combate por turnos vía API  
✅ Testing con curl y Python requests  
✅ Frontend con JavaScript que consume la API

**Siguiente:** [Bloque 5 - Autenticación](../bloque-05-autenticacion/)

---

## Recursos

- **[http.server](https://docs.python.org/3/library/http.server.html)** - HTTP server Python
- **[requests](https://requests.readthedocs.io/)** - HTTP library para testing

Tu juego RPG ahora es una API REST completa. 🎮

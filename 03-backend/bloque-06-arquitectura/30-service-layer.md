# Service Layer Pattern

> **Encapsula lógica de negocio en servicios independientes y reutilizables**

---

## ¿Qué es Service Layer?

**Service Layer** es un patrón que centraliza la lógica de negocio en clases de servicio, separándola de:
- Controllers (HTTP)
- Repositories (persistencia)
- Models (datos)

**Beneficios:**
- ✅ Lógica reutilizable
- ✅ Fácil de testear
- ✅ Transacciones coordinadas
- ✅ Reglas de negocio centralizadas

---

## Flujo con Service Layer

```
Controller → Service → Repository → Database
            ↓
        (Lógica negocio)
        (Validaciones)
        (Coordinación)
```

---

## Sin Service Layer (lógica en controller)

**controllers/pedido_controller.py**

```python
"""
Controller con lógica mezclada (sin service)
"""

from http.server import BaseHTTPRequestHandler
import json

class PedidoController:
    """Controller con demasiada responsabilidad"""
    
    def crear_pedido(self, handler: BaseHTTPRequestHandler):
        """POST /api/pedidos - Lógica mezclada"""
        try:
            datos = self._leer_json(handler)
            
            # ❌ Validación en controller
            if not datos.get('cliente_id'):
                self._enviar_error(handler, 400, 'Cliente requerido')
                return
            
            # ❌ Lógica de negocio en controller
            cliente = clientes_db.get(datos['cliente_id'])
            if not cliente:
                self._enviar_error(handler, 404, 'Cliente no existe')
                return
            
            # ❌ Cálculos en controller
            total = 0
            for item in datos.get('items', []):
                producto = productos_db.get(item['producto_id'])
                if producto:
                    total += producto['precio'] * item['cantidad']
            
            # ❌ Reglas de negocio en controller
            if total < 10:
                self._enviar_error(handler, 400, 'Pedido mínimo $10')
                return
            
            # ❌ Persistencia en controller
            pedido = {
                'id': len(pedidos_db) + 1,
                'cliente_id': datos['cliente_id'],
                'items': datos['items'],
                'total': total,
                'estado': 'pendiente'
            }
            pedidos_db[pedido['id']] = pedido
            
            # ❌ Lógica adicional en controller
            if cliente.get('puntos'):
                cliente['puntos'] += int(total * 0.1)
            
            self._enviar_json(handler, pedido, 201)
            
        except Exception as e:
            self._enviar_error(handler, 500, str(e))
```

**Problemas:**
- ❌ Controller muy complejo
- ❌ Lógica no reutilizable
- ❌ Difícil de testear
- ❌ Reglas de negocio dispersas

---

## Con Service Layer: Models

**models/pedido.py**

```python
"""
Models para el dominio de pedidos
"""

from dataclasses import dataclass, field
from typing import List
from datetime import datetime
from enum import Enum

class EstadoPedido(Enum):
    """Estados posibles de un pedido"""
    PENDIENTE = 'pendiente'
    CONFIRMADO = 'confirmado'
    ENVIADO = 'enviado'
    ENTREGADO = 'entregado'
    CANCELADO = 'cancelado'

@dataclass
class ItemPedido:
    """Item individual en un pedido"""
    producto_id: int
    producto_nombre: str
    cantidad: int
    precio_unitario: float
    
    def calcular_subtotal(self) -> float:
        """Calcula subtotal del item"""
        return self.cantidad * self.precio_unitario
    
    def to_dict(self) -> dict:
        return {
            'producto_id': self.producto_id,
            'producto_nombre': self.producto_nombre,
            'cantidad': self.cantidad,
            'precio_unitario': self.precio_unitario,
            'subtotal': self.calcular_subtotal()
        }

@dataclass
class Pedido:
    """Pedido de cliente"""
    id: int
    cliente_id: int
    items: List[ItemPedido] = field(default_factory=list)
    estado: EstadoPedido = EstadoPedido.PENDIENTE
    creado_en: datetime = field(default_factory=datetime.now)
    
    def calcular_total(self) -> float:
        """Calcula total del pedido"""
        return sum(item.calcular_subtotal() for item in self.items)
    
    def agregar_item(self, item: ItemPedido):
        """Agrega un item al pedido"""
        self.items.append(item)
    
    def confirmar(self):
        """Confirma el pedido"""
        if self.estado != EstadoPedido.PENDIENTE:
            raise ValueError('Solo se pueden confirmar pedidos pendientes')
        self.estado = EstadoPedido.CONFIRMADO
    
    def cancelar(self):
        """Cancela el pedido"""
        if self.estado in [EstadoPedido.ENVIADO, EstadoPedido.ENTREGADO]:
            raise ValueError('No se puede cancelar pedido enviado o entregado')
        self.estado = EstadoPedido.CANCELADO
    
    def to_dict(self) -> dict:
        return {
            'id': self.id,
            'cliente_id': self.cliente_id,
            'items': [item.to_dict() for item in self.items],
            'total': self.calcular_total(),
            'estado': self.estado.value,
            'creado_en': self.creado_en.isoformat()
        }

@dataclass
class Cliente:
    """Cliente del sistema"""
    id: int
    nombre: str
    email: str
    puntos: int = 0
    activo: bool = True
    
    def agregar_puntos(self, puntos: int):
        """Agrega puntos al cliente"""
        self.puntos += puntos
    
    def to_dict(self) -> dict:
        return {
            'id': self.id,
            'nombre': self.nombre,
            'email': self.email,
            'puntos': self.puntos,
            'activo': self.activo
        }

@dataclass
class Producto:
    """Producto disponible"""
    id: int
    nombre: str
    precio: float
    stock: int
    activo: bool = True
    
    def tiene_stock(self, cantidad: int) -> bool:
        """Verifica si hay stock disponible"""
        return self.stock >= cantidad
    
    def reducir_stock(self, cantidad: int):
        """Reduce el stock"""
        if not self.tiene_stock(cantidad):
            raise ValueError(f'Stock insuficiente de {self.nombre}')
        self.stock -= cantidad
    
    def to_dict(self) -> dict:
        return {
            'id': self.id,
            'nombre': self.nombre,
            'precio': self.precio,
            'stock': self.stock,
            'activo': self.activo
        }
```

---

## Con Service Layer: Repositories

**repositories/pedido_repository.py**

```python
"""
Repositories para persistencia
"""

from typing import Dict, List, Optional
from models.pedido import Pedido, Cliente, Producto

class PedidoRepository:
    """Repositorio de pedidos"""
    
    def __init__(self):
        self._pedidos: Dict[int, Pedido] = {}
        self._siguiente_id = 1
    
    def crear(self, pedido: Pedido) -> Pedido:
        """Crea un nuevo pedido"""
        pedido.id = self._siguiente_id
        self._siguiente_id += 1
        self._pedidos[pedido.id] = pedido
        return pedido
    
    def obtener_por_id(self, id_pedido: int) -> Optional[Pedido]:
        """Obtiene pedido por ID"""
        return self._pedidos.get(id_pedido)
    
    def obtener_por_cliente(self, cliente_id: int) -> List[Pedido]:
        """Obtiene pedidos de un cliente"""
        return [
            p for p in self._pedidos.values()
            if p.cliente_id == cliente_id
        ]
    
    def actualizar(self, pedido: Pedido) -> Pedido:
        """Actualiza un pedido"""
        if pedido.id not in self._pedidos:
            raise ValueError(f'Pedido {pedido.id} no existe')
        self._pedidos[pedido.id] = pedido
        return pedido

class ClienteRepository:
    """Repositorio de clientes"""
    
    def __init__(self):
        self._clientes: Dict[int, Cliente] = {}
        self._siguiente_id = 1
    
    def crear(self, cliente: Cliente) -> Cliente:
        """Crea un nuevo cliente"""
        cliente.id = self._siguiente_id
        self._siguiente_id += 1
        self._clientes[cliente.id] = cliente
        return cliente
    
    def obtener_por_id(self, id_cliente: int) -> Optional[Cliente]:
        """Obtiene cliente por ID"""
        return self._clientes.get(id_cliente)
    
    def actualizar(self, cliente: Cliente) -> Cliente:
        """Actualiza un cliente"""
        if cliente.id not in self._clientes:
            raise ValueError(f'Cliente {cliente.id} no existe')
        self._clientes[cliente.id] = cliente
        return cliente

class ProductoRepository:
    """Repositorio de productos"""
    
    def __init__(self):
        self._productos: Dict[int, Producto] = {}
        self._siguiente_id = 1
    
    def crear(self, producto: Producto) -> Producto:
        """Crea un nuevo producto"""
        producto.id = self._siguiente_id
        self._siguiente_id += 1
        self._productos[producto.id] = producto
        return producto
    
    def obtener_por_id(self, id_producto: int) -> Optional[Producto]:
        """Obtiene producto por ID"""
        return self._productos.get(id_producto)
    
    def actualizar(self, producto: Producto) -> Producto:
        """Actualiza un producto"""
        if producto.id not in self._productos:
            raise ValueError(f'Producto {producto.id} no existe')
        self._productos[producto.id] = producto
        return producto

# Instancias globales
pedido_repository = PedidoRepository()
cliente_repository = ClienteRepository()
producto_repository = ProductoRepository()
```

---

## Con Service Layer: Service

**services/pedido_service.py**

```python
"""
Service Layer: Lógica de negocio de pedidos
"""

from typing import List
from models.pedido import Pedido, ItemPedido, EstadoPedido, Cliente, Producto
from repositories.pedido_repository import (
    PedidoRepository,
    ClienteRepository,
    ProductoRepository
)

class PedidoService:
    """Servicio de lógica de negocio para pedidos"""
    
    # Constantes de negocio
    PEDIDO_MINIMO = 10.0
    PUNTOS_POR_DOLAR = 0.1
    
    def __init__(
        self,
        pedido_repo: PedidoRepository,
        cliente_repo: ClienteRepository,
        producto_repo: ProductoRepository
    ):
        self.pedido_repo = pedido_repo
        self.cliente_repo = cliente_repo
        self.producto_repo = producto_repo
    
    def crear_pedido(self, cliente_id: int, items_data: List[dict]) -> Pedido:
        """
        Crea un nuevo pedido con todas las validaciones de negocio
        
        Args:
            cliente_id: ID del cliente
            items_data: Lista de dicts con producto_id y cantidad
        
        Returns:
            Pedido creado
        
        Raises:
            ValueError: Si hay errores de validación o negocio
        """
        # 1. Validar cliente
        cliente = self._validar_cliente(cliente_id)
        
        # 2. Crear items y validar stock
        items = self._crear_items(items_data)
        
        # 3. Crear pedido
        pedido = Pedido(
            id=0,  # Se asignará en repository
            cliente_id=cliente_id,
            items=items
        )
        
        # 4. Validar total mínimo
        total = pedido.calcular_total()
        if total < self.PEDIDO_MINIMO:
            raise ValueError(
                f'Pedido mínimo ${self.PEDIDO_MINIMO}. Total: ${total:.2f}'
            )
        
        # 5. Reducir stock de productos
        self._reducir_stock(items)
        
        # 6. Guardar pedido
        pedido = self.pedido_repo.crear(pedido)
        
        # 7. Agregar puntos al cliente
        self._agregar_puntos_cliente(cliente, total)
        
        return pedido
    
    def obtener_pedido(self, id_pedido: int) -> Pedido:
        """
        Obtiene un pedido por ID
        
        Raises:
            ValueError: Si no existe
        """
        pedido = self.pedido_repo.obtener_por_id(id_pedido)
        if not pedido:
            raise ValueError(f'Pedido {id_pedido} no encontrado')
        return pedido
    
    def listar_pedidos_cliente(self, cliente_id: int) -> List[Pedido]:
        """Lista pedidos de un cliente"""
        # Validar que cliente existe
        self._validar_cliente(cliente_id)
        return self.pedido_repo.obtener_por_cliente(cliente_id)
    
    def confirmar_pedido(self, id_pedido: int) -> Pedido:
        """
        Confirma un pedido pendiente
        
        Raises:
            ValueError: Si no se puede confirmar
        """
        pedido = self.obtener_pedido(id_pedido)
        pedido.confirmar()
        return self.pedido_repo.actualizar(pedido)
    
    def cancelar_pedido(self, id_pedido: int) -> Pedido:
        """
        Cancela un pedido
        
        Raises:
            ValueError: Si no se puede cancelar
        """
        pedido = self.obtener_pedido(id_pedido)
        
        # Devolver stock si se cancela
        if pedido.estado == EstadoPedido.CONFIRMADO:
            self._devolver_stock(pedido.items)
        
        pedido.cancelar()
        return self.pedido_repo.actualizar(pedido)
    
    def _validar_cliente(self, cliente_id: int) -> Cliente:
        """Valida que cliente existe y está activo"""
        cliente = self.cliente_repo.obtener_por_id(cliente_id)
        if not cliente:
            raise ValueError(f'Cliente {cliente_id} no encontrado')
        if not cliente.activo:
            raise ValueError('Cliente inactivo')
        return cliente
    
    def _crear_items(self, items_data: List[dict]) -> List[ItemPedido]:
        """Crea items validando productos y stock"""
        if not items_data:
            raise ValueError('Pedido debe tener al menos un item')
        
        items = []
        for item_data in items_data:
            producto_id = item_data.get('producto_id')
            cantidad = item_data.get('cantidad', 0)
            
            if not producto_id or cantidad <= 0:
                raise ValueError('producto_id y cantidad son requeridos')
            
            # Obtener producto
            producto = self.producto_repo.obtener_por_id(producto_id)
            if not producto:
                raise ValueError(f'Producto {producto_id} no encontrado')
            
            if not producto.activo:
                raise ValueError(f'Producto {producto.nombre} no disponible')
            
            # Verificar stock
            if not producto.tiene_stock(cantidad):
                raise ValueError(
                    f'Stock insuficiente de {producto.nombre}. '
                    f'Disponible: {producto.stock}'
                )
            
            # Crear item
            item = ItemPedido(
                producto_id=producto.id,
                producto_nombre=producto.nombre,
                cantidad=cantidad,
                precio_unitario=producto.precio
            )
            items.append(item)
        
        return items
    
    def _reducir_stock(self, items: List[ItemPedido]):
        """Reduce stock de productos"""
        for item in items:
            producto = self.producto_repo.obtener_por_id(item.producto_id)
            producto.reducir_stock(item.cantidad)
            self.producto_repo.actualizar(producto)
    
    def _devolver_stock(self, items: List[ItemPedido]):
        """Devuelve stock de productos"""
        for item in items:
            producto = self.producto_repo.obtener_por_id(item.producto_id)
            producto.stock += item.cantidad
            self.producto_repo.actualizar(producto)
    
    def _agregar_puntos_cliente(self, cliente: Cliente, total: float):
        """Agrega puntos de lealtad al cliente"""
        puntos = int(total * self.PUNTOS_POR_DOLAR)
        if puntos > 0:
            cliente.agregar_puntos(puntos)
            self.cliente_repo.actualizar(cliente)

# Instancia global con dependencias
from repositories.pedido_repository import (
    pedido_repository,
    cliente_repository,
    producto_repository
)

pedido_service = PedidoService(
    pedido_repository,
    cliente_repository,
    producto_repository
)
```

---

## Controller simplificado

**controllers/pedido_controller.py**

```python
"""
Controller simplificado: solo maneja HTTP
"""

from http.server import BaseHTTPRequestHandler
from urllib.parse import urlparse
import json
from typing import Optional

from services.pedido_service import PedidoService

class PedidoController:
    """Controller que delega toda la lógica al service"""
    
    def __init__(self, service: PedidoService):
        self.service = service
    
    def manejar_request(self, handler: BaseHTTPRequestHandler):
        """Enruta la solicitud"""
        ruta = urlparse(handler.path)
        path = ruta.path
        metodo = handler.command
        
        if path == '/api/pedidos':
            if metodo == 'POST':
                self.crear(handler)
        elif path.startswith('/api/pedidos/'):
            partes = path.split('/')
            if len(partes) >= 4 and partes[3].isdigit():
                id_pedido = int(partes[3])
                
                if metodo == 'GET':
                    self.obtener(handler, id_pedido)
                elif len(partes) == 5:
                    if partes[4] == 'confirmar' and metodo == 'PUT':
                        self.confirmar(handler, id_pedido)
                    elif partes[4] == 'cancelar' and metodo == 'PUT':
                        self.cancelar(handler, id_pedido)
        else:
            self._enviar_error(handler, 404, 'Ruta no encontrada')
    
    def crear(self, handler: BaseHTTPRequestHandler):
        """POST /api/pedidos"""
        try:
            datos = self._leer_json(handler)
            if not datos:
                self._enviar_error(handler, 400, 'Datos requeridos')
                return
            
            # ✅ Toda la lógica en el service
            pedido = self.service.crear_pedido(
                cliente_id=datos.get('cliente_id'),
                items_data=datos.get('items', [])
            )
            
            self._enviar_json(handler, pedido.to_dict(), status=201)
        
        except ValueError as e:
            self._enviar_error(handler, 400, str(e))
        except Exception as e:
            self._enviar_error(handler, 500, str(e))
    
    def obtener(self, handler: BaseHTTPRequestHandler, id_pedido: int):
        """GET /api/pedidos/:id"""
        try:
            pedido = self.service.obtener_pedido(id_pedido)
            self._enviar_json(handler, pedido.to_dict())
        except ValueError as e:
            self._enviar_error(handler, 404, str(e))
        except Exception as e:
            self._enviar_error(handler, 500, str(e))
    
    def confirmar(self, handler: BaseHTTPRequestHandler, id_pedido: int):
        """PUT /api/pedidos/:id/confirmar"""
        try:
            pedido = self.service.confirmar_pedido(id_pedido)
            self._enviar_json(handler, pedido.to_dict())
        except ValueError as e:
            self._enviar_error(handler, 400, str(e))
        except Exception as e:
            self._enviar_error(handler, 500, str(e))
    
    def cancelar(self, handler: BaseHTTPRequestHandler, id_pedido: int):
        """PUT /api/pedidos/:id/cancelar"""
        try:
            pedido = self.service.cancelar_pedido(id_pedido)
            self._enviar_json(handler, pedido.to_dict())
        except ValueError as e:
            self._enviar_error(handler, 400, str(e))
        except Exception as e:
            self._enviar_error(handler, 500, str(e))
    
    @staticmethod
    def _leer_json(handler: BaseHTTPRequestHandler) -> Optional[dict]:
        """Lee JSON del body"""
        content_length = int(handler.headers.get('Content-Length', 0))
        if content_length == 0:
            return None
        body = handler.rfile.read(content_length)
        try:
            return json.loads(body.decode('utf-8'))
        except json.JSONDecodeError:
            return None
    
    @staticmethod
    def _enviar_json(handler: BaseHTTPRequestHandler, datos: dict, status: int = 200):
        """Envía JSON"""
        handler.send_response(status)
        handler.send_header('Content-type', 'application/json')
        handler.end_headers()
        handler.wfile.write(json.dumps(datos, ensure_ascii=False, indent=2).encode())
    
    @staticmethod
    def _enviar_error(handler: BaseHTTPRequestHandler, codigo: int, mensaje: str):
        """Envía error"""
        handler.send_response(codigo)
        handler.send_header('Content-type', 'application/json')
        handler.end_headers()
        error = {'error': mensaje, 'codigo': codigo}
        handler.wfile.write(json.dumps(error, ensure_ascii=False).encode())
```

---

## Aplicación completa

**app_service.py**

```python
"""
Aplicación con Service Layer
"""

from http.server import BaseHTTPRequestHandler, HTTPServer
from urllib.parse import urlparse

from services.pedido_service import pedido_service
from controllers.pedido_controller import PedidoController

# Crear controller con service
pedido_controller = PedidoController(pedido_service)

class ServiceLayerHandler(BaseHTTPRequestHandler):
    """Handler principal"""
    
    def do_GET(self):
        self._manejar()
    
    def do_POST(self):
        self._manejar()
    
    def do_PUT(self):
        self._manejar()
    
    def _manejar(self):
        ruta = urlparse(self.path).path
        
        if ruta == '/':
            self.home()
        elif ruta.startswith('/api/pedidos'):
            pedido_controller.manejar_request(self)
        else:
            self.send_response(404)
            self.end_headers()
    
    def home(self):
        """GET /"""
        html = """
        <!DOCTYPE html>
        <html>
        <head>
            <title>Service Layer</title>
        </head>
        <body>
            <h1>🎯 Service Layer Pattern</h1>
            <h2>Endpoints:</h2>
            <ul>
                <li>POST /api/pedidos - Crear pedido</li>
                <li>GET /api/pedidos/:id - Ver pedido</li>
                <li>PUT /api/pedidos/:id/confirmar - Confirmar pedido</li>
                <li>PUT /api/pedidos/:id/cancelar - Cancelar pedido</li>
            </ul>
            
            <h2>Reglas de negocio (en Service):</h2>
            <ul>
                <li>✅ Pedido mínimo $10</li>
                <li>✅ Validación de stock</li>
                <li>✅ Reducción automática de stock</li>
                <li>✅ Puntos de lealtad (10% del total)</li>
                <li>✅ Devolución de stock al cancelar</li>
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
    # Datos de prueba
    from models.pedido import Cliente, Producto
    from repositories.pedido_repository import cliente_repository, producto_repository
    
    # Crear cliente de prueba
    cliente = Cliente(
        id=0,
        nombre='Ana García',
        email='ana@example.com'
    )
    cliente_repository.crear(cliente)
    
    # Crear productos de prueba
    producto1 = Producto(
        id=0,
        nombre='Laptop',
        precio=1000.0,
        stock=10
    )
    producto_repository.crear(producto1)
    
    producto2 = Producto(
        id=0,
        nombre='Mouse',
        precio=25.0,
        stock=50
    )
    producto_repository.crear(producto2)
    
    puerto = 8000
    servidor = HTTPServer(('localhost', puerto), ServiceLayerHandler)
    
    print('='*60)
    print('🎯 Service Layer Pattern')
    print('='*60)
    print(f'\n🚀 Servidor: http://localhost:{puerto}')
    print('\n📦 Datos de prueba:')
    print(f'   Cliente ID: 1 (Ana García)')
    print(f'   Producto ID: 1 (Laptop - $1000)')
    print(f'   Producto ID: 2 (Mouse - $25)')
    print('\n⛔ Ctrl+C para detener\n')
    
    try:
        servidor.serve_forever()
    except KeyboardInterrupt:
        print('\n\n⛔ Servidor detenido')
        servidor.shutdown()
```

---

## Pruebas con curl

```bash
# Crear pedido
curl -X POST http://localhost:8000/api/pedidos \
  -H "Content-Type: application/json" \
  -d '{
    "cliente_id": 1,
    "items": [
      {"producto_id": 1, "cantidad": 1},
      {"producto_id": 2, "cantidad": 2}
    ]
  }'

# Respuesta:
# {
#   "id": 1,
#   "cliente_id": 1,
#   "items": [
#     {
#       "producto_id": 1,
#       "producto_nombre": "Laptop",
#       "cantidad": 1,
#       "precio_unitario": 1000.0,
#       "subtotal": 1000.0
#     },
#     {
#       "producto_id": 2,
#       "producto_nombre": "Mouse",
#       "cantidad": 2,
#       "precio_unitario": 25.0,
#       "subtotal": 50.0
#     }
#   ],
#   "total": 1050.0,
#   "estado": "pendiente",
#   "creado_en": "2024-01-20T10:30:00"
# }

# Ver pedido
curl http://localhost:8000/api/pedidos/1

# Confirmar pedido
curl -X PUT http://localhost:8000/api/pedidos/1/confirmar

# Cancelar pedido
curl -X PUT http://localhost:8000/api/pedidos/1/cancelar

# Intentar pedido menor a $10 (error)
curl -X POST http://localhost:8000/api/pedidos \
  -H "Content-Type: application/json" \
  -d '{
    "cliente_id": 1,
    "items": [
      {"producto_id": 2, "cantidad": 1}
    ]
  }'

# Respuesta:
# {
#   "error": "Pedido mínimo $10.0. Total: $25.00",
#   "codigo": 400
# }
```

---

## Testing del Service

**tests/test_pedido_service.py**

```python
"""
Test del service (sin HTTP, sin DB real)
"""

import pytest
from models.pedido import Cliente, Producto, EstadoPedido
from repositories.pedido_repository import (
    PedidoRepository,
    ClienteRepository,
    ProductoRepository
)
from services.pedido_service import PedidoService

@pytest.fixture
def service():
    """Service con repositorios limpios"""
    pedido_repo = PedidoRepository()
    cliente_repo = ClienteRepository()
    producto_repo = ProductoRepository()
    
    # Crear datos de prueba
    cliente = Cliente(id=0, nombre='Ana', email='ana@test.com')
    cliente_repo.crear(cliente)
    
    producto = Producto(id=0, nombre='Test', precio=50.0, stock=10)
    producto_repo.crear(producto)
    
    return PedidoService(pedido_repo, cliente_repo, producto_repo)

def test_crear_pedido_valido(service):
    """Crea pedido válido"""
    items_data = [{'producto_id': 1, 'cantidad': 2}]
    
    pedido = service.crear_pedido(cliente_id=1, items_data=items_data)
    
    assert pedido.id == 1
    assert pedido.cliente_id == 1
    assert len(pedido.items) == 1
    assert pedido.calcular_total() == 100.0
    assert pedido.estado == EstadoPedido.PENDIENTE

def test_pedido_minimo(service):
    """Rechaza pedido menor al mínimo"""
    items_data = [{'producto_id': 1, 'cantidad': 1}]
    
    # Cambiar precio del producto a $5 (menor al mínimo de $10)
    from repositories.pedido_repository import producto_repository
    producto = producto_repository.obtener_por_id(1)
    producto.precio = 5.0
    producto_repository.actualizar(producto)
    
    with pytest.raises(ValueError, match='Pedido mínimo'):
        service.crear_pedido(cliente_id=1, items_data=items_data)

def test_stock_insuficiente(service):
    """Rechaza pedido sin stock"""
    items_data = [{'producto_id': 1, 'cantidad': 20}]
    
    with pytest.raises(ValueError, match='Stock insuficiente'):
        service.crear_pedido(cliente_id=1, items_data=items_data)

def test_confirmar_pedido(service):
    """Confirma pedido pendiente"""
    items_data = [{'producto_id': 1, 'cantidad': 2}]
    pedido = service.crear_pedido(cliente_id=1, items_data=items_data)
    
    pedido_confirmado = service.confirmar_pedido(pedido.id)
    
    assert pedido_confirmado.estado == EstadoPedido.CONFIRMADO

def test_cancelar_devuelve_stock(service):
    """Cancelar pedido devuelve stock"""
    from repositories.pedido_repository import producto_repository
    
    items_data = [{'producto_id': 1, 'cantidad': 2}]
    pedido = service.crear_pedido(cliente_id=1, items_data=items_data)
    
    # Verificar que se redujo el stock
    producto = producto_repository.obtener_por_id(1)
    assert producto.stock == 8  # 10 - 2
    
    # Confirmar y cancelar
    service.confirmar_pedido(pedido.id)
    service.cancelar_pedido(pedido.id)
    
    # Verificar que se devolvió el stock
    producto = producto_repository.obtener_por_id(1)
    assert producto.stock == 10
```

---

## Ventajas del Service Layer

✅ **Lógica centralizada**
- Reglas de negocio en un solo lugar
- Fácil de encontrar y modificar

✅ **Reutilizable**
- Mismo service desde API, CLI, jobs, tests
- No depende de HTTP

✅ **Testeable**
- Tests sin HTTP ni base de datos
- Tests rápidos y confiables

✅ **Coordinación**
- Orquesta múltiples repositories
- Maneja transacciones lógicas

✅ **Mantenible**
- Cambios en lógica no afectan controller
- Controller más simple y limpio

---

## Comparación sin/con Service Layer

| Aspecto | Sin Service | Con Service |
|---------|-------------|-------------|
| **Lógica** | En controller | En service |
| **Reutilizar** | Solo desde HTTP | Desde cualquier lugar |
| **Testear** | Requiere HTTP mock | Test directo del service |
| **Complejidad controller** | Alta | Baja |
| **Reglas negocio** | Dispersas | Centralizadas |

---

## Resumen

Has aprendido:

✅ Separar lógica de negocio en services  
✅ Controller solo maneja HTTP  
✅ Service coordina repositories  
✅ Reglas de negocio centralizadas  
✅ Testing sin dependencias externas

**Siguiente:** Repository Pattern

---

## Recursos

- **[Service Layer Pattern](https://martinfowler.com/eaaCatalog/serviceLayer.html)** - Martin Fowler
- **[Domain Logic Patterns](https://martinfowler.com/eaaCatalog/)** - Catálogo de patrones

Lógica de negocio limpia y reutilizable. 🎯

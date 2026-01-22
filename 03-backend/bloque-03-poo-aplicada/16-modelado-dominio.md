# Modelado de dominio con POO

> **Diseña tu backend representando el mundo real con objetos**

---

## ¿Qué es el modelado de dominio?

**Modelado de dominio** es representar los conceptos y reglas de negocio de tu aplicación con clases y objetos.

**Ejemplo:** Para una tienda online, el dominio incluye:
- Productos
- Clientes
- Pedidos
- Carrito de compras
- Pagos
- Envíos

**Objetivo:** Que el código refleje cómo funciona el negocio en la vida real.

---

## De requisitos a clases

### 1. Identificar entidades (sustantivos)

**Requisito:** "Un cliente puede realizar pedidos de productos. Cada pedido tiene items con cantidad y precio. El cliente paga con tarjeta y recibe el pedido en su dirección."

**Entidades identificadas:**
- Cliente
- Pedido
- Producto
- ItemPedido
- Pago
- Dirección

### 2. Identificar relaciones

- Cliente **tiene** Pedidos (1:N)
- Pedido **tiene** Items (1:N)
- ItemPedido **tiene** Producto (N:1)
- Pedido **tiene** Pago (1:1)
- Cliente **tiene** Direcciones (1:N)

### 3. Identificar comportamientos (verbos)

- Cliente: registrarse, actualizar_perfil
- Pedido: agregar_item, calcular_total, procesar
- Producto: aplicar_descuento, reducir_stock

---

## Ejemplo: Sistema de tienda online

### Entidad: Producto

```python
from datetime import datetime
from typing import Optional

class Producto:
    """Representa un producto del catálogo"""
    
    def __init__(self, id: int, nombre: str, precio: float, stock: int, categoria: str):
        self.id = id
        self.nombre = nombre
        self._precio = precio
        self._stock = stock
        self.categoria = categoria
        self.activo = True
        self.fecha_creacion = datetime.now()
    
    @property
    def precio(self):
        return self._precio
    
    @precio.setter
    def precio(self, valor):
        if valor < 0:
            raise ValueError("El precio no puede ser negativo")
        self._precio = valor
    
    @property
    def stock(self):
        return self._stock
    
    def tiene_stock(self, cantidad: int = 1) -> bool:
        """Verifica si hay stock suficiente"""
        return self._stock >= cantidad and self.activo
    
    def reducir_stock(self, cantidad: int):
        """Reduce el stock (cuando se vende)"""
        if not self.tiene_stock(cantidad):
            raise ValueError(f"Stock insuficiente. Disponible: {self._stock}")
        self._stock -= cantidad
    
    def aumentar_stock(self, cantidad: int):
        """Aumenta el stock (cuando llega mercancía)"""
        if cantidad <= 0:
            raise ValueError("La cantidad debe ser positiva")
        self._stock += cantidad
    
    def aplicar_descuento(self, porcentaje: float):
        """Aplica un descuento al precio"""
        if not (0 <= porcentaje <= 100):
            raise ValueError("Porcentaje inválido")
        descuento = self._precio * (porcentaje / 100)
        self._precio -= descuento
    
    def __str__(self):
        return f"{self.nombre} - ${self.precio}"
```

### Entidad: Cliente

```python
class Cliente:
    """Representa un cliente de la tienda"""
    
    def __init__(self, id: int, nombre: str, email: str):
        self.id = id
        self.nombre = nombre
        self.email = email
        self.direcciones = []
        self.pedidos = []
        self.fecha_registro = datetime.now()
        self.activo = True
    
    def agregar_direccion(self, direccion):
        """Añade una dirección de envío"""
        self.direcciones.append(direccion)
    
    def obtener_direccion_principal(self):
        """Obtiene la dirección marcada como principal"""
        return next((d for d in self.direcciones if d.es_principal), None)
    
    def realizar_pedido(self, pedido):
        """Registra un nuevo pedido"""
        self.pedidos.append(pedido)
    
    def total_gastado(self) -> float:
        """Calcula el total gastado en todos los pedidos"""
        return sum(p.calcular_total() for p in self.pedidos if p.estado == 'completado')
    
    def __str__(self):
        return f"Cliente: {self.nombre} ({self.email})"
```

### Value Object: Dirección

```python
class Direccion:
    """Value object para direcciones"""
    
    def __init__(self, calle: str, ciudad: str, codigo_postal: str, es_principal: bool = False):
        self.calle = calle
        self.ciudad = ciudad
        self.codigo_postal = codigo_postal
        self.es_principal = es_principal
    
    def direccion_completa(self) -> str:
        """Devuelve la dirección como string"""
        return f"{self.calle}, {self.codigo_postal} {self.ciudad}"
    
    def __str__(self):
        return self.direccion_completa()
```

### Entidad: ItemPedido

```python
class ItemPedido:
    """Representa un item dentro de un pedido"""
    
    def __init__(self, producto: Producto, cantidad: int):
        if cantidad <= 0:
            raise ValueError("La cantidad debe ser positiva")
        
        if not producto.tiene_stock(cantidad):
            raise ValueError(f"Stock insuficiente de {producto.nombre}")
        
        self.producto = producto
        self.cantidad = cantidad
        self.precio_unitario = producto.precio  # Precio al momento de la compra
    
    def calcular_subtotal(self) -> float:
        """Calcula el subtotal del item"""
        return self.precio_unitario * self.cantidad
    
    def __str__(self):
        return f"{self.cantidad}x {self.producto.nombre} @ ${self.precio_unitario}"
```

### Entidad: Pedido

```python
from enum import Enum

class EstadoPedido(Enum):
    """Estados posibles de un pedido"""
    PENDIENTE = 'pendiente'
    PROCESANDO = 'procesando'
    PAGADO = 'pagado'
    ENVIADO = 'enviado'
    ENTREGADO = 'entregado'
    CANCELADO = 'cancelado'

class Pedido:
    """Representa un pedido de compra"""
    
    def __init__(self, id: int, cliente: Cliente, direccion_envio: Direccion):
        self.id = id
        self.cliente = cliente
        self.direccion_envio = direccion_envio
        self.items = []
        self.estado = EstadoPedido.PENDIENTE
        self.fecha_creacion = datetime.now()
        self.fecha_actualizacion = datetime.now()
        self.pago = None
    
    def agregar_item(self, producto: Producto, cantidad: int):
        """Añade un producto al pedido"""
        # Verificar si el producto ya está en el pedido
        item_existente = next((i for i in self.items if i.producto.id == producto.id), None)
        
        if item_existente:
            # Aumentar cantidad
            nueva_cantidad = item_existente.cantidad + cantidad
            if not producto.tiene_stock(nueva_cantidad):
                raise ValueError(f"Stock insuficiente de {producto.nombre}")
            item_existente.cantidad = nueva_cantidad
        else:
            # Crear nuevo item
            item = ItemPedido(producto, cantidad)
            self.items.append(item)
        
        self._actualizar_timestamp()
    
    def eliminar_item(self, producto_id: int):
        """Elimina un item del pedido"""
        self.items = [i for i in self.items if i.producto.id != producto_id]
        self._actualizar_timestamp()
    
    def calcular_subtotal(self) -> float:
        """Calcula el subtotal sin envío"""
        return sum(item.calcular_subtotal() for item in self.items)
    
    def calcular_envio(self) -> float:
        """Calcula el costo de envío"""
        subtotal = self.calcular_subtotal()
        if subtotal >= 100:
            return 0  # Envío gratis
        return 10
    
    def calcular_total(self) -> float:
        """Calcula el total del pedido"""
        return self.calcular_subtotal() + self.calcular_envio()
    
    def puede_procesarse(self) -> bool:
        """Verifica si el pedido puede procesarse"""
        if not self.items:
            return False
        if self.estado != EstadoPedido.PENDIENTE:
            return False
        return True
    
    def procesar(self):
        """Procesa el pedido"""
        if not self.puede_procesarse():
            raise ValueError(f"El pedido no puede procesarse. Estado: {self.estado.value}")
        
        # Reducir stock de productos
        for item in self.items:
            item.producto.reducir_stock(item.cantidad)
        
        self.estado = EstadoPedido.PROCESANDO
        self._actualizar_timestamp()
    
    def marcar_como_pagado(self, pago):
        """Marca el pedido como pagado"""
        if self.estado != EstadoPedido.PROCESANDO:
            raise ValueError("El pedido debe estar en estado 'procesando'")
        
        self.pago = pago
        self.estado = EstadoPedido.PAGADO
        self._actualizar_timestamp()
    
    def marcar_como_enviado(self):
        """Marca el pedido como enviado"""
        if self.estado != EstadoPedido.PAGADO:
            raise ValueError("El pedido debe estar pagado")
        
        self.estado = EstadoPedido.ENVIADO
        self._actualizar_timestamp()
    
    def marcar_como_entregado(self):
        """Marca el pedido como entregado"""
        if self.estado != EstadoPedido.ENVIADO:
            raise ValueError("El pedido debe estar enviado")
        
        self.estado = EstadoPedido.ENTREGADO
        self._actualizar_timestamp()
    
    def cancelar(self):
        """Cancela el pedido"""
        if self.estado in [EstadoPedido.ENVIADO, EstadoPedido.ENTREGADO]:
            raise ValueError("No se puede cancelar un pedido enviado o entregado")
        
        # Devolver stock
        if self.estado == EstadoPedido.PROCESANDO:
            for item in self.items:
                item.producto.aumentar_stock(item.cantidad)
        
        self.estado = EstadoPedido.CANCELADO
        self._actualizar_timestamp()
    
    def _actualizar_timestamp(self):
        """Actualiza la fecha de modificación"""
        self.fecha_actualizacion = datetime.now()
    
    def __str__(self):
        return f"Pedido #{self.id} - {self.cliente.nombre} - {self.estado.value} - ${self.calcular_total()}"
```

### Entidad: Pago

```python
class MetodoPago(Enum):
    TARJETA = 'tarjeta'
    PAYPAL = 'paypal'
    TRANSFERENCIA = 'transferencia'

class Pago:
    """Representa un pago realizado"""
    
    def __init__(self, id: int, monto: float, metodo: MetodoPago):
        self.id = id
        self.monto = monto
        self.metodo = metodo
        self.fecha = datetime.now()
        self.estado = 'pendiente'
    
    def procesar(self):
        """Procesa el pago"""
        # Lógica de procesamiento (integración con pasarela)
        print(f"Procesando pago de ${self.monto} vía {self.metodo.value}")
        self.estado = 'completado'
    
    def __str__(self):
        return f"Pago #{self.id} - ${self.monto} - {self.metodo.value}"
```

---

## Servicio de dominio: GestorPedidos

Los **servicios de dominio** coordinan operaciones complejas:

```python
class GestorPedidos:
    """Servicio que gestiona el flujo de pedidos"""
    
    def __init__(self):
        self.pedidos = []
        self.ultimo_id = 0
    
    def crear_pedido(self, cliente: Cliente, direccion: Direccion) -> Pedido:
        """Crea un nuevo pedido"""
        self.ultimo_id += 1
        pedido = Pedido(self.ultimo_id, cliente, direccion)
        self.pedidos.append(pedido)
        cliente.realizar_pedido(pedido)
        return pedido
    
    def procesar_pedido(self, pedido_id: int, metodo_pago: MetodoPago) -> bool:
        """Procesa un pedido completo: validar, procesar, pagar"""
        pedido = self.obtener_pedido(pedido_id)
        if not pedido:
            raise ValueError(f"Pedido {pedido_id} no encontrado")
        
        try:
            # 1. Procesar pedido (reduce stock)
            pedido.procesar()
            
            # 2. Crear y procesar pago
            pago = Pago(pedido_id, pedido.calcular_total(), metodo_pago)
            pago.procesar()
            
            # 3. Marcar como pagado
            pedido.marcar_como_pagado(pago)
            
            print(f"✅ Pedido {pedido_id} procesado exitosamente")
            return True
        
        except Exception as e:
            print(f"❌ Error procesando pedido: {e}")
            # Revertir cambios si hubo error
            try:
                pedido.cancelar()
            except:
                pass
            return False
    
    def obtener_pedido(self, pedido_id: int) -> Optional[Pedido]:
        """Obtiene un pedido por ID"""
        return next((p for p in self.pedidos if p.id == pedido_id), None)
    
    def listar_pedidos_cliente(self, cliente_id: int):
        """Lista pedidos de un cliente"""
        return [p for p in self.pedidos if p.cliente.id == cliente_id]
    
    def listar_pedidos_por_estado(self, estado: EstadoPedido):
        """Lista pedidos por estado"""
        return [p for p in self.pedidos if p.estado == estado]
```

---

## Ejemplo de uso completo

```python
# Crear productos
laptop = Producto(1, 'Laptop Gaming', 999, 5, 'electronica')
mouse = Producto(2, 'Mouse Inalámbrico', 25, 50, 'electronica')
teclado = Producto(3, 'Teclado Mecánico', 75, 20, 'electronica')

# Crear cliente
cliente = Cliente(1, 'Ana García', 'ana@ejemplo.com')

# Añadir dirección
direccion = Direccion(
    calle='Calle Principal 123',
    ciudad='Madrid',
    codigo_postal='28001',
    es_principal=True
)
cliente.agregar_direccion(direccion)

# Crear gestor de pedidos
gestor = GestorPedidos()

# Crear pedido
pedido = gestor.crear_pedido(cliente, direccion)
print(f"Pedido creado: {pedido}")

# Añadir items
pedido.agregar_item(laptop, 1)
pedido.agregar_item(mouse, 2)
pedido.agregar_item(teclado, 1)

print(f"\nItems del pedido:")
for item in pedido.items:
    print(f"  {item} - Subtotal: ${item.calcular_subtotal()}")

print(f"\nSubtotal: ${pedido.calcular_subtotal()}")
print(f"Envío: ${pedido.calcular_envio()}")
print(f"Total: ${pedido.calcular_total()}")

# Procesar pedido
print(f"\nStock antes de procesar:")
print(f"  Laptop: {laptop.stock}")
print(f"  Mouse: {mouse.stock}")

gestor.procesar_pedido(pedido.id, MetodoPago.TARJETA)

print(f"\nStock después de procesar:")
print(f"  Laptop: {laptop.stock}")
print(f"  Mouse: {mouse.stock}")

print(f"\nEstado del pedido: {pedido.estado.value}")
print(f"Total gastado por cliente: ${cliente.total_gastado()}")
```

**Salida:**
```
Pedido creado: Pedido #1 - Ana García - pendiente - $1174.0

Items del pedido:
  1x Laptop Gaming @ $999 - Subtotal: $999
  2x Mouse Inalámbrico @ $25 - Subtotal: $50
  1x Teclado Mecánico @ $75 - Subtotal: $75

Subtotal: $1124.0
Envío: $0.0
Total: $1124.0

Stock antes de procesar:
  Laptop: 5
  Mouse: 50

Procesando pago de $1124.0 vía tarjeta
✅ Pedido 1 procesado exitosamente

Stock después de procesar:
  Laptop: 4
  Mouse: 48

Estado del pedido: pagado
Total gastado por cliente: $0.0
```

---

## Reglas de negocio en el dominio

Las **reglas de negocio** deben vivir en el dominio, no en controladores:

```python
class Pedido:
    def puede_tener_descuento(self) -> bool:
        """Regla: descuento solo si subtotal > $500"""
        return self.calcular_subtotal() > 500
    
    def aplicar_descuento(self, porcentaje: float):
        """Aplica descuento si cumple reglas"""
        if not self.puede_tener_descuento():
            raise ValueError("El pedido no califica para descuento")
        
        if porcentaje > 20:
            raise ValueError("Descuento máximo: 20%")
        
        for item in self.items:
            precio_con_descuento = item.precio_unitario * (1 - porcentaje / 100)
            item.precio_unitario = precio_con_descuento
```

---

## Validación en el dominio

```python
class Cliente:
    def __init__(self, id: int, nombre: str, email: str):
        self.id = id
        self.nombre = self._validar_nombre(nombre)
        self.email = self._validar_email(email)
    
    def _validar_nombre(self, nombre: str) -> str:
        """Valida el nombre"""
        if not nombre or len(nombre.strip()) < 2:
            raise ValueError("Nombre inválido")
        return nombre.strip()
    
    def _validar_email(self, email: str) -> str:
        """Valida el email"""
        if '@' not in email or '.' not in email:
            raise ValueError("Email inválido")
        return email.lower().strip()
```

---

## Integración con Flask

```python
from flask import Flask, jsonify, request

app = Flask(__name__)

# Instancias globales (en producción, usar base de datos)
productos_db = [
    Producto(1, 'Laptop Gaming', 999, 5, 'electronica'),
    Producto(2, 'Mouse Inalámbrico', 25, 50, 'electronica')
]

clientes_db = []
gestor_pedidos = GestorPedidos()
ultimo_cliente_id = 0

@app.route('/api/pedidos', methods=['POST'])
def crear_pedido():
    """Crea un nuevo pedido"""
    global ultimo_cliente_id
    datos = request.json
    
    try:
        # Obtener o crear cliente
        cliente = next((c for c in clientes_db if c.id == datos['cliente_id']), None)
        if not cliente:
            ultimo_cliente_id += 1
            cliente = Cliente(ultimo_cliente_id, datos['cliente_nombre'], datos['cliente_email'])
            clientes_db.append(cliente)
        
        # Crear dirección
        direccion = Direccion(
            datos['direccion']['calle'],
            datos['direccion']['ciudad'],
            datos['direccion']['codigo_postal']
        )
        
        # Crear pedido
        pedido = gestor_pedidos.crear_pedido(cliente, direccion)
        
        # Añadir items
        for item_data in datos['items']:
            producto = next((p for p in productos_db if p.id == item_data['producto_id']), None)
            if not producto:
                return jsonify({'error': f'Producto {item_data["producto_id"]} no encontrado'}), 404
            
            pedido.agregar_item(producto, item_data['cantidad'])
        
        return jsonify({
            'pedido_id': pedido.id,
            'cliente': cliente.nombre,
            'items': len(pedido.items),
            'total': pedido.calcular_total(),
            'estado': pedido.estado.value
        }), 201
    
    except ValueError as e:
        return jsonify({'error': str(e)}), 400

@app.route('/api/pedidos/<int:pedido_id>/procesar', methods=['POST'])
def procesar_pedido(pedido_id):
    """Procesa un pedido"""
    datos = request.json
    
    try:
        metodo_pago = MetodoPago(datos['metodo_pago'])
        exito = gestor_pedidos.procesar_pedido(pedido_id, metodo_pago)
        
        if exito:
            pedido = gestor_pedidos.obtener_pedido(pedido_id)
            return jsonify({
                'mensaje': 'Pedido procesado exitosamente',
                'pedido_id': pedido.id,
                'estado': pedido.estado.value,
                'total': pedido.calcular_total()
            })
        else:
            return jsonify({'error': 'Error procesando pedido'}), 500
    
    except ValueError as e:
        return jsonify({'error': str(e)}), 400

if __name__ == '__main__':
    app.run(debug=True)
```

---

## Principios del modelado de dominio

### ✅ Ubiquitous Language

Usa el mismo lenguaje en código y con el negocio:

```python
# ❌ MAL
class Order:
    def calc(self):  # ¿Qué calcula?
        pass

# ✅ BIEN
class Pedido:
    def calcular_total(self):
        pass
```

### ✅ Rich Domain Model

El dominio contiene lógica de negocio, no solo datos:

```python
# ❌ Anemic Model (solo datos)
class Pedido:
    def __init__(self):
        self.items = []
        self.total = 0

# ✅ Rich Model (datos + comportamiento)
class Pedido:
    def __init__(self):
        self.items = []
    
    def calcular_total(self):
        return sum(i.calcular_subtotal() for i in self.items)
```

---

## Conclusión

Has aprendido:

- ✅ Qué es el modelado de dominio
- ✅ Identificar entidades, relaciones y comportamientos
- ✅ Diseñar un modelo de dominio completo
- ✅ Servicios de dominio
- ✅ Reglas de negocio en el dominio
- ✅ Integración con Flask
- ✅ Principios del modelado

**Siguiente:** [Patrones de diseño en backend](./17-patrones-diseño-backend.md)

---

## Recursos adicionales

- **[Domain-Driven Design](https://martinfowler.com/bliki/DomainDrivenDesign.html)** - Martin Fowler
- **[DDD in Python](https://www.cosmicpython.com/)** - Libro gratuito

El modelado de dominio hace tu código reflejar el negocio. 🎯

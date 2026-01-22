# Clases y objetos en el backend

> **Organiza tu código backend con Programación Orientada a Objetos**

---

## ¿Por qué POO en el backend?

Hasta ahora has trabajado con diccionarios para representar datos:

```python
producto = {
    'id': 1,
    'nombre': 'Laptop',
    'precio': 999,
    'stock': 10
}
```

**Problemas:**
- ❌ No hay garantía de estructura (falta validación)
- ❌ No hay comportamiento asociado (métodos)
- ❌ Difícil de mantener en proyectos grandes
- ❌ Propenso a errores de tipado

**Con POO:**
```python
class Producto:
    def __init__(self, id, nombre, precio, stock):
        self.id = id
        self.nombre = nombre
        self.precio = precio
        self.stock = stock
    
    def tiene_stock(self):
        return self.stock > 0
    
    def aplicar_descuento(self, porcentaje):
        self.precio = self.precio * (1 - porcentaje / 100)

producto = Producto(1, 'Laptop', 999, 10)
```

**Ventajas:**
- ✅ Estructura clara y predecible
- ✅ Comportamiento encapsulado (métodos)
- ✅ Más fácil de entender y mantener
- ✅ Reutilizable

---

## Anatomía de una clase

```python
class Producto:
    # Atributo de clase (compartido por todas las instancias)
    iva = 0.21
    
    # Constructor
    def __init__(self, id, nombre, precio, stock):
        # Atributos de instancia (únicos para cada objeto)
        self.id = id
        self.nombre = nombre
        self.precio = precio
        self.stock = stock
    
    # Método de instancia
    def precio_con_iva(self):
        return self.precio * (1 + self.iva)
    
    # Método especial
    def __str__(self):
        return f"Producto({self.nombre}, ${self.precio})"
```

**Componentes:**
- **`class`**: Define una nueva clase
- **`__init__`**: Constructor (inicializa el objeto)
- **`self`**: Referencia al objeto actual
- **Atributos de instancia**: `self.nombre`, `self.precio`, etc.
- **Atributos de clase**: `iva` (compartido por todos)
- **Métodos**: Funciones dentro de la clase

---

## Tu primera clase: Producto

```python
class Producto:
    def __init__(self, id, nombre, precio, stock):
        self.id = id
        self.nombre = nombre
        self.precio = precio
        self.stock = stock
    
    def tiene_stock(self):
        """Verifica si hay stock disponible"""
        return self.stock > 0
    
    def reducir_stock(self, cantidad):
        """Reduce el stock en la cantidad especificada"""
        if cantidad > self.stock:
            raise ValueError(f"Stock insuficiente. Disponible: {self.stock}")
        self.stock -= cantidad
    
    def aumentar_stock(self, cantidad):
        """Aumenta el stock"""
        if cantidad < 0:
            raise ValueError("La cantidad debe ser positiva")
        self.stock += cantidad
    
    def __str__(self):
        """Representación en string del producto"""
        return f"{self.nombre} - ${self.precio} (Stock: {self.stock})"

# Crear instancias (objetos)
laptop = Producto(1, 'Laptop Gaming', 999, 5)
mouse = Producto(2, 'Mouse Inalámbrico', 25, 50)

# Usar métodos
print(laptop.tiene_stock())  # True
laptop.reducir_stock(2)
print(laptop.stock)  # 3
print(laptop)  # Laptop Gaming - $999 (Stock: 3)
```

---

## Encapsulación: Atributos privados

En Python, los atributos que empiezan con `_` son "privados" por convención:

```python
class CuentaBancaria:
    def __init__(self, titular, saldo_inicial=0):
        self.titular = titular
        self._saldo = saldo_inicial  # Privado por convención
    
    def depositar(self, cantidad):
        """Método público para depositar"""
        if cantidad <= 0:
            raise ValueError("La cantidad debe ser positiva")
        self._saldo += cantidad
    
    def retirar(self, cantidad):
        """Método público para retirar"""
        if cantidad > self._saldo:
            raise ValueError("Saldo insuficiente")
        self._saldo -= cantidad
    
    def obtener_saldo(self):
        """Acceso controlado al saldo"""
        return self._saldo

# Uso
cuenta = CuentaBancaria('Juan', 1000)
cuenta.depositar(500)
cuenta.retirar(200)
print(cuenta.obtener_saldo())  # 1300

# Técnicamente puedes acceder a _saldo, pero no deberías
# cuenta._saldo = 999999  # ❌ Rompe la encapsulación
```

**Ventaja:** Control sobre cómo se accede y modifica el estado interno.

---

## Properties: Getters y Setters pythónicos

```python
class Producto:
    def __init__(self, nombre, precio):
        self.nombre = nombre
        self._precio = precio  # Privado
    
    @property
    def precio(self):
        """Getter: se accede como atributo"""
        return self._precio
    
    @precio.setter
    def precio(self, valor):
        """Setter: validación automática"""
        if valor < 0:
            raise ValueError("El precio no puede ser negativo")
        self._precio = valor
    
    @property
    def precio_con_iva(self):
        """Propiedad calculada (solo getter)"""
        return self._precio * 1.21

# Uso (parece un atributo, pero tiene validación)
producto = Producto('Laptop', 999)

print(producto.precio)  # 999 (llama al getter)
producto.precio = 1200  # Llama al setter con validación
print(producto.precio_con_iva)  # 1452.0

# producto.precio = -100  # ❌ ValueError
```

---

## Métodos de clase y estáticos

```python
from datetime import datetime

class Producto:
    contador_productos = 0  # Atributo de clase
    
    def __init__(self, nombre, precio):
        self.nombre = nombre
        self.precio = precio
        self.fecha_creacion = datetime.now()
        Producto.contador_productos += 1
    
    @classmethod
    def from_dict(cls, datos):
        """Constructor alternativo desde diccionario"""
        return cls(datos['nombre'], datos['precio'])
    
    @classmethod
    def obtener_total_productos(cls):
        """Método de clase: accede a atributos de clase"""
        return cls.contador_productos
    
    @staticmethod
    def validar_precio(precio):
        """Método estático: no accede a self ni cls"""
        return precio > 0 and precio < 1000000

# Uso de métodos de clase
producto1 = Producto('Laptop', 999)
producto2 = Producto.from_dict({'nombre': 'Mouse', 'precio': 25})

print(Producto.obtener_total_productos())  # 2

# Uso de métodos estáticos
print(Producto.validar_precio(500))  # True
print(Producto.validar_precio(-10))  # False
```

**Diferencias:**
- **Método de instancia** (`self`): Opera sobre una instancia específica
- **Método de clase** (`@classmethod`, `cls`): Opera sobre la clase
- **Método estático** (`@staticmethod`): No accede a instancia ni clase

---

## Ejemplo completo: Sistema de productos

```python
from datetime import datetime
from typing import List, Optional

class Producto:
    """Representa un producto en el inventario"""
    
    # Atributo de clase
    iva = 0.21
    
    def __init__(self, id: int, nombre: str, precio: float, stock: int, categoria: str):
        # Validaciones
        if precio < 0:
            raise ValueError("El precio no puede ser negativo")
        if stock < 0:
            raise ValueError("El stock no puede ser negativo")
        
        # Atributos de instancia
        self.id = id
        self.nombre = nombre
        self._precio = precio
        self._stock = stock
        self.categoria = categoria
        self.fecha_creacion = datetime.now()
        self.activo = True
    
    # Properties
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
    
    @property
    def precio_con_iva(self):
        """Precio incluyendo IVA"""
        return self._precio * (1 + self.iva)
    
    # Métodos de negocio
    def tiene_stock(self) -> bool:
        """Verifica si hay stock disponible"""
        return self._stock > 0 and self.activo
    
    def reducir_stock(self, cantidad: int):
        """Reduce el stock del producto"""
        if cantidad <= 0:
            raise ValueError("La cantidad debe ser positiva")
        if cantidad > self._stock:
            raise ValueError(f"Stock insuficiente. Disponible: {self._stock}")
        self._stock -= cantidad
    
    def aumentar_stock(self, cantidad: int):
        """Aumenta el stock del producto"""
        if cantidad <= 0:
            raise ValueError("La cantidad debe ser positiva")
        self._stock += cantidad
    
    def aplicar_descuento(self, porcentaje: float):
        """Aplica un descuento al precio"""
        if porcentaje < 0 or porcentaje > 100:
            raise ValueError("El porcentaje debe estar entre 0 y 100")
        descuento = self._precio * (porcentaje / 100)
        self._precio -= descuento
    
    def desactivar(self):
        """Desactiva el producto (soft delete)"""
        self.activo = False
    
    def activar(self):
        """Activa el producto"""
        self.activo = True
    
    # Métodos especiales
    def __str__(self):
        """Representación legible"""
        estado = "Activo" if self.activo else "Inactivo"
        return f"{self.nombre} - ${self.precio} ({estado})"
    
    def __repr__(self):
        """Representación técnica"""
        return f"Producto(id={self.id}, nombre='{self.nombre}', precio={self.precio}, stock={self._stock})"
    
    def __eq__(self, otro):
        """Comparación por igualdad"""
        if not isinstance(otro, Producto):
            return False
        return self.id == otro.id
    
    # Conversión
    def to_dict(self):
        """Convierte el producto a diccionario (para JSON)"""
        return {
            'id': self.id,
            'nombre': self.nombre,
            'precio': self.precio,
            'precio_con_iva': self.precio_con_iva,
            'stock': self._stock,
            'categoria': self.categoria,
            'activo': self.activo,
            'fecha_creacion': self.fecha_creacion.isoformat()
        }
    
    @classmethod
    def from_dict(cls, datos: dict):
        """Crea un producto desde un diccionario"""
        return cls(
            id=datos['id'],
            nombre=datos['nombre'],
            precio=datos['precio'],
            stock=datos['stock'],
            categoria=datos['categoria']
        )

# Uso
laptop = Producto(1, 'Laptop Gaming', 999, 10, 'electronica')

print(laptop)  # Laptop Gaming - $999 (Activo)
print(f"Precio con IVA: ${laptop.precio_con_iva:.2f}")

laptop.reducir_stock(3)
print(f"Stock actual: {laptop.stock}")

laptop.aplicar_descuento(10)
print(f"Nuevo precio: ${laptop.precio:.2f}")

# Convertir a diccionario para API
print(laptop.to_dict())
```

---

## Gestor de inventario

```python
class GestorInventario:
    """Gestiona una colección de productos"""
    
    def __init__(self):
        self._productos: List[Producto] = []
        self._ultimo_id = 0
    
    def agregar_producto(self, nombre: str, precio: float, stock: int, categoria: str) -> Producto:
        """Agrega un nuevo producto al inventario"""
        self._ultimo_id += 1
        producto = Producto(self._ultimo_id, nombre, precio, stock, categoria)
        self._productos.append(producto)
        return producto
    
    def obtener_producto(self, id: int) -> Optional[Producto]:
        """Obtiene un producto por ID"""
        return next((p for p in self._productos if p.id == id), None)
    
    def listar_productos(self, solo_activos: bool = True) -> List[Producto]:
        """Lista todos los productos"""
        if solo_activos:
            return [p for p in self._productos if p.activo]
        return self._productos.copy()
    
    def buscar_por_categoria(self, categoria: str) -> List[Producto]:
        """Busca productos por categoría"""
        return [p for p in self._productos if p.categoria == categoria and p.activo]
    
    def productos_sin_stock(self) -> List[Producto]:
        """Obtiene productos sin stock"""
        return [p for p in self._productos if not p.tiene_stock()]
    
    def valor_total_inventario(self) -> float:
        """Calcula el valor total del inventario"""
        return sum(p.precio * p.stock for p in self._productos if p.activo)
    
    def eliminar_producto(self, id: int):
        """Elimina un producto (soft delete)"""
        producto = self.obtener_producto(id)
        if not producto:
            raise ValueError(f"Producto {id} no encontrado")
        producto.desactivar()

# Uso
inventario = GestorInventario()

# Agregar productos
laptop = inventario.agregar_producto('Laptop Gaming', 999, 5, 'electronica')
mouse = inventario.agregar_producto('Mouse Inalámbrico', 25, 50, 'electronica')
silla = inventario.agregar_producto('Silla Gamer', 200, 10, 'muebles')

# Listar productos
print("Productos activos:")
for producto in inventario.listar_productos():
    print(f"  - {producto}")

# Buscar por categoría
print("\nProductos de electrónica:")
for producto in inventario.buscar_por_categoria('electronica'):
    print(f"  - {producto}")

# Valor total
print(f"\nValor total del inventario: ${inventario.valor_total_inventario():.2f}")
```

---

## Integración con Flask

```python
from flask import Flask, jsonify, request
from typing import List

app = Flask(__name__)

# Instancia global del gestor
inventario = GestorInventario()

# Crear algunos productos de ejemplo
inventario.agregar_producto('Laptop Gaming', 999, 5, 'electronica')
inventario.agregar_producto('Mouse Inalámbrico', 25, 50, 'electronica')

@app.route('/api/productos', methods=['GET'])
def listar_productos():
    """Lista todos los productos"""
    productos = inventario.listar_productos()
    return jsonify([p.to_dict() for p in productos])

@app.route('/api/productos/<int:id>', methods=['GET'])
def obtener_producto(id):
    """Obtiene un producto por ID"""
    producto = inventario.obtener_producto(id)
    if not producto:
        return jsonify({'error': 'Producto no encontrado'}), 404
    return jsonify(producto.to_dict())

@app.route('/api/productos', methods=['POST'])
def crear_producto():
    """Crea un nuevo producto"""
    datos = request.json
    
    try:
        producto = inventario.agregar_producto(
            nombre=datos['nombre'],
            precio=datos['precio'],
            stock=datos['stock'],
            categoria=datos['categoria']
        )
        return jsonify(producto.to_dict()), 201
    except (KeyError, ValueError) as e:
        return jsonify({'error': str(e)}), 400

@app.route('/api/productos/<int:id>/stock', methods=['PATCH'])
def actualizar_stock(id):
    """Actualiza el stock de un producto"""
    producto = inventario.obtener_producto(id)
    if not producto:
        return jsonify({'error': 'Producto no encontrado'}), 404
    
    datos = request.json
    accion = datos.get('accion')
    cantidad = datos.get('cantidad')
    
    try:
        if accion == 'aumentar':
            producto.aumentar_stock(cantidad)
        elif accion == 'reducir':
            producto.reducir_stock(cantidad)
        else:
            return jsonify({'error': 'Acción inválida'}), 400
        
        return jsonify(producto.to_dict())
    except ValueError as e:
        return jsonify({'error': str(e)}), 400

@app.route('/api/productos/<int:id>', methods=['DELETE'])
def eliminar_producto(id):
    """Elimina un producto (soft delete)"""
    try:
        inventario.eliminar_producto(id)
        return '', 204
    except ValueError as e:
        return jsonify({'error': str(e)}), 404

if __name__ == '__main__':
    app.run(debug=True)
```

---

## Ventajas de POO en el backend

### ✅ Código más organizado

```python
# ❌ Sin POO (funciones sueltas)
def calcular_precio_con_iva(precio):
    return precio * 1.21

def reducir_stock(producto, cantidad):
    if cantidad > producto['stock']:
        raise ValueError("Stock insuficiente")
    producto['stock'] -= cantidad

# ✅ Con POO (todo junto)
class Producto:
    def precio_con_iva(self):
        return self.precio * 1.21
    
    def reducir_stock(self, cantidad):
        if cantidad > self.stock:
            raise ValueError("Stock insuficiente")
        self.stock -= cantidad
```

### ✅ Validación automática

```python
class Producto:
    @property
    def precio(self):
        return self._precio
    
    @precio.setter
    def precio(self, valor):
        if valor < 0:
            raise ValueError("Precio inválido")
        self._precio = valor

# Validación automática al asignar
producto.precio = -100  # ❌ ValueError
```

### ✅ Reutilización

```python
# Misma clase para diferentes contextos
inventario = GestorInventario()
carrito = CarritoCompras()
pedido = Pedido()

# Todos usan la clase Producto
```

---

## Conclusión

Has aprendido:

- ✅ Por qué usar POO en el backend
- ✅ Anatomía de una clase (atributos, métodos, constructor)
- ✅ Encapsulación con atributos privados
- ✅ Properties (getters y setters)
- ✅ Métodos de clase y estáticos
- ✅ Métodos especiales (`__str__`, `__repr__`, `__eq__`)
- ✅ Conversión a/desde diccionarios
- ✅ Integración con Flask

**Siguiente:** [Herencia en el backend](./14-herencia-backend.md)

---

## Recursos adicionales

- **[Python Classes](https://docs.python.org/3/tutorial/classes.html)** - Documentación oficial
- **[Real Python - OOP](https://realpython.com/python3-object-oriented-programming/)** - Tutorial completo

POO hace tu código backend más profesional y mantenible. 🎯

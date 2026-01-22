# Herencia en el backend

> **Reutiliza código creando jerarquías de clases**

---

## ¿Qué es la herencia?

**Herencia** permite crear nuevas clases basadas en clases existentes, heredando sus atributos y métodos.

**Analogía:** Es como heredar características de tus padres, pero puedes añadir las tuyas propias.

```python
class Animal:
    def respirar(self):
        print("Respirando...")

class Perro(Animal):  # Hereda de Animal
    def ladrar(self):
        print("Guau!")

perro = Perro()
perro.respirar()  # Heredado de Animal
perro.ladrar()    # Propio de Perro
```

---

## Problema: Código repetido

Sin herencia, repites código:

```python
class ProductoFisico:
    def __init__(self, id, nombre, precio, peso):
        self.id = id
        self.nombre = nombre
        self.precio = precio
        self.peso = peso  # Específico
    
    def calcular_iva(self):
        return self.precio * 0.21

class ProductoDigital:
    def __init__(self, id, nombre, precio, tamaño_mb):
        self.id = id
        self.nombre = nombre
        self.precio = precio
        self.tamaño_mb = tamaño_mb  # Específico
    
    def calcular_iva(self):
        return self.precio * 0.21  # ❌ Repetido
```

**Con herencia:**

```python
class Producto:
    """Clase base con comportamiento común"""
    def __init__(self, id, nombre, precio):
        self.id = id
        self.nombre = nombre
        self.precio = precio
    
    def calcular_iva(self):
        return self.precio * 0.21

class ProductoFisico(Producto):
    """Especialización para productos físicos"""
    def __init__(self, id, nombre, precio, peso):
        super().__init__(id, nombre, precio)  # Llama al constructor padre
        self.peso = peso

class ProductoDigital(Producto):
    """Especialización para productos digitales"""
    def __init__(self, id, nombre, precio, tamaño_mb):
        super().__init__(id, nombre, precio)
        self.tamaño_mb = tamaño_mb

# Ambos tienen calcular_iva() heredado
laptop = ProductoFisico(1, 'Laptop', 999, 2.5)
ebook = ProductoDigital(2, 'eBook Python', 29, 5)

print(laptop.calcular_iva())  # 209.79
print(ebook.calcular_iva())   # 6.09
```

---

## Sintaxis básica

```python
class ClasePadre:
    def metodo_comun(self):
        print("Método común")

class ClaseHija(ClasePadre):  # Hereda de ClasePadre
    def metodo_propio(self):
        print("Método propio")

# La hija tiene ambos métodos
hija = ClaseHija()
hija.metodo_comun()  # Heredado
hija.metodo_propio()  # Propio
```

---

## super(): Llamar al padre

`super()` permite llamar a métodos de la clase padre:

```python
class Usuario:
    def __init__(self, id, nombre, email):
        self.id = id
        self.nombre = nombre
        self.email = email
    
    def to_dict(self):
        return {
            'id': self.id,
            'nombre': self.nombre,
            'email': self.email
        }

class Administrador(Usuario):
    def __init__(self, id, nombre, email, permisos):
        super().__init__(id, nombre, email)  # Llama a Usuario.__init__
        self.permisos = permisos
    
    def to_dict(self):
        # Obtener dict del padre y añadir permisos
        datos = super().to_dict()
        datos['permisos'] = self.permisos
        return datos

admin = Administrador(1, 'Juan', 'juan@ejemplo.com', ['crear', 'eliminar'])
print(admin.to_dict())
# {'id': 1, 'nombre': 'Juan', 'email': 'juan@ejemplo.com', 'permisos': ['crear', 'eliminar']}
```

---

## Sobrescritura de métodos

Las clases hijas pueden **sobrescribir** (override) métodos del padre:

```python
class Producto:
    def __init__(self, nombre, precio):
        self.nombre = nombre
        self.precio = precio
    
    def calcular_envio(self):
        """Método base: envío estándar"""
        return 10.0

class ProductoFisico(Producto):
    def __init__(self, nombre, precio, peso):
        super().__init__(nombre, precio)
        self.peso = peso
    
    def calcular_envio(self):
        """Sobrescribe: envío por peso"""
        return 5.0 + (self.peso * 2)

class ProductoDigital(Producto):
    def calcular_envio(self):
        """Sobrescribe: sin envío"""
        return 0.0

laptop = ProductoFisico('Laptop', 999, 2.5)
ebook = ProductoDigital('eBook', 29)

print(laptop.calcular_envio())  # 10.0 (5 + 2.5*2)
print(ebook.calcular_envio())   # 0.0
```

---

## Ejemplo práctico: Sistema de usuarios

```python
from datetime import datetime
from typing import List

class Usuario:
    """Clase base para todos los usuarios"""
    
    def __init__(self, id: int, nombre: str, email: str):
        self.id = id
        self.nombre = nombre
        self.email = email
        self.fecha_registro = datetime.now()
        self.activo = True
    
    def to_dict(self):
        """Conversión a diccionario"""
        return {
            'id': self.id,
            'nombre': self.nombre,
            'email': self.email,
            'tipo': self.__class__.__name__,
            'fecha_registro': self.fecha_registro.isoformat(),
            'activo': self.activo
        }
    
    def desactivar(self):
        """Desactiva el usuario"""
        self.activo = False
    
    def __str__(self):
        return f"{self.__class__.__name__}: {self.nombre} ({self.email})"

class Cliente(Usuario):
    """Cliente regular de la tienda"""
    
    def __init__(self, id: int, nombre: str, email: str):
        super().__init__(id, nombre, email)
        self.historial_compras = []
        self.direccion_envio = None
    
    def agregar_compra(self, compra):
        """Registra una compra"""
        self.historial_compras.append(compra)
    
    def total_gastado(self):
        """Calcula el total gastado"""
        return sum(compra['total'] for compra in self.historial_compras)
    
    def to_dict(self):
        datos = super().to_dict()
        datos['total_gastado'] = self.total_gastado()
        datos['num_compras'] = len(self.historial_compras)
        return datos

class Administrador(Usuario):
    """Usuario con permisos administrativos"""
    
    def __init__(self, id: int, nombre: str, email: str, permisos: List[str]):
        super().__init__(id, nombre, email)
        self.permisos = permisos
    
    def tiene_permiso(self, permiso: str) -> bool:
        """Verifica si tiene un permiso específico"""
        return permiso in self.permisos
    
    def agregar_permiso(self, permiso: str):
        """Agrega un nuevo permiso"""
        if permiso not in self.permisos:
            self.permisos.append(permiso)
    
    def to_dict(self):
        datos = super().to_dict()
        datos['permisos'] = self.permisos
        return datos

class Vendedor(Usuario):
    """Usuario que puede gestionar productos"""
    
    def __init__(self, id: int, nombre: str, email: str, comision: float):
        super().__init__(id, nombre, email)
        self.comision = comision  # Porcentaje de comisión
        self.ventas = []
    
    def registrar_venta(self, venta):
        """Registra una venta"""
        self.ventas.append(venta)
    
    def calcular_comision_total(self):
        """Calcula la comisión total ganada"""
        total_ventas = sum(venta['total'] for venta in self.ventas)
        return total_ventas * (self.comision / 100)
    
    def to_dict(self):
        datos = super().to_dict()
        datos['comision'] = self.comision
        datos['ventas_realizadas'] = len(self.ventas)
        datos['comision_total'] = self.calcular_comision_total()
        return datos

# Uso
cliente = Cliente(1, 'Ana García', 'ana@ejemplo.com')
cliente.agregar_compra({'producto': 'Laptop', 'total': 999})
cliente.agregar_compra({'producto': 'Mouse', 'total': 25})

admin = Administrador(2, 'Carlos Admin', 'carlos@ejemplo.com', ['crear', 'eliminar'])

vendedor = Vendedor(3, 'Luis Vendedor', 'luis@ejemplo.com', 10)
vendedor.registrar_venta({'producto': 'Laptop', 'total': 999})

print(cliente.to_dict())
print(admin.to_dict())
print(vendedor.to_dict())
```

---

## isinstance() y type()

Verifica el tipo de un objeto:

```python
cliente = Cliente(1, 'Ana', 'ana@ejemplo.com')
admin = Administrador(2, 'Carlos', 'carlos@ejemplo.com', [])

# isinstance: verifica si es de un tipo (incluye herencia)
print(isinstance(cliente, Cliente))      # True
print(isinstance(cliente, Usuario))      # True (Cliente hereda de Usuario)
print(isinstance(cliente, Administrador)) # False

# type: tipo exacto (no considera herencia)
print(type(cliente) == Cliente)  # True
print(type(cliente) == Usuario)  # False

# Uso práctico
def procesar_usuario(usuario):
    if isinstance(usuario, Administrador):
        print(f"Admin con permisos: {usuario.permisos}")
    elif isinstance(usuario, Cliente):
        print(f"Cliente con {len(usuario.historial_compras)} compras")
    elif isinstance(usuario, Usuario):
        print(f"Usuario: {usuario.nombre}")
```

---

## Herencia múltiple (avanzado)

Python permite heredar de múltiples clases:

```python
class Auditable:
    """Mixin para auditoría"""
    def __init__(self):
        self.creado_en = datetime.now()
        self.modificado_en = datetime.now()
    
    def actualizar_timestamp(self):
        self.modificado_en = datetime.now()

class Identificable:
    """Mixin para objetos con ID"""
    def __init__(self, id):
        self.id = id

class Producto(Identificable, Auditable):
    """Hereda de ambas"""
    def __init__(self, id, nombre, precio):
        Identificable.__init__(self, id)
        Auditable.__init__(self)
        self.nombre = nombre
        self.precio = precio
    
    def actualizar_precio(self, nuevo_precio):
        self.precio = nuevo_precio
        self.actualizar_timestamp()

producto = Producto(1, 'Laptop', 999)
print(f"Creado: {producto.creado_en}")
producto.actualizar_precio(899)
print(f"Modificado: {producto.modificado_en}")
```

---

## Clases abstractas

Clases que no se pueden instanciar directamente (solo sirven como base):

```python
from abc import ABC, abstractmethod

class Pago(ABC):
    """Clase abstracta para métodos de pago"""
    
    def __init__(self, monto):
        self.monto = monto
        self.estado = 'pendiente'
    
    @abstractmethod
    def procesar(self):
        """Método abstracto: debe implementarse en clases hijas"""
        pass
    
    @abstractmethod
    def validar(self):
        """Otro método abstracto"""
        pass

class PagoTarjeta(Pago):
    def __init__(self, monto, numero_tarjeta):
        super().__init__(monto)
        self.numero_tarjeta = numero_tarjeta
    
    def procesar(self):
        """Implementación específica"""
        print(f"Procesando pago de ${self.monto} con tarjeta {self.numero_tarjeta[-4:]}")
        self.estado = 'completado'
    
    def validar(self):
        """Validación de tarjeta"""
        return len(self.numero_tarjeta) == 16

class PagoPayPal(Pago):
    def __init__(self, monto, email):
        super().__init__(monto)
        self.email = email
    
    def procesar(self):
        """Implementación específica"""
        print(f"Procesando pago de ${self.monto} vía PayPal ({self.email})")
        self.estado = 'completado'
    
    def validar(self):
        """Validación de email"""
        return '@' in self.email

# No se puede instanciar Pago directamente
# pago = Pago(100)  # ❌ TypeError

# Pero sí las clases hijas
pago_tarjeta = PagoTarjeta(100, '1234567812345678')
pago_paypal = PagoPayPal(50, 'user@ejemplo.com')

if pago_tarjeta.validar():
    pago_tarjeta.procesar()
```

---

## Polimorfismo

Objetos de diferentes clases pueden usarse de forma intercambiable si comparten una interfaz común:

```python
from abc import ABC, abstractmethod

class Notificacion(ABC):
    @abstractmethod
    def enviar(self, destinatario, mensaje):
        pass

class NotificacionEmail(Notificacion):
    def enviar(self, destinatario, mensaje):
        print(f"📧 Email a {destinatario}: {mensaje}")

class NotificacionSMS(Notificacion):
    def enviar(self, destinatario, mensaje):
        print(f"📱 SMS a {destinatario}: {mensaje}")

class NotificacionPush(Notificacion):
    def enviar(self, destinatario, mensaje):
        print(f"🔔 Push a {destinatario}: {mensaje}")

# Polimorfismo: todas tienen el mismo método
def notificar_usuario(notificacion: Notificacion, usuario, mensaje):
    """Función que acepta cualquier tipo de notificación"""
    notificacion.enviar(usuario, mensaje)

# Uso intercambiable
email = NotificacionEmail()
sms = NotificacionSMS()
push = NotificacionPush()

notificar_usuario(email, 'user@ejemplo.com', 'Pedido confirmado')
notificar_usuario(sms, '+34600123456', 'Pedido confirmado')
notificar_usuario(push, 'user123', 'Pedido confirmado')
```

---

## Ejemplo completo: Sistema de productos

```python
from abc import ABC, abstractmethod
from datetime import datetime

class Producto(ABC):
    """Clase base abstracta para todos los productos"""
    
    def __init__(self, id: int, nombre: str, precio: float, stock: int):
        self.id = id
        self.nombre = nombre
        self._precio = precio
        self._stock = stock
        self.fecha_creacion = datetime.now()
    
    @property
    def precio(self):
        return self._precio
    
    @abstractmethod
    def calcular_envio(self):
        """Cada tipo de producto calcula su envío diferente"""
        pass
    
    @abstractmethod
    def puede_enviarse(self):
        """Cada tipo verifica si puede enviarse"""
        pass
    
    def precio_total(self):
        """Precio + envío"""
        return self.precio + self.calcular_envio()
    
    def to_dict(self):
        return {
            'id': self.id,
            'nombre': self.nombre,
            'precio': self.precio,
            'stock': self._stock,
            'tipo': self.__class__.__name__,
            'envio': self.calcular_envio(),
            'precio_total': self.precio_total()
        }

class ProductoFisico(Producto):
    """Productos que requieren envío físico"""
    
    def __init__(self, id: int, nombre: str, precio: float, stock: int, peso: float):
        super().__init__(id, nombre, precio, stock)
        self.peso = peso
    
    def calcular_envio(self):
        """Envío basado en peso"""
        return 5.0 + (self.peso * 2)
    
    def puede_enviarse(self):
        """Solo si hay stock"""
        return self._stock > 0
    
    def to_dict(self):
        datos = super().to_dict()
        datos['peso'] = self.peso
        return datos

class ProductoDigital(Producto):
    """Productos descargables"""
    
    def __init__(self, id: int, nombre: str, precio: float, stock: int, tamaño_mb: float, url_descarga: str):
        super().__init__(id, nombre, precio, stock)
        self.tamaño_mb = tamaño_mb
        self.url_descarga = url_descarga
    
    def calcular_envio(self):
        """Sin costo de envío"""
        return 0.0
    
    def puede_enviarse(self):
        """Siempre disponible (stock ilimitado)"""
        return True
    
    def to_dict(self):
        datos = super().to_dict()
        datos['tamaño_mb'] = self.tamaño_mb
        datos['url_descarga'] = self.url_descarga
        return datos

class ProductoServicio(Producto):
    """Servicios (consultoría, suscripciones, etc.)"""
    
    def __init__(self, id: int, nombre: str, precio: float, duracion_dias: int):
        super().__init__(id, nombre, precio, 999)  # Stock ilimitado
        self.duracion_dias = duracion_dias
    
    def calcular_envio(self):
        """Sin envío físico"""
        return 0.0
    
    def puede_enviarse(self):
        """Siempre disponible"""
        return True
    
    def to_dict(self):
        datos = super().to_dict()
        datos['duracion_dias'] = self.duracion_dias
        return datos

# Uso
laptop = ProductoFisico(1, 'Laptop Gaming', 999, 5, 2.5)
ebook = ProductoDigital(2, 'eBook Python', 29, 999, 5, 'https://descarga.com/ebook')
consultoria = ProductoServicio(3, 'Consultoría Web', 500, 30)

productos = [laptop, ebook, consultoria]

for producto in productos:
    print(f"{producto.nombre}:")
    print(f"  Precio: ${producto.precio}")
    print(f"  Envío: ${producto.calcular_envio()}")
    print(f"  Total: ${producto.precio_total()}")
    print(f"  Puede enviarse: {producto.puede_enviarse()}")
    print()
```

---

## Integración con Flask

```python
from flask import Flask, jsonify, request

app = Flask(__name__)

# Base de datos simulada
productos = [
    ProductoFisico(1, 'Laptop Gaming', 999, 5, 2.5),
    ProductoDigital(2, 'eBook Python', 29, 999, 5, 'https://descarga.com/ebook'),
    ProductoServicio(3, 'Consultoría', 500, 30)
]

@app.route('/api/productos', methods=['GET'])
def listar_productos():
    """Lista todos los productos (polimorfismo)"""
    return jsonify([p.to_dict() for p in productos])

@app.route('/api/productos/<int:id>', methods=['GET'])
def obtener_producto(id):
    """Obtiene un producto por ID"""
    producto = next((p for p in productos if p.id == id), None)
    if not producto:
        return jsonify({'error': 'Producto no encontrado'}), 404
    return jsonify(producto.to_dict())

@app.route('/api/productos/<int:id>/precio-total', methods=['GET'])
def calcular_precio_total(id):
    """Calcula precio total con envío (polimorfismo)"""
    producto = next((p for p in productos if p.id == id), None)
    if not producto:
        return jsonify({'error': 'Producto no encontrado'}), 404
    
    return jsonify({
        'precio': producto.precio,
        'envio': producto.calcular_envio(),
        'total': producto.precio_total()
    })

if __name__ == '__main__':
    app.run(debug=True)
```

---

## Cuándo usar herencia

### ✅ Usa herencia cuando:
- Tienes una relación "es un" (Cliente **es un** Usuario)
- Compartes comportamiento común
- Quieres polimorfismo

### ❌ No uses herencia cuando:
- Solo compartes código (usa composición)
- La relación es "tiene un" (Pedido **tiene un** Cliente)
- Creas jerarquías muy profundas (>3 niveles)

---

## Conclusión

Has aprendido:

- ✅ Qué es la herencia y por qué usarla
- ✅ Sintaxis básica de herencia
- ✅ `super()` para llamar al padre
- ✅ Sobrescritura de métodos
- ✅ Clases abstractas (ABC)
- ✅ Polimorfismo
- ✅ `isinstance()` y `type()`
- ✅ Ejemplos prácticos con Flask

**Siguiente:** [Composición vs herencia](./15-composicion-backend.md)

---

## Recursos adicionales

- **[Python Inheritance](https://docs.python.org/3/tutorial/classes.html#inheritance)** - Docs oficiales
- **[ABC Module](https://docs.python.org/3/library/abc.html)** - Clases abstractas

La herencia bien usada hace tu código más reutilizable. 🎯

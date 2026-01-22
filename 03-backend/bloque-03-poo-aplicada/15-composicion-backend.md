# Composición vs Herencia

> **Elige la estrategia correcta para estructurar tu código**

---

## El dilema: ¿Herencia o composición?

Dos formas de reutilizar código:

**Herencia:** "es un"
```python
class Perro(Animal):  # Perro ES UN Animal
    pass
```

**Composición:** "tiene un"
```python
class Coche:
    def __init__(self):
        self.motor = Motor()  # Coche TIENE UN Motor
```

---

## Cuándo usar cada una

### Herencia: "Es un"

✅ Usa cuando hay una relación clara "es un tipo de":

```python
# ✅ BIEN: Cliente es un tipo de Usuario
class Usuario:
    pass

class Cliente(Usuario):
    pass

# ✅ BIEN: ProductoFisico es un tipo de Producto
class Producto:
    pass

class ProductoFisico(Producto):
    pass
```

### Composición: "Tiene un"

✅ Usa cuando hay una relación "tiene un":

```python
# ✅ BIEN: Pedido tiene un Cliente (no es un Cliente)
class Pedido:
    def __init__(self, cliente):
        self.cliente = cliente

# ✅ BIEN: Coche tiene un Motor (no es un Motor)
class Coche:
    def __init__(self):
        self.motor = Motor()
```

---

## Problema con herencia excesiva

```python
# ❌ MAL: Herencia profunda y rígida
class Animal:
    pass

class Mamifero(Animal):
    pass

class Canino(Mamifero):
    pass

class Perro(Canino):
    pass

class PerroGuia(Perro):
    pass

# Jerarquía demasiado profunda
# Difícil de mantener y modificar
```

**Regla:** Máximo 2-3 niveles de herencia.

---

## Composición: Solución flexible

En lugar de heredar todo, compón objetos más pequeños:

```python
# Componentes reutilizables
class Motor:
    def __init__(self, potencia):
        self.potencia = potencia
    
    def arrancar(self):
        print(f"Motor de {self.potencia}HP arrancando...")

class Ruedas:
    def __init__(self, cantidad):
        self.cantidad = cantidad
    
    def girar(self):
        print(f"Girando {self.cantidad} ruedas...")

class SistemaAudio:
    def __init__(self, marca):
        self.marca = marca
    
    def reproducir(self, cancion):
        print(f"Reproduciendo {cancion} en {self.marca}")

# Composición: ensamblar componentes
class Coche:
    def __init__(self):
        self.motor = Motor(150)
        self.ruedas = Ruedas(4)
        self.audio = SistemaAudio('Sony')
    
    def arrancar(self):
        self.motor.arrancar()
        self.ruedas.girar()
    
    def poner_musica(self, cancion):
        self.audio.reproducir(cancion)

class Moto:
    def __init__(self):
        self.motor = Motor(50)
        self.ruedas = Ruedas(2)
        # Sin audio
    
    def arrancar(self):
        self.motor.arrancar()
        self.ruedas.girar()

# Fácil de extender y reutilizar
coche = Coche()
coche.arrancar()
coche.poner_musica('Bohemian Rhapsody')

moto = Moto()
moto.arrancar()
```

---

## Ejemplo backend: Sistema de notificaciones

### ❌ Con herencia (rígido)

```python
class NotificadorBase:
    def enviar(self, mensaje):
        pass

class NotificadorEmail(NotificadorBase):
    def enviar(self, mensaje):
        print(f"📧 Email: {mensaje}")

class NotificadorSMS(NotificadorBase):
    def enviar(self, mensaje):
        print(f"📱 SMS: {mensaje}")

# ¿Qué pasa si quiero enviar por email Y SMS?
# Necesitaría herencia múltiple o repetir código
```

### ✅ Con composición (flexible)

```python
# Componentes pequeños
class EnviadorEmail:
    def enviar(self, mensaje):
        print(f"📧 Email: {mensaje}")

class EnviadorSMS:
    def enviar(self, mensaje):
        print(f"📱 SMS: {mensaje}")

class EnviadorPush:
    def enviar(self, mensaje):
        print(f"🔔 Push: {mensaje}")

# Compositor: combina múltiples enviadores
class Notificador:
    def __init__(self):
        self.enviadores = []
    
    def agregar_canal(self, enviador):
        """Añade un canal de notificación"""
        self.enviadores.append(enviador)
    
    def notificar(self, mensaje):
        """Envía por todos los canales"""
        for enviador in self.enviadores:
            enviador.enviar(mensaje)

# Uso flexible
notificador = Notificador()
notificador.agregar_canal(EnviadorEmail())
notificador.agregar_canal(EnviadorSMS())

notificador.notificar('Pedido confirmado')
# 📧 Email: Pedido confirmado
# 📱 SMS: Pedido confirmado

# Fácil agregar más canales
notificador.agregar_canal(EnviadorPush())
notificador.notificar('Pedido enviado')
```

---

## Ejemplo: Sistema de pagos

### ❌ Con herencia

```python
class Pago:
    def procesar(self):
        pass

class PagoConValidacion(Pago):
    def procesar(self):
        self.validar()
        super().procesar()

class PagoConNotificacion(Pago):
    def procesar(self):
        super().procesar()
        self.notificar()

# ¿Pago con validación Y notificación?
# Herencia múltiple complicada
class PagoCompleto(PagoConValidacion, PagoConNotificacion):
    pass  # ¿Qué orden se ejecutan los métodos?
```

### ✅ Con composición

```python
# Componentes independientes
class ValidadorPago:
    def validar(self, pago):
        print(f"Validando pago de ${pago.monto}")
        if pago.monto <= 0:
            raise ValueError("Monto inválido")
        return True

class NotificadorPago:
    def notificar(self, pago):
        print(f"Notificando pago de ${pago.monto}")

class RegistradorPago:
    def registrar(self, pago):
        print(f"Registrando pago de ${pago.monto} en BD")

# Clase simple con composición
class Pago:
    def __init__(self, monto):
        self.monto = monto
        self.validador = ValidadorPago()
        self.notificador = NotificadorPago()
        self.registrador = RegistradorPago()
    
    def procesar(self):
        """Procesa el pago usando los componentes"""
        try:
            self.validador.validar(self)
            # Procesar pago...
            print(f"Procesando pago de ${self.monto}")
            self.registrador.registrar(self)
            self.notificador.notificar(self)
            return True
        except Exception as e:
            print(f"Error: {e}")
            return False

# Uso
pago = Pago(100)
pago.procesar()
```

---

## Patrón Strategy con composición

Permite cambiar comportamiento dinámicamente:

```python
# Estrategias intercambiables
class EstrategiaEnvioEstandar:
    def calcular_costo(self, peso, distancia):
        return 10 + (peso * 0.5)

class EstrategiaEnvioExpress:
    def calcular_costo(self, peso, distancia):
        return 20 + (peso * 1) + (distancia * 0.5)

class EstrategiaEnvioGratis:
    def calcular_costo(self, peso, distancia):
        return 0

# Clase que usa estrategias
class Pedido:
    def __init__(self, productos, peso, distancia):
        self.productos = productos
        self.peso = peso
        self.distancia = distancia
        self.estrategia_envio = EstrategiaEnvioEstandar()  # Default
    
    def establecer_estrategia_envio(self, estrategia):
        """Cambia la estrategia dinámicamente"""
        self.estrategia_envio = estrategia
    
    def calcular_envio(self):
        """Delega el cálculo a la estrategia"""
        return self.estrategia_envio.calcular_costo(self.peso, self.distancia)
    
    def total(self):
        subtotal = sum(p['precio'] for p in self.productos)
        envio = self.calcular_envio()
        return subtotal + envio

# Uso
pedido = Pedido(
    productos=[{'nombre': 'Laptop', 'precio': 999}],
    peso=2.5,
    distancia=100
)

# Envío estándar
print(f"Envío estándar: ${pedido.calcular_envio()}")

# Cambiar a express
pedido.establecer_estrategia_envio(EstrategiaEnvioExpress())
print(f"Envío express: ${pedido.calcular_envio()}")

# Cambiar a gratis
pedido.establecer_estrategia_envio(EstrategiaEnvioGratis())
print(f"Envío gratis: ${pedido.calcular_envio()}")
```

---

## Mixins: Composición con herencia

**Mixins** son clases pequeñas que añaden funcionalidad específica:

```python
from datetime import datetime

class TimestampMixin:
    """Añade timestamps automáticos"""
    def __init__(self, *args, **kwargs):
        super().__init__(*args, **kwargs)
        self.creado_en = datetime.now()
        self.actualizado_en = datetime.now()
    
    def actualizar_timestamp(self):
        self.actualizado_en = datetime.now()

class SoftDeleteMixin:
    """Añade soft delete"""
    def __init__(self, *args, **kwargs):
        super().__init__(*args, **kwargs)
        self.eliminado = False
    
    def eliminar(self):
        self.eliminado = True
    
    def restaurar(self):
        self.eliminado = False

class SerializableMixin:
    """Añade conversión a dict"""
    def to_dict(self):
        return {k: v for k, v in self.__dict__.items() if not k.startswith('_')}

# Combinar mixins
class Producto(TimestampMixin, SoftDeleteMixin, SerializableMixin):
    def __init__(self, id, nombre, precio):
        super().__init__()
        self.id = id
        self.nombre = nombre
        self.precio = precio

# Tiene todas las funcionalidades
producto = Producto(1, 'Laptop', 999)
print(producto.creado_en)  # TimestampMixin
producto.eliminar()        # SoftDeleteMixin
print(producto.to_dict())  # SerializableMixin
```

---

## Dependency Injection

Inyecta dependencias en lugar de crearlas internamente:

```python
# ❌ MAL: Dependencias hardcodeadas
class ServicioUsuario:
    def __init__(self):
        self.db = BaseDatos()  # Acoplado a BaseDatos específica
        self.notificador = NotificadorEmail()  # Acoplado a Email
    
    def crear_usuario(self, datos):
        usuario = self.db.guardar(datos)
        self.notificador.enviar("Usuario creado")
        return usuario

# ✅ BIEN: Inyección de dependencias
class ServicioUsuario:
    def __init__(self, db, notificador):
        self.db = db  # Inyectado
        self.notificador = notificador  # Inyectado
    
    def crear_usuario(self, datos):
        usuario = self.db.guardar(datos)
        self.notificador.enviar("Usuario creado")
        return usuario

# Uso: puedes cambiar implementaciones fácilmente
db = BaseDatosPostgreSQL()
notificador = NotificadorSMS()

servicio = ServicioUsuario(db, notificador)
servicio.crear_usuario({'nombre': 'Juan'})
```

---

## Ejemplo completo: Sistema de pedidos

```python
from datetime import datetime
from typing import List

# Componentes independientes
class ValidadorPedido:
    def validar(self, pedido):
        if not pedido.productos:
            raise ValueError("Pedido vacío")
        if pedido.calcular_total() <= 0:
            raise ValueError("Total inválido")
        return True

class CalculadorEnvio:
    def calcular(self, pedido):
        if pedido.calcular_subtotal() >= 100:
            return 0  # Envío gratis
        return 10

class NotificadorPedido:
    def __init__(self, canales):
        self.canales = canales
    
    def notificar(self, pedido, mensaje):
        for canal in self.canales:
            canal.enviar(mensaje)

class RegistradorPedido:
    def registrar(self, pedido):
        print(f"Registrando pedido #{pedido.id} en base de datos")
        # Lógica de guardado...

# Clase principal con composición
class Pedido:
    def __init__(
        self,
        id: int,
        cliente,
        productos: List[dict],
        validador: ValidadorPedido,
        calculador_envio: CalculadorEnvio,
        notificador: NotificadorPedido,
        registrador: RegistradorPedido
    ):
        self.id = id
        self.cliente = cliente
        self.productos = productos
        self.estado = 'pendiente'
        self.fecha_creacion = datetime.now()
        
        # Composición: inyección de dependencias
        self.validador = validador
        self.calculador_envio = calculador_envio
        self.notificador = notificador
        self.registrador = registrador
    
    def calcular_subtotal(self):
        return sum(p['precio'] * p['cantidad'] for p in self.productos)
    
    def calcular_envio(self):
        return self.calculador_envio.calcular(self)
    
    def calcular_total(self):
        return self.calcular_subtotal() + self.calcular_envio()
    
    def procesar(self):
        """Procesa el pedido usando los componentes"""
        try:
            # Validar
            self.validador.validar(self)
            
            # Procesar
            self.estado = 'procesando'
            print(f"Procesando pedido #{self.id}")
            
            # Registrar
            self.registrador.registrar(self)
            
            # Confirmar
            self.estado = 'confirmado'
            
            # Notificar
            self.notificador.notificar(
                self,
                f"Pedido #{self.id} confirmado. Total: ${self.calcular_total()}"
            )
            
            return True
        except Exception as e:
            self.estado = 'error'
            print(f"Error procesando pedido: {e}")
            return False

# Uso
class EnviadorEmail:
    def enviar(self, mensaje):
        print(f"📧 {mensaje}")

class EnviadorSMS:
    def enviar(self, mensaje):
        print(f"📱 {mensaje}")

# Configurar componentes
validador = ValidadorPedido()
calculador = CalculadorEnvio()
notificador = NotificadorPedido([EnviadorEmail(), EnviadorSMS()])
registrador = RegistradorPedido()

# Crear pedido con composición
pedido = Pedido(
    id=1,
    cliente={'nombre': 'Ana'},
    productos=[
        {'nombre': 'Laptop', 'precio': 999, 'cantidad': 1},
        {'nombre': 'Mouse', 'precio': 25, 'cantidad': 2}
    ],
    validador=validador,
    calculador_envio=calculador,
    notificador=notificador,
    registrador=registrador
)

# Procesar
pedido.procesar()
```

---

## Comparación directa

| Aspecto | Herencia | Composición |
|---------|----------|-------------|
| Relación | "Es un" | "Tiene un" |
| Acoplamiento | Alto | Bajo |
| Flexibilidad | Baja | Alta |
| Reutilización | Solo jerarquía | Cualquier combinación |
| Testing | Difícil | Fácil |
| Cambios | Afecta toda jerarquía | Solo componente |

---

## Reglas prácticas

### ✅ Prefiere composición cuando:
- Necesitas flexibilidad
- Quieres combinar comportamientos
- Facilitar testing
- Evitar jerarquías profundas

### ✅ Usa herencia cuando:
- Hay una relación clara "es un"
- Compartes interfaz común (polimorfismo)
- Jerarquía es simple (1-2 niveles)

---

## Principio de composición sobre herencia

**"Favor composition over inheritance"** - Gang of Four

```python
# ❌ Herencia: rígido
class UsuarioConNotificaciones(Usuario):
    pass

class UsuarioConLogging(Usuario):
    pass

# ¿Usuario con ambas? Herencia múltiple compleja

# ✅ Composición: flexible
class Usuario:
    def __init__(self, notificador, logger):
        self.notificador = notificador
        self.logger = logger
    
    def hacer_algo(self):
        self.logger.log("Acción realizada")
        self.notificador.notificar("Acción completada")

# Fácil combinar
usuario = Usuario(
    notificador=Notificador(),
    logger=Logger()
)
```

---

## Conclusión

Has aprendido:

- ✅ Diferencia entre herencia y composición
- ✅ Cuándo usar cada una
- ✅ Problemas de herencia excesiva
- ✅ Ventajas de la composición
- ✅ Patrón Strategy
- ✅ Mixins
- ✅ Dependency Injection
- ✅ Ejemplos prácticos

**Siguiente:** [Modelado de dominio](./16-modelado-dominio.md)

---

## Recursos adicionales

- **[Composition over Inheritance](https://en.wikipedia.org/wiki/Composition_over_inheritance)** - Wikipedia
- **[Design Patterns](https://refactoring.guru/design-patterns)** - Patrones de diseño

La composición te da más flexibilidad. Úsala sabiamente. 🎯

# Patrones de diseño en backend

> **Soluciones probadas para problemas comunes**

---

## ¿Qué es un patrón de diseño?

Un **patrón de diseño** es una solución reutilizable para un problema común en programación.

**Analogía:** Los patrones son como recetas de cocina - describen cómo resolver un problema específico de forma efectiva.

**No son:**
- Código que copiar y pegar
- Reglas obligatorias

**Son:**
- Guías y buenas prácticas
- Soluciones que funcionan

---

## Clasificación de patrones

### 🔨 Patrones creacionales
Cómo **crear** objetos

### 🔧 Patrones estructurales
Cómo **organizar** objetos

### ⚙️ Patrones de comportamiento
Cómo los objetos **se comunican**

---

## 1. Patrón Singleton

**Problema:** Necesitas que una clase tenga **una sola instancia** en toda la aplicación.

**Ejemplo:** Conexión a base de datos, configuración global, logger.

### Implementación

```python
class Database:
    """Singleton para conexión a base de datos"""
    
    _instancia = None
    
    def __new__(cls):
        """Sobrescribe __new__ para controlar la creación"""
        if cls._instancia is None:
            print("Creando primera instancia de Database")
            cls._instancia = super().__new__(cls)
            cls._instancia._inicializado = False
        return cls._instancia
    
    def __init__(self):
        """Inicializa solo la primera vez"""
        if not self._inicializado:
            self.conexion = "postgresql://localhost/tienda"
            self._inicializado = True
            print("Database inicializada")
    
    def ejecutar_query(self, query):
        """Ejecuta una query"""
        print(f"Ejecutando: {query}")
        return f"Resultado de: {query}"

# Uso
db1 = Database()
db2 = Database()

print(db1 is db2)  # True - misma instancia
```

**Salida:**
```
Creando primera instancia de Database
Database inicializada
True
```

### Alternativa moderna con decorador

```python
def singleton(cls):
    """Decorador para convertir una clase en singleton"""
    instancias = {}
    
    def obtener_instancia(*args, **kwargs):
        if cls not in instancias:
            instancias[cls] = cls(*args, **kwargs)
        return instancias[cls]
    
    return obtener_instancia

@singleton
class Config:
    """Configuración global de la aplicación"""
    
    def __init__(self):
        self.debug = True
        self.db_url = "postgresql://localhost/tienda"
        self.secret_key = "mi-secreto"
    
    def cargar_desde_archivo(self, archivo):
        print(f"Cargando config desde {archivo}")

# Uso
config1 = Config()
config2 = Config()

print(config1 is config2)  # True
```

### Singleton en aplicación backend

```python
class AppConfig:
    """Configuración singleton para la aplicación"""
    _instancia = None
    
    def __new__(cls):
        if cls._instancia is None:
            cls._instancia = super().__new__(cls)
        return cls._instancia
    
    def __init__(self):
        if not hasattr(self, 'inicializado'):
            self.API_KEY = 'tu-api-key'
            self.DB_URL = 'sqlite:///tienda.db'
            self.MAX_INTENTOS = 3
            self.inicializado = True

# Uso en API
from http.server import BaseHTTPRequestHandler
import json

config = AppConfig()

class APIHandler(BaseHTTPRequestHandler):
    def do_GET(self):
        if self.path == '/config':
            respuesta = {
                'db_url': config.DB_URL,
                'max_intentos': config.MAX_INTENTOS
            }
            self.send_response(200)
            self.send_header('Content-type', 'application/json')
            self.end_headers()
            self.wfile.write(json.dumps(respuesta).encode())
```

---

## 2. Patrón Factory

**Problema:** Necesitas crear objetos sin especificar su clase exacta.

**Ejemplo:** Crear diferentes tipos de notificaciones, pagos, usuarios.

### Factory simple

```python
class Notificacion:
    """Clase base para notificaciones"""
    def enviar(self, mensaje):
        raise NotImplementedError

class NotificacionEmail(Notificacion):
    def enviar(self, mensaje):
        print(f"📧 Email: {mensaje}")

class NotificacionSMS(Notificacion):
    def enviar(self, mensaje):
        print(f"📱 SMS: {mensaje}")

class NotificacionPush(Notificacion):
    def enviar(self, mensaje):
        print(f"🔔 Push: {mensaje}")

class NotificacionFactory:
    """Factory para crear notificaciones"""
    
    @staticmethod
    def crear_notificacion(tipo: str) -> Notificacion:
        """Crea notificación según tipo"""
        notificaciones = {
            'email': NotificacionEmail,
            'sms': NotificacionSMS,
            'push': NotificacionPush
        }
        
        clase_notificacion = notificaciones.get(tipo.lower())
        if not clase_notificacion:
            raise ValueError(f"Tipo de notificación no soportado: {tipo}")
        
        return clase_notificacion()

# Uso
factory = NotificacionFactory()

notif_email = factory.crear_notificacion('email')
notif_email.enviar('Pedido confirmado')

notif_sms = factory.crear_notificacion('sms')
notif_sms.enviar('Tu código es 1234')
```

**Salida:**
```
📧 Email: Pedido confirmado
📱 SMS: Tu código es 1234
```

### Factory con parámetros

```python
class ProcesadorPago:
    """Clase base para procesadores de pago"""
    def procesar(self, monto):
        raise NotImplementedError

class ProcesadorTarjeta(ProcesadorPago):
    def __init__(self, numero_tarjeta):
        self.numero_tarjeta = numero_tarjeta
    
    def procesar(self, monto):
        print(f"💳 Procesando ${monto} con tarjeta ****{self.numero_tarjeta[-4:]}")
        return True

class ProcesadorPaypal(ProcesadorPago):
    def __init__(self, email):
        self.email = email
    
    def procesar(self, monto):
        print(f"💰 Procesando ${monto} vía PayPal ({self.email})")
        return True

class ProcesadorTransferencia(ProcesadorPago):
    def __init__(self, cuenta_bancaria):
        self.cuenta_bancaria = cuenta_bancaria
    
    def procesar(self, monto):
        print(f"🏦 Procesando ${monto} vía transferencia a {self.cuenta_bancaria}")
        return True

class PagoFactory:
    """Factory para crear procesadores de pago"""
    
    @staticmethod
    def crear_procesador(tipo: str, **kwargs) -> ProcesadorPago:
        """Crea procesador según tipo y parámetros"""
        if tipo == 'tarjeta':
            return ProcesadorTarjeta(kwargs['numero_tarjeta'])
        elif tipo == 'paypal':
            return ProcesadorPaypal(kwargs['email'])
        elif tipo == 'transferencia':
            return ProcesadorTransferencia(kwargs['cuenta_bancaria'])
        else:
            raise ValueError(f"Tipo de pago no soportado: {tipo}")

# Uso
factory = PagoFactory()

procesador1 = factory.crear_procesador('tarjeta', numero_tarjeta='1234567890123456')
procesador1.procesar(100)

procesador2 = factory.crear_procesador('paypal', email='usuario@ejemplo.com')
procesador2.procesar(50)
```

---

## 3. Patrón Repository

**Problema:** Separar la lógica de negocio del acceso a datos.

**Beneficio:** Puedes cambiar la base de datos sin tocar tu lógica.

### Implementación

```python
from typing import List, Optional
from abc import ABC, abstractmethod

class Usuario:
    """Entidad Usuario"""
    def __init__(self, id: int, nombre: str, email: str):
        self.id = id
        self.nombre = nombre
        self.email = email
    
    def __str__(self):
        return f"Usuario(id={self.id}, nombre={self.nombre})"

class RepositorioUsuarios(ABC):
    """Interfaz para repositorio de usuarios"""
    
    @abstractmethod
    def obtener_por_id(self, id: int) -> Optional[Usuario]:
        pass
    
    @abstractmethod
    def obtener_todos(self) -> List[Usuario]:
        pass
    
    @abstractmethod
    def guardar(self, usuario: Usuario) -> Usuario:
        pass
    
    @abstractmethod
    def eliminar(self, id: int) -> bool:
        pass

class RepositorioUsuariosMemoria(RepositorioUsuarios):
    """Repositorio en memoria (para desarrollo/testing)"""
    
    def __init__(self):
        self.usuarios = {}
        self.siguiente_id = 1
    
    def obtener_por_id(self, id: int) -> Optional[Usuario]:
        return self.usuarios.get(id)
    
    def obtener_todos(self) -> List[Usuario]:
        return list(self.usuarios.values())
    
    def guardar(self, usuario: Usuario) -> Usuario:
        if usuario.id is None:
            usuario.id = self.siguiente_id
            self.siguiente_id += 1
        self.usuarios[usuario.id] = usuario
        return usuario
    
    def eliminar(self, id: int) -> bool:
        if id in self.usuarios:
            del self.usuarios[id]
            return True
        return False

class RepositorioUsuariosSQL(RepositorioUsuarios):
    """Repositorio con base de datos SQL (simulado)"""
    
    def __init__(self, conexion):
        self.conexion = conexion
    
    def obtener_por_id(self, id: int) -> Optional[Usuario]:
        print(f"SELECT * FROM usuarios WHERE id = {id}")
        # Simular query
        return Usuario(id, f"Usuario {id}", f"usuario{id}@ejemplo.com")
    
    def obtener_todos(self) -> List[Usuario]:
        print("SELECT * FROM usuarios")
        # Simular query
        return [
            Usuario(1, "Ana", "ana@ejemplo.com"),
            Usuario(2, "Luis", "luis@ejemplo.com")
        ]
    
    def guardar(self, usuario: Usuario) -> Usuario:
        if usuario.id:
            print(f"UPDATE usuarios SET nombre='{usuario.nombre}' WHERE id={usuario.id}")
        else:
            print(f"INSERT INTO usuarios (nombre, email) VALUES ('{usuario.nombre}', '{usuario.email}')")
        return usuario
    
    def eliminar(self, id: int) -> bool:
        print(f"DELETE FROM usuarios WHERE id = {id}")
        return True

# Uso
repo_memoria = RepositorioUsuariosMemoria()

# Guardar usuarios
usuario1 = Usuario(None, "Ana", "ana@ejemplo.com")
usuario1 = repo_memoria.guardar(usuario1)
print(f"Usuario guardado: {usuario1}")

usuario2 = Usuario(None, "Luis", "luis@ejemplo.com")
usuario2 = repo_memoria.guardar(usuario2)

# Obtener todos
usuarios = repo_memoria.obtener_todos()
print(f"Total usuarios: {len(usuarios)}")

# Obtener por ID
usuario = repo_memoria.obtener_por_id(1)
print(f"Usuario encontrado: {usuario}")
```

### Repository en Flask

```python
from flask import Flask, jsonify, request

app = Flask(__name__)

# Inyectar repositorio (en producción, usar SQL)
repo = RepositorioUsuariosMemoria()

@app.route('/api/usuarios', methods=['GET'])
def listar_usuarios():
    usuarios = repo.obtener_todos()
    return jsonify([
        {'id': u.id, 'nombre': u.nombre, 'email': u.email}
        for u in usuarios
    ])

@app.route('/api/usuarios/<int:id>', methods=['GET'])
def obtener_usuario(id):
    usuario = repo.obtener_por_id(id)
    if not usuario:
        return jsonify({'error': 'Usuario no encontrado'}), 404
    
    return jsonify({
        'id': usuario.id,
        'nombre': usuario.nombre,
        'email': usuario.email
    })

@app.route('/api/usuarios', methods=['POST'])
def crear_usuario():
    datos = request.json
    usuario = Usuario(None, datos['nombre'], datos['email'])
    usuario = repo.guardar(usuario)
    
    return jsonify({
        'id': usuario.id,
        'nombre': usuario.nombre,
        'email': usuario.email
    }), 201

@app.route('/api/usuarios/<int:id>', methods=['DELETE'])
def eliminar_usuario(id):
    exito = repo.eliminar(id)
    if not exito:
        return jsonify({'error': 'Usuario no encontrado'}), 404
    
    return '', 204
```

---

## 4. Patrón Strategy

**Problema:** Necesitas cambiar un algoritmo/comportamiento en tiempo de ejecución.

**Ejemplo:** Diferentes estrategias de envío, descuento, autenticación.

### Implementación

```python
from abc import ABC, abstractmethod

class EstrategiaDescuento(ABC):
    """Interfaz para estrategias de descuento"""
    
    @abstractmethod
    def calcular_descuento(self, monto: float) -> float:
        pass

class SinDescuento(EstrategiaDescuento):
    def calcular_descuento(self, monto: float) -> float:
        return 0

class DescuentoPorcentaje(EstrategiaDescuento):
    def __init__(self, porcentaje: float):
        self.porcentaje = porcentaje
    
    def calcular_descuento(self, monto: float) -> float:
        return monto * (self.porcentaje / 100)

class DescuentoFijo(EstrategiaDescuento):
    def __init__(self, cantidad: float):
        self.cantidad = cantidad
    
    def calcular_descuento(self, monto: float) -> float:
        return min(self.cantidad, monto)

class DescuentoClienteVIP(EstrategiaDescuento):
    def calcular_descuento(self, monto: float) -> float:
        if monto >= 500:
            return monto * 0.25  # 25% descuento
        elif monto >= 200:
            return monto * 0.15  # 15% descuento
        else:
            return monto * 0.10  # 10% descuento

class Pedido:
    """Pedido con estrategia de descuento"""
    
    def __init__(self, items: list, estrategia_descuento: EstrategiaDescuento):
        self.items = items
        self.estrategia_descuento = estrategia_descuento
    
    def calcular_subtotal(self) -> float:
        return sum(self.items)
    
    def calcular_descuento(self) -> float:
        subtotal = self.calcular_subtotal()
        return self.estrategia_descuento.calcular_descuento(subtotal)
    
    def calcular_total(self) -> float:
        return self.calcular_subtotal() - self.calcular_descuento()
    
    def cambiar_estrategia(self, nueva_estrategia: EstrategiaDescuento):
        """Cambia la estrategia de descuento"""
        self.estrategia_descuento = nueva_estrategia

# Uso
items = [100, 200, 150]  # Precios de productos

# Sin descuento
pedido = Pedido(items, SinDescuento())
print(f"Sin descuento: ${pedido.calcular_total()}")

# Descuento porcentual
pedido.cambiar_estrategia(DescuentoPorcentaje(10))
print(f"Con 10% descuento: ${pedido.calcular_total()}")

# Descuento fijo
pedido.cambiar_estrategia(DescuentoFijo(50))
print(f"Con $50 descuento: ${pedido.calcular_total()}")

# Cliente VIP
pedido.cambiar_estrategia(DescuentoClienteVIP())
print(f"Descuento VIP: ${pedido.calcular_total()}")
```

**Salida:**
```
Sin descuento: $450.0
Con 10% descuento: $405.0
Con $50 descuento: $400.0
Descuento VIP: $337.5
```

### Strategy para envío

```python
class EstrategiaEnvio(ABC):
    @abstractmethod
    def calcular_costo(self, peso: float, distancia: float) -> float:
        pass
    
    @abstractmethod
    def tiempo_estimado(self) -> str:
        pass

class EnvioEstandar(EstrategiaEnvio):
    def calcular_costo(self, peso: float, distancia: float) -> float:
        return 5 + (peso * 0.5)
    
    def tiempo_estimado(self) -> str:
        return "5-7 días hábiles"

class EnvioExpress(EstrategiaEnvio):
    def calcular_costo(self, peso: float, distancia: float) -> float:
        return 15 + (peso * 1.0) + (distancia * 0.1)
    
    def tiempo_estimado(self) -> str:
        return "1-2 días hábiles"

class EnvioInternacional(EstrategiaEnvio):
    def calcular_costo(self, peso: float, distancia: float) -> float:
        return 50 + (peso * 2.0) + (distancia * 0.5)
    
    def tiempo_estimado(self) -> str:
        return "10-15 días hábiles"

# Uso
peso = 2.5  # kg
distancia = 500  # km

estrategias = [
    EnvioEstandar(),
    EnvioExpress(),
    EnvioInternacional()
]

for estrategia in estrategias:
    costo = estrategia.calcular_costo(peso, distancia)
    tiempo = estrategia.tiempo_estimado()
    print(f"{estrategia.__class__.__name__}: ${costo} - {tiempo}")
```

---

## 5. Patrón Observer

**Problema:** Un objeto necesita notificar a múltiples objetos cuando cambia su estado.

**Ejemplo:** Sistema de eventos, notificaciones, actualizaciones en tiempo real.

### Implementación

```python
from typing import List

class Observador(ABC):
    """Interfaz para observadores"""
    
    @abstractmethod
    def actualizar(self, evento: str, datos: dict):
        pass

class ObservadorEmail(Observador):
    def __init__(self, email: str):
        self.email = email
    
    def actualizar(self, evento: str, datos: dict):
        print(f"📧 Enviando email a {self.email}: {evento}")
        print(f"   Datos: {datos}")

class ObservadorSMS(Observador):
    def __init__(self, telefono: str):
        self.telefono = telefono
    
    def actualizar(self, evento: str, datos: dict):
        print(f"📱 Enviando SMS a {self.telefono}: {evento}")

class ObservadorLog(Observador):
    def actualizar(self, evento: str, datos: dict):
        print(f"📝 Log: {evento} - {datos}")

class Sujeto:
    """Sujeto que notifica a observadores"""
    
    def __init__(self):
        self._observadores: List[Observador] = []
    
    def agregar_observador(self, observador: Observador):
        """Suscribe un observador"""
        if observador not in self._observadores:
            self._observadores.append(observador)
    
    def eliminar_observador(self, observador: Observador):
        """Desuscribe un observador"""
        if observador in self._observadores:
            self._observadores.remove(observador)
    
    def notificar(self, evento: str, datos: dict = None):
        """Notifica a todos los observadores"""
        datos = datos or {}
        for observador in self._observadores:
            observador.actualizar(evento, datos)

class Pedido(Sujeto):
    """Pedido que notifica cambios de estado"""
    
    def __init__(self, id: int, cliente: str):
        super().__init__()
        self.id = id
        self.cliente = cliente
        self.estado = 'pendiente'
    
    def procesar(self):
        """Procesa el pedido"""
        self.estado = 'procesando'
        self.notificar('pedido_procesado', {
            'pedido_id': self.id,
            'cliente': self.cliente
        })
    
    def enviar(self):
        """Marca como enviado"""
        self.estado = 'enviado'
        self.notificar('pedido_enviado', {
            'pedido_id': self.id,
            'cliente': self.cliente
        })
    
    def entregar(self):
        """Marca como entregado"""
        self.estado = 'entregado'
        self.notificar('pedido_entregado', {
            'pedido_id': self.id,
            'cliente': self.cliente
        })

# Uso
pedido = Pedido(1, 'Ana García')

# Suscribir observadores
pedido.agregar_observador(ObservadorEmail('ana@ejemplo.com'))
pedido.agregar_observador(ObservadorSMS('+34666555444'))
pedido.agregar_observador(ObservadorLog())

# Cambiar estados (notifica automáticamente)
print("=== Procesando pedido ===")
pedido.procesar()

print("\n=== Enviando pedido ===")
pedido.enviar()

print("\n=== Entregando pedido ===")
pedido.entregar()
```

**Salida:**
```
=== Procesando pedido ===
📧 Enviando email a ana@ejemplo.com: pedido_procesado
   Datos: {'pedido_id': 1, 'cliente': 'Ana García'}
📱 Enviando SMS a +34666555444: pedido_procesado
📝 Log: pedido_procesado - {'pedido_id': 1, 'cliente': 'Ana García'}

=== Enviando pedido ===
📧 Enviando email a ana@ejemplo.com: pedido_enviado
   Datos: {'pedido_id': 1, 'cliente': 'Ana García'}
📱 Enviando SMS a +34666555444: pedido_enviado
📝 Log: pedido_enviado - {'pedido_id': 1, 'cliente': 'Ana García'}

=== Entregando pedido ===
📧 Enviando email a ana@ejemplo.com: pedido_entregado
   Datos: {'pedido_id': 1, 'cliente': 'Ana García'}
📱 Enviando SMS a +34666555444: pedido_entregado
📝 Log: pedido_entregado - {'pedido_id': 1, 'cliente': 'Ana García'}
```

---

## 6. Patrón Decorator

**Problema:** Añadir funcionalidad a objetos sin modificar su clase.

**Ejemplo:** Añadir características a productos, middleware en Flask.

### Implementación

```python
class Cafe:
    """Componente base"""
    def obtener_descripcion(self) -> str:
        return "Café"
    
    def obtener_costo(self) -> float:
        return 2.0

class DecoradorCafe(Cafe):
    """Decorador base"""
    def __init__(self, cafe: Cafe):
        self.cafe = cafe

class ConLeche(DecoradorCafe):
    def obtener_descripcion(self) -> str:
        return self.cafe.obtener_descripcion() + " + Leche"
    
    def obtener_costo(self) -> float:
        return self.cafe.obtener_costo() + 0.5

class ConCrema(DecoradorCafe):
    def obtener_descripcion(self) -> str:
        return self.cafe.obtener_descripcion() + " + Crema"
    
    def obtener_costo(self) -> float:
        return self.cafe.obtener_costo() + 0.7

class ConChocolate(DecoradorCafe):
    def obtener_descripcion(self) -> str:
        return self.cafe.obtener_descripcion() + " + Chocolate"
    
    def obtener_costo(self) -> float:
        return self.cafe.obtener_costo() + 1.0

# Uso
cafe = Cafe()
print(f"{cafe.obtener_descripcion()}: ${cafe.obtener_costo()}")

# Café con leche
cafe_con_leche = ConLeche(cafe)
print(f"{cafe_con_leche.obtener_descripcion()}: ${cafe_con_leche.obtener_costo()}")

# Café con leche y crema
cafe_especial = ConCrema(ConLeche(cafe))
print(f"{cafe_especial.obtener_descripcion()}: ${cafe_especial.obtener_costo()}")

# Café con todo
cafe_completo = ConChocolate(ConCrema(ConLeche(cafe)))
print(f"{cafe_completo.obtener_descripcion()}: ${cafe_completo.obtener_costo()}")
```

**Salida:**
```
Café: $2.0
Café + Leche: $2.5
Café + Leche + Crema: $3.2
Café + Leche + Crema + Chocolate: $4.2
```

### Decorator en Flask (middleware)

```python
from functools import wraps
from flask import request, jsonify

def requiere_autenticacion(f):
    """Decorador para proteger rutas"""
    @wraps(f)
    def decorado(*args, **kwargs):
        token = request.headers.get('Authorization')
        if not token:
            return jsonify({'error': 'Token requerido'}), 401
        
        # Validar token (simplificado)
        if token != 'Bearer mi-token-secreto':
            return jsonify({'error': 'Token inválido'}), 401
        
        return f(*args, **kwargs)
    return decorado

def requiere_admin(f):
    """Decorador para rutas de administrador"""
    @wraps(f)
    def decorado(*args, **kwargs):
        rol = request.headers.get('X-User-Role')
        if rol != 'admin':
            return jsonify({'error': 'Acceso denegado'}), 403
        
        return f(*args, **kwargs)
    return decorado

@app.route('/api/usuarios')
@requiere_autenticacion
def listar_usuarios():
    return jsonify({'usuarios': ['Ana', 'Luis']})

@app.route('/api/admin/config')
@requiere_autenticacion
@requiere_admin
def configuracion_admin():
    return jsonify({'config': 'Configuración de administrador'})
```

---

## Cuándo usar cada patrón

| Patrón | Cuándo usarlo |
|--------|---------------|
| **Singleton** | Una sola instancia global (DB, Config) |
| **Factory** | Crear objetos sin especificar clase exacta |
| **Repository** | Separar lógica de negocio del acceso a datos |
| **Strategy** | Cambiar algoritmos en tiempo de ejecución |
| **Observer** | Notificar cambios a múltiples objetos |
| **Decorator** | Añadir funcionalidad sin modificar clases |

---

## Ejercicio práctico: Sistema completo

```python
# Combinar patrones en un sistema real

# Singleton para config
@singleton
class Config:
    def __init__(self):
        self.db_url = "postgresql://localhost/tienda"
        self.smtp_server = "smtp.ejemplo.com"

# Repository para productos
class RepositorioProductos:
    def __init__(self):
        self.productos = {}
    
    def guardar(self, producto):
        self.productos[producto.id] = producto

# Factory para notificaciones
class NotificacionFactory:
    @staticmethod
    def crear(tipo):
        if tipo == 'email':
            return NotificacionEmail()
        elif tipo == 'sms':
            return NotificacionSMS()

# Strategy para precios
class EstrategiaPrecio(ABC):
    @abstractmethod
    def calcular_precio_final(self, precio_base):
        pass

# Observer para eventos
class SistemaPedidos(Sujeto):
    def nuevo_pedido(self, pedido):
        self.notificar('nuevo_pedido', {'pedido_id': pedido.id})

# Usar todos juntos
config = Config()
repo = RepositorioProductos()
sistema = SistemaPedidos()
sistema.agregar_observador(NotificacionFactory.crear('email'))
```

---

## Conclusión

Has aprendido:

- ✅ Qué son los patrones de diseño
- ✅ Singleton: Una única instancia
- ✅ Factory: Crear objetos de forma flexible
- ✅ Repository: Separar datos de lógica
- ✅ Strategy: Cambiar comportamientos
- ✅ Observer: Sistema de eventos
- ✅ Decorator: Añadir funcionalidad
- ✅ Cuándo usar cada patrón

**Siguiente:** [SOLID en backend](./18-solid-backend.md)

---

## Recursos adicionales

- **[Refactoring Guru](https://refactoring.guru/es/design-patterns)** - Patrones explicados
- **[Python Design Patterns](https://python-patterns.guide/)** - Patrones en Python

Los patrones no son obligatorios, pero resuelven problemas comunes. 🎯

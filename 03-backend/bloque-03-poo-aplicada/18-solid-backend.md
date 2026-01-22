# Principios SOLID en backend

> **Escribe código mantenible, escalable y profesional**

---

## ¿Qué es SOLID?

**SOLID** son 5 principios de diseño orientado a objetos que hacen tu código:
- ✅ Más fácil de mantener
- ✅ Más fácil de extender
- ✅ Más fácil de testear
- ✅ Menos propenso a bugs

**Creado por:** Robert C. Martin (Uncle Bob)

---

## Los 5 principios

| Letra | Principio | Resumen |
|-------|-----------|---------|
| **S** | Single Responsibility | Una clase, una responsabilidad |
| **O** | Open/Closed | Abierto a extensión, cerrado a modificación |
| **L** | Liskov Substitution | Las subclases deben ser sustituibles |
| **I** | Interface Segregation | Interfaces pequeñas y específicas |
| **D** | Dependency Inversion | Depende de abstracciones, no de implementaciones |

---

## 1. Single Responsibility Principle (SRP)

**Principio:** Una clase debe tener **una sola razón para cambiar**.

**En otras palabras:** Una clase = Una responsabilidad.

### ❌ Violación de SRP

```python
class Usuario:
    """Clase con múltiples responsabilidades"""
    
    def __init__(self, nombre, email):
        self.nombre = nombre
        self.email = email
    
    def guardar_en_bd(self):
        """❌ Responsabilidad: Persistencia"""
        print(f"INSERT INTO usuarios (nombre, email) VALUES ('{self.nombre}', '{self.email}')")
    
    def enviar_email_bienvenida(self):
        """❌ Responsabilidad: Notificaciones"""
        print(f"Enviando email de bienvenida a {self.email}")
    
    def generar_reporte(self):
        """❌ Responsabilidad: Reporting"""
        print(f"Generando reporte para {self.nombre}")

# Problema: Si cambia el formato de BD, email o reportes, modificas la misma clase
```

**Problemas:**
- Si cambia la BD, modificas `Usuario`
- Si cambia el servicio de email, modificas `Usuario`
- Si cambia el formato de reporte, modificas `Usuario`

### ✅ Aplicando SRP

```python
class Usuario:
    """Responsabilidad ÚNICA: Representar un usuario"""
    
    def __init__(self, nombre: str, email: str):
        self.nombre = nombre
        self.email = email
    
    def __str__(self):
        return f"Usuario: {self.nombre} ({self.email})"

class RepositorioUsuarios:
    """Responsabilidad ÚNICA: Persistencia"""
    
    def guardar(self, usuario: Usuario):
        print(f"Guardando usuario en BD: {usuario.nombre}")
        # Lógica de base de datos
    
    def obtener_por_email(self, email: str) -> Usuario:
        print(f"Buscando usuario con email: {email}")
        # Lógica de consulta
        return Usuario("Ejemplo", email)

class ServicioEmail:
    """Responsabilidad ÚNICA: Enviar emails"""
    
    def enviar_bienvenida(self, usuario: Usuario):
        print(f"📧 Enviando email de bienvenida a {usuario.email}")
        # Lógica de envío de email

class GeneradorReportes:
    """Responsabilidad ÚNICA: Generar reportes"""
    
    def generar_reporte_usuario(self, usuario: Usuario):
        print(f"📊 Generando reporte para {usuario.nombre}")
        # Lógica de reporte

# Uso
usuario = Usuario("Ana García", "ana@ejemplo.com")

repo = RepositorioUsuarios()
repo.guardar(usuario)

email_service = ServicioEmail()
email_service.enviar_bienvenida(usuario)

reportes = GeneradorReportes()
reportes.generar_reporte_usuario(usuario)
```

**Beneficios:**
- ✅ Cada clase tiene una sola responsabilidad
- ✅ Cambios aislados (modificar BD no afecta emails)
- ✅ Más fácil de testear
- ✅ Más reutilizable

### Ejemplo en API REST

```python
from http.server import BaseHTTPRequestHandler
import json

# Servicios separados
repo_usuarios = RepositorioUsuarios()
servicio_email = ServicioEmail()

class APIHandler(BaseHTTPRequestHandler):
    def do_POST(self):
        if self.path == '/api/usuarios':
            # Leer datos
            content_length = int(self.headers.get('Content-Length', 0))
            body = self.rfile.read(content_length)
            datos = json.loads(body.decode('utf-8'))
            
            # 1. Crear usuario
            usuario = Usuario(datos['nombre'], datos['email'])
            
            # 2. Guardar en BD
            repo_usuarios.guardar(usuario)
            
            # 3. Enviar email
            servicio_email.enviar_bienvenida(usuario)
            
            # Responder
            self.send_response(201)
            self.send_header('Content-type', 'application/json')
            self.end_headers()
            respuesta = {'mensaje': 'Usuario creado'}
            self.wfile.write(json.dumps(respuesta).encode())
```

---

## 2. Open/Closed Principle (OCP)

**Principio:** Las clases deben estar **abiertas a extensión** pero **cerradas a modificación**.

**En otras palabras:** Añade funcionalidad sin cambiar código existente.

### ❌ Violación de OCP

```python
class CalculadoraDescuentos:
    """Cada vez que hay un nuevo tipo de descuento, modifico esta clase"""
    
    def calcular_descuento(self, pedido, tipo_descuento):
        if tipo_descuento == 'porcentaje':
            return pedido.total * 0.10
        elif tipo_descuento == 'fijo':
            return 50
        elif tipo_descuento == 'vip':
            return pedido.total * 0.25
        # ❌ Si añado 'estudiante', modifico esta clase
        elif tipo_descuento == 'estudiante':
            return pedido.total * 0.15
        else:
            return 0

# Problema: Cada nuevo descuento requiere modificar la clase
```

### ✅ Aplicando OCP

```python
from abc import ABC, abstractmethod

class Descuento(ABC):
    """Clase base abstracta"""
    
    @abstractmethod
    def calcular(self, pedido) -> float:
        pass

class DescuentoPorcentaje(Descuento):
    def __init__(self, porcentaje: float):
        self.porcentaje = porcentaje
    
    def calcular(self, pedido) -> float:
        return pedido.total * (self.porcentaje / 100)

class DescuentoFijo(Descuento):
    def __init__(self, cantidad: float):
        self.cantidad = cantidad
    
    def calcular(self, pedido) -> float:
        return min(self.cantidad, pedido.total)

class DescuentoVIP(Descuento):
    def calcular(self, pedido) -> float:
        if pedido.total >= 500:
            return pedido.total * 0.25
        return pedido.total * 0.15

# ✅ Añadir nuevo descuento SIN modificar código existente
class DescuentoEstudiante(Descuento):
    def calcular(self, pedido) -> float:
        return pedido.total * 0.15

class Pedido:
    def __init__(self, total: float):
        self.total = total
    
    def aplicar_descuento(self, descuento: Descuento) -> float:
        """Aplica cualquier tipo de descuento"""
        descuento_calculado = descuento.calcular(self)
        return self.total - descuento_calculado

# Uso
pedido = Pedido(1000)

print(f"Con descuento 10%: ${pedido.aplicar_descuento(DescuentoPorcentaje(10))}")
print(f"Con descuento fijo: ${pedido.aplicar_descuento(DescuentoFijo(100))}")
print(f"Con descuento VIP: ${pedido.aplicar_descuento(DescuentoVIP())}")
print(f"Con descuento estudiante: ${pedido.aplicar_descuento(DescuentoEstudiante())}")
```

**Salida:**
```
Con descuento 10%: $900.0
Con descuento fijo: $900.0
Con descuento VIP: $750.0
Con descuento estudiante: $850.0
```

**Beneficios:**
- ✅ Añado `DescuentoEstudiante` sin tocar código existente
- ✅ Menos riesgo de introducir bugs
- ✅ Más mantenible

---

## 3. Liskov Substitution Principle (LSP)

**Principio:** Las subclases deben ser **sustituibles** por su clase base sin romper la funcionalidad.

**En otras palabras:** Si heredas, no rompas el contrato de la clase padre.

### ❌ Violación de LSP

```python
class Ave:
    def volar(self):
        print("🦅 Volando")

class Aguila(Ave):
    def volar(self):
        print("🦅 Águila volando")  # ✅ OK

class Pinguino(Ave):
    def volar(self):
        raise Exception("Los pingüinos no pueden volar")  # ❌ Rompe LSP

# Problema: Pinguino no puede sustituir a Ave
def hacer_volar_ave(ave: Ave):
    ave.volar()  # ❌ Falla si ave es Pingüino

aguila = Aguila()
hacer_volar_ave(aguila)  # ✅ OK

pinguino = Pinguino()
hacer_volar_ave(pinguino)  # ❌ Exception!
```

### ✅ Aplicando LSP

```python
class Ave(ABC):
    """Comportamiento común de todas las aves"""
    
    @abstractmethod
    def moverse(self):
        pass

class AveVoladora(Ave):
    """Aves que pueden volar"""
    
    def moverse(self):
        self.volar()
    
    def volar(self):
        print("🦅 Volando")

class AveNadadora(Ave):
    """Aves que nadan"""
    
    def moverse(self):
        self.nadar()
    
    def nadar(self):
        print("🐧 Nadando")

class Aguila(AveVoladora):
    def volar(self):
        print("🦅 Águila volando alto")

class Pinguino(AveNadadora):
    def nadar(self):
        print("🐧 Pingüino nadando rápido")

# Ahora funciona correctamente
def hacer_mover_ave(ave: Ave):
    ave.moverse()

aguila = Aguila()
pinguino = Pinguino()

hacer_mover_ave(aguila)    # ✅ Águila volando
hacer_mover_ave(pinguino)  # ✅ Pingüino nadando
```

### Ejemplo backend: Pagos

```python
class ProcesadorPago(ABC):
    """Contrato para procesadores de pago"""
    
    @abstractmethod
    def procesar(self, monto: float) -> bool:
        """Procesa un pago. Retorna True si fue exitoso"""
        pass

class PagoTarjeta(ProcesadorPago):
    def procesar(self, monto: float) -> bool:
        print(f"💳 Procesando ${monto} con tarjeta")
        return True

class PagoPaypal(ProcesadorPago):
    def procesar(self, monto: float) -> bool:
        print(f"💰 Procesando ${monto} con PayPal")
        return True

class PagoCripto(ProcesadorPago):
    def procesar(self, monto: float) -> bool:
        print(f"₿ Procesando ${monto} con cripto")
        return True

# ✅ Cualquier ProcesadorPago puede usarse intercambiablemente
def realizar_pago(procesador: ProcesadorPago, monto: float):
    if procesador.procesar(monto):
        print("✅ Pago exitoso")
    else:
        print("❌ Pago fallido")

# Uso
realizar_pago(PagoTarjeta(), 100)
realizar_pago(PagoPaypal(), 50)
realizar_pago(PagoCripto(), 200)
```

**Beneficios:**
- ✅ Cualquier subclase puede sustituir a la base
- ✅ Código predecible y confiable

---

## 4. Interface Segregation Principle (ISP)

**Principio:** Los clientes no deben depender de interfaces que no usan.

**En otras palabras:** Muchas interfaces pequeñas > Una interfaz grande.

### ❌ Violación de ISP

```python
class Trabajador(ABC):
    """Interfaz muy grande"""
    
    @abstractmethod
    def trabajar(self):
        pass
    
    @abstractmethod
    def comer(self):
        pass
    
    @abstractmethod
    def dormir(self):
        pass
    
    @abstractmethod
    def recibir_salario(self):
        pass

class Empleado(Trabajador):
    """✅ OK - Usa todos los métodos"""
    
    def trabajar(self):
        print("Trabajando...")
    
    def comer(self):
        print("Comiendo...")
    
    def dormir(self):
        print("Durmiendo...")
    
    def recibir_salario(self):
        print("Recibiendo salario")

class Robot(Trabajador):
    """❌ Problema - No come ni duerme"""
    
    def trabajar(self):
        print("Trabajando 24/7...")
    
    def comer(self):
        raise NotImplementedError("Los robots no comen")
    
    def dormir(self):
        raise NotImplementedError("Los robots no duermen")
    
    def recibir_salario(self):
        raise NotImplementedError("Los robots no cobran")
```

### ✅ Aplicando ISP

```python
class Trabajable(ABC):
    """Interfaz pequeña: solo trabajar"""
    @abstractmethod
    def trabajar(self):
        pass

class Alimentable(ABC):
    """Interfaz pequeña: solo comer"""
    @abstractmethod
    def comer(self):
        pass

class Descansable(ABC):
    """Interfaz pequeña: solo dormir"""
    @abstractmethod
    def dormir(self):
        pass

class Pagable(ABC):
    """Interfaz pequeña: solo pagar"""
    @abstractmethod
    def recibir_salario(self):
        pass

# ✅ Empleado implementa todas las interfaces que necesita
class Empleado(Trabajable, Alimentable, Descansable, Pagable):
    def trabajar(self):
        print("👨‍💼 Trabajando...")
    
    def comer(self):
        print("🍽️ Comiendo...")
    
    def dormir(self):
        print("😴 Durmiendo...")
    
    def recibir_salario(self):
        print("💵 Recibiendo salario")

# ✅ Robot solo implementa Trabajable
class Robot(Trabajable):
    def trabajar(self):
        print("🤖 Trabajando 24/7...")

# Uso
empleado = Empleado()
empleado.trabajar()
empleado.comer()

robot = Robot()
robot.trabajar()
# robot.comer()  # ❌ No existe, evita error
```

### Ejemplo backend: Repositorio

```python
# ❌ Interfaz muy grande
class RepositorioCompleto(ABC):
    @abstractmethod
    def crear(self, entidad): pass
    
    @abstractmethod
    def leer(self, id): pass
    
    @abstractmethod
    def actualizar(self, entidad): pass
    
    @abstractmethod
    def eliminar(self, id): pass
    
    @abstractmethod
    def buscar(self, criterio): pass
    
    @abstractmethod
    def exportar_csv(self): pass
    
    @abstractmethod
    def importar_csv(self, archivo): pass

# ✅ Interfaces segregadas
class Legible(ABC):
    @abstractmethod
    def leer(self, id): pass

class Escribible(ABC):
    @abstractmethod
    def crear(self, entidad): pass
    
    @abstractmethod
    def actualizar(self, entidad): pass

class Eliminable(ABC):
    @abstractmethod
    def eliminar(self, id): pass

class Exportable(ABC):
    @abstractmethod
    def exportar_csv(self): pass

# Repositorio de solo lectura
class RepositorioLectura(Legible):
    def leer(self, id):
        print(f"Leyendo entidad {id}")

# Repositorio completo
class RepositorioCompleto(Legible, Escribible, Eliminable):
    def leer(self, id):
        print(f"Leyendo {id}")
    
    def crear(self, entidad):
        print(f"Creando {entidad}")
    
    def actualizar(self, entidad):
        print(f"Actualizando {entidad}")
    
    def eliminar(self, id):
        print(f"Eliminando {id}")
```

**Beneficios:**
- ✅ Clases solo implementan lo que necesitan
- ✅ Interfaces más claras y específicas

---

## 5. Dependency Inversion Principle (DIP)

**Principio:** Depende de **abstracciones**, no de implementaciones concretas.

**En otras palabras:** Las clases de alto nivel no deben depender de clases de bajo nivel. Ambas deben depender de abstracciones.

### ❌ Violación de DIP

```python
class BaseDatosMySQL:
    """Implementación concreta"""
    def guardar(self, datos):
        print(f"Guardando en MySQL: {datos}")

class ServicioUsuarios:
    """❌ Depende directamente de MySQL"""
    
    def __init__(self):
        self.bd = BaseDatosMySQL()  # ❌ Acoplamiento directo
    
    def crear_usuario(self, nombre):
        usuario = {'nombre': nombre}
        self.bd.guardar(usuario)

# Problema: Si quiero cambiar a PostgreSQL, modifico ServicioUsuarios
```

### ✅ Aplicando DIP

```python
from abc import ABC, abstractmethod

class RepositorioDatos(ABC):
    """Abstracción"""
    
    @abstractmethod
    def guardar(self, datos):
        pass

class BaseDatosMySQL(RepositorioDatos):
    """Implementación concreta 1"""
    def guardar(self, datos):
        print(f"💾 Guardando en MySQL: {datos}")

class BaseDatosPostgreSQL(RepositorioDatos):
    """Implementación concreta 2"""
    def guardar(self, datos):
        print(f"🐘 Guardando en PostgreSQL: {datos}")

class BaseDatosMemoria(RepositorioDatos):
    """Implementación concreta 3 (para testing)"""
    def guardar(self, datos):
        print(f"🧠 Guardando en memoria: {datos}")

class ServicioUsuarios:
    """✅ Depende de la abstracción"""
    
    def __init__(self, repositorio: RepositorioDatos):
        self.repositorio = repositorio  # ✅ Inyección de dependencias
    
    def crear_usuario(self, nombre):
        usuario = {'nombre': nombre}
        self.repositorio.guardar(usuario)

# Uso - Inyección de dependencias
servicio_mysql = ServicioUsuarios(BaseDatosMySQL())
servicio_mysql.crear_usuario("Ana")

servicio_postgres = ServicioUsuarios(BaseDatosPostgreSQL())
servicio_postgres.crear_usuario("Luis")

servicio_memoria = ServicioUsuarios(BaseDatosMemoria())
servicio_memoria.crear_usuario("María")
```

**Salida:**
```
💾 Guardando en MySQL: {'nombre': 'Ana'}
🐘 Guardando en PostgreSQL: {'nombre': 'Luis'}
🧠 Guardando en memoria: {'nombre': 'María'}
```

### Ejemplo backend completo

```python
# Abstracciones
class ServicioEmail(ABC):
    @abstractmethod
    def enviar(self, destinatario, mensaje):
        pass

class RepositorioUsuarios(ABC):
    @abstractmethod
    def guardar(self, usuario):
        pass

# Implementaciones
class EmailSMTP(ServicioEmail):
    def enviar(self, destinatario, mensaje):
        print(f"📧 Enviando email SMTP a {destinatario}")

class EmailSendGrid(ServicioEmail):
    def enviar(self, destinatario, mensaje):
        print(f"📨 Enviando email SendGrid a {destinatario}")

class RepositorioUsuariosSQL(RepositorioUsuarios):
    def guardar(self, usuario):
        print(f"💾 Guardando en SQL: {usuario}")

# Servicio de alto nivel
class GestorRegistro:
    """✅ Depende solo de abstracciones"""
    
    def __init__(self, repositorio: RepositorioUsuarios, email: ServicioEmail):
        self.repositorio = repositorio
        self.email = email
    
    def registrar_usuario(self, nombre, email_usuario):
        usuario = {'nombre': nombre, 'email': email_usuario}
        
        # Guardar
        self.repositorio.guardar(usuario)
        
        # Enviar email de bienvenida
        self.email.enviar(email_usuario, "¡Bienvenido!")

# Uso - Composición flexible
gestor = GestorRegistro(
    repositorio=RepositorioUsuariosSQL(),
    email=EmailSendGrid()
)

gestor.registrar_usuario("Ana", "ana@ejemplo.com")

# ✅ Fácil cambiar implementaciones sin tocar GestorRegistro
gestor_alternativo = GestorRegistro(
    repositorio=RepositorioUsuariosSQL(),
    email=EmailSMTP()  # Cambio solo la implementación
)
```

### DIP con inyección de dependencias

```python
from http.server import BaseHTTPRequestHandler
import json

# Configurar dependencias
repositorio = RepositorioUsuariosSQL()
servicio_email = EmailSendGrid()
gestor_registro = GestorRegistro(repositorio, servicio_email)

class APIHandler(BaseHTTPRequestHandler):
    def do_POST(self):
        if self.path == '/api/registro':
            # Leer datos
            content_length = int(self.headers.get('Content-Length', 0))
            body = self.rfile.read(content_length)
            datos = json.loads(body.decode('utf-8'))
            
            # Registrar usuario usando gestor inyectado
            gestor_registro.registrar_usuario(datos['nombre'], datos['email'])
            
            # Responder
            self.send_response(201)
            self.send_header('Content-type', 'application/json')
            self.end_headers()
            respuesta = {'mensaje': 'Usuario registrado'}
            self.wfile.write(json.dumps(respuesta).encode())

# ✅ Para testing, inyecto mocks
class APIHandlerTesting(BaseHTTPRequestHandler):
    # Mocks para testing
    repo_mock = BaseDatosMemoria()
    email_mock = EmailMock()
    gestor_test = GestorRegistro(repo_mock, email_mock)
    
    def do_POST(self):
        if self.path == '/api/registro':
            content_length = int(self.headers.get('Content-Length', 0))
            body = self.rfile.read(content_length)
            datos = json.loads(body.decode('utf-8'))
            
            # Usar gestor con mocks
            self.gestor_test.registrar_usuario(datos['nombre'], datos['email'])
            
            self.send_response(201)
            self.send_header('Content-type', 'application/json')
            self.end_headers()
            self.wfile.write(json.dumps({'mensaje': 'Usuario registrado'}).encode())
```

**Beneficios:**
- ✅ Fácil cambiar implementaciones
- ✅ Excelente para testing
- ✅ Bajo acoplamiento

---

## Comparación: Código sin SOLID vs con SOLID

### ❌ Sin SOLID

```python
class ServicioUsuario:
    def crear_usuario(self, nombre, email):
        # Múltiples responsabilidades (❌ SRP)
        print(f"INSERT INTO usuarios VALUES ('{nombre}', '{email}')")
        print(f"Enviando email a {email}")
        return {'nombre': nombre}
    
    def aplicar_descuento(self, tipo):
        # Cerrado a extensión (❌ OCP)
        if tipo == 'normal':
            return 0
        elif tipo == 'vip':
            return 10
        # Si añado 'estudiante', modifico esta función
```

### ✅ Con SOLID

```python
# ✅ SRP: Una responsabilidad por clase
class Usuario:
    def __init__(self, nombre, email):
        self.nombre = nombre
        self.email = email

class RepositorioUsuarios:
    def guardar(self, usuario):
        print(f"Guardando usuario: {usuario.nombre}")

class ServicioEmail:
    def enviar_bienvenida(self, usuario):
        print(f"Enviando email a {usuario.email}")

# ✅ OCP: Extensible sin modificación
class Descuento(ABC):
    @abstractmethod
    def calcular(self): pass

class DescuentoNormal(Descuento):
    def calcular(self): return 0

class DescuentoVIP(Descuento):
    def calcular(self): return 10

class DescuentoEstudiante(Descuento):  # ✅ Añadido sin modificar código
    def calcular(self): return 5

# ✅ DIP: Depende de abstracciones
class GestorUsuarios:
    def __init__(self, repo: RepositorioUsuarios, email: ServicioEmail):
        self.repo = repo
        self.email = email
```

---

## Ejercicio práctico: Sistema con SOLID

```python
# Sistema de pedidos aplicando SOLID

# Abstracciones (DIP)
class RepositorioPedidos(ABC):
    @abstractmethod
    def guardar(self, pedido): pass

class ServicioNotificacion(ABC):
    @abstractmethod
    def notificar(self, mensaje): pass

# Implementaciones concretas
class RepositorioPedidosSQL(RepositorioPedidos):
    def guardar(self, pedido):
        print(f"💾 Guardando pedido {pedido.id} en SQL")

class NotificacionEmail(ServicioNotificacion):
    def notificar(self, mensaje):
        print(f"📧 {mensaje}")

# Estrategias de descuento (OCP)
class EstrategiaDescuento(ABC):
    @abstractmethod
    def calcular(self, monto): pass

class SinDescuento(EstrategiaDescuento):
    def calcular(self, monto):
        return 0

class DescuentoPorcentaje(EstrategiaDescuento):
    def __init__(self, porcentaje):
        self.porcentaje = porcentaje
    
    def calcular(self, monto):
        return monto * (self.porcentaje / 100)

# Entidad (SRP)
class Pedido:
    def __init__(self, id, cliente, monto):
        self.id = id
        self.cliente = cliente
        self.monto = monto
    
    def calcular_total(self, descuento: EstrategiaDescuento):
        return self.monto - descuento.calcular(self.monto)

# Servicio (SRP + DIP)
class GestorPedidos:
    def __init__(
        self,
        repositorio: RepositorioPedidos,
        notificacion: ServicioNotificacion
    ):
        self.repositorio = repositorio
        self.notificacion = notificacion
    
    def procesar_pedido(self, pedido: Pedido, descuento: EstrategiaDescuento):
        total = pedido.calcular_total(descuento)
        self.repositorio.guardar(pedido)
        self.notificacion.notificar(f"Pedido {pedido.id} procesado: ${total}")

# Uso
gestor = GestorPedidos(
    repositorio=RepositorioPedidosSQL(),
    notificacion=NotificacionEmail()
)

pedido = Pedido(1, "Ana", 1000)
gestor.procesar_pedido(pedido, DescuentoPorcentaje(10))
```

---

## Resumen SOLID

| Principio | Pregunta clave |
|-----------|----------------|
| **S**RP | ¿Esta clase tiene más de una razón para cambiar? |
| **O**CP | ¿Puedo añadir funcionalidad sin modificar código existente? |
| **L**SP | ¿Las subclases pueden sustituir a la clase base? |
| **I**SP | ¿Hay métodos que algunas clases no usan? |
| **D**IP | ¿Dependo de abstracciones o de implementaciones? |

---

## Conclusión

Has aprendido:

- ✅ Los 5 principios SOLID
- ✅ Single Responsibility: Una clase, una responsabilidad
- ✅ Open/Closed: Extensible sin modificación
- ✅ Liskov Substitution: Subclases sustituibles
- ✅ Interface Segregation: Interfaces pequeñas
- ✅ Dependency Inversion: Depende de abstracciones
- ✅ Cómo aplicar SOLID en backend

**Siguiente:** [README del backend](./README.md)

---

## Recursos adicionales

- **[SOLID Principles](https://www.digitalocean.com/community/conceptual_articles/s-o-l-i-d-the-first-five-principles-of-object-oriented-design)** - Guía completa
- **[Clean Code](https://www.amazon.com/Clean-Code-Handbook-Software-Craftsmanship/dp/0132350882)** - Libro de Uncle Bob

SOLID no es obligatorio, pero hace tu código profesional. 🚀

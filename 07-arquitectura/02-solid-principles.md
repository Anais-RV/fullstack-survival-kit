# SOLID Principles

> **5 principios para escribir código orientado a objetos mantenible**

---

## ¿Qué es SOLID?

**SOLID** = Acrónimo de 5 principios de diseño

```
S - Single Responsibility Principle
O - Open/Closed Principle
L - Liskov Substitution Principle
I - Interface Segregation Principle
D - Dependency Inversion Principle
```

**Objetivo:** Código flexible, testeable y fácil de mantener

---

## S - Single Responsibility Principle

> **Una clase debe tener una sola razón para cambiar**

### ❌ Violando SRP

```python
# models.py
class Usuario(models.Model):
    username = models.CharField(max_length=100)
    email = models.EmailField()
    
    def save(self, *args, **kwargs):
        super().save(*args, **kwargs)
        
        # ❌ Enviar email (responsabilidad extra)
        send_mail(
            'Bienvenido',
            f'Hola {self.username}',
            'admin@example.com',
            [self.email]
        )
        
        # ❌ Logging (otra responsabilidad)
        with open('usuarios.log', 'a') as f:
            f.write(f'Usuario creado: {self.username}\n')
```

**Problemas:**
- 3 razones para cambiar: persistencia, email, logging
- Difícil de testear
- No reutilizable

---

### ✅ Aplicando SRP

```python
# models.py (solo persistencia)
class Usuario(models.Model):
    username = models.CharField(max_length=100)
    email = models.EmailField()

# services/email_service.py (solo emails)
class EmailService:
    @staticmethod
    def enviar_bienvenida(usuario):
        send_mail(
            'Bienvenido',
            f'Hola {usuario.username}',
            'admin@example.com',
            [usuario.email]
        )

# services/logger_service.py (solo logging)
class LoggerService:
    @staticmethod
    def log_usuario_creado(usuario):
        with open('usuarios.log', 'a') as f:
            f.write(f'Usuario creado: {usuario.username}\n')

# use_cases/crear_usuario.py (orquesta todo)
class CrearUsuario:
    def __init__(self, email_service, logger_service):
        self.email_service = email_service
        self.logger_service = logger_service
    
    def execute(self, username, email):
        usuario = Usuario.objects.create(
            username=username,
            email=email
        )
        
        self.email_service.enviar_bienvenida(usuario)
        self.logger_service.log_usuario_creado(usuario)
        
        return usuario
```

**Beneficios:**
- Cada clase tiene una responsabilidad
- Fácil de testear (mock de servicios)
- Reutilizable

---

## O - Open/Closed Principle

> **Abierto para extensión, cerrado para modificación**

### ❌ Violando OCP

```python
# services/precio_service.py
class PrecioService:
    def calcular_precio_final(self, producto, tipo_cliente):
        if tipo_cliente == 'regular':
            return producto.precio
        elif tipo_cliente == 'premium':
            return producto.precio * 0.9  # 10% descuento
        elif tipo_cliente == 'vip':
            return producto.precio * 0.8  # 20% descuento
        # ❌ Para agregar nuevo tipo, hay que modificar este método
```

**Problema:** Cada nuevo tipo de cliente requiere modificar el código existente

---

### ✅ Aplicando OCP

```python
# strategies/descuento_strategy.py
from abc import ABC, abstractmethod

class DescuentoStrategy(ABC):
    @abstractmethod
    def aplicar(self, precio):
        pass

class DescuentoRegular(DescuentoStrategy):
    def aplicar(self, precio):
        return precio

class DescuentoPremium(DescuentoStrategy):
    def aplicar(self, precio):
        return precio * 0.9  # 10%

class DescuentoVIP(DescuentoStrategy):
    def aplicar(self, precio):
        return precio * 0.8  # 20%

# ✅ Agregar nuevo descuento sin modificar código existente
class DescuentoEmpleado(DescuentoStrategy):
    def aplicar(self, precio):
        return precio * 0.5  # 50%

# services/precio_service.py
class PrecioService:
    def __init__(self, descuento_strategy: DescuentoStrategy):
        self.descuento_strategy = descuento_strategy
    
    def calcular_precio_final(self, producto):
        return self.descuento_strategy.aplicar(producto.precio)

# Uso
producto = Producto(nombre="Laptop", precio=1000)

servicio_regular = PrecioService(DescuentoRegular())
print(servicio_regular.calcular_precio_final(producto))  # 1000

servicio_vip = PrecioService(DescuentoVIP())
print(servicio_vip.calcular_precio_final(producto))  # 800
```

---

### Ejemplo en React

```jsx
// ❌ Violando OCP
function Notificacion({ tipo, mensaje }) {
    if (tipo === 'success') {
        return <div className="bg-green-500">{mensaje}</div>;
    } else if (tipo === 'error') {
        return <div className="bg-red-500">{mensaje}</div>;
    } else if (tipo === 'warning') {
        return <div className="bg-yellow-500">{mensaje}</div>;
    }
    // ❌ Para agregar nuevo tipo, modificar esta función
}

// ✅ Aplicando OCP
const notificacionConfig = {
    success: { bg: 'bg-green-500', icon: '✓' },
    error: { bg: 'bg-red-500', icon: '✗' },
    warning: { bg: 'bg-yellow-500', icon: '⚠' },
    // ✅ Agregar nuevos tipos sin modificar componente
    info: { bg: 'bg-blue-500', icon: 'ℹ' },
};

function Notificacion({ tipo, mensaje }) {
    const config = notificacionConfig[tipo] || notificacionConfig.info;
    
    return (
        <div className={`${config.bg} p-4 rounded`}>
            <span className="mr-2">{config.icon}</span>
            {mensaje}
        </div>
    );
}
```

---

## L - Liskov Substitution Principle

> **Las subclases deben ser sustituibles por sus clases base**

### ❌ Violando LSP

```python
class Ave:
    def volar(self):
        return "Volando..."

class Aguila(Ave):
    def volar(self):
        return "Aguila volando alto"

class Pinguino(Ave):
    def volar(self):
        # ❌ Pingüino no puede volar
        raise Exception("Pingüinos no vuelan")

# Uso
def hacer_volar(ave: Ave):
    print(ave.volar())

aguila = Aguila()
hacer_volar(aguila)  # ✅ Funciona

pinguino = Pinguino()
hacer_volar(pinguino)  # ❌ Exception! Viola LSP
```

---

### ✅ Aplicando LSP

```python
class Ave:
    def comer(self):
        return "Comiendo..."

class AveVoladora(Ave):
    def volar(self):
        return "Volando..."

class AveNoVoladora(Ave):
    pass

class Aguila(AveVoladora):
    def volar(self):
        return "Aguila volando alto"

class Pinguino(AveNoVoladora):
    def nadar(self):
        return "Pingüino nadando"

# Uso
def hacer_volar(ave: AveVoladora):
    print(ave.volar())

aguila = Aguila()
hacer_volar(aguila)  # ✅ Funciona

pinguino = Pinguino()
# hacer_volar(pinguino)  # ✅ Error de tipo, no compila
print(pinguino.nadar())  # ✅ Funciona correctamente
```

---

### Ejemplo Django

```python
# ❌ Violando LSP
class Vehiculo(models.Model):
    marca = models.CharField(max_length=100)
    
    def arrancar_motor(self):
        return "Motor arrancado"

class Auto(Vehiculo):
    def arrancar_motor(self):
        return "Auto arrancando motor"

class Bicicleta(Vehiculo):
    def arrancar_motor(self):
        # ❌ Bicicleta no tiene motor
        raise NotImplementedError("Bicicleta no tiene motor")

# ✅ Aplicando LSP
class Vehiculo(models.Model):
    marca = models.CharField(max_length=100)
    
    class Meta:
        abstract = True

class VehiculoMotor(Vehiculo):
    def arrancar_motor(self):
        return "Motor arrancado"
    
    class Meta:
        abstract = True

class Auto(VehiculoMotor):
    pass

class Bicicleta(Vehiculo):
    def pedalear(self):
        return "Pedaleando"
```

---

## I - Interface Segregation Principle

> **No obligar a clientes a depender de interfaces que no usan**

### ❌ Violando ISP

```python
from abc import ABC, abstractmethod

class Trabajador(ABC):
    @abstractmethod
    def trabajar(self):
        pass
    
    @abstractmethod
    def comer(self):
        pass
    
    @abstractmethod
    def dormir(self):
        pass

class Humano(Trabajador):
    def trabajar(self):
        return "Trabajando..."
    
    def comer(self):
        return "Comiendo..."
    
    def dormir(self):
        return "Durmiendo..."

class Robot(Trabajador):
    def trabajar(self):
        return "Robot trabajando..."
    
    def comer(self):
        # ❌ Robot no come
        pass
    
    def dormir(self):
        # ❌ Robot no duerme
        pass
```

---

### ✅ Aplicando ISP

```python
from abc import ABC, abstractmethod

# Interfaces pequeñas y específicas
class Trabajable(ABC):
    @abstractmethod
    def trabajar(self):
        pass

class Comible(ABC):
    @abstractmethod
    def comer(self):
        pass

class Dormible(ABC):
    @abstractmethod
    def dormir(self):
        pass

# Humano implementa todas
class Humano(Trabajable, Comible, Dormible):
    def trabajar(self):
        return "Trabajando..."
    
    def comer(self):
        return "Comiendo..."
    
    def dormir(self):
        return "Durmiendo..."

# Robot solo implementa lo que necesita
class Robot(Trabajable):
    def trabajar(self):
        return "Robot trabajando..."
```

---

### Ejemplo React

```jsx
// ❌ Violando ISP
interface UsuarioCompleto {
    id: number;
    nombre: string;
    email: string;
    password: string;
    fechaNacimiento: Date;
    direccion: string;
    telefono: string;
}

// Componente que solo muestra nombre
function UsuarioCard({ usuario }: { usuario: UsuarioCompleto }) {
    // ❌ Recibe todo el objeto pero solo usa nombre
    return <div>{usuario.nombre}</div>;
}

// ✅ Aplicando ISP
interface UsuarioBasico {
    nombre: string;
}

interface UsuarioConContacto extends UsuarioBasico {
    email: string;
    telefono: string;
}

function UsuarioCard({ usuario }: { usuario: UsuarioBasico }) {
    // ✅ Solo recibe lo que necesita
    return <div>{usuario.nombre}</div>;
}

function UsuarioContacto({ usuario }: { usuario: UsuarioConContacto }) {
    return (
        <div>
            <h2>{usuario.nombre}</h2>
            <p>{usuario.email}</p>
            <p>{usuario.telefono}</p>
        </div>
    );
}
```

---

## D - Dependency Inversion Principle

> **Depender de abstracciones, no de implementaciones concretas**

### ❌ Violando DIP

```python
# services/producto_service.py
from adapters.repositories.producto_repository_django import ProductoRepositoryDjango

class ProductoService:
    def __init__(self):
        # ❌ Depende de implementación concreta
        self.repo = ProductoRepositoryDjango()
    
    def obtener_productos(self):
        return self.repo.obtener_todos()
```

**Problemas:**
- Acoplado a Django ORM
- Difícil de testear (requiere BD)
- No se puede cambiar a otra implementación

---

### ✅ Aplicando DIP

```python
# core/interfaces/producto_repository.py
from abc import ABC, abstractmethod
from typing import List
from core.entities.producto import Producto

class ProductoRepository(ABC):
    """Abstracción (interface)"""
    @abstractmethod
    def obtener_todos(self) -> List[Producto]:
        pass
    
    @abstractmethod
    def guardar(self, producto: Producto) -> Producto:
        pass

# services/producto_service.py
class ProductoService:
    def __init__(self, repo: ProductoRepository):
        # ✅ Depende de abstracción
        self.repo = repo
    
    def obtener_productos(self):
        return self.repo.obtener_todos()

# adapters/repositories/producto_repository_django.py
class ProductoRepositoryDjango(ProductoRepository):
    """Implementación concreta con Django"""
    def obtener_todos(self) -> List[Producto]:
        productos_db = ProductoModel.objects.all()
        return [self._to_entity(p) for p in productos_db]
    
    def guardar(self, producto: Producto) -> Producto:
        p = ProductoModel.objects.create(
            nombre=producto.nombre,
            precio=producto.precio
        )
        producto.id = p.id
        return producto

# adapters/repositories/producto_repository_memory.py
class ProductoRepositoryMemory(ProductoRepository):
    """Implementación en memoria (para tests)"""
    def __init__(self):
        self.productos = []
    
    def obtener_todos(self) -> List[Producto]:
        return self.productos
    
    def guardar(self, producto: Producto) -> Producto:
        producto.id = len(self.productos) + 1
        self.productos.append(producto)
        return producto

# Uso (inyección de dependencias)
repo_django = ProductoRepositoryDjango()
service = ProductoService(repo_django)
productos = service.obtener_productos()

# Testing
repo_memory = ProductoRepositoryMemory()
service_test = ProductoService(repo_memory)
# ✅ Test sin base de datos
```

---

### Ejemplo React

```jsx
// ❌ Violando DIP
import axios from 'axios';

function ProductosList() {
    const [productos, setProductos] = useState([]);
    
    useEffect(() => {
        // ❌ Acoplado a axios
        axios.get('/api/productos/')
            .then(res => setProductos(res.data));
    }, []);
    
    return <div>{/* ... */}</div>;
}

// ✅ Aplicando DIP
// infrastructure/api/ProductoApi.js
class ProductoApi {
    async obtenerTodos() {
        const response = await axios.get('/api/productos/');
        return response.data;
    }
}

// hooks/useProductos.js
function useProductos(productoApi) {
    const [productos, setProductos] = useState([]);
    
    useEffect(() => {
        productoApi.obtenerTodos()
            .then(setProductos);
    }, [productoApi]);
    
    return productos;
}

// components/ProductosList.jsx
function ProductosList({ productoApi }) {
    const productos = useProductos(productoApi);
    
    return <div>{/* ... */}</div>;
}

// App.jsx (inyección de dependencias)
const productoApi = new ProductoApi();

function App() {
    return <ProductosList productoApi={productoApi} />;
}

// Test
const productoApiMock = {
    obtenerTodos: () => Promise.resolve([
        { id: 1, nombre: 'Test' }
    ])
};

test('renderiza productos', () => {
    render(<ProductosList productoApi={productoApiMock} />);
    // ✅ Test sin API real
});
```

---

## Resumen SOLID

| Principio | Resumen | Beneficio |
|-----------|---------|-----------|
| **S** - SRP | Una responsabilidad por clase | Fácil de entender y mantener |
| **O** - OCP | Extensible sin modificar | Agregar features sin romper existente |
| **L** - LSP | Subclases sustituibles | Polimorfismo confiable |
| **I** - ISP | Interfaces pequeñas | No obligar a implementar lo innecesario |
| **D** - DIP | Depender de abstracciones | Desacoplado y testeable |

---

## SOLID en proyecto FullStack

```
backend/
├── core/
│   ├── entities/
│   │   └── producto.py          # S: Solo define entidad
│   ├── interfaces/
│   │   └── producto_repo.py     # D: Abstracción
│   └── use_cases/
│       └── crear_producto.py    # S: Solo lógica de caso de uso
│
├── adapters/
│   ├── repositories/
│   │   ├── producto_repo_django.py   # D: Implementación
│   │   └── producto_repo_memory.py   # O: Extensible
│   └── strategies/
│       └── descuento_strategy.py     # O: Estrategias
│
└── infrastructure/
    └── models.py                # S: Solo Django models

frontend/
├── domain/
│   ├── entities/
│   │   └── Producto.js          # S: Solo entidad
│   └── interfaces/
│       └── ProductoRepository.js # D: Abstracción
│
├── infrastructure/
│   └── api/
│       └── ProductoApiHTTP.js   # D: Implementación
│
└── presentation/
    └── components/
        └── ProductoForm.jsx     # S: Solo UI
```

---

## Ejercicio práctico

**Refactorizar este código:**

```python
# ❌ Viola múltiples principios SOLID
class Usuario(models.Model):
    username = models.CharField(max_length=100)
    email = models.EmailField()
    tipo = models.CharField(max_length=20)  # 'regular', 'premium', 'vip'
    
    def save(self, *args, **kwargs):
        super().save(*args, **kwargs)
        
        # Enviar email
        if self.tipo == 'premium':
            send_mail('Bienvenido Premium', '...', 'admin@example.com', [self.email])
        elif self.tipo == 'vip':
            send_mail('Bienvenido VIP', '...', 'admin@example.com', [self.email])
        
        # Logging
        with open('usuarios.log', 'a') as f:
            f.write(f'{self.username} - {self.tipo}\n')
    
    def obtener_descuento(self):
        if self.tipo == 'regular':
            return 0
        elif self.tipo == 'premium':
            return 10
        elif self.tipo == 'vip':
            return 20
```

**Solución aplicando SOLID:**

```python
# S - Separar responsabilidades
class Usuario(models.Model):
    username = models.CharField(max_length=100)
    email = models.EmailField()

# O - Estrategia de descuento (Open/Closed)
class DescuentoStrategy(ABC):
    @abstractmethod
    def obtener_descuento(self) -> int:
        pass

class DescuentoRegular(DescuentoStrategy):
    def obtener_descuento(self) -> int:
        return 0

class DescuentoPremium(DescuentoStrategy):
    def obtener_descuento(self) -> int:
        return 10

# D - Interfaces para servicios
class EmailService(ABC):
    @abstractmethod
    def enviar_bienvenida(self, usuario: Usuario):
        pass

class LoggerService(ABC):
    @abstractmethod
    def log_usuario(self, usuario: Usuario):
        pass

# Use case orquestando todo
class CrearUsuario:
    def __init__(
        self,
        email_service: EmailService,
        logger_service: LoggerService,
        descuento_strategy: DescuentoStrategy
    ):
        self.email_service = email_service
        self.logger_service = logger_service
        self.descuento_strategy = descuento_strategy
    
    def execute(self, username: str, email: str) -> Usuario:
        usuario = Usuario.objects.create(username=username, email=email)
        
        self.email_service.enviar_bienvenida(usuario)
        self.logger_service.log_usuario(usuario)
        
        return usuario
```

---

## Recursos

- **[SOLID Principles](https://en.wikipedia.org/wiki/SOLID)** - Wikipedia
- **[Clean Code](https://www.amazon.com/Clean-Code-Handbook-Software-Craftsmanship/dp/0132350882)** - Robert C. Martin
- **[SOLID in Python](https://realpython.com/solid-principles-python/)** - Real Python

**Próximo módulo:** Design Patterns (Patrones de Diseño)

¡SOLID completo! 💎

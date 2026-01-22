# Design Patterns

> **Soluciones probadas a problemas comunes de diseño de software**

---

## ¿Qué son Design Patterns?

**Design Patterns (Patrones de Diseño)** = Plantillas reutilizables para resolver problemas recurrentes

**Categorías:**
- **Creacionales:** Cómo crear objetos
- **Estructurales:** Cómo organizar clases y objetos
- **Comportamentales:** Cómo los objetos interactúan

---

## CREACIONALES

### 1. Singleton

> **Asegurar que una clase tenga solo una instancia**

#### Problema
```python
# ❌ Sin Singleton
class Database:
    def __init__(self):
        self.connection = self.connect()
    
    def connect(self):
        print("Conectando a BD...")
        return "connection"

db1 = Database()  # Conectando a BD...
db2 = Database()  # Conectando a BD... (conexión duplicada!)
```

#### Solución: Singleton

```python
# ✅ Con Singleton
class Database:
    _instance = None
    
    def __new__(cls):
        if cls._instance is None:
            print("Creando única instancia...")
            cls._instance = super().__new__(cls)
            cls._instance.connection = cls._instance.connect()
        return cls._instance
    
    def connect(self):
        print("Conectando a BD...")
        return "connection"

db1 = Database()  # Creando única instancia... Conectando a BD...
db2 = Database()  # (No imprime nada, usa instancia existente)
print(db1 is db2)  # True
```

#### Ejemplo en Django

```python
# settings/database.py
class DatabaseConnection:
    _instance = None
    _connection = None
    
    def __new__(cls):
        if cls._instance is None:
            cls._instance = super().__new__(cls)
        return cls._instance
    
    def get_connection(self):
        if self._connection is None:
            self._connection = self._create_connection()
        return self._connection
    
    def _create_connection(self):
        # Configuración de conexión
        return "PostgreSQL Connection"

# Uso
db = DatabaseConnection()
conn1 = db.get_connection()
conn2 = db.get_connection()
print(conn1 is conn2)  # True
```

#### Ejemplo en React

```javascript
// services/ApiClient.js
class ApiClient {
    static instance = null;
    
    constructor() {
        if (ApiClient.instance) {
            return ApiClient.instance;
        }
        
        this.baseURL = 'http://localhost:8000/api';
        this.token = null;
        ApiClient.instance = this;
    }
    
    setToken(token) {
        this.token = token;
    }
    
    async get(endpoint) {
        const response = await fetch(`${this.baseURL}${endpoint}`, {
            headers: {
                'Authorization': `Bearer ${this.token}`
            }
        });
        return response.json();
    }
}

// Uso
const api1 = new ApiClient();
const api2 = new ApiClient();
console.log(api1 === api2);  // true

api1.setToken('abc123');
console.log(api2.token);  // 'abc123' (misma instancia)
```

---

### 2. Factory

> **Crear objetos sin especificar la clase exacta**

#### Problema

```python
# ❌ Sin Factory
def procesar_pago(tipo, monto):
    if tipo == 'tarjeta':
        pago = PagoTarjeta(monto)
    elif tipo == 'paypal':
        pago = PagoPayPal(monto)
    elif tipo == 'transferencia':
        pago = PagoTransferencia(monto)
    # ❌ Para agregar nuevo método, modificar esta función
    
    return pago.procesar()
```

#### Solución: Factory

```python
# ✅ Con Factory
from abc import ABC, abstractmethod

class Pago(ABC):
    def __init__(self, monto):
        self.monto = monto
    
    @abstractmethod
    def procesar(self):
        pass

class PagoTarjeta(Pago):
    def procesar(self):
        return f"Procesando ${self.monto} con tarjeta"

class PagoPayPal(Pago):
    def procesar(self):
        return f"Procesando ${self.monto} con PayPal"

class PagoTransferencia(Pago):
    def procesar(self):
        return f"Procesando ${self.monto} por transferencia"

# Factory
class PagoFactory:
    @staticmethod
    def crear_pago(tipo: str, monto: float) -> Pago:
        pagos = {
            'tarjeta': PagoTarjeta,
            'paypal': PagoPayPal,
            'transferencia': PagoTransferencia
        }
        
        clase_pago = pagos.get(tipo)
        if not clase_pago:
            raise ValueError(f"Tipo de pago '{tipo}' no soportado")
        
        return clase_pago(monto)

# Uso
pago = PagoFactory.crear_pago('tarjeta', 100)
print(pago.procesar())  # Procesando $100 con tarjeta
```

#### Ejemplo en React

```jsx
// components/inputs/InputFactory.jsx
function TextInput({ value, onChange }) {
    return <input type="text" value={value} onChange={onChange} />;
}

function NumberInput({ value, onChange }) {
    return <input type="number" value={value} onChange={onChange} />;
}

function SelectInput({ value, onChange, options }) {
    return (
        <select value={value} onChange={onChange}>
            {options.map(opt => <option key={opt} value={opt}>{opt}</option>)}
        </select>
    );
}

// Factory
function InputFactory({ type, value, onChange, options }) {
    const inputs = {
        text: TextInput,
        number: NumberInput,
        select: SelectInput
    };
    
    const InputComponent = inputs[type] || TextInput;
    
    return <InputComponent value={value} onChange={onChange} options={options} />;
}

// Uso
function Form() {
    return (
        <div>
            <InputFactory type="text" value={nombre} onChange={setNombre} />
            <InputFactory type="number" value={precio} onChange={setPrecio} />
            <InputFactory 
                type="select" 
                value={categoria} 
                onChange={setCategoria}
                options={['Electrónica', 'Ropa', 'Hogar']}
            />
        </div>
    );
}
```

---

## ESTRUCTURALES

### 3. Adapter

> **Hacer que interfaces incompatibles funcionen juntas**

#### Problema

```python
# Sistema antiguo
class SistemaPagoAntiguo:
    def realizar_pago(self, numero_tarjeta, monto):
        return f"Pago de ${monto} con tarjeta {numero_tarjeta}"

# Sistema nuevo
class SistemaPagoNuevo:
    def procesar(self, datos_pago):
        # Espera un diccionario con estructura diferente
        return f"Pago procesado: {datos_pago}"

# ❌ Incompatibles
```

#### Solución: Adapter

```python
# ✅ Adapter
class AdaptadorPagoAntiguo:
    def __init__(self, sistema_antiguo):
        self.sistema_antiguo = sistema_antiguo
    
    def procesar(self, datos_pago):
        # Adaptar estructura de datos
        numero_tarjeta = datos_pago['tarjeta']
        monto = datos_pago['monto']
        
        # Llamar al sistema antiguo con formato correcto
        return self.sistema_antiguo.realizar_pago(numero_tarjeta, monto)

# Uso
sistema_antiguo = SistemaPagoAntiguo()
adapter = AdaptadorPagoAntiguo(sistema_antiguo)

# Ahora tiene la misma interface que el sistema nuevo
resultado = adapter.procesar({'tarjeta': '1234', 'monto': 100})
print(resultado)
```

#### Ejemplo Django

```python
# Adapter para diferentes proveedores de email
from abc import ABC, abstractmethod

class EmailProvider(ABC):
    @abstractmethod
    def send(self, to, subject, body):
        pass

# Gmail
class GmailAdapter(EmailProvider):
    def send(self, to, subject, body):
        # Lógica específica de Gmail
        from django.core.mail import send_mail
        send_mail(subject, body, 'no-reply@example.com', [to])

# SendGrid
class SendGridAdapter(EmailProvider):
    def send(self, to, subject, body):
        # Lógica específica de SendGrid
        import sendgrid
        sg = sendgrid.SendGridAPIClient(api_key='...')
        # ...

# Servicio que usa el adapter
class EmailService:
    def __init__(self, provider: EmailProvider):
        self.provider = provider
    
    def enviar_bienvenida(self, usuario):
        self.provider.send(
            to=usuario.email,
            subject='Bienvenido',
            body=f'Hola {usuario.username}'
        )

# Uso (cambiar provider fácilmente)
email_service = EmailService(GmailAdapter())
# email_service = EmailService(SendGridAdapter())
```

---

### 4. Decorator

> **Agregar funcionalidad a objetos dinámicamente**

#### Ejemplo Django

```python
# decorators/auth_decorators.py
from functools import wraps
from rest_framework.response import Response

def require_premium(view_func):
    """Decorator para permitir solo usuarios premium"""
    @wraps(view_func)
    def wrapper(request, *args, **kwargs):
        if not request.user.is_authenticated:
            return Response({'error': 'No autenticado'}, status=401)
        
        if request.user.tipo != 'premium':
            return Response({'error': 'Requiere cuenta premium'}, status=403)
        
        return view_func(request, *args, **kwargs)
    return wrapper

# Uso
@require_premium
def vista_premium(request):
    return Response({'mensaje': 'Contenido premium'})
```

```python
# decorators/cache_decorator.py
from functools import wraps
from django.core.cache import cache

def cache_result(timeout=60):
    """Decorator para cachear resultados"""
    def decorator(func):
        @wraps(func)
        def wrapper(*args, **kwargs):
            # Generar clave de cache
            cache_key = f"{func.__name__}:{args}:{kwargs}"
            
            # Intentar obtener del cache
            result = cache.get(cache_key)
            if result is not None:
                return result
            
            # Ejecutar función y guardar en cache
            result = func(*args, **kwargs)
            cache.set(cache_key, result, timeout)
            return result
        return wrapper
    return decorator

# Uso
@cache_result(timeout=300)
def obtener_productos_destacados():
    # Consulta costosa
    return Producto.objects.filter(destacado=True).select_related('categoria')
```

#### Ejemplo React (Higher-Order Component)

```jsx
// hoc/withAuth.jsx
function withAuth(Component) {
    return function AuthComponent(props) {
        const { user, loading } = useAuth();
        
        if (loading) {
            return <div>Cargando...</div>;
        }
        
        if (!user) {
            return <Navigate to="/login" />;
        }
        
        return <Component {...props} user={user} />;
    };
}

// Uso
function Dashboard({ user }) {
    return <h1>Bienvenido {user.nombre}</h1>;
}

export default withAuth(Dashboard);
```

```jsx
// hoc/withLoading.jsx
function withLoading(Component) {
    return function LoadingComponent({ isLoading, ...props }) {
        if (isLoading) {
            return <div className="spinner">Cargando...</div>;
        }
        
        return <Component {...props} />;
    };
}

// Uso
function ProductosList({ productos }) {
    return (
        <div>
            {productos.map(p => <ProductoCard key={p.id} producto={p} />)}
        </div>
    );
}

export default withLoading(ProductosList);
```

---

## COMPORTAMENTALES

### 5. Observer

> **Notificar a múltiples objetos cuando cambia el estado**

#### Ejemplo Django (Signals)

```python
# signals.py
from django.db.models.signals import post_save
from django.dispatch import receiver
from .models import Producto

@receiver(post_save, sender=Producto)
def producto_creado(sender, instance, created, **kwargs):
    if created:
        print(f"Producto '{instance.nombre}' fue creado")
        
        # Notificar a otros servicios
        enviar_email_admin(instance)
        actualizar_cache()
        enviar_notificacion_push()

def enviar_email_admin(producto):
    from django.core.mail import send_mail
    send_mail(
        'Nuevo Producto',
        f'Se creó el producto: {producto.nombre}',
        'admin@example.com',
        ['admin@example.com']
    )
```

#### Ejemplo React (Context API como Observer)

```jsx
// context/NotificationContext.jsx
import { createContext, useContext, useState } from 'react';

const NotificationContext = createContext();

export function NotificationProvider({ children }) {
    const [notifications, setNotifications] = useState([]);
    
    const addNotification = (mensaje, tipo = 'info') => {
        const id = Date.now();
        setNotifications(prev => [...prev, { id, mensaje, tipo }]);
        
        // Auto-dismiss después de 3 segundos
        setTimeout(() => {
            setNotifications(prev => prev.filter(n => n.id !== id));
        }, 3000);
    };
    
    return (
        <NotificationContext.Provider value={{ notifications, addNotification }}>
            {children}
            <NotificationList notifications={notifications} />
        </NotificationContext.Provider>
    );
}

export function useNotification() {
    return useContext(NotificationContext);
}

// components/ProductoForm.jsx
function ProductoForm() {
    const { addNotification } = useNotification();
    
    const handleSubmit = async (e) => {
        e.preventDefault();
        try {
            await crearProducto(data);
            addNotification('Producto creado exitosamente', 'success');
        } catch (error) {
            addNotification('Error al crear producto', 'error');
        }
    };
    
    return <form onSubmit={handleSubmit}>{/* ... */}</form>;
}
```

---

### 6. Strategy

> **Definir familia de algoritmos intercambiables**

#### Ejemplo Django

```python
# strategies/descuento_strategy.py
from abc import ABC, abstractmethod
from decimal import Decimal

class DescuentoStrategy(ABC):
    @abstractmethod
    def calcular(self, precio: Decimal) -> Decimal:
        pass

class SinDescuento(DescuentoStrategy):
    def calcular(self, precio: Decimal) -> Decimal:
        return precio

class DescuentoPorcentaje(DescuentoStrategy):
    def __init__(self, porcentaje: int):
        self.porcentaje = porcentaje
    
    def calcular(self, precio: Decimal) -> Decimal:
        return precio * (1 - self.porcentaje / 100)

class DescuentoFijo(DescuentoStrategy):
    def __init__(self, monto: Decimal):
        self.monto = monto
    
    def calcular(self, precio: Decimal) -> Decimal:
        return max(precio - self.monto, Decimal('0'))

class DescuentoBlackFriday(DescuentoStrategy):
    def calcular(self, precio: Decimal) -> Decimal:
        if precio > 1000:
            return precio * Decimal('0.5')  # 50% para productos caros
        else:
            return precio * Decimal('0.7')  # 30% para otros

# services/precio_service.py
class PrecioService:
    def __init__(self, descuento: DescuentoStrategy):
        self.descuento = descuento
    
    def calcular_precio_final(self, producto) -> Decimal:
        return self.descuento.calcular(producto.precio)

# Uso
producto = Producto(nombre="Laptop", precio=Decimal('1500'))

# Sin descuento
servicio = PrecioService(SinDescuento())
print(servicio.calcular_precio_final(producto))  # 1500

# Con 20% descuento
servicio = PrecioService(DescuentoPorcentaje(20))
print(servicio.calcular_precio_final(producto))  # 1200

# Black Friday
servicio = PrecioService(DescuentoBlackFriday())
print(servicio.calcular_precio_final(producto))  # 750
```

---

### 7. Repository Pattern

> **Abstraer acceso a datos**

```python
# core/interfaces/producto_repository.py
from abc import ABC, abstractmethod
from typing import List, Optional
from core.entities.producto import Producto

class ProductoRepository(ABC):
    @abstractmethod
    def obtener_todos(self) -> List[Producto]:
        pass
    
    @abstractmethod
    def obtener_por_id(self, id: int) -> Optional[Producto]:
        pass
    
    @abstractmethod
    def guardar(self, producto: Producto) -> Producto:
        pass
    
    @abstractmethod
    def eliminar(self, id: int) -> None:
        pass

# adapters/repositories/producto_repository_django.py
from infrastructure.models import ProductoModel

class ProductoRepositoryDjango(ProductoRepository):
    def obtener_todos(self) -> List[Producto]:
        productos_db = ProductoModel.objects.all()
        return [self._to_entity(p) for p in productos_db]
    
    def obtener_por_id(self, id: int) -> Optional[Producto]:
        try:
            p = ProductoModel.objects.get(id=id)
            return self._to_entity(p)
        except ProductoModel.DoesNotExist:
            return None
    
    def guardar(self, producto: Producto) -> Producto:
        if producto.id:
            # Actualizar
            ProductoModel.objects.filter(id=producto.id).update(
                nombre=producto.nombre,
                precio=producto.precio,
                stock=producto.stock
            )
        else:
            # Crear
            p = ProductoModel.objects.create(
                nombre=producto.nombre,
                precio=producto.precio,
                stock=producto.stock
            )
            producto.id = p.id
        return producto
    
    def eliminar(self, id: int) -> None:
        ProductoModel.objects.filter(id=id).delete()
    
    def _to_entity(self, modelo: ProductoModel) -> Producto:
        return Producto(
            id=modelo.id,
            nombre=modelo.nombre,
            precio=modelo.precio,
            stock=modelo.stock
        )

# Uso
repo = ProductoRepositoryDjango()
productos = repo.obtener_todos()
```

---

## Resumen de Design Patterns

| Patrón | Tipo | Propósito | Cuándo usar |
|--------|------|-----------|-------------|
| **Singleton** | Creacional | Una sola instancia | Conexiones BD, config global |
| **Factory** | Creacional | Crear objetos sin especificar clase | Múltiples tipos similares |
| **Adapter** | Estructural | Hacer interfaces compatibles | Integrar sistemas legacy |
| **Decorator** | Estructural | Agregar funcionalidad dinámicamente | Auth, logging, cache |
| **Observer** | Comportamental | Notificar cambios | Signals, eventos, suscripciones |
| **Strategy** | Comportamental | Algoritmos intercambiables | Múltiples formas de hacer algo |
| **Repository** | Comportamental | Abstraer acceso a datos | Desacoplar BD de lógica |

---

## Antipatrones (qué NO hacer)

### ❌ God Object

```python
# ❌ Clase que hace TODO
class Usuario:
    def __init__(self):
        pass
    
    def save_to_database(self):
        pass
    
    def send_email(self):
        pass
    
    def generate_pdf(self):
        pass
    
    def calculate_taxes(self):
        pass
    
    # ... 50 métodos más
```

**Solución:** Aplicar Single Responsibility Principle

---

### ❌ Spaghetti Code

```python
# ❌ Código sin estructura
def procesar():
    if x:
        if y:
            if z:
                # ...
                if a:
                    # ...
                    if b:
                        # ... 10 niveles de if
```

**Solución:** Early returns, separar en funciones

```python
# ✅ Código estructurado
def procesar():
    if not x:
        return
    
    if not y:
        return
    
    procesar_z()
    procesar_a()
```

---

## Recursos

- **[Design Patterns](https://refactoring.guru/design-patterns)** - Refactoring Guru
- **[Game Programming Patterns](https://gameprogrammingpatterns.com/)** - Robert Nystrom
- **[Python Patterns](https://python-patterns.guide/)** - Brandon Rhodes

**Próximo módulo:** README de Arquitectura

¡Design Patterns completo! 🎨

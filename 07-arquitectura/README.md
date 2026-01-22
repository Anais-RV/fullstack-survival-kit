# Arquitectura

> **Diseñar aplicaciones FullStack escalables y mantenibles**

---

## ¿Por qué arquitectura?

En proyectos FullStack, la **arquitectura** es crítica:

- ✅ **Escalabilidad:** Crecer sin reescribir todo
- ✅ **Mantenibilidad:** Fácil de modificar y extender
- ✅ **Testeable:** Tests automatizados confiables
- ✅ **Colaboración:** Equipos pueden trabajar en paralelo
- ✅ **Flexibilidad:** Cambiar tecnologías sin dolor

---

## Contenido del módulo

### 01 - Clean Architecture
**Separar código en capas con responsabilidades claras**

#### Conceptos clave:
- **Entities:** Objetos de negocio puros (reglas empresariales)
- **Use Cases:** Lógica de aplicación (casos de uso)
- **Adapters:** Conversión entre capas (serializers, repositorios)
- **Frameworks:** Django, React, PostgreSQL (detalles técnicos)

#### Estructura:
```
backend/
├── core/                    # Lógica de negocio (independiente)
│   ├── entities/
│   └── use_cases/
├── adapters/                # Conversión de datos
│   ├── serializers/
│   ├── repositories/
│   └── views/
└── infrastructure/          # Frameworks (Django, PostgreSQL)
```

#### Principio fundamental:
> **Las dependencias apuntan hacia adentro**

#### Beneficios:
- Independiente de frameworks (cambiar Django por FastAPI es fácil)
- Testeable sin base de datos (mock de repositorios)
- Lógica de negocio protegida de cambios externos

**Tiempo estimado:** 3 horas

---

### 02 - SOLID Principles
**5 principios para código orientado a objetos mantenible**

#### S - Single Responsibility Principle
> Una clase debe tener una sola razón para cambiar

```python
# ❌ Mal: Clase con múltiples responsabilidades
class Usuario(models.Model):
    def save(self):
        super().save()
        self.send_email()      # ❌ Responsabilidad extra
        self.write_log()       # ❌ Otra responsabilidad

# ✅ Bien: Separar responsabilidades
class Usuario(models.Model):
    pass  # Solo persistencia

class EmailService:
    def enviar_bienvenida(self, usuario):
        pass  # Solo emails
```

#### O - Open/Closed Principle
> Abierto para extensión, cerrado para modificación

```python
# ✅ Estrategias de descuento (extensible)
class DescuentoStrategy(ABC):
    @abstractmethod
    def aplicar(self, precio): pass

class DescuentoPremium(DescuentoStrategy):
    def aplicar(self, precio):
        return precio * 0.9  # 10%

# Agregar nuevos descuentos sin modificar código existente
class DescuentoBlackFriday(DescuentoStrategy):
    def aplicar(self, precio):
        return precio * 0.5  # 50%
```

#### L - Liskov Substitution Principle
> Subclases deben ser sustituibles por sus clases base

#### I - Interface Segregation Principle
> No obligar a implementar interfaces innecesarias

#### D - Dependency Inversion Principle
> Depender de abstracciones, no de implementaciones concretas

```python
# ✅ Depender de abstracción
class ProductoService:
    def __init__(self, repo: ProductoRepository):  # Interface
        self.repo = repo

# Inyectar implementación concreta
repo_django = ProductoRepositoryDjango()
service = ProductoService(repo_django)
```

**Tiempo estimado:** 2 horas

---

### 03 - Design Patterns
**Soluciones probadas a problemas comunes**

#### Patrones Creacionales

**Singleton:** Una sola instancia
```python
class Database:
    _instance = None
    
    def __new__(cls):
        if cls._instance is None:
            cls._instance = super().__new__(cls)
        return cls._instance
```

**Factory:** Crear objetos sin especificar clase exacta
```python
class PagoFactory:
    @staticmethod
    def crear_pago(tipo, monto):
        pagos = {
            'tarjeta': PagoTarjeta,
            'paypal': PagoPayPal
        }
        return pagos[tipo](monto)
```

#### Patrones Estructurales

**Adapter:** Hacer interfaces compatibles
```python
class AdaptadorPagoAntiguo:
    def __init__(self, sistema_antiguo):
        self.sistema_antiguo = sistema_antiguo
    
    def procesar(self, datos):
        # Adaptar estructura de datos
        return self.sistema_antiguo.realizar_pago(datos['tarjeta'], datos['monto'])
```

**Decorator:** Agregar funcionalidad dinámicamente
```python
@require_premium
@cache_result(timeout=300)
def vista_premium(request):
    return Response({'data': '...'})
```

#### Patrones Comportamentales

**Observer:** Notificar cambios
```python
# Django Signals
@receiver(post_save, sender=Producto)
def producto_creado(sender, instance, created, **kwargs):
    if created:
        enviar_email_admin(instance)
        actualizar_cache()
```

**Strategy:** Algoritmos intercambiables
```python
class PrecioService:
    def __init__(self, descuento: DescuentoStrategy):
        self.descuento = descuento
    
    def calcular(self, precio):
        return self.descuento.aplicar(precio)
```

**Repository:** Abstraer acceso a datos
```python
class ProductoRepository(ABC):
    @abstractmethod
    def obtener_todos(self): pass
    
    @abstractmethod
    def guardar(self, producto): pass
```

**Tiempo estimado:** 3 horas

---

## Arquitectura de proyecto FullStack

### Backend (Django)

```
backend/
├── core/                              # Lógica de negocio
│   ├── entities/
│   │   ├── producto.py               # Entidades puras
│   │   └── usuario.py
│   ├── use_cases/
│   │   ├── crear_producto.py         # Casos de uso
│   │   ├── comprar_producto.py
│   │   └── listar_productos.py
│   └── interfaces/
│       └── producto_repository.py    # Abstracciones
│
├── adapters/                          # Adaptadores
│   ├── serializers/
│   │   └── producto_serializer.py    # DRF Serializers
│   ├── repositories/
│   │   ├── producto_repo_django.py   # Django ORM
│   │   └── producto_repo_memory.py   # Tests
│   ├── views/
│   │   └── producto_views.py         # DRF ViewSets
│   └── strategies/
│       └── descuento_strategy.py     # Estrategias
│
├── infrastructure/                    # Frameworks
│   ├── models.py                     # Django Models
│   ├── settings.py
│   └── urls.py
│
├── tests/
│   ├── test_entities.py
│   ├── test_use_cases.py
│   └── test_repositories.py
│
└── manage.py
```

### Frontend (React)

```
frontend/
├── src/
│   ├── domain/                       # Lógica de negocio
│   │   ├── entities/
│   │   │   └── Producto.js          # Clases de negocio
│   │   ├── use-cases/
│   │   │   └── crearProducto.js     # Casos de uso
│   │   └── interfaces/
│   │       └── ProductoRepository.js # Abstracciones
│   │
│   ├── infrastructure/               # Implementaciones
│   │   ├── api/
│   │   │   └── ProductoApiHTTP.js   # HTTP client
│   │   └── repositories/
│   │       └── ProductoRepoHTTP.js  # Repository HTTP
│   │
│   ├── presentation/                 # UI
│   │   ├── components/
│   │   │   ├── ProductoForm.jsx     # Componentes UI
│   │   │   └── ProductoCard.jsx
│   │   ├── pages/
│   │   │   ├── ProductosPage.jsx
│   │   │   └── HomePage.jsx
│   │   └── hooks/
│   │       └── useProductos.js      # Custom hooks
│   │
│   └── App.jsx
│
└── package.json
```

---

## Flujo de una request

### Backend: Crear Producto

```
1. HTTP POST /api/productos/
   ↓
2. ProductoView (adapters/views/)
   - Recibe request
   - Valida con serializer
   ↓
3. CrearProducto Use Case (core/use_cases/)
   - Validaciones de negocio
   - Lógica de aplicación
   ↓
4. ProductoRepository (adapters/repositories/)
   - Persistencia en BD
   ↓
5. ProductoModel (infrastructure/models/)
   - Django ORM
   - PostgreSQL
```

### Frontend: Crear Producto

```
1. ProductoForm (presentation/components/)
   - Usuario llena formulario
   - handleSubmit
   ↓
2. CrearProducto Use Case (domain/use-cases/)
   - Validaciones de negocio
   ↓
3. ProductoRepository (infrastructure/repositories/)
   - HTTP POST con Axios
   ↓
4. Backend API
   ↓
5. Actualizar UI
   - Mostrar notificación
   - Redirigir a lista
```

---

## Principios de arquitectura FullStack

### 1. Separation of Concerns
> Separar UI, lógica de negocio y persistencia

```
UI (React)  →  Business Logic (Use Cases)  →  Data (Repository)  →  BD
```

### 2. Dependency Inversion
> Depender de abstracciones

```python
# ✅ Use case depende de interface, no de implementación
class CrearProducto:
    def __init__(self, repo: ProductoRepository):  # Interface
        self.repo = repo
```

### 3. Don't Repeat Yourself (DRY)
> Reutilizar código

```javascript
// ✅ Custom hook reutilizable
function useProductos() {
    const [productos, setProductos] = useState([]);
    // Lógica reutilizable
    return { productos, loading, error };
}
```

### 4. Keep It Simple, Stupid (KISS)
> Preferir simplicidad

```python
# ✅ Simple y claro
def calcular_precio_final(precio, descuento):
    return precio * (1 - descuento / 100)

# ❌ Sobreingeniería
class PriceCalculatorFactory:
    def create_calculator(self, strategy_type):
        # ... 50 líneas innecesarias
```

### 5. You Aren't Gonna Need It (YAGNI)
> No implementar funcionalidad hasta que sea necesaria

---

## Cuándo aplicar arquitectura avanzada

### ✅ Usar Clean Architecture + SOLID + Patterns

**Proyectos medianos/grandes:**
- 3+ desarrolladores
- Lógica de negocio compleja
- Larga duración (años)
- Múltiples integraciones
- Alto nivel de testing

**Ejemplos:**
- E-commerce
- CRM/ERP
- Plataformas SaaS
- Sistemas bancarios

---

### ❌ NO usar arquitectura avanzada

**Proyectos pequeños:**
- MVPs o prototipos
- Menos de 10 endpoints
- 1-2 desarrolladores
- Corta duración (semanas)
- CRUD simple sin lógica compleja

**Ejemplos:**
- Landing pages
- Blogs personales
- Proof of concepts
- Scripts one-off

**En estos casos:**
```python
# ✅ Suficiente para proyecto pequeño
# views.py
from rest_framework import viewsets
from .models import Producto
from .serializers import ProductoSerializer

class ProductoViewSet(viewsets.ModelViewSet):
    queryset = Producto.objects.all()
    serializer_class = ProductoSerializer
```

---

## Testing de arquitectura

### Tests de Entidades (core/)

```python
def test_producto_esta_disponible():
    producto = Producto(id=1, nombre="Test", precio=100, stock=10)
    assert producto.esta_disponible() == True
    
    producto.stock = 0
    assert producto.esta_disponible() == False
```

### Tests de Use Cases (core/)

```python
def test_crear_producto():
    repo_mock = Mock(spec=ProductoRepository)
    repo_mock.guardar.return_value = Producto(1, "Test", 100, 10)
    
    use_case = CrearProducto(repo_mock)
    producto = use_case.execute("Test", 100, 10)
    
    assert producto.nombre == "Test"
    repo_mock.guardar.assert_called_once()
```

### Tests de Repositories (adapters/)

```python
@pytest.mark.django_db
def test_producto_repository_guardar():
    repo = ProductoRepositoryDjango()
    producto = Producto(0, "Test", 100, 10)
    
    producto_guardado = repo.guardar(producto)
    
    assert producto_guardado.id > 0
    assert ProductoModel.objects.filter(nombre="Test").exists()
```

---

## Evolución de arquitectura

### Fase 1: MVP (Simple)
```
Django Views + React Components
```

### Fase 2: Crecimiento (Modular)
```
Separar en Services
Agregar Custom Hooks
```

### Fase 3: Escalabilidad (Clean Architecture)
```
Entities + Use Cases + Repositories
SOLID + Design Patterns
```

**Consejo:** Empezar simple y refactorizar cuando sea necesario

---

## Recursos

### Libros
- **"Clean Architecture"** - Robert C. Martin (Uncle Bob)
- **"Clean Code"** - Robert C. Martin
- **"Design Patterns"** - Gang of Four
- **"Domain-Driven Design"** - Eric Evans

### Artículos
- **[Clean Architecture](https://blog.cleancoder.com/uncle-bob/2012/08/13/the-clean-architecture.html)** - Uncle Bob
- **[SOLID Principles](https://en.wikipedia.org/wiki/SOLID)** - Wikipedia
- **[Design Patterns](https://refactoring.guru/design-patterns)** - Refactoring Guru

### Cursos
- **[Architecture Patterns with Python](https://www.cosmicpython.com/)** - Harry Percival
- **[Clean Architecture in React](https://dev.to/bespoyasov/clean-architecture-on-frontend-4311)** - Bespoyasov

---

## Resumen

**Arquitectura = Organizar código para que sea:**
- **Escalable** (agregar features sin reescribir)
- **Mantenible** (fácil de entender y modificar)
- **Testeable** (automatizar tests confiables)
- **Flexible** (cambiar tecnologías sin dolor)

**Principios clave:**
1. **Clean Architecture:** Separar en capas (Entities, Use Cases, Adapters, Frameworks)
2. **SOLID:** 5 principios de diseño orientado a objetos
3. **Design Patterns:** Soluciones probadas a problemas comunes

**Regla de oro:** Empezar simple y aplicar arquitectura avanzada solo cuando sea necesario

---

## Próximo módulo: 08-despliegue

En el próximo módulo aprenderemos:
- Docker y Docker Compose
- CI/CD con GitHub Actions
- Deployment automatizado
- Monitoring y logs
- Security best practices

¡Arquitectura completa! 🏗️✨

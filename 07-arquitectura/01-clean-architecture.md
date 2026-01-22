# Clean Architecture

> **Organizar código en capas independientes y mantenibles**

---

## ¿Qué es Clean Architecture?

**Clean Architecture** = Separar el código en capas con responsabilidades claras

```
    UI (React)
       ↓
  Controllers (Django Views)
       ↓
   Use Cases (Business Logic)
       ↓
    Entities (Models)
       ↓
   Database (PostgreSQL)
```

**Principio clave:** **Las dependencias apuntan hacia adentro**

---

## Capas de Clean Architecture

### 1. Entities (Entidades) - Centro

**¿Qué son?**
- Objetos de negocio puros
- Reglas de negocio empresariales
- Independientes de frameworks

```python
# core/entities/producto.py
from dataclasses import dataclass
from decimal import Decimal

@dataclass
class Producto:
    id: int
    nombre: str
    precio: Decimal
    stock: int
    
    def esta_disponible(self) -> bool:
        """Regla de negocio: disponible si hay stock"""
        return self.stock > 0
    
    def aplicar_descuento(self, porcentaje: int) -> Decimal:
        """Regla de negocio: calcular precio con descuento"""
        if not 0 <= porcentaje <= 100:
            raise ValueError("Porcentaje debe estar entre 0 y 100")
        return self.precio * (1 - porcentaje / 100)
    
    def puede_venderse(self, cantidad: int) -> bool:
        """Regla de negocio: validar cantidad disponible"""
        return self.stock >= cantidad
```

---

### 2. Use Cases (Casos de Uso) - Lógica de aplicación

**¿Qué son?**
- Orquestan el flujo de datos
- Implementan reglas de negocio de la aplicación
- Independientes de UI y frameworks

```python
# core/use_cases/crear_producto.py
from typing import Protocol
from decimal import Decimal

class ProductoRepository(Protocol):
    """Interface para el repositorio (abstracción)"""
    def guardar(self, producto: Producto) -> Producto: ...
    def existe_nombre(self, nombre: str) -> bool: ...

class CrearProducto:
    def __init__(self, producto_repo: ProductoRepository):
        self.producto_repo = producto_repo
    
    def execute(self, nombre: str, precio: Decimal, stock: int) -> Producto:
        # Validaciones de negocio
        if self.producto_repo.existe_nombre(nombre):
            raise ValueError(f"Ya existe un producto con nombre '{nombre}'")
        
        if precio <= 0:
            raise ValueError("El precio debe ser mayor a 0")
        
        if stock < 0:
            raise ValueError("El stock no puede ser negativo")
        
        # Crear entidad
        producto = Producto(
            id=0,  # Se asignará en BD
            nombre=nombre,
            precio=precio,
            stock=stock
        )
        
        # Guardar usando repositorio
        return self.producto_repo.guardar(producto)
```

```python
# core/use_cases/comprar_producto.py
class ComprarProducto:
    def __init__(self, producto_repo: ProductoRepository):
        self.producto_repo = producto_repo
    
    def execute(self, producto_id: int, cantidad: int) -> Producto:
        # Obtener producto
        producto = self.producto_repo.obtener_por_id(producto_id)
        if not producto:
            raise ValueError("Producto no encontrado")
        
        # Validar disponibilidad (regla de negocio)
        if not producto.puede_venderse(cantidad):
            raise ValueError(f"Stock insuficiente. Disponible: {producto.stock}")
        
        # Reducir stock
        producto.stock -= cantidad
        
        # Guardar cambios
        return self.producto_repo.actualizar(producto)
```

---

### 3. Interface Adapters (Adaptadores) - Conversión de datos

**¿Qué son?**
- Convierten datos entre capas
- Serializers, ViewSets, Services
- Adaptan entidades para UI o BD

```python
# adapters/serializers/producto_serializer.py
from rest_framework import serializers
from core.entities.producto import Producto

class ProductoSerializer(serializers.Serializer):
    id = serializers.IntegerField(read_only=True)
    nombre = serializers.CharField(max_length=200)
    precio = serializers.DecimalField(max_digits=10, decimal_places=2)
    stock = serializers.IntegerField()
    
    def to_entity(self) -> Producto:
        """Convertir de dict a Entidad"""
        return Producto(
            id=self.validated_data.get('id', 0),
            nombre=self.validated_data['nombre'],
            precio=self.validated_data['precio'],
            stock=self.validated_data['stock']
        )
    
    def from_entity(self, producto: Producto) -> dict:
        """Convertir de Entidad a dict"""
        return {
            'id': producto.id,
            'nombre': producto.nombre,
            'precio': producto.precio,
            'stock': producto.stock,
            'disponible': producto.esta_disponible()
        }
```

---

### 4. Frameworks & Drivers (Capa externa) - Detalles técnicos

**¿Qué son?**
- Django, React, PostgreSQL
- APIs, bases de datos, UI
- Fáciles de reemplazar

```python
# adapters/repositories/producto_repository_django.py
from django.db import models
from core.use_cases.crear_producto import ProductoRepository
from core.entities.producto import Producto

class ProductoModel(models.Model):
    """ORM de Django (detalle de implementación)"""
    nombre = models.CharField(max_length=200)
    precio = models.DecimalField(max_digits=10, decimal_places=2)
    stock = models.IntegerField()
    
    class Meta:
        db_table = 'productos'

class ProductoRepositoryDjango(ProductoRepository):
    """Implementación concreta del repositorio usando Django ORM"""
    
    def guardar(self, producto: Producto) -> Producto:
        producto_model = ProductoModel.objects.create(
            nombre=producto.nombre,
            precio=producto.precio,
            stock=producto.stock
        )
        producto.id = producto_model.id
        return producto
    
    def existe_nombre(self, nombre: str) -> bool:
        return ProductoModel.objects.filter(nombre=nombre).exists()
    
    def obtener_por_id(self, producto_id: int) -> Producto | None:
        try:
            p = ProductoModel.objects.get(id=producto_id)
            return Producto(id=p.id, nombre=p.nombre, precio=p.precio, stock=p.stock)
        except ProductoModel.DoesNotExist:
            return None
    
    def actualizar(self, producto: Producto) -> Producto:
        ProductoModel.objects.filter(id=producto.id).update(
            nombre=producto.nombre,
            precio=producto.precio,
            stock=producto.stock
        )
        return producto
```

---

## Vista general del proyecto

```
backend/
├── core/                              # Lógica de negocio (independiente)
│   ├── entities/
│   │   ├── producto.py               # Entidades puras
│   │   └── usuario.py
│   └── use_cases/
│       ├── crear_producto.py         # Casos de uso
│       ├── comprar_producto.py
│       └── listar_productos.py
│
├── adapters/                          # Adaptadores (conversión)
│   ├── serializers/
│   │   └── producto_serializer.py    # DRF Serializers
│   ├── repositories/
│   │   └── producto_repository_django.py  # Django ORM
│   └── views/
│       └── producto_views.py         # DRF ViewSets
│
├── infrastructure/                    # Frameworks & Drivers
│   ├── models.py                     # Django Models
│   ├── settings.py
│   └── urls.py
│
└── manage.py
```

---

## Implementación completa: Django

### 1. Entidad

```python
# core/entities/producto.py
from dataclasses import dataclass
from decimal import Decimal

@dataclass
class Producto:
    id: int
    nombre: str
    precio: Decimal
    stock: int
    
    def esta_disponible(self) -> bool:
        return self.stock > 0
```

### 2. Use Case

```python
# core/use_cases/crear_producto.py
from typing import Protocol
from core.entities.producto import Producto

class ProductoRepository(Protocol):
    def guardar(self, producto: Producto) -> Producto: ...

class CrearProducto:
    def __init__(self, repo: ProductoRepository):
        self.repo = repo
    
    def execute(self, nombre: str, precio: Decimal, stock: int) -> Producto:
        if precio <= 0:
            raise ValueError("Precio debe ser mayor a 0")
        
        producto = Producto(id=0, nombre=nombre, precio=precio, stock=stock)
        return self.repo.guardar(producto)
```

### 3. Repository

```python
# adapters/repositories/producto_repository_django.py
from infrastructure.models import ProductoModel
from core.entities.producto import Producto

class ProductoRepositoryDjango:
    def guardar(self, producto: Producto) -> Producto:
        p = ProductoModel.objects.create(
            nombre=producto.nombre,
            precio=producto.precio,
            stock=producto.stock
        )
        producto.id = p.id
        return producto
```

### 4. View

```python
# adapters/views/producto_views.py
from rest_framework.views import APIView
from rest_framework.response import Response
from core.use_cases.crear_producto import CrearProducto
from adapters.repositories.producto_repository_django import ProductoRepositoryDjango
from adapters.serializers.producto_serializer import ProductoSerializer

class CrearProductoView(APIView):
    def post(self, request):
        serializer = ProductoSerializer(data=request.data)
        serializer.is_valid(raise_exception=True)
        
        # Inyectar dependencias
        repo = ProductoRepositoryDjango()
        use_case = CrearProducto(repo)
        
        # Ejecutar caso de uso
        producto = use_case.execute(
            nombre=serializer.validated_data['nombre'],
            precio=serializer.validated_data['precio'],
            stock=serializer.validated_data['stock']
        )
        
        # Devolver respuesta
        response_serializer = ProductoSerializer()
        return Response(response_serializer.from_entity(producto), status=201)
```

---

## Implementación completa: React

### 1. Entity (Frontend)

```javascript
// src/domain/entities/Producto.js
export class Producto {
    constructor(id, nombre, precio, stock) {
        this.id = id;
        this.nombre = nombre;
        this.precio = precio;
        this.stock = stock;
    }
    
    estaDisponible() {
        return this.stock > 0;
    }
    
    aplicarDescuento(porcentaje) {
        if (porcentaje < 0 || porcentaje > 100) {
            throw new Error('Porcentaje inválido');
        }
        return this.precio * (1 - porcentaje / 100);
    }
}
```

### 2. Use Case (Frontend)

```javascript
// src/domain/use-cases/crearProducto.js
export class CrearProducto {
    constructor(productoRepository) {
        this.productoRepository = productoRepository;
    }
    
    async execute(nombre, precio, stock) {
        // Validaciones
        if (!nombre || nombre.trim() === '') {
            throw new Error('El nombre es requerido');
        }
        
        if (precio <= 0) {
            throw new Error('El precio debe ser mayor a 0');
        }
        
        // Llamar al repositorio
        const producto = await this.productoRepository.crear({
            nombre,
            precio,
            stock
        });
        
        return new Producto(
            producto.id,
            producto.nombre,
            producto.precio,
            producto.stock
        );
    }
}
```

### 3. Repository (Frontend)

```javascript
// src/infrastructure/repositories/ProductoRepositoryHTTP.js
import api from '../api';

export class ProductoRepositoryHTTP {
    async crear(data) {
        const response = await api.post('/api/productos/', data);
        return response.data;
    }
    
    async obtenerTodos() {
        const response = await api.get('/api/productos/');
        return response.data;
    }
    
    async obtenerPorId(id) {
        const response = await api.get(`/api/productos/${id}/`);
        return response.data;
    }
    
    async actualizar(id, data) {
        const response = await api.put(`/api/productos/${id}/`, data);
        return response.data;
    }
    
    async eliminar(id) {
        await api.delete(`/api/productos/${id}/`);
    }
}
```

### 4. Component (UI)

```jsx
// src/presentation/components/ProductoForm.jsx
import { useState } from 'react';
import { CrearProducto } from '../../domain/use-cases/crearProducto';
import { ProductoRepositoryHTTP } from '../../infrastructure/repositories/ProductoRepositoryHTTP';

export function ProductoForm() {
    const [nombre, setNombre] = useState('');
    const [precio, setPrecio] = useState('');
    const [stock, setStock] = useState('');
    const [loading, setLoading] = useState(false);
    
    const handleSubmit = async (e) => {
        e.preventDefault();
        setLoading(true);
        
        try {
            // Inyectar dependencias
            const repo = new ProductoRepositoryHTTP();
            const useCase = new CrearProducto(repo);
            
            // Ejecutar caso de uso
            await useCase.execute(nombre, parseFloat(precio), parseInt(stock));
            
            alert('Producto creado!');
        } catch (error) {
            alert(error.message);
        } finally {
            setLoading(false);
        }
    };
    
    return (
        <form onSubmit={handleSubmit}>
            <input 
                value={nombre}
                onChange={(e) => setNombre(e.target.value)}
                placeholder="Nombre"
            />
            <input 
                type="number"
                value={precio}
                onChange={(e) => setPrecio(e.target.value)}
                placeholder="Precio"
            />
            <input 
                type="number"
                value={stock}
                onChange={(e) => setStock(e.target.value)}
                placeholder="Stock"
            />
            <button type="submit" disabled={loading}>
                {loading ? 'Creando...' : 'Crear Producto'}
            </button>
        </form>
    );
}
```

---

## Ventajas de Clean Architecture

✅ **Independencia de frameworks**
```python
# Cambiar Django por FastAPI es fácil
# Solo cambias la capa de adapters/infrastructure
```

✅ **Testeable**
```python
# Test sin base de datos (mock del repositorio)
def test_crear_producto():
    repo_mock = Mock(spec=ProductoRepository)
    repo_mock.guardar.return_value = Producto(1, "Test", 100, 10)
    
    use_case = CrearProducto(repo_mock)
    producto = use_case.execute("Test", 100, 10)
    
    assert producto.nombre == "Test"
```

✅ **Escalable**
```
# Agregar nueva feature:
# 1. Crear entidad
# 2. Crear use case
# 3. Crear adapter (repository, serializer)
# 4. Crear view
```

✅ **Fácil de entender**
```
# Leer el código:
# 1. Empezar por entities (¿qué objetos maneja?)
# 2. Leer use_cases (¿qué hace la app?)
# 3. Ver adapters (¿cómo se conecta?)
```

---

## Cuándo usar Clean Architecture

✅ **Proyectos medianos/grandes**
- Equipos de 3+ desarrolladores
- Aplicaciones con lógica de negocio compleja
- Proyectos de larga duración

❌ **No usar en:**
- MVPs o prototipos rápidos
- Proyectos pequeños (<10 endpoints)
- CRUDs simples sin lógica de negocio

---

## Comparación: Sin vs Con Clean Architecture

### Sin Clean Architecture (tradicional)

```python
# views.py (todo mezclado)
from django.shortcuts import render
from .models import Producto

def crear_producto(request):
    if request.method == 'POST':
        nombre = request.POST['nombre']
        precio = request.POST['precio']
        stock = request.POST['stock']
        
        # Validación mezclada con lógica
        if float(precio) <= 0:
            return render(request, 'error.html')
        
        # Acceso directo a BD
        Producto.objects.create(
            nombre=nombre,
            precio=precio,
            stock=stock
        )
        
        return render(request, 'success.html')
```

**Problemas:**
- ❌ Difícil de testear (requiere request)
- ❌ Validación mezclada con vista
- ❌ Acoplado a Django
- ❌ No reutilizable

---

### Con Clean Architecture

```python
# core/use_cases/crear_producto.py (lógica pura)
class CrearProducto:
    def __init__(self, repo):
        self.repo = repo
    
    def execute(self, nombre, precio, stock):
        if precio <= 0:
            raise ValueError("Precio inválido")
        
        producto = Producto(0, nombre, precio, stock)
        return self.repo.guardar(producto)

# adapters/views/producto_views.py (solo HTTP)
class CrearProductoView(APIView):
    def post(self, request):
        repo = ProductoRepositoryDjango()
        use_case = CrearProducto(repo)
        
        producto = use_case.execute(
            nombre=request.data['nombre'],
            precio=request.data['precio'],
            stock=request.data['stock']
        )
        
        return Response({'id': producto.id}, status=201)
```

**Beneficios:**
- ✅ Fácil de testear (sin request)
- ✅ Lógica separada de HTTP
- ✅ Independiente de Django
- ✅ Reutilizable en CLI, background jobs, etc.

---

## Resumen

**Clean Architecture = Separar en capas con dependencias hacia adentro**

```
┌─────────────────────────────────┐
│   UI (React/Django Views)       │ ← Frameworks
├─────────────────────────────────┤
│   Adapters (Serializers/Repos)  │ ← Conversión
├─────────────────────────────────┤
│   Use Cases (Business Logic)    │ ← Lógica de app
├─────────────────────────────────┤
│   Entities (Core Domain)        │ ← Reglas de negocio
└─────────────────────────────────┘
```

**Regla de oro:** Las capas internas NO conocen las externas

**Próximo módulo:** SOLID Principles

---

## Recursos

- **[Clean Architecture](https://blog.cleancoder.com/uncle-bob/2012/08/13/the-clean-architecture.html)** - Uncle Bob
- **[Clean Architecture in Python](https://www.thedigitalcatonline.com/blog/2016/11/14/clean-architectures-in-python-a-step-by-step-example/)** - Leonardo Giordani
- **[Clean Architecture in React](https://dev.to/bespoyasov/clean-architecture-on-frontend-4311)** - Bespoyasov

¡Clean Architecture lista! 🏗️

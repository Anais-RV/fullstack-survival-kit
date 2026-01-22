# Proyecto Integrador: E-commerce FullStack

> **Aplicación completa integrando todo lo aprendido**

---

## Descripción del proyecto

**E-commerce completo** con:
- 🛒 Catálogo de productos
- 🔐 Autenticación y registro
- 🛍️ Carrito de compras
- 💳 Proceso de checkout
- 👤 Perfil de usuario y órdenes
- 🔍 Búsqueda y filtros
- 📦 Panel de administración

---

## Stack tecnológico

### Backend
- **Django 5.0** → Framework web
- **Django REST Framework** → API REST
- **PostgreSQL 15** → Base de datos
- **Django Simple JWT** → Autenticación
- **Gunicorn** → WSGI server

### Frontend
- **React 18** → UI library
- **Vite** → Build tool
- **React Router** → Routing
- **Zustand** → Estado global
- **Axios** → HTTP client
- **Tailwind CSS** → Estilos

### Testing
- **Pytest** → Backend tests
- **Vitest + React Testing Library** → Frontend tests
- **Playwright** → E2E tests

### Deployment
- **Docker** → Containerización
- **GitHub Actions** → CI/CD
- **Railway** → Backend hosting
- **Vercel** → Frontend hosting

---

## Arquitectura

```
frontend/ (React + Vite)
  → HTTP requests →
backend/ (Django REST Framework)
  → ORM →
PostgreSQL Database

CORS → JWT Auth → REST API → CRUD
```

---

## Estructura del proyecto

```
ecommerce-fullstack/
├── backend/
│   ├── core/                       # Django project
│   │   ├── settings.py
│   │   ├── urls.py
│   │   └── wsgi.py
│   ├── usuarios/                   # Auth app
│   │   ├── models.py              # Usuario custom
│   │   ├── serializers.py
│   │   ├── views.py
│   │   └── urls.py
│   ├── productos/                  # Productos app
│   │   ├── models.py              # Producto, Categoria
│   │   ├── serializers.py
│   │   ├── views.py
│   │   ├── filters.py
│   │   └── urls.py
│   ├── ordenes/                    # Órdenes app
│   │   ├── models.py              # Orden, OrdenItem
│   │   ├── serializers.py
│   │   ├── views.py
│   │   └── urls.py
│   ├── tests/
│   │   ├── test_usuarios.py
│   │   ├── test_productos.py
│   │   └── test_ordenes.py
│   ├── requirements.txt
│   ├── Dockerfile
│   └── manage.py
│
├── frontend/
│   ├── src/
│   │   ├── components/
│   │   │   ├── Navbar.jsx
│   │   │   ├── ProductoCard.jsx
│   │   │   ├── Carrito.jsx
│   │   │   └── ...
│   │   ├── pages/
│   │   │   ├── HomePage.jsx
│   │   │   ├── LoginPage.jsx
│   │   │   ├── ProductosPage.jsx
│   │   │   ├── ProductoDetailPage.jsx
│   │   │   ├── CarritoPage.jsx
│   │   │   ├── CheckoutPage.jsx
│   │   │   ├── PerfilPage.jsx
│   │   │   └── OrdenesPage.jsx
│   │   ├── services/
│   │   │   ├── api.js
│   │   │   ├── authService.js
│   │   │   ├── productoService.js
│   │   │   └── ordenService.js
│   │   ├── stores/
│   │   │   ├── authStore.js
│   │   │   ├── carritoStore.js
│   │   │   └── notificationStore.js
│   │   ├── App.jsx
│   │   └── main.jsx
│   ├── e2e/
│   │   ├── auth.spec.js
│   │   ├── productos.spec.js
│   │   └── checkout.spec.js
│   ├── Dockerfile
│   ├── nginx.conf
│   └── package.json
│
├── docker-compose.yml
├── .github/
│   └── workflows/
│       ├── backend.yml
│       ├── frontend.yml
│       └── e2e.yml
└── README.md
```

---

## Modelos de datos

### Usuario (usuarios/models.py)

```python
from django.contrib.auth.models import AbstractUser
from django.db import models

class Usuario(AbstractUser):
    email = models.EmailField(unique=True)
    telefono = models.CharField(max_length=20, blank=True)
    direccion = models.TextField(blank=True)
    ciudad = models.CharField(max_length=100, blank=True)
    codigo_postal = models.CharField(max_length=10, blank=True)
    
    def __str__(self):
        return self.username
```

---

### Producto (productos/models.py)

```python
class Categoria(models.Model):
    nombre = models.CharField(max_length=100, unique=True)
    descripcion = models.TextField(blank=True)
    
    class Meta:
        verbose_name_plural = 'Categorías'
    
    def __str__(self):
        return self.nombre

class Producto(models.Model):
    nombre = models.CharField(max_length=200)
    descripcion = models.TextField()
    precio = models.DecimalField(max_digits=10, decimal_places=2)
    stock = models.IntegerField(default=0)
    categoria = models.ForeignKey(Categoria, on_delete=models.CASCADE, related_name='productos')
    imagen = models.ImageField(upload_to='productos/', blank=True, null=True)
    destacado = models.BooleanField(default=False)
    activo = models.BooleanField(default=True)
    creado = models.DateTimeField(auto_now_add=True)
    actualizado = models.DateTimeField(auto_now=True)
    
    class Meta:
        ordering = ['-creado']
    
    def __str__(self):
        return self.nombre
    
    def esta_disponible(self):
        return self.stock > 0 and self.activo
```

---

### Orden (ordenes/models.py)

```python
class Orden(models.Model):
    ESTADO_CHOICES = [
        ('pendiente', 'Pendiente'),
        ('procesando', 'Procesando'),
        ('enviado', 'Enviado'),
        ('entregado', 'Entregado'),
        ('cancelado', 'Cancelado'),
    ]
    
    usuario = models.ForeignKey(Usuario, on_delete=models.CASCADE, related_name='ordenes')
    estado = models.CharField(max_length=20, choices=ESTADO_CHOICES, default='pendiente')
    total = models.DecimalField(max_digits=10, decimal_places=2)
    direccion_envio = models.TextField()
    ciudad = models.CharField(max_length=100)
    codigo_postal = models.CharField(max_length=10)
    creado = models.DateTimeField(auto_now_add=True)
    actualizado = models.DateTimeField(auto_now=True)
    
    class Meta:
        ordering = ['-creado']
        verbose_name_plural = 'Órdenes'
    
    def __str__(self):
        return f'Orden #{self.id} - {self.usuario.username}'

class OrdenItem(models.Model):
    orden = models.ForeignKey(Orden, on_delete=models.CASCADE, related_name='items')
    producto = models.ForeignKey(Producto, on_delete=models.CASCADE)
    cantidad = models.IntegerField()
    precio = models.DecimalField(max_digits=10, decimal_places=2)
    
    def __str__(self):
        return f'{self.cantidad}x {self.producto.nombre}'
    
    def subtotal(self):
        return self.cantidad * self.precio
```

---

## API Endpoints

### Autenticación
```
POST   /api/usuarios/register/       # Registrar usuario
POST   /api/usuarios/login/          # Login (obtener tokens)
POST   /api/usuarios/token/refresh/  # Refrescar access token
GET    /api/usuarios/perfil/         # Obtener perfil
PUT    /api/usuarios/perfil/         # Actualizar perfil
```

### Productos
```
GET    /api/productos/               # Listar productos
GET    /api/productos/{id}/          # Detalle de producto
GET    /api/productos/destacados/    # Productos destacados
GET    /api/productos/categorias/    # Listar categorías
```

### Órdenes
```
GET    /api/ordenes/                 # Mis órdenes
POST   /api/ordenes/                 # Crear orden
GET    /api/ordenes/{id}/            # Detalle de orden
PATCH  /api/ordenes/{id}/            # Actualizar estado (admin)
```

---

## Features principales

### 1. Autenticación

```jsx
// src/pages/LoginPage.jsx
import { useAuthStore } from '../stores/authStore';
import { useNavigate } from 'react-router-dom';

export function LoginPage() {
    const login = useAuthStore(state => state.login);
    const navigate = useNavigate();
    const [formData, setFormData] = useState({ username: '', password: '' });
    
    const handleSubmit = async (e) => {
        e.preventDefault();
        try {
            await login(formData.username, formData.password);
            navigate('/');
        } catch (error) {
            alert('Credenciales incorrectas');
        }
    };
    
    return (
        <form onSubmit={handleSubmit} className="max-w-md mx-auto p-6">
            <h1 className="text-2xl font-bold mb-4">Iniciar Sesión</h1>
            <input
                type="text"
                placeholder="Usuario"
                value={formData.username}
                onChange={(e) => setFormData({...formData, username: e.target.value})}
                className="w-full p-2 border rounded mb-4"
            />
            <input
                type="password"
                placeholder="Contraseña"
                value={formData.password}
                onChange={(e) => setFormData({...formData, password: e.target.value})}
                className="w-full p-2 border rounded mb-4"
            />
            <button type="submit" className="w-full bg-blue-500 text-white p-2 rounded">
                Entrar
            </button>
        </form>
    );
}
```

---

### 2. Catálogo de productos

```jsx
// src/pages/ProductosPage.jsx
import { useEffect, useState } from 'react';
import { productoService } from '../services/productoService';
import { ProductoCard } from '../components/ProductoCard';

export function ProductosPage() {
    const [productos, setProductos] = useState([]);
    const [loading, setLoading] = useState(true);
    const [filtros, setFiltros] = useState({
        categoria: '',
        buscar: '',
        ordenar: '-creado'
    });
    
    useEffect(() => {
        cargarProductos();
    }, [filtros]);
    
    const cargarProductos = async () => {
        setLoading(true);
        try {
            const data = await productoService.obtenerTodos(filtros);
            setProductos(data);
        } catch (error) {
            console.error('Error al cargar productos', error);
        } finally {
            setLoading(false);
        }
    };
    
    return (
        <div className="container mx-auto p-4">
            <h1 className="text-3xl font-bold mb-6">Productos</h1>
            
            {/* Filtros */}
            <div className="flex gap-4 mb-6">
                <input
                    type="text"
                    placeholder="Buscar..."
                    value={filtros.buscar}
                    onChange={(e) => setFiltros({...filtros, buscar: e.target.value})}
                    className="flex-1 p-2 border rounded"
                />
                <select
                    value={filtros.ordenar}
                    onChange={(e) => setFiltros({...filtros, ordenar: e.target.value})}
                    className="p-2 border rounded"
                >
                    <option value="-creado">Más recientes</option>
                    <option value="precio">Precio: menor a mayor</option>
                    <option value="-precio">Precio: mayor a menor</option>
                </select>
            </div>
            
            {/* Grid de productos */}
            {loading ? (
                <div>Cargando...</div>
            ) : (
                <div className="grid grid-cols-1 md:grid-cols-3 lg:grid-cols-4 gap-6">
                    {productos.map(producto => (
                        <ProductoCard key={producto.id} producto={producto} />
                    ))}
                </div>
            )}
        </div>
    );
}
```

---

### 3. Carrito de compras

```javascript
// src/stores/carritoStore.js
import { create } from 'zustand';
import { persist } from 'zustand/middleware';

export const useCarritoStore = create(
    persist(
        (set, get) => ({
            items: [],
            
            agregarItem: (producto, cantidad = 1) => {
                const items = get().items;
                const existente = items.find(item => item.producto.id === producto.id);
                
                if (existente) {
                    set({
                        items: items.map(item =>
                            item.producto.id === producto.id
                                ? { ...item, cantidad: item.cantidad + cantidad }
                                : item
                        )
                    });
                } else {
                    set({ items: [...items, { producto, cantidad }] });
                }
            },
            
            eliminarItem: (productoId) => {
                set({
                    items: get().items.filter(item => item.producto.id !== productoId)
                });
            },
            
            actualizarCantidad: (productoId, cantidad) => {
                if (cantidad <= 0) {
                    get().eliminarItem(productoId);
                    return;
                }
                
                set({
                    items: get().items.map(item =>
                        item.producto.id === productoId
                            ? { ...item, cantidad }
                            : item
                    )
                });
            },
            
            vaciar: () => set({ items: [] }),
            
            get total() {
                return get().items.reduce(
                    (sum, item) => sum + item.producto.precio * item.cantidad,
                    0
                );
            },
            
            get cantidadTotal() {
                return get().items.reduce((sum, item) => sum + item.cantidad, 0);
            }
        }),
        {
            name: 'carrito-storage'
        }
    )
);
```

---

### 4. Checkout

```jsx
// src/pages/CheckoutPage.jsx
import { useCarritoStore } from '../stores/carritoStore';
import { ordenService } from '../services/ordenService';
import { useNavigate } from 'react-router-dom';

export function CheckoutPage() {
    const items = useCarritoStore(state => state.items);
    const total = useCarritoStore(state => state.total);
    const vaciar = useCarritoStore(state => state.vaciar);
    const navigate = useNavigate();
    
    const [datosEnvio, setDatosEnvio] = useState({
        direccion: '',
        ciudad: '',
        codigo_postal: ''
    });
    
    const handleSubmit = async (e) => {
        e.preventDefault();
        
        try {
            const orden = await ordenService.crear({
                items: items.map(item => ({
                    producto_id: item.producto.id,
                    cantidad: item.cantidad,
                    precio: item.producto.precio
                })),
                ...datosEnvio,
                total
            });
            
            vaciar();
            navigate(`/ordenes/${orden.id}`);
        } catch (error) {
            alert('Error al procesar la orden');
        }
    };
    
    return (
        <div className="container mx-auto p-4 max-w-4xl">
            <h1 className="text-3xl font-bold mb-6">Checkout</h1>
            
            <div className="grid md:grid-cols-2 gap-8">
                {/* Formulario de envío */}
                <div>
                    <h2 className="text-xl font-bold mb-4">Datos de envío</h2>
                    <form onSubmit={handleSubmit}>
                        <input
                            type="text"
                            placeholder="Dirección"
                            value={datosEnvio.direccion}
                            onChange={(e) => setDatosEnvio({...datosEnvio, direccion: e.target.value})}
                            className="w-full p-2 border rounded mb-4"
                            required
                        />
                        <input
                            type="text"
                            placeholder="Ciudad"
                            value={datosEnvio.ciudad}
                            onChange={(e) => setDatosEnvio({...datosEnvio, ciudad: e.target.value})}
                            className="w-full p-2 border rounded mb-4"
                            required
                        />
                        <input
                            type="text"
                            placeholder="Código Postal"
                            value={datosEnvio.codigo_postal}
                            onChange={(e) => setDatosEnvio({...datosEnvio, codigo_postal: e.target.value})}
                            className="w-full p-2 border rounded mb-4"
                            required
                        />
                        <button
                            type="submit"
                            className="w-full bg-green-500 text-white p-3 rounded font-bold"
                        >
                            Confirmar Orden (${total.toFixed(2)})
                        </button>
                    </form>
                </div>
                
                {/* Resumen del pedido */}
                <div>
                    <h2 className="text-xl font-bold mb-4">Resumen del pedido</h2>
                    <div className="bg-gray-100 p-4 rounded">
                        {items.map(item => (
                            <div key={item.producto.id} className="flex justify-between mb-2">
                                <span>{item.producto.nombre} x{item.cantidad}</span>
                                <span>${(item.producto.precio * item.cantidad).toFixed(2)}</span>
                            </div>
                        ))}
                        <hr className="my-4" />
                        <div className="flex justify-between font-bold text-lg">
                            <span>Total:</span>
                            <span>${total.toFixed(2)}</span>
                        </div>
                    </div>
                </div>
            </div>
        </div>
    );
}
```

---

## Testing

### Backend (Pytest)

```python
# tests/test_productos.py
import pytest
from productos.models import Producto, Categoria

@pytest.mark.django_db
def test_listar_productos(api_client):
    # Crear categoría y productos
    categoria = Categoria.objects.create(nombre='Electrónica')
    Producto.objects.create(
        nombre='Laptop',
        descripcion='Laptop gaming',
        precio=1500,
        stock=10,
        categoria=categoria
    )
    
    response = api_client.get('/api/productos/')
    assert response.status_code == 200
    assert len(response.data) == 1
    assert response.data[0]['nombre'] == 'Laptop'

@pytest.mark.django_db
def test_crear_orden(authenticated_client):
    # Crear producto
    categoria = Categoria.objects.create(nombre='Electrónica')
    producto = Producto.objects.create(
        nombre='Mouse',
        precio=25,
        stock=50,
        categoria=categoria
    )
    
    # Crear orden
    response = authenticated_client.post('/api/ordenes/', {
        'items': [{'producto_id': producto.id, 'cantidad': 2, 'precio': 25}],
        'total': 50,
        'direccion_envio': 'Calle 123',
        'ciudad': 'Ciudad',
        'codigo_postal': '12345'
    }, format='json')
    
    assert response.status_code == 201
    assert response.data['total'] == '50.00'
```

---

### Frontend (Vitest + RTL)

```javascript
// src/components/ProductoCard.test.jsx
import { render, screen } from '@testing-library/react';
import { ProductoCard } from './ProductoCard';

test('renderiza información del producto', () => {
    const producto = {
        id: 1,
        nombre: 'Laptop',
        precio: 1500,
        imagen: '/laptop.jpg'
    };
    
    render(<ProductoCard producto={producto} />);
    
    expect(screen.getByText('Laptop')).toBeInTheDocument();
    expect(screen.getByText('$1500')).toBeInTheDocument();
});
```

---

### E2E (Playwright)

```javascript
// e2e/checkout.spec.js
import { test, expect } from '@playwright/test';

test('flujo completo de compra', async ({ page }) => {
    // Login
    await page.goto('/login');
    await page.fill('input[name="username"]', 'testuser');
    await page.fill('input[name="password"]', 'password123');
    await page.click('button[type="submit"]');
    
    // Agregar producto al carrito
    await page.goto('/productos');
    await page.click('text=Ver detalle >> first');
    await page.click('button:has-text("Agregar al Carrito")');
    
    // Ir al carrito
    await page.click('text=Carrito');
    await expect(page.locator('text=Total:')).toBeVisible();
    
    // Checkout
    await page.click('button:has-text("Proceder al Checkout")');
    await page.fill('input[placeholder="Dirección"]', 'Calle 123');
    await page.fill('input[placeholder="Ciudad"]', 'Ciudad Test');
    await page.fill('input[placeholder="Código Postal"]', '12345');
    await page.click('button:has-text("Confirmar Orden")');
    
    // Verificar orden creada
    await expect(page.locator('text=Orden #')).toBeVisible();
    await expect(page.locator('text=Pendiente')).toBeVisible();
});
```

---

## Próximos módulos

- **02-best-practices.md** → Checklist de calidad de código
- **03-portfolio.md** → Preparar proyecto para portfolio

¡Proyecto integrador iniciado! 🚀

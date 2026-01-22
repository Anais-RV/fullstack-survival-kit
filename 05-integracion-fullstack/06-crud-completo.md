# CRUD Completo

> **Create, Read, Update, Delete: Operaciones completas con React + Django**

---

## Arquitectura CRUD

```
┌─────────────────────────────────────┐
│   FRONTEND (React)                  │
│   ┌───────────────────────────┐     │
│   │ Formulario                │     │
│   │ ├── Crear                 │     │
│   │ └── Editar                │     │
│   ├───────────────────────────┤     │
│   │ Lista                     │     │
│   │ └── Ver + Eliminar        │     │
│   └───────────────────────────┘     │
└──────────────┬──────────────────────┘
               │
               │ HTTP Requests
               ↓
┌─────────────────────────────────────┐
│   BACKEND (Django REST)             │
│   ┌───────────────────────────┐     │
│   │ ProductoViewSet           │     │
│   │ ├── list()    GET         │     │
│   │ ├── create()  POST        │     │
│   │ ├── retrieve() GET/:id    │     │
│   │ ├── update()  PUT/:id     │     │
│   │ └── destroy() DELETE/:id  │     │
│   └───────────────────────────┘     │
└──────────────┬──────────────────────┘
               │
               ↓
         ┌─────────┐
         │PostgreSQL│
         └─────────┘
```

---

## Backend: Django ViewSet completo

### Modelo

```python
# productos/models.py
from django.db import models
from django.contrib.auth import get_user_model

User = get_user_model()

class Categoria(models.Model):
    nombre = models.CharField(max_length=100)
    descripcion = models.TextField(blank=True)
    
    def __str__(self):
        return self.nombre
    
    class Meta:
        verbose_name_plural = "Categorías"

class Producto(models.Model):
    nombre = models.CharField(max_length=200)
    descripcion = models.TextField(blank=True)
    precio = models.DecimalField(max_digits=10, decimal_places=2)
    stock = models.IntegerField(default=0)
    categoria = models.ForeignKey(Categoria, on_delete=models.CASCADE, related_name='productos')
    imagen = models.ImageField(upload_to='productos/', blank=True, null=True)
    activo = models.BooleanField(default=True)
    creado_por = models.ForeignKey(User, on_delete=models.CASCADE)
    creado_en = models.DateTimeField(auto_now_add=True)
    actualizado_en = models.DateTimeField(auto_now=True)
    
    def __str__(self):
        return self.nombre
    
    class Meta:
        ordering = ['-creado_en']
```

### Serializer

```python
# productos/serializers.py
from rest_framework import serializers
from .models import Producto, Categoria

class CategoriaSerializer(serializers.ModelSerializer):
    class Meta:
        model = Categoria
        fields = ['id', 'nombre', 'descripcion']

class ProductoSerializer(serializers.ModelSerializer):
    categoria_nombre = serializers.CharField(source='categoria.nombre', read_only=True)
    creado_por_nombre = serializers.CharField(source='creado_por.username', read_only=True)
    
    class Meta:
        model = Producto
        fields = [
            'id', 'nombre', 'descripcion', 'precio', 'stock',
            'categoria', 'categoria_nombre', 'imagen', 'activo',
            'creado_por', 'creado_por_nombre', 'creado_en', 'actualizado_en'
        ]
        read_only_fields = ['id', 'creado_por', 'creado_en', 'actualizado_en']
    
    def validate_precio(self, value):
        if value <= 0:
            raise serializers.ValidationError("El precio debe ser positivo")
        return value
    
    def validate_stock(self, value):
        if value < 0:
            raise serializers.ValidationError("El stock no puede ser negativo")
        return value
```

### ViewSet con búsqueda y filtros

```python
# productos/views.py
from rest_framework import viewsets, filters, status
from rest_framework.decorators import action
from rest_framework.response import Response
from rest_framework.permissions import IsAuthenticated, IsAuthenticatedOrReadOnly
from django_filters.rest_framework import DjangoFilterBackend
from .models import Producto, Categoria
from .serializers import ProductoSerializer, CategoriaSerializer

class ProductoViewSet(viewsets.ModelViewSet):
    queryset = Producto.objects.select_related('categoria', 'creado_por')
    serializer_class = ProductoSerializer
    permission_classes = [IsAuthenticatedOrReadOnly]
    
    # Filtros, búsqueda, ordenamiento
    filter_backends = [DjangoFilterBackend, filters.SearchFilter, filters.OrderingFilter]
    filterset_fields = ['categoria', 'activo']
    search_fields = ['nombre', 'descripcion']
    ordering_fields = ['nombre', 'precio', 'creado_en']
    
    def perform_create(self, serializer):
        serializer.save(creado_por=self.request.user)
    
    def get_permissions(self):
        if self.action in ['create', 'update', 'partial_update', 'destroy']:
            return [IsAuthenticated()]
        return [IsAuthenticatedOrReadOnly()]
    
    # Endpoint personalizado: productos destacados
    @action(detail=False, methods=['get'])
    def destacados(self, request):
        productos = self.queryset.filter(activo=True)[:5]
        serializer = self.get_serializer(productos, many=True)
        return Response(serializer.data)
    
    # Endpoint personalizado: bajo stock
    @action(detail=False, methods=['get'])
    def bajo_stock(self, request):
        productos = self.queryset.filter(stock__lt=10)
        serializer = self.get_serializer(productos, many=True)
        return Response(serializer.data)

class CategoriaViewSet(viewsets.ModelViewSet):
    queryset = Categoria.objects.all()
    serializer_class = CategoriaSerializer
    permission_classes = [IsAuthenticatedOrReadOnly]
```

### URLs

```python
# productos/urls.py
from django.urls import path, include
from rest_framework.routers import DefaultRouter
from .views import ProductoViewSet, CategoriaViewSet

router = DefaultRouter()
router.register(r'productos', ProductoViewSet)
router.register(r'categorias', CategoriaViewSet)

urlpatterns = [
    path('', include(router.urls)),
]
```

**Endpoints generados:**
```
GET    /api/productos/              # Lista
POST   /api/productos/              # Crear
GET    /api/productos/{id}/         # Detalle
PUT    /api/productos/{id}/         # Actualizar completo
PATCH  /api/productos/{id}/         # Actualizar parcial
DELETE /api/productos/{id}/         # Eliminar
GET    /api/productos/destacados/   # Custom
GET    /api/productos/bajo_stock/   # Custom

GET    /api/categorias/
POST   /api/categorias/
...
```

---

## Frontend: React con CRUD completo

### Servicio

```javascript
// src/services/productoService.js
import API from './api';

export const productoService = {
    // Lista (con filtros)
    getAll: async (params = {}) => {
        try {
            const response = await API.get('/productos/', { params });
            return { success: true, data: response.data };
        } catch (error) {
            return { success: false, error: error.response?.data };
        }
    },
    
    // Detalle
    getById: async (id) => {
        try {
            const response = await API.get(`/productos/${id}/`);
            return { success: true, data: response.data };
        } catch (error) {
            return { success: false, error: error.response?.data };
        }
    },
    
    // Crear
    create: async (data) => {
        try {
            const response = await API.post('/productos/', data);
            return { success: true, data: response.data };
        } catch (error) {
            return { success: false, error: error.response?.data };
        }
    },
    
    // Actualizar
    update: async (id, data) => {
        try {
            const response = await API.put(`/productos/${id}/`, data);
            return { success: true, data: response.data };
        } catch (error) {
            return { success: false, error: error.response?.data };
        }
    },
    
    // Actualizar parcial
    partialUpdate: async (id, data) => {
        try {
            const response = await API.patch(`/productos/${id}/`, data);
            return { success: true, data: response.data };
        } catch (error) {
            return { success: false, error: error.response?.data };
        }
    },
    
    // Eliminar
    delete: async (id) => {
        try {
            await API.delete(`/productos/${id}/`);
            return { success: true };
        } catch (error) {
            return { success: false, error: error.response?.data };
        }
    },
    
    // Destacados
    getDestacados: async () => {
        try {
            const response = await API.get('/productos/destacados/');
            return { success: true, data: response.data };
        } catch (error) {
            return { success: false, error: error.response?.data };
        }
    },
};

export const categoriaService = {
    getAll: async () => {
        try {
            const response = await API.get('/categorias/');
            return { success: true, data: response.data };
        } catch (error) {
            return { success: false, error: error.response?.data };
        }
    },
};
```

### Hook personalizado

```javascript
// src/hooks/useProductos.js
import { useState, useEffect } from 'react';
import { productoService } from '../services/productoService';

export const useProductos = (params = {}) => {
    const [productos, setProductos] = useState([]);
    const [loading, setLoading] = useState(true);
    const [error, setError] = useState(null);
    
    useEffect(() => {
        cargar();
    }, [JSON.stringify(params)]);
    
    const cargar = async () => {
        setLoading(true);
        const result = await productoService.getAll(params);
        
        if (result.success) {
            setProductos(result.data.results || result.data);
            setError(null);
        } else {
            setError(result.error);
        }
        
        setLoading(false);
    };
    
    return { productos, loading, error, refetch: cargar };
};
```

### Lista de productos

```jsx
// src/components/ProductosList.jsx
import { useState } from 'react';
import { Link } from 'react-router-dom';
import { useProductos } from '../hooks/useProductos';
import { productoService } from '../services/productoService';

export default function ProductosList() {
    const [filtros, setFiltros] = useState({
        search: '',
        categoria: '',
        activo: '',
    });
    
    const { productos, loading, error, refetch } = useProductos(filtros);
    
    const handleSearch = (e) => {
        e.preventDefault();
        refetch();
    };
    
    const eliminar = async (id) => {
        if (!confirm('¿Eliminar este producto?')) return;
        
        const result = await productoService.delete(id);
        
        if (result.success) {
            refetch();
        } else {
            alert('Error al eliminar');
        }
    };
    
    if (loading) return <div>Cargando...</div>;
    if (error) return <div>Error: {JSON.stringify(error)}</div>;
    
    return (
        <div>
            <div style={{ display: 'flex', justifyContent: 'space-between', marginBottom: '1rem' }}>
                <h1>Productos</h1>
                <Link to="/productos/crear">
                    <button>+ Crear Producto</button>
                </Link>
            </div>
            
            {/* Filtros */}
            <form onSubmit={handleSearch} style={{ marginBottom: '1rem' }}>
                <input
                    type="text"
                    placeholder="Buscar..."
                    value={filtros.search}
                    onChange={(e) => setFiltros({ ...filtros, search: e.target.value })}
                />
                <select
                    value={filtros.activo}
                    onChange={(e) => setFiltros({ ...filtros, activo: e.target.value })}
                >
                    <option value="">Todos</option>
                    <option value="true">Activos</option>
                    <option value="false">Inactivos</option>
                </select>
                <button type="submit">Buscar</button>
            </form>
            
            {/* Tabla */}
            <table style={{ width: '100%', borderCollapse: 'collapse' }}>
                <thead>
                    <tr>
                        <th>ID</th>
                        <th>Imagen</th>
                        <th>Nombre</th>
                        <th>Precio</th>
                        <th>Stock</th>
                        <th>Categoría</th>
                        <th>Activo</th>
                        <th>Acciones</th>
                    </tr>
                </thead>
                <tbody>
                    {productos.map(producto => (
                        <tr key={producto.id}>
                            <td>{producto.id}</td>
                            <td>
                                {producto.imagen && (
                                    <img
                                        src={producto.imagen}
                                        alt={producto.nombre}
                                        style={{ width: 50, height: 50, objectFit: 'cover' }}
                                    />
                                )}
                            </td>
                            <td>{producto.nombre}</td>
                            <td>${producto.precio}</td>
                            <td>{producto.stock}</td>
                            <td>{producto.categoria_nombre}</td>
                            <td>{producto.activo ? '✅' : '❌'}</td>
                            <td>
                                <Link to={`/productos/${producto.id}`}>
                                    <button>Ver</button>
                                </Link>
                                <Link to={`/productos/${producto.id}/editar`}>
                                    <button>Editar</button>
                                </Link>
                                <button onClick={() => eliminar(producto.id)}>
                                    Eliminar
                                </button>
                            </td>
                        </tr>
                    ))}
                </tbody>
            </table>
            
            {productos.length === 0 && <p>No hay productos</p>}
        </div>
    );
}
```

### Formulario de crear/editar

```jsx
// src/components/ProductoForm.jsx
import { useState, useEffect } from 'react';
import { useNavigate, useParams } from 'react-router-dom';
import { productoService } from '../services/productoService';
import { categoriaService } from '../services/productoService';

export default function ProductoForm() {
    const { id } = useParams();
    const navigate = useNavigate();
    const isEdit = !!id;
    
    const [formData, setFormData] = useState({
        nombre: '',
        descripcion: '',
        precio: '',
        stock: '',
        categoria: '',
        activo: true,
    });
    const [categorias, setCategorias] = useState([]);
    const [loading, setLoading] = useState(false);
    const [error, setError] = useState(null);
    
    useEffect(() => {
        cargarCategorias();
        if (isEdit) {
            cargarProducto();
        }
    }, [id]);
    
    const cargarCategorias = async () => {
        const result = await categoriaService.getAll();
        if (result.success) {
            setCategorias(result.data);
        }
    };
    
    const cargarProducto = async () => {
        const result = await productoService.getById(id);
        if (result.success) {
            setFormData({
                nombre: result.data.nombre,
                descripcion: result.data.descripcion,
                precio: result.data.precio,
                stock: result.data.stock,
                categoria: result.data.categoria,
                activo: result.data.activo,
            });
        }
    };
    
    const handleChange = (e) => {
        const { name, value, type, checked } = e.target;
        setFormData({
            ...formData,
            [name]: type === 'checkbox' ? checked : value
        });
    };
    
    const handleSubmit = async (e) => {
        e.preventDefault();
        setLoading(true);
        setError(null);
        
        const result = isEdit
            ? await productoService.update(id, formData)
            : await productoService.create(formData);
        
        if (result.success) {
            navigate('/productos');
        } else {
            setError(result.error);
        }
        
        setLoading(false);
    };
    
    return (
        <div>
            <h2>{isEdit ? 'Editar' : 'Crear'} Producto</h2>
            
            {error && (
                <div style={{ color: 'red', marginBottom: '1rem' }}>
                    {JSON.stringify(error)}
                </div>
            )}
            
            <form onSubmit={handleSubmit}>
                <div>
                    <label>Nombre:</label>
                    <input
                        type="text"
                        name="nombre"
                        value={formData.nombre}
                        onChange={handleChange}
                        required
                    />
                </div>
                
                <div>
                    <label>Descripción:</label>
                    <textarea
                        name="descripcion"
                        value={formData.descripcion}
                        onChange={handleChange}
                    />
                </div>
                
                <div>
                    <label>Precio:</label>
                    <input
                        type="number"
                        step="0.01"
                        name="precio"
                        value={formData.precio}
                        onChange={handleChange}
                        required
                    />
                </div>
                
                <div>
                    <label>Stock:</label>
                    <input
                        type="number"
                        name="stock"
                        value={formData.stock}
                        onChange={handleChange}
                        required
                    />
                </div>
                
                <div>
                    <label>Categoría:</label>
                    <select
                        name="categoria"
                        value={formData.categoria}
                        onChange={handleChange}
                        required
                    >
                        <option value="">Seleccionar...</option>
                        {categorias.map(cat => (
                            <option key={cat.id} value={cat.id}>
                                {cat.nombre}
                            </option>
                        ))}
                    </select>
                </div>
                
                <div>
                    <label>
                        <input
                            type="checkbox"
                            name="activo"
                            checked={formData.activo}
                            onChange={handleChange}
                        />
                        Activo
                    </label>
                </div>
                
                <button type="submit" disabled={loading}>
                    {loading ? 'Guardando...' : isEdit ? 'Actualizar' : 'Crear'}
                </button>
                <button type="button" onClick={() => navigate('/productos')}>
                    Cancelar
                </button>
            </form>
        </div>
    );
}
```

### Detalle de producto

```jsx
// src/components/ProductoDetail.jsx
import { useState, useEffect } from 'react';
import { useParams, useNavigate, Link } from 'react-router-dom';
import { productoService } from '../services/productoService';

export default function ProductoDetail() {
    const { id } = useParams();
    const navigate = useNavigate();
    const [producto, setProducto] = useState(null);
    const [loading, setLoading] = useState(true);
    
    useEffect(() => {
        cargar();
    }, [id]);
    
    const cargar = async () => {
        const result = await productoService.getById(id);
        if (result.success) {
            setProducto(result.data);
        }
        setLoading(false);
    };
    
    const eliminar = async () => {
        if (!confirm('¿Eliminar este producto?')) return;
        
        const result = await productoService.delete(id);
        if (result.success) {
            navigate('/productos');
        }
    };
    
    if (loading) return <div>Cargando...</div>;
    if (!producto) return <div>Producto no encontrado</div>;
    
    return (
        <div>
            <h1>{producto.nombre}</h1>
            
            {producto.imagen && (
                <img
                    src={producto.imagen}
                    alt={producto.nombre}
                    style={{ maxWidth: 400 }}
                />
            )}
            
            <div>
                <p><strong>Precio:</strong> ${producto.precio}</p>
                <p><strong>Stock:</strong> {producto.stock}</p>
                <p><strong>Categoría:</strong> {producto.categoria_nombre}</p>
                <p><strong>Descripción:</strong> {producto.descripcion}</p>
                <p><strong>Activo:</strong> {producto.activo ? 'Sí' : 'No'}</p>
                <p><strong>Creado por:</strong> {producto.creado_por_nombre}</p>
                <p><strong>Creado:</strong> {new Date(producto.creado_en).toLocaleString()}</p>
            </div>
            
            <div>
                <Link to={`/productos/${producto.id}/editar`}>
                    <button>Editar</button>
                </Link>
                <button onClick={eliminar}>Eliminar</button>
                <button onClick={() => navigate('/productos')}>Volver</button>
            </div>
        </div>
    );
}
```

### Rutas

```jsx
// src/App.jsx
import { BrowserRouter, Routes, Route } from 'react-router-dom';
import ProductosList from './components/ProductosList';
import ProductoForm from './components/ProductoForm';
import ProductoDetail from './components/ProductoDetail';

function App() {
    return (
        <BrowserRouter>
            <Routes>
                <Route path="/productos" element={<ProductosList />} />
                <Route path="/productos/crear" element={<ProductoForm />} />
                <Route path="/productos/:id" element={<ProductoDetail />} />
                <Route path="/productos/:id/editar" element={<ProductoForm />} />
            </Routes>
        </BrowserRouter>
    );
}

export default App;
```

---

## Validación en ambos lados

### Backend (Django)

```python
# productos/serializers.py
class ProductoSerializer(serializers.ModelSerializer):
    def validate_precio(self, value):
        if value <= 0:
            raise serializers.ValidationError("El precio debe ser positivo")
        if value > 1000000:
            raise serializers.ValidationError("Precio demasiado alto")
        return value
    
    def validate_stock(self, value):
        if value < 0:
            raise serializers.ValidationError("El stock no puede ser negativo")
        return value
    
    def validate_nombre(self, value):
        if len(value) < 3:
            raise serializers.ValidationError("El nombre debe tener al menos 3 caracteres")
        return value
    
    def validate(self, data):
        # Validación entre campos
        if data.get('stock', 0) > 0 and not data.get('activo', True):
            raise serializers.ValidationError("No puedes desactivar un producto con stock")
        return data
```

### Frontend (React)

```jsx
const validate = () => {
    const errors = {};
    
    if (!formData.nombre || formData.nombre.length < 3) {
        errors.nombre = 'El nombre debe tener al menos 3 caracteres';
    }
    
    if (!formData.precio || formData.precio <= 0) {
        errors.precio = 'El precio debe ser positivo';
    }
    
    if (formData.stock < 0) {
        errors.stock = 'El stock no puede ser negativo';
    }
    
    return errors;
};

const handleSubmit = async (e) => {
    e.preventDefault();
    
    const errors = validate();
    if (Object.keys(errors).length > 0) {
        setError(errors);
        return;
    }
    
    // Enviar al servidor...
};
```

---

## Resumen

**Backend (Django):**
- ViewSet con list, create, retrieve, update, destroy
- Serializers con validaciones
- Filtros, búsqueda, ordenamiento

**Frontend (React):**
- Lista con filtros
- Formulario crear/editar (reutilizable)
- Vista de detalle
- Manejo de estados (loading, error)

**Flujo completo:**
1. Usuario abre lista → GET /api/productos/
2. Click "Crear" → Formulario vacío → POST /api/productos/
3. Click "Editar" → Cargar datos → PUT /api/productos/{id}/
4. Click "Eliminar" → Confirmar → DELETE /api/productos/{id}/

**Próximo módulo:** Upload de archivos e imágenes

---

## Recursos

- **[Django Viewsets](https://www.django-rest-framework.org/api-guide/viewsets/)** - Documentación oficial
- **[React Router](https://reactrouter.com/)** - Navegación
- **[React Hook Form](https://react-hook-form.com/)** - Formularios avanzados

¡CRUD completo implementado! 📝

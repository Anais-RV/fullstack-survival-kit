# Axios y API Client

> **Organizar todas las llamadas HTTP en servicios reutilizables**

---

## Por qué usar servicios API

**Problema:**
```jsx
// ❌ Repetido en cada componente
fetch('http://localhost:8000/api/productos/')
    .then(res => res.json())
    .then(data => setProductos(data));
```

**Solución:**
```javascript
// ✅ Servicio reutilizable
const productos = await productoService.getAll();
```

**Ventajas:**
- ✅ Centralizar configuración (baseURL, headers)
- ✅ Reutilizar código
- ✅ Manejo de errores consistente
- ✅ Fácil de mantener
- ✅ Fácil de testear

---

## Instalar Axios

```bash
npm install axios
```

---

## Estructura de servicios

```
src/
├── services/
│   ├── api.js               # Instancia base de Axios
│   ├── authService.js       # Login, registro, perfil
│   ├── productoService.js   # CRUD productos
│   ├── pedidoService.js     # CRUD pedidos
│   └── uploadService.js     # Subir archivos
```

---

## api.js: Configuración base

```javascript
// src/services/api.js
import axios from 'axios';

const API = axios.create({
    baseURL: import.meta.env.VITE_API_URL || 'http://localhost:8000/api',
    headers: {
        'Content-Type': 'application/json',
    },
    timeout: 10000,  // 10 segundos
});

// Request interceptor: agregar token automáticamente
API.interceptors.request.use(
    (config) => {
        const token = localStorage.getItem('access_token');
        if (token) {
            config.headers.Authorization = `Bearer ${token}`;
        }
        return config;
    },
    (error) => {
        return Promise.reject(error);
    }
);

// Response interceptor: manejar errores y refresh token
API.interceptors.response.use(
    (response) => {
        return response;
    },
    async (error) => {
        const originalRequest = error.config;
        
        // Token expirado → refresh automático
        if (error.response?.status === 401 && !originalRequest._retry) {
            originalRequest._retry = true;
            
            try {
                const refresh = localStorage.getItem('refresh_token');
                const { data } = await axios.post(
                    `${API.defaults.baseURL}/usuarios/token/refresh/`,
                    { refresh }
                );
                
                localStorage.setItem('access_token', data.access);
                if (data.refresh) {
                    localStorage.setItem('refresh_token', data.refresh);
                }
                
                originalRequest.headers.Authorization = `Bearer ${data.access}`;
                return API(originalRequest);
            } catch (err) {
                localStorage.removeItem('access_token');
                localStorage.removeItem('refresh_token');
                window.location.href = '/login';
                return Promise.reject(err);
            }
        }
        
        return Promise.reject(error);
    }
);

export default API;
```

---

## authService.js: Autenticación

```javascript
// src/services/authService.js
import API from './api';

export const authService = {
    // Registro
    register: async (data) => {
        try {
            const response = await API.post('/usuarios/register/', data);
            return { success: true, data: response.data };
        } catch (error) {
            return {
                success: false,
                error: error.response?.data || 'Error al registrar'
            };
        }
    },
    
    // Login
    login: async (username, password) => {
        try {
            const response = await API.post('/usuarios/login/', { username, password });
            return { success: true, data: response.data };
        } catch (error) {
            return {
                success: false,
                error: error.response?.data?.detail || 'Credenciales incorrectas'
            };
        }
    },
    
    // Perfil
    getProfile: async () => {
        const response = await API.get('/usuarios/profile/');
        return response.data;
    },
    
    // Actualizar perfil
    updateProfile: async (data) => {
        try {
            const response = await API.patch('/usuarios/profile/', data);
            return { success: true, data: response.data };
        } catch (error) {
            return {
                success: false,
                error: error.response?.data || 'Error al actualizar'
            };
        }
    },
    
    // Refresh token
    refreshToken: async (refresh) => {
        const response = await API.post('/usuarios/token/refresh/', { refresh });
        return response.data;
    },
};
```

---

## productoService.js: CRUD completo

```javascript
// src/services/productoService.js
import API from './api';

export const productoService = {
    // Listar todos (con paginación y filtros)
    getAll: async (params = {}) => {
        try {
            const response = await API.get('/productos/', { params });
            return { success: true, data: response.data };
        } catch (error) {
            return {
                success: false,
                error: error.response?.data || 'Error al cargar productos'
            };
        }
    },
    
    // Obtener uno por ID
    getById: async (id) => {
        try {
            const response = await API.get(`/productos/${id}/`);
            return { success: true, data: response.data };
        } catch (error) {
            return {
                success: false,
                error: error.response?.data || 'Producto no encontrado'
            };
        }
    },
    
    // Crear
    create: async (data) => {
        try {
            const response = await API.post('/productos/', data);
            return { success: true, data: response.data };
        } catch (error) {
            return {
                success: false,
                error: error.response?.data || 'Error al crear producto'
            };
        }
    },
    
    // Actualizar
    update: async (id, data) => {
        try {
            const response = await API.put(`/productos/${id}/`, data);
            return { success: true, data: response.data };
        } catch (error) {
            return {
                success: false,
                error: error.response?.data || 'Error al actualizar producto'
            };
        }
    },
    
    // Actualizar parcial
    partialUpdate: async (id, data) => {
        try {
            const response = await API.patch(`/productos/${id}/`, data);
            return { success: true, data: response.data };
        } catch (error) {
            return {
                success: false,
                error: error.response?.data || 'Error al actualizar producto'
            };
        }
    },
    
    // Eliminar
    delete: async (id) => {
        try {
            await API.delete(`/productos/${id}/`);
            return { success: true };
        } catch (error) {
            return {
                success: false,
                error: error.response?.data || 'Error al eliminar producto'
            };
        }
    },
    
    // Buscar
    search: async (query) => {
        try {
            const response = await API.get('/productos/', {
                params: { search: query }
            });
            return { success: true, data: response.data };
        } catch (error) {
            return {
                success: false,
                error: error.response?.data || 'Error al buscar'
            };
        }
    },
};
```

---

## Usar servicios en componentes

### Lista de productos

```jsx
// src/components/ProductosList.jsx
import { useState, useEffect } from 'react';
import { productoService } from '../services/productoService';

export default function ProductosList() {
    const [productos, setProductos] = useState([]);
    const [loading, setLoading] = useState(true);
    const [error, setError] = useState(null);
    
    useEffect(() => {
        cargarProductos();
    }, []);
    
    const cargarProductos = async () => {
        setLoading(true);
        const result = await productoService.getAll();
        
        if (result.success) {
            setProductos(result.data.results || result.data);
            setError(null);
        } else {
            setError(result.error);
        }
        
        setLoading(false);
    };
    
    const eliminar = async (id) => {
        if (!confirm('¿Eliminar este producto?')) return;
        
        const result = await productoService.delete(id);
        
        if (result.success) {
            setProductos(productos.filter(p => p.id !== id));
        } else {
            alert(`Error: ${JSON.stringify(result.error)}`);
        }
    };
    
    if (loading) return <div>Cargando...</div>;
    if (error) return <div>Error: {JSON.stringify(error)}</div>;
    
    return (
        <div>
            <h1>Productos</h1>
            <button onClick={cargarProductos}>Recargar</button>
            
            <table>
                <thead>
                    <tr>
                        <th>ID</th>
                        <th>Nombre</th>
                        <th>Precio</th>
                        <th>Stock</th>
                        <th>Acciones</th>
                    </tr>
                </thead>
                <tbody>
                    {productos.map(producto => (
                        <tr key={producto.id}>
                            <td>{producto.id}</td>
                            <td>{producto.nombre}</td>
                            <td>${producto.precio}</td>
                            <td>{producto.stock}</td>
                            <td>
                                <button onClick={() => eliminar(producto.id)}>
                                    Eliminar
                                </button>
                            </td>
                        </tr>
                    ))}
                </tbody>
            </table>
        </div>
    );
}
```

### Crear producto

```jsx
// src/components/ProductoForm.jsx
import { useState } from 'react';
import { productoService } from '../services/productoService';
import { useNavigate } from 'react-router-dom';

export default function ProductoForm() {
    const [formData, setFormData] = useState({
        nombre: '',
        descripcion: '',
        precio: '',
        stock: '',
    });
    const [loading, setLoading] = useState(false);
    const [error, setError] = useState(null);
    const navigate = useNavigate();
    
    const handleChange = (e) => {
        setFormData({
            ...formData,
            [e.target.name]: e.target.value
        });
    };
    
    const handleSubmit = async (e) => {
        e.preventDefault();
        setLoading(true);
        setError(null);
        
        const result = await productoService.create(formData);
        
        if (result.success) {
            alert('Producto creado exitosamente');
            navigate('/productos');
        } else {
            setError(result.error);
        }
        
        setLoading(false);
    };
    
    return (
        <div>
            <h2>Crear Producto</h2>
            
            {error && (
                <div style={{ color: 'red' }}>
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
                
                <button type="submit" disabled={loading}>
                    {loading ? 'Guardando...' : 'Crear Producto'}
                </button>
            </form>
        </div>
    );
}
```

---

## Hooks personalizados

### useFetch: Hook genérico

```javascript
// src/hooks/useFetch.js
import { useState, useEffect } from 'react';

export const useFetch = (fetchFunction, dependencies = []) => {
    const [data, setData] = useState(null);
    const [loading, setLoading] = useState(true);
    const [error, setError] = useState(null);
    
    useEffect(() => {
        const fetchData = async () => {
            setLoading(true);
            const result = await fetchFunction();
            
            if (result.success) {
                setData(result.data);
                setError(null);
            } else {
                setError(result.error);
            }
            
            setLoading(false);
        };
        
        fetchData();
    }, dependencies);
    
    return { data, loading, error };
};
```

**Uso:**
```jsx
import { useFetch } from '../hooks/useFetch';
import { productoService } from '../services/productoService';

function ProductosList() {
    const { data, loading, error } = useFetch(() => productoService.getAll());
    
    if (loading) return <div>Cargando...</div>;
    if (error) return <div>Error: {error}</div>;
    
    return (
        <div>
            {data.results.map(producto => (
                <div key={producto.id}>{producto.nombre}</div>
            ))}
        </div>
    );
}
```

### useProductos: Hook específico

```javascript
// src/hooks/useProductos.js
import { useState, useEffect } from 'react';
import { productoService } from '../services/productoService';

export const useProductos = () => {
    const [productos, setProductos] = useState([]);
    const [loading, setLoading] = useState(true);
    const [error, setError] = useState(null);
    
    useEffect(() => {
        cargar();
    }, []);
    
    const cargar = async () => {
        setLoading(true);
        const result = await productoService.getAll();
        
        if (result.success) {
            setProductos(result.data.results || result.data);
            setError(null);
        } else {
            setError(result.error);
        }
        
        setLoading(false);
    };
    
    const crear = async (data) => {
        const result = await productoService.create(data);
        if (result.success) {
            setProductos([...productos, result.data]);
        }
        return result;
    };
    
    const actualizar = async (id, data) => {
        const result = await productoService.update(id, data);
        if (result.success) {
            setProductos(productos.map(p => p.id === id ? result.data : p));
        }
        return result;
    };
    
    const eliminar = async (id) => {
        const result = await productoService.delete(id);
        if (result.success) {
            setProductos(productos.filter(p => p.id !== id));
        }
        return result;
    };
    
    return {
        productos,
        loading,
        error,
        cargar,
        crear,
        actualizar,
        eliminar,
    };
};
```

**Uso simplificado:**
```jsx
import { useProductos } from '../hooks/useProductos';

function ProductosList() {
    const { productos, loading, error, eliminar } = useProductos();
    
    if (loading) return <div>Cargando...</div>;
    if (error) return <div>Error</div>;
    
    return (
        <div>
            {productos.map(producto => (
                <div key={producto.id}>
                    {producto.nombre}
                    <button onClick={() => eliminar(producto.id)}>Eliminar</button>
                </div>
            ))}
        </div>
    );
}
```

---

## Manejo avanzado de errores

### Tipos de errores

```javascript
// src/services/api.js

API.interceptors.response.use(
    (response) => response,
    (error) => {
        if (error.response) {
            // Error del servidor (4xx, 5xx)
            console.error('Error del servidor:', error.response.status);
            console.error('Datos:', error.response.data);
        } else if (error.request) {
            // Request enviado pero sin respuesta (timeout, red caída)
            console.error('Sin respuesta del servidor');
        } else {
            // Error al configurar el request
            console.error('Error de configuración:', error.message);
        }
        
        return Promise.reject(error);
    }
);
```

### Errores de validación

```javascript
// src/utils/parseErrors.js
export const parseErrors = (errorData) => {
    if (typeof errorData === 'string') {
        return errorData;
    }
    
    if (Array.isArray(errorData)) {
        return errorData.join(', ');
    }
    
    if (typeof errorData === 'object') {
        return Object.entries(errorData)
            .map(([field, errors]) => {
                const errorList = Array.isArray(errors) ? errors : [errors];
                return `${field}: ${errorList.join(', ')}`;
            })
            .join('\n');
    }
    
    return 'Error desconocido';
};
```

**Uso:**
```jsx
import { parseErrors } from '../utils/parseErrors';

const result = await productoService.create(formData);

if (!result.success) {
    const errorMsg = parseErrors(result.error);
    alert(errorMsg);
}
```

---

## Retry automático

```javascript
// src/services/api.js
import axiosRetry from 'axios-retry';

const API = axios.create({
    baseURL: import.meta.env.VITE_API_URL,
});

// Reintentar en errores de red
axiosRetry(API, {
    retries: 3,
    retryDelay: axiosRetry.exponentialDelay,
    retryCondition: (error) => {
        return axiosRetry.isNetworkOrIdempotentRequestError(error) ||
               error.response?.status === 429;  // Too Many Requests
    },
});
```

---

## Cancelar requests

```javascript
// src/services/productoService.js
import API from './api';

export const productoService = {
    search: (query, signal) => {
        return API.get('/productos/', {
            params: { search: query },
            signal,  // AbortSignal
        });
    },
};
```

**Uso con AbortController:**
```jsx
import { useState, useEffect } from 'react';
import { productoService } from '../services/productoService';

function BuscarProductos() {
    const [query, setQuery] = useState('');
    const [resultados, setResultados] = useState([]);
    
    useEffect(() => {
        const controller = new AbortController();
        
        const buscar = async () => {
            if (query.length < 3) return;
            
            try {
                const response = await productoService.search(query, controller.signal);
                setResultados(response.data);
            } catch (error) {
                if (error.name !== 'CanceledError') {
                    console.error(error);
                }
            }
        };
        
        buscar();
        
        // Cancelar al desmontar o cambiar query
        return () => controller.abort();
    }, [query]);
    
    return (
        <div>
            <input
                type="text"
                value={query}
                onChange={(e) => setQuery(e.target.value)}
                placeholder="Buscar..."
            />
            {resultados.map(p => (
                <div key={p.id}>{p.nombre}</div>
            ))}
        </div>
    );
}
```

---

## Progreso de subida

```javascript
// src/services/uploadService.js
import API from './api';

export const uploadService = {
    uploadImage: (file, onProgress) => {
        const formData = new FormData();
        formData.append('file', file);
        
        return API.post('/upload/', formData, {
            headers: {
                'Content-Type': 'multipart/form-data',
            },
            onUploadProgress: (progressEvent) => {
                const percentCompleted = Math.round(
                    (progressEvent.loaded * 100) / progressEvent.total
                );
                onProgress(percentCompleted);
            },
        });
    },
};
```

**Uso:**
```jsx
const [progress, setProgress] = useState(0);

const handleUpload = async (file) => {
    try {
        const response = await uploadService.uploadImage(file, setProgress);
        console.log('Subido:', response.data);
    } catch (error) {
        console.error('Error al subir:', error);
    }
};

return (
    <div>
        <input type="file" onChange={(e) => handleUpload(e.target.files[0])} />
        {progress > 0 && <div>Progreso: {progress}%</div>}
    </div>
);
```

---

## Resumen

**Estructura organizada:**
- `api.js` - Configuración base con interceptors
- `*Service.js` - Servicios por entidad (producto, usuario, pedido)
- Hooks personalizados para reutilizar lógica

**Ventajas:**
- Código DRY (Don't Repeat Yourself)
- Manejo consistente de errores
- Fácil de mantener y testear
- Auto-refresh de tokens
- Retry automático

**Próximo módulo:** Estado global con Context API y Zustand

---

## Recursos

- **[Axios Docs](https://axios-http.com/)** - Documentación oficial
- **[Axios Retry](https://github.com/softonic/axios-retry)** - Plugin de reintentos
- **[AbortController](https://developer.mozilla.org/en-US/docs/Web/API/AbortController)** - Cancelar requests

¡Servicios API estructurados! 📡

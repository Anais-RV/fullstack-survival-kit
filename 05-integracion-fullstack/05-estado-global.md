# Estado Global

> **Gestionar estado compartido con Context API y Zustand**

---

## ¿Por qué estado global?

**Problema: Prop Drilling**

```jsx
// ❌ Pasar props por 5 niveles
<App user={user}>
  <Layout user={user}>
    <Dashboard user={user}>
      <Sidebar user={user}>
        <UserProfile user={user} />
      </Sidebar>
    </Dashboard>
  </Layout>
</App>
```

**Solución: Estado Global**

```jsx
// ✅ Acceso directo desde cualquier componente
function UserProfile() {
    const { user } = useAuth();
    return <div>{user.username}</div>;
}
```

---

## Cuándo usar estado global

**✅ Usar para:**
- Autenticación (usuario actual, token)
- Tema (dark/light mode)
- Idioma
- Carrito de compras
- Notificaciones

**❌ NO usar para:**
- Estado de formularios
- Estado de UI local (modales, tabs)
- Datos temporales

---

## Opción 1: Context API (incluida en React)

### Paso 1: Crear Context

```jsx
// src/context/AuthContext.jsx
import { createContext, useContext, useState, useEffect } from 'react';
import { authService } from '../services/authService';

// Crear contexto
const AuthContext = createContext();

// Hook personalizado
export const useAuth = () => {
    const context = useContext(AuthContext);
    if (!context) {
        throw new Error('useAuth debe usarse dentro de AuthProvider');
    }
    return context;
};

// Provider
export const AuthProvider = ({ children }) => {
    const [user, setUser] = useState(null);
    const [loading, setLoading] = useState(true);
    
    // Verificar si hay sesión al cargar
    useEffect(() => {
        checkAuth();
    }, []);
    
    const checkAuth = async () => {
        const token = localStorage.getItem('access_token');
        if (token) {
            try {
                const userData = await authService.getProfile();
                setUser(userData);
            } catch (error) {
                localStorage.removeItem('access_token');
                localStorage.removeItem('refresh_token');
            }
        }
        setLoading(false);
    };
    
    const login = async (username, password) => {
        const result = await authService.login(username, password);
        
        if (result.success) {
            localStorage.setItem('access_token', result.data.access);
            localStorage.setItem('refresh_token', result.data.refresh);
            const userData = await authService.getProfile();
            setUser(userData);
        }
        
        return result;
    };
    
    const register = async (formData) => {
        const result = await authService.register(formData);
        
        if (result.success) {
            localStorage.setItem('access_token', result.data.tokens.access);
            localStorage.setItem('refresh_token', result.data.tokens.refresh);
            setUser(result.data.user);
        }
        
        return result;
    };
    
    const logout = () => {
        localStorage.removeItem('access_token');
        localStorage.removeItem('refresh_token');
        setUser(null);
    };
    
    const value = {
        user,
        loading,
        login,
        register,
        logout,
        isAuthenticated: !!user,
    };
    
    return (
        <AuthContext.Provider value={value}>
            {children}
        </AuthContext.Provider>
    );
};
```

### Paso 2: Envolver App con Provider

```jsx
// src/main.jsx
import React from 'react';
import ReactDOM from 'react-dom/client';
import App from './App';
import { AuthProvider } from './context/AuthContext';

ReactDOM.createRoot(document.getElementById('root')).render(
    <React.StrictMode>
        <AuthProvider>
            <App />
        </AuthProvider>
    </React.StrictMode>
);
```

### Paso 3: Usar en cualquier componente

```jsx
// src/components/Navbar.jsx
import { useAuth } from '../context/AuthContext';

export default function Navbar() {
    const { user, logout, isAuthenticated } = useAuth();
    
    return (
        <nav>
            <h1>Mi App</h1>
            {isAuthenticated ? (
                <>
                    <span>Hola, {user.username}</span>
                    <button onClick={logout}>Cerrar Sesión</button>
                </>
            ) : (
                <a href="/login">Iniciar Sesión</a>
            )}
        </nav>
    );
}
```

---

## Context para Carrito de Compras

```jsx
// src/context/CartContext.jsx
import { createContext, useContext, useState, useEffect } from 'react';

const CartContext = createContext();

export const useCart = () => {
    const context = useContext(CartContext);
    if (!context) {
        throw new Error('useCart debe usarse dentro de CartProvider');
    }
    return context;
};

export const CartProvider = ({ children }) => {
    const [items, setItems] = useState([]);
    
    // Cargar desde localStorage
    useEffect(() => {
        const saved = localStorage.getItem('cart');
        if (saved) {
            setItems(JSON.parse(saved));
        }
    }, []);
    
    // Guardar en localStorage
    useEffect(() => {
        localStorage.setItem('cart', JSON.stringify(items));
    }, [items]);
    
    const addItem = (producto, cantidad = 1) => {
        setItems(prevItems => {
            const existente = prevItems.find(item => item.producto.id === producto.id);
            
            if (existente) {
                // Incrementar cantidad
                return prevItems.map(item =>
                    item.producto.id === producto.id
                        ? { ...item, cantidad: item.cantidad + cantidad }
                        : item
                );
            } else {
                // Agregar nuevo
                return [...prevItems, { producto, cantidad }];
            }
        });
    };
    
    const removeItem = (productoId) => {
        setItems(prevItems => prevItems.filter(item => item.producto.id !== productoId));
    };
    
    const updateQuantity = (productoId, cantidad) => {
        if (cantidad <= 0) {
            removeItem(productoId);
            return;
        }
        
        setItems(prevItems =>
            prevItems.map(item =>
                item.producto.id === productoId
                    ? { ...item, cantidad }
                    : item
            )
        );
    };
    
    const clearCart = () => {
        setItems([]);
    };
    
    const total = items.reduce(
        (sum, item) => sum + parseFloat(item.producto.precio) * item.cantidad,
        0
    );
    
    const itemCount = items.reduce((sum, item) => sum + item.cantidad, 0);
    
    return (
        <CartContext.Provider value={{
            items,
            addItem,
            removeItem,
            updateQuantity,
            clearCart,
            total,
            itemCount,
        }}>
            {children}
        </CartContext.Provider>
    );
};
```

**Uso:**
```jsx
import { useCart } from '../context/CartContext';

function ProductoCard({ producto }) {
    const { addItem } = useCart();
    
    return (
        <div>
            <h3>{producto.nombre}</h3>
            <p>${producto.precio}</p>
            <button onClick={() => addItem(producto)}>
                Agregar al carrito
            </button>
        </div>
    );
}

function CartIcon() {
    const { itemCount } = useCart();
    
    return (
        <div>
            🛒 ({itemCount})
        </div>
    );
}
```

---

## Opción 2: Zustand (más simple y potente)

### Instalar

```bash
npm install zustand
```

### Crear store

```javascript
// src/stores/authStore.js
import { create } from 'zustand';
import { persist } from 'zustand/middleware';
import { authService } from '../services/authService';

export const useAuthStore = create(
    persist(
        (set, get) => ({
            user: null,
            token: null,
            
            setUser: (user) => set({ user }),
            setToken: (token) => set({ token }),
            
            login: async (username, password) => {
                const result = await authService.login(username, password);
                
                if (result.success) {
                    set({
                        token: result.data.access,
                        user: await authService.getProfile(),
                    });
                    localStorage.setItem('access_token', result.data.access);
                    localStorage.setItem('refresh_token', result.data.refresh);
                }
                
                return result;
            },
            
            register: async (formData) => {
                const result = await authService.register(formData);
                
                if (result.success) {
                    set({
                        token: result.data.tokens.access,
                        user: result.data.user,
                    });
                    localStorage.setItem('access_token', result.data.tokens.access);
                    localStorage.setItem('refresh_token', result.data.tokens.refresh);
                }
                
                return result;
            },
            
            logout: () => {
                set({ user: null, token: null });
                localStorage.removeItem('access_token');
                localStorage.removeItem('refresh_token');
            },
            
            isAuthenticated: () => !!get().user,
        }),
        {
            name: 'auth-storage',
            partialize: (state) => ({ user: state.user }),  // Solo persistir user
        }
    )
);
```

### Usar en componentes

```jsx
// src/components/Navbar.jsx
import { useAuthStore } from '../stores/authStore';

export default function Navbar() {
    const user = useAuthStore(state => state.user);
    const logout = useAuthStore(state => state.logout);
    const isAuthenticated = useAuthStore(state => state.isAuthenticated());
    
    return (
        <nav>
            <h1>Mi App</h1>
            {isAuthenticated ? (
                <>
                    <span>Hola, {user.username}</span>
                    <button onClick={logout}>Cerrar Sesión</button>
                </>
            ) : (
                <a href="/login">Iniciar Sesión</a>
            )}
        </nav>
    );
}
```

**Ventajas de Zustand:**
- ✅ Sin providers
- ✅ Menos boilerplate
- ✅ Mejor rendimiento (re-renders selectivos)
- ✅ DevTools incluidas

---

## Store de Carrito con Zustand

```javascript
// src/stores/cartStore.js
import { create } from 'zustand';
import { persist } from 'zustand/middleware';

export const useCartStore = create(
    persist(
        (set, get) => ({
            items: [],
            
            addItem: (producto, cantidad = 1) => {
                set((state) => {
                    const existente = state.items.find(item => item.producto.id === producto.id);
                    
                    if (existente) {
                        return {
                            items: state.items.map(item =>
                                item.producto.id === producto.id
                                    ? { ...item, cantidad: item.cantidad + cantidad }
                                    : item
                            ),
                        };
                    }
                    
                    return {
                        items: [...state.items, { producto, cantidad }],
                    };
                });
            },
            
            removeItem: (productoId) => {
                set((state) => ({
                    items: state.items.filter(item => item.producto.id !== productoId),
                }));
            },
            
            updateQuantity: (productoId, cantidad) => {
                if (cantidad <= 0) {
                    get().removeItem(productoId);
                    return;
                }
                
                set((state) => ({
                    items: state.items.map(item =>
                        item.producto.id === productoId
                            ? { ...item, cantidad }
                            : item
                    ),
                }));
            },
            
            clearCart: () => set({ items: [] }),
            
            getTotal: () => {
                return get().items.reduce(
                    (sum, item) => sum + parseFloat(item.producto.precio) * item.cantidad,
                    0
                );
            },
            
            getItemCount: () => {
                return get().items.reduce((sum, item) => sum + item.cantidad, 0);
            },
        }),
        {
            name: 'cart-storage',
        }
    )
);
```

**Uso:**
```jsx
import { useCartStore } from '../stores/cartStore';

function ProductoCard({ producto }) {
    const addItem = useCartStore(state => state.addItem);
    
    return (
        <button onClick={() => addItem(producto)}>
            Agregar al carrito
        </button>
    );
}

function Cart() {
    const items = useCartStore(state => state.items);
    const removeItem = useCartStore(state => state.removeItem);
    const total = useCartStore(state => state.getTotal());
    
    return (
        <div>
            <h2>Carrito</h2>
            {items.map(item => (
                <div key={item.producto.id}>
                    <span>{item.producto.nombre} x {item.cantidad}</span>
                    <button onClick={() => removeItem(item.producto.id)}>❌</button>
                </div>
            ))}
            <h3>Total: ${total.toFixed(2)}</h3>
        </div>
    );
}
```

---

## Comparación: Context API vs Zustand

| Característica | Context API | Zustand |
|---|---|---|
| Setup | Provider + Context + Hook | Store directamente |
| Código | Más verboso | Menos boilerplate |
| Rendimiento | Re-renders de todo el contexto | Re-renders selectivos |
| TypeScript | Necesita tipos manuales | Inferencia automática |
| DevTools | Requiere extensión | Incluidas |
| Persistencia | localStorage manual | Middleware `persist` |
| Curva de aprendizaje | Más alta | Más simple |

---

## Store de notificaciones (Zustand)

```javascript
// src/stores/notificationStore.js
import { create } from 'zustand';

export const useNotificationStore = create((set) => ({
    notifications: [],
    
    addNotification: (message, type = 'info') => {
        const id = Date.now();
        set((state) => ({
            notifications: [
                ...state.notifications,
                { id, message, type },
            ],
        }));
        
        // Auto-eliminar después de 3 segundos
        setTimeout(() => {
            set((state) => ({
                notifications: state.notifications.filter(n => n.id !== id),
            }));
        }, 3000);
    },
    
    removeNotification: (id) => {
        set((state) => ({
            notifications: state.notifications.filter(n => n.id !== id),
        }));
    },
}));
```

**Componente de notificaciones:**
```jsx
// src/components/Notifications.jsx
import { useNotificationStore } from '../stores/notificationStore';

export default function Notifications() {
    const notifications = useNotificationStore(state => state.notifications);
    const removeNotification = useNotificationStore(state => state.removeNotification);
    
    return (
        <div style={{
            position: 'fixed',
            top: 20,
            right: 20,
            zIndex: 1000,
        }}>
            {notifications.map(notification => (
                <div
                    key={notification.id}
                    style={{
                        padding: '1rem',
                        marginBottom: '0.5rem',
                        backgroundColor: notification.type === 'error' ? '#f44336' : '#4caf50',
                        color: 'white',
                        borderRadius: '4px',
                    }}
                >
                    {notification.message}
                    <button
                        onClick={() => removeNotification(notification.id)}
                        style={{ marginLeft: '1rem' }}
                    >
                        ✕
                    </button>
                </div>
            ))}
        </div>
    );
}
```

**Uso:**
```jsx
import { useNotificationStore } from '../stores/notificationStore';

function ProductoForm() {
    const addNotification = useNotificationStore(state => state.addNotification);
    
    const handleSubmit = async (data) => {
        const result = await productoService.create(data);
        
        if (result.success) {
            addNotification('Producto creado exitosamente', 'success');
        } else {
            addNotification('Error al crear producto', 'error');
        }
    };
    
    return <form onSubmit={handleSubmit}>...</form>;
}
```

---

## Middleware de Zustand

### Persist (localStorage)

```javascript
import { create } from 'zustand';
import { persist } from 'zustand/middleware';

export const useSettingsStore = create(
    persist(
        (set) => ({
            theme: 'light',
            language: 'es',
            
            setTheme: (theme) => set({ theme }),
            setLanguage: (language) => set({ language }),
        }),
        {
            name: 'settings-storage',
        }
    )
);
```

### Immer (actualizaciones inmutables)

```bash
npm install immer
```

```javascript
import { create } from 'zustand';
import { immer } from 'zustand/middleware/immer';

export const useTodosStore = create(
    immer((set) => ({
        todos: [],
        
        addTodo: (text) => set((state) => {
            state.todos.push({ id: Date.now(), text, completed: false });
        }),
        
        toggleTodo: (id) => set((state) => {
            const todo = state.todos.find(t => t.id === id);
            if (todo) {
                todo.completed = !todo.completed;
            }
        }),
    }))
);
```

### Devtools

```javascript
import { create } from 'zustand';
import { devtools } from 'zustand/middleware';

export const useProductosStore = create(
    devtools((set) => ({
        productos: [],
        setProductos: (productos) => set({ productos }, false, 'setProductos'),
    }))
);
```

---

## Combinar múltiples stores

```jsx
// src/App.jsx
import { useAuthStore } from './stores/authStore';
import { useCartStore } from './stores/cartStore';

function App() {
    const user = useAuthStore(state => state.user);
    const itemCount = useCartStore(state => state.getItemCount());
    
    return (
        <div>
            <header>
                <span>{user?.username}</span>
                <span>🛒 ({itemCount})</span>
            </header>
        </div>
    );
}
```

---

## Resumen

**Context API:**
- Incluida en React
- Ideal para proyectos pequeños
- Más verbosa

**Zustand:**
- Más simple y potente
- Mejor rendimiento
- DevTools y middleware incluidos
- Recomendada para proyectos medianos/grandes

**Casos de uso:**
- Autenticación → `useAuthStore`
- Carrito → `useCartStore`
- Notificaciones → `useNotificationStore`
- Configuración → `useSettingsStore`

**Próximo módulo:** CRUD completo con React + Django

---

## Recursos

- **[Context API](https://react.dev/reference/react/useContext)** - Documentación oficial
- **[Zustand](https://zustand-demo.pmnd.rs/)** - Documentación oficial
- **[Zustand Middleware](https://github.com/pmndrs/zustand#middleware)** - Persist, Immer, DevTools

¡Estado global configurado! 🌐

# Módulo 18: Fetching de Datos con useEffect

## Patrón básico de fetching

```jsx
function Usuarios() {
  const [usuarios, setUsuarios] = useState([]);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);
  
  useEffect(() => {
    fetch('/api/usuarios')
      .then(res => res.json())
      .then(data => {
        setUsuarios(data);
        setLoading(false);
      })
      .catch(err => {
        setError(err.message);
        setLoading(false);
      });
  }, []);  // Solo al montar
  
  if (loading) return <p>Cargando...</p>;
  if (error) return <p>Error: {error}</p>;
  
  return (
    <ul>
      {usuarios.map(user => (
        <li key={user.id}>{user.nombre}</li>
      ))}
    </ul>
  );
}
```

---

## Estados de carga

### Los tres estados principales

1. **Loading** (cargando)
2. **Error** (error)
3. **Success** (éxito con datos)

```jsx
function ComponenteConFetch() {
  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);
  
  useEffect(() => {
    fetch('/api/data')
      .then(res => {
        if (!res.ok) throw new Error('Error en la petición');
        return res.json();
      })
      .then(data => {
        setData(data);
        setLoading(false);
      })
      .catch(err => {
        setError(err.message);
        setLoading(false);
      });
  }, []);
  
  if (loading) return <div>⏳ Cargando...</div>;
  if (error) return <div>❌ Error: {error}</div>;
  if (!data) return <div>No hay datos</div>;
  
  return <div>✅ Datos: {JSON.stringify(data)}</div>;
}
```

---

## Fetch con async/await

```jsx
function Productos() {
  const [productos, setProductos] = useState([]);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);
  
  useEffect(() => {
    async function fetchProductos() {
      try {
        const response = await fetch('/api/productos');
        
        if (!response.ok) {
          throw new Error(`HTTP error! status: ${response.status}`);
        }
        
        const data = await response.json();
        setProductos(data);
      } catch (err) {
        setError(err.message);
      } finally {
        setLoading(false);
      }
    }
    
    fetchProductos();
  }, []);
  
  if (loading) return <p>Cargando productos...</p>;
  if (error) return <p>Error: {error}</p>;
  
  return (
    <ul>
      {productos.map(p => (
        <li key={p.id}>{p.nombre} - ${p.precio}</li>
      ))}
    </ul>
  );
}
```

---

## Fetch con parámetros dinámicos

```jsx
function DetalleProducto({ productId }) {
  const [producto, setProducto] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);
  
  useEffect(() => {
    setLoading(true);
    setError(null);
    
    fetch(`/api/productos/${productId}`)
      .then(res => res.json())
      .then(data => {
        setProducto(data);
        setLoading(false);
      })
      .catch(err => {
        setError(err.message);
        setLoading(false);
      });
  }, [productId]);  // Se ejecuta cuando productId cambia
  
  if (loading) return <p>Cargando...</p>;
  if (error) return <p>Error: {error}</p>;
  if (!producto) return <p>Producto no encontrado</p>;
  
  return (
    <div>
      <h2>{producto.nombre}</h2>
      <p>Precio: ${producto.precio}</p>
      <p>{producto.descripcion}</p>
    </div>
  );
}
```

---

## Cancelar peticiones con AbortController

**Problema:** Si el componente se desmonta mientras se hace fetch, intentar actualizar el estado causa un error.

### ❌ Sin cancelación

```jsx
useEffect(() => {
  fetch('/api/data')
    .then(res => res.json())
    .then(data => setData(data));  // ❌ Error si el componente se desmontó
}, []);
```

### ✅ Con AbortController

```jsx
function Datos() {
  const [data, setData] = useState(null);
  
  useEffect(() => {
    const controller = new AbortController();
    const signal = controller.signal;
    
    fetch('/api/data', { signal })
      .then(res => res.json())
      .then(data => setData(data))
      .catch(err => {
        if (err.name === 'AbortError') {
          console.log('Fetch cancelado');
        } else {
          console.error('Error:', err);
        }
      });
    
    // Limpieza: cancela el fetch si el componente se desmonta
    return () => controller.abort();
  }, []);
  
  return <div>{data ? JSON.stringify(data) : 'Cargando...'}</div>;
}
```

---

## Ejemplo: Búsqueda en tiempo real

```jsx
function Buscador() {
  const [query, setQuery] = useState('');
  const [resultados, setResultados] = useState([]);
  const [loading, setLoading] = useState(false);
  
  useEffect(() => {
    if (!query) {
      setResultados([]);
      return;
    }
    
    const controller = new AbortController();
    const signal = controller.signal;
    
    setLoading(true);
    
    // Debounce con timeout
    const timer = setTimeout(() => {
      fetch(`/api/search?q=${query}`, { signal })
        .then(res => res.json())
        .then(data => {
          setResultados(data);
          setLoading(false);
        })
        .catch(err => {
          if (err.name !== 'AbortError') {
            console.error('Error:', err);
            setLoading(false);
          }
        });
    }, 500);
    
    // Limpieza: cancela fetch y timer
    return () => {
      clearTimeout(timer);
      controller.abort();
    };
  }, [query]);
  
  return (
    <div>
      <input 
        value={query}
        onChange={(e) => setQuery(e.target.value)}
        placeholder="Buscar..."
      />
      
      {loading && <p>Buscando...</p>}
      
      <ul>
        {resultados.map(r => (
          <li key={r.id}>{r.nombre}</li>
        ))}
      </ul>
    </div>
  );
}
```

---

## Custom Hook: useFetch

```jsx
function useFetch(url) {
  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);
  
  useEffect(() => {
    const controller = new AbortController();
    
    async function fetchData() {
      try {
        const response = await fetch(url, { signal: controller.signal });
        
        if (!response.ok) {
          throw new Error(`HTTP error! status: ${response.status}`);
        }
        
        const json = await response.json();
        setData(json);
      } catch (err) {
        if (err.name !== 'AbortError') {
          setError(err.message);
        }
      } finally {
        setLoading(false);
      }
    }
    
    fetchData();
    
    return () => controller.abort();
  }, [url]);
  
  return { data, loading, error };
}

// Uso
function Usuarios() {
  const { data, loading, error } = useFetch('/api/usuarios');
  
  if (loading) return <p>Cargando...</p>;
  if (error) return <p>Error: {error}</p>;
  
  return (
    <ul>
      {data.map(user => (
        <li key={user.id}>{user.nombre}</li>
      ))}
    </ul>
  );
}
```

---

## Paginación

```jsx
function ListaPaginada() {
  const [pagina, setPagina] = useState(1);
  const [items, setItems] = useState([]);
  const [loading, setLoading] = useState(false);
  const [hasMore, setHasMore] = useState(true);
  
  useEffect(() => {
    const controller = new AbortController();
    
    async function fetchPagina() {
      setLoading(true);
      
      try {
        const response = await fetch(
          `/api/items?page=${pagina}&limit=10`,
          { signal: controller.signal }
        );
        const data = await response.json();
        
        setItems(data.items);
        setHasMore(data.hasMore);
      } catch (err) {
        if (err.name !== 'AbortError') {
          console.error('Error:', err);
        }
      } finally {
        setLoading(false);
      }
    }
    
    fetchPagina();
    
    return () => controller.abort();
  }, [pagina]);
  
  return (
    <div>
      {loading ? (
        <p>Cargando...</p>
      ) : (
        <ul>
          {items.map(item => (
            <li key={item.id}>{item.nombre}</li>
          ))}
        </ul>
      )}
      
      <div>
        <button 
          onClick={() => setPagina(p => p - 1)}
          disabled={pagina === 1 || loading}
        >
          Anterior
        </button>
        
        <span>Página {pagina}</span>
        
        <button 
          onClick={() => setPagina(p => p + 1)}
          disabled={!hasMore || loading}
        >
          Siguiente
        </button>
      </div>
    </div>
  );
}
```

---

## Infinite scroll (scroll infinito)

```jsx
function InfiniteList() {
  const [items, setItems] = useState([]);
  const [pagina, setPagina] = useState(1);
  const [loading, setLoading] = useState(false);
  const [hasMore, setHasMore] = useState(true);
  
  useEffect(() => {
    if (!hasMore) return;
    
    const controller = new AbortController();
    setLoading(true);
    
    fetch(`/api/items?page=${pagina}&limit=20`, { signal: controller.signal })
      .then(res => res.json())
      .then(data => {
        setItems(prev => [...prev, ...data.items]);
        setHasMore(data.hasMore);
        setLoading(false);
      })
      .catch(err => {
        if (err.name !== 'AbortError') {
          console.error('Error:', err);
          setLoading(false);
        }
      });
    
    return () => controller.abort();
  }, [pagina, hasMore]);
  
  const handleScroll = () => {
    if (
      window.innerHeight + document.documentElement.scrollTop
      >= document.documentElement.offsetHeight - 100
      && !loading
      && hasMore
    ) {
      setPagina(prev => prev + 1);
    }
  };
  
  useEffect(() => {
    window.addEventListener('scroll', handleScroll);
    return () => window.removeEventListener('scroll', handleScroll);
  }, [loading, hasMore]);
  
  return (
    <div>
      <ul>
        {items.map(item => (
          <li key={item.id}>{item.nombre}</li>
        ))}
      </ul>
      {loading && <p>Cargando más...</p>}
      {!hasMore && <p>No hay más items</p>}
    </div>
  );
}
```

---

## Fetch con POST

```jsx
function CrearUsuario() {
  const [nombre, setNombre] = useState('');
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState(null);
  const [success, setSuccess] = useState(false);
  
  const handleSubmit = async (e) => {
    e.preventDefault();
    setLoading(true);
    setError(null);
    setSuccess(false);
    
    try {
      const response = await fetch('/api/usuarios', {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json'
        },
        body: JSON.stringify({ nombre })
      });
      
      if (!response.ok) {
        throw new Error('Error al crear usuario');
      }
      
      setSuccess(true);
      setNombre('');
    } catch (err) {
      setError(err.message);
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
        disabled={loading}
      />
      
      <button type="submit" disabled={loading || !nombre}>
        {loading ? 'Creando...' : 'Crear Usuario'}
      </button>
      
      {error && <p style={{ color: 'red' }}>Error: {error}</p>}
      {success && <p style={{ color: 'green' }}>✅ Usuario creado</p>}
    </form>
  );
}
```

---

## Recargar datos (refetch)

```jsx
function ListaProductos() {
  const [productos, setProductos] = useState([]);
  const [loading, setLoading] = useState(false);
  const [refresh, setRefresh] = useState(0);
  
  useEffect(() => {
    setLoading(true);
    
    fetch('/api/productos')
      .then(res => res.json())
      .then(data => {
        setProductos(data);
        setLoading(false);
      });
  }, [refresh]);  // Se ejecuta cuando refresh cambia
  
  const recargar = () => {
    setRefresh(prev => prev + 1);
  };
  
  return (
    <div>
      <button onClick={recargar} disabled={loading}>
        {loading ? 'Recargando...' : '🔄 Recargar'}
      </button>
      
      <ul>
        {productos.map(p => (
          <li key={p.id}>{p.nombre}</li>
        ))}
      </ul>
    </div>
  );
}
```

---

## Manejo de errores avanzado

```jsx
function DatosConErrores() {
  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);
  const [retry, setRetry] = useState(0);
  
  useEffect(() => {
    const controller = new AbortController();
    
    async function fetchData() {
      setLoading(true);
      setError(null);
      
      try {
        const response = await fetch('/api/data', {
          signal: controller.signal
        });
        
        // Errores HTTP
        if (!response.ok) {
          if (response.status === 404) {
            throw new Error('Datos no encontrados');
          } else if (response.status === 500) {
            throw new Error('Error del servidor');
          } else {
            throw new Error(`Error HTTP: ${response.status}`);
          }
        }
        
        const json = await response.json();
        setData(json);
      } catch (err) {
        if (err.name === 'AbortError') {
          console.log('Fetch cancelado');
        } else {
          setError(err.message);
        }
      } finally {
        setLoading(false);
      }
    }
    
    fetchData();
    
    return () => controller.abort();
  }, [retry]);
  
  const reintentar = () => setRetry(prev => prev + 1);
  
  if (loading) return <p>⏳ Cargando...</p>;
  
  if (error) {
    return (
      <div>
        <p>❌ Error: {error}</p>
        <button onClick={reintentar}>🔄 Reintentar</button>
      </div>
    );
  }
  
  return <div>✅ Datos: {JSON.stringify(data)}</div>;
}
```

---

## Fetch con múltiples endpoints

```jsx
function Dashboard() {
  const [usuario, setUsuario] = useState(null);
  const [estadisticas, setEstadisticas] = useState(null);
  const [notificaciones, setNotificaciones] = useState([]);
  const [loading, setLoading] = useState(true);
  
  useEffect(() => {
    const controller = new AbortController();
    
    async function fetchTodo() {
      try {
        // Fetch paralelo
        const [userRes, statsRes, notifsRes] = await Promise.all([
          fetch('/api/usuario', { signal: controller.signal }),
          fetch('/api/estadisticas', { signal: controller.signal }),
          fetch('/api/notificaciones', { signal: controller.signal })
        ]);
        
        const [userData, statsData, notifsData] = await Promise.all([
          userRes.json(),
          statsRes.json(),
          notifsRes.json()
        ]);
        
        setUsuario(userData);
        setEstadisticas(statsData);
        setNotificaciones(notifsData);
      } catch (err) {
        if (err.name !== 'AbortError') {
          console.error('Error:', err);
        }
      } finally {
        setLoading(false);
      }
    }
    
    fetchTodo();
    
    return () => controller.abort();
  }, []);
  
  if (loading) return <p>Cargando dashboard...</p>;
  
  return (
    <div>
      <h1>Dashboard de {usuario?.nombre}</h1>
      <p>Total ventas: {estadisticas?.total}</p>
      <p>Notificaciones: {notificaciones.length}</p>
    </div>
  );
}
```

---

## Ejercicio 1: Lista de usuarios

Crea un componente que cargue y muestre una lista de usuarios:

```jsx
function ListaUsuarios() {
  // Tu código aquí:
  // - Estado para usuarios, loading, error
  // - useEffect para fetch
  // - Renderizado condicional
}

// API: https://jsonplaceholder.typicode.com/users
```

<details>
<summary>Ver solución</summary>

```jsx
function ListaUsuarios() {
  const [usuarios, setUsuarios] = useState([]);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);
  
  useEffect(() => {
    fetch('https://jsonplaceholder.typicode.com/users')
      .then(res => {
        if (!res.ok) throw new Error('Error al cargar usuarios');
        return res.json();
      })
      .then(data => {
        setUsuarios(data);
        setLoading(false);
      })
      .catch(err => {
        setError(err.message);
        setLoading(false);
      });
  }, []);
  
  if (loading) return <p>Cargando usuarios...</p>;
  if (error) return <p>Error: {error}</p>;
  
  return (
    <ul>
      {usuarios.map(user => (
        <li key={user.id}>
          {user.name} - {user.email}
        </li>
      ))}
    </ul>
  );
}
```
</details>

---

## Ejercicio 2: Detalle de post

Crea un componente que cargue un post según su ID:

```jsx
function DetallePost({ postId }) {
  // Tu código aquí
  // API: https://jsonplaceholder.typicode.com/posts/{id}
}
```

<details>
<summary>Ver solución</summary>

```jsx
function DetallePost({ postId }) {
  const [post, setPost] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);
  
  useEffect(() => {
    setLoading(true);
    setError(null);
    
    fetch(`https://jsonplaceholder.typicode.com/posts/${postId}`)
      .then(res => {
        if (!res.ok) throw new Error('Post no encontrado');
        return res.json();
      })
      .then(data => {
        setPost(data);
        setLoading(false);
      })
      .catch(err => {
        setError(err.message);
        setLoading(false);
      });
  }, [postId]);
  
  if (loading) return <p>Cargando post...</p>;
  if (error) return <p>Error: {error}</p>;
  if (!post) return <p>Post no encontrado</p>;
  
  return (
    <article>
      <h2>{post.title}</h2>
      <p>{post.body}</p>
    </article>
  );
}
```
</details>

---

## Ejercicio 3: Buscador de posts

Crea un buscador que filtre posts por título:

```jsx
function BuscadorPosts() {
  // Tu código aquí:
  // - Input para búsqueda
  // - Fetch con query
  // - Mostrar resultados
  
  // API: https://jsonplaceholder.typicode.com/posts
}
```

<details>
<summary>Ver solución</summary>

```jsx
function BuscadorPosts() {
  const [query, setQuery] = useState('');
  const [posts, setPosts] = useState([]);
  const [loading, setLoading] = useState(false);
  
  useEffect(() => {
    if (!query) {
      setPosts([]);
      return;
    }
    
    const controller = new AbortController();
    setLoading(true);
    
    const timer = setTimeout(() => {
      fetch('https://jsonplaceholder.typicode.com/posts', {
        signal: controller.signal
      })
        .then(res => res.json())
        .then(data => {
          const filtrados = data.filter(post =>
            post.title.toLowerCase().includes(query.toLowerCase())
          );
          setPosts(filtrados);
          setLoading(false);
        })
        .catch(err => {
          if (err.name !== 'AbortError') {
            console.error('Error:', err);
            setLoading(false);
          }
        });
    }, 500);
    
    return () => {
      clearTimeout(timer);
      controller.abort();
    };
  }, [query]);
  
  return (
    <div>
      <input 
        value={query}
        onChange={(e) => setQuery(e.target.value)}
        placeholder="Buscar posts..."
      />
      
      {loading && <p>Buscando...</p>}
      
      <ul>
        {posts.map(post => (
          <li key={post.id}>{post.title}</li>
        ))}
      </ul>
    </div>
  );
}
```
</details>

---

## Resumen

✅ **Patrón estándar:** loading, error, success states  
✅ **Array vacío `[]`** para fetch al montar  
✅ **AbortController** para cancelar fetch al desmontar  
✅ **Async/await** para código más limpio  
✅ **try/catch/finally** para manejo de errores  
✅ **Debounce** para búsquedas en tiempo real  
✅ **Custom Hooks** para reutilizar lógica de fetch  
✅ **Promise.all** para múltiples fetches paralelos

---

## Siguiente paso

Ya sabes hacer fetching de datos. Ahora aprenderás sobre **funciones de limpieza** (cleanup functions) en useEffect.

**Siguiente:** [Módulo 19 - Cleanup functions](./19-cleanup-functions.md)

**Anterior:** [Módulo 17 - Dependencias de useEffect](./17-dependencias-useeffect.md)

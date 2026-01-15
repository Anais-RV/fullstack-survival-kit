# Módulo 09: Renderizado Condicional

## Mostrar/ocultar contenido según condiciones

En React, el **renderizado condicional** funciona igual que en JavaScript: usas `if`, operador ternario, `&&`, etc.

---

## 1. If/Else con return temprano

### Early return

```jsx
function Saludo({ usuario }) {
  if (!usuario) {
    return <p>Por favor inicia sesión</p>;
  }
  
  return <h1>Bienvenido, {usuario.nombre}</h1>;
}
```

### Múltiples condiciones

```jsx
function Estado({ status }) {
  if (status === 'loading') {
    return <div>Cargando...</div>;
  }
  
  if (status === 'error') {
    return <div className="error">Error al cargar datos</div>;
  }
  
  if (status === 'empty') {
    return <div>No hay datos disponibles</div>;
  }
  
  return <div>Datos cargados correctamente</div>;
}
```

---

## 2. Operador ternario

### Sintaxis básica

```jsx
function Mensaje({ esError }) {
  return (
    <div>
      {esError ? (
        <p className="error">❌ Hubo un error</p>
      ) : (
        <p className="success">✅ Todo correcto</p>
      )}
    </div>
  );
}
```

### Ternario para clases CSS

```jsx
function Button({ activo }) {
  return (
    <button className={activo ? 'btn-active' : 'btn-inactive'}>
      {activo ? 'Activado' : 'Desactivado'}
    </button>
  );
}
```

### Ternarios anidados (evita abusar)

```jsx
function Badge({ tipo }) {
  return (
    <span className={
      tipo === 'success' ? 'badge-success' :
      tipo === 'error' ? 'badge-error' :
      tipo === 'warning' ? 'badge-warning' :
      'badge-default'
    }>
      {tipo}
    </span>
  );
}

// Mejor con objeto:
function Badge({ tipo }) {
  const clases = {
    success: 'badge-success',
    error: 'badge-error',
    warning: 'badge-warning',
    default: 'badge-default'
  };
  
  return <span className={clases[tipo] || clases.default}>{tipo}</span>;
}
```

---

## 3. Operador && (AND lógico)

### Mostrar solo si es verdadero

```jsx
function Notificacion({ hayMensajes, cantidad }) {
  return (
    <div>
      {hayMensajes && (
        <div className="notificacion">
          Tienes {cantidad} mensajes nuevos
        </div>
      )}
    </div>
  );
}
```

### Cuidado con valores falsy

```jsx
function Contador({ total }) {
  // ❌ Si total es 0, muestra "0" en pantalla
  return <div>{total && <span>{total} items</span>}</div>;
  
  // ✅ Compara explícitamente
  return <div>{total > 0 && <span>{total} items</span>}</div>;
}
```

### Múltiples condiciones

```jsx
function UserProfile({ usuario }) {
  return (
    <div>
      <h1>{usuario.nombre}</h1>
      {usuario.esAdmin && <span className="badge">Admin</span>}
      {usuario.verificado && <span className="badge">✓ Verificado</span>}
      {usuario.premium && <span className="badge">⭐ Premium</span>}
    </div>
  );
}
```

---

## 4. Operador || (OR lógico)

### Valores por defecto

```jsx
function Avatar({ src, nombre }) {
  return (
    <img 
      src={src || '/default-avatar.png'}
      alt={nombre || 'Usuario'}
    />
  );
}
```

### Nullish coalescing (??)

```jsx
function Precio({ valor }) {
  // || fallaría si valor es 0
  const precioFinal = valor ?? 10;  // Solo null/undefined usan default
  
  return <p>${precioFinal}</p>;
}
```

---

## 5. Variables condicionales

### Asignar antes del return

```jsx
function Estado({ conectado }) {
  let icono;
  let mensaje;
  let clase;
  
  if (conectado) {
    icono = '🟢';
    mensaje = 'En línea';
    clase = 'online';
  } else {
    icono = '🔴';
    mensaje = 'Desconectado';
    clase = 'offline';
  }
  
  return (
    <div className={clase}>
      {icono} {mensaje}
    </div>
  );
}
```

### Con switch

```jsx
function Alerta({ tipo }) {
  let contenido;
  
  switch (tipo) {
    case 'success':
      contenido = <div className="alert-success">✅ Operación exitosa</div>;
      break;
    case 'error':
      contenido = <div className="alert-error">❌ Error</div>;
      break;
    case 'warning':
      contenido = <div className="alert-warning">⚠️ Advertencia</div>;
      break;
    default:
      contenido = <div className="alert-info">ℹ️ Información</div>;
  }
  
  return contenido;
}
```

---

## 6. Renderizar null (no mostrar nada)

```jsx
function PrivateContent({ esAdmin }) {
  if (!esAdmin) {
    return null;  // No renderiza nada
  }
  
  return (
    <div>
      <h2>Panel de Administración</h2>
      <button>Gestionar usuarios</button>
    </div>
  );
}
```

---

## 7. Patrones comunes

### Loading → Error → Success

```jsx
function UserProfile({ userId }) {
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);
  
  useEffect(() => {
    fetch(`/api/users/${userId}`)
      .then(res => res.json())
      .then(data => {
        setUser(data);
        setLoading(false);
      })
      .catch(err => {
        setError(err.message);
        setLoading(false);
      });
  }, [userId]);
  
  if (loading) {
    return <div className="loading">Cargando...</div>;
  }
  
  if (error) {
    return <div className="error">Error: {error}</div>;
  }
  
  return (
    <div>
      <h1>{user.name}</h1>
      <p>{user.email}</p>
    </div>
  );
}
```

### Empty state

```jsx
function ListaTareas({ tareas }) {
  if (tareas.length === 0) {
    return (
      <div className="empty-state">
        <p>No tienes tareas pendientes</p>
        <button>Agregar tarea</button>
      </div>
    );
  }
  
  return (
    <ul>
      {tareas.map(tarea => (
        <li key={tarea.id}>{tarea.texto}</li>
      ))}
    </ul>
  );
}
```

### Autenticación

```jsx
function App() {
  const [usuario, setUsuario] = useState(null);
  
  if (!usuario) {
    return <LoginPage onLogin={setUsuario} />;
  }
  
  return <Dashboard usuario={usuario} />;
}
```

---

## 8. Componente condicional reutilizable

```jsx
function Show({ when, children, fallback = null }) {
  return when ? children : fallback;
}

// Uso:
<Show when={esAdmin}>
  <AdminPanel />
</Show>

<Show when={hayMensajes} fallback={<p>No hay mensajes</p>}>
  <MessageList />
</Show>
```

---

## 9. Inline conditional rendering

### Dentro de JSX

```jsx
function ProductCard({ producto }) {
  return (
    <div className="card">
      <h3>{producto.nombre}</h3>
      <p>${producto.precio}</p>
      
      {producto.descuento && (
        <span className="descuento">-{producto.descuento}%</span>
      )}
      
      {producto.stock > 0 ? (
        <button>Agregar al carrito</button>
      ) : (
        <span className="agotado">Agotado</span>
      )}
      
      {producto.nuevo && <span className="badge">Nuevo</span>}
      {producto.destacado && <span className="badge">⭐ Destacado</span>}
    </div>
  );
}
```

---

## 10. Función auxiliar para condicionales

```jsx
function Dashboard({ usuario }) {
  const puedeVerReportes = () => {
    return usuario.rol === 'admin' || usuario.rol === 'gerente';
  };
  
  const puedeEditarUsuarios = () => {
    return usuario.rol === 'admin';
  };
  
  return (
    <div>
      <h1>Dashboard</h1>
      
      {puedeVerReportes() && (
        <section>
          <h2>Reportes</h2>
          <ReportList />
        </section>
      )}
      
      {puedeEditarUsuarios() && (
        <section>
          <h2>Gestión de Usuarios</h2>
          <UserManagement />
        </section>
      )}
    </div>
  );
}
```

---

## 11. Objeto de configuración

```jsx
function Status({ tipo }) {
  const config = {
    loading: {
      icono: '⏳',
      mensaje: 'Cargando...',
      clase: 'status-loading'
    },
    success: {
      icono: '✅',
      mensaje: 'Completado',
      clase: 'status-success'
    },
    error: {
      icono: '❌',
      mensaje: 'Error',
      clase: 'status-error'
    }
  };
  
  const { icono, mensaje, clase } = config[tipo] || config.loading;
  
  return (
    <div className={clase}>
      {icono} {mensaje}
    </div>
  );
}
```

---

## 12. Componentes de guardia (Guard)

```jsx
function RequireAuth({ children }) {
  const { usuario } = useAuth();
  
  if (!usuario) {
    return <Navigate to="/login" />;
  }
  
  return children;
}

function RequireAdmin({ children }) {
  const { usuario } = useAuth();
  
  if (!usuario || usuario.rol !== 'admin') {
    return <div>No tienes permisos</div>;
  }
  
  return children;
}

// Uso:
<RequireAuth>
  <Dashboard />
</RequireAuth>

<RequireAdmin>
  <AdminPanel />
</RequireAdmin>
```

---

## Ejemplo completo: Perfil de usuario

```jsx
function UserProfile({ userId }) {
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);
  const [editing, setEditing] = useState(false);
  
  useEffect(() => {
    fetch(`/api/users/${userId}`)
      .then(res => res.json())
      .then(data => {
        setUser(data);
        setLoading(false);
      })
      .catch(err => {
        setError(err.message);
        setLoading(false);
      });
  }, [userId]);
  
  // Loading state
  if (loading) {
    return (
      <div className="loading-container">
        <div className="spinner"></div>
        <p>Cargando perfil...</p>
      </div>
    );
  }
  
  // Error state
  if (error) {
    return (
      <div className="error-container">
        <h2>❌ Error</h2>
        <p>{error}</p>
        <button onClick={() => window.location.reload()}>
          Reintentar
        </button>
      </div>
    );
  }
  
  // No user
  if (!user) {
    return <div>Usuario no encontrado</div>;
  }
  
  // Editing mode
  if (editing) {
    return <UserEditForm user={user} onSave={() => setEditing(false)} />;
  }
  
  // Success - mostrar perfil
  return (
    <div className="user-profile">
      <img 
        src={user.avatar || '/default-avatar.png'} 
        alt={user.name}
      />
      
      <h1>{user.name}</h1>
      
      {user.verified && (
        <span className="badge verified">✓ Verificado</span>
      )}
      
      {user.premium && (
        <span className="badge premium">⭐ Premium</span>
      )}
      
      <p>{user.bio || 'Sin biografía'}</p>
      
      <div className="stats">
        <div>
          <strong>{user.followers}</strong>
          <span>Seguidores</span>
        </div>
        <div>
          <strong>{user.following}</strong>
          <span>Siguiendo</span>
        </div>
      </div>
      
      {user.isOwner && (
        <button onClick={() => setEditing(true)}>
          Editar perfil
        </button>
      )}
      
      {!user.isOwner && (
        <button>
          {user.isFollowing ? 'Dejar de seguir' : 'Seguir'}
        </button>
      )}
      
      {user.posts.length > 0 ? (
        <PostList posts={user.posts} />
      ) : (
        <div className="empty-state">
          <p>Este usuario aún no ha publicado nada</p>
        </div>
      )}
    </div>
  );
}
```

---

## Buenas prácticas

✅ **Early returns** para casos simples  
✅ **Operador ternario** para alternativas binarias  
✅ **Operador &&** para mostrar/ocultar  
✅ **Variables** para lógica compleja  
✅ **null** para no renderizar nada  
✅ **Extraer a funciones** cuando sea complejo  
✅ **Componentes guard** para protección  

---

## Errores comunes

### 1. Usar if dentro de JSX

```jsx
// ❌ Error - if no funciona en JSX
<div>
  {if (condicion) { <p>Texto</p> }}
</div>

// ✅ Correcto - ternario
<div>
  {condicion ? <p>Texto</p> : null}
</div>

// ✅ Correcto - &&
<div>
  {condicion && <p>Texto</p>}
</div>
```

### 2. Olvidar valores falsy con &&

```jsx
// ❌ Si count es 0, muestra "0"
<div>{count && <span>{count} items</span>}</div>

// ✅ Compara explícitamente
<div>{count > 0 && <span>{count} items</span>}</div>
```

### 3. Ternarios muy anidados

```jsx
// ❌ Difícil de leer
{tipo === 'a' ? <A /> : tipo === 'b' ? <B /> : tipo === 'c' ? <C /> : <D />}

// ✅ Usa función o early returns
function renderTipo(tipo) {
  if (tipo === 'a') return <A />;
  if (tipo === 'b') return <B />;
  if (tipo === 'c') return <C />;
  return <D />;
}
```

---

## Resumen

✅ **If/else con early returns**  
✅ **Operador ternario para alternativas**  
✅ **&& para mostrar/ocultar**  
✅ **|| y ?? para valores por defecto**  
✅ **null para no renderizar**  
✅ **Variables para lógica compleja**  
✅ **Componentes guard para protección**

---

## Siguiente paso

Dominas renderizado condicional. Ahora aprenderás **listas y keys**.

**Siguiente:** [Módulo 10 - Listas y keys](./10-listas-keys.md)

**Anterior:** [Módulo 08 - Composición de componentes](./08-composicion-componentes.md)

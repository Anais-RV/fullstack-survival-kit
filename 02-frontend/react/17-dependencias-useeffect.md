# Módulo 17: Dependencias de useEffect

## El array de dependencias

El **segundo argumento** de useEffect es un **array de dependencias**:

```jsx
useEffect(() => {
  // Efecto
}, [dependencia1, dependencia2]);
```

**Controla CUÁNDO se ejecuta el efecto.**

---

## Tres casos principales

### 1. Sin array (se ejecuta siempre)

```jsx
useEffect(() => {
  console.log('Se ejecuta en CADA render');
});
```

**Se ejecuta:**
- En el primer render
- Después de cada actualización

---

### 2. Array vacío (se ejecuta una vez)

```jsx
useEffect(() => {
  console.log('Se ejecuta SOLO en el primer render');
}, []);
```

**Se ejecuta:**
- Solo en el primer render
- Equivalente a `componentDidMount`

---

### 3. Con dependencias (se ejecuta cuando cambian)

```jsx
useEffect(() => {
  console.log('Se ejecuta cuando count cambia');
}, [count]);
```

**Se ejecuta:**
- En el primer render
- Cuando `count` cambia

---

## Comparación de los tres casos

```jsx
function Ejemplo() {
  const [count, setCount] = useState(0);
  const [nombre, setNombre] = useState('');
  
  // 1. Siempre
  useEffect(() => {
    console.log('Efecto 1: siempre');
  });
  
  // 2. Solo una vez
  useEffect(() => {
    console.log('Efecto 2: solo al montar');
  }, []);
  
  // 3. Cuando count cambia
  useEffect(() => {
    console.log('Efecto 3: cuando count cambia');
  }, [count]);
  
  return (
    <div>
      <button onClick={() => setCount(count + 1)}>Count: {count}</button>
      <input value={nombre} onChange={(e) => setNombre(e.target.value)} />
    </div>
  );
}
```

**Al hacer click en el botón:**
- Efecto 1 ✅ (siempre)
- Efecto 2 ❌ (solo en el primer render)
- Efecto 3 ✅ (count cambió)

**Al escribir en el input:**
- Efecto 1 ✅ (siempre)
- Efecto 2 ❌ (solo en el primer render)
- Efecto 3 ❌ (count no cambió)

---

## Efecto que se ejecuta una sola vez

```jsx
function ComponenteConFetch() {
  const [data, setData] = useState(null);
  
  useEffect(() => {
    fetch('/api/data')
      .then(res => res.json())
      .then(data => setData(data));
  }, []);  // ✅ Array vacío: solo al montar
  
  return <div>{data ? JSON.stringify(data) : 'Cargando...'}</div>;
}
```

**Sin el array vacío:**
- Fetch en cada render
- Bucle infinito: fetch → setState → render → fetch → ...

**Con array vacío:**
- Fetch solo una vez al montar

---

## Múltiples dependencias

```jsx
function Buscador() {
  const [query, setQuery] = useState('');
  const [categoria, setCategoria] = useState('todas');
  const [resultados, setResultados] = useState([]);
  
  useEffect(() => {
    fetch(`/api/search?q=${query}&cat=${categoria}`)
      .then(res => res.json())
      .then(data => setResultados(data));
  }, [query, categoria]);  // Se ejecuta cuando query O categoria cambian
  
  return (
    <div>
      <input value={query} onChange={(e) => setQuery(e.target.value)} />
      <select value={categoria} onChange={(e) => setCategoria(e.target.value)}>
        <option value="todas">Todas</option>
        <option value="tech">Tech</option>
        <option value="ropa">Ropa</option>
      </select>
      <ul>
        {resultados.map(r => <li key={r.id}>{r.nombre}</li>)}
      </ul>
    </div>
  );
}
```

---

## Dependencias y objetos/arrays

### ⚠️ Problema: Objetos creados en cada render

```jsx
function Problema() {
  const [count, setCount] = useState(0);
  
  const config = { api: '/api', count };  // ⚠️ Nuevo objeto en cada render
  
  useEffect(() => {
    console.log('Efecto ejecutado');
  }, [config]);  // ⚠️ Se ejecuta en CADA render
  
  return <button onClick={() => setCount(count + 1)}>{count}</button>;
}
```

**Problema:**
- `config` es un nuevo objeto en cada render
- React compara por **referencia**, no por valor
- `config !== config` (diferentes referencias)
- Efecto se ejecuta siempre

---

### ✅ Solución 1: Incluir solo valores primitivos

```jsx
function Solucion1() {
  const [count, setCount] = useState(0);
  
  useEffect(() => {
    const config = { api: '/api', count };
    console.log('Efecto ejecutado', config);
  }, [count]);  // ✅ Depende de count (primitivo)
  
  return <button onClick={() => setCount(count + 1)}>{count}</button>;
}
```

---

### ✅ Solución 2: useMemo (avanzado)

```jsx
function Solucion2() {
  const [count, setCount] = useState(0);
  
  const config = useMemo(() => ({
    api: '/api',
    count
  }), [count]);  // Solo crea nuevo objeto si count cambia
  
  useEffect(() => {
    console.log('Efecto ejecutado', config);
  }, [config]);
  
  return <button onClick={() => setCount(count + 1)}>{count}</button>;
}
```

---

## Ejemplo: Título dinámico

```jsx
function PaginaProducto({ productId }) {
  const [producto, setProducto] = useState(null);
  
  useEffect(() => {
    fetch(`/api/productos/${productId}`)
      .then(res => res.json())
      .then(data => setProducto(data));
  }, [productId]);  // Se ejecuta cuando productId cambia
  
  useEffect(() => {
    if (producto) {
      document.title = producto.nombre;
    }
  }, [producto]);  // Se ejecuta cuando producto cambia
  
  return producto ? <div>{producto.nombre}</div> : <div>Cargando...</div>;
}
```

---

## Ejemplo: Guardar en localStorage solo cuando cambia

```jsx
function FormularioPersistente() {
  const [form, setForm] = useState(() => {
    const guardado = localStorage.getItem('form');
    return guardado ? JSON.parse(guardado) : { nombre: '', email: '' };
  });
  
  useEffect(() => {
    localStorage.setItem('form', JSON.stringify(form));
  }, [form]);  // Solo guarda cuando form cambia
  
  const handleChange = (campo, valor) => {
    setForm({ ...form, [campo]: valor });
  };
  
  return (
    <div>
      <input 
        value={form.nombre}
        onChange={(e) => handleChange('nombre', e.target.value)}
      />
      <input 
        value={form.email}
        onChange={(e) => handleChange('email', e.target.value)}
      />
    </div>
  );
}
```

---

## Efecto con múltiples efectos independientes

```jsx
function Dashboard() {
  const [usuario, setUsuario] = useState(null);
  const [estadisticas, setEstadisticas] = useState(null);
  
  // Efecto 1: Cargar usuario (solo una vez)
  useEffect(() => {
    fetch('/api/usuario')
      .then(res => res.json())
      .then(data => setUsuario(data));
  }, []);
  
  // Efecto 2: Actualizar título cuando usuario carga
  useEffect(() => {
    if (usuario) {
      document.title = `Dashboard - ${usuario.nombre}`;
    }
  }, [usuario]);
  
  // Efecto 3: Cargar estadísticas cuando usuario carga
  useEffect(() => {
    if (usuario) {
      fetch(`/api/estadisticas/${usuario.id}`)
        .then(res => res.json())
        .then(data => setEstadisticas(data));
    }
  }, [usuario]);
  
  return (
    <div>
      {usuario && <h1>Hola, {usuario.nombre}</h1>}
      {estadisticas && <p>Total: {estadisticas.total}</p>}
    </div>
  );
}
```

---

## Dependencias con props

```jsx
function Reloj({ intervalo = 1000 }) {
  const [hora, setHora] = useState(new Date());
  
  useEffect(() => {
    const timer = setInterval(() => {
      setHora(new Date());
    }, intervalo);
    
    return () => clearInterval(timer);
  }, [intervalo]);  // Se reinicia cuando intervalo cambia
  
  return <h1>{hora.toLocaleTimeString()}</h1>;
}

// Uso:
<Reloj intervalo={1000} />  // Actualiza cada segundo
<Reloj intervalo={5000} />  // Actualiza cada 5 segundos
```

---

## Errores comunes

### ❌ Olvidar dependencias

```jsx
function Mal() {
  const [count, setCount] = useState(0);
  
  useEffect(() => {
    console.log('Count:', count);
  }, []);  // ❌ Falta count en las dependencias
  
  // Solo muestra el valor inicial
}
```

**Problema:** El efecto usa `count` pero no lo incluye en dependencias. Siempre verá el valor inicial.

---

### ✅ Incluir todas las dependencias

```jsx
function Bien() {
  const [count, setCount] = useState(0);
  
  useEffect(() => {
    console.log('Count:', count);
  }, [count]);  // ✅ Incluye count
}
```

---

### ❌ Incluir funciones sin useCallback

```jsx
function Mal() {
  const [count, setCount] = useState(0);
  
  const log = () => {
    console.log(count);
  };
  
  useEffect(() => {
    log();
  }, [log]);  // ⚠️ log es nuevo en cada render
  
  // Se ejecuta en cada render
}
```

---

### ✅ Mover función dentro del efecto

```jsx
function Bien() {
  const [count, setCount] = useState(0);
  
  useEffect(() => {
    const log = () => {
      console.log(count);
    };
    
    log();
  }, [count]);  // ✅ Solo depende de count
}
```

---

### ✅ O usar useCallback (avanzado)

```jsx
function Bien() {
  const [count, setCount] = useState(0);
  
  const log = useCallback(() => {
    console.log(count);
  }, [count]);
  
  useEffect(() => {
    log();
  }, [log]);
}
```

---

## ESLint y dependencias

React recomienda usar **eslint-plugin-react-hooks**:

```json
{
  "plugins": ["react-hooks"],
  "rules": {
    "react-hooks/exhaustive-deps": "warn"
  }
}
```

**Te avisa si:**
- Faltan dependencias
- Hay dependencias innecesarias

---

## Ejemplo: Búsqueda con debounce

```jsx
function Buscador() {
  const [query, setQuery] = useState('');
  const [resultados, setResultados] = useState([]);
  
  useEffect(() => {
    // Espera 500ms después del último cambio
    const timer = setTimeout(() => {
      if (query) {
        fetch(`/api/search?q=${query}`)
          .then(res => res.json())
          .then(data => setResultados(data));
      }
    }, 500);
    
    // Limpia el timer anterior
    return () => clearTimeout(timer);
  }, [query]);
  
  return (
    <div>
      <input 
        value={query}
        onChange={(e) => setQuery(e.target.value)}
        placeholder="Buscar..."
      />
      <ul>
        {resultados.map(r => <li key={r.id}>{r.nombre}</li>)}
      </ul>
    </div>
  );
}
```

**Flujo:**
1. Usuario escribe "react"
2. Timer de 500ms comienza
3. Usuario escribe "j" → "reactj"
4. Timer anterior se cancela
5. Nuevo timer de 500ms comienza
6. Si pasan 500ms sin cambios, se hace el fetch

---

## Ejemplo: Sincronizar con prop

```jsx
function InputControlado({ valorInicial }) {
  const [valor, setValor] = useState(valorInicial);
  
  // Sincroniza cuando valorInicial cambia
  useEffect(() => {
    setValor(valorInicial);
  }, [valorInicial]);
  
  return (
    <input 
      value={valor}
      onChange={(e) => setValor(e.target.value)}
    />
  );
}
```

---

## Ejemplo: Cargar datos paginados

```jsx
function ListaPaginada() {
  const [pagina, setPagina] = useState(1);
  const [items, setItems] = useState([]);
  const [loading, setLoading] = useState(false);
  
  useEffect(() => {
    setLoading(true);
    
    fetch(`/api/items?page=${pagina}`)
      .then(res => res.json())
      .then(data => {
        setItems(data);
        setLoading(false);
      });
  }, [pagina]);  // Se ejecuta cuando pagina cambia
  
  return (
    <div>
      {loading ? (
        <p>Cargando...</p>
      ) : (
        <ul>
          {items.map(item => <li key={item.id}>{item.nombre}</li>)}
        </ul>
      )}
      
      <button onClick={() => setPagina(pagina - 1)} disabled={pagina === 1}>
        Anterior
      </button>
      <span>Página {pagina}</span>
      <button onClick={() => setPagina(pagina + 1)}>
        Siguiente
      </button>
    </div>
  );
}
```

---

## Ejercicio 1: Contador con intervalo

Crea un contador automático que incremente cada segundo:

```jsx
function ContadorAutomatico() {
  const [count, setCount] = useState(0);
  
  // Tu código aquí: useEffect con setInterval
  
  return <h1>{count}</h1>;
}
```

<details>
<summary>Ver solución</summary>

```jsx
function ContadorAutomatico() {
  const [count, setCount] = useState(0);
  
  useEffect(() => {
    const interval = setInterval(() => {
      setCount(prevCount => prevCount + 1);
    }, 1000);
    
    return () => clearInterval(interval);
  }, []);  // Array vacío: solo se crea un interval
  
  return <h1>{count}</h1>;
}
```
</details>

---

## Ejercicio 2: Título dinámico

Actualiza el título de la página solo cuando `nombre` cambia, no cuando `edad` cambia:

```jsx
function Formulario() {
  const [nombre, setNombre] = useState('');
  const [edad, setEdad] = useState(0);
  
  // Tu código aquí: useEffect que actualice el título con el nombre
  
  return (
    <div>
      <input 
        value={nombre}
        onChange={(e) => setNombre(e.target.value)}
        placeholder="Nombre"
      />
      <input 
        type="number"
        value={edad}
        onChange={(e) => setEdad(Number(e.target.value))}
        placeholder="Edad"
      />
    </div>
  );
}
```

<details>
<summary>Ver solución</summary>

```jsx
function Formulario() {
  const [nombre, setNombre] = useState('');
  const [edad, setEdad] = useState(0);
  
  useEffect(() => {
    document.title = nombre ? `Perfil - ${nombre}` : 'Perfil';
  }, [nombre]);  // Solo depende de nombre
  
  return (
    <div>
      <input 
        value={nombre}
        onChange={(e) => setNombre(e.target.value)}
        placeholder="Nombre"
      />
      <input 
        type="number"
        value={edad}
        onChange={(e) => setEdad(Number(e.target.value))}
        placeholder="Edad"
      />
    </div>
  );
}
```
</details>

---

## Ejercicio 3: Log solo al montar

Crea un componente que muestre un mensaje en consola SOLO cuando se monta (primer render):

```jsx
function Componente() {
  const [count, setCount] = useState(0);
  
  // Tu código aquí: log solo al montar
  
  return (
    <button onClick={() => setCount(count + 1)}>
      Clicks: {count}
    </button>
  );
}
```

<details>
<summary>Ver solución</summary>

```jsx
function Componente() {
  const [count, setCount] = useState(0);
  
  useEffect(() => {
    console.log('Componente montado');
  }, []);  // Array vacío: solo al montar
  
  return (
    <button onClick={() => setCount(count + 1)}>
      Clicks: {count}
    </button>
  );
}
```
</details>

---

## Ejercicio 4: Fetch con dependencia

Crea un componente que cargue información de un usuario según su ID:

```jsx
function PerfilUsuario({ userId }) {
  const [usuario, setUsuario] = useState(null);
  
  // Tu código aquí: fetch cuando userId cambie
  
  return usuario ? <div>{usuario.nombre}</div> : <div>Cargando...</div>;
}
```

<details>
<summary>Ver solución</summary>

```jsx
function PerfilUsuario({ userId }) {
  const [usuario, setUsuario] = useState(null);
  const [loading, setLoading] = useState(true);
  
  useEffect(() => {
    setLoading(true);
    
    fetch(`/api/usuarios/${userId}`)
      .then(res => res.json())
      .then(data => {
        setUsuario(data);
        setLoading(false);
      })
      .catch(error => {
        console.error('Error:', error);
        setLoading(false);
      });
  }, [userId]);  // Se ejecuta cuando userId cambia
  
  if (loading) return <div>Cargando...</div>;
  if (!usuario) return <div>Usuario no encontrado</div>;
  
  return (
    <div>
      <h2>{usuario.nombre}</h2>
      <p>{usuario.email}</p>
    </div>
  );
}
```
</details>

---

## Resumen

✅ **Array vacío `[]`** → Ejecuta solo al montar  
✅ **Con dependencias `[a, b]`** → Ejecuta cuando `a` o `b` cambian  
✅ **Sin array** → Ejecuta en cada render  
✅ **Incluye TODAS las dependencias** que uses en el efecto  
✅ **Objetos/arrays** se comparan por referencia, no por valor  
✅ **Props y estado** deben estar en dependencias si se usan  
✅ **ESLint** ayuda a detectar dependencias faltantes

---

## Siguiente paso

Ya sabes controlar cuándo se ejecuta useEffect. Ahora aprenderás a **hacer fetching de datos** de forma correcta.

**Siguiente:** [Módulo 18 - Fetching de datos con useEffect](./18-fetching-datos.md)

**Anterior:** [Módulo 16 - Introducción a useEffect](./16-intro-useeffect.md)

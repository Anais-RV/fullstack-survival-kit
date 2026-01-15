# Módulo 10: Listas y Keys en React

## Renderizar arrays

En React, usas `.map()` para convertir un array en elementos JSX:

```jsx
function ListaNombres() {
  const nombres = ['Ana', 'Luis', 'María', 'Carlos'];
  
  return (
    <ul>
      {nombres.map((nombre, index) => (
        <li key={index}>{nombre}</li>
      ))}
    </ul>
  );
}
```

---

## La prop `key`

### ¿Por qué es obligatoria?

React usa `key` para **identificar cada elemento** y saber qué cambió, se agregó o eliminó.

```jsx
// ❌ Warning en consola
{items.map(item => <li>{item}</li>)}

// ✅ Con key
{items.map(item => <li key={item.id}>{item}</li>)}
```

### ¿Qué valor usar como key?

**✅ ID único del backend:**
```jsx
const usuarios = [
  { id: 1, nombre: "Ana" },
  { id: 2, nombre: "Luis" }
];

{usuarios.map(user => (
  <li key={user.id}>{user.nombre}</li>
))}
```

**✅ ID generado (si no hay backend):**
```jsx
import { v4 as uuidv4 } from 'uuid';

const tareas = [
  { id: uuidv4(), texto: "Aprender React" },
  { id: uuidv4(), texto: "Hacer ejercicio" }
];
```

**⚠️ Índice del array (solo si el orden nunca cambia):**
```jsx
const items = ['Inicio', 'Acerca', 'Contacto'];

{items.map((item, index) => (
  <li key={index}>{item}</li>
))}
```

**❌ NUNCA uses valores aleatorios:**
```jsx
// ❌ Genera nueva key en cada render
{items.map(item => (
  <li key={Math.random()}>{item}</li>
))}
```

---

## Problemas sin key o con key incorrecta

### Ejemplo del problema

```jsx
function TodoList() {
  const [tareas, setTareas] = useState([
    "Aprender React",
    "Hacer ejercicio",
    "Leer"
  ]);
  
  const eliminar = (index) => {
    setTareas(tareas.filter((_, i) => i !== index));
  };
  
  return (
    <ul>
      {tareas.map((tarea, index) => (
        // ❌ Usando index como key
        <li key={index}>
          <input type="checkbox" />
          {tarea}
          <button onClick={() => eliminar(index)}>X</button>
        </li>
      ))}
    </ul>
  );
}
```

**Problema:** Si marcas el checkbox del segundo item y luego eliminas el primero, el checkbox se mantiene en la misma posición (ahora otro item).

**Solución:** Usa ID único:

```jsx
const [tareas, setTareas] = useState([
  { id: 1, texto: "Aprender React", completada: false },
  { id: 2, texto: "Hacer ejercicio", completada: false },
  { id: 3, texto: "Leer", completada: false }
]);

{tareas.map((tarea) => (
  <li key={tarea.id}>
    <input 
      type="checkbox" 
      checked={tarea.completada}
      onChange={() => toggleTarea(tarea.id)}
    />
    {tarea.texto}
    <button onClick={() => eliminar(tarea.id)}>X</button>
  </li>
))}
```

---

## Renderizar objetos

```jsx
function UserList() {
  const usuarios = [
    { id: 1, nombre: "Ana", rol: "Admin", activo: true },
    { id: 2, nombre: "Luis", rol: "Usuario", activo: false },
    { id: 3, nombre: "María", rol: "Usuario", activo: true }
  ];
  
  return (
    <ul>
      {usuarios.map(user => (
        <li key={user.id}>
          <strong>{user.nombre}</strong> - {user.rol}
          {user.activo && <span> 🟢 Activo</span>}
        </li>
      ))}
    </ul>
  );
}
```

---

## Componentes de lista

### Extraer a componente

```jsx
function ProductList({ productos }) {
  return (
    <div className="product-list">
      {productos.map(producto => (
        <ProductCard key={producto.id} producto={producto} />
      ))}
    </div>
  );
}

function ProductCard({ producto }) {
  return (
    <div className="card">
      <img src={producto.imagen} alt={producto.nombre} />
      <h3>{producto.nombre}</h3>
      <p>${producto.precio}</p>
      <button>Agregar al carrito</button>
    </div>
  );
}
```

---

## Filtrar listas

```jsx
function TaskList() {
  const [filtro, setFiltro] = useState('todas');
  const [tareas] = useState([
    { id: 1, texto: "Aprender React", completada: true },
    { id: 2, texto: "Hacer ejercicio", completada: false },
    { id: 3, texto: "Leer", completada: false }
  ]);
  
  const tareasFiltradas = tareas.filter(tarea => {
    if (filtro === 'completadas') return tarea.completada;
    if (filtro === 'pendientes') return !tarea.completada;
    return true;  // 'todas'
  });
  
  return (
    <div>
      <div>
        <button onClick={() => setFiltro('todas')}>Todas</button>
        <button onClick={() => setFiltro('pendientes')}>Pendientes</button>
        <button onClick={() => setFiltro('completadas')}>Completadas</button>
      </div>
      
      <ul>
        {tareasFiltradas.map(tarea => (
          <li key={tarea.id}>
            {tarea.texto}
            {tarea.completada && ' ✓'}
          </li>
        ))}
      </ul>
    </div>
  );
}
```

---

## Ordenar listas

```jsx
function UserList() {
  const [usuarios, setUsuarios] = useState([
    { id: 1, nombre: "Carlos", edad: 30 },
    { id: 2, nombre: "Ana", edad: 25 },
    { id: 3, nombre: "Beatriz", edad: 35 }
  ]);
  
  const [orden, setOrden] = useState('nombre');
  
  const usuariosOrdenados = [...usuarios].sort((a, b) => {
    if (orden === 'nombre') {
      return a.nombre.localeCompare(b.nombre);
    }
    if (orden === 'edad') {
      return a.edad - b.edad;
    }
    return 0;
  });
  
  return (
    <div>
      <button onClick={() => setOrden('nombre')}>Ordenar por nombre</button>
      <button onClick={() => setOrden('edad')}>Ordenar por edad</button>
      
      <ul>
        {usuariosOrdenados.map(user => (
          <li key={user.id}>
            {user.nombre} ({user.edad} años)
          </li>
        ))}
      </ul>
    </div>
  );
}
```

---

## Buscar en listas

```jsx
function SearchableList() {
  const [busqueda, setBusqueda] = useState('');
  const [items] = useState([
    { id: 1, nombre: "Laptop Dell" },
    { id: 2, nombre: "Mouse Logitech" },
    { id: 3, nombre: "Teclado Mecánico" },
    { id: 4, nombre: "Monitor LG" }
  ]);
  
  const itemsFiltrados = items.filter(item =>
    item.nombre.toLowerCase().includes(busqueda.toLowerCase())
  );
  
  return (
    <div>
      <input
        type="text"
        placeholder="Buscar..."
        value={busqueda}
        onChange={(e) => setBusqueda(e.target.value)}
      />
      
      <ul>
        {itemsFiltrados.map(item => (
          <li key={item.id}>{item.nombre}</li>
        ))}
      </ul>
      
      {itemsFiltrados.length === 0 && (
        <p>No se encontraron resultados</p>
      )}
    </div>
  );
}
```

---

## Listas anidadas

```jsx
function Categories() {
  const categorias = [
    {
      id: 1,
      nombre: "Electrónica",
      productos: [
        { id: 101, nombre: "Laptop" },
        { id: 102, nombre: "Mouse" }
      ]
    },
    {
      id: 2,
      nombre: "Ropa",
      productos: [
        { id: 201, nombre: "Camisa" },
        { id: 202, nombre: "Pantalón" }
      ]
    }
  ];
  
  return (
    <div>
      {categorias.map(categoria => (
        <div key={categoria.id}>
          <h2>{categoria.nombre}</h2>
          <ul>
            {categoria.productos.map(producto => (
              <li key={producto.id}>{producto.nombre}</li>
            ))}
          </ul>
        </div>
      ))}
    </div>
  );
}
```

---

## Lista vacía

```jsx
function TaskList({ tareas }) {
  if (tareas.length === 0) {
    return (
      <div className="empty-state">
        <p>No hay tareas pendientes</p>
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

---

## Paginación

```jsx
function PaginatedList({ items, itemsPorPagina = 10 }) {
  const [paginaActual, setPaginaActual] = useState(1);
  
  const totalPaginas = Math.ceil(items.length / itemsPorPagina);
  const inicio = (paginaActual - 1) * itemsPorPagina;
  const fin = inicio + itemsPorPagina;
  const itemsPagina = items.slice(inicio, fin);
  
  return (
    <div>
      <ul>
        {itemsPagina.map(item => (
          <li key={item.id}>{item.nombre}</li>
        ))}
      </ul>
      
      <div className="pagination">
        <button 
          onClick={() => setPaginaActual(paginaActual - 1)}
          disabled={paginaActual === 1}
        >
          Anterior
        </button>
        
        <span>Página {paginaActual} de {totalPaginas}</span>
        
        <button 
          onClick={() => setPaginaActual(paginaActual + 1)}
          disabled={paginaActual === totalPaginas}
        >
          Siguiente
        </button>
      </div>
    </div>
  );
}
```

---

## Infinite scroll

```jsx
function InfiniteList() {
  const [items, setItems] = useState([]);
  const [pagina, setPagina] = useState(1);
  const [cargando, setCargando] = useState(false);
  
  useEffect(() => {
    setCargando(true);
    fetch(`/api/items?page=${pagina}`)
      .then(res => res.json())
      .then(data => {
        setItems(prev => [...prev, ...data]);
        setCargando(false);
      });
  }, [pagina]);
  
  const handleScroll = (e) => {
    const bottom = e.target.scrollHeight - e.target.scrollTop === e.target.clientHeight;
    if (bottom && !cargando) {
      setPagina(prev => prev + 1);
    }
  };
  
  return (
    <div onScroll={handleScroll} style={{ height: '500px', overflow: 'auto' }}>
      <ul>
        {items.map(item => (
          <li key={item.id}>{item.nombre}</li>
        ))}
      </ul>
      {cargando && <p>Cargando más...</p>}
    </div>
  );
}
```

---

## Drag and drop (reordenar)

Ejemplo conceptual:

```jsx
function ReorderableList() {
  const [items, setItems] = useState([
    { id: 1, texto: "Item 1" },
    { id: 2, texto: "Item 2" },
    { id: 3, texto: "Item 3" }
  ]);
  
  const [arrastrado, setArrastrado] = useState(null);
  
  const handleDragStart = (index) => {
    setArrastrado(index);
  };
  
  const handleDrop = (index) => {
    const newItems = [...items];
    const [item] = newItems.splice(arrastrado, 1);
    newItems.splice(index, 0, item);
    setItems(newItems);
    setArrastrado(null);
  };
  
  return (
    <ul>
      {items.map((item, index) => (
        <li
          key={item.id}
          draggable
          onDragStart={() => handleDragStart(index)}
          onDragOver={(e) => e.preventDefault()}
          onDrop={() => handleDrop(index)}
        >
          {item.texto}
        </li>
      ))}
    </ul>
  );
}
```

---

## Performance en listas grandes

### Virtualización (react-window)

```jsx
import { FixedSizeList } from 'react-window';

function VirtualizedList({ items }) {
  const Row = ({ index, style }) => (
    <div style={style}>
      {items[index].nombre}
    </div>
  );
  
  return (
    <FixedSizeList
      height={500}
      itemCount={items.length}
      itemSize={50}
      width="100%"
    >
      {Row}
    </FixedSizeList>
  );
}
```

---

## Ejemplo completo: Todo List

```jsx
import { useState } from 'react';

function TodoApp() {
  const [tareas, setTareas] = useState([
    { id: 1, texto: "Aprender React", completada: false },
    { id: 2, texto: "Hacer ejercicio", completada: true }
  ]);
  const [input, setInput] = useState('');
  const [filtro, setFiltro] = useState('todas');
  
  const agregarTarea = (e) => {
    e.preventDefault();
    if (input.trim()) {
      setTareas([
        ...tareas,
        { id: Date.now(), texto: input, completada: false }
      ]);
      setInput('');
    }
  };
  
  const toggleTarea = (id) => {
    setTareas(tareas.map(tarea =>
      tarea.id === id ? { ...tarea, completada: !tarea.completada } : tarea
    ));
  };
  
  const eliminarTarea = (id) => {
    setTareas(tareas.filter(tarea => tarea.id !== id));
  };
  
  const tareasFiltradas = tareas.filter(tarea => {
    if (filtro === 'completadas') return tarea.completada;
    if (filtro === 'pendientes') return !tarea.completada;
    return true;
  });
  
  return (
    <div className="todo-app">
      <h1>Lista de Tareas</h1>
      
      <form onSubmit={agregarTarea}>
        <input
          type="text"
          value={input}
          onChange={(e) => setInput(e.target.value)}
          placeholder="Nueva tarea..."
        />
        <button type="submit">Agregar</button>
      </form>
      
      <div className="filtros">
        <button onClick={() => setFiltro('todas')}>Todas</button>
        <button onClick={() => setFiltro('pendientes')}>Pendientes</button>
        <button onClick={() => setFiltro('completadas')}>Completadas</button>
      </div>
      
      {tareasFiltradas.length === 0 ? (
        <p>No hay tareas en este filtro</p>
      ) : (
        <ul>
          {tareasFiltradas.map(tarea => (
            <li key={tarea.id} className={tarea.completada ? 'completada' : ''}>
              <input
                type="checkbox"
                checked={tarea.completada}
                onChange={() => toggleTarea(tarea.id)}
              />
              <span>{tarea.texto}</span>
              <button onClick={() => eliminarTarea(tarea.id)}>Eliminar</button>
            </li>
          ))}
        </ul>
      )}
      
      <p>{tareas.filter(t => !t.completada).length} tareas pendientes</p>
    </div>
  );
}
```

---

## Buenas prácticas

✅ **Siempre usa `key`**  
✅ **Usa ID único, no `index` si el orden cambia**  
✅ **Extrae items a componentes separados**  
✅ **Filtra/ordena antes del map**  
✅ **Maneja estado vacío**  
✅ **Virtualiza listas muy grandes (> 1000 items)**  

---

## Errores comunes

### 1. Olvidar key

```jsx
// ❌ Warning
{items.map(item => <li>{item}</li>)}

// ✅ Con key
{items.map(item => <li key={item.id}>{item}</li>)}
```

### 2. Key no única

```jsx
// ❌ Todas tienen la misma key
{items.map(item => <li key="item">{item}</li>)}

// ✅ Key única
{items.map(item => <li key={item.id}>{item}</li>)}
```

### 3. Key que cambia en cada render

```jsx
// ❌ Nueva key cada vez
{items.map(item => <li key={Math.random()}>{item}</li>)}

// ✅ Key estable
{items.map(item => <li key={item.id}>{item}</li>)}
```

### 4. Mutar array directamente

```jsx
// ❌ Muta el estado
items.push(nuevoItem);
setItems(items);

// ✅ Crea nuevo array
setItems([...items, nuevoItem]);
```

---

## Resumen

✅ **`.map()` para renderizar arrays**  
✅ **`key` es obligatoria y única**  
✅ **Usa ID del backend, no `index`**  
✅ **Filtra/ordena/busca antes del map**  
✅ **Maneja listas vacías**  
✅ **Extrae items a componentes**  
✅ **Virtualiza listas grandes**

---

## Siguiente paso

¡Completaste el **Bloque 2: Componentes**! Ahora entiendes:
- ✅ Anatomía de componentes
- ✅ Props
- ✅ Composición
- ✅ Renderizado condicional
- ✅ Listas y keys

**Siguiente:** [Módulo 11 - ¿Qué es el estado?](./11-que-es-estado.md) → Aprenderás sobre **estado** y **hooks**.

**Anterior:** [Módulo 09 - Renderizado condicional](./09-condicional-rendering.md)

# Módulo 15: Levantar Estado (Lifting State Up)

## El problema: Componentes hermanos

Imagina que tienes dos componentes que necesitan **compartir información**:

```jsx
function App() {
  return (
    <div>
      <ComponenteA />  {/* Necesita saber el count */}
      <ComponenteB />  {/* Necesita saber el count */}
    </div>
  );
}
```

**Problema:** Si cada componente tiene su propio estado, **no pueden comunicarse**.

---

## La solución: Levantar el estado

**"Lifting State Up"** = Mover el estado al **componente padre común** más cercano.

### ❌ Estados separados (no se comunican)
```jsx
function ComponenteA() {
  const [count, setCount] = useState(0);
  return <div>{count}</div>;
}

function ComponenteB() {
  const [count, setCount] = useState(0);  // Estado diferente
  return <div>{count}</div>;
}
```

### ✅ Estado compartido (se comunican)
```jsx
function App() {
  const [count, setCount] = useState(0);  // Estado en el padre
  
  return (
    <div>
      <ComponenteA count={count} />
      <ComponenteB count={count} />
      <button onClick={() => setCount(count + 1)}>+</button>
    </div>
  );
}

function ComponenteA({ count }) {
  return <div>A: {count}</div>;
}

function ComponenteB({ count }) {
  return <div>B: {count}</div>;
}
```

---

## Flujo de datos en React

```
         Padre
          |
    [Estado compartido]
          |
    ------+------
    |           |
ComponenteA  ComponenteB
```

- **Estado vive en el padre**
- **Datos fluyen hacia abajo** (props)
- **Hijos no pueden modificar el estado directamente**

---

## Pasar callbacks para modificar estado

Los hijos pueden **modificar el estado del padre** a través de **callbacks**:

```jsx
function App() {
  const [count, setCount] = useState(0);
  
  const incrementar = () => {
    setCount(count + 1);
  };
  
  return (
    <div>
      <Display count={count} />
      <Boton onIncrement={incrementar} />  {/* Pasa callback */}
    </div>
  );
}

function Display({ count }) {
  return <h1>{count}</h1>;
}

function Boton({ onIncrement }) {
  return <button onClick={onIncrement}>+1</button>;
}
```

**Flujo:**
1. Click en `<Boton>`
2. Ejecuta `onIncrement` (que es `incrementar`)
3. `setCount` actualiza el estado en `App`
4. React re-renderiza `App` y sus hijos
5. `<Display>` recibe el nuevo `count`

---

## Ejemplo: Filtro y lista

```jsx
function App() {
  const [productos] = useState([
    { id: 1, nombre: 'Laptop', categoria: 'tech' },
    { id: 2, nombre: 'Camisa', categoria: 'ropa' },
    { id: 3, nombre: 'Mouse', categoria: 'tech' }
  ]);
  
  const [filtro, setFiltro] = useState('');
  
  const productosFiltrados = productos.filter(p => 
    p.nombre.toLowerCase().includes(filtro.toLowerCase())
  );
  
  return (
    <div>
      <Filtro filtro={filtro} onFiltroChange={setFiltro} />
      <ListaProductos productos={productosFiltrados} />
    </div>
  );
}

function Filtro({ filtro, onFiltroChange }) {
  return (
    <input 
      value={filtro}
      onChange={(e) => onFiltroChange(e.target.value)}
      placeholder="Buscar..."
    />
  );
}

function ListaProductos({ productos }) {
  return (
    <ul>
      {productos.map(p => (
        <li key={p.id}>{p.nombre}</li>
      ))}
    </ul>
  );
}
```

---

## Ejemplo: Tabs (pestañas)

```jsx
function App() {
  const [tabActiva, setTabActiva] = useState('home');
  
  return (
    <div>
      <Tabs activa={tabActiva} onChange={setTabActiva} />
      <Contenido tab={tabActiva} />
    </div>
  );
}

function Tabs({ activa, onChange }) {
  const tabs = ['home', 'perfil', 'configuracion'];
  
  return (
    <div className="tabs">
      {tabs.map(tab => (
        <button 
          key={tab}
          onClick={() => onChange(tab)}
          className={activa === tab ? 'activa' : ''}
        >
          {tab}
        </button>
      ))}
    </div>
  );
}

function Contenido({ tab }) {
  return (
    <div>
      {tab === 'home' && <div>Contenido de Home</div>}
      {tab === 'perfil' && <div>Contenido de Perfil</div>}
      {tab === 'configuracion' && <div>Contenido de Configuración</div>}
    </div>
  );
}
```

---

## Ejemplo: Carrito de compras

```jsx
function App() {
  const [carrito, setCarrito] = useState([]);
  
  const agregarAlCarrito = (producto) => {
    setCarrito([...carrito, producto]);
  };
  
  const eliminarDelCarrito = (id) => {
    setCarrito(carrito.filter(item => item.id !== id));
  };
  
  return (
    <div>
      <CatalogoProductos onAgregar={agregarAlCarrito} />
      <Carrito items={carrito} onEliminar={eliminarDelCarrito} />
    </div>
  );
}

function CatalogoProductos({ onAgregar }) {
  const productos = [
    { id: 1, nombre: 'Laptop', precio: 1000 },
    { id: 2, nombre: 'Mouse', precio: 25 }
  ];
  
  return (
    <div>
      <h2>Productos</h2>
      {productos.map(p => (
        <div key={p.id}>
          <span>{p.nombre} - ${p.precio}</span>
          <button onClick={() => onAgregar(p)}>Agregar</button>
        </div>
      ))}
    </div>
  );
}

function Carrito({ items, onEliminar }) {
  const total = items.reduce((sum, item) => sum + item.precio, 0);
  
  return (
    <div>
      <h2>Carrito</h2>
      {items.map((item, index) => (
        <div key={index}>
          <span>{item.nombre} - ${item.precio}</span>
          <button onClick={() => onEliminar(item.id)}>❌</button>
        </div>
      ))}
      <p>Total: ${total}</p>
    </div>
  );
}
```

---

## Ejemplo: Formulario multi-paso

```jsx
function App() {
  const [paso, setPaso] = useState(1);
  const [datos, setDatos] = useState({
    nombre: '',
    email: '',
    direccion: ''
  });
  
  const siguiente = () => setPaso(paso + 1);
  const anterior = () => setPaso(paso - 1);
  
  const actualizarDatos = (campo, valor) => {
    setDatos({ ...datos, [campo]: valor });
  };
  
  return (
    <div>
      <h2>Paso {paso} de 3</h2>
      
      {paso === 1 && (
        <Paso1 
          datos={datos}
          onChange={actualizarDatos}
          onSiguiente={siguiente}
        />
      )}
      
      {paso === 2 && (
        <Paso2 
          datos={datos}
          onChange={actualizarDatos}
          onAnterior={anterior}
          onSiguiente={siguiente}
        />
      )}
      
      {paso === 3 && (
        <Resumen datos={datos} onAnterior={anterior} />
      )}
    </div>
  );
}

function Paso1({ datos, onChange, onSiguiente }) {
  return (
    <div>
      <input 
        value={datos.nombre}
        onChange={(e) => onChange('nombre', e.target.value)}
        placeholder="Nombre"
      />
      <button onClick={onSiguiente}>Siguiente</button>
    </div>
  );
}

function Paso2({ datos, onChange, onAnterior, onSiguiente }) {
  return (
    <div>
      <input 
        value={datos.email}
        onChange={(e) => onChange('email', e.target.value)}
        placeholder="Email"
      />
      <button onClick={onAnterior}>Anterior</button>
      <button onClick={onSiguiente}>Siguiente</button>
    </div>
  );
}

function Resumen({ datos, onAnterior }) {
  return (
    <div>
      <h3>Resumen</h3>
      <p>Nombre: {datos.nombre}</p>
      <p>Email: {datos.email}</p>
      <button onClick={onAnterior}>Anterior</button>
      <button>Enviar</button>
    </div>
  );
}
```

---

## ¿Dónde colocar el estado?

### Regla general

Coloca el estado en el **ancestro común más cercano** de todos los componentes que lo necesitan.

```
         App
          |
    ------+------
    |           |
  Header      Main
              |
         -----+-----
         |         |
      Sidebar   Content
```

**Si Header y Sidebar necesitan el estado:**
→ Estado en `App`

**Si solo Sidebar y Content necesitan el estado:**
→ Estado en `Main`

---

## Estados múltiples vs estado único

### Opción 1: Estados separados
```jsx
function App() {
  const [nombre, setNombre] = useState('');
  const [email, setEmail] = useState('');
  const [edad, setEdad] = useState(0);
  
  // ...
}
```

**✅ Ventajas:**
- Más simple para estados independientes
- Actualizar uno no afecta a los otros

**❌ Desventajas:**
- Muchas variables si hay muchos campos
- Difícil pasar todo a componentes hijos

---

### Opción 2: Estado agrupado
```jsx
function App() {
  const [form, setForm] = useState({
    nombre: '',
    email: '',
    edad: 0
  });
  
  // ...
}
```

**✅ Ventajas:**
- Fácil pasar todo el objeto
- Organizado cuando hay muchos campos relacionados

**❌ Desventajas:**
- Más complejo actualizar (necesitas spread operator)

---

## Patrón: Componente controlado vs no controlado

### Controlado (estado en el padre)
```jsx
function App() {
  const [valor, setValor] = useState('');
  
  return <Input value={valor} onChange={setValor} />;
}

function Input({ value, onChange }) {
  return (
    <input 
      value={value}
      onChange={(e) => onChange(e.target.value)}
    />
  );
}
```

**✅ Ventajas:**
- El padre controla completamente el valor
- Más fácil de testear y debuguear
- Validaciones centralizadas

---

### No controlado (estado en el hijo)
```jsx
function Input() {
  const [value, setValue] = useState('');
  
  return (
    <input 
      value={value}
      onChange={(e) => setValue(e.target.value)}
    />
  );
}
```

**✅ Ventajas:**
- Más simple si el padre no necesita el valor
- Menos re-renders del padre

---

## Ejemplo completo: Todo App con múltiples componentes

```jsx
function TodoApp() {
  const [todos, setTodos] = useState([]);
  const [filtro, setFiltro] = useState('todos');  // 'todos', 'activas', 'completadas'
  
  const agregarTodo = (texto) => {
    const nuevo = {
      id: Date.now(),
      texto,
      completada: false
    };
    setTodos([...todos, nuevo]);
  };
  
  const toggleTodo = (id) => {
    setTodos(todos.map(todo =>
      todo.id === id ? { ...todo, completada: !todo.completada } : todo
    ));
  };
  
  const eliminarTodo = (id) => {
    setTodos(todos.filter(todo => todo.id !== id));
  };
  
  const todosFiltradas = todos.filter(todo => {
    if (filtro === 'activas') return !todo.completada;
    if (filtro === 'completadas') return todo.completada;
    return true;
  });
  
  return (
    <div>
      <h1>Todo App</h1>
      <TodoForm onAgregar={agregarTodo} />
      <FiltroTodos filtro={filtro} onCambiar={setFiltro} />
      <TodoList 
        todos={todosFiltradas}
        onToggle={toggleTodo}
        onEliminar={eliminarTodo}
      />
      <Estadisticas todos={todos} />
    </div>
  );
}

function TodoForm({ onAgregar }) {
  const [input, setInput] = useState('');
  
  const handleSubmit = (e) => {
    e.preventDefault();
    if (input.trim()) {
      onAgregar(input);
      setInput('');
    }
  };
  
  return (
    <form onSubmit={handleSubmit}>
      <input 
        value={input}
        onChange={(e) => setInput(e.target.value)}
        placeholder="Nueva tarea..."
      />
      <button type="submit">Agregar</button>
    </form>
  );
}

function FiltroTodos({ filtro, onCambiar }) {
  return (
    <div>
      <button 
        onClick={() => onCambiar('todos')}
        className={filtro === 'todos' ? 'activo' : ''}
      >
        Todas
      </button>
      <button 
        onClick={() => onCambiar('activas')}
        className={filtro === 'activas' ? 'activo' : ''}
      >
        Activas
      </button>
      <button 
        onClick={() => onCambiar('completadas')}
        className={filtro === 'completadas' ? 'activo' : ''}
      >
        Completadas
      </button>
    </div>
  );
}

function TodoList({ todos, onToggle, onEliminar }) {
  return (
    <ul>
      {todos.map(todo => (
        <TodoItem 
          key={todo.id}
          todo={todo}
          onToggle={onToggle}
          onEliminar={onEliminar}
        />
      ))}
    </ul>
  );
}

function TodoItem({ todo, onToggle, onEliminar }) {
  return (
    <li>
      <input 
        type="checkbox"
        checked={todo.completada}
        onChange={() => onToggle(todo.id)}
      />
      <span style={{
        textDecoration: todo.completada ? 'line-through' : 'none'
      }}>
        {todo.texto}
      </span>
      <button onClick={() => onEliminar(todo.id)}>❌</button>
    </li>
  );
}

function Estadisticas({ todos }) {
  const total = todos.length;
  const completadas = todos.filter(t => t.completada).length;
  const activas = total - completadas;
  
  return (
    <div>
      <p>Total: {total}</p>
      <p>Activas: {activas}</p>
      <p>Completadas: {completadas}</p>
    </div>
  );
}
```

---

## Ejercicio 1: Temperatura en Celsius y Fahrenheit

Crea una app con dos inputs sincronizados:
- Celsius → Fahrenheit
- Fahrenheit → Celsius

Cuando cambias uno, el otro se actualiza automáticamente.

```jsx
function App() {
  // Estado compartido
  
  return (
    <div>
      <InputTemperatura escala="c" />
      <InputTemperatura escala="f" />
    </div>
  );
}
```

<details>
<summary>Ver solución</summary>

```jsx
function App() {
  const [celsius, setCelsius] = useState(0);
  
  const fahrenheit = celsius * 9/5 + 32;
  
  const handleCelsiusChange = (valor) => {
    setCelsius(valor);
  };
  
  const handleFahrenheitChange = (valor) => {
    setCelsius((valor - 32) * 5/9);
  };
  
  return (
    <div>
      <InputTemperatura 
        escala="Celsius"
        temperatura={celsius}
        onChange={handleCelsiusChange}
      />
      <InputTemperatura 
        escala="Fahrenheit"
        temperatura={fahrenheit}
        onChange={handleFahrenheitChange}
      />
    </div>
  );
}

function InputTemperatura({ escala, temperatura, onChange }) {
  return (
    <div>
      <label>{escala}:</label>
      <input 
        type="number"
        value={temperatura}
        onChange={(e) => onChange(Number(e.target.value))}
      />
    </div>
  );
}
```
</details>

---

## Ejercicio 2: Buscador con resultados

Crea un buscador donde:
- `<Buscador>` tiene el input de búsqueda
- `<Resultados>` muestra los items filtrados

```jsx
function App() {
  const items = ['Manzana', 'Banana', 'Cereza', 'Durazno', 'Fresa'];
  
  // Tu código aquí
}
```

<details>
<summary>Ver solución</summary>

```jsx
function App() {
  const items = ['Manzana', 'Banana', 'Cereza', 'Durazno', 'Fresa'];
  const [busqueda, setBusqueda] = useState('');
  
  const itemsFiltrados = items.filter(item =>
    item.toLowerCase().includes(busqueda.toLowerCase())
  );
  
  return (
    <div>
      <Buscador busqueda={busqueda} onChange={setBusqueda} />
      <Resultados items={itemsFiltrados} />
    </div>
  );
}

function Buscador({ busqueda, onChange }) {
  return (
    <input 
      value={busqueda}
      onChange={(e) => onChange(e.target.value)}
      placeholder="Buscar..."
    />
  );
}

function Resultados({ items }) {
  return (
    <ul>
      {items.map((item, index) => (
        <li key={index}>{item}</li>
      ))}
    </ul>
  );
}
```
</details>

---

## Ejercicio 3: Selector de color

Crea un selector de color con:
- Tres sliders (R, G, B)
- Un div que muestre el color resultante
- El código RGB

```jsx
function App() {
  // Tu código aquí
}
```

<details>
<summary>Ver solución</summary>

```jsx
function App() {
  const [r, setR] = useState(255);
  const [g, setG] = useState(0);
  const [b, setB] = useState(0);
  
  const color = `rgb(${r}, ${g}, ${b})`;
  
  return (
    <div>
      <Slider label="Rojo" valor={r} onChange={setR} />
      <Slider label="Verde" valor={g} onChange={setG} />
      <Slider label="Azul" valor={b} onChange={setB} />
      
      <div 
        style={{ 
          width: 200, 
          height: 200, 
          backgroundColor: color 
        }}
      />
      
      <p>{color}</p>
    </div>
  );
}

function Slider({ label, valor, onChange }) {
  return (
    <div>
      <label>{label}: {valor}</label>
      <input 
        type="range"
        min="0"
        max="255"
        value={valor}
        onChange={(e) => onChange(Number(e.target.value))}
      />
    </div>
  );
}
```
</details>

---

## Cuándo NO levantar estado

A veces **no necesitas** levantar el estado:

### ❌ No levantes si:
- Solo un componente necesita el estado
- Los componentes no necesitan comunicarse
- El estado es puramente visual (hover, focus)

### Ejemplo: Accordion
```jsx
function Accordion({ titulo, children }) {
  const [isOpen, setIsOpen] = useState(false);  // Estado local
  
  return (
    <div>
      <button onClick={() => setIsOpen(!isOpen)}>
        {titulo}
      </button>
      {isOpen && <div>{children}</div>}
    </div>
  );
}

// No necesitas levantar isOpen al padre
// Cada Accordion es independiente
```

---

## Resumen

✅ **Lifting State Up** = mover estado al padre común  
✅ **Estado en el padre**, datos fluyen hacia abajo (props)  
✅ **Callbacks** permiten a los hijos modificar el estado del padre  
✅ **Coloca el estado en el ancestro común más cercano**  
✅ **Componentes controlados**: el padre controla el valor  
✅ **Componentes no controlados**: el hijo controla su propio valor  
✅ **No levantes estado** si solo un componente lo necesita

---

## Siguiente paso

Ya sabes cómo compartir estado entre componentes. En el próximo bloque aprenderás sobre **efectos secundarios** con `useEffect`.

**Siguiente:** [Módulo 16 - Introducción a useEffect](../bloque-04-efectos/16-intro-useeffect.md)

**Anterior:** [Módulo 14 - Estado con objetos y arrays](./14-estado-objetos-arrays.md)

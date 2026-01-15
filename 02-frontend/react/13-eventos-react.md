# Módulo 13: Eventos en React

## Eventos en React

Los eventos en React funcionan similar a HTML, pero con algunas diferencias importantes:

### HTML tradicional
```html
<button onclick="handleClick()">Click</button>
```

### React
```jsx
<button onClick={handleClick}>Click</button>
```

---

## Diferencias clave

| HTML | React |
|------|-------|
| `onclick` (minúsculas) | `onClick` (camelCase) |
| `"handleClick()"` (string) | `{handleClick}` (función) |
| `return false` para prevenir | `e.preventDefault()` |

---

## onClick - Click del mouse

```jsx
function Boton() {
  const handleClick = () => {
    console.log('Click!');
  };
  
  return <button onClick={handleClick}>Hazme click</button>;
}
```

**⚠️ Importante:**
```jsx
// ✅ Correcto: pasa la referencia
<button onClick={handleClick}>Click</button>

// ❌ Incorrecto: ejecuta inmediatamente
<button onClick={handleClick()}>Click</button>

// ✅ Si necesitas pasar argumentos, usa arrow function
<button onClick={() => handleClick('hola')}>Click</button>
```

---

## onClick con estado

```jsx
function Contador() {
  const [count, setCount] = useState(0);
  
  const incrementar = () => {
    setCount(count + 1);
  };
  
  return (
    <div>
      <p>Contador: {count}</p>
      <button onClick={incrementar}>+1</button>
    </div>
  );
}
```

---

## onClick inline

Para funciones simples, puedes definirlas inline:

```jsx
function Toggle() {
  const [activo, setActivo] = useState(false);
  
  return (
    <button onClick={() => setActivo(!activo)}>
      {activo ? 'ON' : 'OFF'}
    </button>
  );
}
```

---

## onChange - Cambios en inputs

```jsx
function InputControlado() {
  const [texto, setTexto] = useState('');
  
  const handleChange = (e) => {
    setTexto(e.target.value);
  };
  
  return (
    <div>
      <input value={texto} onChange={handleChange} />
      <p>Escribiste: {texto}</p>
    </div>
  );
}
```

**Forma corta:**
```jsx
<input 
  value={texto}
  onChange={(e) => setTexto(e.target.value)}
/>
```

---

## onSubmit - Envío de formularios

```jsx
function Formulario() {
  const [nombre, setNombre] = useState('');
  
  const handleSubmit = (e) => {
    e.preventDefault();  // ⚠️ Importante: evita recargar la página
    console.log('Nombre enviado:', nombre);
  };
  
  return (
    <form onSubmit={handleSubmit}>
      <input 
        value={nombre}
        onChange={(e) => setNombre(e.target.value)}
      />
      <button type="submit">Enviar</button>
    </form>
  );
}
```

**⚠️ Siempre usa `e.preventDefault()`** en `onSubmit` para evitar que la página se recargue.

---

## Evento sintético (SyntheticEvent)

React envuelve los eventos nativos del navegador en **eventos sintéticos**:

```jsx
function Ejemplo() {
  const handleClick = (evento) => {
    console.log(evento);              // SyntheticEvent
    console.log(evento.nativeEvent);  // Evento nativo del navegador
    console.log(evento.target);       // Elemento que disparó el evento
    console.log(evento.type);         // 'click', 'change', etc.
  };
  
  return <button onClick={handleClick}>Click</button>;
}
```

---

## e.target vs e.currentTarget

```jsx
function Ejemplo() {
  const handleClick = (e) => {
    console.log(e.target);        // Elemento que recibió el click
    console.log(e.currentTarget); // Elemento que tiene el listener
  };
  
  return (
    <div onClick={handleClick}>
      <button>Click me</button>
    </div>
  );
}
```

Si haces click en el `<button>`:
- `e.target` = `<button>`
- `e.currentTarget` = `<div>`

---

## Pasar argumentos a event handlers

### ❌ Incorrecto
```jsx
<button onClick={handleClick(id)}>Click</button>  // Se ejecuta inmediatamente
```

### ✅ Correcto
```jsx
// Opción 1: Arrow function
<button onClick={() => handleClick(id)}>Click</button>

// Opción 2: bind
<button onClick={handleClick.bind(null, id)}>Click</button>
```

**Ejemplo completo:**
```jsx
function ListaTareas() {
  const [tareas, setTareas] = useState(['Tarea 1', 'Tarea 2']);
  
  const eliminar = (index) => {
    const nuevas = tareas.filter((_, i) => i !== index);
    setTareas(nuevas);
  };
  
  return (
    <ul>
      {tareas.map((tarea, index) => (
        <li key={index}>
          {tarea}
          <button onClick={() => eliminar(index)}>❌</button>
        </li>
      ))}
    </ul>
  );
}
```

---

## Pasar evento Y argumentos

```jsx
function Ejemplo() {
  const handleClick = (id, e) => {
    console.log('ID:', id);
    console.log('Evento:', e);
  };
  
  return (
    <button onClick={(e) => handleClick(123, e)}>
      Click
    </button>
  );
}
```

---

## Eventos de teclado

### onKeyDown, onKeyUp, onKeyPress

```jsx
function InputTeclado() {
  const [texto, setTexto] = useState('');
  
  const handleKeyDown = (e) => {
    console.log('Tecla presionada:', e.key);
    
    if (e.key === 'Enter') {
      console.log('Enter presionado!');
    }
    
    if (e.key === 'Escape') {
      setTexto('');
    }
  };
  
  return (
    <input 
      value={texto}
      onChange={(e) => setTexto(e.target.value)}
      onKeyDown={handleKeyDown}
    />
  );
}
```

**Propiedades útiles:**
- `e.key` → La tecla presionada ('a', 'Enter', 'Escape')
- `e.code` → Código físico ('KeyA', 'Enter', 'Space')
- `e.shiftKey` → true si Shift está presionado
- `e.ctrlKey` → true si Ctrl está presionado
- `e.altKey` → true si Alt está presionado

---

## Ejemplo: Enter para enviar

```jsx
function ChatInput() {
  const [mensaje, setMensaje] = useState('');
  const [mensajes, setMensajes] = useState([]);
  
  const enviar = () => {
    if (mensaje.trim()) {
      setMensajes([...mensajes, mensaje]);
      setMensaje('');
    }
  };
  
  const handleKeyDown = (e) => {
    if (e.key === 'Enter' && !e.shiftKey) {
      e.preventDefault();
      enviar();
    }
  };
  
  return (
    <div>
      <ul>
        {mensajes.map((msg, i) => <li key={i}>{msg}</li>)}
      </ul>
      
      <input 
        value={mensaje}
        onChange={(e) => setMensaje(e.target.value)}
        onKeyDown={handleKeyDown}
        placeholder="Escribe y presiona Enter..."
      />
      <button onClick={enviar}>Enviar</button>
    </div>
  );
}
```

---

## onFocus y onBlur

```jsx
function InputConFoco() {
  const [enfocado, setEnfocado] = useState(false);
  
  return (
    <input 
      onFocus={() => setEnfocado(true)}
      onBlur={() => setEnfocado(false)}
      style={{
        border: enfocado ? '2px solid blue' : '1px solid gray'
      }}
    />
  );
}
```

---

## onMouseEnter y onMouseLeave

```jsx
function HoverCard() {
  const [hover, setHover] = useState(false);
  
  return (
    <div 
      onMouseEnter={() => setHover(true)}
      onMouseLeave={() => setHover(false)}
      style={{
        padding: 20,
        background: hover ? 'lightblue' : 'white',
        border: '1px solid gray'
      }}
    >
      {hover ? 'Mouse dentro' : 'Mouse fuera'}
    </div>
  );
}
```

---

## onDoubleClick

```jsx
function DobleClick() {
  const [count, setCount] = useState(0);
  
  return (
    <div 
      onDoubleClick={() => setCount(count + 1)}
      style={{ padding: 20, cursor: 'pointer' }}
    >
      Doble click aquí. Count: {count}
    </div>
  );
}
```

---

## onChange con diferentes inputs

### Input de texto
```jsx
const [texto, setTexto] = useState('');

<input 
  type="text"
  value={texto}
  onChange={(e) => setTexto(e.target.value)}
/>
```

### Checkbox
```jsx
const [aceptado, setAceptado] = useState(false);

<input 
  type="checkbox"
  checked={aceptado}
  onChange={(e) => setAceptado(e.target.checked)}
/>
```

### Radio buttons
```jsx
const [opcion, setOpcion] = useState('a');

<label>
  <input 
    type="radio"
    value="a"
    checked={opcion === 'a'}
    onChange={(e) => setOpcion(e.target.value)}
  />
  Opción A
</label>
```

### Select
```jsx
const [seleccion, setSeleccion] = useState('opcion1');

<select value={seleccion} onChange={(e) => setSeleccion(e.target.value)}>
  <option value="opcion1">Opción 1</option>
  <option value="opcion2">Opción 2</option>
</select>
```

### Textarea
```jsx
const [texto, setTexto] = useState('');

<textarea 
  value={texto}
  onChange={(e) => setTexto(e.target.value)}
/>
```

---

## Ejemplo: Formulario completo

```jsx
function FormularioCompleto() {
  const [form, setForm] = useState({
    nombre: '',
    email: '',
    edad: '',
    genero: '',
    aceptaTerminos: false
  });
  
  const handleChange = (e) => {
    const { name, value, type, checked } = e.target;
    
    setForm({
      ...form,
      [name]: type === 'checkbox' ? checked : value
    });
  };
  
  const handleSubmit = (e) => {
    e.preventDefault();
    console.log('Formulario enviado:', form);
  };
  
  return (
    <form onSubmit={handleSubmit}>
      <div>
        <label>Nombre:</label>
        <input 
          name="nombre"
          value={form.nombre}
          onChange={handleChange}
        />
      </div>
      
      <div>
        <label>Email:</label>
        <input 
          name="email"
          type="email"
          value={form.email}
          onChange={handleChange}
        />
      </div>
      
      <div>
        <label>Edad:</label>
        <input 
          name="edad"
          type="number"
          value={form.edad}
          onChange={handleChange}
        />
      </div>
      
      <div>
        <label>Género:</label>
        <select name="genero" value={form.genero} onChange={handleChange}>
          <option value="">Selecciona...</option>
          <option value="masculino">Masculino</option>
          <option value="femenino">Femenino</option>
          <option value="otro">Otro</option>
        </select>
      </div>
      
      <div>
        <label>
          <input 
            name="aceptaTerminos"
            type="checkbox"
            checked={form.aceptaTerminos}
            onChange={handleChange}
          />
          Acepto los términos
        </label>
      </div>
      
      <button type="submit" disabled={!form.aceptaTerminos}>
        Enviar
      </button>
    </form>
  );
}
```

---

## preventDefault()

Previene el comportamiento por defecto del navegador:

```jsx
// Prevenir envío de formulario
const handleSubmit = (e) => {
  e.preventDefault();
  // Tu código aquí
};

// Prevenir navegación de enlace
const handleClick = (e) => {
  e.preventDefault();
  // Tu código aquí
};

<a href="/pagina" onClick={handleClick}>Link</a>
```

---

## stopPropagation()

Detiene la propagación del evento hacia arriba:

```jsx
function Padre() {
  const handlePadreClick = () => {
    console.log('Click en padre');
  };
  
  const handleHijoClick = (e) => {
    e.stopPropagation();  // No llega al padre
    console.log('Click en hijo');
  };
  
  return (
    <div onClick={handlePadreClick}>
      <button onClick={handleHijoClick}>Hijo</button>
    </div>
  );
}

// Click en botón: Solo muestra "Click en hijo"
// Click en div: Solo muestra "Click en padre"
```

---

## Ejemplo: Botones con confirmación

```jsx
function BotonEliminar({ onEliminar }) {
  const handleClick = () => {
    if (window.confirm('¿Estás seguro?')) {
      onEliminar();
    }
  };
  
  return (
    <button onClick={handleClick} className="btn-danger">
      Eliminar
    </button>
  );
}
```

---

## Ejemplo: Contador con múltiples botones

```jsx
function ContadorMultiple() {
  const [count, setCount] = useState(0);
  
  const cambiar = (cantidad) => {
    setCount(count + cantidad);
  };
  
  return (
    <div>
      <h1>{count}</h1>
      <button onClick={() => cambiar(-10)}>-10</button>
      <button onClick={() => cambiar(-1)}>-1</button>
      <button onClick={() => setCount(0)}>Reset</button>
      <button onClick={() => cambiar(1)}>+1</button>
      <button onClick={() => cambiar(10)}>+10</button>
    </div>
  );
}
```

---

## Ejemplo: Toggle con animación

```jsx
function ToggleAnimado() {
  const [activo, setActivo] = useState(false);
  
  return (
    <div className="toggle-container">
      <button 
        onClick={() => setActivo(!activo)}
        className={`toggle-btn ${activo ? 'activo' : ''}`}
      >
        {activo ? 'ON' : 'OFF'}
      </button>
      
      <div className={`panel ${activo ? 'visible' : 'oculto'}`}>
        <p>Contenido revelado!</p>
      </div>
    </div>
  );
}
```

---

## Ejemplo: Input con debounce manual

```jsx
function BusquedaDebounce() {
  const [texto, setTexto] = useState('');
  const [busqueda, setBusqueda] = useState('');
  
  const handleChange = (e) => {
    const valor = e.target.value;
    setTexto(valor);
    
    // Simula debounce simple
    setTimeout(() => {
      setBusqueda(valor);
    }, 500);
  };
  
  return (
    <div>
      <input value={texto} onChange={handleChange} />
      <p>Buscando: {busqueda}</p>
    </div>
  );
}
```

---

## Ejercicio 1: Toggle button

Crea un botón que cambie entre "Modo Claro" y "Modo Oscuro" y aplique una clase:

```jsx
function ModoTema() {
  // Tu código aquí
  
  return (
    <div className={/* clase según el modo */}>
      <button>{/* "Modo Claro" o "Modo Oscuro" */}</button>
    </div>
  );
}
```

<details>
<summary>Ver solución</summary>

```jsx
function ModoTema() {
  const [modoOscuro, setModoOscuro] = useState(false);
  
  const toggleModo = () => {
    setModoOscuro(!modoOscuro);
  };
  
  return (
    <div className={modoOscuro ? 'dark' : 'light'}>
      <button onClick={toggleModo}>
        {modoOscuro ? 'Modo Claro' : 'Modo Oscuro'}
      </button>
    </div>
  );
}
```
</details>

---

## Ejercicio 2: Contador de clicks

Crea un botón que cuente cuántas veces se ha hecho click:

```jsx
function ContadorClicks() {
  // Tu código aquí
}
```

<details>
<summary>Ver solución</summary>

```jsx
function ContadorClicks() {
  const [clicks, setClicks] = useState(0);
  
  return (
    <div>
      <p>Has hecho click {clicks} veces</p>
      <button onClick={() => setClicks(clicks + 1)}>
        Click aquí
      </button>
      <button onClick={() => setClicks(0)}>Reset</button>
    </div>
  );
}
```
</details>

---

## Ejercicio 3: Input con límite de caracteres

Crea un input que tenga máximo 50 caracteres y muestre el contador:

```jsx
function InputLimitado() {
  // Tu código aquí
}
```

<details>
<summary>Ver solución</summary>

```jsx
function InputLimitado() {
  const [texto, setTexto] = useState('');
  const MAX = 50;
  
  const handleChange = (e) => {
    const valor = e.target.value;
    if (valor.length <= MAX) {
      setTexto(valor);
    }
  };
  
  return (
    <div>
      <textarea 
        value={texto}
        onChange={handleChange}
        placeholder="Máximo 50 caracteres"
      />
      <p>{texto.length}/{MAX}</p>
    </div>
  );
}
```
</details>

---

## Ejercicio 4: Formulario de login

Crea un formulario con usuario y contraseña que muestre los datos al enviar:

```jsx
function Login() {
  // Tu código aquí
}
```

<details>
<summary>Ver solución</summary>

```jsx
function Login() {
  const [usuario, setUsuario] = useState('');
  const [password, setPassword] = useState('');
  const [enviado, setEnviado] = useState(false);
  
  const handleSubmit = (e) => {
    e.preventDefault();
    setEnviado(true);
  };
  
  if (enviado) {
    return (
      <div>
        <p>Usuario: {usuario}</p>
        <p>Contraseña: {password}</p>
        <button onClick={() => setEnviado(false)}>Volver</button>
      </div>
    );
  }
  
  return (
    <form onSubmit={handleSubmit}>
      <div>
        <input 
          placeholder="Usuario"
          value={usuario}
          onChange={(e) => setUsuario(e.target.value)}
        />
      </div>
      <div>
        <input 
          type="password"
          placeholder="Contraseña"
          value={password}
          onChange={(e) => setPassword(e.target.value)}
        />
      </div>
      <button type="submit">Entrar</button>
    </form>
  );
}
```
</details>

---

## Ejercicio 5: Lista de tareas con eventos

Crea una lista de tareas donde puedas:
- Agregar tarea (Enter o botón)
- Eliminar tarea (botón X)

```jsx
function TodoList() {
  // Tu código aquí
}
```

<details>
<summary>Ver solución</summary>

```jsx
function TodoList() {
  const [tarea, setTarea] = useState('');
  const [tareas, setTareas] = useState([]);
  
  const agregar = () => {
    if (tarea.trim()) {
      setTareas([...tareas, tarea]);
      setTarea('');
    }
  };
  
  const eliminar = (index) => {
    setTareas(tareas.filter((_, i) => i !== index));
  };
  
  const handleKeyDown = (e) => {
    if (e.key === 'Enter') {
      agregar();
    }
  };
  
  return (
    <div>
      <input 
        value={tarea}
        onChange={(e) => setTarea(e.target.value)}
        onKeyDown={handleKeyDown}
        placeholder="Nueva tarea..."
      />
      <button onClick={agregar}>Agregar</button>
      
      <ul>
        {tareas.map((t, index) => (
          <li key={index}>
            {t}
            <button onClick={() => eliminar(index)}>❌</button>
          </li>
        ))}
      </ul>
    </div>
  );
}
```
</details>

---

## Resumen

✅ **Eventos en React usan camelCase**: `onClick`, `onChange`  
✅ **Pasa funciones, no strings**: `{handleClick}`  
✅ **No ejecutes inmediatamente**: `onClick={handleClick}` no `onClick={handleClick()}`  
✅ **preventDefault()** para prevenir comportamiento por defecto  
✅ **stopPropagation()** para detener propagación  
✅ **e.target.value** para obtener valor de inputs  
✅ **e.target.checked** para checkboxes  
✅ **Usa arrow functions para pasar argumentos**

---

## Siguiente paso

Ya sabes manejar eventos. Ahora aprenderás a trabajar con **objetos y arrays en el estado**.

**Siguiente:** [Módulo 14 - Estado con objetos y arrays](./14-estado-objetos-arrays.md)

**Anterior:** [Módulo 12 - useState básico](./12-usestate-basico.md)

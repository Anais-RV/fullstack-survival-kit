# Módulo 11: ¿Qué es el Estado?

## Estado vs Props

Hasta ahora hemos usado **props** para pasar datos. Pero ¿qué pasa cuando los datos necesitan **cambiar**?

### Props (datos externos)
```jsx
function Saludo({ nombre }) {
  // nombre viene del padre
  // Es de SOLO LECTURA
  return <h1>Hola, {nombre}</h1>;
}

<Saludo nombre="Ana" />
```

### Estado (datos internos)
```jsx
function Contador() {
  // count es interno del componente
  // PUEDE CAMBIAR
  const [count, setCount] = useState(0);
  
  return (
    <div>
      <p>Contador: {count}</p>
      <button onClick={() => setCount(count + 1)}>+</button>
    </div>
  );
}
```

---

## ¿Qué es el estado?

**Estado** = **memoria del componente**

Es información que:
- Pertenece al componente
- Puede cambiar con el tiempo
- Hace que el componente se re-renderice cuando cambia

### Analogía: Interruptor de luz

```
Props = Alguien más controla la luz (externa)
Estado = Tú controlas tu propia luz (interna)
```

---

## Componente sin estado (Stateless)

```jsx
function Tarjeta({ titulo, descripcion }) {
  // Solo muestra datos que recibe
  // No tiene memoria
  return (
    <div className="card">
      <h2>{titulo}</h2>
      <p>{descripcion}</p>
    </div>
  );
}
```

**Características:**
- Siempre muestra lo mismo con las mismas props
- No recuerda nada
- No puede cambiar por sí solo

---

## Componente con estado (Stateful)

```jsx
import { useState } from 'react';

function Contador() {
  // TIENE MEMORIA (estado)
  const [count, setCount] = useState(0);
  
  return (
    <div>
      <p>Has hecho click {count} veces</p>
      <button onClick={() => setCount(count + 1)}>
        Incrementar
      </button>
    </div>
  );
}
```

**Características:**
- Recuerda información (el contador)
- Puede cambiar por sí mismo
- Se re-renderiza cuando el estado cambia

---

## ¿Cuándo usar estado?

### ✅ Usa estado cuando:

1. **Datos que cambian con interacciones**
```jsx
function ToggleButton() {
  const [activo, setActivo] = useState(false);
  
  return (
    <button onClick={() => setActivo(!activo)}>
      {activo ? 'Activado' : 'Desactivado'}
    </button>
  );
}
```

2. **Inputs de formularios**
```jsx
function Form() {
  const [nombre, setNombre] = useState('');
  
  return (
    <input 
      value={nombre}
      onChange={(e) => setNombre(e.target.value)}
    />
  );
}
```

3. **Datos cargados de APIs**
```jsx
function UserProfile() {
  const [user, setUser] = useState(null);
  
  useEffect(() => {
    fetch('/api/user')
      .then(res => res.json())
      .then(data => setUser(data));
  }, []);
  
  return <div>{user?.name}</div>;
}
```

4. **UI temporal (modals, tooltips)**
```jsx
function Modal() {
  const [isOpen, setIsOpen] = useState(false);
  
  return (
    <>
      <button onClick={() => setIsOpen(true)}>Abrir</button>
      {isOpen && <div className="modal">...</div>}
    </>
  );
}
```

### ❌ NO uses estado cuando:

1. **Datos que NO cambian**
```jsx
// ❌ No necesita estado
function Logo() {
  const [titulo] = useState('Mi App');  // Nunca cambia
  return <h1>{titulo}</h1>;
}

// ✅ Usa constante
function Logo() {
  const titulo = 'Mi App';
  return <h1>{titulo}</h1>;
}
```

2. **Datos calculados**
```jsx
// ❌ No necesita estado
function Precio({ base, descuento }) {
  const [total, setTotal] = useState(base - descuento);
  return <p>${total}</p>;
}

// ✅ Calcula directamente
function Precio({ base, descuento }) {
  const total = base - descuento;
  return <p>${total}</p>;
}
```

3. **Props que no cambian en el componente**
```jsx
// ❌ No copies props a estado sin razón
function Saludo({ nombre }) {
  const [nombreLocal, setNombreLocal] = useState(nombre);
  return <h1>Hola, {nombreLocal}</h1>;
}

// ✅ Usa la prop directamente
function Saludo({ nombre }) {
  return <h1>Hola, {nombre}</h1>;
}
```

---

## Estado causa re-render

Cuando el estado cambia, React **re-renderiza** el componente:

```jsx
function Contador() {
  const [count, setCount] = useState(0);
  
  console.log('Componente renderizado');  // Se ejecuta en cada render
  
  return (
    <div>
      <p>Contador: {count}</p>
      <button onClick={() => setCount(count + 1)}>+</button>
    </div>
  );
}

// Cada click en el botón:
// 1. setCount actualiza el estado
// 2. React re-renderiza el componente
// 3. Se muestra el nuevo valor
```

---

## Estado es local

Cada instancia de un componente tiene **su propio estado**:

```jsx
function Contador() {
  const [count, setCount] = useState(0);
  
  return (
    <div>
      <p>{count}</p>
      <button onClick={() => setCount(count + 1)}>+</button>
    </div>
  );
}

function App() {
  return (
    <div>
      <Contador />  {/* Estado independiente */}
      <Contador />  {/* Estado independiente */}
      <Contador />  {/* Estado independiente */}
    </div>
  );
}
```

Cada `<Contador>` tiene su propio `count`. Son **independientes**.

---

## Flujo de datos unidireccional

```
       Props ↓
Padre --------→ Hijo
       ↑ Callback
```

- Datos fluyen **hacia abajo** (props)
- Eventos fluyen **hacia arriba** (callbacks)

```jsx
function Parent() {
  const [mensaje, setMensaje] = useState('Hola');
  
  return (
    <Child 
      mensaje={mensaje}                    // Dato hacia abajo
      onCambiar={(nuevo) => setMensaje(nuevo)}  // Callback hacia arriba
    />
  );
}

function Child({ mensaje, onCambiar }) {
  return (
    <div>
      <p>{mensaje}</p>
      <button onClick={() => onCambiar('Chau')}>
        Cambiar mensaje
      </button>
    </div>
  );
}
```

---

## Props vs Estado: Tabla comparativa

| Aspecto | Props | Estado |
|---------|-------|--------|
| **Origen** | Del componente padre | Interno del componente |
| **Mutabilidad** | Solo lectura | Se puede cambiar con `setState` |
| **Causa re-render** | Cuando cambian las props del padre | Cuando llamas `setState` |
| **Quién lo controla** | El padre | El componente mismo |
| **Uso** | Pasar datos hacia abajo | Datos que cambian |

---

## Ejemplo comparativo

### Solo Props (sin estado)
```jsx
function Semaforo({ color }) {
  return (
    <div className={`semaforo semaforo-${color}`}>
      {color === 'verde' && '🟢'}
      {color === 'amarillo' && '🟡'}
      {color === 'rojo' && '🔴'}
    </div>
  );
}

// El padre controla el color
<Semaforo color="verde" />
```

### Con Estado
```jsx
function Semaforo() {
  const [color, setColor] = useState('rojo');
  
  const cambiar = () => {
    if (color === 'rojo') setColor('verde');
    else if (color === 'verde') setColor('amarillo');
    else setColor('rojo');
  };
  
  return (
    <div>
      <div className={`semaforo semaforo-${color}`}>
        {color === 'verde' && '🟢'}
        {color === 'amarillo' && '🟡'}
        {color === 'rojo' && '🔴'}
      </div>
      <button onClick={cambiar}>Cambiar</button>
    </div>
  );
}
```

---

## Estado sincronizado vs independiente

### Estados independientes
```jsx
function Contador() {
  const [count, setCount] = useState(0);
  return <div>{count} <button onClick={() => setCount(count + 1)}>+</button></div>;
}

<Contador />  // count: 0
<Contador />  // count: 0 (independiente)
```

### Estado compartido (lifting state up)
```jsx
function App() {
  const [count, setCount] = useState(0);  // Estado compartido
  
  return (
    <div>
      <Display count={count} />
      <Display count={count} />
      <Boton onIncrement={() => setCount(count + 1)} />
    </div>
  );
}

function Display({ count }) {
  return <p>Contador: {count}</p>;
}

function Boton({ onIncrement }) {
  return <button onClick={onIncrement}>+</button>;
}
```

---

## Ejemplo: Like button

```jsx
function LikeButton() {
  const [likes, setLikes] = useState(0);
  const [liked, setLiked] = useState(false);
  
  const handleLike = () => {
    if (liked) {
      setLikes(likes - 1);
      setLiked(false);
    } else {
      setLikes(likes + 1);
      setLiked(true);
    }
  };
  
  return (
    <button 
      onClick={handleLike}
      className={liked ? 'liked' : ''}
    >
      {liked ? '❤️' : '🤍'} {likes}
    </button>
  );
}
```

---

## Ejemplo: Toggle visibility

```jsx
function Accordion() {
  const [isOpen, setIsOpen] = useState(false);
  
  return (
    <div className="accordion">
      <button onClick={() => setIsOpen(!isOpen)}>
        {isOpen ? '▼' : '▶'} Mostrar contenido
      </button>
      
      {isOpen && (
        <div className="content">
          <p>Este es el contenido oculto</p>
        </div>
      )}
    </div>
  );
}
```

---

## Estado deriva de props (inicialización)

A veces necesitas inicializar estado desde props:

```jsx
function ContadorControlado({ inicial = 0 }) {
  // Inicializa con prop, pero luego es independiente
  const [count, setCount] = useState(inicial);
  
  return (
    <div>
      <p>{count}</p>
      <button onClick={() => setCount(count + 1)}>+</button>
    </div>
  );
}

<ContadorControlado inicial={10} />
```

⚠️ **Importante:** El estado solo se inicializa en el **primer render**. Si `inicial` cambia después, el estado no se actualiza automáticamente.

---

## Cuándo el estado NO se actualiza

### Problema común
```jsx
function Problema({ valor }) {
  const [count, setCount] = useState(valor);
  
  // ❌ Si 'valor' cambia, count NO se actualiza
  return <div>{count}</div>;
}
```

### Solución: useEffect
```jsx
function Solucion({ valor }) {
  const [count, setCount] = useState(valor);
  
  // Sincroniza cuando valor cambia
  useEffect(() => {
    setCount(valor);
  }, [valor]);
  
  return <div>{count}</div>;
}
```

---

## Estado vs Variables normales

### ❌ Variable normal (NO funciona)
```jsx
function Contador() {
  let count = 0;  // Se reinicia en cada render
  
  return (
    <div>
      <p>{count}</p>
      <button onClick={() => count++}>+</button>  {/* No re-renderiza */}
    </div>
  );
}
```

### ✅ Estado (funciona)
```jsx
function Contador() {
  const [count, setCount] = useState(0);  // React lo recuerda
  
  return (
    <div>
      <p>{count}</p>
      <button onClick={() => setCount(count + 1)}>+</button>  {/* Re-renderiza */}
    </div>
  );
}
```

---

## Reglas del estado

1. ✅ **Usa useState en el nivel superior del componente**
2. ✅ **No llames hooks dentro de condicionales o loops**
3. ✅ **El estado es asíncrono** (los cambios no son inmediatos)
4. ✅ **Nunca modifiques el estado directamente** (usa `setState`)
5. ✅ **Estado mínimo necesario** (calcula el resto)

---

## Cuándo dividir estado

### Múltiples estados independientes
```jsx
// ✅ Estados separados si son independientes
function Form() {
  const [nombre, setNombre] = useState('');
  const [email, setEmail] = useState('');
  const [edad, setEdad] = useState(0);
}
```

### Estado agrupado
```jsx
// ✅ Agrupa si están relacionados
function Form() {
  const [form, setForm] = useState({
    nombre: '',
    email: '',
    edad: 0
  });
}
```

---

## Resumen

✅ **Estado = memoria del componente**  
✅ **Cambia con el tiempo**  
✅ **Causa re-renders**  
✅ **Local a cada instancia**  
✅ **Props fluyen hacia abajo, estado es interno**  
✅ **Usa estado solo cuando los datos cambien**  
✅ **Nunca modifiques estado directamente**

---

## Siguiente paso

Entiendes **qué es** el estado. Ahora aprenderás **cómo usarlo** con `useState`.

**Siguiente:** [Módulo 12 - useState básico](./12-usestate-basico.md)

**Anterior:** [Módulo 10 - Listas y keys](./10-listas-keys.md)

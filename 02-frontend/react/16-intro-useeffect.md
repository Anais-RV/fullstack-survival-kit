# Módulo 16: Introducción a useEffect

## ¿Qué es useEffect?

`useEffect` es un Hook que te permite realizar **efectos secundarios** en componentes funcionales.

### ¿Qué es un efecto secundario?

Un efecto secundario es cualquier operación que **afecta algo fuera del componente**:

- 🌐 Llamadas a APIs (fetch)
- 📝 Manipular el DOM directamente
- ⏱️ Timers (setTimeout, setInterval)
- 🔔 Suscripciones (WebSockets, eventos)
- 💾 localStorage / sessionStorage
- 📊 Logging / Analytics
- 🔄 Sincronizar con sistemas externos

---

## ¿Por qué useEffect?

En componentes funcionales, el cuerpo del componente debe ser **puro**:

### ❌ NO hagas esto
```jsx
function Componente() {
  // ❌ Efecto secundario durante el render
  document.title = 'Nueva página';
  
  return <div>Hola</div>;
}
```

**Problema:** Se ejecuta en **cada render**, puede causar problemas de rendimiento.

### ✅ Haz esto
```jsx
function Componente() {
  useEffect(() => {
    // ✅ Efecto secundario en useEffect
    document.title = 'Nueva página';
  }, []);
  
  return <div>Hola</div>;
}
```

---

## Sintaxis básica

```jsx
import { useEffect } from 'react';

function Componente() {
  useEffect(() => {
    // Código del efecto
    console.log('Efecto ejecutado');
  });
  
  return <div>Componente</div>;
}
```

**Partes:**
- `useEffect` → Hook de React
- `() => { ... }` → Función que contiene el efecto
- Se ejecuta **después** de cada render

---

## Cuándo se ejecuta useEffect

```jsx
function Componente() {
  console.log('1. Render');
  
  useEffect(() => {
    console.log('2. Efecto');
  });
  
  return <div>Componente</div>;
}

// Salida:
// 1. Render
// 2. Efecto
```

**Orden de ejecución:**
1. React renderiza el componente
2. React actualiza el DOM
3. React ejecuta useEffect

---

## Ejemplo: Cambiar el título de la página

```jsx
function Contador() {
  const [count, setCount] = useState(0);
  
  useEffect(() => {
    document.title = `Clicks: ${count}`;
  });
  
  return (
    <div>
      <p>Has hecho click {count} veces</p>
      <button onClick={() => setCount(count + 1)}>Click</button>
    </div>
  );
}
```

**Cada vez que `count` cambia:**
1. Componente se re-renderiza
2. DOM se actualiza
3. `useEffect` actualiza el título

---

## useEffect vs código en el render

### Código en el render
```jsx
function Componente() {
  const [count, setCount] = useState(0);
  
  // ❌ Se ejecuta DURANTE el render
  console.log('Renderizando...');
  
  return <div>{count}</div>;
}
```

### useEffect
```jsx
function Componente() {
  const [count, setCount] = useState(0);
  
  // ✅ Se ejecuta DESPUÉS del render
  useEffect(() => {
    console.log('Efecto después del render');
  });
  
  return <div>{count}</div>;
}
```

---

## Ejemplo: Logging

```jsx
function Formulario() {
  const [nombre, setNombre] = useState('');
  
  useEffect(() => {
    console.log('Componente renderizado');
    console.log('Nombre actual:', nombre);
  });
  
  return (
    <input 
      value={nombre}
      onChange={(e) => setNombre(e.target.value)}
    />
  );
}
```

**Cada vez que escribes en el input:**
- El estado `nombre` cambia
- Componente se re-renderiza
- `useEffect` se ejecuta y muestra el log

---

## Múltiples useEffect

Puedes tener varios `useEffect` en un mismo componente:

```jsx
function Componente() {
  const [count, setCount] = useState(0);
  const [nombre, setNombre] = useState('');
  
  useEffect(() => {
    console.log('Efecto 1: count cambió');
  });
  
  useEffect(() => {
    console.log('Efecto 2: nombre cambió');
  });
  
  return (
    <div>
      <p>{count}</p>
      <input value={nombre} onChange={(e) => setNombre(e.target.value)} />
    </div>
  );
}
```

Cada `useEffect` es **independiente**.

---

## useEffect sin dependencias (se ejecuta siempre)

```jsx
function Componente() {
  const [count, setCount] = useState(0);
  
  useEffect(() => {
    console.log('Se ejecuta en CADA render');
  });  // ⚠️ Sin array de dependencias
  
  return (
    <button onClick={() => setCount(count + 1)}>
      Click: {count}
    </button>
  );
}
```

**Cada click:**
- Estado cambia
- Componente se re-renderiza
- `useEffect` se ejecuta

---

## Ejemplo: Scroll al inicio

```jsx
function Pagina() {
  useEffect(() => {
    window.scrollTo(0, 0);
  });
  
  return (
    <div style={{ height: '2000px' }}>
      <h1>Página larga</h1>
    </div>
  );
}
```

Cada vez que el componente se renderiza, hace scroll al inicio.

---

## Ejemplo: Focus automático

```jsx
function InputAutoFocus() {
  const inputRef = useRef(null);
  
  useEffect(() => {
    inputRef.current.focus();
  });
  
  return <input ref={inputRef} />;
}
```

---

## Ejemplo: Actualizar localStorage

```jsx
function Contador() {
  const [count, setCount] = useState(() => {
    const guardado = localStorage.getItem('count');
    return guardado ? Number(guardado) : 0;
  });
  
  useEffect(() => {
    localStorage.setItem('count', count);
    console.log('Guardado en localStorage:', count);
  });
  
  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={() => setCount(count + 1)}>+</button>
    </div>
  );
}
```

**Cada vez que `count` cambia:**
- Se guarda en localStorage
- Persiste entre recargas de página

---

## Diferencia con useLayoutEffect

```jsx
// useEffect: se ejecuta DESPUÉS de pintar
useEffect(() => {
  console.log('Efecto asíncrono');
});

// useLayoutEffect: se ejecuta ANTES de pintar
useLayoutEffect(() => {
  console.log('Efecto síncrono');
});
```

**Usa `useEffect` en el 99% de los casos.**  
Solo usa `useLayoutEffect` si necesitas medir el DOM o hacer cambios visuales antes de que el usuario vea la pantalla.

---

## El problema de ejecutar en cada render

```jsx
function Problema() {
  const [count, setCount] = useState(0);
  
  useEffect(() => {
    // ⚠️ Se ejecuta en CADA render
    fetch('/api/data')
      .then(res => res.json())
      .then(data => console.log(data));
  });
  
  return <button onClick={() => setCount(count + 1)}>Click</button>;
}
```

**Problema:**
- Cada click cambia el estado
- Componente se re-renderiza
- Se hace una nueva llamada a la API
- **Muchas llamadas innecesarias** 🔴

**Solución:** Array de dependencias (siguiente módulo)

---

## useEffect NO es asíncrono

```jsx
// ❌ No puedes hacer esto
useEffect(async () => {
  const data = await fetch('/api');
});
```

**Error:** useEffect no acepta funciones `async` directamente.

### ✅ Solución: Función async interna

```jsx
useEffect(() => {
  async function fetchData() {
    const response = await fetch('/api');
    const data = await response.json();
    console.log(data);
  }
  
  fetchData();
}, []);
```

### ✅ Solución: IIFE

```jsx
useEffect(() => {
  (async () => {
    const response = await fetch('/api');
    const data = await response.json();
    console.log(data);
  })();
}, []);
```

---

## Ejemplo: Timer simple

```jsx
function Reloj() {
  const [hora, setHora] = useState(new Date().toLocaleTimeString());
  
  useEffect(() => {
    const interval = setInterval(() => {
      setHora(new Date().toLocaleTimeString());
    }, 1000);
    
    // ⚠️ Problema: se crean múltiples intervals
    // Solución: cleanup function (módulo 19)
  });
  
  return <h1>{hora}</h1>;
}
```

⚠️ **Problema:** Cada render crea un nuevo interval sin limpiar el anterior.

---

## Ciclo de vida en componentes funcionales

### Componentes de clase (antiguo)
```jsx
class Componente extends React.Component {
  componentDidMount() {
    // Al montar
  }
  
  componentDidUpdate() {
    // Al actualizar
  }
  
  componentWillUnmount() {
    // Al desmontar
  }
}
```

### Componentes funcionales con useEffect
```jsx
function Componente() {
  useEffect(() => {
    // Al montar + al actualizar
    
    return () => {
      // Al desmontar
    };
  });
}
```

---

## Cuándo usar useEffect

### ✅ Usa useEffect para:

1. **Fetch de datos**
```jsx
useEffect(() => {
  fetch('/api/users').then(/* ... */);
}, []);
```

2. **Sincronizar con sistemas externos**
```jsx
useEffect(() => {
  const subscription = chatAPI.subscribe();
  return () => subscription.unsubscribe();
}, []);
```

3. **Efectos secundarios basados en estado**
```jsx
useEffect(() => {
  document.title = `Count: ${count}`;
}, [count]);
```

4. **Manipular el DOM**
```jsx
useEffect(() => {
  const element = document.getElementById('special');
  element.classList.add('active');
}, []);
```

---

### ❌ NO uses useEffect para:

1. **Transformar datos para renderizar**
```jsx
// ❌ No necesita useEffect
useEffect(() => {
  setTotal(precio * cantidad);
}, [precio, cantidad]);

// ✅ Calcula directamente
const total = precio * cantidad;
```

2. **Manejar eventos del usuario**
```jsx
// ❌ No necesita useEffect
useEffect(() => {
  if (clicked) {
    alert('Clicked!');
  }
}, [clicked]);

// ✅ Maneja en el handler
const handleClick = () => {
  alert('Clicked!');
};
```

3. **Reiniciar estado cuando props cambian**
```jsx
// ❌ No necesita useEffect
useEffect(() => {
  setItems(propItems);
}, [propItems]);

// ✅ Usa la prop directamente o deriva el estado
```

---

## Ejemplo: Cambiar clase del body

```jsx
function TemaOscuro() {
  const [oscuro, setOscuro] = useState(false);
  
  useEffect(() => {
    if (oscuro) {
      document.body.classList.add('dark');
    } else {
      document.body.classList.remove('dark');
    }
  });
  
  return (
    <button onClick={() => setOscuro(!oscuro)}>
      {oscuro ? '☀️ Modo Claro' : '🌙 Modo Oscuro'}
    </button>
  );
}
```

---

## Ejemplo: Detectar ancho de ventana

```jsx
function AnchoVentana() {
  const [ancho, setAncho] = useState(window.innerWidth);
  
  useEffect(() => {
    const handleResize = () => {
      setAncho(window.innerWidth);
    };
    
    window.addEventListener('resize', handleResize);
    
    // ⚠️ Falta limpieza (módulo 19)
  });
  
  return <p>Ancho: {ancho}px</p>;
}
```

---

## Reglas de useEffect

1. ✅ **Solo llama Hooks en el nivel superior**
```jsx
// ❌ No en condicionales
if (condicion) {
  useEffect(() => { });
}

// ✅ Condicional dentro del efecto
useEffect(() => {
  if (condicion) { }
});
```

2. ✅ **Solo llama Hooks desde componentes React**
```jsx
// ❌ No en funciones normales
function helper() {
  useEffect(() => { });
}

// ✅ Dentro de componentes
function Componente() {
  useEffect(() => { });
}
```

---

## Ejercicio 1: Contador en el título

Crea un contador que actualice el título de la página con el número actual:

```jsx
function Contador() {
  const [count, setCount] = useState(0);
  
  // Tu código aquí
  
  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={() => setCount(count + 1)}>+</button>
    </div>
  );
}
```

<details>
<summary>Ver solución</summary>

```jsx
function Contador() {
  const [count, setCount] = useState(0);
  
  useEffect(() => {
    document.title = `Contador: ${count}`;
  });
  
  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={() => setCount(count + 1)}>+</button>
    </div>
  );
}
```
</details>

---

## Ejercicio 2: Logger de cambios

Crea un componente que registre en consola cada vez que el estado cambia:

```jsx
function Logger() {
  const [nombre, setNombre] = useState('');
  
  // Tu código aquí (log cuando nombre cambie)
  
  return (
    <input 
      value={nombre}
      onChange={(e) => setNombre(e.target.value)}
    />
  );
}
```

<details>
<summary>Ver solución</summary>

```jsx
function Logger() {
  const [nombre, setNombre] = useState('');
  
  useEffect(() => {
    console.log('Nombre cambió a:', nombre);
  });
  
  return (
    <input 
      value={nombre}
      onChange={(e) => setNombre(e.target.value)}
      placeholder="Escribe algo..."
    />
  );
}
```
</details>

---

## Ejercicio 3: Guardar en localStorage

Crea un input que guarde su valor en localStorage en cada cambio:

```jsx
function InputPersistente() {
  const [texto, setTexto] = useState('');
  
  // Tu código aquí
  
  return (
    <input 
      value={texto}
      onChange={(e) => setTexto(e.target.value)}
    />
  );
}
```

<details>
<summary>Ver solución</summary>

```jsx
function InputPersistente() {
  const [texto, setTexto] = useState(() => {
    return localStorage.getItem('texto') || '';
  });
  
  useEffect(() => {
    localStorage.setItem('texto', texto);
    console.log('Guardado:', texto);
  });
  
  return (
    <div>
      <input 
        value={texto}
        onChange={(e) => setTexto(e.target.value)}
        placeholder="Escribe algo..."
      />
      <button onClick={() => setTexto('')}>Limpiar</button>
    </div>
  );
}
```
</details>

---

## Ejercicio 4: Múltiples efectos

Crea un componente con dos estados y dos efectos separados:

```jsx
function MultipleEfectos() {
  const [count, setCount] = useState(0);
  const [nombre, setNombre] = useState('');
  
  // Efecto 1: actualiza el título con el count
  // Efecto 2: log cuando nombre cambie
  
  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={() => setCount(count + 1)}>+</button>
      
      <input 
        value={nombre}
        onChange={(e) => setNombre(e.target.value)}
      />
    </div>
  );
}
```

<details>
<summary>Ver solución</summary>

```jsx
function MultipleEfectos() {
  const [count, setCount] = useState(0);
  const [nombre, setNombre] = useState('');
  
  useEffect(() => {
    document.title = `Count: ${count}`;
    console.log('Efecto 1: título actualizado');
  });
  
  useEffect(() => {
    console.log('Efecto 2: nombre cambió a:', nombre);
  });
  
  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={() => setCount(count + 1)}>+</button>
      
      <input 
        value={nombre}
        onChange={(e) => setNombre(e.target.value)}
        placeholder="Nombre"
      />
    </div>
  );
}
```
</details>

---

## Resumen

✅ **useEffect** ejecuta efectos secundarios  
✅ **Se ejecuta DESPUÉS del render**  
✅ **Sin dependencias** → se ejecuta en cada render  
✅ **Múltiples useEffect** son independientes  
✅ **No uses `async` directamente** en useEffect  
✅ **Usa para:** fetch, DOM, timers, suscripciones  
✅ **No uses para:** cálculos, transformaciones, eventos

---

## Siguiente paso

Entiendes qué es useEffect y cuándo se ejecuta. Ahora aprenderás a **controlar cuándo se ejecuta** con el **array de dependencias**.

**Siguiente:** [Módulo 17 - Dependencias de useEffect](./17-dependencias-useeffect.md)

**Anterior:** [Módulo 15 - Levantar estado](./15-levantar-estado.md)

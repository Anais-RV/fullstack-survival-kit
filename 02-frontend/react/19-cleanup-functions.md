# Módulo 19: Cleanup Functions (Funciones de Limpieza)

## ¿Qué es una función de limpieza?

Una **cleanup function** es una función que **limpia efectos secundarios** antes de que el componente se desmonte o antes de que el efecto se ejecute de nuevo.

```jsx
useEffect(() => {
  // Efecto
  console.log('Efecto ejecutado');
  
  // Cleanup function
  return () => {
    console.log('Limpieza ejecutada');
  };
}, []);
```

---

## ¿Cuándo se ejecuta la cleanup?

### 1. Antes de que el componente se desmonte

```jsx
function Componente() {
  useEffect(() => {
    console.log('Componente montado');
    
    return () => {
      console.log('Componente se va a desmontar');
    };
  }, []);
  
  return <div>Componente</div>;
}
```

---

### 2. Antes de ejecutar el efecto de nuevo

```jsx
function Componente() {
  const [count, setCount] = useState(0);
  
  useEffect(() => {
    console.log('Efecto ejecutado');
    
    return () => {
      console.log('Limpieza antes del siguiente efecto');
    };
  }, [count]);  // Se ejecuta cuando count cambia
  
  return <button onClick={() => setCount(count + 1)}>{count}</button>;
}

// Flujo:
// 1. Primer render: "Efecto ejecutado"
// 2. Click: "Limpieza antes del siguiente efecto" → "Efecto ejecutado"
// 3. Click: "Limpieza antes del siguiente efecto" → "Efecto ejecutado"
```

---

## ¿Por qué necesitas cleanup?

### Problema sin cleanup: Memory leaks

```jsx
// ❌ SIN CLEANUP (Memory leak)
function Reloj() {
  const [hora, setHora] = useState(new Date());
  
  useEffect(() => {
    const interval = setInterval(() => {
      setHora(new Date());
    }, 1000);
    
    // ❌ No se limpia el interval
  }, []);
  
  return <h1>{hora.toLocaleTimeString()}</h1>;
}

// Problema: Si el componente se desmonta,
// el interval sigue ejecutándose en segundo plano
```

### ✅ Con cleanup

```jsx
function Reloj() {
  const [hora, setHora] = useState(new Date());
  
  useEffect(() => {
    const interval = setInterval(() => {
      setHora(new Date());
    }, 1000);
    
    // ✅ Limpia el interval
    return () => {
      clearInterval(interval);
    };
  }, []);
  
  return <h1>{hora.toLocaleTimeString()}</h1>;
}
```

---

## Casos comunes de cleanup

### 1. setInterval / setTimeout

```jsx
function Temporizador() {
  const [segundos, setSegundos] = useState(0);
  
  useEffect(() => {
    const interval = setInterval(() => {
      setSegundos(s => s + 1);
    }, 1000);
    
    return () => clearInterval(interval);
  }, []);
  
  return <p>{segundos} segundos</p>;
}
```

---

### 2. Event listeners

```jsx
function ComponenteConScroll() {
  const [scrollY, setScrollY] = useState(0);
  
  useEffect(() => {
    const handleScroll = () => {
      setScrollY(window.scrollY);
    };
    
    window.addEventListener('scroll', handleScroll);
    
    // ✅ Remueve el listener
    return () => {
      window.removeEventListener('scroll', handleScroll);
    };
  }, []);
  
  return <p>Scroll: {scrollY}px</p>;
}
```

---

### 3. AbortController (fetch)

```jsx
function Datos() {
  const [data, setData] = useState(null);
  
  useEffect(() => {
    const controller = new AbortController();
    
    fetch('/api/data', { signal: controller.signal })
      .then(res => res.json())
      .then(data => setData(data))
      .catch(err => {
        if (err.name !== 'AbortError') {
          console.error('Error:', err);
        }
      });
    
    // ✅ Cancela el fetch
    return () => {
      controller.abort();
    };
  }, []);
  
  return <div>{data ? JSON.stringify(data) : 'Cargando...'}</div>;
}
```

---

### 4. Subscripciones (WebSocket, observables)

```jsx
function Chat() {
  const [mensajes, setMensajes] = useState([]);
  
  useEffect(() => {
    const socket = new WebSocket('ws://localhost:8080');
    
    socket.onmessage = (event) => {
      setMensajes(prev => [...prev, event.data]);
    };
    
    // ✅ Cierra la conexión
    return () => {
      socket.close();
    };
  }, []);
  
  return (
    <ul>
      {mensajes.map((msg, i) => <li key={i}>{msg}</li>)}
    </ul>
  );
}
```

---

### 5. Clases del DOM

```jsx
function ModalConBloqueo() {
  useEffect(() => {
    // Bloquea el scroll cuando el modal está abierto
    document.body.classList.add('no-scroll');
    
    // ✅ Restaura el scroll cuando el modal se cierra
    return () => {
      document.body.classList.remove('no-scroll');
    };
  }, []);
  
  return <div className="modal">Modal abierto</div>;
}
```

---

## Ejemplo completo: Reloj con play/pause

```jsx
function RelojControlado() {
  const [hora, setHora] = useState(new Date());
  const [activo, setActivo] = useState(true);
  
  useEffect(() => {
    if (!activo) return;  // No hace nada si está pausado
    
    const interval = setInterval(() => {
      setHora(new Date());
    }, 1000);
    
    return () => {
      clearInterval(interval);
      console.log('Interval limpiado');
    };
  }, [activo]);  // Se reinicia cuando activo cambia
  
  return (
    <div>
      <h1>{hora.toLocaleTimeString()}</h1>
      <button onClick={() => setActivo(!activo)}>
        {activo ? 'Pausar' : 'Reanudar'}
      </button>
    </div>
  );
}
```

**Flujo:**
- Activo = true → Crea interval
- Click en "Pausar" → Limpia interval, activo = false
- Click en "Reanudar" → Crea nuevo interval, activo = true

---

## Cleanup con dependencias

```jsx
function BuscadorConDebounce({ query }) {
  const [resultados, setResultados] = useState([]);
  
  useEffect(() => {
    const timer = setTimeout(() => {
      fetch(`/api/search?q=${query}`)
        .then(res => res.json())
        .then(data => setResultados(data));
    }, 500);
    
    // Se limpia en cada cambio de query
    return () => {
      clearTimeout(timer);
    };
  }, [query]);
  
  return (
    <ul>
      {resultados.map(r => <li key={r.id}>{r.nombre}</li>)}
    </ul>
  );
}
```

**Flujo:**
- Usuario escribe "r" → Timer de 500ms
- Usuario escribe "re" → Limpia timer anterior, nuevo timer de 500ms
- Usuario escribe "rea" → Limpia timer anterior, nuevo timer de 500ms
- Pasan 500ms sin cambios → Fetch

---

## Ejemplo: Resize listener

```jsx
function ComponenteResponsive() {
  const [ancho, setAncho] = useState(window.innerWidth);
  
  useEffect(() => {
    const handleResize = () => {
      setAncho(window.innerWidth);
    };
    
    window.addEventListener('resize', handleResize);
    
    return () => {
      window.removeEventListener('resize', handleResize);
    };
  }, []);
  
  return (
    <div>
      <p>Ancho de ventana: {ancho}px</p>
      {ancho < 768 ? <p>📱 Móvil</p> : <p>💻 Desktop</p>}
    </div>
  );
}
```

---

## Ejemplo: Click fuera para cerrar

```jsx
function Dropdown() {
  const [isOpen, setIsOpen] = useState(false);
  const ref = useRef(null);
  
  useEffect(() => {
    if (!isOpen) return;  // Solo si está abierto
    
    const handleClickOutside = (event) => {
      if (ref.current && !ref.current.contains(event.target)) {
        setIsOpen(false);
      }
    };
    
    document.addEventListener('mousedown', handleClickOutside);
    
    return () => {
      document.removeEventListener('mousedown', handleClickOutside);
    };
  }, [isOpen]);
  
  return (
    <div ref={ref}>
      <button onClick={() => setIsOpen(!isOpen)}>Toggle</button>
      {isOpen && (
        <div className="dropdown-menu">
          <p>Opción 1</p>
          <p>Opción 2</p>
        </div>
      )}
    </div>
  );
}
```

---

## Ejemplo: Animación con requestAnimationFrame

```jsx
function Animacion() {
  const [posicion, setPosicion] = useState(0);
  const [animando, setAnimando] = useState(false);
  
  useEffect(() => {
    if (!animando) return;
    
    let requestId;
    
    const animar = () => {
      setPosicion(prev => (prev + 1) % 400);
      requestId = requestAnimationFrame(animar);
    };
    
    requestId = requestAnimationFrame(animar);
    
    return () => {
      cancelAnimationFrame(requestId);
    };
  }, [animando]);
  
  return (
    <div>
      <div 
        style={{
          width: 50,
          height: 50,
          backgroundColor: 'blue',
          transform: `translateX(${posicion}px)`,
          transition: 'transform 0.016s linear'
        }}
      />
      <button onClick={() => setAnimando(!animando)}>
        {animando ? 'Pausar' : 'Animar'}
      </button>
    </div>
  );
}
```

---

## Cleanup con async/await

**⚠️ Problema:** No puedes retornar directamente desde una función async

```jsx
// ❌ Incorrecto
useEffect(async () => {
  const data = await fetch('/api/data');
  return () => cleanup();  // ❌ No funciona
}, []);
```

### ✅ Solución: Función async interna

```jsx
useEffect(() => {
  let cancelled = false;
  
  async function fetchData() {
    try {
      const response = await fetch('/api/data');
      const data = await response.json();
      
      if (!cancelled) {
        setData(data);
      }
    } catch (err) {
      if (!cancelled) {
        setError(err.message);
      }
    }
  }
  
  fetchData();
  
  return () => {
    cancelled = true;
  };
}, []);
```

---

## Ejemplo: Guardar borrador automáticamente

```jsx
function EditorTexto() {
  const [texto, setTexto] = useState('');
  const [guardando, setGuardando] = useState(false);
  
  useEffect(() => {
    setGuardando(true);
    
    const timer = setTimeout(() => {
      localStorage.setItem('borrador', texto);
      setGuardando(false);
      console.log('Borrador guardado');
    }, 1000);
    
    return () => {
      clearTimeout(timer);
    };
  }, [texto]);
  
  return (
    <div>
      <textarea 
        value={texto}
        onChange={(e) => setTexto(e.target.value)}
        rows={10}
        cols={50}
      />
      {guardando && <p>Guardando...</p>}
    </div>
  );
}
```

---

## Ejemplo: Polling (consultar periódicamente)

```jsx
function EstadoServidor() {
  const [estado, setEstado] = useState('desconocido');
  const [activo, setActivo] = useState(true);
  
  useEffect(() => {
    if (!activo) return;
    
    const checkEstado = async () => {
      try {
        const response = await fetch('/api/estado');
        const data = await response.json();
        setEstado(data.estado);
      } catch (err) {
        setEstado('error');
      }
    };
    
    // Consulta inmediata
    checkEstado();
    
    // Consulta cada 5 segundos
    const interval = setInterval(checkEstado, 5000);
    
    return () => {
      clearInterval(interval);
    };
  }, [activo]);
  
  return (
    <div>
      <p>Estado del servidor: {estado}</p>
      <button onClick={() => setActivo(!activo)}>
        {activo ? 'Detener polling' : 'Iniciar polling'}
      </button>
    </div>
  );
}
```

---

## Múltiples cleanups

```jsx
function ComponenteComplejo() {
  useEffect(() => {
    // Event listener 1
    const handleScroll = () => console.log('scroll');
    window.addEventListener('scroll', handleScroll);
    
    // Event listener 2
    const handleResize = () => console.log('resize');
    window.addEventListener('resize', handleResize);
    
    // Interval
    const interval = setInterval(() => console.log('tick'), 1000);
    
    // Cleanup de TODOS
    return () => {
      window.removeEventListener('scroll', handleScroll);
      window.removeEventListener('resize', handleResize);
      clearInterval(interval);
      console.log('Todo limpiado');
    };
  }, []);
  
  return <div>Componente</div>;
}
```

---

## Ejercicio 1: Contador automático

Crea un contador que incremente cada segundo y se limpie correctamente:

```jsx
function ContadorAutomatico() {
  const [count, setCount] = useState(0);
  
  // Tu código aquí: useEffect con setInterval y cleanup
  
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
      setCount(prev => prev + 1);
    }, 1000);
    
    return () => {
      clearInterval(interval);
      console.log('Interval limpiado');
    };
  }, []);
  
  return <h1>{count}</h1>;
}
```
</details>

---

## Ejercicio 2: Detectar tecla presionada

Crea un componente que detecte qué tecla se presionó:

```jsx
function DetectorTecla() {
  const [tecla, setTecla] = useState('');
  
  // Tu código aquí: addEventListener y cleanup
  
  return <p>Última tecla: {tecla}</p>;
}
```

<details>
<summary>Ver solución</summary>

```jsx
function DetectorTecla() {
  const [tecla, setTecla] = useState('');
  
  useEffect(() => {
    const handleKeyDown = (event) => {
      setTecla(event.key);
    };
    
    window.addEventListener('keydown', handleKeyDown);
    
    return () => {
      window.removeEventListener('keydown', handleKeyDown);
    };
  }, []);
  
  return (
    <div>
      <p>Última tecla: {tecla || 'Ninguna'}</p>
      <p>Presiona cualquier tecla...</p>
    </div>
  );
}
```
</details>

---

## Ejercicio 3: Posición del mouse

Crea un componente que muestre la posición del mouse:

```jsx
function PosicionMouse() {
  const [posicion, setPosicion] = useState({ x: 0, y: 0 });
  
  // Tu código aquí: mousemove listener y cleanup
  
  return <p>Mouse: {posicion.x}, {posicion.y}</p>;
}
```

<details>
<summary>Ver solución</summary>

```jsx
function PosicionMouse() {
  const [posicion, setPosicion] = useState({ x: 0, y: 0 });
  
  useEffect(() => {
    const handleMouseMove = (event) => {
      setPosicion({ x: event.clientX, y: event.clientY });
    };
    
    window.addEventListener('mousemove', handleMouseMove);
    
    return () => {
      window.removeEventListener('mousemove', handleMouseMove);
    };
  }, []);
  
  return (
    <div>
      <p>Mouse: X={posicion.x}, Y={posicion.y}</p>
    </div>
  );
}
```
</details>

---

## Ejercicio 4: Temporizador con countdown

Crea un temporizador que cuente hacia atrás desde 10:

```jsx
function Countdown() {
  const [segundos, setSegundos] = useState(10);
  
  // Tu código aquí:
  // - setInterval que decremente cada segundo
  // - Detener cuando llegue a 0
  // - Cleanup
  
  return <h1>{segundos}</h1>;
}
```

<details>
<summary>Ver solución</summary>

```jsx
function Countdown() {
  const [segundos, setSegundos] = useState(10);
  
  useEffect(() => {
    if (segundos === 0) return;
    
    const interval = setInterval(() => {
      setSegundos(prev => {
        if (prev <= 1) {
          return 0;
        }
        return prev - 1;
      });
    }, 1000);
    
    return () => {
      clearInterval(interval);
    };
  }, [segundos]);
  
  return (
    <div>
      <h1>{segundos}</h1>
      {segundos === 0 && <p>¡Tiempo terminado!</p>}
    </div>
  );
}
```
</details>

---

## Resumen

✅ **Cleanup function** limpia efectos secundarios  
✅ **Se ejecuta** antes de desmontar o antes de re-ejecutar el efecto  
✅ **clearInterval / clearTimeout** para timers  
✅ **removeEventListener** para event listeners  
✅ **controller.abort()** para fetch  
✅ **socket.close()** para WebSockets  
✅ **Previene memory leaks** y comportamientos inesperados  
✅ **Siempre limpia** lo que inicias en el efecto

---

## Siguiente paso

¡Completaste el bloque de **Efectos**! Ya dominas useEffect y cleanup functions. Ahora estás listo para el siguiente bloque: **Formularios**.

**Siguiente:** [Módulo 20 - Formularios en React](../bloque-05-formularios/20-formularios-intro.md)

**Anterior:** [Módulo 18 - Fetching de datos](./18-fetching-datos.md)

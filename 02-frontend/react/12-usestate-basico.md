# Módulo 12: useState Básico

## ¿Qué es useState?

`useState` es un **Hook** de React que te permite añadir **estado** a un componente funcional.

### Sintaxis básica

```jsx
import { useState } from 'react';

function Componente() {
  const [estado, setEstado] = useState(valorInicial);
  
  return <div>{estado}</div>;
}
```

**Partes:**
- `useState` → Hook de React
- `valorInicial` → Valor con el que comienza el estado
- `estado` → Variable que contiene el valor actual
- `setEstado` → Función para actualizar el estado

---

## Importar useState

Siempre debes importar `useState` de React:

```jsx
// ✅ Importación nombrada
import { useState } from 'react';

// ❌ No funciona sin importar
function Contador() {
  const [count, setCount] = useState(0);  // ❌ Error
}
```

---

## Tu primer estado

```jsx
import { useState } from 'react';

function Contador() {
  // Declara estado llamado 'count' con valor inicial 0
  const [count, setCount] = useState(0);
  
  return (
    <div>
      <p>El contador es: {count}</p>
    </div>
  );
}
```

**¿Qué pasa aquí?**
1. `useState(0)` crea un estado con valor inicial `0`
2. Devuelve un array `[valorActual, funcionActualizadora]`
3. Usamos destructuring: `const [count, setCount] = ...`
4. `count` contiene el valor actual (0)
5. `setCount` es la función para cambiarlo

---

## Actualizar estado

Usa la función `set` para actualizar el estado:

```jsx
import { useState } from 'react';

function Contador() {
  const [count, setCount] = useState(0);
  
  const incrementar = () => {
    setCount(count + 1);  // Actualiza el estado
  };
  
  return (
    <div>
      <p>Contador: {count}</p>
      <button onClick={incrementar}>Incrementar</button>
    </div>
  );
}
```

**Flujo:**
1. Click en el botón
2. Se ejecuta `incrementar()`
3. `setCount(count + 1)` actualiza el estado
4. React re-renderiza el componente
5. Se muestra el nuevo valor

---

## Múltiples estados

Puedes tener varios `useState` en un mismo componente:

```jsx
function Formulario() {
  const [nombre, setNombre] = useState('');
  const [edad, setEdad] = useState(0);
  const [activo, setActivo] = useState(false);
  
  return (
    <div>
      <p>Nombre: {nombre}</p>
      <p>Edad: {edad}</p>
      <p>Activo: {activo ? 'Sí' : 'No'}</p>
    </div>
  );
}
```

Cada `useState` es **independiente**.

---

## Tipos de valores iniciales

### Primitivos
```jsx
const [numero, setNumero] = useState(0);
const [texto, setTexto] = useState('');
const [booleano, setBooleano] = useState(false);
const [nulo, setNulo] = useState(null);
```

### Objetos
```jsx
const [usuario, setUsuario] = useState({
  nombre: 'Ana',
  edad: 25
});
```

### Arrays
```jsx
const [items, setItems] = useState([]);
const [numeros, setNumeros] = useState([1, 2, 3]);
```

### Funciones (inicialización perezosa)
```jsx
// ✅ Solo se ejecuta en el primer render
const [estado, setEstado] = useState(() => {
  const valorInicial = operacionCostosa();
  return valorInicial;
});

// ❌ Se ejecuta en cada render
const [estado, setEstado] = useState(operacionCostosa());
```

---

## Actualización es asíncrona

Los cambios de estado **no son inmediatos**:

```jsx
function Ejemplo() {
  const [count, setCount] = useState(0);
  
  const incrementar = () => {
    setCount(count + 1);
    console.log(count);  // ❌ Muestra el valor ANTERIOR (0)
  };
  
  return <button onClick={incrementar}>+</button>;
}
```

**¿Por qué?** React agrupa las actualizaciones por rendimiento.

---

## Actualización basada en estado anterior

Si el nuevo valor depende del anterior, usa la **forma funcional**:

### ❌ Problema
```jsx
function Contador() {
  const [count, setCount] = useState(0);
  
  const incrementarTres = () => {
    setCount(count + 1);  // count = 0 + 1 = 1
    setCount(count + 1);  // count = 0 + 1 = 1  (usa el valor anterior)
    setCount(count + 1);  // count = 0 + 1 = 1
    // Resultado: 1 (no 3)
  };
  
  return <button onClick={incrementarTres}>+3</button>;
}
```

### ✅ Solución
```jsx
function Contador() {
  const [count, setCount] = useState(0);
  
  const incrementarTres = () => {
    setCount(prev => prev + 1);  // 0 + 1 = 1
    setCount(prev => prev + 1);  // 1 + 1 = 2
    setCount(prev => prev + 1);  // 2 + 1 = 3
    // Resultado: 3
  };
  
  return <button onClick={incrementarTres}>+3</button>;
}
```

**Regla:**
- Usa `setState(nuevoValor)` si no depende del anterior
- Usa `setState(prev => ...)` si depende del anterior

---

## Resetear estado

```jsx
function Formulario() {
  const [nombre, setNombre] = useState('');
  const [email, setEmail] = useState('');
  
  const resetear = () => {
    setNombre('');
    setEmail('');
  };
  
  return (
    <div>
      <input value={nombre} onChange={(e) => setNombre(e.target.value)} />
      <input value={email} onChange={(e) => setEmail(e.target.value)} />
      <button onClick={resetear}>Limpiar</button>
    </div>
  );
}
```

---

## Ejemplo: Toggle (alternar)

```jsx
function Toggle() {
  const [activo, setActivo] = useState(false);
  
  return (
    <div>
      <p>Estado: {activo ? 'Activo' : 'Inactivo'}</p>
      <button onClick={() => setActivo(!activo)}>
        Alternar
      </button>
    </div>
  );
}
```

---

## Ejemplo: Input controlado

```jsx
function InputControlado() {
  const [texto, setTexto] = useState('');
  
  return (
    <div>
      <input 
        type="text"
        value={texto}
        onChange={(e) => setTexto(e.target.value)}
      />
      <p>Escribiste: {texto}</p>
      <p>Longitud: {texto.length}</p>
    </div>
  );
}
```

---

## Ejemplo: Contador completo

```jsx
function Contador() {
  const [count, setCount] = useState(0);
  
  return (
    <div className="contador">
      <h1>{count}</h1>
      <div className="botones">
        <button onClick={() => setCount(count - 1)}>-</button>
        <button onClick={() => setCount(0)}>Reset</button>
        <button onClick={() => setCount(count + 1)}>+</button>
      </div>
    </div>
  );
}
```

---

## Estado con números

```jsx
function CambioPrecio() {
  const [precio, setPrecio] = useState(100);
  
  return (
    <div>
      <h2>Precio: ${precio}</h2>
      <button onClick={() => setPrecio(precio + 10)}>+$10</button>
      <button onClick={() => setPrecio(precio - 10)}>-$10</button>
      <button onClick={() => setPrecio(precio * 1.1)}>+10%</button>
      <button onClick={() => setPrecio(100)}>Reset</button>
    </div>
  );
}
```

---

## Estado con strings

```jsx
function InputMensaje() {
  const [mensaje, setMensaje] = useState('Hola');
  
  return (
    <div>
      <input 
        value={mensaje}
        onChange={(e) => setMensaje(e.target.value)}
      />
      <p>Mensaje: {mensaje}</p>
      <p>Mayúsculas: {mensaje.toUpperCase()}</p>
      <p>Caracteres: {mensaje.length}</p>
    </div>
  );
}
```

---

## Estado con booleanos

```jsx
function MostrarOcultar() {
  const [visible, setVisible] = useState(true);
  
  return (
    <div>
      <button onClick={() => setVisible(!visible)}>
        {visible ? 'Ocultar' : 'Mostrar'}
      </button>
      
      {visible && (
        <div className="contenido">
          <p>Este contenido se puede ocultar</p>
        </div>
      )}
    </div>
  );
}
```

---

## Ejemplo: Like button

```jsx
function LikeButton() {
  const [likes, setLikes] = useState(0);
  const [liked, setLiked] = useState(false);
  
  const toggleLike = () => {
    if (liked) {
      setLikes(likes - 1);
      setLiked(false);
    } else {
      setLikes(likes + 1);
      setLiked(true);
    }
  };
  
  return (
    <button onClick={toggleLike} className={liked ? 'liked' : ''}>
      {liked ? '❤️' : '🤍'} {likes}
    </button>
  );
}
```

---

## Ejemplo: Temporizador simple

```jsx
function Temporizador() {
  const [segundos, setSegundos] = useState(0);
  
  return (
    <div>
      <h1>{segundos}s</h1>
      <button onClick={() => setSegundos(segundos + 1)}>
        +1 segundo
      </button>
      <button onClick={() => setSegundos(0)}>
        Reset
      </button>
    </div>
  );
}
```

---

## Ejemplo: Selector de color

```jsx
function SelectorColor() {
  const [color, setColor] = useState('red');
  
  return (
    <div>
      <div 
        style={{ 
          width: 200, 
          height: 200, 
          backgroundColor: color 
        }}
      />
      <div className="botones">
        <button onClick={() => setColor('red')}>Rojo</button>
        <button onClick={() => setColor('blue')}>Azul</button>
        <button onClick={() => setColor('green')}>Verde</button>
      </div>
    </div>
  );
}
```

---

## Nombrar variables de estado

### Convenciones
```jsx
// ✅ Nombre descriptivo
const [count, setCount] = useState(0);
const [isOpen, setIsOpen] = useState(false);
const [username, setUsername] = useState('');

// ✅ Booleanos: is, has, should
const [isLoading, setIsLoading] = useState(false);
const [hasError, setHasError] = useState(false);
const [shouldRender, setShouldRender] = useState(true);

// ❌ Nombres genéricos
const [data, setData] = useState(null);
const [flag, setFlag] = useState(false);
const [temp, setTemp] = useState(0);
```

---

## Estado inicial desde props

```jsx
function ContadorConInicial({ inicial = 0 }) {
  // Solo se usa 'inicial' en el primer render
  const [count, setCount] = useState(inicial);
  
  return (
    <div>
      <p>{count}</p>
      <button onClick={() => setCount(count + 1)}>+</button>
    </div>
  );
}

<ContadorConInicial inicial={10} />
```

⚠️ **Importante:** Si `inicial` cambia después, el estado NO se actualiza. Solo se usa en el **primer render**.

---

## Errores comunes

### ❌ Modificar estado directamente
```jsx
function Malo() {
  const [count, setCount] = useState(0);
  
  const incrementar = () => {
    count++;  // ❌ No modifiques el estado directamente
  };
}
```

### ✅ Usar la función set
```jsx
function Bueno() {
  const [count, setCount] = useState(0);
  
  const incrementar = () => {
    setCount(count + 1);  // ✅ Usa setCount
  };
}
```

---

### ❌ Llamar useState condicionalmente
```jsx
function Malo({ mostrar }) {
  if (mostrar) {
    const [count, setCount] = useState(0);  // ❌ No en condicional
  }
}
```

### ✅ Llamar useState en el nivel superior
```jsx
function Bueno({ mostrar }) {
  const [count, setCount] = useState(0);  // ✅ Siempre en el nivel superior
  
  if (!mostrar) return null;
  
  return <div>{count}</div>;
}
```

---

### ❌ Olvidar que setState es asíncrono
```jsx
function Malo() {
  const [count, setCount] = useState(0);
  
  const doble = () => {
    setCount(count + 1);
    setCount(count + 1);  // ❌ Usa el valor anterior
  };
}
```

### ✅ Usar forma funcional
```jsx
function Bueno() {
  const [count, setCount] = useState(0);
  
  const doble = () => {
    setCount(prev => prev + 1);
    setCount(prev => prev + 1);  // ✅ Usa el valor actualizado
  };
}
```

---

## Ejercicio 1: Contador con límites

Crea un contador que:
- No baje de 0
- No suba de 10

```jsx
function ContadorLimitado() {
  const [count, setCount] = useState(0);
  
  const incrementar = () => {
    // Tu código aquí
  };
  
  const decrementar = () => {
    // Tu código aquí
  };
  
  return (
    <div>
      <h1>{count}</h1>
      <button onClick={decrementar} disabled={count <= 0}>-</button>
      <button onClick={incrementar} disabled={count >= 10}>+</button>
    </div>
  );
}
```

<details>
<summary>Ver solución</summary>

```jsx
function ContadorLimitado() {
  const [count, setCount] = useState(0);
  
  const incrementar = () => {
    if (count < 10) {
      setCount(count + 1);
    }
  };
  
  const decrementar = () => {
    if (count > 0) {
      setCount(count - 1);
    }
  };
  
  return (
    <div>
      <h1>{count}</h1>
      <button onClick={decrementar} disabled={count <= 0}>-</button>
      <button onClick={incrementar} disabled={count >= 10}>+</button>
    </div>
  );
}
```
</details>

---

## Ejercicio 2: Input con validación

Crea un input que muestre si el texto tiene más de 5 caracteres:

```jsx
function InputValidado() {
  const [texto, setTexto] = useState('');
  
  return (
    <div>
      <input 
        value={texto}
        onChange={(e) => setTexto(e.target.value)}
      />
      {/* Muestra mensaje si length > 5 */}
    </div>
  );
}
```

<details>
<summary>Ver solución</summary>

```jsx
function InputValidado() {
  const [texto, setTexto] = useState('');
  const esValido = texto.length > 5;
  
  return (
    <div>
      <input 
        value={texto}
        onChange={(e) => setTexto(e.target.value)}
        className={esValido ? 'valido' : 'invalido'}
      />
      <p>{texto.length} caracteres</p>
      {esValido ? (
        <p className="exito">✅ Válido</p>
      ) : (
        <p className="error">❌ Mínimo 6 caracteres</p>
      )}
    </div>
  );
}
```
</details>

---

## Ejercicio 3: Alternar texto

Crea un botón que alterne entre "Día" y "Noche":

```jsx
function Alternador() {
  // Tu código aquí
  
  return (
    <div>
      <h1>{/* Muestra "Día" o "Noche" */}</h1>
      <button>{/* "Cambiar a Noche" o "Cambiar a Día" */}</button>
    </div>
  );
}
```

<details>
<summary>Ver solución</summary>

```jsx
function Alternador() {
  const [modo, setModo] = useState('Día');
  
  const alternar = () => {
    setModo(modo === 'Día' ? 'Noche' : 'Día');
  };
  
  return (
    <div className={modo === 'Día' ? 'dia' : 'noche'}>
      <h1>{modo}</h1>
      <button onClick={alternar}>
        Cambiar a {modo === 'Día' ? 'Noche' : 'Día'}
      </button>
    </div>
  );
}
```
</details>

---

## Ejercicio 4: Múltiples estados

Crea un formulario con:
- Input de nombre
- Input de edad
- Checkbox "Mayor de edad"
- Botón para resetear todo

```jsx
function FormularioMultiple() {
  // Tu código aquí
}
```

<details>
<summary>Ver solución</summary>

```jsx
function FormularioMultiple() {
  const [nombre, setNombre] = useState('');
  const [edad, setEdad] = useState(0);
  const [mayorEdad, setMayorEdad] = useState(false);
  
  const resetear = () => {
    setNombre('');
    setEdad(0);
    setMayorEdad(false);
  };
  
  return (
    <div>
      <div>
        <label>Nombre:</label>
        <input 
          value={nombre}
          onChange={(e) => setNombre(e.target.value)}
        />
      </div>
      
      <div>
        <label>Edad:</label>
        <input 
          type="number"
          value={edad}
          onChange={(e) => setEdad(Number(e.target.value))}
        />
      </div>
      
      <div>
        <label>
          <input 
            type="checkbox"
            checked={mayorEdad}
            onChange={(e) => setMayorEdad(e.target.checked)}
          />
          Mayor de edad
        </label>
      </div>
      
      <button onClick={resetear}>Resetear</button>
      
      <div className="resultado">
        <p>Nombre: {nombre}</p>
        <p>Edad: {edad}</p>
        <p>Mayor de edad: {mayorEdad ? 'Sí' : 'No'}</p>
      </div>
    </div>
  );
}
```
</details>

---

## Resumen

✅ **useState** añade estado a componentes funcionales  
✅ **Sintaxis:** `const [estado, setEstado] = useState(inicial)`  
✅ **Solo llama `useState` en el nivel superior**  
✅ **Actualiza con la función `set`**, nunca directamente  
✅ **Los cambios son asíncronos**  
✅ **Usa forma funcional** si depende del estado anterior  
✅ **Cada useState es independiente**

---

## Siguiente paso

Aprendiste a usar `useState`. Ahora verás cómo manejar **eventos** para actualizar el estado.

**Siguiente:** [Módulo 13 - Eventos en React](./13-eventos-react.md)

**Anterior:** [Módulo 11 - ¿Qué es el estado?](./11-que-es-estado.md)

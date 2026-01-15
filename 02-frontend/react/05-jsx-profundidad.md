# Módulo 05: JSX en Profundidad

## ¿Qué es JSX exactamente?

**JSX** = JavaScript XML

Es una **extensión de sintaxis** que permite escribir HTML dentro de JavaScript.

```jsx
const elemento = <h1>Hola Mundo</h1>;
```

---

## JSX se transforma en JavaScript

### Lo que escribes:

```jsx
const elemento = <h1 className="titulo">Hola {nombre}</h1>;
```

### Lo que JavaScript ejecuta:

```js
const elemento = React.createElement(
  'h1',
  { className: 'titulo' },
  'Hola ',
  nombre
);
```

Por eso **necesitas importar React** en archivos con JSX (aunque con React 17+ ya no es obligatorio con el nuevo JSX Transform).

---

## Reglas fundamentales de JSX

### 1. Solo un elemento raíz

```jsx
// ❌ Error - múltiples elementos raíz
function Componente() {
  return (
    <h1>Título</h1>
    <p>Párrafo</p>
  );
}

// ✅ Correcto - envuelto en div
function Componente() {
  return (
    <div>
      <h1>Título</h1>
      <p>Párrafo</p>
    </div>
  );
}

// ✅ Correcto - Fragment sin div extra
function Componente() {
  return (
    <>
      <h1>Título</h1>
      <p>Párrafo</p>
    </>
  );
}

// ✅ Correcto - Fragment explícito
import { Fragment } from 'react';

function Componente() {
  return (
    <Fragment>
      <h1>Título</h1>
      <p>Párrafo</p>
    </Fragment>
  );
}
```

**¿Cuándo usar Fragment?**
- No quieres agregar un `<div>` extra al DOM
- Tienes problemas de CSS por el contenedor extra

---

### 2. Todas las etiquetas deben cerrarse

```jsx
// ❌ Error - etiquetas sin cerrar
<img src="foto.jpg">
<input type="text">
<br>

// ✅ Correcto - autocerradas
<img src="foto.jpg" />
<input type="text" />
<br />
```

---

### 3. `className` en vez de `class`

```jsx
// ❌ Error
<div class="container"></div>

// ✅ Correcto
<div className="container"></div>
```

**¿Por qué?** `class` es palabra reservada en JavaScript.

---

### 4. Atributos en camelCase

```jsx
// HTML tradicional
<div onclick="handler()">
<input maxlength="10">
<label tabindex="0">

// JSX (camelCase)
<div onClick={handler}>
<input maxLength={10}>
<label tabIndex={0}>
```

**Excepciones:** Atributos HTML nativos como `data-*` y `aria-*` se mantienen con guiones:

```jsx
<div data-id="123" aria-label="Cerrar">
```

---

### 5. Comentarios especiales

```jsx
function Componente() {
  return (
    <div>
      {/* Este es un comentario en JSX */}
      <h1>Título</h1>
      
      {/* 
        Comentario
        multilínea
      */}
      <p>Texto</p>
    </div>
  );
}
```

**No funciona:**
```jsx
<div>
  // Esto NO es un comentario
  <!-- Esto tampoco -->
</div>
```

---

## Expresiones JavaScript en JSX

Usa **llaves `{}`** para inyectar JavaScript:

### Variables

```jsx
function Saludo() {
  const nombre = "Ana";
  const edad = 25;
  
  return (
    <div>
      <h1>Hola, {nombre}</h1>
      <p>Tienes {edad} años</p>
    </div>
  );
}
```

### Operaciones matemáticas

```jsx
function Precio() {
  const precioBase = 100;
  const descuento = 0.2;
  
  return (
    <div>
      <p>Precio: ${precioBase}</p>
      <p>Descuento: {descuento * 100}%</p>
      <p>Total: ${precioBase - precioBase * descuento}</p>
    </div>
  );
}
```

### Llamadas a funciones

```jsx
function Card() {
  const formatFecha = (fecha) => {
    return fecha.toLocaleDateString('es-ES');
  };
  
  return (
    <p>Fecha: {formatFecha(new Date())}</p>
  );
}
```

### Métodos de arrays

```jsx
function Lista() {
  const numeros = [1, 2, 3, 4, 5];
  
  return (
    <p>Suma: {numeros.reduce((a, b) => a + b, 0)}</p>
  );
}
```

---

## Renderizado condicional

### 1. Operador ternario

```jsx
function Mensaje({ esError }) {
  return (
    <div>
      {esError ? (
        <p className="error">❌ Error</p>
      ) : (
        <p className="success">✅ Éxito</p>
      )}
    </div>
  );
}
```

### 2. Operador && (AND lógico)

```jsx
function Notificacion({ hayMensajes, cantidad }) {
  return (
    <div>
      {hayMensajes && (
        <span className="badge">{cantidad} nuevos</span>
      )}
    </div>
  );
}
```

**Cuidado con valores falsy:**
```jsx
function Contador({ total }) {
  // ❌ Si total es 0, muestra "0" en pantalla
  return <div>{total && <span>{total} items</span>}</div>;
  
  // ✅ Compara explícitamente
  return <div>{total > 0 && <span>{total} items</span>}</div>;
}
```

### 3. If temprano (early return)

```jsx
function UserProfile({ usuario }) {
  if (!usuario) {
    return <p>Cargando...</p>;
  }
  
  if (usuario.banned) {
    return <p>Usuario bloqueado</p>;
  }
  
  return (
    <div>
      <h1>{usuario.nombre}</h1>
      <p>{usuario.email}</p>
    </div>
  );
}
```

### 4. Variables condicionales

```jsx
function Estado({ conectado }) {
  let mensaje;
  let icono;
  
  if (conectado) {
    mensaje = "En línea";
    icono = "🟢";
  } else {
    mensaje = "Desconectado";
    icono = "🔴";
  }
  
  return (
    <div>
      {icono} {mensaje}
    </div>
  );
}
```

### 5. Switch con objetos

```jsx
function TipoAlerta({ tipo }) {
  const alertas = {
    success: { icono: "✅", clase: "success" },
    error: { icono: "❌", clase: "error" },
    warning: { icono: "⚠️", clase: "warning" },
    info: { icono: "ℹ️", clase: "info" }
  };
  
  const alerta = alertas[tipo] || alertas.info;
  
  return (
    <div className={alerta.clase}>
      {alerta.icono} Mensaje
    </div>
  );
}
```

---

## Renderizar listas

### Con map()

```jsx
function ListaTareas() {
  const tareas = ["Aprender React", "Hacer ejercicio", "Leer"];
  
  return (
    <ul>
      {tareas.map((tarea, index) => (
        <li key={index}>{tarea}</li>
      ))}
    </ul>
  );
}
```

### Con objetos

```jsx
function ListaUsuarios() {
  const usuarios = [
    { id: 1, nombre: "Ana", rol: "Admin" },
    { id: 2, nombre: "Luis", rol: "Usuario" },
    { id: 3, nombre: "María", rol: "Usuario" }
  ];
  
  return (
    <ul>
      {usuarios.map((usuario) => (
        <li key={usuario.id}>
          {usuario.nombre} - {usuario.rol}
        </li>
      ))}
    </ul>
  );
}
```

### ¿Por qué necesitamos `key`?

```jsx
// ❌ Sin key - React no sabe qué cambió
{items.map(item => <li>{item}</li>)}

// ✅ Con key - React identifica cada elemento
{items.map(item => <li key={item.id}>{item}</li>)}
```

**Reglas de `key`:**
- Debe ser **única** entre hermanos
- Debe ser **estable** (no cambiar entre renders)
- **Evita usar `index`** si el orden puede cambiar

```jsx
// ❌ Malo - el índice cambia si reordenas
{items.map((item, index) => <li key={index}>{item}</li>)}

// ✅ Bueno - ID único del backend
{items.map(item => <li key={item.id}>{item.nombre}</li>)}

// ✅ Aceptable si no hay ID y el orden es fijo
{items.map((item, index) => <li key={index}>{item}</li>)}
```

---

## Atributos dinámicos

### className dinámico

```jsx
function Boton({ activo }) {
  return (
    <button className={activo ? "btn-active" : "btn-inactive"}>
      Click
    </button>
  );
}

// Con múltiples clases
function Card({ destacado, grande }) {
  const clases = `card ${destacado ? 'destacado' : ''} ${grande ? 'grande' : ''}`;
  
  return <div className={clases}>Card</div>;
}

// Con template literals
function Elemento({ tipo }) {
  return <div className={`elemento elemento-${tipo}`}>Texto</div>;
}
```

### style dinámico

```jsx
function BarraProgreso({ porcentaje }) {
  const estilos = {
    width: `${porcentaje}%`,
    backgroundColor: porcentaje > 50 ? 'green' : 'red'
  };
  
  return (
    <div className="barra">
      <div style={estilos} className="progreso"></div>
    </div>
  );
}
```

### src, href y otros atributos

```jsx
function Avatar({ usuario }) {
  return (
    <img 
      src={usuario.avatar || '/default-avatar.png'}
      alt={usuario.nombre}
    />
  );
}

function Enlace({ url, texto }) {
  return <a href={url} target="_blank" rel="noopener noreferrer">{texto}</a>;
}
```

---

## Estilos en línea

Los estilos en línea son un **objeto JavaScript**:

```jsx
function Titulo() {
  const estilos = {
    color: 'blue',
    fontSize: '24px',  // camelCase, no 'font-size'
    marginTop: '10px',
    backgroundColor: '#f0f0f0'
  };
  
  return <h1 style={estilos}>Título</h1>;
}

// Inline directo
function Texto() {
  return <p style={{ color: 'red', fontWeight: 'bold' }}>Texto</p>;
}
```

**Nota:** Doble llave `{{}}`
- Primera `{}` → inyectar JS
- Segunda `{}` → objeto JavaScript

---

## Children

Los componentes pueden recibir contenido entre etiquetas:

```jsx
function Card({ children }) {
  return (
    <div className="card">
      {children}
    </div>
  );
}

// Uso:
<Card>
  <h2>Título</h2>
  <p>Contenido del card</p>
</Card>
```

### Ejemplo: Layout

```jsx
function Layout({ children }) {
  return (
    <div className="layout">
      <header>Header</header>
      <main>{children}</main>
      <footer>Footer</footer>
    </div>
  );
}

// Uso:
<Layout>
  <h1>Bienvenido</h1>
  <p>Contenido de la página</p>
</Layout>
```

---

## Fragmentos

### Fragment corto `<>...</>`

```jsx
function Lista() {
  return (
    <>
      <li>Item 1</li>
      <li>Item 2</li>
    </>
  );
}
```

### Fragment con key

```jsx
import { Fragment } from 'react';

function Tabla() {
  const filas = [
    { id: 1, nombre: "Ana", edad: 25 },
    { id: 2, nombre: "Luis", edad: 30 }
  ];
  
  return (
    <table>
      <tbody>
        {filas.map(fila => (
          <Fragment key={fila.id}>
            <tr>
              <td>{fila.nombre}</td>
              <td>{fila.edad}</td>
            </tr>
          </Fragment>
        ))}
      </tbody>
    </table>
  );
}
```

---

## Prevenir XSS (inyección de scripts)

React **escapa automáticamente** el contenido:

```jsx
const userInput = "<img src=x onerror='alert(1)'>";

// ✅ Seguro - React escapa el HTML
<div>{userInput}</div>
// Renderiza: &lt;img src=x onerror='alert(1)'&gt;
```

### Insertar HTML puro (⚠️ peligroso)

```jsx
function HTMLRaw({ html }) {
  return <div dangerouslySetInnerHTML={{ __html: html }} />;
}
```

**Solo úsalo si:**
- El HTML viene de fuente confiable
- Ya fue sanitizado (con librería como DOMPurify)

---

## JSX vs JavaScript puro

### JSX:
```jsx
const elemento = (
  <div className="container">
    <h1>Hola {nombre}</h1>
    <button onClick={handleClick}>Click</button>
  </div>
);
```

### JavaScript equivalente:
```js
const elemento = React.createElement(
  'div',
  { className: 'container' },
  React.createElement('h1', null, 'Hola ', nombre),
  React.createElement('button', { onClick: handleClick }, 'Click')
);
```

JSX es **mucho más legible**.

---

## Ejercicio práctico: Dashboard

```jsx
function Dashboard() {
  const usuario = {
    nombre: "Ana García",
    rol: "Admin",
    avatar: "/ana.jpg",
    stats: {
      ventas: 1250,
      clientes: 45,
      productos: 120
    }
  };
  
  const esAdmin = usuario.rol === "Admin";
  
  return (
    <div className="dashboard">
      {/* Header */}
      <header>
        <img src={usuario.avatar} alt={usuario.nombre} />
        <h1>{usuario.nombre}</h1>
        <span className={esAdmin ? "badge-admin" : "badge-user"}>
          {usuario.rol}
        </span>
      </header>
      
      {/* Estadísticas */}
      <div className="stats">
        <StatCard titulo="Ventas" valor={usuario.stats.ventas} icono="💰" />
        <StatCard titulo="Clientes" valor={usuario.stats.clientes} icono="👥" />
        <StatCard titulo="Productos" valor={usuario.stats.productos} icono="📦" />
      </div>
      
      {/* Panel de admin */}
      {esAdmin && (
        <div className="admin-panel">
          <h2>Panel de Administración</h2>
          <button>Gestionar Usuarios</button>
          <button>Ver Reportes</button>
        </div>
      )}
    </div>
  );
}

function StatCard({ titulo, valor, icono }) {
  const esAlto = valor > 100;
  
  return (
    <div className={`stat-card ${esAlto ? 'alto' : ''}`}>
      <span className="icono">{icono}</span>
      <h3>{titulo}</h3>
      <p className="valor" style={{ color: esAlto ? 'green' : 'blue' }}>
        {valor.toLocaleString()}
      </p>
    </div>
  );
}
```

---

## Errores comunes

### 1. Olvidar llaves en expresiones

```jsx
// ❌ Error - lo imprime literalmente
<p>Total: total</p>

// ✅ Correcto
<p>Total: {total}</p>
```

### 2. Olvidar return en map

```jsx
// ❌ Error - no devuelve nada
{items.map(item => {
  <li>{item}</li>
})}

// ✅ Correcto - return implícito
{items.map(item => (
  <li>{item}</li>
))}

// ✅ Correcto - return explícito
{items.map(item => {
  return <li>{item}</li>;
})}
```

### 3. Usar if dentro de JSX

```jsx
// ❌ Error - if no funciona en JSX
<div>
  {if (condicion) { <p>Texto</p> }}
</div>

// ✅ Correcto - operador ternario
<div>
  {condicion ? <p>Texto</p> : null}
</div>

// ✅ Correcto - && operator
<div>
  {condicion && <p>Texto</p>}
</div>
```

### 4. Olvidar key en listas

```jsx
// ❌ Warning en consola
{items.map(item => <li>{item}</li>)}

// ✅ Correcto
{items.map(item => <li key={item.id}>{item}</li>)}
```

---

## Resumen

✅ **JSX es JavaScript + XML**  
✅ **Se transforma en `React.createElement()`**  
✅ **Solo un elemento raíz (usa Fragment si es necesario)**  
✅ **Llaves `{}` inyectan JavaScript**  
✅ **`className`, `onClick`, camelCase**  
✅ **`key` obligatorio en listas**  
✅ **Condicionales con ternario o `&&`**  
✅ **React escapa contenido automáticamente**

---

## Siguiente paso

Dominas JSX. Ahora aprenderás sobre la **anatomía completa de un componente**.

**Siguiente:** [Módulo 06 - Anatomía de un componente](./06-anatomia-componente.md)

**Anterior:** [Módulo 04 - Primer componente](./04-primer-componente.md)

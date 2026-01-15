# Módulo 04: Tu Primer Componente React

## ¿Qué es un componente?

Un **componente** es una pieza reutilizable de interfaz.

### Analogía: Bloques de LEGO

```
App (casa completa)
├── Header (techo)
├── Sidebar (ventana izquierda)
├── Content (puerta y paredes)
│   ├── Card (ladrillo individual)
│   ├── Card
│   └── Card
└── Footer (cimientos)
```

Cada bloque es independiente y reutilizable.

---

## Anatomía de un componente

Un componente React es una **función de JavaScript** que devuelve **JSX**.

```jsx
function MiComponente() {
  return <h1>¡Hola Mundo!</h1>;
}
```

Eso es todo. Un componente básico tiene:
1. **Función** (con nombre en PascalCase)
2. **return** con JSX
3. **export** para usarlo en otros archivos

---

## Tu primer componente: Saludo

Crea `src/components/Saludo.jsx`:

```jsx
function Saludo() {
  return <h1>¡Hola desde React!</h1>;
}

export default Saludo;
```

### Usarlo en App.jsx

```jsx
import Saludo from './components/Saludo';

function App() {
  return (
    <div>
      <Saludo />
    </div>
  );
}

export default App;
```

### ¿Qué sucede?

1. `App` importa `Saludo`
2. `App` lo usa como `<Saludo />`
3. React renderiza el JSX de `Saludo`
4. Resultado: `<h1>¡Hola desde React!</h1>`

---

## Reglas de los componentes

### 1. Nombre en PascalCase

```jsx
// ✅ Correcto
function MiComponente() {}
function UserProfile() {}

// ❌ Incorrecto
function miComponente() {}  // camelCase
function user_profile() {}  // snake_case
```

**¿Por qué?** React distingue componentes de HTML por mayúscula inicial:
- `<div>` → HTML nativo
- `<MiComponente>` → Componente React

### 2. Debe devolver JSX

```jsx
// ✅ Correcto
function Componente() {
  return <div>Hola</div>;
}

// ❌ Incorrecto - no devuelve nada
function Componente() {
  console.log("Hola");
}
```

### 3. Solo un elemento raíz

```jsx
// ❌ Incorrecto - múltiples elementos raíz
function Componente() {
  return (
    <h1>Título</h1>
    <p>Párrafo</p>
  );
}

// ✅ Correcto - un div contenedor
function Componente() {
  return (
    <div>
      <h1>Título</h1>
      <p>Párrafo</p>
    </div>
  );
}

// ✅ Correcto - Fragment (sin div extra)
function Componente() {
  return (
    <>
      <h1>Título</h1>
      <p>Párrafo</p>
    </>
  );
}
```

---

## ¿Qué es JSX?

**JSX = JavaScript + XML**

Parece HTML pero es JavaScript:

```jsx
const elemento = <h1>Hola Mundo</h1>;
```

### Babel lo transforma

JSX se transforma en JavaScript puro:

```jsx
// Tú escribes:
const elemento = <h1>Hola Mundo</h1>;

// Babel lo convierte en:
const elemento = React.createElement('h1', null, 'Hola Mundo');
```

Por eso **debes importar React** (aunque no lo uses directamente):

```jsx
import React from 'react'; // Necesario para JSX
```

---

## Componentes con lógica

Los componentes pueden tener JavaScript antes del `return`:

```jsx
function Saludo() {
  const nombre = "Ana";
  const edad = 25;
  const esAdulto = edad >= 18;
  
  return (
    <div>
      <h1>Hola, {nombre}</h1>
      <p>Tienes {edad} años</p>
      <p>{esAdulto ? "Eres mayor de edad" : "Eres menor de edad"}</p>
    </div>
  );
}
```

**Llaves `{}` inyectan JavaScript en JSX:**
- `{nombre}` → Imprime la variable
- `{edad}` → Imprime el número
- `{esAdulto ? "..." : "..."}` → Operador ternario

---

## Tipos de componentes

### 1. Componente de función (moderno)

```jsx
function Boton() {
  return <button>Click</button>;
}
```

### 2. Arrow function (alternativa)

```jsx
const Boton = () => {
  return <button>Click</button>;
};
```

### 3. Arrow function con return implícito

```jsx
const Boton = () => <button>Click</button>;
```

**Recomendación:** Usa función normal para componentes. Es más claro.

---

## Componentes con parámetros (props)

Los componentes pueden recibir datos:

```jsx
function Saludo(props) {
  return <h1>Hola, {props.nombre}</h1>;
}

// Uso:
<Saludo nombre="Carlos" />
// Renderiza: <h1>Hola, Carlos</h1>
```

**Destructuring (más común):**

```jsx
function Saludo({ nombre }) {
  return <h1>Hola, {nombre}</h1>;
}
```

---

## Ejemplo completo: Card de producto

```jsx
function ProductCard({ nombre, precio, imagen }) {
  return (
    <div className="card">
      <img src={imagen} alt={nombre} />
      <h3>{nombre}</h3>
      <p>${precio}</p>
      <button>Agregar al carrito</button>
    </div>
  );
}

// Uso:
<ProductCard 
  nombre="Laptop"
  precio={1200}
  imagen="/laptop.jpg"
/>
```

---

## Composición de componentes

Los componentes se pueden anidar:

```jsx
function Header() {
  return (
    <header>
      <Logo />
      <Nav />
    </header>
  );
}

function Logo() {
  return <h1>Mi Sitio</h1>;
}

function Nav() {
  return (
    <nav>
      <a href="/">Inicio</a>
      <a href="/about">Acerca</a>
    </nav>
  );
}

function App() {
  return (
    <div>
      <Header />
      <main>Contenido principal</main>
    </div>
  );
}
```

---

## Diferencias con HTML

### 1. `className` en vez de `class`

```jsx
// ❌ Incorrecto
<div class="container">

// ✅ Correcto
<div className="container">
```

**¿Por qué?** `class` es palabra reservada en JavaScript.

### 2. Atributos en camelCase

```jsx
// HTML
<button onclick="handleClick()">

// JSX
<button onClick={handleClick}>
```

Otros ejemplos:
- `tabindex` → `tabIndex`
- `maxlength` → `maxLength`
- `readonly` → `readOnly`

### 3. Etiquetas vacías se autocierran

```jsx
// ❌ Incorrecto
<img src="foto.jpg">
<input type="text">

// ✅ Correcto
<img src="foto.jpg" />
<input type="text" />
```

### 4. `style` es un objeto

```jsx
// HTML
<div style="color: red; font-size: 20px;">

// JSX
<div style={{ color: 'red', fontSize: '20px' }}>
```

**Nota:** Doble llave `{{}}`:
- Primera `{}` → Inyectar JS
- Segunda `{}` → Objeto JavaScript

---

## Ejemplo práctico: Tarjeta de usuario

```jsx
function UserCard({ nombre, rol, avatar, activo }) {
  return (
    <div className="user-card">
      <img 
        src={avatar} 
        alt={nombre}
        className="avatar"
      />
      <div className="info">
        <h3>{nombre}</h3>
        <p className="rol">{rol}</p>
        {activo && <span className="badge">En línea</span>}
      </div>
    </div>
  );
}

// Uso:
function App() {
  return (
    <div>
      <UserCard 
        nombre="María García"
        rol="Desarrolladora"
        avatar="/maria.jpg"
        activo={true}
      />
      <UserCard 
        nombre="Juan Pérez"
        rol="Diseñador"
        avatar="/juan.jpg"
        activo={false}
      />
    </div>
  );
}
```

---

## Renderizado condicional

### Con operador ternario

```jsx
function Mensaje({ esError }) {
  return (
    <div className={esError ? "error" : "success"}>
      {esError ? "❌ Error" : "✅ Éxito"}
    </div>
  );
}
```

### Con &&

```jsx
function Notificacion({ hayMensajes, cantidad }) {
  return (
    <div>
      {hayMensajes && <span>Tienes {cantidad} mensajes</span>}
    </div>
  );
}
```

### Con if temprano

```jsx
function Saludo({ usuario }) {
  if (!usuario) {
    return <p>Por favor inicia sesión</p>;
  }
  
  return <h1>Bienvenido, {usuario.nombre}</h1>;
}
```

---

## Listas con map

Renderizar arrays:

```jsx
function ListaTareas() {
  const tareas = [
    "Aprender React",
    "Hacer ejercicio",
    "Leer un libro"
  ];
  
  return (
    <ul>
      {tareas.map((tarea, index) => (
        <li key={index}>{tarea}</li>
      ))}
    </ul>
  );
}
```

**`key` es obligatorio** para que React identifique cada elemento.

---

## Eventos

Manejar clicks, cambios, etc:

```jsx
function Boton() {
  const handleClick = () => {
    alert("¡Botón presionado!");
  };
  
  return (
    <button onClick={handleClick}>
      Presióname
    </button>
  );
}
```

### Eventos comunes

```jsx
<button onClick={handleClick}>Click</button>
<input onChange={handleChange} />
<form onSubmit={handleSubmit}>
<input onFocus={handleFocus} />
<input onBlur={handleBlur} />
<div onMouseEnter={handleHover} />
```

---

## Ejercicio práctico: Card de película

Crea `src/components/MovieCard.jsx`:

```jsx
function MovieCard({ titulo, año, director, poster }) {
  return (
    <div className="movie-card">
      <img src={poster} alt={titulo} />
      <h2>{titulo}</h2>
      <p>Año: {año}</p>
      <p>Director: {director}</p>
      <button onClick={() => alert(`Ver ${titulo}`)}>
        Ver detalles
      </button>
    </div>
  );
}

export default MovieCard;
```

Úsalo en `App.jsx`:

```jsx
import MovieCard from './components/MovieCard';

function App() {
  return (
    <div>
      <h1>Mis Películas</h1>
      <MovieCard 
        titulo="Inception"
        año={2010}
        director="Christopher Nolan"
        poster="/inception.jpg"
      />
      <MovieCard 
        titulo="Interstellar"
        año={2014}
        director="Christopher Nolan"
        poster="/interstellar.jpg"
      />
    </div>
  );
}

export default App;
```

---

## Buenas prácticas

### ✅ Un componente por archivo

```
components/
├── Header.jsx
├── Footer.jsx
└── Button.jsx
```

### ✅ Nombres descriptivos

```jsx
// ✅ Bueno
function ProductCard() {}
function UserProfile() {}

// ❌ Malo
function Card() {}  // ¿Card de qué?
function PC() {}    // Confuso
```

### ✅ Componentes pequeños y enfocados

```jsx
// ✅ Bueno - cada uno hace una cosa
function Avatar({ src, alt }) {
  return <img src={src} alt={alt} className="avatar" />;
}

function UserInfo({ nombre, email }) {
  return (
    <div>
      <h3>{nombre}</h3>
      <p>{email}</p>
    </div>
  );
}

function UserCard({ usuario }) {
  return (
    <div>
      <Avatar src={usuario.avatar} alt={usuario.nombre} />
      <UserInfo nombre={usuario.nombre} email={usuario.email} />
    </div>
  );
}
```

### ✅ Extraer lógica repetida

```jsx
// ❌ Repetido
function Card1() {
  return <div className="card shadow rounded">...</div>;
}
function Card2() {
  return <div className="card shadow rounded">...</div>;
}

// ✅ Reutilizable
function Card({ children }) {
  return <div className="card shadow rounded">{children}</div>;
}
```

---

## Errores comunes

### 1. Olvidar return

```jsx
// ❌ No devuelve nada
function Componente() {
  <div>Hola</div>
}

// ✅ Correcto
function Componente() {
  return <div>Hola</div>;
}
```

### 2. Múltiples elementos raíz

```jsx
// ❌ Error
function Componente() {
  return (
    <h1>Título</h1>
    <p>Texto</p>
  );
}

// ✅ Envolver en <> o <div>
function Componente() {
  return (
    <>
      <h1>Título</h1>
      <p>Texto</p>
    </>
  );
}
```

### 3. Usar `class` en vez de `className`

```jsx
// ❌ Error
<div class="container">

// ✅ Correcto
<div className="container">
```

### 4. Evento sin llaves

```jsx
// ❌ Error - ejecuta la función inmediatamente
<button onClick={alert('Hola')}>

// ✅ Correcto - pasa la función
<button onClick={() => alert('Hola')}>
```

---

## Resumen

✅ **Componente = función que devuelve JSX**  
✅ **Nombre en PascalCase**  
✅ **JSX parece HTML pero es JavaScript**  
✅ **`{}` inyecta JavaScript en JSX**  
✅ **Componentes reciben props**  
✅ **Se pueden anidar y reutilizar**

---

## Siguiente paso

Dominas los componentes básicos. Ahora profundizarás en **todas las reglas de JSX**.

**Siguiente:** [Módulo 05 - JSX en profundidad](./05-jsx-profundidad.md)

**Anterior:** [Módulo 03 - Estructura de proyecto](./03-estructura-proyecto.md)

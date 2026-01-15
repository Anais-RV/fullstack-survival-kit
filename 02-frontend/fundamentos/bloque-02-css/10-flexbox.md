# Flexbox

**Flexbox = La forma moderna de organizar elementos en CSS.**

Antes de Flexbox, alinear elementos verticalmente o crear layouts flexibles era un dolor de cabeza. Flexbox lo hace simple e intuitivo.

---

## ¿Qué vas a aprender?

- Qué es Flexbox y cuándo usarlo
- Contenedor flex (`display: flex`)
- Dirección: horizontal vs vertical
- Alineación: horizontal (`justify-content`) y vertical (`align-items`)
- Distribución de espacio
- Propiedades de los ítems flex

## Por qué es útil

**Analogía:** Imagina que organizas libros en un estante:

- Sin Flexbox: Colocas cada libro con coordenadas precisas (x, y). Si añades uno más, tienes que recalcular todo.
- Con Flexbox: Le dices al estante "organiza estos libros uno al lado del otro, centrados, con espacio igual entre ellos". El estante lo hace automáticamente.

**Flexbox resuelve:**
- Centrar elementos vertical y horizontalmente
- Distribuir espacio automáticamente
- Crear layouts que se adaptan al contenido
- Ordenar elementos sin cambiar el HTML

---

## Concepto básico: Contenedor e ítems

```html
<div class="contenedor">  ← CONTENEDOR FLEX
    <div>Ítem 1</div>     ← ÍTEM FLEX
    <div>Ítem 2</div>     ← ÍTEM FLEX
    <div>Ítem 3</div>     ← ÍTEM FLEX
</div>
```

```css
.contenedor {
    display: flex;  /* Esto convierte al contenedor en un flex container */
}
```

**Resultado:** Los ítems se alinean en horizontal (por defecto).

```
┌────────────────────────────────┐
│ ┌─────┐ ┌─────┐ ┌─────┐       │
│ │Ítem1│ │Ítem2│ │Ítem3│       │
│ └─────┘ └─────┘ └─────┘       │
└────────────────────────────────┘
```

---

## Dirección: `flex-direction`

Controla si los ítems van en fila (horizontal) o columna (vertical).

```css
.contenedor {
    display: flex;
    flex-direction: row;          /* Por defecto: horizontal, izq a der */
    flex-direction: row-reverse;  /* Horizontal, der a izq */
    flex-direction: column;       /* Vertical, arriba a abajo */
    flex-direction: column-reverse; /* Vertical, abajo a arriba */
}
```

### `row` (por defecto)

```css
flex-direction: row;
```

```
┌─────┐ ┌─────┐ ┌─────┐
│  1  │ │  2  │ │  3  │
└─────┘ └─────┘ └─────┘
```

### `column`

```css
flex-direction: column;
```

```
┌─────┐
│  1  │
└─────┘
┌─────┐
│  2  │
└─────┘
┌─────┐
│  3  │
└─────┘
```

---

## Ejes principales y transversales

Flexbox trabaja con dos ejes:

### Eje principal (main axis)

La dirección en la que fluyen los ítems.

- `flex-direction: row` → Eje principal = horizontal
- `flex-direction: column` → Eje principal = vertical

### Eje transversal (cross axis)

Perpendicular al eje principal.

- Si eje principal = horizontal → Eje transversal = vertical
- Si eje principal = vertical → Eje transversal = horizontal

**Esto es importante para entender `justify-content` y `align-items`.**

---

## `justify-content`: Alineación en el eje principal

Controla cómo se distribuyen los ítems **a lo largo** del eje principal.

```css
.contenedor {
    display: flex;
    justify-content: flex-start;    /* Inicio (por defecto) */
    justify-content: flex-end;      /* Final */
    justify-content: center;        /* Centro */
    justify-content: space-between; /* Espacio entre ítems */
    justify-content: space-around;  /* Espacio alrededor de ítems */
    justify-content: space-evenly;  /* Espacio uniforme */
}
```

### Ejemplos visuales (con `flex-direction: row`)

#### `flex-start` (por defecto)

```
┌────────────────────────────────┐
│ ┌───┐ ┌───┐ ┌───┐             │
│ │ 1 │ │ 2 │ │ 3 │             │
│ └───┘ └───┘ └───┘             │
└────────────────────────────────┘
```

#### `flex-end`

```
┌────────────────────────────────┐
│             ┌───┐ ┌───┐ ┌───┐ │
│             │ 1 │ │ 2 │ │ 3 │ │
│             └───┘ └───┘ └───┘ │
└────────────────────────────────┘
```

#### `center`

```
┌────────────────────────────────┐
│       ┌───┐ ┌───┐ ┌───┐       │
│       │ 1 │ │ 2 │ │ 3 │       │
│       └───┘ └───┘ └───┘       │
└────────────────────────────────┘
```

#### `space-between`

```
┌────────────────────────────────┐
│ ┌───┐      ┌───┐      ┌───┐  │
│ │ 1 │      │ 2 │      │ 3 │  │
│ └───┘      └───┘      └───┘  │
└────────────────────────────────┘
```

Espacio entre ítems, pero no en los extremos.

#### `space-around`

```
┌────────────────────────────────┐
│   ┌───┐    ┌───┐    ┌───┐    │
│   │ 1 │    │ 2 │    │ 3 │    │
│   └───┘    └───┘    └───┘    │
└────────────────────────────────┘
```

Espacio alrededor de cada ítem (los extremos tienen la mitad del espacio entre ítems).

#### `space-evenly`

```
┌────────────────────────────────┐
│    ┌───┐   ┌───┐   ┌───┐     │
│    │ 1 │   │ 2 │   │ 3 │     │
│    └───┘   └───┘   └───┘     │
└────────────────────────────────┘
```

Espacio uniforme en todos lados.

---

## `align-items`: Alineación en el eje transversal

Controla cómo se alinean los ítems **perpendicularmente** al eje principal.

```css
.contenedor {
    display: flex;
    align-items: stretch;      /* Por defecto: estiran al alto del contenedor */
    align-items: flex-start;   /* Inicio */
    align-items: flex-end;     /* Final */
    align-items: center;       /* Centro (EL MÁS USADO) */
    align-items: baseline;     /* Línea base del texto */
}
```

### Ejemplos visuales (con `flex-direction: row`)

#### `stretch` (por defecto)

```
┌────────────────────────────────┐
│ ┌─────┐ ┌─────┐ ┌─────┐       │
│ │  1  │ │  2  │ │  3  │       │
│ │     │ │     │ │     │       │
│ └─────┘ └─────┘ └─────┘       │
└────────────────────────────────┘
```

Los ítems se estiran para llenar la altura.

#### `center`

```
┌────────────────────────────────┐
│                                │
│ ┌─────┐ ┌─────┐ ┌─────┐       │
│ │  1  │ │  2  │ │  3  │       │
│ └─────┘ └─────┘ └─────┘       │
│                                │
└────────────────────────────────┘
```

Centrados verticalmente.

#### `flex-start`

```
┌────────────────────────────────┐
│ ┌─────┐ ┌─────┐ ┌─────┐       │
│ │  1  │ │  2  │ │  3  │       │
│ └─────┘ └─────┘ └─────┘       │
│                                │
│                                │
└────────────────────────────────┘
```

Alineados arriba.

#### `flex-end`

```
┌────────────────────────────────┐
│                                │
│                                │
│ ┌─────┐ ┌─────┐ ┌─────┐       │
│ │  1  │ │  2  │ │  3  │       │
│ └─────┘ └─────┘ └─────┘       │
└────────────────────────────────┘
```

Alineados abajo.

---

## Centrar perfectamente (horizontal y vertical)

**El truco más útil de Flexbox:**

```css
.contenedor {
    display: flex;
    justify-content: center;  /* Centrado horizontal */
    align-items: center;      /* Centrado vertical */
    height: 100vh;            /* Altura completa de la ventana */
}
```

```html
<div class="contenedor">
    <div>¡Estoy perfectamente centrado!</div>
</div>
```

**Resultado:** El contenido está centrado horizontal y verticalmente.

---

## `flex-wrap`: Salto de línea

Por defecto, Flexbox intenta meter todos los ítems en una línea, aunque se encojan.

```css
.contenedor {
    display: flex;
    flex-wrap: nowrap;  /* Por defecto: no saltan de línea */
    flex-wrap: wrap;    /* Saltan a la siguiente línea si no caben */
    flex-wrap: wrap-reverse; /* Saltan en orden inverso */
}
```

### Sin `wrap` (por defecto)

```html
<div style="display: flex; width: 300px;">
    <div style="width: 200px; background: lightblue;">1</div>
    <div style="width: 200px; background: lightcoral;">2</div>
    <div style="width: 200px; background: lightgreen;">3</div>
</div>
```

Los ítems se encogen para caber (o se desbordan).

### Con `wrap`

```css
.contenedor {
    display: flex;
    flex-wrap: wrap;
}
```

Si no caben, saltan a la siguiente línea:

```
┌─────┐ ┌─────┐
│  1  │ │  2  │
└─────┘ └─────┘
┌─────┐
│  3  │
└─────┘
```

---

## `gap`: Espaciado entre ítems

La forma moderna de añadir espacio entre ítems (sin márgenes).

```css
.contenedor {
    display: flex;
    gap: 20px;  /* 20px de espacio entre todos los ítems */
}
```

```css
/* Espacio diferente vertical y horizontal */
.contenedor {
    display: flex;
    flex-wrap: wrap;
    gap: 10px 20px;  /* 10px vertical, 20px horizontal */
}
```

**Ventaja sobre `margin`:**
- No añade espacio en los extremos
- Más limpio y predecible

---

## Propiedades de los ítems flex

Hasta ahora, hemos aplicado propiedades al **contenedor**. Ahora vamos a ver propiedades para los **ítems individuales**.

### `flex-grow`: Crecer para llenar espacio

Controla cuánto espacio extra toma un ítem.

```css
.item {
    flex-grow: 0;  /* Por defecto: no crece */
    flex-grow: 1;  /* Crece para llenar espacio disponible */
}
```

#### Ejemplo

```html
<div class="contenedor">
    <div class="item">1</div>
    <div class="item grow">2</div>
    <div class="item">3</div>
</div>
```

```css
.contenedor {
    display: flex;
}

.item {
    background: lightblue;
    padding: 20px;
}

.grow {
    flex-grow: 1;  /* Este ítem crece para llenar el espacio */
    background: lightcoral;
}
```

Resultado:
```
┌───┐ ┌──────────────────┐ ┌───┐
│ 1 │ │       2         │ │ 3 │
└───┘ └──────────────────┘ └───┘
```

El ítem 2 toma todo el espacio disponible.

#### Múltiples ítems con `flex-grow`

```css
.item {
    flex-grow: 1; /* Todos crecen igual */
}
```

```
┌──────┐ ┌──────┐ ┌──────┐
│  1   │ │  2   │ │  3   │
└──────┘ └──────┘ └──────┘
```

Todos ocupan el mismo espacio.

---

### `flex-shrink`: Encogerse cuando no hay espacio

```css
.item {
    flex-shrink: 1;  /* Por defecto: puede encogerse */
    flex-shrink: 0;  /* No se encoge */
}
```

**Ejemplo:** Si un ítem tiene `flex-shrink: 0`, mantendrá su tamaño aunque no haya espacio.

---

### `flex-basis`: Tamaño base

El tamaño inicial del ítem antes de que `flex-grow` o `flex-shrink` actúen.

```css
.item {
    flex-basis: 200px;  /* Tamaño base de 200px */
}
```

Es como `width` pero funciona en el eje principal (horizontal o vertical según `flex-direction`).

---

### Atajo: `flex`

```css
.item {
    flex: 1;  /* Atajo para flex-grow: 1, flex-shrink: 1, flex-basis: 0 */
}
```

**Sintaxis completa:**

```css
flex: [flex-grow] [flex-shrink] [flex-basis];
```

**Ejemplos:**

```css
/* Crece pero no se encoge, tamaño base 200px */
flex: 1 0 200px;

/* No crece ni se encoge, tamaño 100px fijo */
flex: 0 0 100px;

/* Crece y se encoge, sin tamaño base */
flex: 1 1 auto;

/* Atajo común: crece para llenar espacio */
flex: 1;
```

---

### `align-self`: Alineación individual

Sobrescribe `align-items` para un ítem específico.

```html
<div class="contenedor">
    <div class="item">1</div>
    <div class="item especial">2</div>
    <div class="item">3</div>
</div>
```

```css
.contenedor {
    display: flex;
    align-items: flex-start;  /* Todos arriba */
    height: 200px;
}

.especial {
    align-self: flex-end;  /* Solo este va abajo */
    background: lightcoral;
}
```

Resultado:
```
┌────────────────────────────────┐
│ ┌───┐       ┌───┐             │
│ │ 1 │       │ 3 │             │
│ └───┘       └───┘             │
│                                │
│         ┌───┐                 │
│         │ 2 │                 │
│         └───┘                 │
└────────────────────────────────┘
```

---

### `order`: Reordenar ítems

Cambia el orden visual sin tocar el HTML.

```html
<div class="contenedor">
    <div class="item">1</div>
    <div class="item segundo">2</div>
    <div class="item">3</div>
</div>
```

```css
.segundo {
    order: 1;  /* Se mueve al final */
}

/* Por defecto, todos los ítems tienen order: 0 */
```

Resultado visual:
```
┌───┐ ┌───┐ ┌───┐
│ 1 │ │ 3 │ │ 2 │
└───┘ └───┘ └───┘
```

---

## Ejemplos prácticos

### Ejemplo 1: Menú de navegación

```html
<nav class="menu">
    <div class="logo">MiSitio</div>
    <ul class="enlaces">
        <li><a href="#">Inicio</a></li>
        <li><a href="#">Productos</a></li>
        <li><a href="#">Contacto</a></li>
    </ul>
</nav>
```

```css
.menu {
    display: flex;
    justify-content: space-between;  /* Logo a la izq, enlaces a la der */
    align-items: center;             /* Centrados verticalmente */
    background-color: #2c3e50;
    padding: 15px 30px;
}

.logo {
    color: white;
    font-size: 24px;
    font-weight: bold;
}

.enlaces {
    display: flex;
    gap: 20px;
    list-style: none;
    margin: 0;
    padding: 0;
}

.enlaces a {
    color: white;
    text-decoration: none;
}
```

---

### Ejemplo 2: Tarjetas responsivas

```html
<div class="contenedor-tarjetas">
    <div class="tarjeta">Tarjeta 1</div>
    <div class="tarjeta">Tarjeta 2</div>
    <div class="tarjeta">Tarjeta 3</div>
</div>
```

```css
.contenedor-tarjetas {
    display: flex;
    flex-wrap: wrap;
    gap: 20px;
    padding: 20px;
}

.tarjeta {
    flex: 1 1 300px;  /* Crece, se encoge, tamaño base 300px */
    background-color: white;
    padding: 30px;
    border: 1px solid #ddd;
    border-radius: 8px;
}
```

**Comportamiento:**
- En pantallas grandes: 3 tarjetas por fila
- En pantallas medianas: 2 tarjetas por fila
- En pantallas pequeñas: 1 tarjeta por fila

---

### Ejemplo 3: Layout de aplicación

```html
<div class="app">
    <header class="header">Header</header>
    <div class="contenido">
        <aside class="sidebar">Sidebar</aside>
        <main class="main">Contenido principal</main>
    </div>
    <footer class="footer">Footer</footer>
</div>
```

```css
.app {
    display: flex;
    flex-direction: column;
    min-height: 100vh;
}

.header {
    background-color: #2c3e50;
    color: white;
    padding: 20px;
}

.contenido {
    display: flex;
    flex: 1;  /* Ocupa todo el espacio disponible */
}

.sidebar {
    flex: 0 0 250px;  /* Ancho fijo de 250px */
    background-color: #34495e;
    color: white;
    padding: 20px;
}

.main {
    flex: 1;  /* Ocupa el resto del espacio */
    padding: 20px;
    background-color: #ecf0f1;
}

.footer {
    background-color: #2c3e50;
    color: white;
    padding: 20px;
    text-align: center;
}
```

---

## Cuándo usar Flexbox

✅ **Usa Flexbox para:**
- Menús de navegación
- Distribución de elementos en una dimensión (fila o columna)
- Centrar elementos
- Componentes pequeños (tarjetas, botones, etc.)
- Alineación vertical
- Distribuir espacio entre elementos

❌ **NO uses Flexbox para:**
- Layouts complejos en dos dimensiones → Usa **CSS Grid** (próximo módulo)
- Tablas de datos → Usa `<table>`

---

## Buenas prácticas

### 1. Usa `gap` en lugar de márgenes

```css
/* Mal (márgenes manuales) */
.item {
    margin-right: 20px;
}

.item:last-child {
    margin-right: 0;
}

/* Bien (con gap) */
.contenedor {
    display: flex;
    gap: 20px;
}
```

### 2. `flex: 1` para distribuir espacio

```css
.item {
    flex: 1; /* Todos los ítems ocupan el mismo espacio */
}
```

### 3. Combina Flexbox con `max-width`

```css
.contenedor {
    display: flex;
    justify-content: center;
    padding: 20px;
}

.contenido {
    max-width: 1200px;
    width: 100%;
}
```

---

## Errores comunes

### Error 1: Olvidar `display: flex`

```css
/* No funciona */
.contenedor {
    justify-content: center;  /* Sin display: flex, esto no hace nada */
}

/* Correcto */
.contenedor {
    display: flex;
    justify-content: center;
}
```

### Error 2: Confundir ejes

- `justify-content` = Eje principal (dirección de `flex-direction`)
- `align-items` = Eje transversal (perpendicular)

### Error 3: No usar `flex-wrap` cuando es necesario

Si tienes muchos ítems y no caben, añade:

```css
flex-wrap: wrap;
```

---

## Ejercicios prácticos

### Ejercicio 1: Tres botones centrados

Crea tres botones centrados horizontal y verticalmente en la página.

<details>
<summary>✅ Solución</summary>

```html
<div class="contenedor">
    <button>Botón 1</button>
    <button>Botón 2</button>
    <button>Botón 3</button>
</div>
```

```css
body {
    margin: 0;
    height: 100vh;
}

.contenedor {
    display: flex;
    justify-content: center;
    align-items: center;
    gap: 15px;
    height: 100%;
}

button {
    padding: 12px 24px;
    background-color: #3498db;
    color: white;
    border: none;
    border-radius: 5px;
    cursor: pointer;
}
```

</details>

---

### Ejercicio 2: Header con logo y navegación

Crea un header donde:
- Logo a la izquierda
- Enlaces de navegación a la derecha
- Todo centrado verticalmente

<details>
<summary>✅ Solución</summary>

```html
<header class="header">
    <div class="logo">MiLogo</div>
    <nav class="nav">
        <a href="#">Inicio</a>
        <a href="#">Servicios</a>
        <a href="#">Contacto</a>
    </nav>
</header>
```

```css
.header {
    display: flex;
    justify-content: space-between;
    align-items: center;
    background-color: #2c3e50;
    padding: 20px 40px;
}

.logo {
    color: white;
    font-size: 24px;
    font-weight: bold;
}

.nav {
    display: flex;
    gap: 30px;
}

.nav a {
    color: white;
    text-decoration: none;
}
```

</details>

---

### Ejercicio 3: Galería de imágenes responsiva

Crea una galería de 6 imágenes que se adapte:
- 3 por fila en pantallas grandes
- 2 por fila en medianas
- 1 por fila en pequeñas

<details>
<summary>💡 Pista</summary>

Usa `flex-wrap: wrap` y `flex: 1 1 calc(33.333% - 20px)` (con gap de 20px).

</details>

---

## Cheat sheet de Flexbox

```css
/* Contenedor */
.contenedor {
    display: flex;                    /* Activa Flexbox */
    
    /* Dirección */
    flex-direction: row;              /* Horizontal (por defecto) */
    flex-direction: column;           /* Vertical */
    
    /* Alineación eje principal */
    justify-content: flex-start;      /* Inicio */
    justify-content: center;          /* Centro */
    justify-content: space-between;   /* Espacio entre */
    
    /* Alineación eje transversal */
    align-items: stretch;             /* Estirar (por defecto) */
    align-items: center;              /* Centro */
    align-items: flex-start;          /* Inicio */
    
    /* Salto de línea */
    flex-wrap: wrap;                  /* Permitir salto */
    
    /* Espaciado */
    gap: 20px;                        /* Espacio entre ítems */
}

/* Ítems */
.item {
    flex: 1;                          /* Crecer para llenar espacio */
    flex: 0 0 200px;                  /* Ancho fijo de 200px */
    
    align-self: center;               /* Alineación individual */
    order: 1;                         /* Cambiar orden */
}
```

---

## Recursos adicionales

- [Flexbox Froggy](https://flexboxfroggy.com/#es) - Juego para aprender Flexbox
- [CSS Tricks: Guía completa de Flexbox](https://css-tricks.com/snippets/css/a-guide-to-flexbox/)
- [MDN: Flexbox](https://developer.mozilla.org/es/docs/Learn/CSS/CSS_layout/Flexbox)

---

## Siguiente paso

Ya dominas Flexbox (layouts en una dimensión). Ahora vas a aprender **CSS Grid**, el sistema para layouts en dos dimensiones (filas Y columnas).

→ [11-grid-basico.md](11-grid-basico.md)

Grid es perfecto para layouts complejos de página completa.

---

**Recuerda:** Flexbox es tu mejor amigo para layouts simples. Úsalo siempre que puedas. Es simple, potente y predecible.

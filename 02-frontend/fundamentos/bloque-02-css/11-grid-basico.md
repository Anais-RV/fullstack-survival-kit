# CSS Grid Básico

**CSS Grid = El sistema de layout más potente de CSS.**

Si Flexbox es para una dimensión (fila O columna), Grid es para **dos dimensiones** (filas Y columnas al mismo tiempo).

---

## ¿Qué vas a aprender?

- Qué es Grid y cuándo usarlo
- Crear filas y columnas
- Posicionar elementos en la cuadrícula
- Áreas con nombre (grid template areas)
- Alineación de elementos
- Diferencias entre Grid y Flexbox

## Por qué es útil

**Analogía:** Piensa en una hoja de cálculo (Excel):

- Flexbox = Organizar cosas en una sola fila o columna
- Grid = La hoja completa con filas Y columnas, donde puedes colocar cada celda donde quieras

**Grid resuelve:**
- Layouts complejos de página completa
- Posicionar elementos en filas y columnas específicas
- Crear diseños tipo revista o dashboard
- Gallerías de imágenes con tamaños variados

---

## Concepto básico: Contenedor y celdas

```html
<div class="contenedor">  ← CONTENEDOR GRID
    <div class="item">1</div>     ← ÍTEM GRID
    <div class="item">2</div>
    <div class="item">3</div>
    <div class="item">4</div>
</div>
```

```css
.contenedor {
    display: grid;  /* Activa CSS Grid */
    grid-template-columns: 200px 200px;  /* 2 columnas de 200px */
    grid-template-rows: 100px 100px;     /* 2 filas de 100px */
}
```

**Resultado:**

```
┌────────┬────────┐
│   1    │   2    │
├────────┼────────┤
│   3    │   4    │
└────────┴────────┘
```

Una cuadrícula de 2×2.

---

## Definir columnas: `grid-template-columns`

```css
.contenedor {
    display: grid;
    
    /* 3 columnas de 200px cada una */
    grid-template-columns: 200px 200px 200px;
    
    /* 2 columnas: primera 300px, segunda 500px */
    grid-template-columns: 300px 500px;
    
    /* 4 columnas iguales */
    grid-template-columns: 25% 25% 25% 25%;
}
```

---

## Definir filas: `grid-template-rows`

```css
.contenedor {
    display: grid;
    grid-template-columns: 200px 200px;
    
    /* 2 filas: primera 100px, segunda 150px */
    grid-template-rows: 100px 150px;
}
```

**Nota:** Si no defines filas, se crean automáticamente con altura según contenido.

---

## Unidad `fr` (fracción)

`fr` = fracción del espacio disponible. Es como "partes".

```css
.contenedor {
    display: grid;
    grid-template-columns: 1fr 1fr 1fr;  /* 3 columnas iguales */
}
```

Cada columna toma 1/3 del espacio disponible.

### Mezclar `fr` con valores fijos

```css
grid-template-columns: 200px 1fr 1fr;
```

- Primera columna: 200px fijo
- Segunda y tercera: se reparten el espacio restante

```
┌────┬──────────┬──────────┐
│200 │   1fr    │   1fr    │
│px  │          │          │
└────┴──────────┴──────────┘
```

### Proporciones diferentes

```css
grid-template-columns: 2fr 1fr;
```

- Primera columna: 2 partes (66.66%)
- Segunda columna: 1 parte (33.33%)

```
┌────────────────┬────────┐
│      2fr       │  1fr   │
│                │        │
└────────────────┴────────┘
```

---

## `repeat()` - Repetir columnas/filas

En lugar de escribir `1fr 1fr 1fr 1fr`:

```css
grid-template-columns: repeat(4, 1fr);  /* 4 columnas de 1fr */
```

```css
grid-template-columns: repeat(3, 200px);  /* 3 columnas de 200px */
```

```css
grid-template-columns: repeat(2, 1fr 2fr);  /* 1fr 2fr 1fr 2fr */
```

---

## `gap` - Espacio entre celdas

```css
.contenedor {
    display: grid;
    grid-template-columns: repeat(3, 1fr);
    gap: 20px;  /* 20px de espacio entre todas las celdas */
}
```

```css
/* Espacio diferente vertical y horizontal */
gap: 10px 20px;  /* 10px entre filas, 20px entre columnas */

/* O por separado */
row-gap: 10px;
column-gap: 20px;
```

---

## Ejemplo básico: Galería 3×3

```html
<div class="galeria">
    <div class="item">1</div>
    <div class="item">2</div>
    <div class="item">3</div>
    <div class="item">4</div>
    <div class="item">5</div>
    <div class="item">6</div>
    <div class="item">7</div>
    <div class="item">8</div>
    <div class="item">9</div>
</div>
```

```css
.galeria {
    display: grid;
    grid-template-columns: repeat(3, 1fr);  /* 3 columnas iguales */
    grid-template-rows: repeat(3, 150px);   /* 3 filas de 150px */
    gap: 15px;
}

.item {
    background-color: #3498db;
    color: white;
    display: flex;
    justify-content: center;
    align-items: center;
    font-size: 24px;
}
```

**Resultado:**

```
┌───┬───┬───┐
│ 1 │ 2 │ 3 │
├───┼───┼───┤
│ 4 │ 5 │ 6 │
├───┼───┼───┤
│ 7 │ 8 │ 9 │
└───┴───┴───┘
```

---

## Posicionar elementos: `grid-column` y `grid-row`

Controla cuántas columnas/filas ocupa un elemento.

### `grid-column`

```css
.item {
    grid-column: 1 / 3;  /* Desde línea 1 hasta línea 3 (2 columnas) */
}
```

**Líneas de la cuadrícula:**

```
    1    2    3    4
    │    │    │    │
────┼────┼────┼────┼────
    │    │    │    │
    │ A  │ B  │ C  │
    │    │    │    │
```

`grid-column: 1 / 3` → El elemento ocupa desde la línea 1 hasta la 3 (columnas A y B).

### Atajo: `span`

```css
grid-column: span 2;  /* Ocupa 2 columnas */
```

Equivalente a `grid-column: auto / span 2`.

### `grid-row`

```css
.item {
    grid-row: 1 / 3;  /* Ocupa 2 filas */
}
```

### Ejemplo: Elemento grande

```html
<div class="galeria">
    <div class="item destacado">1</div>
    <div class="item">2</div>
    <div class="item">3</div>
    <div class="item">4</div>
    <div class="item">5</div>
</div>
```

```css
.galeria {
    display: grid;
    grid-template-columns: repeat(3, 1fr);
    grid-auto-rows: 150px;
    gap: 15px;
}

.destacado {
    grid-column: span 2;  /* Ocupa 2 columnas */
    grid-row: span 2;     /* Ocupa 2 filas */
    background-color: #e74c3c;
}

.item {
    background-color: #3498db;
    color: white;
    display: flex;
    justify-content: center;
    align-items: center;
    font-size: 24px;
}
```

**Resultado:**

```
┌───────┬───┐
│       │ 2 │
│   1   ├───┤
│       │ 3 │
├───┬───┼───┤
│ 4 │ 5 │   │
└───┴───┴───┘
```

El elemento 1 ocupa 2×2.

---

## `grid-template-areas`: Layout con nombres

La forma más legible de crear layouts complejos.

### Ejemplo: Layout de página

```html
<div class="layout">
    <header class="header">Header</header>
    <aside class="sidebar">Sidebar</aside>
    <main class="main">Main</main>
    <footer class="footer">Footer</footer>
</div>
```

```css
.layout {
    display: grid;
    grid-template-areas:
        "header  header"
        "sidebar main"
        "footer  footer";
    
    grid-template-columns: 250px 1fr;
    grid-template-rows: auto 1fr auto;
    gap: 20px;
    min-height: 100vh;
}

.header {
    grid-area: header;
    background-color: #2c3e50;
    color: white;
    padding: 20px;
}

.sidebar {
    grid-area: sidebar;
    background-color: #34495e;
    color: white;
    padding: 20px;
}

.main {
    grid-area: main;
    background-color: #ecf0f1;
    padding: 20px;
}

.footer {
    grid-area: footer;
    background-color: #2c3e50;
    color: white;
    padding: 20px;
    text-align: center;
}
```

**Resultado:**

```
┌──────────────────────┐
│       HEADER         │
├────────┬─────────────┤
│ SIDE   │    MAIN     │
│ BAR    │             │
│        │             │
├────────┴─────────────┤
│       FOOTER         │
└──────────────────────┘
```

**Ventajas:**
- Súper legible: ves el layout en el CSS
- Fácil de modificar
- Reordenar áreas sin tocar HTML

### Celda vacía: `.`

```css
grid-template-areas:
    "header header header"
    "sidebar main ."
    "footer footer footer";
```

El punto `.` representa una celda vacía.

---

## Alineación de elementos

### `justify-items` - Alineación horizontal de ítems

```css
.contenedor {
    display: grid;
    justify-items: start;    /* Izquierda */
    justify-items: center;   /* Centro */
    justify-items: end;      /* Derecha */
    justify-items: stretch;  /* Estirar (por defecto) */
}
```

### `align-items` - Alineación vertical de ítems

```css
.contenedor {
    display: grid;
    align-items: start;    /* Arriba */
    align-items: center;   /* Centro */
    align-items: end;      /* Abajo */
    align-items: stretch;  /* Estirar (por defecto) */
}
```

### Alineación individual: `justify-self` y `align-self`

```css
.item-especial {
    justify-self: center;  /* Este ítem centrado horizontalmente */
    align-self: end;       /* Este ítem al fondo verticalmente */
}
```

---

## `auto-fill` y `auto-fit` - Grid responsivo automático

Crea columnas automáticamente según el espacio disponible.

### `auto-fill`

```css
.galeria {
    display: grid;
    grid-template-columns: repeat(auto-fill, minmax(250px, 1fr));
    gap: 20px;
}
```

**Significado:**
- Crea tantas columnas como quepan
- Cada columna mínimo 250px, máximo 1fr (espacio disponible)
- Se adapta automáticamente al ancho de la pantalla

**Resultado:**
- Pantalla ancha → 4 columnas
- Pantalla media → 2-3 columnas
- Pantalla pequeña → 1 columna

### `auto-fit` vs `auto-fill`

- **`auto-fill`:** Crea columnas vacías si hay espacio
- **`auto-fit`:** Las columnas existentes se expanden para llenar el espacio

**Usa `auto-fit` casi siempre.**

---

## Ejemplo completo: Dashboard

```html
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <title>Dashboard</title>
    <link rel="stylesheet" href="dashboard.css">
</head>
<body>
    <div class="dashboard">
        <header class="header">Dashboard</header>
        <aside class="sidebar">
            <nav>
                <a href="#">Inicio</a>
                <a href="#">Usuarios</a>
                <a href="#">Reportes</a>
            </nav>
        </aside>
        <main class="main">
            <div class="tarjeta">Tarjeta 1</div>
            <div class="tarjeta">Tarjeta 2</div>
            <div class="tarjeta">Tarjeta 3</div>
            <div class="tarjeta destacada">Tarjeta Grande</div>
            <div class="tarjeta">Tarjeta 5</div>
            <div class="tarjeta">Tarjeta 6</div>
        </main>
        <footer class="footer">© 2026</footer>
    </div>
</body>
</html>
```

```css
* {
    margin: 0;
    padding: 0;
    box-sizing: border-box;
}

body {
    font-family: Arial, sans-serif;
}

.dashboard {
    display: grid;
    grid-template-areas:
        "header header"
        "sidebar main"
        "footer footer";
    grid-template-columns: 200px 1fr;
    grid-template-rows: auto 1fr auto;
    min-height: 100vh;
    gap: 0;
}

.header {
    grid-area: header;
    background-color: #2c3e50;
    color: white;
    padding: 20px;
    font-size: 24px;
}

.sidebar {
    grid-area: sidebar;
    background-color: #34495e;
    padding: 20px;
}

.sidebar nav {
    display: flex;
    flex-direction: column;
    gap: 15px;
}

.sidebar a {
    color: white;
    text-decoration: none;
    padding: 10px;
    border-radius: 5px;
    transition: background-color 0.3s;
}

.sidebar a:hover {
    background-color: #2c3e50;
}

.main {
    grid-area: main;
    background-color: #ecf0f1;
    padding: 20px;
    
    /* Grid dentro del main */
    display: grid;
    grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));
    gap: 20px;
    align-content: start;
}

.tarjeta {
    background-color: white;
    padding: 30px;
    border-radius: 8px;
    box-shadow: 0 2px 4px rgba(0,0,0,0.1);
    display: flex;
    justify-content: center;
    align-items: center;
    min-height: 150px;
}

.destacada {
    grid-column: span 2;
    background-color: #3498db;
    color: white;
}

.footer {
    grid-area: footer;
    background-color: #2c3e50;
    color: white;
    padding: 15px;
    text-align: center;
}
```

---

## Grid vs Flexbox: ¿Cuándo usar cada uno?

| Situación | Usa esto |
|-----------|----------|
| Menú horizontal | Flexbox |
| Galería de fotos (filas y columnas) | Grid |
| Centrar un elemento | Flexbox |
| Layout de página completa | Grid |
| Lista de elementos uno tras otro | Flexbox |
| Dashboard con áreas definidas | Grid |
| Botones uno al lado del otro | Flexbox |
| Diseño tipo revista | Grid |

**Regla simple:**
- **Una dimensión (fila O columna)** → Flexbox
- **Dos dimensiones (filas Y columnas)** → Grid

**Puedes combinarlos:**

```css
.contenedor-principal {
    display: grid;  /* Layout general */
}

.menu {
    display: flex;  /* Menú dentro del grid */
}
```

---

## Buenas prácticas

### 1. Usa `fr` en lugar de porcentajes

```css
/* Bien */
grid-template-columns: 1fr 1fr 1fr;

/* Menos flexible */
grid-template-columns: 33.333% 33.333% 33.333%;
```

### 2. `auto-fit` con `minmax()` para responsividad

```css
grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));
```

Una línea de CSS y tienes responsividad automática.

### 3. `grid-template-areas` para layouts complejos

Es más legible que posicionar con números.

### 4. Combina Grid y Flexbox

```css
.tarjeta {
    /* Grid para el layout general */
    display: grid;
    grid-template-rows: auto 1fr auto;
}

.tarjeta-footer {
    /* Flexbox para alinear botones */
    display: flex;
    gap: 10px;
}
```

---

## Errores comunes

### Error 1: Olvidar `display: grid`

```css
/* No funciona */
.contenedor {
    grid-template-columns: 1fr 1fr;  /* Sin display: grid, esto no hace nada */
}
```

### Error 2: Confundir líneas y celdas

```css
grid-column: 1 / 3;  /* Líneas 1 a 3 = 2 columnas */
grid-column: span 2; /* Ocupa 2 columnas */
```

### Error 3: No usar `gap`

```css
/* Mal (márgenes manuales) */
.item {
    margin: 10px;
}

/* Bien */
.contenedor {
    display: grid;
    gap: 20px;
}
```

---

## Ejercicios prácticos

### Ejercicio 1: Galería 4 columnas

Crea una galería de 12 elementos en 4 columnas.

<details>
<summary>✅ Solución</summary>

```html
<div class="galeria">
    <div class="item">1</div>
    <div class="item">2</div>
    <!-- ... hasta 12 -->
</div>
```

```css
.galeria {
    display: grid;
    grid-template-columns: repeat(4, 1fr);
    gap: 15px;
    padding: 20px;
}

.item {
    background-color: #3498db;
    color: white;
    padding: 40px;
    text-align: center;
    border-radius: 8px;
}
```

</details>

---

### Ejercicio 2: Layout de blog

Crea un layout con:
- Header completo arriba
- Sidebar a la izquierda (250px)
- Contenido principal a la derecha
- Footer completo abajo

Usa `grid-template-areas`.

<details>
<summary>✅ Solución</summary>

```html
<div class="blog">
    <header class="header">Mi Blog</header>
    <aside class="sidebar">Sidebar</aside>
    <main class="main">Contenido</main>
    <footer class="footer">Footer</footer>
</div>
```

```css
.blog {
    display: grid;
    grid-template-areas:
        "header header"
        "sidebar main"
        "footer footer";
    grid-template-columns: 250px 1fr;
    grid-template-rows: auto 1fr auto;
    min-height: 100vh;
}

.header { grid-area: header; background: #2c3e50; color: white; padding: 20px; }
.sidebar { grid-area: sidebar; background: #34495e; color: white; padding: 20px; }
.main { grid-area: main; background: #ecf0f1; padding: 20px; }
.footer { grid-area: footer; background: #2c3e50; color: white; padding: 15px; text-align: center; }
```

</details>

---

### Ejercicio 3: Galería responsiva con elemento destacado

Crea una galería donde:
- El primer elemento ocupa 2×2
- Los demás son normales (1×1)
- Se adapta automáticamente con `auto-fit`

<details>
<summary>💡 Pista</summary>

```css
.galeria {
    display: grid;
    grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
    gap: 15px;
}

.item:first-child {
    grid-column: span 2;
    grid-row: span 2;
}
```

</details>

---

## Cheat sheet de Grid

```css
/* Contenedor */
.contenedor {
    display: grid;
    
    /* Columnas */
    grid-template-columns: 200px 1fr 1fr;       /* Fijas y flexibles */
    grid-template-columns: repeat(3, 1fr);      /* 3 iguales */
    grid-template-columns: repeat(auto-fit, minmax(250px, 1fr)); /* Responsivo */
    
    /* Filas */
    grid-template-rows: 100px auto 50px;
    grid-auto-rows: 150px;                      /* Filas automáticas */
    
    /* Espacio */
    gap: 20px;
    gap: 10px 20px;  /* fila columna */
    
    /* Áreas */
    grid-template-areas:
        "header header"
        "sidebar main"
        "footer footer";
    
    /* Alineación */
    justify-items: center;  /* Horizontal */
    align-items: center;    /* Vertical */
}

/* Ítems */
.item {
    /* Posición */
    grid-column: 1 / 3;     /* Líneas */
    grid-column: span 2;    /* Ocupa 2 columnas */
    grid-row: span 2;       /* Ocupa 2 filas */
    
    /* Área */
    grid-area: header;
    
    /* Alineación individual */
    justify-self: center;
    align-self: end;
}
```

---

## Recursos adicionales

- [Grid Garden](https://cssgridgarden.com/#es) - Juego para aprender Grid
- [CSS Tricks: Guía completa de Grid](https://css-tricks.com/snippets/css/complete-guide-grid/)
- [MDN: CSS Grid](https://developer.mozilla.org/es/docs/Web/CSS/CSS_Grid_Layout)
- [Grid by Example](https://gridbyexample.com/) - Ejemplos prácticos

---

## Siguiente paso

Ya dominas Grid. Ahora el último concepto CSS esencial: **diseño responsive** con media queries.

→ [12-responsive-basico.md](12-responsive-basico.md)

Ahí aprenderás a adaptar tus diseños a móviles, tablets y desktops.

---

**Recuerda:** Grid es perfecto para layouts complejos. No tengas miedo de usarlo. Es más simple de lo que parece una vez que entiendes el concepto de filas y columnas.

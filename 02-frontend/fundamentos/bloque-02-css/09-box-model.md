# El Box Model

Este es **el concepto más importante de CSS**. Si entiendes el Box Model, entiendes el 70% del layout en CSS.

**Box Model = Cómo se calculan las dimensiones y el espacio de cada elemento.**

---

## ¿Qué vas a aprender?

- Qué es el Box Model y sus 4 partes
- Contenido, padding, border, margin
- `box-sizing`: el truco que todo desarrollador usa
- Cómo depurar espaciado con DevTools
- Display: block vs inline vs inline-block

## Por qué es útil

**Analogía:** Piensa en un cuadro en la pared:

1. **Contenido** = La foto
2. **Padding** = El espacio blanco alrededor de la foto (dentro del marco)
3. **Border** = El marco
4. **Margin** = El espacio entre el cuadro y la pared (o entre varios cuadros)

Sin entender esto, no sabrás por qué tu `<div>` de 300px ocupa realmente 340px, o por qué hay espacios extraños entre elementos.

---

## Las 4 partes del Box Model

Todo elemento HTML es una **caja** con 4 capas:

```
┌─────────────────────────────────────┐
│         MARGIN (transparente)       │
│  ┌───────────────────────────────┐  │
│  │     BORDER (visible)          │  │
│  │  ┌─────────────────────────┐  │  │
│  │  │  PADDING (transparente) │  │  │
│  │  │  ┌───────────────────┐  │  │  │
│  │  │  │   CONTENT         │  │  │  │
│  │  │  │   (tu texto,      │  │  │  │
│  │  │  │    imagen, etc.)  │  │  │  │
│  │  │  └───────────────────┘  │  │  │
│  │  └─────────────────────────┘  │  │
│  └───────────────────────────────┘  │
└─────────────────────────────────────┘
```

1. **Content (Contenido):** El texto, imagen o lo que pongas dentro
2. **Padding:** Espacio entre el contenido y el borde
3. **Border:** El borde visible alrededor
4. **Margin:** Espacio entre este elemento y otros elementos

---

## Content (Contenido)

El área donde va tu contenido (texto, imágenes, etc.).

```css
div {
    width: 200px;    /* Ancho del contenido */
    height: 100px;   /* Alto del contenido */
}
```

**Por defecto:**
- **Block elements** (`<div>`, `<p>`, `<h1>`) → ancho = 100% del contenedor, alto = automático según contenido
- **Inline elements** (`<span>`, `<a>`, `<strong>`) → ancho y alto según su contenido

---

## Padding

**Espacio interno** entre el contenido y el borde.

```css
div {
    padding: 20px; /* 20px en los 4 lados */
}
```

### Sintaxis completa

```css
/* Un valor: todos los lados */
padding: 20px;

/* Dos valores: vertical | horizontal */
padding: 10px 20px;  /* 10px arriba/abajo, 20px izq/der */

/* Tres valores: arriba | horizontal | abajo */
padding: 10px 20px 30px;

/* Cuatro valores: arriba | derecha | abajo | izquierda (sentido horario) */
padding: 10px 20px 30px 40px;
```

### Propiedades individuales

```css
padding-top: 10px;
padding-right: 20px;
padding-bottom: 30px;
padding-left: 40px;
```

### Ejemplo visual

```html
<div class="caja">Contenido</div>
```

```css
.caja {
    width: 200px;
    background-color: lightblue;
    padding: 20px; /* Espacio interno */
}
```

**Sin padding:**
```
┌────────────┐
│Contenido   │
└────────────┘
```

**Con `padding: 20px`:**
```
┌────────────────────┐
│                    │
│     Contenido      │
│                    │
└────────────────────┘
```

El contenido tiene aire para respirar.

---

## Border

El **borde visible** alrededor del elemento.

```css
border: 2px solid black;
```

### Sintaxis

```css
border: [grosor] [estilo] [color];
```

### Estilos de borde

```css
border: 2px solid black;   /* Línea sólida */
border: 2px dashed blue;   /* Línea discontinua */
border: 2px dotted red;    /* Línea punteada */
border: 3px double green;  /* Línea doble */
border: 2px groove gray;   /* Efecto 3D hundido */
border: 2px ridge gray;    /* Efecto 3D elevado */
```

### Bordes individuales

```css
border-top: 2px solid black;
border-right: 1px dashed gray;
border-bottom: 3px solid red;
border-left: 1px solid blue;
```

### Border-radius (esquinas redondeadas)

```css
/* Todas las esquinas */
border-radius: 10px;

/* Círculo (si es cuadrado) */
border-radius: 50%;

/* Cada esquina individual */
border-radius: 10px 20px 30px 40px; /* arriba-izq, arriba-der, abajo-der, abajo-izq */
```

**Ejemplo: Botón redondeado**

```css
button {
    padding: 12px 24px;
    background-color: #3498db;
    color: white;
    border: none;
    border-radius: 5px;
}
```

---

## Margin

**Espacio externo** entre este elemento y otros elementos.

```css
div {
    margin: 20px; /* 20px de separación con otros elementos */
}
```

### Sintaxis (igual que padding)

```css
/* Un valor: todos los lados */
margin: 20px;

/* Dos valores: vertical | horizontal */
margin: 10px 20px;

/* Cuatro valores: arriba | derecha | abajo | izquierda */
margin: 10px 20px 30px 40px;
```

### Propiedades individuales

```css
margin-top: 10px;
margin-right: 20px;
margin-bottom: 30px;
margin-left: 40px;
```

### Centrar elementos horizontalmente

```css
.contenedor {
    width: 600px;
    margin: 0 auto; /* 0 arriba/abajo, auto izq/der → centrado */
}
```

**Condición:** El elemento debe tener un `width` definido.

### Márgenes negativos

```css
.superpuesto {
    margin-top: -20px; /* Sube 20px, superponiéndose al elemento de arriba */
}
```

Úsalos con cuidado.

---

## Ejemplo completo del Box Model

```html
<div class="caja">Hola mundo</div>
```

```css
.caja {
    /* Contenido */
    width: 200px;
    height: 100px;
    
    /* Padding (espacio interno) */
    padding: 20px;
    
    /* Border */
    border: 5px solid #3498db;
    
    /* Margin (espacio externo) */
    margin: 30px;
    
    /* Visual */
    background-color: #ecf0f1;
}
```

**¿Cuánto espacio ocupa realmente?**

Sin `box-sizing: border-box` (por defecto):

```
Ancho total = width + padding-left + padding-right + border-left + border-right
            = 200 + 20 + 20 + 5 + 5
            = 250px

Alto total = height + padding-top + padding-bottom + border-top + border-bottom
           = 100 + 20 + 20 + 5 + 5
           = 150px

Espacio ocupado con margin = 250 + (30 * 2) en X, 150 + (30 * 2) en Y
                           = 310px × 210px
```

---

## `box-sizing`: El truco esencial

Por defecto, CSS usa `content-box`:

```css
/* Por defecto */
box-sizing: content-box;
```

**Problema:** Si defines `width: 200px` y añades `padding: 20px`, el ancho real será 240px.

### Solución: `border-box`

```css
box-sizing: border-box;
```

Con `border-box`:
- El `width` incluye padding y border
- El cálculo es intuitivo

```css
.caja {
    box-sizing: border-box;
    width: 200px;
    padding: 20px;
    border: 5px solid black;
}
```

**Ancho real = 200px** (padding y border están incluidos).

### Aplicar a todo el sitio (RECOMENDADO)

```css
/* Al inicio de tu CSS */
*, *::before, *::after {
    box-sizing: border-box;
}
```

**Esto debería ser tu primer CSS en todo proyecto.**

---

## Display: Block, Inline, Inline-Block

### `display: block`

Elementos que ocupan todo el ancho disponible.

```css
div, p, h1, section, header, footer {
    display: block; /* Por defecto */
}
```

**Características:**
- Ocupan todo el ancho del contenedor
- Empiezan en una nueva línea
- Puedes definir `width` y `height`
- Respetan `margin`, `padding`, `border`

```html
<div style="background: lightblue;">Bloque 1</div>
<div style="background: lightcoral;">Bloque 2</div>
```

Resultado:
```
┌───────────────────────┐
│ Bloque 1              │
└───────────────────────┘
┌───────────────────────┐
│ Bloque 2              │
└───────────────────────┘
```

### `display: inline`

Elementos que fluyen con el texto.

```css
span, a, strong, em {
    display: inline; /* Por defecto */
}
```

**Características:**
- Ocupan solo el espacio de su contenido
- No empiezan nueva línea
- **NO** puedes definir `width` ni `height`
- `margin-top` y `margin-bottom` no funcionan
- `padding` funciona pero puede superponerse

```html
<p>
    Este es un <span style="background: yellow;">texto inline</span> dentro de un párrafo.
</p>
```

Resultado:
```
Este es un [texto inline] dentro de un párrafo.
```

### `display: inline-block`

Híbrido: fluye como inline pero se comporta como block.

```css
.boton {
    display: inline-block;
}
```

**Características:**
- Fluye con el texto (como inline)
- Puedes definir `width` y `height` (como block)
- Respeta todos los márgenes y padding

```html
<a href="#" class="boton">Botón 1</a>
<a href="#" class="boton">Botón 2</a>
```

```css
.boton {
    display: inline-block;
    padding: 10px 20px;
    background-color: #3498db;
    color: white;
    text-decoration: none;
}
```

Los botones estarán uno al lado del otro, pero con padding y tamaño controlado.

### Tabla comparativa

| Característica | `block` | `inline` | `inline-block` |
|----------------|---------|----------|----------------|
| Nueva línea | Sí | No | No |
| `width`/`height` | Sí | No | Sí |
| Márgenes verticales | Sí | No | Sí |
| Padding vertical | Sí | Sí* | Sí |

*El padding inline funciona pero puede superponerse visualmente.

---

## Colapso de márgenes (Margin Collapse)

Cuando dos márgenes verticales se tocan, **se colapsan en uno solo** (el mayor).

### Ejemplo

```html
<div class="caja1">Caja 1</div>
<div class="caja2">Caja 2</div>
```

```css
.caja1 {
    margin-bottom: 30px;
}

.caja2 {
    margin-top: 20px;
}
```

**¿Separación entre cajas?**  
**30px** (no 50px). El margen mayor gana.

### Cuándo NO ocurre

- Entre elementos `inline-block` o `inline`
- Con elementos `position: absolute` o `position: fixed`
- Con elementos `float`
- Con `flexbox` o `grid`

**Truco:** Si quieres evitarlo, usa padding en lugar de margin, o usa flexbox/grid.

---

## DevTools: Tu mejor amigo

### Inspeccionar el Box Model

1. Abre DevTools (F12 o click derecho → Inspeccionar)
2. Selecciona un elemento
3. Ve a la pestaña "Computed" o "Layout"
4. Verás una representación visual del Box Model:

```
         margin (naranja)
    ┌─────────────────────┐
    │   border (amarillo) │
    │  ┌────────────────┐ │
    │  │ padding (verde)│ │
    │  │ ┌────────────┐ │ │
    │  │ │  content   │ │ │
    │  │ │  (azul)    │ │ │
    │  │ └────────────┘ │ │
    │  └────────────────┘ │
    └─────────────────────┘
```

**Puedes ver en tiempo real:**
- Tamaño del contenido
- Padding aplicado
- Grosor del borde
- Márgenes

**Usa esto para depurar espaciado.**

---

## Ejemplo práctico: Tarjetas

```html
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <title>Tarjetas</title>
    <link rel="stylesheet" href="tarjetas.css">
</head>
<body>
    <div class="contenedor">
        <div class="tarjeta">
            <h3>Producto 1</h3>
            <p>Descripción del producto.</p>
            <button>Comprar</button>
        </div>
        
        <div class="tarjeta">
            <h3>Producto 2</h3>
            <p>Descripción del producto.</p>
            <button>Comprar</button>
        </div>
        
        <div class="tarjeta">
            <h3>Producto 3</h3>
            <p>Descripción del producto.</p>
            <button>Comprar</button>
        </div>
    </div>
</body>
</html>
```

```css
/* Reset con box-sizing */
*, *::before, *::after {
    box-sizing: border-box;
    margin: 0;
    padding: 0;
}

body {
    font-family: Arial, sans-serif;
    background-color: #f5f5f5;
    padding: 40px;
}

.contenedor {
    max-width: 1200px;
    margin: 0 auto;
}

.tarjeta {
    /* Box Model */
    width: 300px;
    padding: 30px;              /* Espacio interno */
    margin-bottom: 30px;        /* Espacio entre tarjetas */
    border: 1px solid #ddd;     /* Borde sutil */
    border-radius: 8px;         /* Esquinas redondeadas */
    
    /* Visual */
    background-color: white;
    box-shadow: 0 2px 4px rgba(0,0,0,0.1);
}

.tarjeta h3 {
    margin-bottom: 15px;
    color: #2c3e50;
}

.tarjeta p {
    margin-bottom: 20px;
    color: #7f8c8d;
    line-height: 1.6;
}

button {
    /* Box Model */
    padding: 10px 20px;
    border: none;
    border-radius: 4px;
    
    /* Visual */
    background-color: #3498db;
    color: white;
    cursor: pointer;
    transition: background-color 0.3s;
}

button:hover {
    background-color: #2980b9;
}
```

**Nota el uso del Box Model:**
- `padding` en `.tarjeta` para aire interno
- `margin-bottom` para separar tarjetas
- `border` y `border-radius` para marco redondeado
- `padding` en `button` para hacerlo clickeable
- `box-sizing: border-box` para cálculos intuitivos

---

## Anchos máximos y mínimos

### `max-width`

```css
.contenedor {
    max-width: 1200px;  /* No más de 1200px de ancho */
    margin: 0 auto;     /* Centrado */
    padding: 0 20px;    /* Padding en móviles */
}
```

En pantallas grandes: 1200px de ancho, centrado.  
En pantallas pequeñas: 100% del ancho disponible.

### `min-width`

```css
.tarjeta {
    min-width: 250px; /* Al menos 250px de ancho */
}
```

### `max-height` y `min-height`

```css
.caja {
    min-height: 200px; /* Al menos 200px de alto */
    max-height: 500px; /* Máximo 500px de alto */
    overflow: auto;    /* Scroll si el contenido es más alto */
}
```

---

## Overflow: Contenido desbordado

Cuando el contenido es más grande que su contenedor.

```css
.caja {
    width: 200px;
    height: 100px;
    overflow: visible;  /* Por defecto: se desborda */
    overflow: hidden;   /* Oculta lo que sobra */
    overflow: scroll;   /* Siempre muestra scrollbar */
    overflow: auto;     /* Scrollbar solo si es necesario */
}
```

**Uso común:**

```css
.texto-largo {
    height: 200px;
    overflow-y: auto; /* Scroll vertical si es necesario */
}
```

---

## Buenas prácticas

### 1. Usa `box-sizing: border-box` siempre

```css
*, *::before, *::after {
    box-sizing: border-box;
}
```

### 2. Reset básico

```css
* {
    margin: 0;
    padding: 0;
    box-sizing: border-box;
}
```

### 3. Define anchos máximos para contenido

```css
.contenido {
    max-width: 800px;
    margin: 0 auto;
    padding: 0 20px;
}
```

### 4. Usa DevTools para depurar espaciado

Si algo no se ve bien, inspecciona el elemento y mira su Box Model.

### 5. Padding para espacio interno, margin para separación

```css
/* Aire dentro del botón */
button {
    padding: 10px 20px;
}

/* Separación entre botones */
button + button {
    margin-left: 10px;
}
```

---

## Errores comunes

### Error 1: No usar `box-sizing: border-box`

```css
/* Sin box-sizing */
.caja {
    width: 200px;
    padding: 20px;
    border: 5px solid black;
}
/* Ancho real: 250px */
```

### Error 2: Olvidar márgenes al calcular tamaños

```css
.caja {
    width: 50%;
    margin: 20px; /* Esto puede romper el layout */
}
```

Mejor:

```css
.caja {
    width: calc(50% - 40px); /* Resta los márgenes */
    margin: 20px;
}
```

O usa flexbox/grid.

### Error 3: Usar padding/margin en elementos inline

```html
<span style="margin-top: 20px;">Esto no funciona</span>
```

Inline elements no respetan márgenes verticales. Usa `display: inline-block` o `display: block`.

---

## Ejercicios prácticos

### Ejercicio 1: Tarjeta centrada

Crea una tarjeta de 400px de ancho, centrada horizontalmente, con:
- Padding de 30px
- Border de 2px sólido gris
- Margin superior de 50px

<details>
<summary>✅ Solución</summary>

```html
<div class="tarjeta">
    <h2>Mi Tarjeta</h2>
    <p>Contenido de la tarjeta.</p>
</div>
```

```css
*, *::before, *::after {
    box-sizing: border-box;
}

body {
    margin: 0;
    font-family: Arial, sans-serif;
    background-color: #f0f0f0;
}

.tarjeta {
    width: 400px;
    margin: 50px auto 0 auto; /* Superior 50px, centrado horizontal */
    padding: 30px;
    border: 2px solid #ccc;
    background-color: white;
}
```

</details>

---

### Ejercicio 2: Dos columnas con espacio entre ellas

Crea dos cajas una al lado de la otra con espacio entre ellas.

<details>
<summary>💡 Pista</summary>

Usa `display: inline-block` o `float`, o (mejor) aprende Flexbox en el próximo módulo 😉.

Con `inline-block`:

```css
.caja {
    display: inline-block;
    width: calc(50% - 10px);
    margin-right: 20px;
    box-sizing: border-box;
}

.caja:last-child {
    margin-right: 0;
}
```

</details>

---

### Ejercicio 3: Botón con hover

Crea un botón que al hacer hover:
- Aumente el padding (efecto de "hincharse")
- Cambie de color

<details>
<summary>✅ Solución</summary>

```html
<button class="boton">Haz hover aquí</button>
```

```css
.boton {
    padding: 12px 24px;
    background-color: #3498db;
    color: white;
    border: none;
    border-radius: 5px;
    cursor: pointer;
    transition: all 0.3s;
}

.boton:hover {
    padding: 16px 32px;
    background-color: #2980b9;
}
```

</details>

---

### Ejercicio 4: Contenedor con ancho máximo

Crea un contenedor que:
- Tenga máximo 800px de ancho
- Esté centrado
- Tenga padding lateral de 20px (para que no toque los bordes en móviles)

<details>
<summary>✅ Solución</summary>

```html
<div class="contenedor">
    <p>Contenido aquí...</p>
</div>
```

```css
.contenedor {
    max-width: 800px;
    margin: 0 auto;
    padding: 0 20px;
}
```

</details>

---

## Cheat sheet del Box Model

```css
/* Contenido */
width: 200px;
height: 100px;
min-width: 100px;
max-width: 500px;
min-height: 50px;
max-height: 300px;

/* Padding (espacio interno) */
padding: 20px;                      /* Todos los lados */
padding: 10px 20px;                 /* Vertical | Horizontal */
padding: 10px 20px 30px 40px;      /* Arriba | Der | Abajo | Izq */
padding-top: 10px;
padding-right: 20px;
padding-bottom: 30px;
padding-left: 40px;

/* Border */
border: 2px solid black;
border: 1px dashed red;
border-radius: 10px;               /* Esquinas redondeadas */
border-radius: 50%;                /* Círculo */

/* Margin (espacio externo) */
margin: 20px;
margin: 0 auto;                    /* Centrado horizontal */
margin-top: 10px;
margin-right: 20px;
margin-bottom: 30px;
margin-left: 40px;

/* Box sizing */
box-sizing: content-box;           /* Por defecto */
box-sizing: border-box;            /* Incluye padding y border en width */

/* Display */
display: block;
display: inline;
display: inline-block;

/* Overflow */
overflow: visible;
overflow: hidden;
overflow: scroll;
overflow: auto;
```

---

## Recursos adicionales

- [MDN: Box Model](https://developer.mozilla.org/es/docs/Learn/CSS/Building_blocks/The_box_model)
- [CSS Tricks: Box Sizing](https://css-tricks.com/box-sizing/)
- [Guía interactiva del Box Model](https://codepen.io/carolineartz/full/ogVXZj)

---

## Siguiente paso

Ya dominas el Box Model. Ahora vas a aprender **Flexbox**, el sistema moderno de layout que hace el diseño mucho más simple.

→ [10-flexbox.md](10-flexbox.md)

Ahí aprenderás a alinear elementos horizontal y verticalmente sin trucos raros ni matemáticas complejas.

---

**Recuerda:** Entiende el Box Model y todo lo demás en CSS tendrá sentido. Usa DevTools para visualizarlo cuando tengas dudas.

# Colores y Tipografía

Ya sabes seleccionar elementos. Ahora vas a aprender a hacerlos **visualmente atractivos** con colores y textos bien diseñados.

**Colores y tipografía son el 80% del diseño visual.** Domina esto y tus páginas se verán profesionales.

---

## ¿Qué vas a aprender?

- Sistemas de color: nombres, hexadecimal, RGB, RGBA
- Tipografía: familias de fuentes, tamaños, pesos, espaciado
- Google Fonts: usar fuentes personalizadas gratis
- Combinaciones de colores que funcionan
- Accesibilidad: contraste y legibilidad

## Por qué es útil

**Analogía:** Los colores y fuentes son como la ropa que viste tu página. Puedes tener buena estructura (HTML) y buena organización (selectores CSS), pero si los colores son chillones y el texto ilegible, nadie querrá quedarse.

Colores y tipografía transmiten:
- **Profesionalismo** (o falta de él)
- **Emoción** (calma, urgencia, alegría, seriedad)
- **Jerarquía** (qué es importante)
- **Legibilidad** (si la gente puede leer tu contenido)

---

## Colores en CSS

### 1. Nombres de color

CSS reconoce 140 nombres de colores en inglés.

```css
p {
    color: red;
    background-color: lightblue;
}

h1 {
    color: darkgreen;
}

div {
    background-color: cornflowerblue; /* Sí, existe */
}
```

**Ventaja:** Fácil de recordar.  
**Desventaja:** Opciones limitadas, colores no siempre profesionales.

**Algunos útiles:** `black`, `white`, `gray`, `silver`, `navy`, `teal`, `crimson`, `coral`, `gold`.

---

### 2. Hexadecimal (Hex)

El sistema más usado. Colores en formato `#RRGGBB` (Red, Green, Blue).

```css
p {
    color: #ff0000; /* Rojo puro */
}

h1 {
    color: #00ff00; /* Verde puro */
}

div {
    background-color: #0000ff; /* Azul puro */
}
```

Cada par de dígitos va de `00` (nada) a `ff` (máximo).

**Ejemplos:**
```css
.negro { color: #000000; }
.blanco { color: #ffffff; }
.gris { color: #808080; }
.azul-claro { color: #3498db; }
.rojo-oscuro { color: #8b0000; }
```

### Forma corta

Si los dígitos están duplicados, puedes abreviar:

```css
/* Igual que #ff0000 */
color: #f00;

/* Igual que #00ff00 */
color: #0f0;

/* Igual que #333333 */
color: #333;
```

**Herramientas útiles:**
- [Color Picker de Google](https://www.google.com/search?q=color+picker)
- [Coolors.co](https://coolors.co/) - Generador de paletas
- [ColorHunt](https://colorhunt.co/) - Paletas listas para usar

---

### 3. RGB (Red, Green, Blue)

Igual que hex, pero con números decimales (0-255).

```css
p {
    color: rgb(255, 0, 0);  /* Rojo */
}

h1 {
    color: rgb(0, 255, 0);  /* Verde */
}

div {
    background-color: rgb(0, 0, 255);  /* Azul */
}
```

**Equivalencias:**
- `#ff0000` = `rgb(255, 0, 0)` = `red`
- `#00ff00` = `rgb(0, 255, 0)` = `green`
- `#0000ff` = `rgb(0, 0, 255)` = `blue`

---

### 4. RGBA (con transparencia)

Como RGB pero con un cuarto valor: **alpha** (opacidad de 0 a 1).

```css
div {
    background-color: rgba(255, 0, 0, 0.5); /* Rojo al 50% de opacidad */
}

p {
    background-color: rgba(0, 0, 0, 0.8); /* Negro al 80% (casi opaco) */
}

span {
    background-color: rgba(255, 255, 255, 0.1); /* Blanco al 10% (casi transparente) */
}
```

**Útil para:**
- Superposiciones
- Efectos de hover
- Capas semitransparentes

**Ejemplo práctico:**

```html
<div class="caja">
    <div class="overlay">
        <p>Texto sobre fondo semitransparente</p>
    </div>
</div>
```

```css
.caja {
    background-image: url('foto.jpg');
    height: 300px;
    position: relative;
}

.overlay {
    position: absolute;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    background-color: rgba(0, 0, 0, 0.6); /* Capa negra al 60% */
    display: flex;
    align-items: center;
    justify-content: center;
}

.overlay p {
    color: white;
    font-size: 24px;
}
```

El texto se lee bien sobre la foto gracias a la capa oscura semitransparente.

---

### 5. HSL y HSLA (Hue, Saturation, Lightness)

Sistema basado en el círculo cromático.

```css
p {
    color: hsl(0, 100%, 50%);     /* Rojo */
    color: hsl(120, 100%, 50%);   /* Verde */
    color: hsl(240, 100%, 50%);   /* Azul */
}
```

- **Hue (tono):** 0-360 (grados en el círculo cromático)
  - 0° o 360° = rojo
  - 120° = verde
  - 240° = azul
- **Saturation (saturación):** 0-100% (gris a color puro)
- **Lightness (luminosidad):** 0-100% (negro a blanco)

```css
/* Con transparencia */
background-color: hsla(200, 100%, 50%, 0.5); /* Azul claro, 50% opaco */
```

**Ventaja de HSL:**  
Fácil crear variaciones de un mismo color ajustando solo la luminosidad.

```css
:root {
    --color-primario: hsl(200, 100%, 50%); /* Azul */
}

.claro {
    background-color: hsl(200, 100%, 80%); /* Mismo azul, más claro */
}

.oscuro {
    background-color: hsl(200, 100%, 30%); /* Mismo azul, más oscuro */
}
```

---

## ¿Qué sistema usar?

| Sistema | Cuándo usarlo |
|---------|---------------|
| **Hex** | Más común, copia/pega de diseñadores |
| **RGB** | Cuando manipulas colores con JavaScript |
| **RGBA** | Cuando necesitas transparencia |
| **HSL** | Cuando creas variaciones de un color base |
| **Nombres** | Prototipos rápidos o colores básicos |

**Recomendación:** Hex para colores sólidos, RGBA para transparencias.

---

## Propiedades de color

### `color` - Color del texto

```css
p {
    color: #333333; /* Texto gris oscuro */
}
```

### `background-color` - Color de fondo

```css
div {
    background-color: #f0f0f0; /* Fondo gris claro */
}
```

### `border-color` - Color del borde

```css
.caja {
    border: 2px solid #3498db; /* Borde azul */
}
```

---

## Tipografía: Familias de fuentes

### Familias genéricas

CSS tiene 5 familias genéricas:

1. **serif** - Con remates (ej: Times New Roman)
   ```css
   font-family: serif;
   ```
   
2. **sans-serif** - Sin remates (ej: Arial, Helvetica)
   ```css
   font-family: sans-serif;
   ```
   
3. **monospace** - Ancho fijo (ej: Courier)
   ```css
   font-family: monospace;
   ```
   
4. **cursive** - Tipo manuscrita
   ```css
   font-family: cursive;
   ```
   
5. **fantasy** - Decorativa
   ```css
   font-family: fantasy;
   ```

### Fuentes específicas con fallback

```css
p {
    font-family: Arial, Helvetica, sans-serif;
}
```

**Funciona así:**
1. ¿Tiene Arial? → Úsala
2. ¿No? ¿Tiene Helvetica? → Úsala
3. ¿No? Usa cualquier sans-serif disponible

**Ejemplo con serif:**

```css
h1 {
    font-family: Georgia, 'Times New Roman', serif;
}
```

**Nota:** Si el nombre tiene espacios, ponlo entre comillas.

---

## Fuentes del sistema (seguras)

Estas fuentes están en casi todos los dispositivos:

```css
/* Sans-serif */
font-family: Arial, sans-serif;
font-family: Helvetica, sans-serif;
font-family: Verdana, sans-serif;
font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;

/* Serif */
font-family: Georgia, serif;
font-family: 'Times New Roman', Times, serif;

/* Monospace */
font-family: 'Courier New', Courier, monospace;
font-family: Monaco, monospace;
```

---

## Google Fonts: Fuentes personalizadas gratis

Google Fonts tiene cientos de fuentes gratuitas.

### Paso 1: Ve a [fonts.google.com](https://fonts.google.com/)

### Paso 2: Elige una fuente

Por ejemplo, **Roboto**.

### Paso 3: Copia el código de importación

Google te da algo así:

```html
<link rel="preconnect" href="https://fonts.googleapis.com">
<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
<link href="https://fonts.googleapis.com/css2?family=Roboto:wght@400;700&display=swap" rel="stylesheet">
```

Ponlo en el `<head>` de tu HTML, **antes** de tu CSS.

### Paso 4: Úsala en tu CSS

```css
body {
    font-family: 'Roboto', sans-serif;
}
```

### Ejemplo completo

**index.html:**

```html
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <title>Google Fonts</title>
    
    <!-- Google Fonts -->
    <link rel="preconnect" href="https://fonts.googleapis.com">
    <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
    <link href="https://fonts.googleapis.com/css2?family=Roboto:wght@400;700&family=Merriweather:wght@400;700&display=swap" rel="stylesheet">
    
    <link rel="stylesheet" href="estilos.css">
</head>
<body>
    <h1>Título con Merriweather</h1>
    <p>Texto con Roboto. Esta es una fuente moderna y legible.</p>
</body>
</html>
```

**estilos.css:**

```css
body {
    font-family: 'Roboto', sans-serif;
}

h1 {
    font-family: 'Merriweather', serif;
}
```

**Consejos:**
- No uses más de 2-3 fuentes distintas por página
- Combina serif para títulos + sans-serif para cuerpo (o viceversa)
- Fuentes populares: Roboto, Open Sans, Lato, Montserrat, Poppins, Merriweather

---

## Propiedades tipográficas

### `font-size` - Tamaño de fuente

```css
p {
    font-size: 16px;   /* Píxeles (fijo) */
    font-size: 1em;    /* Relativo al padre */
    font-size: 1rem;   /* Relativo a la raíz (html) */
}
```

**Tamaños típicos:**
```css
body {
    font-size: 16px; /* Base estándar */
}

h1 {
    font-size: 32px; /* o 2rem */
}

h2 {
    font-size: 24px; /* o 1.5rem */
}

small {
    font-size: 14px; /* o 0.875rem */
}
```

---

### `font-weight` - Grosor de la fuente

```css
p {
    font-weight: normal;  /* 400 */
    font-weight: bold;    /* 700 */
}

h1 {
    font-weight: 300;  /* Light */
    font-weight: 400;  /* Regular/Normal */
    font-weight: 500;  /* Medium */
    font-weight: 600;  /* Semi-bold */
    font-weight: 700;  /* Bold */
    font-weight: 900;  /* Black/Extra-bold */
}
```

**Nota:** Necesitas importar esos pesos desde Google Fonts:

```html
<!-- Solo 400 y 700 -->
<link href="https://fonts.googleapis.com/css2?family=Roboto:wght@400;700&display=swap" rel="stylesheet">

<!-- Múltiples pesos -->
<link href="https://fonts.googleapis.com/css2?family=Roboto:wght@300;400;500;700;900&display=swap" rel="stylesheet">
```

---

### `font-style` - Cursiva

```css
p {
    font-style: normal;  /* Normal */
    font-style: italic;  /* Cursiva */
}

em {
    font-style: italic; /* Por defecto */
}
```

---

### `text-align` - Alineación horizontal

```css
p {
    text-align: left;     /* Izquierda (por defecto) */
    text-align: center;   /* Centrado */
    text-align: right;    /* Derecha */
    text-align: justify;  /* Justificado */
}
```

---

### `text-decoration` - Subrayado, tachado

```css
a {
    text-decoration: none;        /* Sin decoración */
    text-decoration: underline;   /* Subrayado */
    text-decoration: overline;    /* Línea arriba */
    text-decoration: line-through;/* Tachado */
}

/* Múltiples */
text-decoration: underline overline;
```

**Uso común:** Quitar subrayado de enlaces.

```css
a {
    text-decoration: none;
}

a:hover {
    text-decoration: underline;
}
```

---

### `text-transform` - Mayúsculas/minúsculas

```css
p {
    text-transform: uppercase;   /* TODO MAYÚSCULAS */
    text-transform: lowercase;   /* todo minúsculas */
    text-transform: capitalize;  /* Primera Letra De Cada Palabra */
    text-transform: none;        /* Normal */
}
```

```html
<p class="mayus">hola mundo</p>
```

```css
.mayus {
    text-transform: uppercase;
}
```

Resultado: **HOLA MUNDO**

---

### `line-height` - Altura de línea (interlineado)

```css
p {
    line-height: 1.6;   /* 1.6 veces el tamaño de fuente */
    line-height: 24px;  /* Píxeles fijos */
}
```

**Sin unidad = mejor:**

```css
/* Recomendado */
p {
    font-size: 16px;
    line-height: 1.6; /* 16 * 1.6 = 25.6px */
}
```

**Valores típicos:** 1.4 - 1.8 para texto de cuerpo.

---

### `letter-spacing` - Espaciado entre letras

```css
h1 {
    letter-spacing: 2px;   /* Más espacio */
    letter-spacing: -1px;  /* Menos espacio */
}

.espaciado {
    letter-spacing: 0.1em;
}
```

**Uso común:** Títulos en mayúsculas.

```css
h1 {
    text-transform: uppercase;
    letter-spacing: 3px;
}
```

---

### `word-spacing` - Espaciado entre palabras

```css
p {
    word-spacing: 5px;
}
```

Menos usado que `letter-spacing`.

---

### `text-indent` - Sangría primera línea

```css
p {
    text-indent: 30px; /* Primera línea con sangría */
}
```

---

## Ejemplo completo: Blog estilizado

```html
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <title>Mi Blog</title>
    
    <!-- Google Fonts -->
    <link rel="preconnect" href="https://fonts.googleapis.com">
    <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
    <link href="https://fonts.googleapis.com/css2?family=Playfair+Display:wght@700&family=Open+Sans:wght@400;600&display=swap" rel="stylesheet">
    
    <link rel="stylesheet" href="blog.css">
</head>
<body>
    <header>
        <h1>El Mundo del Diseño</h1>
        <p class="subtitulo">Reflexiones sobre creatividad y estética</p>
    </header>
    
    <main>
        <article>
            <h2>El poder del color</h2>
            <p class="fecha">15 de marzo, 2026</p>
            <p>Los colores no son solo decorativos; comunican emociones, crean atmósferas y guían la atención del usuario.</p>
            <p>El rojo transmite urgencia, el azul confianza, el verde calma. Elegir bien puede ser la diferencia entre una página que retiene o que expulsa.</p>
        </article>
        
        <article>
            <h2>Tipografía legible</h2>
            <p class="fecha">12 de marzo, 2026</p>
            <p>Una fuente legible es invisible: el lector no piensa en las letras, solo en el contenido.</p>
            <p>Usa un <code>line-height</code> de al menos 1.5 para texto de cuerpo. Tus ojos lo agradecerán.</p>
        </article>
    </main>
</body>
</html>
```

```css
/* Variables de color */
:root {
    --color-primario: #2c3e50;
    --color-secundario: #e74c3c;
    --color-fondo: #ecf0f1;
    --color-texto: #34495e;
    --color-texto-claro: #7f8c8d;
}

/* Reset básico */
* {
    margin: 0;
    padding: 0;
    box-sizing: border-box;
}

body {
    font-family: 'Open Sans', sans-serif;
    font-size: 18px;
    line-height: 1.7;
    color: var(--color-texto);
    background-color: var(--color-fondo);
}

/* Header */
header {
    background: linear-gradient(135deg, var(--color-primario), #34495e);
    color: white;
    text-align: center;
    padding: 60px 20px;
}

h1 {
    font-family: 'Playfair Display', serif;
    font-size: 48px;
    font-weight: 700;
    margin-bottom: 15px;
    letter-spacing: 1px;
}

.subtitulo {
    font-size: 20px;
    font-weight: 400;
    color: rgba(255, 255, 255, 0.9);
    font-style: italic;
}

/* Main content */
main {
    max-width: 800px;
    margin: 40px auto;
    padding: 0 20px;
}

article {
    background-color: white;
    padding: 40px;
    margin-bottom: 30px;
    border-left: 5px solid var(--color-secundario);
}

article h2 {
    font-family: 'Playfair Display', serif;
    font-size: 32px;
    color: var(--color-primario);
    margin-bottom: 10px;
}

.fecha {
    color: var(--color-texto-claro);
    font-size: 14px;
    text-transform: uppercase;
    letter-spacing: 1px;
    margin-bottom: 20px;
}

article p {
    margin-bottom: 15px;
}

article p:last-child {
    margin-bottom: 0;
}

code {
    background-color: #f4f4f4;
    padding: 2px 6px;
    border-radius: 3px;
    font-family: 'Courier New', monospace;
    color: var(--color-secundario);
    font-size: 16px;
}
```

**Nota los detalles:**
- Fuentes diferentes para títulos (serif) y cuerpo (sans-serif)
- Variables CSS para colores consistentes
- `line-height` generoso (1.7) para legibilidad
- `letter-spacing` en títulos para elegancia
- Colores semitransparentes con `rgba()`

---

## Combinaciones de colores que funcionan

### 1. Monocromático (variaciones de un color)

```css
:root {
    --azul-oscuro: #1a5490;
    --azul-medio: #2980b9;
    --azul-claro: #5dade2;
    --azul-muy-claro: #aed6f1;
}
```

**Funciona siempre, es seguro pero puede ser aburrido.**

### 2. Colores complementarios (opuestos en el círculo)

- Azul (#3498db) + Naranja (#e67e22)
- Rojo (#e74c3c) + Verde (#2ecc71)
- Púrpura (#9b59b6) + Amarillo (#f1c40f)

**Alto contraste, visualmente impactante.**

### 3. Triádico (tres colores equidistantes)

- Rojo + Amarillo + Azul
- Naranja + Verde + Púrpura

**Vibrante pero equilibrado.**

### 4. Análogos (vecinos en el círculo)

- Azul + Verde-azulado + Verde
- Rojo + Naranja + Amarillo

**Armonioso, calmado.**

---

## Paletas listas para usar

### Paleta 1: Profesional

```css
:root {
    --primario: #2c3e50;     /* Azul oscuro */
    --secundario: #3498db;   /* Azul brillante */
    --acento: #e74c3c;       /* Rojo */
    --fondo: #ecf0f1;        /* Gris muy claro */
    --texto: #2c3e50;        /* Azul oscuro */
}
```

### Paleta 2: Moderna

```css
:root {
    --primario: #1a1a2e;     /* Casi negro */
    --secundario: #16213e;   /* Azul muy oscuro */
    --acento: #0f3460;       /* Azul oscuro */
    --destaque: #e94560;     /* Rosa/rojo */
    --fondo: #f5f5f5;        /* Gris claro */
}
```

### Paleta 3: Cálida

```css
:root {
    --primario: #ff6b6b;     /* Rojo coral */
    --secundario: #feca57;   /* Amarillo */
    --acento: #48dbfb;       /* Azul cielo */
    --fondo: #fff8e1;        /* Amarillo muy claro */
    --texto: #2d3436;        /* Gris oscuro */
}
```

---

## Accesibilidad: Contraste

**El texto debe tener suficiente contraste con el fondo.**

### Ratios de contraste

- **AA (mínimo):** 4.5:1 para texto normal, 3:1 para texto grande
- **AAA (ideal):** 7:1 para texto normal, 4.5:1 para texto grande

### Malas combinaciones

```css
/* Mal contraste */
.malo {
    color: #cccccc;             /* Gris claro */
    background-color: #ffffff;  /* Blanco */
}

.tambien-malo {
    color: yellow;
    background-color: white;    /* Apenas se lee */
}
```

### Buenas combinaciones

```css
/* Buen contraste */
.bueno {
    color: #333333;             /* Gris oscuro */
    background-color: #ffffff;  /* Blanco */
}

.tambien-bueno {
    color: #ffffff;             /* Blanco */
    background-color: #2c3e50;  /* Azul oscuro */
}
```

**Herramientas:**
- [WebAIM Contrast Checker](https://webaim.org/resources/contrastchecker/)
- [Contrast Ratio](https://contrast-ratio.com/)

---

## Buenas prácticas

### 1. Define una paleta al inicio

```css
:root {
    --primario: #3498db;
    --secundario: #2c3e50;
    --acento: #e74c3c;
    --texto: #333333;
    --fondo: #f5f5f5;
}

body {
    background-color: var(--fondo);
    color: var(--texto);
}

h1 {
    color: var(--primario);
}

button {
    background-color: var(--acento);
}
```

### 2. Usa no más de 3-4 colores principales

Más allá de eso, se vuelve caótico.

### 3. Jerarquía tipográfica clara

```css
h1 {
    font-size: 32px;
    font-weight: 700;
}

h2 {
    font-size: 24px;
    font-weight: 600;
}

h3 {
    font-size: 20px;
    font-weight: 600;
}

p {
    font-size: 16px;
    font-weight: 400;
    line-height: 1.6;
}
```

### 4. Texto de cuerpo: 16-18px mínimo

```css
body {
    font-size: 16px; /* Mínimo */
    line-height: 1.6;
}
```

### 5. No uses negro puro (#000) para texto

```css
/* Evita */
color: #000000;

/* Mejor */
color: #333333; /* Gris muy oscuro, más suave para los ojos */
```

---

## Errores comunes

### Error 1: Demasiados colores

```css
/* Mal */
body { background: red; }
header { background: blue; }
section { background: green; }
aside { background: yellow; }
footer { background: purple; }
```

Parece un arcoíris explotó.

### Error 2: Texto sin contraste

```css
/* Ilegible */
p {
    color: #ccc;
    background-color: white;
}
```

### Error 3: Fuentes muy pequeñas

```css
/* Mal */
p {
    font-size: 12px; /* Demasiado pequeño para lectura cómoda */
}
```

### Error 4: Usar 10 fuentes diferentes

```css
/* Mal */
h1 { font-family: 'Fuente1', serif; }
h2 { font-family: 'Fuente2', sans-serif; }
h3 { font-family: 'Fuente3', monospace; }
p { font-family: 'Fuente4', cursive; }
/* ... */
```

Parece una nota de rescate.

**Límite recomendado: 2 fuentes (una para títulos, otra para cuerpo).**

---

## Ejercicios prácticos

### Ejercicio 1: Paleta personalizada

Crea una página "Acerca de mí" con:
- Paleta de 3 colores usando variables CSS
- Fondo con un color
- Títulos con otro color
- Elementos destacados con el color de acento

<details>
<summary>✅ Solución</summary>

```html
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <title>Sobre mí</title>
    <link rel="stylesheet" href="estilos.css">
</head>
<body>
    <header>
        <h1>Juan Pérez</h1>
        <p class="subtitulo">Desarrollador web en formación</p>
    </header>
    
    <main>
        <section>
            <h2>Sobre mí</h2>
            <p>Me apasiona crear <span class="destacado">experiencias web</span> que sean útiles y hermosas.</p>
        </section>
    </main>
</body>
</html>
```

```css
:root {
    --primario: #2c3e50;
    --secundario: #3498db;
    --acento: #e74c3c;
    --fondo: #ecf0f1;
}

body {
    margin: 0;
    font-family: Arial, sans-serif;
    background-color: var(--fondo);
    color: var(--primario);
}

header {
    background-color: var(--primario);
    color: white;
    padding: 40px;
    text-align: center;
}

h1 {
    font-size: 36px;
}

.subtitulo {
    color: var(--secundario);
    font-size: 18px;
}

main {
    padding: 40px;
    max-width: 800px;
    margin: 0 auto;
}

h2 {
    color: var(--secundario);
}

.destacado {
    color: var(--acento);
    font-weight: bold;
}
```

</details>

---

### Ejercicio 2: Google Fonts

Usa dos fuentes de Google Fonts:
- Una serif para títulos
- Una sans-serif para el cuerpo

Crea una página de "Receta" estilizada.

<details>
<summary>💡 Pista</summary>

Busca en Google Fonts: Merriweather (serif) + Open Sans (sans-serif).

</details>

---

### Ejercicio 3: Efectos de hover

Crea un menú de navegación donde:
- Los enlaces cambian de color al hacer hover
- El fondo cambia suavemente (usa `transition`)
- El texto tiene buen contraste en ambos estados

<details>
<summary>✅ Solución</summary>

```html
<nav>
    <a href="#">Inicio</a>
    <a href="#">Productos</a>
    <a href="#">Contacto</a>
</nav>
```

```css
nav {
    background-color: #2c3e50;
    padding: 0;
}

nav a {
    display: inline-block;
    padding: 15px 20px;
    color: white;
    text-decoration: none;
    transition: background-color 0.3s, color 0.3s;
}

nav a:hover {
    background-color: #3498db;
    color: white;
}
```

</details>

---

## Cheat sheet

### Colores

```css
color: red;                         /* Nombre */
color: #ff0000;                     /* Hex */
color: #f00;                        /* Hex corto */
color: rgb(255, 0, 0);              /* RGB */
color: rgba(255, 0, 0, 0.5);        /* RGBA (con transparencia) */
color: hsl(0, 100%, 50%);           /* HSL */
color: hsla(0, 100%, 50%, 0.5);     /* HSLA */
```

### Tipografía

```css
font-family: Arial, sans-serif;
font-size: 16px;
font-weight: bold;                  /* o 400, 700, etc. */
font-style: italic;
text-align: center;
text-decoration: underline;
text-transform: uppercase;
line-height: 1.6;
letter-spacing: 2px;
```

---

## Recursos adicionales

- [Google Fonts](https://fonts.google.com/)
- [Coolors](https://coolors.co/) - Generador de paletas
- [ColorHunt](https://colorhunt.co/) - Paletas curadas
- [Contrast Checker](https://webaim.org/resources/contrastchecker/)
- [Font Pair](https://www.fontpair.co/) - Combinaciones de fuentes

---

## Siguiente paso

Ya dominas colores y tipografía. Ahora el concepto más importante de CSS: **el Box Model**.

→ [09-box-model.md](09-box-model.md)

Ahí entenderás cómo funciona el espaciado (márgenes, bordes, padding) y por qué los elementos ocupan el espacio que ocupan.

---

**Recuerda:** El diseño no es decoración. Es comunicación. Usa colores y fuentes para guiar, no para distraer.

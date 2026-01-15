# Diseño Responsive Básico

**Responsive = Tu página se adapta a cualquier tamaño de pantalla.**

Móvil, tablet, laptop, monitor grande... una sola página, muchos tamaños. Bienvenido al diseño responsive.

---

## ¿Qué vas a aprender?

- Qué es diseño responsive y por qué importa
- Meta viewport tag (esencial para móviles)
- Media queries: CSS que cambia según el tamaño de pantalla
- Breakpoints comunes (móvil, tablet, desktop)
- Mobile-first vs Desktop-first
- Imágenes y videos responsivos

## Por qué es útil

**Datos del mundo real:**
- Más del 60% del tráfico web viene de móviles
- Google penaliza sitios no responsive
- Los usuarios abandonan sitios que no funcionan en su dispositivo

**Analogía:** Es como diseñar ropa con tallas (S, M, L, XL) en lugar de talla única. La misma prenda, ajustada a diferentes cuerpos.

---

## El meta viewport tag

**SIEMPRE incluye esto en el `<head>` de tu HTML:**

```html
<meta name="viewport" content="width=device-width, initial-scale=1.0">
```

### ¿Qué hace?

Sin este tag, los móviles simulan una pantalla de 980px y tu página se ve diminuta (tienes que hacer zoom).

Con este tag, el móvil usa su ancho real (375px, 414px, etc.) y tu CSS responsive funciona.

**Ejemplo completo:**

```html
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Mi página responsive</title>
    <link rel="stylesheet" href="estilos.css">
</head>
<body>
    <!-- Tu contenido -->
</body>
</html>
```

**Nunca olvides esta línea.**

---

## Media Queries: CSS condicional

**Media queries = "Si la pantalla es X, aplica estos estilos".**

### Sintaxis básica

```css
/* Estilos base (para todos) */
body {
    font-size: 16px;
}

/* Si la pantalla es al menos 768px de ancho */
@media (min-width: 768px) {
    body {
        font-size: 18px;
    }
}
```

**Funciona así:**
- Móvil (menos de 768px) → `font-size: 16px`
- Tablet/Desktop (768px o más) → `font-size: 18px`

---

## Breakpoints comunes

Los puntos donde tu diseño cambia.

```css
/* Móvil pequeño (por defecto, sin media query) */
body {
    font-size: 14px;
}

/* Tablet (768px en adelante) */
@media (min-width: 768px) {
    body {
        font-size: 16px;
    }
}

/* Desktop (1024px en adelante) */
@media (min-width: 1024px) {
    body {
        font-size: 18px;
    }
}

/* Desktop grande (1440px en adelante) */
@media (min-width: 1440px) {
    body {
        font-size: 20px;
    }
}
```

### Tabla de breakpoints estándar

| Dispositivo | Rango | Breakpoint |
|-------------|-------|------------|
| Móvil pequeño | 320-479px | Por defecto |
| Móvil grande | 480-767px | `@media (min-width: 480px)` |
| Tablet | 768-1023px | `@media (min-width: 768px)` |
| Desktop | 1024-1439px | `@media (min-width: 1024px)` |
| Desktop grande | 1440px+ | `@media (min-width: 1440px)` |

**No son reglas estrictas.** Ajusta según tu diseño.

---

## Mobile-First vs Desktop-First

### Mobile-First (RECOMENDADO)

Empiezas con estilos para móvil, luego añades para pantallas más grandes.

```css
/* Base: Móvil */
.contenedor {
    width: 100%;
    padding: 10px;
}

.menu {
    flex-direction: column;
}

/* Tablet en adelante */
@media (min-width: 768px) {
    .contenedor {
        width: 750px;
        margin: 0 auto;
    }
    
    .menu {
        flex-direction: row;
    }
}

/* Desktop en adelante */
@media (min-width: 1024px) {
    .contenedor {
        width: 970px;
    }
}
```

**Ventajas:**
- Más eficiente (móviles descargan menos CSS)
- Fuerza a priorizar lo esencial
- Más fácil de mantener

### Desktop-First

Empiezas con estilos para desktop, luego adaptas a móvil.

```css
/* Base: Desktop */
.contenedor {
    width: 1200px;
    margin: 0 auto;
}

/* Tablet y menos */
@media (max-width: 1023px) {
    .contenedor {
        width: 100%;
        padding: 20px;
    }
}

/* Móvil y menos */
@media (max-width: 767px) {
    .contenedor {
        padding: 10px;
    }
}
```

**Desventaja:** Móviles descargan CSS que no necesitan.

**Usa mobile-first siempre que puedas.**

---

## Ejemplo práctico: Menú responsive

```html
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Menú Responsive</title>
    <link rel="stylesheet" href="menu.css">
</head>
<body>
    <nav class="menu">
        <div class="logo">MiSitio</div>
        <ul class="enlaces">
            <li><a href="#">Inicio</a></li>
            <li><a href="#">Servicios</a></li>
            <li><a href="#">Productos</a></li>
            <li><a href="#">Contacto</a></li>
        </ul>
    </nav>
</body>
</html>
```

```css
/* Reset */
* {
    margin: 0;
    padding: 0;
    box-sizing: border-box;
}

/* MÓVIL (por defecto) */
.menu {
    background-color: #2c3e50;
    padding: 15px;
}

.logo {
    color: white;
    font-size: 24px;
    font-weight: bold;
    margin-bottom: 15px;
}

.enlaces {
    list-style: none;
    display: flex;
    flex-direction: column;  /* Vertical en móvil */
    gap: 10px;
}

.enlaces a {
    color: white;
    text-decoration: none;
    padding: 10px;
    display: block;
    background-color: #34495e;
    border-radius: 5px;
}

/* TABLET en adelante (768px+) */
@media (min-width: 768px) {
    .menu {
        display: flex;
        justify-content: space-between;
        align-items: center;
        padding: 20px 40px;
    }
    
    .logo {
        margin-bottom: 0;
    }
    
    .enlaces {
        flex-direction: row;  /* Horizontal en tablet+ */
    }
    
    .enlaces a {
        background-color: transparent;
    }
    
    .enlaces a:hover {
        background-color: #34495e;
    }
}
```

**Resultado:**
- **Móvil:** Logo arriba, enlaces verticales debajo
- **Tablet+:** Logo a la izquierda, enlaces horizontales a la derecha

---

## Ejemplo: Grid responsivo

```html
<div class="galeria">
    <div class="item">1</div>
    <div class="item">2</div>
    <div class="item">3</div>
    <div class="item">4</div>
    <div class="item">5</div>
    <div class="item">6</div>
</div>
```

```css
/* MÓVIL: 1 columna */
.galeria {
    display: grid;
    grid-template-columns: 1fr;
    gap: 15px;
    padding: 15px;
}

.item {
    background-color: #3498db;
    color: white;
    padding: 40px;
    text-align: center;
    border-radius: 8px;
}

/* TABLET: 2 columnas */
@media (min-width: 768px) {
    .galeria {
        grid-template-columns: repeat(2, 1fr);
        padding: 20px;
    }
}

/* DESKTOP: 3 columnas */
@media (min-width: 1024px) {
    .galeria {
        grid-template-columns: repeat(3, 1fr);
        padding: 30px;
    }
}
```

**Resultado:**
- Móvil: 1 columna
- Tablet: 2 columnas
- Desktop: 3 columnas

---

## Grid responsivo automático (sin media queries)

La forma moderna y más simple:

```css
.galeria {
    display: grid;
    grid-template-columns: repeat(auto-fit, minmax(300px, 1fr));
    gap: 20px;
    padding: 20px;
}
```

**Se adapta solo:**
- Si caben 3 columnas de 300px → 3 columnas
- Si caben 2 → 2 columnas
- Si caben 1 → 1 columna

**Sin una sola media query.**

---

## Imágenes responsivas

### 1. Ancho fluido

```css
img {
    max-width: 100%;  /* Nunca más grande que su contenedor */
    height: auto;     /* Mantiene proporción */
}
```

**Aplica esto a TODAS tus imágenes.**

### 2. Elemento `<picture>` (imágenes diferentes según pantalla)

```html
<picture>
    <!-- Móvil: imagen pequeña -->
    <source media="(max-width: 767px)" srcset="imagen-movil.jpg">
    
    <!-- Tablet: imagen mediana -->
    <source media="(max-width: 1023px)" srcset="imagen-tablet.jpg">
    
    <!-- Desktop: imagen grande -->
    <img src="imagen-desktop.jpg" alt="Descripción">
</picture>
```

Ahorra ancho de banda en móviles.

### 3. `srcset` y `sizes`

```html
<img 
    src="imagen-800.jpg"
    srcset="
        imagen-400.jpg 400w,
        imagen-800.jpg 800w,
        imagen-1200.jpg 1200w
    "
    sizes="(max-width: 768px) 100vw, 800px"
    alt="Descripción"
>
```

El navegador elige la imagen óptima.

---

## Videos responsivos

### Método 1: Ancho fluido

```css
video {
    max-width: 100%;
    height: auto;
}
```

### Método 2: Contenedor con ratio (para iframes de YouTube)

```html
<div class="video-responsive">
    <iframe src="https://www.youtube.com/embed/..." frameborder="0"></iframe>
</div>
```

```css
.video-responsive {
    position: relative;
    padding-bottom: 56.25%;  /* Ratio 16:9 (9/16 = 0.5625) */
    height: 0;
    overflow: hidden;
}

.video-responsive iframe {
    position: absolute;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
}
```

---

## Contenedores con ancho máximo

```css
.contenedor {
    max-width: 1200px;  /* Máximo 1200px */
    margin: 0 auto;     /* Centrado */
    padding: 0 20px;    /* Padding lateral en móviles */
}
```

**Comportamiento:**
- Pantalla grande (1400px): Contenedor de 1200px, centrado
- Pantalla media (1000px): Contenedor de 1000px (100%), centrado
- Pantalla pequeña (600px): Contenedor de 560px (600 - 40px de padding)

---

## Ocultar/Mostrar elementos según pantalla

```css
/* MÓVIL */
.solo-desktop {
    display: none;  /* Oculto en móvil */
}

.solo-movil {
    display: block;  /* Visible en móvil */
}

/* DESKTOP (1024px+) */
@media (min-width: 1024px) {
    .solo-desktop {
        display: block;  /* Visible en desktop */
    }
    
    .solo-movil {
        display: none;  /* Oculto en desktop */
    }
}
```

```html
<p class="solo-movil">Estás en móvil</p>
<p class="solo-desktop">Estás en desktop</p>
```

---

## Tipografía responsiva

```css
/* MÓVIL */
body {
    font-size: 16px;
}

h1 {
    font-size: 28px;
    line-height: 1.2;
}

h2 {
    font-size: 22px;
}

/* TABLET (768px+) */
@media (min-width: 768px) {
    body {
        font-size: 18px;
    }
    
    h1 {
        font-size: 36px;
    }
    
    h2 {
        font-size: 28px;
    }
}

/* DESKTOP (1024px+) */
@media (min-width: 1024px) {
    h1 {
        font-size: 48px;
        line-height: 1.1;
    }
    
    h2 {
        font-size: 32px;
    }
}
```

---

## Espaciado responsivo

```css
/* MÓVIL */
.seccion {
    padding: 30px 15px;
}

.tarjeta {
    margin-bottom: 20px;
}

/* TABLET (768px+) */
@media (min-width: 768px) {
    .seccion {
        padding: 50px 30px;
    }
    
    .tarjeta {
        margin-bottom: 30px;
    }
}

/* DESKTOP (1024px+) */
@media (min-width: 1024px) {
    .seccion {
        padding: 80px 60px;
    }
    
    .tarjeta {
        margin-bottom: 40px;
    }
}
```

---

## Ejemplo completo: Landing page responsive

```html
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Landing Responsive</title>
    <link rel="stylesheet" href="landing.css">
</head>
<body>
    <header class="hero">
        <div class="contenedor">
            <h1>Bienvenido a MiServicio</h1>
            <p>La solución que necesitas</p>
            <button>Comenzar ahora</button>
        </div>
    </header>
    
    <section class="caracteristicas">
        <div class="contenedor">
            <div class="caracteristica">
                <h3>Rápido</h3>
                <p>Velocidad increíble</p>
            </div>
            <div class="caracteristica">
                <h3>Seguro</h3>
                <p>Tus datos protegidos</p>
            </div>
            <div class="caracteristica">
                <h3>Confiable</h3>
                <p>Disponible 24/7</p>
            </div>
        </div>
    </section>
</body>
</html>
```

```css
/* Reset */
* {
    margin: 0;
    padding: 0;
    box-sizing: border-box;
}

body {
    font-family: Arial, sans-serif;
}

/* Contenedor reutilizable */
.contenedor {
    max-width: 1200px;
    margin: 0 auto;
    padding: 0 20px;
}

/* HERO (MÓVIL) */
.hero {
    background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
    color: white;
    text-align: center;
    padding: 60px 20px;
}

.hero h1 {
    font-size: 28px;
    margin-bottom: 15px;
}

.hero p {
    font-size: 16px;
    margin-bottom: 25px;
}

.hero button {
    background-color: white;
    color: #667eea;
    padding: 12px 30px;
    border: none;
    border-radius: 25px;
    font-size: 16px;
    font-weight: bold;
    cursor: pointer;
}

/* CARACTERÍSTICAS (MÓVIL) */
.caracteristicas {
    padding: 40px 20px;
    background-color: #f5f5f5;
}

.caracteristicas .contenedor {
    display: flex;
    flex-direction: column;  /* Vertical en móvil */
    gap: 30px;
}

.caracteristica {
    background-color: white;
    padding: 30px;
    border-radius: 8px;
    text-align: center;
    box-shadow: 0 2px 4px rgba(0,0,0,0.1);
}

.caracteristica h3 {
    color: #667eea;
    margin-bottom: 10px;
    font-size: 20px;
}

/* TABLET (768px+) */
@media (min-width: 768px) {
    .hero h1 {
        font-size: 42px;
    }
    
    .hero p {
        font-size: 20px;
    }
    
    .caracteristicas {
        padding: 60px 30px;
    }
    
    .caracteristicas .contenedor {
        flex-direction: row;  /* Horizontal en tablet+ */
    }
    
    .caracteristica {
        flex: 1;  /* Ocupan el mismo espacio */
    }
}

/* DESKTOP (1024px+) */
@media (min-width: 1024px) {
    .hero {
        padding: 120px 20px;
    }
    
    .hero h1 {
        font-size: 56px;
    }
    
    .hero p {
        font-size: 24px;
    }
    
    .hero button {
        padding: 15px 40px;
        font-size: 18px;
    }
    
    .caracteristicas {
        padding: 80px 30px;
    }
    
    .caracteristica h3 {
        font-size: 24px;
    }
}
```

---

## Herramientas para probar responsive

### DevTools del navegador

1. Abre DevTools (F12)
2. Click en el icono de "Toggle device toolbar" (o Ctrl+Shift+M)
3. Selecciona dispositivos preconfigurados (iPhone, iPad, etc.)
4. O arrastra para redimensionar

### Breakpoints para probar

- **iPhone SE:** 375px
- **iPhone 12/13:** 390px
- **iPad:** 768px
- **iPad Pro:** 1024px
- **Desktop:** 1440px

---

## Buenas prácticas

### 1. Siempre incluye el meta viewport

```html
<meta name="viewport" content="width=device-width, initial-scale=1.0">
```

### 2. Mobile-first

```css
/* Base: Móvil */
.elemento { font-size: 14px; }

/* Tablet+ */
@media (min-width: 768px) {
    .elemento { font-size: 16px; }
}
```

### 3. Imágenes fluidas

```css
img {
    max-width: 100%;
    height: auto;
}
```

### 4. Toca targets de al menos 44×44px en móvil

```css
button, a {
    min-height: 44px;
    padding: 12px 20px;
}
```

Los dedos necesitan espacio.

### 5. Usa unidades relativas

```css
/* Bien */
font-size: 1rem;
padding: 2em;

/* Menos flexible */
font-size: 16px;
padding: 32px;
```

---

## Errores comunes

### Error 1: Olvidar el meta viewport

Sin esto, el responsive no funciona en móviles.

### Error 2: Anchos fijos en elementos que deberían fluir

```css
/* Mal */
.contenedor {
    width: 1200px;  /* Se desborda en móviles */
}

/* Bien */
.contenedor {
    max-width: 1200px;
    width: 100%;
}
```

### Error 3: Texto demasiado pequeño en móvil

```css
/* Mal */
body {
    font-size: 12px;  /* Difícil de leer en móvil */
}

/* Bien */
body {
    font-size: 16px;  /* Mínimo recomendado */
}
```

### Error 4: No probar en dispositivos reales

DevTools es bueno, pero prueba en móviles reales cuando puedas.

---

## Ejercicios prácticos

### Ejercicio 1: Tarjetas responsivas

Crea una sección con 3 tarjetas que:
- En móvil: 1 columna (verticales)
- En tablet: 2 columnas (primera fila 2, segunda 1)
- En desktop: 3 columnas (horizontales)

<details>
<summary>✅ Solución</summary>

```html
<div class="tarjetas">
    <div class="tarjeta">Tarjeta 1</div>
    <div class="tarjeta">Tarjeta 2</div>
    <div class="tarjeta">Tarjeta 3</div>
</div>
```

```css
/* MÓVIL */
.tarjetas {
    display: grid;
    grid-template-columns: 1fr;
    gap: 20px;
    padding: 20px;
}

.tarjeta {
    background-color: white;
    padding: 30px;
    border-radius: 8px;
    box-shadow: 0 2px 4px rgba(0,0,0,0.1);
}

/* TABLET (768px+) */
@media (min-width: 768px) {
    .tarjetas {
        grid-template-columns: repeat(2, 1fr);
    }
}

/* DESKTOP (1024px+) */
@media (min-width: 1024px) {
    .tarjetas {
        grid-template-columns: repeat(3, 1fr);
    }
}
```

</details>

---

### Ejercicio 2: Menú hamburguesa (concepto)

Crea un menú que:
- En móvil: Muestra solo el logo (simula que habrá un botón hamburguesa)
- En desktop: Muestra logo + enlaces horizontales

<details>
<summary>✅ Solución</summary>

```html
<nav class="menu">
    <div class="logo">MiLogo</div>
    <ul class="enlaces">
        <li><a href="#">Inicio</a></li>
        <li><a href="#">Servicios</a></li>
        <li><a href="#">Contacto</a></li>
    </ul>
</nav>
```

```css
/* MÓVIL */
.menu {
    background-color: #2c3e50;
    padding: 15px 20px;
}

.logo {
    color: white;
    font-size: 24px;
    font-weight: bold;
}

.enlaces {
    display: none;  /* Oculto en móvil (en realidad estaría en un menú desplegable) */
}

/* DESKTOP (768px+) */
@media (min-width: 768px) {
    .menu {
        display: flex;
        justify-content: space-between;
        align-items: center;
        padding: 20px 40px;
    }
    
    .enlaces {
        display: flex;
        list-style: none;
        gap: 30px;
        margin: 0;
        padding: 0;
    }
    
    .enlaces a {
        color: white;
        text-decoration: none;
    }
}
```

**Nota:** En el próximo bloque (JavaScript) aprenderás a hacer el botón hamburguesa funcional.

</details>

---

### Ejercicio 3: Hero responsive

Crea un hero que:
- En móvil: Título 28px, padding 40px vertical
- En desktop: Título 56px, padding 100px vertical

<details>
<summary>✅ Solución</summary>

```html
<section class="hero">
    <h1>Bienvenido</h1>
    <p>Tu sitio web increíble</p>
</section>
```

```css
/* MÓVIL */
.hero {
    background: linear-gradient(135deg, #667eea, #764ba2);
    color: white;
    text-align: center;
    padding: 40px 20px;
}

.hero h1 {
    font-size: 28px;
    margin-bottom: 15px;
}

.hero p {
    font-size: 16px;
}

/* DESKTOP (1024px+) */
@media (min-width: 1024px) {
    .hero {
        padding: 100px 20px;
    }
    
    .hero h1 {
        font-size: 56px;
        margin-bottom: 20px;
    }
    
    .hero p {
        font-size: 24px;
    }
}
```

</details>

---

## Cheat sheet de Responsive

```html
<!-- Meta viewport (OBLIGATORIO) -->
<meta name="viewport" content="width=device-width, initial-scale=1.0">
```

```css
/* Media queries comunes */
@media (min-width: 480px) { /* Móvil grande */ }
@media (min-width: 768px) { /* Tablet */ }
@media (min-width: 1024px) { /* Desktop */ }
@media (min-width: 1440px) { /* Desktop grande */ }

/* Imágenes fluidas */
img {
    max-width: 100%;
    height: auto;
}

/* Contenedor con ancho máximo */
.contenedor {
    max-width: 1200px;
    margin: 0 auto;
    padding: 0 20px;
}

/* Grid responsivo automático */
.grid {
    display: grid;
    grid-template-columns: repeat(auto-fit, minmax(300px, 1fr));
    gap: 20px;
}

/* Ocultar/Mostrar según pantalla */
.solo-movil { display: block; }
.solo-desktop { display: none; }

@media (min-width: 768px) {
    .solo-movil { display: none; }
    .solo-desktop { display: block; }
}
```

---

## Recursos adicionales

- [MDN: Responsive Design](https://developer.mozilla.org/es/docs/Learn/CSS/CSS_layout/Responsive_Design)
- [CSS Tricks: Media Queries](https://css-tricks.com/a-complete-guide-to-css-media-queries/)
- [Responsive Design Checker](https://responsivedesignchecker.com/)
- [Can I Use](https://caniuse.com/) - Compatibilidad de navegadores

---

## Siguiente paso

**¡Felicidades!** Has completado el **Bloque 2: CSS** del módulo de fundamentos.

Ya sabes:
- ✅ HTML (estructura)
- ✅ CSS (diseño y estilo)

Ahora viene lo que hace que tu página sea **interactiva**: **JavaScript**.

→ [Bloque 3: JavaScript básico](../bloque-03-javascript-basico/)

Ahí empezarás a programar de verdad: variables, funciones, lógica, y mucho más.

---

**Recuerda:** El diseño responsive no es opcional. Es esencial. Siempre incluye el meta viewport y prueba en diferentes tamaños de pantalla.

---

## Resumen del Bloque CSS

Has aprendido:

1. **06-introduccion-css.md:** Qué es CSS, cómo conectarlo, sintaxis básica
2. **07-selectores-css.md:** Clases, IDs, selectores combinados, pseudo-clases
3. **08-colores-tipografia.md:** Sistemas de color, fuentes, Google Fonts
4. **09-box-model.md:** El concepto más importante (padding, border, margin)
5. **10-flexbox.md:** Layout moderno en una dimensión
6. **11-grid-basico.md:** Layout en dos dimensiones (filas y columnas)
7. **12-responsive-basico.md:** Adaptar diseños a cualquier pantalla

**Con esto, puedes crear sitios web visualmente atractivos y profesionales.**

Ahora, a darles vida con JavaScript 🚀

# Introducción a CSS

Ya sabes crear estructura con HTML. Ahora vas a hacerla **bonita**.

**CSS = Cascading Style Sheets (Hojas de Estilo en Cascada)**

---

## ¿Qué vas a aprender?

- Qué es CSS y para qué sirve
- Cómo conectar CSS con HTML (3 formas)
- Sintaxis básica: selectores, propiedades, valores
- Tu primer estilo aplicado
- Comentarios en CSS

## Por qué es útil

HTML sin CSS es como una casa sin pintar: funcional pero aburrida.

**Analogía:** HTML es la estructura (paredes, puertas, ventanas). CSS es la decoración (colores, muebles, cortinas).

Con CSS puedes:
- Cambiar colores y fuentes
- Controlar espaciado y tamaños
- Crear layouts (organizar elementos en la página)
- Hacer animaciones
- Adaptar tu diseño a móviles y tablets

---

## Qué es CSS

CSS es un lenguaje para **describir cómo se ven** los elementos HTML.

**HTML dice QUÉ es algo:**
```html
<h1>Título</h1>
<p>Párrafo</p>
```

**CSS dice CÓMO se ve:**
```css
h1 {
    color: blue;
    font-size: 32px;
}

p {
    color: gray;
    font-size: 16px;
}
```

Resultado: Título azul grande, párrafo gris más pequeño.

---

## Tres formas de aplicar CSS

### 1. CSS en línea (inline) — ❌ No recomendado

Estilos directamente en la etiqueta HTML con el atributo `style`.

```html
<p style="color: red; font-size: 20px;">Este texto es rojo</p>
```

**Ventaja:** Rápido para pruebas.  
**Desventaja:** Difícil de mantener. Si tienes 100 párrafos rojos, tienes que cambiarlos uno por uno.

**Usa esto solo para pruebas rápidas o casos muy específicos.**

---

### 2. CSS interno (en el `<head>`) — ⚠️ Solo para pruebas

Estilos dentro de una etiqueta `<style>` en el HTML.

```html
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <title>Mi página</title>
    <style>
        p {
            color: red;
            font-size: 20px;
        }
        
        h1 {
            color: blue;
        }
    </style>
</head>
<body>
    <h1>Título</h1>
    <p>Este párrafo será rojo.</p>
    <p>Este también será rojo.</p>
</body>
</html>
```

**Ventaja:** Todo en un archivo. Útil para páginas muy simples.  
**Desventaja:** Si tienes varias páginas, tienes que copiar los estilos en cada una.

**Usa esto solo para experimentar o páginas de una sola página.**

---

### 3. CSS externo (archivo separado) — ✅ RECOMENDADO

Estilos en un archivo `.css` separado que vinculas con `<link>`.

**Archivo `estilos.css`:**
```css
p {
    color: red;
    font-size: 20px;
}

h1 {
    color: blue;
}
```

**Archivo `index.html`:**
```html
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <title>Mi página</title>
    <link rel="stylesheet" href="estilos.css">
</head>
<body>
    <h1>Título</h1>
    <p>Este párrafo será rojo.</p>
    <p>Este también será rojo.</p>
</body>
</html>
```

**Ventajas:**
- Reutilizable en múltiples páginas HTML
- Fácil de mantener (un solo archivo para todos los estilos)
- El navegador lo guarda en caché (carga más rápido)

**Esta es la forma profesional. Úsala siempre.**

---

## Sintaxis CSS básica

```css
selector {
    propiedad: valor;
    otra-propiedad: otro-valor;
}
```

**Partes:**
- **Selector:** A qué elemento HTML aplicar los estilos
- **Propiedad:** Qué aspecto cambiar (color, tamaño, margen, etc.)
- **Valor:** El valor específico para esa propiedad
- **;** (punto y coma) al final de cada línea

**Ejemplo:**
```css
h1 {
    color: blue;
    font-size: 32px;
    text-align: center;
}
```

Esto dice: "Todos los `<h1>` serán azules, de 32 píxeles de tamaño, y centrados".

---

## Tu primer archivo CSS

Vamos a crear una página completa con estilos.

### Paso 1: Crea `index.html`

```html
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <title>Mi primera página con CSS</title>
    <link rel="stylesheet" href="estilos.css">
</head>
<body>
    <h1>Bienvenido a mi sitio</h1>
    <p>Este es mi primer párrafo con CSS.</p>
    <p>Y este es el segundo.</p>
</body>
</html>
```

### Paso 2: Crea `estilos.css` (en la misma carpeta)

```css
/* Estilos para el título */
h1 {
    color: #2c3e50;
    font-size: 36px;
    text-align: center;
}

/* Estilos para los párrafos */
p {
    color: #34495e;
    font-size: 18px;
    line-height: 1.6;
}

/* Estilos para el body (toda la página) */
body {
    background-color: #ecf0f1;
    font-family: Arial, sans-serif;
    padding: 20px;
}
```

### Paso 3: Abre `index.html` en tu navegador

Deberías ver:
- Fondo gris claro
- Título oscuro, grande y centrado
- Párrafos con texto espaciado y legible

**¡Felicidades! Ya estás usando CSS.**

---

## Comentarios en CSS

Los comentarios no se muestran en la página. Son para ti y otros desarrolladores.

```css
/* Este es un comentario de una línea */

/*
  Este es un comentario
  de varias líneas
*/

h1 {
    color: blue; /* Esto también es un comentario */
}
```

**Usa comentarios para:**
- Explicar por qué hiciste algo
- Dividir tu CSS en secciones
- Desactivar temporalmente estilos sin borrarlos

---

## Selectores básicos

### Selector de elemento (etiqueta)

Aplica estilos a **todas** las etiquetas de ese tipo.

```css
p {
    color: blue;
}
```

Todos los `<p>` serán azules.

```css
h1 {
    color: red;
}
```

Todos los `<h1>` serán rojos.

### Múltiples selectores

Si quieres aplicar los mismos estilos a varios elementos:

```css
h1, h2, h3 {
    color: navy;
    font-family: Georgia, serif;
}
```

Todos los títulos (`<h1>`, `<h2>`, `<h3>`) tendrán el mismo color y fuente.

---

## Propiedades CSS comunes

### Color de texto

```css
p {
    color: red;        /* Nombre del color */
    color: #ff0000;    /* Código hexadecimal */
    color: rgb(255, 0, 0); /* RGB */
}
```

### Tamaño de fuente

```css
p {
    font-size: 16px;   /* Píxeles */
    font-size: 1em;    /* Relativo al tamaño padre */
    font-size: 1.2rem; /* Relativo al tamaño raíz */
}
```

Por ahora, usa `px` (píxeles). Es lo más simple.

### Familia de fuente (tipo de letra)

```css
p {
    font-family: Arial, sans-serif;
}
```

Si Arial no está disponible, usa cualquier sans-serif (sin serifa).

### Negrita

```css
p {
    font-weight: bold;   /* Negrita */
    font-weight: normal; /* Normal */
    font-weight: 700;    /* Número (700 = bold) */
}
```

### Cursiva

```css
p {
    font-style: italic;  /* Cursiva */
    font-style: normal;  /* Normal */
}
```

### Alineación de texto

```css
h1 {
    text-align: left;    /* Izquierda (por defecto) */
    text-align: center;  /* Centrado */
    text-align: right;   /* Derecha */
    text-align: justify; /* Justificado */
}
```

### Color de fondo

```css
body {
    background-color: #f0f0f0; /* Gris claro */
}

div {
    background-color: yellow;
}
```

---

## Cascada: El orden importa

CSS significa "Cascading" (en cascada). Si aplicas varios estilos al mismo elemento, **el último gana**.

```css
p {
    color: red;
}

p {
    color: blue;
}
```

Los párrafos serán **azules** (el segundo reemplaza al primero).

### Especificidad

Algunos selectores son más "fuertes" que otros. Verás esto más adelante cuando aprendas clases e IDs.

---

## Ejemplo completo: Blog simple

### `index.html`

```html
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <title>Mi Blog</title>
    <link rel="stylesheet" href="estilos.css">
</head>
<body>
    <header>
        <h1>Mi Blog Personal</h1>
        <p>Reflexiones sobre tecnología y vida</p>
    </header>
    
    <main>
        <article>
            <h2>Mi primer post</h2>
            <p>Hoy empiezo mi aventura con CSS. Es más fácil de lo que pensaba.</p>
        </article>
        
        <article>
            <h2>Aprendiendo sobre colores</h2>
            <p>Los colores hexadecimales son interesantes. #ff0000 es rojo puro.</p>
        </article>
    </main>
    
    <footer>
        <p>© 2026 Mi Blog</p>
    </footer>
</body>
</html>
```

### `estilos.css`

```css
/* Reset básico */
body {
    margin: 0;
    padding: 20px;
    font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
    background-color: #f5f5f5;
}

/* Header */
header {
    background-color: #2c3e50;
    color: white;
    padding: 40px 20px;
    text-align: center;
}

header h1 {
    margin: 0;
    font-size: 42px;
}

header p {
    margin: 10px 0 0 0;
    font-size: 18px;
    color: #ecf0f1;
}

/* Main content */
main {
    max-width: 800px;
    margin: 40px auto;
}

article {
    background-color: white;
    padding: 30px;
    margin-bottom: 30px;
}

article h2 {
    color: #2c3e50;
    margin-top: 0;
}

article p {
    color: #555;
    line-height: 1.8;
    font-size: 16px;
}

/* Footer */
footer {
    text-align: center;
    color: #777;
    padding: 20px;
    font-size: 14px;
}
```

**Guarda ambos archivos en la misma carpeta y abre `index.html`.**

Verás un blog con:
- Header oscuro con texto blanco
- Artículos con fondo blanco y texto legible
- Footer discreto al final

---

## Herramientas útiles

### DevTools del navegador

1. Abre tu página en Chrome o Firefox
2. Click derecho en cualquier elemento → "Inspeccionar"
3. Verás el HTML y el CSS aplicado

**Puedes editar CSS en vivo** para experimentar. Los cambios no se guardan, solo son para probar.

### Selector de color

Cuando uses DevTools, puedes hacer click en cualquier color y aparecerá un selector visual.

---

## Errores comunes

### Error 1: Olvidar el punto y coma

```css
/* Mal */
p {
    color: red
    font-size: 16px
}

/* Bien */
p {
    color: red;
    font-size: 16px;
}
```

Sin `;` CSS puede interpretar mal tus estilos.

### Error 2: Ruta incorrecta del archivo CSS

```html
<!-- Mal (si el archivo está en la misma carpeta) -->
<link rel="stylesheet" href="css/estilos.css">

<!-- Bien -->
<link rel="stylesheet" href="estilos.css">
```

Verifica que la ruta sea correcta según dónde esté tu archivo `.css`.

### Error 3: Escribir mal las propiedades

```css
/* Mal */
p {
    colour: red; /* En inglés es "color", no "colour" */
}

/* Bien */
p {
    color: red;
}
```

CSS ignora propiedades que no reconoce.

### Error 4: No cerrar las llaves

```css
/* Mal */
p {
    color: red;
/* Falta la llave de cierre }

h1 {
    color: blue;
}
```

Todo el CSS después del error puede fallar.

---

## Buenas prácticas

### 1. Un archivo CSS para toda la página

```
mi-proyecto/
├── index.html
├── contacto.html
└── estilos.css  ← Compartido por todas las páginas
```

### 2. Organiza tu CSS con comentarios

```css
/* ======================
   HEADER
   ====================== */

header {
    background-color: #2c3e50;
}

/* ======================
   CONTENIDO PRINCIPAL
   ====================== */

main {
    padding: 20px;
}

/* ======================
   FOOTER
   ====================== */

footer {
    text-align: center;
}
```

### 3. Usa nombres descriptivos

```css
/* Bien */
.boton-primario {
    background-color: blue;
}

/* Menos claro */
.azul {
    background-color: blue;
}
```

(Verás clases en el próximo módulo)

### 4. Mantén la indentación

```css
/* Bien */
p {
    color: red;
    font-size: 16px;
}

/* Mal (difícil de leer) */
p{color:red;font-size:16px;}
```

---

## Ejercicios prácticos

### Ejercicio 1: Primera página con CSS

Crea una página `biografia.html` y un archivo `estilos.css` con:
- Título principal (`<h1>`) azul y centrado
- Párrafos grises, tamaño 18px
- Fondo de página color crema (#f9f9e8)

<details>
<summary>✅ Solución</summary>

**biografia.html:**
```html
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <title>Mi biografía</title>
    <link rel="stylesheet" href="estilos.css">
</head>
<body>
    <h1>Sobre mí</h1>
    <p>Hola, soy un estudiante de desarrollo web.</p>
    <p>Me encanta aprender cosas nuevas cada día.</p>
</body>
</html>
```

**estilos.css:**
```css
body {
    background-color: #f9f9e8;
}

h1 {
    color: blue;
    text-align: center;
}

p {
    color: gray;
    font-size: 18px;
}
```

</details>

---

### Ejercicio 2: Página de receta estilizada

Toma la receta de tortilla que hiciste en HTML y añade CSS:
- Título de receta: color chocolate (#5e4434), 36px
- Subtítulos (Ingredientes, Preparación): color verde oscuro (#2d5016)
- Párrafos y listas: 16px, color gris oscuro (#333)
- Fondo: beige claro (#faf8f0)

<details>
<summary>✅ Solución</summary>

**receta.html:**
```html
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <title>Tortilla de patatas</title>
    <link rel="stylesheet" href="receta.css">
</head>
<body>
    <h1>Tortilla de patatas</h1>
    
    <h2>Ingredientes</h2>
    <ul>
        <li>4 huevos</li>
        <li>3 patatas</li>
        <li>Aceite de oliva</li>
        <li>Sal</li>
    </ul>
    
    <h2>Preparación</h2>
    <ol>
        <li>Pelar y cortar las patatas</li>
        <li>Freír en aceite abundante</li>
        <li>Batir los huevos</li>
        <li>Mezclar y cocinar</li>
    </ol>
</body>
</html>
```

**receta.css:**
```css
body {
    background-color: #faf8f0;
    padding: 20px;
}

h1 {
    color: #5e4434;
    font-size: 36px;
}

h2 {
    color: #2d5016;
}

p, li {
    color: #333;
    font-size: 16px;
}
```

</details>

---

### Ejercicio 3: Tres estilos diferentes

Crea `index.html` con un título y dos párrafos. Luego crea **tres archivos CSS** diferentes (`estilo1.css`, `estilo2.css`, `estilo3.css`) con esquemas de color distintos.

Cambia el `<link>` en el HTML para probar cada estilo.

<details>
<summary>💡 Pista</summary>

```html
<!-- Cambia esta línea para probar cada estilo -->
<link rel="stylesheet" href="estilo1.css">
```

</details>

<details>
<summary>✅ Solución ejemplo</summary>

**index.html:**
```html
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <title>Prueba de estilos</title>
    <link rel="stylesheet" href="estilo1.css">
</head>
<body>
    <h1>Mi página de prueba</h1>
    <p>Este es el primer párrafo.</p>
    <p>Este es el segundo párrafo.</p>
</body>
</html>
```

**estilo1.css (tema claro):**
```css
body {
    background-color: white;
    font-family: Arial, sans-serif;
}

h1 {
    color: #333;
}

p {
    color: #666;
}
```

**estilo2.css (tema oscuro):**
```css
body {
    background-color: #1e1e1e;
    color: #d4d4d4;
    font-family: 'Courier New', monospace;
}

h1 {
    color: #4ec9b0;
}

p {
    color: #ce9178;
}
```

**estilo3.css (tema colorido):**
```css
body {
    background-color: #ffe4e1;
    font-family: Georgia, serif;
}

h1 {
    color: #ff1493;
    text-align: center;
}

p {
    color: #8b008b;
    font-size: 18px;
}
```

</details>

---

## Recursos adicionales

- [MDN: CSS Basics](https://developer.mozilla.org/es/docs/Learn/Getting_started_with_the_web/CSS_basics)
- [CSS Reference](https://cssreference.io/) - Referencia visual de propiedades
- [ColorHunt](https://colorhunt.co/) - Paletas de colores listas para usar

---

## Siguiente paso

Ya sabes lo básico de CSS: conectarlo, sintaxis, propiedades comunes. Ahora vamos a aprender a **seleccionar elementos específicos**.

→ [07-selectores-css.md](07-selectores-css.md)

Ahí aprenderás clases, IDs, y cómo apuntar exactamente al elemento que quieres estilizar.

---

**Recuerda:** CSS se aprende practicando. Cambia colores, tamaños, prueba cosas. El navegador no se va a romper. Experimenta sin miedo.

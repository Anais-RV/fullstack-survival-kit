# Selectores CSS

Ya sabes aplicar estilos a **todos** los `<p>` o todos los `<h1>`. Ahora vas a aprender a ser más preciso: estilizar **solo** el párrafo de introducción, o **solo** el botón principal.

**Los selectores son la forma de decirle a CSS "aplica estos estilos AQUÍ y no en otro lado".**

---

## ¿Qué vas a aprender?

- Selectores de tipo (etiqueta)
- Clases: estilizar grupos específicos
- IDs: estilizar un elemento único
- Selectores combinados
- Pseudo-clases básicas (`:hover`)
- Cuándo usar cada selector

## Por qué es útil

Imagina que tienes 50 párrafos en tu página, pero solo quieres que 3 sean rojos. Sin selectores específicos, tendrías que aplicar estilos en línea a cada uno (horrible). Con clases, los marcas una vez y listo.

**Analogía:** Es como poner etiquetas en cajas. En vez de decir "todas las cajas", puedes decir "las cajas con etiqueta FRÁGIL" o "la caja número 42".

---

## Selector de tipo (elemento)

Ya lo has usado. Selecciona **todas** las etiquetas de ese tipo.

```css
p {
    color: blue;
}
```

Todos los `<p>` serán azules.

```css
h1 {
    font-size: 32px;
}
```

Todos los `<h1>` tendrán 32px.

**Cuándo usarlo:**
- Estilos base que quieres en TODOS los elementos de ese tipo
- Resets (establecer márgenes a 0, etc.)

---

## Clases: El selector más útil

Las clases son **etiquetas reutilizables** que pones en HTML.

### En HTML: atributo `class`

```html
<p class="destacado">Este párrafo es especial</p>
<p>Este es normal</p>
<p class="destacado">Este también es especial</p>
```

### En CSS: selector con punto `.`

```css
.destacado {
    color: red;
    font-weight: bold;
}
```

Solo los párrafos con `class="destacado"` serán rojos y en negrita.

### Múltiples clases

Un elemento puede tener varias clases separadas por espacio:

```html
<p class="destacado grande">Texto rojo, negrita Y grande</p>
```

```css
.destacado {
    color: red;
    font-weight: bold;
}

.grande {
    font-size: 24px;
}
```

El párrafo tendrá **ambos** estilos aplicados.

### Misma clase en diferentes elementos

```html
<p class="importante">Párrafo importante</p>
<h2 class="importante">Título importante</h2>
<span class="importante">Texto importante</span>
```

```css
.importante {
    background-color: yellow;
}
```

Todos tendrán fondo amarillo, sin importar que sean `<p>`, `<h2>` o `<span>`.

**Analogía:** Las clases son como uniformes. Cualquiera que lleve el uniforme (clase) "equipo-rojo" se verá igual, sin importar si es jugador, árbitro o entrenador.

---

## IDs: Selector único

Los IDs son para **un solo elemento** en toda la página.

### En HTML: atributo `id`

```html
<h1 id="titulo-principal">Bienvenido</h1>
<p id="intro">Este es el párrafo de introducción</p>
```

### En CSS: selector con almohadilla `#`

```css
#titulo-principal {
    color: navy;
    font-size: 48px;
}

#intro {
    font-style: italic;
}
```

**Regla de oro:** Un ID debe ser único. No uses el mismo ID dos veces en la misma página.

```html
<!-- Mal (dos elementos con el mismo ID) -->
<p id="especial">Primero</p>
<p id="especial">Segundo</p>  ❌

<!-- Bien (cada ID es único) -->
<p id="especial">Primero</p>
<p id="otro-especial">Segundo</p>  ✅
```

### Clases vs IDs: ¿Cuál usar?

| Característica | Clases | IDs |
|----------------|--------|-----|
| Símbolo CSS | `.clase` | `#id` |
| Reutilizable | Sí (muchos elementos) | No (solo uno) |
| Especificidad | Menor | Mayor |
| Uso recomendado | Estilos | Anclas, JavaScript |

**Recomendación:** Usa **clases** para CSS casi siempre. Reserva IDs para JavaScript o anclas de navegación.

---

## Selectores descendientes

Selecciona elementos **dentro** de otros.

```html
<article>
    <p>Párrafo dentro del article</p>
</article>

<p>Párrafo fuera del article</p>
```

```css
article p {
    color: green;
}
```

Solo el párrafo **dentro** de `<article>` será verde.

### Sintaxis

```css
ancestro descendiente {
    /* estilos */
}
```

Puede ser directo o a varios niveles:

```html
<div class="contenedor">
    <section>
        <p>Este párrafo también se selecciona</p>
    </section>
</div>
```

```css
.contenedor p {
    color: blue;
}
```

El `<p>` está dentro de `.contenedor` (aunque no directamente), así que se aplica el estilo.

### Ejemplo práctico: Blog

```html
<header>
    <p>Este párrafo está en el header</p>
</header>

<main>
    <p>Este párrafo está en el main</p>
</main>
```

```css
header p {
    color: white;
}

main p {
    color: black;
}
```

Cada párrafo tendrá el color según dónde esté.

---

## Selector de hijo directo `>`

Selecciona solo hijos **directos**, no nietos.

```html
<div class="padre">
    <p>Hijo directo</p>  ✅ Se selecciona
    <section>
        <p>Nieto</p>  ❌ No se selecciona
    </section>
</div>
```

```css
.padre > p {
    color: red;
}
```

Solo el primer `<p>` (hijo directo) será rojo.

### Diferencia entre espacio y `>`

```css
/* Descendiente (cualquier nivel) */
.contenedor p {
    color: blue;
}

/* Hijo directo (solo primer nivel) */
.contenedor > p {
    color: red;
}
```

```html
<div class="contenedor">
    <p>Rojo (hijo directo)</p>
    <article>
        <p>Azul (descendiente pero no hijo directo)</p>
    </article>
</div>
```

---

## Selector adyacente `+`

Selecciona el elemento **inmediatamente después** de otro.

```html
<h2>Título</h2>
<p>Este párrafo está justo después del h2</p>  ✅
<p>Este no</p>  ❌
```

```css
h2 + p {
    font-weight: bold;
}
```

Solo el primer párrafo después de `<h2>` será negrita.

**Uso común:** Estilar el primer párrafo después de un título.

---

## Selector de hermanos `~`

Selecciona **todos** los hermanos que vengan después.

```html
<h2>Título</h2>
<p>Primer párrafo</p>   ✅
<p>Segundo párrafo</p>  ✅
<p>Tercer párrafo</p>   ✅
```

```css
h2 ~ p {
    color: gray;
}
```

Todos los `<p>` que vengan después de `<h2>` (al mismo nivel) serán grises.

---

## Selectores de atributo

Selecciona elementos según sus atributos.

### Existe el atributo

```css
[type] {
    border: 1px solid gray;
}
```

Todos los elementos con atributo `type` (como `<input type="text">`).

### Atributo con valor específico

```css
[type="email"] {
    background-color: lightyellow;
}
```

Solo `<input type="email">`.

### Ejemplos prácticos

```html
<input type="text" placeholder="Nombre">
<input type="email" placeholder="Email">
<input type="password" placeholder="Contraseña">
```

```css
/* Todos los inputs con type */
input[type] {
    padding: 10px;
    border: 1px solid #ccc;
}

/* Solo el de email */
input[type="email"] {
    background-color: #e8f4f8;
}

/* Solo el de password */
input[type="password"] {
    background-color: #ffe8e8;
}
```

---

## Pseudo-clases: `:hover`, `:focus`, `:active`

Las pseudo-clases seleccionan estados especiales de los elementos.

### `:hover` - Cuando pasas el ratón encima

```css
button {
    background-color: blue;
    color: white;
}

button:hover {
    background-color: darkblue;
}
```

El botón será azul, pero **azul oscuro cuando pases el ratón**.

```css
a {
    color: blue;
}

a:hover {
    color: red;
    text-decoration: underline;
}
```

Los enlaces cambiarán de color al pasar el ratón.

### `:focus` - Cuando el elemento tiene el foco

Útil para inputs y botones.

```css
input:focus {
    border-color: blue;
    outline: 2px solid lightblue;
}
```

Cuando haces click en un input (tiene foco), se verá diferente.

### `:active` - Cuando estás haciendo click

```css
button:active {
    background-color: black;
}
```

El botón será negro mientras mantienes el click presionado.

### Ejemplo completo: Botón interactivo

```html
<button>Haz click aquí</button>
```

```css
button {
    background-color: #3498db;
    color: white;
    padding: 15px 30px;
    border: none;
    font-size: 16px;
    cursor: pointer;
    transition: all 0.3s;
}

button:hover {
    background-color: #2980b9;
    transform: scale(1.05);
}

button:active {
    background-color: #1c5983;
    transform: scale(0.95);
}
```

El botón cambia de color al pasar el ratón y se "hunde" al hacer click.

---

## Otras pseudo-clases útiles

### `:first-child` y `:last-child`

```html
<ul>
    <li>Primero</li>
    <li>Segundo</li>
    <li>Tercero</li>
</ul>
```

```css
li:first-child {
    font-weight: bold;
}

li:last-child {
    color: red;
}
```

El primer `<li>` será negrita, el último rojo.

### `:nth-child()`

```css
li:nth-child(2) {
    color: blue;
}
```

El segundo `<li>` será azul.

```css
li:nth-child(odd) {
    background-color: #f0f0f0;
}

li:nth-child(even) {
    background-color: white;
}
```

Filas alternas con fondo gris/blanco (como en una tabla).

---

## Agrupar selectores

Si varios selectores necesitan los mismos estilos:

```css
/* Sin agrupar (repetitivo) */
h1 {
    color: navy;
    font-family: Georgia, serif;
}

h2 {
    color: navy;
    font-family: Georgia, serif;
}

h3 {
    color: navy;
    font-family: Georgia, serif;
}
```

```css
/* Agrupados (mejor) */
h1, h2, h3 {
    color: navy;
    font-family: Georgia, serif;
}
```

Puedes agrupar cualquier tipo de selector:

```css
.destacado, #importante, p strong {
    background-color: yellow;
}
```

---

## Especificidad: ¿Qué estilo gana?

Si varios selectores apuntan al mismo elemento, gana el más **específico**.

### Orden de especificidad (de menor a mayor)

1. **Selector de tipo:** `p { }` (especificidad baja)
2. **Clase:** `.destacado { }` (especificidad media)
3. **ID:** `#especial { }` (especificidad alta)
4. **Estilos en línea:** `<p style="...">` (especificidad muy alta)

### Ejemplo

```html
<p class="texto" id="principal">Hola mundo</p>
```

```css
p {
    color: blue;
}

.texto {
    color: green;
}

#principal {
    color: red;
}
```

**¿De qué color será el párrafo?**  
**Rojo.** El ID tiene mayor especificidad que la clase, que a su vez tiene mayor que el tipo.

### Consejo práctico

**Usa clases la mayor parte del tiempo.** Si todo tu CSS usa IDs, será difícil de sobrescribir y mantener.

```css
/* Bien (flexible) */
.boton-primario {
    background-color: blue;
}

/* Menos flexible */
#boton1 {
    background-color: blue;
}
```

---

## Selector universal `*`

Selecciona **todos** los elementos.

```css
* {
    margin: 0;
    padding: 0;
    box-sizing: border-box;
}
```

**Uso común:** Reset básico al inicio del CSS.

**No abuses de él** para estilos específicos (muy poco específico).

---

## Ejemplo completo: Tarjetas de productos

```html
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <title>Tienda</title>
    <link rel="stylesheet" href="estilos.css">
</head>
<body>
    <div class="producto">
        <h3 class="titulo">Producto 1</h3>
        <p class="precio">$29.99</p>
        <button class="boton boton-comprar">Comprar</button>
    </div>
    
    <div class="producto destacado">
        <h3 class="titulo">Producto 2 - ¡Oferta!</h3>
        <p class="precio">$19.99</p>
        <button class="boton boton-comprar">Comprar</button>
    </div>
    
    <div class="producto">
        <h3 class="titulo">Producto 3</h3>
        <p class="precio">$39.99</p>
        <button class="boton boton-comprar">Comprar</button>
    </div>
</body>
</html>
```

```css
/* Reset básico */
* {
    margin: 0;
    padding: 0;
    box-sizing: border-box;
}

body {
    font-family: Arial, sans-serif;
    padding: 20px;
    background-color: #f5f5f5;
}

/* Tarjeta de producto */
.producto {
    background-color: white;
    padding: 20px;
    margin-bottom: 20px;
    border: 1px solid #ddd;
}

/* Producto destacado */
.producto.destacado {
    border: 3px solid #e74c3c;
    background-color: #fff3f3;
}

/* Título dentro de producto */
.producto .titulo {
    color: #2c3e50;
    margin-bottom: 10px;
}

/* Título de producto destacado */
.producto.destacado .titulo {
    color: #e74c3c;
}

/* Precio */
.precio {
    font-size: 24px;
    font-weight: bold;
    color: #27ae60;
    margin-bottom: 15px;
}

/* Botones */
.boton {
    padding: 10px 20px;
    border: none;
    cursor: pointer;
    font-size: 16px;
}

.boton-comprar {
    background-color: #3498db;
    color: white;
}

.boton-comprar:hover {
    background-color: #2980b9;
}

.boton-comprar:active {
    background-color: #1c5983;
}
```

**Nota los selectores usados:**
- `.producto` - Clase para todas las tarjetas
- `.producto.destacado` - Dos clases juntas (sin espacio) = elemento que tiene ambas
- `.producto .titulo` - Clase descendiente
- `.boton-comprar:hover` - Pseudo-clase
- `.producto.destacado .titulo` - Combinación de todo

---

## Nomenclatura de clases: BEM (opcional)

BEM (Block Element Modifier) es una convención para nombres de clases.

```html
<!-- Bloque -->
<div class="tarjeta">
    <!-- Elemento (doble guion bajo) -->
    <h3 class="tarjeta__titulo">Título</h3>
    <p class="tarjeta__descripcion">Descripción</p>
    <!-- Modificador (doble guion) -->
    <button class="tarjeta__boton tarjeta__boton--grande">Click</button>
</div>
```

**Ventajas:**
- Nombres claros y predecibles
- Evita conflictos de nombres
- Fácil de leer y mantener

**No es obligatorio**, pero muchos equipos lo usan.

---

## Errores comunes

### Error 1: Olvidar el punto en las clases

```css
/* Mal (selector de etiqueta, no clase) */
destacado {
    color: red;
}

/* Bien */
.destacado {
    color: red;
}
```

### Error 2: Usar almohadilla en HTML

```html
<!-- Mal -->
<p class="#importante">Texto</p>

<!-- Bien -->
<p class="importante">Texto</p>
```

En HTML: `class="nombre"`. En CSS: `.nombre`.  
En HTML: `id="nombre"`. En CSS: `#nombre`.

### Error 3: Espacios en nombres

```html
<!-- Mal (dos clases) -->
<p class="texto rojo">Hola</p>

<!-- Si quieres una clase llamada "texto-rojo" -->
<p class="texto-rojo">Hola</p>
```

Usa guion `-` o guion bajo `_` para unir palabras en una clase.

### Error 4: IDs duplicados

```html
<!-- Mal -->
<div id="contenido">Primero</div>
<div id="contenido">Segundo</div>  ❌

<!-- Bien -->
<div id="contenido1">Primero</div>
<div id="contenido2">Segundo</div>

<!-- Mejor (usa clases) -->
<div class="contenido">Primero</div>
<div class="contenido">Segundo</div>
```

---

## Buenas prácticas

### 1. Prefiere clases sobre IDs para CSS

```css
/* Bien */
.boton-primario { }

/* Menos flexible */
#boton1 { }
```

### 2. Nombres descriptivos

```css
/* Bien */
.encabezado-principal { }
.lista-productos { }

/* Menos claro */
.ep { }
.lp { }
```

### 3. Evita selectores demasiado específicos

```css
/* Mal (muy específico, difícil de sobrescribir) */
body div.contenedor section article.post p.texto { }

/* Bien */
.post p { }
/* o incluso */
.post-texto { }
```

### 4. Usa selectores de estado para interactividad

```css
button:hover { }
input:focus { }
a:visited { }
```

---

## Ejercicios prácticos

### Ejercicio 1: Sistema de botones

Crea una página con 3 botones usando clases:
- `.boton` - Clase base (padding, border-radius)
- `.boton-primario` - Azul
- `.boton-secundario` - Gris
- `.boton-peligro` - Rojo

Añade `:hover` a cada uno.

<details>
<summary>✅ Solución</summary>

```html
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <title>Botones</title>
    <link rel="stylesheet" href="estilos.css">
</head>
<body>
    <button class="boton boton-primario">Guardar</button>
    <button class="boton boton-secundario">Cancelar</button>
    <button class="boton boton-peligro">Eliminar</button>
</body>
</html>
```

```css
/* Clase base */
.boton {
    padding: 12px 24px;
    border: none;
    border-radius: 4px;
    font-size: 16px;
    cursor: pointer;
    transition: all 0.3s;
}

/* Variantes */
.boton-primario {
    background-color: #3498db;
    color: white;
}

.boton-primario:hover {
    background-color: #2980b9;
}

.boton-secundario {
    background-color: #95a5a6;
    color: white;
}

.boton-secundario:hover {
    background-color: #7f8c8d;
}

.boton-peligro {
    background-color: #e74c3c;
    color: white;
}

.boton-peligro:hover {
    background-color: #c0392b;
}
```

</details>

---

### Ejercicio 2: Menú de navegación

Crea un menú con una lista (`<ul>`):
- Quita los puntos de la lista
- Muestra los `<li>` en horizontal
- Los enlaces normales son grises
- El enlace con clase `.activo` es azul
- Al hacer hover sobre los enlaces, cambian a azul oscuro

<details>
<summary>✅ Solución</summary>

```html
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <title>Menú</title>
    <link rel="stylesheet" href="menu.css">
</head>
<body>
    <nav>
        <ul class="menu">
            <li><a href="#" class="activo">Inicio</a></li>
            <li><a href="#">Productos</a></li>
            <li><a href="#">Contacto</a></li>
        </ul>
    </nav>
</body>
</html>
```

```css
/* Reset de lista */
.menu {
    list-style: none;
    margin: 0;
    padding: 0;
    background-color: #2c3e50;
}

/* Items en horizontal */
.menu li {
    display: inline-block;
}

/* Enlaces base */
.menu a {
    display: block;
    padding: 15px 20px;
    color: #ecf0f1;
    text-decoration: none;
    transition: background-color 0.3s;
}

/* Enlace activo */
.menu a.activo {
    background-color: #3498db;
}

/* Hover en enlaces */
.menu a:hover {
    background-color: #34495e;
}
```

</details>

---

### Ejercicio 3: Tarjetas con estados

Crea 3 tarjetas de artículos:
- Tarjeta normal: fondo blanco, borde gris
- Tarjeta con clase `.destacada`: borde dorado, fondo amarillo claro
- Al hacer hover sobre cualquier tarjeta: sombra y elevar ligeramente

<details>
<summary>✅ Solución</summary>

```html
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <title>Tarjetas</title>
    <link rel="stylesheet" href="tarjetas.css">
</head>
<body>
    <div class="tarjeta">
        <h3>Artículo 1</h3>
        <p>Contenido normal</p>
    </div>
    
    <div class="tarjeta destacada">
        <h3>Artículo destacado</h3>
        <p>Este es especial</p>
    </div>
    
    <div class="tarjeta">
        <h3>Artículo 3</h3>
        <p>Contenido normal</p>
    </div>
</body>
</html>
```

```css
body {
    background-color: #f5f5f5;
    padding: 20px;
}

/* Tarjeta base */
.tarjeta {
    background-color: white;
    border: 1px solid #ddd;
    padding: 20px;
    margin-bottom: 20px;
    transition: all 0.3s;
}

/* Tarjeta destacada */
.tarjeta.destacada {
    background-color: #fffbea;
    border: 2px solid #f39c12;
}

/* Hover en cualquier tarjeta */
.tarjeta:hover {
    box-shadow: 0 4px 8px rgba(0,0,0,0.1);
    transform: translateY(-5px);
}
```

</details>

---

### Ejercicio 4: Tabla con filas alternas

Crea una tabla y usa `:nth-child(odd)` y `:nth-child(even)` para que las filas tengan colores alternos.

<details>
<summary>💡 Pista</summary>

```css
tbody tr:nth-child(odd) {
    background-color: #f9f9f9;
}

tbody tr:nth-child(even) {
    background-color: white;
}
```

</details>

---

## Cheat sheet de selectores

| Selector | Ejemplo | Descripción |
|----------|---------|-------------|
| `elemento` | `p` | Todas las etiquetas `<p>` |
| `.clase` | `.destacado` | Elementos con `class="destacado"` |
| `#id` | `#principal` | Elemento con `id="principal"` |
| `A B` | `div p` | `<p>` dentro de `<div>` |
| `A > B` | `div > p` | `<p>` hijo directo de `<div>` |
| `A + B` | `h2 + p` | `<p>` inmediatamente después de `<h2>` |
| `A ~ B` | `h2 ~ p` | Todos los `<p>` después de `<h2>` (hermanos) |
| `[atributo]` | `[type]` | Elementos con ese atributo |
| `[atributo="valor"]` | `[type="email"]` | Atributo con ese valor exacto |
| `:hover` | `a:hover` | Cuando pasas el ratón |
| `:focus` | `input:focus` | Cuando tiene el foco |
| `:first-child` | `li:first-child` | Primer hijo |
| `:last-child` | `li:last-child` | Último hijo |
| `:nth-child(n)` | `li:nth-child(3)` | Tercer hijo |

---

## Recursos adicionales

- [MDN: Selectores CSS](https://developer.mozilla.org/es/docs/Web/CSS/CSS_Selectors)
- [CSS Diner](https://flukeout.github.io/) - Juego para aprender selectores
- [Selector explicado](https://kittygiraudel.github.io/selectors-explained/) - Explica selectores en lenguaje natural

---

## Siguiente paso

Ya sabes seleccionar elementos de forma precisa. Ahora vas a hacer que se vean bien con **colores y tipografía**.

→ [08-colores-tipografia.md](08-colores-tipografia.md)

Ahí aprenderás códigos de color, fuentes de Google, y cómo hacer que tu texto sea legible y atractivo.

---

**Recuerda:** Las clases son tus mejores amigas. Úsalas sin miedo, nómbralas bien, y tu CSS será limpio y mantenible.

# Estructura semántica de una página

Ya sabes crear etiquetas básicas. Ahora vamos a organizar tu página con **significado**.

**HTML semántico = HTML que describe lo que contiene, no solo cómo se ve.**

---

## ¿Qué vas a aprender?

- La diferencia entre `<div>` y etiquetas semánticas
- Estructura típica de una página web
- `<header>`, `<nav>`, `<main>`, `<footer>`
- `<article>`, `<section>`, `<aside>`
- Cuándo usar cada una

## Por qué es útil

Imagina que entras a una casa y todas las habitaciones se llaman "habitación". No sabes cuál es la cocina, cuál el baño, cuál el dormitorio.

**Eso es usar solo `<div>`.**

Las etiquetas semánticas son como poner carteles: "Cocina", "Baño", "Dormitorio". Así Google, lectores de pantalla y otros desarrolladores entienden tu página.

---

## El problema con `<div>` para todo

### Código sin semántica (antes de HTML5)

```html
<div id="cabecera">
    <div id="logo">Mi sitio</div>
    <div id="menu">
        <div>Inicio</div>
        <div>Servicios</div>
        <div>Contacto</div>
    </div>
</div>

<div id="contenido-principal">
    <div class="articulo">
        <h2>Noticia 1</h2>
        <p>Contenido...</p>
    </div>
</div>

<div id="pie">
    <p>© 2026 Mi sitio</p>
</div>
```

**Problema:** Todo es `<div>`. Ni Google ni los lectores de pantalla saben qué es importante.

### Código con semántica (HTML5)

```html
<header>
    <h1>Mi sitio</h1>
    <nav>
        <a href="#">Inicio</a>
        <a href="#">Servicios</a>
        <a href="#">Contacto</a>
    </nav>
</header>

<main>
    <article>
        <h2>Noticia 1</h2>
        <p>Contenido...</p>
    </article>
</main>

<footer>
    <p>© 2026 Mi sitio</p>
</footer>
```

**Ventaja:** Cada etiqueta describe su función. Mucho más claro.

---

## Las etiquetas estructurales principales

### `<header>` - Cabecera

La parte superior de tu página. Normalmente contiene:
- Logo
- Título del sitio
- Menú de navegación

```html
<header>
    <h1>Blog de Cocina</h1>
    <p>Recetas fáciles para todos los días</p>
</header>
```

**Importante:** Puedes tener más de un `<header>`:
- Uno para toda la página
- Otro dentro de un `<article>` (para el título del artículo)

```html
<header>
    <h1>Mi Blog</h1>
</header>

<article>
    <header>
        <h2>Título del post</h2>
        <p>Publicado el 9 de enero, 2026</p>
    </header>
    <p>Contenido del post...</p>
</article>
```

---

### `<nav>` - Menú de navegación

Donde van los enlaces principales de tu sitio.

```html
<nav>
    <a href="index.html">Inicio</a>
    <a href="sobre-mi.html">Sobre mí</a>
    <a href="contacto.html">Contacto</a>
</nav>
```

**Normalmente va dentro de `<header>`:**

```html
<header>
    <h1>Mi Sitio</h1>
    <nav>
        <a href="#">Inicio</a>
        <a href="#">Blog</a>
        <a href="#">Contacto</a>
    </nav>
</header>
```

**También puede ir fuera:**

```html
<header>
    <h1>Mi Sitio</h1>
</header>

<nav>
    <a href="#">Inicio</a>
    <a href="#">Blog</a>
    <a href="#">Contacto</a>
</nav>
```

Ambas formas son válidas. Depende de tu diseño.

**Puedes tener varios `<nav>`:**
- Uno en el header (navegación principal)
- Otro en el footer (enlaces legales)
- Otro en un sidebar (categorías del blog)

```html
<header>
    <nav>
        <a href="#">Inicio</a>
        <a href="#">Blog</a>
    </nav>
</header>

<main>
    <nav>
        <h2>Categorías</h2>
        <a href="#">Recetas</a>
        <a href="#">Postres</a>
        <a href="#">Bebidas</a>
    </nav>
</main>

<footer>
    <nav>
        <a href="#">Privacidad</a>
        <a href="#">Términos</a>
    </nav>
</footer>
```

---

### `<main>` - Contenido principal

El contenido único de esa página. Lo que hace que cada página sea diferente.

```html
<main>
    <h2>Bienvenido a mi blog</h2>
    <p>Aquí encontrarás recetas deliciosas...</p>
</main>
```

**Regla de oro: Solo UN `<main>` por página.**

No incluyas aquí:
- El header (porque se repite en todas las páginas)
- El footer (ídem)
- La navegación (ídem)
- Sidebars repetidos

**Ejemplo completo:**

```html
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <title>Mi blog</title>
</head>
<body>
    <!-- Esto se repite en todas las páginas -->
    <header>
        <h1>Mi Blog</h1>
    </header>
    
    <!-- Esto es ÚNICO de esta página -->
    <main>
        <h2>Post sobre HTML semántico</h2>
        <p>Contenido específico de este post...</p>
    </main>
    
    <!-- Esto se repite en todas las páginas -->
    <footer>
        <p>© 2026 Mi Blog</p>
    </footer>
</body>
</html>
```

---

### `<footer>` - Pie de página

La parte inferior. Normalmente contiene:
- Copyright
- Enlaces a redes sociales
- Información de contacto
- Enlaces legales (privacidad, términos)

```html
<footer>
    <p>© 2026 Mi Sitio. Todos los derechos reservados.</p>
    <nav>
        <a href="privacidad.html">Política de privacidad</a>
        <a href="terminos.html">Términos de uso</a>
    </nav>
</footer>
```

**Como `<header>`, puedes tener varios `<footer>`:**

```html
<article>
    <header>
        <h2>Cómo hacer pan</h2>
    </header>
    <p>Contenido del artículo...</p>
    <footer>
        <p>Publicado por Ana López el 9 de enero, 2026</p>
        <p>Etiquetas: <a href="#">pan</a>, <a href="#">recetas</a></p>
    </footer>
</article>

<footer>
    <p>© 2026 Blog de Recetas</p>
</footer>
```

Uno dentro del `<article>` (metadatos del artículo) y otro para toda la página (copyright).

---

## Etiquetas de contenido

### `<article>` - Contenido independiente

Un bloque de contenido que **tiene sentido por sí solo**.

**Pregunta clave:** ¿Podría esto publicarse en otro sitio y seguir teniendo sentido?

Si la respuesta es sí → usa `<article>`.

**Ejemplos de `<article>`:**
- Un post de blog
- Una noticia
- Un comentario en un foro
- Un producto en una tienda
- Un tweet

```html
<article>
    <h2>Cómo hacer tortilla de patatas</h2>
    <p>La tortilla de patatas es...</p>
</article>

<article>
    <h2>Cómo hacer paella</h2>
    <p>La paella es...</p>
</article>
```

**Estructura típica de un `<article>`:**

```html
<article>
    <header>
        <h2>Título del artículo</h2>
        <p>Por Juan Pérez, 9 de enero, 2026</p>
    </header>
    
    <p>Contenido del artículo...</p>
    <p>Más contenido...</p>
    
    <footer>
        <p>Etiquetas: <a href="#">HTML</a>, <a href="#">CSS</a></p>
    </footer>
</article>
```

---

### `<section>` - Sección temática

Agrupa contenido relacionado bajo un **tema común**.

**Casi siempre debe tener un encabezado (`<h2>`, `<h3>`, etc.).**

```html
<section>
    <h2>Ingredientes</h2>
    <ul>
        <li>Harina</li>
        <li>Huevos</li>
    </ul>
</section>

<section>
    <h2>Preparación</h2>
    <ol>
        <li>Mezclar la harina...</li>
        <li>Añadir los huevos...</li>
    </ol>
</section>
```

**Diferencia entre `<article>` y `<section>`:**

- `<article>` = Contenido independiente (puede vivir solo)
- `<section>` = Parte de algo más grande (necesita contexto)

**Ejemplo:**

```html
<article>
    <h2>Guía completa de HTML</h2>
    
    <section>
        <h3>¿Qué es HTML?</h3>
        <p>HTML es...</p>
    </section>
    
    <section>
        <h3>Etiquetas básicas</h3>
        <p>Las etiquetas básicas son...</p>
    </section>
    
    <section>
        <h3>HTML semántico</h3>
        <p>El HTML semántico...</p>
    </section>
</article>
```

El `<article>` es toda la guía. Cada `<section>` es un capítulo de esa guía.

---

### `<aside>` - Contenido relacionado pero secundario

Contenido que está **relacionado** pero no es esencial para entender el contenido principal.

**Ejemplos:**
- Sidebar con enlaces relacionados
- Caja de "Sabías que..."
- Publicidad
- Biografía del autor
- Definición de un término

```html
<main>
    <article>
        <h2>Historia de la tortilla de patatas</h2>
        <p>La tortilla de patatas se originó...</p>
        
        <aside>
            <h3>Dato curioso</h3>
            <p>Existe un debate sobre si la tortilla debe llevar cebolla o no.</p>
        </aside>
        
        <p>Continuando con la historia...</p>
    </article>
    
    <aside>
        <h3>Artículos relacionados</h3>
        <ul>
            <li><a href="#">Historia de la paella</a></li>
            <li><a href="#">Origen del gazpacho</a></li>
        </ul>
    </aside>
</main>
```

**Regla práctica:** Si eliminas el `<aside>`, el contenido principal sigue teniendo sentido.

---

## Estructura completa de una página típica

Juntemos todo:

```html
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <title>Blog de Recetas</title>
</head>
<body>
    <!-- CABECERA -->
    <header>
        <h1>Blog de Recetas</h1>
        <p>Cocina fácil para todos los días</p>
        
        <nav>
            <a href="index.html">Inicio</a>
            <a href="recetas.html">Recetas</a>
            <a href="sobre-mi.html">Sobre mí</a>
            <a href="contacto.html">Contacto</a>
        </nav>
    </header>
    
    <!-- CONTENIDO PRINCIPAL -->
    <main>
        <article>
            <header>
                <h2>Tortilla de patatas perfecta</h2>
                <p>Publicado por Ana el 9 de enero, 2026</p>
            </header>
            
            <section>
                <h3>Ingredientes</h3>
                <ul>
                    <li>4 huevos</li>
                    <li>3 patatas</li>
                    <li>Aceite de oliva</li>
                    <li>Sal</li>
                </ul>
            </section>
            
            <section>
                <h3>Preparación</h3>
                <ol>
                    <li>Pelar y cortar las patatas</li>
                    <li>Freír las patatas</li>
                    <li>Batir los huevos</li>
                    <li>Mezclar y cocinar</li>
                </ol>
            </section>
            
            <aside>
                <h3>Consejo</h3>
                <p>Para darle la vuelta sin romperla, usa un plato grande.</p>
            </aside>
            
            <footer>
                <p>Tiempo: 45 min | Dificultad: Media</p>
                <p>Etiquetas: <a href="#">española</a>, <a href="#">huevos</a></p>
            </footer>
        </article>
        
        <aside>
            <h3>Recetas relacionadas</h3>
            <ul>
                <li><a href="#">Tortilla francesa</a></li>
                <li><a href="#">Patatas bravas</a></li>
                <li><a href="#">Huevos rotos</a></li>
            </ul>
        </aside>
    </main>
    
    <!-- PIE DE PÁGINA -->
    <footer>
        <p>© 2026 Blog de Recetas</p>
        <nav>
            <a href="privacidad.html">Privacidad</a>
            <a href="terminos.html">Términos</a>
            <a href="contacto.html">Contacto</a>
        </nav>
    </footer>
</body>
</html>
```

**Guarda esto como `receta-completa.html` y ábrelo.**

---

## Cuándo usar cada etiqueta

### Guía rápida de decisión

```
¿Es el contenido único de esta página?
    └─ SÍ → <main>

¿Es un bloque que podría publicarse solo en otro sitio?
    └─ SÍ → <article>
    └─ NO → ¿Agrupa contenido bajo un tema?
              └─ SÍ → <section>
              └─ NO → <div>

¿Es contenido tangencial/relacionado pero no esencial?
    └─ SÍ → <aside>

¿Es navegación?
    └─ SÍ → <nav>

¿Es la cabecera de algo?
    └─ SÍ → <header>

¿Es el pie de algo?
    └─ SÍ → <footer>
```

### Ejemplos prácticos

#### Blog personal

```html
<header>
    <h1>Mi Blog</h1>
    <nav>Enlaces</nav>
</header>

<main>
    <article>Post 1</article>
    <article>Post 2</article>
    <article>Post 3</article>
</main>

<aside>
    <h2>Sobre mí</h2>
    <p>Bio...</p>
</aside>

<footer>© 2026</footer>
```

#### Landing page (página de aterrizaje)

```html
<header>
    <h1>ProductoX</h1>
    <nav>Enlaces</nav>
</header>

<main>
    <section>
        <h2>¿Qué es ProductoX?</h2>
        <p>Descripción...</p>
    </section>
    
    <section>
        <h2>Características</h2>
        <ul>...</ul>
    </section>
    
    <section>
        <h2>Precios</h2>
        <div>...</div>
    </section>
</main>

<footer>© 2026</footer>
```

#### Tienda online (página de producto)

```html
<header>
    <h1>Mi Tienda</h1>
    <nav>Categorías</nav>
</header>

<main>
    <article>
        <header>
            <h2>Camiseta azul</h2>
            <p>Precio: 25€</p>
        </header>
        
        <section>
            <h3>Descripción</h3>
            <p>Camiseta de algodón...</p>
        </section>
        
        <section>
            <h3>Opiniones</h3>
            <article>
                <p>Muy buena - Juan</p>
            </article>
            <article>
                <p>Excelente - María</p>
            </article>
        </section>
    </article>
    
    <aside>
        <h2>Productos relacionados</h2>
        <ul>...</ul>
    </aside>
</main>

<footer>© 2026</footer>
```

---

## Errores comunes

### Error 1: Usar `<section>` sin encabezado

```html
<!-- Mal -->
<section>
    <p>Solo un párrafo sin título</p>
</section>

<!-- Mejor -->
<div>
    <p>Solo un párrafo sin título</p>
</div>

<!-- O si tiene título -->
<section>
    <h2>Título de la sección</h2>
    <p>Ahora sí tiene sentido</p>
</section>
```

Si no tiene encabezado, probablemente no es una `<section>`.

### Error 2: Múltiples `<main>` en una página

```html
<!-- Mal -->
<main>
    <h2>Sección 1</h2>
</main>
<main>
    <h2>Sección 2</h2>
</main>

<!-- Bien -->
<main>
    <section>
        <h2>Sección 1</h2>
    </section>
    <section>
        <h2>Sección 2</h2>
    </section>
</main>
```

Solo un `<main>` por página. Usa `<section>` para subdividir.

### Error 3: Confundir `<article>` con `<section>`

```html
<!-- Mal -->
<article>
    <h2>Ingredientes</h2>
    <ul>...</ul>
</article>
<article>
    <h2>Preparación</h2>
    <ol>...</ol>
</article>

<!-- Bien -->
<article>
    <h2>Receta: Tortilla</h2>
    
    <section>
        <h3>Ingredientes</h3>
        <ul>...</ul>
    </section>
    
    <section>
        <h3>Preparación</h3>
        <ol>...</ol>
    </section>
</article>
```

"Ingredientes" y "Preparación" no tienen sentido solos. Son partes de la receta completa.

### Error 4: Poner todo dentro de `<header>` o `<footer>`

```html
<!-- Mal -->
<header>
    <h1>Mi Sitio</h1>
    <nav>...</nav>
    <p>Bienvenido a mi sitio...</p>
    <article>Post 1</article>
    <article>Post 2</article>
</header>

<!-- Bien -->
<header>
    <h1>Mi Sitio</h1>
    <nav>...</nav>
</header>

<main>
    <p>Bienvenido a mi sitio...</p>
    <article>Post 1</article>
    <article>Post 2</article>
</main>
```

`<header>` solo para la cabecera. El contenido va en `<main>`.

### Error 5: Abusar de `<article>`

```html
<!-- Mal (cada párrafo es un article) -->
<article>
    <p>Primer párrafo</p>
</article>
<article>
    <p>Segundo párrafo</p>
</article>

<!-- Bien (todos los párrafos forman un artículo) -->
<article>
    <h2>Mi post</h2>
    <p>Primer párrafo</p>
    <p>Segundo párrafo</p>
</article>
```

Un párrafo solo no es un artículo independiente.

---

## Buenas prácticas

### 1. Usa la jerarquía correcta de encabezados

```html
<!-- Bien -->
<article>
    <h2>Título del artículo</h2>
    <section>
        <h3>Subtítulo de sección</h3>
        <h4>Sub-subtítulo</h4>
    </section>
</article>

<!-- Mal (saltos en la jerarquía) -->
<article>
    <h2>Título</h2>
    <h5>Subtítulo</h5> <!-- ¿Dónde están h3 y h4? -->
</article>
```

No saltes niveles: h2 → h3 → h4, no h2 → h5.

### 2. Cada `<nav>` debe tener un propósito claro

```html
<!-- Bien (propósitos diferentes) -->
<header>
    <nav aria-label="Navegación principal">
        <a href="#">Inicio</a>
        <a href="#">Servicios</a>
    </nav>
</header>

<footer>
    <nav aria-label="Navegación legal">
        <a href="#">Privacidad</a>
        <a href="#">Términos</a>
    </nav>
</footer>
```

El atributo `aria-label` ayuda a lectores de pantalla a distinguir múltiples `<nav>`.

### 3. Usa `<div>` cuando no haya alternativa semántica

```html
<!-- Bien (no hay etiqueta semántica para "contenedor de botones") -->
<div class="botones">
    <button>Aceptar</button>
    <button>Cancelar</button>
</div>

<!-- Mal (forzar semántica donde no la hay) -->
<section class="botones">
    <button>Aceptar</button>
    <button>Cancelar</button>
</section>
```

Si no encuentras la etiqueta semántica adecuada, usa `<div>`. No pasa nada.

### 4. Piensa en la accesibilidad

Las etiquetas semánticas ayudan a:
- **Lectores de pantalla:** Anuncian "navegación principal", "contenido principal", etc.
- **Teclado:** Los usuarios pueden saltar directamente a `<main>` o `<nav>`
- **SEO:** Google entiende mejor tu contenido

---

## Ejercicios prácticos

### Ejercicio 1: Blog personal

Crea `mi-blog.html` con:
- Header con tu nombre y navegación (Inicio, Blog, Contacto)
- Main con 2 artículos (posts de blog inventados)
- Cada artículo debe tener un header con título y fecha
- Footer con copyright

<details>
<summary>✅ Solución ejemplo</summary>

```html
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <title>Mi Blog Personal</title>
</head>
<body>
    <header>
        <h1>Blog de Laura</h1>
        <nav>
            <a href="#">Inicio</a>
            <a href="#">Blog</a>
            <a href="#">Contacto</a>
        </nav>
    </header>
    
    <main>
        <article>
            <header>
                <h2>Mi primer día aprendiendo HTML</h2>
                <p>Publicado el 9 de enero, 2026</p>
            </header>
            <p>Hoy he descubierto las etiquetas semánticas...</p>
        </article>
        
        <article>
            <header>
                <h2>Por qué la semántica importa</h2>
                <p>Publicado el 8 de enero, 2026</p>
            </header>
            <p>La semántica no es solo para Google...</p>
        </article>
    </main>
    
    <footer>
        <p>© 2026 Laura Martínez. Todos los derechos reservados.</p>
    </footer>
</body>
</html>
```

</details>

---

### Ejercicio 2: Landing page de producto

Crea `producto.html` con:
- Header con nombre del producto y nav
- Main con 3 secciones:
  - "¿Qué es?" (descripción)
  - "Características" (lista de features)
  - "Precio" (información de precio)
- Footer con enlaces a redes sociales

<details>
<summary>✅ Solución ejemplo</summary>

```html
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <title>TaskMaster - Organiza tu vida</title>
</head>
<body>
    <header>
        <h1>TaskMaster</h1>
        <p>Organiza tu vida en segundos</p>
        <nav>
            <a href="#">Inicio</a>
            <a href="#">Características</a>
            <a href="#">Precios</a>
            <a href="#">Descargar</a>
        </nav>
    </header>
    
    <main>
        <section>
            <h2>¿Qué es TaskMaster?</h2>
            <p>TaskMaster es la app de gestión de tareas más simple del mercado. Sin complicaciones, sin curvas de aprendizaje. Solo tú y tus tareas.</p>
        </section>
        
        <section>
            <h2>Características</h2>
            <ul>
                <li>Interfaz minimalista</li>
                <li>Sincronización en la nube</li>
                <li>Recordatorios inteligentes</li>
                <li>Disponible en iOS y Android</li>
            </ul>
        </section>
        
        <section>
            <h2>Precio</h2>
            <p>Gratis para siempre</p>
            <p>Premium: 4,99€/mes (funciones avanzadas)</p>
        </section>
    </main>
    
    <footer>
        <p>Síguenos en:</p>
        <nav>
            <a href="#">Twitter</a>
            <a href="#">Instagram</a>
            <a href="#">LinkedIn</a>
        </nav>
        <p>© 2026 TaskMaster Inc.</p>
    </footer>
</body>
</html>
```

</details>

---

### Ejercicio 3: Página de receta con aside

Crea `receta-completa.html` con:
- Header con título del sitio y nav
- Main con un article que contenga:
  - Header del artículo (título + autor)
  - 2 sections (Ingredientes y Preparación)
  - Aside con un consejo o dato curioso
  - Footer del artículo con tiempo y dificultad
- Sidebar (aside) fuera del article con recetas relacionadas
- Footer de página

<details>
<summary>✅ Solución ejemplo</summary>

```html
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <title>Brownie de chocolate</title>
</head>
<body>
    <header>
        <h1>Recetas Dulces</h1>
        <nav>
            <a href="#">Inicio</a>
            <a href="#">Recetas</a>
            <a href="#">Consejos</a>
        </nav>
    </header>
    
    <main>
        <article>
            <header>
                <h2>Brownie de chocolate perfecto</h2>
                <p>Por Chef María, 9 de enero, 2026</p>
            </header>
            
            <section>
                <h3>Ingredientes</h3>
                <ul>
                    <li>200g chocolate negro</li>
                    <li>150g mantequilla</li>
                    <li>3 huevos</li>
                    <li>200g azúcar</li>
                    <li>100g harina</li>
                </ul>
            </section>
            
            <section>
                <h3>Preparación</h3>
                <ol>
                    <li>Precalentar el horno a 180°C</li>
                    <li>Derretir el chocolate con la mantequilla</li>
                    <li>Batir los huevos con el azúcar</li>
                    <li>Mezclar todo y añadir la harina</li>
                    <li>Hornear 25 minutos</li>
                </ol>
            </section>
            
            <aside>
                <h3>Secreto del chef</h3>
                <p>No hornees más de 25 minutos. El centro debe quedar ligeramente húmedo para un brownie perfecto.</p>
            </aside>
            
            <footer>
                <p>Tiempo: 40 min | Dificultad: Fácil</p>
            </footer>
        </article>
        
        <aside>
            <h2>Recetas relacionadas</h2>
            <ul>
                <li><a href="#">Cookies de chocolate</a></li>
                <li><a href="#">Tarta de chocolate</a></li>
                <li><a href="#">Mousse de chocolate</a></li>
            </ul>
        </aside>
    </main>
    
    <footer>
        <p>© 2026 Recetas Dulces</p>
        <nav>
            <a href="#">Privacidad</a>
            <a href="#">Términos</a>
        </nav>
    </footer>
</body>
</html>
```

</details>

---

### Ejercicio 4: Convertir divs a semántico

Este código usa solo `<div>`. Conviértelo a etiquetas semánticas:

```html
<div class="cabecera">
    <h1>Mi Portafolio</h1>
    <div class="menu">
        <a href="#">Inicio</a>
        <a href="#">Proyectos</a>
        <a href="#">Contacto</a>
    </div>
</div>

<div class="contenido">
    <div class="proyecto">
        <h2>Proyecto 1</h2>
        <p>Descripción...</p>
    </div>
    <div class="proyecto">
        <h2>Proyecto 2</h2>
        <p>Descripción...</p>
    </div>
</div>

<div class="pie">
    <p>© 2026 Mi Portafolio</p>
</div>
```

<details>
<summary>✅ Solución</summary>

```html
<header>
    <h1>Mi Portafolio</h1>
    <nav>
        <a href="#">Inicio</a>
        <a href="#">Proyectos</a>
        <a href="#">Contacto</a>
    </nav>
</header>

<main>
    <article>
        <h2>Proyecto 1</h2>
        <p>Descripción...</p>
    </article>
    <article>
        <h2>Proyecto 2</h2>
        <p>Descripción...</p>
    </article>
</main>

<footer>
    <p>© 2026 Mi Portafolio</p>
</footer>
```

**Cambios:**
- `.cabecera` → `<header>`
- `.menu` → `<nav>`
- `.contenido` → `<main>`
- `.proyecto` → `<article>` (cada proyecto es independiente)
- `.pie` → `<footer>`

</details>

---

## Herramientas para validar semántica

### Extensiones de navegador

- **HTML5 Outliner** (Chrome/Firefox): Muestra la estructura semántica de cualquier página
- **WAVE** (Chrome/Firefox): Evalúa accesibilidad (incluye semántica)

### Validator W3C

[https://validator.w3.org/](https://validator.w3.org/)

Pega tu HTML o sube tu archivo. Te dirá si hay errores estructurales.

---

## Recursos adicionales

- [MDN: Semántica HTML](https://developer.mozilla.org/es/docs/Glossary/Semantics#sem%C3%A1ntica_en_html)
- [HTML5 Doctor: Flowchart para elegir elementos](http://html5doctor.com/downloads/h5d-sectioning-flowchart.pdf)
- [A11y Project: Guía de accesibilidad](https://www.a11yproject.com/)

---

## Siguiente paso

Ya tienes la estructura semántica de una página. Ahora vamos a aprender a recoger información del usuario.

→ [04-formularios-html.md](04-formularios-html.md)

Ahí aprenderás `<form>`, `<input>`, `<textarea>`, `<select>` y cómo validar datos.

---

**Recuerda:** Usa semántica siempre que puedas. No solo para Google, sino para que otros desarrolladores (y tu yo del futuro) entiendan tu código fácilmente.

**Regla de oro:** Si existe una etiqueta específica para lo que necesitas, úsala. `<div>` es el último recurso.

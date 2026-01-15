# Etiquetas básicas de HTML

Ya sabes crear la estructura de una página. Ahora vamos a llenarla de contenido real: listas, enlaces, imágenes, texto con formato.

**Estas son las etiquetas que usarás todos los días.**

---

## ¿Qué vas a aprender?

- Listas ordenadas y desordenadas
- Enlaces (el corazón de la web)
- Imágenes
- Texto con formato (negrita, cursiva, etc.)
- Divisiones y contenedores

## Por qué es útil

Una página web sin estos elementos es solo texto plano. Con listas organizas información, con enlaces conectas páginas, con imágenes das contexto visual. Son los bloques fundamentales de cualquier sitio web.

---

## Listas: Organizar información

Hay dos tipos principales de listas: ordenadas y desordenadas.

### Listas desordenadas (viñetas)

Cuando el orden no importa.

```html
<ul>
    <li>Manzanas</li>
    <li>Naranjas</li>
    <li>Plátanos</li>
</ul>
```

**Resultado:**
- Manzanas
- Naranjas
- Plátanos

**Anatomía:**
- `<ul>` = Unordered List (Lista Desordenada)
- `<li>` = List Item (Elemento de Lista)

Cada `<li>` es un punto de la lista.

### Listas ordenadas (numeradas)

Cuando el orden SÍ importa.

```html
<ol>
    <li>Romper los huevos</li>
    <li>Batir</li>
    <li>Cocinar</li>
</ol>
```

**Resultado:**
1. Romper los huevos
2. Batir
3. Cocinar

**Anatomía:**
- `<ol>` = Ordered List (Lista Ordenada)
- `<li>` = List Item (igual que antes)

El navegador numera automáticamente.

### Listas anidadas

Puedes meter listas dentro de listas.

```html
<ul>
    <li>Frutas
        <ul>
            <li>Manzanas</li>
            <li>Naranjas</li>
        </ul>
    </li>
    <li>Verduras
        <ul>
            <li>Lechuga</li>
            <li>Tomate</li>
        </ul>
    </li>
</ul>
```

**Resultado:**
- Frutas
  - Manzanas
  - Naranjas
- Verduras
  - Lechuga
  - Tomate

**Importante:** La lista anidada va **dentro** del `<li>`, no fuera.

---

## Enlaces: Conectar páginas

Los enlaces (links) son lo que hace a la web "web". Te permiten saltar de una página a otra.

### Sintaxis básica

```html
<a href="https://google.com">Ir a Google</a>
```

**Partes:**
- `<a>` = Anchor (ancla, enlace)
- `href` = Hypertext Reference (la URL de destino)
- `Ir a Google` = El texto visible que se puede hacer clic

**Resultado:** [Ir a Google](#) (texto azul subrayado, clickeable)

### Enlace a otra página de tu sitio

```html
<a href="contacto.html">Ir a Contacto</a>
```

Si `contacto.html` está en la misma carpeta que tu página actual, solo necesitas el nombre del archivo.

### Enlace a una página en otra carpeta

```html
<a href="paginas/about.html">Sobre mí</a>
```

Esto busca `about.html` dentro de la carpeta `paginas`.

### Enlace a la página anterior (subir nivel)

```html
<a href="../index.html">Volver al inicio</a>
```

`../` significa "sube un nivel en la estructura de carpetas".

### Abrir enlace en nueva pestaña

```html
<a href="https://wikipedia.org" target="_blank">Wikipedia</a>
```

`target="_blank"` abre el enlace en una nueva pestaña.

**Buena práctica:** Úsalo solo para enlaces externos. Los enlaces internos de tu sitio deberían abrirse en la misma pestaña.

### Enlace a una sección de la misma página

```html
<!-- El enlace -->
<a href="#contacto">Ir a contacto</a>

<!-- Más abajo en la página -->
<h2 id="contacto">Contacto</h2>
<p>Aquí está mi email...</p>
```

El `#` indica que es un ID dentro de la misma página. Útil para navegación en páginas largas.

### Enlace de email

```html
<a href="mailto:tucorreo@ejemplo.com">Envíame un email</a>
```

Al hacer clic, abre el programa de correo del usuario con tu dirección prellenada.

### Enlace de teléfono

```html
<a href="tel:+34123456789">Llamar</a>
```

En móviles, abre la aplicación de teléfono.

---

## Imágenes: Contenido visual

Las imágenes se insertan con la etiqueta `<img>`.

### Sintaxis básica

```html
<img src="foto.jpg" alt="Descripción de la foto">
```

**Atributos importantes:**
- `src` = Source (la ruta de la imagen)
- `alt` = Alternative text (texto alternativo si la imagen no carga)

**Nota:** `<img>` no tiene etiqueta de cierre. Es una etiqueta vacía.

### Imagen desde tu carpeta

```html
<img src="imagenes/perfil.jpg" alt="Foto de perfil">
```

Busca `perfil.jpg` en la carpeta `imagenes`.

### Imagen desde internet

```html
<img src="https://ejemplo.com/imagen.jpg" alt="Imagen externa">
```

Usa la URL completa.

### Tamaño de imagen

```html
<img src="foto.jpg" alt="Foto" width="300" height="200">
```

`width` y `height` en píxeles.

**Mejor práctica:** Controla el tamaño con CSS más adelante. HTML solo debe definir el contenido, no el estilo.

### ¿Por qué el atributo alt es importante?

1. **Accesibilidad:** Lectores de pantalla lo leen para personas con discapacidad visual
2. **SEO:** Google indexa el texto alt
3. **Fallback:** Si la imagen no carga, se muestra el texto

**Nunca omitas el alt.**

**Ejemplo de buen alt:**
```html
<img src="gato-durmiendo.jpg" alt="Gato naranja durmiendo en un sofá gris">
```

**Ejemplo de mal alt:**
```html
<img src="img001.jpg" alt="imagen">
```

Sé descriptivo.

---

## Texto con formato

### Negrita (bold)

Dos opciones:

```html
<strong>Texto importante</strong>
<b>Texto en negrita</b>
```

**Diferencia:**
- `<strong>` = Semánticamente importante (para lectores de pantalla)
- `<b>` = Solo visual (negrita sin significado especial)

**Usa `<strong>` por defecto.**

### Cursiva (italic)

```html
<em>Texto enfatizado</em>
<i>Texto en cursiva</i>
```

**Diferencia:**
- `<em>` = Énfasis (semántico)
- `<i>` = Solo visual

**Usa `<em>` por defecto.**

### Texto subrayado

```html
<u>Texto subrayado</u>
```

**Cuidado:** Parece un enlace. Úsalo con precaución.

### Texto tachado

```html
<s>Precio anterior: 100€</s>
```

Útil para mostrar precios antiguos, correcciones, etc.

### Texto pequeño

```html
<small>Texto de letra pequeña</small>
```

Para notas al pie, condiciones legales, etc.

### Salto de línea

```html
<p>Primera línea<br>Segunda línea</p>
```

`<br>` fuerza un salto de línea dentro del mismo párrafo.

**Buena práctica:** Úsalo con moderación. Normalmente es mejor usar párrafos separados.

### Línea horizontal

```html
<hr>
```

Crea una línea horizontal para separar secciones.

---

## Contenedores: Agrupar elementos

### Div: Contenedor genérico

```html
<div>
    <h2>Sección de productos</h2>
    <p>Aquí van los productos...</p>
</div>
```

`<div>` no significa nada por sí solo. Es solo un contenedor para agrupar otros elementos.

**Analogía:** Es como una caja donde metes cosas relacionadas.

Lo usarás MUCHO cuando aprendas CSS para aplicar estilos a grupos de elementos.

### Span: Contenedor en línea

```html
<p>Este texto tiene una <span>palabra</span> especial.</p>
```

`<span>` es como `<div>`, pero para fragmentos pequeños dentro de texto.

**Diferencia clave:**
- `<div>` es de bloque (ocupa toda la línea)
- `<span>` es en línea (solo ocupa su contenido)

---

## Ejemplo completo: Página de recetas

Combinemos todo lo aprendido:

```html
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <title>Receta: Tortilla de patatas</title>
</head>
<body>
    <h1>Tortilla de patatas</h1>
    
    <img src="imagenes/tortilla.jpg" alt="Tortilla de patatas recién hecha en un plato blanco" width="400">
    
    <p>La <strong>tortilla de patatas</strong> es uno de los platos más icónicos de la cocina española.</p>
    
    <h2>Ingredientes</h2>
    <ul>
        <li>4 huevos</li>
        <li>3 patatas medianas</li>
        <li>1 cebolla <em>(opcional)</em></li>
        <li>Aceite de oliva</li>
        <li>Sal al gusto</li>
    </ul>
    
    <h2>Preparación</h2>
    <ol>
        <li>Pelar y cortar las patatas en láminas finas</li>
        <li>Freír las patatas en abundante aceite a fuego medio hasta que estén blandas</li>
        <li>Batir los huevos en un bol grande</li>
        <li>Escurrir las patatas y mezclarlas con el huevo batido</li>
        <li>Cocinar en una sartén hasta que cuaje por ambos lados</li>
    </ol>
    
    <hr>
    
    <h2>Consejos</h2>
    <p><strong>Truco:</strong> Para darle la vuelta sin que se rompa, usa un plato grande.</p>
    
    <p><small>Tiempo de preparación: 45 minutos</small></p>
    
    <p><a href="recetas.html">← Volver a todas las recetas</a></p>
</body>
</html>
```

**Guarda esto como `receta-tortilla.html` y ábrelo en tu navegador.**

---

## Buenas prácticas

### 1. Usa semántica cuando puedas

```html
<!-- Mejor -->
<strong>Importante</strong>

<!-- Peor -->
<b>Importante</b>
```

`<strong>` tiene significado, `<b>` no.

### 2. Anida correctamente

```html
<!-- Correcto -->
<ul>
    <li>Elemento 1</li>
    <li>Elemento 2</li>
</ul>

<!-- Incorrecto -->
<ul>
<li>Elemento 1
<li>Elemento 2
</ul>
```

Cierra las etiquetas y usa indentación.

### 3. Siempre incluye alt en imágenes

```html
<!-- Bien -->
<img src="logo.png" alt="Logo de la empresa">

<!-- Mal -->
<img src="logo.png">
```

### 4. Usa rutas relativas para tu sitio

```html
<!-- Mejor (para archivos locales) -->
<a href="contacto.html">Contacto</a>

<!-- Evita esto -->
<a href="file:///C:/Users/yo/sitio/contacto.html">Contacto</a>
```

Las rutas relativas funcionan en cualquier servidor.

### 5. No uses <br> para espaciar

```html
<!-- Mal -->
<p>Texto</p>
<br><br><br>
<p>Más texto</p>

<!-- Bien -->
<p>Texto</p>
<p>Más texto</p>
```

El espaciado se controla con CSS, no con `<br>` múltiples.

---

## Errores comunes

### Error 1: Olvidar cerrar etiquetas de lista

```html
<!-- Mal -->
<ul>
    <li>Elemento 1
    <li>Elemento 2
</ul>

<!-- Bien -->
<ul>
    <li>Elemento 1</li>
    <li>Elemento 2</li>
</ul>
```

### Error 2: Poner <li> fuera de <ul> o <ol>

```html
<!-- Mal -->
<li>Elemento sin lista</li>

<!-- Bien -->
<ul>
    <li>Elemento dentro de lista</li>
</ul>
```

`<li>` siempre debe estar dentro de `<ul>` o `<ol>`.

### Error 3: Rutas de imagen incorrectas

```html
<!-- Si la imagen está en "imagenes/foto.jpg" -->

<!-- Mal -->
<img src="foto.jpg" alt="Foto">

<!-- Bien -->
<img src="imagenes/foto.jpg" alt="Foto">
```

Si la imagen no se ve, revisa la ruta.

### Error 4: Olvidar cerrar etiquetas <a>

```html
<!-- Mal -->
<a href="link.html">Texto

<!-- Bien -->
<a href="link.html">Texto</a>
```

Sin cierre, todo lo que siga será parte del enlace.

---

## Ejercicios prácticos

### Ejercicio 1: Lista de tareas

Crea `tareas.html` con:
- Un título `<h1>`: "Mis tareas de hoy"
- Una lista ordenada con 5 tareas pendientes
- Cada tarea debe tener alguna palabra en negrita o cursiva

<details>
<summary>✅ Solución ejemplo</summary>

```html
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <title>Mis tareas</title>
</head>
<body>
    <h1>Mis tareas de hoy</h1>
    <ol>
        <li>Estudiar <strong>HTML</strong> durante 2 horas</li>
        <li>Hacer ejercicio <em>por la tarde</em></li>
        <li>Llamar a <strong>mi abuela</strong></li>
        <li>Preparar la <em>cena</em></li>
        <li>Leer <strong>30 páginas</strong> del libro</li>
    </ol>
</body>
</html>
```

</details>

---

### Ejercicio 2: Galería de imágenes

Crea `galeria.html` con:
- Un título principal "Mi galería"
- 3 imágenes (pueden ser de URLs externas si no tienes imágenes locales)
- Cada imagen con su texto alt descriptivo
- Bajo cada imagen, un párrafo corto describiéndola

<details>
<summary>💡 Pista</summary>

Puedes usar imágenes de [Unsplash](https://unsplash.com/) o [Pexels](https://pexels.com/) para URLs externas.

</details>

<details>
<summary>✅ Solución ejemplo</summary>

```html
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <title>Mi galería</title>
</head>
<body>
    <h1>Mi galería</h1>
    
    <img src="https://images.unsplash.com/photo-1506905925346-21bda4d32df4" alt="Montañas al amanecer con cielo naranja" width="400">
    <p>Amanecer en las montañas. Esta foto la tomé durante mi viaje a los Pirineos.</p>
    
    <img src="https://images.unsplash.com/photo-1469474968028-56623f02e42e" alt="Bosque con niebla entre los árboles" width="400">
    <p>Bosque misterioso. La niebla crea una atmósfera mágica.</p>
    
    <img src="https://images.unsplash.com/photo-1518837695005-2083093ee35b" alt="Playa tropical con arena blanca y mar turquesa" width="400">
    <p>Paraíso tropical. Mi lugar favorito para desconectar.</p>
</body>
</html>
```

</details>

---

### Ejercicio 3: Página de portfolio personal

Crea `portfolio.html` con:
- Tu nombre como `<h1>`
- Un subtítulo con tu profesión
- Una lista desordenada de 3 habilidades
- Una lista ordenada de 3 proyectos (nombres inventados está bien)
- Enlaces a:
  - Tu email (mailto)
  - Tu GitHub o LinkedIn (inventado está bien, solo practica la sintaxis)

<details>
<summary>✅ Solución ejemplo</summary>

```html
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <title>Portfolio - Ana López</title>
</head>
<body>
    <h1>Ana López</h1>
    <h2>Desarrolladora Frontend</h2>
    
    <h3>Habilidades</h3>
    <ul>
        <li><strong>HTML</strong> y <strong>CSS</strong></li>
        <li>JavaScript básico</li>
        <li>Diseño responsive</li>
    </ul>
    
    <h3>Proyectos destacados</h3>
    <ol>
        <li>Página web para cafetería local</li>
        <li>Portfolio personal interactivo</li>
        <li>Landing page para startup</li>
    </ol>
    
    <hr>
    
    <h3>Contacto</h3>
    <p>
        <a href="mailto:ana@ejemplo.com">Envíame un email</a> |
        <a href="https://github.com/ana" target="_blank">GitHub</a> |
        <a href="https://linkedin.com/in/ana" target="_blank">LinkedIn</a>
    </p>
</body>
</html>
```

</details>

---

### Ejercicio 4: Detectar errores

Este HTML tiene 4 errores. Encuéntralos y corrígelos:

```html
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <title>Página con errores</title>
</head>
<body>
    <h1>Mi lista de compras</h1>
    
    <ol>
        <li>Pan
        <li>Leche</li>
        <li>Huevos
    </ol>
    
    <p>Comprar en <a href="supermercado.html">el supermercado</p>
    
    <img src="lista.jpg">
</body>
</html>
```

<details>
<summary>💡 Pista</summary>

Busca:
- Etiquetas sin cerrar
- Atributos faltantes
- Estructura incorrecta

</details>

<details>
<summary>✅ Soluciones</summary>

**Errores:**
1. `<li>Pan` no tiene cierre (falta `</li>`)
2. `<li>Huevos` no tiene cierre
3. Falta `</a>` para cerrar el enlace
4. La imagen no tiene atributo `alt`

**Correcto:**
```html
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <title>Página con errores</title>
</head>
<body>
    <h1>Mi lista de compras</h1>
    
    <ol>
        <li>Pan</li>
        <li>Leche</li>
        <li>Huevos</li>
    </ol>
    
    <p>Comprar en <a href="supermercado.html">el supermercado</a></p>
    
    <img src="lista.jpg" alt="Lista de compras escrita a mano">
</body>
</html>
```

</details>

---

## Recursos adicionales

### Imágenes gratuitas para practicar

- [Unsplash](https://unsplash.com/) - Fotos de alta calidad
- [Pexels](https://pexels.com/) - Stock photos gratis
- [Pixabay](https://pixabay.com/) - Imágenes y vectores

### Referencia de etiquetas

- [MDN: Elementos HTML](https://developer.mozilla.org/es/docs/Web/HTML/Element)
- [HTML Reference](https://htmlreference.io/)

---

## Siguiente paso

Ya sabes crear contenido con etiquetas básicas. Ahora vamos a organizar ese contenido de forma semántica.

→ [03-estructura-pagina.md](03-estructura-pagina.md)

Ahí aprenderás `<header>`, `<nav>`, `<main>`, `<footer>` y otras etiquetas estructurales que dan significado a tu HTML.

---

**Recuerda:** Estas etiquetas son tu vocabulario básico. Las usarás en absolutamente todo lo que hagas en HTML. Practica hasta que sean automáticas.

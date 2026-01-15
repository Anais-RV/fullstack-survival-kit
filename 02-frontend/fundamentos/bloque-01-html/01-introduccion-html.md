# Introducción a HTML

HTML no es un lenguaje de programación. Es un lenguaje de **marcado**. 

No tiene lógica, no hace cálculos, no toma decisiones. Solo **estructura contenido**.

**HTML es el esqueleto de cualquier página web.**

---

## ¿Qué vas a aprender?

- Qué es HTML y para qué sirve
- Estructura básica de un documento HTML
- Cómo crear tu primera página web
- Qué son las etiquetas y cómo funcionan

## Por qué es útil

HTML es el fundamento absoluto de la web. No puedes hacer frontend sin HTML. CSS lo embellece, JavaScript lo hace interactivo, pero **HTML es la base**. Sin él, no hay nada.

---

## ¿Qué es HTML?

**HTML** = HyperText Markup Language (Lenguaje de Marcado de Hipertexto)

Suena técnico, pero es simple:
- **HyperText:** Texto con enlaces (puedes saltar de una página a otra)
- **Markup:** Marcas que indican qué es cada cosa (esto es un título, esto es un párrafo, esto es una imagen)
- **Language:** Tiene sintaxis y reglas

**En español:** HTML es texto con etiquetas que le dicen al navegador qué es cada cosa.

### Analogía: El currículum

Imagina que escribes tu currículum en Word:

- **Nombre** → Lo pones en negrita y grande
- **Experiencia laboral** → Lo pones como lista con viñetas
- **Educación** → Otro apartado con título
- **Contacto** → Al final, con enlace al email

Tú **marcas** qué es cada sección para que se vea diferente. HTML hace lo mismo: marca qué es un título, qué es una lista, qué es un enlace.

---

## Tu primera página HTML

Crea un archivo llamado `index.html` y escribe esto:

```html
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <title>Mi primera página</title>
</head>
<body>
    <h1>Hola, mundo</h1>
    <p>Esta es mi primera página web.</p>
</body>
</html>
```

**Guarda el archivo y ábrelo con tu navegador.**

Verás "Hola, mundo" como título grande y "Esta es mi primera página web" como texto normal.

**¡Felicidades! Acabas de crear una página web.**

---

## Anatomía de una etiqueta HTML

HTML usa **etiquetas** para marcar el contenido.

```html
<p>Este es un párrafo</p>
```

**Partes:**
- `<p>` → Etiqueta de apertura (indica dónde empieza el párrafo)
- `Este es un párrafo` → Contenido (lo que ves en la página)
- `</p>` → Etiqueta de cierre (indica dónde termina el párrafo)

**Estructura:**
```
<etiqueta>contenido</etiqueta>
```

### Etiquetas que no se cierran

Algunas etiquetas no tienen contenido dentro, así que no necesitan cierre:

```html
<img src="foto.jpg">
<br>
<hr>
```

Se llaman **etiquetas vacías** o **self-closing tags**.

---

## Estructura básica de un documento HTML

Toda página HTML tiene esta estructura:

```html
<!DOCTYPE html>
<html lang="es">
<head>
    <!-- Información sobre la página (metadatos) -->
    <meta charset="UTF-8">
    <title>Título de la página</title>
</head>
<body>
    <!-- Contenido visible de la página -->
    <h1>Título principal</h1>
    <p>Contenido de la página.</p>
</body>
</html>
```

### Desglose línea por línea

#### `<!DOCTYPE html>`
Le dice al navegador: "Esto es HTML5" (la versión actual de HTML).

**No es una etiqueta HTML.** Es una declaración. Siempre va al inicio.

#### `<html lang="es">`
La etiqueta raíz. Todo el documento está dentro de `<html>`.

`lang="es"` indica que el contenido está en español. Esto ayuda a:
- Lectores de pantalla (accesibilidad)
- Traductores automáticos
- Motores de búsqueda

Si tu contenido es en inglés, usa `lang="en"`.

#### `<head>`
Contiene **metadatos** (datos sobre la página). No se ve en pantalla.

Dentro del `<head>` va:
- `<title>`: El título que aparece en la pestaña del navegador
- `<meta charset="UTF-8">`: Codificación de caracteres (para que las tildes y ñ funcionen)
- Enlaces a CSS (lo veremos más adelante)
- Información para redes sociales, buscadores, etc.

#### `<body>`
El contenido visible de la página. Todo lo que el usuario ve.

Aquí van títulos, párrafos, imágenes, listas, enlaces, etc.

---

## Etiquetas básicas de contenido

### Títulos (Headings)

HTML tiene 6 niveles de títulos:

```html
<h1>Título principal</h1>
<h2>Subtítulo</h2>
<h3>Sub-subtítulo</h3>
<h4>Nivel 4</h4>
<h5>Nivel 5</h5>
<h6>Nivel 6</h6>
```

**Reglas:**
- Solo un `<h1>` por página (el título principal)
- Usa niveles en orden: no saltes de `<h1>` a `<h3>`
- `<h1>` es el más grande, `<h6>` el más pequeño

**Analogía del libro:**
- `<h1>`: Título del libro
- `<h2>`: Capítulo
- `<h3>`: Sección dentro del capítulo
- `<h4>`: Subsección

### Párrafos

```html
<p>Este es un párrafo. Puede tener varias frases. El navegador añade espacio antes y después automáticamente.</p>

<p>Este es otro párrafo. Siempre usa etiquetas p para párrafos, no saltos de línea.</p>
```

**Importante:** El navegador ignora saltos de línea en el código HTML.

```html
<p>Este texto
tiene saltos
de línea
en el código</p>
```

Se verá como: "Este texto tiene saltos de línea en el código" (todo en una línea).

Para forzar un salto de línea dentro de un párrafo:

```html
<p>Primera línea<br>Segunda línea</p>
```

Pero es mejor usar párrafos separados.

---

## Ejemplo completo: Página personal

```html
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <title>Sobre mí</title>
</head>
<body>
    <h1>María García</h1>
    <h2>Desarrolladora Web</h2>
    
    <p>Hola, soy María. Me dedico al desarrollo web y me encanta crear cosas útiles con código.</p>
    
    <h2>Mi experiencia</h2>
    <p>Llevo 2 años aprendiendo programación. He trabajado en proyectos personales y estoy construyendo mi portfolio.</p>
    
    <h2>Contacto</h2>
    <p>Si quieres contactarme, escríbeme a maria@example.com</p>
</body>
</html>
```

**Copia este código, guárdalo como `sobre-mi.html` y ábrelo en tu navegador.**

---

## Comentarios en HTML

Los comentarios son notas para ti (o para otros desarrolladores). El navegador los ignora.

```html
<!-- Esto es un comentario -->

<h1>Título visible</h1>

<!-- 
Esto es un comentario
de varias líneas.
También se ignora.
-->
```

**Usa comentarios para:**
- Explicar secciones complejas
- Dejar notas temporales
- Deshabilitar código sin borrarlo

```html
<!-- <p>Este párrafo está comentado, no se verá</p> -->
```

---

## Indentación: Código legible

Aunque el navegador no le importa, **indenta tu código** para que sea legible.

**Mal:**
```html
<html><head><title>Título</title></head><body><h1>Hola</h1><p>Texto</p></body></html>
```

**Bien:**
```html
<html>
<head>
    <title>Título</title>
</head>
<body>
    <h1>Hola</h1>
    <p>Texto</p>
</body>
</html>
```

**Regla:** Cada nivel de anidamiento lleva una indentación (usualmente 4 espacios o 1 tab).

VSCode lo hace automáticamente si presionas `Shift+Alt+F` (Windows) o `Shift+Option+F` (Mac).

---

## Errores comunes de principiantes

### Error 1: Olvidar cerrar etiquetas

```html
<p>Texto sin cerrar
<p>Otro párrafo</p>
```

El navegador intentará arreglarlo, pero puede no verse como esperas.

**Siempre cierra las etiquetas:**
```html
<p>Texto cerrado</p>
<p>Otro párrafo</p>
```

### Error 2: Anidar incorrectamente

```html
<p>Texto en <strong>negrita</p></strong>
```

**Mal.** Las etiquetas deben cerrarse en orden inverso al que se abrieron.

**Bien:**
```html
<p>Texto en <strong>negrita</strong></p>
```

**Piensa en paréntesis:** `( [ ] )` ✅ / `( [ ) ]` ❌

### Error 3: Espacios en nombres de archivo

```html
mi pagina.html  ❌
```

**Usa guiones o guion bajo:**
```html
mi-pagina.html  ✅
mi_pagina.html  ✅
```

### Error 4: No usar UTF-8

Sin `<meta charset="UTF-8">`, las tildes y ñ pueden verse mal.

**Siempre inclúyelo** en el `<head>`.

---

## Herramientas útiles

### Ver el código fuente de cualquier página

En cualquier página web:
- Clic derecho → "Ver código fuente de la página"
- O presiona `Ctrl+U` (Windows/Linux) / `Cmd+Option+U` (Mac)

Verás el HTML de esa página. **Así es como se aprende:** mirando cómo otros lo hacen.

### Inspeccionar elementos

- Clic derecho en cualquier elemento → "Inspeccionar"
- O presiona `F12`

Se abre DevTools. Puedes ver el HTML, modificarlo temporalmente, experimentar.

**Todo lo que cambies aquí no se guarda.** Es solo para probar.

---

## Ejercicios prácticos

### Ejercicio 1: Tu primera página desde cero

Crea un archivo `practica.html` con:
- Un título principal (`<h1>`) con tu nombre
- Dos subtítulos (`<h2>`): "Sobre mí" y "Mis intereses"
- Bajo "Sobre mí": Un párrafo describiendo quién eres
- Bajo "Mis intereses": Otro párrafo con tus hobbies

<details>
<summary>💡 Pista</summary>

Usa la estructura básica:
```html
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <title>Tu nombre</title>
</head>
<body>
    <!-- Tu contenido aquí -->
</body>
</html>
```

</details>

<details>
<summary>✅ Solución ejemplo</summary>

```html
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <title>Juan Pérez</title>
</head>
<body>
    <h1>Juan Pérez</h1>
    
    <h2>Sobre mí</h2>
    <p>Soy estudiante de desarrollo web. Me apasiona la tecnología y crear cosas útiles con código.</p>
    
    <h2>Mis intereses</h2>
    <p>Me gusta leer ciencia ficción, hacer fotografía y tocar la guitarra en mi tiempo libre.</p>
</body>
</html>
```

</details>

---

### Ejercicio 2: Página de receta

Crea `receta.html` con:
- Título principal: nombre del plato
- Subtítulo "Ingredientes"
- Párrafo listando ingredientes (por ahora sin listas, solo texto)
- Subtítulo "Preparación"
- Párrafo explicando los pasos

<details>
<summary>✅ Solución ejemplo</summary>

```html
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <title>Receta: Tortilla de patatas</title>
</head>
<body>
    <h1>Tortilla de patatas</h1>
    
    <h2>Ingredientes</h2>
    <p>4 huevos, 3 patatas medianas, aceite de oliva, sal</p>
    
    <h2>Preparación</h2>
    <p>Pelar y cortar las patatas en láminas finas. Freírlas en aceite hasta que estén blandas. Batir los huevos y mezclar con las patatas. Cocinar en sartén hasta cuajar.</p>
</body>
</html>
```

</details>

---

### Ejercicio 3: Detectar errores

Este HTML tiene 3 errores. Encuéntralos:

```html
<!DOCTYPE html>
<html lang="es">
<head>
    <title>Mi página
</head>
<body>
    <h1>Bienvenido</h1>
    <p>Este es un párrafo con <strong>texto en negrita</p></strong>
    <h2>Sección</h2>
    <p>Más contenido aquí.
</body>
</html>
```

<details>
<summary>💡 Pista</summary>

Busca:
- Etiquetas sin cerrar
- Etiquetas cerradas en orden incorrecto
- Estructura incorrecta

</details>

<details>
<summary>✅ Soluciones</summary>

**Errores:**
1. `<title>` no está cerrado (falta `</title>`)
2. `<strong>` está cerrado fuera del `<p>` (mal anidamiento)
3. El último `<p>` no está cerrado

**Correcto:**
```html
<!DOCTYPE html>
<html lang="es">
<head>
    <title>Mi página</title>
</head>
<body>
    <h1>Bienvenido</h1>
    <p>Este es un párrafo con <strong>texto en negrita</strong></p>
    <h2>Sección</h2>
    <p>Más contenido aquí.</p>
</body>
</html>
```

</details>

---

## Validador HTML

El W3C (organización que define estándares web) tiene un validador gratuito:

[validator.w3.org](https://validator.w3.org/)

Puedes subir tu archivo HTML y te dirá si tiene errores.

**Útil para aprender**, pero no te obsesiones. A veces marca cosas que técnicamente no son errores.

---

## Recursos adicionales

### Para profundizar (opcional)

- [MDN Web Docs: HTML](https://developer.mozilla.org/es/docs/Web/HTML) - Referencia completa
- [HTML Living Standard](https://html.spec.whatwg.org/) - Especificación oficial (técnica)
- [Can I Use](https://caniuse.com/) - Compatibilidad de etiquetas en navegadores

---

## Siguiente paso

Ahora que sabes la estructura básica, es hora de aprender más etiquetas para construir contenido real.

→ [02-etiquetas-basicas.md](02-etiquetas-basicas.md)

Ahí aprenderás listas, enlaces, imágenes y más elementos fundamentales de HTML.

---

**Recuerda:** HTML no tiene lógica compleja. Es solo marcar contenido. Si puedes estructurar un documento en Word, puedes escribir HTML. La sintaxis es solo cuestión de práctica.

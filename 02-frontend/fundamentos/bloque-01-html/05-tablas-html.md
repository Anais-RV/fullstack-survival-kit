# Tablas HTML

Las tablas organizan datos en filas y columnas. Son perfectas para información que **realmente necesita estar en formato de tabla**.

**Importante: Las tablas NO son para maquetar páginas. Son para datos tabulares.**

---

## ¿Qué vas a aprender?

- Estructura básica: `<table>`, `<tr>`, `<td>`, `<th>`
- Secciones: `<thead>`, `<tbody>`, `<tfoot>`
- Combinar celdas: `colspan` y `rowspan`
- Cuándo usar tablas (y cuándo NO)
- Accesibilidad en tablas

## Por qué es útil

Las tablas son perfectas para:
- Horarios
- Precios comparativos
- Resultados deportivos
- Datos estadísticos
- Cualquier dato que naturalmente va en filas y columnas

**No las uses para diseñar tu página.** Eso se hace con CSS (que aprenderás pronto).

---

## Estructura básica

### La tabla más simple

```html
<table>
    <tr>
        <td>Celda 1</td>
        <td>Celda 2</td>
    </tr>
    <tr>
        <td>Celda 3</td>
        <td>Celda 4</td>
    </tr>
</table>
```

**Partes:**
- `<table>` = La tabla completa
- `<tr>` = Table Row (fila)
- `<td>` = Table Data (celda de datos)

**Resultado visual:**
```
┌─────────┬─────────┐
│ Celda 1 │ Celda 2 │
├─────────┼─────────┤
│ Celda 3 │ Celda 4 │
└─────────┴─────────┘
```

### Añadir encabezados

```html
<table>
    <tr>
        <th>Nombre</th>
        <th>Edad</th>
    </tr>
    <tr>
        <td>Ana</td>
        <td>28</td>
    </tr>
    <tr>
        <td>Carlos</td>
        <td>35</td>
    </tr>
</table>
```

**`<th>` = Table Header (encabezado).** Por defecto se muestra en **negrita** y **centrado**.

**Resultado:**
```
┌─────────┬──────┐
│ Nombre  │ Edad │  ← Encabezados (th)
├─────────┼──────┤
│ Ana     │ 28   │  ← Datos (td)
├─────────┼──────┤
│ Carlos  │ 35   │
└─────────┴──────┘
```

---

## Secciones de tabla

Para tablas más complejas, usa `<thead>`, `<tbody>` y `<tfoot>`.

```html
<table>
    <thead>
        <tr>
            <th>Producto</th>
            <th>Precio</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>Manzanas</td>
            <td>2€</td>
        </tr>
        <tr>
            <td>Naranjas</td>
            <td>3€</td>
        </tr>
    </tbody>
    <tfoot>
        <tr>
            <td>Total</td>
            <td>5€</td>
        </tr>
    </tfoot>
</table>
```

**Ventajas:**
- Más semántico (el navegador sabe qué es qué)
- Mejor para lectores de pantalla
- Útil para imprimir (el navegador puede repetir `<thead>` en cada página)
- Puedes aplicar estilos CSS diferentes a cada sección

**No es obligatorio, pero es buena práctica para tablas con datos.**

---

## Bordes y atributo `border`

Por defecto, las tablas no tienen bordes visibles.

```html
<table border="1">
    <tr>
        <th>Nombre</th>
        <th>Edad</th>
    </tr>
    <tr>
        <td>Ana</td>
        <td>28</td>
    </tr>
</table>
```

`border="1"` añade un borde básico.

**Nota:** Esto es HTML antiguo. Lo correcto es hacerlo con CSS, pero `border="1"` funciona para pruebas rápidas.

**Con CSS (más adelante):**
```html
<style>
    table {
        border: 1px solid black;
        border-collapse: collapse;
    }
    th, td {
        border: 1px solid black;
        padding: 8px;
    }
</style>
```

Por ahora, usa `border="1"` mientras practicas.

---

## Combinar celdas

### `colspan` - Combinar columnas

Hace que una celda ocupe **varias columnas**.

```html
<table border="1">
    <tr>
        <th colspan="2">Información Personal</th>
    </tr>
    <tr>
        <td>Nombre</td>
        <td>Ana</td>
    </tr>
    <tr>
        <td>Edad</td>
        <td>28</td>
    </tr>
</table>
```

**Resultado:**
```
┌─────────────────────────┐
│ Información Personal    │  ← Ocupa 2 columnas
├────────────┬────────────┤
│ Nombre     │ Ana        │
├────────────┼────────────┤
│ Edad       │ 28         │
└────────────┴────────────┘
```

### `rowspan` - Combinar filas

Hace que una celda ocupe **varias filas**.

```html
<table border="1">
    <tr>
        <td rowspan="2">Ana</td>
        <td>Email</td>
        <td>ana@email.com</td>
    </tr>
    <tr>
        <td>Teléfono</td>
        <td>600 000 000</td>
    </tr>
</table>
```

**Resultado:**
```
┌─────┬──────────┬─────────────────┐
│     │ Email    │ ana@email.com   │
│ Ana ├──────────┼─────────────────┤
│     │ Teléfono │ 600 000 000     │
└─────┴──────────┴─────────────────┘
```

La celda "Ana" ocupa 2 filas.

### Combinar ambos

```html
<table border="1">
    <tr>
        <th colspan="3">Horario de Clases</th>
    </tr>
    <tr>
        <th>Hora</th>
        <th>Lunes</th>
        <th>Martes</th>
    </tr>
    <tr>
        <td rowspan="2">9:00 - 11:00</td>
        <td>Matemáticas</td>
        <td>Historia</td>
    </tr>
    <tr>
        <td>Física</td>
        <td>Química</td>
    </tr>
</table>
```

---

## Tabla completa: Ejemplo de horario

```html
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <title>Horario</title>
</head>
<body>
    <h1>Mi horario de clases</h1>
    
    <table border="1">
        <thead>
            <tr>
                <th>Hora</th>
                <th>Lunes</th>
                <th>Martes</th>
                <th>Miércoles</th>
                <th>Jueves</th>
                <th>Viernes</th>
            </tr>
        </thead>
        <tbody>
            <tr>
                <td>9:00 - 10:00</td>
                <td>Matemáticas</td>
                <td>Lengua</td>
                <td>Matemáticas</td>
                <td>Inglés</td>
                <td>Historia</td>
            </tr>
            <tr>
                <td>10:00 - 11:00</td>
                <td>Física</td>
                <td>Química</td>
                <td>Física</td>
                <td>Lengua</td>
                <td>Educación Física</td>
            </tr>
            <tr>
                <td>11:00 - 11:30</td>
                <td colspan="5">Recreo</td>
            </tr>
            <tr>
                <td>11:30 - 12:30</td>
                <td>Historia</td>
                <td>Matemáticas</td>
                <td>Inglés</td>
                <td>Química</td>
                <td>Arte</td>
            </tr>
            <tr>
                <td>12:30 - 13:30</td>
                <td>Inglés</td>
                <td>Historia</td>
                <td>Lengua</td>
                <td>Matemáticas</td>
                <td>Música</td>
            </tr>
        </tbody>
    </table>
</body>
</html>
```

**Guarda como `horario.html` y ábrelo.**

---

## Tabla de comparación de precios

Otro ejemplo práctico:

```html
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <title>Planes de suscripción</title>
</head>
<body>
    <h1>Elige tu plan</h1>
    
    <table border="1">
        <thead>
            <tr>
                <th>Característica</th>
                <th>Gratis</th>
                <th>Básico</th>
                <th>Premium</th>
            </tr>
        </thead>
        <tbody>
            <tr>
                <td>Usuarios</td>
                <td>1</td>
                <td>5</td>
                <td>Ilimitado</td>
            </tr>
            <tr>
                <td>Almacenamiento</td>
                <td>1 GB</td>
                <td>10 GB</td>
                <td>100 GB</td>
            </tr>
            <tr>
                <td>Soporte</td>
                <td>Email</td>
                <td>Email + Chat</td>
                <td>24/7</td>
            </tr>
            <tr>
                <td>Precio mensual</td>
                <td>0€</td>
                <td>9€</td>
                <td>29€</td>
            </tr>
        </tbody>
        <tfoot>
            <tr>
                <td colspan="4">Todos los planes incluyen 30 días de prueba</td>
            </tr>
        </tfoot>
    </table>
</body>
</html>
```

---

## Caption: Título de la tabla

```html
<table border="1">
    <caption>Resultados del primer trimestre</caption>
    <tr>
        <th>Asignatura</th>
        <th>Nota</th>
    </tr>
    <tr>
        <td>Matemáticas</td>
        <td>8</td>
    </tr>
    <tr>
        <td>Lengua</td>
        <td>7</td>
    </tr>
</table>
```

`<caption>` va **justo después** de `<table>` y proporciona un título descriptivo.

**Beneficios:**
- Accesibilidad (lectores de pantalla lo anuncian)
- SEO (Google entiende mejor el contenido)
- Claridad (el usuario sabe de qué va la tabla)

---

## Accesibilidad en tablas

### 1. Usa `<th>` para encabezados

```html
<!-- Bien -->
<tr>
    <th>Nombre</th>
    <th>Edad</th>
</tr>

<!-- Mal -->
<tr>
    <td><strong>Nombre</strong></td>
    <td><strong>Edad</strong></td>
</tr>
```

`<th>` tiene significado semántico. `<td>` con negrita no.

### 2. Usa `scope` en encabezados

Para tablas complejas, indica si un `<th>` es de columna o fila.

```html
<table border="1">
    <tr>
        <th scope="col">Producto</th>
        <th scope="col">Precio</th>
    </tr>
    <tr>
        <th scope="row">Manzanas</th>
        <td>2€</td>
    </tr>
    <tr>
        <th scope="row">Naranjas</th>
        <td>3€</td>
    </tr>
</table>
```

- `scope="col"` = Encabezado de columna
- `scope="row"` = Encabezado de fila

Los lectores de pantalla usan esta información para leer "Manzanas, Precio: 2€" en vez de solo "Manzanas, 2€".

### 3. Añade `<caption>` siempre que puedas

```html
<table border="1">
    <caption>Lista de productos y precios</caption>
    <!-- ... -->
</table>
```

### 4. Usa `<thead>`, `<tbody>`, `<tfoot>`

Ayuda a los lectores de pantalla a navegar mejor.

```html
<table>
    <thead>
        <tr><th>Columna 1</th></tr>
    </thead>
    <tbody>
        <tr><td>Datos</td></tr>
    </tbody>
</table>
```

---

## Cuándo usar tablas

### ✅ Usa tablas para:

- **Datos tabulares:** Información que naturalmente va en filas y columnas
- **Horarios:** Días vs horas
- **Comparaciones:** Productos, planes, características
- **Resultados:** Deportes, elecciones, estadísticas
- **Datos científicos:** Mediciones, experimentos

### ❌ NO uses tablas para:

- **Diseño de página:** Header, sidebar, contenido (usa CSS para eso)
- **Formularios:** Alinear labels e inputs (usa CSS)
- **Navegación:** Menús (usa listas `<ul>` y CSS)
- **Galerías de imágenes:** (usa `<div>` o `<figure>` y CSS Grid/Flexbox)

**Regla de oro:** Si quitas todos los bordes y la tabla ya no tiene sentido, entonces NO debería ser una tabla.

**Ejemplo de mal uso (diseño de página con tablas):**

```html
<!-- MAL - No hagas esto -->
<table>
    <tr>
        <td colspan="2">Header</td>
    </tr>
    <tr>
        <td>Sidebar</td>
        <td>Contenido</td>
    </tr>
    <tr>
        <td colspan="2">Footer</td>
    </tr>
</table>
```

Esto se hacía en los años 90 cuando CSS no era tan potente. Hoy es una mala práctica.

**Forma correcta (con HTML semántico y CSS):**

```html
<header>Header</header>
<aside>Sidebar</aside>
<main>Contenido</main>
<footer>Footer</footer>
```

Y luego usas CSS Grid o Flexbox para el diseño.

---

## Errores comunes

### Error 1: Olvidar cerrar `<tr>`

```html
<!-- Mal -->
<table>
    <tr>
        <td>Celda 1</td>
    <tr>
        <td>Celda 2</td>
    </tr>
</table>

<!-- Bien -->
<table>
    <tr>
        <td>Celda 1</td>
    </tr>
    <tr>
        <td>Celda 2</td>
    </tr>
</table>
```

### Error 2: Número desigual de celdas por fila

```html
<!-- Mal -->
<table border="1">
    <tr>
        <td>1</td>
        <td>2</td>
        <td>3</td>
    </tr>
    <tr>
        <td>4</td>
        <td>5</td> <!-- Falta una celda -->
    </tr>
</table>
```

Cada fila debe tener el mismo número de celdas (a menos que uses `colspan` o `rowspan`).

### Error 3: Poner `<tr>` fuera de `<table>`

```html
<!-- Mal -->
<table>
    <tr>
        <td>Dentro</td>
    </tr>
</table>
<tr>
    <td>Fuera</td> <!-- Esto no funcionará -->
</tr>

<!-- Bien -->
<table>
    <tr>
        <td>Dentro</td>
    </tr>
    <tr>
        <td>También dentro</td>
    </tr>
</table>
```

### Error 4: Usar tablas para maquetar

```html
<!-- Mal (año 1998) -->
<table>
    <tr>
        <td>Logo</td>
        <td>Menú</td>
    </tr>
</table>

<!-- Bien (año 2026) -->
<header>
    <h1>Logo</h1>
    <nav>Menú</nav>
</header>
```

### Error 5: No usar `<th>` para encabezados

```html
<!-- Mal -->
<tr>
    <td><strong>Nombre</strong></td>
    <td><strong>Edad</strong></td>
</tr>

<!-- Bien -->
<tr>
    <th>Nombre</th>
    <th>Edad</th>
</tr>
```

---

## Buenas prácticas

### 1. Siempre añade `<caption>`

```html
<table border="1">
    <caption>Lista de estudiantes</caption>
    <!-- ... -->
</table>
```

### 2. Usa estructura semántica

```html
<table>
    <thead>
        <tr><th>Encabezado</th></tr>
    </thead>
    <tbody>
        <tr><td>Datos</td></tr>
    </tbody>
    <tfoot>
        <tr><td>Total</td></tr>
    </tfoot>
</table>
```

### 3. Usa `scope` en `<th>`

```html
<th scope="col">Columna</th>
<th scope="row">Fila</th>
```

### 4. Mantén las tablas simples

Si tu tabla necesita muchos `colspan` y `rowspan` anidados, tal vez no sea la mejor herramienta.

### 5. No uses tablas para diseño

Ya lo hemos dicho, pero vale la pena repetirlo: **tablas = datos, no diseño**.

---

## Ejercicios prácticos

### Ejercicio 1: Tabla de contactos

Crea `contactos.html` con una tabla que tenga:
- Encabezados: Nombre, Email, Teléfono
- Al menos 3 filas de datos inventados
- Un `<caption>` descriptivo

<details>
<summary>✅ Solución ejemplo</summary>

```html
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <title>Lista de contactos</title>
</head>
<body>
    <h1>Mis contactos</h1>
    
    <table border="1">
        <caption>Lista de contactos personales</caption>
        <thead>
            <tr>
                <th>Nombre</th>
                <th>Email</th>
                <th>Teléfono</th>
            </tr>
        </thead>
        <tbody>
            <tr>
                <td>Ana López</td>
                <td>ana@email.com</td>
                <td>600 111 222</td>
            </tr>
            <tr>
                <td>Carlos García</td>
                <td>carlos@email.com</td>
                <td>600 333 444</td>
            </tr>
            <tr>
                <td>María Pérez</td>
                <td>maria@email.com</td>
                <td>600 555 666</td>
            </tr>
        </tbody>
    </table>
</body>
</html>
```

</details>

---

### Ejercicio 2: Tabla de resultados deportivos

Crea `resultados.html` con:
- Tabla de clasificación de un campeonato
- Columnas: Posición, Equipo, Puntos, Partidos Jugados
- 5 equipos (datos inventados)
- Usa `<thead>` y `<tbody>`
- Añade un `<tfoot>` con el total de partidos jugados

<details>
<summary>✅ Solución ejemplo</summary>

```html
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <title>Clasificación</title>
</head>
<body>
    <h1>Clasificación de la liga</h1>
    
    <table border="1">
        <caption>Temporada 2025/2026</caption>
        <thead>
            <tr>
                <th>Pos.</th>
                <th>Equipo</th>
                <th>Puntos</th>
                <th>PJ</th>
            </tr>
        </thead>
        <tbody>
            <tr>
                <td>1</td>
                <td>FC Barcelona</td>
                <td>45</td>
                <td>20</td>
            </tr>
            <tr>
                <td>2</td>
                <td>Real Madrid</td>
                <td>42</td>
                <td>20</td>
            </tr>
            <tr>
                <td>3</td>
                <td>Atlético Madrid</td>
                <td>38</td>
                <td>20</td>
            </tr>
            <tr>
                <td>4</td>
                <td>Sevilla FC</td>
                <td>35</td>
                <td>20</td>
            </tr>
            <tr>
                <td>5</td>
                <td>Real Sociedad</td>
                <td>33</td>
                <td>20</td>
            </tr>
        </tbody>
        <tfoot>
            <tr>
                <td colspan="3">Total de partidos</td>
                <td>100</td>
            </tr>
        </tfoot>
    </table>
</body>
</html>
```

</details>

---

### Ejercicio 3: Tabla con `colspan` y `rowspan`

Crea `horario-personal.html` con una tabla de horario que incluya:
- Al menos un `colspan` (para el recreo u hora de comida)
- Al menos un `rowspan` (para una clase que dure 2 horas)
- Mínimo 3 días y 4 franjas horarias

<details>
<summary>💡 Pista</summary>

Planea tu tabla en papel primero. Dibuja las celdas combinadas antes de escribir el HTML.

</details>

<details>
<summary>✅ Solución ejemplo</summary>

```html
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <title>Mi horario</title>
</head>
<body>
    <h1>Horario semanal</h1>
    
    <table border="1">
        <caption>Primer cuatrimestre</caption>
        <thead>
            <tr>
                <th>Hora</th>
                <th>Lunes</th>
                <th>Martes</th>
                <th>Miércoles</th>
            </tr>
        </thead>
        <tbody>
            <tr>
                <td>9:00 - 10:30</td>
                <td rowspan="2">Programación<br>(Doble sesión)</td>
                <td>Matemáticas</td>
                <td>Bases de Datos</td>
            </tr>
            <tr>
                <td>10:30 - 12:00</td>
                <td>Inglés</td>
                <td>Sistemas</td>
            </tr>
            <tr>
                <td>12:00 - 12:30</td>
                <td colspan="3">Descanso</td>
            </tr>
            <tr>
                <td>12:30 - 14:00</td>
                <td>Redes</td>
                <td>Programación</td>
                <td>Proyecto</td>
            </tr>
        </tbody>
    </table>
</body>
</html>
```

</details>

---

### Ejercicio 4: Detectar errores

Esta tabla tiene 4 errores. Encuéntralos:

```html
<table border="1">
    <tr>
        <td>Nombre</td>
        <td>Edad</td>
    </tr>
    <tr>
        <td>Ana</td>
        <td>28</td>
        <td>España</td>
    </tr>
    <tr>
        <td>Carlos
        <td>35</td>
    </tr>
    <tbody>
        <tr>
            <td>Total</td>
            <td>2 personas</td>
        </tr>
    </tbody>
</table>
```

<details>
<summary>💡 Pistas</summary>

- ¿Los encabezados son `<th>` o `<td>`?
- ¿Todas las filas tienen el mismo número de celdas?
- ¿Todas las etiquetas están cerradas?
- ¿El orden de las secciones es correcto?

</details>

<details>
<summary>✅ Soluciones</summary>

**Errores:**
1. Los encabezados deberían ser `<th>`, no `<td>`
2. La segunda fila tiene 3 celdas (Ana, 28, España) pero la primera solo tiene 2
3. Falta `</td>` en "Carlos"
4. `<tbody>` debería estar **antes** de las filas de datos, no después

**Correcto:**
```html
<table border="1">
    <thead>
        <tr>
            <th>Nombre</th>
            <th>Edad</th>
            <th>País</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>Ana</td>
            <td>28</td>
            <td>España</td>
        </tr>
        <tr>
            <td>Carlos</td>
            <td>35</td>
            <td>México</td>
        </tr>
    </tbody>
    <tfoot>
        <tr>
            <td colspan="3">Total: 2 personas</td>
        </tr>
    </tfoot>
</table>
```

</details>

---

## Recursos adicionales

- [MDN: Tablas HTML](https://developer.mozilla.org/es/docs/Web/HTML/Element/table)
- [W3C: Tablas accesibles](https://www.w3.org/WAI/tutorials/tables/)
- [CSS Tricks: Table guide](https://css-tricks.com/complete-guide-table-element/)

---

## ¡Felicidades! 🎉

Has completado el **bloque 1: Fundamentos de HTML**.

Ahora sabes:
- ✅ Estructura básica de HTML
- ✅ Etiquetas de contenido (headings, párrafos, listas, enlaces, imágenes)
- ✅ HTML semántico (header, nav, main, article, section)
- ✅ Formularios completos (inputs, textarea, select)
- ✅ Tablas de datos

**Esto es tu vocabulario HTML.** Todo lo que hagas de aquí en adelante usará estas bases.

---

## Siguiente paso: CSS

Ya sabes crear contenido con HTML. Ahora vamos a hacerlo **bonito**.

→ [06-introduccion-css.md](../bloque-02-css/06-introduccion-css.md)

En CSS aprenderás:
- Colores, tipografías, tamaños
- Box model (el concepto más importante de CSS)
- Flexbox y Grid (layouts modernos)
- Responsive design (que tu web se vea bien en móviles)

**HTML es la estructura. CSS es el diseño. JavaScript (que viene después) es la interactividad.**

---

**Recuerda:** Las tablas son para **datos**, no para **diseño**. Si necesitas organizar visualmente tu página, CSS Grid y Flexbox son las herramientas correctas (las aprenderás pronto).

**Practica creando tablas reales:** Haz un horario de tu semana, una lista de gastos, una comparación de productos. Así se aprende de verdad.

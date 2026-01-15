# Módulo 23: Seleccionar Elementos del DOM

## Índice
1. [¿Por qué seleccionar elementos?](#por-qué-seleccionar-elementos)
2. [querySelector y querySelectorAll](#queryselector-y-queryselectorall)
3. [getElementById](#getelementbyid)
4. [getElementsByClassName](#getelementsbyclassname)
5. [getElementsByTagName](#getelementsbytagname)
6. [Comparación de métodos](#comparación-de-métodos)
7. [Ejercicios prácticos](#ejercicios-prácticos)
8. [Errores comunes](#errores-comunes)
9. [Buenas prácticas](#buenas-prácticas)
10. [Cheatsheet](#cheatsheet)

---

## ¿Por qué seleccionar elementos?

Para manipular elementos HTML con JavaScript, primero debes **seleccionarlos** (encontrarlos en el DOM).

### Analogía: Buscar en una biblioteca

```
Biblioteca (DOM)
├── Estantería A (sección)
│   ├── Libro 1 (elemento)
│   ├── Libro 2 (elemento)
│   └── Libro 3 (elemento)
└── Estantería B (sección)
    ├── Libro 4 (elemento)
    └── Libro 5 (elemento)
```

Para leer un libro, primero debes **encontrarlo**:
- Por su ID único (código de barras)
- Por su clase/categoría (novela, técnico)
- Por su etiqueta (todos los libros de una estantería)

---

## querySelector y querySelectorAll

**Los métodos más modernos y versátiles**. Usan selectores CSS.

### querySelector() - Selecciona el primero

Retorna **el primer elemento** que coincida con el selector.

```html
<div id="contenedor">
    <p class="destacado">Primer párrafo</p>
    <p class="destacado">Segundo párrafo</p>
    <p>Tercer párrafo</p>
</div>

<script>
// Seleccionar el primer .destacado
const primero = document.querySelector('.destacado');
console.log(primero.textContent); // "Primer párrafo"

// Seleccionar el #contenedor
const contenedor = document.querySelector('#contenedor');
console.log(contenedor);

// Seleccionar el primer <p>
const primerP = document.querySelector('p');
console.log(primerP.textContent); // "Primer párrafo"
</script>
```

### querySelectorAll() - Selecciona todos

Retorna **una NodeList** con todos los elementos que coincidan.

```html
<ul>
    <li>Item 1</li>
    <li>Item 2</li>
    <li>Item 3</li>
</ul>

<script>
// Seleccionar todos los <li>
const items = document.querySelectorAll('li');
console.log(items.length); // 3

// Iterar sobre ellos
items.forEach(item => {
    console.log(item.textContent);
});
// Output:
// "Item 1"
// "Item 2"
// "Item 3"
</script>
```

### Selectores CSS avanzados

```html
<div class="container">
    <p id="intro">Introducción</p>
    <p class="text highlight">Texto destacado</p>
    <p class="text">Texto normal</p>
    <div class="box">
        <span>Dentro del box</span>
    </div>
</div>

<script>
// Por ID
document.querySelector('#intro');

// Por clase
document.querySelector('.text');

// Por múltiples clases
document.querySelector('.text.highlight');

// Por atributo
document.querySelector('[id="intro"]');

// Descendiente (espacio = cualquier descendiente)
document.querySelector('.container p'); // <p> dentro de .container

// Hijo directo (>)
document.querySelector('.container > p'); // <p> hijo directo de .container

// Hermano adyacente (+)
document.querySelector('#intro + p'); // <p> inmediatamente después de #intro

// Múltiples selectores (,)
document.querySelectorAll('p, span'); // Todos los <p> y <span>

// Pseudo-clases
document.querySelector('li:first-child'); // Primer <li>
document.querySelector('li:last-child');  // Último <li>
document.querySelector('li:nth-child(2)'); // Segundo <li>
</script>
```

### Retorno de querySelector

```javascript
// Si encuentra el elemento: retorna el elemento
const elemento = document.querySelector('.existe');
console.log(elemento); // <div class="existe">...</div>

// Si NO encuentra: retorna null
const noExiste = document.querySelector('.no-existe');
console.log(noExiste); // null
```

---

## getElementById

**Selecciona un elemento por su ID**. Los IDs deben ser únicos en la página.

```html
<div id="header">Encabezado</div>
<div id="content">Contenido</div>
<div id="footer">Pie de página</div>

<script>
// Seleccionar por ID (sin #)
const header = document.getElementById('header');
console.log(header.textContent); // "Encabezado"

const content = document.getElementById('content');
console.log(content); // <div id="content">...</div>

// Si no existe, retorna null
const noExiste = document.getElementById('inexistente');
console.log(noExiste); // null
</script>
```

### No uses el símbolo #

```javascript
// ❌ Error: no uses #
const elemento = document.getElementById('#header'); // null (no encontrado)

// ✅ Correcto: sin #
const elemento = document.getElementById('header'); // <div id="header">
```

---

## getElementsByClassName

**Selecciona todos los elementos con una clase específica**. Retorna una **HTMLCollection** (colección live).

```html
<p class="destacado">Párrafo 1</p>
<p class="destacado">Párrafo 2</p>
<p class="normal">Párrafo 3</p>
<div class="destacado">Div destacado</div>

<script>
// Seleccionar todos los .destacado (sin punto)
const destacados = document.getElementsByClassName('destacado');
console.log(destacados.length); // 3

// Acceder por índice
console.log(destacados[0].textContent); // "Párrafo 1"
console.log(destacados[1].textContent); // "Párrafo 2"
console.log(destacados[2].textContent); // "Div destacado"

// Convertir a array para usar forEach
Array.from(destacados).forEach(elemento => {
    console.log(elemento.textContent);
});
</script>
```

### HTMLCollection vs NodeList

```javascript
// getElementsByClassName → HTMLCollection (live)
const coleccion = document.getElementsByClassName('item');
console.log(coleccion.length); // 3

// querySelectorAll → NodeList (estática)
const lista = document.querySelectorAll('.item');
console.log(lista.length); // 3

// HTMLCollection es "live" (se actualiza automáticamente)
// NodeList es estática (no se actualiza)
```

---

## getElementsByTagName

**Selecciona todos los elementos por su etiqueta HTML**. Retorna HTMLCollection.

```html
<h1>Título principal</h1>
<p>Párrafo 1</p>
<p>Párrafo 2</p>
<div>Div 1</div>
<p>Párrafo 3</p>

<script>
// Seleccionar todos los <p>
const parrafos = document.getElementsByTagName('p');
console.log(parrafos.length); // 3

// Seleccionar todos los <div>
const divs = document.getElementsByTagName('div');
console.log(divs.length); // 1

// Seleccionar TODOS los elementos
const todos = document.getElementsByTagName('*');
console.log(todos.length); // Todos los elementos de la página

// Iterar (convertir a array)
Array.from(parrafos).forEach((p, index) => {
    console.log(`Párrafo ${index + 1}: ${p.textContent}`);
});
</script>
```

---

## Comparación de métodos

### Tabla comparativa

| Método | Selector | Retorna | Live? | Ejemplo |
|--------|----------|---------|-------|---------|
| `querySelector` | CSS | Primer elemento o null | No | `querySelector('.clase')` |
| `querySelectorAll` | CSS | NodeList | No | `querySelectorAll('.clase')` |
| `getElementById` | ID sin # | Elemento o null | No | `getElementById('miId')` |
| `getElementsByClassName` | Clase sin . | HTMLCollection | Sí | `getElementsByClassName('clase')` |
| `getElementsByTagName` | Etiqueta | HTMLCollection | Sí | `getElementsByTagName('p')` |

### ¿Cuál usar?

```javascript
// ✅ querySelector/querySelectorAll (recomendado para la mayoría de casos)
// - Sintaxis CSS familiar
// - Muy flexible
// - Funciona en todos los navegadores modernos
const elemento = document.querySelector('#miId');
const elementos = document.querySelectorAll('.miClase');

// ✅ getElementById (cuando solo necesitas un ID)
// - Ligeramente más rápido
// - Más directo
const elemento = document.getElementById('miId');

// ⚠️ getElementsByClassName/getElementsByTagName
// - Colecciones live (pueden ser útiles o problemáticas)
// - Menos flexible que querySelector
const elementos = document.getElementsByClassName('miClase');
```

### Selectores equivalentes

```javascript
// Por ID
document.querySelector('#header');
document.getElementById('header');

// Por clase
document.querySelectorAll('.item');
document.getElementsByClassName('item');

// Por etiqueta
document.querySelectorAll('p');
document.getElementsByTagName('p');

// querySelector es más flexible:
document.querySelector('.container > p.destacado');
// No hay equivalente directo con los otros métodos
```

---

## Ejercicios prácticos

### Ejercicio 1: Seleccionar por ID
**Nivel**: ⭐☆☆☆☆

Selecciona un elemento por ID y muestra su contenido en la consola.

<details>
<summary>Ver solución</summary>

```html
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <title>Selección por ID</title>
</head>
<body>
    <h1 id="titulo">Bienvenido a mi página</h1>
    <p id="descripcion">Esta es una descripción de ejemplo</p>
    <div id="contenedor">Contenedor principal</div>

    <script>
        // Método 1: getElementById
        const titulo = document.getElementById('titulo');
        console.log("Título:", titulo.textContent);

        // Método 2: querySelector
        const descripcion = document.querySelector('#descripcion');
        console.log("Descripción:", descripcion.textContent);

        const contenedor = document.getElementById('contenedor');
        console.log("Contenedor:", contenedor.textContent);
    </script>
</body>
</html>
```

</details>

---

### Ejercicio 2: Seleccionar por clase
**Nivel**: ⭐⭐☆☆☆

Selecciona todos los elementos con una clase y cuenta cuántos hay.

<details>
<summary>Ver solución</summary>

```html
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <title>Selección por Clase</title>
</head>
<body>
    <p class="destacado">Párrafo destacado 1</p>
    <p>Párrafo normal</p>
    <p class="destacado">Párrafo destacado 2</p>
    <div class="destacado">Div destacado</div>
    <p class="destacado">Párrafo destacado 3</p>

    <script>
        // Método 1: querySelectorAll (recomendado)
        const destacados1 = document.querySelectorAll('.destacado');
        console.log(`Total destacados (querySelectorAll): ${destacados1.length}`);

        // Método 2: getElementsByClassName
        const destacados2 = document.getElementsByClassName('destacado');
        console.log(`Total destacados (getElementsByClassName): ${destacados2.length}`);

        // Mostrar cada uno
        destacados1.forEach((elemento, index) => {
            console.log(`${index + 1}. ${elemento.textContent}`);
        });
    </script>
</body>
</html>
```

</details>

---

### Ejercicio 3: Seleccionar por etiqueta
**Nivel**: ⭐⭐☆☆☆

Selecciona todos los párrafos y cambia su texto.

<details>
<summary>Ver solución</summary>

```html
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <title>Selección por Etiqueta</title>
</head>
<body>
    <h1>Lista de tareas</h1>
    <p>Primera tarea</p>
    <p>Segunda tarea</p>
    <p>Tercera tarea</p>

    <script>
        // Seleccionar todos los párrafos
        const parrafos = document.querySelectorAll('p');
        
        console.log(`Total de párrafos: ${parrafos.length}`);

        // Numerar cada párrafo
        parrafos.forEach((parrafo, index) => {
            const textoOriginal = parrafo.textContent;
            parrafo.textContent = `${index + 1}. ${textoOriginal}`;
        });

        // Resultado:
        // 1. Primera tarea
        // 2. Segunda tarea
        // 3. Tercera tarea
    </script>
</body>
</html>
```

</details>

---

### Ejercicio 4: Selectores CSS complejos
**Nivel**: ⭐⭐⭐☆☆

Usa selectores CSS avanzados para seleccionar elementos específicos.

<details>
<summary>Ver solución</summary>

```html
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <title>Selectores Complejos</title>
</head>
<body>
    <div class="container">
        <h2>Artículos</h2>
        <article class="post destacado">
            <h3>Artículo 1</h3>
            <p>Contenido del artículo 1</p>
        </article>
        <article class="post">
            <h3>Artículo 2</h3>
            <p>Contenido del artículo 2</p>
        </article>
        <article class="post">
            <h3>Artículo 3</h3>
            <p>Contenido del artículo 3</p>
        </article>
    </div>

    <script>
        // 1. Párrafos dentro de articles
        const parrafosEnArticles = document.querySelectorAll('article p');
        console.log(`Párrafos en articles: ${parrafosEnArticles.length}`);

        // 2. Solo el primer article
        const primerArticle = document.querySelector('article');
        console.log("Primer artículo:", primerArticle.querySelector('h3').textContent);

        // 3. Articles con clase "destacado"
        const destacados = document.querySelectorAll('article.destacado');
        console.log(`Artículos destacados: ${destacados.length}`);

        // 4. Primer hijo de .container (el h2)
        const primerHijo = document.querySelector('.container > :first-child');
        console.log("Primer hijo:", primerHijo.textContent);

        // 5. Último article
        const ultimoArticle = document.querySelector('article:last-of-type');
        console.log("Último artículo:", ultimoArticle.querySelector('h3').textContent);

        // 6. Todos los h3 dentro de .container
        const titulos = document.querySelectorAll('.container h3');
        console.log("Títulos encontrados:");
        titulos.forEach(titulo => {
            console.log(`  - ${titulo.textContent}`);
        });
    </script>
</body>
</html>
```

</details>

---

### Ejercicio 5: Búsqueda anidada
**Nivel**: ⭐⭐⭐☆☆

Selecciona un contenedor y luego busca elementos dentro de él.

<details>
<summary>Ver solución</summary>

```html
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <title>Búsqueda Anidada</title>
</head>
<body>
    <div id="sidebar">
        <h3>Sidebar</h3>
        <ul>
            <li>Link sidebar 1</li>
            <li>Link sidebar 2</li>
        </ul>
    </div>

    <div id="main">
        <h3>Contenido Principal</h3>
        <ul>
            <li>Link principal 1</li>
            <li>Link principal 2</li>
            <li>Link principal 3</li>
        </ul>
    </div>

    <script>
        // Seleccionar el contenedor principal
        const main = document.getElementById('main');
        
        // Buscar SOLO dentro de main
        const h3EnMain = main.querySelector('h3');
        console.log("Título en main:", h3EnMain.textContent);

        const itemsEnMain = main.querySelectorAll('li');
        console.log(`Items en main: ${itemsEnMain.length}`);
        
        itemsEnMain.forEach((item, index) => {
            console.log(`  ${index + 1}. ${item.textContent}`);
        });

        // Comparar con búsqueda global
        const todosLosLi = document.querySelectorAll('li');
        console.log(`\nTotal de <li> en toda la página: ${todosLosLi.length}`);
    </script>
</body>
</html>
```

</details>

---

### Ejercicio 6: Verificar existencia
**Nivel**: ⭐⭐⭐⭐☆

Crea una función que busque elementos y maneje correctamente cuando no existen.

<details>
<summary>Ver solución</summary>

```html
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <title>Verificar Existencia</title>
</head>
<body>
    <div id="existe">Este elemento existe</div>
    <p class="parrafo">Párrafo de ejemplo</p>

    <script>
        function buscarElemento(selector) {
            const elemento = document.querySelector(selector);
            
            if (elemento) {
                console.log(`✅ Encontrado: ${selector}`);
                console.log(`   Contenido: "${elemento.textContent}"`);
                return elemento;
            } else {
                console.log(`❌ No encontrado: ${selector}`);
                return null;
            }
        }

        // Probar con diferentes selectores
        console.log("=== Búsqueda de elementos ===\n");
        
        buscarElemento('#existe');
        buscarElemento('.parrafo');
        buscarElemento('#no-existe');
        buscarElemento('.clase-inexistente');
        
        // Uso seguro
        function modificarElemento(selector, nuevoTexto) {
            const elemento = document.querySelector(selector);
            
            if (elemento) {
                const textoAnterior = elemento.textContent;
                elemento.textContent = nuevoTexto;
                console.log(`\n✏️ Modificado "${selector}":`);
                console.log(`   Antes: "${textoAnterior}"`);
                console.log(`   Después: "${nuevoTexto}"`);
            } else {
                console.log(`\n⚠️ No se pudo modificar "${selector}" (no existe)`);
            }
        }

        modificarElemento('#existe', 'Texto cambiado');
        modificarElemento('#no-existe', 'Esto no funcionará');
    </script>
</body>
</html>
```

</details>

---

## Errores comunes

### ❌ Error 1: Usar # en getElementById

```javascript
// ❌ Error: getElementById no usa #
const elemento = document.getElementById('#header'); // null

// ✅ Correcto
const elemento = document.getElementById('header');
```

---

### ❌ Error 2: Usar . en getElementsByClassName

```javascript
// ❌ Error: no uses punto
const elementos = document.getElementsByClassName('.destacado'); // []

// ✅ Correcto
const elementos = document.getElementsByClassName('destacado');
```

---

### ❌ Error 3: Intentar usar forEach en HTMLCollection

```javascript
const elementos = document.getElementsByClassName('item');

// ❌ Error: HTMLCollection no tiene forEach
elementos.forEach(item => {
    console.log(item);
}); // TypeError

// ✅ Solución 1: Convertir a array
Array.from(elementos).forEach(item => {
    console.log(item);
});

// ✅ Solución 2: Usar for...of
for (const item of elementos) {
    console.log(item);
}

// ✅ Solución 3: Usar querySelectorAll (tiene forEach)
const elementos2 = document.querySelectorAll('.item');
elementos2.forEach(item => {
    console.log(item);
});
```

---

### ❌ Error 4: No verificar si el elemento existe

```javascript
// ❌ Error potencial
const elemento = document.querySelector('.no-existe');
elemento.textContent = "Hola"; // TypeError: Cannot set property of null

// ✅ Verificar primero
const elemento = document.querySelector('.no-existe');
if (elemento) {
    elemento.textContent = "Hola";
}
```

---

## Buenas prácticas

### ✅ 1. Usa querySelector/querySelectorAll por defecto

```javascript
// ✅ Preferible (sintaxis CSS familiar)
const elemento = document.querySelector('#miId');
const elementos = document.querySelectorAll('.miClase');

// También válido (pero menos flexible)
const elemento = document.getElementById('miId');
```

---

### ✅ 2. Almacena referencias a elementos que usarás múltiples veces

```javascript
// ❌ Ineficiente (busca cada vez)
document.querySelector('#header').textContent = "Hola";
document.querySelector('#header').style.color = "blue";
document.querySelector('#header').classList.add('activo');

// ✅ Eficiente (busca una vez, guarda referencia)
const header = document.querySelector('#header');
header.textContent = "Hola";
header.style.color = "blue";
header.classList.add('activo');
```

---

### ✅ 3. Limita el alcance de búsqueda

```javascript
// ❌ Busca en todo el documento
const items = document.querySelectorAll('.item');

// ✅ Busca solo dentro de un contenedor
const contenedor = document.querySelector('#lista');
const items = contenedor.querySelectorAll('.item');
```

---

### ✅ 4. Usa nombres descriptivos

```javascript
// ❌ No descriptivo
const e = document.querySelector('#h');
const x = document.querySelectorAll('.i');

// ✅ Descriptivo
const header = document.querySelector('#header');
const items = document.querySelectorAll('.item');
```

---

### ✅ 5. Verifica existencia antes de manipular

```javascript
// ✅ Seguro
const elemento = document.querySelector('.mi-clase');
if (elemento) {
    elemento.textContent = "Nuevo texto";
} else {
    console.warn("Elemento no encontrado");
}
```

---

## Cheatsheet

### querySelector / querySelectorAll

```javascript
// Un elemento (el primero)
document.querySelector('#id')           // Por ID
document.querySelector('.clase')        // Por clase
document.querySelector('div')           // Por etiqueta
document.querySelector('[name="email"]') // Por atributo

// Todos los elementos
document.querySelectorAll('.clase')     // NodeList
document.querySelectorAll('p')          // Todos los <p>
```

### getElementById / ClassName / TagName

```javascript
// Por ID (sin #)
document.getElementById('miId')

// Por clase (sin .)
document.getElementsByClassName('miClase') // HTMLCollection

// Por etiqueta
document.getElementsByTagName('p')         // HTMLCollection
document.getElementsByTagName('*')         // Todos los elementos
```

### Selectores CSS comunes

```javascript
'.clase'                    // Clase
'#id'                       // ID
'div'                       // Etiqueta
'.clase1.clase2'            // Múltiples clases
'div.clase'                 // div con clase
'div, p'                    // div O p
'div p'                     // p dentro de div (descendiente)
'div > p'                   // p hijo directo de div
'[href]'                    // Con atributo href
'[type="text"]'             // input type="text"
':first-child'              // Primer hijo
':last-child'               // Último hijo
':nth-child(2)'             // Segundo hijo
```

### Conversión y iteración

```javascript
// NodeList (tiene forEach)
const lista = document.querySelectorAll('.item');
lista.forEach(item => { });

// HTMLCollection (no tiene forEach)
const coleccion = document.getElementsByClassName('item');
Array.from(coleccion).forEach(item => { });
// o
for (const item of coleccion) { }
```

---

## Siguiente paso

Ya sabes cómo seleccionar elementos. Ahora aprenderás a **manipular su contenido, atributos y estilos**.

→ [24-manipular-contenido.md](24-manipular-contenido.md)

Ahí aprenderás a cambiar texto, HTML, atributos, clases CSS y estilos usando JavaScript.

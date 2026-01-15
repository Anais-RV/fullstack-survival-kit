# Módulo 22: Introducción al DOM

## Índice
1. [¿Qué es el DOM?](#qué-es-el-dom)
2. [El árbol del DOM](#el-árbol-del-dom)
3. [Tipos de nodos](#tipos-de-nodos)
4. [El objeto document](#el-objeto-document)
5. [Inspeccionar el DOM](#inspeccionar-el-dom)
6. [Ejercicios prácticos](#ejercicios-prácticos)
7. [Errores comunes](#errores-comunes)
8. [Buenas prácticas](#buenas-prácticas)
9. [Cheatsheet](#cheatsheet)

---

## ¿Qué es el DOM?

**DOM (Document Object Model)** es una representación en forma de objeto de tu página HTML que JavaScript puede manipular.

### Analogía: El documento como un árbol

Imagina tu HTML como un árbol genealógico donde cada elemento tiene:
- **Padre**: El elemento que lo contiene
- **Hijos**: Los elementos que contiene
- **Hermanos**: Elementos al mismo nivel

```html
<!DOCTYPE html>
<html>
  <head>
    <title>Mi página</title>
  </head>
  <body>
    <h1>Hola Mundo</h1>
    <p>Este es un párrafo</p>
  </body>
</html>
```

### El DOM convierte HTML en objetos

```
Document
  └── html
      ├── head
      │   └── title
      │       └── "Mi página"
      └── body
          ├── h1
          │   └── "Hola Mundo"
          └── p
              └── "Este es un párrafo"
```

Cada elemento HTML se convierte en un **objeto JavaScript** que puedes:
- Leer (obtener contenido, atributos)
- Modificar (cambiar texto, estilos)
- Crear (añadir nuevos elementos)
- Eliminar (borrar elementos)

---

## El árbol del DOM

### Ejemplo visual completo

```html
<!DOCTYPE html>
<html>
  <head>
    <title>Ejemplo DOM</title>
  </head>
  <body>
    <header>
      <h1 id="titulo">Mi Blog</h1>
      <nav>
        <a href="#home">Inicio</a>
        <a href="#about">Acerca</a>
      </nav>
    </header>
    <main>
      <article class="post">
        <h2>Primer artículo</h2>
        <p>Contenido del artículo...</p>
      </article>
    </main>
  </body>
</html>
```

### Representación en árbol

```
Document
  └── html
      ├── head
      │   └── title → "Ejemplo DOM"
      └── body
          ├── header
          │   ├── h1#titulo → "Mi Blog"
          │   └── nav
          │       ├── a → "Inicio"
          │       └── a → "Acerca"
          └── main
              └── article.post
                  ├── h2 → "Primer artículo"
                  └── p → "Contenido del artículo..."
```

### Relaciones entre nodos

```html
<div id="padre">
  <p id="hijo1">Primer hijo</p>
  <p id="hijo2">Segundo hijo</p>
  <p id="hijo3">Tercer hijo</p>
</div>
```

**Relaciones**:
- `div` es el **padre** de los tres `p`
- Los tres `p` son **hijos** del `div`
- Los tres `p` son **hermanos** entre sí
- `#hijo2` tiene `#hijo1` como **hermano anterior**
- `#hijo2` tiene `#hijo3` como **hermano siguiente**

---

## Tipos de nodos

El DOM tiene diferentes tipos de nodos:

### 1. Element Node (Nodo elemento)

Elementos HTML como `<div>`, `<p>`, `<h1>`, etc.

```html
<div>
  <p>Párrafo</p>
  <span>Texto</span>
</div>
```

### 2. Text Node (Nodo de texto)

El texto dentro de los elementos.

```html
<p>Este texto es un nodo de texto</p>
```

**Importante**: El texto "Este texto es un nodo de texto" es un nodo separado, hijo del `<p>`.

### 3. Attribute Node (Nodo atributo)

Los atributos de los elementos.

```html
<img src="imagen.jpg" alt="Descripción">
<!-- src y alt son nodos de atributo -->
```

### 4. Comment Node (Nodo comentario)

Los comentarios HTML.

```html
<!-- Este es un comentario -->
```

---

## El objeto document

**`document`** es el objeto principal que representa toda tu página HTML.

### Propiedades básicas

```javascript
// Título de la página
console.log(document.title); // "Ejemplo DOM"

// URL actual
console.log(document.URL); // "https://ejemplo.com/pagina.html"

// Dominio
console.log(document.domain); // "ejemplo.com"

// Tipo de documento
console.log(document.doctype); // <!DOCTYPE html>
```

### Acceso a elementos principales

```javascript
// El elemento <html>
console.log(document.documentElement);

// El elemento <head>
console.log(document.head);

// El elemento <body>
console.log(document.body);
```

### Modificar el título

```javascript
document.title = "Nuevo título";
// El título de la pestaña cambia
```

---

## Inspeccionar el DOM

### DevTools del navegador

**Abre las herramientas de desarrollo**:
- Chrome/Edge: `F12` o `Ctrl + Shift + I`
- Firefox: `F12` o `Ctrl + Shift + I`
- Safari: `Cmd + Option + I`

### Pestaña Elements/Inspector

Muestra el árbol del DOM y te permite:
- Ver la estructura HTML
- Inspeccionar elementos (click derecho → Inspeccionar)
- Ver estilos CSS aplicados
- Modificar HTML/CSS en tiempo real (temporal)

### Pestaña Console

Ejecuta JavaScript para interactuar con el DOM:

```javascript
// Ver el documento completo
console.log(document);

// Ver el body
console.log(document.body);

// Ver todos los párrafos
console.log(document.querySelectorAll('p'));
```

### Experimentar en la consola

```javascript
// Cambiar el color de fondo
document.body.style.backgroundColor = "lightblue";

// Cambiar todo el texto de un elemento
document.querySelector('h1').textContent = "¡Hola desde JavaScript!";

// Crear un alert con el título
alert(document.title);
```

---

## Ejercicios prácticos

### Ejercicio 1: Explorar document
**Nivel**: ⭐☆☆☆☆

Abre la consola del navegador en cualquier página web y explora las propiedades de `document`.

<details>
<summary>Ver solución</summary>

```javascript
// En la consola del navegador:

// 1. Ver el objeto document completo
console.log(document);

// 2. Ver el título
console.log("Título:", document.title);

// 3. Ver la URL
console.log("URL:", document.URL);

// 4. Ver el body
console.log("Body:", document.body);

// 5. Ver el head
console.log("Head:", document.head);

// 6. Ver el elemento html completo
console.log("HTML:", document.documentElement);

// 7. Contar cuántos párrafos hay
console.log("Párrafos:", document.querySelectorAll('p').length);
```

</details>

---

### Ejercicio 2: Cambiar el título
**Nivel**: ⭐☆☆☆☆

Crea una página HTML simple y usa JavaScript para cambiar su título.

<details>
<summary>Ver solución</summary>

```html
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Título Original</title>
</head>
<body>
    <h1>Mi Primera Página DOM</h1>
    <p>Abre la consola para ver el título cambiar</p>

    <script>
        // Mostrar título original
        console.log("Título original:", document.title);
        
        // Cambiar el título después de 2 segundos
        setTimeout(() => {
            document.title = "¡Título Cambiado!";
            console.log("Nuevo título:", document.title);
        }, 2000);
    </script>
</body>
</html>
```

</details>

---

### Ejercicio 3: Inspeccionar el árbol
**Nivel**: ⭐⭐☆☆☆

Dada una estructura HTML, identifica padres, hijos y hermanos de un elemento.

<details>
<summary>Ver solución</summary>

```html
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <title>Árbol DOM</title>
</head>
<body>
    <div id="contenedor">
        <h1 id="titulo">Título</h1>
        <p id="parrafo1">Primer párrafo</p>
        <p id="parrafo2">Segundo párrafo</p>
        <p id="parrafo3">Tercer párrafo</p>
    </div>

    <script>
        // Seleccionar el párrafo 2
        const p2 = document.getElementById('parrafo2');
        
        console.log("Elemento seleccionado:", p2);
        
        // Padre
        console.log("Padre:", p2.parentElement); // <div id="contenedor">
        
        // Hermano anterior
        console.log("Hermano anterior:", p2.previousElementSibling); // <p id="parrafo1">
        
        // Hermano siguiente
        console.log("Hermano siguiente:", p2.nextElementSibling); // <p id="parrafo3">
        
        // Todos los hijos del padre
        console.log("Hermanos (hijos del padre):", p2.parentElement.children);
    </script>
</body>
</html>
```

</details>

---

### Ejercicio 4: Contar elementos
**Nivel**: ⭐⭐☆☆☆

Crea una página con varios elementos y cuenta cuántos hay de cada tipo.

<details>
<summary>Ver solución</summary>

```html
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <title>Contador de Elementos</title>
</head>
<body>
    <h1>Mi Página</h1>
    <p>Primer párrafo</p>
    <p>Segundo párrafo</p>
    <div>
        <p>Tercer párrafo</p>
        <span>Un span</span>
    </div>
    <ul>
        <li>Item 1</li>
        <li>Item 2</li>
        <li>Item 3</li>
    </ul>

    <script>
        // Contar diferentes tipos de elementos
        const stats = {
            parrafos: document.querySelectorAll('p').length,
            divs: document.querySelectorAll('div').length,
            spans: document.querySelectorAll('span').length,
            listas: document.querySelectorAll('ul').length,
            items: document.querySelectorAll('li').length,
            encabezados: document.querySelectorAll('h1, h2, h3, h4, h5, h6').length
        };
        
        console.log("📊 Estadísticas de la página:");
        console.log(`Párrafos: ${stats.parrafos}`);
        console.log(`Divs: ${stats.divs}`);
        console.log(`Spans: ${stats.spans}`);
        console.log(`Listas: ${stats.listas}`);
        console.log(`Items de lista: ${stats.items}`);
        console.log(`Encabezados: ${stats.encabezados}`);
        
        // Total de elementos
        const total = document.querySelectorAll('*').length;
        console.log(`\nTotal de elementos: ${total}`);
    </script>
</body>
</html>
```

</details>

---

### Ejercicio 5: Información del documento
**Nivel**: ⭐⭐⭐☆☆

Crea una función que muestre información completa sobre el documento actual.

<details>
<summary>Ver solución</summary>

```html
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <title>Info del Documento</title>
    <style>
        .info-box {
            background: #f0f0f0;
            padding: 20px;
            border-radius: 8px;
            font-family: monospace;
        }
    </style>
</head>
<body>
    <h1>Información del Documento</h1>
    <div id="info" class="info-box"></div>

    <script>
        function obtenerInfoDocumento() {
            return {
                titulo: document.title,
                url: document.URL,
                dominio: document.domain,
                ultimaModificacion: document.lastModified,
                charset: document.characterSet,
                estadoLectura: document.readyState,
                referrer: document.referrer || "Acceso directo",
                estadisticas: {
                    totalElementos: document.querySelectorAll('*').length,
                    parrafos: document.querySelectorAll('p').length,
                    imagenes: document.querySelectorAll('img').length,
                    enlaces: document.querySelectorAll('a').length,
                    scripts: document.scripts.length,
                    estilos: document.styleSheets.length
                }
            };
        }
        
        function mostrarInfo() {
            const info = obtenerInfoDocumento();
            const infoDiv = document.getElementById('info');
            
            infoDiv.innerHTML = `
                <h2>📄 Información General</h2>
                <p><strong>Título:</strong> ${info.titulo}</p>
                <p><strong>URL:</strong> ${info.url}</p>
                <p><strong>Dominio:</strong> ${info.dominio}</p>
                <p><strong>Última modificación:</strong> ${info.ultimaModificacion}</p>
                <p><strong>Charset:</strong> ${info.charset}</p>
                <p><strong>Estado:</strong> ${info.estadoLectura}</p>
                <p><strong>Referrer:</strong> ${info.referrer}</p>
                
                <h2>📊 Estadísticas</h2>
                <p><strong>Total de elementos:</strong> ${info.estadisticas.totalElementos}</p>
                <p><strong>Párrafos:</strong> ${info.estadisticas.parrafos}</p>
                <p><strong>Imágenes:</strong> ${info.estadisticas.imagenes}</p>
                <p><strong>Enlaces:</strong> ${info.estadisticas.enlaces}</p>
                <p><strong>Scripts:</strong> ${info.estadisticas.scripts}</p>
                <p><strong>Hojas de estilo:</strong> ${info.estadisticas.estilos}</p>
            `;
        }
        
        // Mostrar info cuando el DOM esté listo
        document.addEventListener('DOMContentLoaded', mostrarInfo);
    </script>
</body>
</html>
```

</details>

---

## Errores comunes

### ❌ Error 1: Ejecutar JavaScript antes de cargar el DOM

```html
<!DOCTYPE html>
<html>
<head>
    <script>
        // ❌ Error: body aún no existe
        document.body.style.backgroundColor = "blue";
    </script>
</head>
<body>
    <h1>Mi página</h1>
</body>
</html>
```

**Soluciones**:

```html
<!-- ✅ Solución 1: Script al final del body -->
<!DOCTYPE html>
<html>
<head>
    <title>Mi página</title>
</head>
<body>
    <h1>Mi página</h1>
    
    <script>
        // ✅ Ahora body ya existe
        document.body.style.backgroundColor = "blue";
    </script>
</body>
</html>

<!-- ✅ Solución 2: Esperar a DOMContentLoaded -->
<!DOCTYPE html>
<html>
<head>
    <script>
        document.addEventListener('DOMContentLoaded', () => {
            // ✅ Se ejecuta cuando el DOM está listo
            document.body.style.backgroundColor = "blue";
        });
    </script>
</head>
<body>
    <h1>Mi página</h1>
</body>
</html>
```

---

### ❌ Error 2: Confundir nodos de elemento con nodos de texto

```html
<p id="texto">Hola Mundo</p>

<script>
// ❌ Intenta acceder al texto como propiedad
const p = document.getElementById('texto');
console.log(p); // <p id="texto">Hola Mundo</p> (elemento completo)

// ✅ Usa textContent o innerHTML
console.log(p.textContent); // "Hola Mundo"
</script>
```

---

### ❌ Error 3: No verificar si un elemento existe

```javascript
// ❌ Si el elemento no existe, causará error al intentar usarlo
const elemento = document.getElementById('noExiste');
elemento.textContent = "Nuevo texto"; // Error: Cannot read property 'textContent' of null

// ✅ Verifica primero
const elemento = document.getElementById('noExiste');
if (elemento) {
    elemento.textContent = "Nuevo texto";
} else {
    console.log("El elemento no existe");
}
```

---

## Buenas prácticas

### ✅ 1. Espera a que el DOM esté listo

```javascript
// ✅ Usa DOMContentLoaded
document.addEventListener('DOMContentLoaded', () => {
    // Tu código aquí
    console.log("DOM listo para manipular");
});
```

---

### ✅ 2. Coloca scripts al final del body

```html
<!DOCTYPE html>
<html>
<head>
    <title>Mi página</title>
    <!-- CSS aquí -->
</head>
<body>
    <!-- Contenido HTML -->
    
    <!-- ✅ Scripts al final -->
    <script src="mi-script.js"></script>
</body>
</html>
```

---

### ✅ 3. Usa nombres descriptivos para IDs

```html
<!-- ❌ No descriptivo -->
<div id="d1"></div>
<div id="x"></div>

<!-- ✅ Descriptivo -->
<div id="menu-principal"></div>
<div id="contenedor-articulos"></div>
```

---

### ✅ 4. Entiende la diferencia entre HTML y DOM

```html
<!-- HTML original -->
<p>Hola</p>

<script>
// JavaScript modifica el DOM (en memoria)
document.querySelector('p').textContent = "Adiós";

// El HTML original no cambia
// Pero lo que ves en el navegador sí cambia
</script>
```

---

### ✅ 5. Usa las DevTools para experimentar

```javascript
// Practica en la consola antes de escribir código
document.body.style.backgroundColor = "lightblue";
document.querySelector('h1').style.color = "red";
```

---

## Cheatsheet

### Objeto document

```javascript
document.title                  // Título de la página
document.URL                    // URL completa
document.domain                 // Dominio
document.documentElement        // <html>
document.head                   // <head>
document.body                   // <body>
```

### Propiedades de elementos

```javascript
elemento.parentElement          // Padre
elemento.children               // Hijos (HTMLCollection)
elemento.firstElementChild      // Primer hijo
elemento.lastElementChild       // Último hijo
elemento.nextElementSibling     // Hermano siguiente
elemento.previousElementSibling // Hermano anterior
```

### Eventos de documento

```javascript
// Esperar a que el DOM esté listo
document.addEventListener('DOMContentLoaded', () => {
    // Código aquí
});

// Esperar a que TODO (incluyendo imágenes) esté cargado
window.addEventListener('load', () => {
    // Código aquí
});
```

---

## Siguiente paso

Ahora que entiendes qué es el DOM y cómo está estructurado, aprenderás a **seleccionar elementos específicos** para manipularlos.

→ [23-seleccionar-elementos.md](23-seleccionar-elementos.md)

Ahí aprenderás todos los métodos para encontrar elementos: `querySelector`, `getElementById`, `getElementsByClassName` y más.

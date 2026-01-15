# Módulo 24: Manipular Contenido del DOM

## Índice
1. [¿Qué podemos manipular?](#qué-podemos-manipular)
2. [Manipular texto](#manipular-texto)
3. [Manipular HTML](#manipular-html)
4. [Manipular atributos](#manipular-atributos)
5. [Manipular clases CSS](#manipular-clases-css)
6. [Manipular estilos inline](#manipular-estilos-inline)
7. [Data attributes](#data-attributes)
8. [Ejercicios prácticos](#ejercicios-prácticos)
9. [Errores comunes](#errores-comunes)
10. [Buenas prácticas](#buenas-prácticas)
11. [Cheatsheet](#cheatsheet)

---

## ¿Qué podemos manipular?

Una vez seleccionado un elemento, podemos modificar:

- **Texto**: El contenido de texto
- **HTML**: La estructura HTML interna
- **Atributos**: src, href, id, class, etc.
- **Clases CSS**: Añadir, quitar, alternar clases
- **Estilos**: Cambiar CSS directamente
- **Data attributes**: Atributos personalizados

---

## Manipular texto

### textContent - Texto plano

**Obtiene o establece el texto** sin interpretar HTML.

```html
<p id="parrafo">Hola <strong>Mundo</strong></p>

<script>
const p = document.getElementById('parrafo');

// Leer texto
console.log(p.textContent); // "Hola Mundo" (solo texto)

// Cambiar texto
p.textContent = "Nuevo texto";
// Resultado: <p id="parrafo">Nuevo texto</p>

// Si pones HTML, lo trata como texto
p.textContent = "Texto con <strong>etiquetas</strong>";
// Resultado: <p id="parrafo">Texto con &lt;strong&gt;etiquetas&lt;/strong&gt;</p>
</script>
```

### innerText - Texto visible

Similar a `textContent`, pero considera estilos CSS (elementos ocultos, etc.).

```html
<div id="contenedor">
    Texto visible
    <span style="display: none;">Texto oculto</span>
</div>

<script>
const div = document.getElementById('contenedor');

console.log(div.textContent); // "Texto visible Texto oculto"
console.log(div.innerText);   // "Texto visible" (ignora lo oculto)
</script>
```

### ¿textContent o innerText?

```javascript
// ✅ textContent (más rápido, recomendado)
elemento.textContent = "Nuevo texto";

// ⚠️ innerText (más lento, considera estilos)
elemento.innerText = "Nuevo texto";

// Regla general: usa textContent
```

---

## Manipular HTML

### innerHTML - Contenido HTML

**Lee o establece el HTML interno** del elemento.

```html
<div id="contenedor">
    <p>Párrafo original</p>
</div>

<script>
const div = document.getElementById('contenedor');

// Leer HTML
console.log(div.innerHTML); 
// "<p>Párrafo original</p>"

// Cambiar HTML
div.innerHTML = "<h2>Nuevo título</h2><p>Nuevo párrafo</p>";

// Resultado:
// <div id="contenedor">
//     <h2>Nuevo título</h2>
//     <p>Nuevo párrafo</p>
// </div>
</script>
```

### Añadir HTML (+=)

```html
<ul id="lista">
    <li>Item 1</li>
</ul>

<script>
const lista = document.getElementById('lista');

// Añadir al final
lista.innerHTML += "<li>Item 2</li>";
lista.innerHTML += "<li>Item 3</li>";

// Resultado:
// <ul id="lista">
//     <li>Item 1</li>
//     <li>Item 2</li>
//     <li>Item 3</li>
// </ul>
</script>
```

### ⚠️ Cuidado con innerHTML

```javascript
// ❌ Peligro: XSS (Cross-Site Scripting)
const userInput = '<img src=x onerror="alert(\'Hacked\')">';
div.innerHTML = userInput; // ¡Ejecuta el código malicioso!

// ✅ Más seguro: usa textContent para contenido de usuario
div.textContent = userInput; // Muestra el texto, no lo ejecuta
```

---

## Manipular atributos

### getAttribute() y setAttribute()

```html
<img id="imagen" src="foto1.jpg" alt="Foto 1">
<a id="enlace" href="https://google.com">Google</a>

<script>
const img = document.getElementById('imagen');
const link = document.getElementById('enlace');

// LEER atributos
console.log(img.getAttribute('src')); // "foto1.jpg"
console.log(img.getAttribute('alt')); // "Foto 1"
console.log(link.getAttribute('href')); // "https://google.com"

// MODIFICAR atributos
img.setAttribute('src', 'foto2.jpg');
img.setAttribute('alt', 'Foto 2');
link.setAttribute('href', 'https://github.com');

// Resultado:
// <img id="imagen" src="foto2.jpg" alt="Foto 2">
// <a id="enlace" href="https://github.com">Google</a>
</script>
```

### hasAttribute() y removeAttribute()

```html
<input id="email" type="email" required>

<script>
const input = document.getElementById('email');

// Verificar si tiene un atributo
console.log(input.hasAttribute('required')); // true
console.log(input.hasAttribute('disabled')); // false

// Añadir atributo
input.setAttribute('placeholder', 'tu@email.com');

// Eliminar atributo
input.removeAttribute('required');

console.log(input.hasAttribute('required')); // false
</script>
```

### Acceso directo a atributos comunes

```html
<img id="imagen" src="foto.jpg" alt="Descripción">
<input id="campo" type="text" value="Hola">

<script>
const img = document.getElementById('imagen');
const input = document.getElementById('campo');

// Leer
console.log(img.src);   // Ruta completa: "http://...foto.jpg"
console.log(img.alt);   // "Descripción"
console.log(input.value); // "Hola"
console.log(input.type);  // "text"

// Modificar
img.src = "otra-foto.jpg";
img.alt = "Otra descripción";
input.value = "Nuevo valor";
input.type = "password";
</script>
```

---

## Manipular clases CSS

### className - Todas las clases como string

```html
<div id="caja" class="activo grande"></div>

<script>
const caja = document.getElementById('caja');

// Leer todas las clases
console.log(caja.className); // "activo grande"

// Reemplazar todas las clases
caja.className = "nueva-clase otra-clase";
console.log(caja.className); // "nueva-clase otra-clase"

// Añadir clase (concatenando)
caja.className += " tercera-clase";
console.log(caja.className); // "nueva-clase otra-clase tercera-clase"
</script>
```

### classList - Manipular clases individualmente (recomendado)

```html
<div id="caja" class="contenedor"></div>

<script>
const caja = document.getElementById('caja');

// ADD - Añadir clase
caja.classList.add('activo');
caja.classList.add('grande', 'destacado'); // Múltiples

// REMOVE - Eliminar clase
caja.classList.remove('grande');

// TOGGLE - Alternar (si existe la quita, si no la añade)
caja.classList.toggle('activo'); // La quita
caja.classList.toggle('activo'); // La añade

// CONTAINS - Verificar si tiene clase
console.log(caja.classList.contains('activo')); // true
console.log(caja.classList.contains('inexistente')); // false

// REPLACE - Reemplazar clase
caja.classList.replace('activo', 'inactivo');
</script>
```

### Ejemplo práctico: Toggle de menú

```html
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <title>Toggle Menú</title>
    <style>
        .menu {
            display: none;
        }
        .menu.visible {
            display: block;
        }
    </style>
</head>
<body>
    <button id="boton">Mostrar/Ocultar Menú</button>
    <nav id="menu" class="menu">
        <ul>
            <li>Inicio</li>
            <li>Servicios</li>
            <li>Contacto</li>
        </ul>
    </nav>

    <script>
        const boton = document.getElementById('boton');
        const menu = document.getElementById('menu');

        boton.addEventListener('click', () => {
            menu.classList.toggle('visible');
        });
    </script>
</body>
</html>
```

---

## Manipular estilos inline

### style - Modificar CSS directamente

```html
<div id="caja">Contenido</div>

<script>
const caja = document.getElementById('caja');

// Cambiar estilos individuales
caja.style.backgroundColor = "lightblue";
caja.style.color = "darkblue";
caja.style.padding = "20px";
caja.style.borderRadius = "8px";
caja.style.fontSize = "18px";

// Propiedades con guiones se convierten a camelCase
// CSS: background-color → JS: backgroundColor
// CSS: font-size → JS: fontSize
// CSS: border-radius → JS: borderRadius
</script>
```

### Leer estilos computados

```html
<div id="caja" style="background-color: red; width: 200px;"></div>

<script>
const caja = document.getElementById('caja');

// Leer estilo inline
console.log(caja.style.backgroundColor); // "red"
console.log(caja.style.width); // "200px"

// Leer estilos computados (incluyendo CSS externo)
const estilos = window.getComputedStyle(caja);
console.log(estilos.backgroundColor); // "rgb(255, 0, 0)"
console.log(estilos.width); // "200px"
console.log(estilos.display); // "block"
</script>
```

### cssText - Establecer múltiples estilos

```javascript
const caja = document.getElementById('caja');

// Establecer múltiples estilos a la vez
caja.style.cssText = `
    background-color: lightblue;
    color: darkblue;
    padding: 20px;
    border-radius: 8px;
`;

// O con una línea
caja.style.cssText = "background: blue; color: white; padding: 10px;";
```

### ⚠️ Estilos inline vs clases CSS

```javascript
// ❌ No recomendado: Estilos inline (difícil de mantener)
elemento.style.backgroundColor = "blue";
elemento.style.color = "white";
elemento.style.padding = "20px";

// ✅ Recomendado: Clases CSS (separación de responsabilidades)
// CSS:
// .destacado {
//     background-color: blue;
//     color: white;
//     padding: 20px;
// }

// JavaScript:
elemento.classList.add('destacado');
```

---

## Data attributes

**Atributos personalizados** para almacenar datos en elementos HTML.

### Definir data attributes en HTML

```html
<div id="producto" 
     data-id="123" 
     data-precio="29.99" 
     data-categoria="electronica"
     data-disponible="true">
    iPhone 15
</div>
```

### Acceder con dataset

```javascript
const producto = document.getElementById('producto');

// Leer data attributes
console.log(producto.dataset.id);        // "123"
console.log(producto.dataset.precio);    // "29.99"
console.log(producto.dataset.categoria); // "electronica"
console.log(producto.dataset.disponible); // "true"

// Modificar
producto.dataset.precio = "24.99";
producto.dataset.stock = "50"; // Añadir nuevo

// Eliminar
delete producto.dataset.categoria;
```

### Nombres con guiones

```html
<div id="elemento" data-fecha-creacion="2024-01-15"></div>

<script>
const elem = document.getElementById('elemento');

// data-fecha-creacion → dataset.fechaCreacion (camelCase)
console.log(elem.dataset.fechaCreacion); // "2024-01-15"

// Establecer
elem.dataset.fechaModificacion = "2024-01-20";
// Crea: data-fecha-modificacion="2024-01-20"
</script>
```

### Ejemplo práctico: Productos

```html
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <title>Data Attributes</title>
</head>
<body>
    <div class="producto" data-id="1" data-precio="299" data-nombre="Laptop">
        <h3>Laptop HP</h3>
        <button class="ver-detalles">Ver Detalles</button>
    </div>
    
    <div class="producto" data-id="2" data-precio="799" data-nombre="iPhone">
        <h3>iPhone 15</h3>
        <button class="ver-detalles">Ver Detalles</button>
    </div>

    <script>
        const botones = document.querySelectorAll('.ver-detalles');
        
        botones.forEach(boton => {
            boton.addEventListener('click', () => {
                // Acceder al padre (div.producto)
                const producto = boton.closest('.producto');
                
                // Leer data attributes
                const id = producto.dataset.id;
                const precio = producto.dataset.precio;
                const nombre = producto.dataset.nombre;
                
                console.log(`Producto: ${nombre}`);
                console.log(`ID: ${id}`);
                console.log(`Precio: $${precio}`);
            });
        });
    </script>
</body>
</html>
```

---

## Ejercicios prácticos

### Ejercicio 1: Cambiar texto
**Nivel**: ⭐☆☆☆☆

Cambia el texto de un párrafo cuando se carga la página.

<details>
<summary>Ver solución</summary>

```html
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <title>Cambiar Texto</title>
</head>
<body>
    <h1 id="titulo">Título Original</h1>
    <p id="descripcion">Descripción original</p>

    <script>
        // Cambiar texto
        const titulo = document.getElementById('titulo');
        const descripcion = document.getElementById('descripcion');
        
        titulo.textContent = "¡Nuevo Título!";
        descripcion.textContent = "Esta es la nueva descripción del párrafo.";
        
        console.log("Texto cambiado exitosamente");
    </script>
</body>
</html>
```

</details>

---

### Ejercicio 2: Modificar atributos de imagen
**Nivel**: ⭐⭐☆☆☆

Cambia la imagen src y el texto alternativo.

<details>
<summary>Ver solución</summary>

```html
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <title>Cambiar Imagen</title>
</head>
<body>
    <h2>Galería de Imágenes</h2>
    <img id="foto" src="foto1.jpg" alt="Foto 1" width="300">
    <br>
    <button onclick="cambiarImagen('foto2.jpg', 'Foto 2')">Foto 2</button>
    <button onclick="cambiarImagen('foto3.jpg', 'Foto 3')">Foto 3</button>

    <script>
        function cambiarImagen(nuevaSrc, nuevoAlt) {
            const img = document.getElementById('foto');
            img.src = nuevaSrc;
            img.alt = nuevoAlt;
            console.log(`Imagen cambiada a: ${nuevaSrc}`);
        }
    </script>
</body>
</html>
```

</details>

---

### Ejercicio 3: Toggle de clase
**Nivel**: ⭐⭐☆☆☆

Crea un botón que alterne una clase CSS para mostrar/ocultar contenido.

<details>
<summary>Ver solución</summary>

```html
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <title>Toggle Contenido</title>
    <style>
        .oculto {
            display: none;
        }
        .contenido {
            padding: 20px;
            background: #f0f0f0;
            border-radius: 8px;
            margin-top: 10px;
        }
    </style>
</head>
<body>
    <button id="toggle">Mostrar/Ocultar Contenido</button>
    <div id="contenido" class="contenido oculto">
        <h3>Contenido Oculto</h3>
        <p>Este contenido se puede mostrar u ocultar.</p>
    </div>

    <script>
        const boton = document.getElementById('toggle');
        const contenido = document.getElementById('contenido');
        
        boton.addEventListener('click', () => {
            contenido.classList.toggle('oculto');
            
            // Cambiar texto del botón según el estado
            if (contenido.classList.contains('oculto')) {
                boton.textContent = 'Mostrar Contenido';
            } else {
                boton.textContent = 'Ocultar Contenido';
            }
        });
    </script>
</body>
</html>
```

</details>

---

### Ejercicio 4: Cambiar estilos dinámicamente
**Nivel**: ⭐⭐⭐☆☆

Crea botones para cambiar el color de fondo y el tamaño de texto de un elemento.

<details>
<summary>Ver solución</summary>

```html
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <title>Cambiar Estilos</title>
    <style>
        #caja {
            padding: 30px;
            background-color: lightgray;
            font-size: 16px;
            transition: all 0.3s;
        }
        button {
            margin: 5px;
            padding: 10px 15px;
        }
    </style>
</head>
<body>
    <div id="caja">Este es un texto que cambiará de estilo</div>
    
    <h3>Colores:</h3>
    <button onclick="cambiarColor('lightblue')">Azul</button>
    <button onclick="cambiarColor('lightgreen')">Verde</button>
    <button onclick="cambiarColor('lightcoral')">Coral</button>
    
    <h3>Tamaño de texto:</h3>
    <button onclick="cambiarTamano('14px')">Pequeño</button>
    <button onclick="cambiarTamano('18px')">Mediano</button>
    <button onclick="cambiarTamano('24px')">Grande</button>
    
    <button onclick="resetear()">Resetear</button>

    <script>
        const caja = document.getElementById('caja');
        
        function cambiarColor(color) {
            caja.style.backgroundColor = color;
        }
        
        function cambiarTamano(tamano) {
            caja.style.fontSize = tamano;
        }
        
        function resetear() {
            caja.style.backgroundColor = 'lightgray';
            caja.style.fontSize = '16px';
        }
    </script>
</body>
</html>
```

</details>

---

### Ejercicio 5: Formulario con data attributes
**Nivel**: ⭐⭐⭐⭐☆

Crea un formulario que valide campos usando data attributes para reglas de validación.

<details>
<summary>Ver solución</summary>

```html
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <title>Validación con Data Attributes</title>
    <style>
        .error {
            border: 2px solid red;
        }
        .success {
            border: 2px solid green;
        }
        .mensaje-error {
            color: red;
            font-size: 14px;
            margin-top: 5px;
        }
        input {
            display: block;
            margin: 10px 0;
            padding: 8px;
            width: 300px;
        }
    </style>
</head>
<body>
    <h2>Formulario de Registro</h2>
    <form id="formulario">
        <input 
            type="text" 
            id="nombre" 
            placeholder="Nombre"
            data-min-length="3"
            data-error-message="El nombre debe tener al menos 3 caracteres">
        <span id="nombre-error" class="mensaje-error"></span>
        
        <input 
            type="email" 
            id="email" 
            placeholder="Email"
            data-required="true"
            data-error-message="Ingresa un email válido">
        <span id="email-error" class="mensaje-error"></span>
        
        <input 
            type="password" 
            id="password" 
            placeholder="Contraseña"
            data-min-length="6"
            data-error-message="La contraseña debe tener al menos 6 caracteres">
        <span id="password-error" class="mensaje-error"></span>
        
        <button type="submit">Registrar</button>
    </form>

    <script>
        const formulario = document.getElementById('formulario');
        
        formulario.addEventListener('submit', (e) => {
            e.preventDefault();
            
            let formularioValido = true;
            const campos = formulario.querySelectorAll('input');
            
            campos.forEach(campo => {
                const esValido = validarCampo(campo);
                if (!esValido) {
                    formularioValido = false;
                }
            });
            
            if (formularioValido) {
                alert('✅ Formulario enviado exitosamente');
                formulario.reset();
            }
        });
        
        function validarCampo(campo) {
            const valor = campo.value.trim();
            const minLength = campo.dataset.minLength;
            const mensajeError = campo.dataset.errorMessage;
            const spanError = document.getElementById(`${campo.id}-error`);
            
            // Limpiar estado anterior
            campo.classList.remove('error', 'success');
            spanError.textContent = '';
            
            // Validar longitud mínima
            if (minLength && valor.length < minLength) {
                campo.classList.add('error');
                spanError.textContent = mensajeError;
                return false;
            }
            
            // Validar email
            if (campo.type === 'email' && !valor.includes('@')) {
                campo.classList.add('error');
                spanError.textContent = mensajeError;
                return false;
            }
            
            // Validación exitosa
            campo.classList.add('success');
            return true;
        }
        
        // Validar en tiempo real
        const inputs = formulario.querySelectorAll('input');
        inputs.forEach(input => {
            input.addEventListener('blur', () => validarCampo(input));
        });
    </script>
</body>
</html>
```

</details>

---

## Errores comunes

### ❌ Error 1: Confundir textContent con innerHTML

```javascript
const div = document.getElementById('div');

// ❌ Si quieres texto plano, no uses innerHTML
div.innerHTML = "Solo texto"; // Funciona, pero innecesario

// ✅ Usa textContent para texto
div.textContent = "Solo texto";

// ✅ Usa innerHTML solo si necesitas HTML
div.innerHTML = "<strong>Texto</strong> con <em>formato</em>";
```

---

### ❌ Error 2: XSS con innerHTML

```javascript
// ❌ PELIGROSO: entrada de usuario sin sanitizar
const userInput = prompt("Ingresa tu nombre:");
div.innerHTML = userInput; // Si escriben <script>, se ejecuta

// ✅ SEGURO: usa textContent
div.textContent = userInput; // Muestra el texto tal cual
```

---

### ❌ Error 3: Usar className en lugar de classList

```javascript
// ❌ Difícil de mantener
elemento.className += " nueva-clase"; // Puede crear espacios dobles

// ✅ Más limpio
elemento.classList.add('nueva-clase');
```

---

### ❌ Error 4: Olvidar camelCase en style

```javascript
// ❌ Error: no existe background-color
elemento.style.background-color = "blue"; // SyntaxError

// ✅ Correcto: camelCase
elemento.style.backgroundColor = "blue";
```

---

## Buenas prácticas

### ✅ 1. Usa textContent por defecto, innerHTML solo cuando lo necesites

```javascript
// ✅ Para texto plano
elemento.textContent = "Texto simple";

// ✅ Solo usa innerHTML si necesitas HTML
elemento.innerHTML = "<strong>Texto</strong> <em>con formato</em>";
```

---

### ✅ 2. Prefiere classList sobre className

```javascript
// ✅ Más fácil y seguro
elemento.classList.add('activo');
elemento.classList.remove('inactivo');
elemento.classList.toggle('visible');
```

---

### ✅ 3. Usa clases CSS en lugar de estilos inline

```javascript
// ❌ Difícil de mantener
elemento.style.backgroundColor = "blue";
elemento.style.color = "white";
elemento.style.padding = "20px";

// ✅ Mejor: clases CSS
elemento.classList.add('estilo-destacado');
```

---

### ✅ 4. Usa data attributes para metadatos

```html
<!-- ✅ Correcto: data attributes para datos personalizados -->
<div data-user-id="123" data-role="admin"></div>

<!-- ❌ No uses atributos inventados -->
<div user-id="123" role-custom="admin"></div>
```

---

## Cheatsheet

### Texto

```javascript
elemento.textContent = "Texto"          // Leer/establecer texto plano
elemento.innerText = "Texto"            // Similar, considera CSS
```

### HTML

```javascript
elemento.innerHTML = "<p>HTML</p>"      // Leer/establecer HTML
elemento.innerHTML += "<p>Añadir</p>"   // Añadir al final
```

### Atributos

```javascript
elemento.getAttribute('src')            // Leer
elemento.setAttribute('src', 'img.jpg') // Establecer
elemento.hasAttribute('disabled')       // Verificar
elemento.removeAttribute('disabled')    // Eliminar

// Acceso directo
elemento.src = "img.jpg"
elemento.href = "url"
elemento.value = "texto"
```

### Clases

```javascript
elemento.classList.add('clase')         // Añadir
elemento.classList.remove('clase')      // Eliminar
elemento.classList.toggle('clase')      // Alternar
elemento.classList.contains('clase')    // Verificar
elemento.classList.replace('old', 'new') // Reemplazar
```

### Estilos

```javascript
elemento.style.backgroundColor = "blue" // Establecer
elemento.style.fontSize = "20px"        // camelCase
elemento.style.cssText = "color: red; padding: 10px;"
window.getComputedStyle(elemento)       // Leer estilos computados
```

### Data Attributes

```javascript
elemento.dataset.id                     // Leer data-id
elemento.dataset.userName               // Leer data-user-name
elemento.dataset.precio = "99"          // Establecer
delete elemento.dataset.precio          // Eliminar
```

---

## Siguiente paso

Ya sabes cómo manipular contenido, atributos y estilos. Ahora aprenderás a **responder a eventos** (clicks, inputs, etc.).

→ [25-eventos.md](25-eventos.md)

Ahí aprenderás a hacer tu página interactiva respondiendo a las acciones del usuario.

# Módulo 26: Crear y Eliminar Elementos del DOM

## Índice
1. [¿Por qué crear elementos dinámicamente?](#por-qué-crear-elementos-dinámicamente)
2. [createElement - Crear elementos](#createelement---crear-elementos)
3. [appendChild - Añadir al final](#appendchild---añadir-al-final)
4. [append y prepend](#append-y-prepend)
5. [insertBefore e insertAdjacentElement](#insertbefore-e-insertadjacentelement)
6. [removeChild y remove](#removechild-y-remove)
7. [cloneNode - Clonar elementos](#clonenode---clonar-elementos)
8. [replaceChild - Reemplazar elementos](#replacechild---reemplazar-elementos)
9. [Ejercicios prácticos](#ejercicios-prácticos)
10. [Errores comunes](#errores-comunes)
11. [Buenas prácticas](#buenas-prácticas)
12. [Cheatsheet](#cheatsheet)

---

## ¿Por qué crear elementos dinámicamente?

Crear elementos con JavaScript te permite:
- **Interfaces dinámicas**: Añadir/quitar contenido sin recargar
- **Listas dinámicas**: To-do lists, carritos de compra
- **Respuestas a datos**: Mostrar resultados de APIs
- **Interactividad**: Modales, notificaciones, mensajes

### Ejemplo: Añadir items a una lista

```html
<ul id="lista"></ul>
<button id="agregar">Agregar Item</button>

<script>
const lista = document.getElementById('lista');
const boton = document.getElementById('agregar');
let contador = 1;

boton.addEventListener('click', () => {
    // Crear elemento <li>
    const li = document.createElement('li');
    li.textContent = `Item ${contador++}`;
    
    // Añadir a la lista
    lista.appendChild(li);
});
</script>
```

---

## createElement - Crear elementos

**Crea un nuevo elemento HTML en memoria** (aún no está en la página).

### Sintaxis

```javascript
const elemento = document.createElement('etiqueta');
```

### Crear diferentes elementos

```javascript
const div = document.createElement('div');
const p = document.createElement('p');
const button = document.createElement('button');
const img = document.createElement('img');
const ul = document.createElement('ul');
const li = document.createElement('li');
```

### Configurar el elemento

```javascript
const p = document.createElement('p');

// Establecer texto
p.textContent = "Este es un párrafo creado con JavaScript";

// Establecer HTML
p.innerHTML = "Párrafo con <strong>formato</strong>";

// Añadir clases
p.classList.add('destacado');

// Establecer atributos
p.setAttribute('id', 'mi-parrafo');

// Estilos
p.style.color = "blue";
```

### Ejemplo completo

```html
<div id="contenedor"></div>
<button id="crear">Crear Párrafo</button>

<script>
const contenedor = document.getElementById('contenedor');
const boton = document.getElementById('crear');

boton.addEventListener('click', () => {
    // 1. Crear elemento
    const p = document.createElement('p');
    
    // 2. Configurar
    p.textContent = "Nuevo párrafo creado dinámicamente";
    p.classList.add('parrafo');
    p.style.backgroundColor = "lightyellow";
    p.style.padding = "10px";
    
    // 3. Añadir al DOM
    contenedor.appendChild(p);
});
</script>
```

---

## appendChild - Añadir al final

**Añade un elemento como último hijo**.

```html
<ul id="lista">
    <li>Item 1</li>
    <li>Item 2</li>
</ul>

<script>
const lista = document.getElementById('lista');

// Crear nuevo item
const nuevoItem = document.createElement('li');
nuevoItem.textContent = "Item 3";

// Añadir al final
lista.appendChild(nuevoItem);

// Resultado:
// <ul id="lista">
//     <li>Item 1</li>
//     <li>Item 2</li>
//     <li>Item 3</li>  ← Añadido aquí
// </ul>
</script>
```

### Crear múltiples elementos

```javascript
const lista = document.getElementById('lista');

for (let i = 1; i <= 5; i++) {
    const li = document.createElement('li');
    li.textContent = `Item ${i}`;
    lista.appendChild(li);
}
```

---

## append y prepend

**Métodos más modernos y flexibles**.

### append - Añadir al final

```javascript
const contenedor = document.getElementById('contenedor');

// Puede añadir elementos
const p = document.createElement('p');
p.textContent = "Párrafo";
contenedor.append(p);

// Puede añadir texto directamente
contenedor.append("Texto adicional");

// Puede añadir múltiples elementos a la vez
const div1 = document.createElement('div');
const div2 = document.createElement('div');
contenedor.append(div1, div2, "y más texto");
```

### prepend - Añadir al inicio

```javascript
const lista = document.getElementById('lista');

const primerItem = document.createElement('li');
primerItem.textContent = "Primer item";

// Añadir al inicio
lista.prepend(primerItem);

// Resultado:
// <ul>
//     <li>Primer item</li>  ← Añadido al inicio
//     <li>Item 1</li>
//     <li>Item 2</li>
// </ul>
```

### appendChild vs append

```javascript
// appendChild
// - Solo acepta Nodes (elementos)
// - Retorna el elemento añadido
const elemento = contenedor.appendChild(div);

// append
// - Acepta múltiples argumentos
// - Acepta texto directamente
// - No retorna nada (undefined)
contenedor.append(div1, div2, "texto");
```

---

## insertBefore e insertAdjacentElement

### insertBefore - Insertar antes de un elemento

```html
<ul id="lista">
    <li id="item2">Item 2</li>
    <li>Item 3</li>
</ul>

<script>
const lista = document.getElementById('lista');
const item2 = document.getElementById('item2');

// Crear nuevo elemento
const nuevoItem = document.createElement('li');
nuevoItem.textContent = "Item 1";

// Insertar ANTES de item2
lista.insertBefore(nuevoItem, item2);

// Resultado:
// <ul>
//     <li>Item 1</li>  ← Insertado aquí
//     <li id="item2">Item 2</li>
//     <li>Item 3</li>
// </ul>
</script>
```

### insertAdjacentElement - Insertar en posición específica

```html
<div id="contenedor">
    <p>Párrafo existente</p>
</div>

<script>
const contenedor = document.getElementById('contenedor');
const nuevo = document.createElement('p');
nuevo.textContent = "Nuevo párrafo";

// Posiciones:
// 'beforebegin' - Antes del elemento
contenedor.insertAdjacentElement('beforebegin', nuevo);

// 'afterbegin' - Primer hijo
contenedor.insertAdjacentElement('afterbegin', nuevo);

// 'beforeend' - Último hijo
contenedor.insertAdjacentElement('beforeend', nuevo);

// 'afterend' - Después del elemento
contenedor.insertAdjacentElement('afterend', nuevo);
</script>
```

### Visualización de posiciones

```html
<!-- beforebegin -->
<div id="target">
    <!-- afterbegin -->
    Contenido
    <!-- beforeend -->
</div>
<!-- afterend -->
```

---

## removeChild y remove

### remove - Eliminar elemento (moderno)

```html
<ul id="lista">
    <li id="item1">Item 1</li>
    <li id="item2">Item 2</li>
    <li id="item3">Item 3</li>
</ul>

<script>
const item2 = document.getElementById('item2');

// Eliminar elemento
item2.remove();

// Resultado:
// <ul>
//     <li id="item1">Item 1</li>
//     <li id="item3">Item 3</li>  ← item2 eliminado
// </ul>
</script>
```

### removeChild - Eliminar hijo (clásico)

```javascript
const lista = document.getElementById('lista');
const item2 = document.getElementById('item2');

// El padre elimina al hijo
lista.removeChild(item2);
```

### Eliminar todos los hijos

```javascript
const contenedor = document.getElementById('contenedor');

// Método 1: innerHTML
contenedor.innerHTML = '';

// Método 2: Bucle con removeChild
while (contenedor.firstChild) {
    contenedor.removeChild(contenedor.firstChild);
}

// Método 3: replaceChildren (moderno)
contenedor.replaceChildren();
```

### Ejemplo: Lista con botones de eliminar

```html
<ul id="lista"></ul>
<button id="agregar">Agregar Item</button>

<script>
const lista = document.getElementById('lista');
const btnAgregar = document.getElementById('agregar');
let contador = 1;

btnAgregar.addEventListener('click', () => {
    const li = document.createElement('li');
    li.innerHTML = `
        Item ${contador++}
        <button class="eliminar">✗</button>
    `;
    
    lista.appendChild(li);
});

// Delegación de eventos para eliminar
lista.addEventListener('click', (e) => {
    if (e.target.classList.contains('eliminar')) {
        e.target.closest('li').remove();
    }
});
</script>
```

---

## cloneNode - Clonar elementos

**Crea una copia de un elemento**.

### Clonación superficial (sin hijos)

```html
<div id="original">
    <p>Contenido</p>
</div>

<script>
const original = document.getElementById('original');

// Clonar solo el div (sin el <p>)
const clon = original.cloneNode(false);
console.log(clon); // <div id="original"></div>
</script>
```

### Clonación profunda (con hijos)

```javascript
const original = document.getElementById('original');

// Clonar el div y todo su contenido
const clon = original.cloneNode(true);

// Cambiar el ID para evitar duplicados
clon.id = 'clon';

// Añadir al DOM
document.body.appendChild(clon);
```

### Ejemplo práctico: Duplicar elementos

```html
<div class="tarjeta" id="tarjeta1">
    <h3>Título</h3>
    <p>Descripción de la tarjeta</p>
    <button class="duplicar">Duplicar</button>
</div>
<div id="contenedor"></div>

<script>
document.addEventListener('click', (e) => {
    if (e.target.classList.contains('duplicar')) {
        const tarjeta = e.target.closest('.tarjeta');
        
        // Clonar profundamente
        const clon = tarjeta.cloneNode(true);
        
        // Cambiar ID para evitar duplicados
        clon.id = `tarjeta${Date.now()}`;
        
        // Añadir al contenedor
        document.getElementById('contenedor').appendChild(clon);
    }
});
</script>
```

---

## replaceChild - Reemplazar elementos

**Reemplaza un hijo con otro elemento**.

```html
<ul id="lista">
    <li id="viejo">Item viejo</li>
    <li>Item 2</li>
</ul>

<script>
const lista = document.getElementById('lista');
const viejo = document.getElementById('viejo');

// Crear nuevo elemento
const nuevo = document.createElement('li');
nuevo.textContent = "Item nuevo";
nuevo.id = "nuevo";

// Reemplazar
lista.replaceChild(nuevo, viejo);

// Resultado:
// <ul>
//     <li id="nuevo">Item nuevo</li>  ← Reemplazado
//     <li>Item 2</li>
// </ul>
</script>
```

### replaceWith (moderno)

```javascript
const viejo = document.getElementById('viejo');
const nuevo = document.createElement('li');
nuevo.textContent = "Item nuevo";

// Más simple
viejo.replaceWith(nuevo);
```

---

## Ejercicios prácticos

### Ejercicio 1: Crear lista dinámica
**Nivel**: ⭐⭐☆☆☆

Crea una aplicación que añada items a una lista desde un input.

<details>
<summary>Ver solución</summary>

```html
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <title>Lista Dinámica</title>
</head>
<body>
    <h2>Mi Lista</h2>
    <input type="text" id="input" placeholder="Nuevo item...">
    <button id="agregar">Agregar</button>
    <ul id="lista"></ul>

    <script>
        const input = document.getElementById('input');
        const btnAgregar = document.getElementById('agregar');
        const lista = document.getElementById('lista');

        function agregarItem() {
            const texto = input.value.trim();
            
            if (texto === '') {
                alert('Por favor ingresa un texto');
                return;
            }

            // Crear elemento
            const li = document.createElement('li');
            li.textContent = texto;

            // Añadir a la lista
            lista.appendChild(li);

            // Limpiar input
            input.value = '';
            input.focus();
        }

        btnAgregar.addEventListener('click', agregarItem);
        input.addEventListener('keypress', (e) => {
            if (e.key === 'Enter') agregarItem();
        });
    </script>
</body>
</html>
```

</details>

---

### Ejercicio 2: Tarjetas de productos
**Nivel**: ⭐⭐⭐☆☆

Crea tarjetas de productos dinámicamente desde un array.

<details>
<summary>Ver solución</summary>

```html
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <title>Catálogo de Productos</title>
    <style>
        .contenedor {
            display: grid;
            grid-template-columns: repeat(auto-fill, minmax(200px, 1fr));
            gap: 20px;
            padding: 20px;
        }
        .tarjeta {
            border: 1px solid #ddd;
            border-radius: 8px;
            padding: 15px;
            text-align: center;
        }
        .tarjeta h3 {
            margin: 0 0 10px 0;
        }
        .precio {
            font-size: 24px;
            color: #2ecc71;
            font-weight: bold;
        }
    </style>
</head>
<body>
    <h1>Catálogo de Productos</h1>
    <div id="contenedor" class="contenedor"></div>

    <script>
        const productos = [
            { id: 1, nombre: "Laptop", precio: 799, stock: 5 },
            { id: 2, nombre: "Mouse", precio: 25, stock: 20 },
            { id: 3, nombre: "Teclado", precio: 45, stock: 15 },
            { id: 4, nombre: "Monitor", precio: 299, stock: 8 },
            { id: 5, nombre: "Webcam", precio: 89, stock: 12 }
        ];

        const contenedor = document.getElementById('contenedor');

        productos.forEach(producto => {
            // Crear tarjeta
            const tarjeta = document.createElement('div');
            tarjeta.className = 'tarjeta';
            tarjeta.dataset.id = producto.id;

            // Crear contenido
            const titulo = document.createElement('h3');
            titulo.textContent = producto.nombre;

            const precio = document.createElement('p');
            precio.className = 'precio';
            precio.textContent = `$${producto.precio}`;

            const stock = document.createElement('p');
            stock.textContent = `Stock: ${producto.stock}`;

            const boton = document.createElement('button');
            boton.textContent = "Agregar al carrito";
            boton.addEventListener('click', () => {
                alert(`${producto.nombre} agregado al carrito`);
            });

            // Ensamblar
            tarjeta.append(titulo, precio, stock, boton);
            contenedor.appendChild(tarjeta);
        });
    </script>
</body>
</html>
```

</details>

---

### Ejercicio 3: To-Do List completa
**Nivel**: ⭐⭐⭐⭐☆

Crea una aplicación completa de tareas con agregar, completar y eliminar.

<details>
<summary>Ver solución</summary>

```html
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <title>To-Do List Completa</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            max-width: 600px;
            margin: 50px auto;
            padding: 20px;
        }
        .input-container {
            display: flex;
            gap: 10px;
            margin-bottom: 20px;
        }
        #input-tarea {
            flex: 1;
            padding: 10px;
            font-size: 16px;
        }
        button {
            padding: 10px 20px;
            cursor: pointer;
        }
        .tarea {
            display: flex;
            justify-content: space-between;
            align-items: center;
            padding: 15px;
            margin: 10px 0;
            background: #f8f9fa;
            border-radius: 5px;
        }
        .tarea.completada {
            background: #d4edda;
            text-decoration: line-through;
            opacity: 0.7;
        }
        .tarea-texto {
            flex: 1;
        }
        .acciones {
            display: flex;
            gap: 5px;
        }
        .btn-completar {
            background: #28a745;
            color: white;
            border: none;
            padding: 5px 10px;
            border-radius: 3px;
        }
        .btn-eliminar {
            background: #dc3545;
            color: white;
            border: none;
            padding: 5px 10px;
            border-radius: 3px;
        }
        .estadisticas {
            margin-top: 20px;
            padding: 15px;
            background: #e9ecef;
            border-radius: 5px;
        }
    </style>
</head>
<body>
    <h1>📝 To-Do List</h1>
    
    <div class="input-container">
        <input type="text" id="input-tarea" placeholder="Nueva tarea...">
        <button id="btn-agregar">Agregar</button>
    </div>

    <div id="lista-tareas"></div>

    <div class="estadisticas">
        <p>Total: <span id="total">0</span></p>
        <p>Completadas: <span id="completadas">0</span></p>
        <p>Pendientes: <span id="pendientes">0</span></p>
    </div>

    <script>
        const input = document.getElementById('input-tarea');
        const btnAgregar = document.getElementById('btn-agregar');
        const listaTareas = document.getElementById('lista-tareas');
        
        let tareas = [];

        // Agregar tarea
        function agregarTarea() {
            const texto = input.value.trim();
            if (!texto) {
                alert('Ingresa una tarea');
                return;
            }

            const tarea = {
                id: Date.now(),
                texto: texto,
                completada: false
            };

            tareas.push(tarea);
            renderizarTareas();
            input.value = '';
        }

        // Completar tarea
        function completarTarea(id) {
            const tarea = tareas.find(t => t.id === id);
            if (tarea) {
                tarea.completada = !tarea.completada;
                renderizarTareas();
            }
        }

        // Eliminar tarea
        function eliminarTarea(id) {
            tareas = tareas.filter(t => t.id !== id);
            renderizarTareas();
        }

        // Renderizar todas las tareas
        function renderizarTareas() {
            // Limpiar lista
            listaTareas.innerHTML = '';

            // Crear elementos para cada tarea
            tareas.forEach(tarea => {
                const div = document.createElement('div');
                div.className = `tarea ${tarea.completada ? 'completada' : ''}`;
                div.dataset.id = tarea.id;

                const texto = document.createElement('span');
                texto.className = 'tarea-texto';
                texto.textContent = tarea.texto;

                const acciones = document.createElement('div');
                acciones.className = 'acciones';

                const btnCompletar = document.createElement('button');
                btnCompletar.className = 'btn-completar';
                btnCompletar.textContent = tarea.completada ? '↩' : '✓';
                btnCompletar.addEventListener('click', () => completarTarea(tarea.id));

                const btnEliminar = document.createElement('button');
                btnEliminar.className = 'btn-eliminar';
                btnEliminar.textContent = '✗';
                btnEliminar.addEventListener('click', () => eliminarTarea(tarea.id));

                acciones.append(btnCompletar, btnEliminar);
                div.append(texto, acciones);
                listaTareas.appendChild(div);
            });

            actualizarEstadisticas();
        }

        // Actualizar estadísticas
        function actualizarEstadisticas() {
            const total = tareas.length;
            const completadas = tareas.filter(t => t.completada).length;
            const pendientes = total - completadas;

            document.getElementById('total').textContent = total;
            document.getElementById('completadas').textContent = completadas;
            document.getElementById('pendientes').textContent = pendientes;
        }

        // Event listeners
        btnAgregar.addEventListener('click', agregarTarea);
        input.addEventListener('keypress', (e) => {
            if (e.key === 'Enter') agregarTarea();
        });

        // Renderizar inicial
        renderizarTareas();
    </script>
</body>
</html>
```

</details>

---

## Errores comunes

### ❌ Error 1: No añadir el elemento al DOM

```javascript
// ❌ El elemento se crea pero no se muestra
const div = document.createElement('div');
div.textContent = "Hola";
// Falta: añadirlo al DOM

// ✅ Correcto
const div = document.createElement('div');
div.textContent = "Hola";
document.body.appendChild(div); // Ahora se muestra
```

---

### ❌ Error 2: Usar innerHTML en bucles

```javascript
const lista = document.getElementById('lista');

// ❌ Ineficiente: recrea todo en cada iteración
for (let i = 0; i < 100; i++) {
    lista.innerHTML += `<li>Item ${i}</li>`;
}

// ✅ Eficiente: crea y añade elementos
for (let i = 0; i < 100; i++) {
    const li = document.createElement('li');
    li.textContent = `Item ${i}`;
    lista.appendChild(li);
}
```

---

### ❌ Error 3: IDs duplicados al clonar

```javascript
// ❌ Clon tiene el mismo ID
const original = document.getElementById('elemento');
const clon = original.cloneNode(true);
document.body.appendChild(clon); // Ahora hay 2 elementos con el mismo ID

// ✅ Cambiar el ID del clon
const clon = original.cloneNode(true);
clon.id = 'elemento-clon';
document.body.appendChild(clon);
```

---

## Buenas prácticas

### ✅ 1. Usa createElement para elementos, innerHTML para contenido estático

```javascript
// ✅ Para elementos dinámicos
const div = document.createElement('div');
div.textContent = userInput; // Seguro

// ✅ Para contenido estático conocido
div.innerHTML = '<h2>Título</h2><p>Texto</p>';
```

---

### ✅ 2. Crea elementos fuera del DOM, luego añádelos

```javascript
// ✅ Construye primero, añade después
const contenedor = document.createElement('div');
contenedor.classList.add('tarjeta');

const titulo = document.createElement('h3');
titulo.textContent = "Título";

const texto = document.createElement('p');
texto.textContent = "Descripción";

contenedor.append(titulo, texto);

// Una sola operación en el DOM
document.body.appendChild(contenedor);
```

---

### ✅ 3. Usa fragment para múltiples elementos

```javascript
// ✅ DocumentFragment para mejor rendimiento
const fragment = document.createDocumentFragment();

for (let i = 0; i < 100; i++) {
    const li = document.createElement('li');
    li.textContent = `Item ${i}`;
    fragment.appendChild(li);
}

// Una sola operación en el DOM
lista.appendChild(fragment);
```

---

## Cheatsheet

### Crear elementos

```javascript
document.createElement('div')           // Crear elemento
elemento.textContent = "Texto"          // Establecer texto
elemento.innerHTML = "<p>HTML</p>"      // Establecer HTML
elemento.classList.add('clase')         // Añadir clase
elemento.setAttribute('id', 'miId')     // Establecer atributo
```

### Añadir elementos

```javascript
padre.appendChild(hijo)                 // Añadir al final
padre.append(hijo1, hijo2, "texto")     // Múltiples al final
padre.prepend(hijo)                     // Añadir al inicio
padre.insertBefore(nuevo, referencia)   // Insertar antes
elemento.insertAdjacentElement(pos, nuevo) // Posición específica
```

### Eliminar elementos

```javascript
elemento.remove()                       // Eliminar elemento
padre.removeChild(hijo)                 // Padre elimina hijo
elemento.replaceWith(nuevo)             // Reemplazar
padre.replaceChild(nuevo, viejo)        // Padre reemplaza hijo
```

### Clonar

```javascript
elemento.cloneNode(false)               // Clon superficial
elemento.cloneNode(true)                // Clon profundo
```

---

## ¡Bloque 4 completado! 🎉

**¡Felicidades!** Has completado el Bloque 4: DOM.

Ahora dominas:
- ✅ Introducción al DOM
- ✅ Seleccionar elementos
- ✅ Manipular contenido y atributos
- ✅ Eventos e interactividad
- ✅ Crear y eliminar elementos dinámicamente

### Próximos pasos

Ya puedes crear páginas web completamente interactivas. El siguiente bloque te enseñará conceptos avanzados de JavaScript:

→ **Bloque 5: JavaScript Avanzado**

Ahí aprenderás:
- Higher-order functions
- Métodos de array avanzados
- Destructuring y spread operator
- Promises y async/await
- Modules (import/export)

¡Sigue adelante! 🚀

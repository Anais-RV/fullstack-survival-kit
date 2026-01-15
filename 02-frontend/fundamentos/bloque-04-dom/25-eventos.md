# Módulo 25: Eventos del DOM

## Índice
1. [¿Qué son los eventos?](#qué-son-los-eventos)
2. [addEventListener](#addeventlistener)
3. [Eventos de ratón](#eventos-de-ratón)
4. [Eventos de teclado](#eventos-de-teclado)
5. [Eventos de formulario](#eventos-de-formulario)
6. [El objeto Event](#el-objeto-event)
7. [Delegación de eventos](#delegación-de-eventos)
8. [Ejercicios prácticos](#ejercicios-prácticos)
9. [Errores comunes](#errores-comunes)
10. [Buenas prácticas](#buenas-prácticas)
11. [Cheatsheet](#cheatsheet)

---

## ¿Qué son los eventos?

**Los eventos son acciones que ocurren en la página** y a las que puedes responder con JavaScript.

### Ejemplos de eventos

- Click en un botón
- Escribir en un campo de texto
- Mover el ratón sobre un elemento
- Enviar un formulario
- Presionar una tecla
- Cargar la página
- Hacer scroll

### Analogía: Timbre de casa

```
Usuario → Presiona botón (evento)
Sistema → Detecta evento
JavaScript → Ejecuta función (respuesta)
```

---

## addEventListener

**El método moderno para escuchar eventos**.

### Sintaxis básica

```javascript
elemento.addEventListener('evento', function() {
    // Código a ejecutar
});
```

### Ejemplo: Click en botón

```html
<button id="boton">¡Hazme click!</button>

<script>
const boton = document.getElementById('boton');

boton.addEventListener('click', function() {
    console.log('¡Botón clickeado!');
    alert('Hola desde el evento click');
});
</script>
```

### Con arrow function

```javascript
const boton = document.getElementById('boton');

boton.addEventListener('click', () => {
    console.log('Click con arrow function');
});
```

### Con función nombrada

```javascript
const boton = document.getElementById('boton');

function manejarClick() {
    console.log('Click manejado por función nombrada');
}

boton.addEventListener('click', manejarClick);
```

### Múltiples listeners en el mismo elemento

```javascript
const boton = document.getElementById('boton');

// Puedes añadir varios listeners
boton.addEventListener('click', () => {
    console.log('Primer listener');
});

boton.addEventListener('click', () => {
    console.log('Segundo listener');
});

// Ambos se ejecutan cuando haces click
```

### removeEventListener

```javascript
const boton = document.getElementById('boton');

function manejarClick() {
    console.log('Click');
}

// Añadir
boton.addEventListener('click', manejarClick);

// Eliminar (debe ser la misma función)
boton.removeEventListener('click', manejarClick);
```

---

## Eventos de ratón

### click - Click en elemento

```html
<button id="boton">Click aquí</button>

<script>
const boton = document.getElementById('boton');

boton.addEventListener('click', () => {
    console.log('¡Click!');
});
</script>
```

### dblclick - Doble click

```html
<div id="caja">Haz doble click aquí</div>

<script>
const caja = document.getElementById('caja');

caja.addEventListener('dblclick', () => {
    console.log('¡Doble click!');
});
</script>
```

### mouseenter y mouseleave

```html
<div id="area" style="width: 200px; height: 200px; background: lightblue;">
    Pasa el ratón por aquí
</div>

<script>
const area = document.getElementById('area');

area.addEventListener('mouseenter', () => {
    console.log('Ratón entró');
    area.style.backgroundColor = 'lightgreen';
});

area.addEventListener('mouseleave', () => {
    console.log('Ratón salió');
    area.style.backgroundColor = 'lightblue';
});
</script>
```

### mouseover y mouseout (incluyen elementos hijos)

```javascript
// mouseenter/mouseleave: solo el elemento
// mouseover/mouseout: elemento y sus hijos

elemento.addEventListener('mouseover', () => {
    console.log('Ratón sobre elemento o hijo');
});
```

### mousemove - Movimiento del ratón

```html
<div id="area" style="width: 300px; height: 300px; background: #f0f0f0;">
    <p id="coordenadas">Coordenadas: </p>
</div>

<script>
const area = document.getElementById('area');
const coordenadas = document.getElementById('coordenadas');

area.addEventListener('mousemove', (e) => {
    coordenadas.textContent = `Coordenadas: X=${e.clientX}, Y=${e.clientY}`;
});
</script>
```

### contextmenu - Click derecho

```javascript
elemento.addEventListener('contextmenu', (e) => {
    e.preventDefault(); // Evita el menú contextual por defecto
    console.log('Click derecho detectado');
});
```

---

## Eventos de teclado

### keydown - Tecla presionada

```html
<input type="text" id="campo" placeholder="Escribe algo...">

<script>
const campo = document.getElementById('campo');

campo.addEventListener('keydown', (e) => {
    console.log(`Tecla presionada: ${e.key}`);
});
</script>
```

### keyup - Tecla soltada

```javascript
campo.addEventListener('keyup', (e) => {
    console.log(`Tecla soltada: ${e.key}`);
    console.log(`Valor actual: ${campo.value}`);
});
```

### Detectar teclas específicas

```javascript
document.addEventListener('keydown', (e) => {
    if (e.key === 'Enter') {
        console.log('Enter presionado');
    }
    
    if (e.key === 'Escape') {
        console.log('Escape presionado');
    }
    
    if (e.key === ' ') {
        console.log('Espacio presionado');
    }
});
```

### Detectar modificadores

```javascript
document.addEventListener('keydown', (e) => {
    if (e.ctrlKey) {
        console.log('Ctrl presionado');
    }
    
    if (e.shiftKey) {
        console.log('Shift presionado');
    }
    
    if (e.altKey) {
        console.log('Alt presionado');
    }
    
    // Combinación: Ctrl + S
    if (e.ctrlKey && e.key === 's') {
        e.preventDefault(); // Evita guardar la página
        console.log('Guardar documento');
    }
});
```

---

## Eventos de formulario

### submit - Envío de formulario

```html
<form id="formulario">
    <input type="text" name="nombre" placeholder="Nombre" required>
    <button type="submit">Enviar</button>
</form>

<script>
const formulario = document.getElementById('formulario');

formulario.addEventListener('submit', (e) => {
    e.preventDefault(); // Evita el envío por defecto
    
    const nombre = formulario.nombre.value;
    console.log(`Formulario enviado: ${nombre}`);
});
</script>
```

### input - Cambio en tiempo real

```html
<input type="text" id="busqueda" placeholder="Buscar...">
<p id="resultado"></p>

<script>
const busqueda = document.getElementById('busqueda');
const resultado = document.getElementById('resultado');

busqueda.addEventListener('input', (e) => {
    resultado.textContent = `Buscando: ${e.target.value}`;
});
</script>
```

### change - Cambio confirmado

```html
<select id="color">
    <option value="">Selecciona un color</option>
    <option value="rojo">Rojo</option>
    <option value="azul">Azul</option>
    <option value="verde">Verde</option>
</select>

<script>
const select = document.getElementById('color');

select.addEventListener('change', (e) => {
    console.log(`Color seleccionado: ${e.target.value}`);
});
</script>
```

### focus y blur

```html
<input type="text" id="campo" placeholder="Haz click aquí">

<script>
const campo = document.getElementById('campo');

campo.addEventListener('focus', () => {
    console.log('Campo enfocado');
    campo.style.backgroundColor = 'lightyellow';
});

campo.addEventListener('blur', () => {
    console.log('Campo desenfocado');
    campo.style.backgroundColor = 'white';
});
</script>
```

---

## El objeto Event

**Cada evento pasa un objeto Event con información sobre lo ocurrido**.

### Propiedades comunes

```javascript
elemento.addEventListener('click', (evento) => {
    // El elemento que disparó el evento
    console.log(evento.target);
    
    // El elemento que tiene el listener
    console.log(evento.currentTarget);
    
    // Tipo de evento
    console.log(evento.type); // "click"
    
    // Coordenadas del ratón
    console.log(evento.clientX, evento.clientY);
    
    // Modificadores
    console.log(evento.ctrlKey, evento.shiftKey, evento.altKey);
});
```

### target vs currentTarget

```html
<div id="padre" style="padding: 50px; background: lightblue;">
    <button id="hijo">Click aquí</button>
</div>

<script>
const padre = document.getElementById('padre');

padre.addEventListener('click', (e) => {
    console.log('target:', e.target.id);       // "hijo" (donde se hizo click)
    console.log('currentTarget:', e.currentTarget.id); // "padre" (quien tiene el listener)
});
</script>
```

### preventDefault() - Prevenir comportamiento por defecto

```javascript
// Evitar que un enlace navegue
enlace.addEventListener('click', (e) => {
    e.preventDefault();
    console.log('Navegación cancelada');
});

// Evitar envío de formulario
formulario.addEventListener('submit', (e) => {
    e.preventDefault();
    console.log('Envío cancelado');
});

// Evitar menú contextual
elemento.addEventListener('contextmenu', (e) => {
    e.preventDefault();
});
```

### stopPropagation() - Detener propagación

```html
<div id="padre">
    <button id="hijo">Click</button>
</div>

<script>
const padre = document.getElementById('padre');
const hijo = document.getElementById('hijo');

padre.addEventListener('click', () => {
    console.log('Click en padre');
});

hijo.addEventListener('click', (e) => {
    e.stopPropagation(); // Evita que el evento llegue al padre
    console.log('Click en hijo');
});
</script>
```

---

## Delegación de eventos

**Añadir un listener al padre en lugar de a cada hijo**.

### Sin delegación (ineficiente)

```javascript
// ❌ Añadir listener a cada elemento
const items = document.querySelectorAll('.item');
items.forEach(item => {
    item.addEventListener('click', () => {
        console.log('Click en item');
    });
});
```

### Con delegación (eficiente)

```html
<ul id="lista">
    <li class="item">Item 1</li>
    <li class="item">Item 2</li>
    <li class="item">Item 3</li>
</ul>

<script>
const lista = document.getElementById('lista');

// Un solo listener en el padre
lista.addEventListener('click', (e) => {
    if (e.target.classList.contains('item')) {
        console.log('Click en:', e.target.textContent);
    }
});
</script>
```

### Ventajas de la delegación

1. **Menos memoria**: Un listener en lugar de muchos
2. **Elementos dinámicos**: Funciona con elementos añadidos después
3. **Mejor rendimiento**: Especialmente con muchos elementos

### Ejemplo: Lista dinámica

```html
<ul id="lista"></ul>
<button id="agregar">Agregar Item</button>

<script>
const lista = document.getElementById('lista');
const botonAgregar = document.getElementById('agregar');
let contador = 1;

// Delegación: funciona con items futuros
lista.addEventListener('click', (e) => {
    if (e.target.tagName === 'LI') {
        console.log('Click en:', e.target.textContent);
        e.target.remove();
    }
});

// Añadir items dinámicamente
botonAgregar.addEventListener('click', () => {
    const li = document.createElement('li');
    li.textContent = `Item ${contador++}`;
    lista.appendChild(li);
});
</script>
```

---

## Ejercicios prácticos

### Ejercicio 1: Cambiar color al click
**Nivel**: ⭐☆☆☆☆

Crea un botón que cambie el color de fondo de la página.

<details>
<summary>Ver solución</summary>

```html
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <title>Cambiar Color</title>
</head>
<body>
    <h1>Click para cambiar color</h1>
    <button id="boton">Cambiar Color</button>

    <script>
        const boton = document.getElementById('boton');
        const colores = ['lightblue', 'lightgreen', 'lightcoral', 'lightyellow', 'lavender'];
        let indice = 0;

        boton.addEventListener('click', () => {
            document.body.style.backgroundColor = colores[indice];
            indice = (indice + 1) % colores.length; // Ciclo
        });
    </script>
</body>
</html>
```

</details>

---

### Ejercicio 2: Contador de clicks
**Nivel**: ⭐⭐☆☆☆

Crea un contador que aumente con cada click.

<details>
<summary>Ver solución</summary>

```html
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <title>Contador</title>
    <style>
        #contador {
            font-size: 48px;
            font-weight: bold;
            margin: 20px;
        }
        button {
            padding: 10px 20px;
            font-size: 18px;
            margin: 5px;
        }
    </style>
</head>
<body>
    <h1>Contador de Clicks</h1>
    <div id="contador">0</div>
    <button id="incrementar">➕ Incrementar</button>
    <button id="decrementar">➖ Decrementar</button>
    <button id="resetear">🔄 Resetear</button>

    <script>
        const contadorDiv = document.getElementById('contador');
        const btnIncrementar = document.getElementById('incrementar');
        const btnDecrementar = document.getElementById('decrementar');
        const btnResetear = document.getElementById('resetear');
        
        let contador = 0;

        btnIncrementar.addEventListener('click', () => {
            contador++;
            contadorDiv.textContent = contador;
        });

        btnDecrementar.addEventListener('click', () => {
            contador--;
            contadorDiv.textContent = contador;
        });

        btnResetear.addEventListener('click', () => {
            contador = 0;
            contadorDiv.textContent = contador;
        });
    </script>
</body>
</html>
```

</details>

---

### Ejercicio 3: Validación de formulario
**Nivel**: ⭐⭐⭐☆☆

Valida un formulario antes de enviarlo.

<details>
<summary>Ver solución</summary>

```html
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <title>Validación de Formulario</title>
    <style>
        .error {
            color: red;
            font-size: 14px;
        }
        input.invalido {
            border: 2px solid red;
        }
        input.valido {
            border: 2px solid green;
        }
    </style>
</head>
<body>
    <h2>Registro de Usuario</h2>
    <form id="formulario">
        <div>
            <input type="text" id="nombre" placeholder="Nombre">
            <span id="nombre-error" class="error"></span>
        </div>
        <div>
            <input type="email" id="email" placeholder="Email">
            <span id="email-error" class="error"></span>
        </div>
        <div>
            <input type="password" id="password" placeholder="Contraseña">
            <span id="password-error" class="error"></span>
        </div>
        <button type="submit">Registrar</button>
    </form>

    <script>
        const formulario = document.getElementById('formulario');
        const nombre = document.getElementById('nombre');
        const email = document.getElementById('email');
        const password = document.getElementById('password');

        formulario.addEventListener('submit', (e) => {
            e.preventDefault();
            
            let valido = true;

            // Validar nombre
            if (nombre.value.trim().length < 3) {
                mostrarError('nombre', 'El nombre debe tener al menos 3 caracteres');
                valido = false;
            } else {
                limpiarError('nombre');
            }

            // Validar email
            if (!email.value.includes('@')) {
                mostrarError('email', 'Email inválido');
                valido = false;
            } else {
                limpiarError('email');
            }

            // Validar contraseña
            if (password.value.length < 6) {
                mostrarError('password', 'La contraseña debe tener al menos 6 caracteres');
                valido = false;
            } else {
                limpiarError('password');
            }

            if (valido) {
                alert('✅ Formulario válido. Enviando...');
                formulario.reset();
            }
        });

        function mostrarError(campo, mensaje) {
            const input = document.getElementById(campo);
            const error = document.getElementById(`${campo}-error`);
            input.classList.add('invalido');
            input.classList.remove('valido');
            error.textContent = mensaje;
        }

        function limpiarError(campo) {
            const input = document.getElementById(campo);
            const error = document.getElementById(`${campo}-error`);
            input.classList.remove('invalido');
            input.classList.add('valido');
            error.textContent = '';
        }
    </script>
</body>
</html>
```

</details>

---

### Ejercicio 4: To-Do List con delegación
**Nivel**: ⭐⭐⭐⭐☆

Crea una lista de tareas con delegación de eventos.

<details>
<summary>Ver solución</summary>

```html
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <title>To-Do List</title>
    <style>
        .tarea {
            padding: 10px;
            margin: 5px 0;
            background: #f0f0f0;
            border-radius: 5px;
            display: flex;
            justify-content: space-between;
            align-items: center;
        }
        .tarea.completada {
            text-decoration: line-through;
            opacity: 0.6;
        }
        button {
            margin-left: 10px;
            padding: 5px 10px;
        }
    </style>
</head>
<body>
    <h1>📝 Lista de Tareas</h1>
    <input type="text" id="input-tarea" placeholder="Nueva tarea...">
    <button id="agregar">Agregar</button>
    <ul id="lista-tareas"></ul>

    <script>
        const input = document.getElementById('input-tarea');
        const btnAgregar = document.getElementById('agregar');
        const lista = document.getElementById('lista-tareas');

        // Agregar tarea
        btnAgregar.addEventListener('click', agregarTarea);
        input.addEventListener('keypress', (e) => {
            if (e.key === 'Enter') agregarTarea();
        });

        function agregarTarea() {
            const texto = input.value.trim();
            if (!texto) return;

            const li = document.createElement('li');
            li.className = 'tarea';
            li.innerHTML = `
                <span>${texto}</span>
                <div>
                    <button class="completar">✓</button>
                    <button class="eliminar">✗</button>
                </div>
            `;

            lista.appendChild(li);
            input.value = '';
        }

        // Delegación de eventos
        lista.addEventListener('click', (e) => {
            const tarea = e.target.closest('.tarea');
            if (!tarea) return;

            if (e.target.classList.contains('completar')) {
                tarea.classList.toggle('completada');
            }

            if (e.target.classList.contains('eliminar')) {
                tarea.remove();
            }
        });
    </script>
</body>
</html>
```

</details>

---

## Errores comunes

### ❌ Error 1: Llamar la función en lugar de pasarla

```javascript
// ❌ Error: ejecuta la función inmediatamente
boton.addEventListener('click', miFuncion());

// ✅ Correcto: pasa la referencia
boton.addEventListener('click', miFuncion);

// ✅ O usa arrow function
boton.addEventListener('click', () => miFuncion());
```

---

### ❌ Error 2: No usar preventDefault en formularios

```javascript
// ❌ Sin preventDefault, la página se recarga
formulario.addEventListener('submit', () => {
    console.log('Enviado');
});

// ✅ Con preventDefault
formulario.addEventListener('submit', (e) => {
    e.preventDefault();
    console.log('Enviado');
});
```

---

### ❌ Error 3: Listeners en bucle sin delegación

```javascript
// ❌ Ineficiente: muchos listeners
document.querySelectorAll('.item').forEach(item => {
    item.addEventListener('click', () => {
        console.log('Click');
    });
});

// ✅ Eficiente: un listener con delegación
document.getElementById('contenedor').addEventListener('click', (e) => {
    if (e.target.classList.contains('item')) {
        console.log('Click');
    }
});
```

---

## Buenas prácticas

### ✅ 1. Usa addEventListener en lugar de onclick

```javascript
// ❌ No recomendado
elemento.onclick = function() { };

// ✅ Recomendado
elemento.addEventListener('click', () => { });
```

---

### ✅ 2. Usa delegación para listas dinámicas

```javascript
// ✅ Un listener en el padre
contenedor.addEventListener('click', (e) => {
    if (e.target.matches('.item')) {
        // Manejar click
    }
});
```

---

### ✅ 3. Previene comportamientos por defecto cuando sea necesario

```javascript
// ✅ En formularios
formulario.addEventListener('submit', (e) => {
    e.preventDefault();
});

// ✅ En enlaces personalizados
enlace.addEventListener('click', (e) => {
    e.preventDefault();
});
```

---

## Cheatsheet

### Añadir eventos

```javascript
elemento.addEventListener('evento', funcion)
elemento.removeEventListener('evento', funcion)
```

### Eventos comunes

```javascript
// Ratón
'click', 'dblclick', 'mouseenter', 'mouseleave', 'mousemove'

// Teclado
'keydown', 'keyup', 'keypress'

// Formulario
'submit', 'input', 'change', 'focus', 'blur'

// Documento
'DOMContentLoaded', 'load', 'scroll', 'resize'
```

### Objeto Event

```javascript
event.target              // Elemento que disparó el evento
event.currentTarget       // Elemento con el listener
event.type                // Tipo de evento
event.preventDefault()    // Prevenir comportamiento por defecto
event.stopPropagation()   // Detener propagación
```

---

## Siguiente paso

Ya sabes cómo responder a eventos. Ahora aprenderás a **crear y eliminar elementos dinámicamente**.

→ [26-crear-eliminar-elementos.md](26-crear-eliminar-elementos.md)

Ahí aprenderás a construir contenido HTML desde JavaScript y modificar la estructura del DOM.

# Módulo 13: Introducción a JavaScript

## Índice
1. [¿Qué es JavaScript?](#qué-es-javascript)
2. [¿Para qué sirve?](#para-qué-sirve)
3. [Tu primer programa JavaScript](#tu-primer-programa-javascript)
4. [Cómo ejecutar JavaScript](#cómo-ejecutar-javascript)
5. [La consola del navegador](#la-consola-del-navegador)
6. [Comentarios en código](#comentarios-en-código)
7. [Ejercicios prácticos](#ejercicios-prácticos)
8. [Errores comunes](#errores-comunes)
9. [Buenas prácticas](#buenas-prácticas)
10. [Cheatsheet](#cheatsheet)

---

## ¿Qué es JavaScript?

**JavaScript es el lenguaje de programación de la web**. Es el que hace que las páginas sean interactivas, dinámicas y respondan a las acciones del usuario.

### Analogía: El trío inseparable

Imagina construir una casa:

- **HTML** es la estructura (ladrillos, paredes, habitaciones)
- **CSS** es la decoración (pintura, muebles, cortinas)
- **JavaScript** es la funcionalidad (electricidad, agua, ascensores)

Sin JavaScript, tendrías una casa bonita pero sin luz, sin agua y sin puertas automáticas. Con JavaScript, todo cobra vida.

### Historia breve

- **1995**: Creado por Brendan Eich en solo 10 días
- **1997**: Se estandariza como ECMAScript
- **2015**: ECMAScript 6 (ES6) trae cambios importantes
- **Hoy**: Uno de los lenguajes más populares del mundo

---

## ¿Para qué sirve?

JavaScript te permite:

### 1. Hacer páginas interactivas
- Responder a clics del usuario
- Validar formularios antes de enviarlos
- Crear animaciones y efectos visuales
- Cambiar el contenido de la página sin recargarla

### 2. Crear aplicaciones completas
- **Frontend**: Aplicaciones web (React, Vue, Angular)
- **Backend**: Servidores con Node.js
- **Móviles**: Apps con React Native
- **Desktop**: Aplicaciones con Electron

### 3. Manipular datos
- Procesar información del usuario
- Comunicarse con servidores (APIs)
- Almacenar datos localmente
- Hacer cálculos complejos

### Ejemplo visual

```html
<!-- Sin JavaScript: -->
<button>Hazme clic</button>
<!-- Nada pasa al hacer clic -->

<!-- Con JavaScript: -->
<button onclick="alert('¡Hola!')">Hazme clic</button>
<!-- Muestra un mensaje al hacer clic -->
```

---

## Tu primer programa JavaScript

### Opción 1: Dentro de HTML

```html
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <title>Mi primer JavaScript</title>
</head>
<body>
    <h1>Mi primera página con JS</h1>
    
    <!-- JavaScript dentro de la etiqueta <script> -->
    <script>
        console.log("¡Hola, mundo!");
    </script>
</body>
</html>
```

### Opción 2: Archivo externo (recomendado)

**HTML: index.html**
```html
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <title>Mi primer JavaScript</title>
</head>
<body>
    <h1>Mi primera página con JS</h1>
    
    <!-- JavaScript en archivo externo -->
    <script src="script.js"></script>
</body>
</html>
```

**JavaScript: script.js**
```javascript
// Mi primer programa JavaScript
console.log("¡Hola, mundo!");
console.log("Estoy aprendiendo JavaScript");
```

### ¿Dónde va la etiqueta `<script>`?

```html
<body>
    <h1>Contenido...</h1>
    
    <!-- Al final del body (recomendado) -->
    <script src="script.js"></script>
</body>
```

**¿Por qué al final?**
- El HTML se carga de arriba a abajo
- Si pones JS al principio, puede frenar la carga
- Al final, el contenido ya está visible cuando JS se ejecuta

---

## Cómo ejecutar JavaScript

### Método 1: En el navegador

1. Crea un archivo HTML con tu código
2. Ábrelo en el navegador (doble clic)
3. Abre la consola (F12 o clic derecho → Inspeccionar)
4. Ve a la pestaña "Console"
5. Verás los resultados de `console.log()`

### Método 2: Directamente en la consola

1. Abre cualquier página web
2. Presiona F12
3. Ve a la pestaña "Console"
4. Escribe código y presiona Enter

```javascript
// Prueba esto en la consola:
console.log("¡Funciona!");
```

### Método 3: Con Node.js (opcional)

Si instalas Node.js, puedes ejecutar JS desde la terminal:

```bash
# Crea un archivo
echo console.log("Hola desde Node") > test.js

# Ejecuta
node test.js
```

---

## La consola del navegador

La consola es tu mejor amiga al programar JavaScript.

### ¿Cómo abrirla?

| Sistema | Atajo |
|---------|-------|
| Windows/Linux | F12 o Ctrl + Shift + I |
| Mac | Cmd + Option + I |
| Cualquiera | Clic derecho → Inspeccionar → Console |

### ¿Para qué sirve?

#### 1. Ver mensajes con `console.log()`

```javascript
console.log("Mensaje simple");
console.log("Puedo ver", "múltiples", "valores");
console.log(123);
console.log(true);
```

#### 2. Ver errores

```javascript
// Error intencional:
console.log(variableQueNoExiste);
// Error: variableQueNoExiste is not defined
```

#### 3. Probar código rápidamente

```javascript
// Escribe directamente en la consola:
2 + 2          // 4
"Hola" + " mundo"  // "Hola mundo"
```

#### 4. Otros métodos útiles

```javascript
// Advertencia (amarillo)
console.warn("Esto es una advertencia");

// Error (rojo)
console.error("Esto es un error");

// Información (azul)
console.info("Información útil");

// Limpiar consola
console.clear();
```

### Ejemplo completo

```javascript
console.log("=== Probando la consola ===");
console.log("Nombre:", "Ana");
console.log("Edad:", 25);
console.log("¿Es estudiante?", true);
console.warn("Recuerda guardar tu trabajo");
console.error("Este es un ejemplo de error");
```

---

## Comentarios en código

Los comentarios son notas para ti y otros programadores. JavaScript los ignora.

### Comentarios de una línea

```javascript
// Este es un comentario de una línea
console.log("Hola"); // También puedo estar al final de una línea

// Útil para desactivar código:
// console.log("Esto no se ejecutará");
```

### Comentarios de múltiples líneas

```javascript
/*
  Este es un comentario
  que abarca múltiples líneas.
  Útil para explicaciones largas.
*/

console.log("Hola");

/*
console.log("Esto tampoco se ejecutará");
console.log("Ni esto");
*/
```

### ¿Cuándo usar comentarios?

#### ✅ Buenos comentarios

```javascript
// Calcula el precio con descuento del 20%
const precioFinal = precio * 0.8;

// TODO: Agregar validación de email
function registrarUsuario() {
    // ...
}

/*
  IMPORTANTE: Esta función debe llamarse
  después de cargar los datos del servidor
*/
```

#### ❌ Comentarios innecesarios

```javascript
// Suma dos números
const suma = 2 + 2; // ← obvio, no hace falta

// i es igual a 0
let i = 0; // ← tampoco ayuda
```

**Regla de oro**: Comenta el **por qué**, no el **qué**.

---

## Ejercicios prácticos

### Ejercicio 1: Hola mundo
**Nivel**: ⭐☆☆☆☆

Crea un archivo HTML que muestre "¡Hola, mundo!" en la consola.

<details>
<summary>Ver solución</summary>

```html
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <title>Ejercicio 1</title>
</head>
<body>
    <h1>Ejercicio 1: Hola mundo</h1>
    
    <script>
        console.log("¡Hola, mundo!");
    </script>
</body>
</html>
```

**Cómo verificar**:
1. Abre el archivo en el navegador
2. Abre la consola (F12)
3. Deberías ver el mensaje

</details>

---

### Ejercicio 2: Presentación
**Nivel**: ⭐☆☆☆☆

Muestra en la consola:
- Tu nombre
- Tu edad
- Si eres estudiante (true/false)

<details>
<summary>Ver solución</summary>

```javascript
console.log("Nombre: Ana García");
console.log("Edad: 25");
console.log("¿Estudiante?", true);

// O en una sola línea:
console.log("Nombre: Ana García, Edad: 25, Estudiante: true");
```

</details>

---

### Ejercicio 3: Calculadora simple
**Nivel**: ⭐⭐☆☆☆

Muestra en la consola el resultado de estas operaciones:
- 15 + 8
- 20 - 7
- 6 * 4
- 30 / 5

<details>
<summary>Ver solución</summary>

```javascript
console.log("15 + 8 =", 15 + 8);
console.log("20 - 7 =", 20 - 7);
console.log("6 × 4 =", 6 * 4);
console.log("30 ÷ 5 =", 30 / 5);

// Salida:
// 15 + 8 = 23
// 20 - 7 = 13
// 6 × 4 = 24
// 30 ÷ 5 = 6
```

</details>

---

### Ejercicio 4: Mensajes de consola
**Nivel**: ⭐⭐☆☆☆

Crea una secuencia de mensajes:
1. Un log normal con "Iniciando programa..."
2. Una advertencia con "Modo de prueba activado"
3. Un mensaje de información con "Versión: 1.0"
4. Un error con "Ejemplo de error"

<details>
<summary>Ver solución</summary>

```javascript
console.log("Iniciando programa...");
console.warn("Modo de prueba activado");
console.info("Versión: 1.0");
console.error("Ejemplo de error");
```

**En la consola verás**:
- El primer mensaje en color normal
- La advertencia en amarillo/naranja
- La información en azul
- El error en rojo

</details>

---

### Ejercicio 5: Comentarios útiles
**Nivel**: ⭐⭐☆☆☆

Toma este código sin comentarios y agrégale comentarios útiles:

```javascript
const precioOriginal = 100;
const descuento = 0.15;
const precioFinal = precioOriginal * (1 - descuento);
console.log(precioFinal);
```

<details>
<summary>Ver solución</summary>

```javascript
// Precio del producto sin descuento
const precioOriginal = 100;

// Descuento del 15% (0.15)
const descuento = 0.15;

// Calcula el precio final aplicando el descuento
// Fórmula: precio - (precio × descuento)
const precioFinal = precioOriginal * (1 - descuento);

// Muestra el resultado en la consola
console.log("Precio final:", precioFinal); // 85
```

**Nota**: Los comentarios explican **por qué** hacemos algo o **cómo** funciona algo no obvio, no simplemente **qué** hace cada línea.

</details>

---

### Ejercicio 6: Archivo externo
**Nivel**: ⭐⭐⭐☆☆

Crea una página HTML que cargue JavaScript desde un archivo externo llamado `script.js`. El script debe mostrar 5 mensajes diferentes en la consola.

<details>
<summary>Ver solución</summary>

**index.html**:
```html
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <title>Ejercicio 6</title>
</head>
<body>
    <h1>JavaScript desde archivo externo</h1>
    
    <script src="script.js"></script>
</body>
</html>
```

**script.js**:
```javascript
console.log("=== Inicio del programa ===");
console.log("Mensaje 1: JavaScript desde archivo externo");
console.log("Mensaje 2: Esto es más organizado");
console.log("Mensaje 3: Y más fácil de mantener");
console.warn("Mensaje 4: Este es una advertencia");
console.log("=== Fin del programa ===");
```

**Cómo verificar**:
1. Asegúrate de que ambos archivos estén en la misma carpeta
2. Abre `index.html` en el navegador
3. Abre la consola (F12)
4. Deberías ver los 5 mensajes

</details>

---

## Errores comunes

### ❌ Error 1: Olvidar la etiqueta `<script>`

```html
<!-- MAL: JavaScript sin <script> -->
<body>
    console.log("Hola");
</body>

<!-- BIEN: JavaScript dentro de <script> -->
<body>
    <script>
        console.log("Hola");
    </script>
</body>
```

---

### ❌ Error 2: Archivo no encontrado

```html
<!-- MAL: Ruta incorrecta -->
<script src="scrpt.js"></script> <!-- Typo: "scrpt" en vez de "script" -->

<!-- BIEN: Ruta correcta -->
<script src="script.js"></script>
```

**Síntomas**: No pasa nada, no hay errores, pero tampoco funciona.

**Solución**: Abre la consola y ve a la pestaña "Network". Verás el archivo en rojo si no se cargó.

---

### ❌ Error 3: JavaScript en el `<head>` sin defer

```html
<!-- MAL: JS se ejecuta antes de cargar el HTML -->
<head>
    <script src="script.js"></script>
</head>

<!-- BIEN Opción 1: Al final del body -->
<body>
    <!-- contenido -->
    <script src="script.js"></script>
</body>

<!-- BIEN Opción 2: En el head con defer -->
<head>
    <script src="script.js" defer></script>
</head>
```

---

### ❌ Error 4: Olvidar punto y coma

```javascript
// Funciona pero no es recomendable
console.log("Hola")
console.log("Mundo")

// Mejor: siempre usa punto y coma
console.log("Hola");
console.log("Mundo");
```

**Nota**: JavaScript permite omitir el punto y coma, pero puede causar problemas sutiles.

---

### ❌ Error 5: Consola cerrada

```html
<script>
    console.log("Si no abres la consola, no verás esto");
</script>
```

**Síntoma**: "No funciona mi código".

**Solución**: Presiona F12 y ve a la pestaña Console.

---

### ❌ Error 6: Comillas sin cerrar

```javascript
// MAL: Falta comilla al final
console.log("Hola mundo);

// BIEN: Comillas balanceadas
console.log("Hola mundo");
```

**Error**: `Unterminated string constant`

---

## Buenas prácticas

### ✅ 1. Usa archivos externos

```html
<!-- ❌ Evita esto para proyectos grandes -->
<script>
    // 500 líneas de JavaScript aquí...
</script>

<!-- ✅ Mejor: archivo externo -->
<script src="script.js"></script>
```

**Ventajas**:
- Código organizado
- Reutilizable en múltiples páginas
- Más fácil de mantener
- El navegador puede cachear el archivo

---

### ✅ 2. Pon `<script>` al final del `<body>`

```html
<body>
    <h1>Contenido de la página</h1>
    
    <!-- Scripts al final -->
    <script src="script.js"></script>
</body>
```

**¿Por qué?**
- El contenido se muestra más rápido
- JavaScript ya tiene acceso a todos los elementos HTML

---

### ✅ 3. Usa `console.log()` para debuggear

```javascript
const nombre = "Ana";
console.log("El nombre es:", nombre); // Verifica qué contiene la variable

const edad = 25;
console.log("La edad es:", edad); // Ayuda a encontrar errores
```

---

### ✅ 4. Escribe comentarios útiles

```javascript
// ❌ Evita esto: comenta lo obvio
let x = 5; // x es 5

// ✅ Mejor: explica por qué
let maxIntentos = 5; // Limitar a 5 intentos por razones de seguridad
```

---

### ✅ 5. Usa nombres descriptivos (adelanto)

```javascript
// ❌ Evita esto
let x = 25;
let y = "Ana";

// ✅ Mejor
let edad = 25;
let nombre = "Ana";
```

Veremos más sobre variables en el próximo módulo.

---

### ✅ 6. Mantén la consola abierta

Mientras desarrollas, ten siempre abierta la consola del navegador. Te avisará de errores inmediatamente.

---

### ✅ 7. Prueba en varios navegadores

A veces algo funciona en Chrome pero no en Firefox o Safari. Prueba en al menos 2 navegadores diferentes.

---

## Cheatsheet

### Incluir JavaScript en HTML

```html
<!-- Inline: dentro del HTML -->
<script>
    console.log("JavaScript inline");
</script>

<!-- Externo: archivo separado -->
<script src="script.js"></script>

<!-- Con defer: carga asíncrona -->
<script src="script.js" defer></script>
```

### Métodos de consola

```javascript
// Mensaje normal
console.log("Mensaje");

// Advertencia (amarillo)
console.warn("Advertencia");

// Error (rojo)
console.error("Error");

// Información (azul)
console.info("Info");

// Limpiar consola
console.clear();
```

### Comentarios

```javascript
// Comentario de una línea

/*
  Comentario de
  múltiples líneas
*/
```

### Atajos de teclado

| Acción | Windows/Linux | Mac |
|--------|---------------|-----|
| Abrir consola | F12 o Ctrl+Shift+I | Cmd+Option+I |
| Recargar página | Ctrl+R | Cmd+R |
| Recargar sin caché | Ctrl+Shift+R | Cmd+Shift+R |

### Operaciones básicas

```javascript
// Aritmética
console.log(5 + 3);  // 8
console.log(10 - 4); // 6
console.log(3 * 7);  // 21
console.log(20 / 4); // 5

// Texto
console.log("Hola" + " " + "mundo"); // "Hola mundo"
```

---

## Siguiente paso

Ya sabes qué es JavaScript, cómo ejecutarlo y cómo usar la consola. Ahora vamos a aprender a **almacenar datos**.

→ [14-variables-tipos.md](14-variables-tipos.md)

Ahí aprenderás `let`, `const`, tipos de datos y cómo nombrar variables correctamente.

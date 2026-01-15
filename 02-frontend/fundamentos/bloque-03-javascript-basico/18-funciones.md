# Módulo 18: Funciones en JavaScript

## Índice
1. [¿Qué son las funciones?](#qué-son-las-funciones)
2. [Declaración de funciones](#declaración-de-funciones)
3. [Parámetros y argumentos](#parámetros-y-argumentos)
4. [Return - Devolver valores](#return---devolver-valores)
5. [Arrow functions](#arrow-functions)
6. [Expresiones de función](#expresiones-de-función)
7. [Scope de funciones](#scope-de-funciones)
8. [Funciones como valores](#funciones-como-valores)
9. [Ejercicios prácticos](#ejercicios-prácticos)
10. [Errores comunes](#errores-comunes)
11. [Buenas prácticas](#buenas-prácticas)
12. [Cheatsheet](#cheatsheet)

---

## ¿Qué son las funciones?

**Una función es un bloque de código reutilizable que realiza una tarea específica**.

### Analogía: Receta de cocina

```
Receta: Hacer café ☕
─────────────────────
Ingredientes: agua, café molido
Pasos:
  1. Hervir agua
  2. Agregar café
  3. Filtrar
Resultado: Taza de café
```

En JavaScript:

```javascript
function hacerCafe(agua, cafe) {
    console.log("Hirviendo agua...");
    console.log("Agregando café...");
    console.log("Filtrando...");
    return "☕ Café listo";
}

const resultado = hacerCafe(200, "10g");
console.log(resultado); // "☕ Café listo"
```

### ¿Para qué sirven?

1. **Reutilización**: Escribe el código una vez, úsalo muchas veces
2. **Organización**: Divide código complejo en partes pequeñas
3. **Mantenimiento**: Cambia en un solo lugar
4. **Legibilidad**: El código es más fácil de entender
5. **Testing**: Puedes probar funciones individuales

### Sin funciones vs Con funciones

```javascript
// ❌ Sin funciones: Código repetitivo
console.log("Hola, Ana");
console.log("Hola, Juan");
console.log("Hola, María");

// ✅ Con funciones: Código reutilizable
function saludar(nombre) {
    console.log(`Hola, ${nombre}`);
}

saludar("Ana");
saludar("Juan");
saludar("María");
```

---

## Declaración de funciones

### Sintaxis básica

```javascript
function nombreFuncion() {
    // Código que se ejecuta
}
```

### Ejemplo simple

```javascript
function saludar() {
    console.log("¡Hola, mundo!");
}

// Llamar (invocar) la función
saludar(); // Output: "¡Hola, mundo!"
```

### Declarar vs Llamar

```javascript
// 1. DECLARACIÓN: Define la función (no se ejecuta)
function despedirse() {
    console.log("¡Adiós!");
}

// 2. LLAMADA: Ejecuta la función
despedirse(); // Ahora sí se ejecuta
```

**Importante**: Declarar la función no la ejecuta. Debes llamarla con `()`.

```javascript
function miFuncion() {
    console.log("Esto se ejecuta");
}

miFuncion;   // ❌ No hace nada (solo referencia)
miFuncion(); // ✅ Ejecuta la función
```

### Múltiples llamadas

```javascript
function decirHola() {
    console.log("¡Hola!");
}

decirHola(); // ¡Hola!
decirHola(); // ¡Hola!
decirHola(); // ¡Hola!
```

### Funciones con lógica

```javascript
function verificarMayorDeEdad() {
    const edad = 20;
    
    if (edad >= 18) {
        console.log("Mayor de edad");
    } else {
        console.log("Menor de edad");
    }
}

verificarMayorDeEdad(); // "Mayor de edad"
```

---

## Parámetros y argumentos

**Los parámetros son variables que recibe la función. Los argumentos son los valores que le pasas**.

### Función con un parámetro

```javascript
function saludar(nombre) {
    //         ^^^^^^ parámetro
    console.log(`Hola, ${nombre}`);
}

saludar("Ana");
//      ^^^^^ argumento
// Output: "Hola, Ana"
```

### Múltiples parámetros

```javascript
function sumar(a, b) {
    //        ^^^^^ parámetros
    const resultado = a + b;
    console.log(`${a} + ${b} = ${resultado}`);
}

sumar(5, 3);
//    ^^^^^ argumentos
// Output: "5 + 3 = 8"
```

### Parámetros vs Argumentos

```javascript
// Parámetros: en la definición
function presentar(nombre, edad, ciudad) {
    console.log(`Soy ${nombre}, tengo ${edad} años y vivo en ${ciudad}`);
}

// Argumentos: al llamar la función
presentar("Ana", 25, "Madrid");
// Output: "Soy Ana, tengo 25 años y vivo en Madrid"
```

### Orden de los argumentos

```javascript
function calcularDescuento(precio, descuento) {
    const precioFinal = precio - (precio * descuento);
    console.log(`Precio final: ${precioFinal}€`);
}

calcularDescuento(100, 0.2); // Precio: 100, Descuento: 20%
// Output: "Precio final: 80€"

calcularDescuento(0.2, 100); // ❌ Orden incorrecto
// Output: "Precio final: -19.96€" (¡Error!)
```

### Parámetros con valores por defecto

```javascript
function saludar(nombre = "invitado") {
    //                    ^^^^^^^^^^^ valor por defecto
    console.log(`Hola, ${nombre}`);
}

saludar("Ana");    // "Hola, Ana"
saludar();         // "Hola, invitado" (usa el valor por defecto)
saludar(undefined); // "Hola, invitado"
```

### Más ejemplos de valores por defecto

```javascript
function crearUsuario(nombre, edad = 18, pais = "España") {
    console.log(`Usuario: ${nombre}, ${edad} años, ${pais}`);
}

crearUsuario("Ana", 25, "México");
// "Usuario: Ana, 25 años, México"

crearUsuario("Juan", 30);
// "Usuario: Juan, 30 años, España"

crearUsuario("María");
// "Usuario: María, 18 años, España"
```

### Argumentos faltantes

```javascript
function multiplicar(a, b) {
    console.log(a * b);
}

multiplicar(5, 3);   // 15
multiplicar(5);      // NaN (b es undefined, 5 * undefined = NaN)
multiplicar();       // NaN (ambos son undefined)
```

---

## Return - Devolver valores

**`return` hace que la función devuelva un valor**.

### Sin return

```javascript
function sumar(a, b) {
    const resultado = a + b;
    console.log(resultado);
}

const valor = sumar(5, 3); // Muestra: 8
console.log(valor);        // undefined (no devolvió nada)
```

### Con return

```javascript
function sumar(a, b) {
    const resultado = a + b;
    return resultado; // ← Devuelve el valor
}

const valor = sumar(5, 3);
console.log(valor); // 8 (ahora sí tiene valor)
```

### Return directo

```javascript
function sumar(a, b) {
    return a + b; // Devuelve directamente
}

const resultado = sumar(10, 5);
console.log(resultado); // 15
```

### Return termina la función

```javascript
function verificarEdad(edad) {
    if (edad < 18) {
        return "Menor de edad";
        // ↓ Todo lo que está después de return no se ejecuta
    }
    
    console.log("Esta línea solo se ejecuta si edad >= 18");
    return "Mayor de edad";
}

console.log(verificarEdad(15)); // "Menor de edad"
console.log(verificarEdad(20)); // Muestra el console.log, luego "Mayor de edad"
```

### Return con condicionales

```javascript
function esPar(numero) {
    if (numero % 2 === 0) {
        return true;
    } else {
        return false;
    }
}

console.log(esPar(4));  // true
console.log(esPar(7));  // false
```

### Return simplificado

```javascript
// ❌ Redundante
function esPar(numero) {
    if (numero % 2 === 0) {
        return true;
    } else {
        return false;
    }
}

// ✅ Mejor: devuelve directamente el booleano
function esPar(numero) {
    return numero % 2 === 0;
}
```

### Usar el valor retornado

```javascript
function calcularIVA(precio) {
    return precio * 0.21;
}

const precioBase = 100;
const iva = calcularIVA(precioBase);
const precioTotal = precioBase + iva;

console.log(`Precio base: ${precioBase}€`);
console.log(`IVA (21%): ${iva}€`);
console.log(`Total: ${precioTotal}€`);

// Output:
// Precio base: 100€
// IVA (21%): 21€
// Total: 121€
```

### Funciones que llaman otras funciones

```javascript
function doblar(numero) {
    return numero * 2;
}

function triplicar(numero) {
    return numero * 3;
}

function procesarNumero(numero) {
    const doblado = doblar(numero);
    const triplicado = triplicar(numero);
    return doblado + triplicado;
}

console.log(procesarNumero(5)); // (5*2) + (5*3) = 10 + 15 = 25
```

---

## Arrow functions

**Las arrow functions son una forma más corta de escribir funciones**.

### Sintaxis tradicional vs Arrow

```javascript
// Función tradicional
function sumar(a, b) {
    return a + b;
}

// Arrow function
const sumar = (a, b) => {
    return a + b;
};
```

### Arrow function simplificada

```javascript
// Si solo hay una expresión, puedes omitir llaves y return
const sumar = (a, b) => a + b;

console.log(sumar(5, 3)); // 8
```

### Comparación completa

```javascript
// 1. Función tradicional (forma larga)
function multiplicar(a, b) {
    return a * b;
}

// 2. Arrow function (forma larga)
const multiplicar = (a, b) => {
    return a * b;
};

// 3. Arrow function (forma corta - sin llaves ni return)
const multiplicar = (a, b) => a * b;
```

### Un solo parámetro

```javascript
// Con paréntesis (opcional pero recomendado)
const doblar = (numero) => numero * 2;

// Sin paréntesis (posible pero menos común)
const doblar = numero => numero * 2;

console.log(doblar(5)); // 10
```

### Sin parámetros

```javascript
// Paréntesis vacíos obligatorios
const saludar = () => console.log("¡Hola!");

saludar(); // "¡Hola!"
```

### Arrow function con múltiples líneas

```javascript
const calcularDescuento = (precio, porcentaje) => {
    const descuento = precio * (porcentaje / 100);
    const precioFinal = precio - descuento;
    return precioFinal;
};

console.log(calcularDescuento(100, 20)); // 80
```

### Devolver objetos

```javascript
// ❌ Esto no funciona (JavaScript confunde las llaves)
const crearUsuario = (nombre, edad) => { nombre: nombre, edad: edad };

// ✅ Envuelve el objeto en paréntesis
const crearUsuario = (nombre, edad) => ({ nombre: nombre, edad: edad });

// ✅ O usa llaves y return explícito
const crearUsuario = (nombre, edad) => {
    return { nombre: nombre, edad: edad };
};

console.log(crearUsuario("Ana", 25));
// { nombre: "Ana", edad: 25 }
```

### ¿Cuándo usar arrow functions?

```javascript
// ✅ Arrow: Callbacks y funciones cortas
const numeros = [1, 2, 3, 4, 5];
const dobles = numeros.map(n => n * 2);

// ✅ Tradicional: Funciones con nombre y lógica compleja
function procesarPedido(pedido) {
    // ... muchas líneas de lógica ...
    return resultado;
}
```

---

## Expresiones de función

**Asignar una función a una variable**.

### Función tradicional vs Expresión

```javascript
// Declaración de función
function saludar() {
    console.log("Hola");
}

// Expresión de función
const saludar = function() {
    console.log("Hola");
};

// Ambas se llaman igual
saludar();
```

### Diferencia importante: Hoisting

```javascript
// ✅ FUNCIONA: Las declaraciones se "elevan"
saludar();

function saludar() {
    console.log("Hola");
}

// ❌ ERROR: Las expresiones NO se elevan
despedir(); // Error: despedir is not defined

const despedir = function() {
    console.log("Adiós");
};
```

### Expresiones de función anónimas

```javascript
// Sin nombre (anónima)
const sumar = function(a, b) {
    return a + b;
};

// Con nombre (útil para debugging)
const sumar = function sumar(a, b) {
    return a + b;
};
```

---

## Scope de funciones

**El scope determina dónde puedes acceder a una variable**.

### Variables locales

```javascript
function miFuncion() {
    const mensaje = "Hola"; // Variable local
    console.log(mensaje);   // ✅ Funciona
}

miFuncion();
console.log(mensaje); // ❌ Error: mensaje is not defined
```

### Variables globales

```javascript
const nombre = "Ana"; // Variable global

function saludar() {
    console.log(`Hola, ${nombre}`); // ✅ Accede a global
}

saludar(); // "Hola, Ana"
console.log(nombre); // "Ana"
```

### Shadowing (sombreado)

```javascript
const numero = 10; // Global

function mostrar() {
    const numero = 20; // Local (sombrea la global)
    console.log(numero); // 20 (usa la local)
}

mostrar();           // 20
console.log(numero); // 10 (global no cambia)
```

### Parámetros son locales

```javascript
function cambiarNombre(nombre) {
    nombre = "Nuevo nombre"; // Solo cambia la copia local
    console.log(nombre);     // "Nuevo nombre"
}

const miNombre = "Ana";
cambiarNombre(miNombre);
console.log(miNombre); // "Ana" (no cambió)
```

---

## Funciones como valores

**En JavaScript, las funciones son "ciudadanos de primera clase"**: puedes tratarlas como cualquier otro valor.

### Asignar a variables

```javascript
const miFuncion = function() {
    console.log("Hola");
};

miFuncion(); // "Hola"
```

### Pasar como argumentos

```javascript
function ejecutar(funcion) {
    funcion();
}

function saludar() {
    console.log("¡Hola!");
}

ejecutar(saludar); // Pasa la función como argumento
// Output: "¡Hola!"
```

### Devolver funciones

```javascript
function crearSaludo(idioma) {
    if (idioma === "es") {
        return function() { console.log("¡Hola!"); };
    } else if (idioma === "en") {
        return function() { console.log("Hello!"); };
    }
}

const saludoEspanol = crearSaludo("es");
const saludoIngles = crearSaludo("en");

saludoEspanol(); // "¡Hola!"
saludoIngles();  // "Hello!"
```

---

## Ejercicios prácticos

### Ejercicio 1: Función básica
**Nivel**: ⭐☆☆☆☆

Crea una función llamada `presentarse` que muestre "Hola, soy [tu nombre]".

<details>
<summary>Ver solución</summary>

```javascript
function presentarse() {
    console.log("Hola, soy Ana");
}

presentarse();
// Output: "Hola, soy Ana"
```

</details>

---

### Ejercicio 2: Función con parámetros
**Nivel**: ⭐☆☆☆☆

Crea una función `saludarPersona` que reciba un nombre y muestre "Hola, [nombre]".

<details>
<summary>Ver solución</summary>

```javascript
function saludarPersona(nombre) {
    console.log(`Hola, ${nombre}`);
}

saludarPersona("Juan");   // "Hola, Juan"
saludarPersona("María");  // "Hola, María"
```

</details>

---

### Ejercicio 3: Función con return
**Nivel**: ⭐⭐☆☆☆

Crea una función `calcularArea` que reciba base y altura, y devuelva el área de un rectángulo (base × altura).

<details>
<summary>Ver solución</summary>

```javascript
function calcularArea(base, altura) {
    return base * altura;
}

const area1 = calcularArea(5, 10);
console.log(`Área: ${area1}`); // "Área: 50"

const area2 = calcularArea(7, 3);
console.log(`Área: ${area2}`); // "Área: 21"
```

</details>

---

### Ejercicio 4: Arrow function
**Nivel**: ⭐⭐☆☆☆

Convierte esta función a arrow function:
```javascript
function multiplicar(a, b) {
    return a * b;
}
```

<details>
<summary>Ver solución</summary>

```javascript
// Opción 1: Con llaves
const multiplicar = (a, b) => {
    return a * b;
};

// Opción 2: Forma corta (recomendado)
const multiplicar = (a, b) => a * b;

console.log(multiplicar(4, 5)); // 20
console.log(multiplicar(3, 7)); // 21
```

</details>

---

### Ejercicio 5: Función con condicionales
**Nivel**: ⭐⭐☆☆☆

Crea una función `esParOImpar` que reciba un número y devuelva "par" o "impar".

<details>
<summary>Ver solución</summary>

```javascript
function esParOImpar(numero) {
    if (numero % 2 === 0) {
        return "par";
    } else {
        return "impar";
    }
}

// O más corto:
function esParOImpar(numero) {
    return numero % 2 === 0 ? "par" : "impar";
}

// O como arrow function:
const esParOImpar = (numero) => numero % 2 === 0 ? "par" : "impar";

console.log(esParOImpar(4));  // "par"
console.log(esParOImpar(7));  // "impar"
console.log(esParOImpar(10)); // "par"
```

</details>

---

### Ejercicio 6: Función con valores por defecto
**Nivel**: ⭐⭐☆☆☆

Crea una función `calcularPrecioFinal` que reciba precio e IVA (por defecto 21%), y devuelva el precio con IVA incluido.

<details>
<summary>Ver solución</summary>

```javascript
function calcularPrecioFinal(precio, iva = 21) {
    return precio + (precio * iva / 100);
}

console.log(calcularPrecioFinal(100));     // 121 (usa IVA por defecto)
console.log(calcularPrecioFinal(100, 10)); // 110 (IVA del 10%)
console.log(calcularPrecioFinal(50, 21));  // 60.5
```

</details>

---

### Ejercicio 7: Calculadora
**Nivel**: ⭐⭐⭐☆☆

Crea una función `calculadora` que reciba dos números y una operación (+, -, *, /), y devuelva el resultado.

<details>
<summary>Ver solución</summary>

```javascript
function calculadora(a, b, operacion) {
    switch (operacion) {
        case "+":
            return a + b;
        case "-":
            return a - b;
        case "*":
            return a * b;
        case "/":
            if (b !== 0) {
                return a / b;
            } else {
                return "Error: División por cero";
            }
        default:
            return "Operación no válida";
    }
}

console.log(calculadora(10, 5, "+")); // 15
console.log(calculadora(10, 5, "-")); // 5
console.log(calculadora(10, 5, "*")); // 50
console.log(calculadora(10, 5, "/")); // 2
console.log(calculadora(10, 0, "/")); // "Error: División por cero"
```

</details>

---

### Ejercicio 8: Función que usa otra función
**Nivel**: ⭐⭐⭐☆☆

Crea dos funciones:
- `celsiusAFahrenheit(celsius)`: convierte °C a °F (F = C × 9/5 + 32)
- `fahrenheitACelsius(fahrenheit)`: convierte °F a °C (C = (F - 32) × 5/9)

Luego crea `convertirTemperatura(valor, unidad)` que use las funciones anteriores.

<details>
<summary>Ver solución</summary>

```javascript
function celsiusAFahrenheit(celsius) {
    return celsius * 9/5 + 32;
}

function fahrenheitACelsius(fahrenheit) {
    return (fahrenheit - 32) * 5/9;
}

function convertirTemperatura(valor, unidad) {
    if (unidad === "C") {
        const fahrenheit = celsiusAFahrenheit(valor);
        return `${valor}°C = ${fahrenheit.toFixed(1)}°F`;
    } else if (unidad === "F") {
        const celsius = fahrenheitACelsius(valor);
        return `${valor}°F = ${celsius.toFixed(1)}°C`;
    } else {
        return "Unidad no válida (usa 'C' o 'F')";
    }
}

console.log(convertirTemperatura(0, "C"));    // "0°C = 32.0°F"
console.log(convertirTemperatura(100, "C"));  // "100°C = 212.0°F"
console.log(convertirTemperatura(32, "F"));   // "32°F = 0.0°C"
console.log(convertirTemperatura(98.6, "F")); // "98.6°F = 37.0°C"
```

</details>

---

### Ejercicio 9: Validador de contraseña
**Nivel**: ⭐⭐⭐⭐☆

Crea una función `esPasswordSegura` que valide:
- Longitud mínima de 8 caracteres
- Contiene al menos un número
- Contiene al menos una mayúscula

Devuelve `true` si cumple todos los requisitos, `false` si no.

<details>
<summary>Ver solución</summary>

```javascript
function esPasswordSegura(password) {
    // Verificar longitud
    if (password.length < 8) {
        return false;
    }
    
    // Verificar si contiene al menos un número
    let tieneNumero = false;
    for (let i = 0; i < password.length; i++) {
        if (password[i] >= "0" && password[i] <= "9") {
            tieneNumero = true;
            break;
        }
    }
    
    if (!tieneNumero) {
        return false;
    }
    
    // Verificar si contiene al menos una mayúscula
    let tieneMayuscula = false;
    for (let i = 0; i < password.length; i++) {
        if (password[i] >= "A" && password[i] <= "Z") {
            tieneMayuscula = true;
            break;
        }
    }
    
    if (!tieneMayuscula) {
        return false;
    }
    
    return true;
}

// Versión más compacta con regex (avanzado):
function esPasswordSegura(password) {
    return password.length >= 8 &&
           /\d/.test(password) &&        // Contiene dígito
           /[A-Z]/.test(password);        // Contiene mayúscula
}

console.log(esPasswordSegura("hola"));          // false (muy corta)
console.log(esPasswordSegura("hola1234"));      // false (sin mayúscula)
console.log(esPasswordSegura("Hola"));          // false (muy corta)
console.log(esPasswordSegura("Hola1234"));      // true
console.log(esPasswordSegura("MiPassword123")); // true
```

</details>

---

### Ejercicio 10: Generador de iniciales
**Nivel**: ⭐⭐⭐⭐☆

Crea una función `obtenerIniciales` que reciba un nombre completo y devuelva las iniciales en mayúsculas.

Ejemplo: "ana maría garcía" → "AMG"

<details>
<summary>Ver solución</summary>

```javascript
function obtenerIniciales(nombreCompleto) {
    const palabras = nombreCompleto.split(" ");
    let iniciales = "";
    
    for (let i = 0; i < palabras.length; i++) {
        if (palabras[i].length > 0) {
            iniciales += palabras[i][0].toUpperCase();
        }
    }
    
    return iniciales;
}

// Versión más compacta:
const obtenerIniciales = (nombre) =>
    nombre.split(" ")
        .map(palabra => palabra[0]?.toUpperCase() || "")
        .join("");

console.log(obtenerIniciales("ana garcía"));           // "AG"
console.log(obtenerIniciales("juan carlos pérez"));    // "JCP"
console.log(obtenerIniciales("maría"));                // "M"
console.log(obtenerIniciales("pedro de la torre"));    // "PDLT"
```

</details>

---

## Errores comunes

### ❌ Error 1: Olvidar los paréntesis al llamar

```javascript
function saludar() {
    console.log("Hola");
}

saludar;   // ❌ No hace nada (solo referencia la función)
saludar(); // ✅ Ejecuta la función
```

---

### ❌ Error 2: No devolver nada (cuando se espera un valor)

```javascript
function sumar(a, b) {
    a + b; // ❌ No hace nada con el resultado
}

const resultado = sumar(5, 3);
console.log(resultado); // undefined

// ✅ Solución: usa return
function sumar(a, b) {
    return a + b;
}
```

---

### ❌ Error 3: Código después de return

```javascript
function miFuncion() {
    return "Hola";
    console.log("Esto nunca se ejecuta"); // ❌ Código inalcanzable
}
```

---

### ❌ Error 4: Orden incorrecto de argumentos

```javascript
function dividir(dividendo, divisor) {
    return dividendo / divisor;
}

dividir(10, 2);  // ✅ 5 (correcto)
dividir(2, 10);  // ❌ 0.2 (orden incorrecto)
```

---

### ❌ Error 5: Modificar parámetros primitivos

```javascript
function cambiarNumero(numero) {
    numero = 100; // Solo cambia la copia local
}

let miNumero = 10;
cambiarNumero(miNumero);
console.log(miNumero); // 10 (no cambió)
```

---

### ❌ Error 6: Arrow function sin paréntesis en objeto

```javascript
// ❌ Error: JavaScript confunde las llaves
const crear = (nombre) => { nombre: nombre };

// ✅ Solución: envuelve en paréntesis
const crear = (nombre) => ({ nombre: nombre });
```

---

## Buenas prácticas

### ✅ 1. Una función, una responsabilidad

```javascript
// ❌ Hace demasiado
function procesarUsuario(usuario) {
    validarEmail(usuario.email);
    guardarEnBaseDatos(usuario);
    enviarEmailBienvenida(usuario);
    actualizarEstadisticas();
}

// ✅ Divide en funciones más pequeñas
function registrarUsuario(usuario) {
    if (!validarEmail(usuario.email)) return false;
    guardarUsuario(usuario);
    enviarBienvenida(usuario);
    return true;
}
```

---

### ✅ 2. Nombres descriptivos

```javascript
// ❌ No descriptivo
function proc(u) {
    // ...
}

// ✅ Descriptivo
function procesarUsuario(usuario) {
    // ...
}
```

---

### ✅ 3. Usa verbos para funciones

```javascript
// ✅ Buenos nombres de funciones
function calcularTotal() { }
function obtenerUsuarios() { }
function validarEmail() { }
function actualizarDatos() { }
function mostrarMensaje() { }
```

---

### ✅ 4. Parámetros con valores por defecto

```javascript
// ❌ Sin valor por defecto
function saludar(nombre) {
    nombre = nombre || "invitado"; // Forma antigua
    console.log(`Hola, ${nombre}`);
}

// ✅ Con valor por defecto
function saludar(nombre = "invitado") {
    console.log(`Hola, ${nombre}`);
}
```

---

### ✅ 5. Return temprano para validaciones

```javascript
// ❌ Anidamiento profundo
function procesarPedido(pedido) {
    if (pedido) {
        if (pedido.items.length > 0) {
            if (pedido.total > 0) {
                // procesar...
            }
        }
    }
}

// ✅ Return temprano
function procesarPedido(pedido) {
    if (!pedido) return;
    if (pedido.items.length === 0) return;
    if (pedido.total <= 0) return;
    
    // procesar...
}
```

---

### ✅ 6. Limita el número de parámetros

```javascript
// ❌ Demasiados parámetros
function crearUsuario(nombre, apellido, email, edad, ciudad, pais, telefono) {
    // ...
}

// ✅ Usa un objeto
function crearUsuario(datosUsuario) {
    const { nombre, apellido, email, edad, ciudad, pais, telefono } = datosUsuario;
    // ...
}

crearUsuario({
    nombre: "Ana",
    apellido: "García",
    email: "ana@email.com",
    edad: 25,
    ciudad: "Madrid",
    pais: "España",
    telefono: "123456789"
});
```

---

### ✅ 7. Funciones puras cuando sea posible

```javascript
// ❌ Función impura (depende de variable externa)
let total = 0;
function sumar(numero) {
    total += numero; // Modifica estado externo
}

// ✅ Función pura (solo usa sus parámetros)
function sumar(a, b) {
    return a + b; // No modifica nada externo
}
```

---

## Cheatsheet

### Declaración de funciones

```javascript
// Forma tradicional
function nombreFuncion(parametro1, parametro2) {
    // código
    return valor;
}

// Arrow function
const nombreFuncion = (parametro1, parametro2) => {
    // código
    return valor;
};

// Arrow function corta
const nombreFuncion = (parametro) => expresion;
```

### Parámetros

```javascript
// Sin parámetros
function saludar() { }

// Un parámetro
function saludar(nombre) { }

// Múltiples parámetros
function sumar(a, b, c) { }

// Valores por defecto
function saludar(nombre = "invitado") { }
```

### Return

```javascript
function sumar(a, b) {
    return a + b;
}

// Return termina la función
function verificar(valor) {
    if (valor < 0) return "negativo";
    if (valor === 0) return "cero";
    return "positivo";
}
```

### Arrow functions

```javascript
// Forma larga
const func = (a, b) => {
    return a + b;
};

// Forma corta (return implícito)
const func = (a, b) => a + b;

// Un parámetro (paréntesis opcionales)
const doblar = numero => numero * 2;

// Sin parámetros (paréntesis obligatorios)
const saludar = () => console.log("Hola");

// Devolver objeto (envolver en paréntesis)
const crear = (nombre) => ({ nombre: nombre });
```

### Patrones comunes

```javascript
// Validación
function esValido(valor) {
    if (!valor) return false;
    // ... más validaciones
    return true;
}

// Cálculo
function calcular(a, b) {
    const resultado = a + b;
    return resultado;
}

// Transformación
function convertir(valor) {
    return valor * 2;
}

// Callback
function ejecutar(callback) {
    callback();
}
```

---

## Siguiente paso

Ya sabes crear y usar funciones. Ahora vamos a trabajar con **colecciones de datos: arrays**.

→ [19-arrays.md](19-arrays.md)

Ahí aprenderás a crear arrays, acceder a elementos, métodos útiles y cómo iterar sobre ellos.

# Módulo 21: Scope y Hoisting

## Índice
1. [¿Qué es el scope?](#qué-es-el-scope)
2. [Tipos de scope](#tipos-de-scope)
3. [Scope chain (cadena de scope)](#scope-chain-cadena-de-scope)
4. [Hoisting](#hoisting)
5. [Temporal Dead Zone](#temporal-dead-zone)
6. [Closures (clausuras)](#closures-clausuras)
7. [Ejercicios prácticos](#ejercicios-prácticos)
8. [Errores comunes](#errores-comunes)
9. [Buenas prácticas](#buenas-prácticas)
10. [Cheatsheet](#cheatsheet)

---

## ¿Qué es el scope?

**El scope (alcance) es la región del código donde una variable es accesible**.

### Analogía: Habitaciones de una casa

```
Casa (Global Scope)
├── Cocina (Function Scope)
│   ├── Nevera (solo accesible en cocina)
│   └── Horno (solo accesible en cocina)
├── Dormitorio (Function Scope)
│   └── Armario (solo accesible en dormitorio)
└── Wi-Fi (accesible en toda la casa - global)
```

### Ejemplo visual

```javascript
// GLOBAL SCOPE
const ciudad = "Madrid"; // Accesible en todas partes

function saludar() {
    // FUNCTION SCOPE
    const nombre = "Ana"; // Solo accesible dentro de saludar()
    console.log(nombre);  // ✅ Funciona
    console.log(ciudad);  // ✅ Funciona (accede a global)
}

saludar();
console.log(ciudad);  // ✅ Funciona
console.log(nombre);  // ❌ Error: nombre is not defined
```

---

## Tipos de scope

### 1. Global Scope

**Variables declaradas fuera de funciones o bloques**.

```javascript
// Variables globales
const PI = 3.14159;
let contador = 0;
var mensaje = "Hola";

function calcularArea(radio) {
    return PI * radio * radio; // Accede a PI global
}

console.log(PI);       // 3.14159 (accesible)
console.log(contador); // 0 (accesible)
console.log(mensaje);  // "Hola" (accesible)
```

**Problema con globales**: Pueden causar conflictos y son difíciles de mantener.

```javascript
let usuario = "Ana";

function cambiarUsuario() {
    usuario = "Juan"; // Modifica la variable global
}

cambiarUsuario();
console.log(usuario); // "Juan" (cambió sin querer)
```

---

### 2. Function Scope

**Variables declaradas dentro de una función**.

```javascript
function calcular() {
    const resultado = 10 + 5; // Solo existe dentro de calcular()
    let temporal = 100;
    var antigua = 200;
    
    console.log(resultado); // ✅ Funciona
}

calcular();
console.log(resultado); // ❌ Error: resultado is not defined
console.log(temporal);  // ❌ Error: temporal is not defined
console.log(antigua);   // ❌ Error: antigua is not defined
```

**Scope anidado**: Las funciones internas acceden a variables de funciones externas.

```javascript
function exterior() {
    const mensaje = "Hola";
    
    function interior() {
        console.log(mensaje); // ✅ Accede a mensaje de exterior()
    }
    
    interior();
}

exterior(); // "Hola"
```

---

### 3. Block Scope

**Variables declaradas dentro de bloques `{ }` con `let` y `const`**.

```javascript
if (true) {
    const a = 10;  // Block scope
    let b = 20;    // Block scope
    var c = 30;    // Function scope (ignora el bloque)
}

console.log(a); // ❌ Error: a is not defined
console.log(b); // ❌ Error: b is not defined
console.log(c); // ✅ 30 (var ignora block scope)
```

### let y const respetan block scope

```javascript
for (let i = 0; i < 3; i++) {
    console.log(i); // 0, 1, 2
}
console.log(i); // ❌ Error: i is not defined

// Con var:
for (var j = 0; j < 3; j++) {
    console.log(j); // 0, 1, 2
}
console.log(j); // ✅ 3 (var ignora el bloque)
```

### Bloques en if

```javascript
if (true) {
    let mensaje = "Hola";
    const numero = 42;
    console.log(mensaje); // "Hola"
}

console.log(mensaje); // ❌ Error: mensaje is not defined
console.log(numero);  // ❌ Error: numero is not defined
```

---

## Scope Chain (cadena de scope)

**JavaScript busca variables desde el scope local hacia el global**.

### Ejemplo de cadena

```javascript
const global = "GLOBAL";

function nivel1() {
    const nivel1Var = "NIVEL 1";
    
    function nivel2() {
        const nivel2Var = "NIVEL 2";
        
        function nivel3() {
            const nivel3Var = "NIVEL 3";
            
            // Puede acceder a todas las variables superiores
            console.log(nivel3Var); // "NIVEL 3" (local)
            console.log(nivel2Var); // "NIVEL 2" (padre)
            console.log(nivel1Var); // "NIVEL 1" (abuelo)
            console.log(global);    // "GLOBAL" (global)
        }
        
        nivel3();
        console.log(nivel3Var); // ❌ Error (no puede acceder a hijos)
    }
    
    nivel2();
}

nivel1();
```

### Búsqueda en la cadena

```javascript
const color = "rojo";

function pintar() {
    // No hay 'color' local, busca en el scope superior
    console.log(color); // "rojo" (encontrado en global)
}

pintar();
```

### Shadowing (sombreado)

**Variable local oculta variable con el mismo nombre en scope superior**.

```javascript
const nombre = "Ana";

function saludar() {
    const nombre = "Juan"; // Sombrea la global
    console.log(nombre);   // "Juan" (usa la local)
}

saludar();
console.log(nombre); // "Ana" (la global no cambió)
```

---

## Hoisting

**JavaScript "mueve" declaraciones al inicio del scope antes de ejecutar el código**.

### Hoisting con var

```javascript
console.log(mensaje); // undefined (no error)
var mensaje = "Hola";
console.log(mensaje); // "Hola"

// JavaScript lo interpreta así:
var mensaje;              // Declaración se mueve arriba
console.log(mensaje);     // undefined
mensaje = "Hola";         // Inicialización queda aquí
console.log(mensaje);     // "Hola"
```

### Hoisting con let y const

```javascript
console.log(nombre); // ❌ ReferenceError: Cannot access before initialization
let nombre = "Ana";

console.log(edad); // ❌ ReferenceError: Cannot access before initialization
const edad = 25;
```

**let y const SÍ se hacen hoisting, pero NO se pueden usar antes de su declaración**.

---

### Hoisting con funciones

#### Function declaration (se hace hoisting completo)

```javascript
saludar(); // ✅ "Hola" (funciona antes de declarar)

function saludar() {
    console.log("Hola");
}

// JavaScript lo interpreta así:
function saludar() {
    console.log("Hola");
}
saludar(); // "Hola"
```

#### Function expression (NO se hace hoisting)

```javascript
saludar(); // ❌ TypeError: saludar is not a function

const saludar = function() {
    console.log("Hola");
};

// JavaScript lo interpreta así:
const saludar; // Declaración
saludar();     // Error: saludar es undefined
saludar = function() { ... }; // Inicialización
```

---

## Temporal Dead Zone

**Zona donde una variable existe pero no se puede acceder**.

### let y const tienen TDZ

```javascript
console.log(nombre); // ❌ ReferenceError
// ← TEMPORAL DEAD ZONE (no se puede usar)
let nombre = "Ana";
console.log(nombre); // ✅ "Ana"
```

### var NO tiene TDZ

```javascript
console.log(mensaje); // ✅ undefined (no error)
var mensaje = "Hola";
console.log(mensaje); // "Hola"
```

### TDZ en bloques

```javascript
{
    // ← TDZ empieza aquí
    console.log(x); // ❌ ReferenceError
    // ← TDZ continúa...
    let x = 5; // ← TDZ termina aquí
    console.log(x); // ✅ 5
}
```

---

## Closures (clausuras)

**Una función que "recuerda" variables de su scope exterior, incluso después de que la función exterior haya terminado**.

### Ejemplo básico

```javascript
function crearContador() {
    let contador = 0; // Variable privada
    
    return function() {
        contador++;
        return contador;
    };
}

const incrementar = crearContador();

console.log(incrementar()); // 1
console.log(incrementar()); // 2
console.log(incrementar()); // 3

// contador no es accesible desde fuera
console.log(contador); // ❌ Error: contador is not defined
```

### ¿Por qué funciona?

```javascript
function crearContador() {
    let contador = 0;
    
    return function() {
        contador++; // Accede a contador (closure)
        return contador;
    };
}
// crearContador() termina de ejecutarse
// Pero la función retornada sigue teniendo acceso a 'contador'
```

---

### Closures con múltiples funciones

```javascript
function crearCuenta(saldoInicial) {
    let saldo = saldoInicial; // Variable privada
    
    return {
        depositar(cantidad) {
            saldo += cantidad;
            console.log(`Depositado: ${cantidad}€. Saldo: ${saldo}€`);
        },
        
        retirar(cantidad) {
            if (cantidad > saldo) {
                console.log("Saldo insuficiente");
            } else {
                saldo -= cantidad;
                console.log(`Retirado: ${cantidad}€. Saldo: ${saldo}€`);
            }
        },
        
        verSaldo() {
            console.log(`Saldo actual: ${saldo}€`);
        }
    };
}

const miCuenta = crearCuenta(100);
miCuenta.verSaldo();      // "Saldo actual: 100€"
miCuenta.depositar(50);   // "Depositado: 50€. Saldo: 150€"
miCuenta.retirar(30);     // "Retirado: 30€. Saldo: 120€"
miCuenta.verSaldo();      // "Saldo actual: 120€"

// saldo es privado, no se puede acceder directamente
console.log(miCuenta.saldo); // undefined
```

---

### Closures en loops (problema clásico)

```javascript
// ❌ Problema con var
for (var i = 0; i < 3; i++) {
    setTimeout(function() {
        console.log(i); // 3, 3, 3 (todas acceden al mismo i)
    }, 1000);
}

// ✅ Solución con let (block scope)
for (let i = 0; i < 3; i++) {
    setTimeout(function() {
        console.log(i); // 0, 1, 2 (cada iteración tiene su propio i)
    }, 1000);
}

// ✅ Solución con closure (var)
for (var i = 0; i < 3; i++) {
    (function(j) {
        setTimeout(function() {
            console.log(j); // 0, 1, 2
        }, 1000);
    })(i);
}
```

---

## Ejercicios prácticos

### Ejercicio 1: Entender scope
**Nivel**: ⭐☆☆☆☆

¿Qué mostrará este código? Explica por qué.

```javascript
const x = 10;

function prueba() {
    const x = 20;
    console.log(x);
}

prueba();
console.log(x);
```

<details>
<summary>Ver solución</summary>

```javascript
const x = 10;

function prueba() {
    const x = 20;
    console.log(x); // 20 (usa la x local)
}

prueba();           // Muestra: 20
console.log(x);     // Muestra: 10

// Explicación:
// - La función prueba() tiene su propia variable x (shadowing)
// - Dentro de prueba(), x vale 20 (local)
// - Fuera de prueba(), x vale 10 (global)
// - La x local no afecta a la x global
```

</details>

---

### Ejercicio 2: Block scope
**Nivel**: ⭐⭐☆☆☆

¿Qué mostrará este código? ¿Por qué?

```javascript
if (true) {
    let mensaje = "Hola";
    var numero = 42;
}

console.log(mensaje);
console.log(numero);
```

<details>
<summary>Ver solución</summary>

```javascript
if (true) {
    let mensaje = "Hola";
    var numero = 42;
}

console.log(mensaje); // ❌ ReferenceError: mensaje is not defined
console.log(numero);  // ✅ 42

// Explicación:
// - 'let' respeta block scope: mensaje solo existe dentro del if
// - 'var' ignora block scope: numero es accesible fuera del if
// - Por eso let/const son preferibles (comportamiento más predecible)
```

</details>

---

### Ejercicio 3: Hoisting básico
**Nivel**: ⭐⭐☆☆☆

¿Qué mostrará este código?

```javascript
console.log(a);
var a = 5;
console.log(a);

console.log(b);
let b = 10;
```

<details>
<summary>Ver solución</summary>

```javascript
console.log(a); // undefined
var a = 5;
console.log(a); // 5

console.log(b); // ❌ ReferenceError: Cannot access 'b' before initialization
let b = 10;

// Explicación:
// var hace hoisting: la declaración se mueve arriba (pero no el valor)
// let también hace hoisting, pero tiene Temporal Dead Zone
// No se puede usar 'b' antes de su declaración
```

</details>

---

### Ejercicio 4: Crear closure simple
**Nivel**: ⭐⭐⭐☆☆

Crea una función `crearSaludo(nombre)` que retorne una función que salude a esa persona.

<details>
<summary>Ver solución</summary>

```javascript
function crearSaludo(nombre) {
    return function() {
        console.log(`¡Hola, ${nombre}!`);
    };
}

const saludarAna = crearSaludo("Ana");
const saludarJuan = crearSaludo("Juan");

saludarAna();  // "¡Hola, Ana!"
saludarJuan(); // "¡Hola, Juan!"
saludarAna();  // "¡Hola, Ana!" (recuerda "Ana")

// Explicación:
// - crearSaludo crea un closure
// - La función retornada "recuerda" el nombre
// - Cada closure tiene su propio nombre
```

</details>

---

### Ejercicio 5: Contador privado
**Nivel**: ⭐⭐⭐☆☆

Crea un contador con métodos para incrementar, decrementar y obtener el valor. El contador debe ser privado (no accesible desde fuera).

<details>
<summary>Ver solución</summary>

```javascript
function crearContador(inicial = 0) {
    let contador = inicial; // Variable privada
    
    return {
        incrementar() {
            contador++;
            return contador;
        },
        
        decrementar() {
            contador--;
            return contador;
        },
        
        obtener() {
            return contador;
        },
        
        resetear() {
            contador = inicial;
            return contador;
        }
    };
}

const contador = crearContador(10);

console.log(contador.obtener());     // 10
console.log(contador.incrementar()); // 11
console.log(contador.incrementar()); // 12
console.log(contador.decrementar()); // 11
console.log(contador.resetear());    // 10

// Intento de acceso directo (no funciona)
console.log(contador.contador); // undefined (privado)
```

</details>

---

### Ejercicio 6: Multiplicador con closure
**Nivel**: ⭐⭐⭐☆☆

Crea una función `crearMultiplicador(factor)` que retorne una función que multiplique números por ese factor.

<details>
<summary>Ver solución</summary>

```javascript
function crearMultiplicador(factor) {
    return function(numero) {
        return numero * factor;
    };
}

const duplicar = crearMultiplicador(2);
const triplicar = crearMultiplicador(3);
const porCinco = crearMultiplicador(5);

console.log(duplicar(10));   // 20
console.log(duplicar(7));    // 14
console.log(triplicar(10));  // 30
console.log(triplicar(4));   // 12
console.log(porCinco(6));    // 30

// Explicación:
// - Cada función retornada recuerda su factor
// - duplicar siempre multiplica por 2
// - triplicar siempre multiplica por 3
// - Es un closure: accede a 'factor' de su scope exterior
```

</details>

---

### Ejercicio 7: Acumulador
**Nivel**: ⭐⭐⭐⭐☆

Crea una función `crearAcumulador()` que retorne una función. Cada vez que se llame, suma el número al acumulado y lo retorna.

<details>
<summary>Ver solución</summary>

```javascript
function crearAcumulador(inicial = 0) {
    let total = inicial;
    
    return function(numero) {
        total += numero;
        return total;
    };
}

const acum = crearAcumulador();

console.log(acum(5));   // 5
console.log(acum(10));  // 15
console.log(acum(3));   // 18
console.log(acum(-8));  // 10

const acum2 = crearAcumulador(100);
console.log(acum2(50)); // 150
console.log(acum2(25)); // 175

// Cada acumulador es independiente
console.log(acum(1));   // 11 (continúa desde su total)
console.log(acum2(1));  // 176 (continúa desde su total)
```

</details>

---

### Ejercicio 8: Sistema de caché
**Nivel**: ⭐⭐⭐⭐⭐

Crea una función que almacene en caché los resultados de una operación costosa.

<details>
<summary>Ver solución</summary>

```javascript
function crearCache(funcion) {
    const cache = {};
    
    return function(arg) {
        // Si el resultado ya está en caché, devolverlo
        if (arg in cache) {
            console.log(`📦 Desde caché: ${arg}`);
            return cache[arg];
        }
        
        // Si no, calcular y guardar en caché
        console.log(`⚙️ Calculando: ${arg}`);
        const resultado = funcion(arg);
        cache[arg] = resultado;
        return resultado;
    };
}

// Función costosa de ejemplo
function operacionCostosa(n) {
    let suma = 0;
    for (let i = 0; i < 1000000; i++) {
        suma += n;
    }
    return suma;
}

const conCache = crearCache(operacionCostosa);

console.log(conCache(5));  // ⚙️ Calculando: 5 → 5000000
console.log(conCache(5));  // 📦 Desde caché: 5 → 5000000
console.log(conCache(10)); // ⚙️ Calculando: 10 → 10000000
console.log(conCache(5));  // 📦 Desde caché: 5 → 5000000
console.log(conCache(10)); // 📦 Desde caché: 10 → 10000000

// Explicación:
// - El closure 'cache' persiste entre llamadas
// - La primera vez calcula y guarda el resultado
// - Las siguientes veces retorna el valor guardado
// - Optimización: evita recalcular valores repetidos
```

</details>

---

## Errores comunes

### ❌ Error 1: Confundir var, let y const

```javascript
// ❌ var ignora block scope
if (true) {
    var x = 10;
}
console.log(x); // 10 (accesible, puede causar bugs)

// ✅ let respeta block scope
if (true) {
    let y = 10;
}
console.log(y); // Error (no accesible fuera del bloque)
```

---

### ❌ Error 2: Usar variables antes de declararlas

```javascript
// ❌ Con let/const
console.log(nombre); // ReferenceError
let nombre = "Ana";

// ✅ Declarar antes de usar
let nombre = "Ana";
console.log(nombre); // "Ana"
```

---

### ❌ Error 3: Olvidar que closures capturan variables, no valores

```javascript
// ❌ Problema
const funciones = [];
for (var i = 0; i < 3; i++) {
    funciones.push(function() {
        console.log(i);
    });
}

funciones[0](); // 3 (esperábamos 0)
funciones[1](); // 3 (esperábamos 1)
funciones[2](); // 3 (esperábamos 2)

// ✅ Solución con let
const funciones = [];
for (let i = 0; i < 3; i++) {
    funciones.push(function() {
        console.log(i);
    });
}

funciones[0](); // 0
funciones[1](); // 1
funciones[2](); // 2
```

---

### ❌ Error 4: Shadowing accidental

```javascript
const usuario = "Ana";

function cambiarUsuario() {
    const usuario = "Juan"; // Sombrea la global (no la modifica)
    console.log(usuario);   // "Juan"
}

cambiarUsuario();
console.log(usuario); // "Ana" (no cambió)

// Si querías modificar la global, no uses const dentro
```

---

## Buenas prácticas

### ✅ 1. Usa let y const, evita var

```javascript
// ❌ var (comportamiento impredecible)
var x = 10;

// ✅ let (si necesitas reasignar)
let contador = 0;

// ✅ const (si no necesitas reasignar)
const PI = 3.14159;
```

---

### ✅ 2. Declara variables al inicio del scope

```javascript
// ✅ Claro y organizado
function calcular() {
    const a = 10;
    const b = 20;
    let resultado;
    
    resultado = a + b;
    return resultado;
}
```

---

### ✅ 3. Minimiza variables globales

```javascript
// ❌ Muchas globales
let nombre = "Ana";
let edad = 25;
let ciudad = "Madrid";

// ✅ Agrupa en un objeto
const usuario = {
    nombre: "Ana",
    edad: 25,
    ciudad: "Madrid"
};
```

---

### ✅ 4. Usa closures para privacidad

```javascript
// ✅ Variables privadas con closure
function crearPersona(nombre, edad) {
    // Variables privadas
    let _nombre = nombre;
    let _edad = edad;
    
    return {
        getNombre() {
            return _nombre;
        },
        getEdad() {
            return _edad;
        },
        cumplirAnios() {
            _edad++;
        }
    };
}

const persona = crearPersona("Ana", 25);
console.log(persona.getNombre()); // "Ana"
console.log(persona._nombre);     // undefined (privado)
```

---

### ✅ 5. Entiende el contexto de this en closures

```javascript
const obj = {
    nombre: "Ana",
    
    // ✅ Arrow function en closure mantiene this
    metodo() {
        setTimeout(() => {
            console.log(this.nombre); // "Ana"
        }, 1000);
    },
    
    // ❌ Function tradicional pierde this
    metodo2() {
        setTimeout(function() {
            console.log(this.nombre); // undefined
        }, 1000);
    }
};
```

---

## Cheatsheet

### Tipos de scope

```javascript
const global = "GLOBAL";         // Global scope

function foo() {
    const funcion = "FUNCIÓN";   // Function scope
    
    if (true) {
        const bloque = "BLOQUE"; // Block scope (let/const)
        var noBloque = "VAR";    // Function scope (ignora bloque)
    }
}
```

### Hoisting

```javascript
// var hace hoisting (declaración, no valor)
console.log(x); // undefined
var x = 5;

// let/const tienen TDZ
console.log(y); // ReferenceError
let y = 10;

// Function declarations
foo(); // ✅ Funciona (hoisting completo)
function foo() { }

// Function expressions
bar(); // ❌ Error
const bar = function() { };
```

### Closures

```javascript
function outer() {
    const variable = "Recordada";
    
    return function inner() {
        console.log(variable); // Accede a variable (closure)
    };
}

const fn = outer();
fn(); // "Recordada"
```

### Scope chain

```javascript
const a = 1;                // Global

function nivel1() {
    const b = 2;            // Nivel 1
    
    function nivel2() {
        const c = 3;        // Nivel 2
        console.log(a, b, c); // Accede a todos
    }
}
```

---

## ¡Bloque 3 completado! 🎉

**¡Felicidades!** Has completado el Bloque 3: JavaScript Básico.

Ahora dominas:
- ✅ Variables y tipos de datos
- ✅ Operadores
- ✅ Condicionales
- ✅ Bucles
- ✅ Funciones
- ✅ Arrays
- ✅ Objetos
- ✅ Scope y Hoisting

### Próximos pasos

Ya tienes las bases de JavaScript. El siguiente bloque te enseñará a interactuar con páginas web:

→ **Bloque 4: DOM (Document Object Model)**

Ahí aprenderás a:
- Seleccionar elementos HTML
- Modificar contenido y estilos
- Responder a eventos (clicks, inputs, etc.)
- Crear elementos dinámicamente
- Validar formularios

¡Sigue adelante! 🚀

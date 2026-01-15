# Módulo 17: Bucles en JavaScript

## Índice
1. [¿Qué son los bucles?](#qué-son-los-bucles)
2. [for - El bucle más común](#for---el-bucle-más-común)
3. [while - Mientras se cumpla](#while---mientras-se-cumpla)
4. [do-while - Al menos una vez](#do-while---al-menos-una-vez)
5. [break - Salir del bucle](#break---salir-del-bucle)
6. [continue - Saltar iteración](#continue---saltar-iteración)
7. [Bucles anidados](#bucles-anidados)
8. [Ejercicios prácticos](#ejercicios-prácticos)
9. [Errores comunes](#errores-comunes)
10. [Buenas prácticas](#buenas-prácticas)
11. [Cheatsheet](#cheatsheet)

---

## ¿Qué son los bucles?

**Un bucle repite un bloque de código múltiples veces**.

### Analogía: Lavar platos

Sin bucle (repetitivo):
```javascript
console.log("Lavar plato 1");
console.log("Lavar plato 2");
console.log("Lavar plato 3");
console.log("Lavar plato 4");
console.log("Lavar plato 5");
// ... ¿Y si son 100 platos?
```

Con bucle (eficiente):
```javascript
for (let i = 1; i <= 5; i++) {
    console.log(`Lavar plato ${i}`);
}
```

### ¿Para qué sirven?

- Procesar listas de datos
- Repetir acciones N veces
- Recorrer elementos HTML
- Generar contenido dinámico
- Hacer cálculos repetitivos
- Y mucho más...

---

## for - El bucle más común

**El bucle `for` repite código un número específico de veces**.

### Sintaxis

```javascript
for (inicialización; condición; actualización) {
    // Código que se repite
}
```

### Anatomía del for

```javascript
for (let i = 0; i < 5; i++) {
    console.log(i);
}

/*
1. let i = 0       → Inicialización: i empieza en 0
2. i < 5           → Condición: repetir mientras i < 5
3. i++             → Actualización: incrementar i en cada vuelta
4. console.log(i)  → Código a ejecutar
*/
```

### Ejemplo paso a paso

```javascript
for (let i = 0; i < 3; i++) {
    console.log(`Iteración ${i}`);
}

// Paso 1: i = 0, ¿0 < 3? Sí → Ejecuta → Muestra "Iteración 0" → i++
// Paso 2: i = 1, ¿1 < 3? Sí → Ejecuta → Muestra "Iteración 1" → i++
// Paso 3: i = 2, ¿2 < 3? Sí → Ejecuta → Muestra "Iteración 2" → i++
// Paso 4: i = 3, ¿3 < 3? No → Termina

// Output:
// Iteración 0
// Iteración 1
// Iteración 2
```

### Contar del 1 al 10

```javascript
for (let i = 1; i <= 10; i++) {
    console.log(i);
}
// Output: 1, 2, 3, 4, 5, 6, 7, 8, 9, 10
```

### Contar hacia atrás

```javascript
for (let i = 10; i >= 1; i--) {
    console.log(i);
}
console.log("¡Despegue! 🚀");

// Output:
// 10
// 9
// 8
// ...
// 1
// ¡Despegue! 🚀
```

### Incrementos diferentes

```javascript
// De 2 en 2
for (let i = 0; i <= 10; i += 2) {
    console.log(i);
}
// Output: 0, 2, 4, 6, 8, 10

// De 5 en 5
for (let i = 0; i <= 50; i += 5) {
    console.log(i);
}
// Output: 0, 5, 10, 15, 20, 25, 30, 35, 40, 45, 50
```

### Recorrer un array

```javascript
const frutas = ["manzana", "banana", "naranja", "uva"];

for (let i = 0; i < frutas.length; i++) {
    console.log(`Fruta ${i + 1}: ${frutas[i]}`);
}

// Output:
// Fruta 1: manzana
// Fruta 2: banana
// Fruta 3: naranja
// Fruta 4: uva
```

### Acumular valores

```javascript
// Suma de números del 1 al 100
let suma = 0;

for (let i = 1; i <= 100; i++) {
    suma += i;
}

console.log(`La suma es: ${suma}`);
// Output: "La suma es: 5050"
```

### Construir strings

```javascript
let resultado = "";

for (let i = 1; i <= 5; i++) {
    resultado += "★";
}

console.log(resultado);
// Output: "★★★★★"
```

---

## while - Mientras se cumpla

**El bucle `while` repite mientras una condición sea verdadera**.

### Sintaxis

```javascript
while (condición) {
    // Código que se repite
}
```

### Diferencia con `for`

- **`for`**: Usas cuando sabes cuántas veces repetir
- **`while`**: Usas cuando no sabes cuántas veces repetir

### Ejemplo básico

```javascript
let contador = 0;

while (contador < 5) {
    console.log(`Contador: ${contador}`);
    contador++;
}

// Output:
// Contador: 0
// Contador: 1
// Contador: 2
// Contador: 3
// Contador: 4
```

### Ejemplo: Pedir contraseña

```javascript
let password = "";
let intentos = 0;

while (password !== "1234" && intentos < 3) {
    console.log(`Intento ${intentos + 1}: Ingresa la contraseña`);
    // En un escenario real, aquí pedirías input del usuario
    password = "incorrecta"; // Simulación
    intentos++;
}

if (password === "1234") {
    console.log("✅ Acceso concedido");
} else {
    console.log("❌ Máximo de intentos alcanzado");
}
```

### Ejemplo: Duplicar hasta límite

```javascript
let numero = 1;

while (numero <= 100) {
    console.log(numero);
    numero *= 2; // Duplicar
}

// Output: 1, 2, 4, 8, 16, 32, 64
```

### Ejemplo: Procesar hasta vacío

```javascript
let tareas = ["Comprar", "Estudiar", "Cocinar"];

while (tareas.length > 0) {
    const tarea = tareas.pop(); // Quita el último elemento
    console.log(`Haciendo: ${tarea}`);
}

console.log("✅ Todas las tareas completadas");

// Output:
// Haciendo: Cocinar
// Haciendo: Estudiar
// Haciendo: Comprar
// ✅ Todas las tareas completadas
```

### ⚠️ Cuidado: Bucle infinito

```javascript
// ❌ MAL: Bucle infinito
let i = 0;
while (i < 5) {
    console.log(i);
    // Olvidé incrementar i
}
// Esto se repite infinitamente: 0, 0, 0, 0, ...

// ✅ BIEN: Incrementa la variable
let i = 0;
while (i < 5) {
    console.log(i);
    i++; // ← No olvides esto
}
```

---

## do-while - Al menos una vez

**`do-while` ejecuta el código AL MENOS UNA VEZ, luego verifica la condición**.

### Sintaxis

```javascript
do {
    // Código que se ejecuta al menos una vez
} while (condición);
```

### Diferencia con `while`

```javascript
// while: Puede no ejecutarse nunca
let x = 10;
while (x < 5) {
    console.log("Esto nunca se muestra");
}
// Output: (nada)

// do-while: Se ejecuta al menos una vez
let y = 10;
do {
    console.log("Esto se muestra una vez");
} while (y < 5);
// Output: "Esto se muestra una vez"
```

### Ejemplo: Menú de opciones

```javascript
let opcion;

do {
    console.log("\n=== MENÚ ===");
    console.log("1. Nueva partida");
    console.log("2. Cargar partida");
    console.log("3. Opciones");
    console.log("4. Salir");
    
    opcion = 2; // Simulación de input del usuario
    
    switch (opcion) {
        case 1:
            console.log("Iniciando nueva partida...");
            break;
        case 2:
            console.log("Cargando partida...");
            break;
        case 3:
            console.log("Abriendo opciones...");
            break;
        case 4:
            console.log("Saliendo del juego...");
            break;
        default:
            console.log("Opción no válida");
    }
} while (opcion !== 4);
```

### Ejemplo: Validar input

```javascript
let edad;

do {
    console.log("Ingresa tu edad (debe ser mayor de 0)");
    edad = -5; // Simulación de input
    
    if (edad <= 0) {
        console.log("❌ Edad no válida. Intenta de nuevo.");
    }
} while (edad <= 0);

console.log(`✅ Edad registrada: ${edad}`);
```

---

## break - Salir del bucle

**`break` termina el bucle inmediatamente**.

### En for

```javascript
for (let i = 1; i <= 10; i++) {
    if (i === 5) {
        console.log("Encontré el 5, deteniendo...");
        break; // Sale del bucle
    }
    console.log(i);
}

// Output:
// 1
// 2
// 3
// 4
// Encontré el 5, deteniendo...
```

### En while

```javascript
let intentos = 0;

while (true) { // Bucle infinito controlado
    intentos++;
    console.log(`Intento ${intentos}`);
    
    if (intentos === 3) {
        console.log("Límite alcanzado");
        break; // Sale del bucle
    }
}

// Output:
// Intento 1
// Intento 2
// Intento 3
// Límite alcanzado
```

### Ejemplo: Buscar en array

```javascript
const numeros = [3, 7, 2, 9, 15, 4, 8];
const buscar = 9;
let encontrado = false;

for (let i = 0; i < numeros.length; i++) {
    console.log(`Revisando: ${numeros[i]}`);
    
    if (numeros[i] === buscar) {
        console.log(`✅ Encontrado en posición ${i}`);
        encontrado = true;
        break; // Ya lo encontramos, no seguir buscando
    }
}

if (!encontrado) {
    console.log("❌ No encontrado");
}

// Output:
// Revisando: 3
// Revisando: 7
// Revisando: 2
// Revisando: 9
// ✅ Encontrado en posición 3
```

---

## continue - Saltar iteración

**`continue` salta a la siguiente iteración del bucle**.

### Ejemplo básico

```javascript
for (let i = 1; i <= 5; i++) {
    if (i === 3) {
        continue; // Salta cuando i = 3
    }
    console.log(i);
}

// Output:
// 1
// 2
// (se salta el 3)
// 4
// 5
```

### Ejemplo: Saltar números pares

```javascript
for (let i = 1; i <= 10; i++) {
    if (i % 2 === 0) {
        continue; // Salta los pares
    }
    console.log(i);
}

// Output: 1, 3, 5, 7, 9 (solo impares)
```

### Ejemplo: Procesar solo válidos

```javascript
const edades = [25, -5, 18, 0, 30, -10, 22];

for (let i = 0; i < edades.length; i++) {
    if (edades[i] <= 0) {
        console.log(`Saltando edad no válida: ${edades[i]}`);
        continue; // Salta las edades no válidas
    }
    
    console.log(`Edad válida: ${edades[i]}`);
}

// Output:
// Edad válida: 25
// Saltando edad no válida: -5
// Edad válida: 18
// Saltando edad no válida: 0
// Edad válida: 30
// Saltando edad no válida: -10
// Edad válida: 22
```

### break vs continue

```javascript
// break: Termina el bucle completamente
for (let i = 1; i <= 5; i++) {
    if (i === 3) break;
    console.log(i);
}
// Output: 1, 2

// continue: Salta solo esa iteración
for (let i = 1; i <= 5; i++) {
    if (i === 3) continue;
    console.log(i);
}
// Output: 1, 2, 4, 5
```

---

## Bucles anidados

**Un bucle dentro de otro bucle**.

### Ejemplo: Tabla de multiplicar

```javascript
for (let i = 1; i <= 3; i++) {
    console.log(`\nTabla del ${i}:`);
    
    for (let j = 1; j <= 5; j++) {
        console.log(`${i} × ${j} = ${i * j}`);
    }
}

// Output:
// Tabla del 1:
// 1 × 1 = 1
// 1 × 2 = 2
// 1 × 3 = 3
// 1 × 4 = 4
// 1 × 5 = 5
//
// Tabla del 2:
// 2 × 1 = 2
// 2 × 2 = 4
// ...
```

### Ejemplo: Matriz (filas y columnas)

```javascript
const filas = 3;
const columnas = 4;

for (let fila = 1; fila <= filas; fila++) {
    let linea = "";
    
    for (let col = 1; col <= columnas; col++) {
        linea += "⬜ ";
    }
    
    console.log(linea);
}

// Output:
// ⬜ ⬜ ⬜ ⬜ 
// ⬜ ⬜ ⬜ ⬜ 
// ⬜ ⬜ ⬜ ⬜ 
```

### Ejemplo: Pirámide de asteriscos

```javascript
const niveles = 5;

for (let i = 1; i <= niveles; i++) {
    let linea = "";
    
    // Espacios
    for (let j = 1; j <= niveles - i; j++) {
        linea += " ";
    }
    
    // Asteriscos
    for (let k = 1; k <= i * 2 - 1; k++) {
        linea += "*";
    }
    
    console.log(linea);
}

// Output:
//     *
//    ***
//   *****
//  *******
// *********
```

### Ejemplo: Comparar arrays

```javascript
const lista1 = [1, 2, 3];
const lista2 = [2, 3, 4];
const comunes = [];

for (let i = 0; i < lista1.length; i++) {
    for (let j = 0; j < lista2.length; j++) {
        if (lista1[i] === lista2[j]) {
            comunes.push(lista1[i]);
        }
    }
}

console.log("Elementos comunes:", comunes);
// Output: "Elementos comunes: [2, 3]"
```

---

## Ejercicios prácticos

### Ejercicio 1: Contar del 1 al 20
**Nivel**: ⭐☆☆☆☆

Usa un bucle `for` para imprimir los números del 1 al 20.

<details>
<summary>Ver solución</summary>

```javascript
for (let i = 1; i <= 20; i++) {
    console.log(i);
}
```

</details>

---

### Ejercicio 2: Suma de números
**Nivel**: ⭐⭐☆☆☆

Calcula la suma de todos los números del 1 al 50.

<details>
<summary>Ver solución</summary>

```javascript
let suma = 0;

for (let i = 1; i <= 50; i++) {
    suma += i;
}

console.log(`La suma del 1 al 50 es: ${suma}`);
// Output: "La suma del 1 al 50 es: 1275"
```

</details>

---

### Ejercicio 3: Solo números pares
**Nivel**: ⭐⭐☆☆☆

Imprime solo los números pares del 1 al 30.

<details>
<summary>Ver solución</summary>

```javascript
// Solución 1: Verificar si es par
for (let i = 1; i <= 30; i++) {
    if (i % 2 === 0) {
        console.log(i);
    }
}

// Solución 2: Incrementar de 2 en 2
for (let i = 2; i <= 30; i += 2) {
    console.log(i);
}

// Output: 2, 4, 6, 8, 10, ..., 30
```

</details>

---

### Ejercicio 4: Cuenta regresiva
**Nivel**: ⭐⭐☆☆☆

Crea una cuenta regresiva desde 10 hasta 1, y al final muestra "¡Despegue! 🚀".

<details>
<summary>Ver solución</summary>

```javascript
for (let i = 10; i >= 1; i--) {
    console.log(i);
}
console.log("¡Despegue! 🚀");

// Output:
// 10
// 9
// 8
// ...
// 1
// ¡Despegue! 🚀
```

</details>

---

### Ejercicio 5: Factorial
**Nivel**: ⭐⭐⭐☆☆

Calcula el factorial de un número. El factorial de 5 es: 5 × 4 × 3 × 2 × 1 = 120.

<details>
<summary>Ver solución</summary>

```javascript
const numero = 5;
let factorial = 1;

for (let i = numero; i >= 1; i--) {
    factorial *= i;
}

console.log(`El factorial de ${numero} es: ${factorial}`);
// Output: "El factorial de 5 es: 120"

// Verificación: 5 × 4 × 3 × 2 × 1 = 120
```

</details>

---

### Ejercicio 6: FizzBuzz
**Nivel**: ⭐⭐⭐☆☆

Clásico ejercicio de programación:
- Imprime números del 1 al 30
- Si es múltiplo de 3: imprime "Fizz"
- Si es múltiplo de 5: imprime "Buzz"
- Si es múltiplo de 3 Y 5: imprime "FizzBuzz"
- Si no, imprime el número

<details>
<summary>Ver solución</summary>

```javascript
for (let i = 1; i <= 30; i++) {
    if (i % 3 === 0 && i % 5 === 0) {
        console.log("FizzBuzz");
    } else if (i % 3 === 0) {
        console.log("Fizz");
    } else if (i % 5 === 0) {
        console.log("Buzz");
    } else {
        console.log(i);
    }
}

// Output:
// 1
// 2
// Fizz
// 4
// Buzz
// Fizz
// 7
// 8
// Fizz
// Buzz
// 11
// Fizz
// 13
// 14
// FizzBuzz
// ...
```

</details>

---

### Ejercicio 7: Tabla de multiplicar completa
**Nivel**: ⭐⭐⭐☆☆

Genera las tablas de multiplicar del 1 al 10.

<details>
<summary>Ver solución</summary>

```javascript
for (let tabla = 1; tabla <= 10; tabla++) {
    console.log(`\n=== Tabla del ${tabla} ===`);
    
    for (let num = 1; num <= 10; num++) {
        const resultado = tabla * num;
        console.log(`${tabla} × ${num} = ${resultado}`);
    }
}

// Output:
// === Tabla del 1 ===
// 1 × 1 = 1
// 1 × 2 = 2
// ...
// === Tabla del 2 ===
// 2 × 1 = 2
// 2 × 2 = 4
// ...
```

</details>

---

### Ejercicio 8: Números primos
**Nivel**: ⭐⭐⭐⭐☆

Encuentra todos los números primos del 2 al 50. Un número primo solo es divisible por 1 y por sí mismo.

<details>
<summary>Ver solución</summary>

```javascript
console.log("Números primos del 2 al 50:");

for (let num = 2; num <= 50; num++) {
    let esPrimo = true;
    
    // Verificar si tiene divisores
    for (let divisor = 2; divisor < num; divisor++) {
        if (num % divisor === 0) {
            esPrimo = false;
            break; // Ya no es primo, salir
        }
    }
    
    if (esPrimo) {
        console.log(num);
    }
}

// Output: 2, 3, 5, 7, 11, 13, 17, 19, 23, 29, 31, 37, 41, 43, 47
```

</details>

---

### Ejercicio 9: Patrón de estrellas
**Nivel**: ⭐⭐⭐⭐☆

Crea este patrón:
```
*
**
***
****
*****
```

<details>
<summary>Ver solución</summary>

```javascript
const filas = 5;

for (let i = 1; i <= filas; i++) {
    let linea = "";
    
    for (let j = 1; j <= i; j++) {
        linea += "*";
    }
    
    console.log(linea);
}

// Output:
// *
// **
// ***
// ****
// *****
```

</details>

---

### Ejercicio 10: Validar contraseña con intentos
**Nivel**: ⭐⭐⭐⭐☆

Simula un sistema de login con máximo 3 intentos. La contraseña correcta es "secreto123".

<details>
<summary>Ver solución</summary>

```javascript
const passwordCorrecta = "secreto123";
const maxIntentos = 3;
let intentos = 0;
let accesoConcedido = false;

// Contraseñas simuladas del usuario
const passwordsIntentadas = ["123456", "password", "secreto123"];

while (intentos < maxIntentos && !accesoConcedido) {
    const passwordIngresada = passwordsIntentadas[intentos];
    intentos++;
    
    console.log(`Intento ${intentos}/${maxIntentos}: ${passwordIngresada}`);
    
    if (passwordIngresada === passwordCorrecta) {
        accesoConcedido = true;
        console.log("✅ Acceso concedido");
    } else {
        const intentosRestantes = maxIntentos - intentos;
        
        if (intentosRestantes > 0) {
            console.log(`❌ Contraseña incorrecta. Te quedan ${intentosRestantes} intento(s)`);
        }
    }
}

if (!accesoConcedido) {
    console.log("🔒 Cuenta bloqueada por múltiples intentos fallidos");
}

// Output:
// Intento 1/3: 123456
// ❌ Contraseña incorrecta. Te quedan 2 intento(s)
// Intento 2/3: password
// ❌ Contraseña incorrecta. Te quedan 1 intento(s)
// Intento 3/3: secreto123
// ✅ Acceso concedido
```

</details>

---

## Errores comunes

### ❌ Error 1: Bucle infinito

```javascript
// MAL: Nunca termina
for (let i = 0; i < 10; i--) {
    console.log(i); // i va hacia atrás, nunca llega a 10
}

// MAL: Falta incremento
let i = 0;
while (i < 5) {
    console.log(i); // Falta i++
}

// BIEN: Condición correcta
for (let i = 0; i < 10; i++) {
    console.log(i);
}
```

---

### ❌ Error 2: Off-by-one (error por uno)

```javascript
// MAL: Se salta el último elemento
const frutas = ["manzana", "banana", "naranja"];

for (let i = 0; i < frutas.length - 1; i++) {
    console.log(frutas[i]);
}
// Output: manzana, banana (falta naranja)

// BIEN: Recorre todos los elementos
for (let i = 0; i < frutas.length; i++) {
    console.log(frutas[i]);
}
// Output: manzana, banana, naranja
```

---

### ❌ Error 3: Modificar el contador dentro del bucle

```javascript
// MAL: Comportamiento inesperado
for (let i = 0; i < 10; i++) {
    console.log(i);
    i += 2; // Modificación adicional
}
// Salta valores: 0, 3, 6, 9

// BIEN: Incremento solo en la declaración
for (let i = 0; i < 10; i += 3) {
    console.log(i);
}
// Claro: 0, 3, 6, 9
```

---

### ❌ Error 4: Variable del bucle en el scope incorrecto

```javascript
// MAL: i se filtra al scope global
for (var i = 0; i < 5; i++) {
    // ...
}
console.log(i); // 5 (¡i todavía existe!)

// BIEN: i solo existe dentro del bucle
for (let i = 0; i < 5; i++) {
    // ...
}
console.log(i); // Error: i is not defined
```

---

### ❌ Error 5: Usar índice en bucle vacío

```javascript
const lista = [];

for (let i = 0; i < lista.length; i++) {
    console.log(lista[i].nombre); // Error si lista está vacía
}

// BIEN: Verifica antes
if (lista.length > 0) {
    for (let i = 0; i < lista.length; i++) {
        console.log(lista[i].nombre);
    }
} else {
    console.log("La lista está vacía");
}
```

---

### ❌ Error 6: Olvidar break en búsqueda

```javascript
const numeros = [1, 2, 3, 4, 5];
const buscar = 3;

// MAL: Sigue buscando después de encontrar
for (let i = 0; i < numeros.length; i++) {
    if (numeros[i] === buscar) {
        console.log("Encontrado");
        // Falta break, sigue iterando innecesariamente
    }
}

// BIEN: Sale cuando encuentra
for (let i = 0; i < numeros.length; i++) {
    if (numeros[i] === buscar) {
        console.log("Encontrado");
        break; // Sale del bucle
    }
}
```

---

## Buenas prácticas

### ✅ 1. Usa nombres descriptivos para contadores

```javascript
// ❌ No muy claro
for (let i = 0; i < usuarios.length; i++) {
    for (let j = 0; j < pedidos.length; j++) {
        // ¿Qué es i? ¿Qué es j?
    }
}

// ✅ Más claro
for (let indexUsuario = 0; indexUsuario < usuarios.length; indexUsuario++) {
    for (let indexPedido = 0; indexPedido < pedidos.length; indexPedido++) {
        // Ahora es obvio
    }
}

// ✅ O usa for...of (lo veremos en módulos futuros)
for (const usuario of usuarios) {
    for (const pedido of pedidos) {
        // Aún más claro
    }
}
```

---

### ✅ 2. Extrae la longitud en bucles largos

```javascript
// ❌ Calcula length en cada iteración
for (let i = 0; i < array.length; i++) {
    // ...
}

// ✅ Calcula una sola vez
const longitud = array.length;
for (let i = 0; i < longitud; i++) {
    // ...
}
```

---

### ✅ 3. Evita modificar el array mientras lo recorres

```javascript
// ❌ Puede causar problemas
const numeros = [1, 2, 3, 4, 5];

for (let i = 0; i < numeros.length; i++) {
    numeros.splice(i, 1); // Elimina elementos mientras itera
}

// ✅ Recorre una copia o hacia atrás
for (let i = numeros.length - 1; i >= 0; i--) {
    numeros.splice(i, 1); // Ahora es seguro
}
```

---

### ✅ 4. Usa el bucle apropiado

```javascript
// Sabes cuántas veces → for
for (let i = 0; i < 10; i++) {
    // ...
}

// No sabes cuántas veces → while
while (condicionDinamica) {
    // ...
}

// Al menos una vez → do-while
do {
    // ...
} while (condicion);
```

---

### ✅ 5. Limita la profundidad de bucles anidados

```javascript
// ❌ Demasiados niveles (difícil de leer)
for (let i = 0; i < n; i++) {
    for (let j = 0; j < m; j++) {
        for (let k = 0; k < p; k++) {
            for (let l = 0; l < q; l++) {
                // ...
            }
        }
    }
}

// ✅ Extrae a funciones
for (let i = 0; i < n; i++) {
    procesarItem(i);
}

function procesarItem(i) {
    for (let j = 0; j < m; j++) {
        // ...
    }
}
```

---

### ✅ 6. Usa break para optimizar búsquedas

```javascript
// ❌ Continúa buscando después de encontrar
let encontrado = false;
for (let i = 0; i < array.length; i++) {
    if (array[i] === buscar) {
        encontrado = true;
    }
}

// ✅ Sale inmediatamente
let encontrado = false;
for (let i = 0; i < array.length; i++) {
    if (array[i] === buscar) {
        encontrado = true;
        break; // Ya no hace falta seguir
    }
}
```

---

## Cheatsheet

### for

```javascript
// Sintaxis básica
for (let i = 0; i < 10; i++) {
    console.log(i);
}

// Contar hacia atrás
for (let i = 10; i >= 0; i--) {
    console.log(i);
}

// Incrementos personalizados
for (let i = 0; i <= 100; i += 10) {
    console.log(i);
}
```

### while

```javascript
// Sintaxis básica
let i = 0;
while (i < 5) {
    console.log(i);
    i++;
}

// Bucle con condición dinámica
while (condicionCambiante) {
    // ...
}
```

### do-while

```javascript
// Se ejecuta al menos una vez
let i = 0;
do {
    console.log(i);
    i++;
} while (i < 5);
```

### break

```javascript
// Sale del bucle completamente
for (let i = 0; i < 10; i++) {
    if (i === 5) break;
    console.log(i);
}
// Output: 0, 1, 2, 3, 4
```

### continue

```javascript
// Salta a la siguiente iteración
for (let i = 0; i < 5; i++) {
    if (i === 2) continue;
    console.log(i);
}
// Output: 0, 1, 3, 4
```

### Bucles anidados

```javascript
for (let i = 0; i < 3; i++) {
    for (let j = 0; j < 3; j++) {
        console.log(`i=${i}, j=${j}`);
    }
}
```

### Patrones comunes

```javascript
// Recorrer array
for (let i = 0; i < array.length; i++) {
    console.log(array[i]);
}

// Acumular valores
let suma = 0;
for (let i = 1; i <= 10; i++) {
    suma += i;
}

// Construir string
let resultado = "";
for (let i = 0; i < 5; i++) {
    resultado += "*";
}

// Buscar en array
for (let i = 0; i < array.length; i++) {
    if (array[i] === buscar) {
        console.log("Encontrado");
        break;
    }
}
```

---

## Siguiente paso

Ya sabes repetir acciones con bucles. Ahora vamos a **organizar código en funciones**.

→ [18-funciones.md](18-funciones.md)

Ahí aprenderás a crear funciones, parámetros, return, arrow functions y más.

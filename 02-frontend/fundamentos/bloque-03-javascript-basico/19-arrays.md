# Módulo 19: Arrays en JavaScript

## Índice
1. [¿Qué son los arrays?](#qué-son-los-arrays)
2. [Crear arrays](#crear-arrays)
3. [Acceder a elementos](#acceder-a-elementos)
4. [Modificar elementos](#modificar-elementos)
5. [Métodos básicos de arrays](#métodos-básicos-de-arrays)
6. [Iterar sobre arrays](#iterar-sobre-arrays)
7. [Métodos avanzados](#métodos-avanzados)
8. [Arrays multidimensionales](#arrays-multidimensionales)
9. [Ejercicios prácticos](#ejercicios-prácticos)
10. [Errores comunes](#errores-comunes)
11. [Buenas prácticas](#buenas-prácticas)
12. [Cheatsheet](#cheatsheet)

---

## ¿Qué son los arrays?

**Un array es una lista ordenada de valores**.

### Analogía: Lista de compras

```
Lista de compras:
1. Leche
2. Pan
3. Huevos
4. Queso
5. Frutas
```

En JavaScript:

```javascript
const listaCompras = ["Leche", "Pan", "Huevos", "Queso", "Frutas"];
```

### ¿Para qué sirven?

- Almacenar múltiples valores en una sola variable
- Mantener colecciones ordenadas de datos
- Procesar listas de elementos
- Almacenar resultados de operaciones
- Trabajar con conjuntos de datos

### Sin arrays vs Con arrays

```javascript
// ❌ Sin arrays: Variables separadas
const fruta1 = "manzana";
const fruta2 = "banana";
const fruta3 = "naranja";

// ✅ Con arrays: Una sola variable
const frutas = ["manzana", "banana", "naranja"];
```

---

## Crear arrays

### Sintaxis literal (recomendada)

```javascript
const frutas = ["manzana", "banana", "naranja"];
console.log(frutas); // ["manzana", "banana", "naranja"]
```

### Array vacío

```javascript
const lista = [];
console.log(lista); // []
```

### Array con diferentes tipos

```javascript
const mixto = [1, "texto", true, null, { nombre: "Ana" }];
console.log(mixto); // [1, "texto", true, null, {nombre: "Ana"}]
```

### Constructor Array (menos común)

```javascript
const numeros = new Array(1, 2, 3, 4, 5);
console.log(numeros); // [1, 2, 3, 4, 5]

// Cuidado: Un solo número crea array vacío de esa longitud
const vacio = new Array(5);
console.log(vacio); // [empty × 5]
console.log(vacio.length); // 5
```

### Array de un rango

```javascript
// Crear array de números del 1 al 5
const numeros = [1, 2, 3, 4, 5];

// O con un bucle
const rango = [];
for (let i = 1; i <= 5; i++) {
    rango.push(i);
}
console.log(rango); // [1, 2, 3, 4, 5]
```

---

## Acceder a elementos

**Los arrays usan índices que empiezan en 0**.

### Índices

```javascript
const frutas = ["manzana", "banana", "naranja", "uva"];
//               0          1          2         3

console.log(frutas[0]); // "manzana"
console.log(frutas[1]); // "banana"
console.log(frutas[2]); // "naranja"
console.log(frutas[3]); // "uva"
```

### Primer y último elemento

```javascript
const numeros = [10, 20, 30, 40, 50];

// Primer elemento
console.log(numeros[0]); // 10

// Último elemento
console.log(numeros[numeros.length - 1]); // 50
```

### Índices fuera de rango

```javascript
const frutas = ["manzana", "banana", "naranja"];

console.log(frutas[10]); // undefined (no existe)
console.log(frutas[-1]); // undefined (índice negativo no funciona)
```

### Longitud del array

```javascript
const frutas = ["manzana", "banana", "naranja"];
console.log(frutas.length); // 3

const vacio = [];
console.log(vacio.length); // 0
```

---

## Modificar elementos

### Cambiar un elemento

```javascript
const frutas = ["manzana", "banana", "naranja"];

frutas[1] = "pera";
console.log(frutas); // ["manzana", "pera", "naranja"]
```

### Añadir elemento al final

```javascript
const frutas = ["manzana", "banana"];

frutas[frutas.length] = "naranja";
console.log(frutas); // ["manzana", "banana", "naranja"]

// O mejor: usa push()
frutas.push("uva");
console.log(frutas); // ["manzana", "banana", "naranja", "uva"]
```

### const no impide modificar el contenido

```javascript
const numeros = [1, 2, 3];

// ✅ Puedes modificar elementos
numeros[0] = 10;
numeros.push(4);
console.log(numeros); // [10, 2, 3, 4]

// ❌ Pero no puedes reasignar el array completo
numeros = [5, 6, 7]; // Error: Assignment to constant variable
```

---

## Métodos básicos de arrays

### push() - Añadir al final

```javascript
const frutas = ["manzana", "banana"];

frutas.push("naranja");
console.log(frutas); // ["manzana", "banana", "naranja"]

// Añadir múltiples elementos
frutas.push("uva", "pera");
console.log(frutas); // ["manzana", "banana", "naranja", "uva", "pera"]

// push() devuelve la nueva longitud
const nuevaLongitud = frutas.push("kiwi");
console.log(nuevaLongitud); // 6
```

### pop() - Quitar el último

```javascript
const frutas = ["manzana", "banana", "naranja"];

const eliminado = frutas.pop();
console.log(eliminado); // "naranja"
console.log(frutas);    // ["manzana", "banana"]
```

### unshift() - Añadir al inicio

```javascript
const frutas = ["banana", "naranja"];

frutas.unshift("manzana");
console.log(frutas); // ["manzana", "banana", "naranja"]

// Añadir múltiples
frutas.unshift("pera", "uva");
console.log(frutas); // ["pera", "uva", "manzana", "banana", "naranja"]
```

### shift() - Quitar el primero

```javascript
const frutas = ["manzana", "banana", "naranja"];

const eliminado = frutas.shift();
console.log(eliminado); // "manzana"
console.log(frutas);    // ["banana", "naranja"]
```

### indexOf() - Buscar posición

```javascript
const frutas = ["manzana", "banana", "naranja", "banana"];

console.log(frutas.indexOf("banana"));  // 1 (primera aparición)
console.log(frutas.indexOf("naranja")); // 2
console.log(frutas.indexOf("uva"));     // -1 (no encontrado)
```

### includes() - Verificar si existe

```javascript
const frutas = ["manzana", "banana", "naranja"];

console.log(frutas.includes("banana")); // true
console.log(frutas.includes("uva"));    // false
```

### slice() - Copiar porción

```javascript
const frutas = ["manzana", "banana", "naranja", "uva", "pera"];

// slice(inicio, fin) - no incluye el fin
const algunas = frutas.slice(1, 3);
console.log(algunas); // ["banana", "naranja"]

// Sin fin: hasta el final
const desde2 = frutas.slice(2);
console.log(desde2); // ["naranja", "uva", "pera"]

// El array original no cambia
console.log(frutas); // ["manzana", "banana", "naranja", "uva", "pera"]
```

### splice() - Modificar array

```javascript
const frutas = ["manzana", "banana", "naranja", "uva"];

// splice(inicio, cantidad) - elimina elementos
frutas.splice(1, 2); // Desde índice 1, elimina 2 elementos
console.log(frutas); // ["manzana", "uva"]

// splice(inicio, cantidad, elementos...) - elimina y añade
const colores = ["rojo", "verde", "azul"];
colores.splice(1, 1, "amarillo", "naranja");
console.log(colores); // ["rojo", "amarillo", "naranja", "azul"]
```

### concat() - Unir arrays

```javascript
const arr1 = [1, 2, 3];
const arr2 = [4, 5, 6];

const unidos = arr1.concat(arr2);
console.log(unidos); // [1, 2, 3, 4, 5, 6]

// Los originales no cambian
console.log(arr1); // [1, 2, 3]
console.log(arr2); // [4, 5, 6]

// Múltiples arrays
const arr3 = [7, 8];
const todos = arr1.concat(arr2, arr3);
console.log(todos); // [1, 2, 3, 4, 5, 6, 7, 8]
```

### join() - Convertir a string

```javascript
const frutas = ["manzana", "banana", "naranja"];

const texto = frutas.join(", ");
console.log(texto); // "manzana, banana, naranja"

// Con otro separador
console.log(frutas.join(" - ")); // "manzana - banana - naranja"
console.log(frutas.join(""));    // "manzanabanananaranja"
```

### reverse() - Invertir orden

```javascript
const numeros = [1, 2, 3, 4, 5];

numeros.reverse();
console.log(numeros); // [5, 4, 3, 2, 1]

// ⚠️ Modifica el array original
```

### sort() - Ordenar

```javascript
const frutas = ["naranja", "manzana", "banana"];

frutas.sort();
console.log(frutas); // ["banana", "manzana", "naranja"]

// Con números: ¡cuidado!
const numeros = [10, 5, 40, 25, 1000];
numeros.sort();
console.log(numeros); // [10, 1000, 25, 40, 5] ❌ Orden alfabético

// Solución: función de comparación
numeros.sort((a, b) => a - b);
console.log(numeros); // [5, 10, 25, 40, 1000] ✅ Orden numérico
```

---

## Iterar sobre arrays

### for tradicional

```javascript
const frutas = ["manzana", "banana", "naranja"];

for (let i = 0; i < frutas.length; i++) {
    console.log(`Fruta ${i + 1}: ${frutas[i]}`);
}

// Output:
// Fruta 1: manzana
// Fruta 2: banana
// Fruta 3: naranja
```

### for...of (recomendado)

```javascript
const frutas = ["manzana", "banana", "naranja"];

for (const fruta of frutas) {
    console.log(fruta);
}

// Output:
// manzana
// banana
// naranja
```

### forEach()

```javascript
const frutas = ["manzana", "banana", "naranja"];

frutas.forEach((fruta, indice) => {
    console.log(`${indice}: ${fruta}`);
});

// Output:
// 0: manzana
// 1: banana
// 2: naranja
```

### Comparación

```javascript
const numeros = [1, 2, 3, 4, 5];

// for tradicional: tienes control total
for (let i = 0; i < numeros.length; i++) {
    if (numeros[i] === 3) break; // Puedes usar break
    console.log(numeros[i]);
}

// for...of: más limpio, puedes usar break
for (const num of numeros) {
    if (num === 3) break;
    console.log(num);
}

// forEach: más funcional, NO puedes usar break
numeros.forEach(num => {
    console.log(num);
    // break; ❌ Error de sintaxis
});
```

---

## Métodos avanzados

### map() - Transformar cada elemento

```javascript
const numeros = [1, 2, 3, 4, 5];

const dobles = numeros.map(num => num * 2);
console.log(dobles); // [2, 4, 6, 8, 10]

// El original no cambia
console.log(numeros); // [1, 2, 3, 4, 5]
```

### filter() - Filtrar elementos

```javascript
const numeros = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10];

const pares = numeros.filter(num => num % 2 === 0);
console.log(pares); // [2, 4, 6, 8, 10]

const mayoresQue5 = numeros.filter(num => num > 5);
console.log(mayoresQue5); // [6, 7, 8, 9, 10]
```

### find() - Encontrar primer elemento

```javascript
const usuarios = [
    { id: 1, nombre: "Ana" },
    { id: 2, nombre: "Juan" },
    { id: 3, nombre: "María" }
];

const usuario = usuarios.find(u => u.id === 2);
console.log(usuario); // { id: 2, nombre: "Juan" }

const noExiste = usuarios.find(u => u.id === 10);
console.log(noExiste); // undefined
```

### some() - Al menos uno cumple

```javascript
const numeros = [1, 2, 3, 4, 5];

const hayPares = numeros.some(num => num % 2 === 0);
console.log(hayPares); // true

const hayNegativos = numeros.some(num => num < 0);
console.log(hayNegativos); // false
```

### every() - Todos cumplen

```javascript
const numeros = [2, 4, 6, 8, 10];

const todosPares = numeros.every(num => num % 2 === 0);
console.log(todosPares); // true

const todosPositivos = numeros.every(num => num > 0);
console.log(todosPositivos); // true

const todosMayoresQue5 = numeros.every(num => num > 5);
console.log(todosMayoresQue5); // false
```

### reduce() - Reducir a un solo valor

```javascript
const numeros = [1, 2, 3, 4, 5];

// Sumar todos los números
const suma = numeros.reduce((acumulador, actual) => acumulador + actual, 0);
console.log(suma); // 15

// Encontrar el máximo
const maximo = numeros.reduce((max, actual) => actual > max ? actual : max);
console.log(maximo); // 5
```

---

## Arrays multidimensionales

**Arrays dentro de arrays (matrices)**.

### Crear matriz

```javascript
const matriz = [
    [1, 2, 3],
    [4, 5, 6],
    [7, 8, 9]
];

console.log(matriz);
// [
//   [1, 2, 3],
//   [4, 5, 6],
//   [7, 8, 9]
// ]
```

### Acceder a elementos

```javascript
const matriz = [
    [1, 2, 3],
    [4, 5, 6],
    [7, 8, 9]
];

console.log(matriz[0]);    // [1, 2, 3] (primera fila)
console.log(matriz[0][0]); // 1 (fila 0, columna 0)
console.log(matriz[1][2]); // 6 (fila 1, columna 2)
console.log(matriz[2][1]); // 8 (fila 2, columna 1)
```

### Iterar matriz

```javascript
const matriz = [
    [1, 2, 3],
    [4, 5, 6],
    [7, 8, 9]
];

// Recorrer todas las filas y columnas
for (let fila = 0; fila < matriz.length; fila++) {
    for (let col = 0; col < matriz[fila].length; col++) {
        console.log(`[${fila}][${col}] = ${matriz[fila][col]}`);
    }
}
```

---

## Ejercicios prácticos

### Ejercicio 1: Crear y acceder
**Nivel**: ⭐☆☆☆☆

Crea un array con 5 colores. Muestra el primero y el último.

<details>
<summary>Ver solución</summary>

```javascript
const colores = ["rojo", "azul", "verde", "amarillo", "naranja"];

console.log("Primer color:", colores[0]);
console.log("Último color:", colores[colores.length - 1]);

// Output:
// Primer color: rojo
// Último color: naranja
```

</details>

---

### Ejercicio 2: Añadir y quitar elementos
**Nivel**: ⭐⭐☆☆☆

Crea un array vacío. Añade 3 elementos al final. Quita el primero y el último.

<details>
<summary>Ver solución</summary>

```javascript
const lista = [];

// Añadir elementos
lista.push("Elemento 1");
lista.push("Elemento 2");
lista.push("Elemento 3");
console.log("Después de push:", lista);
// ["Elemento 1", "Elemento 2", "Elemento 3"]

// Quitar el primero
lista.shift();
console.log("Después de shift:", lista);
// ["Elemento 2", "Elemento 3"]

// Quitar el último
lista.pop();
console.log("Después de pop:", lista);
// ["Elemento 2"]
```

</details>

---

### Ejercicio 3: Buscar en array
**Nivel**: ⭐⭐☆☆☆

Dado un array de nombres, verifica si "Juan" está en la lista.

<details>
<summary>Ver solución</summary>

```javascript
const nombres = ["Ana", "Pedro", "Juan", "María", "Carlos"];

// Método 1: includes()
if (nombres.includes("Juan")) {
    console.log("Juan está en la lista");
} else {
    console.log("Juan no está en la lista");
}

// Método 2: indexOf()
const posicion = nombres.indexOf("Juan");
if (posicion !== -1) {
    console.log(`Juan está en la posición ${posicion}`);
} else {
    console.log("Juan no está en la lista");
}

// Output:
// Juan está en la lista
// Juan está en la posición 2
```

</details>

---

### Ejercicio 4: Sumar números
**Nivel**: ⭐⭐☆☆☆

Suma todos los números de un array.

<details>
<summary>Ver solución</summary>

```javascript
const numeros = [5, 10, 15, 20, 25];

// Método 1: for tradicional
let suma1 = 0;
for (let i = 0; i < numeros.length; i++) {
    suma1 += numeros[i];
}
console.log("Suma con for:", suma1); // 75

// Método 2: for...of
let suma2 = 0;
for (const num of numeros) {
    suma2 += num;
}
console.log("Suma con for...of:", suma2); // 75

// Método 3: reduce()
const suma3 = numeros.reduce((total, num) => total + num, 0);
console.log("Suma con reduce:", suma3); // 75
```

</details>

---

### Ejercicio 5: Filtrar pares
**Nivel**: ⭐⭐⭐☆☆

Crea un nuevo array con solo los números pares.

<details>
<summary>Ver solución</summary>

```javascript
const numeros = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10];

const pares = numeros.filter(num => num % 2 === 0);
console.log("Números pares:", pares);
// [2, 4, 6, 8, 10]
```

</details>

---

### Ejercicio 6: Duplicar valores
**Nivel**: ⭐⭐⭐☆☆

Duplica cada número del array.

<details>
<summary>Ver solución</summary>

```javascript
const numeros = [1, 2, 3, 4, 5];

const duplicados = numeros.map(num => num * 2);
console.log("Números duplicados:", duplicados);
// [2, 4, 6, 8, 10]

// El original no cambia
console.log("Original:", numeros);
// [1, 2, 3, 4, 5]
```

</details>

---

### Ejercicio 7: Promedio de calificaciones
**Nivel**: ⭐⭐⭐☆☆

Calcula el promedio de un array de calificaciones.

<details>
<summary>Ver solución</summary>

```javascript
const calificaciones = [85, 90, 78, 92, 88];

const suma = calificaciones.reduce((total, nota) => total + nota, 0);
const promedio = suma / calificaciones.length;

console.log(`Suma: ${suma}`);
console.log(`Promedio: ${promedio.toFixed(2)}`);

// Output:
// Suma: 433
// Promedio: 86.60
```

</details>

---

### Ejercicio 8: Encontrar el mayor
**Nivel**: ⭐⭐⭐☆☆

Encuentra el número más grande en un array.

<details>
<summary>Ver solución</summary>

```javascript
const numeros = [45, 23, 67, 12, 89, 34];

// Método 1: reduce()
const mayor1 = numeros.reduce((max, num) => num > max ? num : max);
console.log("Mayor con reduce:", mayor1); // 89

// Método 2: Math.max() con spread
const mayor2 = Math.max(...numeros);
console.log("Mayor con Math.max:", mayor2); // 89

// Método 3: sort()
const ordenado = [...numeros].sort((a, b) => b - a);
const mayor3 = ordenado[0];
console.log("Mayor con sort:", mayor3); // 89
```

</details>

---

### Ejercicio 9: Eliminar duplicados
**Nivel**: ⭐⭐⭐⭐☆

Elimina los valores duplicados de un array.

<details>
<summary>Ver solución</summary>

```javascript
const numeros = [1, 2, 3, 2, 4, 1, 5, 3, 6];

// Método 1: Set
const unicos1 = [...new Set(numeros)];
console.log("Con Set:", unicos1);
// [1, 2, 3, 4, 5, 6]

// Método 2: filter() con indexOf()
const unicos2 = numeros.filter((num, indice) => {
    return numeros.indexOf(num) === indice;
});
console.log("Con filter:", unicos2);
// [1, 2, 3, 4, 5, 6]
```

</details>

---

### Ejercicio 10: Carrito de compras
**Nivel**: ⭐⭐⭐⭐☆

Crea un carrito de compras con objetos. Calcula el total.

<details>
<summary>Ver solución</summary>

```javascript
const carrito = [
    { producto: "Laptop", precio: 800, cantidad: 1 },
    { producto: "Mouse", precio: 25, cantidad: 2 },
    { producto: "Teclado", precio: 50, cantidad: 1 }
];

// Calcular total
const total = carrito.reduce((suma, item) => {
    return suma + (item.precio * item.cantidad);
}, 0);

console.log("Productos en el carrito:");
carrito.forEach(item => {
    const subtotal = item.precio * item.cantidad;
    console.log(`- ${item.producto}: ${item.cantidad} x ${item.precio}€ = ${subtotal}€`);
});

console.log(`\nTotal: ${total}€`);

// Output:
// Productos en el carrito:
// - Laptop: 1 x 800€ = 800€
// - Mouse: 2 x 25€ = 50€
// - Teclado: 1 x 50€ = 50€
//
// Total: 900€
```

</details>

---

## Errores comunes

### ❌ Error 1: Índice fuera de rango

```javascript
const frutas = ["manzana", "banana"];

console.log(frutas[5]); // undefined (no error, pero no existe)

// Solución: verifica la longitud
if (5 < frutas.length) {
    console.log(frutas[5]);
} else {
    console.log("Índice fuera de rango");
}
```

---

### ❌ Error 2: Modificar array mientras iteras

```javascript
const numeros = [1, 2, 3, 4, 5];

// MAL: Modificar mientras iteras puede causar problemas
for (let i = 0; i < numeros.length; i++) {
    numeros.pop(); // Quita elementos mientras itera
}

// BIEN: Itera sobre una copia o de atrás hacia adelante
for (let i = numeros.length - 1; i >= 0; i--) {
    numeros.pop();
}
```

---

### ❌ Error 3: sort() con números

```javascript
const numeros = [10, 5, 40, 25, 1000];

// MAL: Orden alfabético
numeros.sort();
console.log(numeros); // [10, 1000, 25, 40, 5]

// BIEN: Con función de comparación
numeros.sort((a, b) => a - b);
console.log(numeros); // [5, 10, 25, 40, 1000]
```

---

### ❌ Error 4: Confundir métodos que modifican vs. crean nuevo

```javascript
const arr = [1, 2, 3];

// Métodos que MODIFICAN el original:
arr.push(4);      // arr = [1, 2, 3, 4]
arr.pop();        // arr = [1, 2, 3]
arr.reverse();    // arr = [3, 2, 1]

// Métodos que CREAN nuevo array:
const dobles = arr.map(x => x * 2); // arr no cambia
const filtrado = arr.filter(x => x > 1); // arr no cambia
```

---

## Buenas prácticas

### ✅ 1. Usa const para arrays

```javascript
// ✅ const (puedes modificar el contenido)
const frutas = ["manzana"];
frutas.push("banana"); // OK
frutas[0] = "pera";    // OK

// ❌ let innecesario
let frutas = ["manzana"];
```

---

### ✅ 2. Usa métodos modernos

```javascript
// ❌ Forma antigua
const dobles = [];
for (let i = 0; i < numeros.length; i++) {
    dobles.push(numeros[i] * 2);
}

// ✅ Forma moderna
const dobles = numeros.map(n => n * 2);
```

---

### ✅ 3. Verifica si es array

```javascript
const valor = [1, 2, 3];

if (Array.isArray(valor)) {
    console.log("Es un array");
}
```

---

### ✅ 4. Copia arrays correctamente

```javascript
const original = [1, 2, 3];

// ❌ Esto NO copia, solo referencia
const copia1 = original;
copia1.push(4);
console.log(original); // [1, 2, 3, 4] ← También cambió

// ✅ Copia con spread operator
const copia2 = [...original];
copia2.push(5);
console.log(original); // [1, 2, 3, 4] ← No cambió
console.log(copia2);   // [1, 2, 3, 4, 5]

// ✅ Copia con slice()
const copia3 = original.slice();
```

---

## Cheatsheet

### Crear arrays

```javascript
const arr = [1, 2, 3];                    // Literal
const vacio = [];                          // Vacío
const mixto = [1, "texto", true];         // Diferentes tipos
```

### Acceso y modificación

```javascript
arr[0]                      // Primer elemento
arr[arr.length - 1]         // Último elemento
arr.length                  // Longitud
arr[2] = 10                 // Modificar elemento
```

### Métodos que modifican el original

```javascript
arr.push(item)              // Añadir al final
arr.pop()                   // Quitar el último
arr.unshift(item)           // Añadir al inicio
arr.shift()                 // Quitar el primero
arr.splice(inicio, cant)    // Eliminar/insertar
arr.reverse()               // Invertir orden
arr.sort()                  // Ordenar
```

### Métodos que crean nuevo array

```javascript
arr.slice(inicio, fin)      // Copiar porción
arr.concat(arr2)            // Unir arrays
arr.map(fn)                 // Transformar elementos
arr.filter(fn)              // Filtrar elementos
[...arr]                    // Copia con spread
```

### Búsqueda

```javascript
arr.indexOf(valor)          // Posición (o -1)
arr.includes(valor)         // true/false
arr.find(fn)                // Primer elemento que cumple
arr.findIndex(fn)           // Índice del primero que cumple
```

### Verificación

```javascript
arr.some(fn)                // Al menos uno cumple
arr.every(fn)               // Todos cumplen
```

### Iteración

```javascript
for (let i = 0; i < arr.length; i++) { }    // for tradicional
for (const item of arr) { }                  // for...of
arr.forEach((item, i) => { })                // forEach
```

### Transformación

```javascript
arr.map(x => x * 2)                  // Transformar
arr.filter(x => x > 10)              // Filtrar
arr.reduce((acc, x) => acc + x, 0)   // Reducir
```

---

## Siguiente paso

Ya dominas los arrays. Ahora vamos a aprender sobre **objetos**, la estructura de datos más importante de JavaScript.

→ [20-objetos.md](20-objetos.md)

Ahí aprenderás a crear objetos, acceder a propiedades, métodos, destructuring y más.

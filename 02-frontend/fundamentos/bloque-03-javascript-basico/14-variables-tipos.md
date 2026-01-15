# Módulo 14: Variables y Tipos de Datos

## Índice
1. [¿Qué son las variables?](#qué-son-las-variables)
2. [Declarar variables: let, const, var](#declarar-variables-let-const-var)
3. [Tipos de datos primitivos](#tipos-de-datos-primitivos)
4. [Nombramiento de variables](#nombramiento-de-variables)
5. [typeof: saber el tipo de un dato](#typeof-saber-el-tipo-de-un-dato)
6. [Conversión de tipos](#conversión-de-tipos)
7. [Ejercicios prácticos](#ejercicios-prácticos)
8. [Errores comunes](#errores-comunes)
9. [Buenas prácticas](#buenas-prácticas)
10. [Cheatsheet](#cheatsheet)

---

## ¿Qué son las variables?

**Una variable es una caja con nombre donde guardas información**.

### Analogía: Cajas etiquetadas

Imagina tu escritorio con cajas:

```
┌──────────────┐  ┌──────────────┐  ┌──────────────┐
│   nombre     │  │    edad      │  │  estudiante  │
│              │  │              │  │              │
│   "Ana"      │  │      25      │  │     true     │
└──────────────┘  └──────────────┘  └──────────────┘
```

- La **etiqueta** es el nombre de la variable (`nombre`, `edad`, `estudiante`)
- El **contenido** es el valor que guardas (`"Ana"`, `25`, `true`)

### ¿Para qué sirven?

```javascript
// Sin variables: repetir valores
console.log("Hola Ana");
console.log("Ana tiene 25 años");
console.log("El email de Ana es ana@email.com");

// Con variables: guardar una vez, usar muchas veces
const nombre = "Ana";
const edad = 25;
const email = "ana@email.com";

console.log("Hola " + nombre);
console.log(nombre + " tiene " + edad + " años");
console.log("El email de " + nombre + " es " + email);
```

**Ventajas**:
- Si cambias el nombre, solo lo cambias en un lugar
- El código es más legible
- Puedes hacer operaciones con los valores

---

## Declarar variables: let, const, var

JavaScript tiene **3 formas** de declarar variables.

### `const` - Valor constante (no cambia)

```javascript
const nombre = "Ana";
console.log(nombre); // "Ana"

// ❌ No puedes cambiar su valor:
nombre = "Juan"; // Error: Assignment to constant variable
```

**Úsala cuando**: El valor no cambiará.

```javascript
const PI = 3.14159;
const MAX_INTENTOS = 3;
const URL_API = "https://api.ejemplo.com";
```

---

### `let` - Valor variable (puede cambiar)

```javascript
let edad = 25;
console.log(edad); // 25

// ✅ Puedes cambiar su valor:
edad = 26;
console.log(edad); // 26
```

**Úsala cuando**: El valor cambiará durante la ejecución.

```javascript
let contador = 0;
contador = contador + 1; // 1
contador = contador + 1; // 2

let temperatura = 20;
temperatura = 25; // Cambió la temperatura
```

---

### `var` - La forma antigua (evítala)

```javascript
var nombre = "Ana"; // Funciona pero es antigua
```

**Problema con `var`**: Tiene comportamientos raros que pueden causar bugs. JavaScript moderno usa `let` y `const`.

**Regla**: Usa `const` siempre. Si necesitas cambiar el valor, usa `let`. **Nunca uses `var`**.

---

### Declaración vs. Asignación

```javascript
// Declaración: creas la variable
let edad;

// Asignación: le das un valor
edad = 25;

// O ambas cosas a la vez (recomendado):
let edad = 25;
```

### Reasignación

```javascript
let puntos = 0;    // Declaración + asignación inicial

puntos = 10;       // Reasignación (no hace falta let de nuevo)
puntos = 20;       // Reasignación de nuevo
```

**Importante**: Solo usas `let` o `const` **la primera vez**.

---

## Tipos de datos primitivos

JavaScript tiene **7 tipos primitivos**. Por ahora veremos los 5 más importantes:

### 1. `string` - Texto

Cadenas de caracteres (texto).

```javascript
const nombre = "Ana";
const apellido = 'García'; // Comillas simples también funcionan
const mensaje = "Hola, ¿cómo estás?";
const vacio = "";  // String vacío (válido)
```

#### Comillas: simples vs. dobles

```javascript
// Ambas funcionan igual:
const texto1 = "Hola";
const texto2 = 'Hola';

// Elige una y sé consistente
// (En este curso usaremos comillas dobles)
```

#### Template literals (plantillas)

```javascript
const nombre = "Ana";
const edad = 25;

// Antigua forma (concatenación):
const mensaje1 = "Hola, soy " + nombre + " y tengo " + edad + " años";

// Nueva forma (template literals):
const mensaje2 = `Hola, soy ${nombre} y tengo ${edad} años`;

console.log(mensaje2); // "Hola, soy Ana y tengo 25 años"
```

**Template literals**:
- Usan backticks (`` ` ``) en vez de comillas
- Permiten insertar variables con `${variable}`
- Permiten saltos de línea

```javascript
const parrafo = `
  Este es un texto
  que abarca múltiples
  líneas.
`;
```

---

### 2. `number` - Números

Todos los números (enteros y decimales).

```javascript
const edad = 25;               // Entero
const precio = 19.99;          // Decimal
const temperatura = -5;        // Negativo
const grande = 1000000;        // Un millón
const grandeConSeparador = 1_000_000; // Más legible (ES2021)
```

#### Operaciones básicas

```javascript
const suma = 10 + 5;       // 15
const resta = 10 - 5;      // 5
const multiplicacion = 10 * 5; // 50
const division = 10 / 5;   // 2
const resto = 10 % 3;      // 1 (resto de 10 ÷ 3)
```

#### Números especiales

```javascript
const infinito = Infinity;
const negInfinito = -Infinity;
const noEsNumero = NaN; // "Not a Number"

console.log(10 / 0);  // Infinity
console.log("hola" * 2); // NaN (no puedes multiplicar texto)
```

---

### 3. `boolean` - Verdadero/Falso

Solo tiene 2 valores posibles: `true` o `false`.

```javascript
const esEstudiante = true;
const tieneTrabajo = false;
const mayorDeEdad = true;
```

**Usos típicos**:

```javascript
const estaLogueado = true;
const formularioValido = false;
const aceptoTerminos = true;
```

Los veremos en profundidad cuando aprendamos condicionales.

---

### 4. `undefined` - Sin valor asignado

Variable declarada pero sin valor.

```javascript
let nombre;
console.log(nombre); // undefined

let edad = undefined; // Técnicamente válido pero raro
```

**Típico escenario**:

```javascript
let resultado;

// ... más código ...

if (algunaCondicion) {
    resultado = 42;
}

console.log(resultado); // undefined si la condición no se cumplió
```

---

### 5. `null` - Ausencia intencional de valor

Diferencia con `undefined`:
- `undefined`: "No tiene valor aún"
- `null`: "Intencionalmente vacío"

```javascript
let usuario = null; // Explícitamente no hay usuario

// Luego, cuando se loguea:
usuario = { nombre: "Ana", email: "ana@email.com" };
```

---

### Otros tipos (avanzados)

Por ahora no los usaremos, pero existen:
- `symbol` - Valores únicos e inmutables
- `bigint` - Números enteros muy grandes

---

## Nombramiento de variables

### Reglas obligatorias

```javascript
// ✅ VÁLIDO
const nombre = "Ana";
const edad2 = 25;
const _privado = "valor";
const $especial = "valor";
const nombreCompleto = "Ana García";
const URL_API = "https://...";

// ❌ INVÁLIDO
const 2edad = 25;         // No puede empezar con número
const nombre-completo = "Ana"; // No puede tener guion
const const = "valor";    // No puede ser palabra reservada
const function = "valor"; // Palabra reservada
```

**Resumen de reglas**:
1. Solo letras, números, `_` y `$`
2. No puede empezar con número
3. No puede ser una palabra reservada (`let`, `const`, `if`, `function`, etc.)
4. JavaScript distingue mayúsculas/minúsculas:
   ```javascript
   const nombre = "Ana";
   const Nombre = "Juan";  // Variable diferente
   const NOMBRE = "Luis";  // Variable diferente
   ```

---

### Convenciones (no obligatorias pero recomendadas)

#### 1. **camelCase** para variables y funciones

```javascript
const nombre = "Ana";
const nombreCompleto = "Ana García";
const edadDelUsuario = 25;
const esMayorDeEdad = true;
```

Primera palabra en minúscula, siguientes en mayúscula.

---

#### 2. **SCREAMING_SNAKE_CASE** para constantes globales

```javascript
const MAX_INTENTOS = 3;
const URL_API = "https://api.ejemplo.com";
const DIAS_SEMANA = 7;
```

---

#### 3. Nombres descriptivos

```javascript
// ❌ Evita esto
const x = 25;
const a = "Ana";
const b = true;

// ✅ Mejor
const edad = 25;
const nombre = "Ana";
const esEstudiante = true;
```

---

#### 4. Inglés vs. Español

```javascript
// Opción 1: Todo en inglés (común en proyectos profesionales)
const userName = "Ana";
const userAge = 25;
const isStudent = true;

// Opción 2: Todo en español (válido, especialmente aprendiendo)
const nombreUsuario = "Ana";
const edadUsuario = 25;
const esEstudiante = true;
```

**Elige uno y sé consistente**.

---

#### 5. Booleans: usa preguntas

```javascript
// ✅ Booleanos con prefijos: es, tiene, esta, puede, debe
const esValido = true;
const tienePermisos = false;
const estaActivo = true;
const puedeEditar = false;
const debeValidar = true;
```

---

## typeof: saber el tipo de un dato

El operador `typeof` te dice qué tipo de dato es una variable.

```javascript
const nombre = "Ana";
const edad = 25;
const esEstudiante = true;
const sinValor = undefined;
const vacio = null;

console.log(typeof nombre);      // "string"
console.log(typeof edad);        // "number"
console.log(typeof esEstudiante); // "boolean"
console.log(typeof sinValor);    // "undefined"
console.log(typeof vacio);       // "object" (bug histórico de JavaScript)
```

### Bug curioso: `typeof null`

```javascript
console.log(typeof null); // "object" (debería ser "null")
```

Este es un **bug histórico** de JavaScript que no se puede arreglar porque rompería código antiguo.

### Usos prácticos de `typeof`

```javascript
const valor = "123";

if (typeof valor === "string") {
    console.log("Es un texto");
}

if (typeof valor === "number") {
    console.log("Es un número");
}
```

---

## Conversión de tipos

A veces necesitas convertir un tipo en otro.

### String → Number

```javascript
// Método 1: Number()
const texto = "123";
const numero = Number(texto);
console.log(numero);        // 123
console.log(typeof numero); // "number"

// Método 2: parseInt() para enteros
const entero = parseInt("123");
console.log(entero); // 123

// Método 3: parseFloat() para decimales
const decimal = parseFloat("19.99");
console.log(decimal); // 19.99

// Cuidado con valores inválidos:
console.log(Number("hola")); // NaN
```

---

### Number → String

```javascript
// Método 1: String()
const numero = 123;
const texto = String(numero);
console.log(texto);        // "123"
console.log(typeof texto); // "string"

// Método 2: .toString()
const texto2 = numero.toString();
console.log(texto2); // "123"

// Método 3: Template literal
const texto3 = `${numero}`;
console.log(texto3); // "123"
```

---

### Any → Boolean

```javascript
// Valores "falsy" (se convierten a false):
Boolean(0);         // false
Boolean("");        // false (string vacío)
Boolean(null);      // false
Boolean(undefined); // false
Boolean(NaN);       // false

// Todo lo demás es "truthy" (se convierte a true):
Boolean(1);         // true
Boolean("hola");    // true
Boolean(" ");       // true (espacio no es vacío)
Boolean([]);        // true (array vacío)
Boolean({});        // true (objeto vacío)
```

---

### Conversión automática (coerción)

JavaScript convierte tipos automáticamente en ciertos casos:

```javascript
// String + Number = String
console.log("Edad: " + 25);  // "Edad: 25"

// Number en contexto de string
console.log("5" + 3);   // "53" (concatenación)
console.log("5" - 3);   // 2 (resta, convierte a número)
console.log("5" * 3);   // 15 (multiplicación)

// Boolean en contexto numérico
console.log(true + 1);  // 2 (true = 1)
console.log(false + 1); // 1 (false = 0)
```

**Cuidado**: La coerción automática puede causar bugs. Es mejor convertir explícitamente.

---

## Ejercicios prácticos

### Ejercicio 1: Declarar variables
**Nivel**: ⭐☆☆☆☆

Declara variables para:
- Tu nombre (string)
- Tu edad (number)
- Si eres estudiante (boolean)

Muéstralas en la consola.

<details>
<summary>Ver solución</summary>

```javascript
const nombre = "Ana García";
const edad = 25;
const esEstudiante = true;

console.log("Nombre:", nombre);
console.log("Edad:", edad);
console.log("¿Estudiante?", esEstudiante);
```

</details>

---

### Ejercicio 2: Reasignación
**Nivel**: ⭐☆☆☆☆

1. Declara una variable `puntos` con valor inicial 0
2. Súmale 10
3. Súmale 5 más
4. Muestra el valor final

<details>
<summary>Ver solución</summary>

```javascript
let puntos = 0;
console.log("Puntos iniciales:", puntos); // 0

puntos = puntos + 10;
console.log("Después de sumar 10:", puntos); // 10

puntos = puntos + 5;
console.log("Después de sumar 5:", puntos); // 15

// O más corto con +=:
let puntos2 = 0;
puntos2 += 10;
puntos2 += 5;
console.log("Puntos finales:", puntos2); // 15
```

</details>

---

### Ejercicio 3: Template literals
**Nivel**: ⭐⭐☆☆☆

Crea variables para nombre, edad y ciudad. Crea un mensaje usando template literals que diga:
"Me llamo [nombre], tengo [edad] años y vivo en [ciudad]"

<details>
<summary>Ver solución</summary>

```javascript
const nombre = "Ana";
const edad = 25;
const ciudad = "Madrid";

const mensaje = `Me llamo ${nombre}, tengo ${edad} años y vivo en ${ciudad}`;
console.log(mensaje);
// "Me llamo Ana, tengo 25 años y vivo en Madrid"
```

</details>

---

### Ejercicio 4: Typeof
**Nivel**: ⭐⭐☆☆☆

Crea 5 variables con tipos diferentes y muestra el tipo de cada una usando `typeof`.

<details>
<summary>Ver solución</summary>

```javascript
const texto = "Hola";
const numero = 42;
const booleano = true;
const indefinido = undefined;
const nulo = null;

console.log("tipo de texto:", typeof texto);         // "string"
console.log("tipo de numero:", typeof numero);       // "number"
console.log("tipo de booleano:", typeof booleano);   // "boolean"
console.log("tipo de indefinido:", typeof indefinido); // "undefined"
console.log("tipo de nulo:", typeof nulo);           // "object" (bug)
```

</details>

---

### Ejercicio 5: Conversión de tipos
**Nivel**: ⭐⭐☆☆☆

1. Crea una variable con el texto "25"
2. Conviértela a número
3. Súmale 10
4. Convierte el resultado a string
5. Muestra el tipo de dato en cada paso

<details>
<summary>Ver solución</summary>

```javascript
// Paso 1: String
let valor = "25";
console.log("Valor:", valor, "- Tipo:", typeof valor); // "25" - "string"

// Paso 2: Convertir a número
valor = Number(valor);
console.log("Valor:", valor, "- Tipo:", typeof valor); // 25 - "number"

// Paso 3: Sumar 10
valor = valor + 10;
console.log("Valor:", valor, "- Tipo:", typeof valor); // 35 - "number"

// Paso 4: Convertir a string
valor = String(valor);
console.log("Valor:", valor, "- Tipo:", typeof valor); // "35" - "string"
```

</details>

---

### Ejercicio 6: Calculadora básica
**Nivel**: ⭐⭐⭐☆☆

Declara dos variables `a` y `b` con valores numéricos. Calcula y muestra:
- Suma
- Resta
- Multiplicación
- División
- Resto

<details>
<summary>Ver solución</summary>

```javascript
const a = 20;
const b = 7;

console.log(`${a} + ${b} = ${a + b}`);   // 20 + 7 = 27
console.log(`${a} - ${b} = ${a - b}`);   // 20 - 7 = 13
console.log(`${a} × ${b} = ${a * b}`);   // 20 × 7 = 140
console.log(`${a} ÷ ${b} = ${a / b}`);   // 20 ÷ 7 = 2.857...
console.log(`${a} % ${b} = ${a % b}`);   // 20 % 7 = 6
```

</details>

---

### Ejercicio 7: Precio con IVA
**Nivel**: ⭐⭐⭐☆☆

Crea variables para:
- `precioBase`: 100
- `IVA`: 21 (porcentaje)

Calcula el precio final y muestra un mensaje: "Precio base: 100€, IVA (21%): 21€, Total: 121€"

<details>
<summary>Ver solución</summary>

```javascript
const precioBase = 100;
const IVA = 21; // Porcentaje

// Calcular el monto del IVA
const montoIVA = precioBase * (IVA / 100);

// Calcular precio final
const precioFinal = precioBase + montoIVA;

// Mostrar resultado
const mensaje = `Precio base: ${precioBase}€, IVA (${IVA}%): ${montoIVA}€, Total: ${precioFinal}€`;
console.log(mensaje);
// "Precio base: 100€, IVA (21%): 21€, Total: 121€"
```

</details>

---

## Errores comunes

### ❌ Error 1: Olvidar declarar la variable

```javascript
// MAL: No declaraste "edad"
edad = 25; // En modo estricto, esto da error

// BIEN: Declara con let o const
let edad = 25;
```

---

### ❌ Error 2: Reasignar una constante

```javascript
const nombre = "Ana";
nombre = "Juan"; // ❌ Error: Assignment to constant variable

// Solución: Usa let si necesitas cambiar el valor
let nombre = "Ana";
nombre = "Juan"; // ✅ Funciona
```

---

### ❌ Error 3: Usar `let` o `const` dos veces

```javascript
let edad = 25;
let edad = 26; // ❌ Error: Identifier 'edad' has already been declared

// Solución: Reasigna sin let
let edad = 25;
edad = 26; // ✅ Funciona
```

---

### ❌ Error 4: Operaciones con tipos incompatibles

```javascript
const nombre = "Ana";
const edad = 25;

console.log(nombre * 2); // NaN (no puedes multiplicar texto)
console.log(nombre + edad); // "Ana25" (concatenación, no suma)

// Solución: Asegúrate de que los tipos sean correctos
console.log(edad * 2); // 50
```

---

### ❌ Error 5: Confundir `=` con `==` o `===`

```javascript
let edad = 25; // ✅ Asignación (un solo =)

if (edad = 18) { // ❌ Asignación en vez de comparación
    console.log("Mayor de edad");
}

// Solución: Usa == o === para comparar (lo veremos en módulos futuros)
if (edad === 18) { // ✅ Comparación
    console.log("Tiene 18 años");
}
```

---

### ❌ Error 6: Variable no definida

```javascript
console.log(nombre); // ❌ Error: nombre is not defined

const nombre = "Ana";
console.log(nombre); // ✅ Funciona
```

**Solución**: Declara la variable antes de usarla.

---

## Buenas prácticas

### ✅ 1. Usa `const` por defecto, `let` cuando sea necesario

```javascript
// ✅ Valor que no cambia: const
const nombre = "Ana";
const PI = 3.14159;

// ✅ Valor que cambia: let
let puntos = 0;
puntos += 10;
```

**Regla**: Empieza con `const`. Si necesitas cambiar el valor, cambia a `let`.

---

### ✅ 2. Nombres descriptivos

```javascript
// ❌ Evita esto
const x = 25;
const y = 1.75;
const z = x * y;

// ✅ Mejor
const precio = 25;
const tasaImpuesto = 1.75;
const precioConImpuesto = precio * tasaImpuesto;
```

---

### ✅ 3. Una declaración por línea

```javascript
// ❌ Evita esto
let nombre = "Ana", edad = 25, ciudad = "Madrid";

// ✅ Mejor
let nombre = "Ana";
let edad = 25;
let ciudad = "Madrid";
```

**Excepción**: Variables relacionadas pueden ir juntas si son cortas.

```javascript
let x = 0, y = 0; // Coordenadas: OK
```

---

### ✅ 4. Inicializa las variables

```javascript
// ❌ Evita esto
let puntos;
// ... 50 líneas después ...
puntos += 10; // ¿Cuál era el valor inicial?

// ✅ Mejor
let puntos = 0;
// ... 50 líneas después ...
puntos += 10; // Claro: era 0, ahora es 10
```

---

### ✅ 5. Agrupa variables relacionadas

```javascript
// Datos del usuario
const nombreUsuario = "Ana";
const edadUsuario = 25;
const emailUsuario = "ana@email.com";

// Configuración
const MAX_INTENTOS = 3;
const TIMEOUT = 5000;

// Contadores
let intentos = 0;
let errores = 0;
```

---

### ✅ 6. Evita números mágicos

```javascript
// ❌ Evita esto
const total = precio * 1.21; // ¿Qué es 1.21?

// ✅ Mejor
const IVA = 1.21;
const total = precio * IVA; // Claro: es el IVA
```

---

## Cheatsheet

### Declaración de variables

```javascript
const nombre = "Ana";  // No cambia (recomendado por defecto)
let edad = 25;         // Puede cambiar
var antiguo = "no";    // ❌ No uses var
```

### Tipos primitivos

```javascript
"Hola"          // string
42              // number
true / false    // boolean
undefined       // undefined
null            // null
```

### Template literals

```javascript
const nombre = "Ana";
const edad = 25;
const mensaje = `Hola, soy ${nombre} y tengo ${edad} años`;
```

### Operador typeof

```javascript
typeof "Hola"     // "string"
typeof 42         // "number"
typeof true       // "boolean"
typeof undefined  // "undefined"
typeof null       // "object" (bug histórico)
```

### Conversiones

```javascript
// String → Number
Number("123")      // 123
parseInt("123")    // 123
parseFloat("19.99") // 19.99

// Number → String
String(123)        // "123"
(123).toString()   // "123"
`${123}`           // "123"

// Any → Boolean
Boolean(0)         // false
Boolean(1)         // true
Boolean("")        // false
Boolean("texto")   // true
```

### Nombramiento

```javascript
// ✅ camelCase para variables
const nombreCompleto = "Ana";

// ✅ SCREAMING_SNAKE_CASE para constantes globales
const MAX_INTENTOS = 3;

// ✅ Nombres descriptivos
const edad = 25; // no "x" o "a"
```

---

## Siguiente paso

Ya sabes guardar datos en variables. Ahora vamos a hacer operaciones con esos datos.

→ [15-operadores.md](15-operadores.md)

Ahí aprenderás operadores aritméticos, de comparación, lógicos y más.

# Módulo 15: Operadores en JavaScript

## Índice
1. [¿Qué son los operadores?](#qué-son-los-operadores)
2. [Operadores aritméticos](#operadores-aritméticos)
3. [Operadores de asignación](#operadores-de-asignación)
4. [Operadores de comparación](#operadores-de-comparación)
5. [Operadores lógicos](#operadores-lógicos)
6. [Operador ternario](#operador-ternario)
7. [Precedencia de operadores](#precedencia-de-operadores)
8. [Ejercicios prácticos](#ejercicios-prácticos)
9. [Errores comunes](#errores-comunes)
10. [Buenas prácticas](#buenas-prácticas)
11. [Cheatsheet](#cheatsheet)

---

## ¿Qué son los operadores?

**Un operador es un símbolo que realiza una operación sobre uno o más valores**.

### Analogía: Calculadora

```
5 + 3 = 8
│ │ │   │
│ │ │   └─ Resultado
│ │ └───── Segundo valor (operando)
│ └─────── Operador (suma)
└───────── Primer valor (operando)
```

JavaScript tiene muchos tipos de operadores:
- **Aritméticos**: `+`, `-`, `*`, `/`
- **Comparación**: `===`, `!==`, `>`, `<`
- **Lógicos**: `&&`, `||`, `!`
- **Asignación**: `=`, `+=`, `-=`
- Y más...

---

## Operadores aritméticos

Realizan operaciones matemáticas.

### Operadores básicos

```javascript
// Suma
console.log(5 + 3);    // 8
console.log(10 + 20);  // 30

// Resta
console.log(10 - 3);   // 7
console.log(5 - 10);   // -5

// Multiplicación
console.log(4 * 5);    // 20
console.log(7 * 8);    // 56

// División
console.log(20 / 4);   // 5
console.log(10 / 3);   // 3.3333...

// Resto (módulo)
console.log(10 % 3);   // 1 (10 dividido 3 = 3, resto 1)
console.log(15 % 4);   // 3 (15 dividido 4 = 3, resto 3)
console.log(20 % 5);   // 0 (división exacta, no hay resto)

// Exponenciación (potencia)
console.log(2 ** 3);   // 8 (2³ = 2 × 2 × 2)
console.log(5 ** 2);   // 25 (5² = 5 × 5)
```

---

### El operador resto (`%`) - Módulo

Muy útil para:

#### 1. Saber si un número es par o impar

```javascript
const numero = 7;

if (numero % 2 === 0) {
    console.log("Par");
} else {
    console.log("Impar"); // ← 7 es impar
}
```

#### 2. Ciclos y repeticiones

```javascript
// Ejecutar cada 3 iteraciones
for (let i = 1; i <= 10; i++) {
    if (i % 3 === 0) {
        console.log(`${i} es múltiplo de 3`);
    }
}
// Output: 3, 6, 9
```

#### 3. Obtener último dígito

```javascript
const numero = 12345;
const ultimoDigito = numero % 10;
console.log(ultimoDigito); // 5
```

---

### Incremento y decremento

```javascript
let contador = 0;

// Incremento (++ suma 1)
contador++;
console.log(contador); // 1

contador++;
console.log(contador); // 2

// Decremento (-- resta 1)
contador--;
console.log(contador); // 1
```

#### Pre-incremento vs Post-incremento

```javascript
let x = 5;

// Post-incremento: usa el valor, luego incrementa
console.log(x++); // 5 (muestra 5, luego x = 6)
console.log(x);   // 6

let y = 5;

// Pre-incremento: incrementa, luego usa el valor
console.log(++y); // 6 (primero y = 6, luego muestra 6)
console.log(y);   // 6
```

**Recomendación**: Usa `x++` o `++x` en líneas separadas para evitar confusión.

```javascript
// ✅ Claro:
let contador = 0;
contador++;
console.log(contador);

// ❌ Confuso:
let contador = 0;
console.log(contador++); // ¿Muestra 0 o 1?
```

---

### Concatenación de strings (operador `+`)

El operador `+` también une textos:

```javascript
const nombre = "Ana";
const apellido = "García";

console.log(nombre + " " + apellido); // "Ana García"

// Con números: suma
console.log(5 + 3); // 8

// Con strings: concatenación
console.log("5" + "3"); // "53"

// Mixto: convierte a string
console.log("Edad: " + 25); // "Edad: 25"
console.log(10 + " años"); // "10 años"
```

**Cuidado**: `+` se comporta diferente según el tipo.

```javascript
console.log(5 + 3);      // 8 (suma)
console.log("5" + 3);    // "53" (concatenación)
console.log(5 + "3");    // "53" (concatenación)
console.log("5" + "3");  // "53" (concatenación)
```

---

## Operadores de asignación

Asignan valores a variables.

### Asignación simple (`=`)

```javascript
let edad = 25;
let nombre = "Ana";
```

---

### Asignaciones compuestas

Combinan operación + asignación.

```javascript
let puntos = 10;

// Forma larga
puntos = puntos + 5;
console.log(puntos); // 15

// Forma corta (equivalente)
puntos += 5;
console.log(puntos); // 20
```

#### Todos los operadores de asignación compuesta

```javascript
let x = 10;

x += 5;  // x = x + 5;  → 15
x -= 3;  // x = x - 3;  → 12
x *= 2;  // x = x * 2;  → 24
x /= 4;  // x = x / 4;  → 6
x %= 4;  // x = x % 4;  → 2
x **= 3; // x = x ** 3; → 8
```

#### Ejemplos prácticos

```javascript
// Acumular puntos
let puntos = 0;
puntos += 10; // Ganó 10 puntos
puntos += 5;  // Ganó 5 puntos más
console.log(puntos); // 15

// Descontar inventario
let stock = 100;
stock -= 5;  // Vendió 5 unidades
console.log(stock); // 95

// Duplicar valor
let valor = 50;
valor *= 2;  // Duplicar
console.log(valor); // 100
```

---

## Operadores de comparación

Comparan dos valores y devuelven `true` o `false`.

### Igualdad: `==` vs `===`

```javascript
// == (igualdad con conversión de tipos)
console.log(5 == "5");   // true (convierte "5" a número)
console.log(0 == false); // true (convierte false a 0)
console.log(null == undefined); // true

// === (igualdad estricta, sin conversión)
console.log(5 === "5");   // false (tipos diferentes)
console.log(0 === false); // false (tipos diferentes)
console.log(null === undefined); // false
```

**Recomendación**: **Usa siempre `===`** (igualdad estricta).

```javascript
// ✅ Correcto
if (edad === 18) {
    console.log("Tiene exactamente 18 años");
}

// ❌ Evita esto (puede dar resultados inesperados)
if (edad == 18) {
    // ...
}
```

---

### Desigualdad: `!=` vs `!==`

```javascript
// != (desigualdad con conversión)
console.log(5 != "5");   // false

// !== (desigualdad estricta)
console.log(5 !== "5");  // true (tipos diferentes)
```

**Recomendación**: **Usa siempre `!==`** (desigualdad estricta).

---

### Mayor y menor

```javascript
console.log(10 > 5);   // true
console.log(3 > 10);   // false

console.log(10 < 20);  // true
console.log(15 < 10);  // false

console.log(10 >= 10); // true (mayor o igual)
console.log(10 <= 5);  // false
```

#### Con strings (orden alfabético)

```javascript
console.log("a" < "b");   // true
console.log("Ana" < "Juan"); // true
console.log("z" > "a");   // true

// Cuidado: Mayúsculas < Minúsculas
console.log("A" < "a");   // true
console.log("Z" < "a");   // true
```

---

### Tabla resumen de comparadores

| Operador | Significado | Ejemplo | Resultado |
|----------|-------------|---------|-----------|
| `===` | Igual (estricto) | `5 === 5` | `true` |
| `!==` | Diferente (estricto) | `5 !== "5"` | `true` |
| `>` | Mayor que | `10 > 5` | `true` |
| `<` | Menor que | `3 < 10` | `true` |
| `>=` | Mayor o igual | `10 >= 10` | `true` |
| `<=` | Menor o igual | `5 <= 3` | `false` |

---

## Operadores lógicos

Combinan múltiples condiciones.

### AND (`&&`) - Y lógico

**Devuelve `true` solo si AMBAS condiciones son verdaderas**.

```javascript
const edad = 25;
const tieneCarnet = true;

// Puede conducir si tiene 18+ Y tiene carnet
const puedeConducir = edad >= 18 && tieneCarnet;
console.log(puedeConducir); // true
```

#### Tabla de verdad

| A | B | A && B |
|---|---|--------|
| `true` | `true` | `true` |
| `true` | `false` | `false` |
| `false` | `true` | `false` |
| `false` | `false` | `false` |

#### Ejemplo práctico

```javascript
const usuario = "admin";
const password = "1234";

if (usuario === "admin" && password === "1234") {
    console.log("Acceso concedido");
} else {
    console.log("Acceso denegado");
}
```

---

### OR (`||`) - O lógico

**Devuelve `true` si AL MENOS UNA condición es verdadera**.

```javascript
const esFinDeSemana = true;
const esVacaciones = false;

// Puede descansar si es fin de semana O vacaciones
const puedeDescansar = esFinDeSemana || esVacaciones;
console.log(puedeDescansar); // true
```

#### Tabla de verdad

| A | B | A \|\| B |
|---|---|--------|
| `true` | `true` | `true` |
| `true` | `false` | `true` |
| `false` | `true` | `true` |
| `false` | `false` | `false` |

#### Ejemplo práctico

```javascript
const edad = 15;

if (edad < 12 || edad > 65) {
    console.log("Descuento especial");
} else {
    console.log("Precio regular");
}
```

---

### NOT (`!`) - Negación

**Invierte el valor booleano**.

```javascript
console.log(!true);  // false
console.log(!false); // true

const estaLloviendo = true;
console.log(!estaLloviendo); // false

const usuarioLogueado = false;
if (!usuarioLogueado) {
    console.log("Debes iniciar sesión");
}
```

#### Ejemplo práctico: verificar si algo está vacío

```javascript
const nombre = "";

if (!nombre) { // Si nombre está vacío
    console.log("El nombre es obligatorio");
}
```

---

### Combinando operadores lógicos

```javascript
const edad = 25;
const esEstudiante = true;
const tieneDescuento = false;

// Descuento si: (menor de 18 O estudiante) Y no tiene descuento ya
const aplicaDescuento = (edad < 18 || esEstudiante) && !tieneDescuento;
console.log(aplicaDescuento); // true
```

**Usa paréntesis para claridad**:

```javascript
// Sin paréntesis: difícil de leer
const resultado = a && b || c && d;

// Con paréntesis: mucho más claro
const resultado = (a && b) || (c && d);
```

---

## Operador ternario

Es una forma corta de escribir `if-else`.

### Sintaxis

```javascript
condicion ? valorSiTrue : valorSiFalse
```

### Ejemplo simple

```javascript
const edad = 20;

// Con if-else tradicional:
let mensaje;
if (edad >= 18) {
    mensaje = "Mayor de edad";
} else {
    mensaje = "Menor de edad";
}

// Con operador ternario (equivalente):
const mensaje = edad >= 18 ? "Mayor de edad" : "Menor de edad";
console.log(mensaje); // "Mayor de edad"
```

### Más ejemplos

```javascript
// Determinar si un número es par o impar
const numero = 7;
const tipo = numero % 2 === 0 ? "par" : "impar";
console.log(tipo); // "impar"

// Precio con o sin descuento
const esCliente = true;
const precio = 100;
const precioFinal = esCliente ? precio * 0.9 : precio;
console.log(precioFinal); // 90

// Saludo personalizado
const nombre = "Ana";
const saludo = nombre ? `Hola, ${nombre}` : "Hola, invitado";
console.log(saludo); // "Hola, Ana"
```

### Ternarios anidados (con moderación)

```javascript
const nota = 85;

const calificacion = nota >= 90 ? "A" :
                     nota >= 80 ? "B" :
                     nota >= 70 ? "C" :
                     nota >= 60 ? "D" : "F";

console.log(calificacion); // "B"
```

**Advertencia**: No abuses de ternarios anidados. Más de 2 niveles dificulta la lectura.

---

## Precedencia de operadores

Cuando combinas operadores, JavaScript sigue un orden específico.

### Orden de precedencia (de mayor a menor)

1. **Paréntesis** `()`
2. **Exponenciación** `**`
3. **Multiplicación, División, Resto** `*`, `/`, `%`
4. **Suma, Resta** `+`, `-`
5. **Comparación** `<`, `>`, `<=`, `>=`
6. **Igualdad** `===`, `!==`
7. **AND lógico** `&&`
8. **OR lógico** `||`
9. **Ternario** `? :`
10. **Asignación** `=`, `+=`, `-=`, etc.

### Ejemplos

```javascript
// Multiplicación antes que suma
console.log(2 + 3 * 4); // 14 (no 20)
// 3 * 4 = 12, luego 2 + 12 = 14

// Con paréntesis cambias el orden
console.log((2 + 3) * 4); // 20
// 2 + 3 = 5, luego 5 * 4 = 20

// Comparación y lógicos
console.log(10 > 5 && 3 < 7); // true
// 10 > 5 → true, 3 < 7 → true, true && true → true

// Asignación al final
let x = 5 + 3 * 2; // x = 11 (no x = 16)
```

### Regla práctica: Usa paréntesis

```javascript
// ❌ Difícil de leer
const resultado = a + b * c / d - e;

// ✅ Más claro con paréntesis
const resultado = a + ((b * c) / d) - e;
```

Aunque sepas las reglas, los paréntesis hacen el código más legible.

---

## Ejercicios prácticos

### Ejercicio 1: Calculadora básica
**Nivel**: ⭐☆☆☆☆

Crea variables `a = 15` y `b = 4`. Muestra el resultado de:
- Suma
- Resta
- Multiplicación
- División
- Resto

<details>
<summary>Ver solución</summary>

```javascript
const a = 15;
const b = 4;

console.log("Suma:", a + b);           // 19
console.log("Resta:", a - b);          // 11
console.log("Multiplicación:", a * b); // 60
console.log("División:", a / b);       // 3.75
console.log("Resto:", a % b);          // 3
```

</details>

---

### Ejercicio 2: Par o impar
**Nivel**: ⭐⭐☆☆☆

Crea una variable `numero` y determina si es par o impar usando el operador resto. Usa el operador ternario para asignar "par" o "impar" a una variable.

<details>
<summary>Ver solución</summary>

```javascript
const numero = 17;
const tipo = numero % 2 === 0 ? "par" : "impar";
console.log(`${numero} es ${tipo}`); // "17 es impar"

// Probando con otro número
const numero2 = 20;
const tipo2 = numero2 % 2 === 0 ? "par" : "impar";
console.log(`${numero2} es ${tipo2}`); // "20 es par"
```

</details>

---

### Ejercicio 3: Incremento y decremento
**Nivel**: ⭐☆☆☆☆

Crea una variable `contador = 0`. Increméntala 5 veces y muestra el valor. Luego decrémentala 2 veces y muestra el valor final.

<details>
<summary>Ver solución</summary>

```javascript
let contador = 0;

contador++;
contador++;
contador++;
contador++;
contador++;
console.log("Después de 5 incrementos:", contador); // 5

contador--;
contador--;
console.log("Después de 2 decrementos:", contador); // 3
```

</details>

---

### Ejercicio 4: Comparaciones
**Nivel**: ⭐⭐☆☆☆

Crea variables `edad = 25` y `edadMinima = 18`. Usa operadores de comparación para determinar:
- Si edad es mayor o igual a edadMinima
- Si edad es exactamente 25
- Si edad es diferente de 30

<details>
<summary>Ver solución</summary>

```javascript
const edad = 25;
const edadMinima = 18;

console.log("¿Es mayor de edad?", edad >= edadMinima);    // true
console.log("¿Tiene exactamente 25?", edad === 25);       // true
console.log("¿Es diferente de 30?", edad !== 30);         // true
```

</details>

---

### Ejercicio 5: Operadores lógicos
**Nivel**: ⭐⭐⭐☆☆

Determina si un usuario puede acceder a contenido premium. Las condiciones son:
- Debe ser mayor de 18 años
- Y debe tener suscripción activa O ser usuario VIP

Prueba con diferentes combinaciones.

<details>
<summary>Ver solución</summary>

```javascript
// Caso 1: Usuario válido con suscripción
const edad1 = 25;
const suscripcionActiva1 = true;
const esVIP1 = false;

const puedeAcceder1 = edad1 >= 18 && (suscripcionActiva1 || esVIP1);
console.log("Caso 1:", puedeAcceder1); // true

// Caso 2: Usuario válido VIP sin suscripción
const edad2 = 30;
const suscripcionActiva2 = false;
const esVIP2 = true;

const puedeAcceder2 = edad2 >= 18 && (suscripcionActiva2 || esVIP2);
console.log("Caso 2:", puedeAcceder2); // true

// Caso 3: Menor de edad con suscripción
const edad3 = 15;
const suscripcionActiva3 = true;
const esVIP3 = false;

const puedeAcceder3 = edad3 >= 18 && (suscripcionActiva3 || esVIP3);
console.log("Caso 3:", puedeAcceder3); // false

// Caso 4: Mayor de edad sin suscripción ni VIP
const edad4 = 20;
const suscripcionActiva4 = false;
const esVIP4 = false;

const puedeAcceder4 = edad4 >= 18 && (suscripcionActiva4 || esVIP4);
console.log("Caso 4:", puedeAcceder4); // false
```

</details>

---

### Ejercicio 6: Calculadora de descuento
**Nivel**: ⭐⭐⭐☆☆

Crea un sistema de descuentos:
- Precio base: 100€
- Si es estudiante: 20% de descuento
- Si es mayor de 65: 15% de descuento
- Si es estudiante Y mayor de 65: 30% de descuento

Usa operadores lógicos y calcula el precio final.

<details>
<summary>Ver solución</summary>

```javascript
const precioBase = 100;
const esEstudiante = true;
const edad = 70;
const esMayor65 = edad > 65;

let descuento = 0;

if (esEstudiante && esMayor65) {
    descuento = 0.30; // 30%
} else if (esEstudiante) {
    descuento = 0.20; // 20%
} else if (esMayor65) {
    descuento = 0.15; // 15%
}

const precioFinal = precioBase * (1 - descuento);

console.log(`Precio base: ${precioBase}€`);
console.log(`Descuento: ${descuento * 100}%`);
console.log(`Precio final: ${precioFinal}€`);

// Output:
// Precio base: 100€
// Descuento: 30%
// Precio final: 70€
```

</details>

---

### Ejercicio 7: Operador ternario anidado
**Nivel**: ⭐⭐⭐☆☆

Clasifica la temperatura:
- Menos de 0: "Congelando"
- 0 a 15: "Frío"
- 16 a 25: "Templado"
- Más de 25: "Calor"

Usa operador ternario anidado.

<details>
<summary>Ver solución</summary>

```javascript
const temperatura = 18;

const clima = temperatura < 0 ? "Congelando" :
              temperatura <= 15 ? "Frío" :
              temperatura <= 25 ? "Templado" : "Calor";

console.log(`${temperatura}°C → ${clima}`);
// "18°C → Templado"

// Prueba con otras temperaturas:
console.log(-5, "→", -5 < 0 ? "Congelando" : -5 <= 15 ? "Frío" : -5 <= 25 ? "Templado" : "Calor");  // Congelando
console.log(10, "→", 10 < 0 ? "Congelando" : 10 <= 15 ? "Frío" : 10 <= 25 ? "Templado" : "Calor");  // Frío
console.log(30, "→", 30 < 0 ? "Congelando" : 30 <= 15 ? "Frío" : 30 <= 25 ? "Templado" : "Calor");  // Calor
```

</details>

---

## Errores comunes

### ❌ Error 1: Confundir `=` con `===`

```javascript
let edad = 18;

// MAL: Asignación en vez de comparación
if (edad = 25) { // Asigna 25 a edad (siempre true)
    console.log("Esto siempre se ejecuta");
}

// BIEN: Comparación
if (edad === 25) { // Compara
    console.log("Tiene 25 años");
}
```

---

### ❌ Error 2: Usar `==` en vez de `===`

```javascript
// MAL: Conversión de tipos inesperada
console.log(0 == false);  // true (no quieres esto)
console.log("" == false); // true
console.log(null == undefined); // true

// BIEN: Comparación estricta
console.log(0 === false);  // false
console.log("" === false); // false
console.log(null === undefined); // false
```

---

### ❌ Error 3: Orden de operaciones

```javascript
// MAL: Resultado inesperado
const resultado = 2 + 3 * 4; // 14, no 20

// BIEN: Usa paréntesis
const resultado = (2 + 3) * 4; // 20
```

---

### ❌ Error 4: Concatenación inesperada

```javascript
console.log("10" + 5);  // "105" (concatenación, no suma)
console.log("10" - 5);  // 5 (conversión a número)

// Solución: Asegúrate de que sean números
console.log(Number("10") + 5); // 15
console.log(parseInt("10") + 5); // 15
```

---

### ❌ Error 5: Dividir por cero

```javascript
console.log(10 / 0);  // Infinity (no error, pero no útil)
console.log(0 / 0);   // NaN

// Solución: Valida antes de dividir
const a = 10;
const b = 0;

if (b !== 0) {
    console.log(a / b);
} else {
    console.log("No se puede dividir por cero");
}
```

---

### ❌ Error 6: Negación confusa

```javascript
const activo = true;

// Confuso: doble negación
if (!!activo) { // ...
}

// Mejor: directo
if (activo) { // ...
}
```

---

## Buenas prácticas

### ✅ 1. Usa `===` y `!==` siempre

```javascript
// ❌ Evita == y !=
if (edad == 18) { }

// ✅ Usa === y !==
if (edad === 18) { }
```

---

### ✅ 2. Usa paréntesis para claridad

```javascript
// ❌ Difícil de leer
const resultado = a && b || c && d;

// ✅ Más claro
const resultado = (a && b) || (c && d);
```

---

### ✅ 3. Operador ternario solo para casos simples

```javascript
// ✅ OK: Caso simple
const mensaje = edad >= 18 ? "Adulto" : "Menor";

// ❌ Evita: Demasiado complejo
const mensaje = edad < 12 ? "Niño" :
                edad < 18 ? "Adolescente" :
                edad < 65 ? "Adulto" :
                edad < 80 ? "Adulto mayor" : "Anciano";

// ✅ Mejor: Usa if-else para casos complejos
let mensaje;
if (edad < 12) {
    mensaje = "Niño";
} else if (edad < 18) {
    mensaje = "Adolescente";
} else if (edad < 65) {
    mensaje = "Adulto";
} else {
    mensaje = "Adulto mayor";
}
```

---

### ✅ 4. Agrupa condiciones relacionadas

```javascript
// ❌ Difícil de leer
if (edad >= 18 && edad <= 65 && tieneExperiencia && tieneCarnet) {
    // ...
}

// ✅ Más claro
const edadValida = edad >= 18 && edad <= 65;
const cumpleRequisitos = tieneExperiencia && tieneCarnet;

if (edadValida && cumpleRequisitos) {
    // ...
}
```

---

### ✅ 5. Evita números mágicos

```javascript
// ❌ ¿Qué significa 18?
if (edad >= 18) {
    // ...
}

// ✅ Usa constantes con nombres descriptivos
const EDAD_MINIMA = 18;
if (edad >= EDAD_MINIMA) {
    // ...
}
```

---

## Cheatsheet

### Operadores aritméticos

```javascript
5 + 3    // 8 (suma)
10 - 4   // 6 (resta)
3 * 7    // 21 (multiplicación)
20 / 5   // 4 (división)
10 % 3   // 1 (resto)
2 ** 3   // 8 (potencia: 2³)
x++      // Incremento
x--      // Decremento
```

### Operadores de asignación

```javascript
x = 5    // Asignación
x += 3   // x = x + 3
x -= 2   // x = x - 2
x *= 4   // x = x * 4
x /= 2   // x = x / 2
x %= 3   // x = x % 3
```

### Operadores de comparación

```javascript
===      // Igual (estricto)
!==      // Diferente (estricto)
>        // Mayor que
<        // Menor que
>=       // Mayor o igual
<=       // Menor o igual
```

### Operadores lógicos

```javascript
&&       // AND (y)
||       // OR (o)
!        // NOT (negación)
```

### Operador ternario

```javascript
condicion ? valorTrue : valorFalse
```

### Precedencia (mayor a menor)

```javascript
()       // Paréntesis
**       // Exponenciación
* / %    // Multiplicación, división, resto
+ -      // Suma, resta
< > <= >= // Comparación
=== !==  // Igualdad
&&       // AND lógico
||       // OR lógico
? :      // Ternario
= += -=  // Asignación
```

---

## Siguiente paso

Ya sabes hacer operaciones y comparaciones. Ahora vamos a tomar **decisiones** en el código.

→ [16-condicionales.md](16-condicionales.md)

Ahí aprenderás `if`, `else`, `else if`, `switch` y cómo hacer que tu programa tome decisiones.

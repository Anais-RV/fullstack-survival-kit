# Módulo 16: Condicionales en JavaScript

## Índice
1. [¿Qué son los condicionales?](#qué-son-los-condicionales)
2. [if - La decisión básica](#if---la-decisión-básica)
3. [else - La alternativa](#else---la-alternativa)
4. [else if - Múltiples condiciones](#else-if---múltiples-condiciones)
5. [switch - Múltiples casos](#switch---múltiples-casos)
6. [Valores truthy y falsy](#valores-truthy-y-falsy)
7. [Condiciones complejas](#condiciones-complejas)
8. [Ejercicios prácticos](#ejercicios-prácticos)
9. [Errores comunes](#errores-comunes)
10. [Buenas prácticas](#buenas-prácticas)
11. [Cheatsheet](#cheatsheet)

---

## ¿Qué son los condicionales?

**Los condicionales permiten que tu código tome decisiones**.

### Analogía: El semáforo

```
¿El semáforo está en verde?
├─ SÍ → Avanza
└─ NO → Detente
```

En JavaScript:

```javascript
const semaforoVerde = true;

if (semaforoVerde) {
    console.log("Avanza");
} else {
    console.log("Detente");
}
```

### ¿Para qué sirven?

Los condicionales hacen que tu programa se comporte diferente según la situación:

- Mostrar contenido solo a usuarios logueados
- Validar formularios antes de enviarlos
- Mostrar descuentos según la edad
- Cambiar el diseño según el dispositivo
- Y mucho más...

---

## if - La decisión básica

### Sintaxis

```javascript
if (condicion) {
    // Código que se ejecuta si la condición es true
}
```

### Ejemplo simple

```javascript
const edad = 18;

if (edad >= 18) {
    console.log("Eres mayor de edad");
}
// Si edad >= 18, muestra el mensaje
// Si edad < 18, no pasa nada
```

### Más ejemplos

```javascript
// Verificar si hace frío
const temperatura = 10;

if (temperatura < 15) {
    console.log("Ponte un abrigo");
}

// Verificar si hay stock
const stock = 5;

if (stock > 0) {
    console.log("Producto disponible");
}

// Verificar login
const usuarioLogueado = true;

if (usuarioLogueado) {
    console.log("Bienvenido de nuevo");
}
```

### Con bloques de código

```javascript
const esMayorDeEdad = true;

if (esMayorDeEdad) {
    console.log("Puedes votar");
    console.log("Puedes conducir");
    console.log("Puedes trabajar");
}
```

**Importante**: Usa llaves `{}` aunque sea una sola línea. Es más seguro y legible.

```javascript
// ❌ Funciona pero no recomendado
if (edad >= 18)
    console.log("Mayor de edad");

// ✅ Mejor: siempre con llaves
if (edad >= 18) {
    console.log("Mayor de edad");
}
```

---

## else - La alternativa

**`else` se ejecuta cuando la condición del `if` es `false`**.

### Sintaxis

```javascript
if (condicion) {
    // Se ejecuta si condicion es true
} else {
    // Se ejecuta si condicion es false
}
```

### Ejemplo simple

```javascript
const edad = 15;

if (edad >= 18) {
    console.log("Eres mayor de edad");
} else {
    console.log("Eres menor de edad");
}
// Output: "Eres menor de edad"
```

### Más ejemplos

```javascript
// Login
const passwordCorrecta = false;

if (passwordCorrecta) {
    console.log("Acceso concedido");
} else {
    console.log("Acceso denegado");
}

// Stock
const hayStock = false;

if (hayStock) {
    console.log("Añadido al carrito");
} else {
    console.log("Producto agotado");
}

// Día de la semana
const esDomingo = true;

if (esDomingo) {
    console.log("La tienda está cerrada");
} else {
    console.log("La tienda está abierta");
}
```

### Diagrama de flujo

```
       ┌────────────┐
       │ Condición  │
       └─────┬──────┘
             │
      ┌──────┴──────┐
      │             │
   true           false
      │             │
      ▼             ▼
  ┌───────┐    ┌───────┐
  │  if   │    │ else  │
  └───────┘    └───────┘
```

---

## else if - Múltiples condiciones

**`else if` permite verificar múltiples condiciones en orden**.

### Sintaxis

```javascript
if (condicion1) {
    // Se ejecuta si condicion1 es true
} else if (condicion2) {
    // Se ejecuta si condicion1 es false y condicion2 es true
} else if (condicion3) {
    // Se ejecuta si condicion1 y condicion2 son false y condicion3 es true
} else {
    // Se ejecuta si todas son false
}
```

### Ejemplo: Calificaciones

```javascript
const nota = 85;

if (nota >= 90) {
    console.log("Calificación: A - Excelente");
} else if (nota >= 80) {
    console.log("Calificación: B - Muy bien");
} else if (nota >= 70) {
    console.log("Calificación: C - Bien");
} else if (nota >= 60) {
    console.log("Calificación: D - Suficiente");
} else {
    console.log("Calificación: F - Reprobado");
}
// Output: "Calificación: B - Muy bien"
```

### Ejemplo: Descuentos por edad

```javascript
const edad = 70;
let descuento = 0;

if (edad < 12) {
    descuento = 50; // 50% para niños
    console.log("Descuento infantil: 50%");
} else if (edad >= 65) {
    descuento = 30; // 30% para adultos mayores
    console.log("Descuento tercera edad: 30%");
} else if (edad < 18) {
    descuento = 20; // 20% para adolescentes
    console.log("Descuento adolescente: 20%");
} else {
    console.log("Sin descuento");
}
```

### Ejemplo: Clasificación de temperatura

```javascript
const temperatura = 22;

if (temperatura < 0) {
    console.log("Congelando 🥶");
} else if (temperatura < 10) {
    console.log("Muy frío ❄️");
} else if (temperatura < 20) {
    console.log("Frío 😊");
} else if (temperatura < 28) {
    console.log("Templado ☀️");
} else if (temperatura < 35) {
    console.log("Calor 🔥");
} else {
    console.log("Mucho calor 🌡️");
}
// Output: "Templado ☀️"
```

### Orden importa

```javascript
const numero = 15;

// ✅ CORRECTO: De más específico a menos específico
if (numero > 20) {
    console.log("Mayor que 20");
} else if (numero > 10) {
    console.log("Mayor que 10"); // ← Se ejecuta esto
} else if (numero > 5) {
    console.log("Mayor que 5");
}

// ❌ MAL: Nunca llegará a las demás condiciones
if (numero > 5) {
    console.log("Mayor que 5"); // ← Se ejecuta esto y ya no evalúa las demás
} else if (numero > 10) {
    console.log("Mayor que 10"); // Nunca se ejecuta
} else if (numero > 20) {
    console.log("Mayor que 20"); // Nunca se ejecuta
}
```

---

## switch - Múltiples casos

**`switch` es útil cuando tienes muchos casos específicos para comparar**.

### Sintaxis

```javascript
switch (expresion) {
    case valor1:
        // Código si expresion === valor1
        break;
    case valor2:
        // Código si expresion === valor2
        break;
    case valor3:
        // Código si expresion === valor3
        break;
    default:
        // Código si ningún caso coincide
}
```

### Ejemplo: Días de la semana

```javascript
const dia = 3;

switch (dia) {
    case 1:
        console.log("Lunes");
        break;
    case 2:
        console.log("Martes");
        break;
    case 3:
        console.log("Miércoles");
        break;
    case 4:
        console.log("Jueves");
        break;
    case 5:
        console.log("Viernes");
        break;
    case 6:
        console.log("Sábado");
        break;
    case 7:
        console.log("Domingo");
        break;
    default:
        console.log("Día no válido");
}
// Output: "Miércoles"
```

### Ejemplo: Operaciones matemáticas

```javascript
const operacion = "+";
const a = 10;
const b = 5;
let resultado;

switch (operacion) {
    case "+":
        resultado = a + b;
        console.log(`${a} + ${b} = ${resultado}`);
        break;
    case "-":
        resultado = a - b;
        console.log(`${a} - ${b} = ${resultado}`);
        break;
    case "*":
        resultado = a * b;
        console.log(`${a} × ${b} = ${resultado}`);
        break;
    case "/":
        resultado = a / b;
        console.log(`${a} ÷ ${b} = ${resultado}`);
        break;
    default:
        console.log("Operación no válida");
}
// Output: "10 + 5 = 15"
```

### Importante: El `break`

**Sin `break`, el código continúa ejecutándose en los casos siguientes**:

```javascript
const opcion = 2;

// ❌ Sin break (fall-through):
switch (opcion) {
    case 1:
        console.log("Opción 1");
    case 2:
        console.log("Opción 2"); // Se ejecuta
    case 3:
        console.log("Opción 3"); // También se ejecuta
    default:
        console.log("Default");  // También se ejecuta
}
// Output:
// "Opción 2"
// "Opción 3"
// "Default"

// ✅ Con break:
switch (opcion) {
    case 1:
        console.log("Opción 1");
        break;
    case 2:
        console.log("Opción 2");
        break; // ← Detiene la ejecución
    case 3:
        console.log("Opción 3");
        break;
    default:
        console.log("Default");
}
// Output: "Opción 2"
```

### Casos agrupados (sin break intencional)

A veces **quieres** que varios casos ejecuten el mismo código:

```javascript
const dia = "sábado";

switch (dia) {
    case "lunes":
    case "martes":
    case "miércoles":
    case "jueves":
    case "viernes":
        console.log("Día laboral");
        break;
    case "sábado":
    case "domingo":
        console.log("Fin de semana 🎉");
        break;
    default:
        console.log("Día no válido");
}
// Output: "Fin de semana 🎉"
```

### switch vs if-else

```javascript
// Con switch (mejor para valores específicos):
switch (dia) {
    case 1:
        console.log("Lunes");
        break;
    case 2:
        console.log("Martes");
        break;
    // ...
}

// Con if-else (mejor para rangos):
if (edad < 18) {
    console.log("Menor");
} else if (edad < 65) {
    console.log("Adulto");
} else {
    console.log("Mayor");
}
```

**Usa `switch` cuando**:
- Comparas un valor con varios casos específicos
- Tienes muchos casos (5+)
- Los casos son valores concretos (no rangos)

**Usa `if-else` cuando**:
- Trabajas con rangos (`<`, `>`, `<=`, `>=`)
- Tienes pocas condiciones
- Las condiciones son complejas (`&&`, `||`)

---

## Valores truthy y falsy

JavaScript convierte valores a booleanos en contextos condicionales.

### Valores "falsy" (se convierten a false)

```javascript
if (false)      console.log("NO se ejecuta");
if (0)          console.log("NO se ejecuta");
if (-0)         console.log("NO se ejecuta");
if (0n)         console.log("NO se ejecuta"); // BigInt cero
if ("")         console.log("NO se ejecuta"); // String vacío
if (null)       console.log("NO se ejecuta");
if (undefined)  console.log("NO se ejecuta");
if (NaN)        console.log("NO se ejecuta");
```

**Solo 8 valores son falsy, todo lo demás es truthy**.

### Valores "truthy" (se convierten a true)

```javascript
if (true)       console.log("Sí se ejecuta");
if (1)          console.log("Sí se ejecuta");
if (-1)         console.log("Sí se ejecuta");
if ("hola")     console.log("Sí se ejecuta");
if ("0")        console.log("Sí se ejecuta"); // String "0"
if (" ")        console.log("Sí se ejecuta"); // Espacio
if ([])         console.log("Sí se ejecuta"); // Array vacío
if ({})         console.log("Sí se ejecuta"); // Objeto vacío
if (function(){}) console.log("Sí se ejecuta"); // Función
```

### Ejemplos prácticos

#### Validar que un campo no esté vacío

```javascript
const nombre = "";

if (nombre) {
    console.log(`Hola, ${nombre}`);
} else {
    console.log("El nombre es obligatorio");
}
// Output: "El nombre es obligatorio"
```

#### Verificar si existe un valor

```javascript
let usuario; // undefined

if (usuario) {
    console.log("Usuario encontrado");
} else {
    console.log("No hay usuario");
}
// Output: "No hay usuario"
```

#### Validar número diferente de cero

```javascript
const cantidad = 0;

if (cantidad) {
    console.log(`Tienes ${cantidad} productos`);
} else {
    console.log("Tu carrito está vacío");
}
// Output: "Tu carrito está vacío"
```

### Cuidado con las trampas

```javascript
// ❌ String "0" es truthy (no es lo mismo que número 0)
if ("0") {
    console.log("Esto SÍ se ejecuta"); // ← Sorpresa
}

// ❌ Array vacío es truthy
if ([]) {
    console.log("Esto SÍ se ejecuta"); // ← También sorpresa
}

// ✅ Mejor: sé explícito
const texto = "0";
if (texto !== "") {
    console.log("Hay texto");
}

const array = [];
if (array.length > 0) {
    console.log("El array tiene elementos");
}
```

---

## Condiciones complejas

### Combinando con AND (`&&`)

```javascript
const edad = 25;
const tieneLicencia = true;

if (edad >= 18 && tieneLicencia) {
    console.log("Puede conducir");
} else {
    console.log("No puede conducir");
}
```

### Combinando con OR (`||`)

```javascript
const esAdmin = false;
const esModerador = true;

if (esAdmin || esModerador) {
    console.log("Tiene permisos de gestión");
} else {
    console.log("Usuario normal");
}
```

### Combinando AND y OR

```javascript
const edad = 30;
const esEstudiante = false;
const esMayorDe65 = false;

// Descuento si: (menor de 18 O estudiante) O mayor de 65
if (edad < 18 || esEstudiante || esMayorDe65) {
    console.log("Aplica descuento");
} else {
    console.log("Precio completo");
}
```

### Uso de paréntesis para claridad

```javascript
const edad = 25;
const esEstudiante = true;
const esVIP = false;

// Sin paréntesis: difícil de leer
if (edad >= 18 && esEstudiante || esVIP) {
    // ¿Cuál es la precedencia?
}

// Con paréntesis: mucho más claro
if ((edad >= 18 && esEstudiante) || esVIP) {
    console.log("Acceso concedido");
}
```

### Negación con NOT (`!`)

```javascript
const usuarioLogueado = false;

if (!usuarioLogueado) {
    console.log("Debes iniciar sesión");
}

// Equivalente a:
if (usuarioLogueado === false) {
    console.log("Debes iniciar sesión");
}
```

### Condiciones anidadas

```javascript
const esUsuario = true;
const password = "correcta";
const cuenta2FA = true;
const codigo2FA = "123456";

if (esUsuario) {
    if (password === "correcta") {
        if (cuenta2FA) {
            if (codigo2FA === "123456") {
                console.log("Acceso completo concedido");
            } else {
                console.log("Código 2FA incorrecto");
            }
        } else {
            console.log("Acceso concedido sin 2FA");
        }
    } else {
        console.log("Contraseña incorrecta");
    }
} else {
    console.log("Usuario no encontrado");
}
```

**Mejor**: Simplifica con condiciones combinadas o "early returns":

```javascript
const esUsuario = true;
const password = "correcta";
const cuenta2FA = true;
const codigo2FA = "123456";

// Salidas tempranas (early exits)
if (!esUsuario) {
    console.log("Usuario no encontrado");
} else if (password !== "correcta") {
    console.log("Contraseña incorrecta");
} else if (cuenta2FA && codigo2FA !== "123456") {
    console.log("Código 2FA incorrecto");
} else {
    console.log("Acceso concedido");
}
```

---

## Ejercicios prácticos

### Ejercicio 1: Mayor de edad
**Nivel**: ⭐☆☆☆☆

Crea una variable `edad` y muestra "Mayor de edad" si tiene 18 o más, o "Menor de edad" si tiene menos.

<details>
<summary>Ver solución</summary>

```javascript
const edad = 17;

if (edad >= 18) {
    console.log("Mayor de edad");
} else {
    console.log("Menor de edad");
}
// Output: "Menor de edad"
```

</details>

---

### Ejercicio 2: Calificación de nota
**Nivel**: ⭐⭐☆☆☆

Clasifica una nota en:
- 90-100: "Excelente"
- 80-89: "Muy bien"
- 70-79: "Bien"
- 60-69: "Suficiente"
- 0-59: "Reprobado"

<details>
<summary>Ver solución</summary>

```javascript
const nota = 75;

if (nota >= 90 && nota <= 100) {
    console.log("Excelente");
} else if (nota >= 80) {
    console.log("Muy bien");
} else if (nota >= 70) {
    console.log("Bien");
} else if (nota >= 60) {
    console.log("Suficiente");
} else if (nota >= 0) {
    console.log("Reprobado");
} else {
    console.log("Nota no válida");
}
// Output: "Bien"
```

</details>

---

### Ejercicio 3: Día de la semana con switch
**Nivel**: ⭐⭐☆☆☆

Crea un switch que tome un número del 1 al 7 y muestre el día de la semana correspondiente.

<details>
<summary>Ver solución</summary>

```javascript
const numeroDia = 5;

switch (numeroDia) {
    case 1:
        console.log("Lunes");
        break;
    case 2:
        console.log("Martes");
        break;
    case 3:
        console.log("Miércoles");
        break;
    case 4:
        console.log("Jueves");
        break;
    case 5:
        console.log("Viernes");
        break;
    case 6:
        console.log("Sábado");
        break;
    case 7:
        console.log("Domingo");
        break;
    default:
        console.log("Número no válido (debe ser 1-7)");
}
// Output: "Viernes"
```

</details>

---

### Ejercicio 4: Calculadora simple
**Nivel**: ⭐⭐⭐☆☆

Crea una calculadora que tome dos números y una operación (+, -, *, /) y muestre el resultado. Usa switch para las operaciones.

<details>
<summary>Ver solución</summary>

```javascript
const num1 = 20;
const num2 = 5;
const operacion = "/";

switch (operacion) {
    case "+":
        console.log(`${num1} + ${num2} = ${num1 + num2}`);
        break;
    case "-":
        console.log(`${num1} - ${num2} = ${num1 - num2}`);
        break;
    case "*":
        console.log(`${num1} × ${num2} = ${num1 * num2}`);
        break;
    case "/":
        if (num2 !== 0) {
            console.log(`${num1} ÷ ${num2} = ${num1 / num2}`);
        } else {
            console.log("Error: No se puede dividir por cero");
        }
        break;
    default:
        console.log("Operación no válida");
}
// Output: "20 ÷ 5 = 4"
```

</details>

---

### Ejercicio 5: Validación de formulario
**Nivel**: ⭐⭐⭐☆☆

Valida un formulario de registro que debe cumplir:
- El nombre no debe estar vacío
- La edad debe ser 18 o mayor
- El email debe contener "@"

Muestra mensajes específicos para cada error.

<details>
<summary>Ver solución</summary>

```javascript
const nombre = "Ana";
const edad = 25;
const email = "ana@email.com";

let formularioValido = true;

if (!nombre) {
    console.log("❌ El nombre es obligatorio");
    formularioValido = false;
}

if (edad < 18) {
    console.log("❌ Debes ser mayor de 18 años");
    formularioValido = false;
}

if (!email.includes("@")) {
    console.log("❌ El email debe contener @");
    formularioValido = false;
}

if (formularioValido) {
    console.log("✅ Formulario válido. Registrando usuario...");
}
// Output: "✅ Formulario válido. Registrando usuario..."
```

</details>

---

### Ejercicio 6: Sistema de descuentos
**Nivel**: ⭐⭐⭐⭐☆

Crea un sistema de descuentos:
- Menores de 12: 50% descuento
- Entre 12 y 17: 30% descuento
- Estudiantes (cualquier edad): 20% adicional
- Mayores de 65: 40% descuento
- VIPs: 15% adicional

Calcula el precio final de un producto de 100€ para diferentes casos.

<details>
<summary>Ver solución</summary>

```javascript
const precioBase = 100;
const edad = 25;
const esEstudiante = true;
const esVIP = true;

let descuento = 0;

// Descuento por edad
if (edad < 12) {
    descuento = 0.50; // 50%
    console.log("Descuento infantil: 50%");
} else if (edad >= 12 && edad < 18) {
    descuento = 0.30; // 30%
    console.log("Descuento adolescente: 30%");
} else if (edad >= 65) {
    descuento = 0.40; // 40%
    console.log("Descuento tercera edad: 40%");
}

// Descuento adicional por estudiante
if (esEstudiante) {
    descuento += 0.20; // +20%
    console.log("Descuento estudiante: +20%");
}

// Descuento adicional por VIP
if (esVIP) {
    descuento += 0.15; // +15%
    console.log("Descuento VIP: +15%");
}

// Limitar descuento máximo al 90%
if (descuento > 0.90) {
    descuento = 0.90;
}

const precioFinal = precioBase * (1 - descuento);
const totalDescuento = descuento * 100;

console.log(`\nPrecio base: ${precioBase}€`);
console.log(`Descuento total: ${totalDescuento}%`);
console.log(`Precio final: ${precioFinal.toFixed(2)}€`);

// Output:
// Descuento estudiante: +20%
// Descuento VIP: +15%
//
// Precio base: 100€
// Descuento total: 35%
// Precio final: 65.00€
```

</details>

---

### Ejercicio 7: Juego de adivinanzas
**Nivel**: ⭐⭐⭐⭐☆

Crea un juego donde el usuario intenta adivinar un número del 1 al 10. Da pistas de "muy bajo", "bajo", "cerca", "alto", "muy alto".

<details>
<summary>Ver solución</summary>

```javascript
const numeroSecreto = 7;
const intento = 4;

if (intento === numeroSecreto) {
    console.log("🎉 ¡Correcto! Adivinaste el número");
} else if (intento < numeroSecreto) {
    const diferencia = numeroSecreto - intento;
    
    if (diferencia === 1) {
        console.log("🔥 ¡Muy cerca! El número es un poco más alto");
    } else if (diferencia <= 2) {
        console.log("📈 Cerca, pero el número es más alto");
    } else if (diferencia <= 4) {
        console.log("⬆️ El número es bastante más alto");
    } else {
        console.log("⬆️⬆️ El número es mucho más alto");
    }
} else {
    const diferencia = intento - numeroSecreto;
    
    if (diferencia === 1) {
        console.log("🔥 ¡Muy cerca! El número es un poco más bajo");
    } else if (diferencia <= 2) {
        console.log("📉 Cerca, pero el número es más bajo");
    } else if (diferencia <= 4) {
        console.log("⬇️ El número es bastante más bajo");
    } else {
        console.log("⬇️⬇️ El número es mucho más bajo");
    }
}

// Output: "📈 Cerca, pero el número es más alto"
```

</details>

---

## Errores comunes

### ❌ Error 1: Olvidar las llaves

```javascript
// MAL: Sin llaves puede causar problemas
if (edad >= 18)
    console.log("Mayor de edad");
    console.log("Puede votar"); // ← Siempre se ejecuta

// BIEN: Con llaves es más claro
if (edad >= 18) {
    console.log("Mayor de edad");
    console.log("Puede votar");
}
```

---

### ❌ Error 2: Usar `=` en vez de `===`

```javascript
const edad = 18;

// MAL: Asignación en vez de comparación
if (edad = 25) { // Asigna 25 a edad
    console.log("Esto siempre se ejecuta");
}

// BIEN: Comparación
if (edad === 25) {
    console.log("Tiene 25 años");
}
```

---

### ❌ Error 3: Olvidar el `break` en switch

```javascript
const opcion = 2;

// MAL: Sin break
switch (opcion) {
    case 1:
        console.log("Uno");
    case 2:
        console.log("Dos");   // Se ejecuta
    case 3:
        console.log("Tres");  // También se ejecuta
}

// BIEN: Con break
switch (opcion) {
    case 1:
        console.log("Uno");
        break;
    case 2:
        console.log("Dos");
        break; // ← Detiene aquí
    case 3:
        console.log("Tres");
        break;
}
```

---

### ❌ Error 4: Orden incorrecto en if-else

```javascript
const numero = 15;

// MAL: La primera condición atrapa todo
if (numero > 5) {
    console.log("Mayor que 5"); // ← Se ejecuta esto
} else if (numero > 10) {
    console.log("Mayor que 10"); // Nunca llega aquí
} else if (numero > 20) {
    console.log("Mayor que 20"); // Nunca llega aquí
}

// BIEN: De más específico a menos específico
if (numero > 20) {
    console.log("Mayor que 20");
} else if (numero > 10) {
    console.log("Mayor que 10"); // ← Se ejecuta esto
} else if (numero > 5) {
    console.log("Mayor que 5");
}
```

---

### ❌ Error 5: Comparar con strings en switch

```javascript
const numero = 1; // Number

switch (numero) {
    case "1": // String "1"
        console.log("Uno"); // No se ejecuta
        break;
    case 1: // Number 1
        console.log("Uno"); // Sí se ejecuta
        break;
}
```

**Switch usa comparación estricta (`===`)**.

---

### ❌ Error 6: Condiciones redundantes

```javascript
// MAL: Redundante
if (edad >= 18) {
    if (edad < 65) {
        console.log("Adulto");
    }
}

// BIEN: Condición combinada
if (edad >= 18 && edad < 65) {
    console.log("Adulto");
}
```

---

## Buenas prácticas

### ✅ 1. Siempre usa llaves `{}`

```javascript
// ❌ Evita esto
if (condicion)
    console.log("Mensaje");

// ✅ Mejor
if (condicion) {
    console.log("Mensaje");
}
```

---

### ✅ 2. Usa `===` en vez de `==`

```javascript
// ❌ Evita ==
if (edad == "18") { } // Conversión de tipos

// ✅ Usa ===
if (edad === 18) { } // Comparación estricta
```

---

### ✅ 3. Condiciones positivas cuando sea posible

```javascript
// ❌ Difícil de leer
if (!esInactivo) {
    console.log("Usuario activo");
}

// ✅ Más claro
if (esActivo) {
    console.log("Usuario activo");
}
```

---

### ✅ 4. Early returns para validaciones

```javascript
// ❌ Anidamiento profundo
function procesarUsuario(usuario) {
    if (usuario) {
        if (usuario.edad >= 18) {
            if (usuario.email) {
                console.log("Procesando...");
            }
        }
    }
}

// ✅ Early returns
function procesarUsuario(usuario) {
    if (!usuario) return;
    if (usuario.edad < 18) return;
    if (!usuario.email) return;
    
    console.log("Procesando...");
}
```

---

### ✅ 5. Extrae condiciones complejas a variables

```javascript
// ❌ Difícil de leer
if ((edad >= 18 && edad < 65 && tieneLicencia) || esAdmin) {
    // ...
}

// ✅ Más claro
const esAdultoConLicencia = edad >= 18 && edad < 65 && tieneLicencia;
const puedeConducir = esAdultoConLicencia || esAdmin;

if (puedeConducir) {
    // ...
}
```

---

### ✅ 6. Switch para múltiples valores específicos

```javascript
// ❌ Muchos if-else
if (dia === "lunes") {
    // ...
} else if (dia === "martes") {
    // ...
} else if (dia === "miércoles") {
    // ...
}

// ✅ Usa switch
switch (dia) {
    case "lunes":
        // ...
        break;
    case "martes":
        // ...
        break;
    case "miércoles":
        // ...
        break;
}
```

---

### ✅ 7. Default en switch siempre

```javascript
// ✅ Siempre incluye default
switch (opcion) {
    case 1:
        console.log("Opción 1");
        break;
    case 2:
        console.log("Opción 2");
        break;
    default:
        console.log("Opción no válida");
}
```

---

## Cheatsheet

### if

```javascript
if (condicion) {
    // Se ejecuta si condicion es true
}
```

### if-else

```javascript
if (condicion) {
    // Si true
} else {
    // Si false
}
```

### if-else if-else

```javascript
if (condicion1) {
    // Si condicion1 es true
} else if (condicion2) {
    // Si condicion1 es false y condicion2 es true
} else {
    // Si todas son false
}
```

### switch

```javascript
switch (expresion) {
    case valor1:
        // Código
        break;
    case valor2:
        // Código
        break;
    default:
        // Si ningún caso coincide
}
```

### Valores falsy

```javascript
false, 0, -0, 0n, "", null, undefined, NaN
```

### Operadores en condiciones

```javascript
===  // Igual
!==  // Diferente
>    // Mayor
<    // Menor
>=   // Mayor o igual
<=   // Menor o igual
&&   // AND (y)
||   // OR (o)
!    // NOT (no)
```

### Operador ternario

```javascript
const resultado = condicion ? valorTrue : valorFalse;
```

---

## Siguiente paso

Ya sabes tomar decisiones en tu código. Ahora vamos a aprender a **repetir acciones**.

→ [17-bucles.md](17-bucles.md)

Ahí aprenderás `for`, `while`, `do-while` y cómo iterar sobre datos.

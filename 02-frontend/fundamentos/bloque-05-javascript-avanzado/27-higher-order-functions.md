# Módulo 27: Higher-Order Functions

## Índice
1. [¿Qué son las Higher-Order Functions?](#qué-son-las-higher-order-functions)
2. [Funciones como argumentos](#funciones-como-argumentos)
3. [Funciones que retornan funciones](#funciones-que-retornan-funciones)
4. [Callbacks](#callbacks)
5. [Closures avanzados](#closures-avanzados)
6. [Composición de funciones](#composición-de-funciones)
7. [Ejercicios prácticos](#ejercicios-prácticos)
8. [Errores comunes](#errores-comunes)
9. [Buenas prácticas](#buenas-prácticas)
10. [Cheatsheet](#cheatsheet)

---

## ¿Qué son las Higher-Order Functions?

**Higher-Order Functions** (funciones de orden superior) son funciones que:
1. **Reciben otras funciones como argumentos**, o
2. **Retornan funciones como resultado**

### Analogía: Fábrica de máquinas

```
Máquina Normal → Procesa materiales
Higher-Order Function → Crea o configura otras máquinas
```

### Ejemplo básico

```javascript
// Función normal
function sumar(a, b) {
    return a + b;
}

// Higher-Order Function (recibe una función)
function aplicarOperacion(a, b, operacion) {
    return operacion(a, b);
}

// Usar la HOF
const resultado = aplicarOperacion(5, 3, sumar);
console.log(resultado); // 8
```

---

## Funciones como argumentos

**Pasar funciones como parámetros**.

### Ejemplo simple

```javascript
function saludar(nombre) {
    console.log(`Hola, ${nombre}`);
}

function despedir(nombre) {
    console.log(`Adiós, ${nombre}`);
}

function procesarUsuario(nombre, accion) {
    console.log("Procesando usuario...");
    accion(nombre);
    console.log("Usuario procesado");
}

procesarUsuario("Ana", saludar);
// Output:
// Procesando usuario...
// Hola, Ana
// Usuario procesado

procesarUsuario("Juan", despedir);
// Output:
// Procesando usuario...
// Adiós, Juan
// Usuario procesado
```

### Funciones anónimas como argumentos

```javascript
function procesarNumeros(a, b, operacion) {
    return operacion(a, b);
}

// Pasar función anónima
const suma = procesarNumeros(10, 5, function(x, y) {
    return x + y;
});
console.log(suma); // 15

// Con arrow function
const multiplicacion = procesarNumeros(10, 5, (x, y) => x * y);
console.log(multiplicacion); // 50
```

### Ejemplo práctico: Array.forEach

```javascript
const numeros = [1, 2, 3, 4, 5];

// forEach es una HOF (recibe función como argumento)
numeros.forEach(function(numero) {
    console.log(numero * 2);
});

// Con arrow function
numeros.forEach(numero => console.log(numero * 2));
```

---

## Funciones que retornan funciones

**Crear funciones dinámicamente**.

### Ejemplo básico

```javascript
function crearSumador(x) {
    return function(y) {
        return x + y;
    };
}

const sumar5 = crearSumador(5);
console.log(sumar5(3));  // 8
console.log(sumar5(10)); // 15

const sumar10 = crearSumador(10);
console.log(sumar10(3));  // 13
console.log(sumar10(10)); // 20
```

### Con arrow functions

```javascript
const crearMultiplicador = factor => numero => numero * factor;

const duplicar = crearMultiplicador(2);
const triplicar = crearMultiplicador(3);

console.log(duplicar(5));  // 10
console.log(triplicar(5)); // 15
```

### Ejemplo práctico: Configurador

```javascript
function crearValidador(minLength) {
    return function(texto) {
        return texto.length >= minLength;
    };
}

const validarPassword = crearValidador(8);
const validarUsername = crearValidador(3);

console.log(validarPassword("abc"));      // false
console.log(validarPassword("abcd1234")); // true
console.log(validarUsername("ab"));       // false
console.log(validarUsername("abc"));      // true
```

### Factory functions

```javascript
function crearPersona(nombre, edad) {
    return {
        nombre,
        edad,
        saludar() {
            return `Hola, soy ${this.nombre} y tengo ${this.edad} años`;
        },
        cumplirAnios() {
            this.edad++;
            return `Ahora tengo ${this.edad} años`;
        }
    };
}

const ana = crearPersona("Ana", 25);
const juan = crearPersona("Juan", 30);

console.log(ana.saludar());     // "Hola, soy Ana y tengo 25 años"
console.log(juan.saludar());    // "Hola, soy Juan y tengo 30 años"
console.log(ana.cumplirAnios()); // "Ahora tengo 26 años"
```

---

## Callbacks

**Una función que se pasa como argumento para ser ejecutada después**.

### Callback síncrono

```javascript
function procesar(datos, callback) {
    console.log("Procesando datos...");
    const resultado = datos * 2;
    callback(resultado);
}

procesar(5, function(resultado) {
    console.log("Resultado:", resultado);
});

// Output:
// Procesando datos...
// Resultado: 10
```

### Callback asíncrono (setTimeout)

```javascript
function mostrarMensaje(mensaje, callback) {
    console.log("Preparando mensaje...");
    setTimeout(() => {
        console.log(mensaje);
        callback();
    }, 2000);
}

mostrarMensaje("¡Hola Mundo!", function() {
    console.log("Mensaje mostrado");
});

// Output (después de 2 segundos):
// Preparando mensaje...
// ¡Hola Mundo!
// Mensaje mostrado
```

### Múltiples callbacks

```javascript
function operacionConCallbacks(valor, onSuccess, onError) {
    if (valor > 0) {
        onSuccess(valor * 2);
    } else {
        onError("El valor debe ser positivo");
    }
}

operacionConCallbacks(5, 
    resultado => console.log("Éxito:", resultado),
    error => console.log("Error:", error)
);
// Output: Éxito: 10

operacionConCallbacks(-3,
    resultado => console.log("Éxito:", resultado),
    error => console.log("Error:", error)
);
// Output: Error: El valor debe ser positivo
```

---

## Closures avanzados

**Funciones que "recuerdan" su entorno**.

### Contador privado

```javascript
function crearContador(inicial = 0) {
    let contador = inicial; // Variable privada
    
    return {
        incrementar() {
            return ++contador;
        },
        decrementar() {
            return --contador;
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

const contador1 = crearContador(10);
console.log(contador1.incrementar()); // 11
console.log(contador1.incrementar()); // 12
console.log(contador1.obtener());     // 12
console.log(contador1.resetear());    // 10

// Cada contador es independiente
const contador2 = crearContador(0);
console.log(contador2.incrementar()); // 1
console.log(contador1.obtener());     // 10 (no afectado)
```

### Cuenta bancaria

```javascript
function crearCuenta(titular, saldoInicial) {
    let saldo = saldoInicial;
    const transacciones = [];
    
    function registrarTransaccion(tipo, monto) {
        transacciones.push({
            tipo,
            monto,
            fecha: new Date(),
            saldoResultante: saldo
        });
    }
    
    return {
        depositar(monto) {
            if (monto > 0) {
                saldo += monto;
                registrarTransaccion('depósito', monto);
                return `Depositado: $${monto}. Saldo: $${saldo}`;
            }
            return "Monto inválido";
        },
        
        retirar(monto) {
            if (monto > 0 && monto <= saldo) {
                saldo -= monto;
                registrarTransaccion('retiro', monto);
                return `Retirado: $${monto}. Saldo: $${saldo}`;
            }
            return "Saldo insuficiente o monto inválido";
        },
        
        consultarSaldo() {
            return `Titular: ${titular}. Saldo: $${saldo}`;
        },
        
        obtenerTransacciones() {
            return [...transacciones]; // Copia para proteger el original
        }
    };
}

const cuenta = crearCuenta("Ana García", 1000);
console.log(cuenta.depositar(500));      // "Depositado: $500. Saldo: $1500"
console.log(cuenta.retirar(200));        // "Retirado: $200. Saldo: $1300"
console.log(cuenta.consultarSaldo());    // "Titular: Ana García. Saldo: $1300"
console.log(cuenta.obtenerTransacciones()); // Array con historial
```

### Memoización (caché)

```javascript
function memoizar(fn) {
    const cache = {};
    
    return function(...args) {
        const key = JSON.stringify(args);
        
        if (key in cache) {
            console.log('📦 Desde caché');
            return cache[key];
        }
        
        console.log('⚙️ Calculando...');
        const resultado = fn(...args);
        cache[key] = resultado;
        return resultado;
    };
}

// Función costosa
function fibonacci(n) {
    if (n <= 1) return n;
    return fibonacci(n - 1) + fibonacci(n - 2);
}

const fibMemo = memoizar(fibonacci);

console.log(fibMemo(10)); // ⚙️ Calculando... → 55
console.log(fibMemo(10)); // 📦 Desde caché → 55
console.log(fibMemo(15)); // ⚙️ Calculando... → 610
console.log(fibMemo(15)); // 📦 Desde caché → 610
```

---

## Composición de funciones

**Combinar funciones para crear funciones más complejas**.

### Composición simple

```javascript
const duplicar = x => x * 2;
const incrementar = x => x + 1;
const cuadrado = x => x * x;

// Aplicar funciones en secuencia
const numero = 5;
const resultado = cuadrado(incrementar(duplicar(numero)));
console.log(resultado); // ((5 * 2) + 1)² = 121
```

### Función compose

```javascript
function compose(...funciones) {
    return function(valor) {
        return funciones.reduceRight((acc, fn) => fn(acc), valor);
    };
}

const duplicar = x => x * 2;
const incrementar = x => x + 1;
const cuadrado = x => x * x;

const operacion = compose(cuadrado, incrementar, duplicar);
console.log(operacion(5)); // 121
```

### Función pipe (orden natural)

```javascript
function pipe(...funciones) {
    return function(valor) {
        return funciones.reduce((acc, fn) => fn(acc), valor);
    };
}

const operacion = pipe(duplicar, incrementar, cuadrado);
console.log(operacion(5)); // 121
```

### Ejemplo práctico: Procesamiento de texto

```javascript
const minusculas = texto => texto.toLowerCase();
const eliminarEspacios = texto => texto.trim();
const reemplazarEspacios = texto => texto.replace(/\s+/g, '-');
const eliminarCaracteresEspeciales = texto => texto.replace(/[^a-z0-9-]/g, '');

const crearSlug = pipe(
    minusculas,
    eliminarEspacios,
    reemplazarEspacios,
    eliminarCaracteresEspeciales
);

console.log(crearSlug("  Hola Mundo! ¿Cómo estás?  "));
// "hola-mundo-como-estas"
```

---

## Ejercicios prácticos

### Ejercicio 1: Operaciones matemáticas
**Nivel**: ⭐⭐☆☆☆

Crea una HOF que aplique diferentes operaciones matemáticas.

<details>
<summary>Ver solución</summary>

```javascript
function calcular(a, b, operacion) {
    return operacion(a, b);
}

const sumar = (x, y) => x + y;
const restar = (x, y) => x - y;
const multiplicar = (x, y) => x * y;
const dividir = (x, y) => y !== 0 ? x / y : "Error: división por cero";

console.log(calcular(10, 5, sumar));        // 15
console.log(calcular(10, 5, restar));       // 5
console.log(calcular(10, 5, multiplicar));  // 50
console.log(calcular(10, 5, dividir));      // 2
console.log(calcular(10, 0, dividir));      // "Error: división por cero"
```

</details>

---

### Ejercicio 2: Generador de saludos
**Nivel**: ⭐⭐☆☆☆

Crea una función que genere diferentes tipos de saludos.

<details>
<summary>Ver solución</summary>

```javascript
function crearSaludador(saludo) {
    return function(nombre) {
        return `${saludo}, ${nombre}!`;
    };
}

const saludoFormal = crearSaludador("Buenos días");
const saludoInformal = crearSaludador("Hola");
const saludoNocturno = crearSaludador("Buenas noches");

console.log(saludoFormal("Sr. García"));    // "Buenos días, Sr. García!"
console.log(saludoInformal("Ana"));         // "Hola, Ana!"
console.log(saludoNocturno("familia"));     // "Buenas noches, familia!"
```

</details>

---

### Ejercicio 3: Sistema de descuentos
**Nivel**: ⭐⭐⭐☆☆

Crea funciones para aplicar diferentes tipos de descuentos.

<details>
<summary>Ver solución</summary>

```javascript
function crearDescuento(porcentaje) {
    return function(precio) {
        const descuento = precio * (porcentaje / 100);
        return precio - descuento;
    };
}

const descuento10 = crearDescuento(10);
const descuento25 = crearDescuento(25);
const descuento50 = crearDescuento(50);

console.log(`Precio original: $100`);
console.log(`Con 10% descuento: $${descuento10(100)}`); // $90
console.log(`Con 25% descuento: $${descuento25(100)}`); // $75
console.log(`Con 50% descuento: $${descuento50(100)}`); // $50

// Sistema de cupones
function aplicarCupon(precio, tipoCliente) {
    const descuentos = {
        'regular': crearDescuento(5),
        'premium': crearDescuento(15),
        'vip': crearDescuento(30)
    };
    
    const descuento = descuentos[tipoCliente] || (p => p);
    return descuento(precio);
}

console.log(aplicarCupon(100, 'regular'));  // $95
console.log(aplicarCupon(100, 'premium'));  // $85
console.log(aplicarCupon(100, 'vip'));      // $70
```

</details>

---

### Ejercicio 4: Logger con niveles
**Nivel**: ⭐⭐⭐☆☆

Crea un sistema de logging con diferentes niveles.

<details>
<summary>Ver solución</summary>

```javascript
function crearLogger(nivel) {
    const niveles = {
        'debug': 0,
        'info': 1,
        'warn': 2,
        'error': 3
    };
    
    const nivelMinimo = niveles[nivel] || 0;
    
    return {
        debug(mensaje) {
            if (niveles.debug >= nivelMinimo) {
                console.log(`🐛 DEBUG: ${mensaje}`);
            }
        },
        info(mensaje) {
            if (niveles.info >= nivelMinimo) {
                console.log(`ℹ️ INFO: ${mensaje}`);
            }
        },
        warn(mensaje) {
            if (niveles.warn >= nivelMinimo) {
                console.log(`⚠️ WARN: ${mensaje}`);
            }
        },
        error(mensaje) {
            if (niveles.error >= nivelMinimo) {
                console.log(`❌ ERROR: ${mensaje}`);
            }
        }
    };
}

const loggerDebug = crearLogger('debug');
loggerDebug.debug("Mensaje de debug");
loggerDebug.info("Mensaje de info");
loggerDebug.warn("Mensaje de advertencia");
loggerDebug.error("Mensaje de error");

console.log("\n--- Logger solo errores ---\n");

const loggerError = crearLogger('error');
loggerError.debug("No se mostrará");
loggerError.info("No se mostrará");
loggerError.warn("No se mostrará");
loggerError.error("Este SÍ se muestra");
```

</details>

---

### Ejercicio 5: Pipeline de transformaciones
**Nivel**: ⭐⭐⭐⭐☆

Crea un pipeline para transformar datos de usuarios.

<details>
<summary>Ver solución</summary>

```javascript
function pipe(...funciones) {
    return function(valor) {
        return funciones.reduce((acc, fn) => fn(acc), valor);
    };
}

// Transformaciones
const normalizarNombre = usuario => ({
    ...usuario,
    nombre: usuario.nombre.trim().toLowerCase()
});

const agregarEmail = usuario => ({
    ...usuario,
    email: `${usuario.nombre.replace(/\s+/g, '.')}@ejemplo.com`
});

const agregarFechaRegistro = usuario => ({
    ...usuario,
    fechaRegistro: new Date().toISOString()
});

const agregarId = usuario => ({
    ...usuario,
    id: Math.random().toString(36).substr(2, 9)
});

const validarEdad = usuario => ({
    ...usuario,
    esMayorDeEdad: usuario.edad >= 18
});

// Pipeline completo
const procesarUsuario = pipe(
    normalizarNombre,
    agregarEmail,
    agregarFechaRegistro,
    agregarId,
    validarEdad
);

const usuarioRaw = {
    nombre: "  Ana García  ",
    edad: 25
};

const usuarioProcesado = procesarUsuario(usuarioRaw);
console.log(usuarioProcesado);

// Output:
// {
//   nombre: "ana garcía",
//   edad: 25,
//   email: "ana.garcía@ejemplo.com",
//   fechaRegistro: "2024-01-15T10:30:00.000Z",
//   id: "a1b2c3d4e",
//   esMayorDeEdad: true
// }
```

</details>

---

## Errores comunes

### ❌ Error 1: Ejecutar la función en lugar de pasarla

```javascript
function procesar(callback) {
    callback();
}

function miFuncion() {
    console.log("Ejecutada");
}

// ❌ Error: ejecuta inmediatamente
procesar(miFuncion()); // "Ejecutada" (antes de tiempo)

// ✅ Correcto: pasa la referencia
procesar(miFuncion); // "Ejecutada" (en el momento correcto)
```

---

### ❌ Error 2: No retornar la función en HOF

```javascript
// ❌ Error: no retorna función
function crear Multiplicador(factor) {
    function multiplicar(numero) {
        return numero * factor;
    }
    // Falta: return multiplicar
}

// ✅ Correcto
function crearMultiplicador(factor) {
    return function(numero) {
        return numero * factor;
    };
}
```

---

### ❌ Error 3: Perder el contexto de this

```javascript
const objeto = {
    nombre: "Ana",
    saludar: function() {
        setTimeout(function() {
            console.log(`Hola, ${this.nombre}`); // undefined
        }, 1000);
    }
};

// ✅ Solución: arrow function
const objeto = {
    nombre: "Ana",
    saludar: function() {
        setTimeout(() => {
            console.log(`Hola, ${this.nombre}`); // "Ana"
        }, 1000);
    }
};
```

---

## Buenas prácticas

### ✅ 1. Usa nombres descriptivos

```javascript
// ❌ No descriptivo
const f = x => y => x + y;

// ✅ Descriptivo
const crearSumador = base => numero => base + numero;
```

---

### ✅ 2. Mantén las funciones pequeñas y enfocadas

```javascript
// ✅ Una responsabilidad por función
const duplicar = x => x * 2;
const incrementar = x => x + 1;
const cuadrado = x => x * x;

// Combínalas según necesites
const resultado = cuadrado(incrementar(duplicar(5)));
```

---

### ✅ 3. Usa closures para encapsular estado privado

```javascript
// ✅ Estado privado
function crearContador() {
    let contador = 0; // Privado
    
    return {
        incrementar: () => ++contador,
        obtener: () => contador
    };
}
```

---

## Cheatsheet

### HOF que reciben funciones

```javascript
function ejecutar(fn) {
    return fn();
}

ejecutar(() => console.log("Hola"));
```

### HOF que retornan funciones

```javascript
function crear(x) {
    return function(y) {
        return x + y;
    };
}

const fn = crear(5);
fn(3); // 8
```

### Callbacks

```javascript
function procesar(callback) {
    // ... hacer algo
    callback(resultado);
}

procesar(resultado => console.log(resultado));
```

### Closures

```javascript
function outer() {
    const variable = "valor";
    return function inner() {
        return variable; // Accede a variable
    };
}
```

### Composición

```javascript
const compose = (...fns) => x => fns.reduceRight((acc, fn) => fn(acc), x);
const pipe = (...fns) => x => fns.reduce((acc, fn) => fn(acc), x);
```

---

## Siguiente paso

Ya dominas Higher-Order Functions y closures avanzados. Ahora aprenderás los **métodos de array más potentes**.

→ [28-metodos-array-avanzados.md](28-metodos-array-avanzados.md)

Ahí profundizarás en map, filter, reduce, y otros métodos esenciales para programación funcional.

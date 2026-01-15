# Módulo 20: Objetos en JavaScript

## Índice
1. [¿Qué son los objetos?](#qué-son-los-objetos)
2. [Crear objetos](#crear-objetos)
3. [Acceder a propiedades](#acceder-a-propiedades)
4. [Modificar y añadir propiedades](#modificar-y-añadir-propiedades)
5. [Métodos de objetos](#métodos-de-objetos)
6. [this - El contexto](#this---el-contexto)
7. [Destructuring](#destructuring)
8. [Objetos anidados](#objetos-anidados)
9. [Ejercicios prácticos](#ejercicios-prácticos)
10. [Errores comunes](#errores-comunes)
11. [Buenas prácticas](#buenas-prácticas)
12. [Cheatsheet](#cheatsheet)

---

## ¿Qué son los objetos?

**Un objeto es una colección de propiedades (pares clave-valor)**.

### Analogía: Ficha de persona

```
Persona:
- Nombre: Ana García
- Edad: 25
- Ciudad: Madrid
- Email: ana@email.com
```

En JavaScript:

```javascript
const persona = {
    nombre: "Ana García",
    edad: 25,
    ciudad: "Madrid",
    email: "ana@email.com"
};
```

### ¿Para qué sirven?

- Agrupar datos relacionados
- Modelar entidades del mundo real
- Organizar código de forma lógica
- Crear estructuras de datos complejas
- Base para la programación orientada a objetos

### Arrays vs Objetos

```javascript
// Array: Lista ordenada (índices numéricos)
const persona = ["Ana", 25, "Madrid"];
console.log(persona[0]); // "Ana"

// Objeto: Colección con nombres (claves descriptivas)
const persona = {
    nombre: "Ana",
    edad: 25,
    ciudad: "Madrid"
};
console.log(persona.nombre); // "Ana"
```

---

## Crear objetos

### Sintaxis literal (recomendada)

```javascript
const persona = {
    nombre: "Ana",
    edad: 25,
    ciudad: "Madrid"
};

console.log(persona);
// { nombre: "Ana", edad: 25, ciudad: "Madrid" }
```

### Objeto vacío

```javascript
const vacio = {};
console.log(vacio); // {}
```

### Propiedades con diferentes tipos

```javascript
const usuario = {
    nombre: "Ana",              // String
    edad: 25,                   // Number
    activo: true,               // Boolean
    hobbies: ["leer", "nadar"], // Array
    direccion: {                // Objeto anidado
        calle: "Gran Vía",
        numero: 123
    },
    saludar: function() {       // Función
        console.log("Hola");
    }
};
```

### Propiedades con nombres dinámicos

```javascript
const propiedad = "nombre";
const valor = "Ana";

const persona = {
    [propiedad]: valor  // nombre: "Ana"
};

console.log(persona); // { nombre: "Ana" }
```

### Shorthand properties

```javascript
const nombre = "Ana";
const edad = 25;

// ❌ Forma larga
const persona1 = {
    nombre: nombre,
    edad: edad
};

// ✅ Forma corta (mismo nombre)
const persona2 = {
    nombre,
    edad
};

console.log(persona2); // { nombre: "Ana", edad: 25 }
```

---

## Acceder a propiedades

### Notación de punto (más común)

```javascript
const persona = {
    nombre: "Ana",
    edad: 25,
    ciudad: "Madrid"
};

console.log(persona.nombre);  // "Ana"
console.log(persona.edad);    // 25
console.log(persona.ciudad);  // "Madrid"
```

### Notación de corchetes

```javascript
const persona = {
    nombre: "Ana",
    edad: 25,
    "código postal": "28001"
};

console.log(persona["nombre"]);        // "Ana"
console.log(persona["edad"]);          // 25
console.log(persona["código postal"]); // "28001"
```

### ¿Cuándo usar corchetes?

```javascript
const persona = {
    nombre: "Ana",
    "código postal": "28001"
};

// 1. Propiedades con espacios o caracteres especiales
console.log(persona["código postal"]); // ✅ Funciona
// console.log(persona.código postal); // ❌ Error de sintaxis

// 2. Propiedades dinámicas
const propiedad = "nombre";
console.log(persona[propiedad]); // "Ana"

// 3. Nombres de propiedades desde variables
const campo = "edad";
console.log(persona[campo]); // 25
```

### Propiedad inexistente

```javascript
const persona = {
    nombre: "Ana"
};

console.log(persona.edad); // undefined (no existe)
console.log(persona.ciudad); // undefined
```

### Optional chaining (?.)

```javascript
const usuario = {
    nombre: "Ana"
    // dirección no existe
};

// ❌ Error si dirección es undefined
// console.log(usuario.direccion.calle); // Error

// ✅ Con optional chaining
console.log(usuario.direccion?.calle); // undefined (no error)
```

---

## Modificar y añadir propiedades

### Modificar propiedad existente

```javascript
const persona = {
    nombre: "Ana",
    edad: 25
};

persona.edad = 26;
console.log(persona); // { nombre: "Ana", edad: 26 }
```

### Añadir nueva propiedad

```javascript
const persona = {
    nombre: "Ana"
};

persona.edad = 25;
persona.ciudad = "Madrid";

console.log(persona);
// { nombre: "Ana", edad: 25, ciudad: "Madrid" }
```

### Eliminar propiedad

```javascript
const persona = {
    nombre: "Ana",
    edad: 25,
    ciudad: "Madrid"
};

delete persona.ciudad;
console.log(persona); // { nombre: "Ana", edad: 25 }
```

### const no impide modificar propiedades

```javascript
const persona = {
    nombre: "Ana"
};

// ✅ Puedes modificar propiedades
persona.nombre = "Juan";
persona.edad = 30;
console.log(persona); // { nombre: "Juan", edad: 30 }

// ❌ Pero no reasignar el objeto completo
persona = { nombre: "María" }; // Error: Assignment to constant variable
```

---

## Métodos de objetos

**Un método es una función dentro de un objeto**.

### Crear métodos

```javascript
const persona = {
    nombre: "Ana",
    edad: 25,
    
    // Método tradicional
    saludar: function() {
        console.log(`Hola, soy ${this.nombre}`);
    },
    
    // Método corto (ES6+)
    despedir() {
        console.log("¡Adiós!");
    }
};

persona.saludar();  // "Hola, soy Ana"
persona.despedir(); // "¡Adiós!"
```

### Métodos con parámetros

```javascript
const calculadora = {
    sumar(a, b) {
        return a + b;
    },
    
    restar(a, b) {
        return a - b;
    },
    
    multiplicar(a, b) {
        return a * b;
    }
};

console.log(calculadora.sumar(5, 3));       // 8
console.log(calculadora.restar(10, 4));     // 6
console.log(calculadora.multiplicar(3, 7)); // 21
```

---

## this - El contexto

**`this` se refiere al objeto que contiene el método**.

### Usar this en métodos

```javascript
const persona = {
    nombre: "Ana",
    edad: 25,
    
    presentarse() {
        console.log(`Hola, soy ${this.nombre} y tengo ${this.edad} años`);
    },
    
    cumplirAnios() {
        this.edad++;
        console.log(`Ahora tengo ${this.edad} años`);
    }
};

persona.presentarse(); // "Hola, soy Ana y tengo 25 años"
persona.cumplirAnios(); // "Ahora tengo 26 años"
persona.presentarse(); // "Hola, soy Ana y tengo 26 años"
```

### Problema con arrow functions

```javascript
const persona = {
    nombre: "Ana",
    
    // ❌ Arrow function: this no apunta al objeto
    saludar: () => {
        console.log(`Hola, soy ${this.nombre}`); // undefined
    },
    
    // ✅ Función tradicional: this apunta al objeto
    despedir: function() {
        console.log(`Adiós, soy ${this.nombre}`);
    },
    
    // ✅ Método corto: this apunta al objeto
    presentar() {
        console.log(`Soy ${this.nombre}`);
    }
};
```

**Regla**: No uses arrow functions como métodos de objetos.

---

## Destructuring

**Extraer propiedades de un objeto en variables**.

### Destructuring básico

```javascript
const persona = {
    nombre: "Ana",
    edad: 25,
    ciudad: "Madrid"
};

// Sin destructuring
const nombre = persona.nombre;
const edad = persona.edad;

// Con destructuring
const { nombre, edad, ciudad } = persona;

console.log(nombre);  // "Ana"
console.log(edad);    // 25
console.log(ciudad);  // "Madrid"
```

### Renombrar variables

```javascript
const persona = {
    nombre: "Ana",
    edad: 25
};

const { nombre: nombreCompleto, edad: años } = persona;

console.log(nombreCompleto); // "Ana"
console.log(años);           // 25
// console.log(nombre); // Error: nombre is not defined
```

### Valores por defecto

```javascript
const persona = {
    nombre: "Ana"
    // edad no existe
};

const { nombre, edad = 18 } = persona;

console.log(nombre); // "Ana"
console.log(edad);   // 18 (valor por defecto)
```

### Destructuring en parámetros

```javascript
function presentar({ nombre, edad, ciudad }) {
    console.log(`${nombre}, ${edad} años, de ${ciudad}`);
}

const persona = {
    nombre: "Ana",
    edad: 25,
    ciudad: "Madrid"
};

presentar(persona); // "Ana, 25 años, de Madrid"
```

### Destructuring anidado

```javascript
const usuario = {
    nombre: "Ana",
    direccion: {
        calle: "Gran Vía",
        numero: 123,
        ciudad: "Madrid"
    }
};

const {
    nombre,
    direccion: { calle, ciudad }
} = usuario;

console.log(nombre); // "Ana"
console.log(calle);  // "Gran Vía"
console.log(ciudad); // "Madrid"
```

---

## Objetos anidados

**Objetos dentro de objetos**.

### Crear objeto anidado

```javascript
const usuario = {
    nombre: "Ana",
    edad: 25,
    direccion: {
        calle: "Gran Vía",
        numero: 123,
        ciudad: "Madrid",
        pais: "España"
    },
    contacto: {
        email: "ana@email.com",
        telefono: "123456789"
    }
};
```

### Acceder a propiedades anidadas

```javascript
console.log(usuario.nombre);                  // "Ana"
console.log(usuario.direccion.ciudad);        // "Madrid"
console.log(usuario.contacto.email);          // "ana@email.com"
console.log(usuario.direccion.pais);          // "España"
```

### Modificar propiedades anidadas

```javascript
usuario.direccion.ciudad = "Barcelona";
usuario.contacto.telefono = "987654321";

console.log(usuario.direccion.ciudad);   // "Barcelona"
console.log(usuario.contacto.telefono); // "987654321"
```

### Arrays de objetos

```javascript
const usuarios = [
    { id: 1, nombre: "Ana", edad: 25 },
    { id: 2, nombre: "Juan", edad: 30 },
    { id: 3, nombre: "María", edad: 28 }
];

// Acceder a un usuario específico
console.log(usuarios[0].nombre); // "Ana"
console.log(usuarios[1].edad);   // 30

// Iterar sobre los usuarios
usuarios.forEach(usuario => {
    console.log(`${usuario.nombre}: ${usuario.edad} años`);
});
```

---

## Ejercicios prácticos

### Ejercicio 1: Crear objeto persona
**Nivel**: ⭐☆☆☆☆

Crea un objeto `persona` con nombre, edad y ciudad. Muestra cada propiedad.

<details>
<summary>Ver solución</summary>

```javascript
const persona = {
    nombre: "Ana García",
    edad: 25,
    ciudad: "Madrid"
};

console.log("Nombre:", persona.nombre);
console.log("Edad:", persona.edad);
console.log("Ciudad:", persona.ciudad);

// Output:
// Nombre: Ana García
// Edad: 25
// Ciudad: Madrid
```

</details>

---

### Ejercicio 2: Método de objeto
**Nivel**: ⭐⭐☆☆☆

Crea un objeto `persona` con un método `presentarse()` que muestre un mensaje de presentación.

<details>
<summary>Ver solución</summary>

```javascript
const persona = {
    nombre: "Ana",
    edad: 25,
    ciudad: "Madrid",
    
    presentarse() {
        console.log(`Hola, soy ${this.nombre}, tengo ${this.edad} años y vivo en ${this.ciudad}`);
    }
};

persona.presentarse();
// "Hola, soy Ana, tengo 25 años y vivo en Madrid"
```

</details>

---

### Ejercicio 3: Calculadora de objetos
**Nivel**: ⭐⭐☆☆☆

Crea un objeto `calculadora` con métodos para sumar, restar, multiplicar y dividir.

<details>
<summary>Ver solución</summary>

```javascript
const calculadora = {
    sumar(a, b) {
        return a + b;
    },
    
    restar(a, b) {
        return a - b;
    },
    
    multiplicar(a, b) {
        return a * b;
    },
    
    dividir(a, b) {
        if (b === 0) {
            return "Error: División por cero";
        }
        return a / b;
    }
};

console.log(calculadora.sumar(10, 5));       // 15
console.log(calculadora.restar(10, 5));      // 5
console.log(calculadora.multiplicar(10, 5)); // 50
console.log(calculadora.dividir(10, 5));     // 2
console.log(calculadora.dividir(10, 0));     // "Error: División por cero"
```

</details>

---

### Ejercicio 4: Contador con métodos
**Nivel**: ⭐⭐⭐☆☆

Crea un objeto `contador` con valor inicial 0 y métodos para incrementar, decrementar y resetear.

<details>
<summary>Ver solución</summary>

```javascript
const contador = {
    valor: 0,
    
    incrementar() {
        this.valor++;
        console.log(`Contador: ${this.valor}`);
    },
    
    decrementar() {
        this.valor--;
        console.log(`Contador: ${this.valor}`);
    },
    
    resetear() {
        this.valor = 0;
        console.log("Contador reseteado a 0");
    },
    
    obtenerValor() {
        return this.valor;
    }
};

contador.incrementar();  // "Contador: 1"
contador.incrementar();  // "Contador: 2"
contador.incrementar();  // "Contador: 3"
contador.decrementar();  // "Contador: 2"
console.log(contador.obtenerValor()); // 2
contador.resetear();     // "Contador reseteado a 0"
```

</details>

---

### Ejercicio 5: Destructuring
**Nivel**: ⭐⭐⭐☆☆

Dado un objeto con datos de usuario, extrae nombre, email y ciudad usando destructuring.

<details>
<summary>Ver solución</summary>

```javascript
const usuario = {
    id: 1,
    nombre: "Ana García",
    email: "ana@email.com",
    edad: 25,
    ciudad: "Madrid",
    pais: "España"
};

// Extraer solo las propiedades necesarias
const { nombre, email, ciudad } = usuario;

console.log("Nombre:", nombre);  // "Ana García"
console.log("Email:", email);    // "ana@email.com"
console.log("Ciudad:", ciudad);  // "Madrid"
```

</details>

---

### Ejercicio 6: Objeto anidado
**Nivel**: ⭐⭐⭐☆☆

Crea un objeto `estudiante` con información personal y un objeto anidado `notas` con sus calificaciones.

<details>
<summary>Ver solución</summary>

```javascript
const estudiante = {
    nombre: "Ana García",
    edad: 20,
    carrera: "Ingeniería",
    notas: {
        matematicas: 85,
        fisica: 90,
        programacion: 95
    },
    
    calcularPromedio() {
        const { matematicas, fisica, programacion } = this.notas;
        const promedio = (matematicas + fisica + programacion) / 3;
        return promedio.toFixed(2);
    }
};

console.log(`Estudiante: ${estudiante.nombre}`);
console.log(`Edad: ${estudiante.edad}`);
console.log("Notas:");
console.log(`  - Matemáticas: ${estudiante.notas.matematicas}`);
console.log(`  - Física: ${estudiante.notas.fisica}`);
console.log(`  - Programación: ${estudiante.notas.programacion}`);
console.log(`Promedio: ${estudiante.calcularPromedio()}`);

// Output:
// Estudiante: Ana García
// Edad: 20
// Notas:
//   - Matemáticas: 85
//   - Física: 90
//   - Programación: 95
// Promedio: 90.00
```

</details>

---

### Ejercicio 7: Array de objetos
**Nivel**: ⭐⭐⭐⭐☆

Crea un array de productos con nombre, precio y stock. Encuentra el producto más caro.

<details>
<summary>Ver solución</summary>

```javascript
const productos = [
    { nombre: "Laptop", precio: 800, stock: 5 },
    { nombre: "Mouse", precio: 25, stock: 50 },
    { nombre: "Teclado", precio: 50, stock: 30 },
    { nombre: "Monitor", precio: 300, stock: 10 }
];

// Encontrar el más caro
const masCaro = productos.reduce((max, producto) => {
    return producto.precio > max.precio ? producto : max;
});

console.log("Producto más caro:");
console.log(`  Nombre: ${masCaro.nombre}`);
console.log(`  Precio: ${masCaro.precio}€`);
console.log(`  Stock: ${masCaro.stock} unidades`);

// Output:
// Producto más caro:
//   Nombre: Laptop
//   Precio: 800€
//   Stock: 5 unidades
```

</details>

---

### Ejercicio 8: Biblioteca de libros
**Nivel**: ⭐⭐⭐⭐☆

Crea un objeto `biblioteca` que gestione libros (añadir, buscar, listar).

<details>
<summary>Ver solución</summary>

```javascript
const biblioteca = {
    libros: [],
    
    agregarLibro(titulo, autor, año) {
        const libro = {
            id: this.libros.length + 1,
            titulo,
            autor,
            año
        };
        this.libros.push(libro);
        console.log(`✅ Libro "${titulo}" agregado`);
    },
    
    buscarPorTitulo(titulo) {
        return this.libros.find(libro => 
            libro.titulo.toLowerCase().includes(titulo.toLowerCase())
        );
    },
    
    listarLibros() {
        if (this.libros.length === 0) {
            console.log("La biblioteca está vacía");
            return;
        }
        
        console.log("\n📚 Libros en la biblioteca:");
        this.libros.forEach(libro => {
            console.log(`  ${libro.id}. "${libro.titulo}" - ${libro.autor} (${libro.año})`);
        });
    },
    
    librosPorAutor(autor) {
        return this.libros.filter(libro => 
            libro.autor.toLowerCase().includes(autor.toLowerCase())
        );
    }
};

// Usar la biblioteca
biblioteca.agregarLibro("Cien años de soledad", "Gabriel García Márquez", 1967);
biblioteca.agregarLibro("Don Quijote", "Miguel de Cervantes", 1605);
biblioteca.agregarLibro("1984", "George Orwell", 1949);

biblioteca.listarLibros();

const encontrado = biblioteca.buscarPorTitulo("1984");
if (encontrado) {
    console.log(`\n🔍 Encontrado: "${encontrado.titulo}" por ${encontrado.autor}`);
}

// Output:
// ✅ Libro "Cien años de soledad" agregado
// ✅ Libro "Don Quijote" agregado
// ✅ Libro "1984" agregado
//
// 📚 Libros en la biblioteca:
//   1. "Cien años de soledad" - Gabriel García Márquez (1967)
//   2. "Don Quijote" - Miguel de Cervantes (1605)
//   3. "1984" - George Orwell (1949)
//
// 🔍 Encontrado: "1984" por George Orwell
```

</details>

---

## Errores comunes

### ❌ Error 1: Acceder a propiedad inexistente

```javascript
const persona = {
    nombre: "Ana"
};

console.log(persona.edad); // undefined (no error, pero no existe)

// Solución: verifica antes
if (persona.edad !== undefined) {
    console.log(persona.edad);
} else {
    console.log("La edad no está definida");
}
```

---

### ❌ Error 2: Arrow function como método

```javascript
const persona = {
    nombre: "Ana",
    
    // ❌ this no funciona en arrow functions
    saludar: () => {
        console.log(`Hola, ${this.nombre}`); // undefined
    }
};

// ✅ Usa función tradicional o método corto
const persona = {
    nombre: "Ana",
    
    saludar() {
        console.log(`Hola, ${this.nombre}`); // "Ana"
    }
};
```

---

### ❌ Error 3: Olvidar this

```javascript
const contador = {
    valor: 0,
    
    incrementar() {
        valor++; // ❌ Error: valor is not defined
    }
};

// ✅ Usa this
const contador = {
    valor: 0,
    
    incrementar() {
        this.valor++; // ✅ Correcto
    }
};
```

---

### ❌ Error 4: Modificar objeto en const

```javascript
const persona = {
    nombre: "Ana"
};

// ✅ Puedes modificar propiedades
persona.nombre = "Juan"; // OK
persona.edad = 25;       // OK

// ❌ No puedes reasignar el objeto
persona = { nombre: "María" }; // Error
```

---

## Buenas prácticas

### ✅ 1. Usa const para objetos

```javascript
// ✅ const (puedes modificar propiedades)
const persona = { nombre: "Ana" };
persona.edad = 25; // OK

// ❌ let innecesario
let persona = { nombre: "Ana" };
```

---

### ✅ 2. Nombres descriptivos

```javascript
// ❌ No descriptivo
const obj = { n: "Ana", e: 25 };

// ✅ Descriptivo
const persona = { nombre: "Ana", edad: 25 };
```

---

### ✅ 3. Agrupa propiedades relacionadas

```javascript
// ❌ Propiedades sueltas
const usuario = {
    nombre: "Ana",
    calle: "Gran Vía",
    numero: 123,
    ciudad: "Madrid"
};

// ✅ Agrupadas en objeto anidado
const usuario = {
    nombre: "Ana",
    direccion: {
        calle: "Gran Vía",
        numero: 123,
        ciudad: "Madrid"
    }
};
```

---

### ✅ 4. Usa métodos cortos

```javascript
// ❌ Forma larga
const persona = {
    saludar: function() {
        console.log("Hola");
    }
};

// ✅ Forma corta
const persona = {
    saludar() {
        console.log("Hola");
    }
};
```

---

### ✅ 5. Usa destructuring cuando sea apropiado

```javascript
const usuario = {
    nombre: "Ana",
    edad: 25,
    ciudad: "Madrid",
    pais: "España"
};

// ❌ Repetitivo
function mostrarInfo(usuario) {
    console.log(usuario.nombre);
    console.log(usuario.edad);
    console.log(usuario.ciudad);
}

// ✅ Con destructuring
function mostrarInfo({ nombre, edad, ciudad }) {
    console.log(nombre);
    console.log(edad);
    console.log(ciudad);
}
```

---

## Cheatsheet

### Crear objetos

```javascript
const obj = { clave: valor };                    // Literal
const vacio = {};                                 // Vacío
const { nombre, edad } = persona;                 // Destructuring
```

### Acceso a propiedades

```javascript
obj.propiedad                    // Notación de punto
obj["propiedad"]                 // Notación de corchetes
obj.prop?.subprop                // Optional chaining
```

### Modificar

```javascript
obj.propiedad = valor            // Modificar/añadir
delete obj.propiedad             // Eliminar
```

### Métodos

```javascript
const obj = {
    metodo() {                   // Método corto
        return this.propiedad;
    },
    
    metodo2: function() {        // Método tradicional
        return this.propiedad;
    }
};
```

### Destructuring

```javascript
const { a, b } = obj;                    // Básico
const { a: nuevoNombre } = obj;          // Renombrar
const { a = 10 } = obj;                  // Valor por defecto
const { a: { b } } = obj;                // Anidado
```

### Object methods

```javascript
Object.keys(obj)                 // Array de claves
Object.values(obj)               // Array de valores
Object.entries(obj)              // Array de pares [clave, valor]
Object.assign({}, obj1, obj2)    // Combinar objetos
```

### Copia

```javascript
const copia = { ...obj };        // Spread (copia superficial)
const copia = Object.assign({}, obj); // assign
```

---

## Siguiente paso

Ya dominas objetos, la estructura de datos fundamental de JavaScript. Ahora vamos a entender mejor cómo funcionan las **variables y el scope**.

→ [21-scope-hoisting.md](21-scope-hoisting.md)

Ahí aprenderás sobre scope (alcance), hoisting, closures y cómo JavaScript maneja las variables internamente.

# Módulo 28: Métodos de Array Avanzados

## Índice
1. [Introducción a los métodos funcionales](#introducción-a-los-métodos-funcionales)
2. [map() - Transformar arrays](#map---transformar-arrays)
3. [filter() - Filtrar elementos](#filter---filtrar-elementos)
4. [reduce() - Reducir a un valor](#reduce---reducir-a-un-valor)
5. [find() y findIndex()](#find-y-findindex)
6. [some() y every()](#some-y-every)
7. [flat() y flatMap()](#flat-y-flatmap)
8. [sort() avanzado](#sort-avanzado)
9. [Encadenar métodos](#encadenar-métodos)
10. [Ejercicios prácticos](#ejercicios-prácticos)
11. [Errores comunes](#errores-comunes)
12. [Buenas prácticas](#buenas-prácticas)
13. [Cheatsheet](#cheatsheet)

---

## Introducción a los métodos funcionales

Los **métodos de array avanzados** son Higher-Order Functions que permiten trabajar con arrays de forma declarativa y funcional.

### Analogía: Línea de producción

```
Array original → [Métodos] → Nuevo array transformado

Como una fábrica:
- map()    → Transformar cada pieza
- filter() → Seleccionar solo ciertas piezas
- reduce() → Combinar todas las piezas en un resultado
```

### Características principales

✅ **No mutan el array original** (excepto sort)  
✅ **Retornan nuevos arrays o valores**  
✅ **Reciben callbacks como argumentos**  
✅ **Se pueden encadenar**

### Comparación con loops tradicionales

```javascript
const numeros = [1, 2, 3, 4, 5];

// ❌ Forma imperativa (con for)
const dobles = [];
for (let i = 0; i < numeros.length; i++) {
    dobles.push(numeros[i] * 2);
}

// ✅ Forma declarativa (con map)
const dobles = numeros.map(num => num * 2);
```

---

## map() - Transformar arrays

**Transforma cada elemento del array aplicando una función**.

### Sintaxis

```javascript
array.map(callback(elemento, índice, array))
```

### Ejemplo básico

```javascript
const numeros = [1, 2, 3, 4, 5];

const dobles = numeros.map(num => num * 2);
console.log(dobles); // [2, 4, 6, 8, 10]

const cuadrados = numeros.map(num => num ** 2);
console.log(cuadrados); // [1, 4, 9, 16, 25]
```

### Transformar objetos

```javascript
const usuarios = [
    { nombre: "Ana", edad: 25 },
    { nombre: "Juan", edad: 30 },
    { nombre: "María", edad: 28 }
];

// Extraer solo los nombres
const nombres = usuarios.map(usuario => usuario.nombre);
console.log(nombres); // ["Ana", "Juan", "María"]

// Crear nuevos objetos
const conEmail = usuarios.map(usuario => ({
    ...usuario,
    email: `${usuario.nombre.toLowerCase()}@ejemplo.com`
}));

console.log(conEmail);
// [
//   { nombre: "Ana", edad: 25, email: "ana@ejemplo.com" },
//   { nombre: "Juan", edad: 30, email: "juan@ejemplo.com" },
//   { nombre: "María", edad: 28, email: "maría@ejemplo.com" }
// ]
```

### Usando índice y array

```javascript
const letras = ['a', 'b', 'c', 'd'];

const conIndice = letras.map((letra, indice) => {
    return `${indice + 1}. ${letra.toUpperCase()}`;
});

console.log(conIndice);
// ["1. A", "2. B", "3. C", "4. D"]
```

### Ejemplo práctico: Formatear precios

```javascript
const productos = [
    { nombre: "Laptop", precio: 1200 },
    { nombre: "Mouse", precio: 25 },
    { nombre: "Teclado", precio: 80 }
];

const productosFormateados = productos.map(producto => ({
    ...producto,
    precioFormateado: `$${producto.precio.toFixed(2)}`,
    precioConIVA: producto.precio * 1.21
}));

console.log(productosFormateados);
// [
//   { nombre: "Laptop", precio: 1200, precioFormateado: "$1200.00", precioConIVA: 1452 },
//   ...
// ]
```

---

## filter() - Filtrar elementos

**Crea un nuevo array con los elementos que cumplen una condición**.

### Sintaxis

```javascript
array.filter(callback(elemento, índice, array))
```

### Ejemplo básico

```javascript
const numeros = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10];

// Números pares
const pares = numeros.filter(num => num % 2 === 0);
console.log(pares); // [2, 4, 6, 8, 10]

// Números mayores que 5
const mayoresQue5 = numeros.filter(num => num > 5);
console.log(mayoresQue5); // [6, 7, 8, 9, 10]
```

### Filtrar objetos

```javascript
const usuarios = [
    { nombre: "Ana", edad: 17, activo: true },
    { nombre: "Juan", edad: 30, activo: true },
    { nombre: "María", edad: 28, activo: false },
    { nombre: "Pedro", edad: 15, activo: true }
];

// Usuarios mayores de edad
const mayoresDeEdad = usuarios.filter(usuario => usuario.edad >= 18);
console.log(mayoresDeEdad);
// [{ nombre: "Juan", ... }, { nombre: "María", ... }]

// Usuarios activos y mayores de edad
const activosYMayores = usuarios.filter(usuario => {
    return usuario.edad >= 18 && usuario.activo;
});
console.log(activosYMayores);
// [{ nombre: "Juan", edad: 30, activo: true }]
```

### Filtrar valores únicos

```javascript
const numeros = [1, 2, 2, 3, 4, 4, 5, 5, 5];

const unicos = numeros.filter((num, indice, array) => {
    return array.indexOf(num) === indice;
});

console.log(unicos); // [1, 2, 3, 4, 5]
```

### Ejemplo práctico: Búsqueda

```javascript
const productos = [
    { nombre: "Laptop HP", categoria: "Electrónica", precio: 1200 },
    { nombre: "Mouse Logitech", categoria: "Electrónica", precio: 25 },
    { nombre: "Silla Ergonómica", categoria: "Muebles", precio: 300 },
    { nombre: "Teclado Mecánico", categoria: "Electrónica", precio: 150 }
];

function buscarProductos(texto, categoriaFiltro, precioMax) {
    return productos.filter(producto => {
        const coincideTexto = producto.nombre.toLowerCase().includes(texto.toLowerCase());
        const coincideCategoria = !categoriaFiltro || producto.categoria === categoriaFiltro;
        const coincidePrecio = !precioMax || producto.precio <= precioMax;
        
        return coincideTexto && coincideCategoria && coincidePrecio;
    });
}

console.log(buscarProductos("mouse", "", 50));
// [{ nombre: "Mouse Logitech", ... }]

console.log(buscarProductos("", "Electrónica", 200));
// [{ Mouse... }, { Teclado... }]
```

---

## reduce() - Reducir a un valor

**Reduce el array a un único valor aplicando una función acumuladora**.

### Sintaxis

```javascript
array.reduce(callback(acumulador, elemento, índice, array), valorInicial)
```

### Sumar números

```javascript
const numeros = [1, 2, 3, 4, 5];

const suma = numeros.reduce((acc, num) => acc + num, 0);
console.log(suma); // 15

// Paso a paso:
// acc = 0, num = 1 → return 0 + 1 = 1
// acc = 1, num = 2 → return 1 + 2 = 3
// acc = 3, num = 3 → return 3 + 3 = 6
// acc = 6, num = 4 → return 6 + 4 = 10
// acc = 10, num = 5 → return 10 + 5 = 15
```

### Encontrar máximo/mínimo

```javascript
const numeros = [10, 5, 8, 15, 3, 20];

const maximo = numeros.reduce((max, num) => {
    return num > max ? num : max;
}, numeros[0]);

console.log(maximo); // 20

const minimo = numeros.reduce((min, num) => num < min ? num : min, numeros[0]);
console.log(minimo); // 3
```

### Contar elementos

```javascript
const frutas = ["manzana", "pera", "manzana", "uva", "pera", "manzana"];

const conteo = frutas.reduce((acc, fruta) => {
    acc[fruta] = (acc[fruta] || 0) + 1;
    return acc;
}, {});

console.log(conteo);
// { manzana: 3, pera: 2, uva: 1 }
```

### Agrupar por propiedad

```javascript
const productos = [
    { nombre: "Laptop", categoria: "Electrónica" },
    { nombre: "Mouse", categoria: "Electrónica" },
    { nombre: "Silla", categoria: "Muebles" },
    { nombre: "Mesa", categoria: "Muebles" },
    { nombre: "Teclado", categoria: "Electrónica" }
];

const porCategoria = productos.reduce((acc, producto) => {
    const cat = producto.categoria;
    if (!acc[cat]) {
        acc[cat] = [];
    }
    acc[cat].push(producto);
    return acc;
}, {});

console.log(porCategoria);
// {
//   Electrónica: [{ Laptop... }, { Mouse... }, { Teclado... }],
//   Muebles: [{ Silla... }, { Mesa... }]
// }
```

### Aplanar arrays (flatten)

```javascript
const arrays = [[1, 2], [3, 4], [5, 6]];

const aplanado = arrays.reduce((acc, array) => acc.concat(array), []);
console.log(aplanado); // [1, 2, 3, 4, 5, 6]
```

### Ejemplo práctico: Carrito de compras

```javascript
const carrito = [
    { producto: "Laptop", precio: 1200, cantidad: 1 },
    { producto: "Mouse", precio: 25, cantidad: 2 },
    { producto: "Teclado", precio: 80, cantidad: 1 }
];

const estadisticas = carrito.reduce((acc, item) => {
    const subtotal = item.precio * item.cantidad;
    
    return {
        items: acc.items + item.cantidad,
        subtotal: acc.subtotal + subtotal,
        iva: (acc.subtotal + subtotal) * 0.21,
        total: (acc.subtotal + subtotal) * 1.21
    };
}, { items: 0, subtotal: 0, iva: 0, total: 0 });

console.log(estadisticas);
// {
//   items: 4,
//   subtotal: 1330,
//   iva: 279.3,
//   total: 1609.3
// }
```

---

## find() y findIndex()

**Encontrar elementos en un array**.

### find() - Encuentra el primer elemento

```javascript
const usuarios = [
    { id: 1, nombre: "Ana" },
    { id: 2, nombre: "Juan" },
    { id: 3, nombre: "María" }
];

const usuario = usuarios.find(u => u.id === 2);
console.log(usuario); // { id: 2, nombre: "Juan" }

const noExiste = usuarios.find(u => u.id === 99);
console.log(noExiste); // undefined
```

### findIndex() - Encuentra el índice

```javascript
const numeros = [10, 20, 30, 40, 50];

const indice = numeros.findIndex(num => num > 25);
console.log(indice); // 2 (el índice de 30)

const noEncontrado = numeros.findIndex(num => num > 100);
console.log(noEncontrado); // -1
```

### Ejemplo práctico: Actualizar elemento

```javascript
const productos = [
    { id: 1, nombre: "Laptop", stock: 5 },
    { id: 2, nombre: "Mouse", stock: 20 },
    { id: 3, nombre: "Teclado", stock: 15 }
];

function actualizarStock(id, nuevoStock) {
    const indice = productos.findIndex(p => p.id === id);
    
    if (indice !== -1) {
        productos[indice].stock = nuevoStock;
        return `Stock actualizado para ${productos[indice].nombre}`;
    }
    
    return "Producto no encontrado";
}

console.log(actualizarStock(2, 25));
// "Stock actualizado para Mouse"
console.log(productos[1].stock); // 25
```

---

## some() y every()

**Verificar condiciones en el array**.

### some() - Al menos uno cumple

```javascript
const numeros = [1, 3, 5, 8, 9];

const hayPares = numeros.some(num => num % 2 === 0);
console.log(hayPares); // true (8 es par)

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
```

### Ejemplo práctico: Validación

```javascript
const usuarios = [
    { nombre: "Ana", edad: 25, email: "ana@ejemplo.com" },
    { nombre: "Juan", edad: 30, email: "juan@ejemplo.com" },
    { nombre: "María", edad: 17, email: "maria@ejemplo.com" }
];

// ¿Hay algún menor de edad?
const hayMenores = usuarios.some(u => u.edad < 18);
console.log(hayMenores); // true

// ¿Todos tienen email?
const todosTienenEmail = usuarios.every(u => u.email && u.email.includes("@"));
console.log(todosTienenEmail); // true

// ¿Todos son mayores de edad?
const todosMayores = usuarios.every(u => u.edad >= 18);
console.log(todosMayores); // false
```

### Validar formulario

```javascript
const camposFormulario = [
    { nombre: "email", valor: "usuario@ejemplo.com", requerido: true },
    { nombre: "nombre", valor: "Ana", requerido: true },
    { nombre: "telefono", valor: "", requerido: false },
    { nombre: "edad", valor: "25", requerido: true }
];

const formularioValido = camposFormulario.every(campo => {
    if (campo.requerido) {
        return campo.valor.trim() !== "";
    }
    return true;
});

console.log(formularioValido); // true

const hayErrores = camposFormulario.some(campo => {
    return campo.requerido && campo.valor.trim() === "";
});

console.log(hayErrores); // false
```

---

## flat() y flatMap()

**Trabajar con arrays anidados**.

### flat() - Aplanar arrays

```javascript
const arrayAnidado = [1, 2, [3, 4], [5, [6, 7]]];

// Aplanar un nivel
const nivel1 = arrayAnidado.flat();
console.log(nivel1); // [1, 2, 3, 4, 5, [6, 7]]

// Aplanar dos niveles
const nivel2 = arrayAnidado.flat(2);
console.log(nivel2); // [1, 2, 3, 4, 5, 6, 7]

// Aplanar todos los niveles
const todoAplanado = arrayAnidado.flat(Infinity);
console.log(todoAplanado); // [1, 2, 3, 4, 5, 6, 7]
```

### Eliminar elementos vacíos

```javascript
const conVacios = [1, 2, , , 3, , 4];

const sinVacios = conVacios.flat();
console.log(sinVacios); // [1, 2, 3, 4]
```

### flatMap() - map() + flat()

```javascript
const frases = ["Hola mundo", "Adiós amigo"];

// Con map + flat
const palabrasMap = frases.map(frase => frase.split(" ")).flat();
console.log(palabrasMap); // ["Hola", "mundo", "Adiós", "amigo"]

// Con flatMap (más eficiente)
const palabrasFlatMap = frases.flatMap(frase => frase.split(" "));
console.log(palabrasFlatMap); // ["Hola", "mundo", "Adiós", "amigo"]
```

### Ejemplo práctico: Etiquetas

```javascript
const articulos = [
    { titulo: "JavaScript", tags: ["programación", "web", "frontend"] },
    { titulo: "Node.js", tags: ["programación", "backend", "servidor"] },
    { titulo: "React", tags: ["frontend", "library", "UI"] }
];

// Obtener todas las etiquetas únicas
const todasLasTags = articulos
    .flatMap(articulo => articulo.tags)
    .filter((tag, indice, array) => array.indexOf(tag) === indice);

console.log(todasLasTags);
// ["programación", "web", "frontend", "backend", "servidor", "library", "UI"]
```

---

## sort() avanzado

**Ordenar arrays** (⚠️ **Muta el array original**).

### Ordenar números

```javascript
const numeros = [3, 1, 4, 1, 5, 9, 2, 6];

// ❌ Error común: sort sin comparador
console.log(numeros.sort()); // [1, 1, 2, 3, 4, 5, 6, 9] (funciona por suerte)

const numerosRaros = [10, 5, 40, 25, 1000, 1];
console.log(numerosRaros.sort()); // [1, 10, 1000, 25, 40, 5] ❌ (orden de strings)

// ✅ Correcto: con función comparadora
console.log(numerosRaros.sort((a, b) => a - b)); // [1, 5, 10, 25, 40, 1000]

// Descendente
console.log(numerosRaros.sort((a, b) => b - a)); // [1000, 40, 25, 10, 5, 1]
```

### Ordenar strings

```javascript
const frutas = ["banana", "manzana", "uva", "pera"];

// Ascendente
frutas.sort();
console.log(frutas); // ["banana", "manzana", "pera", "uva"]

// Descendente
frutas.sort((a, b) => b.localeCompare(a));
console.log(frutas); // ["uva", "pera", "manzana", "banana"]

// Case-insensitive
const palabras = ["Zebra", "apple", "Banana"];
palabras.sort((a, b) => a.toLowerCase().localeCompare(b.toLowerCase()));
console.log(palabras); // ["apple", "Banana", "Zebra"]
```

### Ordenar objetos

```javascript
const usuarios = [
    { nombre: "María", edad: 28 },
    { nombre: "Ana", edad: 25 },
    { nombre: "Juan", edad: 30 }
];

// Por edad (ascendente)
usuarios.sort((a, b) => a.edad - b.edad);
console.log(usuarios);
// [{ Ana, 25 }, { María, 28 }, { Juan, 30 }]

// Por nombre (alfabético)
usuarios.sort((a, b) => a.nombre.localeCompare(b.nombre));
console.log(usuarios);
// [{ Ana... }, { Juan... }, { María... }]
```

### Ordenamiento múltiple

```javascript
const productos = [
    { nombre: "Laptop", precio: 1200, stock: 5 },
    { nombre: "Mouse", precio: 25, stock: 0 },
    { nombre: "Teclado", precio: 80, stock: 15 },
    { nombre: "Monitor", precio: 300, stock: 0 }
];

// Ordenar por stock (con stock primero), luego por precio
productos.sort((a, b) => {
    // Primero por stock (descendente)
    if (b.stock !== a.stock) {
        return b.stock - a.stock;
    }
    // Si tienen mismo stock, por precio (ascendente)
    return a.precio - b.precio;
});

console.log(productos);
// [{ Teclado, stock: 15 }, { Laptop, stock: 5 }, { Mouse, stock: 0 }, { Monitor, stock: 0 }]
```

### No mutar el original

```javascript
const numeros = [3, 1, 4, 1, 5, 9];

// ✅ Crear copia antes de ordenar
const ordenados = [...numeros].sort((a, b) => a - b);

console.log(numeros);    // [3, 1, 4, 1, 5, 9] (original sin cambios)
console.log(ordenados);  // [1, 1, 3, 4, 5, 9]
```

---

## Encadenar métodos

**Combinar múltiples métodos para operaciones complejas**.

### Ejemplo básico

```javascript
const numeros = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10];

const resultado = numeros
    .filter(num => num % 2 === 0)  // Pares: [2, 4, 6, 8, 10]
    .map(num => num * 2)            // Duplicar: [4, 8, 12, 16, 20]
    .reduce((acc, num) => acc + num, 0); // Sumar: 60

console.log(resultado); // 60
```

### Procesamiento complejo

```javascript
const usuarios = [
    { nombre: "Ana García", edad: 17, ciudad: "Madrid", activo: true },
    { nombre: "Juan Pérez", edad: 30, ciudad: "Barcelona", activo: true },
    { nombre: "María López", edad: 28, ciudad: "Madrid", activo: false },
    { nombre: "Pedro Sánchez", edad: 25, ciudad: "Madrid", activo: true }
];

const resultado = usuarios
    .filter(u => u.edad >= 18)           // Mayores de edad
    .filter(u => u.activo)               // Activos
    .filter(u => u.ciudad === "Madrid")  // De Madrid
    .map(u => u.nombre.split(" ")[0])    // Solo nombres
    .sort();                              // Ordenar

console.log(resultado); // ["Pedro"]
```

### Ejemplo práctico: Análisis de ventas

```javascript
const ventas = [
    { producto: "Laptop", precio: 1200, cantidad: 2, categoria: "Electrónica" },
    { producto: "Mouse", precio: 25, cantidad: 5, categoria: "Electrónica" },
    { producto: "Silla", precio: 300, cantidad: 3, categoria: "Muebles" },
    { producto: "Mesa", precio: 500, cantidad: 1, categoria: "Muebles" },
    { producto: "Teclado", precio: 80, cantidad: 4, categoria: "Electrónica" }
];

// Top 3 productos por ingreso total
const top3 = ventas
    .map(venta => ({
        ...venta,
        total: venta.precio * venta.cantidad
    }))
    .sort((a, b) => b.total - a.total)
    .slice(0, 3)
    .map(venta => ({
        producto: venta.producto,
        ingresos: `$${venta.total}`
    }));

console.log(top3);
// [
//   { producto: "Laptop", ingresos: "$2400" },
//   { producto: "Silla", ingresos: "$900" },
//   { producto: "Mesa", ingresos: "$500" }
// ]

// Ingresos por categoría
const ingresosPorCategoria = ventas
    .reduce((acc, venta) => {
        const cat = venta.categoria;
        const total = venta.precio * venta.cantidad;
        acc[cat] = (acc[cat] || 0) + total;
        return acc;
    }, {});

console.log(ingresosPorCategoria);
// { Electrónica: 2845, Muebles: 1400 }
```

### Encadenamiento legible

```javascript
const datos = [
    { nombre: "Ana", edad: 25, salario: 3000 },
    { nombre: "Juan", edad: 30, salario: 4000 },
    { nombre: "María", edad: 28, salario: 3500 }
];

// ✅ Formato legible
const resultado = datos
    .filter(persona => persona.edad >= 25)
    .map(persona => ({
        nombre: persona.nombre,
        salarioAnual: persona.salario * 12
    }))
    .sort((a, b) => b.salarioAnual - a.salarioAnual);

console.log(resultado);
```

---

## Ejercicios prácticos

### Ejercicio 1: Transformar productos
**Nivel**: ⭐⭐☆☆☆

Dado un array de productos, crea uno nuevo con precios aumentados un 15% e incluye IVA.

```javascript
const productos = [
    { nombre: "Laptop", precio: 1000 },
    { nombre: "Mouse", precio: 20 },
    { nombre: "Teclado", precio: 75 }
];

// Resultado esperado:
// [
//   { nombre: "Laptop", precio: 1000, precioFinal: 1380.50 },
//   ...
// ]
```

<details>
<summary>Ver solución</summary>

```javascript
const productos = [
    { nombre: "Laptop", precio: 1000 },
    { nombre: "Mouse", precio: 20 },
    { nombre: "Teclado", precio: 75 }
];

const productosActualizados = productos.map(producto => {
    const precioAumentado = producto.precio * 1.15;
    const precioConIVA = precioAumentado * 1.21;
    
    return {
        ...producto,
        precioFinal: Math.round(precioConIVA * 100) / 100
    };
});

console.log(productosActualizados);
// [
//   { nombre: "Laptop", precio: 1000, precioFinal: 1391.50 },
//   { nombre: "Mouse", precio: 20, precioFinal: 27.83 },
//   { nombre: "Teclado", precio: 75, precioFinal: 104.44 }
// ]
```

</details>

---

### Ejercicio 2: Filtrar y calcular promedio
**Nivel**: ⭐⭐☆☆☆

Filtra estudiantes aprobados (nota >= 6) y calcula su promedio.

```javascript
const estudiantes = [
    { nombre: "Ana", nota: 8 },
    { nombre: "Juan", nota: 5 },
    { nombre: "María", nota: 9 },
    { nombre: "Pedro", nota: 4 },
    { nombre: "Laura", nota: 7 }
];
```

<details>
<summary>Ver solución</summary>

```javascript
const estudiantes = [
    { nombre: "Ana", nota: 8 },
    { nombre: "Juan", nota: 5 },
    { nombre: "María", nota: 9 },
    { nombre: "Pedro", nota: 4 },
    { nombre: "Laura", nota: 7 }
];

const aprobados = estudiantes.filter(est => est.nota >= 6);

const promedio = aprobados.reduce((acc, est, _, array) => {
    return acc + est.nota / array.length;
}, 0);

console.log("Estudiantes aprobados:", aprobados);
// [{ Ana, 8 }, { María, 9 }, { Laura, 7 }]

console.log("Promedio:", promedio.toFixed(2));
// "Promedio: 8.00"
```

</details>

---

### Ejercicio 3: Buscar producto más caro
**Nivel**: ⭐⭐⭐☆☆

Encuentra el producto más caro de cada categoría.

```javascript
const productos = [
    { nombre: "Laptop HP", categoria: "Electrónica", precio: 1200 },
    { nombre: "Laptop Dell", categoria: "Electrónica", precio: 1500 },
    { nombre: "Silla Gaming", categoria: "Muebles", precio: 300 },
    { nombre: "Mesa Oficina", categoria: "Muebles", precio: 500 },
    { nombre: "Mouse", categoria: "Electrónica", precio: 25 }
];
```

<details>
<summary>Ver solución</summary>

```javascript
const productos = [
    { nombre: "Laptop HP", categoria: "Electrónica", precio: 1200 },
    { nombre: "Laptop Dell", categoria: "Electrónica", precio: 1500 },
    { nombre: "Silla Gaming", categoria: "Muebles", precio: 300 },
    { nombre: "Mesa Oficina", categoria: "Muebles", precio: 500 },
    { nombre: "Mouse", categoria: "Electrónica", precio: 25 }
];

const masCarosPorCategoria = productos.reduce((acc, producto) => {
    const cat = producto.categoria;
    
    if (!acc[cat] || producto.precio > acc[cat].precio) {
        acc[cat] = producto;
    }
    
    return acc;
}, {});

console.log(masCarosPorCategoria);
// {
//   Electrónica: { nombre: "Laptop Dell", ... precio: 1500 },
//   Muebles: { nombre: "Mesa Oficina", ... precio: 500 }
// }
```

</details>

---

### Ejercicio 4: Estadísticas completas
**Nivel**: ⭐⭐⭐⭐☆

Calcula estadísticas completas de un array de números.

```javascript
const numeros = [10, 5, 8, 15, 3, 20, 7, 12];

// Debe calcular: suma, promedio, mínimo, máximo, mediana
```

<details>
<summary>Ver solución</summary>

```javascript
const numeros = [10, 5, 8, 15, 3, 20, 7, 12];

function calcularEstadisticas(nums) {
    const suma = nums.reduce((acc, num) => acc + num, 0);
    const promedio = suma / nums.length;
    
    const minimo = nums.reduce((min, num) => num < min ? num : min, nums[0]);
    const maximo = nums.reduce((max, num) => num > max ? num : max, nums[0]);
    
    const ordenados = [...nums].sort((a, b) => a - b);
    const medio = Math.floor(ordenados.length / 2);
    const mediana = ordenados.length % 2 === 0
        ? (ordenados[medio - 1] + ordenados[medio]) / 2
        : ordenados[medio];
    
    return {
        cantidad: nums.length,
        suma,
        promedio: Math.round(promedio * 100) / 100,
        minimo,
        maximo,
        mediana,
        rango: maximo - minimo
    };
}

const estadisticas = calcularEstadisticas(numeros);
console.log(estadisticas);
// {
//   cantidad: 8,
//   suma: 80,
//   promedio: 10,
//   minimo: 3,
//   maximo: 20,
//   mediana: 9,
//   rango: 17
// }
```

</details>

---

### Ejercicio 5: Procesar pedidos
**Nivel**: ⭐⭐⭐⭐☆

Sistema completo de procesamiento de pedidos.

```javascript
const pedidos = [
    {
        id: 1,
        cliente: "Ana",
        items: [
            { producto: "Laptop", precio: 1200, cantidad: 1 },
            { producto: "Mouse", precio: 25, cantidad: 2 }
        ],
        estado: "completado"
    },
    {
        id: 2,
        cliente: "Juan",
        items: [
            { producto: "Teclado", precio: 80, cantidad: 1 }
        ],
        estado: "completado"
    },
    {
        id: 3,
        cliente: "María",
        items: [
            { producto: "Monitor", precio: 300, cantidad: 2 }
        ],
        estado: "pendiente"
    }
];

// Calcular:
// 1. Total de ventas (solo completados)
// 2. Cliente con mayor gasto
// 3. Producto más vendido
```

<details>
<summary>Ver solución</summary>

```javascript
const pedidos = [
    {
        id: 1,
        cliente: "Ana",
        items: [
            { producto: "Laptop", precio: 1200, cantidad: 1 },
            { producto: "Mouse", precio: 25, cantidad: 2 }
        ],
        estado: "completado"
    },
    {
        id: 2,
        cliente: "Juan",
        items: [
            { producto: "Teclado", precio: 80, cantidad: 1 }
        ],
        estado: "completado"
    },
    {
        id: 3,
        cliente: "María",
        items: [
            { producto: "Monitor", precio: 300, cantidad: 2 }
        ],
        estado: "pendiente"
    }
];

// 1. Total de ventas (solo completados)
const totalVentas = pedidos
    .filter(pedido => pedido.estado === "completado")
    .flatMap(pedido => pedido.items)
    .reduce((acc, item) => acc + (item.precio * item.cantidad), 0);

console.log(`Total ventas: $${totalVentas}`);
// "Total ventas: $1330"

// 2. Cliente con mayor gasto
const gastoPorCliente = pedidos
    .filter(pedido => pedido.estado === "completado")
    .map(pedido => ({
        cliente: pedido.cliente,
        total: pedido.items.reduce((acc, item) => 
            acc + (item.precio * item.cantidad), 0)
    }))
    .sort((a, b) => b.total - a.total);

console.log("Cliente top:", gastoPorCliente[0]);
// { cliente: "Ana", total: 1250 }

// 3. Producto más vendido
const ventasPorProducto = pedidos
    .filter(pedido => pedido.estado === "completado")
    .flatMap(pedido => pedido.items)
    .reduce((acc, item) => {
        acc[item.producto] = (acc[item.producto] || 0) + item.cantidad;
        return acc;
    }, {});

const productoMasVendido = Object.entries(ventasPorProducto)
    .sort((a, b) => b[1] - a[1])[0];

console.log(`Producto más vendido: ${productoMasVendido[0]} (${productoMasVendido[1]} unidades)`);
// "Producto más vendido: Mouse (2 unidades)"
```

</details>

---

### Ejercicio 6: Normalizar datos de API
**Nivel**: ⭐⭐⭐⭐⭐

Procesa datos crudos de una API a formato utilizable.

```javascript
const datosAPI = [
    { user_name: "  ANA garcía  ", user_age: "25", user_email: "ANA@EJEMPLO.COM", user_active: "true" },
    { user_name: "juan  PÉREZ", user_age: "30", user_email: "juan@ejemplo.com", user_active: "false" },
    { user_name: "MARÍA lópez ", user_age: "invalid", user_email: "maria@ejemplo", user_active: "true" }
];

// Debe:
// - Limpiar nombres (trim, capitalizar)
// - Convertir edad a número (usar 0 si inválido)
// - Validar email
// - Convertir active a boolean
// - Cambiar nombres de propiedades (user_name → nombre)
```

<details>
<summary>Ver solución</summary>

```javascript
const datosAPI = [
    { user_name: "  ANA garcía  ", user_age: "25", user_email: "ANA@EJEMPLO.COM", user_active: "true" },
    { user_name: "juan  PÉREZ", user_age: "30", user_email: "juan@ejemplo.com", user_active: "false" },
    { user_name: "MARÍA lópez ", user_age: "invalid", user_email: "maria@ejemplo", user_active: "true" }
];

function capitalizarNombre(nombre) {
    return nombre
        .toLowerCase()
        .split(' ')
        .map(palabra => palabra.charAt(0).toUpperCase() + palabra.slice(1))
        .join(' ');
}

function validarEmail(email) {
    return email.includes('@') && email.includes('.');
}

const datosNormalizados = datosAPI
    .map(usuario => ({
        nombre: capitalizarNombre(usuario.user_name.trim()),
        edad: isNaN(Number(usuario.user_age)) ? 0 : Number(usuario.user_age),
        email: usuario.user_email.toLowerCase().trim(),
        emailValido: validarEmail(usuario.user_email),
        activo: usuario.user_active === "true"
    }))
    .filter(usuario => usuario.emailValido) // Solo emails válidos
    .sort((a, b) => a.nombre.localeCompare(b.nombre)); // Ordenar por nombre

console.log(datosNormalizados);
// [
//   { nombre: "Ana García", edad: 25, email: "ana@ejemplo.com", emailValido: true, activo: true },
//   { nombre: "Juan Pérez", edad: 30, email: "juan@ejemplo.com", emailValido: true, activo: false }
// ]
// (María filtrada por email inválido)
```

</details>

---

## Errores comunes

### ❌ Error 1: No retornar en map

```javascript
const numeros = [1, 2, 3];

// ❌ No retorna nada
const dobles = numeros.map(num => {
    num * 2; // Falta return
});
console.log(dobles); // [undefined, undefined, undefined]

// ✅ Correcto
const dobles = numeros.map(num => num * 2);
// O con llaves:
const dobles = numeros.map(num => {
    return num * 2;
});
```

---

### ❌ Error 2: Mutar en map/filter

```javascript
const usuarios = [
    { nombre: "Ana", edad: 25 },
    { nombre: "Juan", edad: 30 }
];

// ❌ Mutar el original
const conEmail = usuarios.map(usuario => {
    usuario.email = `${usuario.nombre}@ejemplo.com`; // Mutación
    return usuario;
});

console.log(usuarios[0]); // { nombre: "Ana", edad: 25, email: "ana@ejemplo.com" } ❌

// ✅ Crear nuevo objeto
const conEmail = usuarios.map(usuario => ({
    ...usuario,
    email: `${usuario.nombre}@ejemplo.com`
}));
```

---

### ❌ Error 3: Olvidar valor inicial en reduce

```javascript
const numeros = [1, 2, 3, 4];

// ⚠️ Sin valor inicial (puede funcionar)
const suma = numeros.reduce((acc, num) => acc + num);

// ❌ Problema con arrays vacíos
const vacio = [];
const sumaVacio = vacio.reduce((acc, num) => acc + num); // Error!

// ✅ Siempre con valor inicial
const suma = numeros.reduce((acc, num) => acc + num, 0);
const sumaVacio = vacio.reduce((acc, num) => acc + num, 0); // 0
```

---

### ❌ Error 4: sort() sin comparador para números

```javascript
const numeros = [10, 5, 40, 25, 1000, 1];

// ❌ Ordena como strings
numeros.sort();
console.log(numeros); // [1, 10, 1000, 25, 40, 5]

// ✅ Con comparador numérico
numeros.sort((a, b) => a - b);
console.log(numeros); // [1, 5, 10, 25, 40, 1000]
```

---

## Buenas prácticas

### ✅ 1. Usa const para arrays inmutables

```javascript
// ✅ map, filter, reduce no mutan
const numeros = [1, 2, 3];
const dobles = numeros.map(num => num * 2);

// ⚠️ sort muta el original
const numerosOrdenados = [...numeros].sort((a, b) => a - b);
```

---

### ✅ 2. Nombres descriptivos

```javascript
// ❌ No claro
usuarios.filter(u => u.a >= 18);

// ✅ Claro
usuarios.filter(usuario => usuario.edad >= 18);

// ✅ Aún mejor con función nombrada
const esMayorDeEdad = usuario => usuario.edad >= 18;
usuarios.filter(esMayorDeEdad);
```

---

### ✅ 3. Encadena con criterio

```javascript
// ❌ Encadenamiento excesivo
const resultado = datos
    .filter(x => x.a)
    .map(x => x.b)
    .filter(x => x.c)
    .map(x => x.d)
    .reduce((acc, x) => acc + x, 0);

// ✅ Divide operaciones complejas
const conA = datos.filter(x => x.a);
const valioresB = conA.map(x => x.b);
const conC = valoresB.filter(x => x.c);
const resultado = conC.reduce((acc, x) => acc + x.d, 0);
```

---

### ✅ 4. Evita efectos secundarios

```javascript
// ❌ Efecto secundario en map
let total = 0;
const dobles = numeros.map(num => {
    total += num; // Efecto secundario
    return num * 2;
});

// ✅ Sin efectos secundarios
const dobles = numeros.map(num => num * 2);
const total = numeros.reduce((acc, num) => acc + num, 0);
```

---

### ✅ 5. Usa métodos específicos

```javascript
// ❌ filter + [0]
const usuario = usuarios.filter(u => u.id === 1)[0];

// ✅ find
const usuario = usuarios.find(u => u.id === 1);

// ❌ filter + length
const hayMenores = usuarios.filter(u => u.edad < 18).length > 0;

// ✅ some
const hayMenores = usuarios.some(u => u.edad < 18);
```

---

## Cheatsheet

### map() - Transformar

```javascript
[1, 2, 3].map(x => x * 2); // [2, 4, 6]
```

### filter() - Filtrar

```javascript
[1, 2, 3, 4].filter(x => x > 2); // [3, 4]
```

### reduce() - Reducir

```javascript
[1, 2, 3].reduce((acc, x) => acc + x, 0); // 6
```

### find() - Encontrar

```javascript
[1, 2, 3].find(x => x > 1); // 2
```

### findIndex() - Índice

```javascript
[1, 2, 3].findIndex(x => x > 1); // 1
```

### some() - Al menos uno

```javascript
[1, 2, 3].some(x => x > 2); // true
```

### every() - Todos

```javascript
[2, 4, 6].every(x => x % 2 === 0); // true
```

### flat() - Aplanar

```javascript
[1, [2, 3]].flat(); // [1, 2, 3]
```

### flatMap() - map + flat

```javascript
["a b", "c d"].flatMap(s => s.split(" ")); // ["a", "b", "c", "d"]
```

### sort() - Ordenar

```javascript
[3, 1, 2].sort((a, b) => a - b); // [1, 2, 3]
```

### Encadenar

```javascript
numeros
    .filter(x => x > 0)
    .map(x => x * 2)
    .reduce((acc, x) => acc + x, 0);
```

---

## Siguiente paso

Has dominado los métodos de array más potentes de JavaScript. Ahora aprenderás **destructuring y spread operators**.

→ [29-destructuring-spread.md](29-destructuring-spread.md)

Ahí descubrirás cómo extraer valores de arrays y objetos de forma elegante y trabajar con copias de datos.

# Módulo 29: Destructuring y Spread Operator

## Índice
1. [Introducción](#introducción)
2. [Destructuring de arrays](#destructuring-de-arrays)
3. [Destructuring de objetos](#destructuring-de-objetos)
4. [Valores por defecto](#valores-por-defecto)
5. [Destructuring anidado](#destructuring-anidado)
6. [Spread operator en arrays](#spread-operator-en-arrays)
7. [Spread operator en objetos](#spread-operator-en-objetos)
8. [Rest operator](#rest-operator)
9. [Casos prácticos](#casos-prácticos)
10. [Ejercicios prácticos](#ejercicios-prácticos)
11. [Errores comunes](#errores-comunes)
12. [Buenas prácticas](#buenas-prácticas)
13. [Cheatsheet](#cheatsheet)

---

## Introducción

**Destructuring** y **Spread/Rest operators** son características de ES6 que simplifican el trabajo con arrays y objetos.

### Analogía: Desempacar cajas

```
Array/Objeto → Caja cerrada con contenido
Destructuring → Abrir y sacar elementos específicos
Spread → Esparcir todo el contenido
Rest → Recoger lo que sobra
```

### ¿Por qué usarlos?

✅ **Código más limpio y legible**  
✅ **Menos líneas de código**  
✅ **Extracción de valores más fácil**  
✅ **Copias y combinaciones simples**

---

## Destructuring de arrays

**Extraer valores de un array en variables individuales**.

### Sintaxis básica

```javascript
// ❌ Forma tradicional
const numeros = [1, 2, 3];
const primero = numeros[0];
const segundo = numeros[1];
const tercero = numeros[2];

// ✅ Con destructuring
const [primero, segundo, tercero] = [1, 2, 3];
console.log(primero);  // 1
console.log(segundo);  // 2
console.log(tercero);  // 3
```

### Saltar elementos

```javascript
const colores = ["rojo", "verde", "azul", "amarillo"];

// Saltar el segundo elemento
const [primero, , tercero] = colores;
console.log(primero);  // "rojo"
console.log(tercero);  // "azul"
```

### Intercambiar variables

```javascript
let a = 1;
let b = 2;

// ❌ Forma tradicional (con variable temporal)
let temp = a;
a = b;
b = temp;

// ✅ Con destructuring
[a, b] = [b, a];
console.log(a);  // 2
console.log(b);  // 1
```

### Retornar múltiples valores

```javascript
function obtenerCoordenadas() {
    return [40.7128, -74.0060];
}

const [latitud, longitud] = obtenerCoordenadas();
console.log(`Lat: ${latitud}, Lon: ${longitud}`);
// "Lat: 40.7128, Lon: -74.006"
```

### Ejemplo práctico: Split

```javascript
const fechaCompleta = "2024-01-15";

const [año, mes, dia] = fechaCompleta.split("-");
console.log(`Día: ${dia}, Mes: ${mes}, Año: ${año}`);
// "Día: 15, Mes: 01, Año: 2024"
```

---

## Destructuring de objetos

**Extraer propiedades de un objeto en variables**.

### Sintaxis básica

```javascript
// ❌ Forma tradicional
const usuario = {
    nombre: "Ana",
    edad: 25,
    ciudad: "Madrid"
};

const nombre = usuario.nombre;
const edad = usuario.edad;
const ciudad = usuario.ciudad;

// ✅ Con destructuring
const { nombre, edad, ciudad } = usuario;
console.log(nombre);  // "Ana"
console.log(edad);    // 25
console.log(ciudad);  // "Madrid"
```

### Renombrar variables

```javascript
const usuario = {
    nombre: "Ana",
    edad: 25
};

// Renombrar al extraer
const { nombre: nombreUsuario, edad: edadUsuario } = usuario;
console.log(nombreUsuario);  // "Ana"
console.log(edadUsuario);    // 25
```

### Propiedades específicas

```javascript
const producto = {
    id: 1,
    nombre: "Laptop",
    precio: 1200,
    stock: 5,
    categoria: "Electrónica"
};

// Solo extraer lo que necesitas
const { nombre, precio } = producto;
console.log(`${nombre}: $${precio}`);
// "Laptop: $1200"
```

### En parámetros de función

```javascript
// ❌ Forma tradicional
function mostrarUsuario(usuario) {
    console.log(`${usuario.nombre} tiene ${usuario.edad} años`);
}

// ✅ Con destructuring
function mostrarUsuario({ nombre, edad }) {
    console.log(`${nombre} tiene ${edad} años`);
}

mostrarUsuario({ nombre: "Ana", edad: 25 });
// "Ana tiene 25 años"
```

### Ejemplo práctico: API Response

```javascript
function procesarRespuesta(response) {
    const { data, status, message } = response;
    
    if (status === "success") {
        console.log("Datos:", data);
    } else {
        console.error("Error:", message);
    }
}

procesarRespuesta({
    status: "success",
    data: { id: 1, nombre: "Ana" },
    message: null
});
```

---

## Valores por defecto

**Asignar valores predeterminados si la propiedad no existe**.

### Arrays con defaults

```javascript
const colores = ["rojo"];

const [primero = "negro", segundo = "blanco"] = colores;
console.log(primero);   // "rojo" (existe)
console.log(segundo);   // "blanco" (default)
```

### Objetos con defaults

```javascript
const usuario = {
    nombre: "Ana"
    // edad no existe
};

const { nombre, edad = 18, ciudad = "Madrid" } = usuario;
console.log(nombre);  // "Ana"
console.log(edad);    // 18 (default)
console.log(ciudad);  // "Madrid" (default)
```

### En funciones

```javascript
function crearUsuario({ nombre, edad = 18, rol = "usuario" }) {
    return {
        nombre,
        edad,
        rol,
        activo: true
    };
}

console.log(crearUsuario({ nombre: "Ana" }));
// { nombre: "Ana", edad: 18, rol: "usuario", activo: true }

console.log(crearUsuario({ nombre: "Juan", edad: 30, rol: "admin" }));
// { nombre: "Juan", edad: 30, rol: "admin", activo: true }
```

### Combinado con renombre

```javascript
const config = {
    host: "localhost"
    // port no existe
};

const { host: servidor, port: puerto = 3000 } = config;
console.log(servidor);  // "localhost"
console.log(puerto);    // 3000 (default)
```

---

## Destructuring anidado

**Extraer valores de estructuras anidadas**.

### Arrays anidados

```javascript
const matriz = [
    [1, 2],
    [3, 4],
    [5, 6]
];

const [[a, b], [c, d]] = matriz;
console.log(a, b, c, d);  // 1 2 3 4
```

### Objetos anidados

```javascript
const usuario = {
    nombre: "Ana",
    direccion: {
        calle: "Gran Vía",
        numero: 123,
        ciudad: "Madrid"
    },
    contacto: {
        email: "ana@ejemplo.com",
        telefono: "123456789"
    }
};

// Extraer propiedades anidadas
const {
    nombre,
    direccion: { ciudad, calle },
    contacto: { email }
} = usuario;

console.log(nombre);   // "Ana"
console.log(ciudad);   // "Madrid"
console.log(calle);    // "Gran Vía"
console.log(email);    // "ana@ejemplo.com"
```

### Con renombre y defaults

```javascript
const producto = {
    nombre: "Laptop",
    detalles: {
        precio: 1200
        // stock no existe
    }
};

const {
    nombre: nombreProducto,
    detalles: { precio, stock = 0 }
} = producto;

console.log(nombreProducto);  // "Laptop"
console.log(precio);          // 1200
console.log(stock);           // 0 (default)
```

### Ejemplo práctico: Datos de usuario

```javascript
const respuestaAPI = {
    status: 200,
    data: {
        user: {
            id: 1,
            profile: {
                name: "Ana García",
                age: 25,
                location: {
                    city: "Madrid",
                    country: "España"
                }
            }
        }
    }
};

const {
    status,
    data: {
        user: {
            id,
            profile: {
                name,
                location: { city }
            }
        }
    }
} = respuestaAPI;

console.log(`Usuario ${id}: ${name} de ${city}`);
// "Usuario 1: Ana García de Madrid"
```

---

## Spread operator en arrays

**Expandir elementos de un array** (`...`).

### Copiar arrays

```javascript
const original = [1, 2, 3];

// ❌ Referencia (no copia)
const referen = original;
referencia[0] = 999;
console.log(original);  // [999, 2, 3] ❌

// ✅ Copia con spread
const copia = [...original];
copia[0] = 999;
console.log(original);  // [1, 2, 3] ✅
console.log(copia);     // [999, 2, 3]
```

### Concatenar arrays

```javascript
const arr1 = [1, 2, 3];
const arr2 = [4, 5, 6];

// ❌ Forma tradicional
const concatenado = arr1.concat(arr2);

// ✅ Con spread
const concatenado = [...arr1, ...arr2];
console.log(concatenado);  // [1, 2, 3, 4, 5, 6]

// Intercalar elementos
const mezclado = [...arr1, "x", ...arr2, "y"];
console.log(mezclado);  // [1, 2, 3, "x", 4, 5, 6, "y"]
```

### Agregar elementos

```javascript
const numeros = [2, 3, 4];

// Al inicio
const conInicio = [1, ...numeros];
console.log(conInicio);  // [1, 2, 3, 4]

// Al final
const conFinal = [...numeros, 5];
console.log(conFinal);  // [2, 3, 4, 5]

// En el medio
const enMedio = [...numeros.slice(0, 2), 999, ...numeros.slice(2)];
console.log(enMedio);  // [2, 3, 999, 4]
```

### Convertir string a array

```javascript
const texto = "Hola";
const letras = [...texto];
console.log(letras);  // ["H", "o", "l", "a"]
```

### Math con arrays

```javascript
const numeros = [5, 10, 3, 8, 1];

const maximo = Math.max(...numeros);
const minimo = Math.min(...numeros);

console.log(maximo);  // 10
console.log(minimo);  // 1
```

### Ejemplo práctico: Eliminar duplicados

```javascript
const conDuplicados = [1, 2, 2, 3, 4, 4, 5];

const sinDuplicados = [...new Set(conDuplicados)];
console.log(sinDuplicados);  // [1, 2, 3, 4, 5]
```

---

## Spread operator en objetos

**Expandir propiedades de un objeto**.

### Copiar objetos

```javascript
const original = { nombre: "Ana", edad: 25 };

// ✅ Copia superficial
const copia = { ...original };
copia.nombre = "Juan";

console.log(original.nombre);  // "Ana" (no afectado)
console.log(copia.nombre);     // "Juan"
```

### Combinar objetos

```javascript
const persona = { nombre: "Ana", edad: 25 };
const contacto = { email: "ana@ejemplo.com", telefono: "123456789" };

const completo = { ...persona, ...contacto };
console.log(completo);
// {
//   nombre: "Ana",
//   edad: 25,
//   email: "ana@ejemplo.com",
//   telefono: "123456789"
// }
```

### Sobrescribir propiedades

```javascript
const original = { nombre: "Ana", edad: 25, ciudad: "Madrid" };

// El orden importa (último valor gana)
const actualizado = { ...original, edad: 26, ciudad: "Barcelona" };
console.log(actualizado);
// { nombre: "Ana", edad: 26, ciudad: "Barcelona" }
```

### Agregar propiedades

```javascript
const usuario = { nombre: "Ana", edad: 25 };

const conId = { id: 1, ...usuario };
console.log(conId);
// { id: 1, nombre: "Ana", edad: 25 }

const conTimestamp = {
    ...usuario,
    createdAt: new Date().toISOString()
};
console.log(conTimestamp);
```

### Eliminar propiedades

```javascript
const usuario = {
    id: 1,
    nombre: "Ana",
    password: "secreto123",
    email: "ana@ejemplo.com"
};

// Extraer password, mantener el resto
const { password, ...usuarioSeguro } = usuario;
console.log(usuarioSeguro);
// { id: 1, nombre: "Ana", email: "ana@ejemplo.com" }
```

### Ejemplo práctico: Actualizar estado

```javascript
const estadoInicial = {
    usuario: { nombre: "Ana", edad: 25 },
    autenticado: false,
    token: null
};

// Actualizar solo ciertas propiedades
const nuevoEstado = {
    ...estadoInicial,
    autenticado: true,
    token: "abc123",
    usuario: {
        ...estadoInicial.usuario,
        ultimoAcceso: new Date()
    }
};

console.log(nuevoEstado);
```

---

## Rest operator

**Recoger elementos restantes** (`...`).

### En arrays

```javascript
const numeros = [1, 2, 3, 4, 5];

const [primero, segundo, ...resto] = numeros;
console.log(primero);  // 1
console.log(segundo);  // 2
console.log(resto);    // [3, 4, 5]
```

### En objetos

```javascript
const usuario = {
    id: 1,
    nombre: "Ana",
    edad: 25,
    ciudad: "Madrid",
    email: "ana@ejemplo.com"
};

const { id, nombre, ...otrosDatos } = usuario;
console.log(id);          // 1
console.log(nombre);      // "Ana"
console.log(otrosDatos);  // { edad: 25, ciudad: "Madrid", email: "ana@ejemplo.com" }
```

### En parámetros de función

```javascript
// Número variable de argumentos
function sumar(...numeros) {
    return numeros.reduce((acc, num) => acc + num, 0);
}

console.log(sumar(1, 2));           // 3
console.log(sumar(1, 2, 3, 4, 5));  // 15
```

### Combinar con parámetros normales

```javascript
function presentar(saludo, ...nombres) {
    return `${saludo} ${nombres.join(", ")}!`;
}

console.log(presentar("Hola", "Ana", "Juan", "María"));
// "Hola Ana, Juan, María!"
```

### Ejemplo práctico: Logger flexible

```javascript
function log(nivel, ...mensajes) {
    const timestamp = new Date().toISOString();
    const texto = mensajes.join(" ");
    console.log(`[${timestamp}] ${nivel}: ${texto}`);
}

log("INFO", "Usuario", "Ana", "ha iniciado sesión");
log("ERROR", "Fallo en conexión");
```

---

## Casos prácticos

### Clonar y modificar

```javascript
const productoOriginal = {
    id: 1,
    nombre: "Laptop",
    precio: 1200,
    specs: {
        ram: "16GB",
        cpu: "i7"
    }
};

// Clonar y actualizar
const productoOferta = {
    ...productoOriginal,
    precio: productoOriginal.precio * 0.8,  // 20% descuento
    enOferta: true,
    specs: {
        ...productoOriginal.specs,
        ssd: "512GB"  // Agregar nueva spec
    }
};

console.log(productoOferta);
```

### Merge con prioridad

```javascript
const defaultConfig = {
    theme: "light",
    language: "es",
    notifications: true,
    timeout: 5000
};

const userConfig = {
    theme: "dark",
    timeout: 3000
};

// User config sobrescribe defaults
const finalConfig = { ...defaultConfig, ...userConfig };
console.log(finalConfig);
// {
//   theme: "dark",        (sobrescrito)
//   language: "es",       (default)
//   notifications: true,  (default)
//   timeout: 3000        (sobrescrito)
// }
```

### Construir objetos dinámicamente

```javascript
function crearProducto(nombre, precio, opciones = {}) {
    return {
        id: Math.random().toString(36).substr(2, 9),
        nombre,
        precio,
        ...opciones,
        createdAt: new Date()
    };
}

console.log(crearProducto("Laptop", 1200, {
    categoria: "Electrónica",
    stock: 5
}));
```

### Filtrar propiedades

```javascript
const usuario = {
    id: 1,
    nombre: "Ana",
    password: "secreto",
    token: "abc123",
    email: "ana@ejemplo.com",
    edad: 25
};

function omitirPropiedades(obj, ...keysToOmit) {
    const copia = { ...obj };
    keysToOmit.forEach(key => delete copia[key]);
    return copia;
}

const usuarioSeguro = omitirPropiedades(usuario, "password", "token");
console.log(usuarioSeguro);
// { id: 1, nombre: "Ana", email: "ana@ejemplo.com", edad: 25 }
```

### Combinar arrays únicos

```javascript
const equipo1 = ["Ana", "Juan", "Pedro"];
const equipo2 = ["María", "Ana", "Luis"];

const todosSinDuplicados = [...new Set([...equipo1, ...equipo2])];
console.log(todosSinDuplicados);
// ["Ana", "Juan", "Pedro", "María", "Luis"]
```

---

## Ejercicios prácticos

### Ejercicio 1: Intercambio de variables
**Nivel**: ⭐☆☆☆☆

Crea una función que intercambie valores de tres variables en orden rotativo.

```javascript
// Input: a=1, b=2, c=3
// Output: a=3, b=1, c=2
```

<details>
<summary>Ver solución</summary>

```javascript
function rotarVariables(a, b, c) {
    [a, b, c] = [c, a, b];
    return { a, b, c };
}

console.log(rotarVariables(1, 2, 3));  // { a: 3, b: 1, c: 2 }
console.log(rotarVariables(10, 20, 30));  // { a: 30, b: 10, c: 20 }
```

</details>

---

### Ejercicio 2: Extraer datos de usuario
**Nivel**: ⭐⭐☆☆☆

Extrae información específica de un objeto usuario complejo.

```javascript
const usuario = {
    id: 1,
    info: {
        nombre: "Ana García",
        edad: 25,
        direccion: {
            ciudad: "Madrid",
            pais: "España"
        }
    },
    preferencias: {
        idioma: "es",
        tema: "oscuro"
    }
};

// Extraer: nombre, ciudad, idioma
```

<details>
<summary>Ver solución</summary>

```javascript
const usuario = {
    id: 1,
    info: {
        nombre: "Ana García",
        edad: 25,
        direccion: {
            ciudad: "Madrid",
            pais: "España"
        }
    },
    preferencias: {
        idioma: "es",
        tema: "oscuro"
    }
};

const {
    info: {
        nombre,
        direccion: { ciudad }
    },
    preferencias: { idioma }
} = usuario;

console.log(`${nombre} de ${ciudad} habla ${idioma}`);
// "Ana García de Madrid habla es"
```

</details>

---

### Ejercicio 3: Combinar configuraciones
**Nivel**: ⭐⭐☆☆☆

Crea una función que combine configuración predeterminada con configuración de usuario.

```javascript
const defaultSettings = {
    theme: "light",
    language: "en",
    notifications: {
        email: true,
        push: false
    },
    privacy: {
        public: false,
        searchable: true
    }
};

const userSettings = {
    theme: "dark",
    notifications: {
        push: true
    }
};

// Resultado esperado: merge profundo
```

<details>
<summary>Ver solución</summary>

```javascript
const defaultSettings = {
    theme: "light",
    language: "en",
    notifications: {
        email: true,
        push: false
    },
    privacy: {
        public: false,
        searchable: true
    }
};

const userSettings = {
    theme: "dark",
    notifications: {
        push: true
    }
};

function mergeSettings(defaults, user) {
    const result = { ...defaults };
    
    for (const key in user) {
        if (typeof user[key] === 'object' && !Array.isArray(user[key])) {
            result[key] = { ...defaults[key], ...user[key] };
        } else {
            result[key] = user[key];
        }
    }
    
    return result;
}

const finalSettings = mergeSettings(defaultSettings, userSettings);
console.log(finalSettings);
// {
//   theme: "dark",         (sobrescrito)
//   language: "en",        (default)
//   notifications: {
//     email: true,         (default)
//     push: true          (sobrescrito)
//   },
//   privacy: {             (default completo)
//     public: false,
//     searchable: true
//   }
// }
```

</details>

---

### Ejercicio 4: Función suma flexible
**Nivel**: ⭐⭐⭐☆☆

Crea una función que sume cualquier cantidad de números y arrays de números.

```javascript
// Debe funcionar con:
suma(1, 2, 3);              // 6
suma([1, 2], 3, [4, 5]);    // 15
suma(1, [2, 3], [[4], 5]);  // 15
```

<details>
<summary>Ver solución</summary>

```javascript
function suma(...args) {
    return args
        .flat(Infinity)
        .reduce((acc, num) => acc + num, 0);
}

console.log(suma(1, 2, 3));              // 6
console.log(suma([1, 2], 3, [4, 5]));    // 15
console.log(suma(1, [2, 3], [[4], 5]));  // 15
console.log(suma(10));                   // 10
console.log(suma());                     // 0
```

</details>

---

### Ejercicio 5: Sistema de usuarios
**Nivel**: ⭐⭐⭐⭐☆

Crea funciones para gestionar usuarios usando destructuring y spread.

```javascript
// Funciones a implementar:
// 1. crearUsuario(datos) - crear con defaults
// 2. actualizarUsuario(usuario, cambios)
// 3. ocultarDatosSensibles(usuario)
// 4. combinarUsuarios(...usuarios)
```

<details>
<summary>Ver solución</summary>

```javascript
// 1. Crear usuario con defaults
function crearUsuario({ nombre, email, edad = 18, rol = "usuario" }) {
    return {
        id: Math.random().toString(36).substr(2, 9),
        nombre,
        email,
        edad,
        rol,
        activo: true,
        createdAt: new Date().toISOString()
    };
}

// 2. Actualizar usuario
function actualizarUsuario(usuario, cambios) {
    return {
        ...usuario,
        ...cambios,
        updatedAt: new Date().toISOString()
    };
}

// 3. Ocultar datos sensibles
function ocultarDatosSensibles(usuario) {
    const { password, token, email, ...datosPublicos } = usuario;
    return {
        ...datosPublicos,
        emailOculto: email ? email.replace(/(.{2})(.*)(@.*)/, '$1***$3') : null
    };
}

// 4. Combinar usuarios (merge de arrays)
function combinarUsuarios(...gruposUsuarios) {
    const todosLosUsuarios = gruposUsuarios.flat();
    
    // Eliminar duplicados por ID
    const unicos = todosLosUsuarios.reduce((acc, usuario) => {
        if (!acc.find(u => u.id === usuario.id)) {
            acc.push(usuario);
        }
        return acc;
    }, []);
    
    return unicos;
}

// Pruebas
const usuario1 = crearUsuario({
    nombre: "Ana",
    email: "ana@ejemplo.com",
    edad: 25
});
console.log("Usuario creado:", usuario1);

const usuario1Actualizado = actualizarUsuario(usuario1, {
    edad: 26,
    ciudad: "Madrid"
});
console.log("Usuario actualizado:", usuario1Actualizado);

const usuario1Seguro = ocultarDatosSensibles({
    ...usuario1,
    password: "secreto123",
    token: "abc123"
});
console.log("Usuario seguro:", usuario1Seguro);

const grupo1 = [usuario1, { id: 2, nombre: "Juan" }];
const grupo2 = [{ id: 3, nombre: "María" }, usuario1];  // usuario1 duplicado
const combinados = combinarUsuarios(grupo1, grupo2);
console.log("Usuarios combinados:", combinados);  // 3 usuarios (sin duplicados)
```

</details>

---

### Ejercicio 6: Procesador de pedidos
**Nivel**: ⭐⭐⭐⭐⭐

Sistema completo para procesar pedidos de e-commerce.

```javascript
const pedido = {
    id: "ORD-001",
    cliente: {
        nombre: "Ana García",
        email: "ana@ejemplo.com",
        direccion: {
            calle: "Gran Vía 123",
            ciudad: "Madrid",
            cp: "28013"
        }
    },
    items: [
        { id: 1, nombre: "Laptop", precio: 1200, cantidad: 1 },
        { id: 2, nombre: "Mouse", precio: 25, cantidad: 2 }
    ],
    descuento: 10
};

// Implementar:
// 1. calcularTotal(pedido)
// 2. aplicarDescuentoAdicional(pedido, porcentaje)
// 3. agregarItem(pedido, item)
// 4. generarFactura(pedido)
```

<details>
<summary>Ver solución</summary>

```javascript
const pedido = {
    id: "ORD-001",
    cliente: {
        nombre: "Ana García",
        email: "ana@ejemplo.com",
        direccion: {
            calle: "Gran Vía 123",
            ciudad: "Madrid",
            cp: "28013"
        }
    },
    items: [
        { id: 1, nombre: "Laptop", precio: 1200, cantidad: 1 },
        { id: 2, nombre: "Mouse", precio: 25, cantidad: 2 }
    ],
    descuento: 10
};

// 1. Calcular total
function calcularTotal(pedido) {
    const { items, descuento = 0 } = pedido;
    
    const subtotal = items.reduce((acc, item) => {
        return acc + (item.precio * item.cantidad);
    }, 0);
    
    const descuentoMonto = subtotal * (descuento / 100);
    const total = subtotal - descuentoMonto;
    
    return {
        subtotal,
        descuento: descuentoMonto,
        total
    };
}

// 2. Aplicar descuento adicional
function aplicarDescuentoAdicional(pedido, porcentajeAdicional) {
    const { descuento = 0 } = pedido;
    
    return {
        ...pedido,
        descuento: descuento + porcentajeAdicional,
        descuentoAplicado: new Date().toISOString()
    };
}

// 3. Agregar item
function agregarItem(pedido, nuevoItem) {
    const { items } = pedido;
    
    // Verificar si ya existe
    const existeIndex = items.findIndex(item => item.id === nuevoItem.id);
    
    let nuevosItems;
    if (existeIndex !== -1) {
        // Incrementar cantidad
        nuevosItems = items.map((item, index) => {
            if (index === existeIndex) {
                return {
                    ...item,
                    cantidad: item.cantidad + nuevoItem.cantidad
                };
            }
            return item;
        });
    } else {
        // Agregar nuevo
        nuevosItems = [...items, nuevoItem];
    }
    
    return {
        ...pedido,
        items: nuevosItems
    };
}

// 4. Generar factura
function generarFactura(pedido) {
    const {
        id,
        cliente: {
            nombre,
            email,
            direccion: { ciudad, cp }
        },
        items
    } = pedido;
    
    const { subtotal, descuento, total } = calcularTotal(pedido);
    
    return {
        numeroFactura: `FAC-${id}`,
        fecha: new Date().toISOString(),
        cliente: {
            nombre,
            email,
            ubicacion: `${ciudad}, ${cp}`
        },
        items: items.map(({ nombre, precio, cantidad }) => ({
            producto: nombre,
            precioUnitario: precio,
            cantidad,
            subtotal: precio * cantidad
        })),
        resumen: {
            subtotal: subtotal.toFixed(2),
            descuento: descuento.toFixed(2),
            total: total.toFixed(2)
        }
    };
}

// Pruebas
console.log("Total inicial:", calcularTotal(pedido));

const pedidoConDescuento = aplicarDescuentoAdicional(pedido, 5);
console.log("Con descuento adicional:", calcularTotal(pedidoConDescuento));

const pedidoConNuevoItem = agregarItem(pedido, {
    id: 3,
    nombre: "Teclado",
    precio: 80,
    cantidad: 1
});
console.log("Con nuevo item:", calcularTotal(pedidoConNuevoItem));

const factura = generarFactura(pedido);
console.log("Factura:", factura);
```

</details>

---

## Errores comunes

### ❌ Error 1: Confundir spread con rest

```javascript
// ❌ Error: spread en parámetros
function sumar(...numeros) {  // ✅ Rest (correcto)
    return [...numeros];      // ❌ Spread innecesario
}

// ✅ Correcto
function sumar(...numeros) {
    return numeros.reduce((a, b) => a + b, 0);
}
```

---

### ❌ Error 2: Copia superficial vs profunda

```javascript
const original = {
    nombre: "Ana",
    direccion: {
        ciudad: "Madrid"
    }
};

const copia = { ...original };
copia.direccion.ciudad = "Barcelona";

console.log(original.direccion.ciudad);  // "Barcelona" ❌ (mutado)

// ✅ Copia profunda manual
const copiaCorrecta = {
    ...original,
    direccion: { ...original.direccion }
};
```

---

### ❌ Error 3: Rest debe ser el último

```javascript
// ❌ Error: rest no es el último
function fn(...rest, ultimo) {  // SyntaxError
}

// ✅ Correcto
function fn(primero, ...rest) {
    console.log(primero, rest);
}
```

---

### ❌ Error 4: Destructuring de undefined

```javascript
const usuario = null;

// ❌ Error: Cannot destructure
const { nombre } = usuario;  // TypeError

// ✅ Con valor por defecto
const { nombre } = usuario || {};
console.log(nombre);  // undefined (sin error)

// ✅ O con optional chaining
const nombre = usuario?.nombre;
```

---

## Buenas prácticas

### ✅ 1. Usa destructuring en parámetros

```javascript
// ❌ No claro
function crearUsuario(datos) {
    return {
        nombre: datos.nombre,
        edad: datos.edad || 18
    };
}

// ✅ Claro y conciso
function crearUsuario({ nombre, edad = 18 }) {
    return { nombre, edad };
}
```

---

### ✅ 2. Nombra variables al extraer

```javascript
// ❌ Confuso
const { x, y, z } = coordenadas;

// ✅ Descriptivo
const { latitud: lat, longitud: lon, altitud: alt } = coordenadas;
```

---

### ✅ 3. Usa spread para inmutabilidad

```javascript
// ❌ Muta el original
function agregarPropiedad(obj, key, value) {
    obj[key] = value;
    return obj;
}

// ✅ Retorna nuevo objeto
function agregarPropiedad(obj, key, value) {
    return { ...obj, [key]: value };
}
```

---

### ✅ 4. Combina con valores por defecto

```javascript
// ✅ Robusto con defaults
function configurar({ host = "localhost", port = 3000, ssl = false } = {}) {
    return { host, port, ssl };
}

configurar();  // Funciona sin argumentos
configurar({ port: 8080 });  // Parcial
```

---

## Cheatsheet

### Destructuring Arrays

```javascript
const [a, b] = [1, 2];
const [x, , z] = [1, 2, 3];  // Saltar
const [first, ...rest] = [1, 2, 3];
```

### Destructuring Objetos

```javascript
const { nombre, edad } = usuario;
const { nombre: n } = usuario;  // Renombrar
const { edad = 18 } = usuario;  // Default
```

### Spread Arrays

```javascript
[...arr1, ...arr2]        // Concatenar
[...array]                // Copiar
[1, ...arr, 5]           // Intercalar
```

### Spread Objetos

```javascript
{ ...obj1, ...obj2 }      // Combinar
{ ...obj }                // Copiar
{ ...obj, x: 10 }        // Agregar/actualizar
```

### Rest

```javascript
function fn(...args) {}   // Parámetros
const [a, ...rest] = arr;
const { x, ...otros } = obj;
```

---

## Siguiente paso

Has dominado destructuring y spread operators. Ahora aprenderás **programación asíncrona**.

→ [30-asincronia.md](30-asincronia.md)

Ahí descubrirás callbacks, Promises, async/await y cómo manejar operaciones asíncronas en JavaScript.

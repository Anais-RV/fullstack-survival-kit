# Módulo 30: Asincronía en JavaScript

## Índice
1. [Introducción a la asincronía](#introducción-a-la-asincronía)
2. [Callbacks](#callbacks)
3. [Promises](#promises)
4. [async/await](#asyncawait)
5. [Manejo de errores](#manejo-de-errores)
6. [Promise.all y Promise.race](#promiseall-y-promiserace)
7. [Fetch API](#fetch-api)
8. [Patrones comunes](#patrones-comunes)
9. [Ejercicios prácticos](#ejercicios-prácticos)
10. [Errores comunes](#errores-comunes)
11. [Buenas prácticas](#buenas-prácticas)
12. [Cheatsheet](#cheatsheet)

---

## Introducción a la asincronía

**Asincronía** permite ejecutar código sin bloquear el hilo principal de JavaScript.

### Analogía: Restaurante

```
Código Síncrono → El camarero atiende una mesa, espera que terminen, luego va a la siguiente
Código Asíncrono → El camarero toma pedidos de varias mesas y atiende cuando están listos
```

### ¿Por qué asincronía?

✅ **No bloquear la interfaz** mientras se cargan datos  
✅ **Mejor experiencia de usuario**  
✅ **Operaciones en paralelo**  
✅ **Manejar operaciones lentas** (red, archivos, BD)

### Ejemplo del problema

```javascript
// ❌ Código síncrono (bloqueante)
console.log("1. Inicio");

// Esta operación tarda 5 segundos
const datos = esperarDatos(); // BLOQUEA todo durante 5 segundos

console.log("2. Fin");

// ✅ Código asíncrono (no bloqueante)
console.log("1. Inicio");

// Esta operación se ejecuta en segundo plano
cargarDatos().then(datos => {
    console.log("3. Datos recibidos:", datos);
});

console.log("2. Fin (sin esperar los datos)");

// Output:
// 1. Inicio
// 2. Fin (sin esperar los datos)
// 3. Datos recibidos: [datos]
```

---

## Callbacks

**Una función que se pasa como argumento para ejecutarse después**.

### Callback simple

```javascript
function hacerAlgo(callback) {
    console.log("Haciendo algo...");
    callback();
}

hacerAlgo(() => {
    console.log("¡Terminado!");
});

// Output:
// Haciendo algo...
// ¡Terminado!
```

### setTimeout - Callback después de un tiempo

```javascript
console.log("1. Inicio");

setTimeout(() => {
    console.log("2. Después de 2 segundos");
}, 2000);

console.log("3. Fin (sin esperar)");

// Output:
// 1. Inicio
// 3. Fin (sin esperar)
// ... (2 segundos después)
// 2. Después de 2 segundos
```

### Callback con datos

```javascript
function obtenerUsuario(id, callback) {
    console.log(`Buscando usuario ${id}...`);
    
    setTimeout(() => {
        const usuario = { id, nombre: "Ana", edad: 25 };
        callback(usuario);
    }, 1000);
}

obtenerUsuario(1, (usuario) => {
    console.log("Usuario encontrado:", usuario);
});
```

### Callback Hell (problema)

```javascript
// ❌ Pirámide de la perdición
obtenerUsuario(1, (usuario) => {
    obtenerPedidos(usuario.id, (pedidos) => {
        obtenerDetalles(pedidos[0].id, (detalles) => {
            calcularTotal(detalles, (total) => {
                console.log("Total:", total);
            });
        });
    });
});
```

---

## Promises

**Representan un valor que estará disponible ahora, en el futuro, o nunca**.

### Estados de una Promise

```
pending   → Pendiente (inicial)
fulfilled → Cumplida (éxito)
rejected  → Rechazada (error)
```

### Crear una Promise

```javascript
const promesa = new Promise((resolve, reject) => {
    // Simular operación asíncrona
    setTimeout(() => {
        const exito = true;
        
        if (exito) {
            resolve("¡Éxito!"); // Cumplir la promesa
        } else {
            reject("Error"); // Rechazar la promesa
        }
    }, 1000);
});
```

### Consumir una Promise - then/catch

```javascript
promesa
    .then(resultado => {
        console.log("Éxito:", resultado);
    })
    .catch(error => {
        console.error("Error:", error);
    });
```

### Ejemplo práctico: Obtener usuario

```javascript
function obtenerUsuario(id) {
    return new Promise((resolve, reject) => {
        console.log(`Buscando usuario ${id}...`);
        
        setTimeout(() => {
            if (id > 0) {
                resolve({ id, nombre: "Ana", edad: 25 });
            } else {
                reject("ID inválido");
            }
        }, 1000);
    });
}

obtenerUsuario(1)
    .then(usuario => {
        console.log("Usuario:", usuario);
    })
    .catch(error => {
        console.error("Error:", error);
    });
```

### Encadenar Promises

```javascript
function obtenerUsuario(id) {
    return new Promise(resolve => {
        setTimeout(() => {
            resolve({ id, nombre: "Ana" });
        }, 1000);
    });
}

function obtenerPedidos(usuarioId) {
    return new Promise(resolve => {
        setTimeout(() => {
            resolve([
                { id: 1, total: 100 },
                { id: 2, total: 200 }
            ]);
        }, 1000);
    });
}

function calcularTotal(pedidos) {
    return new Promise(resolve => {
        setTimeout(() => {
            const total = pedidos.reduce((acc, p) => acc + p.total, 0);
            resolve(total);
        }, 1000);
    });
}

// ✅ Encadenar promises (mejor que callbacks)
obtenerUsuario(1)
    .then(usuario => {
        console.log("Usuario:", usuario.nombre);
        return obtenerPedidos(usuario.id);
    })
    .then(pedidos => {
        console.log(`${pedidos.length} pedidos encontrados`);
        return calcularTotal(pedidos);
    })
    .then(total => {
        console.log("Total:", total);
    })
    .catch(error => {
        console.error("Error en algún paso:", error);
    });
```

### finally

```javascript
obtenerUsuario(1)
    .then(usuario => console.log(usuario))
    .catch(error => console.error(error))
    .finally(() => {
        console.log("Operación completada (éxito o error)");
    });
```

---

## async/await

**Sintaxis moderna para trabajar con Promises de forma más legible**.

### Función async

```javascript
// Función async siempre retorna una Promise
async function saludar() {
    return "Hola";
}

saludar().then(mensaje => console.log(mensaje));
// "Hola"
```

### await - Esperar una Promise

```javascript
function esperar(ms) {
    return new Promise(resolve => setTimeout(resolve, ms));
}

async function ejemplo() {
    console.log("1. Inicio");
    
    await esperar(2000); // Espera 2 segundos
    
    console.log("2. Después de 2 segundos");
}

ejemplo();
```

### Ejemplo con obtener datos

```javascript
function obtenerUsuario(id) {
    return new Promise(resolve => {
        setTimeout(() => {
            resolve({ id, nombre: "Ana" });
        }, 1000);
    });
}

// ❌ Con then (verboso)
function procesoConThen() {
    obtenerUsuario(1)
        .then(usuario => {
            console.log(usuario);
        });
}

// ✅ Con async/await (más limpio)
async function procesoConAwait() {
    const usuario = await obtenerUsuario(1);
    console.log(usuario);
}

procesoConAwait();
```

### Múltiples awaits

```javascript
async function procesarTodo() {
    console.log("Inicio");
    
    const usuario = await obtenerUsuario(1);
    console.log("Usuario:", usuario.nombre);
    
    const pedidos = await obtenerPedidos(usuario.id);
    console.log(`${pedidos.length} pedidos`);
    
    const total = await calcularTotal(pedidos);
    console.log("Total:", total);
    
    return total;
}

procesarTodo().then(total => {
    console.log("Proceso completado con total:", total);
});
```

### Comparación: then vs async/await

```javascript
// ❌ Con then (anidado)
function conThen() {
    return obtenerUsuario(1)
        .then(usuario => {
            return obtenerPedidos(usuario.id)
                .then(pedidos => {
                    return calcularTotal(pedidos);
                });
        });
}

// ✅ Con async/await (secuencial y claro)
async function conAwait() {
    const usuario = await obtenerUsuario(1);
    const pedidos = await obtenerPedidos(usuario.id);
    const total = await calcularTotal(pedidos);
    return total;
}
```

---

## Manejo de errores

**Capturar y manejar errores en código asíncrono**.

### try/catch con async/await

```javascript
async function obtenerDatos() {
    try {
        const usuario = await obtenerUsuario(1);
        console.log("Usuario:", usuario);
        
        const pedidos = await obtenerPedidos(usuario.id);
        console.log("Pedidos:", pedidos);
        
        return pedidos;
    } catch (error) {
        console.error("Error:", error);
        return null;
    }
}
```

### Múltiples catches

```javascript
async function procesarConErrores() {
    try {
        const usuario = await obtenerUsuario(-1); // Error
        return usuario;
    } catch (error) {
        if (error.message.includes("ID")) {
            console.error("Error de ID:", error);
            return null;
        }
        throw error; // Re-lanzar si es otro tipo de error
    }
}
```

### finally con async/await

```javascript
async function cargarDatos() {
    let cargando = true;
    
    try {
        console.log("Cargando...");
        const datos = await obtenerDatos();
        console.log("Datos:", datos);
        return datos;
    } catch (error) {
        console.error("Error:", error);
        return null;
    } finally {
        cargando = false;
        console.log("Carga finalizada");
    }
}
```

### Validación y manejo robusto

```javascript
function obtenerUsuario(id) {
    return new Promise((resolve, reject) => {
        setTimeout(() => {
            if (!id) {
                reject(new Error("ID es requerido"));
            } else if (id < 0) {
                reject(new Error("ID debe ser positivo"));
            } else if (id > 1000) {
                reject(new Error("Usuario no encontrado"));
            } else {
                resolve({ id, nombre: "Ana" });
            }
        }, 1000);
    });
}

async function buscarUsuario(id) {
    try {
        const usuario = await obtenerUsuario(id);
        return { exito: true, usuario };
    } catch (error) {
        return { exito: false, error: error.message };
    }
}

// Uso
const resultado = await buscarUsuario(-5);
if (resultado.exito) {
    console.log("Usuario:", resultado.usuario);
} else {
    console.error("Error:", resultado.error);
}
```

---

## Promise.all y Promise.race

**Trabajar con múltiples Promises simultáneamente**.

### Promise.all - Esperar todas

```javascript
const promesa1 = Promise.resolve(1);
const promesa2 = Promise.resolve(2);
const promesa3 = Promise.resolve(3);

Promise.all([promesa1, promesa2, promesa3])
    .then(resultados => {
        console.log(resultados); // [1, 2, 3]
    });
```

### Ejemplo práctico: Cargar múltiples recursos

```javascript
function obtenerUsuario(id) {
    return new Promise(resolve => {
        setTimeout(() => resolve({ id, nombre: "Ana" }), 1000);
    });
}

function obtenerConfig() {
    return new Promise(resolve => {
        setTimeout(() => resolve({ theme: "dark" }), 800);
    });
}

function obtenerPermisos() {
    return new Promise(resolve => {
        setTimeout(() => resolve(["read", "write"]), 1200);
    });
}

async function cargarTodo() {
    console.log("Cargando...");
    
    // ❌ Secuencial (3.0 segundos total)
    // const usuario = await obtenerUsuario(1);
    // const config = await obtenerConfig();
    // const permisos = await obtenerPermisos();
    
    // ✅ Paralelo con Promise.all (1.2 segundos, el más lento)
    const [usuario, config, permisos] = await Promise.all([
        obtenerUsuario(1),
        obtenerConfig(),
        obtenerPermisos()
    ]);
    
    console.log("Todo cargado:", { usuario, config, permisos });
}

cargarTodo();
```

### Promise.all con errores

```javascript
const promesa1 = Promise.resolve("éxito");
const promesa2 = Promise.reject("error");
const promesa3 = Promise.resolve("éxito");

Promise.all([promesa1, promesa2, promesa3])
    .then(resultados => {
        console.log(resultados); // No se ejecuta
    })
    .catch(error => {
        console.error("Falló una:", error); // "error"
    });
```

### Promise.allSettled - Esperar todas (incluso con errores)

```javascript
const promesas = [
    Promise.resolve("éxito 1"),
    Promise.reject("error"),
    Promise.resolve("éxito 2")
];

Promise.allSettled(promesas)
    .then(resultados => {
        resultados.forEach((resultado, index) => {
            if (resultado.status === "fulfilled") {
                console.log(`${index}: Éxito -`, resultado.value);
            } else {
                console.log(`${index}: Error -`, resultado.reason);
            }
        });
    });

// Output:
// 0: Éxito - éxito 1
// 1: Error - error
// 2: Éxito - éxito 2
```

### Promise.race - La primera que termina

```javascript
const lenta = new Promise(resolve => {
    setTimeout(() => resolve("Lenta"), 3000);
});

const rapida = new Promise(resolve => {
    setTimeout(() => resolve("Rápida"), 1000);
});

Promise.race([lenta, rapida])
    .then(ganadora => {
        console.log("Primera:", ganadora); // "Rápida"
    });
```

### Timeout con Promise.race

```javascript
function timeout(ms) {
    return new Promise((_, reject) => {
        setTimeout(() => reject(new Error("Timeout")), ms);
    });
}

function obtenerDatos() {
    return new Promise(resolve => {
        setTimeout(() => resolve("Datos"), 5000);
    });
}

async function conTimeout() {
    try {
        const datos = await Promise.race([
            obtenerDatos(),
            timeout(2000) // Máximo 2 segundos
        ]);
        console.log(datos);
    } catch (error) {
        console.error("Tardó demasiado:", error.message);
    }
}

conTimeout(); // "Tardó demasiado: Timeout"
```

---

## Fetch API

**API moderna para hacer peticiones HTTP**.

### GET básico

```javascript
fetch("https://api.ejemplo.com/usuarios")
    .then(response => response.json())
    .then(datos => {
        console.log(datos);
    })
    .catch(error => {
        console.error("Error:", error);
    });
```

### Con async/await

```javascript
async function obtenerUsuarios() {
    try {
        const response = await fetch("https://api.ejemplo.com/usuarios");
        
        if (!response.ok) {
            throw new Error(`HTTP error! status: ${response.status}`);
        }
        
        const datos = await response.json();
        return datos;
    } catch (error) {
        console.error("Error:", error);
        return [];
    }
}
```

### POST - Enviar datos

```javascript
async function crearUsuario(usuario) {
    try {
        const response = await fetch("https://api.ejemplo.com/usuarios", {
            method: "POST",
            headers: {
                "Content-Type": "application/json"
            },
            body: JSON.stringify(usuario)
        });
        
        const datos = await response.json();
        return datos;
    } catch (error) {
        console.error("Error:", error);
    }
}

// Uso
crearUsuario({ nombre: "Ana", edad: 25 });
```

### PUT - Actualizar

```javascript
async function actualizarUsuario(id, cambios) {
    const response = await fetch(`https://api.ejemplo.com/usuarios/${id}`, {
        method: "PUT",
        headers: {
            "Content-Type": "application/json"
        },
        body: JSON.stringify(cambios)
    });
    
    return await response.json();
}
```

### DELETE - Eliminar

```javascript
async function eliminarUsuario(id) {
    const response = await fetch(`https://api.ejemplo.com/usuarios/${id}`, {
        method: "DELETE"
    });
    
    if (response.ok) {
        return { mensaje: "Usuario eliminado" };
    }
    
    throw new Error("No se pudo eliminar");
}
```

### Ejemplo completo: API de usuarios

```javascript
const API_URL = "https://jsonplaceholder.typicode.com";

class UsuariosAPI {
    async obtener(id) {
        const response = await fetch(`${API_URL}/users/${id}`);
        if (!response.ok) throw new Error("Usuario no encontrado");
        return await response.json();
    }
    
    async listar() {
        const response = await fetch(`${API_URL}/users`);
        return await response.json();
    }
    
    async crear(usuario) {
        const response = await fetch(`${API_URL}/users`, {
            method: "POST",
            headers: { "Content-Type": "application/json" },
            body: JSON.stringify(usuario)
        });
        return await response.json();
    }
    
    async actualizar(id, cambios) {
        const response = await fetch(`${API_URL}/users/${id}`, {
            method: "PATCH",
            headers: { "Content-Type": "application/json" },
            body: JSON.stringify(cambios)
        });
        return await response.json();
    }
    
    async eliminar(id) {
        const response = await fetch(`${API_URL}/users/${id}`, {
            method: "DELETE"
        });
        return response.ok;
    }
}

// Uso
const api = new UsuariosAPI();

async function ejemplos() {
    // Obtener usuario
    const usuario = await api.obtener(1);
    console.log("Usuario:", usuario);
    
    // Listar todos
    const usuarios = await api.listar();
    console.log("Total:", usuarios.length);
    
    // Crear
    const nuevo = await api.crear({ nombre: "Ana", email: "ana@ejemplo.com" });
    console.log("Creado:", nuevo);
}
```

---

## Patrones comunes

### Retry automático

```javascript
async function fetchConReintentos(url, intentosMax = 3) {
    for (let i = 0; i < intentosMax; i++) {
        try {
            const response = await fetch(url);
            if (response.ok) {
                return await response.json();
            }
        } catch (error) {
            if (i === intentosMax - 1) {
                throw error;
            }
            console.log(`Intento ${i + 1} falló, reintentando...`);
            await new Promise(resolve => setTimeout(resolve, 1000));
        }
    }
}
```

### Cache de promesas

```javascript
class CachePromesas {
    constructor() {
        this.cache = new Map();
    }
    
    async obtener(key, fn) {
        if (this.cache.has(key)) {
            console.log("Desde caché:", key);
            return this.cache.get(key);
        }
        
        console.log("Cargando:", key);
        const valor = await fn();
        this.cache.set(key, valor);
        return valor;
    }
    
    limpiar() {
        this.cache.clear();
    }
}

// Uso
const cache = new CachePromesas();

async function obtenerUsuario(id) {
    return await cache.obtener(`usuario-${id}`, async () => {
        const response = await fetch(`/api/usuarios/${id}`);
        return await response.json();
    });
}
```

### Debounce asíncrono

```javascript
function debounceAsync(fn, delay) {
    let timeoutId;
    
    return function(...args) {
        return new Promise((resolve, reject) => {
            clearTimeout(timeoutId);
            
            timeoutId = setTimeout(async () => {
                try {
                    const result = await fn(...args);
                    resolve(result);
                } catch (error) {
                    reject(error);
                }
            }, delay);
        });
    };
}

// Uso
const buscarDebounced = debounceAsync(async (termino) => {
    const response = await fetch(`/api/buscar?q=${termino}`);
    return await response.json();
}, 500);

// Solo ejecuta después de 500ms sin cambios
buscarDebounced("javascript");
```

### Procesar en lotes

```javascript
async function procesarEnLotes(items, batchSize, procesarFn) {
    const resultados = [];
    
    for (let i = 0; i < items.length; i += batchSize) {
        const lote = items.slice(i, i + batchSize);
        console.log(`Procesando lote ${Math.floor(i / batchSize) + 1}`);
        
        const resultadosLote = await Promise.all(
            lote.map(item => procesarFn(item))
        );
        
        resultados.push(...resultadosLote);
    }
    
    return resultados;
}

// Uso
const usuarios = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10];

const datos = await procesarEnLotes(usuarios, 3, async (id) => {
    const response = await fetch(`/api/usuarios/${id}`);
    return await response.json();
});
```

---

## Ejercicios prácticos

### Ejercicio 1: Simulador de carga
**Nivel**: ⭐⭐☆☆☆

Crea una función que simule cargar datos con diferentes tiempos.

```javascript
// Debe retornar una Promise que se resuelve después de 'tiempo' ms
function simularCarga(mensaje, tiempo) {
    // Tu código aquí
}

// Uso esperado:
simularCarga("Datos", 2000).then(msg => console.log(msg));
// (después de 2 segundos) "Datos"
```

<details>
<summary>Ver solución</summary>

```javascript
function simularCarga(mensaje, tiempo) {
    return new Promise(resolve => {
        setTimeout(() => {
            resolve(mensaje);
        }, tiempo);
    });
}

// Pruebas
async function probar() {
    console.log("Inicio");
    
    const resultado1 = await simularCarga("Primer dato", 1000);
    console.log(resultado1);
    
    const resultado2 = await simularCarga("Segundo dato", 500);
    console.log(resultado2);
    
    console.log("Fin");
}

probar();
```

</details>

---

### Ejercicio 2: Cargar usuarios en paralelo
**Nivel**: ⭐⭐☆☆☆

Carga múltiples usuarios simultáneamente usando Promise.all.

```javascript
function obtenerUsuario(id) {
    return new Promise(resolve => {
        setTimeout(() => {
            resolve({ id, nombre: `Usuario ${id}` });
        }, Math.random() * 1000);
    });
}

// Cargar usuarios 1, 2, 3, 4, 5 en paralelo
```

<details>
<summary>Ver solución</summary>

```javascript
function obtenerUsuario(id) {
    return new Promise(resolve => {
        setTimeout(() => {
            resolve({ id, nombre: `Usuario ${id}` });
        }, Math.random() * 1000);
    });
}

async function cargarMultiplesUsuarios(ids) {
    console.log("Cargando usuarios...");
    const inicio = Date.now();
    
    const usuarios = await Promise.all(
        ids.map(id => obtenerUsuario(id))
    );
    
    const tiempo = Date.now() - inicio;
    console.log(`Cargados ${usuarios.length} usuarios en ${tiempo}ms`);
    
    return usuarios;
}

// Prueba
cargarMultiplesUsuarios([1, 2, 3, 4, 5])
    .then(usuarios => {
        usuarios.forEach(u => console.log(u.nombre));
    });
```

</details>

---

### Ejercicio 3: Manejo de errores robusto
**Nivel**: ⭐⭐⭐☆☆

Crea una función que maneje errores de API correctamente.

```javascript
async function obtenerDatosConManejo(url) {
    // Debe:
    // - Hacer fetch a la URL
    // - Verificar si la respuesta es ok
    // - Retornar { exito: true, datos } o { exito: false, error }
}
```

<details>
<summary>Ver solución</summary>

```javascript
async function obtenerDatosConManejo(url) {
    try {
        const response = await fetch(url);
        
        if (!response.ok) {
            return {
                exito: false,
                error: `HTTP ${response.status}: ${response.statusText}`
            };
        }
        
        const datos = await response.json();
        
        return {
            exito: true,
            datos
        };
    } catch (error) {
        return {
            exito: false,
            error: error.message
        };
    }
}

// Prueba
async function probar() {
    // URL válida
    const resultado1 = await obtenerDatosConManejo(
        "https://jsonplaceholder.typicode.com/users/1"
    );
    
    if (resultado1.exito) {
        console.log("Usuario:", resultado1.datos.name);
    } else {
        console.error("Error:", resultado1.error);
    }
    
    // URL inválida
    const resultado2 = await obtenerDatosConManejo(
        "https://jsonplaceholder.typicode.com/users/9999"
    );
    
    if (resultado2.exito) {
        console.log("Datos:", resultado2.datos);
    } else {
        console.error("Error:", resultado2.error);
    }
}

probar();
```

</details>

---

### Ejercicio 4: Sistema de reintentos
**Nivel**: ⭐⭐⭐⭐☆

Implementa una función que reintente operaciones fallidas.

```javascript
async function conReintentos(fn, intentosMax, delay) {
    // Debe:
    // - Intentar ejecutar fn()
    // - Si falla, esperar 'delay' ms y reintentar
    // - Máximo 'intentosMax' intentos
    // - Lanzar error si todos los intentos fallan
}
```

<details>
<summary>Ver solución</summary>

```javascript
async function conReintentos(fn, intentosMax = 3, delay = 1000) {
    let ultimoError;
    
    for (let intento = 1; intento <= intentosMax; intento++) {
        try {
            console.log(`Intento ${intento}/${intentosMax}`);
            const resultado = await fn();
            console.log(`✅ Éxito en intento ${intento}`);
            return resultado;
        } catch (error) {
            ultimoError = error;
            console.log(`❌ Intento ${intento} falló:`, error.message);
            
            if (intento < intentosMax) {
                console.log(`Esperando ${delay}ms antes de reintentar...`);
                await new Promise(resolve => setTimeout(resolve, delay));
            }
        }
    }
    
    throw new Error(`Falló después de ${intentosMax} intentos: ${ultimoError.message}`);
}

// Función que falla las primeras veces
let intentoActual = 0;
async function operacionInestable() {
    intentoActual++;
    
    if (intentoActual < 3) {
        throw new Error("Fallo temporal");
    }
    
    return "¡Éxito!";
}

// Prueba
conReintentos(operacionInestable, 5, 500)
    .then(resultado => console.log("Resultado:", resultado))
    .catch(error => console.error("Error final:", error.message));
```

</details>

---

### Ejercicio 5: API de tareas (CRUD completo)
**Nivel**: ⭐⭐⭐⭐⭐

Implementa un cliente completo para una API de tareas.

```javascript
class TareasAPI {
    constructor(baseURL) {
        this.baseURL = baseURL;
    }
    
    // Implementar:
    // - listar()
    // - obtener(id)
    // - crear(tarea)
    // - actualizar(id, cambios)
    // - eliminar(id)
    // - completar(id)
    // - obtenerCompletadas()
}
```

<details>
<summary>Ver solución</summary>

```javascript
class TareasAPI {
    constructor(baseURL) {
        this.baseURL = baseURL;
    }
    
    async _request(endpoint, options = {}) {
        const url = `${this.baseURL}${endpoint}`;
        
        try {
            const response = await fetch(url, {
                headers: {
                    "Content-Type": "application/json",
                    ...options.headers
                },
                ...options
            });
            
            if (!response.ok) {
                throw new Error(`HTTP ${response.status}: ${response.statusText}`);
            }
            
            // DELETE puede no retornar contenido
            if (response.status === 204) {
                return null;
            }
            
            return await response.json();
        } catch (error) {
            console.error(`Error en ${endpoint}:`, error);
            throw error;
        }
    }
    
    async listar() {
        return await this._request("/todos");
    }
    
    async obtener(id) {
        return await this._request(`/todos/${id}`);
    }
    
    async crear(tarea) {
        return await this._request("/todos", {
            method: "POST",
            body: JSON.stringify(tarea)
        });
    }
    
    async actualizar(id, cambios) {
        return await this._request(`/todos/${id}`, {
            method: "PATCH",
            body: JSON.stringify(cambios)
        });
    }
    
    async eliminar(id) {
        return await this._request(`/todos/${id}`, {
            method: "DELETE"
        });
    }
    
    async completar(id) {
        return await this.actualizar(id, { completed: true });
    }
    
    async obtenerCompletadas() {
        const todas = await this.listar();
        return todas.filter(tarea => tarea.completed);
    }
    
    async obtenerPendientes() {
        const todas = await this.listar();
        return todas.filter(tarea => !tarea.completed);
    }
}

// Uso
const api = new TareasAPI("https://jsonplaceholder.typicode.com");

async function ejemploCompleto() {
    try {
        // Listar todas
        console.log("=== Listando tareas ===");
        const tareas = await api.listar();
        console.log(`Total: ${tareas.length} tareas`);
        
        // Obtener una específica
        console.log("\n=== Obteniendo tarea 1 ===");
        const tarea = await api.obtener(1);
        console.log(tarea);
        
        // Crear nueva
        console.log("\n=== Creando tarea ===");
        const nueva = await api.crear({
            title: "Aprender async/await",
            completed: false,
            userId: 1
        });
        console.log("Creada:", nueva);
        
        // Actualizar
        console.log("\n=== Actualizando tarea ===");
        const actualizada = await api.actualizar(1, {
            title: "Tarea actualizada"
        });
        console.log("Actualizada:", actualizada);
        
        // Completar
        console.log("\n=== Completando tarea ===");
        const completada = await api.completar(1);
        console.log("Completada:", completada);
        
        // Obtener completadas
        console.log("\n=== Tareas completadas ===");
        const completadas = await api.obtenerCompletadas();
        console.log(`${completadas.length} tareas completadas`);
        
        // Obtener pendientes
        console.log("\n=== Tareas pendientes ===");
        const pendientes = await api.obtenerPendientes();
        console.log(`${pendientes.length} tareas pendientes`);
        
    } catch (error) {
        console.error("Error:", error);
    }
}

ejemploCompleto();
```

</details>

---

## Errores comunes

### ❌ Error 1: Olvidar await

```javascript
// ❌ Error: no espera
async function mal() {
    const usuario = obtenerUsuario(1); // Retorna Promise
    console.log(usuario.nombre); // undefined (es una Promise)
}

// ✅ Correcto
async function bien() {
    const usuario = await obtenerUsuario(1);
    console.log(usuario.nombre);
}
```

---

### ❌ Error 2: No usar try/catch

```javascript
// ❌ Error no manejado
async function mal() {
    const datos = await fetch("/api/datos"); // Puede fallar
    return datos.json();
}

// ✅ Con manejo de errores
async function bien() {
    try {
        const response = await fetch("/api/datos");
        return await response.json();
    } catch (error) {
        console.error("Error:", error);
        return null;
    }
}
```

---

### ❌ Error 3: Await en loops secuenciales

```javascript
// ❌ Secuencial (lento)
async function mal(ids) {
    const usuarios = [];
    for (const id of ids) {
        const usuario = await obtenerUsuario(id); // Uno por uno
        usuarios.push(usuario);
    }
    return usuarios;
}

// ✅ Paralelo (rápido)
async function bien(ids) {
    return await Promise.all(
        ids.map(id => obtenerUsuario(id))
    );
}
```

---

### ❌ Error 4: No retornar Promise

```javascript
// ❌ No retorna
async function mal() {
    fetch("/api/datos"); // Se pierde la Promise
}

// ✅ Retorna
async function bien() {
    return await fetch("/api/datos");
}
```

---

## Buenas prácticas

### ✅ 1. Usa async/await en lugar de then

```javascript
// ❌ Verboso
fetch("/api/usuarios")
    .then(r => r.json())
    .then(datos => console.log(datos))
    .catch(error => console.error(error));

// ✅ Más limpio
async function obtenerUsuarios() {
    try {
        const response = await fetch("/api/usuarios");
        const datos = await response.json();
        console.log(datos);
    } catch (error) {
        console.error(error);
    }
}
```

---

### ✅ 2. Usa Promise.all para operaciones paralelas

```javascript
// ✅ Paralelo cuando sea posible
const [usuarios, productos, categorias] = await Promise.all([
    obtenerUsuarios(),
    obtenerProductos(),
    obtenerCategorias()
]);
```

---

### ✅ 3. Maneja errores apropiadamente

```javascript
// ✅ Manejo específico por tipo
async function procesarDatos() {
    try {
        const datos = await obtenerDatos();
        return procesarDatos(datos);
    } catch (error) {
        if (error.name === "NetworkError") {
            return await obtenerDatosCache();
        }
        throw error;
    }
}
```

---

### ✅ 4. Usa finally para limpieza

```javascript
// ✅ Limpieza garantizada
async function cargar() {
    mostrarSpinner();
    try {
        return await obtenerDatos();
    } finally {
        ocultarSpinner(); // Siempre se ejecuta
    }
}
```

---

## Cheatsheet

### Crear Promise

```javascript
new Promise((resolve, reject) => {
    // resolve(valor) o reject(error)
});
```

### Consumir Promise

```javascript
promesa
    .then(resultado => {})
    .catch(error => {})
    .finally(() => {});
```

### async/await

```javascript
async function fn() {
    const resultado = await promesa;
    return resultado;
}
```

### try/catch

```javascript
try {
    const datos = await obtener();
} catch (error) {
    console.error(error);
} finally {
    limpiar();
}
```

### Promise.all

```javascript
const [a, b, c] = await Promise.all([p1, p2, p3]);
```

### Promise.race

```javascript
const primera = await Promise.race([p1, p2, p3]);
```

### Fetch

```javascript
const response = await fetch(url, {
    method: "POST",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify(data)
});
const datos = await response.json();
```

---

## Siguiente paso

Has dominado la programación asíncrona en JavaScript. Ahora aplicarás estos conocimientos consumiendo **APIs públicas reales**.

**Siguiente:** [Módulo 31 - Consumir APIs públicas](../bloque-06-async-apis/31-consumir-apis-publicas.md)

**Anterior:** [Módulo 29 - Destructuring y Spread](./29-destructuring-spread.md)

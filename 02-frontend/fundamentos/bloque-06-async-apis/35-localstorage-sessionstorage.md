# Módulo 35: LocalStorage y SessionStorage

## Índice
1. [¿Qué es Web Storage?](#qué-es-web-storage)
2. [LocalStorage](#localstorage)
3. [SessionStorage](#sessionstorage)
4. [Diferencias entre LocalStorage y SessionStorage](#diferencias-entre-localstorage-y-sessionstorage)
5. [Guardar objetos y arrays](#guardar-objetos-y-arrays)
6. [Casos de uso comunes](#casos-de-uso-comunes)
7. [Límites y restricciones](#límites-y-restricciones)
8. [Ejercicios prácticos](#ejercicios-prácticos)
9. [Errores comunes](#errores-comunes)
10. [Buenas prácticas](#buenas-prácticas)
11. [Cheatsheet](#cheatsheet)

---

## ¿Qué es Web Storage?

**Web Storage permite guardar datos en el navegador del usuario**.

### Tipos de Web Storage

| Tipo | Duración | Alcance |
|------|----------|---------|
| **LocalStorage** | Permanente (hasta que se borre) | Todo el navegador |
| **SessionStorage** | Temporal (se borra al cerrar pestaña) | Solo la pestaña actual |

### ¿Para qué sirve?

✅ **Guardar preferencias** del usuario  
✅ **Mantener sesión** (tokens)  
✅ **Caché de datos** para offline  
✅ **Estado temporal** de formularios  
✅ **Datos de UI** (tema, idioma, etc.)

### ⚠️ Limitaciones importantes

❌ **Solo strings** (no objetos directamente)  
❌ **Límite de ~5-10 MB** por origen  
❌ **Sincrónico** (puede bloquear la UI con datos grandes)  
❌ **No seguro** (JavaScript puede leerlo)  
❌ **Solo cliente** (no se envía al servidor automáticamente)

---

## LocalStorage

**Datos que persisten incluso después de cerrar el navegador**.

### Métodos básicos

```javascript
// Guardar
localStorage.setItem('clave', 'valor');

// Obtener
const valor = localStorage.getItem('clave');

// Eliminar una clave
localStorage.removeItem('clave');

// Eliminar todo
localStorage.clear();

// Verificar cantidad de items
console.log(localStorage.length);

// Obtener clave por índice
const clave = localStorage.key(0);
```

### Ejemplos prácticos

#### Guardar y recuperar datos

```javascript
// Guardar
localStorage.setItem('nombre', 'Ana García');
localStorage.setItem('email', 'ana@example.com');
localStorage.setItem('edad', '25');

// Recuperar
const nombre = localStorage.getItem('nombre');
console.log(nombre); // "Ana García"

// Verificar si existe
if (localStorage.getItem('nombre')) {
    console.log('El nombre existe');
}

// Eliminar
localStorage.removeItem('email');

// Limpiar todo
localStorage.clear();
```

#### Contador de visitas

```javascript
function contarVisitas() {
    let visitas = localStorage.getItem('visitas');
    
    if (visitas) {
        visitas = parseInt(visitas) + 1;
    } else {
        visitas = 1;
    }
    
    localStorage.setItem('visitas', visitas);
    
    console.log(`Has visitado esta página ${visitas} veces`);
}

contarVisitas();
```

#### Guardar preferencias de tema

```html
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Tema Oscuro/Claro</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            padding: 40px;
            transition: all 0.3s;
        }
        
        body.dark {
            background: #1a1a1a;
            color: #ffffff;
        }
        
        body.light {
            background: #ffffff;
            color: #000000;
        }
        
        button {
            padding: 10px 20px;
            font-size: 16px;
            cursor: pointer;
            border: none;
            border-radius: 4px;
        }
        
        .dark button {
            background: #ffffff;
            color: #000000;
        }
        
        .light button {
            background: #1a1a1a;
            color: #ffffff;
        }
    </style>
</head>
<body>
    <h1>Selector de Tema</h1>
    <p>Tu preferencia se guarda automáticamente</p>
    <button id="btnTema">Cambiar Tema</button>

    <script>
        // Cargar tema guardado al iniciar
        function cargarTema() {
            const temaGuardado = localStorage.getItem('tema') || 'light';
            document.body.className = temaGuardado;
            actualizarBoton();
        }
        
        // Cambiar tema
        function cambiarTema() {
            const temaActual = document.body.className;
            const nuevoTema = temaActual === 'light' ? 'dark' : 'light';
            
            document.body.className = nuevoTema;
            localStorage.setItem('tema', nuevoTema);
            actualizarBoton();
        }
        
        // Actualizar texto del botón
        function actualizarBoton() {
            const tema = document.body.className;
            const boton = document.getElementById('btnTema');
            boton.textContent = tema === 'light' ? '🌙 Modo Oscuro' : '☀️ Modo Claro';
        }
        
        // Event listener
        document.getElementById('btnTema').addEventListener('click', cambiarTema);
        
        // Inicializar
        cargarTema();
    </script>
</body>
</html>
```

---

## SessionStorage

**Datos que se borran al cerrar la pestaña**.

### API idéntica a localStorage

```javascript
// Guardar
sessionStorage.setItem('clave', 'valor');

// Obtener
const valor = sessionStorage.getItem('clave');

// Eliminar
sessionStorage.removeItem('clave');

// Limpiar todo
sessionStorage.clear();
```

### Casos de uso

#### Formulario multipaso

```javascript
// Página 1: Datos personales
function guardarPaso1() {
    const nombre = document.getElementById('nombre').value;
    const email = document.getElementById('email').value;
    
    sessionStorage.setItem('paso1_nombre', nombre);
    sessionStorage.setItem('paso1_email', email);
    
    window.location.href = 'paso2.html';
}

// Página 2: Recuperar datos
function cargarPaso1() {
    const nombre = sessionStorage.getItem('paso1_nombre');
    const email = sessionStorage.getItem('paso1_email');
    
    console.log('Datos del paso anterior:', { nombre, email });
}
```

#### Guardar scroll position

```javascript
// Guardar posición al cambiar de página
window.addEventListener('beforeunload', () => {
    sessionStorage.setItem('scrollPos', window.scrollY);
});

// Restaurar al volver
window.addEventListener('load', () => {
    const scrollPos = sessionStorage.getItem('scrollPos');
    if (scrollPos) {
        window.scrollTo(0, parseInt(scrollPos));
    }
});
```

---

## Diferencias entre LocalStorage y SessionStorage

### Comparación

| Característica | LocalStorage | SessionStorage |
|----------------|--------------|----------------|
| **Duración** | Permanente | Solo la sesión |
| **Alcance** | Todas las pestañas del mismo origen | Solo la pestaña actual |
| **Persistencia** | Se mantiene al cerrar navegador | Se borra al cerrar pestaña |
| **Tamaño** | ~5-10 MB | ~5-10 MB |
| **Uso típico** | Preferencias, tema, tokens | Datos temporales, formularios |

### Ejemplo comparativo

```javascript
// LocalStorage - Persiste
localStorage.setItem('usuario', 'Ana');
// Cierra el navegador, abre de nuevo
console.log(localStorage.getItem('usuario')); // ✅ "Ana" (sigue ahí)

// SessionStorage - Temporal
sessionStorage.setItem('temp', 'datos temporales');
// Cierra la pestaña, abre de nuevo
console.log(sessionStorage.getItem('temp')); // ❌ null (se borró)
```

---

## Guardar objetos y arrays

**Storage solo acepta strings, pero podemos usar JSON**.

### Guardar objetos

```javascript
const usuario = {
    nombre: 'Ana García',
    email: 'ana@example.com',
    edad: 25,
    premium: true
};

// ❌ Mal - Guarda "[object Object]"
localStorage.setItem('usuario', usuario);

// ✅ Bien - Convertir a JSON
localStorage.setItem('usuario', JSON.stringify(usuario));

// Recuperar
const usuarioGuardado = JSON.parse(localStorage.getItem('usuario'));
console.log(usuarioGuardado.nombre); // "Ana García"
```

### Guardar arrays

```javascript
const tareas = [
    { id: 1, texto: 'Comprar leche', completada: false },
    { id: 2, texto: 'Estudiar JavaScript', completada: true },
    { id: 3, texto: 'Hacer ejercicio', completada: false }
];

// Guardar
localStorage.setItem('tareas', JSON.stringify(tareas));

// Recuperar
const tareasGuardadas = JSON.parse(localStorage.getItem('tareas'));
console.log(tareasGuardadas); // Array de tareas
```

### Helper functions

```javascript
const storage = {
    // Guardar cualquier tipo de dato
    set(clave, valor) {
        localStorage.setItem(clave, JSON.stringify(valor));
    },
    
    // Obtener y parsear automáticamente
    get(clave) {
        const valor = localStorage.getItem(clave);
        if (!valor) return null;
        
        try {
            return JSON.parse(valor);
        } catch {
            return valor; // Si no es JSON, devolver como string
        }
    },
    
    // Eliminar
    remove(clave) {
        localStorage.removeItem(clave);
    },
    
    // Limpiar todo
    clear() {
        localStorage.clear();
    }
};

// Uso
storage.set('usuario', { nombre: 'Ana', edad: 25 });
storage.set('contador', 42);
storage.set('texto', 'Hola mundo');

const usuario = storage.get('usuario');
console.log(usuario.nombre); // "Ana"
```

---

## Casos de uso comunes

### 1. Sistema de autenticación

```javascript
// Login
async function login(email, password) {
    const respuesta = await fetch('https://api.ejemplo.com/login', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ email, password })
    });
    
    const datos = await respuesta.json();
    
    // Guardar token
    localStorage.setItem('token', datos.token);
    localStorage.setItem('usuario', JSON.stringify(datos.usuario));
}

// Verificar si está autenticado
function estaAutenticado() {
    return localStorage.getItem('token') !== null;
}

// Logout
function logout() {
    localStorage.removeItem('token');
    localStorage.removeItem('usuario');
    window.location.href = '/login';
}

// Obtener usuario actual
function obtenerUsuario() {
    const usuarioJSON = localStorage.getItem('usuario');
    return usuarioJSON ? JSON.parse(usuarioJSON) : null;
}
```

### 2. Lista de favoritos

```javascript
class Favoritos {
    constructor() {
        this.clave = 'favoritos';
    }
    
    obtenerTodos() {
        const favs = localStorage.getItem(this.clave);
        return favs ? JSON.parse(favs) : [];
    }
    
    agregar(item) {
        const favoritos = this.obtenerTodos();
        if (!this.existe(item.id)) {
            favoritos.push(item);
            localStorage.setItem(this.clave, JSON.stringify(favoritos));
        }
    }
    
    eliminar(id) {
        let favoritos = this.obtenerTodos();
        favoritos = favoritos.filter(item => item.id !== id);
        localStorage.setItem(this.clave, JSON.stringify(favoritos));
    }
    
    existe(id) {
        return this.obtenerTodos().some(item => item.id === id);
    }
    
    contar() {
        return this.obtenerTodos().length;
    }
}

// Uso
const favs = new Favoritos();
favs.agregar({ id: 1, nombre: 'Producto 1' });
favs.agregar({ id: 2, nombre: 'Producto 2' });
console.log(favs.obtenerTodos());
console.log('Total favoritos:', favs.contar());
```

### 3. Caché de datos de API

```javascript
const cache = {
    // Guardar con timestamp
    set(clave, datos, ttl = 3600000) { // TTL default: 1 hora
        const item = {
            datos,
            timestamp: Date.now(),
            ttl
        };
        localStorage.setItem(clave, JSON.stringify(item));
    },
    
    // Obtener si no expiró
    get(clave) {
        const itemJSON = localStorage.getItem(clave);
        if (!itemJSON) return null;
        
        const item = JSON.parse(itemJSON);
        const ahora = Date.now();
        
        // Verificar si expiró
        if (ahora - item.timestamp > item.ttl) {
            localStorage.removeItem(clave);
            return null;
        }
        
        return item.datos;
    }
};

// Función con caché
async function obtenerPosts() {
    // Intentar obtener del caché
    const cacheado = cache.get('posts');
    if (cacheado) {
        console.log('Datos del caché');
        return cacheado;
    }
    
    // Si no hay caché, obtener de la API
    console.log('Obteniendo de la API...');
    const respuesta = await fetch('https://jsonplaceholder.typicode.com/posts');
    const posts = await respuesta.json();
    
    // Guardar en caché (válido por 5 minutos)
    cache.set('posts', posts, 5 * 60 * 1000);
    
    return posts;
}
```

### 4. Carrito de compras

```javascript
class Carrito {
    constructor() {
        this.clave = 'carrito';
    }
    
    obtenerProductos() {
        const carrito = localStorage.getItem(this.clave);
        return carrito ? JSON.parse(carrito) : [];
    }
    
    agregar(producto, cantidad = 1) {
        const productos = this.obtenerProductos();
        const existente = productos.find(p => p.id === producto.id);
        
        if (existente) {
            existente.cantidad += cantidad;
        } else {
            productos.push({ ...producto, cantidad });
        }
        
        this.guardar(productos);
    }
    
    eliminar(idProducto) {
        let productos = this.obtenerProductos();
        productos = productos.filter(p => p.id !== idProducto);
        this.guardar(productos);
    }
    
    actualizarCantidad(idProducto, cantidad) {
        const productos = this.obtenerProductos();
        const producto = productos.find(p => p.id === idProducto);
        
        if (producto) {
            producto.cantidad = cantidad;
            this.guardar(productos);
        }
    }
    
    vaciar() {
        localStorage.removeItem(this.clave);
    }
    
    calcularTotal() {
        const productos = this.obtenerProductos();
        return productos.reduce((total, p) => total + (p.precio * p.cantidad), 0);
    }
    
    guardar(productos) {
        localStorage.setItem(this.clave, JSON.stringify(productos));
    }
}

// Uso
const carrito = new Carrito();
carrito.agregar({ id: 1, nombre: 'Laptop', precio: 999 }, 1);
carrito.agregar({ id: 2, nombre: 'Mouse', precio: 29 }, 2);
console.log('Total:', carrito.calcularTotal());
```

---

## Límites y restricciones

### Tamaño máximo

```javascript
// Verificar cuánto espacio usas
function calcularTamañoStorage() {
    let total = 0;
    
    for (let clave in localStorage) {
        if (localStorage.hasOwnProperty(clave)) {
            total += localStorage[clave].length + clave.length;
        }
    }
    
    const kb = (total / 1024).toFixed(2);
    const mb = (total / (1024 * 1024)).toFixed(2);
    
    console.log(`Espacio usado: ${total} bytes (${kb} KB / ${mb} MB)`);
}

calcularTamañoStorage();
```

### Manejar cuota excedida

```javascript
function guardarSeguro(clave, valor) {
    try {
        localStorage.setItem(clave, valor);
    } catch (e) {
        if (e.name === 'QuotaExceededError') {
            console.error('Storage lleno. Limpiando datos antiguos...');
            // Opción 1: Limpiar todo
            localStorage.clear();
            // Opción 2: Limpiar selectivamente
            // limpiarDatosAntiguos();
            // Reintentar
            localStorage.setItem(clave, valor);
        } else {
            console.error('Error al guardar:', e);
        }
    }
}
```

---

## Ejercicios prácticos

### Ejercicio 1: Preferencias de usuario
Crea una app que guarde idioma, tema y tamaño de fuente.

### Ejercicio 2: Lista de tareas con persistencia
Crea una todo app donde las tareas se guarden en localStorage.

### Ejercicio 3: Historial de búsquedas
Guarda las últimas 10 búsquedas del usuario.

### Ejercicio 4: Carrito de compras
Implementa un carrito completo con localStorage.

### Ejercicio 5: Modo offline
Crea una app que funcione sin internet usando caché en localStorage.

---

## Errores comunes

### ❌ No usar JSON.stringify/parse

```javascript
// Mal
localStorage.setItem('usuario', {nombre: 'Ana'}); // Guarda "[object Object]"

// Bien
localStorage.setItem('usuario', JSON.stringify({nombre: 'Ana'}));
```

### ❌ No verificar si existe

```javascript
// Mal - Puede causar errores
const usuario = JSON.parse(localStorage.getItem('usuario'));

// Bien - Verificar primero
const usuarioJSON = localStorage.getItem('usuario');
const usuario = usuarioJSON ? JSON.parse(usuarioJSON) : null;
```

### ❌ Guardar datos sensibles

```javascript
// ❌ NUNCA guardes esto en localStorage
localStorage.setItem('password', 'secreto123');
localStorage.setItem('tarjetaCredito', '1234-5678-9012-3456');

// ✅ Está bien guardar
localStorage.setItem('tema', 'dark');
localStorage.setItem('idioma', 'es');
```

---

## Buenas prácticas

### ✅ Usar prefijos para las claves

```javascript
const APP_PREFIX = 'miapp_';

storage.set = (key, value) => {
    localStorage.setItem(APP_PREFIX + key, JSON.stringify(value));
};

storage.get = (key) => {
    const value = localStorage.getItem(APP_PREFIX + key);
    return value ? JSON.parse(value) : null;
};
```

### ✅ Limpiar datos expirados

```javascript
function limpiarExpirados() {
    const ahora = Date.now();
    
    Object.keys(localStorage).forEach(clave => {
        try {
            const item = JSON.parse(localStorage.getItem(clave));
            if (item.expiracion && item.expiracion < ahora) {
                localStorage.removeItem(clave);
            }
        } catch {
            // Si no es JSON, ignorar
        }
    });
}

// Limpiar al cargar la app
limpiarExpirados();
```

### ✅ Escuchar cambios en storage

```javascript
// Detectar cambios en otras pestañas
window.addEventListener('storage', (e) => {
    console.log('Cambio en storage:');
    console.log('Clave:', e.key);
    console.log('Valor anterior:', e.oldValue);
    console.log('Valor nuevo:', e.newValue);
    console.log('URL:', e.url);
});
```

---

## Cheatsheet

```javascript
// ========== LocalStorage ==========
localStorage.setItem('clave', 'valor');
const valor = localStorage.getItem('clave');
localStorage.removeItem('clave');
localStorage.clear();

// ========== SessionStorage ==========
sessionStorage.setItem('clave', 'valor');
const valor = sessionStorage.getItem('clave');
sessionStorage.removeItem('clave');
sessionStorage.clear();

// ========== Objetos y Arrays ==========
localStorage.setItem('datos', JSON.stringify(objeto));
const datos = JSON.parse(localStorage.getItem('datos'));

// ========== Verificar existencia ==========
if (localStorage.getItem('clave')) {
    // Existe
}

// ========== Helper seguro ==========
const storage = {
    set: (k, v) => localStorage.setItem(k, JSON.stringify(v)),
    get: (k) => {
        const v = localStorage.getItem(k);
        return v ? JSON.parse(v) : null;
    },
    remove: (k) => localStorage.removeItem(k)
};
```

---

**Siguiente:** [Módulo 36 - Patrones async avanzados](./36-patrones-async-avanzados.md)

**Anterior:** [Módulo 34 - Headers y autenticación](./34-headers-autenticacion.md)

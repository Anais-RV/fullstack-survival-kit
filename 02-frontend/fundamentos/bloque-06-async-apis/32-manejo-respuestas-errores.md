# Módulo 32: Manejo de Respuestas y Errores

## Índice
1. [Códigos de estado HTTP](#códigos-de-estado-http)
2. [Verificar respuestas exitosas](#verificar-respuestas-exitosas)
3. [Manejo de errores](#manejo-de-errores)
4. [Estados de carga (Loading states)](#estados-de-carga-loading-states)
5. [Timeout y reintentos](#timeout-y-reintentos)
6. [Errores de red vs errores HTTP](#errores-de-red-vs-errores-http)
7. [Ejercicios prácticos](#ejercicios-prácticos)
8. [Errores comunes](#errores-comunes)
9. [Buenas prácticas](#buenas-prácticas)
10. [Cheatsheet](#cheatsheet)

---

## Códigos de estado HTTP

**Los códigos HTTP indican el resultado de una petición**.

### Tipos de códigos

| Rango | Significado | Ejemplos |
|-------|-------------|----------|
| **2xx** | ✅ Éxito | 200 (OK), 201 (Creado), 204 (Sin contenido) |
| **3xx** | ↪️ Redirección | 301 (Movido permanentemente), 304 (No modificado) |
| **4xx** | ❌ Error del cliente | 400 (Bad Request), 404 (Not Found), 401 (No autorizado) |
| **5xx** | 💥 Error del servidor | 500 (Internal Server Error), 503 (Servicio no disponible) |

### Códigos más comunes

```javascript
// 200 OK - Todo correcto
fetch('https://jsonplaceholder.typicode.com/posts/1')
// → Status: 200

// 404 Not Found - Recurso no existe
fetch('https://jsonplaceholder.typicode.com/posts/99999')
// → Status: 404

// 500 Internal Server Error - Error del servidor
fetch('https://ejemplo-servidor-roto.com/api')
// → Status: 500
```

---

## Verificar respuestas exitosas

### ⚠️ Fetch NO rechaza promesas con errores HTTP

```javascript
// ❌ PROBLEMA: Esto NO captura errores 404, 500, etc.
try {
    const respuesta = await fetch('https://jsonplaceholder.typicode.com/posts/99999');
    const datos = await respuesta.json(); // ❌ Puede fallar aquí
    console.log(datos);
} catch (error) {
    console.error('Error:', error); // ❌ NO se ejecuta con 404
}
```

### ✅ Solución: Verificar `response.ok`

```javascript
try {
    const respuesta = await fetch('https://jsonplaceholder.typicode.com/posts/99999');
    
    // ✅ Verificar si la respuesta es exitosa (status 200-299)
    if (!respuesta.ok) {
        throw new Error(`Error HTTP: ${respuesta.status}`);
    }
    
    const datos = await respuesta.json();
    console.log(datos);
} catch (error) {
    console.error('Error:', error); // ✅ Ahora SÍ captura el error
}
```

### Propiedades de Response

```javascript
const respuesta = await fetch(url);

console.log(respuesta.ok);          // true si status 200-299
console.log(respuesta.status);      // Código numérico (200, 404, 500...)
console.log(respuesta.statusText);  // Texto del estado ("OK", "Not Found"...)
console.log(respuesta.headers);     // Headers de la respuesta
console.log(respuesta.url);         // URL final (puede ser diferente si hubo redirección)
```

### Ejemplo completo

```javascript
async function obtenerPost(id) {
    try {
        const respuesta = await fetch(`https://jsonplaceholder.typicode.com/posts/${id}`);
        
        console.log('Status:', respuesta.status);
        console.log('OK?', respuesta.ok);
        console.log('Status Text:', respuesta.statusText);
        
        if (!respuesta.ok) {
            throw new Error(`Error ${respuesta.status}: ${respuesta.statusText}`);
        }
        
        const datos = await respuesta.json();
        return datos;
        
    } catch (error) {
        console.error('Error al obtener post:', error.message);
        return null;
    }
}

// Prueba con ID válido
await obtenerPost(1); // ✅ Status: 200

// Prueba con ID inválido
await obtenerPost(99999); // ❌ Error 404: Not Found
```

---

## Manejo de errores

### Estructura básica con try/catch

```javascript
async function obtenerDatos(url) {
    try {
        const respuesta = await fetch(url);
        
        if (!respuesta.ok) {
            throw new Error(`Error HTTP: ${respuesta.status}`);
        }
        
        return await respuesta.json();
        
    } catch (error) {
        console.error('Error:', error.message);
        throw error; // Re-lanzar para que el llamador pueda manejarlo
    }
}
```

### Errores específicos según status

```javascript
async function obtenerDatosConErrores(url) {
    try {
        const respuesta = await fetch(url);
        
        // Manejar diferentes tipos de errores
        if (respuesta.status === 404) {
            throw new Error('Recurso no encontrado');
        }
        
        if (respuesta.status === 401) {
            throw new Error('No autorizado. Inicia sesión primero');
        }
        
        if (respuesta.status === 500) {
            throw new Error('Error del servidor. Intenta más tarde');
        }
        
        if (!respuesta.ok) {
            throw new Error(`Error ${respuesta.status}: ${respuesta.statusText}`);
        }
        
        return await respuesta.json();
        
    } catch (error) {
        console.error('Error:', error.message);
        return null;
    }
}
```

### Función reutilizable

```javascript
async function fetchJSON(url, opciones = {}) {
    try {
        const respuesta = await fetch(url, opciones);
        
        // Verificar tipo de contenido
        const contentType = respuesta.headers.get('content-type');
        if (!contentType || !contentType.includes('application/json')) {
            throw new Error('La respuesta no es JSON');
        }
        
        if (!respuesta.ok) {
            // Intentar leer mensaje de error del servidor
            let mensajeError = `Error ${respuesta.status}`;
            try {
                const errorData = await respuesta.json();
                mensajeError = errorData.message || mensajeError;
            } catch {
                // Si no se puede parsear, usar mensaje genérico
            }
            throw new Error(mensajeError);
        }
        
        return await respuesta.json();
        
    } catch (error) {
        // Distinguir entre errores de red y errores HTTP
        if (error.name === 'TypeError' && error.message.includes('fetch')) {
            throw new Error('Error de red. Verifica tu conexión');
        }
        throw error;
    }
}

// Uso
try {
    const datos = await fetchJSON('https://jsonplaceholder.typicode.com/posts/1');
    console.log(datos);
} catch (error) {
    console.error('Error:', error.message);
}
```

---

## Estados de carga (Loading states)

**Mostrar al usuario qué está pasando durante la petición**.

### Estados posibles

1. **Idle** - Sin cargar (estado inicial)
2. **Loading** - Cargando datos
3. **Success** - Datos cargados correctamente
4. **Error** - Error al cargar

### Implementación con HTML

```html
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Estados de carga</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            max-width: 800px;
            margin: 40px auto;
            padding: 20px;
        }
        button {
            padding: 10px 20px;
            font-size: 16px;
            background: #3498db;
            color: white;
            border: none;
            cursor: pointer;
            border-radius: 4px;
        }
        button:hover {
            background: #2980b9;
        }
        button:disabled {
            background: #95a5a6;
            cursor: not-allowed;
        }
        #resultado {
            margin-top: 20px;
        }
        .loading {
            text-align: center;
            color: #3498db;
            font-size: 18px;
        }
        .success {
            background: #d4edda;
            border: 1px solid #c3e6cb;
            color: #155724;
            padding: 15px;
            border-radius: 4px;
        }
        .error {
            background: #f8d7da;
            border: 1px solid #f5c6cb;
            color: #721c24;
            padding: 15px;
            border-radius: 4px;
        }
        .post {
            background: #f5f5f5;
            padding: 15px;
            margin-bottom: 10px;
            border-radius: 4px;
        }
        .spinner {
            border: 4px solid #f3f3f3;
            border-top: 4px solid #3498db;
            border-radius: 50%;
            width: 40px;
            height: 40px;
            animation: spin 1s linear infinite;
            margin: 20px auto;
        }
        @keyframes spin {
            0% { transform: rotate(0deg); }
            100% { transform: rotate(360deg); }
        }
    </style>
</head>
<body>
    <h1>Estados de Carga</h1>
    
    <button id="btnCargar" onclick="cargarPosts()">Cargar Posts</button>
    <button onclick="simularError()">Simular Error</button>
    
    <div id="resultado"></div>

    <script>
        // Estados
        let estado = 'idle'; // idle | loading | success | error

        async function cargarPosts() {
            const resultado = document.getElementById('resultado');
            const btnCargar = document.getElementById('btnCargar');
            
            // Estado: LOADING
            estado = 'loading';
            btnCargar.disabled = true;
            resultado.innerHTML = `
                <div class="loading">
                    <div class="spinner"></div>
                    <p>Cargando posts...</p>
                </div>
            `;
            
            try {
                const respuesta = await fetch('https://jsonplaceholder.typicode.com/posts?_limit=5');
                
                if (!respuesta.ok) {
                    throw new Error(`Error HTTP: ${respuesta.status}`);
                }
                
                const posts = await respuesta.json();
                
                // Estado: SUCCESS
                estado = 'success';
                resultado.innerHTML = `
                    <div class="success">
                        <h3>✅ Posts cargados correctamente</h3>
                    </div>
                    ${posts.map(post => `
                        <div class="post">
                            <h4>${post.title}</h4>
                            <p>${post.body}</p>
                        </div>
                    `).join('')}
                `;
                
            } catch (error) {
                // Estado: ERROR
                estado = 'error';
                resultado.innerHTML = `
                    <div class="error">
                        <h3>❌ Error al cargar posts</h3>
                        <p>${error.message}</p>
                        <button onclick="cargarPosts()">Reintentar</button>
                    </div>
                `;
            } finally {
                btnCargar.disabled = false;
            }
        }
        
        async function simularError() {
            const resultado = document.getElementById('resultado');
            resultado.innerHTML = '<div class="loading"><div class="spinner"></div><p>Cargando...</p></div>';
            
            try {
                // URL inválida para provocar error
                const respuesta = await fetch('https://jsonplaceholder.typicode.com/posts/99999');
                
                if (!respuesta.ok) {
                    throw new Error('Post no encontrado (404)');
                }
                
            } catch (error) {
                resultado.innerHTML = `
                    <div class="error">
                        <h3>❌ Error</h3>
                        <p>${error.message}</p>
                    </div>
                `;
            }
        }
    </script>
</body>
</html>
```

### Con variables de estado

```javascript
class EstadoCarga {
    constructor() {
        this.loading = false;
        this.data = null;
        this.error = null;
    }
    
    setLoading() {
        this.loading = true;
        this.data = null;
        this.error = null;
    }
    
    setSuccess(data) {
        this.loading = false;
        this.data = data;
        this.error = null;
    }
    
    setError(error) {
        this.loading = false;
        this.data = null;
        this.error = error;
    }
}

const estado = new EstadoCarga();

async function cargarDatos() {
    estado.setLoading();
    console.log('Estado:', estado); // { loading: true, data: null, error: null }
    
    try {
        const respuesta = await fetch(url);
        if (!respuesta.ok) throw new Error('Error en la petición');
        
        const datos = await respuesta.json();
        estado.setSuccess(datos);
        console.log('Estado:', estado); // { loading: false, data: [...], error: null }
        
    } catch (error) {
        estado.setError(error.message);
        console.log('Estado:', estado); // { loading: false, data: null, error: "..." }
    }
}
```

---

## Timeout y reintentos

### Timeout (tiempo máximo de espera)

```javascript
async function fetchConTimeout(url, timeout = 5000) {
    const controller = new AbortController();
    const timeoutId = setTimeout(() => controller.abort(), timeout);
    
    try {
        const respuesta = await fetch(url, { signal: controller.signal });
        clearTimeout(timeoutId);
        
        if (!respuesta.ok) {
            throw new Error(`Error HTTP: ${respuesta.status}`);
        }
        
        return await respuesta.json();
        
    } catch (error) {
        clearTimeout(timeoutId);
        
        if (error.name === 'AbortError') {
            throw new Error('La petición tardó demasiado (timeout)');
        }
        
        throw error;
    }
}

// Uso
try {
    const datos = await fetchConTimeout('https://jsonplaceholder.typicode.com/posts', 3000);
    console.log(datos);
} catch (error) {
    console.error('Error:', error.message);
}
```

### Reintentos automáticos

```javascript
async function fetchConReintentos(url, intentos = 3) {
    for (let i = 0; i < intentos; i++) {
        try {
            console.log(`Intento ${i + 1} de ${intentos}`);
            
            const respuesta = await fetch(url);
            
            if (!respuesta.ok) {
                throw new Error(`Error HTTP: ${respuesta.status}`);
            }
            
            return await respuesta.json();
            
        } catch (error) {
            console.error(`Intento ${i + 1} falló:`, error.message);
            
            // Si es el último intento, lanzar el error
            if (i === intentos - 1) {
                throw new Error(`Falló después de ${intentos} intentos: ${error.message}`);
            }
            
            // Esperar antes del siguiente intento (exponential backoff)
            await new Promise(resolve => setTimeout(resolve, 1000 * (i + 1)));
        }
    }
}

// Uso
try {
    const datos = await fetchConReintentos('https://jsonplaceholder.typicode.com/posts/1');
    console.log('Datos obtenidos:', datos);
} catch (error) {
    console.error('Error final:', error.message);
}
```

### Combinar timeout y reintentos

```javascript
async function fetchRobusto(url, opciones = {}) {
    const { timeout = 5000, intentos = 3 } = opciones;
    
    for (let i = 0; i < intentos; i++) {
        try {
            const datos = await fetchConTimeout(url, timeout);
            return datos;
        } catch (error) {
            console.error(`Intento ${i + 1}/${intentos} falló:`, error.message);
            
            if (i === intentos - 1) {
                throw error;
            }
            
            // Esperar antes de reintentar
            await new Promise(resolve => setTimeout(resolve, 1000 * (i + 1)));
        }
    }
}
```

---

## Errores de red vs errores HTTP

### Diferencias

| Tipo | Cuándo ocurre | Código de estado |
|------|---------------|------------------|
| **Error de red** | Sin conexión, DNS fallo, CORS | Sin status (TypeError) |
| **Error HTTP** | El servidor respondió con error | 404, 500, etc. |

### Distinguir entre tipos de error

```javascript
async function manejarErrores(url) {
    try {
        const respuesta = await fetch(url);
        
        if (!respuesta.ok) {
            // Error HTTP (4xx, 5xx)
            throw new Error(`Error HTTP ${respuesta.status}: ${respuesta.statusText}`);
        }
        
        return await respuesta.json();
        
    } catch (error) {
        // Distinguir tipo de error
        if (error.name === 'TypeError' && error.message.includes('fetch')) {
            console.error('❌ Error de red:', 'Sin conexión a internet o servidor no disponible');
        } else if (error.message.includes('HTTP')) {
            console.error('❌ Error del servidor:', error.message);
        } else {
            console.error('❌ Error desconocido:', error.message);
        }
        
        throw error;
    }
}
```

### Ejemplo completo con mensajes amigables

```javascript
async function obtenerDatosConMensajes(url) {
    try {
        const respuesta = await fetch(url);
        
        // Errores HTTP específicos
        switch (respuesta.status) {
            case 404:
                throw new Error('😕 No encontramos lo que buscas');
            case 401:
                throw new Error('🔒 Necesitas iniciar sesión');
            case 403:
                throw new Error('🚫 No tienes permiso para acceder');
            case 500:
                throw new Error('💥 El servidor tiene problemas. Intenta más tarde');
            case 503:
                throw new Error('⏳ El servidor está en mantenimiento');
        }
        
        if (!respuesta.ok) {
            throw new Error(`Error ${respuesta.status}: ${respuesta.statusText}`);
        }
        
        return await respuesta.json();
        
    } catch (error) {
        // Error de red
        if (error.name === 'TypeError') {
            return {
                error: true,
                mensaje: '📡 Sin conexión a internet. Verifica tu red'
            };
        }
        
        // Otros errores
        return {
            error: true,
            mensaje: error.message
        };
    }
}
```

---

## Ejercicios prácticos

### Ejercicio 1: Sistema de notificaciones
Crea una función que muestre notificaciones toast según el estado de la petición (loading, success, error).

### Ejercicio 2: Buscador con validación
Crea un buscador que valide que la API devolvió resultados y muestre mensaje apropiado si no encuentra nada.

### Ejercicio 3: Dashboard con estados
Crea un dashboard que muestre el estado de múltiples APIs simultáneamente.

### Ejercicio 4: Reintentos con feedback visual
Implementa reintentos que muestren al usuario cuántos intentos quedan.

### Ejercicio 5: Timeout configurable
Crea una app donde el usuario pueda configurar el timeout y ver cómo afecta a las peticiones.

---

## Errores comunes

### ❌ No verificar response.ok

```javascript
// Mal
const datos = await fetch(url).then(r => r.json());

// Bien
const respuesta = await fetch(url);
if (!respuesta.ok) throw new Error('Error en la petición');
const datos = await respuesta.json();
```

### ❌ Asumir que siempre llega JSON

```javascript
// Mal
const datos = await respuesta.json(); // Puede fallar si no es JSON

// Bien
const contentType = respuesta.headers.get('content-type');
if (contentType && contentType.includes('application/json')) {
    const datos = await respuesta.json();
} else {
    const texto = await respuesta.text();
}
```

### ❌ No mostrar estados de carga

```javascript
// Mal - Usuario no sabe qué está pasando
const datos = await fetch(url).then(r => r.json());
mostrarDatos(datos);

// Bien - Usuario ve el progreso
mostrarLoading();
try {
    const datos = await fetch(url).then(r => r.json());
    mostrarDatos(datos);
} catch (error) {
    mostrarError(error);
}
```

---

## Buenas prácticas

### ✅ Siempre manejar errores

```javascript
async function obtenerDatos(url) {
    try {
        const respuesta = await fetch(url);
        if (!respuesta.ok) throw new Error(`Error ${respuesta.status}`);
        return await respuesta.json();
    } catch (error) {
        console.error('Error:', error);
        throw error;
    }
}
```

### ✅ Mostrar mensajes claros al usuario

```javascript
const MENSAJES_ERROR = {
    404: 'No encontramos lo que buscas',
    500: 'El servidor tiene problemas. Intenta más tarde',
    'network': 'Sin conexión a internet'
};

function obtenerMensajeError(error, status) {
    return MENSAJES_ERROR[status] || MENSAJES_ERROR['network'] || 'Error desconocido';
}
```

### ✅ Deshabilitar botones durante la carga

```javascript
async function cargarDatos() {
    const boton = document.getElementById('btnCargar');
    boton.disabled = true;
    boton.textContent = 'Cargando...';
    
    try {
        const datos = await fetch(url).then(r => r.json());
        mostrarDatos(datos);
    } finally {
        boton.disabled = false;
        boton.textContent = 'Cargar';
    }
}
```

---

## Cheatsheet

```javascript
// ========== Verificar respuesta ==========
const respuesta = await fetch(url);
if (!respuesta.ok) {
    throw new Error(`Error ${respuesta.status}: ${respuesta.statusText}`);
}
const datos = await respuesta.json();

// ========== Función reutilizable ==========
async function fetchJSON(url) {
    try {
        const r = await fetch(url);
        if (!r.ok) throw new Error(`HTTP ${r.status}`);
        return await r.json();
    } catch (e) {
        console.error('Error:', e.message);
        throw e;
    }
}

// ========== Con timeout ==========
const controller = new AbortController();
setTimeout(() => controller.abort(), 5000);
const respuesta = await fetch(url, { signal: controller.signal });

// ========== Con reintentos ==========
async function fetchConReintentos(url, intentos = 3) {
    for (let i = 0; i < intentos; i++) {
        try {
            return await fetchJSON(url);
        } catch (error) {
            if (i === intentos - 1) throw error;
            await new Promise(r => setTimeout(r, 1000 * (i + 1)));
        }
    }
}

// ========== Estados de carga ==========
// idle → loading → success | error
let loading = false;
let error = null;
let data = null;
```

---

**Siguiente:** [Módulo 33 - Peticiones POST/PUT/DELETE](./33-peticiones-post-put-delete.md)

**Anterior:** [Módulo 31 - Consumir APIs públicas](./31-consumir-apis-publicas.md)

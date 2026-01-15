# Módulo 36: Patrones Async Avanzados

## Índice
1. [Debounce](#debounce)
2. [Throttle](#throttle)
3. [Retry automático](#retry-automático)
4. [Caché inteligente](#caché-inteligente)
5. [Promise.all y Promise.race](#promiseall-y-promiserace)
6. [AbortController](#abortcontroller)
7. [Ejercicios prácticos](#ejercicios-prácticos)
8. [Cheatsheet](#cheatsheet)

---

## Debounce

**Esperar a que el usuario deje de hacer algo antes de ejecutar**.

### ¿Cuándo usarlo?
- Búsqueda mientras escribes
- Validación de formularios
- Resize de ventana

### Implementación

```javascript
function debounce(func, delay = 300) {
    let timeoutId;
    
    return function(...args) {
        clearTimeout(timeoutId);
        timeoutId = setTimeout(() => func.apply(this, args), delay);
    };
}

// Uso
const buscar = debounce(async (termino) => {
    console.log('Buscando:', termino);
    const respuesta = await fetch(`https://api.ejemplo.com/search?q=${termino}`);
    const resultados = await respuesta.json();
    mostrarResultados(resultados);
}, 500);

// Solo ejecuta después de 500ms sin escribir
document.getElementById('input').addEventListener('input', (e) => {
    buscar(e.target.value);
});
```

### Ejemplo: Búsqueda en tiempo real

```html
<input type="text" id="busqueda" placeholder="Buscar...">
<div id="resultados"></div>

<script>
function debounce(func, delay) {
    let timeout;
    return (...args) => {
        clearTimeout(timeout);
        timeout = setTimeout(() => func(...args), delay);
    };
}

const buscarAPI = debounce(async (termino) => {
    if (!termino) {
        document.getElementById('resultados').innerHTML = '';
        return;
    }
    
    const url = `https://jsonplaceholder.typicode.com/posts?q=${termino}`;
    const respuesta = await fetch(url);
    const posts = await respuesta.json();
    
    document.getElementById('resultados').innerHTML = posts
        .slice(0, 5)
        .map(p => `<div>${p.title}</div>`)
        .join('');
}, 500);

document.getElementById('busqueda').addEventListener('input', (e) => {
    buscarAPI(e.target.value);
});
</script>
```

---

## Throttle

**Limitar cuántas veces se ejecuta una función en un período**.

### ¿Cuándo usarlo?
- Scroll infinito
- Eventos de mouse/touch
- Animaciones

### Implementación

```javascript
function throttle(func, limit = 1000) {
    let inThrottle;
    
    return function(...args) {
        if (!inThrottle) {
            func.apply(this, args);
            inThrottle = true;
            setTimeout(() => inThrottle = false, limit);
        }
    };
}

// Uso
const handleScroll = throttle(() => {
    console.log('Scroll detectado');
}, 1000);

window.addEventListener('scroll', handleScroll);
```

---

## Retry automático

**Reintentar peticiones fallidas automáticamente**.

```javascript
async function fetchConReintentos(url, opciones = {}) {
    const { intentos = 3, delay = 1000 } = opciones;
    
    for (let i = 0; i < intentos; i++) {
        try {
            const respuesta = await fetch(url);
            if (!respuesta.ok) throw new Error(`HTTP ${respuesta.status}`);
            return await respuesta.json();
        } catch (error) {
            console.log(`Intento ${i + 1}/${intentos} falló`);
            
            if (i === intentos - 1) throw error;
            
            // Exponential backoff
            await new Promise(resolve => setTimeout(resolve, delay * (i + 1)));
        }
    }
}

// Uso
try {
    const datos = await fetchConReintentos('https://api.ejemplo.com/data', {
        intentos: 3,
        delay: 1000
    });
} catch (error) {
    console.error('Error después de todos los intentos:', error);
}
```

---

## Caché inteligente

**Guardar respuestas para no repetir peticiones**.

```javascript
class CacheAPI {
    constructor(ttl = 5 * 60 * 1000) { // 5 minutos
        this.cache = new Map();
        this.ttl = ttl;
    }
    
    async fetch(url) {
        // Verificar caché
        const cached = this.cache.get(url);
        if (cached && Date.now() - cached.timestamp < this.ttl) {
            console.log('Datos del caché');
            return cached.data;
        }
        
        // Obtener de la API
        console.log('Obteniendo de la API...');
        const respuesta = await fetch(url);
        const data = await respuesta.json();
        
        // Guardar en caché
        this.cache.set(url, {
            data,
            timestamp: Date.now()
        });
        
        return data;
    }
    
    clear() {
        this.cache.clear();
    }
    
    remove(url) {
        this.cache.delete(url);
    }
}

// Uso
const api = new CacheAPI(5 * 60 * 1000);

// Primera llamada - obtiene de la API
const posts1 = await api.fetch('https://jsonplaceholder.typicode.com/posts');

// Segunda llamada (dentro de 5 min) - obtiene del caché
const posts2 = await api.fetch('https://jsonplaceholder.typicode.com/posts');
```

---

## Promise.all y Promise.race

### Promise.all - Esperar todas

```javascript
async function obtenerTodo() {
    try {
        const [posts, users, todos] = await Promise.all([
            fetch('https://jsonplaceholder.typicode.com/posts').then(r => r.json()),
            fetch('https://jsonplaceholder.typicode.com/users').then(r => r.json()),
            fetch('https://jsonplaceholder.typicode.com/todos').then(r => r.json())
        ]);
        
        console.log('Posts:', posts.length);
        console.log('Users:', users.length);
        console.log('Todos:', todos.length);
    } catch (error) {
        console.error('Error en alguna petición:', error);
    }
}
```

### Promise.race - La más rápida gana

```javascript
async function obtenerRapido() {
    const urls = [
        'https://jsonplaceholder.typicode.com/posts/1',
        'https://jsonplaceholder.typicode.com/posts/2',
        'https://jsonplaceholder.typicode.com/posts/3'
    ];
    
    try {
        const resultado = await Promise.race(
            urls.map(url => fetch(url).then(r => r.json()))
        );
        
        console.log('Primera en responder:', resultado);
    } catch (error) {
        console.error('Error:', error);
    }
}
```

---

## AbortController

**Cancelar peticiones en curso**.

```javascript
const controller = new AbortController();

// Iniciar petición cancelable
fetch(url, { signal: controller.signal })
    .then(r => r.json())
    .then(data => console.log(data))
    .catch(error => {
        if (error.name === 'AbortError') {
            console.log('Petición cancelada');
        }
    });

// Cancelar después de 5 segundos
setTimeout(() => controller.abort(), 5000);
```

### Ejemplo: Búsqueda cancelable

```javascript
let controllerActual = null;

async function buscar(termino) {
    // Cancelar búsqueda anterior
    if (controllerActual) {
        controllerActual.abort();
    }
    
    // Nuevo controller
    controllerActual = new AbortController();
    
    try {
        const respuesta = await fetch(
            `https://api.ejemplo.com/search?q=${termino}`,
            { signal: controllerActual.signal }
        );
        
        const resultados = await respuesta.json();
        mostrarResultados(resultados);
        
    } catch (error) {
        if (error.name === 'AbortError') {
            console.log('Búsqueda cancelada');
        } else {
            console.error('Error:', error);
        }
    }
}
```

---

## Ejercicios prácticos

### Ejercicio 1: Búsqueda con debounce
Crea un buscador que solo haga peticiones después de que el usuario deje de escribir.

### Ejercicio 2: Scroll infinito con throttle
Implementa scroll infinito que cargue más datos al llegar al final.

### Ejercicio 3: Sistema de caché
Crea una app que cachee respuestas y muestre indicador de "desde caché".

### Ejercicio 4: Peticiones paralelas
Carga datos de múltiples APIs simultáneamente y muestra un indicador de progreso.

---

## Cheatsheet

```javascript
// ========== Debounce ==========
const debounce = (fn, delay) => {
    let timeout;
    return (...args) => {
        clearTimeout(timeout);
        timeout = setTimeout(() => fn(...args), delay);
    };
};

// ========== Throttle ==========
const throttle = (fn, limit) => {
    let inThrottle;
    return (...args) => {
        if (!inThrottle) {
            fn(...args);
            inThrottle = true;
            setTimeout(() => inThrottle = false, limit);
        }
    };
};

// ========== Retry ==========
async function retry(fn, tries = 3) {
    for (let i = 0; i < tries; i++) {
        try {
            return await fn();
        } catch (e) {
            if (i === tries - 1) throw e;
            await new Promise(r => setTimeout(r, 1000 * (i + 1)));
        }
    }
}

// ========== Promise.all ==========
const [a, b, c] = await Promise.all([promesa1, promesa2, promesa3]);

// ========== AbortController ==========
const controller = new AbortController();
fetch(url, { signal: controller.signal });
controller.abort(); // Cancelar
```

---

**Siguiente:** [Módulo 37 - Proyecto: Todo App con API](./37-proyecto-todo-app.md)

**Anterior:** [Módulo 35 - LocalStorage y SessionStorage](./35-localstorage-sessionstorage.md)

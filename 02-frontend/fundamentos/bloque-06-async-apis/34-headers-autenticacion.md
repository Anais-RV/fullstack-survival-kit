# Módulo 34: Headers y Autenticación

## Índice
1. [¿Qué son los headers HTTP?](#qué-son-los-headers-http)
2. [Headers comunes](#headers-comunes)
3. [Enviar headers personalizados](#enviar-headers-personalizados)
4. [API Keys](#api-keys)
5. [Bearer Tokens](#bearer-tokens)
6. [Basic Authentication](#basic-authentication)
7. [CORS](#cors)
8. [Ejercicios prácticos](#ejercicios-prácticos)
9. [Errores comunes](#errores-comunes)
10. [Buenas prácticas](#buenas-prácticas)
11. [Cheatsheet](#cheatsheet)

---

## ¿Qué son los headers HTTP?

**Los headers son metadatos que acompañan las peticiones y respuestas HTTP**.

### Analogía: Sobre de carta

```
Carta (body)           → Los datos que envías/recibes
Sobre (headers)        → Información sobre la carta (remitente, destino, urgente, etc.)
```

### Headers de petición vs respuesta

| Tipo | Quién los envía | Ejemplos |
|------|-----------------|----------|
| **Request headers** | Cliente (navegador) | `Authorization`, `Content-Type`, `User-Agent` |
| **Response headers** | Servidor | `Content-Type`, `Set-Cookie`, `Cache-Control` |

---

## Headers comunes

### Headers de petición

```javascript
'Content-Type': 'application/json'      // Tipo de datos que envías
'Authorization': 'Bearer token123'      // Autenticación
'Accept': 'application/json'            // Tipo de datos que aceptas
'User-Agent': 'Mozilla/5.0...'         // Identificación del navegador
'Accept-Language': 'es-ES,es'          // Idioma preferido
```

### Headers de respuesta

```javascript
'Content-Type': 'application/json'      // Tipo de datos que recibes
'Content-Length': '348'                 // Tamaño de la respuesta
'Cache-Control': 'max-age=3600'        // Control de caché
'Set-Cookie': 'session=abc123'         // Establecer cookies
```

### Ver headers en la consola

```javascript
const respuesta = await fetch('https://jsonplaceholder.typicode.com/posts/1');

// Ver todos los headers
console.log('Headers:');
respuesta.headers.forEach((valor, nombre) => {
    console.log(`${nombre}: ${valor}`);
});

// Ver un header específico
console.log('Content-Type:', respuesta.headers.get('content-type'));
console.log('Content-Length:', respuesta.headers.get('content-length'));
```

---

## Enviar headers personalizados

### Headers básicos

```javascript
await fetch(url, {
    headers: {
        'Content-Type': 'application/json',
        'Accept': 'application/json'
    }
});
```

### Múltiples headers

```javascript
const respuesta = await fetch('https://api.ejemplo.com/data', {
    headers: {
        'Content-Type': 'application/json',
        'Accept': 'application/json',
        'X-Custom-Header': 'valor-personalizado',
        'User-Agent': 'MiApp/1.0'
    }
});
```

### Headers dinámicos

```javascript
const token = 'mi-token-secreto';
const idioma = 'es';

const respuesta = await fetch(url, {
    headers: {
        'Authorization': `Bearer ${token}`,
        'Accept-Language': idioma,
        'Content-Type': 'application/json'
    }
});
```

---

## API Keys

**API Keys son tokens simples para identificar tu aplicación**.

### Dónde se envían

Hay 3 formas comunes:

#### 1. En el header

```javascript
const API_KEY = 'tu-api-key-aqui';

await fetch('https://api.ejemplo.com/data', {
    headers: {
        'X-API-Key': API_KEY
        // o también:
        // 'Api-Key': API_KEY
        // 'Authorization': `ApiKey ${API_KEY}`
    }
});
```

#### 2. En la URL (query parameter)

```javascript
const API_KEY = 'tu-api-key-aqui';

await fetch(`https://api.ejemplo.com/data?api_key=${API_KEY}`);
// o:
await fetch(`https://api.ejemplo.com/data?apikey=${API_KEY}`);
```

#### 3. En el body (raro)

```javascript
await fetch(url, {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({
        api_key: API_KEY,
        ...otrosDatos
    })
});
```

### Ejemplo: OpenWeather API

```javascript
const API_KEY = 'tu-api-key-de-openweather';
const ciudad = 'Madrid';

async function obtenerClima() {
    const url = `https://api.openweathermap.org/data/2.5/weather?q=${ciudad}&appid=${API_KEY}&units=metric&lang=es`;
    
    try {
        const respuesta = await fetch(url);
        
        if (!respuesta.ok) {
            if (respuesta.status === 401) {
                throw new Error('API Key inválida o no proporcionada');
            }
            throw new Error(`Error HTTP: ${respuesta.status}`);
        }
        
        const datos = await respuesta.json();
        console.log(`Temperatura en ${datos.name}: ${datos.main.temp}°C`);
        
    } catch (error) {
        console.error('Error:', error.message);
    }
}

obtenerClima();
```

### Ejemplo: TMDB (The Movie Database)

```javascript
const API_KEY = 'tu-api-key-de-tmdb';

async function buscarPelicula(nombre) {
    const url = `https://api.themoviedb.org/3/search/movie?api_key=${API_KEY}&query=${encodeURIComponent(nombre)}&language=es`;
    
    const respuesta = await fetch(url);
    const datos = await respuesta.json();
    
    console.log(`Encontradas ${datos.total_results} películas`);
    datos.results.forEach(pelicula => {
        console.log(`- ${pelicula.title} (${pelicula.release_date?.substring(0, 4)})`);
    });
}

buscarPelicula('matrix');
```

---

## Bearer Tokens

**Tokens JWT (JSON Web Tokens) para autenticación**.

### Formato

```
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
```

### Cómo usarlos

```javascript
const token = 'eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...';

await fetch('https://api.ejemplo.com/user/profile', {
    headers: {
        'Authorization': `Bearer ${token}`,
        'Content-Type': 'application/json'
    }
});
```

### Flujo completo: Login + peticiones autenticadas

```javascript
// 1. Login - Obtener token
async function login(email, password) {
    const respuesta = await fetch('https://api.ejemplo.com/login', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ email, password })
    });
    
    const datos = await respuesta.json();
    const token = datos.token;
    
    // Guardar token
    localStorage.setItem('token', token);
    
    return token;
}

// 2. Usar token en peticiones
async function obtenerPerfil() {
    const token = localStorage.getItem('token');
    
    if (!token) {
        throw new Error('No estás autenticado');
    }
    
    const respuesta = await fetch('https://api.ejemplo.com/user/profile', {
        headers: {
            'Authorization': `Bearer ${token}`
        }
    });
    
    if (respuesta.status === 401) {
        // Token inválido o expirado
        localStorage.removeItem('token');
        throw new Error('Sesión expirada. Inicia sesión de nuevo');
    }
    
    return respuesta.json();
}

// 3. Crear datos autenticados
async function crearPost(datos) {
    const token = localStorage.getItem('token');
    
    const respuesta = await fetch('https://api.ejemplo.com/posts', {
        method: 'POST',
        headers: {
            'Authorization': `Bearer ${token}`,
            'Content-Type': 'application/json'
        },
        body: JSON.stringify(datos)
    });
    
    return respuesta.json();
}

// Uso
await login('usuario@example.com', 'password123');
const perfil = await obtenerPerfil();
await crearPost({ title: 'Nuevo post', content: 'Contenido...' });
```

### Clase helper para auth

```javascript
class AuthService {
    constructor() {
        this.token = localStorage.getItem('token');
    }
    
    async login(email, password) {
        const respuesta = await fetch('https://api.ejemplo.com/login', {
            method: 'POST',
            headers: { 'Content-Type': 'application/json' },
            body: JSON.stringify({ email, password })
        });
        
        if (!respuesta.ok) {
            throw new Error('Credenciales inválidas');
        }
        
        const datos = await respuesta.json();
        this.token = datos.token;
        localStorage.setItem('token', this.token);
        
        return datos;
    }
    
    logout() {
        this.token = null;
        localStorage.removeItem('token');
    }
    
    isAuthenticated() {
        return this.token !== null;
    }
    
    async fetch(url, opciones = {}) {
        if (!this.token) {
            throw new Error('No autenticado');
        }
        
        // Añadir token a los headers
        opciones.headers = {
            ...opciones.headers,
            'Authorization': `Bearer ${this.token}`
        };
        
        const respuesta = await fetch(url, opciones);
        
        if (respuesta.status === 401) {
            this.logout();
            throw new Error('Sesión expirada');
        }
        
        return respuesta;
    }
}

// Uso
const auth = new AuthService();

await auth.login('user@example.com', 'pass123');
const respuesta = await auth.fetch('https://api.ejemplo.com/profile');
const perfil = await respuesta.json();
```

---

## Basic Authentication

**Autenticación básica con usuario y contraseña** (menos seguro, usar solo con HTTPS).

### Formato

```
Authorization: Basic base64(usuario:contraseña)
```

### Cómo usarlo

```javascript
const usuario = 'admin';
const password = 'secreto123';

// Codificar en Base64
const credenciales = btoa(`${usuario}:${password}`);

await fetch('https://api.ejemplo.com/data', {
    headers: {
        'Authorization': `Basic ${credenciales}`
    }
});
```

### Helper function

```javascript
function basicAuth(usuario, password) {
    const credenciales = btoa(`${usuario}:${password}`);
    return `Basic ${credenciales}`;
}

// Uso
await fetch(url, {
    headers: {
        'Authorization': basicAuth('admin', 'pass123')
    }
});
```

---

## CORS

**CORS (Cross-Origin Resource Sharing) controla qué orígenes pueden acceder a una API**.

### ¿Qué es CORS?

```
Tu web en:     https://mi-app.com
API en:        https://api.otra-app.com

⚠️ El navegador bloquea la petición por seguridad (orígenes diferentes)
✅ El servidor debe permitir explícitamente el acceso
```

### Error típico de CORS

```
Access to fetch at 'https://api.ejemplo.com' from origin 'http://localhost:3000' 
has been blocked by CORS policy: No 'Access-Control-Allow-Origin' header is present
```

### Headers relacionados con CORS

```javascript
// Petición preflight (OPTIONS)
'Access-Control-Request-Method': 'POST'
'Access-Control-Request-Headers': 'Content-Type'

// Respuesta del servidor
'Access-Control-Allow-Origin': '*'                    // o dominio específico
'Access-Control-Allow-Methods': 'GET, POST, PUT'
'Access-Control-Allow-Headers': 'Content-Type, Authorization'
'Access-Control-Allow-Credentials': 'true'
```

### Modo de petición

```javascript
// Petición con credenciales (cookies, auth headers)
await fetch(url, {
    credentials: 'include',    // Enviar cookies
    headers: {
        'Authorization': `Bearer ${token}`
    }
});

// Opciones de credentials
// 'omit'       - No enviar credenciales (default para cross-origin)
// 'same-origin' - Solo si mismo origen
// 'include'    - Siempre enviar credenciales
```

---

## Ejercicios prácticos

### Ejercicio 1: API con API Key
Crea una app que consuma OpenWeather o TMDB API usando tu API Key.

### Ejercicio 2: Sistema de login
Simula un sistema de login que guarde un token y lo use en peticiones.

### Ejercicio 3: Inspector de headers
Crea una herramienta que muestre todos los headers de una petición.

### Ejercicio 4: Auth con refresh
Implementa un sistema que refresque el token automáticamente cuando expire.

### Ejercicio 5: Rate limiting
Crea una app que detecte cuando alcanzas el límite de peticiones de una API.

---

## Errores comunes

### ❌ Olvidar "Bearer" en el token

```javascript
// Mal
'Authorization': token

// Bien
'Authorization': `Bearer ${token}`
```

### ❌ Exponer API Keys en el código

```javascript
// ❌ MAL - Nunca hagas esto en producción
const API_KEY = 'sk_live_12345abcde...';

// ✅ BIEN - Usa variables de entorno
const API_KEY = import.meta.env.VITE_API_KEY;
```

### ❌ No manejar tokens expirados

```javascript
// Mal - No verifica si el token expiró
const respuesta = await fetch(url, {
    headers: { 'Authorization': `Bearer ${token}` }
});

// Bien - Maneja expiración
if (respuesta.status === 401) {
    localStorage.removeItem('token');
    window.location.href = '/login';
}
```

---

## Buenas prácticas

### ✅ Centralizar configuración de auth

```javascript
const API_CONFIG = {
    baseURL: 'https://api.ejemplo.com',
    getHeaders() {
        const token = localStorage.getItem('token');
        return {
            'Content-Type': 'application/json',
            ...(token && { 'Authorization': `Bearer ${token}` })
        };
    }
};

async function request(endpoint, opciones = {}) {
    const url = `${API_CONFIG.baseURL}${endpoint}`;
    const headers = { ...API_CONFIG.getHeaders(), ...opciones.headers };
    
    return fetch(url, { ...opciones, headers });
}
```

### ✅ No guardar API Keys sensibles en el frontend

```javascript
// ❌ API Keys privadas (de servidor) NO deben estar en el frontend
const STRIPE_SECRET_KEY = 'sk_live_...';  // ¡NUNCA!

// ✅ Solo API Keys públicas
const STRIPE_PUBLIC_KEY = 'pk_live_...';  // OK, es pública
```

### ✅ Usar HTTPS siempre con autenticación

```javascript
// ❌ Nunca envíes credenciales por HTTP
fetch('http://api.ejemplo.com/login', ...)

// ✅ Siempre HTTPS
fetch('https://api.ejemplo.com/login', ...)
```

---

## Cheatsheet

```javascript
// ========== Headers básicos ==========
headers: {
    'Content-Type': 'application/json',
    'Accept': 'application/json'
}

// ========== API Key en header ==========
headers: {
    'X-API-Key': 'tu-api-key'
}

// ========== API Key en URL ==========
fetch(`${url}?api_key=${API_KEY}`)

// ========== Bearer Token ==========
headers: {
    'Authorization': `Bearer ${token}`
}

// ========== Basic Auth ==========
const creds = btoa(`${user}:${pass}`);
headers: {
    'Authorization': `Basic ${creds}`
}

// ========== Con credenciales ==========
fetch(url, {
    credentials: 'include',
    headers: { 'Authorization': `Bearer ${token}` }
})

// ========== Ver headers de respuesta ==========
const res = await fetch(url);
res.headers.forEach((val, key) => console.log(key, val));
console.log(res.headers.get('content-type'));

// ========== Manejar token expirado ==========
if (respuesta.status === 401) {
    localStorage.removeItem('token');
    // Redirigir a login
}
```

---

**Siguiente:** [Módulo 35 - LocalStorage y SessionStorage](./35-localstorage-sessionstorage.md)

**Anterior:** [Módulo 33 - Peticiones POST/PUT/DELETE](./33-peticiones-post-put-delete.md)

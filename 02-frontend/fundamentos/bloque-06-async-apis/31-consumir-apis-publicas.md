# Módulo 31: Consumir APIs Públicas

## Índice
1. [¿Qué es una API pública?](#qué-es-una-api-pública)
2. [APIs para practicar](#apis-para-practicar)
3. [JSONPlaceholder - API de prueba](#jsonplaceholder---api-de-prueba)
4. [PokeAPI - Datos de Pokémon](#pokeapi---datos-de-pokémon)
5. [OpenWeather - Clima](#openweather---clima)
6. [Rick and Morty API](#rick-and-morty-api)
7. [Ejercicios prácticos](#ejercicios-prácticos)
8. [Errores comunes](#errores-comunes)
9. [Buenas prácticas](#buenas-prácticas)
10. [Cheatsheet](#cheatsheet)

---

## ¿Qué es una API pública?

**Una API pública es un servicio web que ofrece datos gratuitamente para que practiques o construyas aplicaciones**.

### Características

✅ **Gratuitas** (la mayoría tienen límites de peticiones)  
✅ **No requieren autenticación** (o solo API key simple)  
✅ **Documentación pública**  
✅ **Perfectas para aprender**

### Requisito previo

**Antes de continuar, asegúrate de dominar:**
- [Módulo 30: Asincronía](../bloque-05-javascript-avanzado/30-asincronia.md) — Promises, async/await, Fetch API

Este módulo es 100% práctica. Ya debes conocer la teoría.

---

## APIs para practicar

### APIs sin autenticación

| API | Descripción | URL |
|-----|-------------|-----|
| **JSONPlaceholder** | Posts, usuarios, comentarios (datos fake) | https://jsonplaceholder.typicode.com |
| **PokeAPI** | Datos de Pokémon | https://pokeapi.co |
| **Rick and Morty** | Personajes de la serie | https://rickandmortyapi.com |
| **Dog CEO** | Imágenes de perros | https://dog.ceo/dog-api |
| **Cat Facts** | Datos sobre gatos | https://catfact.ninja |

### APIs con API key (gratis)

| API | Descripción | URL |
|-----|-------------|-----|
| **OpenWeather** | Clima en tiempo real | https://openweathermap.org/api |
| **TMDB** | Películas y series | https://www.themoviedb.org/documentation/api |
| **NewsAPI** | Noticias | https://newsapi.org |

---

## JSONPlaceholder - API de prueba

**La mejor API para empezar**. Simula un backend real con posts, usuarios, comentarios.

### Base URL
```
https://jsonplaceholder.typicode.com
```

### Obtener todos los posts

```javascript
async function obtenerPosts() {
    try {
        const respuesta = await fetch('https://jsonplaceholder.typicode.com/posts');
        const posts = await respuesta.json();
        console.log(posts);
        return posts;
    } catch (error) {
        console.error('Error:', error);
    }
}

obtenerPosts();
```

**Respuesta** (array con 100 posts):
```json
[
  {
    "userId": 1,
    "id": 1,
    "title": "sunt aut facere repellat",
    "body": "quia et suscipit..."
  },
  ...
]
```

### Obtener un post específico

```javascript
async function obtenerPost(id) {
    const respuesta = await fetch(`https://jsonplaceholder.typicode.com/posts/${id}`);
    const post = await respuesta.json();
    console.log(post);
}

obtenerPost(5);
```

**Respuesta**:
```json
{
  "userId": 1,
  "id": 5,
  "title": "nesciunt quas odio",
  "body": "repudiandae veniam quaerat..."
}
```

### Obtener usuarios

```javascript
async function obtenerUsuarios() {
    const respuesta = await fetch('https://jsonplaceholder.typicode.com/users');
    const usuarios = await respuesta.json();
    
    usuarios.forEach(usuario => {
        console.log(`${usuario.name} - ${usuario.email}`);
    });
}

obtenerUsuarios();
```

### Ejercicio: Mostrar posts en HTML

```html
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Posts desde API</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            max-width: 800px;
            margin: 40px auto;
            padding: 20px;
        }
        .post {
            background: #f5f5f5;
            padding: 20px;
            margin-bottom: 15px;
            border-radius: 8px;
        }
        .post h3 {
            margin-top: 0;
            color: #333;
        }
        .loading {
            text-align: center;
            font-size: 1.2em;
            color: #666;
        }
    </style>
</head>
<body>
    <h1>Posts desde JSONPlaceholder</h1>
    <div id="posts" class="loading">Cargando posts...</div>

    <script>
        async function cargarPosts() {
            const contenedor = document.getElementById('posts');
            
            try {
                const respuesta = await fetch('https://jsonplaceholder.typicode.com/posts?_limit=10');
                const posts = await respuesta.json();
                
                // Limpiar mensaje de carga
                contenedor.innerHTML = '';
                
                // Mostrar cada post
                posts.forEach(post => {
                    const postDiv = document.createElement('div');
                    postDiv.className = 'post';
                    postDiv.innerHTML = `
                        <h3>${post.title}</h3>
                        <p>${post.body}</p>
                        <small>Post ID: ${post.id} | User ID: ${post.userId}</small>
                    `;
                    contenedor.appendChild(postDiv);
                });
                
            } catch (error) {
                contenedor.innerHTML = `<p style="color: red;">Error al cargar los posts: ${error.message}</p>`;
            }
        }
        
        // Cargar posts al cargar la página
        cargarPosts();
    </script>
</body>
</html>
```

---

## PokeAPI - Datos de Pokémon

**API con datos de todos los Pokémon**.

### Base URL
```
https://pokeapi.co/api/v2
```

### Obtener información de un Pokémon

```javascript
async function obtenerPokemon(nombre) {
    try {
        const respuesta = await fetch(`https://pokeapi.co/api/v2/pokemon/${nombre}`);
        const pokemon = await respuesta.json();
        
        console.log('Nombre:', pokemon.name);
        console.log('Altura:', pokemon.height);
        console.log('Peso:', pokemon.weight);
        console.log('Tipos:', pokemon.types.map(t => t.type.name).join(', '));
        console.log('Imagen:', pokemon.sprites.front_default);
        
        return pokemon;
    } catch (error) {
        console.error('Pokémon no encontrado:', error);
    }
}

obtenerPokemon('pikachu');
obtenerPokemon('charizard');
obtenerPokemon('25'); // También acepta números
```

### Mostrar Pokémon en HTML

```html
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Pokédex</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            max-width: 600px;
            margin: 40px auto;
            padding: 20px;
            text-align: center;
        }
        input {
            padding: 10px;
            font-size: 16px;
            width: 300px;
            margin-right: 10px;
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
        #resultado {
            margin-top: 30px;
        }
        .pokemon-card {
            background: #f5f5f5;
            padding: 20px;
            border-radius: 12px;
            box-shadow: 0 2px 8px rgba(0,0,0,0.1);
        }
        .pokemon-card img {
            width: 200px;
            height: 200px;
        }
        .pokemon-card h2 {
            text-transform: capitalize;
            color: #333;
        }
        .tipos {
            display: flex;
            justify-content: center;
            gap: 10px;
            margin-top: 10px;
        }
        .tipo {
            background: #3498db;
            color: white;
            padding: 5px 15px;
            border-radius: 20px;
            text-transform: capitalize;
        }
        .error {
            color: red;
            padding: 20px;
        }
    </style>
</head>
<body>
    <h1>🎮 Pokédex</h1>
    
    <div>
        <input type="text" id="pokemonInput" placeholder="Nombre o número del Pokémon">
        <button onclick="buscarPokemon()">Buscar</button>
    </div>
    
    <div id="resultado"></div>

    <script>
        async function buscarPokemon() {
            const input = document.getElementById('pokemonInput');
            const resultado = document.getElementById('resultado');
            const nombre = input.value.toLowerCase().trim();
            
            if (!nombre) {
                resultado.innerHTML = '<p class="error">Por favor, escribe un nombre o número</p>';
                return;
            }
            
            resultado.innerHTML = '<p>Buscando...</p>';
            
            try {
                const respuesta = await fetch(`https://pokeapi.co/api/v2/pokemon/${nombre}`);
                
                if (!respuesta.ok) {
                    throw new Error('Pokémon no encontrado');
                }
                
                const pokemon = await respuesta.json();
                
                const tipos = pokemon.types
                    .map(t => `<span class="tipo">${t.type.name}</span>`)
                    .join('');
                
                resultado.innerHTML = `
                    <div class="pokemon-card">
                        <img src="${pokemon.sprites.front_default}" alt="${pokemon.name}">
                        <h2>${pokemon.name}</h2>
                        <p><strong>Altura:</strong> ${pokemon.height / 10} m</p>
                        <p><strong>Peso:</strong> ${pokemon.weight / 10} kg</p>
                        <p><strong>Tipos:</strong></p>
                        <div class="tipos">${tipos}</div>
                    </div>
                `;
            } catch (error) {
                resultado.innerHTML = `<p class="error">❌ ${error.message}</p>`;
            }
        }
        
        // Buscar con Enter
        document.getElementById('pokemonInput').addEventListener('keypress', (e) => {
            if (e.key === 'Enter') {
                buscarPokemon();
            }
        });
    </script>
</body>
</html>
```

---

## OpenWeather - Clima

**Requiere API key gratuita**.

### Obtener API Key

1. Ir a https://openweathermap.org/api
2. Crear cuenta gratis
3. Obtener API key

### Base URL
```
https://api.openweathermap.org/data/2.5/weather
```

### Obtener clima de una ciudad

```javascript
const API_KEY = 'TU_API_KEY_AQUI'; // ⚠️ Reemplaza con tu key real

async function obtenerClima(ciudad) {
    try {
        const url = `https://api.openweathermap.org/data/2.5/weather?q=${ciudad}&appid=${API_KEY}&units=metric&lang=es`;
        
        const respuesta = await fetch(url);
        const datos = await respuesta.json();
        
        console.log('Ciudad:', datos.name);
        console.log('Temperatura:', datos.main.temp, '°C');
        console.log('Sensación térmica:', datos.main.feels_like, '°C');
        console.log('Descripción:', datos.weather[0].description);
        console.log('Humedad:', datos.main.humidity, '%');
        console.log('Viento:', datos.wind.speed, 'km/h');
        
        return datos;
    } catch (error) {
        console.error('Error al obtener clima:', error);
    }
}

obtenerClima('Madrid');
obtenerClima('Barcelona');
```

**Respuesta** (simplificada):
```json
{
  "name": "Madrid",
  "main": {
    "temp": 18.5,
    "feels_like": 17.2,
    "humidity": 65
  },
  "weather": [
    {
      "description": "algo de nubes",
      "icon": "02d"
    }
  ],
  "wind": {
    "speed": 3.5
  }
}
```

### Mostrar clima en HTML

```html
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>App del Clima</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            max-width: 500px;
            margin: 40px auto;
            padding: 20px;
            text-align: center;
            background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
            min-height: 100vh;
        }
        h1 {
            color: white;
        }
        .busqueda {
            margin-bottom: 30px;
        }
        input {
            padding: 12px;
            font-size: 16px;
            width: 250px;
            border: none;
            border-radius: 25px;
            margin-right: 10px;
        }
        button {
            padding: 12px 25px;
            font-size: 16px;
            background: white;
            color: #667eea;
            border: none;
            cursor: pointer;
            border-radius: 25px;
            font-weight: bold;
        }
        button:hover {
            background: #f0f0f0;
        }
        .clima-card {
            background: white;
            padding: 30px;
            border-radius: 20px;
            box-shadow: 0 10px 30px rgba(0,0,0,0.3);
        }
        .temperatura {
            font-size: 60px;
            font-weight: bold;
            color: #667eea;
            margin: 10px 0;
        }
        .descripcion {
            font-size: 20px;
            color: #666;
            text-transform: capitalize;
            margin-bottom: 20px;
        }
        .detalles {
            display: flex;
            justify-content: space-around;
            margin-top: 20px;
            padding-top: 20px;
            border-top: 1px solid #eee;
        }
        .detalle {
            text-align: center;
        }
        .detalle-label {
            font-size: 12px;
            color: #999;
            text-transform: uppercase;
        }
        .detalle-valor {
            font-size: 18px;
            font-weight: bold;
            color: #333;
        }
        .error {
            background: #ff6b6b;
            color: white;
            padding: 15px;
            border-radius: 10px;
        }
    </style>
</head>
<body>
    <h1>🌤️ App del Clima</h1>
    
    <div class="busqueda">
        <input type="text" id="ciudadInput" placeholder="Nombre de la ciudad">
        <button onclick="buscarClima()">Buscar</button>
    </div>
    
    <div id="resultado"></div>

    <script>
        // ⚠️ IMPORTANTE: Reemplaza con tu API key real
        const API_KEY = 'TU_API_KEY_AQUI';
        
        async function buscarClima() {
            const input = document.getElementById('ciudadInput');
            const resultado = document.getElementById('resultado');
            const ciudad = input.value.trim();
            
            if (!ciudad) {
                resultado.innerHTML = '<p class="error">Por favor, escribe una ciudad</p>';
                return;
            }
            
            if (API_KEY === 'TU_API_KEY_AQUI') {
                resultado.innerHTML = '<p class="error">⚠️ Necesitas agregar tu API key de OpenWeather</p>';
                return;
            }
            
            resultado.innerHTML = '<div class="clima-card"><p>Cargando...</p></div>';
            
            try {
                const url = `https://api.openweathermap.org/data/2.5/weather?q=${ciudad}&appid=${API_KEY}&units=metric&lang=es`;
                const respuesta = await fetch(url);
                
                if (!respuesta.ok) {
                    throw new Error('Ciudad no encontrada');
                }
                
                const datos = await respuesta.json();
                
                resultado.innerHTML = `
                    <div class="clima-card">
                        <h2>${datos.name}, ${datos.sys.country}</h2>
                        <div class="temperatura">${Math.round(datos.main.temp)}°C</div>
                        <div class="descripcion">${datos.weather[0].description}</div>
                        
                        <div class="detalles">
                            <div class="detalle">
                                <div class="detalle-label">Sensación</div>
                                <div class="detalle-valor">${Math.round(datos.main.feels_like)}°C</div>
                            </div>
                            <div class="detalle">
                                <div class="detalle-label">Humedad</div>
                                <div class="detalle-valor">${datos.main.humidity}%</div>
                            </div>
                            <div class="detalle">
                                <div class="detalle-label">Viento</div>
                                <div class="detalle-valor">${Math.round(datos.wind.speed)} km/h</div>
                            </div>
                        </div>
                    </div>
                `;
            } catch (error) {
                resultado.innerHTML = `<p class="error">❌ ${error.message}</p>`;
            }
        }
        
        document.getElementById('ciudadInput').addEventListener('keypress', (e) => {
            if (e.key === 'Enter') buscarClima();
        });
    </script>
</body>
</html>
```

---

## Rick and Morty API

**API simple y divertida con personajes de la serie**.

### Base URL
```
https://rickandmortyapi.com/api
```

### Obtener personajes

```javascript
async function obtenerPersonajes() {
    const respuesta = await fetch('https://rickandmortyapi.com/api/character');
    const datos = await respuesta.json();
    
    console.log('Total personajes:', datos.info.count);
    console.log('Personajes en esta página:', datos.results.length);
    
    datos.results.forEach(personaje => {
        console.log(`${personaje.name} - ${personaje.species} (${personaje.status})`);
    });
}

obtenerPersonajes();
```

### Buscar por nombre

```javascript
async function buscarPersonaje(nombre) {
    const respuesta = await fetch(`https://rickandmortyapi.com/api/character/?name=${nombre}`);
    const datos = await respuesta.json();
    
    datos.results.forEach(personaje => {
        console.log('Nombre:', personaje.name);
        console.log('Especie:', personaje.species);
        console.log('Estado:', personaje.status);
        console.log('Género:', personaje.gender);
        console.log('Imagen:', personaje.image);
        console.log('---');
    });
}

buscarPersonaje('morty');
```

---

## Ejercicios prácticos

### Ejercicio 1: Lista de usuarios
Crea una página que muestre todos los usuarios de JSONPlaceholder en formato de tarjetas con nombre, email y sitio web.

### Ejercicio 2: Pokédex básico
Crea una app que muestre los primeros 20 Pokémon con sus imágenes y nombres.

### Ejercicio 3: Clima de múltiples ciudades
Crea una app que permita agregar varias ciudades y mostrar el clima de todas simultáneamente.

### Ejercicio 4: Galería de personajes
Usa Rick and Morty API para crear una galería con filtros por estado (alive, dead, unknown).

### Ejercicio 5: Buscador universal
Combina varias APIs en una sola app con pestañas o selectores.

---

## Errores comunes

### ❌ No verificar si la respuesta es exitosa

```javascript
// Mal
const datos = await (await fetch(url)).json(); // Puede fallar silenciosamente

// Bien
const respuesta = await fetch(url);
if (!respuesta.ok) {
    throw new Error(`Error ${respuesta.status}: ${respuesta.statusText}`);
}
const datos = await respuesta.json();
```

### ❌ No manejar errores de red

```javascript
// Mal
async function obtenerDatos() {
    const datos = await fetch(url);
    return datos.json();
}

// Bien
async function obtenerDatos() {
    try {
        const respuesta = await fetch(url);
        if (!respuesta.ok) throw new Error('Error en la petición');
        return await respuesta.json();
    } catch (error) {
        console.error('Error:', error);
        return null;
    }
}
```

### ❌ Hacer demasiadas peticiones simultáneas

```javascript
// Mal - Puede ser muy lento
for (let i = 1; i <= 100; i++) {
    await obtenerPokemon(i);
}

// Bien - Usa Promise.all con límite
const ids = Array.from({length: 20}, (_, i) => i + 1);
const promesas = ids.map(id => obtenerPokemon(id));
const resultados = await Promise.all(promesas);
```

---

## Buenas prácticas

### ✅ Centralizar las URLs

```javascript
const API = {
    POKEMON: 'https://pokeapi.co/api/v2',
    JSONPLACEHOLDER: 'https://jsonplaceholder.typicode.com',
    CLIMA: 'https://api.openweathermap.org/data/2.5'
};

async function obtenerPokemon(id) {
    const respuesta = await fetch(`${API.POKEMON}/pokemon/${id}`);
    return respuesta.json();
}
```

### ✅ Crear funciones reutilizables

```javascript
async function obtenerJSON(url) {
    try {
        const respuesta = await fetch(url);
        if (!respuesta.ok) {
            throw new Error(`Error ${respuesta.status}: ${respuesta.statusText}`);
        }
        return await respuesta.json();
    } catch (error) {
        console.error('Error al obtener datos:', error);
        throw error;
    }
}

// Uso
const posts = await obtenerJSON('https://jsonplaceholder.typicode.com/posts');
const pokemon = await obtenerJSON('https://pokeapi.co/api/v2/pokemon/1');
```

### ✅ Mostrar estados de carga

```javascript
async function cargarDatos() {
    const contenedor = document.getElementById('contenedor');
    
    // Estado: Cargando
    contenedor.innerHTML = '<p>Cargando...</p>';
    
    try {
        const datos = await fetch(url).then(r => r.json());
        
        // Estado: Éxito
        contenedor.innerHTML = generarHTML(datos);
    } catch (error) {
        // Estado: Error
        contenedor.innerHTML = `<p class="error">Error: ${error.message}</p>`;
    }
}
```

---

## Cheatsheet

```javascript
// ========== JSONPlaceholder ==========
// Posts
fetch('https://jsonplaceholder.typicode.com/posts')
fetch('https://jsonplaceholder.typicode.com/posts/1')

// Usuarios
fetch('https://jsonplaceholder.typicode.com/users')

// Comentarios
fetch('https://jsonplaceholder.typicode.com/comments?postId=1')

// ========== PokeAPI ==========
// Pokémon por nombre o ID
fetch('https://pokeapi.co/api/v2/pokemon/pikachu')
fetch('https://pokeapi.co/api/v2/pokemon/25')

// Lista de Pokémon
fetch('https://pokeapi.co/api/v2/pokemon?limit=20')

// ========== OpenWeather ==========
// Clima por ciudad (requiere API key)
const API_KEY = 'tu_key';
fetch(`https://api.openweathermap.org/data/2.5/weather?q=Madrid&appid=${API_KEY}&units=metric&lang=es`)

// ========== Rick and Morty ==========
// Personajes
fetch('https://rickandmortyapi.com/api/character')
fetch('https://rickandmortyapi.com/api/character/1')

// Buscar
fetch('https://rickandmortyapi.com/api/character/?name=rick')

// ========== Patrón básico ==========
async function obtenerDatos(url) {
    try {
        const respuesta = await fetch(url);
        if (!respuesta.ok) throw new Error('Error en la petición');
        return await respuesta.json();
    } catch (error) {
        console.error('Error:', error);
        return null;
    }
}
```

---

## Recursos adicionales

- 📚 [JSONPlaceholder Docs](https://jsonplaceholder.typicode.com/)
- 📚 [PokeAPI Docs](https://pokeapi.co/docs/v2)
- 📚 [OpenWeather API](https://openweathermap.org/api)
- 📚 [Rick and Morty API](https://rickandmortyapi.com/documentation)
- 📚 [Public APIs - Lista completa](https://github.com/public-apis/public-apis)

---

**Siguiente:** [Módulo 32 - Manejo de respuestas y errores](./32-manejo-respuestas-errores.md)

**Anterior:** [Módulo 30 - Asincronía en JavaScript](../bloque-05-javascript-avanzado/30-asincronia.md)

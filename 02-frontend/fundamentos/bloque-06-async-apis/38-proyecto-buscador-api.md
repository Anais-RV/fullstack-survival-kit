# Módulo 38: Proyecto - Buscador con API

## Proyecto: Buscador de Pokémon con PokeAPI

Aplicación completa que integra:
- ✅ Consumo de API real (PokeAPI)
- ✅ Búsqueda con debounce
- ✅ Estados de carga y errores
- ✅ Diseño responsive
- ✅ Favoritos con localStorage

---

## Código completo

```html
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Pokédex</title>
    <style>
        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
        }
        
        body {
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            background: linear-gradient(135deg, #ff6b6b 0%, #ffd93d 100%);
            min-height: 100vh;
            padding: 20px;
        }
        
        .container {
            max-width: 1200px;
            margin: 0 auto;
        }
        
        header {
            text-align: center;
            color: white;
            margin-bottom: 30px;
        }
        
        h1 {
            font-size: 3em;
            text-shadow: 2px 2px 4px rgba(0,0,0,0.2);
        }
        
        .search-box {
            background: white;
            padding: 20px;
            border-radius: 15px;
            box-shadow: 0 4px 20px rgba(0,0,0,0.1);
            margin-bottom: 30px;
        }
        
        .search-input {
            width: 100%;
            padding: 15px;
            font-size: 18px;
            border: 2px solid #e0e0e0;
            border-radius: 10px;
        }
        
        .search-input:focus {
            outline: none;
            border-color: #ff6b6b;
        }
        
        .tabs {
            display: flex;
            gap: 10px;
            margin-bottom: 20px;
            justify-content: center;
        }
        
        .tabs button {
            padding: 12px 24px;
            border: none;
            background: rgba(255,255,255,0.3);
            color: white;
            border-radius: 25px;
            cursor: pointer;
            font-size: 16px;
            transition: all 0.3s;
        }
        
        .tabs button.active {
            background: white;
            color: #ff6b6b;
        }
        
        .loading {
            text-align: center;
            padding: 60px;
            background: white;
            border-radius: 15px;
            font-size: 1.5em;
            color: #ff6b6b;
        }
        
        .error {
            background: white;
            padding: 40px;
            border-radius: 15px;
            text-align: center;
            color: #ff6b6b;
        }
        
        .pokemon-grid {
            display: grid;
            grid-template-columns: repeat(auto-fill, minmax(250px, 1fr));
            gap: 20px;
        }
        
        .pokemon-card {
            background: white;
            border-radius: 15px;
            padding: 20px;
            text-align: center;
            box-shadow: 0 4px 15px rgba(0,0,0,0.1);
            transition: all 0.3s;
            cursor: pointer;
            position: relative;
        }
        
        .pokemon-card:hover {
            transform: translateY(-5px);
            box-shadow: 0 8px 25px rgba(0,0,0,0.15);
        }
        
        .pokemon-card img {
            width: 150px;
            height: 150px;
        }
        
        .pokemon-card h3 {
            font-size: 1.5em;
            color: #333;
            margin: 10px 0;
            text-transform: capitalize;
        }
        
        .pokemon-id {
            color: #999;
            font-size: 0.9em;
        }
        
        .pokemon-types {
            display: flex;
            gap: 8px;
            justify-content: center;
            margin-top: 10px;
        }
        
        .type-badge {
            padding: 5px 12px;
            border-radius: 15px;
            color: white;
            font-size: 0.85em;
            text-transform: capitalize;
        }
        
        .type-normal { background: #A8A878; }
        .type-fire { background: #F08030; }
        .type-water { background: #6890F0; }
        .type-electric { background: #F8D030; }
        .type-grass { background: #78C850; }
        .type-ice { background: #98D8D8; }
        .type-fighting { background: #C03028; }
        .type-poison { background: #A040A0; }
        .type-ground { background: #E0C068; }
        .type-flying { background: #A890F0; }
        .type-psychic { background: #F85888; }
        .type-bug { background: #A8B820; }
        .type-rock { background: #B8A038; }
        .type-ghost { background: #705898; }
        .type-dragon { background: #7038F8; }
        .type-dark { background: #705848; }
        .type-steel { background: #B8B8D0; }
        .type-fairy { background: #EE99AC; }
        
        .fav-btn {
            position: absolute;
            top: 10px;
            right: 10px;
            background: none;
            border: none;
            font-size: 1.5em;
            cursor: pointer;
            transition: transform 0.3s;
        }
        
        .fav-btn:hover {
            transform: scale(1.2);
        }
        
        .empty {
            background: white;
            padding: 60px;
            border-radius: 15px;
            text-align: center;
            color: #999;
            font-size: 1.2em;
        }
        
        .load-more {
            text-align: center;
            margin-top: 30px;
        }
        
        .load-more button {
            padding: 15px 30px;
            background: white;
            border: none;
            border-radius: 25px;
            font-size: 16px;
            cursor: pointer;
            transition: all 0.3s;
        }
        
        .load-more button:hover {
            transform: scale(1.05);
        }
    </style>
</head>
<body>
    <div class="container">
        <header>
            <h1>🎮 Pokédex</h1>
        </header>
        
        <div class="search-box">
            <input 
                type="text" 
                class="search-input" 
                id="searchInput"
                placeholder="Buscar Pokémon por nombre o número..."
            >
        </div>
        
        <div class="tabs">
            <button class="active" data-tab="explore">Explorar</button>
            <button data-tab="favorites">Favoritos (<span id="favCount">0</span>)</button>
        </div>
        
        <div id="content"></div>
        <div class="load-more" id="loadMore" style="display: none;">
            <button onclick="cargarMas()">Cargar más</button>
        </div>
    </div>

    <script>
        const API_URL = 'https://pokeapi.co/api/v2';
        let offset = 0;
        const limit = 20;
        let todosLosPokemon = [];
        let tabActual = 'explore';
        let timeoutBusqueda = null;
        
        const searchInput = document.getElementById('searchInput');
        const content = document.getElementById('content');
        const loadMore = document.getElementById('loadMore');
        const favCount = document.getElementById('favCount');
        
        // ========== Favoritos ==========
        function obtenerFavoritos() {
            const favs = localStorage.getItem('pokemonFavs');
            return favs ? JSON.parse(favs) : [];
        }
        
        function guardarFavoritos(favoritos) {
            localStorage.setItem('pokemonFavs', JSON.stringify(favoritos));
            actualizarContadorFavs();
        }
        
        function esFavorito(id) {
            return obtenerFavoritos().some(p => p.id === id);
        }
        
        function toggleFavorito(pokemon) {
            let favoritos = obtenerFavoritos();
            const index = favoritos.findIndex(p => p.id === pokemon.id);
            
            if (index >= 0) {
                favoritos.splice(index, 1);
            } else {
                favoritos.push(pokemon);
            }
            
            guardarFavoritos(favoritos);
            
            if (tabActual === 'favorites') {
                mostrarFavoritos();
            }
        }
        
        function actualizarContadorFavs() {
            favCount.textContent = obtenerFavoritos().length;
        }
        
        // ========== Cargar Pokémon ==========
        async function cargarPokemon() {
            mostrarCargando();
            loadMore.style.display = 'none';
            
            try {
                const respuesta = await fetch(`${API_URL}/pokemon?offset=${offset}&limit=${limit}`);
                const datos = await respuesta.json();
                
                const pokemonPromesas = datos.results.map(p => 
                    fetch(p.url).then(r => r.json())
                );
                
                const pokemon = await Promise.all(pokemonPromesas);
                todosLosPokemon = [...todosLosPokemon, ...pokemon];
                
                mostrarPokemon(todosLosPokemon);
                loadMore.style.display = 'block';
                
            } catch (error) {
                mostrarError('Error al cargar Pokémon');
            }
        }
        
        async function cargarMas() {
            offset += limit;
            await cargarPokemon();
        }
        
        // ========== Buscar ==========
        async function buscarPokemon(termino) {
            termino = termino.toLowerCase().trim();
            
            if (!termino) {
                mostrarPokemon(todosLosPokemon);
                loadMore.style.display = 'block';
                return;
            }
            
            mostrarCargando();
            loadMore.style.display = 'none';
            
            try {
                // Buscar por nombre exacto o número
                const respuesta = await fetch(`${API_URL}/pokemon/${termino}`);
                
                if (respuesta.ok) {
                    const pokemon = await respuesta.json();
                    mostrarPokemon([pokemon]);
                } else {
                    // Buscar en los pokémon cargados
                    const resultados = todosLosPokemon.filter(p => 
                        p.name.includes(termino) || 
                        p.id.toString() === termino
                    );
                    
                    if (resultados.length > 0) {
                        mostrarPokemon(resultados);
                    } else {
                        content.innerHTML = '<div class="empty">No se encontró ningún Pokémon</div>';
                    }
                }
            } catch (error) {
                const resultados = todosLosPokemon.filter(p => p.name.includes(termino));
                if (resultados.length > 0) {
                    mostrarPokemon(resultados);
                } else {
                    content.innerHTML = '<div class="empty">No se encontró ningún Pokémon</div>';
                }
            }
        }
        
        // ========== Renderizar ==========
        function mostrarPokemon(listaPokemon) {
            if (listaPokemon.length === 0) {
                content.innerHTML = '<div class="empty">No hay Pokémon para mostrar</div>';
                return;
            }
            
            const html = listaPokemon.map(p => {
                const tipos = p.types.map(t => 
                    `<span class="type-badge type-${t.type.name}">${t.type.name}</span>`
                ).join('');
                
                const isFav = esFavorito(p.id);
                
                return `
                    <div class="pokemon-card">
                        <button class="fav-btn" onclick='toggleFavorito(${JSON.stringify(p)})'>
                            ${isFav ? '❤️' : '🤍'}
                        </button>
                        <img src="${p.sprites.front_default}" alt="${p.name}">
                        <div class="pokemon-id">#${p.id.toString().padStart(3, '0')}</div>
                        <h3>${p.name}</h3>
                        <div class="pokemon-types">${tipos}</div>
                    </div>
                `;
            }).join('');
            
            content.innerHTML = `<div class="pokemon-grid">${html}</div>`;
        }
        
        function mostrarFavoritos() {
            const favoritos = obtenerFavoritos();
            loadMore.style.display = 'none';
            
            if (favoritos.length === 0) {
                content.innerHTML = '<div class="empty">No tienes Pokémon favoritos todavía</div>';
            } else {
                mostrarPokemon(favoritos);
            }
        }
        
        function mostrarCargando() {
            content.innerHTML = '<div class="loading">Cargando Pokémon...</div>';
        }
        
        function mostrarError(mensaje) {
            content.innerHTML = `<div class="error">❌ ${mensaje}</div>`;
        }
        
        // ========== Event Listeners ==========
        searchInput.addEventListener('input', (e) => {
            clearTimeout(timeoutBusqueda);
            
            timeoutBusqueda = setTimeout(() => {
                if (tabActual === 'explore') {
                    buscarPokemon(e.target.value);
                }
            }, 500);
        });
        
        // Tabs
        document.querySelectorAll('.tabs button').forEach(btn => {
            btn.addEventListener('click', () => {
                document.querySelector('.tabs button.active').classList.remove('active');
                btn.classList.add('active');
                tabActual = btn.dataset.tab;
                searchInput.value = '';
                
                if (tabActual === 'favorites') {
                    mostrarFavoritos();
                } else {
                    mostrarPokemon(todosLosPokemon);
                    loadMore.style.display = 'block';
                }
            });
        });
        
        // ========== Inicializar ==========
        actualizarContadorFavs();
        cargarPokemon();
    </script>
</body>
</html>
```

---

## Funcionalidades implementadas

### ✅ Consumo de API
- GET lista de Pokémon
- GET detalle de cada Pokémon
- Búsqueda por nombre o número
- Carga paginada (20 por vez)

### ✅ Búsqueda inteligente
- Debounce de 500ms
- Busca por nombre parcial o número exacto
- Fallback a búsqueda local

### ✅ Favoritos con localStorage
- Agregar/quitar favoritos
- Contador actualizado
- Tab dedicada a favoritos
- Persistencia entre sesiones

### ✅ UI/UX
- Diseño responsive con grid
- Estados de carga
- Manejo de errores
- Botón "Cargar más"
- Badges de tipos con colores

---

## Mejoras opcionales

1. **Modal con detalles completos**
2. **Filtros por tipo**
3. **Comparador de Pokémon**
4. **Generaciones**
5. **Modo offline con cache**

---

## 🎉 ¡Felicidades!

Has completado el **Bloque 6: APIs y práctica asíncrona**.

Ahora dominas:
- ✅ Consumir APIs públicas
- ✅ Manejo robusto de errores
- ✅ Peticiones POST/PUT/DELETE
- ✅ Autenticación con tokens
- ✅ LocalStorage y SessionStorage
- ✅ Patrones async avanzados
- ✅ Proyectos completos

**Siguiente:** Continúa con [React](../../react/README.md) o explora [Backend](../../../03-backend/README.md)

**Anterior:** [Módulo 37 - Proyecto: Todo App](./37-proyecto-todo-app.md)

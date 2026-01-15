# Módulo 37: Proyecto - Todo App con API

## Proyecto: Aplicación de Tareas con JSONPlaceholder y LocalStorage

Este proyecto integra todo lo aprendido en el bloque 6:
- ✅ Consumir API (GET, POST, PUT, DELETE)
- ✅ Manejo de estados de carga y errores
- ✅ LocalStorage para persistencia
- ✅ Interfaz responsive y funcional

---

## Código completo

```html
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Todo App</title>
    <style>
        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
        }
        
        body {
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
            min-height: 100vh;
            padding: 20px;
        }
        
        .container {
            max-width: 600px;
            margin: 0 auto;
        }
        
        header {
            text-align: center;
            color: white;
            margin-bottom: 30px;
        }
        
        h1 {
            font-size: 2.5em;
            margin-bottom: 10px;
        }
        
        .stats {
            font-size: 0.9em;
            opacity: 0.9;
        }
        
        .add-form {
            background: white;
            padding: 20px;
            border-radius: 12px;
            box-shadow: 0 4px 20px rgba(0,0,0,0.1);
            margin-bottom: 20px;
        }
        
        .input-group {
            display: flex;
            gap: 10px;
        }
        
        input[type="text"] {
            flex: 1;
            padding: 12px;
            border: 2px solid #e0e0e0;
            border-radius: 8px;
            font-size: 16px;
        }
        
        input[type="text"]:focus {
            outline: none;
            border-color: #667eea;
        }
        
        .btn {
            padding: 12px 24px;
            border: none;
            border-radius: 8px;
            font-size: 16px;
            cursor: pointer;
            transition: all 0.3s;
        }
        
        .btn-primary {
            background: #667eea;
            color: white;
        }
        
        .btn-primary:hover {
            background: #5568d3;
        }
        
        .btn-primary:disabled {
            background: #ccc;
            cursor: not-allowed;
        }
        
        .btn-delete {
            background: #ff6b6b;
            color: white;
            padding: 6px 12px;
            font-size: 14px;
        }
        
        .btn-delete:hover {
            background: #ee5a52;
        }
        
        .loading {
            text-align: center;
            padding: 40px;
            background: white;
            border-radius: 12px;
            color: #667eea;
        }
        
        .error {
            background: #ff6b6b;
            color: white;
            padding: 15px;
            border-radius: 8px;
            margin-bottom: 20px;
        }
        
        .todos {
            background: white;
            border-radius: 12px;
            padding: 20px;
            box-shadow: 0 4px 20px rgba(0,0,0,0.1);
        }
        
        .todo {
            padding: 15px;
            border-bottom: 1px solid #e0e0e0;
            display: flex;
            align-items: center;
            gap: 15px;
            transition: all 0.3s;
        }
        
        .todo:last-child {
            border-bottom: none;
        }
        
        .todo:hover {
            background: #f5f5f5;
        }
        
        .todo.completed {
            opacity: 0.6;
        }
        
        .todo input[type="checkbox"] {
            width: 20px;
            height: 20px;
            cursor: pointer;
        }
        
        .todo-text {
            flex: 1;
            font-size: 16px;
        }
        
        .todo.completed .todo-text {
            text-decoration: line-through;
        }
        
        .empty {
            text-align: center;
            padding: 40px;
            color: #999;
        }
        
        .filters {
            display: flex;
            gap: 10px;
            margin-bottom: 20px;
            justify-content: center;
        }
        
        .filters button {
            padding: 8px 16px;
            border: 2px solid white;
            background: transparent;
            color: white;
            border-radius: 20px;
            cursor: pointer;
            transition: all 0.3s;
        }
        
        .filters button.active {
            background: white;
            color: #667eea;
        }
    </style>
</head>
<body>
    <div class="container">
        <header>
            <h1>📝 Todo App</h1>
            <div class="stats">
                <span id="stats">0 tareas</span>
            </div>
        </header>
        
        <div class="add-form">
            <form id="formTodo">
                <div class="input-group">
                    <input 
                        type="text" 
                        id="inputTodo" 
                        placeholder="¿Qué necesitas hacer?"
                        required
                    >
                    <button type="submit" class="btn btn-primary">
                        Agregar
                    </button>
                </div>
            </form>
        </div>
        
        <div class="filters">
            <button data-filter="all" class="active">Todas</button>
            <button data-filter="active">Pendientes</button>
            <button data-filter="completed">Completadas</button>
        </div>
        
        <div id="mensaje"></div>
        <div id="listaTodos"></div>
    </div>

    <script>
        const API_URL = 'https://jsonplaceholder.typicode.com/todos';
        let todos = [];
        let filtroActual = 'all';
        
        // Elementos DOM
        const formTodo = document.getElementById('formTodo');
        const inputTodo = document.getElementById('inputTodo');
        const listaTodos = document.getElementById('listaTodos');
        const mensaje = document.getElementById('mensaje');
        const stats = document.getElementById('stats');
        
        // ========== Cargar datos ==========
        async function cargarTodos() {
            mostrarCargando();
            
            try {
                // Intentar cargar del localStorage primero
                const todosLocal = localStorage.getItem('todos');
                
                if (todosLocal) {
                    todos = JSON.parse(todosLocal);
                    renderizarTodos();
                } else {
                    // Si no hay datos locales, cargar de la API
                    const respuesta = await fetch(`${API_URL}?_limit=10&userId=1`);
                    
                    if (!respuesta.ok) {
                        throw new Error('Error al cargar tareas');
                    }
                    
                    todos = await respuesta.json();
                    guardarLocal();
                    renderizarTodos();
                }
            } catch (error) {
                mostrarError(error.message);
            }
        }
        
        // ========== Agregar tarea ==========
        async function agregarTodo(titulo) {
            const boton = formTodo.querySelector('button');
            boton.disabled = true;
            boton.textContent = 'Agregando...';
            
            try {
                const respuesta = await fetch(API_URL, {
                    method: 'POST',
                    headers: { 'Content-Type': 'application/json' },
                    body: JSON.stringify({
                        title: titulo,
                        completed: false,
                        userId: 1
                    })
                });
                
                if (!respuesta.ok) {
                    throw new Error('Error al agregar tarea');
                }
                
                const nuevoTodo = await respuesta.json();
                todos.unshift(nuevoTodo);
                guardarLocal();
                renderizarTodos();
                inputTodo.value = '';
                
            } catch (error) {
                mostrarError(error.message);
            } finally {
                boton.disabled = false;
                boton.textContent = 'Agregar';
            }
        }
        
        // ========== Cambiar estado ==========
        async function toggleTodo(id) {
            const todo = todos.find(t => t.id === id);
            if (!todo) return;
            
            try {
                const respuesta = await fetch(`${API_URL}/${id}`, {
                    method: 'PATCH',
                    headers: { 'Content-Type': 'application/json' },
                    body: JSON.stringify({
                        completed: !todo.completed
                    })
                });
                
                if (respuesta.ok) {
                    todo.completed = !todo.completed;
                    guardarLocal();
                    renderizarTodos();
                }
            } catch (error) {
                mostrarError('Error al actualizar tarea');
            }
        }
        
        // ========== Eliminar tarea ==========
        async function eliminarTodo(id) {
            if (!confirm('¿Eliminar esta tarea?')) return;
            
            try {
                const respuesta = await fetch(`${API_URL}/${id}`, {
                    method: 'DELETE'
                });
                
                if (respuesta.ok) {
                    todos = todos.filter(t => t.id !== id);
                    guardarLocal();
                    renderizarTodos();
                }
            } catch (error) {
                mostrarError('Error al eliminar tarea');
            }
        }
        
        // ========== Renderizar ==========
        function renderizarTodos() {
            const todosFiltrados = filtrarTodos();
            
            if (todosFiltrados.length === 0) {
                listaTodos.innerHTML = `
                    <div class="todos">
                        <div class="empty">
                            ${filtroActual === 'all' ? 'No hay tareas. ¡Agrega una!' : 'No hay tareas en este filtro'}
                        </div>
                    </div>
                `;
            } else {
                const html = todosFiltrados.map(todo => `
                    <div class="todo ${todo.completed ? 'completed' : ''}">
                        <input 
                            type="checkbox" 
                            ${todo.completed ? 'checked' : ''}
                            onchange="toggleTodo(${todo.id})"
                        >
                        <span class="todo-text">${todo.title}</span>
                        <button 
                            class="btn btn-delete" 
                            onclick="eliminarTodo(${todo.id})"
                        >
                            Eliminar
                        </button>
                    </div>
                `).join('');
                
                listaTodos.innerHTML = `<div class="todos">${html}</div>`;
            }
            
            actualizarStats();
        }
        
        // ========== Filtrar tareas ==========
        function filtrarTodos() {
            switch (filtroActual) {
                case 'active':
                    return todos.filter(t => !t.completed);
                case 'completed':
                    return todos.filter(t => t.completed);
                default:
                    return todos;
            }
        }
        
        // ========== Estadísticas ==========
        function actualizarStats() {
            const total = todos.length;
            const pendientes = todos.filter(t => !t.completed).length;
            const completadas = todos.filter(t => t.completed).length;
            
            stats.textContent = `${total} tareas | ${pendientes} pendientes | ${completadas} completadas`;
        }
        
        // ========== LocalStorage ==========
        function guardarLocal() {
            localStorage.setItem('todos', JSON.stringify(todos));
        }
        
        // ========== UI States ==========
        function mostrarCargando() {
            listaTodos.innerHTML = '<div class="loading">Cargando tareas...</div>';
        }
        
        function mostrarError(mensajeError) {
            mensaje.innerHTML = `<div class="error">❌ ${mensajeError}</div>`;
            setTimeout(() => mensaje.innerHTML = '', 5000);
        }
        
        // ========== Event Listeners ==========
        formTodo.addEventListener('submit', (e) => {
            e.preventDefault();
            const titulo = inputTodo.value.trim();
            if (titulo) {
                agregarTodo(titulo);
            }
        });
        
        // Filtros
        document.querySelectorAll('.filters button').forEach(btn => {
            btn.addEventListener('click', () => {
                document.querySelector('.filters button.active').classList.remove('active');
                btn.classList.add('active');
                filtroActual = btn.dataset.filter;
                renderizarTodos();
            });
        });
        
        // ========== Inicializar ==========
        cargarTodos();
    </script>
</body>
</html>
```

---

## Funcionalidades implementadas

### ✅ CRUD completo
- GET: Cargar tareas
- POST: Crear nueva tarea
- PATCH: Cambiar estado
- DELETE: Eliminar tarea

### ✅ Persistencia
- LocalStorage guarda todas las tareas
- Carga desde localStorage primero
- Fallback a API si no hay datos locales

### ✅ Filtros
- Ver todas las tareas
- Solo pendientes
- Solo completadas

### ✅ Estados de UI
- Loading al cargar
- Botón deshabilitado al agregar
- Mensajes de error
- Estadísticas en tiempo real

### ✅ Manejo de errores
- Try/catch en todas las peticiones
- Mensajes amigables al usuario
- Timeout automático de errores

---

## Mejoras opcionales

1. **Editar tareas inline**
2. **Arrastrar y reordenar**
3. **Fechas de vencimiento**
4. **Categorías o etiquetas**
5. **Modo offline completo**

---

**Siguiente:** [Módulo 38 - Proyecto: Buscador con API](./38-proyecto-buscador-api.md)

**Anterior:** [Módulo 36 - Patrones async avanzados](./36-patrones-async-avanzados.md)

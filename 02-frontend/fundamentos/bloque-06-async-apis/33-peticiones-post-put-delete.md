# Módulo 33: Peticiones POST, PUT y DELETE

## Índice
1. [Métodos HTTP](#métodos-http)
2. [POST - Crear recursos](#post---crear-recursos)
3. [PUT - Actualizar recursos](#put---actualizar-recursos)
4. [DELETE - Eliminar recursos](#delete---eliminar-recursos)
5. [PATCH - Actualización parcial](#patch---actualización-parcial)
6. [Enviar JSON](#enviar-json)
7. [Enviar FormData](#enviar-formdata)
8. [Ejercicios prácticos](#ejercicios-prácticos)
9. [Errores comunes](#errores-comunes)
10. [Buenas prácticas](#buenas-prácticas)
11. [Cheatsheet](#cheatsheet)

---

## Métodos HTTP

**Los métodos HTTP indican qué operación quieres realizar**.

### CRUD - Operaciones básicas

| Operación | Método HTTP | Descripción |
|-----------|-------------|-------------|
| **Create** | POST | Crear un nuevo recurso |
| **Read** | GET | Obtener un recurso |
| **Update** | PUT / PATCH | Actualizar un recurso |
| **Delete** | DELETE | Eliminar un recurso |

### Características

| Método | Idempotente | Seguro | Tiene body |
|--------|-------------|--------|------------|
| **GET** | ✅ Sí | ✅ Sí | ❌ No |
| **POST** | ❌ No | ❌ No | ✅ Sí |
| **PUT** | ✅ Sí | ❌ No | ✅ Sí |
| **PATCH** | ❌ No | ❌ No | ✅ Sí |
| **DELETE** | ✅ Sí | ❌ No | ❌ No |

**Idempotente** = Hacer la misma petición varias veces tiene el mismo efecto que hacerla una vez.

---

## POST - Crear recursos

**POST se usa para crear nuevos recursos**.

### Sintaxis básica

```javascript
const opciones = {
    method: 'POST',
    headers: {
        'Content-Type': 'application/json'
    },
    body: JSON.stringify(datos)
};

const respuesta = await fetch(url, opciones);
```

### Crear un post

```javascript
async function crearPost() {
    const nuevoPost = {
        title: 'Mi primer post',
        body: 'Este es el contenido del post',
        userId: 1
    };
    
    try {
        const respuesta = await fetch('https://jsonplaceholder.typicode.com/posts', {
            method: 'POST',
            headers: {
                'Content-Type': 'application/json'
            },
            body: JSON.stringify(nuevoPost)
        });
        
        if (!respuesta.ok) {
            throw new Error(`Error HTTP: ${respuesta.status}`);
        }
        
        const postCreado = await respuesta.json();
        console.log('Post creado:', postCreado);
        return postCreado;
        
    } catch (error) {
        console.error('Error al crear post:', error);
    }
}

crearPost();
```

**Respuesta**:
```json
{
  "title": "Mi primer post",
  "body": "Este es el contenido del post",
  "userId": 1,
  "id": 101
}
```

### Crear un usuario

```javascript
async function crearUsuario() {
    const nuevoUsuario = {
        name: 'Ana García',
        email: 'ana@example.com',
        username: 'anagarcia'
    };
    
    const respuesta = await fetch('https://jsonplaceholder.typicode.com/users', {
        method: 'POST',
        headers: {
            'Content-Type': 'application/json'
        },
        body: JSON.stringify(nuevoUsuario)
    });
    
    const usuarioCreado = await respuesta.json();
    console.log('Usuario creado:', usuarioCreado);
}

crearUsuario();
```

### Formulario que envía POST

```html
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Crear Post</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            max-width: 600px;
            margin: 40px auto;
            padding: 20px;
        }
        .form-group {
            margin-bottom: 15px;
        }
        label {
            display: block;
            margin-bottom: 5px;
            font-weight: bold;
        }
        input, textarea {
            width: 100%;
            padding: 8px;
            border: 1px solid #ddd;
            border-radius: 4px;
            box-sizing: border-box;
        }
        textarea {
            min-height: 100px;
        }
        button {
            padding: 10px 20px;
            background: #3498db;
            color: white;
            border: none;
            border-radius: 4px;
            cursor: pointer;
            font-size: 16px;
        }
        button:hover {
            background: #2980b9;
        }
        button:disabled {
            background: #95a5a6;
            cursor: not-started;
        }
        .success {
            background: #d4edda;
            color: #155724;
            padding: 15px;
            border-radius: 4px;
            margin-top: 20px;
        }
        .error {
            background: #f8d7da;
            color: #721c24;
            padding: 15px;
            border-radius: 4px;
            margin-top: 20px;
        }
    </style>
</head>
<body>
    <h1>Crear Nuevo Post</h1>
    
    <form id="formPost">
        <div class="form-group">
            <label for="title">Título:</label>
            <input type="text" id="title" required>
        </div>
        
        <div class="form-group">
            <label for="body">Contenido:</label>
            <textarea id="body" required></textarea>
        </div>
        
        <button type="submit">Crear Post</button>
    </form>
    
    <div id="resultado"></div>

    <script>
        document.getElementById('formPost').addEventListener('submit', async (e) => {
            e.preventDefault();
            
            const boton = e.target.querySelector('button');
            const resultado = document.getElementById('resultado');
            
            // Obtener datos del formulario
            const datos = {
                title: document.getElementById('title').value,
                body: document.getElementById('body').value,
                userId: 1
            };
            
            // Deshabilitar botón
            boton.disabled = true;
            boton.textContent = 'Creando...';
            resultado.innerHTML = '';
            
            try {
                const respuesta = await fetch('https://jsonplaceholder.typicode.com/posts', {
                    method: 'POST',
                    headers: {
                        'Content-Type': 'application/json'
                    },
                    body: JSON.stringify(datos)
                });
                
                if (!respuesta.ok) {
                    throw new Error(`Error HTTP: ${respuesta.status}`);
                }
                
                const postCreado = await respuesta.json();
                
                resultado.innerHTML = `
                    <div class="success">
                        <h3>✅ Post creado exitosamente</h3>
                        <p><strong>ID:</strong> ${postCreado.id}</p>
                        <p><strong>Título:</strong> ${postCreado.title}</p>
                        <p><strong>Contenido:</strong> ${postCreado.body}</p>
                    </div>
                `;
                
                // Limpiar formulario
                e.target.reset();
                
            } catch (error) {
                resultado.innerHTML = `
                    <div class="error">
                        <h3>❌ Error al crear post</h3>
                        <p>${error.message}</p>
                    </div>
                `;
            } finally {
                boton.disabled = false;
                boton.textContent = 'Crear Post';
            }
        });
    </script>
</body>
</html>
```

---

## PUT - Actualizar recursos

**PUT reemplaza completamente un recurso existente**.

### Actualizar un post completo

```javascript
async function actualizarPost(id) {
    const postActualizado = {
        id: id,
        title: 'Título actualizado',
        body: 'Contenido completamente nuevo',
        userId: 1
    };
    
    try {
        const respuesta = await fetch(`https://jsonplaceholder.typicode.com/posts/${id}`, {
            method: 'PUT',
            headers: {
                'Content-Type': 'application/json'
            },
            body: JSON.stringify(postActualizado)
        });
        
        if (!respuesta.ok) {
            throw new Error(`Error HTTP: ${respuesta.status}`);
        }
        
        const resultado = await respuesta.json();
        console.log('Post actualizado:', resultado);
        return resultado;
        
    } catch (error) {
        console.error('Error al actualizar:', error);
    }
}

actualizarPost(1);
```

### Diferencia entre PUT y PATCH

```javascript
// PUT - Reemplaza TODO el recurso
// Debes enviar TODOS los campos
await fetch(url, {
    method: 'PUT',
    body: JSON.stringify({
        id: 1,
        title: 'Nuevo título',
        body: 'Nuevo contenido',
        userId: 1  // ⚠️ Obligatorio aunque no cambie
    })
});

// PATCH - Actualiza SOLO los campos que envías
await fetch(url, {
    method: 'PATCH',
    body: JSON.stringify({
        title: 'Nuevo título'  // ✅ Solo actualiza el título
    })
});
```

---

## DELETE - Eliminar recursos

**DELETE elimina un recurso**.

### Eliminar un post

```javascript
async function eliminarPost(id) {
    try {
        const respuesta = await fetch(`https://jsonplaceholder.typicode.com/posts/${id}`, {
            method: 'DELETE'
        });
        
        if (!respuesta.ok) {
            throw new Error(`Error HTTP: ${respuesta.status}`);
        }
        
        console.log('Post eliminado correctamente');
        // Nota: JSONPlaceholder devuelve un objeto vacío {}
        
    } catch (error) {
        console.error('Error al eliminar:', error);
    }
}

eliminarPost(1);
```

### Con confirmación del usuario

```javascript
async function eliminarConConfirmacion(id) {
    const confirmar = confirm('¿Estás seguro de eliminar este post?');
    
    if (!confirmar) {
        console.log('Eliminación cancelada');
        return;
    }
    
    try {
        const respuesta = await fetch(`https://jsonplaceholder.typicode.com/posts/${id}`, {
            method: 'DELETE'
        });
        
        if (!respuesta.ok) {
            throw new Error('No se pudo eliminar');
        }
        
        alert('Post eliminado correctamente');
        
    } catch (error) {
        alert('Error al eliminar: ' + error.message);
    }
}
```

---

## PATCH - Actualización parcial

**PATCH actualiza solo los campos que envías**.

### Actualizar solo el título

```javascript
async function actualizarTitulo(id, nuevoTitulo) {
    try {
        const respuesta = await fetch(`https://jsonplaceholder.typicode.com/posts/${id}`, {
            method: 'PATCH',
            headers: {
                'Content-Type': 'application/json'
            },
            body: JSON.stringify({
                title: nuevoTitulo
            })
        });
        
        if (!respuesta.ok) {
            throw new Error(`Error HTTP: ${respuesta.status}`);
        }
        
        const resultado = await respuesta.json();
        console.log('Título actualizado:', resultado);
        return resultado;
        
    } catch (error) {
        console.error('Error:', error);
    }
}

actualizarTitulo(1, 'Nuevo título increíble');
```

---

## Enviar JSON

**La forma más común de enviar datos**.

### Headers necesarios

```javascript
headers: {
    'Content-Type': 'application/json'
}
```

### Convertir objeto a JSON

```javascript
const datos = {
    name: 'Ana',
    email: 'ana@example.com',
    age: 25
};

// Convertir a JSON string
const json = JSON.stringify(datos);
console.log(json); // '{"name":"Ana","email":"ana@example.com","age":25}'

// Enviar
await fetch(url, {
    method: 'POST',
    headers: {
        'Content-Type': 'application/json'
    },
    body: json
});
```

### Ejemplo completo - CRUD

```javascript
const API_URL = 'https://jsonplaceholder.typicode.com/posts';

// CREATE
async function crear(datos) {
    const respuesta = await fetch(API_URL, {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(datos)
    });
    return respuesta.json();
}

// READ
async function obtener(id) {
    const respuesta = await fetch(`${API_URL}/${id}`);
    return respuesta.json();
}

// UPDATE (completo)
async function actualizar(id, datos) {
    const respuesta = await fetch(`${API_URL}/${id}`, {
        method: 'PUT',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(datos)
    });
    return respuesta.json();
}

// UPDATE (parcial)
async function actualizarParcial(id, datos) {
    const respuesta = await fetch(`${API_URL}/${id}`, {
        method: 'PATCH',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(datos)
    });
    return respuesta.json();
}

// DELETE
async function eliminar(id) {
    const respuesta = await fetch(`${API_URL}/${id}`, {
        method: 'DELETE'
    });
    return respuesta.json();
}

// Uso
const nuevoPost = await crear({ title: 'Test', body: 'Contenido', userId: 1 });
const post = await obtener(1);
await actualizar(1, { title: 'Actualizado', body: 'Nuevo', userId: 1 });
await actualizarParcial(1, { title: 'Solo título' });
await eliminar(1);
```

---

## Enviar FormData

**Para enviar archivos o datos de formularios**.

### FormData básico

```javascript
const formData = new FormData();
formData.append('name', 'Ana García');
formData.append('email', 'ana@example.com');

await fetch(url, {
    method: 'POST',
    body: formData
    // ⚠️ NO pongas Content-Type, el navegador lo hace automáticamente
});
```

### Desde un formulario HTML

```html
<form id="miForm">
    <input type="text" name="name" value="Ana">
    <input type="email" name="email" value="ana@example.com">
    <input type="file" name="avatar">
    <button type="submit">Enviar</button>
</form>

<script>
document.getElementById('miForm').addEventListener('submit', async (e) => {
    e.preventDefault();
    
    const formData = new FormData(e.target);
    
    const respuesta = await fetch('https://ejemplo.com/api/users', {
        method: 'POST',
        body: formData
    });
    
    const resultado = await respuesta.json();
    console.log(resultado);
});
</script>
```

### Con archivos

```javascript
async function subirArchivo(archivo) {
    const formData = new FormData();
    formData.append('file', archivo);
    formData.append('name', archivo.name);
    
    try {
        const respuesta = await fetch('https://ejemplo.com/upload', {
            method: 'POST',
            body: formData
        });
        
        if (!respuesta.ok) {
            throw new Error('Error al subir archivo');
        }
        
        const resultado = await respuesta.json();
        console.log('Archivo subido:', resultado);
        
    } catch (error) {
        console.error('Error:', error);
    }
}

// Uso con input file
document.getElementById('inputFile').addEventListener('change', (e) => {
    const archivo = e.target.files[0];
    if (archivo) {
        subirArchivo(archivo);
    }
});
```

---

## Ejercicios prácticos

### Ejercicio 1: CRUD completo
Crea una app que permita crear, leer, actualizar y eliminar posts de JSONPlaceholder.

### Ejercicio 2: Lista de tareas
Crea una app de tareas donde puedas agregar, marcar como completadas y eliminar tareas.

### Ejercicio 3: Formulario de contacto
Crea un formulario que envíe datos via POST y muestre confirmación.

### Ejercicio 4: Edición inline
Crea una lista donde puedas hacer clic en un elemento para editarlo directamente (PATCH).

### Ejercicio 5: Gestor de usuarios
Crea una app que liste usuarios y permita agregar, editar y eliminar.

---

## Errores comunes

### ❌ Olvidar Content-Type

```javascript
// Mal
await fetch(url, {
    method: 'POST',
    body: JSON.stringify(datos) // ❌ Sin Content-Type
});

// Bien
await fetch(url, {
    method: 'POST',
    headers: {
        'Content-Type': 'application/json'
    },
    body: JSON.stringify(datos)
});
```

### ❌ No usar JSON.stringify

```javascript
// Mal
body: datos // ❌ Objeto directo

// Bien
body: JSON.stringify(datos) // ✅ Convertir a JSON string
```

### ❌ Enviar Content-Type con FormData

```javascript
// Mal
await fetch(url, {
    method: 'POST',
    headers: {
        'Content-Type': 'multipart/form-data' // ❌ No pongas esto
    },
    body: formData
});

// Bien
await fetch(url, {
    method: 'POST',
    body: formData // ✅ El navegador añade el Content-Type correcto
});
```

---

## Buenas prácticas

### ✅ Centralizar configuración

```javascript
const API_CONFIG = {
    baseURL: 'https://jsonplaceholder.typicode.com',
    headers: {
        'Content-Type': 'application/json'
    }
};

async function post(endpoint, datos) {
    const respuesta = await fetch(`${API_CONFIG.baseURL}${endpoint}`, {
        method: 'POST',
        headers: API_CONFIG.headers,
        body: JSON.stringify(datos)
    });
    return respuesta.json();
}
```

### ✅ Funciones reutilizables

```javascript
async function request(url, method = 'GET', datos = null) {
    const opciones = {
        method,
        headers: {
            'Content-Type': 'application/json'
        }
    };
    
    if (datos) {
        opciones.body = JSON.stringify(datos);
    }
    
    const respuesta = await fetch(url, opciones);
    
    if (!respuesta.ok) {
        throw new Error(`Error HTTP: ${respuesta.status}`);
    }
    
    return respuesta.json();
}

// Uso
const posts = await request(url, 'GET');
const nuevo = await request(url, 'POST', { title: 'Test' });
const actualizado = await request(url, 'PUT', { title: 'Updated' });
await request(url, 'DELETE');
```

### ✅ Validar antes de enviar

```javascript
async function crearPost(datos) {
    // Validar
    if (!datos.title || !datos.body) {
        throw new Error('Título y contenido son obligatorios');
    }
    
    if (datos.title.length < 3) {
        throw new Error('El título debe tener al menos 3 caracteres');
    }
    
    // Enviar
    return await fetch(url, {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(datos)
    });
}
```

---

## Cheatsheet

```javascript
// ========== POST - Crear ==========
await fetch(url, {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({ title: 'Test', body: 'Content' })
});

// ========== PUT - Actualizar completo ==========
await fetch(`${url}/${id}`, {
    method: 'PUT',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({ id, title: 'Updated', body: 'New', userId: 1 })
});

// ========== PATCH - Actualizar parcial ==========
await fetch(`${url}/${id}`, {
    method: 'PATCH',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({ title: 'Only title' })
});

// ========== DELETE - Eliminar ==========
await fetch(`${url}/${id}`, {
    method: 'DELETE'
});

// ========== FormData ==========
const formData = new FormData();
formData.append('name', 'Ana');
await fetch(url, {
    method: 'POST',
    body: formData
});

// ========== Función reutilizable ==========
async function api(url, method, data) {
    const options = { method, headers: { 'Content-Type': 'application/json' } };
    if (data) options.body = JSON.stringify(data);
    const res = await fetch(url, options);
    if (!res.ok) throw new Error(`HTTP ${res.status}`);
    return res.json();
}
```

---

**Siguiente:** [Módulo 34 - Headers y autenticación](./34-headers-autenticacion.md)

**Anterior:** [Módulo 32 - Manejo de respuestas y errores](./32-manejo-respuestas-errores.md)

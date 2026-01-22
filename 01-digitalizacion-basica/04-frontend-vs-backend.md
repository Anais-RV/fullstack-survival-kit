# Frontend vs Backend

> **La diferencia entre lo que ves y lo que no ves**

---

## La web es como un restaurante

Imagina que vas a un restaurante:

### Frontend (sala del restaurante)

**Lo que tú ves:**
- La mesa, las sillas, la decoración
- El menú con fotos bonitas
- El camarero que te atiende
- La presentación del plato

**Tu experiencia:**
- Eliges del menú
- Pides tu comida
- Ves lo que comes
- Interactúas con la interfaz (menú, camarero)

### Backend (cocina del restaurante)

**Lo que NO ves:**
- Los cocineros preparando la comida
- La nevera con ingredientes
- Las recetas y procesos
- El sistema de pedidos
- El inventario

**El trabajo oculto:**
- Recibir tu pedido
- Preparar la comida según la receta
- Verificar que hay ingredientes
- Controlar los tiempos
- Enviar el plato listo

---

## En desarrollo web

### Frontend (lo que el usuario ve)

**Dónde se ejecuta:** En el navegador del usuario

**Tecnologías principales:**
- **HTML** → Estructura (el esqueleto)
- **CSS** → Diseño visual (la ropa)
- **JavaScript** → Interactividad (el movimiento)

**Responsabilidades:**
- Mostrar información
- Capturar lo que el usuario hace (clics, escritura...)
- Validar datos antes de enviarlos
- Hacer la experiencia agradable y fluida

**Ejemplo visual:**

```html
<!-- HTML: Estructura -->
<button>Enviar mensaje</button>

<!-- CSS: Diseño -->
<style>
  button {
    background: blue;
    color: white;
    padding: 10px;
  }
</style>

<!-- JavaScript: Interactividad -->
<script>
  button.onclick = () => {
    alert('¡Mensaje enviado!');
  }
</script>
```

### Backend (lo que el usuario no ve)

**Dónde se ejecuta:** En un servidor (otro ordenador)

**Tecnologías comunes:**
- **Python** (Django, Flask)
- **JavaScript** (Node.js)
- **Java** (Spring)
- **PHP** (Laravel)
- **Ruby** (Rails)

**Responsabilidades:**
- Procesar lógica de negocio
- Guardar y recuperar datos
- Autenticar usuarios
- Proteger información sensible
- Realizar cálculos complejos
- Enviar emails, procesar pagos...

**Ejemplo conceptual:**

```python
# Backend: Procesar login
def login(usuario, contraseña):
    # 1. Buscar usuario en base de datos
    user = database.find(usuario)
    
    # 2. Verificar contraseña
    if user.contraseña == hash(contraseña):
        # 3. Generar token de sesión
        return crear_token(user)
    else:
        return "Error: Credenciales incorrectas"
```

---

## Ejemplos reales

### Ejemplo 1: Iniciar sesión

#### Frontend

```html
<!-- Formulario que el usuario ve -->
<form>
  <input type="text" name="email" placeholder="Email">
  <input type="password" name="password" placeholder="Contraseña">
  <button>Iniciar sesión</button>
</form>

<script>
  // JavaScript captura el envío
  form.onsubmit = (e) => {
    e.preventDefault();
    
    // Envía datos al backend
    fetch('/api/login', {
      method: 'POST',
      body: JSON.stringify({
        email: email.value,
        password: password.value
      })
    })
    .then(response => {
      if (response.ok) {
        alert('¡Bienvenido!');
      } else {
        alert('Credenciales incorrectas');
      }
    });
  }
</script>
```

#### Backend

```python
# Python/Django: Procesar login en servidor
@app.route('/api/login', methods=['POST'])
def login():
    data = request.json
    
    # Buscar usuario en base de datos
    user = User.objects.get(email=data['email'])
    
    # Verificar contraseña (hasheada)
    if user.check_password(data['password']):
        # Crear sesión
        token = create_token(user)
        return {'token': token, 'user': user.name}
    else:
        return {'error': 'Credenciales incorrectas'}, 401
```

**Flujo completo:**
1. Usuario escribe email y contraseña (frontend)
2. JavaScript envía datos al servidor (petición)
3. Servidor verifica en base de datos (backend)
4. Servidor responde éxito o error (respuesta)
5. JavaScript muestra el resultado (frontend)

---

### Ejemplo 2: Ver tu perfil

#### Frontend

```javascript
// JavaScript pide datos al servidor
fetch('/api/perfil/123')
  .then(response => response.json())
  .then(data => {
    // Mostrar datos en la página
    document.getElementById('nombre').textContent = data.nombre;
    document.getElementById('email').textContent = data.email;
    document.getElementById('foto').src = data.foto;
  });
```

#### Backend

```python
# Backend busca los datos
@app.route('/api/perfil/<user_id>')
def get_perfil(user_id):
    # Buscar en base de datos
    user = database.users.find_one({'id': user_id})
    
    # Devolver solo datos públicos
    return {
        'nombre': user.nombre,
        'email': user.email,
        'foto': user.foto_url
        # NO devolver contraseña ni datos sensibles
    }
```

**Flujo:**
1. JavaScript pide datos (frontend)
2. Servidor busca en base de datos (backend)
3. Servidor responde con datos (backend)
4. JavaScript los muestra en pantalla (frontend)

---

## ¿Qué hace cada uno?

### Frontend

| Tarea | Frontend |
|-------|----------|
| Mostrar información | ✅ |
| Formularios y botones | ✅ |
| Animaciones visuales | ✅ |
| Validación básica (formato email, campos vacíos) | ✅ |
| Navegación entre páginas | ✅ |
| **Guardar datos permanentemente** | ❌ |
| **Verificar contraseñas** | ❌ |
| **Procesar pagos** | ❌ |

### Backend

| Tarea | Backend |
|-------|---------|
| Guardar datos en base de datos | ✅ |
| Autenticación y seguridad | ✅ |
| Lógica de negocio compleja | ✅ |
| Enviar emails | ✅ |
| Procesar pagos | ✅ |
| Conectar con otros servicios | ✅ |
| **Dibujar la interfaz** | ❌ |
| **Manejar clics y eventos** | ❌ |

---

## La comunicación: Frontend ↔ Backend

### Peticiones HTTP

Frontend envía peticiones, backend responde.

#### GET (pedir datos)

```javascript
// Frontend: "Dame la lista de productos"
fetch('/api/productos')
  .then(res => res.json())
  .then(productos => console.log(productos));
```

```python
# Backend: "Aquí están"
@app.route('/api/productos')
def get_productos():
    productos = database.productos.find()
    return productos
```

#### POST (enviar datos)

```javascript
// Frontend: "Guarda este nuevo producto"
fetch('/api/productos', {
  method: 'POST',
  body: JSON.stringify({
    nombre: 'Laptop',
    precio: 999
  })
});
```

```python
# Backend: "Ok, guardado"
@app.route('/api/productos', methods=['POST'])
def crear_producto():
    data = request.json
    database.productos.insert(data)
    return {'status': 'ok'}
```

---

## ¿Necesitas saber los dos?

**Sí, si quieres ser fullstack.**

**Fullstack developer** = Puede trabajar en frontend y backend.

**Ventajas:**
- Entiendes el sistema completo
- Puedes trabajar solo/a en proyectos pequeños
- Mejor comunicación entre equipos
- Más oportunidades laborales

**Orden de aprendizaje recomendado:**
1. **Frontend primero** (HTML, CSS, JavaScript)
2. **Backend después** (Python, Node.js...)
3. **Integración** (conectar ambos)

¿Por qué? Frontend es más visual y rápido de ver resultados. Backend requiere entender conceptos más abstractos.

---

## Analogía visual

```
┌─────────────────────────────────────┐
│         NAVEGADOR (Cliente)         │
│                                     │
│  ┌─────────────────────────────┐   │
│  │        FRONTEND              │   │
│  │                              │   │
│  │  - HTML (estructura)         │   │
│  │  - CSS (diseño)              │   │
│  │  - JavaScript (interacción)  │   │
│  │                              │   │
│  └─────────────────────────────┘   │
│                                     │
│           ↕ Peticiones HTTP         │
│                                     │
└─────────────────────────────────────┘

                    ↕

┌─────────────────────────────────────┐
│         SERVIDOR (Backend)          │
│                                     │
│  ┌─────────────────────────────┐   │
│  │         BACKEND              │   │
│  │                              │   │
│  │  - Lógica de negocio         │   │
│  │  - Base de datos             │   │
│  │  - Autenticación             │   │
│  │  - APIs                      │   │
│  │                              │   │
│  └─────────────────────────────┘   │
│                                     │
└─────────────────────────────────────┘
```

---

## Ejemplos de apps reales

### Instagram

**Frontend:**
- El feed que ves
- Los botones de like
- Los comentarios que escribes
- Los filtros de fotos
- Las animaciones

**Backend:**
- Guardar las fotos en servidores
- Procesar filtros pesados
- Sistema de seguidores
- Notificaciones
- Recomendaciones de cuentas

### Gmail

**Frontend:**
- La interfaz del correo
- El editor de mensajes
- Los botones y menús
- La búsqueda visual

**Backend:**
- Guardar los emails
- Enviar y recibir correos
- Filtro de spam
- Búsqueda en todos tus emails
- Organizar en carpetas

---

## Conclusión

### Frontend (lo visible)

- ✅ Se ejecuta en el navegador
- ✅ HTML, CSS, JavaScript
- ✅ Lo que el usuario ve y toca
- ✅ Diseño, interacción, experiencia

### Backend (lo invisible)

- ✅ Se ejecuta en el servidor
- ✅ Python, Node.js, Java, PHP...
- ✅ Lógica, datos, seguridad
- ✅ Procesamiento, almacenamiento

### Los dos trabajan juntos

- Frontend pide → Backend responde
- Backend prepara → Frontend muestra
- Usuario interactúa con frontend → Backend procesa

---

## Ahora entiende esto

Cuando haces clic en "Me gusta" en una red social:

1. **Frontend** detecta tu clic
2. **Frontend** envía petición al servidor: "Usuario X le dio like a publicación Y"
3. **Backend** guarda esto en la base de datos
4. **Backend** responde: "Ok, guardado"
5. **Frontend** cambia el botón a "liked" (corazón rojo)
6. **Backend** actualiza el contador de likes
7. **Frontend** muestra el nuevo número

**Todo en milisegundos.**

**Siguiente:** [Tu primer HTML](./05-primer-html.md)

---

## Reflexión

Ahora, cada vez que uses una web o app, pregúntate:
- ¿Qué parte es frontend?
- ¿Qué parte es backend?
- ¿Qué información viaja entre ellos?

**Este ejercicio mental te ayudará a pensar como desarrollador/a.**

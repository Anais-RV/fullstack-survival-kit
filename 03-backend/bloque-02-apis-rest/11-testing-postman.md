# Testing con Postman

> **Pruebas profesionales de APIs REST**

---

## ¿Qué es Postman?

**Postman** es una herramienta para probar APIs:

- ✅ Enviar requests HTTP (GET, POST, PUT, DELETE, etc.)
- ✅ Ver responses completas (headers, body, status)
- ✅ Organizar requests en colecciones
- ✅ Automatizar tests
- ✅ Compartir colecciones con el equipo
- ✅ Documentar APIs

**Descargar:** [postman.com](https://www.postman.com/)

---

## Alternativas a Postman

- **Thunder Client** (extensión de VS Code)
- **Insomnia**
- **httpie** (línea de comandos)
- **curl** (línea de comandos)

---

## Primera request en Postman

### 1. Crear nueva request

1. Click en **"New"** → **"HTTP Request"**
2. Seleccionar método: **GET**
3. URL: `http://localhost:8000/api/usuarios`
4. Click en **"Send"**

### 2. Ver response

- **Status:** `200 OK`
- **Body:**
  ```json
  [
    {
      "id": 1,
      "nombre": "Juan",
      "email": "juan@ejemplo.com"
    }
  ]
  ```
- **Headers:** Content-Type, CORS, etc.
- **Time:** Tiempo de respuesta

---

## Testing de endpoints

### GET /api/usuarios

```
GET http://localhost:8000/api/usuarios
```

**Expected:**
- Status: `200 OK`
- Body: Array de usuarios

### GET /api/usuarios/:id

```
GET http://localhost:8000/api/usuarios/1
```

**Expected:**
- Status: `200 OK`
- Body: Usuario con id=1

### POST /api/usuarios

```
POST http://localhost:8000/api/usuarios
Content-Type: application/json

{
  "nombre": "Pedro",
  "email": "pedro@ejemplo.com",
  "edad": 28
}
```

**Expected:**
- Status: `201 Created`
- Header: `Location: /api/usuarios/3`
- Body: Usuario creado con id

### PUT /api/usuarios/:id

```
PUT http://localhost:8000/api/usuarios/1
Content-Type: application/json

{
  "nombre": "Juan Actualizado",
  "email": "juan.nuevo@ejemplo.com",
  "edad": 31
}
```

**Expected:**
- Status: `200 OK`
- Body: Usuario actualizado

### PATCH /api/usuarios/:id

```
PATCH http://localhost:8000/api/usuarios/1
Content-Type: application/json

{
  "edad": 32
}
```

**Expected:**
- Status: `200 OK`
- Body: Usuario con solo edad actualizada

### DELETE /api/usuarios/:id

```
DELETE http://localhost:8000/api/usuarios/1
```

**Expected:**
- Status: `204 No Content`
- Body: (vacío)

---

## Tests automáticos en Postman

### Tab "Tests"

Postman permite escribir tests en JavaScript:

```javascript
// Test 1: Status code es 200
pm.test("Status code es 200", function () {
    pm.response.to.have.status(200);
});

// Test 2: Response es JSON
pm.test("Response es JSON", function () {
    pm.response.to.be.json;
});

// Test 3: Body contiene array
pm.test("Body es un array", function () {
    const data = pm.response.json();
    pm.expect(data).to.be.an('array');
});

// Test 4: Array no está vacío
pm.test("Array tiene elementos", function () {
    const data = pm.response.json();
    pm.expect(data.length).to.be.above(0);
});

// Test 5: Primer usuario tiene campos requeridos
pm.test("Usuario tiene campos requeridos", function () {
    const usuario = pm.response.json()[0];
    pm.expect(usuario).to.have.property('id');
    pm.expect(usuario).to.have.property('nombre');
    pm.expect(usuario).to.have.property('email');
});
```

### Tests para POST

```javascript
// Test: Status 201 Created
pm.test("Status code es 201", function () {
    pm.response.to.have.status(201);
});

// Test: Location header presente
pm.test("Location header presente", function () {
    pm.response.to.have.header("Location");
});

// Test: Usuario creado tiene ID
pm.test("Usuario tiene ID", function () {
    const usuario = pm.response.json();
    pm.expect(usuario).to.have.property('id');
    pm.expect(usuario.id).to.be.a('number');
});

// Test: Nombre coincide
pm.test("Nombre coincide", function () {
    const usuario = pm.response.json();
    pm.expect(usuario.nombre).to.equal("Pedro");
});

// Guardar ID para siguiente request
const usuario = pm.response.json();
pm.environment.set("ultimo_usuario_id", usuario.id);
```

### Tests para errores

```javascript
// Test 404
pm.test("Status 404 cuando usuario no existe", function () {
    pm.response.to.have.status(404);
});

pm.test("Error contiene mensaje", function () {
    const error = pm.response.json();
    pm.expect(error).to.have.property('error');
    pm.expect(error.error).to.include('no encontrado');
});

// Test 422 (validación)
pm.test("Status 422 cuando validación falla", function () {
    pm.response.to.have.status(422);
});

pm.test("Error tiene detalles de validación", function () {
    const error = pm.response.json();
    pm.expect(error).to.have.property('detalles');
    pm.expect(error.detalles).to.have.property('errores');
});
```

---

## Variables y entornos

### Variables de entorno

**Crear entorno:**
1. Click en icono de ojo (top right)
2. **Add** → Nombre: `Desarrollo`
3. Agregar variables:
   - `base_url`: `http://localhost:8000`
   - `usuario_id`: (vacío, se llenará dinámicamente)

**Usar en requests:**
```
GET {{base_url}}/api/usuarios
GET {{base_url}}/api/usuarios/{{usuario_id}}
```

### Pre-request Scripts

Ejecutar antes de enviar request:

```javascript
// Generar datos aleatorios
const nombre = `Usuario_${Math.floor(Math.random() * 1000)}`;
const email = `user${Math.floor(Math.random() * 1000)}@ejemplo.com`;

// Guardar en variables
pm.environment.set("nombre_aleatorio", nombre);
pm.environment.set("email_aleatorio", email);
```

**En body:**
```json
{
  "nombre": "{{nombre_aleatorio}}",
  "email": "{{email_aleatorio}}"
}
```

---

## Colecciones

### Crear colección

1. **Collections** → **New Collection**
2. Nombre: `API Usuarios`
3. Descripción: API REST para gestión de usuarios

### Organizar requests

```
📁 API Usuarios
  📂 Usuarios
    GET    Listar usuarios
    GET    Obtener usuario
    POST   Crear usuario
    PUT    Actualizar completo
    PATCH  Actualizar parcial
    DELETE Eliminar usuario
  📂 Posts
    GET    Listar posts
    GET    Posts del usuario
```

### Compartir colección

1. Click en colección → **Share**
2. **Export** → JSON
3. Compartir archivo `.json`
4. Otros lo importan: **Import** → seleccionar archivo

---

## Test automatizado completo

### Suite de tests para CRUD

**1. GET /api/usuarios**
```javascript
pm.test("Status 200", () => pm.response.to.have.status(200));
pm.test("Es array", () => {
    const data = pm.response.json();
    pm.expect(data).to.be.an('array');
});
```

**2. POST /api/usuarios (Crear)**
```javascript
pm.test("Status 201", () => pm.response.to.have.status(201));
pm.test("Location header", () => pm.response.to.have.header("Location"));

const usuario = pm.response.json();
pm.test("Tiene ID", () => pm.expect(usuario.id).to.exist);
pm.environment.set("test_usuario_id", usuario.id);
```

**3. GET /api/usuarios/:id (Verificar creación)**
```javascript
const usuarioId = pm.environment.get("test_usuario_id");

pm.test("Status 200", () => pm.response.to.have.status(200));

const usuario = pm.response.json();
pm.test("ID coincide", () => {
    pm.expect(usuario.id).to.equal(Number(usuarioId));
});
```

**4. PATCH /api/usuarios/:id (Actualizar)**
```javascript
pm.test("Status 200", () => pm.response.to.have.status(200));

const usuario = pm.response.json();
pm.test("Edad actualizada", () => {
    pm.expect(usuario.edad).to.equal(35);
});
```

**5. DELETE /api/usuarios/:id (Eliminar)**
```javascript
pm.test("Status 204", () => pm.response.to.have.status(204));
```

**6. GET /api/usuarios/:id (Verificar eliminación)**
```javascript
pm.test("Status 404", () => pm.response.to.have.status(404));
```

---

## Collection Runner

**Ejecutar toda la colección:**

1. Click en colección → **Run**
2. Seleccionar requests a ejecutar
3. Click **Run API Usuarios**

**Resultado:**
```
Listar usuarios           ✅ 2/2 tests passed
Crear usuario             ✅ 3/3 tests passed
Obtener usuario           ✅ 2/2 tests passed
Actualizar parcial        ✅ 2/2 tests passed
Eliminar usuario          ✅ 1/1 tests passed
Verificar eliminación     ✅ 1/1 tests passed

Total: 11/11 tests passed (100%)
```

---

## Documentación automática

Postman genera documentación HTML automáticamente:

1. Click en colección → **View documentation**
2. Se genera docs con:
   - Descripción de cada endpoint
   - Ejemplos de request
   - Ejemplos de response
   - Tests incluidos

**Publicar:**
1. **Publish** → URL pública
2. Compartir con equipo o clientes

---

## Exportar para curl

Postman puede generar código curl:

1. Request → **Code** (icono `</>`)
2. Seleccionar: **cURL**
3. Copiar código:

```bash
curl --location 'http://localhost:8000/api/usuarios' \
--header 'Content-Type: application/json' \
--data-raw '{
    "nombre": "Pedro",
    "email": "pedro@ejemplo.com"
}'
```

También soporta:
- Python requests
- JavaScript fetch
- Node.js axios
- PHP
- Java
- Go

---

## Mock Server

Postman puede crear servidor mock:

1. Colección → **Mocks**
2. **Create Mock Server**
3. Ahora tienes URL: `https://xyz.mock.pstmn.io`
4. Responde con ejemplos guardados

**Uso:** Desarrollar frontend sin backend real.

---

## Mejores prácticas

### 1. Usar variables para todo

```javascript
// ❌ Hardcoded
GET http://localhost:8000/api/usuarios/123

// ✅ Con variables
GET {{base_url}}/api/usuarios/{{usuario_id}}
```

### 2. Tests en cada request

Todos los requests deben tener al menos:
```javascript
pm.test("Status correcto", () => {
    pm.response.to.have.status(200);
});
```

### 3. Organizar por flujo

```
📁 Flujo: Registro y Login
  POST   Registrar usuario
  POST   Login
  GET    Perfil (con token)
```

### 4. Limpiar después de tests

```javascript
// Al final de DELETE
pm.test("Limpieza: eliminar variable", () => {
    pm.environment.unset("test_usuario_id");
});
```

---

## Thunder Client (VS Code)

Alternativa integrada en VS Code:

1. Instalar extensión **Thunder Client**
2. Abrir panel lateral (icono rayo ⚡)
3. **New Request**

**Ventajas:**
- ✅ Integrado en VS Code
- ✅ No requiere cuenta
- ✅ Guarda en workspace
- ✅ Similar a Postman

---

## Ejercicios

### 1. Suite completa

Crea colección con todos los endpoints y tests para:
- Usuarios
- Posts
- Comentarios

### 2. Test de paginación

```javascript
pm.test("Tiene metadata de paginación", () => {
    const data = pm.response.json();
    pm.expect(data).to.have.property('meta');
    pm.expect(data.meta).to.have.property('total');
    pm.expect(data.meta).to.have.property('page');
});
```

### 3. Test de performance

```javascript
pm.test("Response time < 200ms", () => {
    pm.expect(pm.response.responseTime).to.be.below(200);
});
```

---

## Resumen

Has aprendido:

✅ Usar Postman para testing de APIs  
✅ Tests automáticos con JavaScript  
✅ Variables y entornos  
✅ Colecciones organizadas  
✅ Collection Runner para suites completas  
✅ Documentación automática  
✅ Exportar a curl y otros lenguajes

**Siguiente:** [Versionado de APIs](./12-versionado-api.md)

---

## Recursos

- **[Postman Learning Center](https://learning.postman.com/)** - Documentación oficial
- **[Thunder Client](https://www.thunderclient.com/)** - Alternativa VS Code
- **[Insomnia](https://insomnia.rest/)** - Otra alternativa

Postman es tu mejor aliado para testing de APIs. 🧪

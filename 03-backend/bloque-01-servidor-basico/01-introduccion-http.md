# Introducción a HTTP

> **El protocolo que hace funcionar la web**

---

## ¿Qué es HTTP?

**HTTP** (HyperText Transfer Protocol) es el protocolo de comunicación que permite la transferencia de información en la World Wide Web.

```
Cliente (navegador)  ←→  Servidor (backend)
      REQUEST  →
      ← RESPONSE
```

**Analogía simple:**
- HTTP es como el lenguaje que hablan tu navegador y el servidor
- El navegador hace **preguntas** (requests)
- El servidor da **respuestas** (responses)

---

## Conceptos fundamentales

### 1. Cliente y Servidor

**Cliente:**
- Tu navegador (Chrome, Firefox, etc.)
- Una app móvil
- Otro servidor haciendo peticiones

**Servidor:**
- Una computadora que espera peticiones
- Procesa la petición
- Devuelve una respuesta

### 2. Request (Petición)

Un request HTTP tiene:

```http
GET /usuarios HTTP/1.1
Host: api.ejemplo.com
User-Agent: Mozilla/5.0
Accept: application/json
```

**Partes:**
- **Método**: GET, POST, PUT, DELETE
- **Ruta**: /usuarios, /productos/123
- **Headers**: información adicional
- **Body**: datos (opcional)

### 3. Response (Respuesta)

Un response HTTP tiene:

```http
HTTP/1.1 200 OK
Content-Type: application/json
Content-Length: 45

{"mensaje": "Hola desde el servidor"}
```

**Partes:**
- **Status code**: 200, 404, 500, etc.
- **Headers**: tipo de contenido, fecha, etc.
- **Body**: los datos que devuelve

---

## Métodos HTTP principales

| Método | Propósito | Ejemplo |
|--------|-----------|---------|
| **GET** | Obtener datos | Listar usuarios |
| **POST** | Crear nuevos datos | Crear un usuario |
| **PUT** | Actualizar datos existentes | Modificar un usuario |
| **DELETE** | Eliminar datos | Borrar un usuario |

---

## Status Codes (Códigos de estado)

Los códigos de estado indican el resultado de la petición:

### 2xx - Éxito
- **200 OK**: Todo correcto
- **201 Created**: Recurso creado exitosamente
- **204 No Content**: Éxito pero sin contenido

### 3xx - Redirección
- **301 Moved Permanently**: El recurso se movió permanentemente
- **302 Found**: Redirección temporal

### 4xx - Error del cliente
- **400 Bad Request**: Petición mal formada
- **401 Unauthorized**: No autenticado
- **403 Forbidden**: No tienes permisos
- **404 Not Found**: No se encontró el recurso

### 5xx - Error del servidor
- **500 Internal Server Error**: Error en el servidor
- **502 Bad Gateway**: Problema con servidor intermedio
- **503 Service Unavailable**: Servidor no disponible

---

## Headers importantes

### Request Headers

```http
Host: api.ejemplo.com
User-Agent: Mozilla/5.0
Accept: application/json
Authorization: Bearer token123
Content-Type: application/json
```

- **Host**: dominio del servidor
- **User-Agent**: información del cliente
- **Accept**: tipo de respuesta que acepta
- **Authorization**: credenciales/token
- **Content-Type**: tipo de datos que envía

### Response Headers

```http
Content-Type: application/json
Content-Length: 123
Set-Cookie: session=abc123
Access-Control-Allow-Origin: *
```

- **Content-Type**: tipo de datos que devuelve
- **Content-Length**: tamaño de la respuesta
- **Set-Cookie**: establecer cookies
- **Access-Control-Allow-Origin**: CORS

---

## Ejemplo completo de comunicación

### 1. Cliente hace un GET

```http
GET /api/usuarios/1 HTTP/1.1
Host: localhost:3000
Accept: application/json
```

### 2. Servidor procesa

```python
# El servidor busca el usuario con ID 1
usuario = encontrar_usuario(1)
```

### 3. Servidor responde

```http
HTTP/1.1 200 OK
Content-Type: application/json

{
  "id": 1,
  "nombre": "Juan",
  "email": "juan@ejemplo.com"
}
```

---

## URLs y rutas

Una URL completa tiene varias partes:

```
https://api.ejemplo.com:443/usuarios?activo=true#seccion
└─┬─┘  └────┬────────┘ └┬┘ └───┬───┘ └────┬─────┘ └──┬──┘
  │         │           │      │           │          │
Protocolo  Dominio    Puerto  Ruta     Query params  Fragment
```

**Para tu servidor:**
- **Ruta**: `/usuarios`, `/productos/123`
- **Query params**: `?nombre=Juan&edad=25`
- **Fragment**: `#seccion` (solo frontend)

---

## Ejemplo práctico: ver HTTP en acción

### Opción 1: Con curl

```powershell
# Ver headers y respuesta
curl -v http://ejemplo.com

# Hacer POST con datos
curl -X POST http://localhost:3000/api/usuarios `
  -H "Content-Type: application/json" `
  -d '{"nombre":"Juan"}'
```

### Opción 2: Con navegador

1. Abre Chrome
2. Presiona **F12** (DevTools)
3. Ve a **Network**
4. Visita cualquier página
5. Haz clic en una petición
6. Verás:
   - Headers de request
   - Headers de response
   - Body de la respuesta

---

## Estados de conexión HTTP

HTTP es **stateless** (sin estado):

```
Request 1:  Cliente → Servidor
             (el servidor olvida todo)

Request 2:  Cliente → Servidor
             (el servidor no recuerda Request 1)
```

**Problema**: ¿Cómo mantener sesiones de usuario?

**Soluciones:**
1. **Cookies**: el servidor envía un ID de sesión
2. **Tokens JWT**: el cliente envía un token en cada request
3. **Local Storage**: el frontend guarda información

---

## HTTP/1.1 vs HTTP/2 vs HTTP/3

| Versión | Año | Características |
|---------|-----|-----------------|
| **HTTP/1.1** | 1997 | Una petición a la vez |
| **HTTP/2** | 2015 | Múltiples peticiones simultáneas, compresión de headers |
| **HTTP/3** | 2022 | Basado en QUIC, más rápido, más confiable |

**Para aprender**: usaremos HTTP/1.1 porque es más simple de entender.

---

## HTTPS - HTTP Seguro

**HTTPS** = HTTP + SSL/TLS (cifrado)

```
HTTP:  Cliente ←→ Servidor
       (todo visible)

HTTPS: Cliente ←[🔒cifrado🔒]→ Servidor
       (nadie puede leer)
```

**Diferencias:**
- **HTTP**: puerto 80, sin cifrado
- **HTTPS**: puerto 443, todo cifrado

---

## Ejercicio práctico

### 1. Inspeccionar una petición real

1. Abre DevTools (F12) en tu navegador
2. Ve a la pestaña Network
3. Visita https://jsonplaceholder.typicode.com/users/1
4. Observa:
   - El método (GET)
   - Los headers
   - El status code (200)
   - El body de la respuesta

### 2. Hacer peticiones con curl

```powershell
# GET simple
curl https://jsonplaceholder.typicode.com/users/1

# Ver headers
curl -i https://jsonplaceholder.typicode.com/users/1

# POST de datos
curl -X POST https://jsonplaceholder.typicode.com/posts `
  -H "Content-Type: application/json" `
  -d '{\"title\":\"Mi post\",\"body\":\"Contenido\",\"userId\":1}'
```

---

## Resumen visual

```
┌──────────────────────────────────────────┐
│         FLUJO HTTP COMPLETO              │
└──────────────────────────────────────────┘

1. Cliente hace REQUEST
   ├─ Método: GET
   ├─ Ruta: /api/usuarios
   ├─ Headers: Accept, Authorization
   └─ Body: (opcional)

2. Servidor recibe y procesa
   ├─ Parsea el request
   ├─ Ejecuta lógica
   └─ Prepara respuesta

3. Servidor envía RESPONSE
   ├─ Status: 200 OK
   ├─ Headers: Content-Type
   └─ Body: {"usuarios": [...]}

4. Cliente recibe y muestra
   └─ Renderiza la información
```

---

## Conceptos clave para recordar

✅ **HTTP es request/response**: cliente pregunta, servidor responde  
✅ **Métodos HTTP**: GET (leer), POST (crear), PUT (actualizar), DELETE (borrar)  
✅ **Status codes**: 2xx éxito, 4xx error cliente, 5xx error servidor  
✅ **Headers**: metadata sobre el request/response  
✅ **HTTP es stateless**: cada petición es independiente  
✅ **HTTPS es HTTP cifrado**: más seguro para producción

---

## Próximo paso

Ahora que entiendes HTTP, es hora de crear tu primer servidor que responda peticiones HTTP.

**Siguiente:** [Primer servidor Python](./02-primer-servidor-python.md)

---

## Recursos adicionales

- **[MDN HTTP](https://developer.mozilla.org/es/docs/Web/HTTP)** - Guía completa
- **[HTTP Status Codes](https://httpstatuses.com/)** - Lista de todos los códigos
- **[HTTP Methods](https://developer.mozilla.org/es/docs/Web/HTTP/Methods)** - Detalle de métodos

El HTTP es el fundamento de la web. ¡Domínalo! 🌐

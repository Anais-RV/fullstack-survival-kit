# Backend

Ya sabes programar y ya sabes hacer interfaces. Ahora vas a construir el servidor que hace que todo funcione.

**Este bloque te enseña a crear la lógica de negocio, APIs y sistemas del lado del servidor.**

---

## Sobre este bloque

Backend es todo lo que el usuario no ve: la lógica, los datos, la seguridad, las reglas del sistema.

Este bloque introduce también **Programación Orientada a Objetos (POO)** de forma aplicada: no como teoría abstracta, sino como herramienta para modelar sistemas reales.

La estructura sigue este orden:
1. **Servidor básico** — Qué es y cómo funciona
2. **APIs REST** — Exponer datos y funcionalidad
3. **POO aplicada** — Modelar sistemas con clases y objetos
4. **Sistema práctico (tipo juego)** — Aplicar POO en un sistema vivo
5. **Autenticación** — Proteger recursos
6. **Arquitectura backend** — Estructurar código mantenible

Al final de este bloque serás capaz de construir servidores completos con lógica compleja.

---

## Bloque 1: Servidor básico

Qué es un servidor y cómo funciona por dentro.

### `01-que-es-servidor.md`
**Descripción:** Qué hace un servidor, diferencia con el navegador, request-response.  
**Nivel:** Esencial

### `02-primer-servidor-http.md`
**Descripción:** Crear tu primer servidor que responda "Hola mundo".  
**Nivel:** Esencial

### `03-rutas-parametros.md`
**Descripción:** Definir rutas, capturar parámetros de URL, query params.  
**Nivel:** Esencial

### `04-metodos-http-servidor.md`
**Descripción:** GET, POST, PUT, DELETE. Qué hace cada uno en el servidor.  
**Nivel:** Esencial

### `05-request-body.md`
**Descripción:** Recibir datos del cliente. Parsear JSON.  
**Nivel:** Esencial

### `06-response-formatos.md`
**Descripción:** Devolver JSON, texto, HTML. Códigos de estado.  
**Nivel:** Esencial

---

## Bloque 2: API REST completa

Diseñar y construir APIs profesionales.

### `07-que-es-api-rest.md`
**Descripción:** Principios REST, recursos, verbos HTTP. Diseño de URLs.  
**Nivel:** Esencial

### `08-crud-completo.md`
**Descripción:** Create, Read, Update, Delete. CRUD de un recurso completo.  
**Nivel:** Esencial

### `09-validacion-datos.md`
**Descripción:** Validar datos antes de procesarlos. Devolver errores claros.  
**Nivel:** Esencial

### `10-middleware.md`
**Descripción:** Funciones que se ejecutan antes de las rutas. Logging, validación, autenticación.  
**Nivel:** Esencial

### `11-manejo-errores-servidor.md`
**Descripción:** Capturar errores, no romper el servidor, devolver respuestas útiles.  
**Nivel:** Esencial

### `12-cors-servidor.md`
**Descripción:** Permitir que tu frontend se comunique con tu backend. Configurar CORS.  
**Nivel:** Esencial

---

## Bloque 3: POO aplicada

Programación Orientada a Objetos como herramienta para modelar sistemas.

### `13-por-que-poo.md`
**Descripción:** Qué problemas resuelve POO. Cuándo usarla y cuándo no.  
**Nivel:** Esencial

### `14-clases-objetos.md`
**Descripción:** Crear clases, instanciar objetos, propiedades y métodos.  
**Nivel:** Esencial

### `15-constructor-inicializacion.md`
**Descripción:** Inicializar objetos con datos. Constructor y valores por defecto.  
**Nivel:** Esencial

### `16-encapsulacion.md`
**Descripción:** Propiedades privadas, getters, setters. Controlar acceso a datos.  
**Nivel:** Recomendado

### `17-herencia.md`
**Descripción:** Clases que heredan de otras. Reutilizar comportamiento.  
**Nivel:** Recomendado

### `18-polimorfismo.md`
**Descripción:** Mismo método, comportamiento diferente según la clase.  
**Nivel:** Opcional

### `19-composicion-vs-herencia.md`
**Descripción:** Cuándo componer objetos en lugar de heredar. Flexibilidad.  
**Nivel:** Opcional

---

## Bloque 4: Sistema práctico (tipo juego)

**🎮 Aplicar POO en un sistema vivo con reglas y estado**

Este bloque construye un sistema completo (puede ser un juego por turnos, un sistema de inventario, una simulación, etc.) usando POO de forma práctica.

### `20-diseño-sistema.md`
**Descripción:** Analizar el sistema a modelar. Identificar entidades, relaciones y comportamientos.  
**Nivel:** Esencial

### `21-entidades-base.md`
**Descripción:** Crear las clases principales del sistema (ej: Personaje, Objeto, Enemigo).  
**Nivel:** Esencial

### `22-sistema-combate.md`
**Descripción:** Implementar lógica de interacción entre entidades (ej: sistema de combate o intercambio).  
**Nivel:** Esencial

### `23-gestion-estado.md`
**Descripción:** Mantener el estado del sistema. Quién está vivo, qué objetos existen, turnos.  
**Nivel:** Esencial

### `24-validaciones-reglas.md`
**Descripción:** Reglas de negocio. Qué se puede y no se puede hacer en el sistema.  
**Nivel:** Esencial

### `25-api-sistema.md`
**Descripción:** Exponer el sistema como API REST. El frontend podrá interactuar con él.  
**Nivel:** Esencial

---

## Bloque 5: Autenticación y autorización

Proteger recursos y gestionar usuarios.

### `26-autenticacion-basica.md`
**Descripción:** Qué es autenticación. Registro y login conceptual.  
**Nivel:** Esencial

### `27-hashing-passwords.md`
**Descripción:** Nunca guardar contraseñas en texto plano. Bcrypt.  
**Nivel:** Esencial

### `28-jwt.md`
**Descripción:** JSON Web Tokens. Cómo funcionan, qué contienen, cuándo usarlos.  
**Nivel:** Esencial

### `29-proteccion-rutas.md`
**Descripción:** Middleware de autenticación. Solo usuarios autenticados acceden a ciertas rutas.  
**Nivel:** Esencial

### `30-autorizacion.md`
**Descripción:** Diferencia entre autenticación y autorización. Roles y permisos.  
**Nivel:** Recomendado

### `31-refresh-tokens.md`
**Descripción:** Tokens de larga duración. Renovar sesión sin volver a hacer login.  
**Nivel:** Opcional

---

## Bloque 6: Arquitectura backend

Estructurar código para que escale y sea mantenible.

### `32-separacion-capas.md`
**Descripción:** Rutas, controladores, servicios, modelos. Por qué separar responsabilidades.  
**Nivel:** Esencial

### `33-modelos-datos.md`
**Descripción:** Representar datos del negocio. Validaciones y lógica de dominio.  
**Nivel:** Esencial

### `34-servicios-logica-negocio.md`
**Descripción:** Lógica que no depende de HTTP. Reutilizable desde cualquier parte.  
**Nivel:** Recomendado

### `35-variables-entorno.md`
**Descripción:** Configuración sensible fuera del código. .env y process.env.  
**Nivel:** Esencial

### `36-manejo-archivos.md`
**Descripción:** Subir, guardar y servir archivos (imágenes, documentos).  
**Nivel:** Recomendado

### `37-logs-depuracion.md`
**Descripción:** Registrar eventos del servidor. Depurar en producción.  
**Nivel:** Recomendado

---

## Resumen de niveles

### Esencial (26 módulos)
Base para construir backends completos y funcionales.

### Recomendado (8 módulos)
Mejoran arquitectura y hacen el código más profesional.

### Opcional (3 módulos)
Profundización en temas específicos. Útiles según el proyecto.

---

## Orden de estudio sugerido

### Si vienes de `02-frontend`:
Sigue el orden numérico 01 → 37. La progresión está diseñada para no dar saltos.

**Parada importante:** El Bloque 4 (módulos 20-25) es donde todo cobra sentido. Ahí aplicas POO de forma práctica en un sistema completo antes de seguir.

### Si ya conoces backend básico pero no POO:
- Repasa rápido bloques 1-2 (módulos 01-12)
- Céntrate en el bloque 3 (POO aplicada, módulos 13-19)
- Trabaja el bloque 4 completo (sistema práctico, módulos 20-25)
- Avanza con autenticación y arquitectura

### Si conoces POO pero vienes de otro lenguaje:
- Revisa la sintaxis en tu entorno (Node.js, Python, etc.)
- Trabaja el bloque 4 (sistema práctico) para aplicar POO en backend
- Enfócate en arquitectura backend (bloque 6)

---

## Sobre POO en este bloque

**POO no se enseña como teoría académica.** Se introduce como herramienta para resolver un problema concreto: modelar sistemas con muchas entidades que interactúan.

El flujo es:
1. Entender qué es POO y para qué sirve (módulos 13-19)
2. Aplicarla en un sistema completo y realista (módulos 20-25)
3. Usarla en la arquitectura del backend (módulos 32-34)

Si POO no te convence después del módulo 25, al menos sabrás por qué existe y cuándo usarla.

---

## Relación con otros bloques

### Antes de este bloque:
- **`01-primer-contacto-digital`** → Primer contacto con código
- **`02-fundamentos`** → Base de programación
- **`03-frontend`** → Interfaces y consumo de APIs

### Después de este bloque:
- **`05-bases-de-datos`** → Persistencia de datos
- **`06-integracion-fullstack`** → Conectar frontend + backend + BD

---

## Proyecto recomendado

Al terminar este bloque, construye una API REST completa que:
- Tenga múltiples recursos relacionados
- Use POO para modelar lógica compleja (puede ser el sistema del Bloque 4 extendido)
- Incluya autenticación con JWT
- Esté bien estructurada en capas

Todavía no conectarás con base de datos (eso viene después). Usa arrays en memoria o archivos JSON.

---

## Validación de contenido

Antes de escribir estos módulos, valida que:
- [ ] POO se introduce como herramienta, no como dogma
- [ ] El sistema práctico (Bloque 4) es suficientemente complejo pero no abrumador
- [ ] La progresión de servidor básico → API → POO → sistema → arquitectura tiene sentido
- [ ] No se asume conocimiento previo de backend
- [ ] El bloque prepara para integración con frontend y base de datos

---

**Backend es el corazón del sistema. Aquí defines las reglas.**

**Última actualización:** 21 diciembre 2025

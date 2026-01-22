# Índice del Repositorio FullStack

Este documento define la arquitectura completa del repositorio: qué carpetas existen, qué contienen y cómo se relacionan.

---

## Estructura global

```
EDUCACION/FullStack/
│
├── README.md
├── MANIFIESTO_PEDAGOGICO.md
│
├── meta/
│   ├── principios.md
│   ├── checklist.md
│   ├── plantilla-modulo.md
│   └── guia-plantilla.md
│
├── 00-orientacion/
├── 01-digitalizacion-basica/
├── 02-frontend/
│   ├── fundamentos/        # HTML, CSS, JS vanilla
│   └── react/              # Framework moderno
├── 03-backend/
├── 04-bases-de-datos/
├── 05-integracion-fullstack/
├── 06-testing/
├── 07-arquitectura/
├── 08-despliegue/
├── 09-proyecto-integrador/
└── recursos/
```

---

## Descripción de carpetas

### `/` (Raíz)

**Contenido:**
- `README.md` — Punto de entrada del repositorio
- `MANIFIESTO_PEDAGOGICO.md` — Filosofía y compromiso pedagógico

**Propósito:**  
Primera impresión. Define qué es el repositorio y cómo usarlo.

---

### `/meta`

**Contenido:**
- Principios operativos
- Checklist de coherencia
- Plantilla canónica de módulo
- Guías de uso interno

**Propósito:**  
Material de trabajo para mantener coherencia pedagógica. No es contenido técnico para estudiantes, sino herramientas para quien escribe/mantiene el repo.

**Acceso:**  
Interno. Puede ser consultable, pero no forma parte del recorrido de aprendizaje.

---

### `/00-orientacion`

**Contenido:**
- Cómo usar este repositorio
- Qué es el desarrollo fullstack
- Mapa del recorrido completo
- Cómo configurar el entorno de trabajo
- Qué herramientas se usarán y por qué

**Propósito:**  
Contexto inicial antes de entrar en contenido técnico. Responde: ¿Qué voy a aprender? ¿Cómo está organizado? ¿Qué necesito instalar?

**Ejemplos de módulos:**
- `01-como-navegar-el-repositorio.md`
- `02-que-es-fullstack.md`
- `03-mapa-del-recorrido.md`
- `04-configuracion-entorno.md`
- `05-herramientas-esenciales.md`

**Nota:**  
No es programación todavía. Es orientación práctica.

---

### `/01-digitalizacion-basica`

**Contenido:**
- Conceptos digitales muy básicos
- Primer contacto con código
- Herramientas básicas del desarrollador
- Experimentación sin miedo

**Propósito:**  
Aterrizaje suave para quien nunca ha programado. Reducir vértigo, generar confianza y entender el terreno antes de profundizar.

**Ejemplos de módulos:**
- Qué es un ordenador y cómo piensa
- Qué es internet y desarrollo web
- Diferencia frontend vs backend (visual)
- Primer documento HTML
- Primer toque CSS y JavaScript
- Inspector del navegador
- Experimentar y romper código sin miedo

**Nota:**  
No es técnico profundo. Es preparación mental y práctica. Si ya tienes experiencia, salta directamente a `/02-frontend/fundamentos`.

---

### `/02-frontend`

**Contenido:**
- HTML, CSS, JavaScript en el navegador
- Frameworks y librerías frontend
- Interacción con el usuario
- Consumo de APIs desde el cliente

**Propósito:**  
Construir interfaces que el usuario ve y con las que interactúa.

**Organización:**

Se divide en dos subcarpetas:

#### `/fundamentos/` — HTML, CSS y JavaScript vanilla
Bases fundamentales del desarrollo web sin frameworks. Los cimientos.

**Bloques:**
- **HTML semántico** (5 módulos) - Estructura, formularios, tablas
- **CSS moderno** (7 módulos) - Selectores, box model, flexbox, grid, responsive
- **JavaScript básico** (9 módulos) - Variables, funciones, arrays, objetos, scope
- **DOM** (5 módulos) - Manipulación del DOM, eventos, formularios
- **JavaScript avanzado** (4 módulos) - Higher-order functions, métodos avanzados, destructuring
- **Asincronía y APIs** (8 módulos) - Promises, async/await, Fetch, APIs REST

**Total:** 39 módulos

#### `/react/` — Framework moderno
Una vez dominas vanilla JavaScript, aprendes React y el ecosistema profesional.

**Bloques:**
- Introducción a React y JSX
- Componentes, props y composición
- Estado y hooks
- Routing y navegación
- Formularios y validación
- Consumo de APIs
- Estado global (Context API / Redux)
- Optimización y deployment
- Gestión de estado global
- Consumo de APIs desde frontend

#### `05-estilos-avanzados/`
- CSS Modules / Styled Components
- Animaciones
- Preprocesadores (opcional)

**Nota:**  
El enfoque está en fundamentos primero (vanilla) y frameworks después. React sin bases sólidas es frustrante.

---

### `/03-backend`

**Contenido:**
- Servidor y lógica de negocio
- APIs y comunicación
- Autenticación y autorización
- Manejo de datos en servidor

**Propósito:**  
Construir la parte del sistema que el usuario no ve pero que hace que todo funcione.

**Ejemplos de bloques:**

#### `01-servidor-basico/`
- Qué es un servidor
- Node.js (u otro entorno)
- Primer servidor HTTP
- Rutas y controladores

#### `02-api-rest/`
- Diseño de APIs RESTful
- CRUD completo
- Validación de datos
- Middleware
- Manejo de errores en servidor

#### `03-autenticacion/`
- Sesiones vs tokens
- JWT
- Bcrypt y hashing
- Protección de rutas
- CORS

#### `04-arquitectura-backend/`
- Separación de capas (rutas, controladores, servicios)
- Modelos de datos
- Validadores
- Configuración y variables de entorno

**Nota:**  
Debe ser posible usar Express, Fastify, o cualquier framework. La estructura conceptual es lo importante.

---

### `/04-bases-de-datos`

**Contenido:**
- Persistencia de datos
- SQL y NoSQL
- Relaciones y modelado
- Consultas y optimización

**Propósito:**  
Guardar, recuperar y organizar datos de forma permanente.

**Ejemplos de bloques:**

#### `01-conceptos-bases-de-datos/`
- Qué es una base de datos
- SQL vs NoSQL
- Cuándo usar cada una
- Modelado de datos

#### `02-sql-fundamentos/`
- Tablas y columnas
- Tipos de datos
- INSERT, SELECT, UPDATE, DELETE
- WHERE, ORDER BY, LIMIT
- Relaciones (1:N, N:M)
- JOINs
- Índices básicos

#### `03-nosql-fundamentos/`
- Documentos y colecciones
- CRUD en NoSQL
- Referencias vs embebido
- Consultas básicas

#### `04-integracion-backend-db/`
- Conexión desde Node.js (u otro)
- ORMs / ODMs
- Migraciones
- Seeds

**Nota:**  
Debe cubrir tanto SQL (ej: PostgreSQL) como NoSQL (ej: MongoDB) sin ser dependiente de uno.

---

### `/05-integracion-fullstack`

**Contenido:**
- Conectar frontend con backend
- Flujo completo de datos
- Autenticación end-to-end
- Subida de archivos
- Websockets / tiempo real (opcional)

**Propósito:**  
Unir todas las piezas en un sistema completo y funcional.

**Ejemplos de bloques:**

#### `01-comunicacion-frontend-backend/`
- Fetch desde React a API REST
- Manejo de respuestas y errores
- Loading states
- Gestión de tokens en cliente

#### `02-autenticacion-completa/`
- Login y registro (frontend + backend)
- Protección de rutas en ambos lados
- Refresh tokens
- Logout

#### `03-crud-completo/`
- Flujo de crear, leer, actualizar, eliminar
- Formularios en frontend
- Validación en ambos lados
- Feedback de usuario

#### `04-casos-especiales/`
- Subida de archivos
- Paginación
- Búsqueda y filtros
- Tiempo real (WebSockets, opcional)

**Nota:**  
Aquí es donde se consolida todo. Debe haber proyectos pequeños que integren frontend, backend y base de datos.

---

### `/06-testing`

**Contenido:**
- Cómo y por qué testear
- Tipos de tests
- Testing en frontend y backend
- TDD básico

**Propósito:**  
Asegurar que el código funciona y seguir funcionando al hacer cambios.

**Ejemplos de bloques:**

#### `01-introduccion-testing/`
- Por qué testear
- Tipos de tests (unitarios, integración, e2e)
- Anatomía de un test

#### `02-testing-backend/`
- Tests de funciones puras
- Tests de rutas/APIs
- Mocking de base de datos
- Tests de integración

#### `03-testing-frontend/`
- Tests de componentes
- Tests de interacción
- Tests de integración con API
- Mocking de peticiones

#### `04-tdd-basico/`
- Flujo red-green-refactor
- Ejercicios con TDD
- Cuándo aplicar TDD

**Nota:**  
No debe ser excesivamente teórico. Debe ser práctico y útil. Puede ser opcional marcar como tal, pero es recomendable.

---

### `/07-arquitectura`

**Contenido:**
- Patrones de diseño aplicados
- Buenas prácticas
- Clean code
- Escalabilidad básica

**Propósito:**  
Escribir código mantenible, legible y preparado para crecer.

**Ejemplos de bloques:**

#### `01-principios-solid/` (opcional avanzado)
- Single Responsibility
- Open/Closed
- Liskov Substitution
- Interface Segregation
- Dependency Inversion

#### `02-patrones-comunes/`
- Repository pattern
- Factory pattern
- Singleton
- Middleware pattern

#### `03-clean-code/`
- Nombres descriptivos
- Funciones pequeñas
- Evitar duplicación
- Comentarios útiles vs innecesarios
- Formateo y estilo

#### `04-refactoring/`
- Cuándo refactorizar
- Técnicas básicas
- Refactoring seguro con tests

**Nota:**  
Esta carpeta puede ser más avanzada. Marcar claramente qué es opcional.

---

### `/08-despliegue`

**Contenido:**
- Cómo llevar el código a producción
- Servidores y hosting
- Variables de entorno
- CI/CD básico

**Propósito:**  
Hacer que el proyecto sea accesible en internet, no solo en local.

**Ejemplos de bloques:**

#### `01-conceptos-despliegue/`
- Desarrollo vs producción
- Variables de entorno
- Build process
- Configuración de producción

#### `02-despliegue-frontend/`
- Hosting estático (Vercel, Netlify, etc.)
- Build de producción
- Dominios y DNS básico

#### `03-despliegue-backend/`
- Hosting de servidor (Render, Railway, etc.)
- Configuración de entorno
- Logs y monitoreo básico

#### `04-despliegue-base-de-datos/`
- Bases de datos en la nube
- Conexión segura
- Backups básicos

#### `05-ci-cd-basico/` (opcional)
- Qué es CI/CD
- GitHub Actions básico
- Tests automáticos antes de desplegar

**Nota:**  
Debe ser práctico y actualizado. Las plataformas cambian, pero los conceptos no.

---

### `/09-proyecto-integrador`

**Contenido:**
- Proyecto completo que usa todo lo aprendido
- Especificación y requisitos
- Guía de desarrollo paso a paso
- Versiones progresivas del proyecto

**Propósito:**  
Aplicar todos los conceptos en un proyecto real, desde cero hasta producción.

**Ejemplos de contenido:**

#### `00-especificacion.md`
- Qué se va a construir
- Funcionalidades requeridas
- Stack tecnológico sugerido

#### `01-planificacion/`
- Diseño de base de datos
- Arquitectura de la API
- Wireframes básicos

#### `02-iteraciones/`
- Iteración 1: Configuración y estructura
- Iteración 2: Backend básico
- Iteración 3: Frontend básico
- Iteración 4: Integración
- Iteración 5: Testing
- Iteración 6: Despliegue
- Iteración 7: Mejoras y refactoring

#### `03-extensiones-opcionales/`
- Funcionalidades adicionales
- Mejoras de UX
- Optimizaciones

**Nota:**  
Este proyecto debe ser suficientemente complejo para ser realista, pero suficientemente guiado para no abrumar.

---

### `/recursos`

**Contenido:**
- Enlaces externos útiles
- Documentación oficial
- Artículos recomendados
- Comunidades y foros
- Glosario de términos

**Propósito:**  
Complementar el material del repo con recursos externos de calidad.

**Ejemplos de archivos:**

- `01-enlaces-utiles.md`
- `02-documentacion-oficial.md`
- `03-glosario.md`
- `04-comunidades.md`
- `05-lecturas-recomendadas.md`

**Nota:**  
Solo enlaces de alta calidad. Evitar listas infinitas de recursos que nadie usa.

---

## Recorrido sugerido

### Para quien empieza de cero:

1. **00-orientacion** → Entender cómo usar el repo
2. **01-fundamentos** → Base de programación
3. **02-frontend** → Interfaces de usuario
4. **03-backend** → Lógica de servidor
5. **04-bases-de-datos** → Persistencia de datos
6. **05-integracion-fullstack** → Conectar todo
7. **06-testing** → Asegurar calidad
8. **08-despliegue** → Publicar en internet
9. **09-proyecto-integrador** → Proyecto completo

### Para quien ya tiene base:

- Saltar a los bloques específicos que necesites
- Usar las secciones "Conexiones" de cada módulo para navegar
- Consultar `/recursos/glosario.md` si encuentras términos desconocidos

### Para quien vuelve después de tiempo:

- Usar el README de cada carpeta como índice rápido
- Consultar módulos específicos sin seguir orden
- Revisar la sección "Resumen rápido" de cada módulo

---

## Principios de navegación

1. **No hay un único camino correcto**  
   Puedes seguir el orden sugerido o saltar a lo que necesites.

2. **Las conexiones son explícitas**  
   Cada módulo indica qué conceptos previos necesitas y dónde encontrarlos.

3. **Consultar no es retroceder**  
   Volver a un módulo anterior es parte normal del proceso.

4. **El material crece contigo**  
   Cuando vuelvas con más experiencia, verás detalles que antes no notaste.

---

## Mantenimiento del índice

Este índice es estable, pero no inamovible. Si se detecta:
- Solapamiento entre carpetas
- Vacíos de contenido
- Necesidad de reorganización

Se puede actualizar, **documentando por qué** en este mismo archivo.

---

**Última actualización:** 22 enero 2026  
**Versión del índice:** 2.0

### Registro de cambios

#### v2.0 - 22 enero 2026
- ✅ Completado bloque 00-orientacion (6 módulos)
- ✅ Completado bloque 01-digitalizacion-basica (8 módulos)
- ✅ Completado bloque 03-backend (42 módulos: servidor básico, APIs REST, POO, sistema práctico, autenticación, arquitectura, Django)
- ✅ Completado bloque 04-bases-de-datos (20 módulos: fundamentos, SQL, PostgreSQL)
- ✅ Completado bloque 05-integracion-fullstack (8 módulos)
- ✅ Completado bloque 06-testing (4 módulos)
- ✅ Completado bloque 07-arquitectura (3 módulos)
- ✅ Completado bloque 08-despliegue (2 módulos)
- ✅ Completado bloque 09-proyecto-integrador (3 módulos)
- 🔄 Frontend React en progreso (módulos 01-23 completados)
- ✅ Frontend fundamentos completo (39 módulos)

#### v1.0 - 21 diciembre 2025
- Estructura inicial del repositorio definida

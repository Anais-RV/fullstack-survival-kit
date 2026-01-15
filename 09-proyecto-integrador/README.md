# Proyecto Integrador

Has aprendido frontend, backend y bases de datos por separado. Ahora vas a conectarlo todo.

**Este proyecto consolida todo el repositorio en una aplicación fullstack completa.**

---

## Sobre este proyecto

Este no es un proyecto "heroico" ni infinito. Es un proyecto guiado, iterativo y completable.

El objetivo es:
- Integrar frontend, backend y base de datos
- Aplicar conceptos de todos los bloques anteriores
- Construir algo funcional de principio a fin
- Sentir que eres capaz de crear aplicaciones completas

**El proyecto se construye en 7 iteraciones claras.** Cada iteración es completable y funcional por sí misma.

---

## Estructura del proyecto

```
proyecto-integrador/
│
├── docs/
│   ├── 00-especificacion.md
│   ├── 01-requisitos.md
│   └── 02-modelo-datos.md
│
├── iteracion-1-configuracion/
├── iteracion-2-backend-basico/
├── iteracion-3-frontend-basico/
├── iteracion-4-integracion/
├── iteracion-5-autenticacion/
├── iteracion-6-funcionalidades-completas/
└── iteracion-7-mejoras-despliegue/
```

---

## Dominio del proyecto

**El dominio específico se definirá más adelante**, pero debe cumplir:

- Tener al menos 3 entidades relacionadas (ej: usuarios, publicaciones, comentarios)
- Requerir autenticación
- Permitir crear, leer, actualizar y eliminar datos (CRUD)
- Ser suficientemente realista pero no excesivamente complejo

Ejemplos posibles:
- Sistema de gestión de tareas con equipos
- Plataforma de recetas con valoraciones
- Sistema de biblioteca personal
- Gestor de inventario simple

---

## Iteración 1: Configuración y estructura

### Objetivo
Preparar el entorno de desarrollo y la estructura del proyecto.

### Qué se hace
- Inicializar repositorio Git
- Configurar estructura de carpetas (frontend / backend)
- Instalar dependencias básicas
- Configurar variables de entorno
- Probar que todo arranca correctamente

### Qué se consolida
- **`00-orientacion`** → Configuración de entorno
- **`02-fundamentos`** → Estructura de proyecto

### Resultado esperado
Proyecto vacío pero funcional. Frontend y backend arrancan sin errores.

---

## Iteración 2: Backend básico

### Objetivo
Crear la API REST con las rutas principales.

### Qué se hace
- Configurar servidor HTTP
- Definir rutas para los recursos principales
- Implementar CRUD básico (sin base de datos todavía, usar arrays en memoria)
- Añadir validación de datos
- Configurar CORS

### Qué se consolida
- **`03-backend`** → Servidor, rutas, API REST, validación
- **POO** → Modelar entidades con clases

### Resultado esperado
API funcional que responde a peticiones. Se puede probar con Postman o cliente HTTP.

---

## Iteración 3: Frontend básico

### Objetivo
Crear la interfaz de usuario con las vistas principales.

### Qué se hace
- Configurar React (o framework elegido)
- Crear componentes principales
- Implementar enrutamiento entre vistas
- Diseñar layout responsive básico
- Mostrar datos de prueba (hardcoded, sin conectar con backend todavía)

### Qué se consolida
- **`03-frontend`** → React, componentes, routing, estilos

### Resultado esperado
Interfaz navegable con datos de prueba. Se ve bien en diferentes pantallas.

---

## Iteración 4: Integración frontend-backend

### Objetivo
Conectar frontend con backend. Hacer que los datos fluyan.

### Qué se hace
- Implementar peticiones HTTP desde el frontend
- Mostrar datos reales del backend
- Manejar estados de loading y errores
- Implementar formularios que envíen datos al backend
- Sincronizar estado del frontend con respuestas del servidor

### Qué se consolida
- **`06-integracion-fullstack`** → Comunicación entre capas
- **`03-frontend`** → useEffect, fetch, manejo de estado

### Resultado esperado
Aplicación funcional que crea, lee, actualiza y elimina datos. Frontend y backend se comunican correctamente.

---

## Iteración 5: Autenticación end-to-end

### Objetivo
Proteger recursos y gestionar usuarios.

### Qué se hace
- Implementar registro y login en el backend
- Hash de contraseñas con bcrypt
- Generar y validar JWT
- Proteger rutas en el backend
- Implementar login/registro en el frontend
- Guardar token en el cliente
- Proteger rutas en el frontend
- Implementar logout

### Qué se consolida
- **`03-backend`** → Autenticación, JWT, middleware
- **`03-frontend`** → Formularios, Context API (estado global)
- **`06-integracion-fullstack`** → Flujo de autenticación completo

### Resultado esperado
Sistema de autenticación funcional. Solo usuarios autenticados acceden a ciertas partes de la aplicación.

---

## Iteración 6: Funcionalidades completas

### Objetivo
Añadir funcionalidades específicas del dominio y pulir la aplicación.

### Qué se hace
- Conectar con base de datos real (MongoDB o PostgreSQL)
- Implementar relaciones entre entidades
- Añadir búsqueda y filtros
- Implementar paginación si es necesario
- Validación completa en frontend y backend
- Mensajes de feedback claros para el usuario

### Qué se consolida
- **`05-bases-de-datos`** → Conexión, consultas, relaciones
- **`03-backend`** → Arquitectura en capas, servicios
- **`03-frontend`** → Formularios complejos, UX

### Resultado esperado
Aplicación completa con base de datos real. Funciona como sistema de producción (aunque local).

---

## Iteración 7: Mejoras y despliegue

### Objetivo
Pulir detalles, añadir tests básicos y desplegar.

### Qué se hace
- Añadir tests básicos (frontend y backend)
- Mejorar estilos y UX
- Optimizar rendimiento
- Configurar build de producción
- Desplegar frontend (Vercel, Netlify)
- Desplegar backend (Render, Railway)
- Desplegar base de datos (MongoDB Atlas, Supabase)
- Configurar variables de entorno en producción

### Qué se consolida
- **`07-testing`** → Tests unitarios y de integración
- **`09-despliegue`** → Configuración de producción, hosting

### Resultado esperado
Aplicación pública en internet. Funcional, accesible desde cualquier lugar.

---

## Duración estimada

- **Iteración 1:** 1-2 días
- **Iteración 2:** 3-5 días
- **Iteración 3:** 3-5 días
- **Iteración 4:** 2-4 días
- **Iteración 5:** 3-4 días
- **Iteración 6:** 4-6 días
- **Iteración 7:** 2-4 días

**Total:** 3-4 semanas trabajando de forma constante.

Esto es orientativo. Cada persona avanza a su ritmo.

---

## Principios pedagógicos aplicados

### Iteraciones completables
Cada iteración termina con algo funcional. No hay "mitades" del proyecto.

### Progresión lógica
Primero backend, luego frontend, luego integración, luego autenticación. Cada pieza sobre la anterior.

### Consolidación de conceptos
Cada iteración aplica conceptos de bloques específicos del repositorio. No se inventa nada nuevo aquí.

### Proyecto realista pero guiado
No es un tutorial paso a paso, pero sí tiene estructura clara y expectativas definidas.

---

## Relación con otros bloques

Este proyecto usa:
- **`00-orientacion`** → Configuración inicial
- **`02-fundamentos`** → Base de todo
- **`03-frontend`** → Interfaz completa
- **`03-backend`** → API, POO, autenticación
- **`05-bases-de-datos`** → Persistencia
- **`06-integracion-fullstack`** → Conexión entre capas
- **`07-testing`** → Calidad del código
- **`09-despliegue`** → Publicación

Es la suma de todo el repositorio.

---

## Qué hacer si te atascas

1. **Revisa el módulo correspondiente** en el bloque que estás aplicando
2. **Consulta los errores** en la sección "Errores comunes" del módulo
3. **Simplifica** la iteración si es demasiado compleja
4. **Pide ayuda** o busca ejemplos similares
5. **Avanza** aunque no sea perfecto. Puedes volver después

**El objetivo es terminar, no que sea perfecto.**

---

## Después de este proyecto

Cuando termines, tendrás:
- Una aplicación fullstack completa en tu portafolio
- Experiencia integrando todas las piezas
- Conocimiento práctico de todo el stack
- Confianza para empezar proyectos propios

Puedes:
- Extender este proyecto con nuevas funcionalidades
- Construir un proyecto similar desde cero (ahora sin guía)
- Explorar el bloque `08-arquitectura` para mejorar tu código
- Especializarte en frontend o backend según tus intereses

---

## Validación antes de empezar

Antes de definir el dominio concreto y escribir las guías, valida que:
- [ ] Las 7 iteraciones tienen sentido como progresión
- [ ] Cada iteración es completable y autónoma
- [ ] El proyecto no es demasiado ambicioso
- [ ] Cubre los bloques principales del repositorio
- [ ] El tiempo estimado es razonable

---

**Este proyecto demuestra que puedes construir aplicaciones completas. No es el final, es el comienzo.**

**Última actualización:** 21 diciembre 2025

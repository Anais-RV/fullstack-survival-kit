# Backend con Python

> **Del servidor vanilla a Django profesional**

---

## ¿Qué aprenderás aquí?

Este módulo te enseña a crear el **backend** de aplicaciones web desde cero: primero construyendo servidores con Python puro para entender los fundamentos, y luego utilizando Django para desarrollo profesional.

**Tecnologías principales:**
- **Python vanilla** - http.server, manejo directo de HTTP
- **MVC manual** - Arquitectura sin frameworks
- **Django** - Framework completo y profesional
- **REST APIs** - Arquitectura de comunicación
- **JWT** - Autenticación con tokens
- **bcrypt** - Hashing de contraseñas

---

## Estructura del módulo

### Bloque 1: Servidor básico
**[📁 bloque-01-servidor-basico/](./bloque-01-servidor-basico/)**

Fundamentos de HTTP y servidores con Python vanilla.

1. [Introducción a HTTP](./bloque-01-servidor-basico/01-introduccion-http.md) - Protocolo web, peticiones/respuestas
2. [Primer servidor Python](./bloque-01-servidor-basico/02-primer-servidor-python.md) - http.server, hola mundo
3. [Procesamiento de requests](./bloque-01-servidor-basico/03-procesamiento-requests.md) - Manejo manual de HTTP
4. [Rutas y enrutamiento](./bloque-01-servidor-basico/04-rutas-enrutamiento.md) - Sistema de rutas desde cero
5. [Debugging y testing](./bloque-01-servidor-basico/05-debugging-testing.md) - Herramientas de desarrollo

### Bloque 2: APIs REST vanilla
**[📁 bloque-02-apis-rest/](./bloque-02-apis-rest/)**

Arquitectura REST con Python puro.

6. [Introducción a REST](./bloque-02-apis-rest/06-introduccion-rest.md) - Arquitectura, principios
7. [Métodos HTTP](./bloque-02-apis-rest/07-metodos-http.md) - GET, POST, PUT, DELETE
8. [Sistema de enrutamiento](./bloque-02-apis-rest/08-sistema-enrutamiento.md) - Router personalizado
9. [JSON y serialización](./bloque-02-apis-rest/09-json-serializacion.md) - Manejo de datos
10. [Status codes](./bloque-02-apis-rest/10-status-codes.md) - Códigos de respuesta HTTP
11. [Manejo de errores](./bloque-02-apis-rest/11-errores.md) - try/except, error handlers
12. [Testing con Postman](./bloque-02-apis-rest/12-testing-postman.md) - Pruebas de APIs

### Bloque 3: POO aplicada
**[📁 bloque-03-poo-aplicada/](./bloque-03-poo-aplicada/)**

Programación orientada a objetos para backend profesional.

13. [Clases y objetos](./bloque-03-poo-aplicada/13-clases-objetos.md) - Fundamentos de POO
14. [Herencia](./bloque-03-poo-aplicada/14-herencia.md) - Jerarquías de clases
15. [Composición](./bloque-03-poo-aplicada/15-composicion.md) - Relaciones entre objetos
16. [Modelado de dominio](./bloque-03-poo-aplicada/16-modelado-dominio.md) - Diseño de entidades
17. [Patrones de diseño](./bloque-03-poo-aplicada/17-patrones-diseno.md) - Singleton, Factory, Repository
18. [Principios SOLID](./bloque-03-poo-aplicada/18-solid.md) - Código mantenible

### Bloque 4: Sistema práctico
**[📁 bloque-04-sistema-practico/](./bloque-04-sistema-practico/)**

Proyecto completo aplicando POO: juego RPG con API REST vanilla.

19. [Proyecto juego RPG](./bloque-04-sistema-practico/19-proyecto-juego-rpg.md) - Introducción y arquitectura
20. [Personajes y entidades](./bloque-04-sistema-practico/20-personajes-entidades.md) - Sistema de personajes
21. [Sistema de combate](./bloque-04-sistema-practico/21-sistema-combate.md) - Mecánicas de batalla
22. [API REST del juego](./bloque-04-sistema-practico/22-api-rest-juego.md) - REST API completa vanilla

### Bloque 5: Autenticación
**[📁 bloque-05-autenticacion/](./bloque-05-autenticacion/)**

Seguridad, autenticación y control de acceso.

23. [Introducción a autenticación](./bloque-05-autenticacion/23-introduccion-autenticacion.md) - Conceptos básicos, JWT
24. [JWT en profundidad](./bloque-05-autenticacion/24-jwt-profundidad.md) - Tokens, claims, refresh tokens
25. [bcrypt y hashing](./bloque-05-autenticacion/25-bcrypt-hashing.md) - Protección de contraseñas
26. [Protección de rutas](./bloque-05-autenticacion/26-proteccion-rutas.md) - Roles, permisos, decoradores
27. [CORS y seguridad](./bloque-05-autenticacion/27-cors-seguridad.md) - Seguridad web, headers, rate limiting
28. [Roles y sesiones](./bloque-05-autenticacion/28-roles-sesiones.md) - Sistema completo de autenticación

### Bloque 6: Arquitectura
**[📁 bloque-06-arquitectura/](./bloque-06-arquitectura/)**

Patrones de arquitectura para aplicaciones escalables y mantenibles.

29. [Patrón MVC](./bloque-06-arquitectura/29-patron-mvc.md) - Model-View-Controller, separación de responsabilidades
30. [Service Layer](./bloque-06-arquitectura/30-service-layer.md) - Capa de servicios, lógica de negocio
31. [Repository Pattern](./bloque-06-arquitectura/31-repository-pattern.md) - Abstracción de datos, Unit of Work
32. [Validadores y DTOs](./bloque-06-arquitectura/32-validadores-dtos.md) - Validación con Pydantic, Marshmallow
33. [Variables de entorno](./bloque-06-arquitectura/33-variables-entorno.md) - python-dotenv, configuración por entorno
34. [Sistema de configuración](./bloque-06-arquitectura/34-sistema-configuracion.md) - ConfigManager, feature flags

### Bloque 7: Django Framework
**[📁 bloque-07-django/](./bloque-07-django/)**

Framework profesional para desarrollo rápido y escalable.

35. [Introducción a Django](./bloque-07-django/35-introduccion-django.md) - Por qué Django, instalación, proyecto
36. [Models y ORM](./bloque-07-django/36-models-orm.md) - Modelos, migraciones, QuerySets
37. [Views y URLs](./bloque-07-django/37-views-urls.md) - Vistas basadas en funciones y clases
38. [Django REST Framework](./bloque-07-django/38-django-rest-framework.md) - Serializers, ViewSets, Routers
39. [Autenticación en Django](./bloque-07-django/39-autenticacion-django.md) - User model, permisos, JWT
40. [Admin y gestión](./bloque-07-django/40-admin-gestion.md) - Panel de administración personalizado
41. [Testing en Django](./bloque-07-django/41-testing-django.md) - Tests unitarios e integración
42. [Deployment Django](./bloque-07-django/42-deployment-django.md) - Configuración para producción

---

## Requisitos previos

- Python 3.8+ instalado
- Editor de código (VS Code recomendado)
- Conocimientos básicos de programación
- Git (para control de versiones)

---

## Instalación

```powershell
# Crear entorno virtual
python -m venv venv

# Activar entorno
.\venv\Scripts\Activate

# Para Bloques 1-6 (Python vanilla)
pip install pyjwt bcrypt python-dotenv pydantic

# Para Bloque 7 (Django)
pip install django djangorestframework djangorestframework-simplejwt django-cors-headers
```

---

## Proyectos del módulo

### 1. API REST de tareas (Bloques 1-2) - Python Vanilla
- Servidor HTTP desde cero
- CRUD completo sin frameworks
- Router personalizado
- Manejo de errores

### 2. Juego RPG con API (Bloques 3-4) - Python Vanilla
- Sistema de personajes con POO
- Combate por turnos
- Gestión de items e inventario
- API REST completa vanilla

### 3. Sistema de autenticación (Bloque 5) - Python Vanilla
- Registro y login sin frameworks
- JWT tokens manual
- Protección de rutas custom
- Roles y permisos

### 4. Arquitectura escalable (Bloque 6) - Python Vanilla
- Patrón MVC completo manual
- Service Layer desde cero
- Repository Pattern custom
- Validación con Pydantic
- Sistema de configuración robusto

### 5. Aplicación completa con Django (Bloque 7)
- Django REST Framework
- ORM y migraciones
- Autenticación JWT
- Panel de administración
- Testing completo

---

## Progresión recomendada

1. **Bloques 1-2**: Fundamentos con Python vanilla (principiante)
2. **Bloques 3-4**: POO y proyecto práctico vanilla (intermedio)
3. **Bloque 5**: Autenticación sin frameworks (avanzado)
4. **Bloque 6**: Arquitectura MVC manual (avanzado)
5. **Bloque 7**: Django framework profesional (experto)

**Filosofía pedagógica**: Primero construyes TODO desde cero para entender cómo funciona internamente. Luego usas Django y apreciarás la magia que hace por ti.

---

## Recursos adicionales

- **[Python http.server](https://docs.python.org/3/library/http.server.html)** - Documentación oficial
- **[REST API Tutorial](https://restfulapi.net/)** - Guía de REST
- **[Django Documentation](https://docs.djangoproject.com/)** - Documentación oficial Django
- **[Django REST Framework](https://www.django-rest-framework.org/)** - DRF Docs
- **[JWT.io](https://jwt.io/)** - Debugger de JWT
- **[OWASP](https://owasp.org/)** - Seguridad web

---

## Próximos pasos

Después de completar este módulo, continuarás con:
- **[04-bases-de-datos](../04-bases-de-datos/)** - SQL, ORMs, migraciones
- **[05-integracion-fullstack](../05-integracion-fullstack/)** - Conectar frontend y backend
- **[06-testing](../06-testing/)** - Pruebas automatizadas

¡Comencemos construyendo tu primer servidor desde cero! 🚀

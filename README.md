# FullStack Survival Kit

[![LinkedIn](https://img.shields.io/badge/LinkedIn-Anaïs_Rodríguez-0077B5?style=flat&logo=linkedin&logoColor=white)](https://www.linkedin.com/in/anaisvillanueva/)
[![GitHub](https://img.shields.io/badge/GitHub-Anais--RV-181717?style=flat&logo=github&logoColor=white)](https://github.com/Anais-RV/fullstack-survival-kit)
[![License](https://img.shields.io/badge/License-CC_BY--NC--SA_4.0-lightgrey.svg)](LICENSE)

> **Nota:** Material en revisión continua. Estoy matizando explicaciones, puliendo ejemplos y ajustando detalles según uso real en formaciones. El contenido principal está estable, pero pueden aparecer mejoras puntuales.

---

## Qué encontrarás aquí

Este es un **curriculum completo de desarrollo fullstack** que escribí después de años viendo dónde se atascan las personas cuando aprenden a programar. No es una colección de tutoriales ni un curso express. Es material pensado para que entiendas, no solo para que copies.

Cubre desde HTML hasta arquitecturas fullstack con Django y React. Todo el contenido está cerrado y probado. Lo he usado, ajustado y mejorado en formación real. No prometo que aprendas en X semanas. Prometo que si trabajas el material con cabeza, vas a entender lo que estás haciendo.

**Proyecto personal** creado por **[Anaïs Rodríguez Villanueva](https://www.linkedin.com/in/anaisvillanueva/)** en tiempo libre, con licencia abierta para que cualquiera pueda aprender sin barreras económicas.

---

## Para quién está pensado

**Te va a servir si:**
- Empiezas desde cero y necesitas una ruta clara sin perderte en tutoriales infinitos
- Vienes de otro sector y quieres entender los fundamentos de verdad, no solo usar frameworks
- Ya programas un poco pero tienes lagunas grandes y te cuesta integrar frontend, backend y base de datos
- Eres docente y buscas material estructurado con progresión pedagógica pensada

**No te va a servir si:**
- Buscas aprender React en 3 días sin tocar JavaScript
- Quieres un bootcamp ultra-rápido sin profundizar
- No tienes tiempo para hacer ejercicios y solo quieres leer teoría
- Esperas que alguien te resuelva cada duda (aquí hay material, no tutorías)

Este contenido asume que tienes autonomía, ganas de pensar y tolerancia al error. Si te frustras cuando algo no funciona a la primera, la programación no va a ser fácil.

---

## Antes de empezar: lee el manifiesto

El [Manifiesto Pedagógico](./MANIFIESTO_PEDAGOGICO.md) explica la filosofía detrás de este material: cómo está diseñado, por qué está en este orden, cómo entiendo el error y el ritmo de aprendizaje.

No es un documento corporativo. Es honesto. Léelo antes de sumergirte en el contenido técnico, te va a ayudar a entender el enfoque.

---

## Estructura del curriculum

El material está organizado en **bloques progresivos**. Puedes saltarte módulos si ya los dominas, pero la progresión está pensada: cada bloque construye sobre los anteriores.

### Stack tecnológico

**Frontend:** HTML5, CSS3, JavaScript ES6+, React 19, Vite 6, TypeScript  
**Backend:** Python 3.12+, Django 5.x, Django REST Framework  
**Base de datos:** PostgreSQL 16+  
**Herramientas:** Git, Docker, pytest, Playwright

Este stack no es arbitrario. Es el que veo en ofertas reales de trabajo y el que mejor equilibra curva de aprendizaje con empleabilidad.

### Bloques de contenido

| Bloque | Contenido | Módulos |
|--------|-----------|---------|
| **00-orientacion** | Configurar entorno, herramientas, Git básico | 6 |
| **01-digitalizacion-basica** | Conceptos digitales, primer contacto con código | 8 |
| **02-frontend** | HTML, CSS, JavaScript, DOM, asincronía, React | 81 |
| **03-backend** | HTTP, APIs REST, POO, Django, autenticación | 42 |
| **04-bases-de-datos** | SQL, PostgreSQL, modelado, optimización | 20 |
| **05-integracion-fullstack** | Conectar frontend + backend + BD | 8 |
| **06-testing** | Jest, pytest, Playwright, TDD | 4 |
| **07-arquitectura** | Clean Architecture, SOLID, patrones | 3 |
| **08-despliegue** | Docker, CI/CD, producción | 2 |
| **09-proyecto-integrador** | Ecommerce completo, buenas prácticas | 3 |

**Total:** ~177 módulos

Cada módulo incluye teoría, ejemplos ejecutables y ejercicios. No es lectura pasiva.

---

## Cómo recorrer el material sin perderte

### Si empiezas desde cero

Sigue el orden: `00-orientacion → 01-digitalizacion-basica → 02-frontend/fundamentos → 02-frontend/react → ...`

**No te saltes los fundamentos de JavaScript.** He visto demasiada gente intentando aprender React sin entender funciones, arrays u objetos. No funciona. Te frustras y abandonas.

### Si ya tienes experiencia

Ve al bloque que necesites. Cada módulo indica qué conocimientos previos asume. Si algo no cuadra, vuelve atrás sin drama. Consultar material anterior no es retroceder, es sentar bases.

### Ritmo recomendado

No hay prisa. Un módulo al día si puedes. Dos o tres por semana si no. Lo importante no es la velocidad, es que ejecutes los ejercicios y entiendas lo que pasa.

**El código no se aprende leyendo. Se aprende escribiendo, rompiendo cosas y arreglándolas.**

### Navegación

- Cada módulo tiene enlaces al anterior y al siguiente
- El [INDICE.md](./INDICE.md) tiene la estructura completa con descripciones
- Los README de cada carpeta funcionan como puntos de entrada

---

## Decisiones pedagógicas clave

### Por qué vanilla antes que frameworks

Enseño HTML, CSS y JavaScript puro antes de React porque he visto qué pasa cuando alguien aprende frameworks sin bases: en cuanto algo falla, no sabe qué buscar. No entiende qué es DOM, qué es un evento, qué es asincronía.

React es fantástico. Pero sin JavaScript, React es magia incomprensible.

### Por qué Python y Django en backend

Python tiene sintaxis clara y Django es un framework completo que enseña arquitectura web de verdad. No es el stack más cool, pero es sólido, empleable y didáctico.

Si después prefieres Node, Go o Rust, adelante. Los conceptos se transfieren.

### Por qué PostgreSQL

Porque las bases de datos relacionales siguen siendo el estándar en la industria. SQL es una habilidad que usarás durante décadas. NoSQL también importa, pero SQL primero.

### Por qué tanto énfasis en fundamentos

Porque los frameworks cambian cada dos años. Los fundamentos no.

Si entiendes HTTP, APIs REST, arquitectura cliente-servidor, autenticación con tokens y manejo de estado, puedes aprender cualquier framework en semanas.

Si solo memorizas sintaxis de React sin entender nada de lo anterior, cada proyecto nuevo es un infierno.

---

## Estado actual del curriculum

**El contenido está cerrado y estable.** Todos los bloques principales están escritos, revisados y probados. React tiene ~23 módulos activos (componentes, estado, efectos, formularios) y se irá completando según vea necesidad en formaciones reales.

Esto no significa que sea perfecto. Encontrarás typos, ejemplos que se pueden mejorar, explicaciones que se pueden aclarar. Si ves algo, abre un issue o manda un PR. El material está vivo, pero no "en construcción eterna".

**No esperes actualizaciones semanales.** Actualizo cuando detecto lagunas reales o cuando cambian tecnologías de forma significativa, no por moda.

---

## Sobre mí

### Anaïs Rodríguez Villanueva

[![LinkedIn](https://img.shields.io/badge/LinkedIn-anaisvillanueva-0077B5?style=flat&logo=linkedin&logoColor=white)](https://www.linkedin.com/in/anaisvillanueva/)
[![GitHub](https://img.shields.io/badge/GitHub-Anais--RV-181717?style=flat&logo=github&logoColor=white)](https://github.com/Anais-RV)

Desarrolladora fullstack, formadora técnica. Vengo del mundo del marketing digital, pasé por un bootcamp, aprendí a base de frustrarme y rehacer proyectos, y ahora formo a otras personas.

**Formación:**
- Técnico Superior en Desarrollo de Aplicaciones Informáticas y Analista
- Máster en Inteligencia Artificial y Big Data
- Bootcamp Desarrollo Web Fullstack

**Experiencia:**
- Desarrollo de aplicaciones web (frontend y backend)
- Docencia y formación tecnológica
- Marketing online (vida anterior)

### Filosofía de enseñanza

Aprender a programar no es memorizar sintaxis. Es aprender a pensar, equivocarse, ajustar y volver a intentar. Conozco bien el vértigo que da ver un error incomprensible o el bloqueo de "no sé ni qué buscar en Google".

Por eso este material prioriza:

- **Aprender a aprender**, no a seguir tutoriales
- **Autonomía real**: mis alumnos incendian proyectos y yo les enseño a ser bomberos
- **Pedagogía activa**: problemas reales, decisiones técnicas reales
- **Explicaciones claras**, sin asumir que ya sabes términos que nadie te explicó
- **Buenas prácticas desde el inicio**: código legible, control de versiones, criterio técnico

Este repositorio no pretende que lo sepas todo.  
**Pretende que sepas volver, entender qué haces y seguir creciendo por tu cuenta.**

---

## Licencia

**Creative Commons Attribution-NonCommercial-ShareAlike 4.0 International (CC BY-NC-SA 4.0)**

Puedes compartir, adaptar y redistribuir este material siempre que:
- **Atribuyas la autoría** (Anaïs Rodríguez Villanueva)
- **No lo uses comercialmente**
- **Compartas tus adaptaciones bajo la misma licencia**

Ver [LICENSE](LICENSE) para texto legal completo.

**Autor:** Anaïs Rodríguez Villanueva  
**Contacto:** [LinkedIn](https://www.linkedin.com/in/anaisvillanueva/)

---

## Contribuciones

Si encuentras errores, mejoras en explicaciones o ejemplos que se pueden aclarar:

1. Abre un **issue** con descripción clara
2. Manda un **pull request** si sabes cómo solucionarlo
3. Comparte tu experiencia usando el material

Todas las contribuciones se reconocen apropiadamente.

---

## Uso y difusión

Si este material te ayuda:
- Dale una estrella al repositorio
- Compártelo con quien lo necesite
- Sígueme en [LinkedIn](https://www.linkedin.com/in/anaisvillanueva/) si quieres ver más contenido técnico

No hace falta permiso para usar el material. Está aquí para eso.

---

## Contacto

**LinkedIn:** https://www.linkedin.com/in/anaisvillanueva/  
**GitHub:** https://github.com/Anais-RV

---

**Este material está completo, mantenible y abierto.  
Tu aprendizaje es lo importante.**

---

*Última actualización: 22 Enero 2026*

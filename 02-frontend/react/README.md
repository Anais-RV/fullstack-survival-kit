# Frontend con React

Ya dominas HTML, CSS y JavaScript vanilla. Ahora vas a aprender a construir interfaces modernas con React.

**Este bloque te enseña el ecosistema profesional de desarrollo frontend.**

---

## Sobre este bloque

**Requisito previo:** Haber completado [01-fundamentos](../01-fundamentos/README.md)

Necesitas saber:
- ✅ HTML semántico (header, nav, main, article, section)
- ✅ CSS (box model, flexbox, grid, responsive)
- ✅ JavaScript (funciones, arrays, objetos, destructuring, spread)
- ✅ DOM (querySelector, eventos, manipulación)
- ✅ Asincronía (promises, async/await, fetch)

**Si no dominas estos temas, vuelve a 01-fundamentos.** React sin bases sólidas es frustrante.

---

## ¿Por qué React?

Cuando una aplicación web crece, manipular el DOM manualmente se vuelve caótico:
- Código repetitivo y difícil de mantener
- Bugs difíciles de rastrear
- Difícil coordinar estado entre componentes
- Performance problems con muchas actualizaciones

**React resuelve esto con:**
- Componentes reutilizables (como piezas de LEGO)
- Gestión declarativa del estado
- Virtual DOM (actualiza solo lo necesario)
- Ecosistema robusto de herramientas

---

## Estructura del bloque

Los módulos siguen esta progresión:

1. **React fundamentos** → Qué es React y cómo funciona
2. **Componentes y JSX** → Crear piezas reutilizables
3. **Estado y props** → Manejar datos y comunicación
4. **Hooks** → La forma moderna de React
5. **Routing** → Navegación entre páginas
6. **Formularios** → Capturar y validar datos
7. **APIs y efectos** → Conectar con backend
8. **Estado global** → Compartir datos entre componentes
9. **Optimización** → Hacer tu app más rápida
10. **Deployment** → Publicar tu aplicación

---

## Bloque 1: Introducción a React

### `01-por-que-react.md`
**Descripción:** Qué problemas resuelve React. Diferencia con vanilla JavaScript.  
**Nivel:** Esencial

### `02-instalacion-entorno.md`
**Descripción:** Instalar Node.js, crear tu primer proyecto con Vite.  
**Nivel:** Esencial

### `03-estructura-proyecto.md`
**Descripción:** Qué son todos esos archivos. package.json, node_modules, src.  
**Nivel:** Esencial

### `04-primer-componente.md`
**Descripción:** Tu primer componente React. Qué es JSX.  
**Nivel:** Esencial

### `05-jsx-profundidad.md`
**Descripción:** Reglas de JSX, expresiones, condicionales, listas.  
**Nivel:** Esencial

---

## Bloque 2: Componentes

### `06-anatomia-componente.md`
**Descripción:** Estructura de un componente. Imports, export, return.  
**Nivel:** Esencial

### `07-props.md`
**Descripción:** Pasar datos de padre a hijo. Props como parámetros.  
**Nivel:** Esencial

### `08-composicion-componentes.md`
**Descripción:** Anidar componentes. Children. Reutilización.  
**Nivel:** Esencial

### `09-condicional-rendering.md`
**Descripción:** Mostrar contenido según condiciones. && y operador ternario.  
**Nivel:** Esencial

### `10-listas-keys.md`
**Descripción:** Renderizar arrays. Por qué necesitas keys.  
**Nivel:** Esencial

---

## Bloque 3: Estado

### `11-que-es-estado.md`
**Descripción:** Qué es el estado. Diferencia con props.  
**Nivel:** Esencial

### `12-usestate-basico.md`
**Descripción:** Tu primer hook. Crear y actualizar estado.  
**Nivel:** Esencial

### `13-eventos-react.md`
**Descripción:** onClick, onChange, onSubmit. Eventos sintéticos.  
**Nivel:** Esencial

### `14-estado-objetos-arrays.md`
**Descripción:** Actualizar estado complejo. Inmutabilidad.  
**Nivel:** Esencial

### `15-levantar-estado.md`
**Descripción:** Lifting state up. Compartir estado entre hermanos.  
**Nivel:** Esencial

---

## Bloque 4: Efectos

### `16-useeffect-intro.md`
**Descripción:** Qué son efectos secundarios. Cuándo usar useEffect.  
**Nivel:** Esencial

### `17-useeffect-dependencias.md`
**Descripción:** Array de dependencias. Cuándo se ejecuta un efecto.  
**Nivel:** Esencial

### `18-fetch-useeffect.md`
**Descripción:** Cargar datos desde una API. Loading, error, success.  
**Nivel:** Esencial

### `19-cleanup-efectos.md`
**Descripción:** Limpiar efectos. Prevenir memory leaks.  
**Nivel:** Recomendado

---

## Bloque 5: Formularios

### `20-inputs-controlados.md`
**Descripción:** Conectar inputs con estado. Controlled components.  
**Nivel:** Esencial

### `21-formularios-multiples-campos.md`
**Descripción:** Manejar varios inputs. Objetos de estado.  
**Nivel:** Esencial

### `22-validacion-formularios.md`
**Descripción:** Validar datos antes de enviar. Mensajes de error.  
**Nivel:** Esencial

### `23-formularios-avanzados.md`
**Descripción:** Selects, checkboxes, radio buttons en React.  
**Nivel:** Recomendado

---

## Bloque 6: Routing

### `24-react-router-intro.md`
**Descripción:** Qué es routing. Instalar React Router.  
**Nivel:** Esencial

### `25-rutas-basicas.md`
**Descripción:** BrowserRouter, Routes, Route. Navegación entre páginas.  
**Nivel:** Esencial

### `26-link-navegacion.md`
**Descripción:** Link vs anchor. Navegación sin recargar.  
**Nivel:** Esencial

### `27-parametros-ruta.md`
**Descripción:** useParams. Rutas dinámicas. /user/:id  
**Nivel:** Esencial

### `28-rutas-protegidas.md`
**Descripción:** Proteger rutas según autenticación.  
**Nivel:** Recomendado

---

## Bloque 7: Hooks avanzados

### `29-useref.md`
**Descripción:** Referencias a elementos del DOM. Valores que persisten.  
**Nivel:** Recomendado

### `30-usememo.md`
**Descripción:** Memorizar cálculos costosos. Optimización.  
**Nivel:** Recomendado

### `31-usecallback.md`
**Descripción:** Memorizar funciones. Prevenir re-renders innecesarios.  
**Nivel:** Recomendado

### `32-custom-hooks.md`
**Descripción:** Crear tus propios hooks. Reutilizar lógica.  
**Nivel:** Esencial

---

## Bloque 8: Estado global

### `33-context-api.md`
**Descripción:** Compartir estado sin prop drilling. useContext.  
**Nivel:** Esencial

### `34-context-patron-provider.md`
**Descripción:** Crear un provider reutilizable. Buenas prácticas.  
**Nivel:** Esencial

### `35-cuando-usar-context.md`
**Descripción:** Context vs props vs estado local. Cuándo usar cada uno.  
**Nivel:** Recomendado

---

## Bloque 9: Estilos en React

### `36-css-react.md`
**Descripción:** Formas de estilizar: CSS puro, módulos, inline styles.  
**Nivel:** Esencial

### `37-css-modules.md`
**Descripción:** CSS Modules. Estilos con scope local.  
**Nivel:** Recomendado

### `38-styled-components.md`
**Descripción:** CSS-in-JS. Estilos dentro de componentes.  
**Nivel:** Opcional

---

## Bloque 10: Optimización y deployment

### `39-react-devtools.md`
**Descripción:** Herramientas de desarrollo. Debuggear React.  
**Nivel:** Recomendado

### `40-lazy-suspense.md`
**Descripción:** Code splitting. Cargar componentes bajo demanda.  
**Nivel:** Opcional

### `41-build-produccion.md`
**Descripción:** Compilar para producción. Optimizaciones automáticas.  
**Nivel:** Esencial

### `42-deploy-vercel-netlify.md`
**Descripción:** Publicar tu aplicación. Vercel y Netlify.  
**Nivel:** Esencial

---

## Resumen de niveles

### Esencial (28 módulos)
Lo que necesitas para trabajar profesionalmente con React.

### Recomendado (11 módulos)
Te harán más eficiente y resolverán problemas comunes.

### Opcional (3 módulos)
Profundización en temas específicos. Puedes saltarlos inicialmente.

---

## Orden de estudio

Sigue el orden numérico 01 → 42. React se construye incrementalmente.

**No saltes los primeros 20 módulos.** Son la base sobre la que se construye todo lo demás.

---

## Proyecto integrador

Al final de este bloque deberías ser capaz de crear:

**Una aplicación de gestión de tareas (Todo App) completa:**
- ✅ Múltiples vistas (Home, Tareas, Perfil)
- ✅ Formularios para crear/editar tareas
- ✅ Estado compartido entre componentes
- ✅ Conexión con API (backend del módulo 03)
- ✅ Autenticación básica
- ✅ Responsive y estilizada
- ✅ Desplegada en internet

---

## Siguiente paso

Cuando completes React, estarás listo para aprender backend.

→ Siguiente: [03-backend](../03-backend/README.md)

Ahí aprenderás a crear servidores con Python (Django/FastAPI) que alimenten tus aplicaciones React.

---

**React es una herramienta poderosa, pero requiere práctica. No te frustres si al principio parece complejo. Con cada componente que crees, todo tendrá más sentido.**

### `16-formularios-javascript.md`
**Descripción:** Capturar datos de formularios, validar con JS, mostrar errores.  
**Nivel:** Esencial

### `17-localstorage-sessionstorage.md`
**Descripción:** Guardar datos en el navegador. Persistir información sin backend (todavía).  
**Nivel:** Recomendado

---

## Bloque 4: Aplicación sin framework

Construir una SPA básica a mano para entender qué hacen los frameworks por ti.

### `18-spa-manual.md`
**Descripción:** Single Page Application sin framework. Cambiar vistas sin recargar la página.  
**Nivel:** Esencial

### `19-routing-manual.md`
**Descripción:** Gestionar rutas con History API. Sincronizar URL con lo que se muestra.  
**Nivel:** Recomendado

### `20-patron-componentes.md`
**Descripción:** Estructurar código en "componentes" reutilizables sin usar un framework.  
**Nivel:** Recomendado

---

## Bloque 5: React fundamentos

Por qué usar un framework y cómo funciona React.

### `21-por-que-frameworks.md`
**Descripción:** Qué problemas resuelven frameworks. Comparación: hacer todo a mano vs con herramientas.  
**Nivel:** Esencial

### `22-primer-componente-react.md`
**Descripción:** Crear componentes, JSX, renderizar en el DOM.  
**Nivel:** Esencial

### `23-props.md`
**Descripción:** Pasar datos de padres a hijos. Hacer componentes reutilizables.  
**Nivel:** Esencial

### `24-estado-useState.md`
**Descripción:** Estado local de un componente. Hacer que la interfaz reaccione a cambios.  
**Nivel:** Esencial

### `25-eventos-react.md`
**Descripción:** Manejar clicks, cambios en inputs, submit de formularios en React.  
**Nivel:** Esencial

### `26-listas-keys.md`
**Descripción:** Renderizar arrays. Por qué React necesita keys.  
**Nivel:** Esencial

### `27-renderizado-condicional.md`
**Descripción:** Mostrar u ocultar elementos según condiciones.  
**Nivel:** Esencial

---

## Bloque 6: React aplicado

Construir funcionalidades reales.

### `28-useEffect.md`
**Descripción:** Efectos secundarios: peticiones, suscripciones, sincronización con el exterior.  
**Nivel:** Esencial

### `29-fetch-react.md`
**Descripción:** Hacer peticiones a APIs desde React. Mostrar loading, manejar errores.  
**Nivel:** Esencial

### `30-formularios-react.md`
**Descripción:** Formularios controlados, validación, estado de formulario complejo.  
**Nivel:** Esencial

### `31-react-router.md`
**Descripción:** Enrutamiento con React Router. Navegación entre vistas.  
**Nivel:** Esencial

### `32-context-api.md`
**Descripción:** Estado global sin prop drilling. Compartir datos entre componentes lejanos.  
**Nivel:** Recomendado

### `33-custom-hooks.md`
**Descripción:** Crear tus propios hooks. Reutilizar lógica entre componentes.  
**Nivel:** Opcional

---

## Bloque 7: Estilos en React

Diferentes formas de aplicar estilos en aplicaciones React.

### `34-css-modules.md`
**Descripción:** CSS con scope local. Evitar conflictos de nombres de clases.  
**Nivel:** Recomendado

### `35-styled-components.md`
**Descripción:** CSS-in-JS. Estilos como componentes de JavaScript.  
**Nivel:** Opcional

### `36-librerias-ui.md`
**Descripción:** Material UI, Chakra, Ant Design. Cuándo usar componentes prefabricados.  
**Nivel:** Opcional

---

## Resumen de niveles

### Esencial (24 módulos)
Lo que necesitas para construir interfaces completas y funcionales.

### Recomendado (9 módulos)
Mejoran tu flujo de trabajo y hacen el código más mantenible.

### Opcional (3 módulos)
Profundización o alternativas. Útiles según el proyecto.

---

## Orden de estudio sugerido

### Si vienes de `02-fundamentos`:
Sigue el orden numérico 01 → 36. La progresión está pensada para no dar saltos bruscos.

**Parada importante:** Después del módulo 20, tienes una base sólida de frontend sin frameworks. Podrías construir aplicaciones completas así. React viene después para hacerlo más eficiente.

### Si ya conoces HTML/CSS pero no frameworks:
- Repasa rápido los bloques 1-2 (módulos 01-11)
- Trabaja el bloque 3 (módulos 12-17)
- El bloque 4 (módulos 18-20) es clave para entender qué hace React por ti
- Estudia React completo (módulos 21-36)

### Si ya conoces React pero vienes de otro framework:
- Revisa los módulos de React (21-33) adaptándote a la sintaxis
- Salta los opcionales si no te interesan

---

## Relación con otros bloques

### Antes de este bloque:
- **`01-primer-contacto-digital`** → Primer contacto con HTML/CSS/JS
- **`02-fundamentos`** → Base de programación y JavaScript

### Después de este bloque:
- **`04-backend`** → Construir el servidor que da datos al frontend
- **`06-integracion-fullstack`** → Conectar tu frontend con tu backend

---

## Proyecto recomendado

Al terminar este bloque, construye una aplicación frontend completa que consuma una API pública (ej: Pokemon API, Rick & Morty API, GitHub API).

Debe incluir:
- Múltiples vistas con React Router
- Peticiones a la API con loading y errores
- Formularios con validación
- Estado global con Context
- Estilos responsive

Esto consolida todo lo aprendido antes de pasar a backend.

---

## Validación de contenido

Antes de escribir estos módulos, valida que:
- [ ] La progresión de vanilla JS a React tiene sentido
- [ ] No hay solapamiento con `02-fundamentos`
- [ ] React no se introduce demasiado pronto
- [ ] Cada módulo tiene un "para qué" claro
- [ ] El bloque no es excesivamente largo (36 módulos es manejable)

---

**Este bloque es la mitad del fullstack. Frontend sólido = mejor backend después.**

**Última actualización:** 21 diciembre 2025

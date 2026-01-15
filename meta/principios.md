# Principios Operativos del Repositorio

Este documento recoge los principios derivados del [Manifiesto Pedagógico](../MANIFIESTO_PEDAGOGICO.md).

Son guías accionables para diseñar, revisar y mantener coherencia en los módulos del repositorio.

---

## 1. Un concepto, un momento

**Descripción:** Cada nuevo concepto se introduce de forma aislada antes de combinarlo con otros. No se mezclan múltiples ideas nuevas en un mismo ejercicio o explicación.

**Aplicación práctica:** En un módulo sobre peticiones HTTP, primero se explica `fetch()` con un ejemplo mínimo. Solo después se introduce el manejo de errores con `try/catch`. No se presentan ambos conceptos simultáneamente si alguno es nuevo.

---

## 2. Repetición contextualizada

**Descripción:** Los conceptos se revisitan en distintos contextos a lo largo del repositorio. Cada reaparición aporta un ángulo nuevo o una aplicación diferente.

**Aplicación práctica:** El concepto de "estado" se presenta primero en JavaScript básico (variables), luego en componentes de frontend (useState), después en gestión global (context), y finalmente en backend (sesiones). Cada vez se enlaza explícitamente con lo anterior.

---

## 3. Conexiones explícitas

**Descripción:** Cuando un módulo usa conceptos de bloques anteriores, se señala claramente con enlaces o referencias directas. No se asume que el lector recuerde todo.

**Aplicación práctica:** Si un ejercicio de autenticación requiere entender promesas, el módulo incluye: *"Este ejercicio usa promesas, que vimos en [Módulo 3: Asincronía](enlace)"*. El material funciona como red consultable, no como libro lineal.

---

## 4. Ejercicios con propósito visible

**Descripción:** Cada ejercicio tiene un objetivo claro y explícito. No se pide hacer algo "por práctica" sin explicar qué habilidad se está desarrollando.

**Aplicación práctica:** En lugar de *"Crea una función que ordene un array"*, el enunciado dice: *"Crea una función que ordene un array. Este ejercicio trabaja manipulación de datos, un patrón que usarás constantemente en frontend y backend"*.

---

## 5. Errores documentados

**Descripción:** Los errores comunes se anticipan y se documentan explícitamente en los módulos, junto con su interpretación y solución.

**Aplicación práctica:** Un módulo sobre async/await incluye una sección: *"Errores típicos: 'Cannot read property of undefined' → Esto suele pasar cuando la promesa no devuelve lo que esperas. Usa console.log para verificar qué devuelve realmente"*.

---

## 6. Invitación a experimentar

**Descripción:** Los módulos incluyen sugerencias concretas de modificaciones que el lector puede hacer para ver qué pasa. Se normaliza romper el código para entenderlo mejor.

**Aplicación práctica:** Después de un ejemplo funcional, el módulo incluye: *"Prueba a eliminar el `return` de esta función y observa el error. Luego, cambia el nombre de la variable y mira qué dice el mensaje. Esto te ayuda a leer mensajes de error con confianza"*.

---

## 7. Capas progresivas sin saltos

**Descripción:** El material avanza de conceptos simples a complejos sin dar saltos que dejen vacíos. Cada nuevo nivel se construye directamente sobre el anterior.

**Aplicación práctica:** No se presenta un módulo de API REST completo si antes no se han trabajado: HTTP, JSON, métodos de petición, respuestas y códigos de estado. Si falta algún fundamento, se añade explícitamente antes.

---

## 8. Opcional claramente marcado

**Descripción:** Todo contenido que no sea estrictamente necesario se marca como opcional, con indicación clara del nivel de profundidad.

**Aplicación práctica:** Un módulo sobre hooks de React incluye una sección final: *"[Opcional - Avanzado] Custom hooks: cómo crear tus propios hooks reutilizables"*. El lector sabe que puede saltarlo sin comprometer su comprensión del flujo principal.

---

## Validación rápida

Un módulo cumple estos principios si:
- Introduces **una idea nueva a la vez**
- Enlazas explícitamente conceptos previos necesarios
- Documentas errores previsibles
- Invitas a experimentar de forma concreta
- Marcas claramente lo opcional

---

**Este documento es material de trabajo interno. Se actualiza conforme el repositorio crece.**

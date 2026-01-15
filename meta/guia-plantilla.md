# Guía de uso de la plantilla

Esta guía explica cómo usar la [plantilla canónica de módulo](./plantilla-modulo.md) para mantener coherencia con el [Manifiesto Pedagógico](../MANIFIESTO_PEDAGOGICO.md).

---

## Criterios de aplicación

### ✅ Cuándo usar esta plantilla

Úsala para **todo módulo técnico** del repositorio que:
- Enseñe un concepto nuevo
- Incluya código ejecutable
- Requiera práctica activa

### ⚠️ Cuándo NO usar esta plantilla

No la uses para:
- Documentos de contexto (como el manifiesto)
- Índices o navegación
- Guías de configuración sin código
- Material puramente teórico

---

## Cómo rellenar cada sección

### 📌 Introducción

**"Qué vas a aprender"**
- Lista 2-4 conceptos concretos
- Evita generalidades: en lugar de "aprenderás funciones", di "crear funciones, pasarles parámetros y recibir valores de retorno"
- Si introduces más de 3 conceptos, considera dividir en dos módulos

**"Por qué es útil"**
- 1-3 párrafos máximo
- Menciona casos de uso reales en proyectos
- Sin vender ni motivar: solo contextualizar

---

### 🔑 Conceptos clave

**Regla de oro: UN concepto nuevo a la vez**

Si introduces múltiples conceptos:
- Sepáralos en subsecciones
- Trabaja uno antes de pasar al siguiente
- Cada concepto debe tener: definición + forma básica

**Conceptos que ya conoces:**
- Menciona explícitamente qué se necesita de módulos anteriores
- Enlaza directamente
- No asumas que se recuerda: facilita la consulta

---

### 💡 Ejemplo mínimo funcional

**Características del ejemplo:**
- ✅ Simple: una sola cosa a la vez
- ✅ Ejecutable: se puede copiar y probar tal cual
- ✅ Comentado: explica lo no obvio
- ❌ Sin abstracciones innecesarias
- ❌ Sin dependencias complejas

**Después del código:**
- Explica QUÉ hace (línea por línea si es necesario)
- Explica POR QUÉ funciona así (concepto aplicado)

---

### ✏️ Ejercicios prácticos

**Diseño de ejercicios:**
- Mínimo 2, máximo 5
- Progresión gradual: simple → complejo
- Cada ejercicio debe tener **objetivo explícito**

**Objetivo del ejercicio:**
En lugar de: *"Practica arrays"*  
Escribe: *"Practicar recorrer arrays con métodos nativos, un patrón que usarás constantemente en manipulación de datos"*

**Pistas progresivas:**
- Usa `<details>` colapsables
- Primera pista: sutil, orienta
- Segunda pista: más explícita
- Nunca des la solución completa

---

### ⚠️ Errores comunes

**Qué incluir:**
- Mensaje de error **exacto** (copiado literalmente)
- Cuándo aparece (situación que lo provoca)
- Qué significa (interpretación del mensaje)
- Cómo arreglarlo (proceso, no solo solución)
- Qué se aprende (concepto que refuerza)

**Cuántos incluir:**
- Mínimo 2 errores
- Solo errores realmente comunes (no casos raros)
- Prioriza errores que enseñan conceptos clave

---

### 🔥 Experimenta

**Objetivo:** Normalizar romper código como forma de aprender

**Formato de cada experimento:**
- **Modifica:** qué cambiar concretamente
- **Observa:** qué resultado esperar
- **Aprende:** qué muestra esto sobre el sistema

**Ejemplos de modificaciones:**
- Eliminar una línea y ver el error
- Cambiar un valor y observar el efecto
- Combinar con conceptos previos
- Forzar un caso límite

**Mínimo:** 3 experimentos por módulo

---

### 🔗 Conexiones

**Conceptos previos relacionados:**
- Enlaza directamente a módulos anteriores
- Menciona solo los realmente necesarios
- Facilita repaso sin sentimiento de retroceso

**Dónde se usará esto:**
- Anticipa aplicaciones futuras
- No des spoilers: solo nombra
- Refuerza que esto es parte de algo mayor

---

### 🚀 Opcional: Profundiza

**Cuándo usar:**
- Conceptos avanzados no esenciales
- Patrones alternativos
- Detalles técnicos especializados
- Material que ralentiza el flujo principal

**Cómo marcar:**
- **Siempre** con advertencia clara al inicio
- Ejemplo: *"⚠️ Esta sección es opcional. Puedes saltarla sin problema."*
- El contenido opcional debe ser **realmente omitible**

**Test de omisibilidad:**
Si al saltarlo el lector no puede hacer los ejercicios siguientes, **NO es opcional**.

---

## Validación antes de publicar

Antes de considerar terminado un módulo, valida con la [checklist de coherencia](./checklist.md).

Revisa especialmente:
- ¿Los conceptos se introducen de uno en uno?
- ¿Las conexiones a módulos previos son explícitas?
- ¿Los ejercicios invitan a crear, no solo a copiar?
- ¿Los errores comunes están documentados?
- ¿Hay invitación concreta a experimentar?

---

## Tono y estilo

**✅ Hacer:**
- Lenguaje técnico claro y preciso
- Tono cercano y profesional
- Explicar sin subestimar
- Normalizar el error como aprendizaje

**❌ Evitar:**
- Lenguaje institucional
- Tono motivacional o paternalista
- Etiquetar dificultades ("esto es fácil/difícil")
- Asumir perfiles o ritmos específicos
- Mencionar necesidades especiales

---

## Mantenimiento

Esta plantilla puede evolucionar. Si detectas que:
- Falta una sección importante
- Algo es redundante
- El formato no sirve en ciertos casos

Actualiza la plantilla y documenta por qué.

El objetivo es coherencia, no rigidez.

---

**Recuerda:** La plantilla es una herramienta, no una camisa de fuerza. Si algo no encaja, pregúntate primero si el problema es la plantilla o el diseño del módulo.

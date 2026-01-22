# Experimentar sin miedo

> **El error no es el enemigo. El miedo a equivocarse sí.**

---

## La paradoja del aprendizaje

**Para aprender a programar necesitas:**
1. Escribir código
2. Que el código falle
3. Entender por qué falló
4. Arreglarlo
5. Volver al paso 1

**El problema:** Mucha gente se queda atascada en el paso 2, pensando "soy malo en esto".

**La realidad:** Los errores son parte del proceso, no señal de fracaso.

---

## Cambia tu mentalidad sobre los errores

### ❌ Mentalidad fija

- "Si no me sale a la primera, no sirvo para esto"
- "Los buenos programadores no cometen errores"
- "Debería saber esto sin buscarlo en Google"
- "Si tengo un error, es porque hice algo mal"

### ✅ Mentalidad de crecimiento

- "Si no me sale, aprenderé qué falla"
- "Todos los programadores ven errores a diario"
- "Google es una herramienta, no una muleta"
- "Los errores me dicen exactamente qué debo arreglar"

---

## Los errores son información, no juicios

Cuando tu código falla, el ordenador no está diciendo:
- ❌ "Eres malo programando"
- ❌ "No deberías hacer esto"
- ❌ "Eres lento"

El ordenador está diciendo:
- ✅ "En la línea 15, esperaba un punto y coma"
- ✅ "Esta variable no existe todavía"
- ✅ "Esta función necesita 2 argumentos, le diste 1"

**Es información objetiva, no una evaluación de tu capacidad.**

---

## Tipos de errores (y cómo leerlos)

### 1. Error de sintaxis

**Qué es:** Escribiste el código mal (como una falta de ortografía).

**Ejemplo:**
```javascript
console.log("Hola"  // ❌ Falta cerrar paréntesis
```

**Error:**
```
Uncaught SyntaxError: missing ) after argument list
```

**Traducción:** "Falta un paréntesis de cierre"

**Solución:**
```javascript
console.log("Hola")  // ✅
```

### 2. Error de referencia

**Qué es:** Usaste algo que no existe.

**Ejemplo:**
```javascript
console.log(nombre);  // ❌ nombre no está definido
```

**Error:**
```
Uncaught ReferenceError: nombre is not defined
```

**Traducción:** "La variable 'nombre' no existe"

**Solución:**
```javascript
let nombre = "Ana";
console.log(nombre);  // ✅
```

### 3. Error de tipo

**Qué es:** Intentaste hacer algo incompatible.

**Ejemplo:**
```javascript
let numero = 5;
numero.toUpperCase();  // ❌ Los números no tienen toUpperCase
```

**Error:**
```
Uncaught TypeError: numero.toUpperCase is not a function
```

**Traducción:** "Los números no pueden convertirse a mayúsculas"

**Solución:**
```javascript
let texto = "hola";
texto.toUpperCase();  // ✅ "HOLA"
```

---

## Estrategias para enfrentar errores

### 1. Lee el error completo

No entres en pánico. Lee despacio:

```
Uncaught ReferenceError: suma is not defined
    at script.js:15
```

**Información útil:**
- **Tipo:** ReferenceError (algo no existe)
- **Qué:** suma is not defined (la función "suma" no existe)
- **Dónde:** script.js línea 15

**Ya sabes qué buscar:** En la línea 15 estás usando `suma`, pero no la has definido antes.

### 2. Aísla el problema

Si tienes mucho código, comenta partes para encontrar dónde falla:

```javascript
// console.log(a);  // ← Comenta esta línea
console.log(b);
// console.log(c);  // ← Comenta esta línea
```

Ejecuta. ¿Funciona? El problema está en `a` o `c`.

### 3. Usa console.log() estratégicamente

```javascript
function calcularTotal(precio, cantidad) {
    console.log("precio:", precio);  // ← Ver qué valor tiene
    console.log("cantidad:", cantidad);
    
    let total = precio * cantidad;
    console.log("total:", total);
    
    return total;
}
```

**Ver los valores te ayuda a entender qué está pasando.**

### 4. Simplifica al mínimo

Si algo no funciona, hazlo más simple:

**Complejo (no funciona):**
```javascript
document.querySelector('#lista').innerHTML = usuarios.map(u => `<li>${u.nombre}</li>`).join('');
```

**Simple (para entender):**
```javascript
let lista = document.querySelector('#lista');
console.log(lista);  // ¿Existe el elemento?

console.log(usuarios);  // ¿Hay usuarios?

let items = usuarios.map(u => `<li>${u.nombre}</li>`);
console.log(items);  // ¿Se generan bien?

lista.innerHTML = items.join('');
```

### 5. Búscalo en Google

**Copia el error y pégalo en Google.**

Ejemplo de búsqueda:
```
Uncaught ReferenceError: $ is not defined
```

**Encontrarás:**
- Stack Overflow con respuestas
- Tutoriales explicando el problema
- Documentación oficial

**Buscar en Google NO es trampa.** Es lo que hacen todos los programadores, incluso los que llevan 20 años.

---

## Ejercicio: Depurar código con errores

Crea `debugging-practica.html`:

```html
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <title>Práctica de Debugging</title>
</head>
<body>
    <h1>Encuentra los errores</h1>
    
    <button onclick="saludar()">Saludar</button>
    
    <script>
        function saludar() {
            let nombre = "Ana"
            console.log("Hola, " + nombre)
        }
        
        saludar();
    </script>
</body>
</html>
```

**Este código funciona.** Ahora, introduce errores uno por uno y observa qué pasa:

### Error 1: Olvida punto y coma

Borra el `;` después de `"Ana"`.

**¿Qué pasa?** ¿Sale error o funciona igual?

(JavaScript perdona esto, pero es mala práctica)

### Error 2: Escribe mal el nombre de función

Cambia:
```html
<button onclick="saludar()">Saludar</button>
```

Por:
```html
<button onclick="saludarr()">Saludar</button>
```

**¿Qué error sale?**

### Error 3: Usa una variable que no existe

Dentro de `saludar()`, añade:
```javascript
console.log(apellido);
```

**¿Qué error sale?**

### Error 4: Olvida cerrar comillas

Cambia:
```javascript
let nombre = "Ana"
```

Por:
```javascript
let nombre = "Ana
```

**¿Qué error sale?**

---

## El ciclo de debugging

```
1. Escribir código
        ↓
2. Ejecutar
        ↓
   ¿Funciona?
    /      \
  SÍ       NO
   |         \
   |      3. Leer error
   |          ↓
   |      4. Entender problema
   |          ↓
   |      5. Arreglar
   |          ↓
   └──────────┘
     (Volver a 2)
```

**Este ciclo lo harás miles de veces en tu carrera.** Es normal. Es el proceso.

---

## Historias reales de errores

### Historia 1: El punto y coma que costó millones

En 1990, una nave espacial falló porque el código tenía un `.` en lugar de `,`.

```fortran
DO 10 I = 1. 10  ! ❌ Punto en lugar de coma
```

**Lección:** Hasta los expertos cometen errores. Los sistemas profesionales tienen tests para evitar esto.

### Historia 2: El error de 1 píxel

Un desarrollador pasó 3 horas buscando por qué su diseño se veía mal.

**Razón:** Había escrito `margn` en lugar de `margin`.

**Lección:** Los typos (errores de escritura) son súper comunes. Revisa ortografía primero.

### Historia 3: El bug más caro de la historia

Un banco perdió $500 millones por un error en un script de trading automatizado.

**Lección:** En producción, los errores pueden ser costosos. Por eso existen:
- Tests automáticos
- Revisión de código
- Entornos de prueba

Pero como principiante, **tus errores no cuestan nada**. Aprovecha para equivocarte ahora.

---

## Consejos prácticos

### 1. Comienza con código que funciona

No escribas 200 líneas de una vez. Escribe 10, prueba. Escribe 10 más, prueba.

**Si falla, sabes que el error está en las últimas 10 líneas.**

### 2. Guarda versiones que funcionan

Cuando algo funciona bien, copia el archivo con otro nombre:

```
proyecto.html
proyecto-funcional-v1.html
proyecto-funcional-v2.html
```

Si rompes algo, puedes volver atrás.

### 3. Comenta tu código

```javascript
// Esta función calcula el total con impuestos
function calcularTotal(precio) {
    let impuesto = precio * 0.21;  // IVA del 21%
    return precio + impuesto;
}
```

**Ayuda a tu yo futuro** a entender qué hace cada parte.

### 4. Toma descansos

Si llevas 30 minutos atascado/a:
1. Aléjate del ordenador
2. Haz otra cosa 10-15 minutos
3. Vuelve con mente fresca

**Muchas veces la solución aparece cuando no estás mirando el código.**

### 5. Explícaselo a alguien (o a un patito de goma)

Método real usado por programadores:

1. Coge un objeto (un patito de goma, un peluche...)
2. Explícale en voz alta qué hace tu código, línea por línea

**Muchas veces, al explicarlo, encuentras el error.**

---

## Tu espacio seguro para experimentar

### Crea una carpeta de experimentos

```
mis-experimentos/
├── experimento-1.html
├── experimento-2.html
├── pruebas-locas.html
├── codigo-roto.html
└── sandbox/
```

**En esta carpeta:**
- No hay consecuencias
- Puedes romper todo
- Nadie lo va a ver
- Es tu laboratorio privado

### Regla de oro

> "Si no estás rompiendo código, no estás aprendiendo lo suficientemente rápido"

---

## Ejercicio final: Rompe código a propósito

Crea `experimento-romper.html` con este código funcional:

```html
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <title>Experimento de errores</title>
</head>
<body>
    <h1>Contador</h1>
    <p id="numero">0</p>
    <button onclick="sumar()">+1</button>
    
    <script>
        let contador = 0;
        
        function sumar() {
            contador = contador + 1;
            document.getElementById('numero').textContent = contador;
        }
    </script>
</body>
</html>
```

**Ahora rómpelo de 10 formas diferentes** y observa qué errores salen:

1. Borra una etiqueta de cierre
2. Escribe mal el id
3. Olvida `let` en la variable
4. Pon texto donde va un número
5. Borra un paréntesis
6. Cambia `contador` por `Contador` en un sitio
7. Olvida cerrar comillas
8. Escribe mal el nombre de la función
9. Borra el `+` en `contador + 1`
10. Pon un `;` donde no va

**Para cada error:**
- ¿Qué mensaje sale?
- ¿Entiendes por qué?
- ¿Cómo lo arreglas?

---

## Conclusión

### Los errores son tus maestros

Cada error que resuelves:
- ✅ Te enseña algo nuevo
- ✅ Te hace mejor programador/a
- ✅ Te prepara para el siguiente desafío

### No hay programadores sin errores

- Los juniors ven errores y aprenden
- Los seniors ven errores y los arreglan rápido
- Los expertos ven errores y previenen los siguientes

**Todos ven errores.** La diferencia está en cómo reaccionas.

### Experimenta sin miedo

Esta es la fase perfecta para:
- Probar ideas locas
- Romper código sin consecuencias
- Aprender haciendo

**No desperdicies esta oportunidad siendo precavido/a.**

---

## Lo que has aprendido en este módulo

✅ Los errores son información, no fracasos  
✅ Cómo leer mensajes de error  
✅ Estrategias para debuggear  
✅ Google es tu amigo  
✅ Experimentar es la forma de aprender  

---

## Siguiente paso

Ya tienes las bases mentales y prácticas para empezar.

**Siguiente bloque:** [02-frontend/fundamentos](../../02-frontend/fundamentos/README.md)

Aquí empezarás el aprendizaje estructurado de:
- HTML en profundidad
- CSS para diseño
- JavaScript desde cero
- Manipulación del DOM
- Proyectos reales

---

## Reflexión final

Antes de continuar, piensa:

1. ¿Cuál fue tu primer error en código y cómo lo resolviste?
2. ¿Qué aprendiste experimentando que no habrías aprendido solo leyendo?
3. ¿Qué te da más miedo ahora y cómo puedes enfrentarlo?

**Escribe las respuestas.** Vuelve a leerlas en 6 meses. Te sorprenderá cuánto habrás crecido.

---

## Recursos adicionales

- **[Stack Overflow](https://stackoverflow.com/)** — Pregunta y responde dudas de programación
- **[MDN Web Docs](https://developer.mozilla.org/es/)** — Documentación técnica de referencia
- **[The Odin Project](https://www.theodinproject.com/)** — Curso completo de desarrollo web

---

## Un último consejo

> "El código perfecto no existe.  
> El código funcional que aprende de sus errores, sí."

**Ahora, ve y rompe cosas (digitalmente). Es la mejor forma de aprender.** 🚀

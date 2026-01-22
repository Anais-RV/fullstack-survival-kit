# Tu ordenador por dentro

> **Cómo piensa y funciona el ordenador que estás usando ahora mismo**

---

## ¿Para qué necesito saber esto?

Para programar, no necesitas ser ingeniero informático. Pero entender **lo básico** de cómo funciona un ordenador te ayuda a:

- Saber qué puede hacer y qué no puede hacer
- Entender por qué las cosas funcionan como funcionan
- Resolver problemas cuando algo falla
- Comunicarte con otros programadores

No te preocupes, **no vamos a entrar en detalles técnicos profundos**. Solo lo esencial.

---

## El ordenador es una máquina muy obediente (y muy tonta)

Un ordenador puede:
- ✅ Hacer millones de operaciones por segundo
- ✅ Recordar cantidades enormes de información
- ✅ Seguir instrucciones al pie de la letra

Un ordenador **NO** puede:
- ❌ Pensar por sí mismo
- ❌ Entender lo que "quieres decir"
- ❌ Improvisar
- ❌ Aprender solo (bueno, hay excepciones con IA, pero eso es otro tema)

**Tu trabajo como programador/a:** Darle instrucciones tan claras y específicas que no haya lugar a dudas.

---

## Las piezas básicas de un ordenador

### 1. Procesador (CPU) — El cerebro

El procesador **ejecuta instrucciones**.

Piensa en él como un chef que sigue una receta:
- Lee la instrucción
- La ejecuta
- Pasa a la siguiente

Hace esto millones de veces por segundo.

**Ejemplo:**
```
Instrucción: "Suma 5 + 3"
Procesador: "8"
```

### 2. Memoria RAM — La mesa de trabajo

La RAM es donde el ordenador **guarda temporalmente** lo que está usando ahora.

Es como la mesa del chef:
- Tiene los ingredientes a mano
- Trabaja rápido porque está todo cerca
- Cuando terminas, se limpia (se apaga el ordenador)

**Por qué importa:**
- Si abres muchos programas, la RAM se llena
- Si la RAM se llena, el ordenador va lento
- Cuando apagas el ordenador, la RAM se vacía

### 3. Disco duro — El almacén

El disco duro (o SSD) **guarda archivos de forma permanente**.

Es como la despensa del chef:
- Guarda ingredientes para el futuro
- Más lento de acceder que la RAM
- Los datos permanecen aunque apagues el ordenador

**Aquí se guardan:**
- Tus archivos
- Tus programas
- Tu sistema operativo
- El código que vas a escribir

### 4. Sistema operativo — El director de orquesta

El sistema operativo (Windows, macOS, Linux) **coordina todo**.

Decide:
- Qué programa usa el procesador ahora
- Qué ocupa la RAM
- Qué se guarda en el disco
- Cómo interactúan los programas

**Como programador/a**, escribes código que el sistema operativo ejecuta.

---

## Archivos y carpetas: la organización

### ¿Qué es un archivo?

Un archivo es **información guardada con un nombre**.

Tipos de archivos que usarás:
- `.txt` → Texto plano
- `.html` → Página web
- `.css` → Estilos visuales
- `.js` → Código JavaScript
- `.py` → Código Python
- `.jpg`, `.png` → Imágenes

**Importante:** La extensión (`.html`, `.css`) le dice al ordenador **cómo interpretar** ese archivo.

### ¿Qué es una carpeta?

Una carpeta es **un contenedor de archivos** (y otras carpetas).

Ejemplo de estructura:
```
mi-proyecto/
├── index.html
├── styles.css
├── script.js
└── imagenes/
    ├── logo.png
    └── foto.jpg
```

**Por qué importa:**
- Mantener el código organizado
- Saber dónde está cada cosa
- Trabajar en proyectos grandes

---

## Rutas: cómo encontrar archivos

Una ruta es la **dirección** de un archivo.

### Ruta absoluta

La ruta completa desde la raíz del disco:

**Windows:**
```
C:\Users\TuNombre\Documentos\mi-proyecto\index.html
```

**Mac/Linux:**
```
/Users/TuNombre/Documentos/mi-proyecto/index.html
```

### Ruta relativa

Relativa a dónde estás ahora:

Si estás en `mi-proyecto/`:
```
index.html          → archivo en la misma carpeta
imagenes/logo.png   → archivo en subcarpeta
../otro-proyecto/   → subir un nivel y entrar a otra carpeta
```

**Vas a usar rutas relativas constantemente** para conectar archivos HTML con CSS y JavaScript.

---

## Cómo entiende el ordenador el código

### 1. Escribes código

Tú escribes en un lenguaje de programación (HTML, JavaScript, Python...).

```html
<h1>Hola mundo</h1>
```

### 2. El ordenador lo interpreta

Dependiendo del lenguaje:
- **HTML/CSS:** El navegador lo lee y dibuja en pantalla
- **JavaScript:** El navegador lo ejecuta paso a paso
- **Python:** El intérprete de Python lo ejecuta

### 3. Ves el resultado

- Una página web
- Un cálculo
- Un mensaje
- Un programa funcionando

---

## Conceptos clave para recordar

### El ordenador es literal

Si escribes:
```javascript
let nombre = "Ana";
console.log(Nombre);  // ❌ Error: Nombre no está definido
```

El ordenador dice: "No conozco `Nombre` (con mayúscula)".  
Tú le dijiste `nombre` (con minúscula).

**Para el ordenador, `nombre` y `Nombre` son cosas diferentes.**

### El ordenador sigue instrucciones en orden

```javascript
console.log(x);  // ❌ Error: x no existe todavía
let x = 10;
```

El ordenador lee de arriba a abajo. Primero intentas mostrar `x`, pero aún no existe.

### El ordenador necesita sintaxis correcta

Si escribes:
```javascript
console.log("Hola"  // ❌ Falta cerrar paréntesis
```

El ordenador no puede ejecutar el código porque **no está bien escrito**.

Es como escribir una frase sin punto final. El ordenador no puede "adivinar" qué quisiste decir.

---

## Ejercicio mental: Piensa como un ordenador

Lee estas instrucciones y ejecútalas **literalmente**:

```
1. Piensa en un número
2. Súmale 5
3. Muéstralo
```

¿Qué pasa?  
No puedes hacer el paso 2 si no has hecho el paso 1.

**Eso es programar:** Dar instrucciones en el orden correcto, con la información necesaria.

---

## Analogía: El ordenador como robot de cocina

Imagina un robot de cocina que solo entiende comandos exactos:

**Tú dices:** "Haz una tortilla"  
**Robot:** "No entiendo 'tortilla'"

**Tú dices:**
```
1. Toma 2 huevos
2. Rómpelos en un bol
3. Bate durante 30 segundos
4. Pon una sartén al fuego medio
5. Vierte los huevos
6. Espera 3 minutos
7. Da la vuelta
8. Espera 2 minutos
9. Sirve en un plato
```

**Robot:** "Entendido. Ejecutando..."

**Así es programar:** Instrucciones específicas, en orden, sin ambigüedad.

---

## Conclusión

No necesitas ser experto/a en hardware para programar. Pero entender esto te ayuda:

- ✅ El ordenador es rápido pero literal
- ✅ Los archivos tienen extensiones que importan
- ✅ Las carpetas organizan el código
- ✅ Las rutas conectan archivos entre sí
- ✅ El código se ejecuta en orden, paso a paso

**Siguiente:** [Internet y la web](./03-internet-y-web.md)

---

## Preguntas frecuentes

**¿Necesito un ordenador potente para programar?**  
No. Un ordenador básico es más que suficiente para aprender desarrollo web.

**¿Puedo programar en una tablet o móvil?**  
Técnicamente sí, pero es incómodo. Un ordenador (portátil o de escritorio) es mucho mejor.

**¿Windows, Mac o Linux?**  
Cualquiera funciona. Este curso es compatible con los tres.

**¿Qué pasa si no entiendo algo?**  
Continúa leyendo. Muchas cosas se entienden mejor cuando ves ejemplos prácticos. Siempre puedes volver.

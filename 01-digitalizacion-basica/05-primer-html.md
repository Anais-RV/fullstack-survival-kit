# Tu primer HTML

> **Crea tu primera página web (de verdad, en los próximos 10 minutos)**

---

## Vas a crear esto

Al final de este módulo, tendrás un archivo HTML funcionando que se ve así en el navegador:

```
═══════════════════════════════════
Mi Primera Página Web

Hola, soy [tu nombre]

Esta es mi primera página web. 
Estoy aprendiendo desarrollo web.

Cosas que me gustan:
• Música
• Viajar  
• Aprender

═══════════════════════════════════
```

Simple, pero **es tuya** y **funciona**.

---

## Paso 1: Crea una carpeta para tus experimentos

1. Crea una carpeta en tu escritorio o documentos
2. Nómbrala `mis-experimentos-web`

**Windows:**
```
C:\Users\TuNombre\Documentos\mis-experimentos-web\
```

**Mac/Linux:**
```
/Users/TuNombre/Documentos/mis-experimentos-web/
```

Esta carpeta guardará todos tus archivos de prueba.

---

## Paso 2: Crea tu primer archivo HTML

### Opción A: Con editor de código (recomendado)

Si tienes Visual Studio Code instalado:

1. Abre VS Code
2. `Archivo → Abrir carpeta` → Selecciona `mis-experimentos-web`
3. Clic derecho en el panel izquierdo → `Nuevo archivo`
4. Nómbralo `mi-primera-pagina.html`

### Opción B: Con el Bloc de notas (Windows)

1. Abre el Bloc de notas
2. `Archivo → Guardar como`
3. Navega a tu carpeta `mis-experimentos-web`
4. Nombre: `mi-primera-pagina.html`
5. **Importante:** En "Tipo", elige "Todos los archivos"
6. Guarda

**⚠️ Cuidado:** Si guardas como `.txt` no funcionará. Debe terminar en `.html`

---

## Paso 3: Escribe tu primer código HTML

Copia este código **exactamente** en tu archivo:

```html
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Mi Primera Página</title>
</head>
<body>
    <h1>Mi Primera Página Web</h1>
    
    <p>Hola, soy [tu nombre]</p>
    
    <p>Esta es mi primera página web. Estoy aprendiendo desarrollo web.</p>
    
    <h2>Cosas que me gustan:</h2>
    <ul>
        <li>Música</li>
        <li>Viajar</li>
        <li>Aprender</li>
    </ul>
</body>
</html>
```

**Cambia `[tu nombre]` por tu nombre real.**

**Guarda el archivo** (Ctrl+S o Cmd+S).

---

## Paso 4: Abre tu página en el navegador

### Opción 1: Doble clic

Busca el archivo `mi-primera-pagina.html` en tu carpeta y haz doble clic.

Se abrirá en tu navegador predeterminado.

### Opción 2: Desde VS Code

Si usas VS Code con la extensión "Live Server":
- Clic derecho en el archivo → "Open with Live Server"

### Opción 3: Arrastrar

Arrastra el archivo a tu navegador (Chrome, Firefox...).

---

## ¡Felicidades! 🎉

Acabas de crear tu primera página web.

**No es impresionante visualmente**, pero:
- ✅ Es una página web real
- ✅ Funciona en cualquier navegador
- ✅ La creaste tú
- ✅ Está en tu ordenador

---

## ¿Qué significa ese código?

Vamos línea por línea (no te preocupes si no entiendes todo):

### `<!DOCTYPE html>`

Le dice al navegador: "Esto es HTML moderno".

### `<html lang="es">`

Inicio del documento HTML. `lang="es"` dice que está en español.

### `<head>` ... `</head>`

Información sobre la página (no se muestra visualmente):

- `<meta charset="UTF-8">` → Permite usar acentos y ñ
- `<meta name="viewport"...>` → Hace que se vea bien en móviles
- `<title>` → El texto que ves en la pestaña del navegador

### `<body>` ... `</body>`

El contenido visible de la página.

### `<h1>`, `<h2>`

Encabezados (títulos). `h1` es el más grande, `h2` el siguiente.

### `<p>`

Párrafo de texto.

### `<ul>` y `<li>`

Lista sin numerar:
- `<ul>` = Unordered List (lista sin orden)
- `<li>` = List Item (elemento de lista)

---

## Anatomía de una etiqueta HTML

```html
<etiqueta atributo="valor">Contenido</etiqueta>
   │         │        │        │          │
   │         │        │        │          └─ Etiqueta de cierre
   │         │        │        └─ Contenido
   │         │        └─ Valor del atributo
   │         └─ Atributo
   └─ Etiqueta de apertura
```

**Ejemplo:**
```html
<a href="https://google.com">Ir a Google</a>
```

- Etiqueta: `a` (anchor = enlace)
- Atributo: `href` (dirección)
- Valor: `https://google.com`
- Contenido: "Ir a Google"

---

## Experimenta: Modifica el código

### Experimento 1: Cambia el título

```html
<h1>Mi Primera Página Web</h1>
```

Cámbialo a:
```html
<h1>¡Hola Mundo!</h1>
```

Guarda (Ctrl+S) y recarga el navegador (F5).

### Experimento 2: Añade más texto

```html
<p>Esta es mi primera página web. Estoy aprendiendo desarrollo web.</p>
```

Añade otro párrafo debajo:
```html
<p>Pronto podré crear sitios web increíbles.</p>
```

### Experimento 3: Añade más elementos a la lista

```html
<ul>
    <li>Música</li>
    <li>Viajar</li>
    <li>Aprender</li>
    <li>Programar</li>  <!-- ← Añade este -->
</ul>
```

### Experimento 4: Añade un enlace

Añade esto antes del `</body>`:

```html
<p>Visita <a href="https://google.com">Google</a></p>
```

**Cada vez que cambies algo, guarda y recarga el navegador.**

---

## ¿Y si algo no funciona?

### La página no se abre

- ¿El archivo termina en `.html`?
- ¿Lo guardaste correctamente?
- ¿Estás haciendo doble clic en el archivo correcto?

### Se ven caracteres raros

Asegúrate de que tienes esta línea:
```html
<meta charset="UTF-8">
```

### No veo cambios

- ¿Guardaste el archivo? (Ctrl+S / Cmd+S)
- ¿Recargaste el navegador? (F5)

### Aparece el código, no la página

El archivo debe terminar en `.html`, no `.txt`

---

## Conceptos clave que acabas de aprender

### 1. Estructura básica HTML

Todo documento HTML tiene:
```html
<!DOCTYPE html>
<html>
  <head> <!-- Info del documento --> </head>
  <body> <!-- Contenido visible --> </body>
</html>
```

### 2. Etiquetas de apertura y cierre

```html
<etiqueta>contenido</etiqueta>
```

Algunas etiquetas se cierran solas:
```html
<br>  <!-- Salto de línea -->
<img> <!-- Imagen -->
```

### 3. Anidamiento

Etiquetas dentro de otras:
```html
<body>
  <h1>Título</h1>
  <p>Párrafo</p>
</body>
```

### 4. HTML es texto plano

Un archivo HTML es **solo texto**. No es un documento de Word ni un PDF. Por eso puedes editarlo con cualquier editor de texto.

---

## Siguiente nivel: Añade color

Añade esto dentro de `<head>`, justo después de `<title>`:

```html
<style>
    body {
        background-color: lightblue;
        font-family: Arial, sans-serif;
        padding: 20px;
    }
    h1 {
        color: darkblue;
    }
</style>
```

Guarda, recarga. ¡Tu página ahora tiene color!

**Esto es CSS** (lo aprenderás en [02-frontend/fundamentos/bloque-02-css](../../02-frontend/fundamentos/bloque-02-css/06-introduccion-css.md)).

---

## Desafío: Crea tu segunda página

Crea un archivo `mi-segunda-pagina.html` con:

- Un título principal (`<h1>`)
- Tu presentación personal (varios `<p>`)
- Una lista de tus hobbies (`<ul>` con `<li>`)
- Un enlace a tu red social favorita (`<a>`)

**No copies y pegues.** Escribe el código tú mismo/a. Así se aprende.

---

## Lo que has logrado

✅ Creaste tu primer archivo HTML  
✅ Lo abriste en un navegador  
✅ Entiendes la estructura básica  
✅ Modificaste contenido  
✅ Experimentaste sin miedo  

**Eso es más de lo que el 90% de la gente ha hecho.**

---

## Conclusión

HTML no es complicado. Es:
- Etiquetas que abren: `<etiqueta>`
- Contenido
- Etiquetas que cierran: `</etiqueta>`

El navegador lee el HTML y lo convierte en lo que ves.

**Profundizarás en HTML en:** [02-frontend/fundamentos/bloque-01-html](../../02-frontend/fundamentos/bloque-01-html/01-introduccion-html.md)

**Siguiente:** [Juega con código real](./06-juega-con-codigo.md)

---

## Recursos

- **[MDN: Introducción al HTML](https://developer.mozilla.org/es/docs/Learn/HTML/Introduction_to_HTML)** — Documentación oficial
- **[HTML Cheat Sheet](https://htmlcheatsheet.com/)** — Referencia rápida de etiquetas
- **Consejo:** Mira el código fuente de cualquier página (clic derecho → "Ver código fuente")

---

## Ejercicio de reflexión

Ahora que has creado tu primera página:

1. ¿Qué fue lo más difícil?
2. ¿Qué te sorprendió?
3. ¿Qué quieres aprender ahora?

**Escribir código por primera vez es un momento importante. Guarda este archivo como recuerdo.** 📁

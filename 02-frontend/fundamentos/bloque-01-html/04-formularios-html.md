# Formularios HTML

Hasta ahora has mostrado información. Ahora vamos a **recoger información** del usuario.

**Los formularios son la forma en que las webs conversan contigo.**

---

## ¿Qué vas a aprender?

- La etiqueta `<form>` y cómo funciona
- Tipos de inputs (`text`, `email`, `password`, `number`, etc.)
- Otros elementos: `<textarea>`, `<select>`, `<button>`
- Atributos importantes: `name`, `id`, `placeholder`, `required`
- Validación básica de HTML5

## Por qué es útil

Cada vez que:
- Inicias sesión en una web
- Buscas en Google
- Compras algo online
- Comentas en un post
- Te registras en un servicio

**Estás usando un formulario.**

Sin formularios, la web sería solo lectura. Con formularios, se vuelve interactiva.

---

## La etiqueta `<form>`

Todo formulario empieza con `<form>` y termina con `</form>`.

### Sintaxis básica

```html
<form action="/procesar" method="POST">
    <!-- Aquí van los campos del formulario -->
</form>
```

**Atributos principales:**
- `action` = URL donde se envían los datos
- `method` = Cómo se envían (`GET` o `POST`)

**Por ahora no te preocupes del `action` ni del `method`.** Eso lo verás cuando aprendas backend. Puedes omitirlos mientras practicas.

```html
<form>
    <!-- Formulario simple para práctica -->
</form>
```

---

## Input: El campo más versátil

La etiqueta `<input>` es tu mejor amiga. Con diferentes `type`, se comporta de formas distintas.

### Input de texto

```html
<label for="nombre">Nombre:</label>
<input type="text" id="nombre" name="nombre">
```

**Partes:**
- `<label>` = Etiqueta descriptiva (el texto que explica qué va ahí)
- `for="nombre"` = Conecta el label con el input (deben tener el mismo valor)
- `type="text"` = Campo de texto normal
- `id="nombre"` = Identificador único (para el label y JavaScript)
- `name="nombre"` = Nombre del campo (para enviar al servidor)

**¿Por qué usar `<label>`?**
1. **Accesibilidad:** Los lectores de pantalla lo leen
2. **Usabilidad:** Al hacer clic en el label, el input se activa automáticamente
3. **Buena práctica:** Siempre, siempre usa labels

### Input de email

```html
<label for="email">Email:</label>
<input type="email" id="email" name="email">
```

**Ventaja:** En móviles, muestra un teclado con `@` fácil de acceder. Además, valida automáticamente el formato de email.

### Input de contraseña

```html
<label for="password">Contraseña:</label>
<input type="password" id="password" name="password">
```

**Diferencia:** Oculta el texto con puntos (••••••).

### Input de número

```html
<label for="edad">Edad:</label>
<input type="number" id="edad" name="edad" min="18" max="100">
```

**Atributos útiles:**
- `min` = Valor mínimo permitido
- `max` = Valor máximo permitido
- `step` = Incremento (por ejemplo, `step="5"` permite 5, 10, 15...)

### Input de fecha

```html
<label for="fecha">Fecha de nacimiento:</label>
<input type="date" id="fecha" name="fecha">
```

Muestra un selector de calendario. Muy útil y no tienes que programarlo tú.

### Input de teléfono

```html
<label for="telefono">Teléfono:</label>
<input type="tel" id="telefono" name="telefono">
```

En móviles, muestra el teclado numérico automáticamente.

### Input de URL

```html
<label for="web">Tu sitio web:</label>
<input type="url" id="web" name="web">
```

Valida que sea una URL válida (con `http://` o `https://`).

### Input de búsqueda

```html
<label for="buscar">Buscar:</label>
<input type="search" id="buscar" name="buscar">
```

Similar a `type="text"`, pero con una X para borrar el texto rápidamente.

### Input de color

```html
<label for="color">Elige un color:</label>
<input type="color" id="color" name="color">
```

Muestra un selector de color. ¡Gratis!

### Input de rango (slider)

```html
<label for="volumen">Volumen:</label>
<input type="range" id="volumen" name="volumen" min="0" max="100">
```

Crea un deslizador. Perfecto para volumen, brillo, etc.

---

## Atributos útiles para inputs

### `placeholder` - Texto de ayuda

```html
<input type="text" placeholder="Ej: Juan Pérez">
```

Muestra un texto gris dentro del input que desaparece al escribir.

**No confundir con `<label>`.** El placeholder es **adicional**, no reemplazo del label.

```html
<!-- Bien -->
<label for="nombre">Nombre:</label>
<input type="text" id="nombre" placeholder="Ej: Juan Pérez">

<!-- Mal -->
<input type="text" placeholder="Nombre"> <!-- Sin label -->
```

### `required` - Campo obligatorio

```html
<input type="text" required>
```

El formulario no se enviará si este campo está vacío. El navegador muestra un mensaje de error automáticamente.

### `value` - Valor predefinido

```html
<input type="text" value="Valor por defecto">
```

El input ya viene con ese valor. Útil para editar información existente.

### `readonly` - Solo lectura

```html
<input type="text" value="No puedes cambiar esto" readonly>
```

El usuario puede ver el valor pero no modificarlo.

### `disabled` - Deshabilitado

```html
<input type="text" disabled>
```

El campo aparece gris y no se puede interactuar con él. Tampoco se envía al servidor.

### `maxlength` - Límite de caracteres

```html
<input type="text" maxlength="10">
```

Solo permite escribir hasta 10 caracteres.

### `pattern` - Validación con expresión regular

```html
<input type="text" pattern="[0-9]{4}" title="Debe tener 4 dígitos">
```

Solo acepta valores que coincidan con el patrón (en este caso, 4 números).

---

## Checkbox: Casillas de verificación

Para opciones que se pueden activar/desactivar.

```html
<input type="checkbox" id="acepto" name="acepto">
<label for="acepto">Acepto los términos y condiciones</label>
```

**Nota:** El `<label>` va **después** del checkbox.

### Múltiples checkboxes

```html
<p>¿Qué te gusta?</p>

<input type="checkbox" id="pizza" name="comida" value="pizza">
<label for="pizza">Pizza</label>

<input type="checkbox" id="sushi" name="comida" value="sushi">
<label for="sushi">Sushi</label>

<input type="checkbox" id="tacos" name="comida" value="tacos">
<label for="tacos">Tacos</label>
```

Puedes seleccionar varias opciones a la vez.

### Checkbox marcado por defecto

```html
<input type="checkbox" id="newsletter" checked>
<label for="newsletter">Suscribirse al boletín</label>
```

El atributo `checked` lo marca automáticamente.

---

## Radio buttons: Selección única

Cuando solo puedes elegir **una** opción de varias.

```html
<p>Elige tu plan:</p>

<input type="radio" id="gratis" name="plan" value="gratis">
<label for="gratis">Gratis</label>

<input type="radio" id="basico" name="plan" value="basico">
<label for="basico">Básico (5€/mes)</label>

<input type="radio" id="premium" name="plan" value="premium">
<label for="premium">Premium (10€/mes)</label>
```

**Importante:** Todos los radio buttons del mismo grupo deben tener el **mismo `name`** (en este caso, `"plan"`). Así el navegador sabe que son opciones mutuamente excluyentes.

### Radio button seleccionado por defecto

```html
<input type="radio" id="gratis" name="plan" value="gratis" checked>
<label for="gratis">Gratis</label>
```

---

## Textarea: Texto multilínea

Para textos largos (comentarios, mensajes, etc.).

```html
<label for="mensaje">Mensaje:</label>
<textarea id="mensaje" name="mensaje" rows="5" cols="30"></textarea>
```

**Atributos:**
- `rows` = Número de líneas visibles
- `cols` = Ancho en caracteres

**Nota:** `<textarea>` tiene etiqueta de cierre, a diferencia de `<input>`.

### Textarea con texto predefinido

```html
<textarea id="mensaje">Este es el texto que viene por defecto</textarea>
```

El texto va **entre** las etiquetas, no en un atributo `value`.

### Limitar caracteres en textarea

```html
<textarea maxlength="200" placeholder="Máximo 200 caracteres"></textarea>
```

---

## Select: Menú desplegable

Para elegir una opción de una lista.

```html
<label for="pais">País:</label>
<select id="pais" name="pais">
    <option value="">Selecciona un país</option>
    <option value="es">España</option>
    <option value="mx">México</option>
    <option value="ar">Argentina</option>
    <option value="co">Colombia</option>
</select>
```

**Partes:**
- `<select>` = El menú desplegable
- `<option>` = Cada opción del menú
- `value` = Valor que se envía al servidor
- Texto entre `<option>` = Lo que ve el usuario

### Opción seleccionada por defecto

```html
<select id="pais" name="pais">
    <option value="es" selected>España</option>
    <option value="mx">México</option>
</select>
```

### Agrupar opciones

```html
<label for="comida">Elige tu comida:</label>
<select id="comida" name="comida">
    <optgroup label="Europea">
        <option value="pizza">Pizza</option>
        <option value="pasta">Pasta</option>
    </optgroup>
    <optgroup label="Asiática">
        <option value="sushi">Sushi</option>
        <option value="ramen">Ramen</option>
    </optgroup>
</select>
```

`<optgroup>` crea categorías visibles en el desplegable.

### Selección múltiple

```html
<select id="idiomas" name="idiomas" multiple size="4">
    <option value="es">Español</option>
    <option value="en">Inglés</option>
    <option value="fr">Francés</option>
    <option value="de">Alemán</option>
</select>
```

`multiple` permite seleccionar varias opciones (con Ctrl o Cmd). `size` define cuántas opciones se ven a la vez.

---

## Button: Botones

### Button de submit (enviar)

```html
<button type="submit">Enviar</button>
```

Envía el formulario. Es el comportamiento por defecto de `<button>` dentro de un `<form>`.

### Button normal

```html
<button type="button">Hacer algo</button>
```

No envía el formulario. Útil para JavaScript.

### Button de reset (limpiar)

```html
<button type="reset">Limpiar formulario</button>
```

Borra todos los campos del formulario y los vuelve a sus valores por defecto.

### Input como botón (alternativa antigua)

```html
<input type="submit" value="Enviar">
<input type="button" value="Hacer algo">
<input type="reset" value="Limpiar">
```

Funcionan igual que `<button>`, pero `<button>` es más moderno y flexible (puede contener HTML como iconos).

**Recomendación:** Usa `<button>`, no `<input type="button">`.

---

## Fieldset y Legend: Agrupar campos

Para organizar visualmente campos relacionados.

```html
<form>
    <fieldset>
        <legend>Datos personales</legend>
        
        <label for="nombre">Nombre:</label>
        <input type="text" id="nombre" name="nombre">
        
        <label for="email">Email:</label>
        <input type="email" id="email" name="email">
    </fieldset>
    
    <fieldset>
        <legend>Preferencias</legend>
        
        <input type="checkbox" id="news" name="news">
        <label for="news">Recibir noticias</label>
    </fieldset>
    
    <button type="submit">Enviar</button>
</form>
```

**Resultado:** Los campos se agrupan dentro de un recuadro visual con un título (`<legend>`).

---

## Formulario completo: Registro de usuario

Juntemos todo:

```html
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <title>Registro</title>
</head>
<body>
    <h1>Crea tu cuenta</h1>
    
    <form>
        <fieldset>
            <legend>Información personal</legend>
            
            <label for="nombre">Nombre completo:</label>
            <input type="text" id="nombre" name="nombre" required placeholder="Ej: Ana López">
            <br><br>
            
            <label for="email">Email:</label>
            <input type="email" id="email" name="email" required placeholder="tu@email.com">
            <br><br>
            
            <label for="password">Contraseña:</label>
            <input type="password" id="password" name="password" required minlength="8">
            <br><br>
            
            <label for="fecha">Fecha de nacimiento:</label>
            <input type="date" id="fecha" name="fecha" required>
            <br><br>
            
            <label for="telefono">Teléfono:</label>
            <input type="tel" id="telefono" name="telefono" placeholder="+34 600 000 000">
            <br><br>
        </fieldset>
        
        <fieldset>
            <legend>Preferencias</legend>
            
            <label for="pais">País:</label>
            <select id="pais" name="pais" required>
                <option value="">Selecciona tu país</option>
                <option value="es">España</option>
                <option value="mx">México</option>
                <option value="ar">Argentina</option>
                <option value="co">Colombia</option>
            </select>
            <br><br>
            
            <p>¿Cómo te enteraste de nosotros?</p>
            <input type="radio" id="buscador" name="origen" value="buscador">
            <label for="buscador">Buscador (Google, etc.)</label>
            <br>
            <input type="radio" id="redes" name="origen" value="redes">
            <label for="redes">Redes sociales</label>
            <br>
            <input type="radio" id="amigo" name="origen" value="amigo">
            <label for="amigo">Un amigo me lo recomendó</label>
            <br><br>
            
            <input type="checkbox" id="newsletter" name="newsletter">
            <label for="newsletter">Quiero recibir el boletín semanal</label>
            <br><br>
        </fieldset>
        
        <fieldset>
            <legend>Comentarios (opcional)</legend>
            
            <label for="comentarios">¿Algo que quieras decirnos?</label><br>
            <textarea id="comentarios" name="comentarios" rows="4" cols="50" placeholder="Escribe aquí..."></textarea>
        </fieldset>
        
        <br>
        
        <input type="checkbox" id="terminos" name="terminos" required>
        <label for="terminos">Acepto los términos y condiciones</label>
        
        <br><br>
        
        <button type="submit">Crear cuenta</button>
        <button type="reset">Limpiar formulario</button>
    </form>
</body>
</html>
```

**Guarda esto como `registro.html` y ábrelo.**

Intenta enviar el formulario sin rellenar campos obligatorios. El navegador te avisará automáticamente.

---

## Validación HTML5

HTML5 valida automáticamente algunos campos. No necesitas JavaScript para esto.

### Validaciones automáticas

```html
<!-- Email válido -->
<input type="email" required>

<!-- URL válida -->
<input type="url" required>

<!-- Número en rango -->
<input type="number" min="1" max="10" required>

<!-- Mínimo de caracteres -->
<input type="password" minlength="8" required>

<!-- Máximo de caracteres -->
<input type="text" maxlength="50">

<!-- Patrón personalizado -->
<input type="text" pattern="[A-Za-z]{3}" title="Debe tener 3 letras">
```

### Mensajes de error personalizados (con JavaScript)

Por ahora solo verás los mensajes por defecto del navegador. Más adelante aprenderás a personalizarlos con JavaScript.

---

## Accesibilidad en formularios

### 1. Siempre usa `<label>`

```html
<!-- Bien -->
<label for="nombre">Nombre:</label>
<input type="text" id="nombre">

<!-- Mal -->
<span>Nombre:</span>
<input type="text"> <!-- Sin label ni conexión -->
```

### 2. Usa `placeholder` como ayuda, no como reemplazo

```html
<!-- Bien -->
<label for="email">Email:</label>
<input type="email" id="email" placeholder="tu@email.com">

<!-- Mal -->
<input type="email" placeholder="Email"> <!-- Sin label -->
```

### 3. Agrupa campos relacionados con `<fieldset>`

```html
<fieldset>
    <legend>Dirección de envío</legend>
    <!-- Campos de dirección -->
</fieldset>
```

### 4. Indica campos obligatorios

```html
<label for="nombre">Nombre <span style="color: red;">*</span></label>
<input type="text" id="nombre" required>
```

O al principio del formulario:
```html
<p><span style="color: red;">*</span> Campos obligatorios</p>
```

---

## Errores comunes

### Error 1: No conectar `<label>` con `<input>`

```html
<!-- Mal -->
<label>Nombre</label>
<input type="text" id="nombre">

<!-- Bien -->
<label for="nombre">Nombre</label>
<input type="text" id="nombre">
```

Sin el `for` y el `id` coincidentes, no están conectados.

### Error 2: Olvidar el atributo `name`

```html
<!-- Mal (no se enviará al servidor) -->
<input type="text" id="nombre">

<!-- Bien -->
<input type="text" id="nombre" name="nombre">
```

El `id` es para el navegador. El `name` es para el servidor. Necesitas ambos.

### Error 3: Radio buttons sin el mismo `name`

```html
<!-- Mal -->
<input type="radio" id="si" name="respuesta1">
<input type="radio" id="no" name="respuesta2">

<!-- Bien -->
<input type="radio" id="si" name="respuesta">
<input type="radio" id="no" name="respuesta">
```

Si no comparten `name`, puedes seleccionar ambos (cuando debería ser excluyente).

### Error 4: Botón fuera del `<form>`

```html
<!-- Mal -->
<form>
    <input type="text" name="nombre">
</form>
<button type="submit">Enviar</button> <!-- Fuera del form -->

<!-- Bien -->
<form>
    <input type="text" name="nombre">
    <button type="submit">Enviar</button> <!-- Dentro del form -->
</form>
```

El botón debe estar **dentro** del `<form>` para enviar el formulario.

### Error 5: Usar `<br>` para espaciar campos

```html
<!-- Mal -->
<label>Nombre:</label>
<input type="text">
<br><br><br><br>
<label>Email:</label>

<!-- Mejor (con CSS más adelante) -->
<div>
    <label>Nombre:</label>
    <input type="text">
</div>
<div>
    <label>Email:</label>
    <input type="email">
</div>
```

Usa CSS para espaciar, no múltiples `<br>`.

---

## Buenas prácticas

### 1. Orden lógico de campos

```html
<!-- Bien -->
<form>
    <input type="text" name="nombre"> <!-- Primero lo básico -->
    <input type="email" name="email">
    <input type="password" name="password">
    <textarea name="comentarios"></textarea> <!-- Lo opcional al final -->
    <button type="submit">Enviar</button>
</form>
```

Los campos más importantes primero. Los opcionales al final.

### 2. Agrupa campos relacionados

```html
<fieldset>
    <legend>Dirección</legend>
    <input type="text" name="calle">
    <input type="text" name="ciudad">
    <input type="text" name="codigo_postal">
</fieldset>
```

### 3. Usa el `type` correcto

No uses `type="text"` para todo. Usa `email`, `tel`, `url`, `number`, etc. según corresponda.

### 4. Proporciona feedback visual

Aunque HTML5 valida, es mejor dar pistas visuales con CSS (que aprenderás pronto):
- Borde rojo para campos con error
- Borde verde para campos válidos
- Iconos de check/cross

### 5. No pidas información innecesaria

**Mal:**
```html
<!-- ¿Por qué necesitas mi dirección completa para un newsletter? -->
<input name="calle">
<input name="ciudad">
<input name="codigo_postal">
```

**Bien:**
```html
<!-- Solo lo necesario -->
<input type="email" name="email">
```

Respeta la privacidad. Pide solo lo que realmente necesitas.

---

## Ejercicios prácticos

### Ejercicio 1: Formulario de contacto

Crea `contacto.html` con un formulario que tenga:
- Nombre (requerido)
- Email (requerido, tipo email)
- Asunto (requerido)
- Mensaje (textarea, requerido)
- Botón de enviar

<details>
<summary>✅ Solución ejemplo</summary>

```html
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <title>Contacto</title>
</head>
<body>
    <h1>Contáctanos</h1>
    
    <form>
        <label for="nombre">Nombre:</label>
        <input type="text" id="nombre" name="nombre" required>
        <br><br>
        
        <label for="email">Email:</label>
        <input type="email" id="email" name="email" required>
        <br><br>
        
        <label for="asunto">Asunto:</label>
        <input type="text" id="asunto" name="asunto" required>
        <br><br>
        
        <label for="mensaje">Mensaje:</label><br>
        <textarea id="mensaje" name="mensaje" rows="5" cols="40" required></textarea>
        <br><br>
        
        <button type="submit">Enviar mensaje</button>
    </form>
</body>
</html>
```

</details>

---

### Ejercicio 2: Formulario de pedido de pizza

Crea `pizza.html` con:
- Nombre del cliente (requerido)
- Teléfono (tipo tel, requerido)
- Tamaño de pizza (select: pequeña, mediana, grande)
- Ingredientes extra (3 checkboxes: jamón, champiñones, aceitunas)
- ¿Recoger o entregar? (2 radio buttons)
- Comentarios adicionales (textarea opcional)
- Botón de realizar pedido

<details>
<summary>✅ Solución ejemplo</summary>

```html
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <title>Pedido de Pizza</title>
</head>
<body>
    <h1>Haz tu pedido</h1>
    
    <form>
        <fieldset>
            <legend>Datos del cliente</legend>
            
            <label for="nombre">Nombre:</label>
            <input type="text" id="nombre" name="nombre" required>
            <br><br>
            
            <label for="telefono">Teléfono:</label>
            <input type="tel" id="telefono" name="telefono" required>
            <br><br>
        </fieldset>
        
        <fieldset>
            <legend>Tu pizza</legend>
            
            <label for="tamano">Tamaño:</label>
            <select id="tamano" name="tamano" required>
                <option value="">Selecciona</option>
                <option value="pequena">Pequeña (8€)</option>
                <option value="mediana">Mediana (12€)</option>
                <option value="grande">Grande (15€)</option>
            </select>
            <br><br>
            
            <p>Ingredientes extra (+1€ cada uno):</p>
            <input type="checkbox" id="jamon" name="extras" value="jamon">
            <label for="jamon">Jamón</label>
            <br>
            <input type="checkbox" id="champinones" name="extras" value="champinones">
            <label for="champinones">Champiñones</label>
            <br>
            <input type="checkbox" id="aceitunas" name="extras" value="aceitunas">
            <label for="aceitunas">Aceitunas</label>
            <br><br>
            
            <p>Método de entrega:</p>
            <input type="radio" id="recoger" name="entrega" value="recoger" required>
            <label for="recoger">Recoger en local</label>
            <br>
            <input type="radio" id="domicilio" name="entrega" value="domicilio">
            <label for="domicilio">Entregar a domicilio (+2€)</label>
            <br><br>
        </fieldset>
        
        <label for="comentarios">Comentarios adicionales:</label><br>
        <textarea id="comentarios" name="comentarios" rows="3" cols="40" placeholder="Ej: Sin cebolla, por favor"></textarea>
        <br><br>
        
        <button type="submit">Realizar pedido</button>
    </form>
</body>
</html>
```

</details>

---

### Ejercicio 3: Formulario de encuesta

Crea `encuesta.html` con:
- ¿Cómo calificarías nuestro servicio? (radio: Excelente, Bueno, Regular, Malo)
- ¿Qué aspectos te gustaron? (checkboxes: Atención, Rapidez, Precio, Calidad)
- ¿Nos recomendarías? (select: Sí, Tal vez, No)
- Sugerencias (textarea)
- Edad (number, min 18, max 100)
- Botón de enviar

<details>
<summary>✅ Solución ejemplo</summary>

```html
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <title>Encuesta de satisfacción</title>
</head>
<body>
    <h1>Encuesta de satisfacción</h1>
    <p>Ayúdanos a mejorar respondiendo esta breve encuesta.</p>
    
    <form>
        <fieldset>
            <legend>Tu opinión</legend>
            
            <p>¿Cómo calificarías nuestro servicio?</p>
            <input type="radio" id="excelente" name="calificacion" value="excelente" required>
            <label for="excelente">Excelente</label>
            <br>
            <input type="radio" id="bueno" name="calificacion" value="bueno">
            <label for="bueno">Bueno</label>
            <br>
            <input type="radio" id="regular" name="calificacion" value="regular">
            <label for="regular">Regular</label>
            <br>
            <input type="radio" id="malo" name="calificacion" value="malo">
            <label for="malo">Malo</label>
            <br><br>
            
            <p>¿Qué aspectos te gustaron? (puedes elegir varios)</p>
            <input type="checkbox" id="atencion" name="aspectos" value="atencion">
            <label for="atencion">Atención al cliente</label>
            <br>
            <input type="checkbox" id="rapidez" name="aspectos" value="rapidez">
            <label for="rapidez">Rapidez</label>
            <br>
            <input type="checkbox" id="precio" name="aspectos" value="precio">
            <label for="precio">Precio</label>
            <br>
            <input type="checkbox" id="calidad" name="aspectos" value="calidad">
            <label for="calidad">Calidad</label>
            <br><br>
            
            <label for="recomendar">¿Nos recomendarías?</label>
            <select id="recomendar" name="recomendar" required>
                <option value="">Selecciona una opción</option>
                <option value="si">Sí</option>
                <option value="tal_vez">Tal vez</option>
                <option value="no">No</option>
            </select>
            <br><br>
            
            <label for="sugerencias">Sugerencias de mejora:</label><br>
            <textarea id="sugerencias" name="sugerencias" rows="4" cols="50"></textarea>
            <br><br>
            
            <label for="edad">Edad:</label>
            <input type="number" id="edad" name="edad" min="18" max="100">
            <br><br>
        </fieldset>
        
        <button type="submit">Enviar encuesta</button>
    </form>
    
    <p><small>Gracias por tu tiempo. Tu opinión es muy valiosa para nosotros.</small></p>
</body>
</html>
```

</details>

---

### Ejercicio 4: Detectar y corregir errores

Este formulario tiene 5 errores. Encuéntralos:

```html
<form>
    <label>Nombre:</label>
    <input type="text" id="nombre">
    
    <label for="email">Email</label>
    <input type="text" id="email" name="email">
    
    <input type="radio" id="si" name="respuesta1" value="si">
    <label for="si">Sí</label>
    <input type="radio" id="no" name="respuesta2" value="no">
    <label for="no">No</label>
    
    <button>Enviar</button>
</form>
```

<details>
<summary>💡 Pistas</summary>

- ¿El label está conectado al input?
- ¿El input tiene el `name`?
- ¿El tipo de input es correcto?
- ¿Los radio buttons están bien configurados?

</details>

<details>
<summary>✅ Soluciones</summary>

**Errores:**
1. El primer `<label>` no tiene `for`, no está conectado al input
2. El primer `<input>` no tiene atributo `name`
3. El input de email es `type="text"` en vez de `type="email"`
4. Los radio buttons tienen diferentes `name` (deberían compartir el mismo)
5. El botón no especifica `type="submit"` (aunque funciona por defecto, es mejor ser explícito)

**Correcto:**
```html
<form>
    <label for="nombre">Nombre:</label>
    <input type="text" id="nombre" name="nombre">
    
    <label for="email">Email:</label>
    <input type="email" id="email" name="email">
    
    <input type="radio" id="si" name="respuesta" value="si">
    <label for="si">Sí</label>
    <input type="radio" id="no" name="respuesta" value="no">
    <label for="no">No</label>
    
    <button type="submit">Enviar</button>
</form>
```

</details>

---

## Recursos adicionales

- [MDN: Formularios web](https://developer.mozilla.org/es/docs/Learn/Forms)
- [HTML5 Input Types](https://www.w3schools.com/html/html_form_input_types.asp)
- [Can I Use - Compatibilidad de inputs](https://caniuse.com/)

---

## Siguiente paso

Ya sabes crear formularios completos. El último tema de HTML: tablas.

→ [05-tablas-html.md](05-tablas-html.md)

Ahí aprenderás `<table>`, `<thead>`, `<tbody>`, `<tr>`, `<td>` y cuándo (y cuándo NO) usar tablas.

---

**Recuerda:** Los formularios son la conversación de tu web con el usuario. Hazlos simples, claros, accesibles. Pide solo lo necesario. Y siempre usa labels.

**Prueba tus formularios** antes de publicarlos. Intenta rellenarlos como si fueras el usuario. Si encuentras algo confuso, probablemente lo sea.

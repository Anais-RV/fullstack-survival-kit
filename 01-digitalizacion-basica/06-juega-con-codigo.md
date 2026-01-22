# Juega con código

> **Experimenta, modifica, rompe y arregla (así se aprende de verdad)**

---

## El secreto del aprendizaje: tocar

**Leer sobre programación ≠ Saber programar**

Es como:
- Leer sobre nadar ≠ Saber nadar
- Ver vídeos de cocina ≠ Saber cocinar
- Estudiar piano ≠ Saber tocar

**Tienes que meter las manos.**

Este módulo es 100% práctica: Vas a tomar código que funciona y cambiarlo para ver qué pasa.

---

## Ejercicio 1: Modifica estilos

Crea un archivo `experimento-1.html` con este código:

```html
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <title>Experimento 1</title>
    <style>
        body {
            background-color: white;
            color: black;
            font-family: Arial;
            font-size: 16px;
        }
        h1 {
            color: blue;
            font-size: 32px;
        }
    </style>
</head>
<body>
    <h1>Experimenta conmigo</h1>
    <p>Este es un párrafo de ejemplo.</p>
</body>
</html>
```

### Ahora experimenta:

#### 1. Cambia el color de fondo

Busca:
```css
background-color: white;
```

Prueba estos valores (uno por uno, guarda y recarga):
- `lightblue`
- `#FF6B6B` (un rojo suave)
- `rgb(100, 200, 100)` (un verde)

#### 2. Cambia el tamaño de letra

Busca:
```css
font-size: 16px;
```

Prueba:
- `12px` (pequeño)
- `24px` (grande)
- `10px` (muy pequeño)

#### 3. Cambia la fuente

Busca:
```css
font-family: Arial;
```

Prueba:
- `Courier New` (tipo máquina de escribir)
- `Georgia` (con serifas)
- `Comic Sans MS` (informal)
- `Verdana`

#### 4. Cambia el color del título

Busca:
```css
h1 {
    color: blue;
}
```

Prueba:
- `red`
- `green`
- `purple`
- `#FF0000` (rojo en hexadecimal)

### Preguntas guía:

- ¿Qué combinación de colores se ve mejor?
- ¿Qué tamaño de letra es más cómodo de leer?
- ¿Cómo afecta la fuente a la sensación de la página?

---

## Ejercicio 2: Rompe el código (a propósito)

**Objetivo:** Ver qué errores producen qué resultados.

Copia el código del ejercicio 1. Ahora:

### Experimento 1: Olvida cerrar una etiqueta

Cambia:
```html
<h1>Experimenta conmigo</h1>
```

Por:
```html
<h1>Experimenta conmigo
```

**¿Qué pasa?** El navegador intenta "adivinar" y aplica el estilo de h1 a todo.

**Arréglalo** añadiendo `</h1>` de nuevo.

### Experimento 2: Escribe mal una etiqueta

Cambia:
```html
<p>Este es un párrafo de ejemplo.</p>
```

Por:
```html
<parrafo>Este es un párrafo de ejemplo.</parrafo>
```

**¿Qué pasa?** El navegador no conoce `<parrafo>`, así que lo ignora (pero muestra el texto).

**Arréglalo** volviendo a `<p>`.

### Experimento 3: Olvida comillas en CSS

Cambia:
```css
font-family: "Courier New";
```

Por:
```css
font-family: Courier New;
```

**¿Qué pasa?** CSS intenta usar "Courier" pero puede que no funcione correctamente.

**Lección:** Los nombres de fuentes con espacios necesitan comillas.

### Experimento 4: Typo en CSS

Cambia:
```css
background-color: lightblue;
```

Por:
```css
bakground-color: lightblue;
```

**¿Qué pasa?** CSS ignora la regla incorrecta. El fondo queda blanco.

**Arréglalo** corrigiendo `background`.

---

## Ejercicio 3: Añade interactividad básica

Crea `experimento-2.html`:

```html
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <title>Experimento 2</title>
</head>
<body>
    <h1>Haz clic en el botón</h1>
    
    <button onclick="alert('¡Hola! Has hecho clic')">
        Haz clic aquí
    </button>
    
    <p id="texto">Este texto cambiará</p>
    
    <button onclick="document.getElementById('texto').textContent = 'Texto cambiado!'">
        Cambiar texto
    </button>
</body>
</html>
```

### Ahora experimenta:

#### 1. Cambia el mensaje del alert

Busca:
```javascript
alert('¡Hola! Has hecho clic')
```

Cambia el mensaje entre comillas por el que quieras.

#### 2. Cambia el nuevo texto

Busca:
```javascript
'Texto cambiado!'
```

Pon lo que quieras.

#### 3. Añade un tercer botón

Copia y pega uno de los botones, modifica el texto y la acción.

```html
<button onclick="alert('Este es el tercer botón')">
    Tercer botón
</button>
```

---

## Ejercicio 4: Cambia colores con botones

Crea `experimento-3.html`:

```html
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <title>Experimento 3: Colores</title>
    <style>
        body {
            padding: 20px;
            transition: background-color 0.5s;
        }
        button {
            font-size: 18px;
            padding: 10px 20px;
            margin: 5px;
            cursor: pointer;
        }
    </style>
</head>
<body>
    <h1>Cambia el color de fondo</h1>
    
    <button onclick="document.body.style.backgroundColor = 'lightblue'">
        Azul
    </button>
    
    <button onclick="document.body.style.backgroundColor = 'lightcoral'">
        Coral
    </button>
    
    <button onclick="document.body.style.backgroundColor = 'lightgreen'">
        Verde
    </button>
    
    <button onclick="document.body.style.backgroundColor = 'lightyellow'">
        Amarillo
    </button>
    
    <button onclick="document.body.style.backgroundColor = 'white'">
        Blanco
    </button>
</body>
</html>
```

### Ahora experimenta:

#### 1. Añade más botones con más colores

Copia un botón y cambia:
- El texto del botón
- El color entre comillas

Prueba colores:
- `lavender`
- `peachpuff`
- `mistyrose`
- `#FF6B6B`

#### 2. Cambia el color del título en vez del fondo

Añade un `id` al `<h1>`:
```html
<h1 id="titulo">Cambia el color de fondo</h1>
```

Crea un nuevo botón:
```html
<button onclick="document.getElementById('titulo').style.color = 'red'">
    Título rojo
</button>
```

#### 3. Combina: fondo Y título

```html
<button onclick="document.body.style.backgroundColor = 'black'; document.getElementById('titulo').style.color = 'white'">
    Modo oscuro
</button>
```

---

## Ejercicio 5: Contador simple

Crea `experimento-4.html`:

```html
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <title>Experimento 4: Contador</title>
    <style>
        body {
            text-align: center;
            padding: 50px;
            font-family: Arial;
        }
        #numero {
            font-size: 72px;
            margin: 30px 0;
        }
        button {
            font-size: 24px;
            padding: 15px 30px;
            margin: 10px;
            cursor: pointer;
        }
    </style>
</head>
<body>
    <h1>Contador</h1>
    
    <div id="numero">0</div>
    
    <button onclick="incrementar()">➕ Sumar</button>
    <button onclick="decrementar()">➖ Restar</button>
    <button onclick="resetear()">🔄 Reiniciar</button>
    
    <script>
        let contador = 0;
        
        function incrementar() {
            contador = contador + 1;
            document.getElementById('numero').textContent = contador;
        }
        
        function decrementar() {
            contador = contador - 1;
            document.getElementById('numero').textContent = contador;
        }
        
        function resetear() {
            contador = 0;
            document.getElementById('numero').textContent = contador;
        }
    </script>
</body>
</html>
```

### Ahora experimenta:

#### 1. Cambia cuánto suma/resta

En `incrementar()`, cambia:
```javascript
contador = contador + 1;
```

Por:
```javascript
contador = contador + 5;
```

Ahora suma de 5 en 5.

#### 2. Añade un botón de suma grande

```html
<button onclick="sumaGrande()">➕➕ +10</button>
```

Y en el `<script>`:
```javascript
function sumaGrande() {
    contador = contador + 10;
    document.getElementById('numero').textContent = contador;
}
```

#### 3. Cambia el color según el número

Modifica `incrementar()`:
```javascript
function incrementar() {
    contador = contador + 1;
    document.getElementById('numero').textContent = contador;
    
    if (contador > 10) {
        document.getElementById('numero').style.color = 'red';
    } else {
        document.getElementById('numero').style.color = 'black';
    }
}
```

---

## Reflexión: ¿Qué has aprendido?

Sin darte cuenta, has:

- ✅ Modificado CSS (colores, tamaños, fuentes)
- ✅ Visto cómo funcionan los errores
- ✅ Añadido interactividad con JavaScript
- ✅ Usado funciones (sin saberlo)
- ✅ Manipulado el DOM (sin saberlo)
- ✅ Trabajado con eventos (clics)

**No importa que no entiendas el 100%.** Lo importante es que **experimentaste**.

---

## Consejos para seguir experimentando

### 1. Copia código de otros

Busca "HTML CSS ejemplos" en Google. Copia, pégalo, modifícalo.

**No es trampa.** Así empiezan todos.

### 2. Rompe código a propósito

La mejor forma de aprender qué hace cada cosa: bórrala y mira qué pasa.

### 3. Pregunta "¿Qué pasa si...?"

- ¿Qué pasa si duplico este botón?
- ¿Qué pasa si cambio este número?
- ¿Qué pasa si pongo texto donde va un número?

**Experimenta sin miedo.**

### 4. Usa herramientas online

- **[CodePen](https://codepen.io/)** — Escribe HTML/CSS/JS y ve el resultado en tiempo real
- **[JSFiddle](https://jsfiddle.net/)** — Similar a CodePen
- **[JS Bin](https://jsbin.com/)** — Otro editor online

Son perfectos para experimentos rápidos.

---

## Desafío final: Crea tu propia página de experimento

Crea un archivo `mi-experimento.html` que tenga:

- Un título
- Al menos 3 botones que hagan cosas diferentes
- Estilos CSS personalizados
- Alguna interacción con JavaScript

**No copies exactamente** los ejemplos. Modifícalos. Hazlo tuyo.

---

## Conclusión

**Aprender a programar es:**
- 20% leer
- 80% practicar

No tengas miedo de:
- Romper cosas (se arreglan)
- Probar ideas locas
- No entender todo
- Buscar en Google (todos lo hacemos)

**Cada experimento, aunque falle, es aprendizaje.**

**Siguiente:** [Herramientas del desarrollador](./07-herramientas-desarrollador.md)

---

## Recursos para seguir jugando

- **[W3Schools Try It Yourself](https://www.w3schools.com/html/tryit.asp)** — Editor online con ejemplos
- **[CSS Tricks](https://css-tricks.com/)** — Tutoriales y trucos de CSS
- **[JavaScript30](https://javascript30.com/)** — 30 proyectos en JavaScript (más adelante)

---

## Recuerda

> "El código no se rompe de verdad. Solo deja de funcionar temporalmente."

Experimenta, juega, diviértete. Esa es la forma real de aprender. 🚀

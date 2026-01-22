# Herramientas del desarrollador

> **El superpoder secreto de todo programador web**

---

## ¿Qué son las DevTools?

**DevTools** (Herramientas de Desarrollo) son un conjunto de herramientas incluidas en todos los navegadores modernos que te permiten:

- 🔍 Ver el código de cualquier página web
- ✏️ Modificar esa página en tiempo real (solo tú lo ves)
- 🐛 Encontrar errores en tu código
- 📊 Ver cómo se carga la página
- 📱 Simular móviles y tablets
- 🎨 Experimentar con estilos CSS

**Son gratis, están en tu navegador ahora mismo, y son increíblemente poderosas.**

---

## Cómo abrir las DevTools

### Windows / Linux:

- **F12**
- **Ctrl + Shift + I**
- **Ctrl + Shift + C** (modo inspección directa)
- Clic derecho en cualquier parte → "Inspeccionar"

### Mac:

- **Cmd + Option + I**
- **Cmd + Option + C** (modo inspección)
- Clic derecho → "Inspeccionar elemento"

---

## Ejercicio 1: Abre las DevTools ahora mismo

1. Crea un archivo simple `devtools-practica.html`:

```html
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <title>Practicando DevTools</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            padding: 20px;
            background-color: lightblue;
        }
        h1 {
            color: darkblue;
        }
        .caja {
            background-color: white;
            padding: 20px;
            margin: 20px 0;
            border-radius: 10px;
        }
    </style>
</head>
<body>
    <h1>Mi página de práctica</h1>
    
    <div class="caja">
        <h2>Sección 1</h2>
        <p>Este es un párrafo dentro de una caja.</p>
    </div>
    
    <div class="caja">
        <h2>Sección 2</h2>
        <p>Este es otro párrafo.</p>
    </div>
    
    <button onclick="console.log('¡Botón clicado!')">
        Haz clic aquí
    </button>
</body>
</html>
```

2. Abre este archivo en tu navegador
3. Presiona **F12** (o el método de tu sistema)

**¿Ves un panel que aparece?** Esas son las DevTools. 🎉

---

## Las pestañas principales

### 1. Elements (Elementos)

**Qué muestra:** El código HTML y CSS de la página.

**Para qué sirve:**
- Ver la estructura de la página
- Modificar HTML en tiempo real
- Cambiar estilos CSS temporalmente
- Ver qué estilos se aplican a cada elemento

### 2. Console (Consola)

**Qué muestra:** Mensajes, errores y te permite escribir código JavaScript.

**Para qué sirve:**
- Ver errores
- Probar código JavaScript rápido
- Ver mensajes de debug (`console.log()`)

### 3. Sources (Fuentes)

**Qué muestra:** Todos los archivos de la página.

**Para qué sirve:**
- Ver archivos JavaScript
- Debuggear código (pausar ejecución)
- Modificar y guardar cambios localmente

### 4. Network (Red)

**Qué muestra:** Todas las peticiones que hace la página.

**Para qué sirve:**
- Ver qué archivos se cargan
- Cuánto tardan
- Si alguna petición falla

Hay más pestañas, pero estas son las más importantes para empezar.

---

## Ejercicio 2: Inspeccionar elementos

Con las DevTools abiertas en tu `devtools-practica.html`:

### Paso 1: Selecciona el ícono de inspección

En la esquina superior izquierda de las DevTools, verás un ícono de cursor sobre un cuadrado.  
Haz clic en él (o presiona Ctrl+Shift+C / Cmd+Option+C).

### Paso 2: Pasa el cursor sobre elementos de la página

Al mover el cursor sobre la página, verás que cada elemento se resalta con un color.

**Haz clic en el título "Mi página de práctica".**

En las DevTools verás:
```html
<h1>Mi página de práctica</h1>
```

Y a la derecha, los estilos CSS aplicados:
```css
h1 {
    color: darkblue;
}
```

### Paso 3: Modifica el texto

En las DevTools, haz doble clic en "Mi página de práctica" y cámbialo por:
```html
<h1>Título modificado desde DevTools</h1>
```

**¡El texto en la página cambia!**

**Nota:** Esto solo lo ves tú. Si recargas la página, vuelve al original.

---

## Ejercicio 3: Modifica estilos en tiempo real

### Paso 1: Selecciona un elemento

Inspecciona una de las cajas blancas (`.caja`).

### Paso 2: Ve los estilos aplicados

A la derecha verás:
```css
.caja {
    background-color: white;
    padding: 20px;
    margin: 20px 0;
    border-radius: 10px;
}
```

### Paso 3: Cambia un valor

Haz clic en `white` (junto a `background-color`) y cámbialo a `lightcoral`.

**¡La caja cambia de color instantáneamente!**

### Paso 4: Añade una nueva propiedad

Haz clic en el espacio vacío dentro de `.caja` y escribe:
```css
border: 3px solid darkblue;
```

**¡Aparece un borde!**

### Paso 5: Activa/desactiva propiedades

Verás checkboxes junto a cada propiedad. Desmarca `border-radius`.

**¡Las esquinas redondeadas desaparecen!**

---

## Ejercicio 4: La Consola

### Paso 1: Ve a la pestaña "Console"

### Paso 2: Escribe código JavaScript

Escribe esto y presiona Enter:
```javascript
2 + 2
```

Responde: `4`

Ahora escribe:
```javascript
alert("Hola desde la consola")
```

**¡Aparece un alert!**

### Paso 3: Manipula la página desde la consola

Escribe:
```javascript
document.body.style.backgroundColor = "lavender"
```

**El fondo cambia.**

Escribe:
```javascript
document.querySelector('h1').textContent = "Cambiado desde consola"
```

**El título cambia.**

### Paso 4: Prueba el botón

Haz clic en el botón de tu página.

En la consola verás:
```
¡Botón clicado!
```

Este mensaje viene de:
```javascript
console.log('¡Botón clicado!')
```

**`console.log()` es tu mejor amigo para debuggear.**

---

## Ejercicio 5: Ver errores

Añade este botón a tu HTML:

```html
<button onclick="funcionQueNoExiste()">
    Botón con error
</button>
```

Haz clic en ese botón.

En la consola verás un error en **rojo**:
```
Uncaught ReferenceError: funcionQueNoExiste is not defined
```

**Esto es súper útil.** Te dice exactamente qué está mal.

---

## Ejercicio 6: Modo responsive (simular móviles)

### Paso 1: Activa el modo responsive

Haz clic en el ícono de móvil/tablet en DevTools (o presiona Ctrl+Shift+M / Cmd+Shift+M).

### Paso 2: Elige un dispositivo

En el menú desplegable superior, elige "iPhone 12 Pro" o "iPad".

**¡Tu página se ve como en ese dispositivo!**

### Paso 3: Cambia orientación

Haz clic en el ícono de rotación para ver cómo se ve en horizontal.

**Esto es esencial para diseño responsive.**

---

## Trucos útiles

### Truco 1: Copia HTML de cualquier elemento

Clic derecho en un elemento en DevTools → "Copy" → "Copy element".

Ahora puedes pegarlo en tu código.

### Truco 2: Toma capturas de pantalla

Abre DevTools → Ctrl+Shift+P (Cmd+Shift+P) → Escribe "screenshot" → Elige:
- "Capture screenshot" (pantalla completa)
- "Capture full size screenshot" (página completa, incluso lo que no se ve)
- "Capture node screenshot" (solo un elemento)

### Truco 3: Ve todos los colores usados

En la pestaña Elements, cuando veas un color, haz clic en el cuadradito de color.

Aparece un selector de color interactivo.

### Truco 4: Encuentra qué CSS está afectando un elemento

Selecciona un elemento. A la derecha verás **todos** los estilos aplicados, incluso los que vienen del navegador.

Los que están ~~tachados~~ están siendo sobrescritos por otros.

---

## Ejercicio desafío: Hackea una web famosa

1. Ve a **wikipedia.org** (o cualquier página)
2. Abre DevTools (F12)
3. Inspecciona el logo de Wikipedia
4. Cambia el texto
5. Cambia colores de la página
6. Modifica el contenido

**Recuerda:** Solo lo ves tú. No estás hackeando de verdad. Si recargas, todo vuelve a la normalidad.

**Pero es una forma divertida de practicar.**

---

## Conceptos clave

### 1. Las DevTools no modifican el código fuente

Solo modifican lo que ves en el navegador **temporalmente**.

Para hacer cambios permanentes, edita tus archivos `.html` y `.css`.

### 2. Todos los navegadores tienen DevTools

Chrome, Firefox, Safari, Edge... todos.

La interfaz puede cambiar un poco, pero la funcionalidad es la misma.

### 3. Los profesionales usan DevTools constantemente

No es una herramienta solo para principiantes. Los desarrolladores expertos las usan todo el día.

### 4. No tengas miedo de romper cosas

Estás en DevTools. No puedes romper nada permanentemente. Recarga y todo vuelve.

---

## Comandos útiles de consola

### Ver un elemento

```javascript
document.querySelector('h1')
```

### Cambiar contenido

```javascript
document.querySelector('h1').textContent = "Nuevo texto"
```

### Cambiar estilos

```javascript
document.querySelector('h1').style.color = "red"
```

### Ver todos los elementos de un tipo

```javascript
document.querySelectorAll('p')
```

### Limpiar la consola

```javascript
clear()
```

o simplemente `Ctrl+L` / `Cmd+K`

---

## Atajos de teclado útiles

| Acción | Windows/Linux | Mac |
|--------|---------------|-----|
| Abrir DevTools | F12 | Cmd+Option+I |
| Inspeccionar elemento | Ctrl+Shift+C | Cmd+Option+C |
| Ir a Consola | Ctrl+Shift+J | Cmd+Option+J |
| Modo responsive | Ctrl+Shift+M | Cmd+Shift+M |
| Reload (recarga) | F5 o Ctrl+R | Cmd+R |
| Reload sin caché | Ctrl+Shift+R | Cmd+Shift+R |

---

## Práctica final

Abre DevTools en diferentes sitios web y:

1. Inspecciona cómo están construidos
2. Mira qué etiquetas HTML usan
3. Ve qué estilos CSS aplican
4. Experimenta cambiando cosas

**Los mejores diseñadores web "roban" ideas inspeccionando otras webs.**

No es copiar, es aprender de los mejores.

---

## Conclusión

Las DevTools son:

- ✅ Tu laboratorio de experimentos
- ✅ Tu debugger de errores
- ✅ Tu forma de aprender de otros
- ✅ Tu herramienta más poderosa

**Úsalas constantemente.** Inspecciona todo. Experimenta sin miedo.

**Siguiente:** [Experimentar sin miedo](./08-experimentar-sin-miedo.md)

---

## Recursos adicionales

- **[Chrome DevTools Docs](https://developer.chrome.com/docs/devtools/)** — Documentación oficial
- **[Firefox DevTools](https://firefox-source-docs.mozilla.org/devtools-user/)** — Herramientas de Firefox
- **[DevTools Tips](https://devtoolstips.org/)** — Trucos y tips avanzados

---

## Desafío semanal

**Durante esta semana:** Inspecciona al menos 5 sitios web diferentes.

Pregúntate:
- ¿Cómo lograron este efecto visual?
- ¿Qué etiquetas HTML usaron?
- ¿Puedo replicar esto?

**Inspeccionar sitios web reales es una de las mejores formas de aprender.** 🔍

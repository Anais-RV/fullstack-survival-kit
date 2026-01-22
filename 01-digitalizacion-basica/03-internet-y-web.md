# Internet y la web

> **Qué es internet, cómo funciona y dónde entra tu código**

---

## ¿Internet = la web?

**No.**

- **Internet** es la red de ordenadores conectados entre sí
- **La web** (World Wide Web) es una de las cosas que funcionan sobre internet

**Analogía:**
- Internet = Red de carreteras
- La web = Los coches que circulan por esas carreteras

Otras cosas que usan internet:
- Email
- Videollamadas
- Streaming de música
- Juegos online
- Mensajería (WhatsApp, Telegram...)

**Pero este curso se enfoca en la web:** páginas y aplicaciones web.

---

## ¿Cómo funciona internet? (versión simple)

### 1. Tu ordenador (cliente)

Cuando escribes `www.google.com` en el navegador:

1. Tu ordenador envía una **petición**: "Quiero ver google.com"
2. Viaja por internet (cables, routers, satélites...)
3. Llega al servidor de Google

### 2. El servidor

Un servidor es **otro ordenador** que:
- Está siempre encendido
- Espera peticiones
- Responde con información

El servidor de Google:
1. Recibe tu petición
2. Prepara la respuesta (HTML, CSS, JavaScript, imágenes...)
3. Te la envía de vuelta

### 3. Tu navegador

Tu navegador:
1. Recibe la respuesta
2. La interpreta (lee HTML, CSS, JavaScript)
3. Dibuja la página en tu pantalla

**Todo esto pasa en milisegundos.**

---

## Cliente y servidor: la conversación fundamental

Imagina un restaurante:

**Cliente (tú):**
- Pides comida
- Esperas respuesta
- Recibes el plato

**Servidor (cocina):**
- Recibe el pedido
- Prepara la comida
- Entrega el plato

**En programación web:**
- **Cliente** = Tu navegador (Chrome, Firefox, Safari...)
- **Servidor** = Ordenador que guarda y envía información
- **Petición** = "Quiero ver esta página"
- **Respuesta** = "Aquí está el HTML, CSS, JavaScript..."

---

## ¿Qué es una página web?

Una página web es un **conjunto de archivos** que el navegador interpreta:

### HTML (estructura)
```html
<h1>Título</h1>
<p>Este es un párrafo</p>
```

El esqueleto de la página. Qué elementos hay.

### CSS (diseño visual)
```css
h1 {
  color: blue;
  font-size: 32px;
}
```

Cómo se ve: colores, tamaños, posiciones.

### JavaScript (interactividad)
```javascript
button.addEventListener('click', () => {
  alert('¡Me hiciste clic!');
});
```

Qué pasa cuando interactúas: clics, formularios, animaciones.

**El navegador lee estos archivos y los convierte en lo que ves en pantalla.**

---

## URLs: las direcciones de internet

Una URL (Uniform Resource Locator) es la **dirección** de un recurso en internet.

### Anatomía de una URL

```
https://www.ejemplo.com:443/ruta/pagina.html?buscar=hola#seccion
  │      │      │        │     │             │           │
  │      │      │        │     │             │           └─ Ancla (sección de la página)
  │      │      │        │     │             └─ Query params (parámetros)
  │      │      │        │     └─ Ruta (path)
  │      │      │        └─ Puerto (opcional, por defecto 80/443)
  │      │      └─ Dominio
  │      └─ Subdominio (opcional)
  └─ Protocolo
```

### Protocolo

- `http://` → Sin cifrar (inseguro)
- `https://` → Cifrado (seguro) ← Siempre usa este

### Dominio

El "nombre" del servidor:
- `google.com`
- `github.com`
- `tuempresa.com`

### Ruta

La ubicación del archivo en el servidor:
- `/productos`
- `/usuarios/perfil`
- `/blog/articulo-1.html`

### Query params (parámetros)

Información adicional:
- `?buscar=javascript`
- `?color=rojo&talla=M`
- `?pagina=2`

---

## ¿Qué es un navegador?

Un navegador (Chrome, Firefox, Safari, Edge) es un programa que:

1. **Pide páginas web** al servidor
2. **Descarga** HTML, CSS, JavaScript, imágenes...
3. **Interpreta** el código
4. **Dibuja** la página en pantalla
5. **Ejecuta** JavaScript para hacerla interactiva

**El navegador es tu herramienta principal** como desarrollador frontend.

---

## Cómo se carga una página web (paso a paso)

1. **Escribes la URL** en el navegador
   ```
   https://ejemplo.com/inicio
   ```

2. **DNS traduce el dominio** a una IP
   ```
   ejemplo.com → 192.0.2.1
   ```
   (Como buscar en una agenda de teléfonos)

3. **Petición al servidor**
   ```
   GET /inicio HTTP/1.1
   Host: ejemplo.com
   ```

4. **Servidor responde**
   ```html
   HTTP/1.1 200 OK
   Content-Type: text/html
   
   <html>
     <head>
       <link rel="stylesheet" href="estilos.css">
     </head>
     <body>
       <h1>Bienvenido</h1>
       <script src="script.js"></script>
     </body>
   </html>
   ```

5. **Navegador ve que necesita más archivos**
   - Pide `estilos.css`
   - Pide `script.js`
   - Pide imágenes, fuentes...

6. **Navegador construye la página**
   - Lee el HTML
   - Aplica estilos CSS
   - Ejecuta JavaScript
   - Muestra el resultado

**Todo esto en menos de 1 segundo** (si la conexión es buena).

---

## Frontend vs Backend (introducción rápida)

### Frontend (cliente)

**Dónde:** En tu navegador  
**Qué hace:** Muestra la interfaz y maneja interacción  
**Tecnologías:** HTML, CSS, JavaScript

**Ejemplo:** El formulario de login que ves.

### Backend (servidor)

**Dónde:** En el servidor (otro ordenador)  
**Qué hace:** Procesa lógica, guarda datos, protege información  
**Tecnologías:** Python, Node.js, Java, PHP...

**Ejemplo:** Verificar que tu contraseña es correcta.

**Los dos se comunican constantemente.**

Profundizaremos en esto en el [siguiente módulo](./04-frontend-vs-backend.md).

---

## Tipos de sitios web

### Sitio estático

- Solo HTML, CSS, JavaScript básico
- El contenido no cambia sin que alguien edite el código
- Rápido, simple, barato de alojar

**Ejemplo:** Un portfolio personal, una página de empresa sencilla.

### Sitio dinámico

- Contenido generado en tiempo real
- Se conecta a una base de datos
- Cambia según el usuario, la hora, los datos...

**Ejemplo:** Facebook, Twitter, una tienda online.

### Aplicación web (Web App)

- Como un programa, pero en el navegador
- Muy interactiva
- Usa mucho JavaScript

**Ejemplo:** Gmail, Spotify Web, Notion, Figma.

---

## Herramientas del desarrollador web

### 1. Editor de código

Donde escribes el código:
- Visual Studio Code (recomendado)
- Sublime Text
- Atom

### 2. Navegador

Donde pruebas tu código:
- Chrome (recomendado para desarrollo)
- Firefox
- Safari
- Edge

### 3. Herramientas de desarrollo (DevTools)

Incluidas en el navegador (F12):
- Ver el HTML de cualquier página
- Inspeccionar CSS
- Ver errores de JavaScript
- Simular móviles

**Las veremos en detalle en [módulo 07](./07-herramientas-desarrollador.md).**

---

## Hosting: poner tu web online

Una vez creas tu web, ¿cómo la ves en internet?

### Servicios de hosting

Empresas que "alquilan" espacio en sus servidores:

**Para empezar (gratis):**
- GitHub Pages
- Vercel
- Netlify

**Profesionales (de pago):**
- DigitalOcean
- AWS
- Google Cloud

**Más adelante aprenderás a desplegar** en [módulo 08-despliegue](../08-despliegue/README.md).

---

## Conceptos clave

### Petición-Respuesta

El patrón fundamental de la web:
1. Cliente pide
2. Servidor responde

### HTTP/HTTPS

El "idioma" que usan cliente y servidor para comunicarse.

### Navegador

Tu herramienta para ver y probar páginas web.

### HTML, CSS, JavaScript

Los tres pilares de cualquier página web.

---

## Analogía: Internet como una biblioteca gigante

- **Internet** = La biblioteca entera
- **Servidores** = Estanterías con libros
- **Tu navegador** = Tú, el lector
- **URL** = La dirección del libro que buscas
- **HTTP** = El idioma en que pides el libro
- **HTML/CSS/JS** = El contenido del libro

Cuando quieres un libro:
1. Pides por su dirección (URL)
2. El bibliotecario (servidor) te lo trae
3. Lo lees (navegador interpreta)

---

## Ejercicio mental

Piensa en una web que uses mucho (YouTube, Instagram, tu banco...).

Identifica:
- ¿Qué es **frontend**? (lo que ves)
- ¿Qué es **backend**? (lo que no ves)
- ¿Qué información viaja del cliente al servidor?
- ¿Qué información responde el servidor?

**Ejemplo: YouTube**
- Frontend: La interfaz, el reproductor, los botones
- Backend: Los vídeos guardados, las recomendaciones, los comentarios
- Cliente → Servidor: "Quiero ver este vídeo"
- Servidor → Cliente: "Aquí está el vídeo y los comentarios"

---

## Conclusión

Ya entiendes:
- ✅ Qué es internet y la web
- ✅ Cómo se comunican cliente y servidor
- ✅ Qué es una URL
- ✅ Qué hace un navegador
- ✅ Qué son HTML, CSS y JavaScript
- ✅ La diferencia entre sitio estático y dinámico

**Siguiente:** [Frontend vs Backend explicado con ejemplos](./04-frontend-vs-backend.md)

---

## Recursos adicionales

- **[MDN Web Docs](https://developer.mozilla.org/es/)** — Documentación oficial de tecnologías web
- **[¿Cómo funciona internet?](https://www.youtube.com/watch?v=7_LPdttKXPc)** — Vídeo explicativo (inglés con subtítulos)
- **Consejo:** Abre las herramientas de desarrollo (F12) en cualquier página y explora el HTML

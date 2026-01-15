# Módulo 03: Estructura de Proyecto React

## La estructura creada por Vite

Cuando creaste tu proyecto con `npm create vite@latest`, se generó esta estructura:

```
mi-app-react/
├── node_modules/          ← 🚫 NO TOCAR - Dependencias instaladas
├── public/                ← 📁 Archivos estáticos públicos
│   └── vite.svg          
├── src/                   ← 💻 TU CÓDIGO VA AQUÍ
│   ├── assets/           
│   │   └── react.svg     
│   ├── App.css           
│   ├── App.jsx           ← Componente principal
│   ├── index.css         
│   └── main.jsx          ← Punto de entrada
├── .eslintrc.cjs          ← Configuración ESLint
├── .gitignore             ← Archivos ignorados por Git
├── index.html             ← HTML base (punto de entrada HTML)
├── package-lock.json      ← 🚫 NO EDITAR - Lockfile de dependencias
├── package.json           ← Configuración del proyecto
├── README.md              ← Documentación
└── vite.config.js         ← Configuración de Vite
```

---

## Carpeta `node_modules/`

### ¿Qué es?

Contiene **todas las dependencias** instaladas:
- React, ReactDOM
- Vite y sus plugins
- Librerías que instales con `npm install`

```
node_modules/
├── react/
├── react-dom/
├── vite/
└── ... (miles de carpetas)
```

### ⚠️ Reglas importantes

- ❌ **NUNCA edites** archivos aquí
- ❌ **NUNCA subas** `node_modules/` a Git (pesa cientos de MB)
- ✅ Está en `.gitignore` automáticamente
- ✅ Se regenera con `npm install`

### Si borraste `node_modules/` por accidente

No pasa nada. Ejecuta:

```bash
npm install
```

Se regenerará usando `package.json`.

---

## Carpeta `public/`

### ¿Para qué sirve?

Archivos que **NO necesitan procesarse** por Vite:
- Imágenes estáticas
- Fuentes
- Favicon
- Archivos JSON públicos

```
public/
├── favicon.ico
├── robots.txt
└── logo.png
```

### ¿Cómo usarlos?

Se copian tal cual a la carpeta `dist/` al hacer build.

```jsx
// ❌ NO hagas esto
import logo from '../public/logo.png'

// ✅ Referencia directa
<img src="/logo.png" alt="Logo" />
```

La ruta es relativa a `public/`.

---

## Carpeta `src/`

### ¿Qué es?

**Tu código fuente.** Todo tu código React va aquí.

```
src/
├── assets/           ← Recursos importados (imágenes, CSS)
├── components/       ← Componentes reutilizables (crear tú)
├── pages/           ← Páginas/vistas (crear tú)
├── hooks/           ← Custom hooks (crear tú)
├── utils/           ← Funciones auxiliares (crear tú)
├── App.jsx          ← Componente raíz
├── App.css          
├── index.css        ← Estilos globales
└── main.jsx         ← Punto de entrada
```

### Estructura recomendada (expandida)

Conforme crece tu app, organiza así:

```
src/
├── components/
│   ├── Header.jsx
│   ├── Footer.jsx
│   ├── Button.jsx
│   └── Card.jsx
├── pages/
│   ├── Home.jsx
│   ├── About.jsx
│   └── Contact.jsx
├── hooks/
│   ├── useAuth.js
│   └── useFetch.js
├── services/
│   └── api.js
├── utils/
│   ├── format.js
│   └── validate.js
├── styles/
│   └── global.css
├── App.jsx
└── main.jsx
```

---

## Archivo `package.json`

### El archivo más importante del proyecto

```json
{
  "name": "mi-app-react",
  "private": true,
  "version": "0.0.0",
  "type": "module",
  "scripts": {
    "dev": "vite",
    "build": "vite build",
    "preview": "vite preview"
  },
  "dependencies": {
    "react": "^18.2.0",
    "react-dom": "^18.2.0"
  },
  "devDependencies": {
    "@vitejs/plugin-react": "^4.2.1",
    "vite": "^5.0.8"
  }
}
```

### Secciones clave

#### `name` y `version`
```json
"name": "mi-app-react",
"version": "0.0.0"
```
Identifican tu proyecto.

#### `scripts`
```json
"scripts": {
  "dev": "vite",
  "build": "vite build",
  "preview": "vite preview"
}
```

Comandos personalizados que ejecutas con `npm run`:
- `npm run dev` → Servidor de desarrollo
- `npm run build` → Compilar para producción
- `npm run preview` → Previsualizar build

#### `dependencies`
```json
"dependencies": {
  "react": "^18.2.0",
  "react-dom": "^18.2.0"
}
```

Librerías **necesarias en producción**.

**Instalar nueva dependencia:**
```bash
npm install axios
```

Automáticamente se agrega a `dependencies`:
```json
"dependencies": {
  "react": "^18.2.0",
  "react-dom": "^18.2.0",
  "axios": "^1.6.0"
}
```

#### `devDependencies`
```json
"devDependencies": {
  "@vitejs/plugin-react": "^4.2.1",
  "vite": "^5.0.8"
}
```

Herramientas **solo para desarrollo** (no van a producción).

**Instalar como dev dependency:**
```bash
npm install -D eslint
```

---

## Archivo `package-lock.json`

### ¿Qué es?

Registra las **versiones exactas** de cada dependencia instalada (incluyendo sub-dependencias).

```json
{
  "name": "mi-app-react",
  "lockfileVersion": 3,
  "packages": {
    "node_modules/react": {
      "version": "18.2.0",
      "resolved": "https://registry.npmjs.org/react/-/react-18.2.0.tgz",
      "integrity": "sha512-..."
    }
    // ... miles de líneas más
  }
}
```

### ⚠️ Reglas

- ❌ **NUNCA edites** este archivo manualmente
- ✅ **SÍ súbelo** a Git
- ✅ Garantiza que todos instalen las mismas versiones
- ✅ Se actualiza automáticamente con `npm install`

---

## Archivo `index.html`

### El punto de entrada HTML

```html
<!DOCTYPE html>
<html lang="es">
  <head>
    <meta charset="UTF-8" />
    <link rel="icon" type="image/svg+xml" href="/vite.svg" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Mi App React</title>
  </head>
  <body>
    <div id="root"></div>
    <script type="module" src="/src/main.jsx"></script>
  </body>
</html>
```

### Puntos clave

#### `<div id="root"></div>`
**Aquí se monta tu app React.** Inicialmente vacío.

React lo buscará con:
```jsx
ReactDOM.createRoot(document.getElementById('root'))
```

#### `<script type="module" src="/src/main.jsx">`
**Carga el punto de entrada de React.**

Vite procesa este archivo y todos sus imports.

#### ¿Puedo agregar más cosas?

Sí, puedes agregar:
```html
<head>
  <!-- Meta tags -->
  <meta name="description" content="Mi aplicación React">
  
  <!-- Google Fonts -->
  <link href="https://fonts.googleapis.com/css2?family=Roboto" rel="stylesheet">
  
  <!-- Título dinámico -->
  <title>Mi App - Inicio</title>
</head>
```

---

## Archivo `src/main.jsx`

### El punto de entrada de React

```jsx
import React from 'react'
import ReactDOM from 'react-dom/client'
import App from './App.jsx'
import './index.css'

ReactDOM.createRoot(document.getElementById('root')).render(
  <React.StrictMode>
    <App />
  </React.StrictMode>,
)
```

### Línea por línea

```jsx
import React from 'react'
```
Importa React (necesario para JSX).

```jsx
import ReactDOM from 'react-dom/client'
```
Importa ReactDOM para renderizar en el navegador.

```jsx
import App from './App.jsx'
```
Importa tu componente principal.

```jsx
import './index.css'
```
Importa estilos globales.

```jsx
ReactDOM.createRoot(document.getElementById('root'))
```
Crea una "raíz" React en el elemento `#root` del HTML.

```jsx
.render(
  <React.StrictMode>
    <App />
  </React.StrictMode>,
)
```
Renderiza el componente `<App>` dentro de `<StrictMode>`.

---

## Archivo `src/App.jsx`

### Tu componente principal

```jsx
import { useState } from 'react'
import './App.css'

function App() {
  const [count, setCount] = useState(0)

  return (
    <div className="App">
      <h1>Vite + React</h1>
      <button onClick={() => setCount(count + 1)}>
        count is {count}
      </button>
    </div>
  )
}

export default App
```

Este es un **componente funcional**. Lo estudiaremos en el próximo módulo.

---

## Archivo `vite.config.js`

### Configuración de Vite

```js
import { defineConfig } from 'vite'
import react from '@vitejs/plugin-react'

export default defineConfig({
  plugins: [react()],
})
```

### ¿Para qué sirve?

Configurar cómo Vite procesa tu código:
- Habilitar plugins (React, Vue, etc.)
- Configurar alias de rutas
- Configurar proxy para APIs
- Variables de entorno

### Ejemplo con alias

```js
import { defineConfig } from 'vite'
import react from '@vitejs/plugin-react'
import path from 'path'

export default defineConfig({
  plugins: [react()],
  resolve: {
    alias: {
      '@': path.resolve(__dirname, './src'),
      '@components': path.resolve(__dirname, './src/components'),
    },
  },
})
```

Ahora puedes importar así:
```jsx
// En vez de
import Button from '../../components/Button.jsx'

// Puedes hacer
import Button from '@components/Button.jsx'
```

---

## Archivo `.gitignore`

### Archivos ignorados por Git

```
# Dependencias
node_modules

# Build
dist
dist-ssr
*.local

# Editor
.vscode/*
!.vscode/extensions.json
.idea
.DS_Store

# Logs
*.log
npm-debug.log*
```

### ¿Por qué?

- `node_modules/` → Pesa demasiado (se regenera con `npm install`)
- `dist/` → Es código generado (se regenera con `npm run build`)
- Logs y archivos temporales → No son parte del código fuente

---

## Flujo de carga de la aplicación

```
1. Navegador carga index.html
         ↓
2. index.html carga /src/main.jsx
         ↓
3. main.jsx importa App.jsx
         ↓
4. main.jsx renderiza <App /> en #root
         ↓
5. App.jsx renderiza su contenido
         ↓
6. Usuario ve la interfaz
```

---

## Ejercicio práctico: Organizar proyecto

Crea esta estructura en tu proyecto:

```bash
src/
├── components/
│   └── Saludo.jsx
├── App.jsx
└── main.jsx
```

**Paso 1:** Crea `src/components/Saludo.jsx`:

```jsx
function Saludo() {
  return <h2>¡Hola desde un componente!</h2>;
}

export default Saludo;
```

**Paso 2:** Importa en `App.jsx`:

```jsx
import Saludo from './components/Saludo';

function App() {
  return (
    <div>
      <h1>Mi App</h1>
      <Saludo />
    </div>
  );
}

export default App;
```

**Paso 3:** Ejecuta `npm run dev` y verifica que funciona.

---

## Comandos útiles

```bash
# Ver estructura de carpetas (Windows)
tree /F

# Ver estructura de carpetas (Linux/Mac)
tree

# Ver tamaño de node_modules
du -sh node_modules

# Limpiar y reinstalar dependencias
rm -rf node_modules package-lock.json
npm install
```

---

## Errores comunes

### "Cannot find module './App.jsx'"
- **Causa:** Ruta incorrecta o archivo no existe
- **Solución:** Verifica que `App.jsx` esté en la ubicación correcta

### "Unexpected token '<'"
- **Causa:** Intentas importar JSX sin extensión `.jsx` configurada
- **Solución:** Usa extensiones `.jsx` para archivos con JSX

### Cambios no se reflejan
- **Causa:** Error de compilación silencioso o cache
- **Solución:** Reinicia el servidor (`Ctrl + C`, luego `npm run dev`)

---

## Mejores prácticas

✅ **Organiza desde el principio**
- Crea carpetas `components/`, `pages/`, `hooks/` desde el inicio

✅ **Un componente por archivo**
- `Button.jsx` solo contiene el componente `Button`

✅ **Nombres descriptivos**
- `UserProfile.jsx` mejor que `UP.jsx`

✅ **Estilos junto al componente**
```
components/
├── Button.jsx
└── Button.css
```

✅ **Agrupa por feature, no por tipo**
```
# ✅ Bueno
features/
├── auth/
│   ├── Login.jsx
│   ├── Signup.jsx
│   └── authService.js
└── products/
    ├── ProductList.jsx
    └── ProductCard.jsx

# ❌ Malo (todo mezclado)
components/
├── Login.jsx
├── ProductList.jsx
└── ProductCard.jsx
services/
├── authService.js
└── productService.js
```

---

## Resumen

✅ **Entiendes cada archivo del proyecto**  
✅ **Sabes qué va en `src/` vs `public/`**  
✅ **Conoces `package.json` y `package-lock.json`**  
✅ **Entiendes el flujo de carga**  
✅ **Puedes organizar tu código**

---

## Siguiente paso

Entiendes la estructura. Ahora crearás tu **primer componente desde cero**.

**Siguiente:** [Módulo 04 - Primer componente](./04-primer-componente.md)

**Anterior:** [Módulo 02 - Instalación y entorno](./02-instalacion-entorno.md)

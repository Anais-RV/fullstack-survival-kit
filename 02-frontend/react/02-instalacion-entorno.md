# Módulo 02: Instalación y Entorno de Desarrollo

## Requisitos previos

Antes de instalar React, necesitas:

1. **Node.js** - Motor de JavaScript fuera del navegador
2. **pnpm** - Gestor de paquetes moderno (más rápido que npm)
3. **Editor de código** - VS Code (recomendado)

---

## ⚡ Por qué pnpm en 2026

En 2026, **pnpm** es el estándar de la industria por:

- **3x más rápido** que npm/yarn
- **Ahorra espacio en disco** (almacenamiento eficiente)
- **Instalaciones más seguras** (strict dependency resolution)
- **Monorepos nativos** (usado por React, Vue, Angular)
- **Compatible 100%** con npm (misma sintaxis)

### npm vs pnpm vs yarn

| Característica | npm | yarn | pnpm |
|----------------|-----|------|------|
| Velocidad | 🐢 | 🚀 | ⚡ |
| Espacio en disco | ❌ Duplica todo | ⚠️ Caché | ✅ Almacén global |
| Monorepos | ⚠️ Workspaces | ✅ Workspaces | ✅ Nativo |
| Uso 2026 | 📉 Declinando | 📊 Estable | 📈 Creciendo |

> **Nota:** Si ya usas npm, todo sigue funcionando. pnpm es una mejora, no un requisito.

---

## Paso 1: Instalar Node.js

### ¿Qué es Node.js?

**Node.js** es JavaScript que corre fuera del navegador. Lo necesitas para:
- Instalar dependencias (React, librerías)
- Ejecutar herramientas de desarrollo (Vite)
- Compilar tu código para producción

---

### Instalar Node.js (2026)

**Versión recomendada:** Node.js **22 LTS** o superior

**Windows/Mac/Linux:**

1. Ve a [nodejs.org](https://nodejs.org)
2. Descarga la versión **LTS** (Long Term Support)
3. Ejecuta el instalador
4. Sigue el asistente (opciones por defecto están bien)

**Verificar instalación:**

```bash
node --version
# Debería mostrar: v22.x.x o superior
```

---

## Paso 2: Instalar pnpm

### ¿Qué es pnpm?

**pnpm** (performant npm) es el gestor de paquetes más rápido y eficiente:
- Instala paquetes **hasta 3x más rápido** que npm
- Ahorra **gigabytes de espacio** con almacenamiento compartido
- Usado por empresas como **Microsoft, TikTok, Google**

### Instalar pnpm

**Con npm (viene con Node.js):**

```bash
npm install -g pnpm
```

**Verificar instalación:**

```bash
pnpm --version
# Debería mostrar: 9.x.x o superior
```

### Compatibilidad con npm

pnpm usa los **mismos comandos** que npm:

```bash
# npm install → pnpm install
# npm run dev → pnpm dev
# npm install react → pnpm add react
```

---

## Paso 3: Crear proyecto con Vite

### ¿Qué es Vite?

**Vite** es el bundler estándar en 2026 para proyectos React:
- ⚡ **Lightning fast** - HMR en < 50ms
- 🔧 **Zero config** - Funciona out-of-the-box
- 📦 **Optimizado** - Build ultrarápido con Rollup
- 🎯 **Estándar industria** - Usado por React, Vue, Svelte
- 🔋 **Moderno** - ESM nativo, TypeScript sin config

### Herramientas en 2026

| Herramienta | Estado | Uso |
|-------------|--------|-----|
| **Vite** | ✅ Recomendado | Apps SPA, librerías |
| **Next.js** | ✅ Framework completo | SSR, SEO, fullstack |
| Create React App | ❌ Deprecado | No usar |
| Webpack | ⚠️ Legacy | Solo mantenimiento |
| **Turbopack** | 🆕 Emergente | Next.js 15+ |

---

### Crear tu primer proyecto

Abre la terminal y ejecuta:

```bash
pnpm create vite mi-primera-app-react
```

Te hará algunas preguntas:

```
? Select a framework: › React
? Select a variant: › JavaScript
```

Selecciona:
1. **React** (con las flechas ↓↑, Enter para confirmar)
2. **JavaScript** (o TypeScript si quieres tipado)

### 💡 JavaScript vs TypeScript en 2026

**Para aprender React:** JavaScript (más simple)  
**Para proyectos profesionales:** TypeScript (85% de proyectos en 2026)

```bash
# Con JavaScript
pnpm create vite mi-app -- --template react

# Con TypeScript (recomendado para producción)
pnpm create vite mi-app -- --template react-ts
```

---

### Estructura creada

```
mi-primera-app-react/
├── node_modules/         ← Librerías instaladas (no tocar)
├── public/               ← Archivos estáticos (imágenes, favicon)
├── src/                  ← Tu código va aquí
│   ├── App.css          ← Estilos del componente principal
│   ├── App.jsx          ← Componente principal
│   ├── index.css        ← Estilos globales
│   └── main.jsx         ← Punto de entrada
├── .gitignore           ← Archivos ignorados por Git
├── index.html           ← HTML base
├── package.json         ← Configuración del proyecto
└── vite.config.js       ← Configuración de Vite
```

---

## Paso 4: Instalar dependencias

Entra a la carpeta del proyecto:

```bash
cd mi-primera-app-react
```

Instala las dependencias:

```bash
pnpm install
```

Esto instala React y todas las librerías necesarias. **Con pnpm tarda solo segundos.**

### Velocidad de instalación (2026)

```
npm install     → ~45 segundos
yarn install    → ~30 segundos  
pnpm install    → ~15 segundos  ⚡
```

---

## Paso 5: Ejecutar el proyecto

Inicia el servidor de desarrollo:

```bash
pnpm dev
```

**Nota:** Con pnpm no necesitas escribir `run`, es más corto.

Verás algo como:

```
  VITE v6.0.0  ready in 320 ms

  ➜  Local:   http://localhost:5173/
  ➜  Network: use --host to expose
  ➜  press h + enter to show help
```

**Abre tu navegador en:** `http://localhost:5173`

¡Deberías ver la página de bienvenida de React! 🎉

---

## Entender los scripts

En `package.json` hay varios scripts:

```json
{
  "scripts": {
    "dev": "vite",
    "build": "vite build",
    "preview": "vite preview"
  }
}
```

### Comandos con pnpm

```bash
# Desarrollo
pnpm dev          # Inicia servidor (más corto que pnpm run dev)

# Producción  
pnpm build        # Compila para producción
pnpm preview      # Previsualiza build

# Agregar dependencias
pnpm add react-router-dom    # Instala librería
pnpm add -D eslint           # Instala como devDependency
pnpm remove react-router     # Desinstala librería
```

### `pnpm dev`
Inicia el servidor de desarrollo:
- **Hot Module Replacement** (HMR) instantáneo
- Source maps para debugging
- Puerto: 5173

### `pnpm build`
Compila para producción:
- Optimiza y minifica código
- Tree-shaking (elimina código no usado)
- Genera carpeta `dist/`
- **Build en 2026:** < 5 segundos

### `pnpm preview`
Previsualiza el build de producción:
- Sirve la carpeta `dist/`
- Simula producción localmente

---

## Explicación de archivos clave

### `package.json`

Configuración del proyecto:

```json
{
  "name": "mi-primera-app-react",
  "version": "0.0.0",
  "type": "module",
  "scripts": {
    "dev": "vite",
    "build": "vite build",
    "preview": "vite preview"
  },
  "dependencies": {
    "react": "^19.0.0",
    "react-dom": "^19.0.0"
  },
  "devDependencies": {
    "@vitejs/plugin-react": "^4.3.0",
    "vite": "^6.0.0"
  }
}
```

**Secciones importantes:**
- `dependencies` → Librerías necesarias en producción
- `devDependencies` → Herramientas solo para desarrollo
- `scripts` → Comandos que puedes ejecutar

---

### `index.html`

El HTML base de tu aplicación:

```html
<!DOCTYPE html>
<html lang="es">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Mi App React</title>
  </head>
  <body>
    <div id="root"></div>
    <script type="module" src="/src/main.jsx"></script>
  </body>
</html>
```

**Puntos clave:**
- `<div id="root">` → Aquí React inyecta tu app
- `<script src="/src/main.jsx">` → Punto de entrada

---

### `src/main.jsx`

Punto de entrada de React:

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

**¿Qué hace?**
1. Importa React y ReactDOM
2. Importa el componente principal `App`
3. Busca el elemento `#root` en `index.html`
4. Renderiza `<App>` dentro de `#root`

**StrictMode:** Modo estricto que ayuda a detectar errores comunes.

---

### `src/App.jsx`

Tu primer componente:

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

Este es un **componente funcional** de React. Lo estudiaremos a fondo en el siguiente módulo.

---

## Extensiones de VS Code recomendadas

Instala estas extensiones para mejor experiencia:

### 1. **ES7+ React/Redux/React-Native snippets**
- Autor: dsznajder
- Atajos para crear componentes rápido

### 2. **Simple React Snippets**
- Autor: Burke Holland
- Snippets adicionales útiles

### 3. **Auto Rename Tag**
- Autor: Jun Han
- Renombra etiquetas HTML automáticamente

### 4. **Prettier - Code formatter**
- Autor: Prettier
- Formatea código automáticamente
- `pnpm add -D prettier`

### 5. **ESLint**
- Autor: Microsoft
- Detecta errores en tu código
- `pnpm add -D eslint`

### 6. **Biome (2026 - Alternativa moderna)** ⚡
- Autor: Biome Team
- **Reemplaza ESLint + Prettier** en una sola herramienta
- **100x más rápido** que ESLint
- Escrito en Rust
- `pnpm add -D @biomejs/biome`

---

## Configurar Prettier (opcional pero recomendado)

Crea `.prettierrc` en la raíz:

```json
{
  "semi": true,
  "singleQuote": true,
  "tabWidth": 2,
  "trailingComma": "es5"
}
```

Crea `.prettierignore`:

```
node_modules
dist
build
```

Instala Prettier como dependencia:

```bash
pnpm add -D prettier
```

Ahora tu código se formateará automáticamente al guardar.

---

## Hot Module Replacement (HMR)

Con el servidor corriendo (`pnpm dev`), prueba esto:

1. Abre `src/App.jsx`
2. Cambia el título: `<h1>¡Hola Mundo!</h1>`
3. Guarda el archivo

**El navegador se actualiza INSTANTÁNEAMENTE sin recargar la página.** Esto es HMR (Hot Module Replacement).

Con Vite 6 + React 19, el HMR es **sub-100ms** ⚡

---

## 🚀 Tendencias React 2026

### 1. **React Server Components (RSC)**
El futuro de React:
- Componentes que se ejecutan **solo en el servidor**
- Reducen JavaScript enviado al navegador
- Integrados nativamente en Next.js 15+
- Mejoran performance dramáticamente

```jsx
// Componente servidor (sin "use client")
async function Products() {
  const products = await db.products.findAll() // Directo desde servidor
  return <ProductList products={products} />
}
```

### 2. **Biome - ESLint + Prettier unificado**
- **100x más rápido** que ESLint
- Escrito en Rust
- Formateador + linter en una herramienta
- Configuración cero

```bash
pnpm add -D @biomejs/biome
pnpm biome check --write .
```

### 3. **Bun - Runtime alternativo a Node.js**
- **3x más rápido** que Node.js
- Bundler, transpiler, test runner integrado
- Compatible con Node.js

```bash
bun create vite mi-app
bun install  # Instala en 0.5 segundos 🚀
bun dev
```

### 4. **Turbopack (Next.js 15)**
- Reemplazo de Webpack
- **700x más rápido** que Webpack
- Escrito en Rust
- HMR instantáneo

### 5. **Signals - Estado reactivo**
Alternativa a useState más eficiente:

```jsx
import { signal } from '@preact/signals-react'

const count = signal(0)

function Counter() {
  return (
    <button onClick={() => count.value++}>
      {count.value}  {/* Auto re-render */}
    </button>
  )
}
```

### 6. **React 19 - Novedades principales**
- **Actions**: Manejo de formularios simplificado
- **use() hook**: Consumir promesas y contexto
- **React Compiler**: Optimizaciones automáticas
- **Document Metadata**: `<title>` y `<meta>` nativos

```jsx
// React 19 - Formularios simplificados
function Form() {
  async function submitAction(formData) {
    'use server' // Action del servidor
    await saveToDatabase(formData)
  }
  
  return <form action={submitAction}>...</form>
}
```

### Comparativa Herramientas 2026

| Herramienta | Propósito | Estado 2026 |
|-------------|----------|-------------|
| **Vite** | Bundler rápido | ✅ Estándar actual |
| **Turbopack** | Bundler ultra-rápido | 🔥 Next.js 15 |
| **pnpm** | Package manager | ✅ Más usado |
| **Bun** | Runtime todo-en-uno | 🚀 Creciendo rápido |
| **Biome** | Linter + Formatter | 🔥 Reemplaza ESLint |
| **TypeScript** | Tipado estático | ✅ 85% proyectos |
| **React Server Components** | SSR moderno | 🚀 Futuro de React |

---

## Solución de problemas comunes

### Error: "command not found: pnpm"
- **Solución:** pnpm no está instalado. Ejecuta: `npm install -g pnpm`

### Error: "command not found: node"
- **Solución:** Node.js no está instalado correctamente. Reinstala desde nodejs.org

### Error: "EACCES: permission denied"
- **Solución (Linux/Mac):** Ejecuta con `sudo npm install -g pnpm`

### Error: "Port 5173 is already in use"
- **Solución:** Otro proceso está usando el puerto. Cierra otras terminales o usa `pnpm dev --port 3000`

### El navegador no se actualiza automáticamente
- **Solución:** Refresca manualmente con `Ctrl + R`. Si persiste, reinicia el servidor (`Ctrl + C`, luego `pnpm dev`)

### Cambios en `package.json` no aplican
- **Solución:** Detén el servidor, ejecuta `pnpm install`, luego `pnpm dev`

---

## Comandos útiles de pnpm

Cheat sheet de comandos esenciales:

| Acción | npm | pnpm |
|--------|-----|------|
| Instalar dependencias | `npm install` | `pnpm install` |
| Añadir librería | `npm install lodash` | `pnpm add lodash` |
| Añadir dev dependency | `npm install -D eslint` | `pnpm add -D eslint` |
| Eliminar dependencia | `npm uninstall lodash` | `pnpm remove lodash` |
| Actualizar paquetes | `npm update` | `pnpm update` |
| Ver dependencias obsoletas | `npm outdated` | `pnpm outdated` |
| Ejecutar script | `npm run dev` | `pnpm dev` |
| Limpiar caché | `npm cache clean --force` | `pnpm store prune` |

### Comandos más usados

```bash
# Crear proyecto
pnpm create vite nombre-proyecto

# Entrar al proyecto
cd nombre-proyecto

# Instalar dependencias
pnpm install

# Iniciar servidor desarrollo
pnpm dev

# Compilar para producción
pnpm build

# Instalar una librería nueva
pnpm add nombre-libreria

# Instalar dev dependency
pnpm add -D eslint

# Desinstalar una librería
pnpm remove nombre-libreria

# Ver versión de Node
node --version

# Ver versión de pnpm
pnpm --version

# Limpiar store de pnpm (si hay problemas)
pnpm store prune
```

### Por qué pnpm es mejor en 2026

```
Eficiencia de espacio:
npm:  ~500 MB por proyecto
pnpm: ~50 MB por proyecto (10x menos) ⚡

Velocidad:
npm:  ~45 segundos
pnpm: ~15 segundos (3x más rápido) ⚡

Seguridad:
- Enlaces simbólicos estrictos
- Evita hoisting fantasma
- Dependency hell resuelto
```

---

## Estructura de trabajo recomendada

```bash
proyectos-react/
├── 01-mi-primera-app/
├── 02-contador/
├── 03-lista-tareas/
└── 04-proyecto-final/
```

Crea una carpeta para **cada módulo/ejercicio**. No trabajes todo en un solo proyecto.

---

## Resumen

✅ **Instalaste Node.js 22 LTS**  
✅ **Instalaste pnpm 9+ (package manager moderno)**  
✅ **Creaste un proyecto con Vite 6**  
✅ **Ejecutaste el servidor de desarrollo**  
✅ **Entiendes la estructura básica**  
✅ **Configuraste VS Code**  
✅ **Conoces las tendencias React 2026**

Ahora tienes un entorno profesional con las herramientas más modernas para desarrollar con React.

---

## Ejercicio práctico

1. Crea un nuevo proyecto llamado `ejercicio-react-01` usando `pnpm create vite`
2. Instala dependencias con `pnpm install`
3. Ejecuta el servidor de desarrollo con `pnpm dev`
4. Cambia el título de `App.jsx` a "Mi Primera App"
5. Cambia el texto del botón a "Haz click aquí"
6. Observa cómo se actualiza automáticamente (HMR)
7. Compara el tiempo de instalación vs npm (si tienes ambos)

**Bonus:** Instala Biome y prueba el linter:

```bash
pnpm add -D @biomejs/biome
pnpm biome init
pnpm biome check src/
```

---

## Siguiente paso

Entiendes el entorno. Ahora descubrirás **qué es cada archivo y para qué sirve**.

**Siguiente:** [Módulo 03 - Estructura de proyecto](./03-estructura-proyecto.md)

**Anterior:** [Módulo 01 - Por qué React](./01-por-que-react.md)

# Configuración del entorno de desarrollo

Antes de escribir una línea de código, necesitas las herramientas correctas. Es como cocinar: necesitas los utensilios antes de empezar la receta.

**Esta guía te lleva paso a paso por la instalación de todo lo necesario.**

No es emocionante. Pero es necesario. Y solo lo haces una vez.

---

## ¿Qué vamos a instalar?

1. **Python** — Lenguaje para backend
2. **Node.js** — Entorno para frontend y herramientas de desarrollo
3. **Visual Studio Code** — Editor de código
4. **Git** — Control de versiones
5. **PostgreSQL** — Base de datos (lo instalaremos más adelante, en el módulo 04)

**Tiempo estimado:** 30-60 minutos, dependiendo de tu conexión a internet.

---

## Antes de empezar

### Requisitos
- Windows 10/11, macOS 10.14+, o Linux (Ubuntu 20.04+)
- 8GB de RAM mínimo (16GB recomendado)
- 20GB de espacio libre en disco
- Conexión a internet

### ⚠️ Importante
- **Lee las instrucciones antes de ejecutar comandos**
- **Copia y pega los comandos exactamente como aparecen**
- **No cierres la terminal mientras se instala algo**
- **Si algo falla, ve a la sección de Troubleshooting al final**

---

## 1. Instalación de Python

Python es el lenguaje que usaremos para el backend.

### Windows

#### Opción 1: Desde el sitio oficial (recomendado)

1. Ve a [python.org/downloads](https://www.python.org/downloads/)
2. Descarga **Python 3.11** o superior (no uses Python 2.x)
3. Ejecuta el instalador
4. **✅ IMPORTANTE:** Marca la casilla **"Add Python to PATH"**
5. Haz clic en "Install Now"
6. Espera a que termine (2-5 minutos)

#### Opción 2: Desde Microsoft Store

1. Abre Microsoft Store
2. Busca "Python 3.11"
3. Haz clic en "Obtener"
4. Espera a que se instale

#### Verificar instalación

Abre una nueva terminal (PowerShell o CMD) y escribe:

```bash
python --version
```

Deberías ver algo como: `Python 3.11.x`

Si vez `Python 2.x` o un error, ve a la sección de Troubleshooting.

---

### macOS

#### Opción 1: Con Homebrew (recomendado)

Si no tienes Homebrew instalado, primero instálalo:

```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

Luego instala Python:

```bash
brew install python@3.11
```

#### Opción 2: Desde el sitio oficial

1. Ve a [python.org/downloads](https://www.python.org/downloads/)
2. Descarga Python 3.11 o superior
3. Ejecuta el instalador `.pkg`
4. Sigue las instrucciones en pantalla

#### Verificar instalación

Abre Terminal y escribe:

```bash
python3 --version
```

Deberías ver: `Python 3.11.x`

> **Nota:** En macOS, usa `python3` en lugar de `python` para evitar conflictos con la versión del sistema.

---

### Linux (Ubuntu/Debian)

Abre la terminal y ejecuta:

```bash
sudo apt update
sudo apt install python3.11 python3.11-venv python3-pip
```

Si Python 3.11 no está disponible en tu distribución:

```bash
sudo add-apt-repository ppa:deadsnakes/ppa
sudo apt update
sudo apt install python3.11 python3.11-venv python3.11-dev
```

#### Verificar instalación

```bash
python3 --version
```

---

## 2. Instalación de Node.js

Node.js te permite ejecutar JavaScript fuera del navegador. Lo usaremos para React y herramientas de desarrollo.

### Windows

1. Ve a [nodejs.org](https://nodejs.org/)
2. Descarga la versión **LTS** (Long Term Support)
3. Ejecuta el instalador
4. Acepta los términos y haz clic en "Next" hasta "Install"
5. Espera a que termine (2-5 minutos)

#### Verificar instalación

Abre una nueva terminal y escribe:

```bash
node --version
npm --version
```

Deberías ver dos versiones (ej: `v20.x.x` y `10.x.x`).

---

### macOS

#### Con Homebrew (recomendado)

```bash
brew install node
```

#### Desde el sitio oficial

1. Ve a [nodejs.org](https://nodejs.org/)
2. Descarga la versión LTS
3. Ejecuta el instalador `.pkg`
4. Sigue las instrucciones

#### Verificar instalación

```bash
node --version
npm --version
```

---

### Linux (Ubuntu/Debian)

```bash
curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -
sudo apt-get install -y nodejs
```

#### Verificar instalación

```bash
node --version
npm --version
```

---

## 3. Instalación de Visual Studio Code

VSCode es el editor de código que usaremos. Es gratis, potente y tiene extensiones para todo.

### Todas las plataformas

1. Ve a [code.visualstudio.com](https://code.visualstudio.com/)
2. Descarga la versión para tu sistema operativo
3. Instala siguiendo las instrucciones por defecto

### Windows
- En el instalador, marca: **"Agregar acción 'Abrir con Code' al menú contextual"**
- Esto te permite abrir carpetas con clic derecho → "Abrir con Code"

### macOS
- Después de instalar, abre VSCode
- Presiona `Cmd+Shift+P`
- Escribe "shell command" y selecciona **"Install 'code' command in PATH"**
- Esto te permite abrir VSCode desde la terminal con `code .`

### Linux
- Algunas distribuciones necesitan que añadas el comando manualmente
- Sigue las instrucciones en pantalla después de la instalación

#### Verificar instalación

Abre VSCode. Si se abre, está instalado correctamente.

---

## 4. Instalación de Git

Git es el sistema de control de versiones. Te permite guardar cambios, volver atrás si algo se rompe, y colaborar con otros.

### Windows

#### Opción 1: Git for Windows

1. Ve a [git-scm.com](https://git-scm.com/)
2. Descarga el instalador para Windows
3. Ejecuta el instalador
4. **Configuración recomendada:**
   - Editor: "Use Visual Studio Code as Git's default editor"
   - PATH: "Git from the command line and also from 3rd-party software"
   - HTTPS: "Use the native Windows Secure Channel library"
   - Line endings: "Checkout Windows-style, commit Unix-style"
   - Terminal: "Use Windows' default console window"
5. El resto: valores por defecto

#### Verificar instalación

Abre una nueva terminal:

```bash
git --version
```

Deberías ver: `git version 2.x.x`

---

### macOS

Git suele venir preinstalado. Verifica:

```bash
git --version
```

Si no está instalado, macOS te pedirá instalar las "Command Line Tools". Acéptalo.

O instálalo con Homebrew:

```bash
brew install git
```

---

### Linux (Ubuntu/Debian)

```bash
sudo apt update
sudo apt install git
```

#### Verificar instalación

```bash
git --version
```

---

## 5. Configuración inicial de Git

Ahora que tienes Git, configúralo con tu información:

```bash
git config --global user.name "Tu Nombre"
git config --global user.email "tuemail@ejemplo.com"
```

Cambia "Tu Nombre" y "tuemail@ejemplo.com" por tus datos reales.

Esto es importante porque cada cambio que guardes en Git llevará esta información.

#### Verificar configuración

```bash
git config --global user.name
git config --global user.email
```

Deberías ver tu nombre y email.

---

## 6. Extensiones esenciales de VSCode

VSCode por sí solo es poderoso, pero con extensiones se vuelve increíble.

Abre VSCode y ve a la sección de extensiones (icono de cuadrados en la barra lateral, o `Ctrl+Shift+X` / `Cmd+Shift+X`).

Busca e instala estas extensiones:

### Esenciales (instala YA)

1. **Python** (Microsoft)
   - ID: `ms-python.python`
   - Autocompletado, debugging, linting para Python

2. **Pylance** (Microsoft)
   - ID: `ms-python.vscode-pylance`
   - Inteligencia de código avanzada para Python

3. **ES7+ React/Redux/React-Native snippets** (dsznajder)
   - ID: `dsznajder.es7-react-js-snippets`
   - Atajos para escribir React más rápido

4. **ESLint** (Microsoft)
   - ID: `dbaeumer.vscode-eslint`
   - Detecta errores en JavaScript

5. **Prettier - Code formatter** (Prettier)
   - ID: `esbenp.prettier-vscode`
   - Formatea tu código automáticamente

6. **GitLens** (GitKraken)
   - ID: `eamodio.gitlens`
   - Superpoderes para Git en VSCode

### Recomendadas (instala cuando quieras)

7. **Auto Rename Tag** (Jun Han)
   - ID: `formulahendry.auto-rename-tag`
   - Renombra automáticamente las etiquetas HTML emparejadas

8. **Path Intellisense** (Christian Kohler)
   - ID: `christian-kohler.path-intellisense`
   - Autocompletado de rutas de archivos

9. **Live Server** (Ritwick Dey)
   - ID: `ritwickdey.liveserver`
   - Servidor local con recarga automática para HTML/CSS

10. **Material Icon Theme** (Philipp Kief)
    - ID: `pkief.material-icon-theme`
    - Iconos bonitos para diferentes tipos de archivos

---

## 7. Configuración de VSCode

Algunas configuraciones útiles para tu `settings.json`:

1. Abre VSCode
2. Presiona `Ctrl+,` (Windows/Linux) o `Cmd+,` (macOS)
3. Haz clic en el icono de archivo en la esquina superior derecha (Open Settings JSON)
4. Añade esta configuración:

```json
{
  "editor.formatOnSave": true,
  "editor.defaultFormatter": "esbenp.prettier-vscode",
  "editor.tabSize": 2,
  "editor.wordWrap": "on",
  "files.autoSave": "onFocusChange",
  "python.linting.enabled": true,
  "python.linting.pylintEnabled": false,
  "python.linting.flake8Enabled": true,
  "python.formatting.provider": "black",
  "[python]": {
    "editor.defaultFormatter": "ms-python.black-formatter",
    "editor.formatOnSave": true,
    "editor.codeActionsOnSave": {
      "source.organizeImports": true
    }
  },
  "javascript.updateImportsOnFileMove.enabled": "always",
  "typescript.updateImportsOnFileMove.enabled": "always"
}
```

**¿Qué hace esto?**
- Formatea automáticamente al guardar
- Ajusta el texto al ancho de la ventana
- Guarda automáticamente cuando cambias de archivo
- Configura linters para Python
- Actualiza imports automáticamente

---

## 8. Entorno virtual de Python

Los entornos virtuales aíslan las dependencias de cada proyecto. Es como tener cocinas separadas para diferentes recetas: no mezclas ingredientes.

### Crear tu primer entorno virtual

Crea una carpeta de prueba:

```bash
mkdir test-python
cd test-python
```

Crea el entorno virtual:

**Windows:**
```bash
python -m venv venv
```

**macOS/Linux:**
```bash
python3 -m venv venv
```

Esto crea una carpeta `venv` con todo lo necesario.

### Activar el entorno virtual

**Windows (PowerShell):**
```bash
.\venv\Scripts\Activate.ps1
```

Si te da un error de permisos, ejecuta esto primero (como administrador):
```bash
Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser
```

Luego intenta activar de nuevo.

**Windows (CMD):**
```bash
venv\Scripts\activate.bat
```

**macOS/Linux:**
```bash
source venv/bin/activate
```

### ¿Cómo saber si está activo?

Verás `(venv)` al inicio de la línea en tu terminal:

```
(venv) C:\test-python>
```

### Desactivar el entorno

Simplemente escribe:

```bash
deactivate
```

El `(venv)` desaparecerá.

---

## 9. Instalación de paquetes Python básicos

Con el entorno virtual activado, instala estos paquetes que usaremos más adelante:

```bash
pip install --upgrade pip
pip install django fastapi uvicorn psycopg2-binary python-dotenv
```

**¿Qué acabas de instalar?**
- `django`: Framework completo para backend
- `fastapi`: Framework moderno para APIs
- `uvicorn`: Servidor para ejecutar FastAPI
- `psycopg2-binary`: Conector para PostgreSQL
- `python-dotenv`: Para manejar variables de entorno

Esto tomará 1-2 minutos.

---

## 10. Verificación completa

Ejecuta estos comandos para verificar que todo funciona:

```bash
# Python
python --version          # (o python3 --version en macOS/Linux)

# Node.js
node --version
npm --version

# Git
git --version

# Pip (con entorno virtual activo)
pip --version

# Django
python -m django --version

# FastAPI (intentar importar)
python -c "import fastapi; print(fastapi.__version__)"
```

Si todos dan versiones sin errores, **estás listo**.

---

## 11. Estructura recomendada de carpetas

Organiza tus proyectos desde el inicio:

```
Documentos/
└── Proyectos/
    ├── fullstack-learning/    ← Tu carpeta para este repositorio
    ├── practicas/             ← Experimentos y pruebas
    └── proyectos-propios/     ← Tus proyectos personales
```

Crea estas carpetas ahora:

**Windows:**
```bash
mkdir %USERPROFILE%\Documentos\Proyectos\fullstack-learning
mkdir %USERPROFILE%\Documentos\Proyectos\practicas
```

**macOS/Linux:**
```bash
mkdir -p ~/Documentos/Proyectos/fullstack-learning
mkdir -p ~/Documentos/Proyectos/practicas
```

---

## Troubleshooting

### Python no se reconoce como comando

**Windows:**
1. Busca "Variables de entorno" en el menú inicio
2. Haz clic en "Variables de entorno"
3. En "Variables del sistema", busca `Path`
4. Haz clic en "Editar"
5. Añade estas rutas (adapta según tu instalación):
   - `C:\Users\TuUsuario\AppData\Local\Programs\Python\Python311`
   - `C:\Users\TuUsuario\AppData\Local\Programs\Python\Python311\Scripts`
6. Cierra y abre una nueva terminal

**macOS/Linux:**
Añade esto a tu `~/.zshrc` o `~/.bashrc`:

```bash
export PATH="/usr/local/bin/python3:$PATH"
```

Luego ejecuta:
```bash
source ~/.zshrc  # o source ~/.bashrc
```

---

### Node no se reconoce como comando

**Windows:**
Reinstala Node.js y asegúrate de marcar "Add to PATH" durante la instalación.

**macOS:**
Verifica que Homebrew esté configurado:
```bash
echo 'export PATH="/usr/local/bin:$PATH"' >> ~/.zshrc
source ~/.zshrc
```

---

### Error "cannot be loaded because running scripts is disabled"

**Windows PowerShell:**

Ejecuta PowerShell como administrador:

```bash
Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser
```

Luego intenta activar el entorno virtual de nuevo.

---

### pip install falla con errores de compilación

**Windows:**
Instala las "Build Tools for Visual Studio":
- Ve a [visualstudio.microsoft.com/downloads](https://visualstudio.microsoft.com/downloads/)
- Descarga "Build Tools for Visual Studio"
- Instala el workload "C++ build tools"

**macOS:**
Instala Xcode Command Line Tools:
```bash
xcode-select --install
```

**Linux:**
Instala las dependencias de desarrollo:
```bash
sudo apt install build-essential python3-dev
```

---

### VSCode no reconoce el intérprete de Python

1. Abre VSCode
2. Abre una carpeta con código Python
3. Presiona `Ctrl+Shift+P` (o `Cmd+Shift+P` en macOS)
4. Escribe "Python: Select Interpreter"
5. Selecciona el intérprete del entorno virtual (aparecerá como `./venv/bin/python`)

Si no aparece:
- Asegúrate de que el entorno virtual esté creado
- Recarga la ventana: `Ctrl+Shift+P` → "Developer: Reload Window"

---

### "command not found: git"

**macOS:**
Instala Xcode Command Line Tools:
```bash
xcode-select --install
```

**Linux:**
```bash
sudo apt install git
```

---

## ¿Listo para empezar?

Si todo funciona:
- ✅ Python instalado y funcionando
- ✅ Node.js instalado y funcionando
- ✅ VSCode configurado con extensiones
- ✅ Git configurado con tu información
- ✅ Entorno virtual creado y activado

**Estás oficialmente listo para programar.**

---

## Siguiente paso

Ahora que tienes las herramientas, entiende para qué sirve cada una:

→ [05-herramientas-esenciales.md](./05-herramientas-esenciales.md)

Ahí te explico qué hace cada herramienta, cuándo usarla y cómo funciona en conjunto.

---

**Recuerda:** Esta configuración solo la haces una vez. Puede ser tedioso ahora, pero te ahorrará dolores de cabeza después. Tómate tu tiempo, sigue los pasos con calma y no te saltes las verificaciones.

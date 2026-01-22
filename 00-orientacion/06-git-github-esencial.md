# Git y GitHub - Control de Versiones Esencial

> **"Sin Git, no eres desarrollador. Con Git, eres profesional."**

---

## ¿Por qué es imprescindible?

Git/GitHub no es opcional en el desarrollo moderno. Es la herramienta más importante después de tu editor de código.

### Razones críticas:

1. **Historial de cambios** - Nunca pierdes código, puedes volver atrás
2. **Colaboración** - Trabaja en equipo sin destruir el código de otros
3. **Portfolio** - Tu perfil de GitHub ES tu CV técnico
4. **Deploy** - Todas las plataformas se integran con Git
5. **Backup automático** - Tu código está seguro en la nube
6. **Estándar de la industria** - 100% de las empresas lo usan

**Sin Git, no puedes completar este curriculum.** Punto.

---

## Conceptos básicos

### ¿Qué es Git?

Git es un **sistema de control de versiones distribuido**. En cristiano:
- Guarda "fotos" de tu código en diferentes momentos (commits)
- Puedes volver a cualquier "foto" anterior
- Puedes crear ramas (branches) para experimentar sin romper nada
- Puedes fusionar (merge) cambios de diferentes personas

### ¿Qué es GitHub?

GitHub es una **plataforma web para alojar repositorios Git**:
- Almacena tu código en la nube
- Permite colaboración visual (Pull Requests)
- Es tu portfolio público
- Se integra con herramientas de deploy (Railway, Vercel)

**Analogía:** Git es Word con "Guardar versión", GitHub es Google Drive.

---

## Instalación

### Windows

1. Descargar Git desde: https://git-scm.com/download/win
2. Instalar con opciones por defecto
3. Verificar en PowerShell:
   ```bash
   git --version
   ```
   Debe mostrar algo como: `git version 2.43.0`

### macOS

```bash
# Instalar con Homebrew
brew install git

# O instalar Xcode Command Line Tools
xcode-select --install
```

### Linux

```bash
# Ubuntu/Debian
sudo apt-get install git

# Fedora
sudo dnf install git
```

---

## Configuración inicial (OBLIGATORIA)

```bash
# Configurar tu nombre (aparecerá en cada commit)
git config --global user.name "Tu Nombre"

# Configurar tu email (debe coincidir con tu cuenta de GitHub)
git config --global user.email "tuemail@example.com"

# Configurar editor por defecto (opcional)
git config --global core.editor "code --wait"  # Para VS Code

# Verificar configuración
git config --list
```

**⚠️ IMPORTANTE:** El email debe ser el mismo que usarás en GitHub.

---

## Flujo básico de trabajo

### 1. Crear un repositorio local

```bash
# Crear carpeta para tu proyecto
mkdir mi-proyecto
cd mi-proyecto

# Inicializar Git
git init
```

Esto crea una carpeta oculta `.git` que guarda todo el historial.

---

### 2. Hacer cambios y guardarlos (commit)

```bash
# Crear un archivo
echo "# Mi Proyecto" > README.md

# Ver el estado (qué archivos cambiaron)
git status

# Agregar archivos al "staging area" (preparar para commit)
git add README.md

# O agregar todos los archivos
git add .

# Crear un commit (guardar una "foto" del código)
git commit -m "Agregar README inicial"
```

**Flujo visual:**

```
Archivos modificados
    ↓  git add
Staging Area (listos para commit)
    ↓  git commit
Historial (commit guardado)
```

---

### 3. Ver historial

```bash
# Ver lista de commits
git log

# Ver historial resumido
git log --oneline

# Ver cambios de un archivo
git log -p README.md
```

---

### 4. Crear una rama (branch)

Las ramas te permiten trabajar en nuevas features sin romper el código principal.

```bash
# Crear rama nueva
git branch nueva-feature

# Cambiar a esa rama
git checkout nueva-feature

# O crear y cambiar en un solo comando
git checkout -b nueva-feature

# Ver todas las ramas
git branch
```

**Analogía:** Tienes un documento Word. Haces "Guardar como" con otro nombre para experimentar. La rama `main` es tu documento original, `nueva-feature` es tu copia experimental.

---

### 5. Fusionar ramas (merge)

```bash
# Volver a la rama main
git checkout main

# Fusionar los cambios de nueva-feature
git merge nueva-feature

# Eliminar rama ya fusionada (opcional)
git branch -d nueva-feature
```

---

## GitHub - Trabajar en la nube

### 1. Crear cuenta en GitHub

1. Ve a https://github.com
2. Regístrate con el mismo email que configuraste en Git
3. Verifica tu email

---

### 2. Crear un repositorio en GitHub

1. Click en el botón **"+"** arriba a la derecha
2. **"New repository"**
3. Nombre: `mi-primer-repo`
4. Público o Privado (elige Público para portfolio)
5. **NO** marcar "Initialize with README" (ya lo tienes local)
6. Click en **"Create repository"**

---

### 3. Conectar tu repo local con GitHub

```bash
# Agregar el repositorio remoto (solo una vez)
git remote add origin https://github.com/tu-usuario/mi-primer-repo.git

# Verificar
git remote -v

# Subir tus commits a GitHub (push)
git push -u origin main
```

**⚠️ Nota:** Si tu rama se llama `master`, usa `git push -u origin master`

---

### 4. Flujo diario de trabajo con GitHub

```bash
# 1. Hacer cambios en tu código
# (editar archivos, crear nuevos, etc.)

# 2. Ver qué cambió
git status

# 3. Agregar archivos modificados
git add .

# 4. Crear commit
git commit -m "Agregar nueva funcionalidad"

# 5. Subir a GitHub
git push

# 6. Si alguien más hizo cambios, descargarlos primero
git pull
```

---

## Comandos esenciales que DEBES memorizar

| Comando | Qué hace |
|---------|----------|
| `git init` | Crear repo nuevo |
| `git clone <url>` | Descargar repo de GitHub |
| `git status` | Ver qué archivos cambiaron |
| `git add .` | Agregar todos los archivos al staging |
| `git commit -m "mensaje"` | Guardar una "foto" del código |
| `git push` | Subir commits a GitHub |
| `git pull` | Descargar cambios de GitHub |
| `git log` | Ver historial de commits |
| `git branch` | Ver/crear ramas |
| `git checkout <rama>` | Cambiar de rama |
| `git merge <rama>` | Fusionar rama |

---

## .gitignore - Archivos que NO debes subir

Crea un archivo `.gitignore` en la raíz de tu proyecto:

```
# Para proyectos Python
__pycache__/
*.pyc
venv/
.env

# Para proyectos Node.js/React
node_modules/
.env
.env.local
build/
dist/

# Archivos de sistema
.DS_Store
Thumbs.db

# Editor
.vscode/
.idea/

# Archivos sensibles
*.log
secrets.txt
database.sqlite3
```

**⚠️ MUY IMPORTANTE:** NUNCA subas:
- Contraseñas o API keys (usar `.env` y agregarlo a `.gitignore`)
- `node_modules/` (son miles de archivos, se reinstalan con `npm install`)
- Bases de datos con datos reales
- Archivos generados automáticamente

---

## Convenciones de commits (profesional)

### Formato estándar (Conventional Commits)

```
tipo(scope): descripción corta

Descripción larga opcional
```

**Tipos comunes:**

- `feat:` Nueva funcionalidad
- `fix:` Corrección de bug
- `docs:` Cambios en documentación
- `style:` Formato de código (sin cambios funcionales)
- `refactor:` Reestructuración de código
- `test:` Agregar o modificar tests
- `chore:` Tareas de mantenimiento

**Ejemplos:**

```bash
git commit -m "feat: agregar login con JWT"
git commit -m "fix: corregir bug en el carrito de compras"
git commit -m "docs: actualizar README con instrucciones de instalación"
git commit -m "refactor: extraer lógica de validación a función separada"
```

---

## Branches comunes en proyectos

```
main (o master) ────────────────────────────
  ↓
develop ────────────────────────────────────
  ↓        ↓         ↓
feature/ fix/    hotfix/
  ↓        ↓         ↓
  └──────┬┘         │
         ↓          ↓
      develop    main
```

- **`main`**: Código en producción (siempre funcional)
- **`develop`**: Desarrollo actual (integración)
- **`feature/nueva-feature`**: Nueva funcionalidad
- **`fix/nombre-bug`**: Corrección de bugs
- **`hotfix/urgente`**: Parche urgente en producción

---

## Pull Requests (PR) - Trabajo en equipo

Un **Pull Request** es una solicitud para fusionar tu código en otra rama.

### Flujo:

1. **Crear rama** para tu feature:
   ```bash
   git checkout -b feature/login
   ```

2. **Hacer commits** con tus cambios:
   ```bash
   git add .
   git commit -m "feat: implementar login"
   ```

3. **Subir rama a GitHub**:
   ```bash
   git push origin feature/login
   ```

4. **Crear PR en GitHub**:
   - Ve a tu repositorio en GitHub
   - Aparecerá un banner: "Compare & pull request"
   - Escribe descripción de qué hiciste
   - Asigna revisores (si trabajas en equipo)
   - Click en "Create pull request"

5. **Esperar review y aprobar**

6. **Merge a main** (fusionar)

7. **Eliminar rama** ya fusionada

---

## Errores comunes y soluciones

### Error: "Please tell me who you are"

```bash
# Falta configurar nombre y email
git config --global user.name "Tu Nombre"
git config --global user.email "tu@email.com"
```

---

### Error: "fatal: not a git repository"

```bash
# No estás en una carpeta con Git inicializado
git init
```

---

### Error: "Your branch is behind"

```bash
# Alguien hizo cambios en GitHub, debes descargarlos
git pull
```

---

### Error: "CONFLICT (content): Merge conflict"

Hay conflicto de merge (dos personas editaron las mismas líneas).

**Solución:**

1. Abre el archivo con conflicto
2. Verás algo así:
   ```
   <<<<<<< HEAD
   Tu código
   =======
   Código de otro
   >>>>>>> rama
   ```
3. Decide qué código mantener
4. Elimina las marcas `<<<<<<<`, `=======`, `>>>>>>>`
5. Guarda el archivo
6. Haz commit:
   ```bash
   git add .
   git commit -m "Resolver conflicto de merge"
   ```

---

### "Cagué todo, quiero volver atrás"

```bash
# Ver commits anteriores
git log --oneline

# Volver a un commit específico (sin perder cambios)
git reset --soft <hash-del-commit>

# Volver y PERDER todos los cambios (CUIDADO)
git reset --hard <hash-del-commit>

# Descartar cambios no guardados en un archivo
git checkout -- archivo.txt

# Descartar TODOS los cambios no guardados
git reset --hard HEAD
```

---

## GitHub como portfolio profesional

Tu perfil de GitHub es **tu CV técnico**. Los reclutadores lo revisan.

### Cómo tener un perfil profesional:

#### 1. README de perfil

Crea un repositorio con tu nombre de usuario (ej: `tu-usuario/tu-usuario`):

```markdown
# 👋 Hola, soy [Tu Nombre]

## 🚀 FullStack Developer

- 💻 React + Django + PostgreSQL
- 🎯 Apasionado por crear productos escalables
- 📫 Contacto: tu@email.com

## 🛠️ Tech Stack

![React](https://img.shields.io/badge/React-20232A?style=for-the-badge&logo=react)
![Django](https://img.shields.io/badge/Django-092E20?style=for-the-badge&logo=django)
![PostgreSQL](https://img.shields.io/badge/PostgreSQL-316192?style=for-the-badge&logo=postgresql)

## 📊 GitHub Stats

![Tus stats](https://github-readme-stats.vercel.app/api?username=tu-usuario&show_icons=true)

## 🔥 Proyectos destacados

- [E-commerce FullStack](link) - React + Django + PostgreSQL
- [API REST](link) - Django REST Framework
- [Portfolio](link) - React + Tailwind
```

---

#### 2. READMEs completos en tus repositorios

Cada proyecto debe tener un `README.md` profesional:

```markdown
# Nombre del Proyecto

> Descripción breve y atractiva

![Demo](screenshot.png)

## ✨ Features

- ✅ Autenticación JWT
- ✅ CRUD completo
- ✅ Testing (80%+ coverage)
- ✅ Deploy en Railway

## 🛠️ Tech Stack

- Frontend: React 18, Vite, Tailwind
- Backend: Django 5, DRF
- Database: PostgreSQL 15
- Testing: Pytest, Vitest, Playwright

## 📦 Instalación

\```bash
# Clonar repositorio
git clone https://github.com/tu-usuario/proyecto.git

# Instalar dependencias backend
cd backend
pip install -r requirements.txt

# Instalar dependencias frontend
cd frontend
npm install

# Correr proyecto
docker-compose up
\```

## 🚀 Deploy

[Ver demo en vivo](https://tu-proyecto.vercel.app)

## 📸 Screenshots

### Homepage
![Home](screenshots/home.png)

### Dashboard
![Dashboard](screenshots/dashboard.png)

## 📫 Contacto

- GitHub: [@tu-usuario](https://github.com/tu-usuario)
- LinkedIn: [Tu Nombre](https://linkedin.com/in/tu-perfil)
- Email: tu@email.com
```

---

#### 3. Commits regulares

Los reclutadores miran tu gráfico de contribuciones (los cuadraditos verdes).

**Tips:**
- Haz commits todos los días (aunque sean pequeños)
- Escribe mensajes de commit descriptivos
- Mantén racha de commits (consistency es clave)

---

#### 4. Proyectos públicos de calidad

**Mejor tener:**
- 3 proyectos completos y bien documentados
- Con READMEs profesionales
- Con deploy en vivo
- Con tests

**En vez de:**
- 20 proyectos a medias
- Sin documentación
- Código sin estructura

---

## Flujo completo: Tu primer proyecto en GitHub

### Paso 1: Crear proyecto local

```bash
# Crear carpeta
mkdir mi-portfolio
cd mi-portfolio

# Inicializar Git
git init

# Crear archivos
echo "# Mi Portfolio" > README.md
echo "node_modules/" > .gitignore

# Primer commit
git add .
git commit -m "feat: setup inicial del proyecto"
```

---

### Paso 2: Crear repo en GitHub

1. Ve a https://github.com/new
2. Nombre: `mi-portfolio`
3. Público
4. NO marcar "Initialize with README"
5. Create repository

---

### Paso 3: Conectar y subir

```bash
# Conectar con GitHub
git remote add origin https://github.com/tu-usuario/mi-portfolio.git

# Subir código
git push -u origin main
```

---

### Paso 4: Trabajar en features

```bash
# Crear rama para nueva feature
git checkout -b feature/homepage

# Hacer cambios, crear archivos...
# (código, código, código...)

# Guardar cambios
git add .
git commit -m "feat: agregar homepage con hero section"

# Subir rama
git push origin feature/homepage
```

---

### Paso 5: Pull Request

1. Ve a GitHub, verás banner "Compare & pull request"
2. Describe qué hiciste
3. Create pull request
4. Si trabajas solo, haz merge directamente
5. Si trabajas en equipo, espera review

---

### Paso 6: Merge y continuar

```bash
# Volver a main local
git checkout main

# Descargar cambios de GitHub (incluye tu merge)
git pull

# Eliminar rama ya fusionada
git branch -d feature/homepage
```

---

## Integraciones útiles de GitHub

### GitHub Pages
Hosting gratis para sitios estáticos (HTML/CSS/JS).

1. En tu repo → Settings → Pages
2. Source: `main` branch, `/root` folder
3. Save
4. Tu sitio estará en: `https://tu-usuario.github.io/repo-name`

---

### GitHub Actions
CI/CD automatizado (lo veremos en el módulo 08-despliegue).

---

### GitHub Discussions
Foro para tu proyecto.

---

### GitHub Projects
Tablero Kanban tipo Trello.

---

## Checklist: ¿Estoy listo para continuar?

Antes de avanzar al siguiente módulo, verifica:

- [ ] Tengo Git instalado (`git --version` funciona)
- [ ] Configuré mi nombre y email (`git config --list`)
- [ ] Tengo cuenta en GitHub (verificada)
- [ ] Creé mi primer repositorio en GitHub
- [ ] Hice al menos 5 commits
- [ ] Entiendo `git add`, `git commit`, `git push`
- [ ] Creé un README.md en mi perfil de GitHub
- [ ] Sé crear y fusionar ramas básicas

**Si marcaste todo ✅, estás listo para continuar.**

---

## Recursos adicionales

### Tutoriales interactivos
- **Learn Git Branching**: https://learngitbranching.js.org/ (visualización interactiva)
- **GitHub Skills**: https://skills.github.com/ (tutoriales oficiales)

### Documentación
- **Git oficial**: https://git-scm.com/doc
- **GitHub Docs**: https://docs.github.com/

### Cheat Sheets
- **Git Cheat Sheet**: https://education.github.com/git-cheat-sheet-education.pdf
- **GitHub Git Cheat Sheet**: https://training.github.com/downloads/github-git-cheat-sheet/

### Herramientas GUI (opcional)
- **GitHub Desktop**: https://desktop.github.com/
- **GitKraken**: https://www.gitkraken.com/
- **VS Code** tiene Git integrado (recomendado)

---

## Próximo paso

Una vez domines Git/GitHub, puedes continuar con el módulo:

📂 **[02-frontend/fundamentos](../02-frontend/fundamentos)** - Empezar con HTML, CSS y JavaScript

**Recuerda:** A partir de ahora, TODO tu código debe estar en Git/GitHub. No hay excusas.

---

## Errores que NO debes cometer

❌ **Subir contraseñas o API keys**
   Usa `.env` y agrégalo a `.gitignore`

❌ **No hacer commits frecuentes**
   Haz commits pequeños y frecuentes

❌ **Mensajes de commit vagos**
   ❌ "fix"
   ❌ "actualización"
   ❌ "asdf"
   ✅ "feat: agregar validación de email en formulario de registro"

❌ **Trabajar siempre en `main`**
   Usa ramas para cada feature

❌ **No leer mensajes de error**
   Git te dice exactamente qué hacer

❌ **Hacer `git push --force` sin entender**
   Puede destruir el trabajo de otros

---

## Comandos avanzados (para después)

Cuando domines lo básico, explora:

```bash
# Ver cambios antes de commit
git diff

# Modificar último commit
git commit --amend

# Guardar cambios temporalmente
git stash
git stash pop

# Ver quién modificó cada línea
git blame archivo.txt

# Buscar en el historial
git log --grep="keyword"

# Revertir un commit
git revert <hash>

# Rebase (reescribir historial)
git rebase main
```

---

## ¡Felicitaciones! 🎉

Ya tienes la herramienta más importante del desarrollo moderno.

**Git/GitHub no es opcional. Es obligatorio.**

Ahora estás listo para empezar a programar de verdad, sabiendo que:
- Nunca perderás tu código
- Puedes experimentar sin miedo
- Tienes un portfolio automático
- Puedes colaborar con otros
- Estás trabajando como un profesional

**Siguiente módulo:** [02-frontend/fundamentos/bloque-01-html](../02-frontend/fundamentos/bloque-01-html)

---

*"El mejor momento para aprender Git fue ayer. El segundo mejor momento es ahora."*

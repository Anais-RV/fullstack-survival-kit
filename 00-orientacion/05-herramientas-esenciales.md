# Herramientas esenciales

Ya las instalaste. Ahora vamos a entender **para qué sirve cada una** y cómo trabajar con ellas sin volverse loco.

No son herramientas por capricho. Cada una resuelve un problema real que tendrás todos los días.

---

## Visual Studio Code: Tu taller

**¿Qué es?**  
Un editor de código. Tu espacio de trabajo. Donde escribirás, editarás, debuggearás y organizarás tu código.

**Analogía:**  
Es como el escritorio de un carpintero. Puedes tallar madera con un cuchillo en cualquier sitio, pero en un taller con las herramientas adecuadas eres 10 veces más eficiente.

### ¿Por qué VSCode y no el Bloc de notas?

Porque VSCode te ayuda mientras escribes:

- **Autocompletado:** Empieza a escribir `cons` y te sugiere `console.log()`
- **Errores en tiempo real:** Te marca en rojo si escribes mal algo
- **Saltar a definiciones:** Haz clic en una función y te lleva a donde está definida
- **Buscar en todo el proyecto:** Encuentra todas las veces que usas una variable
- **Terminal integrada:** No tienes que cambiar de ventana constantemente
- **Git integrado:** Ves los cambios, haces commits, todo sin salir del editor

**El Bloc de notas no hace nada de esto.** Te deja solo con el texto. VSCode es tu copiloto.

### Atajos que debes memorizar

Estos te ahorrarán horas:

**Navegación:**
- `Ctrl+P` (o `Cmd+P` en Mac): Abre cualquier archivo escribiendo su nombre
- `Ctrl+Shift+P`: Paleta de comandos (acceso a todo)
- `Ctrl+B`: Ocultar/mostrar barra lateral
- `Ctrl+J`: Abrir/cerrar terminal

**Edición:**
- `Ctrl+D`: Selecciona la siguiente ocurrencia de la palabra
- `Alt+↑/↓`: Mueve la línea hacia arriba/abajo
- `Shift+Alt+↑/↓`: Duplica la línea
- `Ctrl+/`: Comenta/descomenta la línea
- `Ctrl+Shift+K`: Elimina la línea

**Multiples cursores:**
- `Alt+Click`: Añade cursor donde haces clic
- `Ctrl+Alt+↑/↓`: Añade cursor arriba/abajo

**Búsqueda:**
- `Ctrl+F`: Buscar en archivo actual
- `Ctrl+Shift+F`: Buscar en todo el proyecto
- `Ctrl+H`: Buscar y reemplazar

**No intentes memorizarlos todos ahora.** Usa 2-3 al principio. Con el tiempo, los internalizarás.

### Extensiones que ya instalaste (y qué hacen)

#### Python
Te da superpoderes para Python:
- Autocompletado inteligente
- Detecta errores antes de ejecutar
- Formatea tu código automáticamente
- Te deja hacer debugging línea por línea

#### ESLint
El "corrector ortográfico" de JavaScript:
- Te dice cuando algo puede causar errores
- Te avisa de malas prácticas
- Te ayuda a mantener un código consistente

#### Prettier
El "auto-formato" de código:
- Alinea las llaves, paréntesis y sangrías automáticamente
- Aplica un estilo consistente en todo el proyecto
- Se ejecuta al guardar (si configuraste `formatOnSave`)

**No más debates sobre si la llave va en la misma línea o abajo.** Prettier decide por ti.

#### GitLens
Git con esteroides:
- Ves quién escribió cada línea (y cuándo)
- Historial de cambios de un archivo
- Comparación visual de commits

---

## Terminal: La línea de comandos

**¿Qué es?**  
Una forma de dar órdenes a tu ordenador escribiendo texto en lugar de hacer clics.

**Analogía:**  
Es como hablar directamente con el ordenador en lugar de usar un mando a distancia. Es más rápido una vez que aprendes los comandos.

### ¿Por qué usarla si tengo interfaz gráfica?

Porque muchas cosas en desarrollo **solo se hacen por terminal**:

- Instalar paquetes (`npm install`, `pip install`)
- Ejecutar servidores (`python manage.py runserver`)
- Usar Git (`git commit`, `git push`)
- Crear proyectos (`npx create-react-app`)
- Ejecutar tests (`pytest`, `npm test`)

**No es opcional.** Es parte del trabajo.

### Comandos básicos que necesitas

#### Navegación

**Ver dónde estás:**
```bash
pwd          # "print working directory" - muestra tu ubicación actual
```

**Ver qué hay en la carpeta:**
```bash
ls           # Linux/Mac
dir          # Windows
```

**Entrar en una carpeta:**
```bash
cd nombre-carpeta        # "change directory"
cd ..                    # Subir un nivel
cd ~                     # Ir a tu carpeta de usuario
```

**Crear carpeta:**
```bash
mkdir nombre-carpeta     # "make directory"
```

**Crear archivo:**
```bash
touch archivo.txt        # Linux/Mac
echo. > archivo.txt      # Windows
```

**Borrar archivo:**
```bash
rm archivo.txt           # Linux/Mac
del archivo.txt          # Windows
```

**Borrar carpeta (¡cuidado!):**
```bash
rm -rf carpeta           # Linux/Mac
rmdir /s carpeta         # Windows
```

> ⚠️ **Advertencia:** `rm -rf` borra sin pedir confirmación y no va a la papelera. No lo uses a la ligera.

#### Gestión de paquetes

**Python:**
```bash
pip install nombre-paquete      # Instala un paquete
pip list                        # Lista paquetes instalados
pip freeze > requirements.txt   # Guarda lista de paquetes
```

**Node.js:**
```bash
npm install nombre-paquete      # Instala un paquete
npm install                     # Instala todo lo del package.json
npm start                       # Ejecuta el script "start"
```

### Terminal en VSCode

No necesitas abrir una terminal aparte. VSCode tiene una integrada:

- Presiona `Ctrl+`` (acento grave) o `Ctrl+J`
- Aparece en la parte inferior
- Puedes tener varias terminales abiertas a la vez
- Puedes dividirlas en paneles

**Ventaja:** Estás en la carpeta correcta automáticamente (la del proyecto abierto).

### PowerShell vs CMD vs Bash

**Windows tiene tres terminales:**
- **CMD** (Command Prompt): La antigua. Funcional pero limitada.
- **PowerShell**: Más moderna, más poderosa. Usa esta si estás en Windows.
- **Git Bash**: Terminal tipo Linux en Windows. Útil si vienes de Mac/Linux.

**macOS/Linux:**
- **Bash** o **Zsh**: Tu terminal por defecto. Usa esa.

**¿Cuál usar?**  
En Windows: PowerShell o Git Bash.  
En Mac/Linux: La que viene por defecto.

---

## Git: Control de versiones

**¿Qué es?**  
Un sistema que guarda el historial de cambios de tu código. Como "Ctrl+Z" con superpoderes.

**Analogía:**  
Es como escribir un documento con "Control de cambios" activado. Puedes ver qué cambió, quién lo cambió, cuándo y por qué. Y puedes volver a cualquier versión anterior.

### ¿Por qué lo necesitas?

#### Razón 1: Historial de cambios
Tu código funcionaba ayer. Hoy no. ¿Qué cambió?  
Con Git: ves exactamente qué líneas cambiaron.

#### Razón 2: Experimentar sin miedo
Quieres probar algo arriesgado pero no quieres romper lo que funciona.  
Con Git: creas una rama, experimentas. Si funciona, lo integras. Si no, lo borras.

#### Razón 3: Colaboración
Dos personas trabajando en el mismo proyecto.  
Con Git: cada uno trabaja por su lado, luego combinan los cambios sin pisarse.

#### Razón 4: Backup automático
Tu disco duro se rompe.  
Con Git (y GitHub/GitLab): tu código está en la nube. No pierdes nada.

### Conceptos básicos de Git

#### Repository (Repo)
Tu proyecto. Una carpeta con Git activado.

#### Commit
Una "foto" de tu código en un momento específico. Un punto de guardado.

#### Branch (Rama)
Una línea de desarrollo paralela. Puedes tener:
- `main`: La versión estable
- `feature-login`: Donde estás haciendo el login
- `bugfix-navbar`: Donde arreglas el menú

#### Remote
La versión del repo en internet (GitHub, GitLab, etc.).

### Flujo básico de Git

Este es el ciclo que harás 100 veces al día:

```bash
# 1. Ver qué cambió
git status

# 2. Añadir archivos al "stage" (preparar para guardar)
git add archivo.txt              # Un archivo específico
git add .                        # Todo lo que cambió

# 3. Guardar los cambios con un mensaje
git commit -m "Añadir validación de email"

# 4. Subir los cambios al servidor
git push
```

**En español:**
1. ¿Qué archivos cambié?
2. Estos son los que quiero guardar.
3. Guárdalos con esta descripción.
4. Súbelos a internet.

### Comandos Git esenciales

**Empezar:**
```bash
git init                    # Crear un repo nuevo
git clone url-del-repo      # Copiar un repo existente
```

**Día a día:**
```bash
git status                  # ¿Qué cambió?
git add archivo             # Preparar archivo para commit
git add .                   # Preparar todo
git commit -m "mensaje"     # Guardar cambios
git push                    # Subir al servidor
git pull                    # Traer cambios del servidor
```

**Ramas:**
```bash
git branch                  # Ver ramas
git branch nombre           # Crear rama
git checkout nombre         # Cambiar a rama
git checkout -b nombre      # Crear y cambiar a rama
git merge nombre            # Fusionar rama actual con "nombre"
```

**Historial:**
```bash
git log                     # Ver historial de commits
git log --oneline           # Versión compacta
git diff                    # Ver cambios no guardados
```

**Emergencias:**
```bash
git reset --hard            # ⚠️ Descartar TODOS los cambios
git checkout -- archivo     # Descartar cambios de un archivo
git revert commit-id        # Deshacer un commit específico
```

### Git en VSCode

No necesitas usar la terminal para Git. VSCode tiene interfaz gráfica:

1. **Icono de control de código fuente** (tercer icono en la barra lateral)
2. Ves los archivos cambiados
3. Haces clic en `+` para añadirlos
4. Escribes el mensaje del commit arriba
5. Haces clic en ✓ para commitear
6. Haces clic en `...` → `Push` para subir

**Ventaja:** Visual, intuitivo, no tienes que memorizar comandos.  
**Desventaja:** A veces necesitarás la terminal para cosas avanzadas.

**Recomendación:** Empieza con la interfaz de VSCode. Aprende los comandos después.

---

## Node.js: JavaScript fuera del navegador

**¿Qué es?**  
Un entorno que te permite ejecutar JavaScript en tu ordenador, no solo en el navegador.

**Analogía:**  
JavaScript nació en el navegador (como los peces en el agua). Node.js es como darle pulmones: ahora puede vivir fuera del navegador también.

### ¿Por qué lo necesitas si hacemos backend en Python?

Porque el **frontend usa herramientas JavaScript**:

- **React** necesita Node.js para compilar
- **npm** (el gestor de paquetes de JavaScript) viene con Node.js
- **Vite** (herramienta de desarrollo) necesita Node.js
- **ESLint** y **Prettier** necesitan Node.js

**Node.js no es para tu backend. Es para tus herramientas de frontend.**

### npm: El gestor de paquetes

**¿Qué es?**  
Un catálogo gigante de código reutilizable. Como una biblioteca infinita donde puedes tomar prestado código que otros escribieron.

**Ejemplo:**  
Necesitas validar emails. En lugar de escribir toda la lógica desde cero:

```bash
npm install validator
```

Y ya tienes una librería con validación probada y mantenida.

### package.json: La lista de ingredientes

Cada proyecto frontend tiene un archivo `package.json`:

```json
{
  "name": "mi-app",
  "version": "1.0.0",
  "dependencies": {
    "react": "^18.2.0",
    "axios": "^1.4.0"
  },
  "scripts": {
    "start": "vite",
    "build": "vite build"
  }
}
```

**¿Qué es esto?**
- **dependencies**: Los paquetes que tu proyecto necesita
- **scripts**: Comandos abreviados que puedes ejecutar

**Si clonas un proyecto con package.json:**

```bash
npm install
```

Y npm instala todo lo que necesita automáticamente.

---

## Python: El lenguaje del backend

**¿Qué es?**  
El lenguaje de programación que usarás para el backend.

### ¿Por qué Python?

#### Razón 1: Sintaxis clara
```python
if usuario.esta_autenticado():
    mostrar_dashboard()
else:
    redirigir_a_login()
```

Se lee casi como inglés. Otros lenguajes requieren más símbolos raros.

#### Razón 2: Ecosistema enorme
Paquetes para TODO: APIs, machine learning, automatización, scraping, procesamiento de imágenes...

#### Razón 3: Demanda laboral
Backend en Python está en todas partes: startups, grandes empresas, ciencia de datos, IA.

### pip: El npm de Python

**¿Qué es?**  
El gestor de paquetes de Python. Funciona igual que npm pero para Python.

```bash
pip install django        # Instalar un paquete
pip list                  # Ver qué tienes instalado
pip uninstall django      # Desinstalar
```

### requirements.txt: El package.json de Python

Cada proyecto Python tiene (o debería tener) un `requirements.txt`:

```
django==4.2.0
fastapi==0.100.0
psycopg2-binary==2.9.6
```

**Es la lista de dependencias.** Si clonas un proyecto:

```bash
pip install -r requirements.txt
```

Y se instala todo.

### Entornos virtuales: Por qué son importantes

**El problema:**  
Proyecto A necesita Django 3.2.  
Proyecto B necesita Django 4.2.  
Si instalas ambos globalmente, se pisan.

**La solución: Entornos virtuales**  
Cada proyecto tiene su propia "burbuja" de paquetes.

```bash
# Crear entorno
python -m venv venv

# Activar
source venv/bin/activate    # Mac/Linux
.\venv\Scripts\Activate.ps1  # Windows

# Ahora todo lo que instales va a este entorno
pip install django

# Desactivar
deactivate
```

**Regla de oro:** Siempre trabaja con entornos virtuales activados. Siempre.

---

## Navegador: Más que solo navegar

**¿Qué es?**  
Tu herramienta de frontend. Donde ejecutas, pruebas y debuggeas aplicaciones web.

### DevTools: Tus rayos X

Todos los navegadores modernos tienen herramientas de desarrollo. Presiona **F12** (o `Cmd+Option+I` en Mac).

#### Pestaña Elements
- Ve el HTML y CSS en tiempo real
- Modifica estilos temporalmente para probar
- Inspecciona cualquier elemento de la página

#### Pestaña Console
- Ejecuta JavaScript en tiempo real
- Ves errores y logs
- Prueba código rápidamente

#### Pestaña Network
- Ve todas las peticiones HTTP
- Cuánto tardan
- Qué datos envían y reciben

#### Pestaña Application
- Ve cookies, localStorage, sessionStorage
- Gestiona datos guardados en el navegador

### ¿Chrome, Firefox, Safari, Edge?

**Para desarrollo: Chrome o Firefox.**

Ambos tienen DevTools excelentes. Chrome tiene una comunidad más grande de extensiones de desarrollo.

**Para probar compatibilidad:** Usa todos. Cada navegador interpreta cosas ligeramente diferente.

---

## PostgreSQL: La base de datos (más adelante)

**¿Qué es?**  
La base de datos relacional que usarás. No la instales todavía, esperamos al módulo 04-bases-de-datos.

**¿Por qué la menciono ahora?**  
Para que sepas que viene. Es parte del stack.

**Por ahora:** Solo ten en mente que existe.

---

## Cómo trabajan juntas todas estas herramientas

Imagina que estás construyendo una aplicación fullstack:

### Frontend (React)
1. Escribes código en **VSCode**
2. Usas **npm** para instalar paquetes
3. **Node.js** ejecuta las herramientas de desarrollo
4. Pruebas en el **navegador**
5. Guardas cambios con **Git**

### Backend (Python)
1. Escribes código en **VSCode**
2. Usas **pip** para instalar paquetes
3. **Python** ejecuta tu servidor
4. Pruebas con **Postman** o el navegador
5. Guardas cambios con **Git**

### Base de datos (PostgreSQL)
1. Diseñas tablas
2. El **backend Python** se conecta a **PostgreSQL**
3. Guardas los esquemas con **Git** (en archivos de migración)

### Todo junto
```
VSCode (escribes código)
   ↓
Git (guardas cambios)
   ↓
Terminal (ejecutas comandos)
   ↓
Navegador (ves resultados)
```

**Es un ciclo continuo.** Escribes, ejecutas, pruebas, ajustas, guardas. Repite.

---

## Atajos de productividad

### En VSCode
- `Ctrl+P`: Abre archivos instantáneamente
- `Ctrl+D`: Selecciona múltiples ocurrencias
- `Ctrl+Shift+P`: Busca cualquier comando

### En Terminal
- `↑` (flecha arriba): Repite el último comando
- `Ctrl+C`: Cancela el comando actual
- `Ctrl+L`: Limpia la pantalla
- `Tab`: Autocompletar nombres de archivos

### En Navegador
- `F12`: Abre DevTools
- `Ctrl+Shift+C`: Selecciona elemento en la página
- `Ctrl+Shift+R`: Recarga sin caché

---

## Errores comunes de principiantes

### Error 1: No activar el entorno virtual
Instalas paquetes globalmente. Luego proyectos se rompen entre sí.

**Solución:** Siempre `source venv/bin/activate` antes de trabajar.

### Error 2: No hacer commits frecuentemente
Trabajas 3 días sin hacer commit. Algo se rompe. No sabes qué cambió.

**Solución:** Commit cada vez que algo funciona. "Funciona → commit" es el ritmo.

### Error 3: No leer mensajes de error
Ves un error rojo y entras en pánico.

**Solución:** Lee el mensaje. Casi siempre te dice exactamente qué está mal y en qué línea.

### Error 4: Olvidar `npm install` / `pip install`
Clonas un proyecto. Intentas ejecutarlo. Falla con "módulo no encontrado".

**Solución:** Siempre instala dependencias primero (`npm install` o `pip install -r requirements.txt`).

### Error 5: Trabajar en `main` directamente
Haces cambios experimentales en la rama principal. Se rompe. No tienes vuelta atrás limpia.

**Solución:** Crea ramas para experimentos (`git checkout -b feature-nueva`).

---

## Recursos para profundizar

### VSCode
- [Documentación oficial](https://code.visualstudio.com/docs)
- [Atajos de teclado](https://code.visualstudio.com/shortcuts/keyboard-shortcuts-windows.pdf)

### Git
- [Pro Git (libro gratuito)](https://git-scm.com/book/es/v2)
- [Learn Git Branching (interactivo)](https://learngitbranching.js.org/)

### Terminal
- [The Linux Command Line (libro gratuito)](http://linuxcommand.org/tlcl.php)
- [Curso interactivo de terminal](https://www.codecademy.com/learn/learn-the-command-line)

### Python
- [Documentación oficial](https://docs.python.org/3/)
- [Python Package Index (PyPI)](https://pypi.org/)

### Node.js/npm
- [Documentación de Node.js](https://nodejs.org/docs/)
- [npm Docs](https://docs.npmjs.com/)

---

## Siguiente paso

Ya tienes todo configurado y entiendes qué hace cada herramienta.

**Es hora de programar.**

→ Continúa con el módulo **01-fundamentos**

Ahí empezarás a escribir código de verdad, no solo configurar herramientas.

---

**Recuerda:** Estas herramientas parecen muchas ahora, pero en dos semanas las usarás sin pensar. Es como aprender a conducir: al principio todo es abrumador (espejo, intermitente, cambios...), luego se vuelve automático. Dale tiempo.

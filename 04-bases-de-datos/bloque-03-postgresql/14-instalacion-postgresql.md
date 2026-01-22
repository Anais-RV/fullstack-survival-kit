# Instalación y Configuración de PostgreSQL

> **Instalar y configurar el motor de base de datos PostgreSQL**

---

## ¿Qué es PostgreSQL?

**PostgreSQL** es un sistema de gestión de bases de datos relacional **open source** y **profesional**.

**Características:**
- ✅ ACID completo (transacciones confiables)
- ✅ Tipos de datos avanzados (JSON, Arrays, UUID, etc.)
- ✅ Rendimiento excepcional
- ✅ Extensible (plugins, funciones personalizadas)
- ✅ Usado por empresas grandes (Instagram, Spotify, Netflix)

**SQLite vs PostgreSQL:**

| Característica | SQLite | PostgreSQL |
|---|---|---|
| **Instalación** | Sin servidor | Servidor dedicado |
| **Concurrencia** | Lectura múltiple, escritura única | Múltiples conexiones simultáneas |
| **Uso** | Apps móviles, desarrollo local | Producción, aplicaciones web |
| **Tamaño BD** | < 1 GB | Terabytes |
| **Usuarios** | 1 | Miles simultáneos |

**Cuándo usar cada uno:**
- **SQLite**: Aprendizaje, prototipos, apps móviles
- **PostgreSQL**: Aplicaciones en producción

---

## Instalación en Windows

### Método 1: Instalador oficial

**1. Descargar:**
- Ir a: https://www.postgresql.org/download/windows/
- Click en "Download the installer"
- Elegir versión (16.x recomendado)

**2. Ejecutar instalador:**
- Siguiente → Siguiente
- Directorio por defecto: `C:\Program Files\PostgreSQL\16`
- Componentes a instalar:
  - ✅ PostgreSQL Server
  - ✅ pgAdmin 4
  - ✅ Command Line Tools
  - ❌ Stack Builder (opcional)

**3. Configuración:**
- **Puerto**: `5432` (por defecto)
- **Contraseña superusuario (postgres)**: ¡Guardarla!
- **Locale**: Spanish, Spain (o Default locale)

**4. Finalizar:**
- Deseleccionar "Launch Stack Builder"
- Finish

**Verificar instalación:**

```powershell
# Abrir PowerShell
psql --version
# Resultado: psql (PostgreSQL) 16.x
```

---

### Método 2: Chocolatey (más rápido)

```powershell
# Instalar Chocolatey si no lo tienes
# Ver: https://chocolatey.org/install

# Instalar PostgreSQL
choco install postgresql16 -y

# Verificar
psql --version
```

---

## Instalación en Linux (Ubuntu/Debian)

```bash
# Actualizar paquetes
sudo apt update

# Instalar PostgreSQL
sudo apt install postgresql postgresql-contrib -y

# Verificar servicio
sudo systemctl status postgresql

# Iniciar servicio (si no está activo)
sudo systemctl start postgresql
sudo systemctl enable postgresql

# Verificar versión
psql --version
```

---

## Instalación en macOS

### Método 1: Homebrew

```bash
# Instalar Homebrew si no lo tienes
# Ver: https://brew.sh

# Instalar PostgreSQL
brew install postgresql@16

# Iniciar servicio
brew services start postgresql@16

# Verificar
psql --version
```

### Método 2: Postgres.app

- Descargar de: https://postgresapp.com/
- Arrastrar a Applications
- Click en "Initialize" para crear cluster
- Agregar a PATH (opcional):

```bash
# En ~/.zshrc o ~/.bash_profile
export PATH="/Applications/Postgres.app/Contents/Versions/latest/bin:$PATH"
```

---

## Primer acceso a PostgreSQL

### Windows

**Opción 1: SQL Shell (psql)**

1. Buscar "SQL Shell (psql)" en el menú inicio
2. Presionar Enter para valores por defecto:
   - Server: localhost
   - Database: postgres
   - Port: 5432
   - Username: postgres
3. Ingresar contraseña del superusuario

**Opción 2: PowerShell**

```powershell
# Conectar como superusuario
psql -U postgres

# Si pide contraseña, ingresarla
```

---

### Linux/macOS

```bash
# Cambiar a usuario postgres (Linux)
sudo -u postgres psql

# O conectar directamente
psql -U postgres
```

---

## Comandos básicos de psql

Una vez dentro de `psql`:

```sql
-- Ver versión
SELECT version();

-- Listar bases de datos
\l

-- Conectar a una base de datos
\c nombre_base_datos

-- Listar tablas
\dt

-- Describir tabla
\d nombre_tabla

-- Listar usuarios (roles)
\du

-- Ejecutar archivo SQL
\i /ruta/archivo.sql

-- Salir
\q
```

---

## Crear usuario y base de datos

### Desde psql

```sql
-- Crear usuario
CREATE USER miusuario WITH PASSWORD 'mipassword';

-- Crear base de datos
CREATE DATABASE mibasedatos OWNER miusuario;

-- Dar privilegios
GRANT ALL PRIVILEGES ON DATABASE mibasedatos TO miusuario;

-- Conectar como nuevo usuario
\c mibasedatos miusuario
```

### Desde terminal

```bash
# Crear usuario
createuser -U postgres -P miusuario
# Te pedirá contraseña

# Crear base de datos
createdb -U postgres -O miusuario mibasedatos

# Conectar
psql -U miusuario -d mibasedatos
```

---

## Ejemplo práctico: Proyecto FullStack

```sql
-- Como superusuario (postgres)
CREATE USER fullstack_user WITH PASSWORD 'fullstack123';

CREATE DATABASE fullstack_db 
    OWNER fullstack_user
    ENCODING 'UTF8';

GRANT ALL PRIVILEGES ON DATABASE fullstack_db TO fullstack_user;

-- Conectar a la nueva BD
\c fullstack_db fullstack_user

-- Crear tabla de prueba
CREATE TABLE usuarios (
    id SERIAL PRIMARY KEY,
    nombre VARCHAR(100) NOT NULL,
    email VARCHAR(150) UNIQUE NOT NULL,
    creado_en TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Insertar datos
INSERT INTO usuarios (nombre, email) VALUES
    ('Ana García', 'ana@email.com'),
    ('Bob Smith', 'bob@email.com');

-- Consultar
SELECT * FROM usuarios;
```

---

## Configuración del servidor

### Archivo postgresql.conf

**Ubicación:**
- **Windows**: `C:\Program Files\PostgreSQL\16\data\postgresql.conf`
- **Linux**: `/etc/postgresql/16/main/postgresql.conf`
- **macOS (Homebrew)**: `/usr/local/var/postgresql@16/postgresql.conf`

**Configuraciones importantes:**

```conf
# Conexiones
listen_addresses = 'localhost'  # Solo local
max_connections = 100           # Conexiones máximas

# Memoria
shared_buffers = 128MB          # Caché compartido
work_mem = 4MB                  # Memoria por operación

# Logs
logging_collector = on
log_directory = 'log'
log_filename = 'postgresql-%Y-%m-%d.log'
log_line_prefix = '%t [%p]: [%l-1] user=%u,db=%d '

# Rendimiento
effective_cache_size = 4GB      # Caché del sistema
```

**Aplicar cambios:**

```bash
# Linux
sudo systemctl restart postgresql

# macOS
brew services restart postgresql@16

# Windows (PowerShell como Administrador)
Restart-Service postgresql-x64-16
```

---

### Archivo pg_hba.conf (autenticación)

**Ubicación:** Misma carpeta que `postgresql.conf`

**Configuración por defecto:**

```conf
# TYPE  DATABASE    USER        ADDRESS         METHOD

# Local connections
local   all         all                         peer

# IPv4 local connections
host    all         all         127.0.0.1/32    scram-sha-256

# IPv6 local connections
host    all         all         ::1/128         scram-sha-256
```

**Métodos de autenticación:**
- `trust` - Sin contraseña (¡peligroso!)
- `peer` - Usuario del sistema (Linux/macOS)
- `md5` - Contraseña MD5 (obsoleto)
- `scram-sha-256` - Contraseña segura (recomendado)
- `reject` - Rechazar conexión

**Permitir conexiones remotas:**

```conf
# Agregar al final
host    all         all         0.0.0.0/0       scram-sha-256
```

Y en `postgresql.conf`:

```conf
listen_addresses = '*'  # Escuchar en todas las interfaces
```

---

## Roles y permisos

### Crear roles

```sql
-- Rol sin login (grupo)
CREATE ROLE lectores;

-- Rol con login (usuario)
CREATE ROLE usuario_app WITH LOGIN PASSWORD 'pass123';

-- Rol con múltiples permisos
CREATE ROLE admin_app WITH 
    LOGIN 
    PASSWORD 'admin123'
    CREATEDB 
    CREATEROLE;

-- Superusuario (¡cuidado!)
CREATE ROLE superadmin WITH 
    LOGIN 
    PASSWORD 'super123' 
    SUPERUSER;
```

### Asignar permisos

```sql
-- Permisos de lectura en una tabla
GRANT SELECT ON usuarios TO lectores;

-- Permisos de escritura
GRANT INSERT, UPDATE, DELETE ON usuarios TO usuario_app;

-- Todos los permisos en una tabla
GRANT ALL PRIVILEGES ON usuarios TO admin_app;

-- Permisos en todas las tablas de un schema
GRANT SELECT ON ALL TABLES IN SCHEMA public TO lectores;

-- Permisos por defecto para tablas futuras
ALTER DEFAULT PRIVILEGES IN SCHEMA public
    GRANT SELECT ON TABLES TO lectores;
```

### Asignar roles a usuarios

```sql
-- Usuario pertenece al grupo "lectores"
GRANT lectores TO usuario_app;

-- Revocar permisos
REVOKE SELECT ON usuarios FROM lectores;

-- Ver permisos de una tabla
\dp usuarios

-- Ver roles de un usuario
\du usuario_app
```

---

## Herramientas incluidas

### createdb / dropdb

```bash
# Crear BD
createdb -U postgres mi_nueva_bd

# Eliminar BD
dropdb -U postgres mi_nueva_bd
```

### pg_dump (backup)

```bash
# Backup completo
pg_dump -U postgres -d mi_bd -f backup.sql

# Solo datos
pg_dump -U postgres -d mi_bd --data-only -f datos.sql

# Solo estructura
pg_dump -U postgres -d mi_bd --schema-only -f estructura.sql

# Formato comprimido
pg_dump -U postgres -d mi_bd -Fc -f backup.dump
```

### pg_restore (restaurar)

```bash
# Restaurar desde .sql
psql -U postgres -d nueva_bd -f backup.sql

# Restaurar desde .dump
pg_restore -U postgres -d nueva_bd backup.dump

# Limpiar BD antes de restaurar
pg_restore -U postgres -d nueva_bd -c backup.dump
```

---

## Caso práctico: Setup completo

```bash
# 1. Crear usuario para tu app
createuser -U postgres -P app_user
# Contraseña: app_pass123

# 2. Crear base de datos
createdb -U postgres -O app_user tienda_db

# 3. Conectar
psql -U app_user -d tienda_db

# 4. Crear estructura
CREATE TABLE categorias (
    id SERIAL PRIMARY KEY,
    nombre VARCHAR(100) NOT NULL UNIQUE
);

CREATE TABLE productos (
    id SERIAL PRIMARY KEY,
    nombre VARCHAR(200) NOT NULL,
    precio DECIMAL(10, 2) NOT NULL CHECK (precio > 0),
    stock INTEGER DEFAULT 0 CHECK (stock >= 0),
    categoria_id INTEGER REFERENCES categorias(id),
    creado_en TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_productos_categoria ON productos(categoria_id);

# 5. Insertar datos de prueba
INSERT INTO categorias (nombre) VALUES 
    ('Electrónica'),
    ('Ropa'),
    ('Hogar');

INSERT INTO productos (nombre, precio, stock, categoria_id) VALUES
    ('Laptop Dell', 1200.00, 10, 1),
    ('Mouse Logitech', 25.50, 50, 1),
    ('Camisa Polo', 35.00, 30, 2),
    ('Mesa Escritorio', 150.00, 5, 3);

# 6. Consultar
SELECT 
    p.nombre AS producto,
    p.precio,
    c.nombre AS categoria
FROM productos p
INNER JOIN categorias c ON p.categoria_id = c.id
ORDER BY p.precio DESC;

# 7. Hacer backup
\! pg_dump -U app_user -d tienda_db -f backup_tienda.sql

# 8. Salir
\q
```

---

## Variables de entorno

Para no escribir usuario/contraseña cada vez:

### Windows (PowerShell)

```powershell
# Temporal (solo sesión actual)
$env:PGUSER = "app_user"
$env:PGPASSWORD = "app_pass123"
$env:PGDATABASE = "tienda_db"

# Conectar sin parámetros
psql
```

### Linux/macOS

```bash
# En ~/.bashrc o ~/.zshrc
export PGUSER=app_user
export PGPASSWORD=app_pass123
export PGDATABASE=tienda_db

# Recargar
source ~/.bashrc

# Conectar sin parámetros
psql
```

### Archivo .pgpass

**Más seguro que variables de entorno:**

**Ubicación:**
- **Linux/macOS**: `~/.pgpass`
- **Windows**: `%APPDATA%\postgresql\pgpass.conf`

**Formato:**
```
hostname:port:database:username:password
```

**Ejemplo:**

```
localhost:5432:tienda_db:app_user:app_pass123
localhost:5432:*:app_user:app_pass123
*:*:*:postgres:admin123
```

**Permisos (Linux/macOS):**

```bash
chmod 600 ~/.pgpass
```

---

## Conectar desde aplicaciones

### Python (psycopg2)

```python
import psycopg2

# Conectar
conn = psycopg2.connect(
    host="localhost",
    port=5432,
    database="tienda_db",
    user="app_user",
    password="app_pass123"
)

# Cursor
cur = conn.cursor()

# Ejecutar query
cur.execute("SELECT * FROM productos")
productos = cur.fetchall()

for producto in productos:
    print(producto)

# Cerrar
cur.close()
conn.close()
```

### Node.js (pg)

```javascript
const { Client } = require('pg');

const client = new Client({
    host: 'localhost',
    port: 5432,
    database: 'tienda_db',
    user: 'app_user',
    password: 'app_pass123'
});

await client.connect();

const result = await client.query('SELECT * FROM productos');
console.log(result.rows);

await client.end();
```

---

## Comandos útiles de administración

```sql
-- Ver todas las bases de datos
SELECT datname FROM pg_database;

-- Ver todos los usuarios
SELECT usename FROM pg_user;

-- Tamaño de una base de datos
SELECT pg_size_pretty(pg_database_size('tienda_db'));

-- Tamaño de una tabla
SELECT pg_size_pretty(pg_total_relation_size('productos'));

-- Conexiones activas
SELECT 
    pid,
    usename,
    datname,
    client_addr,
    state
FROM pg_stat_activity
WHERE state = 'active';

-- Terminar conexión
SELECT pg_terminate_backend(12345);  -- Reemplazar con PID

-- Ver tablas más grandes
SELECT 
    schemaname,
    tablename,
    pg_size_pretty(pg_total_relation_size(schemaname||'.'||tablename)) AS size
FROM pg_tables
WHERE schemaname = 'public'
ORDER BY pg_total_relation_size(schemaname||'.'||tablename) DESC
LIMIT 10;

-- Vaciar tabla (mantener estructura)
TRUNCATE TABLE productos;

-- Resetear secuencia
ALTER SEQUENCE productos_id_seq RESTART WITH 1;
```

---

## Resolución de problemas

### No puedo conectar

**Error:** `psql: error: connection to server at "localhost" (::1), port 5432 failed`

**Soluciones:**

```bash
# 1. Verificar que PostgreSQL está corriendo
# Windows
Get-Service postgresql-x64-16

# Linux
sudo systemctl status postgresql

# macOS
brew services list

# 2. Verificar puerto
netstat -an | findstr 5432  # Windows
sudo netstat -tulnp | grep 5432  # Linux

# 3. Verificar pg_hba.conf permite conexión
```

### Olvidé la contraseña del superusuario

```bash
# 1. Editar pg_hba.conf
# Cambiar método de autenticación a "trust" temporalmente

# Linux
sudo nano /etc/postgresql/16/main/pg_hba.conf

# Cambiar:
# local   all   postgres   peer
# Por:
# local   all   postgres   trust

# 2. Reiniciar PostgreSQL
sudo systemctl restart postgresql

# 3. Conectar sin contraseña
psql -U postgres

# 4. Cambiar contraseña
ALTER USER postgres PASSWORD 'nueva_contraseña';

# 5. Revertir pg_hba.conf a "peer" o "scram-sha-256"

# 6. Reiniciar nuevamente
```

### Puerto 5432 ya en uso

```bash
# Windows: Cambiar puerto en postgresql.conf
# Buscar: port = 5432
# Cambiar a: port = 5433

# Reiniciar servicio
```

---

## Buenas prácticas

✅ **No uses el superusuario (postgres) en aplicaciones**
- Crea usuarios específicos con permisos limitados

✅ **Usa contraseñas seguras**
- Mínimo 12 caracteres, letras, números, símbolos

✅ **Backups regulares**
- Automatizar `pg_dump` diario/semanal

✅ **Monitorea conexiones**
- Revisa `pg_stat_activity` periódicamente

✅ **Actualiza PostgreSQL**
- Mantén versión actualizada para seguridad

✅ **Usa SSL en producción**
- Encripta conexiones remotas

✅ **Configura log apropiadamente**
- Ayuda en debugging y auditoría

---

## Resumen

**Instalación:**
- Windows: Instalador oficial o Chocolatey
- Linux: `apt install postgresql`
- macOS: Homebrew o Postgres.app

**Acceso:**
- `psql -U usuario -d basedatos`
- Comandos: `\l`, `\dt`, `\du`, `\q`

**Configuración:**
- `postgresql.conf` - Configuración del servidor
- `pg_hba.conf` - Autenticación

**Herramientas:**
- `createdb` / `dropdb` - Gestión de BD
- `pg_dump` / `pg_restore` - Backups

**Próximo módulo:** psql avanzado y pgAdmin

---

## Recursos

- **[PostgreSQL Download](https://www.postgresql.org/download/)** - Descarga oficial
- **[PostgreSQL Docs](https://www.postgresql.org/docs/)** - Documentación completa
- **[pg_hba.conf](https://www.postgresql.org/docs/current/auth-pg-hba-conf.html)** - Configuración de autenticación

¡PostgreSQL instalado y listo! 🐘

# psql y Herramientas de PostgreSQL

> **Cliente CLI psql, pgAdmin y otras herramientas gráficas**

---

## psql - Cliente de línea de comandos

**psql** es el cliente interactivo oficial de PostgreSQL.

**Características:**
- ✅ Ejecutar queries SQL
- ✅ Comandos especiales (meta-comandos)
- ✅ Scripts automatizados
- ✅ Salida formateada
- ✅ Historial de comandos

---

## Conectar con psql

### Sintaxis básica

```bash
psql -h host -p puerto -U usuario -d basedatos
```

**Parámetros:**
- `-h` - Host (localhost por defecto)
- `-p` - Puerto (5432 por defecto)
- `-U` - Usuario
- `-d` - Base de datos
- `-W` - Pedir contraseña

### Ejemplos

```bash
# Conectar como postgres a base de datos postgres
psql -U postgres

# Conectar a base de datos específica
psql -U miusuario -d mibasedatos

# Conectar a servidor remoto
psql -h 192.168.1.100 -U usuario -d basedatos

# Con variables de entorno
export PGUSER=miusuario
export PGDATABASE=mibasedatos
psql  # Conecta automáticamente
```

---

## Meta-comandos de psql

**Meta-comandos** empiezan con `\` (backslash).

### Información de bases de datos

```sql
-- Listar bases de datos
\l
\l+  -- Más detalles

-- Conectar a otra BD
\c nombre_bd

-- Ver tamaño de BD actual
\l+ nombre_bd

-- Cambiar a otra BD como otro usuario
\c nueva_bd otro_usuario
```

### Información de tablas

```sql
-- Listar tablas
\dt

-- Listar tablas con detalles
\dt+

-- Listar todas las tablas (incluyendo system)
\dt *.*

-- Describir tabla
\d nombre_tabla

-- Describir con más detalle
\d+ nombre_tabla

-- Solo índices de una tabla
\di nombre_tabla

-- Listar vistas
\dv

-- Listar secuencias
\ds

-- Listar funciones
\df
```

### Información de usuarios y permisos

```sql
-- Listar usuarios (roles)
\du

-- Ver permisos de una tabla
\dp nombre_tabla
\z nombre_tabla  -- Alias de \dp

-- Ver permisos de BD
\l+
```

### Información de schemas

```sql
-- Listar schemas
\dn

-- Ver schema actual
SELECT current_schema();

-- Cambiar schema
SET search_path TO mi_schema;
```

---

## Ejecutar queries

### Queries interactivas

```sql
-- Query simple (terminar con ;)
SELECT * FROM usuarios;

-- Query multilínea
SELECT 
    nombre,
    email
FROM usuarios
WHERE activo = true;

-- Ver último query ejecutado
\g

-- Ejecutar último query y guardar en archivo
\g archivo.txt
```

### Ejecutar archivos SQL

```sql
-- Ejecutar archivo
\i /ruta/archivo.sql

-- Ejecutar con ruta relativa
\i scripts/crear_tablas.sql

-- Ejecutar mostrando comandos
\i script.sql
```

**Ejemplo de archivo SQL:**

```sql
-- crear_tablas.sql
CREATE TABLE usuarios (
    id SERIAL PRIMARY KEY,
    nombre VARCHAR(100),
    email VARCHAR(150) UNIQUE
);

INSERT INTO usuarios (nombre, email) VALUES
    ('Ana', 'ana@email.com'),
    ('Bob', 'bob@email.com');

SELECT * FROM usuarios;
```

Ejecutar:

```bash
psql -U postgres -d mibd -f crear_tablas.sql
```

---

## Formato de salida

### Cambiar formato

```sql
-- Formato alineado (por defecto)
\a

-- Formato HTML
\H

-- Formato sin alinear (CSV-like)
\a

-- Ver formato actual
\pset

-- Formato extendido (vertical)
\x
SELECT * FROM usuarios;
\x  -- Desactivar

-- Formato expandido automático
\x auto
```

### Ejemplo formato vertical

```sql
\x
SELECT * FROM usuarios WHERE id = 1;
```

**Salida:**
```
-[ RECORD 1 ]------------------
id        | 1
nombre    | Ana García
email     | ana@email.com
creado_en | 2024-01-20 10:30:00
```

### Paginación

```sql
-- Activar paginador (less en Linux, more en Windows)
\pset pager on

-- Desactivar paginador
\pset pager off

-- Paginador automático (solo si muchas filas)
\pset pager always
```

### Título y pie de página

```sql
-- Mostrar/ocultar títulos de columnas
\pset tuples_only on  -- Solo datos
\pset tuples_only off  -- Con títulos

-- Borde de tablas
\pset border 0  -- Sin bordes
\pset border 1  -- Bordes simples
\pset border 2  -- Bordes completos
```

---

## Exportar resultados

### A archivo

```sql
-- Redirigir salida
\o resultado.txt
SELECT * FROM usuarios;
\o  -- Volver a STDOUT

-- Copiar tabla a CSV
\copy usuarios TO 'usuarios.csv' CSV HEADER

-- Copiar query a CSV
\copy (SELECT * FROM usuarios WHERE activo = true) TO 'activos.csv' CSV HEADER

-- Importar desde CSV
\copy usuarios FROM 'datos.csv' CSV HEADER
```

### Formato personalizado

```sql
-- Sin encabezados
\copy usuarios TO 'datos.csv' CSV

-- Con delimitador personalizado
\copy usuarios TO 'datos.tsv' (DELIMITER E'\t', HEADER)

-- Solo columnas específicas
\copy usuarios (nombre, email) TO 'emails.csv' CSV HEADER
```

---

## Variables en psql

```sql
-- Definir variable
\set nombre 'Ana García'

-- Usar variable
SELECT * FROM usuarios WHERE nombre = :'nombre';

-- Variable numérica
\set limite 10
SELECT * FROM productos LIMIT :limite;

-- Ver todas las variables
\set

-- Eliminar variable
\unset nombre
```

### Variables especiales

```sql
-- Usuario actual
\echo :USER

-- Base de datos actual
\echo :DBNAME

-- Host actual
\echo :HOST

-- Puerto actual
\echo :PORT

-- Versión de psql
\echo :VERSION
```

---

## Timing de queries

```sql
-- Activar timing
\timing on

SELECT COUNT(*) FROM usuarios;
-- Time: 0.234 ms

-- Desactivar timing
\timing off
```

---

## Transacciones en psql

```sql
-- Iniciar transacción
BEGIN;

-- Queries
INSERT INTO usuarios (nombre, email) VALUES ('Carlos', 'carlos@email.com');
UPDATE usuarios SET activo = true WHERE id = 1;

-- Ver cambios sin commit
SELECT * FROM usuarios;

-- Confirmar
COMMIT;

-- O revertir
ROLLBACK;
```

### Auto-commit

```sql
-- Ver estado de autocommit
\echo :AUTOCOMMIT

-- Desactivar autocommit (cada query es una transacción)
\set AUTOCOMMIT off

-- Activar autocommit (por defecto)
\set AUTOCOMMIT on
```

---

## Historial de comandos

```sql
-- Ver historial
\s

-- Guardar historial en archivo
\s historial.txt

-- Ejecutar comando del historial
-- Usar flechas arriba/abajo

-- Buscar en historial
-- Ctrl + R (en Linux/macOS)
```

**Ubicación del historial:**
- Linux/macOS: `~/.psql_history`
- Windows: `%APPDATA%\postgresql\psql_history`

---

## Scripts útiles

### Script de inicialización (.psqlrc)

**Ubicación:**
- Linux/macOS: `~/.psqlrc`
- Windows: `%APPDATA%\postgresql\psqlrc`

**Ejemplo:**

```sql
-- .psqlrc
\set QUIET 1

-- Configuración
\timing on
\x auto
\pset null '(null)'
\pset border 2
\pset format wrapped

-- Prompt personalizado
\set PROMPT1 '%n@%/%R%# '

-- Shortcuts
\set version 'SELECT version();'
\set tables '\dt'

\set QUIET 0

\echo 'Bienvenido a PostgreSQL!'
\echo 'Usa :version para ver la versión'
```

### Alias útiles

```sql
-- En .psqlrc
\set menu '\\i menu.sql'
\set top10 'SELECT * FROM usuarios ORDER BY id DESC LIMIT 10;'
\set clear '\\! clear'

-- Usar con :menu, :top10, :clear
```

---

## pgAdmin 4

**pgAdmin** es la interfaz gráfica oficial de PostgreSQL.

### Instalación

**Windows/macOS:** Incluido con instalador de PostgreSQL

**Linux:**

```bash
# Ubuntu/Debian
sudo apt install pgadmin4 pgadmin4-desktop

# O versión web
sudo apt install pgadmin4-web
```

### Primer uso

1. **Abrir pgAdmin**
   - Windows: Buscar "pgAdmin 4" en menú inicio
   - Linux: Comando `pgadmin4`
   - URL: http://localhost:5050 (versión web)

2. **Crear master password**
   - Primera vez: Se pide contraseña maestra
   - Guarda credenciales de servidores

3. **Agregar servidor:**
   - Click derecho en "Servers" → "Register" → "Server"
   - **General:**
     - Name: `Local PostgreSQL`
   - **Connection:**
     - Host: `localhost`
     - Port: `5432`
     - Maintenance database: `postgres`
     - Username: `postgres`
     - Password: (tu contraseña)
     - ☑ Save password
   - Click "Save"

---

## Funciones de pgAdmin

### Query Tool

1. Click derecho en BD → "Query Tool"
2. Escribir query
3. F5 o ⚡ para ejecutar

```sql
SELECT * FROM usuarios;
```

**Características:**
- ✅ Autocompletado (Ctrl + Space)
- ✅ Múltiples queries con `;`
- ✅ Ejecutar selección (resaltar + F5)
- ✅ Explicar query (F7)
- ✅ Historial de queries
- ✅ Exportar resultados (CSV, Excel)

### Diseñador visual de tablas

1. Click derecho en "Tables" → "Create" → "Table"
2. **General:**
   - Name: `productos`
   - Owner: `postgres`
3. **Columns:**
   - Click "+" para agregar columnas
   - `id`: integer, NOT NULL, PK
   - `nombre`: varchar(200), NOT NULL
   - `precio`: numeric(10,2)
4. **Constraints:**
   - Primary Key: `id`
   - Check: `precio > 0`
5. Click "Save"

### ERD (Entity Relationship Diagram)

1. Click derecho en BD → "ERD For Database"
2. Muestra relaciones entre tablas
3. Arrastrar tablas para organizar
4. Exportar como imagen

### Backup y Restore

**Backup:**
1. Click derecho en BD → "Backup..."
2. Filename: `backup_20240120.sql`
3. Format: Plain (SQL)
4. Encoding: UTF8
5. Click "Backup"

**Restore:**
1. Click derecho en BD → "Restore..."
2. Seleccionar archivo
3. Click "Restore"

### Monitoreo

**Dashboard:**
- Server Activity
- Database Activity
- Sessions
- Locks
- Prepared Transactions

**Estadísticas:**
- Ver tamaños de tablas
- Queries más lentas
- Conexiones activas

---

## DBeaver (alternativa multiplataforma)

**DBeaver** es un cliente universal para múltiples bases de datos.

### Instalación

- **Descargar:** https://dbeaver.io/download/
- Versión Community (gratis)
- Soporta PostgreSQL, MySQL, SQLite, MongoDB, etc.

### Conectar a PostgreSQL

1. Abrir DBeaver
2. Click en "Nueva Conexión" (cable con +)
3. Seleccionar PostgreSQL
4. **Configuración:**
   - Host: `localhost`
   - Port: `5432`
   - Database: `mibasedatos`
   - Username: `miusuario`
   - Password: (tu contraseña)
   - ☑ Save password
5. Test Connection
6. Finish

### Características de DBeaver

- ✅ Editor SQL con autocompletado
- ✅ ER Diagram automático
- ✅ Exportar/Importar datos (CSV, JSON, XML, Excel)
- ✅ Comparar estructuras de BD
- ✅ Query History
- ✅ Data Editor visual
- ✅ SSH Tunneling
- ✅ Dark mode

---

## Comparación de herramientas

| Característica | psql | pgAdmin | DBeaver |
|---|---|---|---|
| **Tipo** | CLI | GUI (Web) | GUI (Desktop) |
| **Curva aprendizaje** | Alta | Media | Baja |
| **Velocidad** | ⚡⚡⚡ | ⚡⚡ | ⚡⚡ |
| **Scripts** | ✅ Excelente | ❌ | ✅ |
| **Visual** | ❌ | ✅ | ✅ |
| **Multiplataforma** | ✅ | ✅ | ✅ |
| **Multi-BD** | ❌ Solo PostgreSQL | ❌ Solo PostgreSQL | ✅ Todas |
| **SSH Tunnel** | Manual | ✅ | ✅ |
| **Autocomplete** | ❌ | ✅ | ✅ |
| **ER Diagram** | ❌ | ✅ | ✅ |

**Recomendación:**
- **psql** - Administración rápida, scripts, CI/CD
- **pgAdmin** - Administración PostgreSQL completa
- **DBeaver** - Desarrollo, múltiples BD

---

## Caso práctico: Workflow completo

### 1. Crear BD y estructura (psql)

```bash
psql -U postgres
```

```sql
-- Crear BD
CREATE DATABASE ecommerce;

-- Conectar
\c ecommerce

-- Crear tablas
CREATE TABLE categorias (
    id SERIAL PRIMARY KEY,
    nombre VARCHAR(100) NOT NULL UNIQUE
);

CREATE TABLE productos (
    id SERIAL PRIMARY KEY,
    nombre VARCHAR(200) NOT NULL,
    descripcion TEXT,
    precio NUMERIC(10, 2) NOT NULL CHECK (precio > 0),
    stock INTEGER DEFAULT 0 CHECK (stock >= 0),
    categoria_id INTEGER REFERENCES categorias(id) ON DELETE SET NULL,
    activo BOOLEAN DEFAULT true,
    creado_en TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_productos_categoria ON productos(categoria_id);
CREATE INDEX idx_productos_activo ON productos(activo);

-- Verificar
\dt+

-- Salir
\q
```

### 2. Insertar datos de prueba (pgAdmin)

1. Abrir pgAdmin
2. Conectar a `ecommerce`
3. Query Tool:

```sql
INSERT INTO categorias (nombre) VALUES
    ('Electrónica'),
    ('Ropa'),
    ('Hogar'),
    ('Deportes');

INSERT INTO productos (nombre, descripcion, precio, stock, categoria_id) VALUES
    ('Laptop Dell XPS', 'Laptop profesional 16GB RAM', 1299.99, 10, 1),
    ('Mouse Logitech', 'Mouse inalámbrico', 25.99, 50, 1),
    ('Camisa Polo', 'Camisa 100% algodón', 39.99, 30, 2),
    ('Pantalón Jean', 'Jean clásico', 49.99, 25, 2),
    ('Mesa Escritorio', 'Mesa de madera', 199.99, 5, 3),
    ('Balón Fútbol', 'Balón oficial', 29.99, 20, 4);
```

### 3. Consultas (psql con timing)

```bash
psql -U postgres -d ecommerce
```

```sql
\timing on

-- Productos por categoría
SELECT 
    c.nombre AS categoria,
    COUNT(p.id) AS total_productos,
    AVG(p.precio) AS precio_promedio,
    SUM(p.stock) AS stock_total
FROM categorias c
LEFT JOIN productos p ON c.id = p.categoria_id
GROUP BY c.id, c.nombre
ORDER BY total_productos DESC;

-- Exportar a CSV
\copy (SELECT c.nombre AS categoria, p.nombre AS producto, p.precio, p.stock FROM categorias c INNER JOIN productos p ON c.id = p.categoria_id ORDER BY c.nombre, p.nombre) TO 'inventario.csv' CSV HEADER
```

### 4. Backup (terminal)

```bash
# Backup completo
pg_dump -U postgres -d ecommerce -f backup_ecommerce_$(date +%Y%m%d).sql

# Backup solo datos
pg_dump -U postgres -d ecommerce --data-only -f datos_ecommerce.sql

# Backup comprimido
pg_dump -U postgres -d ecommerce -Fc -f ecommerce.dump
```

### 5. Análisis visual (DBeaver)

1. Conectar a `ecommerce` en DBeaver
2. ER Diagram:
   - Click derecho en `ecommerce` → "View Diagram"
   - Ver relaciones entre tablas
3. Data Editor:
   - Doble click en `productos`
   - Editar filas directamente
   - Filtros visuales
4. Exportar:
   - Click derecho en resultados → "Export Data"
   - Elegir formato (Excel, JSON, etc.)

---

## Scripts de mantenimiento

### Vacuum y Analyze

```sql
-- Limpiar espacio muerto
VACUUM productos;

-- Vacuum completo (más lento)
VACUUM FULL productos;

-- Actualizar estadísticas
ANALYZE productos;

-- Ambos
VACUUM ANALYZE productos;

-- Todo la BD
VACUUM ANALYZE;
```

### Reindexar

```sql
-- Reindexar tabla
REINDEX TABLE productos;

-- Reindexar índice específico
REINDEX INDEX idx_productos_categoria;

-- Reindexar toda la BD
REINDEX DATABASE ecommerce;
```

---

## Tips y trucos

### 1. Atajos de teclado en psql

- `Ctrl + L` - Limpiar pantalla
- `Ctrl + C` - Cancelar query
- `Ctrl + D` - Salir (EOF)
- `↑` / `↓` - Historial
- `Tab` - Autocompletar nombres de tablas

### 2. Queries útiles guardadas

```sql
-- En .psqlrc
\set tablas 'SELECT tablename FROM pg_tables WHERE schemaname = \'public\';'
\set tamanio 'SELECT pg_size_pretty(pg_database_size(current_database()));'
\set conexiones 'SELECT count(*) FROM pg_stat_activity;'

-- Usar: :tablas, :tamanio, :conexiones
```

### 3. Prompt personalizado

```sql
-- En .psqlrc
\set PROMPT1 '%[%033[1;32m%]%n@%/%[%033[0m%]%R%# '
-- Resultado: usuario@basedatos=#
```

### 4. Ejecutar comando shell desde psql

```sql
-- Linux/macOS
\! ls -la

-- Windows
\! dir

-- Limpiar pantalla
\! clear  -- Linux/macOS
\! cls    -- Windows
```

---

## Resumen

**psql:**
- Cliente CLI oficial
- Meta-comandos: `\l`, `\dt`, `\d`, `\du`, `\c`
- Ejecutar archivos: `\i archivo.sql`
- Exportar: `\copy tabla TO 'archivo.csv' CSV HEADER`
- Variables: `\set nombre valor`, usar con `:nombre`

**pgAdmin:**
- GUI oficial de PostgreSQL
- Query Tool, diseñador visual, ER diagrams
- Backup/Restore gráfico
- Monitoreo y estadísticas

**DBeaver:**
- Cliente universal multi-BD
- Interfaz intuitiva
- Exportar a múltiples formatos
- ER Diagram automático

**Próximo módulo:** Tipos de datos avanzados en PostgreSQL

---

## Recursos

- **[psql Documentation](https://www.postgresql.org/docs/current/app-psql.html)** - Manual completo
- **[pgAdmin](https://www.pgadmin.org/)** - Web oficial
- **[DBeaver](https://dbeaver.io/)** - Descarga y docs

¡Dominas las herramientas de PostgreSQL! 🛠️

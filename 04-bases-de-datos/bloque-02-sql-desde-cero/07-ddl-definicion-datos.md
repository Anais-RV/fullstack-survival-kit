# DDL - Data Definition Language

> **Crear, modificar y eliminar estructuras de base de datos**

---

## ¿Qué es DDL?

**DDL** (Data Definition Language) son los comandos SQL para **definir la estructura** de la base de datos:

```
DDL
├── CREATE    (Crear)
├── ALTER     (Modificar)
├── DROP      (Eliminar)
└── TRUNCATE  (Vaciar)
```

**No manipulan datos, manipulan estructura.**

---

## CREATE TABLE

Crear una nueva tabla.

### Sintaxis básica

```sql
CREATE TABLE nombre_tabla (
    columna1 tipo restricciones,
    columna2 tipo restricciones,
    ...
);
```

### Ejemplo simple

```sql
CREATE TABLE usuarios (
    id INTEGER,
    nombre TEXT,
    email TEXT,
    edad INTEGER
);
```

---

### PRIMARY KEY

Identificador único de cada fila.

```sql
CREATE TABLE usuarios (
    id INTEGER PRIMARY KEY,  -- ← clave primaria
    nombre TEXT,
    email TEXT
);
```

**En SQLite:**
```sql
-- Autoincrementar (genera IDs automáticamente)
CREATE TABLE usuarios (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    nombre TEXT
);
```

---

### NOT NULL

Columna obligatoria.

```sql
CREATE TABLE usuarios (
    id INTEGER PRIMARY KEY,
    nombre TEXT NOT NULL,      -- ← obligatorio
    email TEXT NOT NULL,       -- ← obligatorio
    telefono TEXT              -- ← opcional
);
```

**Prueba:**
```sql
-- ✅ Válido
INSERT INTO usuarios (id, nombre, email) 
VALUES (1, 'Ana', 'ana@example.com');

-- ❌ ERROR: nombre no puede ser NULL
INSERT INTO usuarios (id, email) 
VALUES (2, 'bob@example.com');
```

---

### UNIQUE

Sin duplicados.

```sql
CREATE TABLE usuarios (
    id INTEGER PRIMARY KEY,
    email TEXT UNIQUE  -- ← no se repite
);
```

**Prueba:**
```sql
INSERT INTO usuarios (id, email) VALUES (1, 'ana@example.com');

-- ❌ ERROR: email duplicado
INSERT INTO usuarios (id, email) VALUES (2, 'ana@example.com');
```

---

### DEFAULT

Valor por defecto.

```sql
CREATE TABLE usuarios (
    id INTEGER PRIMARY KEY,
    nombre TEXT NOT NULL,
    activo INTEGER DEFAULT 1,           -- ← por defecto: 1 (true)
    fecha_registro DATE DEFAULT CURRENT_DATE  -- ← fecha actual
);
```

**Prueba:**
```sql
-- No especificamos activo ni fecha_registro
INSERT INTO usuarios (id, nombre) VALUES (1, 'Ana');

SELECT * FROM usuarios;
-- +----+--------+--------+------------------+
-- | id | nombre | activo | fecha_registro   |
-- +----+--------+--------+------------------+
-- |  1 | Ana    |      1 | 2024-01-20       |
-- +----+--------+--------+------------------+
```

---

### CHECK

Validación personalizada.

```sql
CREATE TABLE productos (
    id INTEGER PRIMARY KEY,
    nombre TEXT NOT NULL,
    precio REAL CHECK (precio > 0),     -- ← debe ser positivo
    stock INTEGER CHECK (stock >= 0)    -- ← no negativo
);
```

**Prueba:**
```sql
-- ✅ Válido
INSERT INTO productos (id, nombre, precio, stock)
VALUES (1, 'Laptop', 1000, 10);

-- ❌ ERROR: precio debe ser > 0
INSERT INTO productos (id, nombre, precio, stock)
VALUES (2, 'Mouse', -50, 5);
```

---

### FOREIGN KEY

Referencia a otra tabla.

```sql
CREATE TABLE usuarios (
    id INTEGER PRIMARY KEY,
    nombre TEXT
);

CREATE TABLE posts (
    id INTEGER PRIMARY KEY,
    usuario_id INTEGER,
    titulo TEXT,
    FOREIGN KEY (usuario_id) REFERENCES usuarios(id)
);
```

**Prueba:**
```sql
INSERT INTO usuarios (id, nombre) VALUES (1, 'Ana');

-- ✅ Válido (usuario 1 existe)
INSERT INTO posts (id, usuario_id, titulo) 
VALUES (1, 1, 'Mi primer post');

-- ❌ ERROR: usuario 999 no existe
INSERT INTO posts (id, usuario_id, titulo) 
VALUES (2, 999, 'Post');
```

---

### Restricciones combinadas

```sql
CREATE TABLE usuarios (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    nombre TEXT NOT NULL,
    email TEXT UNIQUE NOT NULL,
    edad INTEGER CHECK (edad >= 18),
    activo INTEGER DEFAULT 1,
    fecha_registro DATETIME DEFAULT CURRENT_TIMESTAMP
);
```

---

## Tipos de datos en SQLite

### Numéricos

```sql
INTEGER        -- Enteros: 1, 2, 100
REAL           -- Decimales: 19.99, 3.14
```

**Nota:** SQLite es flexible con tipos.

### Texto

```sql
TEXT           -- Cualquier texto
VARCHAR(100)   -- Tratado como TEXT en SQLite
CHAR(5)        -- Tratado como TEXT en SQLite
```

### Fecha y hora

```sql
DATE           -- Almacenado como TEXT (YYYY-MM-DD)
DATETIME       -- Almacenado como TEXT (YYYY-MM-DD HH:MM:SS)
TIMESTAMP      -- Similar a DATETIME
```

**SQLite no tiene tipo DATE nativo**, usa TEXT o INTEGER.

### Booleano

```sql
-- SQLite no tiene BOOLEAN, usa INTEGER
activo INTEGER  -- 0 = false, 1 = true
```

### Blob

```sql
BLOB           -- Datos binarios (imágenes, archivos)
```

---

## Ejemplo completo: E-commerce

```sql
-- Tabla: categorias
CREATE TABLE categorias (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    nombre TEXT UNIQUE NOT NULL,
    descripcion TEXT
);

-- Tabla: productos
CREATE TABLE productos (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    nombre TEXT NOT NULL,
    descripcion TEXT,
    precio REAL CHECK (precio > 0),
    stock INTEGER CHECK (stock >= 0) DEFAULT 0,
    categoria_id INTEGER,
    activo INTEGER DEFAULT 1,
    fecha_creacion DATETIME DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (categoria_id) REFERENCES categorias(id)
);

-- Tabla: clientes
CREATE TABLE clientes (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    nombre TEXT NOT NULL,
    email TEXT UNIQUE NOT NULL,
    telefono TEXT,
    direccion TEXT,
    fecha_registro DATE DEFAULT CURRENT_DATE
);

-- Tabla: pedidos
CREATE TABLE pedidos (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    cliente_id INTEGER NOT NULL,
    fecha DATETIME DEFAULT CURRENT_TIMESTAMP,
    total REAL CHECK (total >= 0),
    estado TEXT DEFAULT 'pendiente',
    FOREIGN KEY (cliente_id) REFERENCES clientes(id)
);

-- Tabla: items_pedido
CREATE TABLE items_pedido (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    pedido_id INTEGER NOT NULL,
    producto_id INTEGER NOT NULL,
    cantidad INTEGER CHECK (cantidad > 0),
    precio_unitario REAL CHECK (precio_unitario > 0),
    FOREIGN KEY (pedido_id) REFERENCES pedidos(id),
    FOREIGN KEY (producto_id) REFERENCES productos(id)
);
```

---

## ALTER TABLE

Modificar tabla existente.

### Agregar columna

```sql
ALTER TABLE usuarios
ADD COLUMN telefono TEXT;
```

### Renombrar tabla

```sql
ALTER TABLE usuarios
RENAME TO users;
```

### Renombrar columna

```sql
ALTER TABLE usuarios
RENAME COLUMN nombre TO nombre_completo;
```

### Eliminar columna

```sql
-- ⚠️ SQLite no soporta DROP COLUMN directamente
-- Necesitas recrear la tabla

-- Alternativa: crear nueva tabla sin esa columna
CREATE TABLE usuarios_nueva (
    id INTEGER PRIMARY KEY,
    nombre TEXT
    -- sin columna edad
);

INSERT INTO usuarios_nueva SELECT id, nombre FROM usuarios;
DROP TABLE usuarios;
ALTER TABLE usuarios_nueva RENAME TO usuarios;
```

---

## DROP TABLE

Eliminar tabla **completamente**.

```sql
DROP TABLE nombre_tabla;
```

**Ejemplo:**
```sql
DROP TABLE usuarios;
-- Tabla eliminada (estructura y datos)
```

**⚠️ CUIDADO:** No hay confirmación, es irreversible.

---

### DROP IF EXISTS

Evitar error si la tabla no existe.

```sql
DROP TABLE IF EXISTS usuarios;
-- No error si no existe
```

---

## TRUNCATE TABLE

Vaciar tabla (eliminar todos los datos, mantener estructura).

**⚠️ SQLite no soporta TRUNCATE.**

**Alternativa:**
```sql
DELETE FROM usuarios;
-- Vacía la tabla
```

---

## CREATE INDEX

Acelerar búsquedas.

```sql
CREATE INDEX nombre_indice ON tabla(columna);
```

**Ejemplo:**
```sql
-- Índice en email para búsquedas rápidas
CREATE INDEX idx_email ON usuarios(email);

-- Búsqueda rápida
SELECT * FROM usuarios WHERE email = 'ana@example.com';
```

### Índice único

```sql
CREATE UNIQUE INDEX idx_email ON usuarios(email);
-- Combina índice + restricción UNIQUE
```

### Índice compuesto

```sql
-- Índice en múltiples columnas
CREATE INDEX idx_nombre_edad ON usuarios(nombre, edad);
```

---

## DROP INDEX

Eliminar índice.

```sql
DROP INDEX nombre_indice;
```

**Ejemplo:**
```sql
DROP INDEX idx_email;
```

---

## Crear tabla desde otra (CREATE TABLE AS)

```sql
CREATE TABLE usuarios_backup AS
SELECT * FROM usuarios;
```

**Copia estructura y datos.**

---

## Crear tabla temporal

```sql
CREATE TEMP TABLE temp_usuarios (
    id INTEGER,
    nombre TEXT
);

-- Se elimina al cerrar conexión
```

---

## Ver estructura de tabla

### SQLite

```sql
.schema usuarios
```

**Salida:**
```sql
CREATE TABLE usuarios (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    nombre TEXT NOT NULL,
    email TEXT UNIQUE NOT NULL,
    edad INTEGER
);
```

### Listar todas las tablas

```sql
.tables
```

---

## Restricciones a nivel de tabla

### PRIMARY KEY compuesta

```sql
CREATE TABLE inscripciones (
    estudiante_id INTEGER,
    curso_id INTEGER,
    fecha_inscripcion DATE,
    PRIMARY KEY (estudiante_id, curso_id)  -- ← clave compuesta
);
```

### UNIQUE compuesto

```sql
CREATE TABLE reservas (
    usuario_id INTEGER,
    fecha DATE,
    hora TIME,
    UNIQUE (fecha, hora)  -- ← combinación única
);
```

---

## ON DELETE y ON UPDATE

Comportamiento cuando se elimina/actualiza fila referenciada.

```sql
CREATE TABLE posts (
    id INTEGER PRIMARY KEY,
    usuario_id INTEGER,
    titulo TEXT,
    FOREIGN KEY (usuario_id) REFERENCES usuarios(id)
        ON DELETE CASCADE      -- ← elimina posts si eliminas usuario
        ON UPDATE CASCADE      -- ← actualiza usuario_id si cambias id
);
```

**Opciones:**
- `CASCADE` - Propaga cambio
- `SET NULL` - Pone NULL
- `RESTRICT` - Impide cambio (default)
- `NO ACTION` - No hace nada

**Ejemplo:**
```sql
-- usuario_id = 1
INSERT INTO posts (id, usuario_id, titulo) VALUES (1, 1, 'Post 1');

-- Eliminar usuario
DELETE FROM usuarios WHERE id = 1;

-- Con CASCADE: post también se elimina
-- Con RESTRICT: error (no puedes eliminar usuario con posts)
-- Con SET NULL: usuario_id del post se pone NULL
```

---

## Ejemplo práctico: Blog

```sql
-- Crear base de datos
sqlite3 blog.db

-- Configurar formato
.mode column
.headers on

-- Tabla: usuarios
CREATE TABLE usuarios (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    nombre TEXT NOT NULL,
    email TEXT UNIQUE NOT NULL,
    password TEXT NOT NULL,
    bio TEXT,
    fecha_registro DATETIME DEFAULT CURRENT_TIMESTAMP
);

-- Tabla: categorias
CREATE TABLE categorias (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    nombre TEXT UNIQUE NOT NULL,
    slug TEXT UNIQUE NOT NULL
);

-- Tabla: posts
CREATE TABLE posts (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    usuario_id INTEGER NOT NULL,
    categoria_id INTEGER,
    titulo TEXT NOT NULL,
    contenido TEXT NOT NULL,
    fecha_creacion DATETIME DEFAULT CURRENT_TIMESTAMP,
    fecha_actualizacion DATETIME,
    publicado INTEGER DEFAULT 0,
    FOREIGN KEY (usuario_id) REFERENCES usuarios(id) ON DELETE CASCADE,
    FOREIGN KEY (categoria_id) REFERENCES categorias(id)
);

-- Tabla: comentarios
CREATE TABLE comentarios (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    post_id INTEGER NOT NULL,
    usuario_id INTEGER NOT NULL,
    texto TEXT NOT NULL,
    fecha DATETIME DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (post_id) REFERENCES posts(id) ON DELETE CASCADE,
    FOREIGN KEY (usuario_id) REFERENCES usuarios(id)
);

-- Tabla: etiquetas
CREATE TABLE etiquetas (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    nombre TEXT UNIQUE NOT NULL
);

-- Tabla intermedia: posts ↔ etiquetas (N:M)
CREATE TABLE posts_etiquetas (
    post_id INTEGER,
    etiqueta_id INTEGER,
    PRIMARY KEY (post_id, etiqueta_id),
    FOREIGN KEY (post_id) REFERENCES posts(id) ON DELETE CASCADE,
    FOREIGN KEY (etiqueta_id) REFERENCES etiquetas(id) ON DELETE CASCADE
);

-- Índices para búsquedas rápidas
CREATE INDEX idx_posts_usuario ON posts(usuario_id);
CREATE INDEX idx_posts_categoria ON posts(categoria_id);
CREATE INDEX idx_comentarios_post ON comentarios(post_id);
CREATE INDEX idx_email ON usuarios(email);

-- Ver estructura
.schema
```

---

## Buenas prácticas

### ✅ Siempre PRIMARY KEY

```sql
CREATE TABLE usuarios (
    id INTEGER PRIMARY KEY AUTOINCREMENT,  -- ← siempre
    ...
);
```

### ✅ NOT NULL en campos críticos

```sql
CREATE TABLE usuarios (
    nombre TEXT NOT NULL,
    email TEXT NOT NULL
);
```

### ✅ UNIQUE en campos únicos

```sql
CREATE TABLE usuarios (
    email TEXT UNIQUE
);
```

### ✅ CHECK para validaciones

```sql
CREATE TABLE productos (
    precio REAL CHECK (precio > 0),
    stock INTEGER CHECK (stock >= 0)
);
```

### ✅ DEFAULT para valores comunes

```sql
CREATE TABLE usuarios (
    activo INTEGER DEFAULT 1,
    fecha_registro DATE DEFAULT CURRENT_DATE
);
```

### ✅ Índices en foreign keys

```sql
CREATE INDEX idx_post_usuario ON posts(usuario_id);
```

### ✅ ON DELETE CASCADE para dependencias

```sql
FOREIGN KEY (usuario_id) REFERENCES usuarios(id) ON DELETE CASCADE
```

---

## Errores comunes

### ❌ Olvidar PRIMARY KEY

```sql
-- MAL
CREATE TABLE usuarios (
    nombre TEXT
);

-- BIEN
CREATE TABLE usuarios (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    nombre TEXT
);
```

### ❌ No validar datos

```sql
-- MAL
CREATE TABLE productos (
    precio REAL  -- puede ser negativo!
);

-- BIEN
CREATE TABLE productos (
    precio REAL CHECK (precio > 0)
);
```

### ❌ No definir relaciones

```sql
-- MAL
CREATE TABLE posts (
    usuario_id INTEGER  -- sin FOREIGN KEY
);

-- BIEN
CREATE TABLE posts (
    usuario_id INTEGER,
    FOREIGN KEY (usuario_id) REFERENCES usuarios(id)
);
```

---

## Migración: Modificar tabla con datos

**Problema:** SQLite no soporta todas las operaciones ALTER.

**Solución:** Recrear tabla.

```sql
-- 1. Crear tabla nueva con cambios
CREATE TABLE usuarios_nueva (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    nombre TEXT NOT NULL,
    email TEXT UNIQUE NOT NULL,
    telefono TEXT  -- ← columna nueva
);

-- 2. Copiar datos
INSERT INTO usuarios_nueva (id, nombre, email)
SELECT id, nombre, email FROM usuarios;

-- 3. Eliminar tabla vieja
DROP TABLE usuarios;

-- 4. Renombrar nueva tabla
ALTER TABLE usuarios_nueva RENAME TO usuarios;
```

---

## Resumen

**DDL:**
- `CREATE TABLE` - Crear estructura
- `ALTER TABLE` - Modificar estructura
- `DROP TABLE` - Eliminar tabla
- `CREATE INDEX` - Acelerar búsquedas

**Restricciones:**
- `PRIMARY KEY` - Identificador único
- `FOREIGN KEY` - Relación entre tablas
- `UNIQUE` - Sin duplicados
- `NOT NULL` - Obligatorio
- `CHECK` - Validación personalizada
- `DEFAULT` - Valor por defecto

**Opciones CASCADE:**
- `ON DELETE CASCADE` - Elimina filas relacionadas
- `ON UPDATE CASCADE` - Actualiza referencias

**Próximo módulo:** DML - Insertar, actualizar y eliminar datos

---

## Ejercicios

### Ejercicio 1

Crear tabla `productos` con:
- id (PRIMARY KEY, autoincrementar)
- nombre (obligatorio)
- precio (debe ser > 0)
- stock (no negativo, default 0)

**Solución:**
```sql
CREATE TABLE productos (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    nombre TEXT NOT NULL,
    precio REAL CHECK (precio > 0),
    stock INTEGER CHECK (stock >= 0) DEFAULT 0
);
```

---

### Ejercicio 2

Agregar columna `descripcion` a tabla `productos`.

**Solución:**
```sql
ALTER TABLE productos
ADD COLUMN descripcion TEXT;
```

---

### Ejercicio 3

Crear tabla `pedidos` relacionada con `clientes`:
- id (PRIMARY KEY)
- cliente_id (FOREIGN KEY a clientes)
- total (debe ser >= 0)
- Si eliminas cliente, elimina pedidos (CASCADE)

**Solución:**
```sql
CREATE TABLE pedidos (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    cliente_id INTEGER NOT NULL,
    total REAL CHECK (total >= 0),
    FOREIGN KEY (cliente_id) REFERENCES clientes(id) ON DELETE CASCADE
);
```

---

## Recursos

- **[SQLite Data Types](https://www.sqlite.org/datatype3.html)** - Tipos de datos
- **[SQLite Constraints](https://www.sqlite.org/lang_createtable.html)** - Restricciones
- **[CREATE TABLE](https://www.sqlite.org/lang_createtable.html)** - Documentación oficial

Ahora sabes **crear y modificar estructuras** de base de datos. 🏗️
